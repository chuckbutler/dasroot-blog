---
category: Devops
date: 2015-11-15T21:09:32-05:00
post_id: container-networking-flannel
self-hosted: null
tags:
  - containers
  - SDN
  - flannel
  - LXC
time: 7:30
title: Networking LXC containers with Flannel
youtube: https://www.youtube.com/embed/bCvl-TsxVXA
---

Have you ever deployed a service in juju to a LXC container and wished you could
 separate your services among hosts? Run a farm of web-heads with a separate
 farm of database servers?

Leveraging Flannel from CoreOS this is completely possible. No longer are your
LXC containers restricted to only communicating on a single host. In 7 minutes
I'll take you through a sample deployment illustrating the containers networking
 on different hosts and operating as we would expect them to.
