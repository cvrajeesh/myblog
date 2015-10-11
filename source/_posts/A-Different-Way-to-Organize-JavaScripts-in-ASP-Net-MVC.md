title: A Different Way to Organize JavaScripts in ASP.Net MVC
date: 2015-10-07 20:57:54
tags:
---

New JavaScript SPA(Single Page Application) frameworks are getting written day by day, generally one core feature these frameworks solves is how to organizing the code so that maintaining it in the future won't be a night mare. When it comes to organizing JavaScripts in a large scale server side application, resources available is very less SPA frameworks.

Organizing JavaScripts in a server side application is entirely different from SPA, most of the articles that I came across is suggesting to include page level script inside the server side page itself - e.g. in ASP.Net MVC you include common JavaScript files inside the layout file and views will include the scripts it required

**Layout(_Layout.cshtml)**
```html
<!DOCTYPE html>
<html>
<head>
  .....
</head>
<body>
    ......
    @Scripts.Render("~/bundles/commonjs")
    @RenderSection("scripts", required: false)
</body>
</html>

```

**View(Index.cshml)**
```html
.....
@section Scripts {
    <script src="~/....."></script>
}

```

For large applications a page will have different components which requires it's own set of JavaScript files in order work. Including and organizing these component level scripts and it's dependencies can cause lot of issues.

So here I want explain a different approach which I am not going to claim is best optimal solution for all applications but I think it does work for majority of the applications.

In this article I am using [TypeScript] instead of JavaScript. If you don't know about [TypeScript] it's a typed superset of JavaScript, learn more about it from  [http://www.typescriptlang.org][TypeScript]

##Application Class
Application is a TypeScript class which is the main entry point to client side script, this class is responsible for initializing page component. This is done by reading `data-js-component` attribute on body, then import the component dynamically and initialize.


```typescript
import { Component } from "Component"
import * as Utils from "Utils"

export class Application {

    public initialize(): void {

        // Get page component name
        var componentName = document.body.dataset["jsComponent"];
        if (!componentName) {
            // No page level component found
            return;
        }

        System.import("Pages/" + componentName)
            .then((mod) => {
                if (typeof mod[componentName] === "function") {
                    var pageComponent = Utils.createInstance<Component>(mod[componentName], document.body);
                    pageComponent.initialize();
                }
            });
    }
}
```
In order to load the component dynamically I am using a module loader called [SystemJS], line 15 shows dynamically importing a page component. Once the component is imported it's `initialize()` method is called after creating an instance of that component. One of the main advantage of using a module loader here is it will manage the dependencies which means any dependent modules will be imported automatically.

Every page will have a corresponding page component, name of this page level component is set as a `data-js-component` attribute on the body tag. Page component name is generated based on a convention - it will be same as the controller name. e.g. component name for `CustomerController` will be `Customer`. All page level components resides in a separate directory called *Pages*.

I have created a Html helper extension method which will return controller name, it also provide flexibility where an action method could change the page component name using a view data `jsComponent`.

```
<body data-js-component="@Html.JsPage()">
    ......
</body>
```
Helper extension
```csharp
public static MvcHtmlString JsPage(this HtmlHelper helper)
{
    var pageNameViewData = helper.ViewContext.ViewData["JsComponent"];
    var pageName = pageNameViewData != null
                       ? pageNameViewData.ToString()
                       : helper.ViewContext.RouteData.GetRequiredString("controller");

    return new MvcHtmlString(pageName);
}
```

##Component
All page level components derive from a base class `Component`, this base class implements the core initialization logic. Component can have child components as well, so initializing parent component will initialize child components as well. When `Application` class initializes page component, then any child components will also get initialized. `OnReady` method on the component is called after initialization is completed.

`Component` is associated with an HTML element on which the component is going to act up on. This container element is passed as a constructor argument. Page level component is initialized with body as associated element.

**Component Class**
```typescript
export class Component {

    ....

    constructor(private element: HTMLElement) {
    }

    public get container() {
        return this.element;
    }

    public get componentName() {
        return this.container.dataset["jsComponent"];
    }

    public initialize(): Promise<any> {
        ......
    }

    public onReady() {
    }

    ....    
}
```
**Home page component**
```typescript
import { Component } from "Component";

export class Home extends Component {

}
```

##Child Components
A page will have many child components as well. To create these child component element associated with that component should have a `data-js-component` attribute.

Below code shows `Popover` component which initializes the bootstrap popover 

```typescript
import { Component } from "Component"

export class Popover extends Component {

    public onReady() {
        $(this.container).popover({
            content: "Popover Component",
            placement: "right"
        });
    }
}
```
In order apply this in the razor view, I will be setting `data-js-component` attribute like below

```html
<a  href="#" 
    class="btn btn-primary btn-lg js-component" 
    data-js-component="Popover">
    Learn more &raquo;
</a>
```

##Connecting the dots
Finally to wire up everything - `Application` class needs to initialized. For that import `Application` module in **_Layout.cshtml** and initialize it.

```html
<script src="~/Scripts/system.js"></script>
<script>
    System.config({
        baseURL: '/Scripts/App',
        defaultJSExtensions: true
    });

    System.import('Application')
        .then(function (m) {
            var app = new m.Application();
            app.initialize();
        });
</script>
```

Source code used in this article can be found here https://github.com/cvrajeesh/myblogsamples/tree/master/MVC/PageScripts

[TypeScript]: http://www.typescriptlang.org/
[SystemJS]: https://github.com/systemjs/systemjs
