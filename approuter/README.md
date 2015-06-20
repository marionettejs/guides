# Implementing Routing

In this chapter, we'll take a brief look at the history of JavaScript web apps
so we can understand the need for routing. We'll then examine how Marionette
aims to solve the issues we uncover.


## History

One of the main benefits to building a JavaScript application is showing/hiding
views based on user input without lengthy calls to the server. For example, when
choosing an item from a list, we can just render that item based on information
stored in a `Collection` without making any lengthy calls to the server to fetch
data we have in our browser already. Over time this method has been successfully
used in many applications, reducing server load and increasing the
responsiveness of our web applications.

However, a major downside of this method has historically been the inability to
get back to that page. You'll undoubtedly remember the frustration of
accidentally clicking back, refreshing a page, or closing your browser only to
get sent to the home page and have to find your way back to the screen you were
working on, or that interesting article you were reading.


### Solutions

Over the years different methods were tried, including having the server try to
remember your position in an application, mostly with poor results, bugs,
complex code, inability to bookmark or share a post, and many other issues you
have likely encountered. What we really wanted was to just use the trusted
browser URL in our applications.

Twitter popularized the use of the fragment - the portion of the URL after `#` -
to store information about the current location. This part of the URL isn't read
by the server but is understood by web browsers to jump to certain  sections of
a page. It is also the part of the `window.location` object that we can change
without causing the browser to change the whole page from underneath us. When we
display some information we may want to retrieve later, we could simply do:
`window.location.hashCode = 'newLocation';` and the fragment would be updated.
When we want to retrieve the data on page load, we can read the value of
`window.location.hashCode` and map it to the code we want to execute to retrieve
and render the information referenced by the fragment.


## Routing in Marionette

The Marionette `AppRouter` is an attempt to abstract over this behavior and
provide a URL routing engine that mirrors those found in the more mature
server-side web frameworks such as those provided by Django, Flask, and Ruby on
Rails.

There are two aspects to routing in Marionette:

  1. Set the fragment for state to restore later.
  2. Render the view matching the fragment on page load.


### Setting the fragment

To remember a state that we want to come back to later, we simply call:
`Backbone.history.navigate('route/to/restore');` to update the fragment. The key
thing we need to remember is that _this is all `navigate` does!_ We must not
call `navigate` if we want to run code from our router, we'll see how to do this
in the Router and Controller section below.


## Router and Controller

Routing in Marionette is split into two connected parts: a router and a
controller. The router takes a map of URL fragments to listen for and maps them
to method names on an attached controller to call. The controller is a simple
object with matching methods to be called by the router.

The Marionette class used is called the `AppRouter`. To use it we simply attach
an `appRoutes` object to map the routes to methods:

```js
var Router = Marionette.AppRouter.extend({
  appRoutes: {
    'blog/': 'blogList',
    'blog/:entry': 'blogEntry',
    'blog/:entry/comments/:comment': 'blogComment'
  }
});

module.exports = Router;
```

The methods referenced in `appRoutes` must exist on an attached controller. We
can attach this controller in a number of ways which
[we'll explore shortly][controllers].


### What happens now?

The router watches the fragment when the page loads, and calls the mapped mathod
for that route. For example, `http://example.com#blog/` will, once the router is
initialized, call `blogList` for us.

The other two routes have variables in their URLs (`entry` and `comment`) which
we pass into the mapped methods. Some examples:

  * `http://example.com#blog/3` will call `blogEntry('3')`
  * `http://example.com#blog/5/comments/32` will call `blogComment('5', '32')`
  * `http://example.com#blog/my-title/comments/2015-02-11` will call
    `blogComment('my-title', '2015-02-11')`

As we learned above, the controller's methods will _not_ be called when
`Backbone.history.navigate` is called. For example,
`Backbone.history.navigate('blog/5/comments/32')` will set the fragment to
`#blog/5/comments/32` and that's it.


### Attaching a Controller

Once you've created your `AppRouter` you'll need to
[attach a `controller`][controllers] to it before any routes will be activated.
To attach a controller, simply extend `Marionette.Object` with the method names
matching the values of `appRoutes` and attach an instance to our
`AppRouter.controller` attribute:

```js
var Controller = Marionette.Object.extend({
  blogList: function() {
    // ...
  },
  blogEntry: function(entry) {
    // ...
  },
  blogComment: function(entry, comment) {

  }
});

var Router = Marionette.AppRouter.extend({
  controller: new Controller(),
  appRoutes: {
    // ...
  }
});
```


## A simple app

Now we have all the building blocks in place, we can look at a simple app that
uses a single region and manages which view is being rendered using routes.
As is common in many applications, the `Controller` is going to be the core of
our app, initializing our layouts and handling rendering.

We'll just show the Router and View here. For the full app,
[visit the appendix][appendix].

### Our Router

Our router and controller will be stored in `router.js` and looks like:

```js
var LayoutView = require('./blog');


var Controller = Marionette.Object.extend({
  initialize: function() {
    var layout = new LayoutView();
    layout.render();
    this.options.layout = layout;
  },

  blogList: function() {
    var layout = this.getOption('layout');
    layout.triggerMethod('show:blog:list');
  },

  blogEntry: function(entry) {
    var layout = this.getOption('layout');
    layout.triggerMethod('show:blog:entry', entry);
  }
})

var Router = Marionette.AppRouter.extend({
  appRoutes: {
    'blog/': 'blogList',
    'blog/:entry': 'blogEntry'
  },

  controller: new Controller
});

module.exports = Router;
```

Here we setup our top-level view in the controller, which simply renders by
triggering events on our view. The view, described below, then renders the
correct view based on the layout.


### Our View

In our `blog.js` file, we'll just outline the top-level layout. If you want to
see the full application, check out the [appendix][appendix]:

```js
var LayoutView = Marionette.LayoutView.extend({

  template: require('./templates/layout'),

  regions: {
    main: '.app-hook'
  },

  onShowBlogList: function() {
    this.showChildView('main', new BlogListView());
  },

  onShowBlogEntry: function(entry) {
    var model = this.collection.get(entry);
    this.showChildView('main', new BlogEntryView({model: model}));
  },

  /** Called when `BlogEntryView` triggers `show:blog:list` */
  onChildviewShowBlogList: function() {
    this.triggerMethod('show:blog:list');
    Backbone.history.navigate('blog/');
  },

  /** Called when `BlogListView` triggers `show:blog:entry` */
  onChildviewShowBlogEntry: function(entry) {
    this.triggerMethod('show:blog:entry', entry);
    Backbone.history.navigate('blog/' + entry);
  }
});

module.exports = LayoutView;
```

The key is that the layout can listen to its children, renders its main region
and calls `Backbone.history.navigate` to let us know that a URL change occurred.
Our router then, at page load, attempts to match a fragment and triggers the
main layout to render the correct view.


## Starting the Router

When you load the page, you'll notice the page hasn't responded to the fragment.
There's one last thing we need to do - start the routing framework. Luckily for
us, this is quite simple. We just need to call the following method after
[initializing our application][application]:

```js
Backbone.history.start();
```


## Browser History API
More recently, browser vendors recognized the benefits of this pattern and began
working on the Browser History and Push State APIs. These combined the benefits
of using the fragment with more natural looking URLs that could be recognized by
the server too.

Backbone and Marionette support this API by setting `{pushState: true}` in the
options passed to `start` like so: `Backbone.history.start({pushState: true})`.


[appendix]: ../appendix/approuter/router.md "Full Router Example"
[application]: ../application/README.md "Creating an Application"
[controllers]: ./attach_controllers.md "Attaching Controllers"
