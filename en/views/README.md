# Views and Templates

In the web, the user interacts with your application using their web browser.
The web browser displays your page by rendering the HTML your server sends it.
It can also, crucially, render HTML that you generate inside the browser itself
using JavaScript. This is what allows us to build dynamic applications (like
Google Docs, Trello, and Slack) inside the web browser.

Early web browsers weren't very good at executing JavaScript and, as a result,
early web developers didn't write much JavaScript, preferring to do all the
processing and HTML generation on the server. Over time, with the development of
complex client-side applications like Gmail, and the improvement of JavaScript
engines in Chrome and Firefox, more and more processing and data rendering could
be moved into the user's browser.

You've probably heard of jQuery - the first popular library for manipulating
the HTML in your browser based on user input. Even today, jQuery is used by
many sites to handle simple, and complex, HTML manipulation. Marionette itself
uses jQuery to handle the low-level details of rendering your data.

Developers used the simpler syntax offered by jQuery and began building
extremely complex web applications that would listen to user events, interact
with web servers, and render the page based on the outcome. However, over time
this code would become more and more difficult to manage. Developers would embed
application state inside the page itself which, as the system grew more complex,
would become harder to reason about. If multiple sources were acting on a part
of a page, what would be the outcome of the new function we're adding? What if
we didn't expect certain data to exist in the HTML elements we're acting on?

Marionette aims to solve this problem by taking lessons from elsewhere in the
application development world - desktop and mobile apps. By splitting data
storage, rendering, and handling user-input, it becomes easier to reason about
the expected states of the application and to extend it.


## Marionette Views

To structure the data on your page, Marionette requires you to split up the
logical structure into different views. Each view takes a template which it can
render (transform into HTML) and display. Your view will then watch this section
of your application for user input, providing the hooks you need to react to
user actions.

Let's start by building a simple view with the following template which we'll
call `mytemplate.html`:

```html
<div class="mytext">Some text to render</div>
<input class="myinput" />
<button class="mybutton" type="button">Click Me</button>
```

Now that we have the template, we'll create a view to draw it:

```javascript
var Marionette = require('backbone.marionette');

var MyView = Marionette.LayoutView.extend({
  template: require('mytemplate.html')
});

view = new MyView();
view.render();
```

This is among the simplest views we could build - it simply renders the HTML
displayed and (with some extra code) will attach it to our page. We have a
button on display, let's do something when it gets clicked:

```javascript
var Marionette = require('backbone.marionette');

var MyView = Marionette.LayoutView.extend({
  template: require('mytemplate.html'),

  events: {
    'click .mybutton': 'alertBox'
  },

  alertBox: function() {
    alert('Button Clicked');
  }
});

view = new MyView();
view.render();
```

Now, whenever we click our button, we'll get an alert box. This is handled
through the [`events` object][events]. Put simply, the events object maps a
combination of a DOM event (e.g. click, keyup) with a jQuery selector
(`.mtbutton`) to a method to call on the view (`alertBox`). We can do something
a little more complex like so:

```javascript
var Marionette = require('backbone.marionette');

var MyView = Marionette.LayoutView.extend({
  template: require('mytemplate.html'),

  events: {
    'keyup .myinput': 'changeDiv'
  },

  changeDiv: function() {
    var text = this.$el.find('.myinput').val();
    this.$el.find('.mytext').text(text);
  }
});

view = new MyView();
view.render();
```

Now, whenever we modify the input, the contents of the `div` tag will change to
reflect it.

You might find yourself asking why we'll go to all these lengths to do something
we could do in 2 lines of jQuery. We're just using jQuery anyway, aren't we?
Let's do something a little more complex:

```javascript
var Marionette = require('backbone.marionette');

var MyView = Marionette.LayoutView.extend({
  template: require('mytemplate.html'),

  ui: {
    content: '.mytext',
    input: '.myinput',
    save: '.mybutton'
  },

  events: {
    'click @ui.save': 'changeDiv'
  },

  changeDiv: function() {
    var text = this.ui.input.val();
    this.ui.content.text(text);
  }
});

view = new MyView();
view.render();
```

