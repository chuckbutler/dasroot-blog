---
category: monitoring
date: 2016-06-17T17:07:40-04:00
post_id: juju-beats-in-beta
tags:
 - devops
 - charming
 - containers
 - juju
 - monitoring
title: Beats stack is now in beta
---

Prior to today, we had released a fraction of the Elastic Beats stack.

Primarily composed of Filebeat, and Topbeat. Targeted at log shipping,
visualization, and host metric monitoring (cpu, memory, net, etc.). This was
great as a start, but it only scratched the surface.

Today I'm releasing 2 more in -BETA status:

- Packetbeat: A network monitoring tool, targeted at application network monitoring.
 - Analyse MYSQL queries
 - Get HTTP status results over a range of time
 - Monitor your mongodb collections

- Dockerbeat: An application container daemon monitoring tool
  - Get aggregate counts of how many containers you have running on many hosts
  - CPU, Network, Memory statistics of the containers

These two beats give a nice rounded edge finish to our beats bundle, and will
be making their debut in the beats-core bundle after prelimiary user testing
and feedback.

If you want to get started today, I've prepared a small clone of the
observable swarm bundle:

<script src="https://assets.ubuntu.com/v1/juju-cards-v1.3.0.js"></script>
<div class="juju-card" data-id="~lazypower/bundle/observable-swarm-0"></div>

Log into your swarm unit, and deploy some workloads to get the activity
moving so we have interesting graphs to look at:

    juju ssh swarm/0
    tar xvf swarm_credentials.tar
    cd swarm_credentials
    source enable.sh

Now that you're talking to swarm, launch some containers

    docker run -d lazypower/2048

you can spam that over and over and launch several copies of a light weight
2048 game server. For more complex workloads I highly recommend pulling
a `docker-compose.yml` and spinning up more labor intensive workloads with

    docker-compose up -d

With our workload now humming away, lets check in on our kibana dashboard:

![dockerbeat dash](/images/2016/june/dockerbeat-dashboard.png)

And here we see all our container metrics:

 - CPU usage per container
 - CPU usage in aggregate
 - Network usage in aggregate
 - Top 10 consumers of cpu/memory/network

In summary, this is a great x-ray into not only your server statistics
but also your container workloads. If you encounter any bugs, feel free to
file them against the layer repository
[juju-solutions/layer-dockerbeat](https://github.com/juju-solutions/layer-dockerbeat)

Happy Hacking!
