title: "Inline Editing Using JQuery"
date: 2009-06-25 10:22:25
tags:
- WebDev
---

Here I wanted to show you the flexibility of JQuery. Below is a HTML code that generates a grid with two rows and ten columns.

```html
<div id="container">
    <div class="column">1</div>
    <div class="column">2</div>
    <div class="column">3</div>
    <div class="column">4</div>
    <div class="column">5</div>
    <div class="column">6</div>
    <div class="column">7</div>
    <div class="column">8</div>
    <div class="column">9</div>
    <div class="column">10</div>
</div>
```

When the user double clicks on any of the cell, he can edit that content. The cell value get updated when user double clicks on any other cell.

```js
$(document).ready(function() {
    $("#container .column").dblclick(function() {

        var content = $(this).text();
        var width = $(this).width() - 1;
        var height = $(this).height() - 4;

        var $editbox = $("<input type='text'" +
            "style='width:" + width + ";" +
            "height:" + height + ";" +
            "border:none" +
            "' value='" + content + "' />");

        $(this).empty();
        $(this).prepend($editbox);
        $editbox.focus();
        $editbox.select();

        $($editbox).bind("blur", function() {

            content = $(this).val();
            var parent = $(this).parent();
            parent.html(content);

        });

    });
});
```

The above code does all the trick. First we hook the `Double Click` event to all the DIV with class name `column` using this `$("#container .column").dblclick(function(){...});` Inside that, we dynamically add a textbox inside the DIV and set its text value as the HTML content of the parent DIV. And also we bind the `blur` event of this texbox to a function, so that whenever the focus leaves from the textbox, that function is called. Then inside that function, we set the HTML content of the DIV with the Textbox value.

```js
$($editbox).bind("blur", function() {
     content = $(this).val();
     var parent = $(this).parent();
     parent.html(content);

 });
```

See the working demo below

{% jsfiddle s7jz6qbn %}
