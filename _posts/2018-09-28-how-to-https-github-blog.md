---
layout: post
title: "How to add HTTPS to your GitHub hosted blog"
date: 2018-09-29 14:00:00
tags: ["security", "http", "tls"]
description: "Today, Google Chrome said my blog was insecure! That happened because I was running by website over just plain text HTTP. This is how I fixed it."
image: /img/https_blog.png
---

Today, Google Chrome said my blog was insecure! Can you believe it? That happened because I was running by website sonely on HTTP. This is how I fixed it.

### The HTTP-only Stack
First, here's how I run my blog before having HTTPS:

* The content is hosted in a GitHub repository with Jekyll. Yeah, you can run a static web site directly from GitHub. You can see more information [here](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/).
* My domain, _deployeveryday.com_, was bought at [GoDaddy](https://godaddy.com).
* DNS (the service that tells the world where my site lives) is also managed by GoDaddy.

### Why should I use HTTPS?
Since July this year, Google Chrome started flaggin HTTP-only sites as insecure, because, well, they _are_ insecure. With HTTP, all data exchange between users and servers are transported in plain-text, which means everyone can potentially see how are you interacting with online services, including credit card details and your credentials.

On the other hand, HTTPS adds one more layer of security: encryption. With an [asymmetric encryption system](https://www.youtube.com/watch?v=AQDCe585Lnc) – using a private/public key – it guarantees that all communication between you and the services implementing HTTPS is protected.

> Eventually, our goal is to make it so that the only markings you see in Chrome are when a site is not secure, and the default unmarked state is secure. We will roll this out over time, starting by removing the “Secure” wording in September 2018. And in October 2018, we’ll start showing a red “not secure” warning when users enter data on HTTP pages.  
> **[A milestone for Chrome security: marking HTTP as "not secure" - Google Blog](https://www.blog.google/products/chrome/milestone-chrome-security-marking-http-not-secure/)**

### How can I enable HTTPS?
Practically speaking, to enable HTTPS you need to get a certificate (which is secret file) from a trusted source and add it to your web server configuration. In the old days, this simple-looking task could be a nightmare: you _needed_ to buy an expensive certificate and set it up in your site by yourself, even information about how do it it was scarce.

Nowadays, we are living in a happy pro-HTTPS world ✨ services like [Let's Encrypt](https://letsencrypt.org/) gives you TLS certificates for free. If you are using AWS, they will also provide you with [certificates](https://aws.amazon.com/certificate-manager/).

In my scenario, the process is a little bit different. Since my blog is hosted at GitHub, I have no control of the web server whatsoever, meaning that I can't just get a certificate from Let's Encrypt and use it. To solve the problem, I used a service called [CloudFlare](https://www.cloudflare.com).

Cloudflare is a CDN (Content Delivery Network) platform. What it means is that they will have "copies" of your site in servers all around the world, so your users in Australia don't need to access servers in Canada (or wherever your site is hosted). On top of that, they also provide other features in their menu, such as access control and _free TLS_. Bear in mind that this is paid service, but they do have a free tier for personal websites.

The actual configuration is very simple:

- Create an account at Cloudflare and select the free plan.
- Add your website link in the next screen. Now, they will collect all the DNS information and populate on their services, since they will become your DNS manager.
- After that, they will give you two _nameserver_ addresses. You should configure these address where you bought your domain. In my case, GoDaddy has a DNS manager where I can specify such information.
- Now, just wait. Since the DNS service is cached all around the world, it takes some time for these changes to be reflected all around.

And, well, that's it. Now you have CDN _and_ TLS in your website, Chrome won't bother you anymore 😉.
