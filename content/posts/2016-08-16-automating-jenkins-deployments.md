---
category: devops
date: 2016-08-16T14:01:05-05:00
image: "/images/2016/aug/packaging.jpg"
imagecaption: "Copyright: <a href='http://www.123rf.com/profile_scanrail'>scanrail / 123RF Stock Photo</a>"
post_id: snaps-everywhere
tags: snappy, ubuntu, packages, docker
title: Snaps, Snaps Everywhere!
---

I've been reading a lot of the buzz around Snappy as a packaging format. With the publication of
great tooling like [Snapcraft](https://snapcraft.io) I was really feeling left out in the cold.
The tooling only seems to support linux platforms. Which stands to reason, its a linux package
format. However, being the dedicated little canonical'ers that we are, we wanted snappy everywhere.
Even if its only in a fake capacity to enable developers to cycle on their payload
packaging, run in CI, whatever the case may be. And since we already built a rather comprehensive
workspace image for Juju, this was a real treat as an afternoon hack session.

## Snapbox! A box for all your snappyness

Since we created [jujubox](https://hub.docker.com/r/jujusolutions/charmbox) we wanted
an interactive shell to work and debug our snaps in. This is where `snapbox:latest`
comes into play. You can pull the project down with `docker`.

```bash
$ docker pull jujusolutions/snapbox:latest
$ docker pull jujusolutions/snapbox:onbuild


$ docker images
jujusolutions/snapbox      latest      c5ccc59ea884   27 hours ago    594.1 MB
jujusolutions/snapbox      onbuild     183830bb1b7b   4 hours ago     593.8 MB
```

As you can see, snapbox comes in two flavors. `:latest` and `:onbuild`. These
boxes have slightly different purposes, and different behavior characteristics.

#### :latest

The latest image is intended to be a throw-away hermetically sealed REPL and
debug environment for building snaps. It gives the user an Ubuntu Xenial host
with `snapcraft` and `snapd` packages installed. Defaults to a non-root user,
and has explicit instructions on use in the `README.md`.

#### :onbuild

The onbuild image is intended to be a throw-away build-job oriented container
that assumes its always doing a full build (prefixed by a clean), and assumes
that $PWD is mounted to /snaps


Curious about this whole thing and want to try it yourself?

copy this snapcraft.yaml somewhere on your disk

```yaml
name: etcd
version: "2.3.2"
summary: etcd is a distributed key value store based on raft consensus
description: |
  etcd is an open-source distributed key value store that provides shared
  configuration and service discovery for CoreOS clusters. etcd runs on each
  machine in a cluster and gracefully handles master election during network
  partitions and the loss of the current master.
confinement: devmode
apps:
  etcd:
    command: bin/etcd
parts:
  etcd:
    source: https://github.com/coreos/etcd.git
    plugin: go
    go-importpath: github.com/coreos/etcd
    build-packages:
      - libpcap-dev
```

Then run the following command:

```
docker run --rm -v $PWD:/snaps snapbox:onbuild'
```

You will notice some new directories:

```
├── etcd_2.3.2_amd64.snap
├── parts
│   └── etcd
├── prime
│   ├── bin
│   ├── command-etcd.wrapper
│   └── meta
├── snapcraft.yaml
└── stage
    └── bin

7 directories, 3 files
```

However, notice at the top: `etcd_2.3.2_amd64.snap`

How many hours have any of us spent writing build scripts for deb or rpm
systems? With about 15 lines of yaml, we now have a reproduceable snap
installable nearly everywhere (this was only built on an amd64 host.. keep in mind!).


I hope this gives you some good insight into how quickly you can get started
writing snaps, and subsequently how to build them on your laptop/desktop/calculator.

All that's required is a docker daemon  thanks to this neat little hack  :)

Snap Happy!


