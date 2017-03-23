title: "EmberJS Error Logging in Production"
date: 2014-06-22 12:52:57
tags:
- EmberJS
- SPA
---

In order to log all errors in an [EmberJS] application, [EmberJS] provides a global hook called `onerror`

Here is a link to the official documentation - http://emberjs.com/guides/understanding-ember/debugging/#toc_implement-an-ember-onerror-hook-to-log-all-errors-in-production

Code sample from the documentation (v 1.5.1) look like this

```js
Ember.onerror = function(error) {
  Em.$.ajax('/error-notification', {
    type: 'POST',
    data: {
      stack: error.stack,
      otherInformation: 'exception message'
    }
  });
}
```

What this piece of code does is, whenever an error occurs that stack trace is send to the server using an AJAX request. This will works great for most of the scenarios but....

Consider a situation where an error occurs inside a mouse move event handler. Then you will be trying to sending hundreds of requests to server in short interval. This can bring down your server if not handled properly, it's like [DDOS](http://en.wikipedia.org/wiki/Denial-of-service_attack)-ing your own server.

Good approach will be to control the rate at which errors are send to the server for logging. This can be done using a concept called Throttling - which ensures that the target method is never called more frequently than the specified period of time.

Google for JavaScript Throttling you can find lot different implementations. EmberJS also provide it's own throttling function Ember.run.throttle

Here is the example from the official [EmberJS] website

```js
var myFunc = function() { console.log(this.name + ' ran.'); };
var myContext = {name: 'throttle'};

Ember.run.throttle(myContext, myFunc, 150);
// myFunc is invoked with context myContext
```

This `throttling` function can be used for sending errors to the server. Here is a working demo which shows the throttled error logging -
http://cvrajeesh.github.io/emberjs/errorhandling.html

[EmberJS]: http://emberjs.com/
