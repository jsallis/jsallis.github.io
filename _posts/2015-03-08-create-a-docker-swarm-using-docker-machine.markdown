---
layout: post
title: "Create a Docker Swarm Using Docker Machine"
date: 2015-03-08T20:39:49-07:00
---

[Docker Machine](https://docs.docker.com/machine) makes creating a [Docker Swarm](https://docs.docker.com/swarm) fairly trivial. In this post, we'll be creating a swarm with three nodes (a master and two additional nodes) using VirtualBox and then deploy some containers into the swarm to see how it behaves.

Assuming we have a docker machine already running and active, we first need to create a swarm token:

{% highlight bash %}
export SWARM_TOKEN=$(docker run swarm create)
{% endhighlight %}

Next, we create our swarm master, passing our token for discovery purposes:

{% highlight bash %}
docker-machine create --d virtualbox --swarm --swarm-master --swarm-discovery token://$SWARM_TOKEN test-swarm
{% endhighlight %}

Finally, we create two additional swarm nodes:

{% highlight bash %}
docker-machine create --d virtualbox --swarm --swarm-discovery token://$SWARM_TOKEN test-swarm-node1
docker-machine create --d virtualbox --swarm --swarm-discovery token://$SWARM_TOKEN test-swarm-node2
{% endhighlight %}

Running `docker-machine ls`, we can see that we now have three machines up and running in our __test-swarm__ swarm - our master and two additional nodes:

{% highlight text %}
NAME               ACTIVE   DRIVER       STATE     URL                         SWARM
dev                         virtualbox   Running   tcp://192.168.99.100:2376
test-swarm                  virtualbox   Running   tcp://192.168.99.104:2376   test-swarm (master)
test-swarm-node1            virtualbox   Running   tcp://192.168.99.105:2376   test-swarm
test-swarm-node2   *        virtualbox   Running   tcp://192.168.99.106:2376   test-swarm
{% endhighlight %}

We can load our swarm environment into our shell:

{% highlight bash %}
eval "$(docker-machine env --swarm test-swarm)"
{% endhighlight %}

Now, running `docker info` will output the status of our swarm:

{% highlight text %}
Containers: 4
Nodes: 3
 test-swarm-node2: 192.168.99.106:2376
  └ Containers: 1
  └ Reserved CPUs: 0 / 4
  └ Reserved Memory: 0 B / 999.9 MiB
 test-swarm: 192.168.99.104:2376
  └ Containers: 2
  └ Reserved CPUs: 0 / 4
  └ Reserved Memory: 0 B / 999.9 MiB
 test-swarm-node1: 192.168.99.105:2376
  └ Containers: 1
  └ Reserved CPUs: 0 / 4
  └ Reserved Memory: 0 B / 999.9 MiB
{% endhighlight %}

To test our swarm out, we'll bring up a few [docker-uhttpd](https://github.com/fnichol/docker-uhttpd) containers:

{% highlight bash %}
docker run -d -p 80 --name uhttpd1 fnichol/uhttpd
docker run -d -p 80 --name uhttpd2 fnichol/uhttpd
docker run -d -p 80 --name uhttpd3 fnichol/uhttpd
docker run -d -p 80 --name uhttpd4 fnichol/uhttpd
{% endhighlight %}

Running `docker ps`, we can see that the containers have been distributed throughout the swarm, as indicated by the container names being prefixed with the name of the swarn node on which they running (ie. `test-swarm-node2/`):

{% highlight text %}
CONTAINER ID        IMAGE                   COMMAND                CREATED             STATUS              PORTS                          NAMES
d4a97ceeecb8        fnichol/uhttpd:latest   "/usr/sbin/run_uhttp   47 seconds ago      Up 28 seconds       192.168.99.105:49153->80/tcp   test-swarm-node1/uhttpd4
40e40e2a04a3        fnichol/uhttpd:latest   "/usr/sbin/run_uhttp   49 seconds ago      Up 44 seconds       192.168.99.104:49154->80/tcp   test-swarm/uhttpd3
60ff5aa201ad        fnichol/uhttpd:latest   "/usr/sbin/run_uhttp   52 seconds ago      Up 47 seconds       192.168.99.104:49153->80/tcp   test-swarm/uhttpd2
e70c71a86b6a        fnichol/uhttpd:latest   "/usr/sbin/run_uhttp   56 seconds ago      Up 46 seconds       192.168.99.106:49153->80/tcp   test-swarm-node2/uhttpd1
{% endhighlight %}

Additionally, running `docker info` again will outline how many containers are currently running on each node:

{% highlight text %}
Containers: 8
Nodes: 3
 test-swarm: 192.168.99.104:2376
  └ Containers: 4
  └ Reserved CPUs: 0 / 4
  └ Reserved Memory: 0 B / 999.9 MiB
 test-swarm-node1: 192.168.99.105:2376
  └ Containers: 2
  └ Reserved CPUs: 0 / 4
  └ Reserved Memory: 0 B / 999.9 MiB
 test-swarm-node2: 192.168.99.106:2376
  └ Containers: 2
  └ Reserved CPUs: 0 / 4
  └ Reserved Memory: 0 B / 999.9 MiB
{% endhighlight %}

As you can see, it's fairly straightforward to bring up a swarm using Docker Machine. The nice part is you can run this on Azure, EC2, Rackspace, or any other platform supported by Docker Machine simply by changing your driver.

#### Additional Resources

- [Docker Machine on GitHub](https://github.com/docker/machine)
- [Docker Swarm on GitHub](https://github.com/docker/swarm) 
