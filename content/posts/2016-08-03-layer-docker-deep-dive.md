---
category: charming
date: 2016-08-02T22:48:59-05:00
image: "/images/2016/aug/layer_docker_deep_dive.jpg"
imagecaption: Copyright <a href='http://www.123rf.com/profile_nexusplexus'>nexusplexus / 123RF Stock Photo</a>
post_id: layer-docker-deep-dive
tags:
  - tutorial
  - docker
  - charm-authoring
title: Layer Docker Deep Dive
---

Juju is all about modeling your application. That means that an application can be in a docker container, use a configuration management utility on top of a traditional machine, or is a single binary golang application. This is the beauty of abstracting via models; we can concentrate on the applications themselves instead of their delivery methods. In this article we will cover the use case of being a developer, with different docker containers, and how we can model our application. Additionally how we can scale out leveraging Juju’s relationships to horizontally scale to other applications.

## Expected pre-requisites:

We’ll assume you have the following:

- A [prepared Juju installation](https://jujucharms.com/docs/stable/getting-started) (version 1.25 assumed for this article)
- [charm-tools and the charm](https://jujucharms.com/docs/devel/tools-charm-tools) command
- Git installed
- Credentials for a cloud provider
  - Why not try this with a [trial GCE Account](https://jujucharms.com/docs/stable/config-gce)? Get started with $300 credit for free
- Installed Docker for Mac/Windows or have installed docker locally
  - 1.12 is current stable
  - Docker-compose 1.8.x
- About an hour of time
- An understanding of basic Docker tooling: such as [Dockerfiles](https://docs.docker.com/engine/reference/builder/) and [Compose files](https://docs.docker.com/compose/compose-file/).

> Note:  This tutorial will be an interactive process. We will be writing and revising a reactive layered charm to deliver this tutorial. Be prepared to write, revise, and re-deploy the application as we extend the charm.

## Clone the examples:

If you’re not a docker expert, that’s not a problem! All the code contained in this post is available online for you to clone and get started quickly. We’ll start with the voting-app example provided by the folks over at Docker

git clone https://github.com/docker/example-voting-app.git
cd example-voting-ap

### Build and run the voting app

The voting app has a slightly complex suite of services. It’s composed from:

- A Python webapp which lets you vote between two options
- A Redis queue which collects new votes
- A .Net worker which consumes votes from redis and stores them in a database.
- A Postgres database backed by a Docker volume
- A Node.js webapp which shows the results of the voting in real time

![Voting Application Architecture](/images/2016/aug/vote-app-architecture.png)

Test the deployment with docker-compose.

```bash
docker-compose up
```

The applications will be available on the IP of your docker host, on ports 5000 (vote), and 5001(results) respectively.

Feel free to interact with the application. Follow along with the logs. See the data flow from the web-services to the database, and then back out again with the results dashboard.

All of this with a single command locally. Pretty straight forward.

## We’re production ready now right?

What questions about our workflow have we answered so far?

- How we deliver and run our applications on one host

When it comes time to ship this to production, you could just send your docker-compose file to your ops team and hope for the best. But there’s a lot of missing information surrounding the actual deployment we’ve just examined from this single compose file.

- What do we do when we want to use an external Postgresql database?
- Are there common actions we will need to take against these applications? (maintenance)
- How do we monitor the suite of applications?
- How do we backup the database (because docker containers are ephemeral)?

It’s becoming obvious that we’ve only covered a small fraction of our production concerns. There’s a lot of operational knowledge that we can capture, and provide to the end operator/user. This is where Juju really shines, in modeling open source operations.

We’ll take the docker primitives we’ve verified above, and model it in a charm layer. Charm layers combine with other layers to build Operational Artifacts. Not just the containers, but lifecycle management, monitoring, and application relations.

## Modeling the Voting App as a Juju Charm

As a prereq, let’s ensure we have our build environment properly configured to make the examples consistent.

```bash
export JUJU_REPOSITORY=$HOME
export LAYER_PATH=$JUJU_REPOSITORY/layers
export INTERFACE_PATH=$JUJU_REPOSITORY/interfaces
mkdir -p $LAYER_PATH $INTERFACE_PATH
```

The first step to creating any Juju charm, is by creating a layer. Charm-tools has made this simple with the **charm create** command

```bash
$ cd $LAYER_PATH
$ charm create voting-app
INFO: Using default charm template (reactive-python). To select a different template, use the -t option.
INFO: Generating charm for voting-app in ./voting-app
INFO: No voting-app in apt cache; creating an empty charm instead.
Cloning into '/var/folders/75/7pkxbs4d13d_37y3cyn8j47r0000gn/T/tmp6fwUDN'...
remote: Counting objects: 22, done.
remote: Total 22 (delta 0), reused 0 (delta 0), pack-reused 22
Unpacking objects: 100% (22/22), done.
Checking connectivity... done.
```

We’ve just created an empty layer for us to start populating. The basic structure looks like the following:

```
├── README.ex
├── config.yaml
├── icon.svg
├── layer.yaml
├── metadata.yaml
├── reactive
│   └── voting_app.py
└── tests
    ├── 00-setup
    └── 10-deploy

```

First and foremost, let’s set our charms **metadata.yaml**. The metadata is important because It warehouses metadata for the Juju charm store to make your charm easy to find, and declares what your charm will connect to via relations.

We’ll take the basic template, and make it look similar to the following:

#### metadata.yaml

```yaml
name: voting-app
summary: multi-tiered voting app
maintainer: Juanita Bagodonuts <test@test.com>
description: |
  Deploys a 5 service application as standalone to perform voting and result
  analysis. Supports relations which will reconfigure the application from
  stand-alone to scale-out usage.
tags:
  - application-server
  - containers
subordinate: false
provides:
  results:
    interface: http
  voting:
    interface: http
requires:
  postgres:
    interface: pgsql
  redis:
    interface: redis
```

What we’ve defined above has declared to juju:

We have 2 potential website endpoints available. This could be be useful when setting up a reverse proxy.

We have additionally defined 2 potentially relatable external sources:

- PostgreSQL database
- Redis Data Store

At this point we have outlined these concerns for the charm:

- Scalable applications by virtue of being able to consume an external database and key/value store. This allows us to horizontally scale those services independently.
  - Additionally, we can scale our voting-app web tier via juju add-unit, and gain the benefit of scaling our dockerized application horizontally.

  However, declaring these points in metadata was only the first step. Let’s actually grab some helpful layers to make our final charm much smaller in terms of code we have to maintain. We do this in **layer.yaml**.

#### layer.yaml

  ```yaml
  includes:
  - 'layer:docker'
  - 'interface:pgsql'
  - 'interface:redis'
  - 'interface:http'
repo: https://github.com/charm-school/layer-docker-blog-samples
```

With this little bit of meta we have declared, we can build a charm from our layer and deploy it to exercise the model and how it will interact with services. It won’t do anything useful, but for reference, it looks like the following


![Voting app bundle in juju gui](/images/2016/aug/vote_app_jujugui.png)

We can further inspect what we were afforded by starting with layer-docker:

```bash
$ juju ssh voting-app/0

ubuntu@ip-172-31-29-112:~$ docker info
Server Version: 1.11.2
…
ubuntu@ip-172-31-29-112:~$ docker-compose --version
docker-compose version 1.7.1, build 6c29830
```


With just 4 declarations in layer.yaml we’ve gained:
 - Installation and setup of stable Docker Inc. tooling
 - Communication pipelines for postgres, redis, and HAProxy

 This leaves us only the delivery and concerns of our voting-app and how our containers should respond when being related to these external services.


## Layer-docker - Just enough batteries


Before we get into the implementation details, let’s review what has been created already. What we have now is a skeleton of a charm layer that is akin to a docker-compose file. We need to build the layer into a charm before it will deploy with one simple juju command. To model the whole stack we will actually use a compose file and some operational logic:

- Include the docker-compose file as a template
- Write logic to handle the deployment and configuration as a [reactive module](https://pythonhosted.org/charms.reactive/)

Our first order of business is to add the `docker-compose.yml` as a template in `templates/docker-compose.yml`

#### templates/docker-compose.yml

```yaml
version: "2"
services:
  vote:
    image: lazypower/vote-example
    command: python app.py
    ports:
      - "5000:80"
  redis:
    image: redis:alpine
    ports: ["6379"]
  worker:
    image: lazypower/worker-example
  db:
    image: postgres:9.4
  result:
    image: lazypower/result-example
    command: nodemon --debug server.js
    ports:
      - "5001:80"
      - "5858:5858"
```
> Notice: we have substituted the build directives for published images under my namespace. This was done for expediency and to  allow ENV vars to configure the extant services. This also carries a benefit of preventing vendoring of the code repositories into the charm. To keep things legit, here’s [what I modified](https://github.com/chuckbutler/example-voting-app/commit/68727c9c2044d74e46e22af424f67bb090f8dfad) to make this demo possible.

## Charms.docker - Model your docker operations with style

To ease charming with docker, we created the [charms.docker](https://github.com/juju-solutions/charms.docker) module to keep our reactive modules resembling the docker CLI as closely as possible. In the end, this is simply a CLI wrapper that helps encapsulate best practices when working with Docker primitives in charms. You have access to:


- Configuring the workload daemon via DockerOpts
- Launching workloads with the help of docker-compose
  - Manage lifecycles using compose primitives
  - Scale applications using those same primitives
  - Mutate workloads via templating
- Interact with docker via one-off commands
  - Log into a private registry
  - Execute ephemeral containers with Docker.run
- Manage and validate Docker workspaces

To learn more about charms.docker there are [API docs](http://pythonhosted.org/charms.docker/) published on pythonhosted.

Back to our voting-app layer:

With the compose template placed, we are now ready to start placing logic in the reactive module `reactive/voting_app.py`. In order to keep things tidy in here, we will leverage the charms.docker module from the docker layer to make interacting with Docker easier. The steps are:

- Render the docker-compose yaml into a workspace
- Launch the compose file with some syntax sugar
- Handle any cloud level network firewalling (opening/closing ports)

#### reactive/voting_app.py

```python
from charms.docker import Compose
from charms.templating.jinja2 import render
from charmhelpers.core.hookenv import config
from charmhelpers.core.hookenv import open_port
from charmhelpers.core.hookenv import status_set
from charms.reactive import when, when_not, set_state


@when('docker.available')
@when_not('voting-app.standalone.running')
def launch_standalone_formation():
    """ By default we want to execute the stand-alone formation """

    # By default, the render method looks in the `templates` directory
    # This defines src, tgt, and context.  Context is used for variable
    # substitution during the rendering of the template
    render('docker-compose.yml', 'files/voting-app/docker-compose.yml',
           config())
    # Define our compose object, initialize it with a workspace.
    compose = Compose('files/voting-app')
    # Launch the workload
    compose.up()
    # Declare ports to Juju - this requires a manual step of juju expose
    # otherwise the public facing ports never actually get exposed for traffic.
    open_port(5000)
    open_port(5001)
    # Set our idempotency state
    set_state('voting-app.standalone.running')
    status_set('active', 'Ready to vote!')
```

## Mutations aren’t only for super-heroes

We have successfully captured all the logic to deploy the standalone formation, with no mutations. This mirrors what we would do were we doing this from a workstation pointed at a docker enabled host. It’s a great first step to modeling the app. Let’s do the smallest change possible. Make the ports and images configurable:

#### config.yaml

```yaml
options:
  vote_image:
    type: string
    default: lazypower/vote-example
    description: Image to pull from the registry for the vote app
  result_image:
    type: string
    default: lazypower/result-example
    description: Image to pull from the registry for the result app
  worker_image:
    type: string
    default: lazypower/worker-example
    description: Image to pull from the registry for the result app
  redis_image:
    type: string
    default: redis:alpine
    description: Image to pull from the registry for the redis app
  postgres_image:
    type: string
    default: postgres:9.4
    description: Image to pull from the registry for the postgres app
  vote_port:
    type: int
    default: 5000
    description: Host port to bind the vote application
  result_port:
    type: int
    default: 5001
    description: Host port to bind the result application
```

With all our options defined, we will need to update the docker-compose defined workload to reflect the configuration values. To do this, I like to use [Jinja2 templating](http://jinja.pocoo.org/docs/dev/templates/), as its very straight forward and supports just enough logic to make it very useful.

#### docker-compose.yml

```yaml
version: "2"
services:
  vote:
    image: {{ vote_image }}
    command: python app.py
    ports:
      - "{{ vote_port }}:80"
  redis:
    image: {{ redis_image }}
    ports: ["6379"]
  worker:
    image: {{ worker_image }}
  db:
    image: {{ postgres_image }}
  result:
    image: {{ result_image }}
    command: nodemon --debug server.js
    ports:
      - "{{ result_port }}:80"
      - "5858:5858"
```

Note that we encapsulate the config keys as variables. This will become much clearer now that we are looking at the reactive module implementation.

#### reactive/voting_app.py snippet

```python
render('docker-compose.yml', 'files/voting-app/docker-compose.yml',
       config())
```

This line of the reactive module is reading config, and we pass that into jinja as a context. Each config key will populate and replace a variable. If no variable key is found, we will wind up with an error in our docker-compose logic. Ensure you’ve checked your keys and they are 1:1 with what’s listed in `config.yaml`.

One additional modification will be required to the reactive module, and its for the networking port(s). In our existing lines of code, we will change those hard coded ports, and add an additional method body to handle configuration changes:


#### reactive/voting_app.py snippet

```python
from charmhelpers.core.hookenv import close_port
from charms.reactive import when_any
from charms.reactive import remove_state

…

    open_port(config('vote_port'))
    open_port(config('result_port'))


@when_any('config.changed.vote_port', 'config.changed.result_port')
def recycle_networking_and_app():
    status_set('maintenance', 'Re-configuring port bindings.')
    # Close previously configured ports
    close_port(config.previous('vote_port'))
    close_port(config.previous('result_port'))
    # as the open port and app spinup are in another method, consume
    # that and tell juju to re-execute that method body by removing
    # the idempotency state. Docker-compose will run the diff and adjust
    # as required
    remove_state('voting-app.standalone.running')
```

Now that we have a stand alone formation, we can assemble the layers and and test our work.

```python
charm build -r --no-local-layers
juju deploy $JUJU_REPOSITORY/trusty/voting-app
juju expose voting-app
```

At this point, you should be able to visit the public url on port 5000 and vote, and on port 5001 see the results.

![Vote app screenie](/images/2016/aug/vote_app.png)
![Result app screenie](/images/2016/aug/result_app.png)

You can even adjust the configuration of the application at this point. Lets test this and see our port bindings change.

```bash
juju set voting-app vote_port=4000 result_port=4001
```

## Connect the dots, enable externally scalable applications

The first step to enabling external services, is to block our stand alone deployment. We don’t want to race when we’ve connected to these services. Additionally we will refactor our starting application code to a method to keep the reactive module DRY.

#### reactive/voting_app.py snippet

```python
@when('docker.available')
@when_not('voting-app.standalone.running')
@when_not('block_standalone')
def launch_standalone_formation():

…
    # Start our application, and open the ports
    start_application()

@when('docker.available', 'redis.available')
@when_not('postgres.connected', 'voting-app.running')
def replace_redis_container(redis):
    """ Prepare the data for the docker-compose template """
    # Block the stand alone profile
    set_state('block_standalone')
    status_set('maintenance', 'Configuring charm for external Redis.')
    hosts = []
    # iterate over all the connected redis hosts
    for unit in redis.redis_data():
        hosts.append(unit['private_address'])
    redis_host = ','.join(hosts)

    # Create a merged dict with the config values we expect
    context = {}
    context.update(config())
    context.update({'redis_host': redis_host})

    render('docker-compose.yml', 'files/voting-app/docker-compose.yml',
           context)

    start_application()
    status_set('active', 'Ready to vote!')
    # Set our idempotency state
    set_state('voting-app.running')


def start_application():
    compose = Compose('files/voting-app')
    # Launch the workload
    compose.up()
    # Declare ports to Juju - this requires a manual step of juju expose
    # otherwise the public facing ports never actually get exposed for traffic.
    open_port(config('vote_port'))
    open_port(config('result_port'))
```

We also will need to make some modifications to our docker-compose template. This becomes somewhat trivial with [Jinja templating](http://jinja.pocoo.org/docs/dev/templates/):

#### templates/docker-compose.yml snippet

```yaml
vote:
   image: {{ vote_image }}
   command: python app.py
 {% if redis_host %}
   environment:
     - REDIS_HOST={{ redis_host }}
 {% endif %}
   ports:
     - "{{ vote_port }}:80"
 {% if not redis_host %}
 redis:
   image: {{ redis_image }}
   ports: ["6379"]
 {% endif %}
 worker:
   image: {{ worker_image }}
   {% if redis_host %}
   environment:
     - REDIS_HOST={{ redis_host }}
   {% endif %}
```
Notice the conditional logic to populate ENV vars with our external host connection details.

Let’s assemble, and upgrade our charm in place to see the new behavior once we relate redis.

```bash
charm build -r
juju upgrade-charm voting-app --path=$JUJU_REPOSITORY/trusty/voting-app
juju deploy redis
juju add-relation voting-app:redis redis:db
```

The redis container is now substituted for the external redis application, running on another host. Independently scalable. Our docker-compose file was formatted to drop the container and you may see a warning about orphan containers. There’s a bug to track this behavior here.


We’ve addressed an external redis, let’s do the same for PostGreSQL. We’ll start with the reactive code for handling -only- when the postgres is connected (not redis).

#### reactive/voting_app.py snippet

```python
@when('postgres.database.available')
@when_not('redis.connected', 'voting-app.running')
def replace_postgres_container(postgres):
    """ Prepare the data for the docker-compose template """
    set_state('block_standalone')
    status_set('maintenance', 'Configuring charm for external Postgres.')
    # iterate over all the connected redis hosts
    pgdata = {'pg_host': postgres.host(),
              'pg_user': postgres.user(),
              'pg_pass': postgres.password(),
              'pg_db': postgres.database()}
    context = {}
    context.update(config())
    context.update(pgdata)
    render('docker-compose.yml', 'files/voting-app/docker-compose.yml',
           context)

    start_application()
    status_set('active', 'Ready to vote!')
```

#### Docker-compose.yml snippet

```yaml
worker:
   image: {{ worker_image }}
   {% if redis_host or pg_host %}
   environment:
     {% if redis_host %}
     - REDIS_HOST={{ redis_host }}
     {% endif %}
     {% if pg_host %}
     - PG_HOST={{ pg_host }}
     - PG_USER={{ pg_user }}
     - PG_PASS={{ pg_pass }}
     - PG_DB={{ pg_db }}
     {% endif %}
   {% endif %}
 {% if not pg_host %}
 db:
   image: {{ postgres_image }}
   {% if pg_host %}
   environment:
     - PG_HOST={{ pg_host }}
     - PG_USER={{ pg_user }}
     - PG_PASS={{ pg_pass}}
     - PG_DB={{ pg_db }}
   {% endif %}
 {% endif %}
 result:
   image: {{ result_image }}
   {% if pg_host %}
   environment:
     - PG_HOST={{ pg_host }}
     - PG_USER={{ pg_user }}
     - PG_PASS={{ pg_pass }}
     - PG_DB={{ pg_db }}
   {% endif %}
```

And finally, lets go ahead and implement the logic for when both redis and postgres are connected in our reactive module.

#### reactive/voting_app.py snippet

```python
@when('docker.available', 'postgres.database.available', 'redis.available')
def run_with_external_services(postgres, redis):
    set_state('block_standalone')
    # Grab redis data
    hosts = []
    # iterate over all the connected redis hosts
    for unit in redis.redis_data():
        hosts.append(unit['private_address'])
    redis_host = ','.join(hosts)
    # grab postgres data
    pgdata = {'pg_host': postgres.host(),
              'pg_user': postgres.user(),
              'pg_pass': postgres.password(),
              'pg_db': postgres.database()}

    # Create a merged dict with the config values we expect
    context = {}
    context.update(config())
    context.update({'redis_host': redis_host})
    context.update(pgdata)

    render('docker-compose.yml', 'files/voting-app/docker-compose.yml',
           context)

    start_application()
    status_set('active', 'Ready to vote!')
```

## Assemble, deploy, dominate

It’s been a bit windy to get here, but we now have a fully modeled operational model that supports standalone deployment, external db, and external redis. Instantly shareable to anyone, to deploy, relate, and interact with.  All that’s left is to assemble, deploy, and dominate your model with your newly assembled charm.

```bash
charm build -r
juju deploy $JUJU_REPOSITORY/trusty/voting-app
juju deploy redis
juju deploy postgresql
juju add-relation voting-app:redis redis:db
juju add-relation voting-app:postgres postgres:db
juju expose voting-app
```

## Homework if you’re still hungry to learn more


We’ve implemented the backend data stores as external relations. We did include the http interface, yet we didn’t actually use it. This was intentional. Now that we’ve shown you the classified “difficult” parts of the charm - I leave this challenge to those of you reading along at home to implement. Properly load balancing from the front end to the middle-ware layer is extremely simple with juju charms. And once you’ve done it for yourself once, you’ll have a reference implementation for every time.

Hints are:   See the [http-interface readme](https://github.com/juju-solutions/interface-http), and peruse the charm store for reverse proxy enabled services - like [HAProxy](https://jujucharms.com/haproxy/).

## Summation

We’ve illustrated how using layer-docker can ship a stellar, scalable, interconnected application ready to horizontally scale out for production usage. It doesn’t have to stop there though! There are many areas for additional improvement to this application, such as actions for data dump + restore, reverse proxy mentioned above, and even extending the usage to a windows charm to run the worker.

Congratulations on building your first layer-docker empowered charm, and any issues, questions, comments that you have about the experience are most welcome on the [juju mailing list](mailto:juju@lists.ubuntu.com) or the [project’s issue tracker](https://github.com/juju-solutions/layer-docker/issues) over on github.

## TL;DR

If reading isn’t your thing, the repository for the charm layer code is [here](https://github.com/charm-school/layer-docker-blog-samples). And you can deploy this solution instantly from my namespace:

```
juju deploy cs:~lazypower/voting-app
```

And follow along with the rest of the story above to relate the extant services, and see the changes happening in juju debug-log, the gui, or juju status.
