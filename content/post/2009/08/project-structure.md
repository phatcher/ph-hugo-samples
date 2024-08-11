---
title: "Project Structure"
date: 2009-08-27 22:57:34Z
draft: false
aliases:
- /post/project-structure/
categories:
- DevEnv
---
I try to keep the same project structure for all projects, this makes it easy when initiating new projects as you can automate the creation of the core project structure.

We have the usual subversion branches/tags/trunk structure and then within trunk we have

* builds : Contains all continous integration build files for the project.
* code : All the code for the project, flat directory structure internally with one project per directory.
* database: Database structure, produced using the Red Gate schema compare tool
* docs: Documentation source and any other project documentation
* lib: Holds the library files for the project, typically referenced via svn:externals but might really be here if only used by one project.
* model: Location of any models, I generally use Enterprise Architect for UML modelling so the files go here.

I'll explain about the builds directory a bit more in the post about continous integration, but the reason it is here physically is that then keeps **everything** about the project in one location rather than some bits being held by the CI project.