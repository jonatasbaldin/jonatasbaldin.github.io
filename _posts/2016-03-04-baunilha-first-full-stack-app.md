---
layout: post
title: "Baunilha, my first full stack app"
date: 2016-03-04 11:00:00
tags: ['python', 'flask', 'api', 'twitter', 'bootstrap']
description: "Inside joke: me and some friends were drinking and hanging out one night. As it goes, we started to read one of my friend's tweets. He just posts melancholic stuff about life and his pursued lovers, it was so silly it turns out comic. To make them even funnier, we put in Google Translator to be read. The night was over, I can't remember a day that I laugh so hard. This day the idea behind Baunilha was born."
comments: True
---

Inside joke: me and some friends were drinking and hanging out one night. As it goes, we started to read one of my friend's tweets. He just posts melancholic stuff about life and his pursued lovers, it was so silly it turns out comic. To make them even funnier, we put in Google Translator to be read. The night was over, I can't remember a day that I laugh so hard. This day the idea behind [Baunilha](http://baunilha.deployeveryday.com) was born.


<a href="http://baunilha.deployeveryday.com">![Bunilha Index Page](/img/baunilha_index.png)</a>


## Useless, but a chance to learn
I wanted to learn how to develop a full stack web application, from the operating system to the presentation layer, so I took that idea and transformed it in code. That's what Baunilha is all about, an **app that reads someone tweets**. Even more, it should read in PT-BR (I'm from Brazil, the inside joke makes more sense in Portuguese) and EN for the rest of the world.     
So I sketched what I was willing to learn and what I needed to. After some research, I got this:

* Where: [Amazon AWS](https://aws.amazon.com). I have some credit there, so...
* OS: [CentOS](https://www.centos.org/) my favorite Linux distribution.
* Web server: [NGINX](http://nginx.org/) for those who like performance.
* Language: [Python](https://www.python.org/)! Love it by heart and would be nice to improve it. Also, virtualenv to create a separated environment.
* Web server code gateway: [uWSGI](https://uwsgi-docs.readthedocs.org/), integrate seamleslly with NGINX.
* Talk to Twitter: [python-twitter](https://github.com/bear/python-twitter). Easy way to collect tweets, only works on Python 2.6+. Not a big problem tough.
* Read the tweets: [ResponsiveVoice.JS](http://responsivevoice.org) handle it nicely. Just one JavaScript line can do the job.
* Web framework: [Flask](http://flask.pocoo.org/) is simple enough to begginers and powerful enough to do whatever you want.
* Web presentation: [Bootstrap](http://getbootstrap.com/) is a gift from heaven (and Twitter's Team). I lose my patience too fast with CSS, and it helped a lot (even so, the app looks kinda ugly).

And this is my tool belt for now. Not hard, but indeed some fun work.     
The challenge was set. Insert a Twitter username, retrive its tweets and, when clicked, read it!    
I'll open source the code soon, just making some adjustments and developing an automated deploy.

## Hey Twitter, can you provide me [@vuashhhh](https://twitter.com) tweets?
First of all, I need to get the tweets, enter python-twitter. Haply reading its docs, I discovered that the entire Twitter's API (including the **public** part) needed a combination of API keys and secrets to allow access. Those can be generated at [Twitter Application Management](https://apps.twitter.com/) under my personal account.     
With the keys in hand, I wrote a little wrapper around python-twitter to get just what I need: username, tweets and its ID and a profile picture.

## Sugar, spice and everything nice
Or if you prefer: Flask, HTML and Bootstrap. Combining this three tools, you can create short and lean code, with a high level of customization, beauty and responsive web design.    
Flask is responsible for the backend stuff: dynamic page routing, error handling, forms, app configuration, inserting external logic and rendering templates (with [Jinja2](http://jinja.pocoo.org/)) to HTML. [Flask-WTF](https://flask-wtf.readthedocs.org) is a Flask extension that simplify creation and manipulation of forms, also providing security mechanisms against CSRF attacks.
Bootstrap gives life to the code. It is a CSS framework to arrange HTML objects in a neat way. Even being damm easy to use, I still stumble on cascading styles, and as a full stack personal project, it came out not as wonderful as I expected.

## Give the app a voice!
Get tweets from a user? Check. Render pages dynamically in HTML? Check. Read it when cliked? ResponsiveVoice.JS!     

```javascript
<script src='https://code.responsivevoice.org/responsivevoice.js'></script>
<p onclick="responsiveVoice.speak('I"m a nice robot voice!', 'US English Female');">
```

The lines above are an example of a clickable `<p>` HTML tag with some text. With that, I just need to put the tweet text as the first parameter. Voilà, a talking app.

## What about the core?
From AWS to uWSGI was smooth, it's the piece I'm used to. Just got a micro CentOS 7 EC2 instance up and running, installed NGINX and configured a basic uWSGI. This section was strictly manual my next goal is to refactor/automate the whole process.

## Work ahead
Besides all work (ok, maybe not that much), my Trello board still have some cards, like:

* Display the page in both Portuguese and English. Flask-Babel, maybe?
* Order tweets by published date.
* Load tweets as the page scrolls. Ajax? JS?
* Run the app with Docker.
* Write unit tests.
* Put the application behind a load balancer (manually with NGINX reverse proxy and automagically with AWS Auto Scaling).
* Learn how to monitor Python applications, set some type of warnings, logs, information.
* Automate the whole deploy process, from Github to use ready. **Coming soon!**

This list will possibly get outdated, but you can follow it on the GitHub repository when I publish the code.

## Conclusions
If develop a really simple app envolves so many different techs and areas of knowledge, I can just imagine how a massive application is engineered. Baunilha, as silly as it can be,  was the first step of a long journey through web dev!
