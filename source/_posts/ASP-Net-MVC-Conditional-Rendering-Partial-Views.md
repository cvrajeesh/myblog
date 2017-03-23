title: "ASP.Net MVC - Conditional Rendering Partial Views"
date: 2010-01-21 11:26:47
tags:
- ASP.NET MVC
---

**Update:** Later I found a cleaner and simple approach to do the same – read this post {% post_link ASP-Net-MVC–Conditional-Rndering-Partial-Views-with-Action-T-Delegate "ASP.Net MVC – Conditional rendering Partial Views with Action<T> delegate" %}

Following my previous post about {% post_link ASP-Net-MVC-Conditional-Rendering "Conditional Rendering" %}, one of my colleague asked me how to render the partial view based on a condition.

Normal way of doing this is

```
<% if(this.Model.Exists)
 {
     Html.RenderPartial("MyPartialView");
 } %>
```

I am not sure about any other technique for rendering partial view conditionally other than this (correct me if I am wrong :) ).

Then I thought about copying the pattern I have used in my {% post_link ASP-Net-MVC-Conditional-Rendering previous %} post and came up with this code which could conditionally render partial views and you could use the Html extension like below, which more clean than the previous

`<% Html.PartialIf(this.Model.Exists, "MyPartialView"); %>`

Below is the Html extension I have created

```cs
public static void PartialIf(this HtmlHelper htmlHelper, bool condition, string viewName)
{
   if (condition)
   {
       RenderPartialInternal(
           htmlHelper.ViewContext,
           htmlHelper.ViewDataContainer,
           viewName,
           htmlHelper.ViewData,
           null,
           ViewEngines.Engines);
   }
}

public static void PartialIf(this HtmlHelper htmlHelper, bool condition, string viewName, ViewDataDictionary viewData)
{
   if (condition)
   {
       RenderPartialInternal(
           htmlHelper.ViewContext,
           htmlHelper.ViewDataContainer,
           viewName,
           viewData,
           null,
           ViewEngines.Engines);
   }
}

public static void PartialIf(this HtmlHelper htmlHelper, bool condition, string viewName, object model)
{
   if (condition)
   {
       RenderPartialInternal(
           htmlHelper.ViewContext,
           htmlHelper.ViewDataContainer,
           viewName,
           htmlHelper.ViewData,
           model,
           ViewEngines.Engines);
   }
}

public static void PartialIf(this HtmlHelper htmlHelper, bool condition, string viewName, object model, ViewDataDictionary viewData)
{
   if (condition)
   {
       RenderPartialInternal(
           htmlHelper.ViewContext,
           htmlHelper.ViewDataContainer,
           viewName,
           viewData,
           model,
           ViewEngines.Engines);
   }
}
```

And here is the helper method I have used in the about extension ( most of the code snippet is taken from the MVC method `Html.Renderpartial(…)`, thanks to open source )

```cs
internal static IView FindPartialView(ViewContext viewContext, string partialViewName, ViewEngineCollection viewEngineCollection)
{
    var result = viewEngineCollection.FindPartialView(viewContext, partialViewName);
    if (result != null)
    {
        if (result.View != null)
        {
            return result.View;
        }
    }

    throw new InvalidOperationException("Partial view not found");
}

internal static void RenderPartialInternal(ViewContext viewContext, IViewDataContainer viewDataContainer, string partialViewName, ViewDataDictionary viewData, object model, ViewEngineCollection viewEngineCollection)
{
    if (String.IsNullOrEmpty(partialViewName))
    {
        throw new ArgumentException("PartialViewName can't be empty or null.");
    }

    ViewDataDictionary newViewData = null;

    if (model == null)
    {
        newViewData = viewData == null ? new ViewDataDictionary(viewDataContainer.ViewData) : new ViewDataDictionary(viewData);
    }
    else
    {
        newViewData = viewData == null ? new ViewDataDictionary(model) : new ViewDataDictionary(viewData) { Model = model };
    }

    var newViewContext = new ViewContext(viewContext, viewContext.View, newViewData, viewContext.TempData);
    var view = FindPartialView(newViewContext, partialViewName, viewEngineCollection);
    view.Render(newViewContext, viewContext.HttpContext.Response.Output);
}
 ```

Hope this helps
