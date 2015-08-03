# Events and Triggers

Communicating data within and between views is one of the major challenges for
any application framework. The method chosen by Backbone was to provide a
framework for triggering and handling events fired by objects. The `modelEvents`
object we've already seen is an example of this from Backbone. Marionette takes
this a step further by providing a more powerful trigger framework on top of
Backbone's event handlers. With triggers, we are able to decouple the firing of
an event from its handler. Many views fire triggers by default, leaving it to
the developer to choose how (or if) they handle them.


## Using Events

Marionette provides an `events` object that allows us to listen to activity in
our view's template and respond to it with a specific method. For example, we
can listen to data entry or a button press and determine the method to call. For
example:

```javascript
var MyView = Marionette.LayoutView.extend({
  template: require('./events.html'),

  ui: {
    title: '.title',
    save: '.save'
  },

  events: {
    'keyup @ui.title': 'setTitle',
    'blur @ui.title': 'setTitle',
    'click @ui.save': 'saveForm'
  },

  setTitle: function(domEvent) {
    this.model.set({
      title: this.ui.title.val()
    });
  },

  saveForm: function(domEvent) {
    this.model.save();
  }
});
```

This is a simple example of setting a model field and saving the data to the
server. It should be clear what methods are being called when we perform actions
on the page. The `events` object must be bound at initialization and will cause
an error if any of the referenced methods don't exist on the view.


## Triggers

What happens if we want to provide more generic behavior? We could want to build
a base view for our application that gets extended by other views, providing
events that can be listened to when we want to extend the behavior of the view.
Forcing us to define all the methods when creating the base class would be
overkill, especially in a language like JavaScript that aims for a certain
amount of brevity and clarity.

Luckily Marionette gives us the `triggers` framework. To use a trigger, we
simply define the name of the trigger to be fired and either provide listeners
or, for simplicity, a special method name that will be called. We'll rewrite our
above example to demonstrate:

```javascript
var MyView = Marionette.LayoutView.extend({
  template: require('./events.html'),

  ui: {
    title: '.title',
    save: '.save'
  },

  triggers: {
    'keyup @ui.title': 'set:title',
    'blur @ui.title': 'set:title',
    'click @ui.save': 'save:form'
  },

  onSetTitle: function(domEvent) {
    this.model.set({
      title: this.ui.title.val()
    });
  },

  onSaveForm: function(domEvent) {
    this.model.save();
  }
});
```

As you can see, the changes were minor - we changes the `events` object to
`triggers` and specified some trigger names. Triggers are typically named
using the `:` as word separators. This special syntax is recognized by
Marionette to work out which methods to call. As you can see, `set:title` will
call `onSetTitle` when triggered and `save:form` will call `onSaveForm` when
triggered.

We can also manually fire triggers. A common pattern is to use a `before:`
trigger to alert listeners that an action is about to be performed. Take our
save example:

```javascript
var MyView = Marionette.LayoutView.extend({
  template: require('./events.html'),

  ui: {
    title: '.title',
    save: '.save'
  },

  triggers: {
    'keyup @ui.title': 'set:title',
    'blur @ui.title': 'set:title',
    'click @ui.save': 'save:form'
  },

  onSetTitle: function(domEvent) {
    this.model.set({
      title: this.ui.title.val()
    });
  },

  onSaveForm: function(domEvent) {
    this.triggerMethod('before:model:save');
    var view = this;

    this.model.save({
      success: function() {
        view.triggerMethod('model:save');
      }
    });
  }
});
```

This style is used throughout Marionette to allow developers to bind hooks
before and after common actions occur. Some of the most common examples are the
render and show hooks: `before:render`, `render`, `before:show`, `show`.

We can listen to these standard triggers just like any other:

```javascript
var MyView = Marionette.LayoutView.extend({
  template: require('./events.html'),

  onBeforeRender: function() {
    alert('This view is about to be rendered');
  },

  onRender: function() {
    alert('This view was rendered');
  }
});
```
