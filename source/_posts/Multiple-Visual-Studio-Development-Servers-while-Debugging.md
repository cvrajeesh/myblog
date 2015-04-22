title: "Multiple Visual Studio Development Servers while Debugging"
date: 2011-11-24 15:33:44
tags:
---

You might have noticed that when you start debugging an ASP.NET web application, it start more than one visual studio development servers and in the system tray you see something like this

![Multiple Development Servers](http://cdn.rajeeshcv.com/images/2011/11/20111124063108_Mutiple_development_server_2.png)

This happens when your solution contains more than one web application, setting one as the StartUp project is not going to help..

The reason for this is - by default any web application is set to start when we trigger the debugging process in visual studio. We need to disable that auto start feature so that only one visual studio development server is getting started when we are debugging.

These are the steps for disabling it

1. Select the web project which you don’t want to start
2. Go to the properties window (Shortcut – press F4)
3. Set “Always Start When Debugging” to False

![Disable always start](http://cdn.rajeeshcv.com/images/2011/11/20111124063119_Disable_Start_When_Debuggin_thumb.png)

This is how you could do it in Visual studio 2010, I hope the same applies to Visual studio 2008 too. Now if you start debugging, you will see only one development webserver.

Hope this helps
