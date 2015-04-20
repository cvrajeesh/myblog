title: "ASP.Net MVC – Conditional Rndering Partial Views with Action<T> Delegate"
date: 2010-01-28 11:33:12
tags:
---

This is an update to my previous post regarding {% post_link ASP-Net-MVC-Conditional-Rendering-Partial-Views "conditional rendering partial views" %}, in that I used the internal implementation of the `Html.RenderPartail(…)` method to create the Html extension. Later I found a simple way to achieve the same using `Action<T>` delegate

`<% Html.PartialIf(this.Model.Exists, html => html.RenderPartial("MyPartialView")); %>`

If you look at the `PartialIf` implementation, it is simple, cleaner than the previous technique I have mentioned in my post.

```cs
public static void PartialIf(this HtmlHelper htmlHelper, bool condition, Action<HtmlHelper> action)
{
    if (condition)
    {
        action.Invoke(htmlHelper);
    }
}
```

That’s it :)
