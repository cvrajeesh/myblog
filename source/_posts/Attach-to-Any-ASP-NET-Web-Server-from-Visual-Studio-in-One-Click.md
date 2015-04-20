title: "Attach to Any ASP.NET Web Server from Visual Studio in One Click"
date: 2011-11-30 11:55:45
tags:
---

This is an update to my previous blog post {% post_link Attach-to-Visual-Studio-Development-Server-with-One-Click "Attach to Visual Studio Development Server with One Click" %}.

The Visual Studio Macro from previous article doesn’t support IISExpress or IIS; it only supported the Visual Studio Development Server, more over it doesn’t detect latest Development Web Server *WebDev.WebServer40.exe*.

Now I have updated the Macro so that it will automatically detect the Web Server setting from the project properties and attach it accordingly.

Below is the code for the new Macro, I think it is self-explanatory

```vb
Public Sub AttachToWebServer()
    Dim attached As Boolean = False
    Dim project As EnvDTE.Project = GetStartupProject()

    If (project Is Nothing) Then
        MsgBox("Couldn't find a web project that can be attached")
        Return
    End If

    attached = AttachToProcess(project)

    If (Not attached) Then
        MsgBox("Couldn't attach to the process")
    End If
End Sub

Private Function GetStartupProject() As EnvDTE.Project
    Dim startUpProject As String = DTE.Solution.Properties.Item("StartupProject").Value

    For Each currentProject As EnvDTE.Project In DTE.Solution.Projects
        If currentProject.Name = startUpProject Then
            Return currentProject
        End If
    Next
    Return Nothing
End Function

Private Function AttachToProcess(ByVal project As EnvDTE.Project) As Boolean
    Dim serverProcessNamePattern As String
    ' Using either IIS express or IIS
    If project.Properties.Item("WebApplication.UseIIS").Value = "True" Then
        ' Using IIS Express
        If project.Properties.Item("WebApplication.DevelopmentServerCommandLine").Value.ToString().Length > 0 Then
            serverProcessNamePattern = ".*iisexpress.exe"
        Else ' Real IIS
            serverProcessNamePattern = ".*w3wp.exe"
        End If

    Else ' Assume development web server
        serverProcessNamePattern = ".*WebDev.WebServer\d+.EXE"
    End If

    Return AttachToWebServer(serverProcessNamePattern)

End Function

Private Function AttachToWebServer(ByVal serverProcessNamePattern As String) As Boolean

    Dim attached As Boolean = False

    For Each process In DTE.Debugger.LocalProcesses
        If (Regex.IsMatch(process.Name, serverProcessNamePattern)) Then
            process.Attach()
            attached = True
            Exit For
        End If
    Next

    Return attached
End Function
```

Read my previous article {% post_link Attach-to-Visual-Studio-Development-Server-with-One-Click "Attach to Visual Studio Development Server with One Click" %} to understand how we can add this Macro as command to the Visual Studio toolbar.
