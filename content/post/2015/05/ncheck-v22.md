---
title: "NCheck V2.2"
date: 2015-05-28 19:30:00Z
draft: false
aliases:
- /post/ncheck-v2-2/
categories:
- Testing
- Patterns
---
Updated [NCheck](https://www.nuget.org/packages/NCheck/), my test object comparision helper, to version 2.2 which extends `ICheckerFactory` to support `Compare<T>` - this allows us to override checkers on a per-test basis which I've found useful on some of my current projects.

### Customizing the CheckerFactory

The `CheckerFactory` has a number conventions which are used to automatically construct Checkers for each class; these conventions can be overridden by the developer if they don't suit a particular scenario.

We can have type conventions, which apply to all instances of a particular type, property conventions which apply to all properties of a specified name irrespective of type, and comparer conventions which allow overriding of value comparison from object.Equals, e.g. to allow for error bounds on floating point numbers.

```csharp
public class CheckerFactory : NCheck.CheckerFactory
{
    public CheckerFactory()
    {
        PropertyCheck.IdentityChecker = new IdentifiableChecker();

        Initialize();
    }

    private void Initialize()
    {
        // NB Conventions must be before type registrations if they are to apply.
        Convention(x => typeof(IIdentifiable).IsAssignableFrom(x), CompareTarget.Id);
        Convention((PropertyInfo x) => x.Name == "Ignore", CompareTarget.Ignore);

        // NB We have an extension to use a function for a type or we can do it explicitly if we want more context
        ComparerConvention<double>(AbsDouble);
        ComparerConvention<double>(x => (x == typeof(double)), AbsDouble);

        // Pick up all explicitly defined checkers.
        Register(typeof(CheckerFactory).Assembly);
        Register(typeof(NCheck.CheckerFactory).Assembly);
    }

    public bool AbsDouble(double x, double y)
    {
        return Math.Abs(x - y) < 0.001;
    }

    public bool AbsFloat(float x, float y)
    {
        return Math.Abs(x - y) < 0.001;
    }
}
```

These conventions apply when using the automatic checker creation or when using Initialize inside a custom Checker, and any changes to the default should be registered inside `CheckerFactory.Initialize`.

```csharp
...
[Test]
public void CheckerFactoryRegisterTypeViaGeneric()
{
    var cf = new CheckerFactory();
    cf.Convention<SampleClass>(CompareTarget.Ignore);

    Assert.AreEqual(CompareTarget.Ignore, PropertyCheck.TypeConventions.CompareTarget.Convention(typeof(SampleClass)));
}

[Test]
public void DetermineValueBasedOnName()
{
    var targeter = new PropertyConventions();
    targeter.CompareTarget.Register(x => x.Name == "Ignore", CompareTarget.Ignore);

    CheckTargetType<SampleClass>(targeter, x => x.Ignore, CompareTarget.Ignore);  
}
...
```

### Customizing a checker

If you can't acheive your required behaviour with conventions, you will need to define a custom checker, example below...

```csharp
public class SimpleChecker : Checker<Simple>
{
    public SimpleChecker()
    {
        Compare(x => x.Id);
        Compare(x => x.Name);
        Compare(x => x.Value).Value<double>((x, y) => Math.Abs(x - y) < 0.001);
    }
}
```

This fully defines the behaviour of the Checker, including the use of a custom comparer to limit the precision of comparison for the double values.

You can also initialize the Checker with the default behaviour, and then override it as necessary as follows...

```csharp
public class SimpleChecker : Checker<Simple>
{
    public SimpleChecker()
    {
        Initialize();
        Compare(x => x.Id).Ignore;
    }
}
```

Let me know what you think.