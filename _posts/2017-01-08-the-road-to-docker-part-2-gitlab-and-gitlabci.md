---
layout: post
title: "The Road to Docker Part 2: GitLab and GitLabCI"
summary:
hn-discussion:
---

# Last time
To refresh our memory, [last time](/2016/12/29/the-road-to-docker-part-1-vcs-and-ci/) we started talking about improving the day to day development process using specific tools and technologies that can ease and improve the work flow and communication around code reviews, branching, and merging. We also discussed continuous integration and wanting it to be well connected to the version control system (Git in our case) using an elastic set up that was easy to configure and maintain for multiple repositories and branches.

We discussed Docker, Docker Compose, Docker Swarm, Jenkins 2, Selenium Grid, and GitLab(CI).

# Introduction
In this episode, we'll talk about setting up GitLab and GitLabCI using Docker and some of the interesting discoveries I ran into along the way. I started off by catching up on GitLab and GitLabCI. The last time I installed them and actively used them was almost 2 years ago. I was happy to see that they have matured a lot since then. Also, last time I used them, I installed them on the machine itself without the use of VMs or Docker. This time around, everything is going to be ran via Docker.

# GitLab
To begin, I installed GitLab locally via docker-compose, and I tested it out by having it import one of our bigger code repositories. This is the documentation I followed to get up and running:

