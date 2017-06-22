---
layout: post
title: "API Design: Think First, Code Later"
date: 2017-06-22 18:00:00
tags: ['api', 'design', 'http', 'json', 'api blueprint', 'html', 'restfulapi', 'apiary']
description: "As a software developer, I know how hard it is to contain the urge to start coding as soon as we can. After the first sprint planning, our fingers – uncontrolled, hungry creatures – want to start smashing the keyboard, translating our ideas into code fastly and furiously.

Despite how great we feel while developing, it's always a good idea to take a step back, especially when building something that could be used by many different users – like an API is. A. In this post, I’ll show you why and how to design a properly-thought API."
comments: True
---

*Originally posted at [Cheesecake Labs](https://cheesecakelabs.com/blog/api-design-think-first-code-later/).*

As a software developer, I know how hard it is to contain the urge to start coding as soon as we can. After the first sprint planning, our fingers – uncontrolled, hungry creatures – want to start smashing the keyboard, translating our ideas into code fastly and furiously.

Despite how great we feel while developing, it's always a good idea to take a step back, especially when building something that could be used by many different users – like an API is. A. In this post, I’ll show you why and how to design a properly-thought API.

API is a generic term to define an Application Program Interface, in other words, how a user (human or machine) interacts with a program. In the web development world, an API is generally a website with a collection of endpoints that respond to client requests with structured text data.

Another concept that's widely used and discussed by web developers is the one of a [RESTFul Web API](https://en.wikipedia.org/wiki/Representational_state_transfer). It was defined by [Roy Fielding](https://en.wikipedia.org/wiki/Roy_Fielding) as an architecture style that provides a well-established communication protocol between client and server. It outlines some constraints: stateless communication, respect of the underlying technology (generally HTTP) and the use of [hypermedia as the engine of application state](http://en.wikipedia.org/wiki/HATEOAS). In other words, it proposes some patterns for building a web API. For simplicity's sake, I'll be referring to Web APIs simply as APIs throughout this post.

# One JSON is worth a thousand words
Let’s see an example of an API response:

```
{
  "data": [
    {
      "story": "Jonatas Baldin was writing a blog post.",
       "created_time": "2017-15-01T00:02:57+0000",
       "id": "624510964324764_1046909755418214"
    },
  ]
}
```

This snippet of data, called [JSON,](https://www.w3schools.com/js/js_json_intro.asp) is an example of how a smartphone sees a Facebook status message, gathered from the [Facebook API](https://developers.facebook.com/docs/graph-api). Pretty neat, right?

Now imagine that you are an engineer at Facebook, responsible for this API. You wake up one day and decide to change the id field name to message_id. Well, this small change could break Facebook. For real. All devices that depend on the previously-defined structure would now stop being able to collect and display the content to the user. Definitely not a great day at work.

# The good, the bad and the ugly
A bad API design – or the lack of it – will, sooner or later, cause all kinds of trouble:

* **Absence of consistency**: once an API grows, the endpoints tend to be created just to fulfill immediate needs.
* **Difficulty to scale**: there’s no reference when troubleshooting an endpoint.
* **Hard to learn**: a steep learning curve to consume data from and develop the API.
* **Performance issues**: throwing unplanned API code usually creates performance bottlenecks on the long run.
* **APIs tend to be forever**: it’s always better to get it right at the first chance.

An API is a contract between your application and its users. It can’t be changed abruptly without causing chaos for those who already signed it and it shouldn't be built without forethought. This is where [design](https://en.wikipedia.org/wiki/Design) enters: the creation of a plan or convention for the construction of an object, system or measurable human interaction.

# First things first: Let's talk HTTP
Before diving into API code, let’s get a glimpse of the Hypertext Transfer Protocol. HTTP, as the name suggests, is used to transfer data on the web. It has a set of rules on how clients and servers should exchange information. Its main components are:

* **URL**: where your resources are on the web, the address of your endpoint. An example is using http://example.org/users to list your users.
* **Request methods**: the action a client wants to perform on a specific endpoint. GET is used to retrieve a resource, POST, to create one, PUT and PATCH to update an existing one, and DELETE… well, deletes stuff.
* **Headers**: contain information about the client or the server. For instance: content type (format), method, authentication token and others.
* **Body**: the data sent to the server or received by the client. JSON is the de facto standard.
* **Status code**: a three-digit number which tells the request status. To summarize, 2xx statuses means success, 4xx means client errors and 5xx means service errors. You can get a full list [here](https://http.cat/) or [here](https://httpstatusdogs.com/).

Let's see an HTTP request/response example:

```
Client request:
GET /users/1 HTTP/1.1
Host: www.example.org/users

Server response:
HTTP/1.1 200 OK
Content-Type: application/json
Date: Sun, 19 Feb 2017 00:43:51 GMT

{
  "id": 1,
  "first_name": "Jonatas",
  "fast_name": "Baldin",
  "role": "Caker"
}
```

# Good API design demands good specifications
There are some specifications to help you out when designing your API. The most used ones are **Swagger** with pure JSON, **RAML** with YAML notation and **API Blueprint** backed by the markdown syntax. I felt in love with the latter: even though it is the new cool kid in the neighborhood, it’s already mature enough and delightful to work with. That’s the one I’ll be covering here.

From the official website:

> API Blueprint is simple and accessible to everybody involved in the API lifecycle. Its syntax is concise yet expressive. With API Blueprint you can quickly design and prototype APIs to be created or document and test already deployed mission-critical APIs.

It’s an open source, focused on collaboration, easy to learn and well-documented specification created by Apiary with design-first in its core, surrounded by awesome tools: from mock server generators to full-feature life-cycle solutions.

In addition to Blueprint, there’s **MSON** (Markdown Syntax Object Notation), which defines data structures in a human-readable way. Instead of writing manually the endpoints body data, you represent them in reusable objects. Pretty nifty, huh?

Here’s what you need to know to get started:

* **API Name, Description and Metadata**: describe a little bit about the API and the Blueprint version.
* **Resource Groups**: group of related resources, like Users.
* **Resource**: Defines a unique resource, its endpoint and action.
* **Parameters**: Used in an endpoint to specify a dynamic parameter, like an ID or query search.
* **Response**: The content-type, HTTP status code and body data.

Besides that, there’s [Apiary](https://apiary.io/). It’s a collaborative platform to create, render, test and serve your API. They have a free plan for public projects and you can sign up directly from your GitHub account to create your own designs.

Here’s a Blueprint code sample:

```
FORMAT: 1A
HOST: http://cakes.ckl.io/

# Cakes API
Cakes is an API used to store and consume information about the most loved Cakes here at Cheesecake Labs.

# Group Cakes

## Cakes [/cakes/]

### List all Cakes [GET]

+ Response 200 (application/json)

+ Attributes (array[Cake])

# Data Structures

## Cake (object)
+ id: `1` (string, required) - The unique ID of a Cake.
+ name: `Cheesecake` (string, required) - The name of a Cake.
+ rating: `5/5` (string, optional) - The rating of a Cake.
```

If you access the [project](http://docs.cheesecakelabs.apiary.io/) above, you’ll see a prettily-rendered page. Now the awesome part: a mock server is automatically created for every project – look at [this](http://private-b604b-cheesecakelabs.apiary-mock.com/cakes/)! See the magic? No code was needed to make it work. Anyone with basic knowledge of HTTP and Blueprint can create a mock API and get feedback from customers.

The example above is as simple as it can get. You can check the Blueprint [tutorials and documentation](https://apiblueprint.org/documentation/tutorial.html) to go deeper into its syntax and read the specs [here](https://github.com/apiaryio/api-blueprint/blob/master/API%20Blueprint%20Specification.md). Also, this is an open-source project – any contribution from the community is welcomed.

Congratulations! Now you are armed with the knowledge necessary to design your APIs. But before you start rocking your systems, here are some little tips to keep on your toolbelt:

# Dos and Don'ts
We have walked a long path together, from understanding what is an API and why its design matters, passing by the HTTP protocol and landing on Blueprint API, giving you the foundation to start creating your own designs. I’ll list some amazing dos and don'ts. Note that these aren’t rules, just best practices you generally find on the web.

## Treat endpoint actions as CRUD(L) operations
Our /cakes/ endpoint may have Create, Read, Update, Delete and List operations. You can construct these actions with HTTP verbs and the URL, like:

* **List**: `GET /cakes/`
* **Create**: `POST /cakes/`
* **Read**: `GET /cakes/1/`
* **Update**: `PATCH /cakes/1/`
* **Delete**: `DELETE /cakes/1/`

## Correctly use the HTTP methods

GET is for getting, POST if for posting. Don’t use POST if you just want a list of Cakes.

## HTTP has methods, use them!

`POST /cakes/createCake`
You don’t need to specify the action in the URL, we already know that POST creates something.

`POST /cakes/`
Cleaner URL, also telling us that probably the other methods will work as expected, like GET /cakes.

## Singular or Plural? Plural!

`GET /cakes` should return a list of Cakes, so `GET /cake/1` should return the first Cake, right? Unfortunately, no. Even though it makes sense in our language, it will just confuse clients and developers with one more endpoint.

## Searching, ordering, limiting? Use query parameters!

This feature allows you to specify some filtering on List endpoints, here’s an example:

`GET /cakes/?name=apple`
If implemented, should list all the Cakes with apple in the name.

`GET /cakes/?name=apple&rating=4`
You can also concatenate parameters with &, search for apple and a 4 rating.

## Return. Errors. With. 4xx.
If you want to take just one thing from this article, make sure it’s this one: everybody freaking hates a response with HTTP 2xx and a message of error! Use the correct codes:

* **401**: Unauthorized access, where the authorization process wasn't done correctly.
* **403**: Forbidden access, the client is authorized, but has no access to the resource.
* **404**: The famous not found, indicating the resource is not available.

## Describe your errors with clarity
When something fails, inform the client about what happened and how it can recover. Here’s a nice way to do it:

```
{
  "error": {
    "type": "Authentication Error",
    "details": "Authentication could not be established with the given username and password",
   }
}
```

Everyone understands that.

## I’m a teapot
Implement the [status 418](https://en.wikipedia.org/wiki/Hyper_Text_Coffee_Pot_Control_Protocol) in a HTTP response. You know, just for fun!

## Resources are like object classes
The API endpoints will respond with a resource representation. Think about these resources as object classes, they then to represent things in the real world.


I could write a lot of tips here, but this post would become a book. And there’s already a good one, [Build APIs You Won’t Hate](https://apisyouwonthate.com/) by Phil Sturgeon.

Trust me on this, using a Design-first philosophy will give you better nights of sleep. Here are some advantages and characteristics of a good API:

* **Talk to your customers**: understand what they need, not what they want. An API without clients is nothing but a bad API.
* **Easy to use**: endpoints, resources and output data should follow the same structure as much as possible.
* **Hard to misuse**: if a bad request is made, return an error and be informative.
* **Simple is better than complex**: as a Pythonista, this is carved into my heart. Simple things are easy, in every aspect.
* **Use your API before implementing it**: create a mock server to get a feel of the end result. If you can, talk to your future customers and ask their opinions.
* **Be resilient**: when a crash happens, informe why and how to handle the situation.
* **Test everything**. Really. Write tests for every endpoint, method, parameter, input and output data.
* At the end of the day, your API is a new little language you’ll have to teach to other people.


# I’m almost done, just some references
If more tips could fit a book, talking about all aspects of API Design would fit a library. I’ll leave you, my patient reader, with some amazing references on further topics:

* [HTTP on Wikipedia](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) - a good reading about the Hypertext Transfer Protocol.
* [How To Design A Good API and Why it Matters](https://www.youtube.com/watch?v=aAb7hSCtvGw) - a talk by Joshua Bloch Java APIs, but the concepts apply to the web as well. Here are the slides.
* [RESTful Web APIs](http://shop.oreilly.com/product/0636920028468.do) by By Leonard Richardson, Mike Amundsena and Sam Ruby - learn everything about  the web, HTTP, hypermedia and domain-specific designs.
* [HATEOAS](https://spring.io/understanding/HATEOAS) - one of the constraints of RESTful APIs.
* [JSON API](http://jsonapi.org/) - a specification for build APIs in JSON.

# Wrapping up
API Design matters. It tends to stop people from just hacking things together to take a step back and see the bigger picture. Here at Cheesecake Labs we do our best to develop APIs following the best practices. We know that it seems time-consuming, but in the long run it pays off!
