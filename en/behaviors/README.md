# Sharing common view Behavior

While building your [views][views], you'll find that you're repeating some
common operations such as displaying modals, handling form validation, or
custom form fields with complex behavior. We can try to share this using
inheritance:

```javascript
var BaseFormView = Marionette.LayoutView.extend({
  ui: {
    save: '.save-button'
  },

  triggers: {
    'click @ui.save': 'save:form'
  },

  onSaveForm: function() {
    this.model.save();
  }
});

var PersonForm = BaseFormView.extend({
  template: require('./templates/form.html')
});
```

This will work until you need to override anything that the base view provides
i.e. the `ui` or `triggers` hashes. If the form is also a modal, we then have
to choose the View to extend, forsaking the other. Clearly we need a better way.


## What is a Behavior?

At its most basic level, a Behavior is an object attached to a view that is able
to listen, and respond, to events/triggers on the view. It has access to the
view object, allowing it to perform modifications if it needs to. Let's take
a look at a simple form Behavior:

```javascript
var FormBehavior = Marionette.Behavior.extend({
  ui: {
    save: '.save-button'
  },

  events: {
    'click @ui.save': 'saveForm'
  },

  saveForm: function() {
    this.view.model.save({})
  }
});


var PersonForm = Marionette.LayoutView.extend({
  behaviors: {
    form: {
      behaviorClass: FormBehavior
    }
  },

  template: require('./templates/form.html')
});
```

Now, with a slightly different configuration, we've achieved the same effect in
a way that can be shared across all views that contain a `.save-button` element.


[views]: ../views/README.md