By using the `ui` object we can make the code a little easier to read and a lot
less brittle - changing the underlying template only requires us to update the
`ui` object with the new selectors. We're still just fiddling round the edges
though. Let's take a more complex example. What happens if the data we're
entering and how it needs to be handled are on completely different parts of the
application? In fact, there's no reason for them to be aware of each other in
the system.

### Using Models to share data

We can use a Backbone Model to store data changes and share them between
different views in a structured way. Assuming we have two views that share a
model instance, actions on one view can affect another.

We'll start with the view being affected, with the template `output.html`:

```html
<div class="mytext"><%- mytext %></div>
```

```javascript
var Marionette = require('backbone.marionette');

var Output = Marionette.LayoutView.extend({
  template: require('./output.html'),

  modelEvents: {
    'change:mytext': 'render'
  }
});

module.exports = Output;
```

Because this view is so simple, we'll just completely redraw the template
whenever the underlying data changes.

The view where our user enters data with the template `input.html`:

```html
<input class="myinput" />
<button class="mybutton">Click Me</button>
```

```javascript
var Marionette = require('backbone.marionette');

var Input = Marionette.LayoutView.extend({
  template: require('./input.html'),

  ui: {
    input: '.myinput',
    button: '.mybutton'
  },

  events: {
    'click @ui.button': 'updateModel'
  },

  updateModel: function() {
    var text = this.ui.input.val();
    this.model.update({
      mytext: text
    });
  }
});

module.exports = Input;
```

At the top-level, all we need to do is attach the same model to both views and
they can then both change and listen to it.

## Binding to Models

The above is an example of binding views to models. This is a key aspect of
building Marionette applications, especially those with dynamic data.

To bind a view to a model, simply pass it in when you create a new instance of
the view:

```javascript
var MyView = require('./myview');

var view = new MyView({
  model: new Backbone.Model()
});
```

Once we have a model bound to our view, we can access it from `this.model` and
listen to events on the model. The official [Backbone documentation][backbone]
contains the full list of events, and what they apply to.


### Listening to Model events

If we want our view to listen to events on its attached model, simply bind it
in the `modelEvents` object like so:

```javascript
var MyView = Marionette.LayoutView.extend({
  template: require('./mytemplate.html'),

  modelEvents: {
    'change': 'changeAnything',
    'change:myfield': 'changeSpecificField'
  },

  changeAnything: function(model, options) {
    alert('Triggered on any field change');
  },

  changeSpecificField: function(model, value, options) {
    alert('Triggered because myfield changed - ' + value);
  }
});
```


### Listening to custom events

If the built-in model events aren't sufficient, it's also possible to set and
trigger custom events. For example, `Backbone.Model` only defines a `sync` event
but no special event to tell us what triggered the sync e.g. `fetch()` or
`save()`. Let's imagine we want to execute some custom code after a `save` such
as updating our collection:

```javascript
var MyView = Marionette.LayoutView.extend({
  template: require('mytemplate.html'),

  modelEvents: {
    save: 'afterSave'
  },

  afterSave: function(model, options) {
    alert('Model was saved');
  },

  onButtonClicked: function() {
    var model = this.model;
    model.save({
      success: function() {
        model.trigger('save', model, {});
      }
    });
  }
});
```

Now, whenever that event is fired by the model, we can listen to the save event
and act on it from whichever views are bound to the model. While this works,
this ad-hoc method for triggering model events only works for one-off sections
of your application. If you want to standardize this behavior across your app's
models then we would recommend you provide a custom method or extend the model's
`save()` method (and/or any others).


[backbone]: http://backbonejs.org/#Events-catalog
[events]: ../messaging/README.md
