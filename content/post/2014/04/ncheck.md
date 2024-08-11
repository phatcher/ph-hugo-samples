---
title: "NCheck: A object comparison helper"
date: 2014-04-16 19:30:00Z
draft: false
aliases:
- /post/ncheck-a-object-comparison-helper/
categories:
- Testing
- Patterns
---
With unit testing one of the precepts is to check one thing in your test, but when you have objects this is difficult to express, since you want to check that the your business process has updated all of the expected properties (positive testing) and also that has not modified things that are not expected to have changed (negative testing).

This becomes even more problematic when you have an object graph, e.g. Order –> OrderLine –> Product where you have to navigate the graph, and reporting accurately where any errors is also a major issue as you need to track property names, collection indexes, etc. etc. 

I wrote something to help me a few years ago and due to the ~~nagging~~ nudging of a colleague (I’m looking at you Rob Bagby), I’ve finally reformed it into something that’s hopefully useable by everyone else, called [NCheck](https://github.com/phatcher/NCheck).

What this allows you to do is compare objects and the library will do a property by property comparison for you, handling collection properties and other referenced entities. What is nice I think, is that you still have control over what properties are included in the comparison and there are helpers for breaking object cycles using the your domain’s concept of identity.

Here’s a short introduction, given a simple class
```csharp
public class Simple
{
    public int Id { get; set; }

    public string Name { get; set; }

    public double Value { get; set; }
}
```
we would like to test what happens after running some business service over it, we can do this easily with an NUnit test

```csharp
[TestFixture]
public class SimpleTest
{
    [Test]
    public void AlgoTest()
    {
        var algo = new ShinyBusinssService();
        var source = new Simple { Id = 1, Name = "A", Value = 1.0 } ;
        var candidate = algo.Run(source);
        Assert.AreEqual(2, candidate.Id, "Id differs");
        Assert.AreEqual("B", candidate.Name, "Name differs");
        Assert.AreEqual(1.2, candidate.Value, "Value differs");
    }
}
```

If we re-write this using NCheck we get

```csharp
[TestFixture]
public class SimpleTest
{
    [Test]
    public void AlgoTest()
    {
        var checkerFactory = new CheckerFactory();
        var algo = new ShinyBusinessService();
        var source = new Simple { Id = 1, Name = "A", Value = 1.0 } ;

        var expected = new Simple { Id = 2, Name = "B", Value = 1.2 } ;

        var candidate = algo.Run(source);
        checkerFactory.Check(expected, candidate);
    }
}
```
Behind the scenes, the CheckFactory class has created a comparison checker (using reflection) to compare each property of the Simple class, but you can also explicitly create a Checker<T> class if you want complete control.

The library is available as a [NuGet package](https://www.nuget.org/packages/NCheck/) and the source code is on [GitHub](https://github.com/phatcher/NCheck)
