title: "ASP.NET AJAX - Calling Server Side methods from Client Side"
date: 2008-10-24 00:53:13
tags:
- ASP.NET
---

> {% post_link ASP-NET-AJAX-Passing-JSON-Object-from-Client-to-Server "Part2"%} - ASP.NET AJAX - Passing JSON Object from Client to Server

ASP.net AJAX has got nice bridging between your server side and client side technologies. Using the ASP.net AJAX you can easily call a method in a page. The restriction here is that, the methods your are trying to call should be static and public.

This feature(calling server side static method) is not enabled by default, so you have to manually enable this and it is very very simple; just set the `EnablePageMethods` property of the `ScriptManager` to true like this

```
<asp:scriptmanager id="ScriptManager1" runat="server" enablepagemethods="true"></asp:scriptmanager>
```

After this, create a public static method in your asp.net page and decorate it with `WebMethod`(System.Web.Services.dll) and `ScriptMethod`(System.Web.Extensions.dll) attributes. For example I have created a static method called `GetTime()` which returns current time as string.

```cs
public partial class _Default : System.Web.UI.Page
{
    protected void Page_Load(object sender, EventArgs e)
    {
    }

    [WebMethod]
    [ScriptMethod]
    public static string GetTime()
    {
        return DateTime.Now.ToLongTimeString();
    }
}
```

Now you have configured everything correctly and I will show you how to call this method from the client side.

Earlier you have set the `EnablePageMethods` property of `ScriptManager` to true, as a result of that a new object called `PageMethods` is created by the `ScriptManager` when the page is rendered. This object has all the public static methods in that page. So that you call the server side method like **PageMethods.[ServerSideMethodName]**. When calling this methods from client side you have to provide a callback function because here all the call `scriptmanager` made will be asynchronous, and this callback method will get the returned result.

Below is the sample code

```
<form id="form1" runat="server">
  <asp:ScriptManager ID="ScriptManager1" runat="server" EnablePageMethods="true">
  </asp:ScriptManager>
  <div>
    <a href="javascript:callServerMethod()" >Call Server method</a>
  </div>
  <script language="javascript" type="text/javascript">
  //<![CDATA[function callServerMethod() {
            PageMethods.GetTime(callBackMethod);
          }
          function callBackMethod(result) {
            alert(result);
          }
  //]]>
  </script>
</form>
```

###Extra bits
In the above code you can see that I have placed link, when user click on that link a message is shown with the server time. So behind an asynchronous request is send to the server in this format *http://[sitename]/[MethodName]*. Server in turn returns the result as JSON back to the client. Here is a screen shot from the firebug

![Firebug headers](/images/2008/10/image.png)

![Firebug response](/images/2008/10/image1.png)

There is an optimization we have do here - you can see that, by default the request is a POST request, but in our case a POST request is heavy because we could have achieved the same using a GET request ([Difference between POST and GET](http://www.cs.tut.fi/~jkorpela/forms/methods.html)).

In order to change this request in to a GET request, we need to do a minor tweaking in the server side, set `UseHttpGet=true` parameter for the `ScriptMethod` attribute.

```cs
[WebMethod]
[ScriptMethod(UseHttpGet=true)]
public static string GetTime()
{
    return DateTime.Now.ToLongTimeString();
}
```

After this you can see that, request is changed to a GET request

![Firebug response after changing to GET](/images/2008/10/image2.png)
