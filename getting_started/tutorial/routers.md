# Route handling

If we want to build a single-page application, we need to be generate and
respond to URLs. One of the key advantages of the web over other platforms is
the ease of which we can share and bookmark information through the URL.
Because of this, Backbone has a powerful route-handling framework for responding
to URLs as they come in.

## Routers/Controllers - The C in MVC

Web frameworks in most languages include a routing system - Django's `urls.py`
and Flask's route decorators are two prominent examples of different styles.

The approach taken by Backbone and Marionette is slightly different, reflecting the
nature of existing state in client applications. So how and why do we use
routing?

The purpose of the router is to restore state on page load with help from the
browser's `window.location` property. We can either construct the state directly
from the URL or we can use it to start pulling extra data from the server. In
our tutorial, we'll use the URL to locate data already in our client.

We use `Backbone.history` to set points to restore from later. If we wanted to
build a single-page app, we can use the Router and History to setup a
navigate-able system. A common example could be `#items` list where clicking an
item would navigate to `#items/id` and render the item we just clicked.

## A simpler app

We've really stretched the hello world as far as we can take it, so we're going
to cut it back a little to make it easier to demo the router features. The
biggest change we'll make is to use a single layout to manage all of our views.
We'll also assume that we have pre-written all the templates that we're going
to render.

Our application structure will look like:

```
app/
 |-- app.js
 |-- router.js
 |-- collections/
 |    |-- person.js
 |
 |-- models/
 |    |-- person.js
 |
 |-- views/
      |-- personlist.js
      |-- person.js
```

## Creating our Router

A key part of refactoring our app will move almost all our logic into our
controller. This will greatly simplify our application structure and more
closely follow the MVC principles that Backbone is modelled on.

Let's open our `router.js` file and set up some simple routes.
We'll alter our collection view so we can click on names to get a little more
detail about each name.

```js
var Marionette = require('backbone.marionette');

var PersonList = require('./collections/person');

var IndexView = require('./views/personlist');
var PersonView = require('./views/person');


var Controller = Marionette.Object.extend({  // 1
  initialize: function() {  // 2
    this.regions = new Marionette.RegionManager({
      regions: {
        root: '#view-hook'
      }
    });

    this.collection = new PersonList(this.options.data);
  },

  listNames: function() {  // 3
    var view = new IndexView({
      collection: this.collection,
      controller: this
    });

    this.regions.get('root').show(view);  },

  displayName: function(id) {  // 4
    var model = this.collection.get(id);

    var view = new PersonView({
      controller: this,
      model: model
    });

    this.regions.get('root').show(view);
  }
});

var Router = Marionette.AppRouter.extend({
  initialize: function(options) {  // 5
    this.controller = new Controller(options);  // 6
  },

  appRoutes: {  // 7
    '': 'listNames',
    'name/:id': 'displayName'
  }
});

module.exports = Router;
```

Let's break down the router and controller at the most interesting points:

  1. We create a `Marionette.Object` to be our controller.
  2. Our new `Object` has an initialize method and sets up the regions.
  3. We name some methods as functions, which we'll use later.
  4. We can specify some arguments that the route can pass in.
  5. As with other Backbone and Marionette features, we can pass options.
  6. We specify the `controller` to use on the `AppRouter`.
  7. We define our `appRoutes` as a key-value hash of route to method.

You can see how the bulk of our application initialization has been moved into
our controller. This gives our controller direct access to everything it needs
to manage the regions and views of the application.

### Using `appRoutes`

We use `appRoutes` to map URLs to methods. We use the fragment (`#`) as our
client-side URL - everything after the fragment is parsed as the total URL. We
can pass variables in from the URL using the `:<var_name>` format as we can see
in the route for `displayName` that will match `name/15`, for example.

An important consideration for routers is that the routes themselves must not
run any initialization logic - this must stay in `initialize`. This is because
the routes are triggered on back/forward which, in fragments, doesn't cause a
page reload. If our page initialization logic were inside the route (and not
`initialize`) then it would get run whenever that route gets matched!

### What is a `Marionette.Object`?

You'll have noticed we introduced a `Marionette.Object` here. For our purposes
this behaves like a regular object mixed with the usual Marionette attributes,
such as automatically attaching options to `this.options`.

### Carrying on

Now that we have our Router, we'll use it to manage our page loads and start
creating our views:

```js
require('backbone').$ = require('jquery');
var Marionette = require('backbone.marionette');

var Router = require('./router');  // 1

var listData = [
  {
    id: 1,
    first_name: 'Dave',
    last_name: 'Jones'
  },
  {
    id: 2,
    first_name: 'Steve',
    last_name: 'Hansen'
  }
]


var App = Marionette.Application.extend({
  onStart: function(options) {
    var router = new Router({
      data: options.data
    });  // 2

    Backbone.history.start();  // 3
  }
});

var app = new App();
app.start({data: listData});
```

The changes are as follows:

  1. We import our Router.
  2. We pass our app into the `router` so we can access views when routing.
  3. We start the `Backbone.history` which will trigger the route-handling.

Our `Application` has now been stripped into a shell that interfaces directly
with the initializing page and kick-starts the rest of the application. This
has the effect of isolating most of the application from its operating
environment. If we wanted to take this a step further, we could expose the
`Application` through `module.exports` and start it from a separate driver
file - allowing us to write automated integration tests against our app with
custom test data.

### The views

We'll now create our list index view `views/personlist.js`:

```js
var Marionette = require('backbone.marionette');


var PersonView = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require('../templates/person/item.html'),

  triggers: {
    'click': 'select:person'
  }
});


var PersonList = Marionette.CollectionView.extend({
  childView: PersonView,
  tagName: 'ul',
  template: require('../templates/person/list.html'),

  onChildviewSelectPerson: function(child, options) {
    Backbone.history.navigate('name/' + model.id);  // 1
    this.options.controller.displayName(child.model.id);  // 2
  }
})


module.exports = PersonList;
```

There are two interesting parts in our `PersonList`:

  1. We then navigate to the new URL.
  2. We passed the controller into our view so we can call methods on it.

### What does `Backbone.history.navigate` do?

The `navigate` method is a simple way to update the fragment with the new URL.
It's important to note that we don't execute any extra code using this. Its only
purpose is to update the page state so we can either retrieve it, or access it
with the back/forward buttons on the browser. We could forcibly execute that
code by passing the `{trigger: true}` argument to `navigate` but it's best not
to if we can avoid it - it leads to a flaky design that's extremely difficult
to manage.

### The detail view

Our detail view is pretty straightforward and just contains a way to return to
the list. Let's open up `views/person.js` and get started:

```js
var Marionette = require('backbone.marionette');


var PersonView = Marionette.LayoutView.extend({
  template: require('../templates/person/detail.html'),

  ui: {
    back: '.back'
  },

  triggers: {
    'click @ui.back': 'click:back'
  },

  onClickBack: function() {
    Backbone.history.navigate('');
    this.options.controller.listNames();
  }
});


module.exports = PersonView;
```

As an aside, you can quickly see the advantages of using the `ui` hash - we
don't know what exactly `back` looks like in HTML but we can decide that later
and not worry about having to make lots of little fixes later.

## What next?

Now that we know how to route single-page applications, we can build a fairly
complex application that handles user input, saves state in the client, and
can be easily navigated from the location bar in your browser. We can share
the URLs we generate as if this application were just a web page.

The next major building block is saving and retrieving data
[from a web server](./servers.md).
