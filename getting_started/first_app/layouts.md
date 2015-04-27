# Applications, initializers and regions

Now that we've built a simple view, let's start constructing a slightly more
complex application by nesting our views. This will let us show and hide
different views based on user input, location in the application, or any other
criteria you could come up with.

## Creating an Application

Taking our initial outline from
[creating our first view](./firstview.md#project-outline), let's move things
around a little. First, create a new file called `app/layout.js` with the
following:

```js
var Marionette = require('backbone.marionette');

var HelloView = Marionette.LayoutView.extend({
  el: '#view-hook',
  template: require('./hello.html')
});

module.exports = HelloView;
```

Your `hello.html` can remain unchanged. You'll notice that this is just the text
from `app.js` moved into a separate file. We'll modify `app.js` to look like:

```js
require('backbone').$ = require('jquery');
var Marionette = require('backbone.marionette');

var HelloView = require('./layout');


var App = Marionette.Application.extend({
  onStart: function(options) {
    var hello = new HelloView():
    hello.render();
  }
});

var app = new App();
app.start({});
```

### What have we achieved?

When you open your `index.html` file in your browser, you'll see exactly the
same output as before. So what have we achieved here?

We started by wrapping our application start inside our application. Notice the
empty object literal passed to `app.start({})` - this is where we can inject
options into our application from the template. A couple of examples would be
initial data or URLs to download the initial data to render.

Our `onStart` method is an event handler that is called when `start()` gets
executed. We also have access to an `onBeforeStart` method which runs before
anything in `app.start()` is executed.

Notice how we called our file `layout` instead of `view`: this reflects the
purpose of our `LayoutView` - a central hook to render all of our application's
templates and data.

## Laying out our views

The key component of a `LayoutView` is that it lets us attach and render
different views and different types of views. Let's create a new view and render
it from our `LayoutView`. Create a file called `goodbye.js`:

```js
var Marionette = require('backbone.marionette');


var GoodbyeView = Marionette.LayoutView.extend({
  template: require('./goodbye.html')
});


module.exports = GoodbyeView;
```

You'll notice that we don't set `el` here - this will be created for us by the
`LayoutView` when we render it. Our next step is to create a template file
called `goodbye.html`:

```html
<p class="goodbye-message">Goodbye, world! I'll miss you!</p>
```

Let's go back to our `hello.html` template and tell it where we're going to want
to render the `goodbye.html` template:

```html
<p>Hello, world!</p>
<div id="goodbye-hook"></div>
```

Finally, we need to tell our `LayoutView` to actually render the `GoodbyeView`
when it is rendered itself. Open up `layout.js` and change `HelloView` to look
like:

```js
var GoodbyeView = require('./goodbye');


var HelloView = Marionette.LayoutView.extend({
  el: '#view-hook',
  template: require('./hello.html'),

  regions: {
    goodbye: '#goodbye-hook'
  },

  onRender: function() {
    var goodbyeView = new GoodbyeView();
    this.goodbye.show(goodbyeView);
  }
});
```

This might seem like a lot of code to just render an extra `<p>` - but we can
use this to render lots of different types of views in a hierarchy. For example,
we can embed lists built from collections of data, or bind different types of
views to different data models. We'll discover more about
[tying Backbone models to views](./models.md) shortly.

### Customizing views

When we create views inside regions, Marionette will create a new `<div>` tag
to place the template inside. If we want to customize it, we can do this by
setting attributes on the view itself. Let's take the `<p>` tags out of the
template and have the `GoodbyeView` render them. Our `goodbye.html` template is
now:

```html
Goodbye world, I'll miss you!
```

and our GoodbyeView is modified to look like:

```js
var GoodbyeView = Marionette.LayoutView.extend({
  tagName: 'p',
  className: 'goodbye-message',
  template: require('./goodbye.html')
});
```

Now when you build the app and inspect the HTML, you'll see one less `<div>` in
the rendered output and the `<p>` tag will have the class we set.

## Chaining Layouts

As you can see, we used another `LayoutView` for our `GoodbyeView`. Does that
mean we can set a `regions` hash on that? Yes, we can! Let's do it. This time,
we'll just modify the view inside the same `goodbye.js` file:

```js
var MiddleView = Marionette.LayoutView.extend({
  template: require('./hello.html')
});


var GoodbyeView = Marionette.LayoutView.extend({
  template: require('./goodbye.html'),

  regions: {
    middle: '.middle-hook'
  },

  onShow: function() {
    var middleView = new MiddleView():
    this.middle.show(middleView);
  }
})
```

To keep it simple, I've just re-referenced the initial `hello.html` template.

We've removed the `tagName` as we're going back to building a more complex view,
so our `goodbye.html` could look like:

```html
<div class="middle-hook"></div>
<p class="goodbye-message">Goodbye world, I'll miss you!</p>
```

Now when we build and refresh the page, we'll see the contents of the
`MiddleView` rendered above the `GoodbyeView`.

## Regions everywhere

You'll notice that sometimes we call `render()`, sometimes we call `show()` and
we set different event handlers. This is a little inconsistent, can't we just
use the region manager pattern everywhere? Actually, we can! Marionette lets us
create our own `RegionManager` classes for handling just this case.

Let's go back to our top-level application and use a region manager to handle
the initial rendering in `app.js`:

```js
var App = Marionette.Application.extend({
  onStart: function(options) {
    var regions = new Marionette.RegionManager({
      regions: {
        hello: '#view-hook'
      }
    });

    var hello = new HelloView();

    regions.get('hello').show(hello);
  }
});
```

We also need to modify `HelloView` so it no longer handles its own `el`:

```js
var HelloView = Marionette.LayoutView.extend({
  template: require('./hello.html'),

  regions: {
    goodbye: '#goodbye-hook'
  },

  onShow: function() {
    var goodbyeView = new GoodbyeView();
    this.goodbye.show(goodbyeView);
  }
});
```

This gives us two benefits:
  1. We consistently use `onShow` and only use `onRender` where we absolutely
    must.
  2. We can re-use `HelloView` in other views and attach it to different
    elements - we no longer rely on `#view-hook` being in the DOM.

### What's the difference between `onShow` and `onRender`?

`onShow` is only called when the view is attached to the region manager with
`show(view)`, whereas `onRender` is called every time the DOM has to be
re-rendered. This means `onRender` can get called a lot when you have an
attached model or collection that changes frequently. This will make your
application faster as it gets larger and more complex.


## What's next?

That should cover the basics for managing your views. You should understand the
principles of managing regions and why it's a great thing to do. You should
familiarize yourself with the concepts, try adding extra regions and keep
nesting your layouts until you're comfortable with creating layouts and nesting
them.

When you're ready, let's move onto
[tying Backbone models to views](./models.md).
