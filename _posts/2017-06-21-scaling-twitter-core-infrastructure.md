---
layout: post
title: "Tech Talk: Scaling Twitter Core Infrastructure"
author: Evan Kuhn
date: 2017-06-21 11:37:00 -0400
categories: techtalks
summary: A few scaling principles learned while scaling Twitter's infrastructure.
---

From the YouTube video: [Flight Lightning - Scaling Twitter core infrastructure](https://www.youtube.com/watch?v=6OvrFkLSoZ0)

- This talk presents a handful of principles to help you scale your system
- Separate concerns
  - Don't run everything in one monolithic app
  - Break things down into separate services, subsystems, etc
  - Easier to fix, scale, develop for, etc
- Abstraction is the Soul of Scalability
  - Good abstractions allow you to reason easily about the system, develop for it, divide up work, reduce dependencies, etc.
  - Refactoring early is much less painful than trying to do so later.  Make sure you create good abstractions early!
  - Abstractions evolve
- Visibility is King
  - Expose visibility into your systems via metrics, logging, and tracing tools.
  - You need visibility in order to solve problems (and anticipate them)
- Systems Knowledge is Important for Diagnosis
  - Abstractions hide the details and are great when everything works
  - But when things break, you want to have systems knowledge
  - The underlying system can have issues that percolate all the way up to the app
- Own the Master Switch
  - Have a system that gives you the power to rate-limit requests.
  - When things fail, you may need to turn down traffic to allow services to come back up.
  - Or when you get slammed, everything may grind to a halt, so no requests are serviced.
