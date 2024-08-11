---
title: "Lazy Programming"
date: 2009-08-02 12:19:44Z
draft: false
aliases:
- /post/lazy-programming/
---
No this isn't about how Haskell evaluates stuff, but how people design and write code.

There's a mobile phone ad showing in the UK at the moment about how it's the impatient people that drive towards new innovation and in software at least I strongly disagree.

On the contrary, lazy programmers rule. I'm a lazy programmer, I can't bear writing the same code more than a couple of times and when I recognise the second instance I refactor it into a function. Why - basically its about how to keep the essential stuff in your head and good use of patterns.

Commercial software development isn't about being 'clever', but very open and explicit. Bear in mind that you may have to write an urgent fix for something you last looked at 6 months ago and if it looks like something written by an obfuscating program you won't enjoy the experience.

As an example I team I was on wrote a telemarketing system which ended up at around 70K lines of code, but there were only three or four major patterns, so we could easily isolate bugs and work out where new functionality belonged very quickly.

For me this leads on very nicely to unit testing and automated building. If I've unit tested my code, then it will be some *other*; guy that breaks the build and has to sort it all out. Again, if I can spend a little bit of time setting up my and my team's development environment to remove friction from the development process, this will pay for itself every single day of the project.