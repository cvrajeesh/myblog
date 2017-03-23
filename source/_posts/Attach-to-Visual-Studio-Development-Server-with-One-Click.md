title: "Attach to Visual Studio Development Server with One Click"
date: 2011-11-26 11:44:24
tags:
- Visual Studio
---

In my day to day work, during the development I had to attach an ASP.NET application to the development server (Cassini) several times in order to debug and fix a problem.

This task is little bit time consuming because this is how we normally do it

1. Click on the “Attach to Process” menu under the Debug menu
2. Select the correct process from the list of available processes
3. Either double click on the select process or click the “Attach” button

You can reduce these into two steps, if you assign a short cut key to the *Attach to Process* command.

What I found is, most of the time is lost for finding and selecting the correct process from the available list of processes in the *Attach to Process* dialog.

So in order to be more productive, I have created a macro which will automatically attach to the Visual Studio Development Server automatically, saves your time from scroll through the big list of processes.

Below is the code for this macro

```vb
Public Sub AttachToDevelopmentServer()
    Dim attached As Boolean = False
    Dim process As EnvDTE.Process
    For Each process In DTE.Debugger.LocalProcesses
        If (process.Name.EndsWith("WebDev.WebServer20.EXE")) Then
            process.Attach()
            attached = True
            Exit For
        End If
    Next

    If (Not attached) Then
        MsgBox("Couldn't attach to the process")
    End If

End Sub
 ```

If you don’t know to how to create a macro, these are the steps for doing it

1. In Visual Studio Go To the Tools menu
2. Then select the Macros and then Macros IDE (Short cut is Alt+F11). This will open up a new IDE similar to Visual Studio.
3. In the project explorer under *My Macros* you will see *My Module* or you can create a new module by right clicking on the project, then Add -> Add Module. In my case I created a new module and called it as *Extensions*
4. Copy and paste above subroutine to that module.

Now we have to execute the newly created macros on clicking on a toolbar button. For that, follow these steps

1. In Visual Studio, go to the Tool menu, under that select the Customize menu. This will open up the Customize dialog
2. Select the Commands tab
3. Select the Toolbar checkbox and then select the Toolbar on which you want this button to appear from dropdown list. I chose the *Standard Toolbar*
4. Now click on the “Add Command” button. Then select the Macros from the categories in the *Add Command* dialog.
5. Now you will see the list of available macros on the list box on. From that list select your Macro *AttachToDevelopmentServer* and click OK
6. If you want, you can customize the text which appears on the command button, by clicking on the *Modify Selection*
Here it is how it looked in my machine

![Adding Command to the Visual Studio Toolbar](http://cdn.rajeeshcv.com/images/2011/11/20111126012529_AddCommand_thumb.png)

That’s it, now you will be able to new button appeared in the standard toolbar. If you click on that, it will automatically attach your web project to the running application in the development server.

![Visual Studio Toolbar after Customization](http://cdn.rajeeshcv.com/images/2011/11/20111126012540_AttachToProcess_thumb.png)

**Note:** There are couples of problems here

1. This doesn’t work if you have set the IIS Express/IIS itself as the server for running the application.( *Update:* A new macro to support this feature is exlplained here {% post_link Attach-to-Any-ASP-NET-Web-Server-from-Visual-Studio-in-One-Click "Attach to Any ASP.NET Web Server from Visual Studio in One Click" %} )
2. If you have multiple web applications in your project, it will attach to the first development server. So I suggest you to read my previous post about {% post_link Multiple-Visual-Studio-Development-Servers-while-Debugging "disabling multiple Visual Studio development webservers" %}
