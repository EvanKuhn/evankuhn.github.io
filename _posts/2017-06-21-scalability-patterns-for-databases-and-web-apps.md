---
layout: post
title: "Scalability Patterns for Databases and Web Apps"
author: Evan Kuhn
date: 2017-06-17 21:45:00 -0400
categories: distributed-computing
summary: Common patterns used to scale relational databases and web apps.
---

This post outlines some commonly-used techniques and patterns for scaling out a relational database or web application. You can find many resources online that dive further into the details.

## Scaling a Relational Database

Scaling a database's storage space and read and write capacity usually proceeds according to the following steps:

For scaling up storage space, and read and write capacity:

1. **Scale Up:** Buy a bigger box. You can only do this once. It's usually done first, because it's the quickest fix.

1. **Read Replicas:** Here we add a number of 'read slaves' to which we replicate all data from our master database.  The master continues to accept all writes. You then modify your app to direct all reads to the slaves, thus reducing load on the master. This approach provides additional read and write capacity. Keep in mind that there are practical limitations on the number of slaves you can have. Further, this approach does impose potential consistency problems. as there is a race condition between writing data to the master and reading it back from a slave.

1. **Multi-Master:** Set up two masters that replicate to each other. This allows for some more write capacity, as well as failover, but comes with a lot more complexity.  I'll only briefly mention it.

1. **Sharding:** Eventually, additional read slaves no longer provide additional capacity, and/or you may run out of storage space. At this point you need to think about *sharding*, or breaking up the data in your database to live in multiple database clusters. This allows you to scale out even more, wrt both storage and read/write capacity, but now your app must deal with reading/writing to multiple databases instead of just one. This adds complexity and presents new problems, esp. for ACID requirements. Moving to a sharded architecture also requires a significant investment in engineering time, as your app must be updated to support the new structure.

1. **Multi-Site:** Further split your data, perhaps by geographic location, and build a complete set of database clusters in each location.  You'll need to build services, or add logic to existing services, to route your requests to the proper location.  This is similar to sharding, but at a higher level.  It divides storage and throughput requirements across multiple sites instead of just one.

We can also apply other techniques for improving database efficiency, thus giving us some extra capacity:

1. **Connection Pooling:** App servers pre-create a pool of connections to the database hosts, so they don't have to continually create and destroy connections for each query.

1. **Connection Load-Balancing:** Databases support a max number of connections.  The connection load-balancer will support more connections and will balance them across the backend DBs.

## Scaling a Web Application

We want to separate out components, scale them separately and horizontally:

1. Start with your web app and database on a single server.<br/>
![Database and web app on same host](https://docs.google.com/drawings/d/16_XK_s0Ti2bDEy6nO4tYFKcFmc9qmJts9SU3BGk2WS8/pub?w=293&h=100)

1. **Move your database** to a separate server, allowing you to scale your database separately. Eg:
  - Scale up, create read slaves, start sharding...
  - Create user clusters, where each user is mapped to a specific cluster.  A cluster is a standalone set of all the apps and servers needed to serve the app.
![Separate database from app server](https://docs.google.com/drawings/d/1FHw4Z6W09R91KQbhFiCQHVSYCKA5mMenkIE-SBRgio0/pub?w=469&h=100)

1. **Horizontally scale** your web app servers.  Put a load balancer in front.  This requires that the app servers do not share state, which we took care of by moving the database to a separate host.
![Scale the web app horizontally](https://docs.google.com/drawings/d/1ddaa3gqEDVX6eqWsuvL1Nd-ZcCqwxUTGAJk7oepW5d4/pub?w=589&h=227)

1. Further split your app into **smaller services** and scale them separately.  Capacity planning now becomes more complicated, as we have multiple services to plan plan for.

1. Add a distributed **caching layer** such as redis or memcached.  This will reduce load on the database and increase response time.
  - Check the cache before calling the DB or other service.
  - Or use Varnish, which sits in front of the webapp and serves as a cache and proxy.
![Add a caching layer](https://docs.google.com/drawings/d/1MyZNAq0WXsZSdl2U4WOBMOJrpOQFF9wRo3DS923kO9Q/pub?w=800&h=180)

1. Use **HTTP caching headers** to control client-side caching of resources.  Ideally the client can cache many of the resources locally rather than request them multiple times.

1. Use a **CDN** to offload traffic to the CDN and make resource requests faster (since the CDN endpoint will likely be closer than your app servers).

1. **Parallelize your app,** eg: via Message Queues (explained below)

1. Use **non-blocking IO** to handle more concurrent requests per webserver process.

1. For any DB data that is frequently written and read (eg: session data), move it out to a faster **in-memory database.**

1. Allow long-running calculations to be queued for **asynchronous processing,** so the webapp does not need to wait for them to be finished.  Eg: throw them on a message queue, let separate workers deal w them.

### Parallelization via Message Queues

You might also look into parallelizing components of your web application logic via Message Queues:

1. Split up your web app workflow into separate pieces that each take a significant amount of time.
1. For each piece, replace the logic with a message queue call.
1. Write daemons / microservices that read these MQ requests and respond.
1. In your web app, wait for all MQ responses and return the result to the client.

![UML sequence diagram for serial vs parallel processing](https://docs.google.com/drawings/d/13UsGTkTSj_QE4jiIiEaZXXDtb3voZOg3FE88lM2nkwQ/pub?w=1128&h=518)

Results:
- You've parallelized slow pieces of code that previously ran in series.
- You can scale your MQ daemons onto multiple backend servers.
- You can also scale MQ daemon servers on demand.

## References

[1] ["Database Server Scaling Strategies."](http://realscale.cloud66.com/database-server-scaling-strategies/) RealScale. <br/>
[2] Gulati, Shekhar. ["Best Practices For Horizontal Application Scaling."](https://blog.openshift.com/best-practices-for-horizontal-application-scaling/) 10 Jul 2013. OpenShift. <br/>
[3] Brody, Hartley. ["Scaling Your Web App 101: Lessons in Architecture Under Load."](https://blog.hartleybrody.com/scale-load/) 08 Sep 2015. <br/>
[4] Engates, John. ["7 Stages of Scaling Web Applications."](https://www.slideshare.net/davemitz/7-stages-of-scaling-web-applications) 06 Aug 2008. SlideShare slides.<br/>
[5] Gschwendtner, Lenz. ["scaling web applications with message queues."](https://www.youtube.com/watch?v=aOrGq9yb6og&t=1365s) 19 Jan 2012. YouTube. <br/>
