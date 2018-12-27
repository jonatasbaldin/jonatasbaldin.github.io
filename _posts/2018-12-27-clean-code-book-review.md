---
layout: post
title: "[Book Review] Clean Code, by Uncle Bob"
date: 2018-12-27 10:00:00
description: "Way less mystical than I though, way more required than I imagined."
---

The first time I tried to read the Clean Code, by Robert C. Martin – or Uncle Bob –, I got a fright by this paragraph at the Introduction chapter:

> Be prepared to work hard while reading this book. This is not a "feel good" book that you can read on an airplane and finish before you land. This book will make you work, and work hard. What kind of work will you be doing? You'll be reading code—lots of code.

This plus my lack of time at the moment made me put the book back in the Kindle virtual shelve. 

Two (I think?) years later, I finally picked it up again and gave it a try. So far, one of the best decisions I've made as a software developer.

# What is in there?
We usually hear about the importance of naming when we learn about variables and functions, generally with statements like: `Create good names`. `Instead of c, the word car  customer`. As simply as it looks, it's incredible how we often don't give enough attention to it. Uncle Bob devotes a full chapter on the subject, referring back to it at any opportunity.

* Use intention revealing names.
* Avoid inconsistency. Don't use different – but similar – words like `retrieve` and `get` in the same meaning context.
* Use pronounceable names. Having a discussion about the code base with hard to say words is awful.
* Use searchable names. Searching for `MAX_CACHE_DAYS` is easier than searching for the constant `10`.

This is just a glimpse of the first chapter. Besides Naming, the book also touches topics like Functions, Comments, Formatting, Error Handling etc, all of them treated with the same care and full with precious, well written information.

The utter premise of the book is to show you bad code and how to refactor it into clean code. To accomplish that, the author illustrates all of his concepts with the Java language, which can be annoying.

* First, everything is Java. From a Pythonista point of view, some sections the refactoring is only necessary because it's Java, where in other languages you may not have the problem in the first place. The book could try to use examples in other dialects or be language agnostic. 
* Some code blocks are huge – multiple pages huge – and without any highlight. It can be hard to interpret code when you need to go back and forth in paper (or in a Kindle). Code is way easier to read in our favorite editor or IDE, where we can quickly jump all around the code.

Somehow this title seemed kinda mystical to me. Every time I looked at it I got that "I'm not read for you" feeling. I thought it was full with precious but complex information, and that my lack of experience would make it hard to grasp. It wasn't quit the case for most of the book, where the precious content was written in a simple way, but certainly not easy to apply withing a team. 

Also, I believe reading Clean Code at different levels of your career will give you different insights. I hope to pick it up again in a couple of years with a different mindset, consuming its ideas in a distinct way.

# Half Good Half Bad
The first half of the book is precious. The author dissects the fundamentals of what clean code means, taking often given advices like "choose your names carefully" and "misleading comment is worse than no comment" to another level, dedicating whole chapters on that. He goes through the basics of software development in a deeper and thoughtful way, proving that bad code doesn't happen out of nowhere: it is constructed by lousy everyday choices and lack of ongoing critical analysis.

At the second half, the book becomes too Java. A lot of people around the Internet agree the book should be titled Clean Code (but mainly Java). For instance, there are three chapters focused on refactoring huge amounts of Java code, which can be disappointing for developers that don't use it in a day-to-day basis.

---

At the end, Clean Code made me a better developer. It changed how I read and understand code, burning in my head a lot of small and useful concepts about writing better software. Today I feel much more able and comfortable to spot smells and their possible solutions. 