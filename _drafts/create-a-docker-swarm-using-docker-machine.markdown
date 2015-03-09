---
layout: post
title: "Create a Docker Swarm Using Docker Machine"
date: 2015-03-08T20:39:49-07:00
---

Assuming you have a docker machine already running, create a swarm token:

{% highlight bash %}
export SWARM_TOKEN=`docker run swarm create`
{% endhighlight %}

Then create your swarm master:

{% highlight bash %}
docker-machine create --d virtualbox --swarm --swarm-master --swarm-discovery token://$SWARM_TOKEN test-swarm
{% endhighlight %}

Then create other swarm nodes:

{% highlight bash %}
docker-machine create --d virtualbox --swarm --swarm-discovery token://$SWARM_TOKEN test-swarm-node1
docker-machine create --d virtualbox --swarm --swarm-discovery token://$SWARM_TOKEN test-swarm-node2
{% endhighlight %}

Now, load your swarm environment into your shell:

{% highlight bash %}
$(docker-machine env --swarm test-swarm)
{% endhighlight %}

And now you can view the status of your swarm:

{% highlight bash %}
➔ docker info
Containers: 4
Nodes: 3
 test-swarm: 192.168.99.109:2376
  └ Containers: 2
  └ Reserved CPUs: 0 / 4
  └ Reserved Memory: 0 B / 999.9 MiB
 test-swarm-node1: 192.168.99.110:2376
  └ Containers: 1
  └ Reserved CPUs: 0 / 4
  └ Reserved Memory: 0 B / 999.9 MiB
 test-swarm-node2: 192.168.99.111:2376
  └ Containers: 1
  └ Reserved CPUs: 0 / 4
  └ Reserved Memory: 0 B / 999.9 MiB
{% endhighlight %}

To test your swarm out:

{% highlight bash %}
docker run -d --name redis1 redis
{% endhighlight %}
