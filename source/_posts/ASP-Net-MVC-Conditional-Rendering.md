title: "ASP.Net MVC - Conditional Rendering"
date: 2010-01-03 11:15:15
tags:
- ASP.NET MVC
---

We come across situations like rendering elements based on the conditions in Model. For example, in the view if we want to show a TextBox if some property is true. A normal way of doing this is like below

```
<% if (this.Model.Exists) { %>
      <%= Html.TextBox("Test") %>
<% } %>
```

This looks like old classic asp style, and when it comes to code maintenance this will be a pain. So a clean way is to use an Html helper to generate this

`<%= Html.If(this.Model.Exists, action => action.TextBox("Name")) %>`

which looks cleaner than the old one. Source code for this helper method is

```cs
public static string If(this HtmlHelper htmlHelper, bool condition, Func<HtmlHelper, string> action)
{
    if (condition)
    {
        return action.Invoke(htmlHelper);
    }

    return string.Empty;
}
```

What about IfElse condition, we could write another helper method for that

```cs
public static string IfElse(this HtmlHelper htmlHelper, bool condition, Func<HtmlHelper, string> trueAction, Func<HtmlHelper, string> falseAction)
{
    if (condition)
    {
        return trueAction.Invoke(htmlHelper);
    }

    return falseAction.Invoke(htmlHelper);
}
```

Ok, now we got a conditionally rendered element, sometimes we may have to put a wrapper around this element. So I have written another Html helper method which will help you to put any html element around a particular element.

```cs
public static string HtmlTag(this HtmlHelper htmlHelper, HtmlTextWriterTag tag, object htmlAttributes, Func<HtmlHelper, string> action)
{
    var attributes = new RouteValueDictionary(htmlAttributes);

    using (var sw = new StringWriter())
    {
        using (var htmlWriter = new HtmlTextWriter(sw))
        {
            // Add attributes
            foreach (var attribute in attributes)
            {
                htmlWriter.AddAttribute(attribute.Key, attribute.Value != null ? attribute.Value.ToString() : string.Empty);
            }

            htmlWriter.RenderBeginTag(tag);
            htmlWriter.Write(action.Invoke(htmlHelper));
            htmlWriter.RenderEndTag();
        }

        return sw.ToString();
    }

    return string.Empty;
}
```

An overloaded version which doesn’t accept `HtmlAttributes` as parameter

```cs
public static string HtmlTag(this HtmlHelper htmlHelper, HtmlTextWriterTag tag, Func<HtmlHelper, string> action)
{
    return HtmlTag(htmlHelper, tag, null, action);
}
```

Below are some examples of using these helpers.

```
<%= Html.HtmlTag(HtmlTextWriterTag.Div, action => action.ActionLink("Without attributes","About") ) %>

<%= Html.HtmlTag(HtmlTextWriterTag.Div, new { name = "wrapper", @class = "styleclass" }, action => action.ActionLink("With Attributes", "About")) %>

<%= Html.HtmlTag(HtmlTextWriterTag.Div, null, action => action.ActionLink("Null attributes", "About")) %>  

<%= Html.IfElse(this.Model.Exists, trueAction => trueAction.Encode("Sample"), falseAction => falseAction.TextBox("SampleName")) %>

<%--
Nesting
<div>
   <span>
       <a href="/Home/About">About</a>
   </span>
</div>
--%>
<%= Html.If(this.Model.Exists,
    div => div.HtmlTag(HtmlTextWriterTag.Div,
        span => span.HtmlTag(HtmlTextWriterTag.Span,
anchor => anchor.ActionLink("About", "About"))
)) %>
```

Hope this will help you.
