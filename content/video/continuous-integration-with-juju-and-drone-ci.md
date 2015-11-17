---
category: Devops
date: 2015-07-15T21:33:00-05:00
post_id: drone-ci-juju-integration
self-hosted: null
tags:
  - build-pipeline
  - continuous-delivery
time: 10:30
title: Continuous Integration with Juju and Drone CI
youtube: https://www.youtube.com/embed/ZIcf4mefX14
---

First of all, big thanks to @bradrydzewski for leading the Drone-CI project.
 Drone is a GOLang based Continuous Integration server built on top of Docker.
  Its super light weight, runs efficiently on a small Digital Ocean instance
  without any fuss, and tackles most jobs you want to plug into a CI service.

This video illustrates how to Continuously Integrate your Juju Charms against a
 single cloud substrate with minimal configuration. Store the CI configuration
 in your project VCS, and move on with innovation.


Repositories:
- http://github.com/chuckbutler/drone-ci-charm.git
  Deploy the Drone-CI server

- http://github.com/chuckbutler/docker-charm.git
  A reference for charm tests, and CI setup

- http://github.com/kapilt/juju-digitalocean.git
 Use Digital Ocean in Juju

- https://github.com/chuckbutler/hubot-drone.git
  Build the latest hash on the stack in drone from Hubot (pre-alpha)
