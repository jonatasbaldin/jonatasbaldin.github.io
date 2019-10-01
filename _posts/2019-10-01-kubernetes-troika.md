---
layout: post
title: "The Kubernetes Troika"
date: 2019-09-30 10:00:00
description: "Using three K8S services to deploy virtually any modern application with an external IP address, DNS entry and TLS certificate very quickly. Make K8S your  favorite platform to play around with other platforms."
image: "/img/troika.jpg"
---

![Three horses pulling a cart. A Troika in Russian](/img/troika.jpg)
<center><i>troika, a Russian vehicle drawn by three horses abreast</i></center>

I'm a sysadmin by heart. As a teenager, I had a 16U rack filled with old routers, switches, storage and servers. It was _a lot_ of fun (and a lot of noise). Running my own personal data center thought me a lot about system administration, but exposing an application externally was painful due to all the moving parts. Are cables plugged? Is the VM running? Is the VLAN working? Firewall rules? NAS storage? DNS? TLS? All-the-application-level-stuff? It was _a lot_ of fun (and zero Ansible).

Fast forward to 2019 and the only piece of infrastructure I have is a tiny little Kubernetes cluster in GCP. It is _a lot_ of fun (and way less noise).

After dancing with Kubernetes for a while I fell in love with _three amazing services_ – [ingress-nginx](https://github.com/kubernetes/ingress-nginx), [external-dns](https://github.com/kubernetes-incubator/external-dns) and [cert-manager](https://github.com/jetstack/cert-manager). With them, I can deploy virtually any modern application with an external IP address, DNS entry and TLS certificate very _very_ quickly. K8S just became my favorite platform to play around with other platforms.

Today I'll show you how to enable these services and make them best friends ✨

## List of Stuff You Will Need ™️
Kubernetes is a massive beast full of replaceable parts running anywhere. Some commands and outputs from this article _may_ be different if you are in a different environment, but _most_ should be OK.

Here's what I'll need from you:
- K8S Cluster and `kubectl`. Mine is on GCP. You can get one [here](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster). If you dislike Google, make sure your provider has support for  `LoadBalancer`
- A domain! I'll use `deployeveryday.com`
- A DNS provider from [this](https://github.com/kubernetes-incubator/external-dns#status-of-providers) list. I'll use Cloudflare
- Helm. Here is the [installation guide](https://helm.sh/docs/using_helm/#installing-helm)

PS: You _might_ be able to do everything locally with [Minikube](https://github.com/kubernetes/minikube), but doing things in the Official Internet ™️ is waaaaay more fun! You can send to your friends real Internet Addresses ™️ to brag about how much you know 😉

## A tiny little app
Let's start our adventure by deploying [httpbin](https://github.com/postmanlabs/httpbin), a service to test the HTTP Request/Response life cycle. Our mission is not about the app itself, but how we can automatically give it external access, DNS and TLS.

Ay, let's start YAMLing. 

Create a file named `httpbin.yaml` with the content below:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpbin
  labels:
    app: httpbin
spec:
  containers:
  - name: httpbin
    image: kennethreitz/httpbin
    ports:
      - containerPort: 80

---

apiVersion: v1
kind: Service
metadata:
  name: httpbin
spec:
  selector:
    app: httpbin
  ports:
  - port: 80
```

And apply it:
```bash
$ kubectl apply -f httpbin.yaml
```

It will create a `Pod` and a `Service`. 

A `Pod` is the smallest deployable _thing_ in Kubernetes, containing one or more containers. The first section of the YAML above says:
> Sup K8S, could you run the httpbin container and expose its port 80?

_[I wanna a better explanation](https://kubernetes.io/docs/concepts/workloads/pods/pod/)_.

The second part is a `Service`. In our case, it is exposing the `httpbin`'s port 80 _inside the cluster_ to other K8S resources, like another Pod. _[It can't just be it](https://kubernetes.io/docs/concepts/services-networking/service/)_.

Wait for the Pod to be in the `Running` state:
```bash
$ kubectl get pod -l app=httpbin -w
```

We still cannot access the app from the Internet, however, we can create a tunnel from our local machine to the cluster just to take a quick look. I know, Kubernetes is crazy shit.

```bash
# Type this in a new terminal and let it open
$ kubectl port-forward httpbin 8080:80
```

```bash
# Back to the main terminal, let's test our app
$ curl localhost:8080/get
```

You should see some JSON back to you 🎉 But why `localhost` if we can Take It To Another Level?

## The (NGINX) Ingress
An `Ingress` allows external access (from the Internet) to internal Services – like the one we just created – with path-based or named-based rules, SSL termination, etc. 

Let's break down this statement.

- `allows external access (from the Internet)`: defines an specification to access Services from outside the cluster

- `path-based rules`: traffic coming to the address `x.x.x.x/foo` goes to the `Foo` Service and traffic coming to the address `x.x.x.x/bar` goes to the `Bar` Service

- `named-based rules`: traffic coming to the hostname `foo.example.com` goes to Service `Foo` and traffic coming to the address `bar.example.com` goes to Service `Bar`

- `SSL termination`: it does the SSL dance for you, usually when in cahoots with another K8S controller, like `cert-manager` 

If you are still confused, don't worry, it is a hard concept to grasp. You'll understand it better once we use it. See the [official docs](https://kubernetes.io/docs/concepts/services-networking/ingress/) if you'd like more information.

BUT, the Ingress just _specifies_ the rules, someone else needs to apply and enforce them. Enters the Ingress Controller, a program that understands the Ingress specification and executes all the shenanigans to make it work. It usually manages the entry point (external IP address) as well, provided by the cloud provider. 

The [official docs](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) have the most known controllers. We will be using the [ingress-nginx](https://github.com/kubernetes/ingress-nginx/).  

Install it with the command below:
```bash
# `controller.publishService.enabled=true` is necessary for the DNS automation later
$ helm install stable/nginx-ingress --name ingress-nginx --set controller.publishService.enabled=true --wait
```

The external IP address takes some time to be created. Watch its status with the command below looking for the `EXTERNAL-IP` column. Note down the address, you will use it to access the application in a bit.
```bash
$ kubectl get services ingress-nginx-nginx-ingress-controller -w
```

Now let's create the Ingress. It uses the NGINX controller to direct traffic into our `httpbin` application.

Create a file named `httpbin-ingress.yaml`:
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx  # controller class name
    nginx.ingress.kubernetes.io/ssl-redirect: "false"  # disable forced SSL for now, we will fix it later
  name: httpbin
spec:
  rules:
  - http:
      paths:
      - path: /  # path-based rule
        backend:
          serviceName: httpbin
          servicePort: 80
```

Apply it:
```bash
$ kubectl apply -f httpbin-ingress.yaml
```

Access your external IP in your browser and 🎉🎊🥳! Our service is up and running in the Real Internet ™️

## Names are Better than x.x.x.x
If IP addresses are cool, names are cooler. Would it be fascinating to get a DNS name with the Ingress? Well, [External DNS](https://github.com/kubernetes-incubator/external-dns) does exactly that.

> ExternalDNS synchronizes exposed Kubernetes Services and Ingresses with DNS providers.

We will install using the [external-dns Helm Chart](https://github.com/helm/charts/tree/master/stable/external-dns). 

This section uses Cloudflare's DNS. If you are using another provider, follow the instructions from the [chart's `values.yaml`](https://github.com/helm/charts/blob/master/stable/external-dns/values.yaml) and the [official documentation](https://github.com/kubernetes-incubator/external-dns#deploying-to-a-cluster).

With Cloudflare, I need my account's email and API key. [This article](https://github.com/kubernetes-incubator/external-dns/blob/master/docs/tutorials/cloudflare.md) explains how to get them. The API key will be stored in a Kubernetes secret and the email as a text plain value inside the Helm chart.


```bash
$ kubectl create secret generic external-dns --from-literal=cloudflare_api_key=<api key>
```

Create a file named `external-dns-values.yaml` with the following. Don't forget to change the `cloudflare.email` key to your own.
```yaml
sources:
- ingress

provider: cloudflare

cloudflare:
  # cloudflare's account email
  email: youremail@domain.com

  # disables cloudflare's proxy 
  proxied: false

# Creates RBAC account
# Apparently obligatory in the Helm chart, see https://github.com/kubernetes-incubator/external-dns/issues/1202
rbac:
  create: true
```

And deploy the Helm chart:
```bash
$ helm install --name external-dns -f external-dns-values.yaml stable/external-dns --wait
```

Magic time 🧙‍♀️. Change the `httpbin-ingress.yaml` file to:
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx  # controller class name
    nginx.ingress.kubernetes.io/ssl-redirect: "false"  # disable forced SSL for now, we will fix it later
  name: httpbin
spec:
  rules:
  # use your domain here
  # now this is a name-based rule
  - host: httpbin.deployeveryday.com
    http:
      paths:
      - path: /
        backend:
          serviceName: httpbin
          servicePort: 80
```

```bash
$ kubectl apply -f httpbin-ingress.yaml
```

And 🎉! A new DNS entry with the specified domain and the Ingress IP should be created. 

Here are the logs from External DNS:
```bash
$ kubectl logs -l "app.kubernetes.io/name=external-dns,app.kubernetes.io/instance=external-dns"

time="2019-09-25T17:25:20Z" level=info msg="Changing record." action=CREATE record=<your domain> targets=1 ttl=1 type=TXT zone=e919cd53f62fc30c7a25396992ca2472
```

Verify the DNS propagation with:
```bash
$ dig <domain> +short
```

When the last command returns your IP address, you can access the app via the created DNS!

## Be safe with TLS certificates
Without TLS, traffic between the client and the server might be spoofed by an attacker and Google will flag your website as insecure. Nowadays you can get TLS certificates for _free_ with [Let's Encrypt](https://letsencrypt.org/). On top of that, [cert-manager](https://github.com/jetstack/cert-manager) generates and renews certificates for our applications automatically, using information from the Ingress and DNS.

> cert-manager is a Kubernetes add-on to automate the management and issuance of TLS certificates from various issuing sources. It will ensure certificates are valid and up to date periodically, and attempt to renew certificates at an appropriate time before expiry.

Practically speaking, cert-manager can be configured to watch our Ingress and create a certificate for the defined domain with Let's Encrypt. 

The most widely used method to validate the certificate is `http01`, BUT we won't use here. You see, to use `http01` we need a DNS entry in place already propagated _before_ starting the certification process. The objective here is to get an external IP address, a DNS entry _and_ the TLS certificate in one shot, without any _DNS-propagation-waiting-time_. `dns01` solves this issue since it connects directly to our DNS provider to validate the domain.

If you want more details about these methods (and others), please take a look at their [docs](https://letsencrypt.org/docs/challenge-types/).

Install cert-manager with this shameless copied from the [official docs](https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html#steps).

```bash
# Install the CustomResourceDefinition resources separately
$ kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.10/deploy/manifests/00-crds.yaml

# Create the namespace for cert-manager
$ kubectl create namespace cert-manager

# Label the cert-manager namespace to disable resource validation
$ kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true

# Add the Jetstack Helm repository
$ helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
$ helm repo update

# Install the cert-manager Helm chart
$ helm install \
  --name cert-manager \
  --namespace cert-manager \
  --version v0.10.1 \
  --wait \
  jetstack/cert-manager
```

Verify the installation with:
```bash
$ kubectl get pods --namespace cert-manager -w
```

## Setting up the Issuer
The [Issuer](https://docs.cert-manager.io/en/latest/tasks/issuers/index.html) contains your Let's Encrypt "account" and the method used to generate the certificates. The best practice is to create a _staging_ Issuer to test the waters first due Let's Encrypt API rate limit. For the sake of this article, we will go straight to production.

Cloudflare resolver is set under the `solvers` key. Look at the [supported providers](https://docs.cert-manager.io/en/latest/tasks/issuers/setup-acme/dns01/index.html#id1) if you are using a different one.

Create a file named `issuer.yaml` with the following YAML:
```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: Issuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: youremail@domain.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - dns01: 
        # Your DNS provider configuration
        cloudflare:
          email: youremail@domain.com
          apiKeySecretRef:
            name: external-dns
            key: cloudflare_api_key
```

And apply it:
```bash
$ kubectl apply -f issuer.yaml
```

Now let's tell our Ingress that it *wants* a certificate. Edit the `httpbin-ingress.yaml` to the following:
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx  # controller class name
    certmanager.k8s.io/issuer: letsencrypt-prod  # issuer name
  name: httpbin
spec:
  rules:
  # use your domain here
  # now this is a name-based rule
  - host: httpbin.deployeveryday.com
    http:
      paths:
      - path: /
        backend:
          serviceName: httpbin
          servicePort: 80
  tls:  # specify we want TSL
  - hosts:
    - httpbin.deployeveryday.com
    secretName: httpbin-tls  # secret to store our TLS
```

```bash
$ kubectl apply -f httpbin-ingress.yaml
```

It takes about 3 minutes for cert-manager to cook a certificate. Watch for the `READY` column with the command below:
```bash
$ kubectl get cert httpbin-tls
```

Access your domain and 🎉, we have a TLS certificate ready to go 🔒

## All together now, _all together now!_
You are sitting at your desk, sipping some tea and discussing your evil plans to conquer the world in [Reddit](https://www.reddit.com/r/myevilplan/). Your colleague approaches you:

> – "We need to get this app up and running onb the Internet _today_, with DNS and TLS".  
> – "No worry, give me 5 minutes".

You put your tea down and quickly slams a YAML file named `yet-another-service.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
  labels:
    app: kuard
spec:
  containers:
  - name: kuard
    image: gcr.io/kuar-demo/kuard-amd64:blue
    ports:
      - containerPort: 8080

---

apiVersion: v1
kind: Service
metadata:
  name: kuard
  labels:
    app: kuard
spec:
  selector:
    app: kuard
  ports:
    - port: 80
      targetPort: 8080

---

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    certmanager.k8s.io/issuer: letsencrypt-prod
  name: kuard
  labels:
    app: kuard
spec:
  rules:
  - host: kuard.deployeveryday.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kuard
          servicePort: 80
  tls:
  - hosts:
    - kuard.deployeveryday.com
    secretName: kuard-tls
```

```bash
$ kubectl apply -f yet-another-service.yaml
```

```bash
$ kubectl get pod,svc,ingress,cert -l app=kuard
```

> "Done".

## Prologue
This is my favorite Kubernetes setup. With some building blocks, you can transform the abstract container orchestrator into a platform ready to serve your workloads.

However, it can be tricky. There's a bunch of possible combinations between services, providers and configurations. What if you have more than one DNS provider? What if your applications need different rules according to their tier? What about monitoring and observability? I bet our trinity can answer all these questions, and they should be evaluated before any hard work starts.

If you have any thoughts, please leave in the comments below 😌

Thanks ❤️

---

[Troika image source](https://blog.radissonblu.com/wp-content/uploads/2014/11/BP2-MOWZA-Troika-horse-and-sleigh-in-Russia-on-the-snow.jpg).