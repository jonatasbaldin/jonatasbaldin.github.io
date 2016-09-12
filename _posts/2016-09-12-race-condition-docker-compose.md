---
layout: post
title: "Avoiding race condition on Docker Compose"
date: 2016-09-12 09:30:00
tags: ['race condition', 'docker', 'docker-compose', 'python', 'django', 'postgres']
description: "When using multiple containers approach with Docker Compose, we often get race condition issues, where the application server depends on the database, but the first one ends up running too early and may break the stack. Here, we're gonna see a Python way to solve this problem."
comments: True
---

When using multiple containers approach with Docker Compose, we often get race condition issues, where the application server depends on the database, but the first one ends up running too early and may break the stack. Here, we're gonna see a Python way to solve this problem.

## Django + Postgres, a race condition example
One of the most used stacks in the Python community is the Django web framework with Postgres database. Making the use of Docker Compose, it's easy to create both in separate containers, like this:

{% highlight yaml %}
# docker-compose.yml

version: '2'
services:
  db:
    image: postgres
    ports:
      - "5432:5432"
    volumes:
      # Make DB data persistent on the host
      - "./docker/db/pgdata:/var/lib/postgresql/data"
    environment:
      POSTGRES_PASSWORD: '123456A!'
  web:
    build: ./docker/web
                     # Avoid race condition and get django up
    command: bash -c "python docker/web/check_db.py --service-name Postgres --ip db --port 5432 &&
                      python manage.py migrate &&
                      python manage.py runserver 0.0.0.0:8000"
    volumes:
      - ".:/code"
    ports:
      - "8000:8000"
    depends_on:
      - db
{% endhighlight %}

Here, we've created two services with basic configuration, exposing the right ports and mouting our local project directory on `/code` inside the Django container. To build the Django container, I used the following Dockerfile:

{% highlight bash %}
# docker/web/Dockerfile

FROM python:3.4
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
ADD requirements.txt /code/
RUN pip install -r requirements.txt
{% endhighlight %}


## Python Script to the rescue
Now, the tricky part. When we run `docker-compuse up` to start our containers, possbily the web container will come up before the db. Django will try to execute its migrations and fail with a database connection error. To avoid that, we'll use the `check_db.py` script, listed as first command to be executed on the web service. It will test the database port and exits when it is open. Here's the little guy:

{% highlight python %}
# docker/web/check_db.py 

import socket
import time
import argparse
""" Check if port is open, avoid docker-compose race condition """

parser = argparse.ArgumentParser(description='Check if port is open, avoid\
                                 docker-compose race condition')
parser.add_argument('--service-name', required=True)
parser.add_argument('--ip', required=True)
parser.add_argument('--port', required=True)

args = parser.parse_args()

# Get arguments
service_name = str(args.service_name)
port = int(args.port)
ip = str(args.ip)

# Infinite loop
while True:
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    result = sock.connect_ex((ip, port))
    if result == 0:
        print("{0} port is open! Bye!".format(service_name))
        break
    else:
        print("{0} port is not open! I'll check it soon!".format(service_name))
        time.sleep(3)
{% endhighlight %}

Now when you run `docker-compose up`, the script's output should appear on the logs:

{% highlight bash %}
web_1  | Postgres port is not open! I'll check it soon!
db_1   | LOG:  database system was shut down at 2016-09-12 12:27:04 UTC
db_1   | LOG:  MultiXact member wraparound protections are now enabled
db_1   | LOG:  database system is ready to accept connections
db_1   | LOG:  autovacuum launcher started
web_1  | Postgres port is open! Bye!
db_1   | LOG:  incomplete startup packet
web_1  |   No migrations to apply.
web_1  | System check identified no issues (0 silenced).
web_1  | September 12, 2016 - 12:27:27
web_1  | Django version 1.9.8, using settings 'app.settings'
web_1  | Starting development server at http://0.0.0.0:8000/
web_1  | Quit the server with CONTROL-C.
{% endhighlight %}

The first and sixth lines show our script in action,  holding the container until the database is up and running.

## Wrapping up
If your container have Python 3, you can easily use the same script. If not, it shouldn't be hard to code the same logic on your prefered language, just create an infinite loop to check the port and break it when your service is up.
