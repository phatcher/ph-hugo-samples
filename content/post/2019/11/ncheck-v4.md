---
title: "NCheck V4"
date: 2019-11-21 19:30:00Z
draft: false
aliases:
- /post/ncheck-v4/
categories:
- Testing
- Patterns
---
I've release [NCheck V4](https://www.nuget.org/packages/NCheck/), largely tidy up and removal of legacy code and changed the framwork  to support net472, net48 and netstandard2.0.

The dictionary/list checking has been reworked as well so we can get per-element reporting e.g. if we look at the list test fixture we can see which elements are not in alignment,

We now get error messages like this...

```
SampleList.Children
Count: Expected:<3>. Actual:<2>
[1].Id: Expected:<2>. Actual:<4>
```

Useful when you have more than a couple of items in the list.