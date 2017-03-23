title: "Introducing SourceCodeReader"
date: 2012-08-02 12:21:32
tags:
- Projects
---

Have you ever tried to understand a project just by going through the source code?

That’s how I do most of the time, till now I haven’t seen any up to date documentation which gives the clear understanding of how the code is implemented. In reality after a certain period during the development, documentation goes out of sync with the source code. So some one joins the project lately has only one way to understand the project *Go through the source code*

Sometimes I go through the open source projects hosted on either Github, Codeplex or Google code. All the hosting site provide you with a user interface for browsing through the source code with syntax highlighting support, which is nice. The problem I had with this is – if I see a object creation statement, method call there is no way for me to find out how it is implemented other than manually going through the files and finding it out.

Most of the full blown editors provide a feature where you can navigate to the implementation directly from where it is used for e.g. Microsoft Visual Studio provides *Go To Definition* feature. I find this feature very much useful for understanding the project. Only draw back with this approach is you have to completely get the latest version of source code to your local machine.

[SourceCodeReader][1] is trying solve this navigation problem on the web. With this application you can open a project and browse through the files with code navigation support.

Source code for this project can be found at https://github.com/cvrajeesh/SourceCodeReader

### Limitations

1. Now supports only Github code repository
2. Only supports .Net C#  projects, other type of projects also works without code navigation support


###How to use this application

Go to [SourceCodeReader][1] and enter the URL of a C# project on Github

![SourceCodeReader home page](http://cdn.rajeeshcv.com/images/2012/08/20120802065851_image_thumb.png)

Once you have entered a Github project link and open the project, it get the source code from the Github and present you with file browser.

![SourceCodeReader project directory](http://cdn.rajeeshcv.com/images/2012/08/20120802065857_image_thumb_2.png)

When you navigate to a C# file you will be able to see clickable links for identifiers which takes you the file location where that is declared.

![SourceCodeReader Go To Definition feature](http://cdn.rajeeshcv.com/images/2012/08/20120802065903_image_thumb_1.png)

**Sample projects :**

* http://sourcecodereader.apphb.com/#/open/cvrajeesh/SourceCodeReader
* http://sourcecodereader.apphb.com/#/open/ayende/ravendb

Hope you liked this idea, please provide me with your valuable feedback.

[1]: http://sourcecodereader.apphb.com/
