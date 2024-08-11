---
title: "Publishing a different branch to Azure WebSites"
date: 2016-09-09 11:00:00Z
draft: false
aliases:
- /post/publishing-a-different-branch-to-azure-websites/
categories:
- Azure
---
I’ve just started blogging again and decided to adopt [MiniBlog](https://github.com/madskristensen/MiniBlog), a minimal modern blog engine written by Mads Kristensen who also wrote BlogEngine.NET, as SubText has died.

It only has a small set of features and Mads intends to keep it like this, the concept is that you make your own fork on GitHub, adding the features you want and publish from there; this has a few consequences…

* Your blog credentials are visible in your GitHub fork 
* Potentially your content is there as well 
* How to test out new features written by yourself and/or from other forks.

I decided on a three branch strategy

* **master** : Pushed to GitHub, features that are completed in my own
* **dev** : Local branch to add new features, merged to publish once completed 
* **publish** : Local branch containing my configuration and content

I set up an AppService in Azure and configured it to [deploy](https://azure.microsoft.com/en-gb/documentation/articles/web-sites-deploy/) using a local Git repository, but hit an issue in that the Azure script is expecting to deploy the "master" branch (in Azure) and I want to publish my local “publish” branch.

One Google later and the answer is a slightly different form of git push than you are probably used to... `git push azure publish:master`

This tells git to push to Azure but to take the local publish branch and copy that onto the remote master branch; I know I’ll forget this as some point, hence the post :wink: