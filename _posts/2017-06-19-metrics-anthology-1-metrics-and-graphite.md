---
layout: post
title:  "Metrics Anthology 1: Metrics and Graphite"
date:   2017-06-19 19:05:00 -0400
categories: metrics
---
# Metrics Anthology 1: Metrics and Graphite


## Why are Metrics Important?

If you work with systems of even modest complexity, you'll eventually want to record metrics about what those systems are doing and view those metrics over time. Metrics give you better insight into the operation of your systems, and they can be consumed by other tools to perform tasks like monitoring and alerting, trending, anomaly detection, etc.

If you want to build a metrics collection and visualization system, a decent place to start is with Graphite, one of the most well-known of such tools.


## What is Graphite?

Graphite is made up of:

- [Whisper](https://github.com/graphite-project/whisper): a file-based time series database format, similar to RRD files. Each whisper file contains the historical values for a single metric.
- [The Carbon Daemons](https://github.com/graphite-project/carbon)
  - **carbon-cache** performs I/O to and from whisper files. It is responsible for accepting metrics for storage, caching them in memory, and writing them to whisper files on disk. It can also be queried for metrics values, in which case it returns the union of data points in memory and on disk.
  - **carbon-relay** accepts metrics and relays them to one or more backends.
  - **carbon-aggregator** accepts metrics and combines similar ones, based on rules you configure, into aggregated metrics. For example, say you run a number of web servers and you collect login_requests metrics for each server. You might configure carbon-aggregator to sum all login_requests across all hosts to produce a single aggregated metric for your entire site.
- [Graphite Web](https://github.com/graphite-project/graphite-web): this is the web UI for Graphite. It allows you to explore your metrics, graph them, apply functions to the data, and build useful dashboards for visualization.


## Graphite Component Replacements

While the original carbon daemons work fine, they have some serious drawbacks:

- They are single-threaded Python apps.  If you want (or need) to take advantage of all the cores on your host to process metrics, you'll need to run multiple daemons.
- The daemons are also slow: in my experience, carbon-c-relay is about 10x more efficient than carbon-relay, and go-carbon is likewise 10x more efficient than carbon-cache, based on the load averge on the box.  In both cases, machines under heavy metrics load displayed a load average of 40.  Replacing the carbon daemon with its modern replacement dropped that load average to around 4.
- The daemons are controlled by a large number (10+) of configuration files. These files will become a pain to manage.
- The daemons - particularly the relay - cannot be configured to behave in all the ways you may want them to, thus significantly constraining your architectural options. For example, you cannot tell the relay to send a copy of all metrics to multiple destinations.

These drawbacks are serious and will quickly become apparent once you start scaling up the number of metrics you collect.  Fortunately, engineers have written replacements for these components that are faster, more efficient, more flexible, and in the case of the UI, more beautiful.  Typical replacements are:

- Relay + Aggregator:
  - [carbon-c-relay](https://github.com/grobian/carbon-c-relay) - multithreaded C relay and aggregator implementation.  Single config file.
  - [carbon-relay-ng](https://github.com/graphite-ng/carbon-relay-ng) - relay and aggregator written in Go. Has a web-based admin UI.
- Cache + Aggregator:
  - [go-carbon](https://github.com/lomik/go-carbon) - multithreaded Go-based replacement for carbon-cache. Much more efficient.
- Other storage backends:
  - [InfluxDB](https://www.influxdata.com/)
  - [ElasticSearch](https://github.com/elastic/elasticsearch)
  - [OpenTSDB](http://opentsdb.net/)
  - [Gorilla](https://blog.acolyer.org/2016/05/03/gorilla-a-fast-scalable-in-memory-time-series-database/)
- UI:
  - [Grafana](http://grafana.org/) - a popular, modern metrics UI
  - [many more](http://dashboarddude.com/blog/2013/01/23/dashboards-for-graphite/)
- Other Utilities
  - [StatsD](https://github.com/etsy/statsd) - Collects UDP events and transforms them to metrics
  - [CollectD](https://collectd.org/) - Plugin-based metrics collection daemon.

The nice thing about Graphite is its highly componentized, so you can replace individual components with others that perform the same task.

For our purposes, we'll use carbon-c-relay for relaying and aggregation, go-carbon and whisper for storage, and Grafana for our UI.  We'll also use StatsD to enable easy metrics collection from our apps, and collectd to collect system metrics.


## Up Next

Over the next few posts, I'll outline the set of tools that I've used to build a working system that collects 3.5 million metrics every 10 seconds from nearly 1000 machines.
