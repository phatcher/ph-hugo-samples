---
title: "Frictionless Development Environment"
date: 2009-08-03 12:24:26Z
draft: false
aliases:
- /post/frictionless-development/
categories:
- DevEnv
---
One of the things I've been working on over the last couple of years for myself and for clients, is making the development environment as frictionless as possible. By this I mean removing all the little annoyances that get in the way of producing good quality software.

This guiding idea has led me to one guiding principal: keep everything under source control!

For example, one classic problem I've encountered is that the build works on the developer's workstation but not on the build server or vice versa. To solve this, we now put all of our tools, excluding Visual Studio, under source control, this minimizes the differences between the developers' machines and build environment.

Another little trick is the test server used for database unit testing; most tests run without using the database but it's difficult to do that when you're testing your NHibernate mappings! What we do is use a standard server name in the connection string e.g. "dbtestserver" and then map this via the hosts file to the appropriate physical box. The developer can then adjust the hosts file to point at their local box so that the local test runs don't interfere with anyone else.

I'll be posting a lot more on this once I can organize it properly.