---
layout: post
title: "Ravello Systems REST API"
date: 2016-01-25 23:30:00
tags: ['cloud', 'ravello', 'api', 'python', 'powershell']
description: "In the last post, I wrote about how Ravello Systems brought a new way of doing cloud computing, specializing in cloud abstraction, nested virtualizaion and L2/L3 networking. Naturally, as a mature application, Ravello also has a REST API that allows to manage its resources in a programmatically way. That is what I'm going to talk about today."
---

In the [last post](http://deployeveryday.com/2016/01/20/cloud-inception-with-ravello-systems.html), I wrote about how [Ravello Systems](https://www.ravellosystems.com/) brought a new way of doing cloud computing, specializing in cloud abstraction, nested virtualizaion and L2/L3 networking.      

Naturally, as a mature application, Ravello also has a REST API that allows to manage its resources in a programmatically way. That is what I'm going to talk about today.

## The API
[Well documented](https://www.ravellosystems.com/ravello-api-doc/), the API is accessed basically by HTTP GET, POST, PUT and DELETE methods, passing as arguments the username and password, JSON structure as data and an API URL poiting to a resource. Like this: 

{% highlight bash %}
curl -v -X PUT -d @/path/to/jsonfile -H "Content-Type: application/json"
-H "Accept: application/json" --user ravello@ravello.com:password
https://cloud.ravellosystems.com/api/v1/applications/414244
{% endhighlight %}

The response is a JSON seriallized object, which is easily trated in a language like Python as a dictonary (or hash) object.     

## Some Examples

# Get a list of applications:
{% highlight bash %}
curl -v -X GET -H "Accept: application/json" --user jonatas.baldin@consteltecnologia.com:password 
https://cloud.ravellosystems.com/api/v1/applications
{% endhighlight %}

In the respose, you can see that I have just one application, named **app003**.
{% highlight bash %}
[
    {
        "id":67699091, 
        "name":"app003",    
        "owner":"Jonatas Baldin",
        "ownerDetails": {
            "userId":67403830,
            "name":"Jonatas Baldin",
            "deleted":false
        },
        "creationTime":1453375788101,
        "design": {
            "stopVmsByOrder":false
        },
        "designDiffersFromDeployment":false,
        "published":false,
        "version":1
    }
]
{% endhighlight %}

#Upload a public key:
First, the **key.json** file, a JSON serialized object:
{% highlight bash %}
{"publicKey": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDdEbE2sE99pW2trq5fV1PNQGES8WJc4AO29VH8VhVhYqTvXklP9kU1I739ZRANztTbvYKlLyJKFRw8VdvUVWlApHvH4UNFqqK18iifE+mPl+PVpmKypyggK9QUjKJq284LABz2TH8jQuPNAKm/Wc+ZRRtY2BsYxxvQ5Rh61Bcb8U6pHxzhRATVWKqMRuXJOH2VPVll3SU62iYjnYAkVnncaZJFIhROJhae/PZ3wQliDF3oz7gdIYfHNHD/eFXfujDIMj+rMQtZUKU6BheltWXmE5rE0W2lvxYvlXs4ZPA8026qD7MZlnZFk5r0xQI9nSNolrEGGosckiNUVmB2j3gB", "name": "NewPublicKey"}
{% endhighlight %}

The HTTP POST request:
{% highlight bash %}
curl -v -X POST -d @key.json -H "Content-Type: application/json" -H "Accept: application/json" 
--user jonatas.baldin@consteltecnologia.com:password https://cloud.ravellosystems.com/api/v1/keypairs
{% endhighlight %}

And the response:
{% highlight bash %}
{
    "id": 67796997,
    "name": "NewPublicKey",
    "creationTime": "1453767084263",
    "creator": {
        "id": 67403830,
        "roles": ["USER", "ADMIN"],
        "uuid": "uuidID",
        "name": "Jonatas",
        "surname": "Baldin",
        "email": "jonatas.baldin@consteltecnologia.com",
        "enabled": true,
        "activated": true,
        "deleted": false,
        "organization": 67403792,
        "invitationTime": "1452560573523",
        "activateTime": "1452560715932",
        "nickname": "Jonatas Baldin",
        "publicUrlClassifier": "jonatasbaldin779",
        "socialNetworks": {},
        "publicProfileAccess": "ONLY_ME",
        "organizationProfile": {
            "organizationName": "Lab",
            "id": 67403792,
            "accountStatus": "UNQUALIFIED",
            "uuid": "uudiID",
            "nickname": "lab438",
            "publicProfileAccess": "MY_ORG_ONLY"
        }
    }
}
{% endhighlight %}

To use more features, read the docs, they are great!

## python-sdk
If you use Python, Ravello offers [python-sdk](https://github.com/ravello/python-sdk), a thin and simple layer between the language and the API.     

Also comes with [good documentation](http://ravello-sdk.readthedocs.org/en/ravello-sdk-1.4/), providing a bunch of methods to interact with the principals API's resources.     

I wrote a little (maybe not so useful) program to create an application with CentOS 7 (an image from repository). You can read it to get some idea how to develop over the SDK. Check it out in the [Github Repository](https://github.com/jonatasbaldin/stuff/tree/master/ravello-stuff).

## Powershell Module
Personally I did not test this [Ravello Powershell Module](http://www.lucd.info/2016/01/20/ravello-powershell-module-v1-1/), but it looks amazing! LucD made an excelent job, writing over 4800 SLOC. If you may use, leave a comment talking about it :)

## Conclusions
I already told that Ravello is awesome (certainly more than twice), and its REST API is the icing on the cloud cake. As software eats the world, everyone working directly with IT should learn to code, and dealing with web resources is a great way to start doing something useful, automating mundane tasks and [getting more time free](https://xkcd.com/1319/).
