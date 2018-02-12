---
layout: post
title: "What is Serverless all about?"
date: 2017-07-02 12:00:00
tags: []
description: "Serverless. The new hype buzzword is taking over the development universe, promising big savings in infrastructure for applications and less deployment headaches to developers. In an agile world, shipping scalable software with budget constraints has become a big puzzle: one that Serverless may solve."
image: "/img/lambda_symbol.png"
---

*Originally posted at [Cheesecake Labs](https://cheesecakelabs.com/blog/what-is-serverless-all-about/).*

**Serverless**. The new hype buzzword is taking over the development universe, promising big savings in infrastructure for applications and less deployment headaches to developers. In an agile world, shipping scalable software with budget constraints has become a big puzzle: one that Serverless may solve.

Today I'm going to give you an overview of its history, basic concepts, providers, frameworks, benefits and drawbacks. One thing, though: personally, I don't like the term Serverless, as it was coined by the Serverless Framework, but now it stuck and everybody is using it. What can I do?

# Mommy, what is a *virtual machine*?
You better be prepared to answer this question. As we barely remember floppy disks, the future developers may never need to spin up an instance, or touch a big server. In a future not far far away, we will be telling this kind of story to our grandchildren, while roasting some marshmallows in a campfire:

> Kids, I'll tell you something: **virtual machines**, that's where it all started. Once any person was able to run multiple operating systems in a unique, common physical machine, the door of possibilities was opened. After a while, some people at Amazon – that old company that tried to dominate the world – started the revolution of **cloud computing**, where just a credit card was needed to get almost infinite virtual machines, anywhere in the world. If not enough, a **blue whale** came to change everything again. Oh, the golden age of containers! The first years were rough, it took some time to really understand the concept, but developing software has never been so easy! And then, oh, then there was **Serverless**...

At this moment, you realize that your grandchildren left you wondering alone to play with their virtual reality real-time life simulator games. Yeah, it won't be easy to be an old retired developer.

# Function as a Service
One more *thing as a Service*, are you counting them? FaaS is the base concept to understand what Serverless is all about. It's defined as:

> Code that runs on event-driven, ephemeral and stateless environments, fully managed by a third-party.

Ok, let's dive in! *PS: I may explore this using some AWS jargons and services. Keep reading, you'll understand why.*  

**Code.** This is easy, right? Code is just, well, code. However, there are some nuances here: you have to pack everything you need – including external libraries – into a *.zip* file. Also, your function should have an execution entry point, what we call *handler*. Fortunately, there are awesome frameworks to *handle* this for us.

**Event-driven.** In the FaaS world, everything is an **event** that **triggers** some action. A classical example is the tale of resizing an uploaded image. Instead of writing code to receive a file, store it, resize it and store its versions again, you can trigger some code which resizes an image every time a new file is sent to an [S3 bucket](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html). Besides, there are all kinds of events, such as inserting lines in a database, watching a stream of logs, getting new messages in a queue and even an HTTP request.

**Ephemeral environment.** The secret sauce of FaaS: every time you call a function, a completely new environment is built from scratch – and it is killed right after your call is processed. Seems tragic, but it has some considerations: there's a delay to run the inactive function for the first time – we call this **cold start**. If the function is called too much, the environment may **last more than one execution**, eliminating the cold start delay and making the function **warm**. Another point is cost: you only pay per execution time (more on this later).

**Stateless.** Yep, no state between function invocations! You can't rely on keeping information locally or in memory for future use, all context necessary for processing the request must be available at every execution. It's true that **AWS Lambda** has some disk memory that **may** be available if the function is warm, but you have to check it for consistency. 

**Fully managed by a third-party.** You don't need to manage the servers. In fact, you **can't manage them**. As of today, most of the Serverless providers fully maintain all the services. While you can forget about the infrastructure, it does create a culture of vendor control and lock-in.

By the end of the day, Serverless is all about integrating external services and developing the interfaces between them, worrying less and less about server management. In this new era, thinking *serverlessly* is crucial to take advantage of the event-driven architecture. *Should I write this piece of code or use a third-party? Can I make this part be triggered by an event? Does my function have all the vital context to be executed correctly?* A lot of thought and iterative experimentation must be made over these questions.

![FaaS Symbol](/img/lambda_symbol.png){: .center-image }
*The FaaS symbol (unfortunately no Half-Life 3 yet)*{: .center-text}

# A king named AWS
Vendor control is so strong that even this article got trapped. We do have other providers, such as Google Cloud Functions, IBM OpenWhisk and Auth0 Webtask, but AWS was the first one to introduce FaaS to the world, evolving its product beyond any other company so far. If there’s a Serverless **belt**, AWS is still holding it.

The **AWS Lambda** was introduced in 2014, the first popular **FaaS platform**. Fast forward today, it supports Java, C#, Node and Python. It’s available in a variety of regions around the globe and it has [awesome features](https://aws.amazon.com/lambda/).

The greater power comes from the integration with all other AWS services. S3, DynamoDB, SNS, Kinesis and even Alexa can trigger events to be executed by Lambda, enabling developers to create all kinds of applications by writing minimum or no code at all.

As a web backend developer, the masterpiece to me is the [AWS API Gateway](https://aws.amazon.com/api-gateway/). It creates HTTP endpoints that, once requested, trigger Lambda functions passing along all its context. You are able to create a full web API by just using these two services, since they also have authentication, documentation, throttling, infinite scaling etc.

# Need for Frameworks
Serverless is **hard**. Events coming from all over the place, integrations with dozen of services, multiple Lambda functions executing asynchronously and synchronously, generating even more events. It can become impossible to orchestrate, deploy and maintain all these moving parts and still have a good sleep at night. On top of that, Serverless is a bleeding edge service, the new cool kid in the neighborhood. Its providers are still figuring out which features to deliver and how to make the service more friendly and robust, resulting in changes all over the place.

After seeing the mentioned problems increasing more and more, people started to develop Frameworks to make the life easier. [Serverless](https://serverless.com/) **(the framework)**, [Zappa](https://github.com/Miserlou/Zappa), [Apex](http://apex.run/), [Chalice](https://github.com/awslabs/chalice) and [Gordon](https://github.com/jorgebastida/gordon) are just a few of almost twenty Frameworks available to manage this complex infrastructure. And believe me, **you will need them**. The one I’ve been using the most is Zappa, a tool to build and deploy Python WSGI applications on AWS Lambda and API Gateway, meaning Django and Flask without servers! I loved it so much that I’ll be giving a talk about it on [Caipyra](http://caipyra.python.org.br/) and I even started to contribute to the project!

# The Pros
So far, so good. Let’s take a better view into the benefits that Serverless is bringing to the table.

**COST**. All upper case. Certainly the main benefit of using Serverless today is its price. For instance, having about **10.000** hits per day, with 200ms milliseconds in a 256MB memory Lambda function, you would pay about [$0.31/mo](https://www.trek10.com/blog/lambda-cost/). Thirty one cents of dollar per month. What the hell? This is because you pay just the data transferred out of AWS and when your function is running, **by the millisecond**, an amount of $0.20 per **1 million requests**. Furthermore, the expenses with server management are dropped to near zero, making Serverless the cheapest computing power on the cloud today.

Now, don’t **#NoOps** me. In Serverless-only world, my bet is that Operators will diverge into two segments: first of all, they will be running the **servers** behind the Serverless. Secondly, there are still networking, debugging and monitoring to be managed and observed, tasks that due the circumstances, will be harder than ever.

![Every time you say #NoOps](/img/every_time_you_say_noops.jpg){: .center-image }
*Talking about Serverless @ Cheesecake Labs*{: .center-text}

**Time to Market**. With all integrations, low cost and velocity, reaching marketability is just faster now. Putting code out there has become pretty easy.

**Infinite Scale**. One request? Ok. A hundred requests? Ok. A thousand? Meh. **A hundred thousand?** It won’t even tickle. Serverless was made to handle any amount of requests at any given time, charging by the use. Now, the **auto-scaling** is *really* automatically, there’s no plan and no setup, it’s just intrinsically built in the architecture. Be sure to keep an eye on [Billing DDoS](https://www.theregister.co.uk/2015/03/20/greatfire_chinese_activists_under_ddos/), though.

**Package and Deploy**. No more Ansible, Chef or Puppet. No more layered Dockerfiles and their entry points. All that Lambda needs is a *.zip* compressed file with all your code and libraries. Packaging has become a piece of cake. What about deploying? You just need to upload a *.zip* compressed file with all your code and libraries. Got it? Just a *.zip* compressed file with all your code and libraries.

# The Opportunities
Lack of persistent storage/memory and absence of server introspection/tuning shouldn’t be seen as Serverless’ cons, but rather its characteristics. Instead of fighting, we should build our applications around them.

We also read a lot on the Internet about how testing, debugging, monitoring and orchestration are cons on the Serverless land. Well, I see them as **opportunities**. There’s an enormous room to develop features and better tooling around this ecosystem. Likewise, learning and teaching are huge growing areas. The technology is brand-new and quite unfamiliar for a considerable amount of people, turning blog posts, articles, books, videos, courses and workshops into precious content to be created and consumed. Also, if you are a conference addict, make sure to check out [ServerlessConf](http://serverlessconf.io/)!

# Wrapping Up
Now, the one million dollar question: **is it production ready**? I’ll tell y’all: **yes**, *but try it first*. Still young and immature, it may not be the right approach for your workload, or it might require some change on your side. More than that, you’ll need a team who’s willing to face a fast growing movement, spending a lot of time learning and testing on their own, as there’s not so much content available out there. Either way, companies like [Netflix](https://aws.amazon.com/solutions/case-studies/netflix-and-aws-lambda/), [Localytics](https://aws.amazon.com/solutions/case-studies/localytics/), [VidRoll](https://aws.amazon.com/solutions/case-studies/vidroll/), and [Square Enix](https://aws.amazon.com/solutions/case-studies/square-enix/) are already running it!

My opinion? Serverless is the future, right here. It’s growing quickly and changing – once again – how we think about software.