[https://docs.gitlab.com/omnibus/docker/](https://docs.gitlab.com/omnibus/docker/)

The next step was looking more at GitLabCI and figuring out how it worked. After looking at Jenkins 2 and other CI systems that I have used in the past on some of my projects, I was glad to find out about GitLabCI's `.gitlab-ci.yml` file. 

> "The .gitlab-ci.yml file tells the GitLab runner what to do. By default it runs a pipeline with three stages: build, test, and deploy. You don't need to use all three stages; stages with no jobs are simply ignored."

I started here and made my way through these docs pretty easily.

[https://docs.gitlab.com/ce/ci/quick_start/](https://docs.gitlab.com/ce/ci/quick_start/)

Then I quickly had to learn more about GitLab CI runners, so I looked here:

[https://docs.gitlab.com/ce/ci/runners/README.html](https://docs.gitlab.com/ce/ci/runners/README.html)

and here:

[https://docs.gitlab.com/runner/install/](https://docs.gitlab.com/runner/install/)

then here:

[https://docs.gitlab.com/runner/install/docker.html](https://docs.gitlab.com/runner/install/docker.html)

The main decision points were around how I wanted to run CI if I had different projects to handle, and how the runners (think building and running tests) would be shared, or not, amongst the projects. Also, how would the runners be spawned and what are they running exactly? My goal was to start with something simple that I could build upon. The first target was to have GitLabCI use Docker to spin up a container and use it for building and running tests. The above documentation showed how to do that, and although it involved a manual process to get the runner registered and authenticated with the instance of GitLab it was pretty straight forward overall. The intersting bit was around choosing the Docker image that would be spun up (I started with openjdk:8 since it made sense for me) and how GitLab was going to spin it up. In a nutshell, I wanted GitLab to use my existing Docker daemon to do this, so I gave it access to my `/var/run/docker.sock` in order to do that.

After I spent some time reading and learning how the above work using hand-crafted Docker commands, I decided to use `docker-compose` again to set up GitLab and the GitLabCI runner.

This is how my compose file looked like initially.

    gitlab:
      image: gitlab/gitlab-ce:latest
      dns: ${DOCKER_DNS_SERVER}
      environment:
        GITLAB_OMNIBUS_CONFIG: |
          external_url '${GITLAB_EXTERNAL_URL}'
          gitlab_rails['gitlab_shell_ssh_port'] = ${GITLAB_SHELL_SSH_PORT}
      ports:
        - '80:80'
        - '${GITLAB_SHELL_SSH_PORT}:22'
      volumes:
        - './volumes/config:/etc/gitlab'
        - './volumes/logs:/var/log/gitlab'
        - './volumes/data:/var/opt/gitlab'
     
    gitlab-runner:
      image:   gitlab/gitlab-runner:latest
      dns: ${DOCKER_DNS_SERVER}
      volumes:
        - '/var/run/docker.sock:/var/run/docker.sock'
        - './volumes/gitlab-runner/config:/etc/gitlab-runner'

With that being done, I had them both working and seeing one another (of course, I had to do some manual set up work to register the runner after it was created, as mentioned in the docs above). Note that I decided to make use of `DOCKER_DNS_SERVER` since I am working with an internal DNS server and I wanted to make sure the containers could resolve other services and hosts in the system.

# GitLab CI YAML config
The next step was to add a `.gitlab-ci.yml` into the the Git code repository to replicate what I had previously done with Jenkins and the Jenkinsfile. I added the file and pushed it up to invoke the CI pipeline.
 
After that, I saw the following error in the GitLab logs:

    Running with gitlab-ci-multi-runner 1.9.2 (ade6572)
    Using Docker executor with image openjdk:8 ...
    Pulling docker image openjdk:8 ...
    Running on runner-d9c99cb0-project-2-concurrent-0 via 84ad585835b6...
    Cloning repository...
    Cloning into '/builds/acme/webapp'...
    fatal: unable to access 'http://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@gitlab/acme/webapp.git/': Couldn't resolve host 'gitlab'
    ERROR: Build failed: exit code 1

Which is good, and bad. Good because it means that it is trying to create a Docker container using openjdk:8 as I specified. The bad news is that the on the fly created Docker containers doesn't have the proper DNS configuration so it failed to clone the code.

After consulting with the docs again, this was useful:

[https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/blob/master/docs/configuration/advanced-configuration.md#the-runnersdocker-section](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/blob/master/docs/configuration/advanced-configuration.md#the-runnersdocker-section)

Specifically:

    dns:    a list of DNS servers for the container to use

That helped. After making this change and restarting the runner, I saw it spinning the Docker container on the fly as expected. `docker ps` to verify real quick.

    CONTAINER ID IMAGE         COMMAND               CREATED            STATUS             PORTS  NAMES
    111c584cf48d a3c31f55e86b  "gitlab-runner-build" About a minute ago Up About a minute         runner-d9c99cb0-project-2-concurrent-0-predefined

And the GitLab logs look much better (keep in mind, this is a pristine stock openjdk:8 Docker image, so it has nothing else, no environment variables needed for the build, etc.).

    Running with gitlab-ci-multi-runner 1.9.2 (ade6572)
    Using Docker executor with image openjdk:8 ...
    Pulling docker image openjdk:8 ...
    Running on runner-d9c99cb0-project-2-concurrent-0 via 84ad585835b6...
    Cloning repository...
    Cloning into '/builds/acme/webapp'...
    Checking out ba040066 as master...
    $ ./sbt clean compile
    Downloading sbt launcher 0.13.7:
      From  https://repo.typesafe.com/typesafe/ivy-releases/org.scala-sbt/sbt-launch/0.13.7/sbt-launch.jar
        To  /root/.sbt/launchers/0.13.7/sbt-launch.jar
    Getting org.scala-sbt sbt 0.13.7 ...
    
    but then it failed with the following:
    
    Caused by: java.util.NoSuchElementException: key not found: WEBAPP_HOME
        at scala.collection.MapLike$class.default(MapLike.scala:228)

Which meant I had to export that environment variable. 

Next up, I wanted to see what would happen if I make another commit. Specifically, I wanted to see if it was going to re-clone the repo or use a volume cache for the on the fly Docker container.

This is what I got when I pushed up again.

    Running with gitlab-ci-multi-runner 1.9.2 (ade6572)
    Using Docker executor with image openjdk:8 ...
    Pulling docker image openjdk:8 ...
    Running on runner-d9c99cb0-project-2-concurrent-0 via 84ad585835b6...
    Fetching changes...
    Removing project/project/
    Removing project/target/
    HEAD is now at ba040066dc [web-tests-2.0] Added a space between - and first char.
    From http://xxxxxxxx/acme/webapp
     + ba040066dc...e68818a035 master -> origin/master  (forced update)
    Checking out e68818a0 as master...

Clearly, it did not re-clone. It fetched.

    [success] Total time: 857 s, completed Jan 7, 2017 10:08:17 PM
    Build succeeded

The other optimization I wanted to have was caching of the ~/.sbt and ~/.ivy2 directories, so I added a volume on the runner's config for Docker. After which, this is what the build times looked like:

    Status    Build ID  Time
    passed    #7         02:56 
    passed    #6       17:21 
    passed    #4       19:46

So the caching made a difference.

With the above being done, the basic wiring was in place for GitLabCI to compile our branches. The next step was to be able to package applications up and run tests. For the latter two, the Docker images that the runners use must have certain system tools installed. I decided to build a few custom Docker images to facilitate this, which also meant I had to build a private Docker registry as well and allow GitLab to use it. Both, and more topics for next time's post.
