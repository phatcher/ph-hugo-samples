---
title: "Unauthenticated users in Meerkat.Security"
date: 2016-09-20 14:00:00Z
draft: false
aliases:
- /post/unauthenticated-users-in-meerkatsecurity/
categories:
- MVC
- Security
---
Couple of changes released today, one to [Meerkat.Security](https://github.com/phatcher/Meerkat.Security) and the other to [Meerkat.Caching](https://github.com/phatcher/Meerkat.Caching).

The first was to introduce handling for grant/deny of actions for unauthenticated users. I’d missed this use case before as to date, the project was just used in a corporate Windows environment and so everyone was authenticated by the time we saw them. 

As I’m now working on a project using [Azure B2C Active Directory](https://azure.microsoft.com/en-gb/services/active-directory-b2c/) we are now back to the point where users need to authenticate, so you want to filter the actions they can see/perform before and after they authenticate, e.g. you want to allow unauthenticated users access to Home.Index and error pages but probably not much more, and as the principle of the project was to centralize the security checking, we need a way to allow unauthenticated users without using AllowAnonymous on the controller action.

This lead to discovering a bug in Meerkat.Caching, where we tried to store a null in the cache and the underlying implementation, MemoryObjectCache, throws an exception in this case. The fix was to trap the nulls and to ignore attempts to persist null to cache; don’t want to have to check on every call to the cache that we don’t have a null!