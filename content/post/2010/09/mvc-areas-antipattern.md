---
title: "MVC Areas Anti-Pattern"
date: 2010-09-08 22:35:54Z
draft: false
aliases:
- /post/mvc-areas-anti-pattern/
categories:
- Patterns
- MVC
---
I was working on the design of a small site where we have public display of information in one format and a set of administrative screens that allow authorised users to create and edit the information.

“I know”, I thought, “this is a perfect case for MVC Areas” – and boy was I wrong!
I should have realised that the problems had started when all the routing started playing up.. 

* Route names must be unique across *all* areas
* If there are controller names in common in different areas, all routes need to be updated to bind the namespace of the controller you want!
 
Yuck! – did the MVC team think about the [open-closed principle](http://en.wikipedia.org/wiki/Open/closed_principle)?

Then I realised I’d made a category error – mis-applying the area concept. I was making the same sort of error that people make when implementing SOA, breaking down services according to functional granularity rather than business function.

So, instead of having two Product controllers, one in the main area, the other in the admin area, I should have one Product controller, still securing the appropriate methods, and still using routes to present some of the views under ~/admin/product and some under ~/product.

Areas should be reserved for coherent chunks of business functionality e.g. Sales, Marketing etc and I’m still not particularly happy that if I have a concept in common e.g. Customer, then the routes are going to clash, but I’m glad I realised my error before I’d built even more in the wrong direction.