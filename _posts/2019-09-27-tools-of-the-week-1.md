---
layout: post
title: "Tools of the Week #1"
date: 2019-09-27 01:00:00
description: ""
image: "/img/totw.png"
---

> Tools of the week is a top of my head idea to write a tiny little bit about the tools and libraries I discovered every week. I love tools. I now you love them too ❤️

## [wharfee](https://github.com/j-bennet/wharfee)
A shell for Docker with auto complete and highlight. I love this niche shells with asteroids, like [saws](https://github.com/donnemartin/saws).

If you use Docker a lot, it makes life soooo much easier. You know when you are trying to nailed down that set of flags to get a specific behavior? Having a tool that basically _tells_ you what's available without you even need to ask it's just amazing.

![Screenshot showing the usage of wharfee](https://i.imgur.com/BtmpPSZ.png)

---

## [kubebox](https://github.com/astefanutti/kubebox)
A nice tool to have a quick overview of your pods with their metrics and logs AND by pressing `r` to remote login into them 🤯 You can run it locally, from inside or cluster or even in their [webpage](https://github.com/astefanutti/kubebox). Way better than doing a bunch of `kubectl` commands to navigate your environment. Also, there are a lot of interesting [open issues](https://github.com/astefanutti/kubebox/issues), a good start for Hacktoberfest!

![GIF showing the usage of kubebox](https://camo.githubusercontent.com/f657bda0847eeaf09459f6d3c045af177f6c6f28/68747470733a2f2f6173746566616e757474692e6769746875622e696f2f6b756265626f782f6b756265626f782e737667)

---

## [kubespy](https://github.com/pulumi/kubespy)
Have you ever wondered what actually happens when you create a `Service` in K8S? I just discovered that for a `ClusterIP` one it also creates an `Endpoint` which lead me to go and understand what an Endpoint actually is. 

Kubespy displays **real time** changes to complex structures in Kubernetes. Crazy cool.

You issue a command like:
```bash
$ kubespy trace service mysvc
Waiting for Service 'default/mysvc'
```

Then deploy a `Service` in another console:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysvc
spec:
  selector:
    app: myapp
  ports:
  - port: 80
```

And your previous terminal will look like this:
```bash
[MODIFIED v1/Service]  default/mysvc
    ✅ Successfully created Endpoints object 'mysvc' to direct traffic to Pods
    ✅ Successfully allocated a cluster-internal IP: mysvc

[MODIFIED v1/Endpoints]  default/mysvc
    ✅ Directs traffic to the following live Pods:
       - [Ready] myapp @ 10.28.0.12
```

Awesome, right? You can also inspect `status` and `changes` for complex objects, getting some nice JSON diffs.

---

## [octant](https://github.com/vmware/octant)
> A web-based, highly extensible platform for developers to better understand the complexity of Kubernetes clusters.


Having an overview of your entire cluster can't be easier! Just install the binary and:
```bash
$ octant
```

Boom! 

Octant allows you to take a quick look on your resources, like `Services` and `Pods`, including their events, logs and configurations. You can also change your cluster by deleting specific objects.

For me, the best feature is the `Resource Viewer`, showing the relationship between _stuff_:

![Screenshot with the Resource Viewer page from octant](https://i.imgur.com/qN3AkXS.png)

It also has a plugin system, but I didn't find any quick-and-use plugin out there :(