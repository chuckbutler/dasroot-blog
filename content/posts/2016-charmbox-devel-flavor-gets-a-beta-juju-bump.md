---
category: announcement
date: 2016-02-26T09:49:24-05:00
image: null
imagecaption: null
post_id: charmbox-devel-flavor-gets-beta-update
tags:
  - juju
  - charmbox
  - charming
  - docker
title: charmbox devel flavor gets a beta juju bump
Status: published
---

## Charmbox

You might remember a post last year about unofficial Juju Images, namely the
**charmbox** image. This docker container contains everything required to
perform the duties of a ~charmer in the Juju ecosystem. From tooling, to
isolated dependency environment, this image is the basis for allowing
charmers to be agile in their job duties.


#### What does it do again?

Charmbox includes the complete tooling environment to build, deploy, configure,
scale, manage, and evaluate charms incoming from the community. It features:

 - Juju
 - Charm-Tools
 - Bundletester
 - A pre-baked $JUJU_REPOSITORY
 - Supports any user/environment via volume mapping
 - Dependency isolation and re-useable sanitary environment (when run with --rm)
 - Python virtualenv support so you can isolate charm dependencies for more involved sessions


## :devel flavor is now Juju 2.0-beta1

You can now `docker pull jujusolutions/charmbox:devel` and start looking around at juju 2.0 in a
completely isolated sandbox. Allowing you to poke prod and fully isolate
juju 2.0 from your current stable system.

This is important to us as an ecosystem as we are seeing the ripple effect
of the API changes in the Juju 1.x => 2.x transition. Some of the tooling
binds very heavily to the 1.x api spec and thus the tools no longer function.

This is basically -beta all around, and will eventually supersede what is serving
as :latest from `jujusolutions/charmbox` - which delivers the current -stable
Juju 1.25 now.

## Get Started on OSX

<script src="https://gist.github.com/chuckbutler/02af4d7f2838c4da5498.js"></script>
