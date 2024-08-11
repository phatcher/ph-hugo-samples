---
title: "Subversion Upgrade and svn:externals changes"
date: 2010-03-23 23:23:22Z
draft: false
aliases:
- /post/subversion-upgrade-and-svnexternals-changes/
---
I'm just in the process of restructuring my source control environment, splitting the single subversion repository into multiple repositories.

Couple of reasons for this..

My repository is getting a bit big at 1.5Gb a big chunk of this are binary files produced by my continuous integration process, so I want to break this out to a dedicated repository.

I'd also like to put each client's work into a separate repository, this makes it easier to archive off and/or remove it at the end of a project as there's no way to delete files from a repository apart from dumping it and filtering it into a new one.

One nice new feature I've found that was introduced in 1.5 (I'm using 1.6.4) is the new syntax for svn:externals

Externals are a way of referencing another repository location which can be in the same repository or a different one. This is how I use my floating tags idea to keep projects up to date across minor changes.

In the project's lib directory, I have a svn:externals entry that points at all the libraries that the project depends on

The old format looked like this..

```
logging svn://myrepository.com/Binaries/Log4Net/tags/1.2.10.0/bin
nhibernate svn://myrepository.com/Binaries/NHibernate/tags/2.0.x/bin
```

In svn 1.5 they have introduced a couple of new syntax options that means that you don't have to embed the access type (svn, http, https etc) or the server, but the syntax changes slightly to

```
/Binaries/Log4Net/tags/1.2.10.0/bin logging
/Binaries/NHibernate/tags/2.0.x/bin nhibernate
```

This will make life much easier if I have to relocate a repository etc. Check out the [release notes](http://subversion.tigris.org/svn_1.5_releasenotes.htm) for more details
