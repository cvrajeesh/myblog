title: "How Web Servers Identify Your Session?"
date: 2008-06-12 23:47:43
tags:
---

I think, you have all created ```session``` variables inside an asp.net application. Then how the web server knows you are the same guy created that session when successive request goes to the server.

Before explain this, we have to think that every request that we sent from a web browser is considered as a new request by the IIS. This is the behavior of every web application i.e. the state less nature. For example if you open a url, first the browser sent a request for the default document. Then the browser parses the HTML response and it sends another set of requests based on the required images, scripts that are included in that page.

Below you could see, the different requests that sent by my browser when I opened my blog

![Fiddler Trace](http://cdn.rajeeshcv.com/images/2008/06/image-4.jpg)

Some think **State less** nature is the biggest drawback of a web application, but in my view this is an advantage because you could distribute your application as you like, because it is not tightly coupled to a particular request.

Developing **State aware** application is one of the main challenge involved in any web development. For developing a **State aware** application, you don’t need to do much. All most all server side scripting languages supports creation of ```Session``` variables. Session variables are created in the memory location where your web server runs and it is created per user basis. That means, *User1* creates a session variable but *User2* can’t modify that particular session variable, he can only work with the session variables he created. Ok thats about session, but how your server identify this users?

I will explain that with an example, for that I have created a simple website with one page and I have placed two buttons in that page. When user click on the *Button1*, a session variable is set. If the user click on the *Buttton2*, the value from session variable is written back on the page using ```Response.Write(..)``` method.

![Test Page](http://cdn.rajeeshcv.com/images/2008/06/image-5.jpg)

When I first hit this url, the HTTP headers of request and response looks like below

![First request headers](http://cdn.rajeeshcv.com/images/2008/06/image-6.jpg)

![First response header](http://cdn.rajeeshcv.com/images/2008/06/image-8.jpg)

This request simply means, we are looking for a *Default.aspx* page and the web server is returning that page. Ok. Now we click on *Button 1*, at that time if you look at the HTTP request header

![Button1 click request headers](http://cdn.rajeeshcv.com/images/2008/06/image-7.jpg)

From that header you could understand that, it is a simple form post header. As I said earlier, we are setting a session variable. Because of that, your server will create id for identifying your session and that value is send back as a cookie to browser through response header. Below is the response header of *Button 1* click

![Button1 click response headers](http://cdn.rajeeshcv.com/images/2008/06/image-9.jpg)

In that response, you could see a ```Set-Cookie``` header with cookie name **AS.NET_SessionId** with some value is sent back to the browser. From now onwards, each an every request send from that page will have this cookie value. To show that, we click on the *Button 1* again, see below is the request header for that

![Button1 click second time request headers](http://cdn.rajeeshcv.com/images/2008/06/image-10.jpg)

There your browser passes the **AS.NET_SessionId** cookie value to the server through each and every request. With this cookie value only your server identify the sessions, Hope you have got some idea about this.

> Here I have used **Fiddler** as the tool for intercepting these request/response.
> If you need this tool please goto http://www.telerik.com/fiddler and download it.
