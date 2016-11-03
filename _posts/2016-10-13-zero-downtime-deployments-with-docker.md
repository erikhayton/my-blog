---
layout: post
title: Zero-downtime deployments with Docker
---
Docker became in a couple of years the DevOps' new best friend. The goal of this article is not to present the multiple benefits of Docker, but rather to talk about an use case where Docker can really be useful: Zero-downtime deployments. In other words, the ability to update your online dockerized service (an app, a website, etc.) without any downtime.

To do so, we need at least two containers running our service (we'll call them <i>app</i>) and one container running our load balancer (we'll call him <i>lb</i>). The [dockercloud/haproxy](https://github.com/docker/dockercloud-haproxy) image is a great candidate for the load balancing part, as it embeds HAProxy and some useful commands to scale up and down our app containers. For the app, we'll simply use the [dockercloud/hello-world](https://github.com/docker/dockercloud-hello-world) image.

Basically, our [docker-compose.yml](https://github.com/BastienL/zero-downtime-deployment-with-docker/blob/master/docker-compose.yml) file will be:
{% highlight yaml %}
version: '2'
services:
  app:
    image: dockercloud/hello-world
  lb:
    image: dockercloud/haproxy
    links:
      - app
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - 80:80
      - 1936:1936
{% endhighlight %}

NB: we exposed port 1936 to have access to the HAProxy stats. You should remove it once in production.

Run the service (<i>docker-compose up -d</i>) and browse to <i>http://127.0.0.1</i>. Your service should be working!

Now, we are going to make use of a really useful command given by the dockercloud/haproxy image: <b>docker-compose scale</b>. This command creates new containers and automatically connects them to HAProxy. Great!

To try it, just type <i>docker-compose scale app=2</i> and see the magic happens! Now the load balancer must balance your visitors to one of the two running "app" containers (go to <i>http://127.0.0.1:1936</i> - user: stats, password: stats - if you want to make sure).

So basically, we could imagine that to update our app, we could simply scale up our app, as the new containers based on the new image will be created and connected to HAProxy and then scale it down to remove the old ones. Unfortunately, <i>docker-compose scale</i> does the opposite: it will remove the new ones and keep the old ones. Bummer!

To fix this, I created a simple [bash script](https://github.com/BastienL/zero-downtime-deployment-with-docker/blob/master/deploy.sh) that:
<ol>
<li>Builds the new app image</li>
<li>Scales up your app with new containers based on the new image</li>
<li>Sleeps (waiting for your app containers to be ready to accept connections)</li>
<li>Removes all the "old" app containers</li>
</ol>

To use it, just update the constants at the beginning of the file and run it. Now you don't have any downtime anymore during your deployments. Enjoy :)

If you want to give it a try, you can get the sources on my [repository](https://github.com/BastienL/zero-downtime-deployment-with-docker).

<small>PS: I'm aware some parts could be improved (using a health-check instead of a sleep for example), but I think this script is a good start. Let me know if you updated or improved it!</small>
