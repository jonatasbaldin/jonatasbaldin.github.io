---
layout: post
title: "Composing Docker Environments: Storage"
date: 2016-09-26 22:00:00
tags: ['docker', 'docker-compose', 'compose', 'container', 'storage']
description: "Docker storage is a huuuuuuuge topic. It includes images, containers, drivers, volumes, layered file system and so on. If you want to start mastering your data, this is the right place! :whale:"
comments: True
---

Want to get onboard on Docker Compose? Here's the past posts:

- [Composing Docker Environments](http://deployeveryday.com/2016/09/20/composing-docker-environments.html)
- [Composing Docker Environments: Services, Images and Builds](http://deployeveryday.com/2016/09/21/composing-docker-environments-services-images-builds.html) 
- [Composing Docker Environments: Networking](http://deployeveryday.com/2016/09/22/composing-docker-environments-networking.html)

Docker storage is a huuuuuuuge topic. It includes images, containers, drivers, volumes, layered file system and so on. I'll write expose a little of these concepts and focus on what they mean in Docker Compose, but if you are new to the subject, I recommend you to see the following links:

- [Understand images, containers, and storage drivers - Docker documentation](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/)
- [Manage data in containers - Docker documentation](https://docs.docker.com/engine/tutorials/dockervolumes/)
- [Select a storage driver - Docker documentation](https://docs.docker.com/engine/userguide/storagedriver/selectadriver/)
- [Docker Storage and Volumes Deep Dive and Considerations - By Brian Goff on Dockercon 16](https://www.youtube.com/watch?v=X_q2l8hotAc)

:whale:

# A informational snippet
Putting all together: Docker uses a pluggable storage system, allowing you to change how it interacts with data. This system may use filesystem (generally default to AUFS) or block storage. Images are built with layers identified by a cryptographic hash, created by each new line in a Dockerfile. These layers are shared, so if you use the same `ubuntu` image for 30 containers, the base image will be used once. On top of that, Docker creates a *container layer* to hold your modified data, but this layer is ephemeral, meaning that it will be deleted when the container dies. If you truly love your bytes, volumes can be used to store them persistently. 

PS: Once upon a time, *data-only containers* [were used](https://github.com/docker/docker/issues/17798) to store persistent information. With named volumes, there's no need for that anymore.

Seriously, read the links above, they explain much better what I just said :sweat_smile:

To get your Docker's storage information, run the following:
{% highlight bash %}
$ docker info
...
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 407
 Dirperm1 Supported: true
...
{% endhighlight %}

# New stuff
Until our last post, we were just mounting volumes, but now we are going to use those named ones. Checkout the new code version on git:
{% highlight bash %}
# Go to the repo
$ cd /path/to/the/repository
# Get the new code
$ git pull
$ git checkout tags/v0.3
# Remove our last mounted volume
$ docker-compose rm db
# Get our stack running 
$ docker-compose up -d
# After a few seconds, run Django migration again
$ docker-compose run web python manage.py migrate
{% endhighlight %}

Again, access `http://localhost:8000` on your machine and you should see our simple counter.

Let's inspect our storage configuration:
{% highlight yaml %}
# Snippet from docker-compose.yml
...
  db:
    # Create a named volume to store persistent data, without the inital dot
    volumes:
      - data-postgres:/var/lib/postgresql/data

  web:
    volumes:
      - "./deployeveryday:/code"

# Need to describe named volumes, weird, I know
volumes:
  data-postgres:
{% endhighlight %}

On `db` we created a named volume called `data-postgres`. This will create the folder `/var/lib/docker/volumes/composingdockerenvironments_data-postgres` in our Docker host and will map to `/var/lib/postgresql/data` inside the container (this is the storage path to Postgres image as described in the documentation). This way, we can destroy entirely our loved container and the data will remain alive. We can also map the same volume on any other container to use the data.     

Note that your application must deal with file locks to avoid data corruption. Also, the volume name is created after the Docker Compose project, the directory holding `docker-compose.yml` file.

A benefit of named volumes is backup, now that our data path has a proper name (instead of old hashes), it is easier to copy our data to a safe place.

*I hate my data, delete it!* To get rid of our counter (or any other named volume) you need to stop the containers and remove the volume:
{% highlight bash %}
# Destroying the container
$ docker-compose rm
# Deleting the volume
$ docker volume rm composingdockerenvironments_data-postgres
{% endhighlight %}

Note that Docker won't delete your data if you just stop the container, you must delete it manually! You can find *orphans* volumes to be deleted with the command below:
{% highlight bash %}
$ docker volume ls -f dangling=true
{% endhighlight %}

And finally, our `web` friend just maps a local folder to the container, used when we want to put our files inside the container after its built and see modifications in real time. This does not create a new volume, just make the mapping.

---

As I said, storage is huge! This article just covers its surface, so be sure to read the documentation if you want a full packed knowledge. Thank you for your time and if you got any issues, drop a comment!
