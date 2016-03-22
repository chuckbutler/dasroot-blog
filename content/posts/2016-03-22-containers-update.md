---
category: charming
date: 2016-03-22T11:42:55-04:00
image: /images/2016/mar/beats-formation.png
imagecaption: Dashboard anything with juju
post_id: null
tags:
 - devops
 - charming
 - containers
 - juju
title:  Container Ecosystem Update (March 2016)
---

 <script src="https://assets.ubuntu.com/v1/juju-cards-v1.0.9.js"></script>

Heres an update update for the
Containers team. Its been an exciting couple of weeks, and I'd like to take a
moment to run you down what we've been up to, and what’s coming.

Before we get into that, a friendly reminder that all the work we are doing can
be seen in the charm store under our namespace

[http://jujucharms.com/u/containers](http://jujucharms.com/u/containers)

as well as our team landing page

[http://containers.juju.solutions](http://containers.juju.solutions)


# Beats

I recently delivered on some “extras” while working on revamping our ELK stack.
Beats are light weight golang agents that you can relate to *any* charm (except
  subordinates) and immediately begin collecting telemetry metrics. Log files,
  distributed HTOP, and network analysis. These are all backended by Logstash
  or Elasticsearch. There are visualizations attached at the bottom of the model,
   and the running application dashboards.

There is currently no visualization(s) created for the log shipping, and docker
 host metrics provided by dockerbeat. These are TODO: and if anyone has
 dashboarding experience with kibana4, I'd <3 to talk to you.

These charms are so new, and haven’t made their way through the queue, and
should be treated as concept work - Please file bugs, and leave feedback :)

What’s left:

- TLS Support
- Any additional configuration that isn’t currently exposed in config.yaml per charm
- Full Packetbeat integration (its finicky as best i can tell)

But this is a solid start to exposing an x-ray to any Juju model out there.

<div class="juju-card" data-id="~containers/trusty/topbeat"></div>
<div class="juju-card" data-id="~containers/trusty/filebeat"></div>

<div class="juju-card" data-id="~containers/bundle/beats-core"></div>

![/images/2016/mar/kibana.png](/images/2016/mar/kibana.png)
![/images/2016/mar/topbeat-dashboard.png](/images/2016/mar/topbeat-dashboard.png)

# ELK Stack

New Logstash charm submission!
[https://bugs.launchpad.net/charms/+bug/1560167](https://bugs.launchpad.net/charms/+bug/1560167)

New ELK stack bundle complete w/ integration tests from the top down
[https://bugs.launchpad.net/charms/+bug/1560187](https://bugs.launchpad.net/charms/+bug/1560187)

Various fixes to the Kibana and Elasticsearch charm(s), as well as some version
 bumps to the 2.x series of services.


 <div class="juju-card" data-id="~containers/bundle/elk-stack-0"></div>


# Kubernetes

We’ve finally stabilized the Kubernetes charm rewrite, and have submit it for
the upstream repository, bringing with it a net code deletion of 1500+ lines.
This is the power of layers ladies and gentleman. We’ve reduced our footprint
in an upstream repository and scoped it only to their projects concerns. That
sounds like a win!

[https://github.com/kubernetes/kubernetes/pull/22726](https://github.com/kubernetes/kubernetes/pull/22726)

Logging: Our first major feature other than “just providing kubernetes” is an
auditable kubernetes stack with centralized log analysis nexus and backup.
We’re ¾ complete with the feature and moving towards final export from
elasticsearch as our end goal which is on track to be delivered this week.



Storage: is our next step, ensuring our containers are run on external volumes,
and we can snapshot/migrate those volumes in the event of disaster recovery.


<div class="juju-card" data-id="~containers/bundle/kubernetes-core-0"></div>

# Swarm

It’s been a little stagnant here while we ramp up and clean up the Kubernetes
codebase. However I’ve been circling back and starting to land support for TLS,
and Consul as a service discovery mechanism. Next steps are to wrap that up and
move on to the logging backend to bring it to feature parity (estimated) with
Kubernetes.

<div class="juju-card" data-id="~lazypower/bundle/swarm-core-0"></div>


That concludes our update, but thanks for reading and we look forward to the
next cycle ahead of us and bringing you even greater features for your datacenter.
