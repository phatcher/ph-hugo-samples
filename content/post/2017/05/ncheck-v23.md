---
title: "NCheck V2.3"
date: 2015-05-29 19:30:00Z
draft: false
aliases:
- /post/ncheck-v2-3/
categories:
- Testing
- Patterns
---
Updated [NCheck](https://www.nuget.org/packages/NCheck/), my test object comparision helper, to version 2.3 which...

* Introduces a new technique to wire up conventions
* Supports multi-threading e.g. test fixtures using NUnit Parallelizable

### Customizing the CheckerFactory

The `CheckerFactory` has a number conventions which are used to automatically construct Checkers for each class; these conventions can be overridden by the developer if they don't suit a particular scenario.

We support...

* Type conventions: Applied to all instances of a particular type
* Property conventions: Applied to properties which satisfy a function e.g. matching a particular name
* Comparer conventions: Operates on a type or property convention and changes Value comparison from object.Equals to a specified comparer; useful for values such putting error bound on floating point comparison.

The suggested technique for this is to introduce an extension class to hold the factory methods
```csharp  
public static class ConventionExtensions
{
    public static void AssignPropertyInfoConventions(this CheckerConventions conventions)
    {
        //conventions.Convention(x => typeof(IIdentifiable).IsAssignableFrom(x), CompareTarget.Id);
        conventions.Convention((PropertyInfo x) => x.Name == "Ignore", CompareTarget.Ignore);
    }

    public static void AssignTypeConventions(this CheckerConventions conventions)
    {
        // NB Must have this one to put base behaviour in suchs as Guid
        conventions.TypeConventions.InitializeTypeConventions();

        // NB Conventions must be after general type registrations if they are to apply.
        conventions.Convention(x => typeof(IIdentifiable).IsAssignableFrom(x), CompareTarget.Id);
    }

    public static void AssignComparerConventions(this CheckerConventions conventions)
    {
        // NB We have an extension to use a function for a type or we can do it explicitly if we want more context
        conventions.ComparerConvention<double>(AbsDouble);
        conventions.ComparerConvention<double?>(AbsDouble);

        conventions.ComparerConvention<float>(AbsFloat);
        conventions.ComparerConvention<float?>(AbsFloat);
        conventions.ComparerConvention<float>(x => (x == typeof(float)), AbsFloat);
    }

    public static bool AbsDouble(double? x, double? y)
    {
        return x.HasValue && y.HasValue && AbsDouble(x.Value, y.Value);
    }

    public static bool AbsDouble(double x, double y)
    {
        return NearlyEqual(x, y, 0.001);
    }

    public static bool AbsFloat(float? x, float? y)
    {
        return x.HasValue && y.HasValue && AbsFloat(x.Value, y.Value);
    }

    public static bool AbsFloat(float x, float y)
    {
        return NearlyEqual(x, y, 0.00001);
    }

    /// <summary>
    /// Compare two floats and check if they are approximately equal
    /// </summary>
    /// <param name="a"></param>
    /// <param name="b"></param>
    /// <param name="epsilon"></param>
    /// <returns></returns>
    /// <remarks>http://stackoverflow.com/questions/3874627/floating-point-comparison-functions-for-c-sharp</remarks>
    public static bool NearlyEqual(float a, float b, float epsilon)
    {
        var absA = Math.Abs(a);
        var absB = Math.Abs(b);
        var diff = Math.Abs(a - b);

        if (a == b)
        {
            // shortcut, handles infinities
            return true;
        }

        if (a == 0 || b == 0 || diff < float.Epsilon)
        {
            // a or b is zero or both are extremely close to it
            // relative error is less meaningful here
            return diff < epsilon;
        }

        // use relative error
        return diff / (absA + absB) < epsilon;
    }

    public static bool NearlyEqual(double a, double b, double epsilon)
    {
        var absA = Math.Abs(a);
        var absB = Math.Abs(b);
        var diff = Math.Abs(a - b);

        if (a == b)
        {
            // shortcut, handles infinities
            return true;
        }

        if (a == 0 || b == 0 || diff < double.Epsilon)
        {
            // a or b is zero or both are extremely close to it
            // relative error is less meaningful here
            return diff < epsilon;
        }

        // use relative error
        return diff / (absA + absB) < epsilon;
    }
}
```

Then it's easy to introduce this in your `CheckerFactory` where you register any custom checkers

```csharp  
public class CheckerFactory : NCheck.CheckerFactory
{
    public CheckerFactory()
    {
        // NB Deliberate virtual call so we invoke AssignConventions in the most derived CheckerFactory.
        AssignConventions();

        Register(typeof(CheckerFactory).Assembly);
        Register(typeof(NCheck.CheckerFactory).Assembly);
    }

    /// <summary>
    /// Can assigns the conventions for the instance or configure ConventionsFactory as needed.
    /// </summary>
    protected virtual void AssignConventions()
    {
        if (ConventionsFactory.FactoryType == null)
        {
            // Ok, first time through so set it up
            ConventionsFactory.IdentityCheckerFactory = () => new IdentifiableChecker();
            ConventionsFactory.TypeConventionsFactory = c => c.AssignTypeConventions();
            ConventionsFactory.PropertyConventionsFactory = c => c.AssignPropertyInfoConventions();
            ConventionsFactory.ComparerConventionsFactory = c => c.AssignComparerConventions();

            // Mark it as setup
            ConventionsFactory.FactoryType = GetType();
        }

        // Sanity check - only needed if you are changing conventions on a per-test basis
        if (Conventions.IdentityChecker is IdentifiableChecker)
        {
            // Assume it's ok
            return;
        }

        Conventions.IdentityChecker = new IdentifiableChecker();
    }
}
```

These conventions apply when using the automatic checker creation or when using Initialize inside a custom Checker, and any changes to the default should be registered inside `CheckerFactory.Initialize`.

```csharp
...
    [Test]
    public void CheckerFactoryRegisterTypeViaGeneric()
    {
        var cc = new CheckerConventions();
        cc.Convention<SampleClass>(CompareTarget.Ignore);

        Assert.That(cc.TypeConventions.CompareTarget.Convention(typeof(SampleClass)), Is.EqualTo(CompareTarget.Ignore));
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

If you can't achieve your required behaviour with convention, you will need to define a custom checker, example below...

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

### Managing Object Identity

For complex object graphs it may not be necessary, or useful (think about cyclic references) to compare all properties, but to just ensure than the identity of the expected and candidate entities are the same. To this end the library supports the idea of an identity checker via IIdentityChecker which allows the developer to implement a domain-specific checker.

Say we have an interface in our domain model IIdentifiable as follows, with a sample usage

```csharp
public interface IIdentifiable
{
    int Id { get; set; }
}

public class Customer : IIdentifiable
{
    public int Id { get; set; }

    public string Name { get; set; }
}
```

We can write an implementation of IIdentityChecker which limits to just checking the identity of the object instances, this is useful if the database is involved and values
such as audit information may have been updated.

```csharp
public class IdentifiableChecker : IIdentityChecker
{
    public bool SupportsId(Type type)
    {
        return typeof(IIdentifiable).IsAssignableFrom(type);
    }

    public object ExtractId(object value)
    {
        var x = value as IIdentifiable;
        return x == null ? null : x.Id;
    }
}
```

You then register and instance of this class in your custom CheckerFactory
```csharp
...
    public CheckerFactory()
    {
        PropertyCheck.IdentityChecker = new IdentifiableChecker();

        Initialize();
    }
...
```

This can then be easily used to break object cycles as follows...

```csharp
public interface IIdentifiable
{
    int Id { get; set; }
}

public class Order : IIdentifiable
{
    public int Id { get; set; }

    public string Name { get; set; }
    
    public IList<OrderLine> { get; set; }
    
    ....
}

public class OrderLine : IIdentifiable
{
    public int Id { get; set; }
    
    public Order Order { get; set; }
    
    ....
}

public class OrderLineChecker : Checker<OrderLine>
{
    public OrderLineChecker()
    {
        Initialize();
        Compare(x => x.Order).Id;
    }
}
```

### Allowing for failure

For negative testing, you might need to prove that the comparison fails in a particular manner, we allow for this with a couple of overloads...

```csharp
[TestFixture]
public class SimpleTest
{
    [Test]
    public void AlgoFailTest()
    {
        var checkerFactory = new CheckerFactory();

        var algo = new ShinyBusinessService();
        var source = new Simple { Id = 1, Name = "A", Value = 1.0 } ;

        var expected = new Simple { Id = 2, Name = "B", Value = 1.0 } ;

        var candidate = algo.Run(source);

        CheckFault(expected, candidate, "Simple.Value", 1.0, 1.2);
    }
}
```

This will check for a specific failure in the comparsion, the other overload allows you to provide the exact message rather than letting the library format it.

### Per-Test customization

For specific tests, you might want to override the standard Checker for the class, be it an automatically constructed or one you have explicitly defined.

To do this, `ICheckerFactory` exposes the `Compare<T>` interface used to specify property comparisons, here are some examples taken from the unit tests; the Parent checker has been defined to specifically ignore the Another property.
```csharp
...
public class ParentChecker : Checker<Parent>
{
    public ParentChecker()
    {
        Initialize();
        Compare(x => x.Another).Ignore();
    }
}
...
[Test]
public void IncludeAnotherProperty()
{
    var expected = new Parent { Id = 1, Name = "A", Another = 1, };
    var candidate = new Parent { Id = 1, Name = "A", Another = 1, };

    Compare<Parent>(x => x.Another).Value();
    Check(expected, candidate);
}

[Test]
public void IncludeAnotherPropertyComparisonFails()
{
    var expected = new Parent { Id = 1, Name = "A", Another = 1, };
    var candidate = new Parent { Id = 1, Name = "A", Another = 2, };

    Compare<Parent>(x => x.Another).Value();
    CheckFault(expected, candidate, "Parent.Another", 1, 2);
}

[Test]
public void ExcludeNameProperty()
{
    var expected = new Parent { Id = 1, Name = "B", Another = 2 };
    var candidate = new Parent { Id = 1, Name = "A", Another = 1, };

    Compare<Parent>(x => x.Name).Ignore();
    Check(expected, candidate);
}
...
```