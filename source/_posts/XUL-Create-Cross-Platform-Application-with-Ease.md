title: "XUL - Create Cross Platform Application with Ease"
date: 2009-05-19 01:39:43
tags:
---

You want to create simple GUI cross platform application without knowing the high level programming languages like C++, Java, C#~(using mono)? XUL is that answer.

Yes it is possible using XUL, the prerequisites  for this is, knowledge of a declarative programming language(like HTML or XHTML, XAML etc...) and JavaScript.

XUL is basic building block of the Firefox UI and as we know Firefox works in Windows, Linux and MAC.

Actually XUL is a language which run on the XULRunner(Gecko) runtime from Mozilla.

If you really wants see how Firefox created this UI, go the FireFox installation folder. Normally it will be `C:\Program Files\Mozilla Firefox\`. Under that installation root folder, find a folder named "chrome". Firefox has packaged the xul files in a zip format and they have renamed it as `.jar`. So look for a file `browser.jar` and rename it to zip. Extract it and have a look at `content\browser\browser.xul`. This file holds the UI definition for Firefox.

To learn more about XUL, refer these web sites

1. http://www.mozilla.org/projects/xul/
2. http://www.ibm.com/developerworks/web/library/wa-xul1/
