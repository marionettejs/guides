# Attaching controllers

There are a number of ways to attach a `controller` to your `AppRouter`,
depending on how you use it in your application. This chapter covers the
different strategies and why you might want to use each one.


#### Simple objects

The easiest way to attach a controller is to simple attach an object to the
`controller` key when extending `AppRouter` as such:

```js
var Router = Marionette.AppRouter.extend({
  appRoutes: {
    'blog/': 'blogList',
    'blog/:entry': 'blogEntry',
    'blog/:entry/comments/:comment': 'blogComment'
  },

  controller: {
    blogList: function() {
      // ...
    },
    blogEntry: function(entry) {
      // ...
    },
    blogComment: function(entry, comment) {
      // ...
    }
  }
});

module.exports = Router;
```

This method works for simpler controllers that don't have much internal state.
We just treat this like any other JavaScript object.

#### Marionette Object class

If we want a more complex controller, we can use `Marionette.Object` - a custom
object that contains many of the event handling and initializer logic shared
with other Marionette components:

```js
var Controller = Marionette.Object.extend({
  blogList: function() {
    // ...
  },
  blogEntry: function(entry) {
    // ...
  },
  blogComment: function(entry, comment) {
    // ...
  }
});


var Router = Marionette.AppRouter.extend({
  appRoutes: {
    'blog/': 'blogList',
    'blog/:entry': 'blogEntry',
    'blog/:entry/comments/:comment': 'blogComment'
  },

  controller: new Controller()
});

module.exports = Router;
```

We can now have the `Object` listen to events that occur in the application -
for example, we might handle major layout changes like route-changing logic in
our controller by listening to events on views. This will reduce the amount of
code needed to change route inside the application versus setting up the views
on page load.

#### During initialization

We can also attach a controller during `initialize` to pass options through the
router:

```js
var Controller = Marionette.Object.extend({
  blogList: function() {
    // ...
  },
  blogEntry: function(entry) {
    // ...
  },
  blogComment: function(entry, comment) {
    // ...
  }
});


var Router = Marionette.AppRouter.extend({
  appRoutes: {
    'blog/': 'blogList',
    'blog/:entry': 'blogEntry',
    'blog/:entry/comments/:comment': 'blogComment'
  },

  initialize: function() {
    this.controller = new Controller(this.options);
  }
});

module.exports = Router;
```

This pattern lets us initialize a router inside our application and pass any
options directly through to the controller without having to expose it to the
rest of the application.
