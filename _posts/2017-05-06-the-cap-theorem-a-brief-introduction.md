---
layout: post
title: "The CAP Theorem: a Brief Introduction"
author: Evan Kuhn
date: 2017-05-06 09:00:00 -0400
categories: distributed-computing
description: An introduction to the CAP theorem, it's proof, and a discussion of partition tolerance.
---

## The Basics

The CAP theorem states that distributed systems have three desirable properties, described below, but that we can choose at most two.  They are:

- **Consistency** - A read is guaranteed to return the most recent write for a given client.

- **Availability** - A non-failing node will return a reasonable response within a reasonable amount of time (no error or timeout).

- **Partition Tolerance** - Even if the connections between nodes are down, the other two promises (A and/or C) are kept.

The CAP theorem follows from studying the scaling of single-node databases to distributed systems.  While relational databases make strict ACID promises to preserve consistency, as we scale out to more than one node we realize that we can't keep all three CAP properties.  We must choose two.

And as we'll see below, we really only have two choices: **consistency (CP) vs availability (AP).**

## Proof

Say we have a simple distributed system of two nodes.  Further, assume that communication between the two is not currently possible (a network partition exists between them).

Now say a client connects to node A to execute a query.  One of three things can happen:

1. Node A can return the value it has, though it may not be consistent with the value on node B.  In this case, the system is **available but not consistent.**

1. Node A can decide that, since it cannot communicate with B to compare values, it will either block indefinitely or return an error.  In this case, the system is **consistent but not available.**

1. Node A communicates with node B, determines the consistent value to return, and returns it.  In this case the system is both consistent and available.  But this could only happen if the network were not partitioned, so in this case **the system is not partition tolerant.**

[This video](https://youtu.be/Jw1iFr4v58M?t=2m31s) does a great job of explaining it.

## You Must Choose P

Essentially this argument boils down to the fact that in the real world, networks fail.  Once a partition occurs, the system may either:

- refuse to respond / respond with an error, thus favoring consistency (CP), or...
- respond with potentially inconsistent data, thus favoring availability (AP)

Thinking about it in a more realistic way: if we favor availability, we'll also want to choose partition tolerance, because without it, our system won't be available during a network partition.  Thus, we can either choose AP or CP.

[Nicolas Liochon](http://blog.thislongrun.com/2015/04/the-unclear-cp-vs-ca-case-in-cap.html) looks at this another way, which I like: he argues that **CA defines an operating range,** while **CP/AP define the behavior during a partition.**  Consider that the CAP theorem allows us to build a system that promises CA, with the caveat that *it requires a functioning, non-partitioned network.* Once a partition occurs, however, the system will no longer be able to keep those promises and may behave in one of the two ways noted above (CP or AP).  (Or it may fail to be either consistent *or* available, though we can do better).

Thus, CA defines the operating range (eg: *"this system requires a non-partitioned network to function"*), while CP/AP define the behavior during a partition (eg: *"this system will remain available but become inconsistent during a partition"*).

So, must you choose P?  While a CA system is valid, network partitions can occur for any system, so you must consider the desired behavior for the system in such a case.  Would you prefer that it be consistent (CP) or available (AP)?

## References

[1] Perry, Michael L. ["CAP Theorem."](https://www.youtube.com/watch?v=Jw1iFr4v58M) 04 Jul 2010. YouTube. <br/>
*Simple and clear explanation, including proof.*

[2] Messinger, Lior. ["Better explaining the CAP Theorem."](https://dzone.com/articles/better-explaining-cap-theorem) 17 Feb 2013. DZone. <br/>
*Simple, clear. Touches on why partition tolerance cannot be sacrificed.*

[3] Hale, Coda. ["You Can’t Sacrifice Partition Tolerance."](https://codahale.com/you-cant-sacrifice-partition-tolerance/) 10 Oct 2010. <br/>
*In-depth discussion of partition tolerance.*

[4] Greiner, Robert. ["CAP Theorem Revisited."](http://robertgreiner.com/2014/08/cap-theorem-revisited/) 14 Aug 2014. <br/>
*Simple explanation of why you must choose P, so your choices are AP or CP.*

[5] Liochon, Nicolas. ["The unclear CP vs. CA case in CAP."](http://blog.thislongrun.com/2015/04/the-unclear-cp-vs-ca-case-in-cap.html) 05 Apr 2015. <br/>
*In-depth discussion of CP vs CA, and why CA is valid.*
