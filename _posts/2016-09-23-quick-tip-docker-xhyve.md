---
layout: post
title: "Quick Tip: Accessing Docker xvhyve machine!"
date: 2016-09-23 21:30:00
tags: ['docker', 'docker-compose', 'container', 'quicktip', 'xhyve']
description: "Docker is ~almost~ native on Mac OS X. In its later version, 1.12, the team implemented Docker Machine with xhyve by default. But how can I access it?"
comments: True
---

Hi everyone, a new post style is coming, called **Quick Tips**. Here, I will write very small posts on a very specific subject. This way I can write every single day, even when I'm *drunk*.

As you may know, Docker runs native on Linux because of its kernel, cgroups etc, which it is kinda bad for OS X and Windows users. They need to use a virtualized Linux box to execute Docker. This process was simplified by software like boot2docker and docker-machine. 

BUT, Docker 1.12 finally brings [xhyve](https://github.com/mist64/xhyve) support natively for Docker on OS X (and I guess Hyper-V support for Windows). What this means?

**The xhyve hypervisor is a port of bhyve to OS X. It is built on top of Hypervisor.framework in OS X 10.10 Yosemite and higher, runs entirely in userspace, and has no other dependencies. It can run FreeBSD and vanilla Linux distributions and may gain support for other guest operating systems in the future.**

This means *almost* native support. We still use a Linux distribution behind the scenes, but it is way more transparent to the final user.

So, here's come the Quick Tip: How can I access this xhyve virtualized machine? :thinking_face: GUI? *Nah*. SSH? *Nop*. Telnet? *Huh, nop*.

From now on, by default, you use a tool called [screen](https://www.gnu.org/software/screen/). It does a lot of things, but also connects to **tty**, and can be used to connect on the Linux machine. One issue tough, when I use screen inside [tmux](https://tmux.github.io/), I got a lot of gibberish. So you might need to leave the multiplexer if you are using one.

Enough words, give me the command!

{% highlight bash %}
# Get on the magic Linux box
$ screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty
{% endhighlight %}

If it asks a user login, just type `root` and hit enter. You'll see this:
{% highlight bash %}
Welcome to Moby
Kernel 4.4.20-moby on an x86_64 (/dev/ttyS0)

                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/

moby login: root
Welcome to Moby, based on Alpine Linux.
moby:~# 
{% endhighlight %}

I admit that sometimes I get stranger characters, but then I just restart Docker and everything goes smooth.

Well, this is it! You can run some Unix commands like `ifconfig`, `netstat` and `df` to explore the new environment. Also, any Docker command will run just fine!

---

I hope you enjoy this quick posts. They will be available at a random rate, specially when I drink to many beers :grin: If you have any doubts, drop a comment!
