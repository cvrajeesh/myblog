title: "Asp.Net MVC - Fluent Html Helper for FlexiGrid"
date: 2010-04-18 12:35:04
tags:
---

There are so many free JQuery Grid plugins out there, in that I liked [FlexiGrid](https://github.com/paulopmx/Flexigrid) just because of it’s look and style. In order to use it in your MVC application you may have to put the Javascript code into your view, which requires the property names of your model in order to generates the Grid columns as well the search options etc… as everybody knows when you deal with hard coded string as the property names in any code, it is error prone.

In order to avoid this problem I thought of creating a html extension which is tightly coupled with your data that is going to bound to the Grid. Which helps the developer from writing any javascript codes.

> Full source code can found in this Git repo  [mvc-fluent-jquery-plugin-controls](https://github.com/cvrajeesh/mvc-fluent-jquery-plugin-controls)
