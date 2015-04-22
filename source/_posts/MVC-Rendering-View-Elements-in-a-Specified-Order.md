title: "MVC - Rendering View Elements in a Specified Order"
date: 2011-02-27 14:31:29
tags:
---

In my current project, I had to render elements in the view based on a setting provided by the model(basically it is a configurable thing). Few clients need view element to be rendered in a particular order and few others in a different way. What we did was, saved this elements order in a settings file which could be changed based on the clients. Then created an extension to render this based on the order.

This is what was I was trying to explain. for  Client 1 the *Login section* to be displayed first followed by *Password reminder section*

![Client 1 requirement](http://cdn.rajeeshcv.com/images/2011/02/image_thumb.png)

For Client 2 , these sections needs be ordered differently

![Client 2 requirement](http://cdn.rajeeshcv.com/images/2011/02/image_thumb1.png)

In order to achieve this, I came up with an `HtmlHelper` extension

```cs
/// <summary>
/// Renders the render items in the provided sequence order.
/// </summary>
/// <param name="htmlHelper">The HTML helper which is extended.</param>
/// <param name="sequenceOrder">The order in which items to be rendered. Sequence starts at an index of 0.</param>
/// <param name="renderItems">The items to be rendered in order.</param>
/// <remarks>
/// Values in the sequence order should match with the total number of render items.
/// Invalid sequnce numbers are ignored.
/// </remarks>
public static void OrderBy(this HtmlHelper htmlHelper, int[] sequenceOrder, params Action<HtmlHelper>[] renderItems)
{
    if (sequenceOrder != null && renderItems != null)
    {
        foreach (var sequnce in sequenceOrder)
        {
            // CHeck whether the sequence is with inthe bounds
            if (sequnce < renderItems.Length && sequnce >= 0)
            {
                renderItems[sequnce].Invoke(htmlHelper);
            }
        }
    }
    else if (renderItems != null)
    {
        // If the sequence order is not provided, render it in normal order in which items are declared.
        foreach (var renderItem in renderItems)
        {
            renderItem.Invoke(htmlHelper);
        }
    }
    else
    {
        // Do Nothing
    }
}
```

In the view, you could do

```
<% Html.OrderBy(this.Model.LoginDisplayOrder, (html) => { %>
    <div class="container"></div>
    <% Html.RenderPartial("LoginSection", this.Model); %>
<% }, (html) => { %>
    <div class="container"></div>
    <% Html.RenderPartial("ReminderPassword", this.Model); %>
<% }); %>
```

Here `Model.LoginDisplayOrder` is just an array of integers in which the items to be rendered.

Hope this will help.
