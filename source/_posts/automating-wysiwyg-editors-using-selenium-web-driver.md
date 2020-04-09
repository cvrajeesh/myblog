title: "Automating WYSIWYG Editors using Selenium Web Driver"
date: 2012-12-01 12:28:23
tags:

- UI Testing
- QA

---

Automating web application using [selenium for acceptance/integration tests][1] is very easy. Recently I have faced few issues when automating a page with a JavaScript based [WYSIWYG] editors.

The reason why it is hard because most of these editors create a new editable HTML document inside an iframe. There are two ways I aware that you can set text on these editors

1. Executing a JavaScript code in the current page
2. Sending keys on the editable iframe

Here is how I automated two [WYSIWYG] editors using selenium web driver.

### bootstrap-wysihtml5

Website: http://jhollingworth.github.io/bootstrap-wysihtml5/

```cs
private void SetBootstrapWysihtml5Content(IWebElement textArea, string content)
{
    string script = string.Format(@"$('#{0}').data('wysihtml5').editor.setValue('{1}');", textArea.GetAttribute("id"), content);
    ((IJavaScriptExecutor)this.WebDriver).ExecuteScript(script);
}
```

What this method does is, it accepts the `TextArea` web element which you are converting to an editor and the actual content to be set. Then it executes a piece of JavaScript on the current `WebDriver` context.

For e.g. if you have a TextArea element with id `summary` then executing below JavaScript statement will insert a `h1` tag with "Test" to the editor

`$('#summary').data('wysihtml5').editor.setValue('<h1>Test</h1>');`

### TinyMCE

Website: http://www.tinymce.com/

```cs
private void SetTinyMCEContent(IWebElement textArea, string content)
{
    string textAreaId = textArea.GetAttribute("id");
    string frameId = string.Format("{0}_ifr", textAreaId);
    var frame = this.WebDriver.FindElement(By.Id(frameId));
    this.WebDriver.SwitchTo().Frame(frame);
    ((IJavaScriptExecutor)this.WebDriver).ExecuteScript("document.body.innerHTML = '" + content + "'");
    this.WebDriver.SwitchTo().DefaultContent();
}
```

This method also accepts the `TextArea` web element which you are converting to an editor and the actual content to be set. Then it finds the corresponding `iframe` and set the `innerHTML` by executing a JavaScript statement.

[1]: https://github.com/SeleniumHQ/selenium/wiki/Getting-Started
[wysiwyg]: http://en.wikipedia.org/wiki/WYSIWYG
