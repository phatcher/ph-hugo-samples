---
title: "OData Delta Patch Security"
date: 2018-01-05 19:30:00Z
draft: false
categories:
- OData
---
My main project is using OData a lot and one of the requirements was to ensure that certain properties should not be updatable via PATCH - this is to ensure the integrity of the object e.g. audit fields should not be changeable post-hoc.

There's a couple of possibilities, depending on your use case...

1. You want to exclude the changes if they are supplied 
1. You want to throw an error if non-editable fields are updated.

The way we approached this was to start with an attribute

```csharp
/// <summary>
/// Marks a property as non-editable.
/// </summary>
[AttributeUsage(AttributeTargets.Property, AllowMultiple = false, Inherited = true)]
public class NonEditableAttribute : Attribute
{
}
```

Then we can write some extensions against Delta to take advantage of this

```csharp
public static class PatchExtensions
{
    /// <summary>
    /// Get the properties of a type that are non-editable.
    /// </summary>
    /// <param name="type"></param>
    /// <returns></returns>
    public static IList<string> NonEditableProperties(this Type type)
    {
        return type.GetProperties().Where(x => Attribute.IsDefined(x, typeof(NonEditableAttribute))).Select(prop => prop.Name).ToList();
    }

    /// <summary>
    /// Get this list of non-editable changes in a <see cref="Delta{T}"/>.
    /// </summary>
    /// <typeparam name="T"></typeparam>
    /// <param name="delta"></param>
    /// <returns></returns>
    public static IList<string> NonEditableChanges<T>(this Delta<T> delta)
        where T : class
    {
        var nec = new List<string>();
        var excluded = typeof(T).NonEditableProperties();

        nec.AddRange(delta.GetChangedPropertyNames().Where(x => excluded.Contains(x)));

        return nec;
    }

    /// <summary>
    /// Exclude changes from a <see cref="Delta{T}"/> based on a list of property names
    /// </summary>
    /// <typeparam name="T"></typeparam>
    /// <param name="delta"></param>
    /// <param name="excluded"></param>
    /// <returns></returns>
    public static Delta<T> Exclude<T>(this Delta<T> delta, IList<string> excluded)
        where T : class
    {
        var changed = new Delta<T>();

        foreach (var prop in delta.GetChangedPropertyNames().Where(x => !excluded.Contains(x)))
        {
            object value;
            if (delta.TryGetPropertyValue(prop, out value))
            {
                changed.TrySetPropertyValue(prop, value);
            }
        }

        return changed;
    }

    /// <summary>
    /// Exclude changes from a <see cref="Delta{T}"/> where the properties are marked with <see cref="NonEditableAttribute"/>
    /// </summary>
    /// <typeparam name="T"></typeparam>
    /// <param name="delta"></param>
    /// <returns></returns>
    public static Delta<T> ExcludeNonEditable<T>(this Delta<T> delta)
        where T : class
    {
        var excluded = typeof(T).NonEditableProperties();
        return delta.Exclude(excluded);
    }
}
```

And a domain class

```csharp
public class Customer 
{
    public int Id { get; set; }

    public string Name { get; set; }

    [NonEditable]
    public string SecurityId { get; set; }
}
```

Finally your controller can then take advantage of this in the Patch method

```csharp
public async Task<IHttpActionResult> Patch([FromODataUri] int key, Delta<Customer> delta)
{
    var patch = delta.ExcludeNonEditable();

    // TODO: Your patching action here
}
```

or 

```csharp
public async Task<IHttpActionResult> Patch([FromODataUri] int key, Delta<Customer> delta)
{
    var nonEditable = delta.NonEditableChanges();
    if (nonEditable.Count > 0)
    {
        throw new HttpException(409, "Cannot update as non-editable fields included");
    }

    // TODO: Your patching action here
}
```
