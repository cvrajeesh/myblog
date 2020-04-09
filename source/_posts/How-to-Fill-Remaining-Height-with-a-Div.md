title: "How to Fill Remaining Height with a Div"
date: 2015-05-08 12:26:23
tags:

- WebDev

---

Sometimes things are too obvious but don't work the way we think.

This is the HTML I have, my requirement is `child1` height will be auto(based on the content) and `child2` will occupy the remaining height of the main-container.

```html
<div class="main-container">
  <div class="child1">Child1 contents</div>
  <div class="child2">Child2 contents</div>
</div>
```

First I thought this is easy and created the following styles

```css
.main-container {
  height: 500px;
  width: 100%;
}
.child2 {
  height: 100%;
}
```

This produces a wrong result - if `child1` takes 100px and I expected `child2` to be 400px, but the `child2` height is 500px.

![Child2 takes full height](/images/2015/Fill_height_div.PNG)

Let's see how to solve this problem.

### Using Flexbox

Very easy to solve this using **flexbox** (`display:flex`), most of the latest browsers supports it. Make `container` a Flexbox with the direction set to `column`, then allow `child2` to grow using `flex-grow:1`. This ensure second child height is automatically adjusted.

```css
.container {
  height: 500px;
  width: 100%;
  display: flex;
  flex-direction: column;
}
.container .child2 {
  flex-grow: 1;
}
```

Here is the result using flexbox

{% jsfiddle utdwL1mb html,css,result %}

### Using table-row

Another option is to use `display:table-row`. Below code shows how to use it.

```css
.main-container {
  height: 500px;
  width: 100%;
  display: table;
}
.child1 {
  display: table-row;
}
.child2 {
  display: table-row;
  height: 100%;
}
```

see this in action below

{% jsfiddle j2Lpehvf html,css,result %}
