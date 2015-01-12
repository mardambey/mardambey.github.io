---
layout: post
title: ! 'Mate1''s activity feed: Cassandra, Kafka, Netty, Varnish, Akka, Ejabberd
  *updated*'
date: '2011-12-08T04:25:00-05:00'
tags:
- Code
- mate1
- cassandra
- kafka
- netty
- varnish
tumblr_url: http://www.hisham.cc/post/30203546432/mate1s-activity-feed-cassandra-kafka-netty
---
This is a re-write of the original article that was here. Since then we’ve integrated Akka and Ejabberd.

A while ago we decided to give our users a news feed like system that would show them what’s going on on the site and around them. Users on Mate1 can make friends with others users by adding them to their friend list. A user can also express interest in another by flirting with them, asking them for a photo, or attempting various other forms of communication. We decided to keep track of these interests and apply certain algorithms to figure out of these people we should recommend to you and show you information about.  Another part of the system monitors users that are interested in you and will try to recommend these users to you.

As well as keeping track of this data we generate and emit events when anything related to you happens in the system, be it the on-line part or the offline one. These events are produced and fired off in an asynchronous fashion using our EventManager.

The EventManager (Java compatibility layer is called EventManagerUtil)  is a very simple interface that is written in Akka that provides calls like the following:

object EventManagerUtil {
  val log = Logger.getLogger(this.getClass.getName)

  val WORKERS_MAX_NUM = 24

  val router = EventManager.
    system.actorOf(Props[EventManagerWorker].
     withRouter(RoundRobinRouter(nrOfInstances = WORKERS_MAX_NUM)))

  def publish(msg: EventData) {...}

  def publish(msg: EventDataAvro) {...}

  def subscribe(topicCb: java.util.Map[EventType, SubscribeCallback], groupId: String) {...}

  def resetGroupTopicOffset(group: String, topic: EventType) {...}
}


Every one of those calls is in fact a wrapper around a message to an Akka actor. In fact the actor is not just a single one, its a router with several worker actors behind it. The router distributes work to its worker in a round robin fashion. This allows us to change the number of works up and down based on load (can be done dynamically). We can also change the type of router used so we can allow certain events to propagate faster than others (priority) or have certain workers always handle certain types of events (consistent hashing). The Akka actors will then continue to push the event into the queue asynchronously. The EventManager also allows for subscribers to tune into events (via a pull, not a push, model in order to circumvent subscriber or consumer overloading). Subscribers or event consumers pull messages, act on them, then ack the message queue and pull more. Subscribers can also “go back in time” by resetting their “offset” in the queue in order to re-process messages. This can be very useful if we discover corruption later down the pipeline or simply need to recalculate parts of the data.

The message queue is built using a Kafka cluster. Kafka allows us to have an elastic message queue (brokers can be added or removed) that is highly available. Kafka is a messaging system that was originally developed at LinkedIn to serve as the foundation for LinkedIn’s activity stream and operational data processing pipeline. There is a small number of major design decisions that make Kafka different from most other messaging systems:

Kafka is designed for persistent messages as the common case
Throughput rather than features are the primary design constraint
State about what has been consumed is maintained as part of the consumer not the server
Kafka is explicitly distributed. It is assumed that producers, brokers, and consumers are all spread over multiple machines.
After messages are stored in the queue they are then pulled out by several consumers that analyse them and decide what to do with them. Some of these events contribute towards counters. Others signal that our automatic review system has approved or denied certain messages, images, etc. so that other processes can pick them up. We also use these consumers to generate log files and log databases that are later used by our analytics system.

For the purposes of the news feed we have consumers that are tuned in to a bunch of interesting events and changes in the system. These consumers will write data based on these events into the permanent data store. The consumers make use of Akka actors and futures to parallelize blocking network requests to services or data stores and try to be as efficient as possible in order to process as much events as possible so the activity feed is in as near real time as it can be. As soon as the data is ready it is pushed into our Cassandra cluster for permanent store.

In Cassandra we’re storing 4 tiers of activity for each user. These tiers are also “rolled up” in order to provide a more compressed view where we combine events based on user and type (screen real estate is valuable!). The tiers are used to rank events based on their types. This ranking is then taken into account when a roll-up happens in order to display the most important events on top. The 4 tiers translate to 4 wide rows per user. Each row contains a lot of columns having values like:

{
    key: TimeUUID // event time

    {
        from  // user_id
        type // event type
        read // read or unread
    }
}


All we need to store is just enough data so we can reconstruct the time line. Once this data is stored in Cassandra it is then read out by the website and displayed. In order to keep the interface simple between the website and Cassandra we built a tiny Netty service that has the business logic to fetch the activity feed data and return it to the website in JSON format.

The Netty service is supports GZIP compression and sits behind a few Varnish servers that act as a cache. The presence of Varnish in front of Netty means that in case we have Cassandra issues due to maintenance or load then we’re saved by the cache (sometimes Cassandra reads can slow down under load since we do not have a separate Cassandra cluster for the roll-up calculation and analytics, yet). The service provides access in a format similar to the following:

http://server/feed?uid=18413598&tier=tier1&start=0&count=40


This allows us to display the activity feed and page around it rather easily. The HTTP interface also allows the website to perform some write operations on the activity feeds (manipulate counters, mark items as read, etc.)

The last and latest addition to this system has been the inclusion of Ejabberd into it. Ejabberd is a Jabber (XMPP) server implemented in Erlang. As we have been working on our mobile Mate1 platform we decided to integrate real time notifications to the user. Since Ejabberd speaks XMPP (can’t be done through a web browser) we use its BOSH module to allow web browser to connect to our servers (we are using Ejabberd for these notifications and also for chat). The BOSH protocol defines how arbitrary XML elements can be transported efficiently and reliably over HTTP in both directions between a client and server. The same consumers system that was mentioned before is also used to route events directly to our users in as real time as we can, through BOSH, to their mobile devices (or desktops).

At the time of this writing we’re getting ready to finish up and release our mobile platform and will put the latest parts of this system live. Some code snippets that we’ve shared while doing this project (and others) can be found here.
