---
layout: post
title: "Composing Docker Environments: Scaling I"
date: 2016-09-28 23:30:00
tags: ['docker', 'docker-compose', 'compose', 'container', 'scale']
description: "Application scaling is the magic trick everyone is talking about. All the cool kids are scaling their environments up and down and using containers. Scaling Docker is not an exact science: you can do it by hand (but please don't) or use orchestration software like Kubernetes or Swarm. Today we are going to see how to scale Docker Compose using a single host :whale:"
comments: True
---

Want to get onboard on Docker Compose? Here's the past posts:

- [Composing Docker Environments](http://deployeveryday.com/2016/09/20/composing-docker-environments.html)
- [Composing Docker Environments: Services, Images and Builds](http://deployeveryday.com/2016/09/21/composing-docker-environments-services-images-builds.html) 
- [Composing Docker Environments: Networking](http://deployeveryday.com/2016/09/22/composing-docker-environments-networking.html)
- [Composing Docker Environments: Storage](http://deployeveryday.com/2016/09/26/composing-docker-environments-storage.html)

Application scaling is the magic trick everyone is talking about. All the cool kids are scaling their environments up and down sing containers. Scaling Docker is not an exact science: you can do it by hand (but please don't) or use orchestration software like [Kubernetes](http://kubernetes.io/) or [Swarm](https://docs.docker.com/swarm/). Today we are going to see how to scale Docker Compose using a single host :whale:

Even if it works pretty well in development, I think it lacks maturity for production. For that, be sure to check out the solutions above.

# docker-compose scale web=x
One command is all you need. Docker Compose scale its services using the `scale` command parameter, creating as many containers as you need. Using the code from our last [post](http://deployeveryday.com/2016/09/26/composing-docker-environments-storage.html), let's try it out:

{% highlight bash %}
# Get our stack running 
$ docker-compose up -d

# After a few seconds, scale the application
$ docker-compose scale web=5
WARNING: The "web" service specifies a port on the host. If multiple containers for this service are created on a single host, the port will clash.
Creating and starting composingdockerenvironments_web_2 ... error
Creating and starting composingdockerenvironments_web_3 ... error
Creating and starting composingdockerenvironments_web_4 ... error
Creating and starting composingdockerenvironments_web_5 ... error

ERROR: for composingdockerenvironments_web_2  Cannot start service web: driver failed programming external connectivity on endpoint composingdockerenvironments_web_2 (d2c5d510a70fd9ff596ce8518625e5af528c9886c912c3cb49e1bdf65eaf4a51): Bind for 0.0.0.0:8000 failed: port is already allocated
ERROR: for composingdockerenvironments_web_3  Cannot start service web: driver failed programming external connectivity on endpoint composingdockerenvironments_web_3 (d15f5d0afe5278e239dc976181b0e30aea8c53568a3094565bf3e68c4602cb6b): Bind for 0.0.0.0:8000 failed: port is already allocated
ERROR: for composingdockerenvironments_web_4  Cannot start service web: driver failed programming external connectivity on endpoint composingdockerenvironments_web_4 (ccc44ef9e2ff53ebaf24e2e51ae8a6752223a791ac30cdacc19972adcfa34e8d): Bind for 0.0.0.0:8000 failed: port is already allocated
ERROR: for composingdockerenvironments_web_5  Cannot start service web: driver failed programming external connectivity on endpoint composingdockerenvironments_web_5 (bd2d6aa89d7126cf9ce4b27619af1f1f4360f16bf468830abc5fb7f343b57238): Bind for 0.0.0.0:8000 failed: port is already allocated
{% endhighlight %}

*Oops!* What happened? We've tried to create five `web` containers and all of them bound to the 8000 port, which isn't supported on the host. Scaling has its perks :cry:

# Reverse proxy to the rescue!
We need one more piece to make it works, a *reverse proxy*. What it does is listen to one port - generally 80 - and *forward* all HTTP requests to a running web server, such as Django's built in. The most used out there is [HAProxy](http://www.haproxy.org/), [AWS ELB](https://aws.amazon.com/elasticloadbalancing/) and [NGINX](https://www.nginx.com/resources/admin-guide/reverse-proxy/). Techniques and features may vary between proxies, make sure to read the documentation carefully. Today the chosen one is NGINX.

Docker Compose scalability brings a big challenge: The reverse proxy needs to update its configuration dynamically for each container created or destroyed, adding or removing the container's IP address at any change. It can be achieved using [docker-gen](https://github.com/jwilder/docker-gen), which listens to Docker's API events and generate files based on templates. 

I'm not the first person to think about it: [this guy](https://github.com/jwilder) created a [nginx-proxy](https://github.com/jwilder/nginx-proxy) image to handle that. Neat.

# Scaling for real
To continue, make sure you have the latest code! Fortunately, we have [named volumes](http://deployeveryday.com/2016/09/26/composing-docker-environments-storage.html), so our data will be safe and sound.
{% highlight bash %}
# Go to the repo
$ cd /path/to/the/repository

# Get the new code
$ git pull
$ git checkout tags/v0.4

# Delete the containers
$ docker-compose down
$ docker-compose rm 

# Get our stack running 
$ docker-compose up -d
{% endhighlight %}

Access `http://localhost` (don't use port 8000) on your machine and you should see our simple counter.

See the differences in the new `docker-compose.yml`:

{% highlight yaml %}
# ...
  web:
  # ...
    # Now we expose the port to other containers instead for the host (port parameter)
    # This is needed to NGINX forward requests
    expose:
      - "8000"
    depends_on:
      - db
      # Our web depends on nginx too
      - nginx
    environment:
      # Image requirement, the virtual host name it will listen too
      # The virtual host is a group of hosts, containers or anything that responds HTTP
      VIRTUAL_HOST: 'localhost'
  # Our new image!
  nginx:
    image: jwilder/nginx-proxy
    volumes:
      # It needs to access Docker's API, so we mount the Unix socket
      - "/var/run/docker.sock:/tmp/docker.sock:ro"
    # Listens on port 80, accessed by our host on http://localhost
    ports:
      - "80:80"
# ...
{% endhighlight %}

Using `docker-compose ps`, we can see the three containers running:
{% highlight bash %}
               Name                              Command               State              Ports            
----------------------------------------------------------------------------------------------------------
composingdockerenvironments_db_1      /docker-entrypoint.sh postgres   Up      5432/tcp                    
composingdockerenvironments_nginx_1   /app/docker-entrypoint.sh  ...   Up      443/tcp, 0.0.0.0:80->80/tcp 
composingdockerenvironments_web_1     bash -c sleep 10 && python ...   Up      8000/tcp                    
{% endhighlight %}

OK, get ready, **let's scale**:

{% highlight bash %}
# Create more two containers
$ docker-compose scale web=3

# After a little while, run ps again
$ docker-compose ps
               Name                              Command               State              Ports            
----------------------------------------------------------------------------------------------------------
composingdockerenvironments_db_1      /docker-entrypoint.sh postgres   Up      5432/tcp                    
composingdockerenvironments_nginx_1   /app/docker-entrypoint.sh  ...   Up      443/tcp, 0.0.0.0:80->80/tcp 
composingdockerenvironments_web_1     bash -c sleep 10 && python ...   Up      8000/tcp                    
composingdockerenvironments_web_2     bash -c sleep 10 && python ...   Up      8000/tcp                    
composingdockerenvironments_web_3     bash -c sleep 10 && python ...   Up      8000/tcp                    

# See what happened accessing the logs (truncated version here)
$ docker-compose logs

# docker-gen generating our new NGINX configuration
nginx_1  | dockergen.1 | 2016/09/29 03:54:15 Generated '/etc/nginx/conf.d/default.conf' from 3 containers
nginx_1  | dockergen.1 | 2016/09/29 03:54:15 Running 'nginx -s reload'
...

# The last container coming up
web_3    | Performing system checks...
web_3    | 
web_3    | System check identified no issues (0 silenced).
jeb_3    | September 29, 2016 - 03:56:34
web_3    | Django version 1.10.1, using settings 'deployeveryday.settings'
web_3    | Starting development server at http://0.0.0.0:8000/
web_3    | Quit the server with CONTROL-C.
{% endhighlight %}

Again, access `http://localhost`, but this time, open a new terminal and run the command `docker-compose logs -f`. You should see that every request will be processed by a different `web`, based on the `web_x` part of the log. Awesome :heart_eyes:

You can also scaled down, decreasing the number of containers. 

Here, a snippet of the NGINX configuration responsible for this black magic:
{% highlight bash %}
...
# Virtual host definition with all containers
upstream localhost {
    ## Can be connect with "composingdockerenvironments_default" network
    # composingdockerenvironments_web_2
    server 172.20.0.6:8000;
    ## Can be connect with "composingdockerenvironments_default" network
    # composingdockerenvironments_web_1
    server 172.20.0.4:8000;
}

# NGINX listening on 80 and using the upstream above
server {
    server_name localhost;
    listen 80 ;
    access_log /var/log/nginx/access.log vhost;
    location / {
        proxy_pass http://localhost;
    }
}
{% endhighlight %}

---

The web is built on blocks, like these. Every simple application today may use a reverse proxy and a bunch of little web servers behind it, for scalability and resilience. Docker simplifies this process, letting you focus on your application!

As always, any feedback is appreciated! Share with your friends :metal:
