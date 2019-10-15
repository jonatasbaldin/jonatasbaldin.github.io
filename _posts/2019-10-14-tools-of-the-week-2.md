---
layout: post
title: "Tools of the Week #2"
date: 2019-10-14 05:00:00
description: ""
image: "/img/totw.png"
---

> Tools of the week is a top of my head idea to write a tiny little bit about the tools and libraries I discovered every week. I love tools. I now you love them too ❤️

## [sls-dev-tools](https://github.com/Theodo-UK/sls-dev-tools/)
Like htop, but for AWS Lambda functions. Honestly, I had this idea a while back! I'm thrilled to find out that other people made it ✨

Within the console app a lot of information is available, like a list of functions, metrics, logs, invocation duration, location in a map and more!

🚨 PRICE ALERT 🚨 The tool calls AWS API, a lot, especially `GetMetricData` from CloudWatch, costing $0.01 per 1k metrics requested.

![](https://raw.githubusercontent.com/Theodo-UK/sls-dev-tools/master/demo.png)

---

## [kubeinvaders](https://github.com/lucky-sideburn/KubeInvaders)
THE👏BEST👏CHAOS👏ENGINEERING👏TOOL👏EVER👏

IT'S SPACE INVADERS

BUT THE ALIENS

ARE PODS

I'M TOO EXCITED TO TURN OFF THE CAPS LOCK!!!

ENJOY IT.

![](https://raw.githubusercontent.com/lucky-sideburn/KubeInvaders/master/kubeinvaders.gif)

---

## [stern](https://github.com/wercker/stern)
This week do yourself a favor and forget `kubectl logs`, really, pretend it never existed.

Stern is the best tool I know of for tailing logs in K8S, working with _multiple_ Pods AND _multiple_ containers. Its filtering system understands Pods without their "unique Deployment ID", so `$ stern prometheus-server` just works.

---

## [inlets](https://github.com/alexellis/inlets)
Inlets is like ngrok, but open source (and better). Basically, you spin up a VM somewhere with an external IP address and run inlets in server mode. On your local machine, execute inlets specifying the VM's IP address and your local port. BOOM! Tunneling!

🍀 BONUS: Check [inlets-operator](https://github.com/alexellis/inlets-operator). It does the same tunneling dance, but you end up with a public LoadBalancer for your local Kubernetes cluster. A M A Z I N G.

---

## [reverse-interview](https://github.com/viraptor/reverse-interview)
It's time to flip the table and learn about questions you should make to your interviewer, including subjects like the company's role, tech stack, potential coworkers and more, nicely translated into different languages ⚡️