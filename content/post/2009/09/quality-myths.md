---
title: "Quality Myths"
date: 2009-10-09 13:44:31Z
draft: false
aliases:
- /post/quality-myths/
categories:
- DevEnv
- DevOps
- Testing
---
Microsoft has [published](http://research.microsoft.com/en-us/news/features/nagappan-100609.aspx) the results of some empirical studies about how development practices affect quality

* Test Driven Development improves quality by 60 to 90 percent but takes 15 to 35 percent more ‘up front’ time. The time spent is compensated by savings in maintenance time later on.
* Team & organization structure has a huge impact on quality. Although this is conventional wisdom the study publishes figures to prove this. The metrics used data such as how many engineers are involved in a project, how many times individual source files were modified.
* Code coverage in tests must be used intelligently, and has less overall impact than other factors.

Another [paper](http://research.microsoft.com/en-us/projects/esm/nagappan_tdd.pdf) highlights the issues of not running the unit tests....

> Specifically, recent contact with the IBM team indicated that in one of the subsequent releases (more than five releases since the case study) some members of the team (grown 50% since the first release) have taken some shortcuts by not running the unit tests, and consequently the defect density increased temporally compared to previous releases.

A friend of mine pointed me at this video as a further illustration {{< youtube l1wKO3rID9g >}}