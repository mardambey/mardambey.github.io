---
layout: post
title: "The Road to Docker Part 1: VCS and CI"
summary:
hn-discussion:
tags:
- docker
- git
- continuous integration
- selenium
- jenkins
---

# What is this series about?
As the year neared it's end, we had a quieter time in the office. So, I started thinking with my engineering team about what we wanted to change in our day to day work in order to make improvements and efficiencies. After some chats, we decided that we wanted to work on our code reviews, CI, and increase visibility and communication around branching and merging. So far, we have been using Git through via the command line, and performing code reviews through email. Most of us have been using GitHub or GitLab, Jenkins, GitLabCI, and other similar tools on our own or on other projects. As such, we decided to experiment with various tools and find what worked for us the most.

During this blog series, I'll walk you guys through our thought process and decisions and I'll share with you as much as possible regarding configuration snippets and resources.

# 'Tis the season
Over the holidays I began to experiment and learn about whats new and happening in the world of CI with relation to distributed build infrastructure. I looked into the following areas and tools, either because I thought they were interesting and useful in order to accomplish our goals, or because I ran across them in dicussions with the team. 

# Portability, packaging, distribution 
For portability, packaging, and distribution I like to use Docker to run most services and `docker-compose`to define services and environments. Docker is useful for application packaging and distribution as well as for prototyping, testing,and finally deploying the tools that we end up using to manage our Git repositories, perform code revies, and track branches and merges. With Docker, I can build most, if not all, of the CI stack on my personal machine, and deploy it onto dev, QA, and other environments easily. 

# Version Control System
Version control (VCS) and a clear usage policy are essential to any kind of CI system. My choice remains to be Git for it's distributed offline design as well as its simple and powerful branch management. Git plugs into the CI system by delivering the code to be compiled, tested, and deployed. The CI system must be able to understand and have support for branches natively, ie: we must not have to hack around branches by integrating that with scripts into the CI system.

# Continuous Integration
As mentioned already, CI must be plugged into the VCS and understand that multiple branches exist and need to be handled. When the VCS is updated the CI system is informed, usually either by polling or a hook from the VCS system itself. 

So far, Jenkins is our choice of CI server. I'll be looking at others during my research. Since we already know and use it, I decided to try out Jenkins first. After reading up on Jekins, I chose to use version 2. So far, we were using version 1. 

I was glad to see that Jenkins 2 is a lot more geared and targeted towards today's development teams, practices, and systems. Native multi-branch VCS support is amazing and exactly what we had to hack into Jenkins 1 with our current system. 

I took some time to try out Jenkins 2 in Docker, and liked what I saw. It worked well, was easy to set up, and I managed to get a dummy CI pipeline with multi-branch support working fairly easily. After the initial experiment, I built a Jenkins Docker image for our use. This image creates the Jenkins master node which then uses the`swarm`plugin in order to allow build slaves to be dynamically added.

Jenkins slaves are provided via the another Docker image I created that autojoings the Jenkins master. With our current Jenkins 1 set up we have some slaves, however, they are manually bootstrapped and connected to the master. Being able to bring up Docker containers that autojoin the Jenkins 2 cluster and take over the workload is simpler and definitely allows for elasticity.

Going back to the current Jenkins 1 set up, we currently define the build and test instructions within Jenkins itself. This is no longer the case with Jenkins 2. Jenkins now figures out what it needs to do based on the included `Jenkinsfile`. The presence or absence of this file includes or excludes a repo / branch from CI. This is accomplished when configuring the Jenkins project as a `multi-branch pipeline`.

# Selenium Grid
Since we build a web proudct, we rely on Selenium for automation and testing. Currently, we install Selenium and all the require tools on the build (CI) nodes in order to run tests there as well. This makes the CI nodes more difficult to install and configure, and we need to keep more software and packages in alignment. As such, splitting this responsibility away and delegating it to a specialized set of nodes / containers is beneficial.

For tests that require Selenium, we decided to provide a Selenium Grid via `docker-selenium` that can be accessed via the `RemoteWebDriver`. The grid is scalable as well using `docker-compose`. Multiple Chrome and Firefox nodes can be added dynamically. 

Being able to decouple the functions of the CI pipeline allows us to scale parts of it as needed. It also makes every single part of it simpler and less aware of the other parts of the system.

# Next Steps
Next, I want to spent some time and review the current state of GitLabCI. I used it over a year ago, and I know a lot has changed in there. Once I've refreshed my GitLab and GitLabCI knowledge I'm going to spend some time figuring out how code going through CI (being tested) can require additional resources or services to run these tests, and how they will be provided dynamically (for example, through spinning up Docker containers on the fly and tearing them down after).

Until then, happy hacking!

# References
* [Jenkins Multi-branch Pipeline](https://wiki.jenkins-ci.org/display/JENKINS/Pipeline+Multibranch+Plugin)
* [Selenium Grid](http://www.seleniumhq.org/docs/07_selenium_grid.jsp)
* [Selenium Swarm Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Swarm+Plugin)
* [Docker Selenium](https://github.com/elgalu/docker-selenium)
