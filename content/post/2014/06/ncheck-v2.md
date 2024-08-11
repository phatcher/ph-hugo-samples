---
title: "NCheck V2"
date: 2014-06-29 19:30:00Z
draft: false
aliases:
- /post/ncheck-v2/
categories:
- Testing
- Patterns
---
Just release V2 of [NCheck](https://www.nuget.org/packages/NCheck/), my test object comparision helper which tidies up a few
things that simplify custom comparisions and excluding properties.

1. Reworked the `ITypeCompareTargter`/`IPropertyCompareTargeter` into a a more generic `IConvention<Source, Target>`
1. Introduced custom comparers and use convention-based registration for them.
1. Retired Exclude, you can now use `Compare(x => x.PropertyName).Ignore`, unifies the syntax compared to other options.