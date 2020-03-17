title: "Visual Studio : Debugging Silverlight Application with Chrome and Firefox"
date: 2013-08-03 12:43:01
tags:
- Silverlight
---

Here is a tip if you want to debug Silverlight applications from Visual Studio 2012 if it is running in Google Chrome or Firefox

**Google Chrome :**

1. Run the application and select Debug –> Attach to Process from the menu
2. From the Available Processes list, look for process named “chrome.exe” with type *Silverlight*
3. Select that process and click the *Attach* button

Here is screenshot from my machine
![Visual Studio attach to Silverlight in Chrome](/images/2013/08/201308301039_chrome_silverlight_debugging.png)

**Mozilla Firefox :**

Firefox has a slightly different approach

1. Run the application and select Debug –> Attach to Process from the menu
2. From the Available Processes list, look for process named *plugin-container.exe*
3. Select that process and click the *Attach* button

Here is the screenshot
![Visual Studio attach to Silverlight in Firefox](/images/2013/08/201308301039_firefox_silverlight_debugging.png)

Voila!!! now you are able to get the breakpoints in Visual Studio
