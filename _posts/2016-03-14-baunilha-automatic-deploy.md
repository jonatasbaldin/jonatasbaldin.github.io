---
layout: post
title: "Deploying Baunilha with Ansible"
date: 2016-03-14 09:00:00
tags:
  - "#python "
  - "#api "
  - "#ansible "
  - "#deploy "
description: "Nice way to deploy Baunilha, using Ansible."
comments: True
---

After creating [Baunilha](http://baunilha.deployeveryday.com/) and running inside Flask's own webserver, I faced the task to configure a proprer webserver and CGI to put it on production. Using NGINX and uWSGI step-by-step was pretty straightforward and I decided to automate it with [Ansible](https://www.ansible.com/).     
Creating infrastructure automation with Ansible is amazing, easy and pratically self documented. I already wrote how I did and instructions to deploy your own in the [Baunilha's Repository](https://github.com/jonatasbaldin/baunilha) so I won't do this again here.     

If you have any comments, recommendations or improvements, please leave a comment. Thank you!
