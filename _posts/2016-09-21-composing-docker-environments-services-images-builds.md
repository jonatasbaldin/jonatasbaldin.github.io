---
layout: post
title: "Composing Docker Environments: Services, Images and Builds"
date: 2016-09-21 08:00:00
tags: ['docker', 'docker-compose', 'compose', 'container', 'services', 'images', 'builds']
description: "Today we continue talking about Docker Compose, following by the last post. We are going to lay out a foundation to build our environment, commit by commit, and also explore the concepts of services, images and builds. :whale:"
comments: True
---

Today we continue talking about Docker Compose, following the [last post](http://deployeveryday.com/2016/09/20/composing-docker-environments.html). We are going to lay out a foundation to build our environment, commit by commit, and also explore the concepts of services, images and builds. :whale:

# git commit -m 'first commit'
Instead of throwing code snippets over these pages, I'll use [this GitHub repository](https://github.com/jonatasbaldin/composing-docker-environments) to store and organize our journey. This is not something planned, neither do I have a draft, but I'll try to put everything together commit by commit. If you have any tip during the read, drop a comment and I'll be glad to discuss it.

# The foundation
To start, clone the repository and run the command:
{% highlight bash %}
git clone https://github.com/jonatasbaldin/composing-docker-environments.git
cd composing-docker-environments
docker-compose up
{% endhighlight %}

If everything goes well, you should be able to see the Django Hello World page at `http://localhost:8000`. Otherwise, yell at me in the comments or create a issue in GitHub.

Oh, this is the versions I'm running right now:
{% highlight bash %}
# docker -v
Docker version 1.12.1, build 6f9534c
# docker-compose -v
docker-compose version 1.8.0, build f3628c7
# OS -v
Mac OS X Yosemite 10.10.5
{% endhighlight %}

# To the content!
So, *services*, *images* and *builds*. What is these things?  
In Docker Compose, each stack of your application is a *service*. It can be your framework, database, cache service, web server, proxy and so on. The service has its own configuration, like exposed ports, mounted volumes and entrypoint commands. The neat about services is that it can be scaled! One web server can't handle the traffic? Throw more ten of them!

On our current environment, we got two services: `db` and `web`, the first being an *image* and the last, a *build*. Excuse me?

The first is easy! `db` is a Postgres image pulled from [Docker Hub](https://hub.docker.com/_/postgres/) and executed without any customization. It just pulls and run. In the image documentation you can read about how it is built and various ways configure it.

On the other hand, `web` is a *build* created with a Dockerfile, that is what this parameter means. The accepted parameters can be just a directory with a Dockerfile, a `context` with a alternative file and even a mix of `build` and `image`, where Compose will create a build named by the image parameter value. See below:
{% highlight yaml %}
# Builds an image from web/Dockerfile
web:
  build: ./web 

# Builds an image from web/Dockerfile-production
web:
  build:
    context: ./web
    dockerfile: Dockerfile-production

# Builds an image from web/Dockerfile and name it myweb with v1 tag
web:
  build: ./web
  imagem: myweb:v1
{% endhighlight %}

The Dockerfile is exactly the same from Docker. You can view the one I'm using [here](https://github.com/jonatasbaldin/composing-docker-environments/blob/master/web/Dockerfile).

---

This is it for today folks! I hope your understanding of services, images and builds are clearer now :smile: If you got any issue, let me know!
