# Integrating with a Web Server

After understanding how we can build applications that we can navigate, interact
with and list data through, it's time to pull it all together by integrating
with a web server.

## Server API

Before going any further, we need to define an imaginary API that we'll use.
For the uninitiated, a web service API is a way of communicating with our web
application so we can save state and access it from other machines connected to
the internet. Common tools for building web application APIs are
[Ruby on Rails](http://rubyonrails.org/) and
[Django REST Framework](http://django-rest-framework.org/), though there are
plenty available.

We'll define two endpoints and assume we're serving the API from the same host
as our web application:

`/people` returns the following JSON list:

```js
[
  {
    "id": 1,
    "first_name": "Steve",
    "last_name": "Jones",
    "url": "/people/1"
  },
  {
    "id": 4,
    "first_name": "James",
    "last_name": "Haskell",
    "url": "/people/4"
  }
]
```

In addition, we can issue a POST request to this URL with the following JSON
object to create a new person:

```js
{
  "first_name": "Jack",
  "last_name": "Doe"
}
```

`/people/:id` returns a JSON object matching the ID, if passed 1:

```js
{
  "id": 1,
  "first_name": "Steve",
  "last_name": "Jones",
  "url": "/people/1"
}
```

We can issue a PUT request against any endpoint with the following JSON
structure to update the identified person:

```js
{
  "first_name": "Jack",
  "last_name": "Doe"
}
```

## Integrating our Application

Now we have our API defined, we can start to work with it! Now, instead of
pre-defining a list when we load our page, we can just define our URLs on our
collections and Backbone will work everything out for us.

Our application structure is identical to before, so we'll start by amending our
`collections/person.js`:

```js
var Backbone = require('backbone');

var Person = require('../models/person');


var PersonCollection = Backbone.Collection.extend({
  model: Person,

  url: '/people'
});

module.exports = PersonCollection;
```

Now we'll change our `app.js` file to remove references to existing data:

```js
require('backbone').$ = require('jquery');
var Marionette = require('backbone.marionette');

var Router = require('./router');


var App = Marionette.Application.extend({
  onStart: function(options) {
    var router = new Router();

    Backbone.history.start();
  }
});

var app = new App();
app.start();
```

Finally we'll amend our `router.js` so we get the data from the server:

```js
var Marionette = require('backbone.marionette');

var PersonList = require('./collections/person');

var IndexView = require('./views/personlist');
var PersonView = require('./views/person');


var Controller = Marionette.Object.extend({
  initialize: function() {
    this.regions = new Marionette.RegionManager({
      regions: {
        root: '#view-hook'
      }
    });

    this.collection = new PersonList();  // 1
    this.collection.fetch();  // 2
  },

  listNames: function() {
    var view = new IndexView({
      collection: this.collection,
      controller: this
    });

    this.regions.get('root').show(view);  
  },

  displayName: function(id) {
    var model = this.collection.get(id);

    var view = new PersonView({
      controller: this,
      model: model
    });

    this.regions.get('root').show(view);
  }
});

var Router = Marionette.AppRouter.extend({
  controller: new Controller(),  // 3

  appRoutes: {
    '': 'listNames',
    'name/:id': 'displayName'
  }
});

module.exports = Router;
```

We have made a few changes to simplify this:

  1. We no longer pass any initial data into the Collection.
  2. We execute `fetch()` to pull this data from the server.
  3. We can just define a `new Controller()` directly on the `AppRouter`.

When we load our application, the data will be fetched from the server and our
list view will automatically render when the data is fetched. Also notice how
the URLs on the router don't need to be associated in any way with the router's
URL scheme. We can easily use multiple collections/models and blend the data
together in different views under our application hierarchy.

## Navigating directly to a person

If we navigate directly to `#name/1` you'll notice it doesn't work at all! This
is because, when the controller executes, the collection is empty, so
`this.collection.get(1);` returns `undefined` and we just get a cascade of
errors.

Let's have the view always grab the data from the server before it attempts to
render as a first solution:

```js
var Controller = Marionette.Object.extend({
  initialize: function() {
    this.regions = new Marionette.RegionManager({
      regions: {
        root: '#view-hook'
      }
    });

    this.collection = new PersonList();  
    this.collection.fetch();  
  },

  listNames: function() {
    var view = new IndexView({
      collection: this.collection,
      controller: this
    });

    this.regions.get('root').show(view);  
  },

  displayName: function(id) {
    var model = new Person({id: id});

    var view = new PersonView({
      controller: this,
      model: model
    });

    this.listenTo(model, 'sync', function() {
      this.regions.get('root').show(view);
    });
    model.fetch();
  }
});
```

We just need to amend our `models/person.js`:

```js
var Backbone = require('backbone');


var Person = Backbone.Model.extend({
  urlRoot: '/person'
});

module.exports = Person;
```

With this in place, navigating to `#name/1` won't display anything until the
model has loaded from the server. There is a downside to this, however, in that
we no longer attempt to get the model from the collection, even though we might
have all its data sitting in memory ready to use!

## A more effective solution

Let's use the model defaults and events to get the right balance between
performance and correct behavior.

We'll go back to our `models/person.js` to start:

```js
var Backbone = require('backbone');


var Person = Backbone.Model.extend({
  defaults: {
    first_name: '',
    last_name: ''
  }
  urlRoot: '/person'
});

module.exports = Person;
```

With our defaults in place, we can safely render our view with an empty `Person`
and backfill its details later. Let's open up `router.js` and do this:

```js
var Controller = Marionette.Object.extend({
  initialize: function() {
    this.regions = new Marionette.RegionManager({
      regions: {
        root: '#view-hook'
      }
    });

    this.collection = new PersonList();  
    this.collection.fetch();  
  },

  listNames: function() {
    var view = new IndexView({
      collection: this.collection,
      controller: this
    });

    this.regions.get('root').show(view);  
  },

  displayName: function(id) {
    var model = this.collection.get(id);
    if (_.isUndefined(model)) {
      model = new Person({id: id});  // 1
      model.fetch();
    }

    var view = new PersonView({
      controller: this,
      model: model
    });

    this.regions.get('root').show(view);
  }
});
```

  1. Our only major change is to try to grab the item before creating it from
  scratch

Our view will now, most likely, be rendered as empty and never change. Let's
open up `views/person.js` and add our event handler so it will re-render when
`fetch()` completes:

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

  modelEvents: {
    'sync': 'render'
  },

  onClickBack: function() {
    Backbone.history.navigate('');
    this.options.controller.listNames();
  }
});


