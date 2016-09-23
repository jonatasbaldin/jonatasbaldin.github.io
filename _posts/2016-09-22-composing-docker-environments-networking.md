---
layout: post
title: "Composing Docker Environments: Networking"
date: 2016-09-22 23:30:00
tags: ['docker', 'docker-compose', 'compose', 'container', 'networking']
description: "Web framework? Got it! Database? Yep! Today we are going to see how can we link both services through the Docker network. It can a be pretty simple and quick configuration, but it also exposes a mature surface to be explored. :whale:"
comments: True
---

Web framework? Got it! Database? Yep! Today we are going to see how can we link both services through the Docker network. It can be a pretty simple and quick configuration, but it also exposes a mature surface to be explored. :whale:

# Previously on Deploy Everyday
If you are arriving here for the first time, make sure you also read the [introduction on Docker Compose]() and what [services, images and builds]() mean. It's a quick and (I hope) fun post :smile:

Getting started: jump to the following Git tag:
{% highlight bash %}
$ git checkout tags/v0.2
{% endhighlight %}

Also, as Django needs `psycopg2`, our Docker `web` image needs to be rebuilt. You can do that with:
{% highlight bash %}
# Build our image, but don't run any containers
$ docker-compose build
{% endhighlight %}


# Talk network to me
Computers network exists since, well, a lot of time. Modern web applications (and the old ones) depend heavily on them. Databases need to be connected to applications, APIs must answer HTTP requests, caches need to cache and proxies need to proxy. With Docker Compose, it's easy to get all components working. 
In this post, we are going to connect Django web framework with a Postgres database, as they are by nature great friends.

To get our database powered application up and running, type this at the repository root:
{% highlight bash %}
# Get our containers up in background
$ docker-compose up -d

# And configure Django, don't bother with this
# Create Django SQL migrations
$ docker-compose run web python manage.py makemigrations

# Run the SQL commands to create our database
$ docker-compose run web python manage.py migrate
{% endhighlight %}

After a couple seconds, open in your browser the address `http://localhost:8000`. You'll see a simple counter being incremented at every request. Neat.

# docker-compose.yml barely changed!
Yep, I know. If you see the diff, the only parameters changed in our `docker-compose.yml` file is:
- An `environment` variable to define our database password.
- Change the `web` command to avoid race condition (what was done here is a poor implementation, see [this article]() for a best practice!).
- Tell out loud that `web` `depends_on` `db`. This does not provide any network configuration, just says that `web` will not run without a `db`.

The magic is: Docker Compose creates a default network out of the box, named by the project name (based on the project's directory). To see all networks and their details, use these commands:


{% highlight bash %}
# Get all Docker networks (you output you certainly be different)
$ docker network ls
NETWORK ID          NAME                                     DRIVER              SCOPE
3cf1cab64833        bridge                                   bridge              local
c17ce9330275        composingdockerenvironments_default      bridge              local
3eece4f29a8b        host                                     host                local
4f86206f5da0        none                                     null                local

# Inspect the composingdockerenvironments_default network
$ docker network inspect c17ce9330275
[
    {
        "Name": "composingdockerenvironments_default",
        "Id": "c17ce93302757bc858b5678564aabc67263959094ccffa891792937f1dedb704",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "Gateway": "172.20.0.1"
                }
            ]
        },
        "Internal": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]

{% endhighlight %}

As the output shows, the network driver is a `bridge` which allows communication between the container and its host (machine it lives on). It also has information about the IP addressing, scope, ID etc.    
Another awesome trick Docker does is to create an entry in `/etc/hosts` for all containers, mapping the service name to its IP address. What this means? Our web application can connect with the database by using something like `postgres://db:5432`, as configured in our Django app:

{% highlight python %}
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'PASSWORD': '123456A!',
        'HOST': 'db',
        'PORT': '5432',
    }
}
{% endhighlight %}

# Links, expose and ports
More cool stuff can be done with networking!

`links` creates aliases to make a service reachable by another name, `expose` *exposes* ports between the linked services and `ports` *exposes* a container port to a host port (`HOST:CONTAINER`), as we are already doing with the web application.

{% highlight bash %}
# A tiny snippet
...
  web:
    links:
      - "db:postgres_db"
    expose:
      - 5432
{% endhighlight %}

And to complete, Compose also allows to use custom networks and multi-hosting deployment, but this is a topic for another day :wink:

---

**Awesome**, right?! Compose makes Docker networking stack pretty easy to use! If you have any doubts, drop a comment and I'll be glad to chat!
