---
layout: post
title: "Cloud Inception with Ravello Systems"
date: 2016-01-20 15:00:00
tags:
  - "#cloud "
  - "#ravello "
  - "#nested "
  - "#amazon "
  - "#gpc "
  - "#api "
  - "#python "
share: "Share this, pls"
comments: True
---

Playing around with cloud computing is much more fun with [Ravello Systems](https://www.ravellosystems.com/). Why? Just read they description on Google:

*Easily run any virtual environment lab (VMware / KVM) with any networking topology in AWS or Google cloud without modifications using nested virtualization.*

Nice, right? Now you can have your homelab on the cloud! Well, maybe just part of it, most of us still like cables lying around and noisy fans. But if you want to go down that road, [here](http://www.virtuallanger.com/2015/07/09/can-you-replace-your-home-lab-with-ravello-systems/) is an interesting article discussing if is worth it.

## What can I do with Ravello? 
You can run VMware with VSAN technology, nested KVM and a complete OpenStack. With L2/L3 networking, you might try running a board firewall like pfSense and test Cumulus Linux Networking. Do you wanna test configuration management? Just fire up some VMs and install Puppet or Ansible. Wanna try some NAS services? There's a ready to deploy FreeNAS in the repository.     
The possibilities that Ravello opens to lab, study and POC are huge.

## The Principles
# Nested
Ravello can run nested hypervisors due its distributed hypervisor infrastructure, called [HVX](https://www.ravellosystems.com/technology/hvx), which is basically a layer that encapsulates multi-VM application inside existing clouds.

# Applications
Applications are the projects inside Ravello. Here you drag and drop VMs, customizing them as you want. After the application is finished, it can be *published* into the cloud, where the fun beggins.

# VMs
Inside the application reside the VMs. They are presented as small boxes in a canvas area. There are some ways to create them:     

* Ravello offers they own, including Ubuntu 14.04 with qemu-kvm pre-installed, an Empty VMware ESXi, VMs with software builtin and others.
* [Ravello Repo](https://www.ravellosystems.com/repo) has  blueprints and VMs shared by other users.
* Upload an ISO image and bootstrap the SO from an empty VM.
* And even upload your VM directly from your virtualization infrastructure! That's right, you can take a production VM and export it fully to your cloud lab. Simple download a tool (CLI or GUI), that will guide you through the process.

After creating or importing some VMs, you may edit its configuration, like CPU, memory, network and disk. Also, most VMs requires a keypair to be accessed from the Internet, which you can create on the fly or upload your own.    
If the virtual machine has a public IP, you should use the 'Services' area to open the necessary ports, behaving like a firewall.

# Blueprints
After your application is ready, it's possible to share it with other users with *blueprints*. This is basically a snapshot of the entire infrastructure (VM, disk, configuration). It is amazing how you can produce a POC and share with anyone to test it.

# Pricing
Behind the scenes, Ravello spins up VMs in Google Cloud Plataform or Amazon AWS. You can choose manually or play in *best pricing* mode, where it will pick up the best price available.     
There's a price calculator [here](https://www.ravellosystems.com/pricing) to help you out.

# Trial
To test Ravello, a trial account can be used for 15 days, just create it with a corporate email and start playing around.

# API
As a mature application, Ravello offers a REST API to manage all of its features. [Here](https://www.ravellosystems.com/ravello-api-doc/) is the documentation for more details.     
For Python lovers, you might use the [python-sdk](https://github.com/ravello/python-sdk), which is a thin layer between the API and the language.     
In the next post, I'll bring an example using `python-sdk` to create an entire application.

## Time to Play
Now that you know the basics, get your trial account and spin up some application. Below is some screenshots of the process.  

<center>
![ravello_application](/img/rav_create_application.png)     
*Creating an application*

![ravello_application](/img/rav_canvas.png)     
*Draggin VMs into the canvas*

![ravello_application](/img/rav_settings.png)     
*Tweaking VM settings*

![ravello_application](/img/rav_publish.png)     
*Publishing an application*

![ravello_application](/img/rav_published.png)     
*Application published, VM FQDN address in the right bottom*
</center>

I repeat, you must try it out! Ravello System is an amazing place to create and test a full-stack application, from the virtualization layer to the end user interface.
