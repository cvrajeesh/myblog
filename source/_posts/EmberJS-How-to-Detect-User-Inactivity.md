title: "EmberJS - How to Detect User Inactivity"
date: 2014-06-12 12:47:49
tags:
---

**Scenario:**

I am working on a web application that has a Single Page Application module which is built using [EmberJS] framework.

This EmberJS application is a page designer where user can drag and drop items to the design surface and change properties of these items. Finally when the page is ready, he can save the changes back to server.

**Problem:**

Whenever a user works on the page designer for long period of time without saving the changes in between - finally when he try to save all of his changes in one shot, server side session was already timed out.

**Solution:**

Implemented an API on the server side which can refresh the session if this API is invoked.

So on the [EmberJS] application side we had to calculate how long a user not active. Then we can check for this inactivity time periodically and decide whether to refresh session or not.

An easy way to do it in EmberJS is by hooking to the `mousemove`, `keydown` events of the `ApplicationView`. Details can be found in the EmberJS documentation http://emberjs.com/api/classes/Ember.View.html#toc_responding-to-browser-events

Here is code sample which does this

First I am creating the application object with an additional property for tracking the last active time.

```js
var App = Ember.Application.create({
    lastActiveTime: null,
    ready: function(){
        App.lastActiveTime = new Date();
    }
});
```

After that, hook the events in which you are interested in `ApplicationView`, which will update the last active time defined in the application object.

```js
App.ApplicationView = Ember.View.extend({
    templateName: 'application',

    mouseMove: function() {
        App.lastActiveTime = new Date();
    },

    touchStart: function() {
        App.lastActiveTime = new Date();
    },

    keyDown: function() {
        App.lastActiveTime = new Date();
    }
});
```
In our case, I am interested on `mouseMove`, `touchStart` (for touch devices) and `keyDown` events. When they are getting fired I will update the last active time with current time.

Now you can use the last active time property to find out how long the user is active.

> Demo: http://cvrajeesh.github.io/emberjs/inactivity.html

This demo uses [moment.js](http://momentjs.com/) for calculating time difference as well as for formatting the time.

[EmberJS]: http://emberjs.com/
