# Integrating with Routing

One of the biggest uses of the `Application` is to start your [router][router],
which in turn starts rendering your initial views. This is normally quite
straightforward from our `driver.js` file:

```js
var Marionette = require('backbone.marionette');

var Router = require('./router');


var app = new Marionette.Application({
  onStart: function(options) {
    var router = new Router(options);
  }
});

app.start();
```

This is an ideal way to pass page data into router, as well as the rest of our
application.


## Loading Initial Data for Push State

We can use the Router in combination with the [initial data pattern][readme] to
allow our server to respond to HTML5 Push State URLs by sending initial data to
the application to be rendered from the Router. Simply modify our driver file
like so:

```js
var app = new Marionette.Application({
  onStart: function(options) {
    var router = new Router({
      pushState: true,
      initialData: options.initialData
    });
  }
});

app.start({initialData: window.initialData});
```

Now, when our Router responds to the URL from HTML Push State, we'll also send
in any data specific to that page to be immediately rendered.


[readme]: ./README.md "Application"
[router]: ../approuter/README.md "AppRouter"
