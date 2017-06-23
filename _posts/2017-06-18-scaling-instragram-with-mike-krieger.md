---
layout: post
title: "Tech Talk: Scaling Instagram with Mike Krieger"
author: Evan Kuhn
date: 2017-06-18 18:05:00 -0400
categories: techtalks
description: How Instragram scaled their infrastructure, and lessons learned.
---

From the YouTube video [Scaling Instagram with Mike Krieger
](https://www.youtube.com/watch?v=oNA2C1vC8FQ)

- Founded 2010.  2012: acquired, 4 engineers, 30MM MAU.  2015: 95 engineers, 300MM MAU.
- **DO THE SIMPLE THING FIRST!**  Don't build things you don't actually need right now.
- Use "boring" technology that's operationally quiet.
- Tech stack:
  - Originally: nginx, redis, memcached, postgres, gearman, django.
  - Currently: nginx, cassandra, memcached, postgres, rabbitmq, django.  Also unicorn, proxygen, thrift, scribe (Facebook tech)
- Async Tasks (site scale)
  - Gearman (async task broker).  Started with single host.  Chose gearman bc easy to setup.  Lasted for 1.5 years.
  - Scaled to 8 gearmon brokers, 400 app servers.  Web requests became slow.  No failover.  Couldn't enable persistence bc of crashes.
  - Do the simple thing next: chose celery + rabbitmq as a replacement for gearman.  60 ms mean response time dropped to 10 ms.
- Code Deployment (team scale)
  - Initially: git pull + fabric (remote scripting tool)
  - Fabric parallel mode came out, helped when number of machines grew to 10+
  - Then they wrote a 'rollout' command to upload tarball to S3, pull it down on each machine, and restart the service.  Useful for 1.5 years.
  - Then wrote Sauron: could lock a resource and deploy.  Helped coordinate deploys.  Lasted 1.5 years.
  - But: much cargo cult knowledge around how to deploy.
  - So: updated Sauron scripts to write that knowledge into scripts.
  - Next problem: people waiting on locks.  So they extended Sauron for Jenkins integration.
  - **LESSON:** take a human procedure and at each stage, figure out what to automate.  Also, do not automate things you don't need yet!
- Search (product scale)
  - MySQL isn't good for regex / wildcard searching
  - V2: used Solr (Lucene-based)
  - V3: used ElasticSearch.  Easy to setup, easier to scale out.  But: ops problems surfaced later.
  - V4: moved to Unicorn (Facebook graph DB tech).  Were able to then tweak their search algo to show results people wanted.
  - Kept iterating on search algo logic to improve results.
- **Lessons:**
  - Do the simple thing first, until your scale / team / product / changes
  - Then do the simple thing next
  - Ground your evolution in problem-solving
