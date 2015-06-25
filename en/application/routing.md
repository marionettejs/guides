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


### Examples

We'll just take a few minutes to look at potential example pages that a server
could generate for given URLs. Assuming the rest of the page remains the same,
we'll just be looking at the `<script>` tag that generates the `initialData`
key.

#### /

```js
window.initialData = {
  guides: [
    {
      name: 'Marionette',
      url: '/guides/marionette'
    },
    {
      name: 'Ember',
      url: '/guides/ember'
    }
  ],
  authors: [
    {
      name: 'Scott Walton',
      email: 'scott@example.com'
    },
    {
      name: 'Joanne Daudier',
      email: 'joanne@example.com'
    }
  ]
}
```

The Router will trigger the handler bound to the `''` route and inject two lists
of guides and authors into the application to be rendered (or not).

#### /guides/marionette

```js
window.initialData = [
  {
    name: 'The Marionette Guide',
    authors: [
      {
        name: 'Scott Walton',
        email: 'scott@example.com'
      },
      {
        name: 'Joanne Daudier',
        email: 'joanne@example.com'
      }
    ],
    published: '2015'
  },
  {
    name: 'Marionette: A Gentle Introduction',
    authors: [
      {
        name: 'David Sulc',
        email: 'david@example.com'
      }
    ]
  }
];
```

This data will trigger the handler bound to the `'guides/:guide'` route and
inject a list of Marionette guides and their authors to be rendered. This is
obviously useful when we have a short list that's easy for the server to include
with the page being loaded. When the application has loaded, we'll only need to
request the data as and when we need it. On the initial application load, no
matter what page we start from, we'll always get some data that we can render
immediately.

This pattern lets us make the best use of HTML5's Push State to get the best of
both worlds. Even better, this pattern only requires us to change the template
that our server generates - our JavaScript files are safely cached and we don't
need to keep duplicating the same basic functionality!


[readme]: ./README.md "Application"
[router]: ../approuter/README.md "AppRouter"
