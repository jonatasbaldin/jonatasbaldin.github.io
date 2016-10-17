---
layout: post
title: "Composing Docker Environments"
date: 2016-09-20 08:00:00
tags: ['docker', 'docker-compose', 'compose', 'container', 'django', 'postgres']
description: "Docker is awesome, but running containers by hand is awful. You can improve your environment using docker-compose to create multi-container applications."
comments: True
---

*This is a blog series! You can find the other posts at the links below*:

- [Composing Docker Environments: Services, Images and Builds](http://deployeveryday.com/2016/09/21/composing-docker-environments-services-images-builds.html)
- [Composing Docker Environments: Networking](http://deployeveryday.com/2016/09/22/composing-docker-environments-networking.html)
- [Composing Docker Environments: Storage](http://deployeveryday.com/2016/09/26/composing-docker-environments-storage.html)
- [Composing Docker Environments: Scaling I](http://deployeveryday.com/2016/09/28/composing-docker-environments-scale.html)

Docker is awesome, but running containers by hand is awful. You can improve your environment using docker-compose to create multi-container applications.

# You container is (almost) never alone
Rarely you run your application with one service, at least you generally need a Web[language,framework] and a database. There you go, two services. And this can be done in a large variety of ways:

- Installing everything locally (bad, don't!)
- Using virtual environments if your stack allows (cool, but read on)
- Virtual machines, generally backed up with a service like Vagrant (nice, but maybe overkill)

**Or containers!** :whale:

Docker containers are lightweight systems running as an isolated process over your kernel. Not going too deep, as this subject needs its own post and you can find [a lot](https://www.google.com.br/?q=docker) of content about it.

One challenge of running containers its their orchestration. They are so ephemeral, that putting your stack together can take out the fun of it. Enters `docker-compose`, a tool to address this issue.

# Development, almost production
Before showing you some code, I should say that Docker Compose is *almost* production ready. I mean, [people are doing it](https://github.com/docker/compose/issues/1264#issuecomment-93604915), but it lacks some production features to reduce risk and downtime. Meanwhile, its last version supports Docker Swarm cluster and is introducing Docker Stacks and Distributed Application Bundles, so things are about to get better on Compose land.

That said, the [official documentation](https://docs.docker.com/compose/) outlines these common use cases: Development environments, automated testing services and single host deployments.

# Development it is!
Compose use a YAML file named `docker-compose.yml` with directives that allows you to configure every container aspect, including network, ports, volumes etc. In this file you specify you can specify your entire application in a few lines and bring everything up with a simple command, `docker-compose up`. In a development environment, it reduces the time to get people on boarding and simplify your README.md :)

A quick, dirty and commented Django + PostgreSQL is shown below:
{% highlight yaml %}
# Compose file version (v1 is legacy)
version: '2'
# Each part of your stack is represented by a service, which can be scaled
services:
  # Service name, known by the other services
  db:
    # Uses default image from registry
    image: postgres
    # Exposes the port to the host
    ports:
      - "5432:5432"
    # Mount persistent data on the host
    volumes:
      - "./docker/db/pgdata:/var/lib/postgresql/data"
    # Define environment variables
    environment:
      POSTGRES_PASSWORD: '123456A!'
  web:
    # Builds image based on a Dockerfile in this directory
    build: ./docker/web
    # The entrypoint command
    command: python manage.py runserver 0.0.0.0:8000"
    volumes:
      - ".:/code"
    ports:
      - "8000:8000"
    # Will come up after db service
    depends_on:
      - db
{% endhighlight %}

For completeness, the `Dockerfile` inside `docker/web/`.

{% highlight bash %}
FROM python:3.4
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
ADD requirements.txt /code/
RUN pip install -r requirements.txt
{% endhighlight %}

With these files, just run `docker-compose up` and your application will be access-ready on `http://localhost:8000`. Note that, as the [postgres image documentation](https://hub.docker.com/_/postgres/) points out, this configuration will create a database named `postgres` with a `postgres` username and password set as `123456A!`. Make sure your Django app connect to that!

Awesome, right!? You built a multi-container app with one command! :rocket:

But do not think that's all! In the next days, I'll bring more Compose features and examples, stay tuned!
