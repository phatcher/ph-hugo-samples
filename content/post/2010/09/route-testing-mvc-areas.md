---
title: "Route Testing and MVC 2.0 Areas"
date: 2010-09-05 18:12:31Z
draft: false
aliases:
- /post/route-testing-and-mvc-2-0-areas/
categories:
- MVC
---
I’ve been using the MvcContrib.TestHelper class for a while now to get rather nice fluent testing of the routes in my MVC applications.

A typical test class would look like this
```csharp
public class CategoryRoutesFixture : RoutesFixture
{
    private const int categoryId = 34;

    private static string BaseUrl
    {
        get { return "~/admin/Category"; }
    }

    [Test]
    public void Index()
    {
        BaseUrl.ShouldMapTo&amp;lt;CategoryController&amp;gt;(x =&amp;gt; x.Index());
    }

    [Test]
    public void Create()
    {
        (BaseUrl + "/create").ShouldMapTo&amp;lt;CategoryController&amp;gt;(x =&amp;gt; x.Create());
    }

    [Test]
    public void Show()
    {
        (BaseUrl + "/" + categoryId).ShouldMapTo&amp;lt;CategoryController&amp;gt;(x =&amp;gt; x.Show(categoryId));
    }

    [Test]
    public void Edit()
    {
        (BaseUrl + string.Format("/{0}/edit", categoryId)).ShouldMapTo&amp;lt;CategoryController&amp;gt;(x =&amp;gt; x.Edit(categoryId));
    }
}
```
The base class just takes care of initialising the routes collection from the MvcApplication.

I’ve just had a need to use the new Areas functionality in MVC 2.0, and one interesting wrinkle (there are a few!) is that that the     `RegisterAllAreas` functionality wasn’t built with testing particularly in mind as it seems to need a running instance of ASP.NET to function correctly. After trying a few things and wading through even more blog posts I finally found an answer on StackOverflow.

The TestHelper class is not area aware, so the workaround is to configure the RoutesCollection appropriately for the class you are testing, so now for my main area I have

```csharp
protected override void InitializeRoutes(RouteCollection routes)
{
    MvcApplication.RegisterRoutes(routes);
}
```

Whereas in my admin area I have

```csharp
protected override void InitializeRoutes(RouteCollection routes)
{
    RouteTable.Routes.Clear();
    var adminReg = new AdminAreaRegistration();
    adminReg.RegisterArea(new AreaRegistrationContext(adminReg.AreaName, RouteTable.Routes));
}
```

Not particularly tricky once you know about it, but a pain to discover if you don't!