module.exports = PersonView;
```

Now, when the `fetch()` completes, `render()` will be called and the view will
be redrawn with the data from the server. And, even better, if we navigated
there from the root URL, the page load will be instant! We could even move the
`model.fetch()` call outside the `if()` block and have our view refreshed with
the latest data if we think it'll change.

## Creating new records

Let's start by creating new records on the server from our person list. For the
following sections we'll use a sample template fragment for handling our form
data that could look something like this:

```html
<label for="id_first_name">First Name</label>
<input type="text" name="first_name" class="first_name" id="id_first_name"
  value="<%- first_name %>" />
<label for="id_last_name">Last Name</label>
<input type="text" name="last_name" class="last_name" id="id_last_name"
  value="<%- last_name %>" />
<button type="button" class="save">Save</button>
```

We don't need all the repetition but I want to identify these form elements by
their class, rather than id.

Now, with this template fragment at the top of our names list, we'll amend our
`views/personlist.js` to have more visibility on the data:

```js
var Marionette = require('backbone.marionette');

var Person = require('../models/person');


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

  ui: {  // 1
    first_name: '.first_name',
    last_name: '.last_name',
    save: '.save'
  },

  triggers: {  // 2
    'click @ui.save': 'create:person'
  },

  modelEvents: function() {  // 3
    'sync': 'addPerson'
  },

  initialize: function() {
    this.model = new Person();
  },

  onChildviewSelectPerson: function(child, options) {
    Backbone.history.navigate('name/' + model.id);  
    this.options.controller.displayName(child.model.id);  
  },

  onCreatePerson: function() {
    this.model.save({  // 4
      first_name: this.ui.first_name.val(),
      last_name: this.ui.last_name.val()
    });
  },

  addPerson: function() {  // 5
    this.collection.add(this.model);
  }
})


module.exports = PersonList;
```

We have a few things going on at once here, so let's break it down:

  1. We define some new `ui` elements pointing at our form.
  2. When the user clicks `Save` we want to record the entered data.
  3. When the data is saved successfully, we want to add it to our list.
  4. Transfer the data from the form to the model and saves it.
  5. The model, with an ID field from the server, will be added to our list.

You'll notice that our form hasn't been cleared, and our model will still be
wired up. We'll need to do a little clean-up here or we'll just be modifying the
same model forever.

The easiest way, in this application, would be to navigate to the new person,
effectively forcing the model to be recreated. That's sort of cheating, so let's
do it the hard way:

```js
addPerson: function() {
  this.collection.add(this.model.clone());
  this.model.clear();
  this.render();
}
```

That wasn't even that hard! The only downside to what we've done here is we've
actually re-rendered everything, including our list. If we have a long list,
we've just given the browser a lot of work to do and made our application really
slow! Without going too much into it, the trick to speeding this up would be
to refactor a `LayoutView` and store the form in its own `region` and re-render
just that. I'll leave that as an exercise for the reader.

## Updating existing records

We'll now update our existing records on the server and propagate those changes
back down. Let's open up our `views/person.js` and have a look. We'll assume
that our person template uses the template fragment described above:

```js
var Marionette = require('backbone.marionette');


var PersonView = Marionette.LayoutView.extend({
  template: require('../templates/person/detail.html'),

  ui: {
    back: '.back',
    first_name: '.first_name',
    last_name: '.last_name',
    save: '.save'
  },

  triggers: {
    'click @ui.back': 'click:back',
    'click @ui.save': 'update:person'
  },

  modelEvents: {
    'sync': 'render'
  },

  onClickBack: function() {
    Backbone.history.navigate('');
    this.options.controller.listNames();
  },

  onUpdatePerson: function() {
    this.model.save({
      first_name: this.ui.first_name.val(),
      last_name: this.ui.last_name.val()
    });
  }
});


module.exports = PersonView;
```

Update is more-or-less identical. However, since we don't need to add/remove
items from a collection, it's even simpler than before. Since the data contains
an `id` field from the server, Backbone knows to update the model instead of
creating a new one!

To make our application more powerful, we could render the edit forms in a
sidebar and any updates on the form models would be reflected back in the
visible list.

## What next?

You now have all the basic building blocks you need to construct applications
using Marionette. The next steps are to start building your own apps and extend
your existing web applications to be more interactive.
