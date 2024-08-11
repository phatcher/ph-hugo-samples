---
title: "MVC 2 EditorForModel and DropDownList"
date: 2010-06-28 19:44:29Z
draft: false
aliases:
- /post/mvc-2-editorformodel-and-dropdownlist/
categories:
- Patterns
- MVC
---
Sometimes doing very basic things can be quite tricky in new technologies; case in point presenting a DropDownList (a ComboBox to those, including me, with a VB background) using the new EditorForModel syntax in ASP.NET MVC 2.

I had a little class in my MVP framework called `IdLabel` which as the name implied just held an Id and a Label so that I wasn’t forced to have a separate display and edit model for every trivial case, so I wanted to do the same sort of thing when writing my shiny new MVC apps. There’s a class called `SelectListItem` which fits the bill, but unfortunately a decision late in the MVC 2 Beta meant that by default, the object display/edit templates will not process anything other than basic types.

There is a slight hacky workaround which is to implement your own `TypeConverter` to tell the framework that it can be turned into a string, but this is normally then coupled by using an Attribute at compile time – and we don’t own `SelectListItem`. Fortunately, there’s a way around this using a little bit of reflection

```csharp
public static class TypeUtility
{
    /// <summary>
    /// Registers a type converter against a type we don't own
    /// </summary>
    /// <typeparam name="T"></typeparam>
    /// <typeparam name="TConverter"></typeparam>
    public static void RegisterTypeConverter<t tconverter="" />()
        where TConverter : TypeConverter
    {
        var attr = new Attribute[1];

        var vConv = new TypeConverterAttribute(typeof(TConverter));
        attr[0] = vConv;
        TypeDescriptor.AddAttributes(typeof(T), attr);
    }
}
```

So now starting with our very simple domain model

```csharp
public class Supplier
{
    public int Id { get; set; }
    public string Name { get; set; }
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public Supplier Supplier { get; set; }
}
```

we can have an equally simple view model

```csharp
public class ProductViewModel
{
    public int Id { get; set; }
    public string Name { get; set; }
    [DropDownList]
    public SelectListItem Supplier { get; set; }
}
```

Now the custom DropDownList attribute and associated classes are taken from [Kazi's blog](http://weblogs.asp.net/rashid/archive/2010/02/09/asp-net-mvc-complex-object-modelmetadata-issue.aspx) with the slight modification of adding a couple of constructor overloads so that the defaults work for `SelectListItem`.

Having done all that we can proceed to what the controller has to do, what’s shown below is a simplification of my current controller which I'll tell you about shortly.

```csharp
public ProductController 
{
     ...
     public ActionResult Edit(int id)
     {
          var entity = repository.FindOne<product />(id);
          var model = builder.Convert<productmodel />(entity);
          
          var types = repository.FindAll<supplier />();
          ViewData["SupplierList"] = builder.Convert<selectlistitem />(types);

          return VIew(model);
     }
     ...
}
```

The builder abstracts all of the work mapping the domain and view models and is based on AutoMapper.

Then we come to the display template, idea here is to show the label rather than the idea – one extension I have in mind is to convert this to a URL to allow display/edit of the related data.

```csharp
<%@ Control Language="C#" Inherits="System.Web.Mvc.ViewUserControl" %>
<script runat="server">
    string FormattedModelValue
    {
        get
        {
            // TODO: Might want to test for DropDownList attribute for other complex models.
            if (Model == null)
            {
                return null;
            }
            return Model is SelectListItem ? ((SelectListItem) Model).Text : Model.ToString();
        }        
    }
</script>
<%= Html.Encode(FormattedModelValue) %>
```

The next bit is the edit template. I’m using magic strings to couple the list definition of the DropDownList to avoid having to make the views strongly-typed on SelectLists; I view this as outside of the scope of the view model, though you may disagree. If there’s a need to alter this, the first parameter of the DropDownList attribute takes the ViewData key name.

```csharp
<%@ Control Language="C#" Inherits="System.Web.Mvc.ViewUserControl" %>
<%@ Import Namespace="Hippo.Web.Mvc.Templating" %>
<script runat="server">
    DropDownListAttribute GetDropDownListAttribute()
    {
        var metaData = ViewData.ModelMetadata as FieldTemplateMetadata;

        return (metaData != null) ? metaData.Attributes.OfType<DropDownListAttribute>().SingleOrDefault() : null;
    }
    
    IEnumerable<SelectListItem> GetSelectList()
    {
        var metaData = ViewData.ModelMetadata as FieldTemplateMetadata;        
        if (metaData == null)
        {
            return null;
        }

        var attribute = metaData.Attributes.OfType<DropDownListAttribute>().SingleOrDefault();
        if (attribute == null)
        {
            return null;
        }

        var key = attribute.ViewDataKey ?? (metaData.PropertyName + "List");
        var selected = attribute.GetSelectedValue(Model);
        // This makes it work for both SelectList and any other enumerable
        var sl = new SelectList(ViewData[key] as IEnumerable, attribute.DataValueField, attribute.DataTextField, selected);

        return sl;
    }
</script>
<% var attribute = GetDropDownListAttribute();%>
<% if (attribute != null) {%>
    <%= Html.DropDownList(null, GetSelectList(), attribute.OptionLabel, attribute.HtmlAttributes) %>
<% }%>
<% else {%>
    <%= Html.DisplayForModel()%>
<% }%>
```

The non-obvious problem here is that due to a bug in MVC 2, the Html.DropDownList builder ignores any selected item that you pass it and also does not look at the model (the offending line appears to be line 188 in SelectExtensions). The trick I found (today) was to push the Model value into the ViewData against the PropertyName, it does pick that up and pushes it as the selected item – there won’t be anything else there as we have a strongly-typed view, apart from the SelectLists.

One other thing to note is that Kazi’s DropDownList attribute can be given a list of anything and you can specify the Value and Text properties to be pushed into the SelectList. Also, your model doesn’t have to use SelectListItem, but if you don’t be prepared to write a TypeConverter each time you want a DropDownList!