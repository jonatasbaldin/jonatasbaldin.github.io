---
layout: post
title: "Three Solutions to Run Knative Locally"
date: 2020-03-31 05:00:00
description: "One problem with Knative is running it locally. Kubernetes is quite heavy, running yet another platform on top of it is a killer. Today we will take a look at some different approaches to test Knative without burning down your computer."
image: /img/knative_logo.png 
---

[Knative](https://knative.dev/docs/install/) is a set of building blocks to run Serverless workloads on top of Kubernetes. You can also use those building blocks to build _your own_ Serverless platform.

One problem with Knative is running it locally. Kubernetes is quite heavy, running yet another platform on top of it is a killer. Today we will take a look at some different approaches to test Knative without burning down your computer.

Some notes before proceeding:
- I'll be focusing on the Eventing part of Knative. Serving will _probably_ work just fine – you just need more resources
- I choose Kourier for the Knative network layer – it's simpler and lighter than Istio
- Ubuntu was used for everything, minus the Docker on Mac part ;)
- You can find the full instructions to install Knative [here](ihttps://knative.dev/docs/install/any-kubernetes-cluster/)

---

## k3s
[k3s](https://k3s.io/) is a lite version of Kubernetes, made for IoT & Edge computing – perfect for your laptop. k3s is packaged as a single binary (!!!), making it very easy to run. It only works on Linux :( Mac and Windows users will need a virtual machine – take a look at [multipass](https://multipass.run/) or [Digital Ocean](https://www.digitalocean.com/).

Let's do it!

```bash
export KNATIVE_VERSION="0.13.0"
export KUBECONFIG="/var/lib/rancher/k3s/server/cred/admin.kubeconfig"

# Since Knative has its own network layer, we need to disable k3s' Traefik during its installation
# to make sure Kourier proxy gets a LoadBalancer IP
curl -sfL https://get.k3s.io | sh -s - --disable traefik

# Install Knative Serving
kubectl apply --filename "https://github.com/knative/serving/releases/download/v$KNATIVE_VERSION/serving-crds.yaml"
kubectl apply --filename "https://github.com/knative/serving/releases/download/v$KNATIVE_VERSION/serving-core.yaml"

# Configure the magic xip.io DNS name
kubectl apply --filename "https://github.com/knative/serving/releases/download/v$KNATIVE_VERSION/serving-default-domain.yaml"

# Install and configure Kourier
kubectl apply --filename https://raw.githubusercontent.com/knative/serving/v$KNATIVE_VERSION/third_party/kourier-latest/kourier.yaml
kubectl patch configmap/config-network --namespace knative-serving --type merge --patch '{"data":{"ingress.class":"kourier.ingress.networking.knative.dev"}}'
```

Alright, the installation is done! Make sure all the Pods are running (if not, wait a bit):
```bash
kubectl get pod -A
```

And that k3s gave Kourier a `LoadBalancer` IP address:
```bash
kubectl get svc kourier -n kourier-system

                                       # yay!
NAME      TYPE           CLUSTER-IP    EXTERNAL-IP       PORT(S)                      AGE
kourier   LoadBalancer   10.43.94.13   172.17.255.1      80:31619/TCP,443:30649/TCP   80s
```

Let's deploy the Knative Service sample:
```bash
kubectl apply -f https://gist.githubusercontent.com/jonatasbaldin/bc04de2e376be23f75bb5815041fdd61/raw/d2345ac9aa01d0f3c771e9b3d4a1421dd766e0f9/service.yaml

service.serving.knative.dev/helloworld-go created
```

Make sure it is running and find its "public" address:
```bash
kubectl get ksvc

NAME            URL                                                   LATESTCREATED         LATESTREADY           READY   REASON
helloworld-go   http://helloworld-go.default.172.17.255.1.xip.io      helloworld-go-2xvpx   helloworld-go-2xvpx   True    
```

Finally:
```bash
curl http://helloworld-go.default.172.17.255.1.xip.io

Hello Go Sample v1!
```

✨🎉🍾

### ✨BONUS ✨
If you run the same commands in a Digital Ocean droplet, _you will get a public IP address!_ Quick and easy to test a web facing Service:
```bash
kubectl get ksvc

NAME                   URL                                                       LATESTCREATED                LATESTREADY                  READY   REASON
helloworld-go          http://helloworld-go.default.<my-public-ip>.xip.io        helloworld-go-7ejj4          helloworld-go-7ejj4          True    
```

## kind
[kind](https://kind.sigs.k8s.io) runs Kubernetes _inside_ a Docker container. As crazy as it seems, it works amazingly for testing and local development with a small resource footprint.

kind _do works_ on Docker for Mac, but getting the LoadBalancer [is not trivial](https://github.com/kubernetes-sigs/kind/issues/411) :( With Linux, we can get a LB using [MetalLB](https://metallb.universe.tf) – I used [this incredible tutorial](https://mauilion.dev/posts/kind-metallb/) to learn about it. Again, you can totes use a virtual machine here.

Let's start with kind and Knative:
```bash
# Getting kind
curl -Lo kind https://github.com/kubernetes-sigs/kind/releases/download/v0.7.0/kind-$(uname)-amd64
chmod +x kind
mv kind /usr/local/bin

# Getting kubectl
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x kubectl
mv kubectl /usr/local/bin

# Installing Docker (just copied from the Docker docs)
apt-get update
apt-get install apt-transport-https ca-certificates  curl gnupg-agent software-properties-common -y 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io -y

# Create a kind cluster
kind create cluster

# Install the Knative Serving
export KNATIVE_VERSION="0.13.0"
kubectl apply --filename "https://github.com/knative/serving/releases/download/v$KNATIVE_VERSION/serving-crds.yaml"
kubectl apply --filename "https://github.com/knative/serving/releases/download/v$KNATIVE_VERSION/serving-core.yaml"

# Configure the magic xip.io DNS name
kubectl apply --filename "https://github.com/knative/serving/releases/download/v$KNATIVE_VERSION/serving-default-domain.yaml"

# Install and configure Kourier
kubectl apply --filename https://raw.githubusercontent.com/knative/serving/v$KNATIVE_VERSION/third_party/kourier-latest/kourier.yaml
kubectl patch configmap/config-network --namespace knative-serving --type merge --patch '{"data":{"ingress.class":"kourier.ingress.networking.knative.dev"}}'
```

Now, some MetalLB steps:
```bash
# https://mauilion.dev/posts/kind-metallb/

# First, find the network CIDR used by the Docker bridge network
# MetalLB will use some IPs from there
docker inspect network bridge

[
    {
        "Name": "bridge",
        # ...
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    # this is what we want
                    # we will give the 10 addresses to MetalLB
                    # let's say, 172.17.255.1-172.17.255.250
                    "Subnet": "172.17.0.0/16"
                }
            ]
        },
        # ...
    }
]
```

Installing MetalLB:
```bash
# god bless the yaml
kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.9.3/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.9.3/manifests/metallb.yaml
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"

# Creating a MetalLB configuration
# It states that we will use those IP addresses for the LoadBalancer
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      # the range from before!
      - 172.17.255.1-172.17.255.250
EOF
```

Alright, the installation is done! Make sure all the Pods are running (if not, wait a bit):
```bash
kubectl get pod -A
```

After that, take a look at Kourier:
```bash
kubectl get svc kourier -n kourier-system

                                       # yes!
NAME      TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
kourier   LoadBalancer   10.96.7.120   172.17.255.1   80:30201/TCP,443:30909/TCP   110s
```

Let's deploy the Knative Service sample:
```bash
kubectl apply -f https://gist.githubusercontent.com/jonatasbaldin/bc04de2e376be23f75bb5815041fdd61/raw/d2345ac9aa01d0f3c771e9b3d4a1421dd766e0f9/service.yaml

service.serving.knative.dev/helloworld-go created
```

Make sure it is running and find its "public" address:
```bash
kubectl get ksvc

NAME                   URL                                                       LATESTCREATED                LATESTREADY                  READY   REASON
helloworld-go          http://helloworld-go.default.172.17.255.1.xip.io          helloworld-go-7sttx          helloworld-go-7sttx          True    
```

Finally:
```bash
curl http://helloworld-go.default.172.17.255.1.xip.io

Hello Go Sample v1!
```

## Docker for Mac
Might be my favorite 😅 The Kubernetes shipped with Docker for Mac is a _very_ simple way to use the platform locally: click on the button on your Docker configurations and you are good to go. Unfortunately, not _that_ easy for Knative, but let's fix that.

First, Knative only supports K8S v1.15.0+ and the default Docker for Mac comes with v1.14.6, without an upgrade path. BUT, you can get v1.16.0+ with the _Edge_ release. It incorporates experimental features and, as such, might not be 100% stable. Whatever, we like living on the _Edge_. Read the [Switch between Stable and Edge versions](https://docs.docker.com/docker-for-mac/install/#what-to-know-before-you-install#switch-between-stable-and-edge-versions) before proceeding to the [download](https://docs.docker.com/docker-for-mac/edge-release-notes/) page. 

Once installed, let's do the Knative dance:
```bash
# Install the Knative Serving
export KNATIVE_VERSION="0.13.0"
kubectl apply --filename "https://github.com/knative/serving/releases/download/v$KNATIVE_VERSION/serving-crds.yaml"
kubectl apply --filename "https://github.com/knative/serving/releases/download/v$KNATIVE_VERSION/serving-core.yaml"

# Install and configure Kourier
kubectl apply --filename https://raw.githubusercontent.com/knative/serving/v$KNATIVE_VERSION/third_party/kourier-latest/kourier.yaml
kubectl patch configmap/config-network --namespace knative-serving --type merge --patch '{"data":{"ingress.class":"kourier.ingress.networking.knative.dev"}}'
```

If we inspect the Kourier LoadBalancer, it gets a _localhost_ address 🤔 
```bash
kubectl get svc kourier -n kourier-system

NAME      TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
kourier   LoadBalancer   10.110.153.164   localhost     80:31650/TCP,443:30315/TCP   34m
```

The LB is being exposed to our _localhost_, on ports 80 and 443. This is important and will affect the way we communicate with the Knative Service.

Let's deploy the Knative Service sample:
```bash
kubectl apply -f https://gist.githubusercontent.com/jonatasbaldin/bc04de2e376be23f75bb5815041fdd61/raw/d2345ac9aa01d0f3c771e9b3d4a1421dd766e0f9/service.yaml

service.serving.knative.dev/helloworld-go created
```

Make sure it is running and find its "public" address:
```bash
kubectl get ksvc

NAME            URL                                        LATESTCREATED         LATESTREADY           READY   REASON
helloworld-go   http://helloworld-go.default.example.com   helloworld-go-9qsq7   helloworld-go-9qsq7   True    
```

Differently from the other steps, we didn't apply the _Magic xip.io DNS_ Knative file because we don't work when the LoadBalancer is pre-configured with a domain (which, in this case, is _localhost_).

Finally, to access the Service, hit the LoadBalancer on `localhost:80` and pass the Service domain as the Host header:
```bash
curl localhost:80 -H "Host: helloworld-go.default.example.com"

Hello Go Sample v1!
```

Awesome, right!?

> PS: I didn't dive too much into this solution. The `localhost` LoadBalancer might have its quirks that I haven't encountered. I'll keep this article updated if I see anything suspicious.

## ✨ Bonus: inlets-operator ✨
Imagine if you could easily expose all these solutions to the public Internet? Well, [inlets-operator](https://blog.alexellis.io/ingress-for-your-local-kubernetes-cluster/) does just that. I'll leave you with [this article](https://blog.alexellis.io/ingress-for-your-local-kubernetes-cluster/) for your next adventure.
