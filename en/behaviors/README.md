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
The `behaviors` object is used to map Behaviors to Views with extra options. The
object key is completely arbitrary, here we've decided to call it `form` to
reflect its use. We then tell the view which class to use with the
`behaviorClass` key. Next we'll examine ways of adding extra information to our
behavior.


## Listening to View events

Behaviors are able to listen to events and triggers fired on views, models, and
collections. Let's take an example of a behavior that highlights newly created
items in collections:


```javascript
var SuccessBehavior = Marionette.Behavior.extend({
  collectionEvents: {
    add: 'highlightModel'
  },

  highlightModel: function(model, collection, options) {
    var unhighlight = collection.where({newlyAdded: true});
    _.each(unhighlight, function(item) {
      item.set('newlyAdded', false);
    });

    model.set('newlyAdded', true);
  }
});


/** Using Bootstrap CSS */
var RowView = Marionette.LayoutView.extend({
  tagName: 'tr',
  template: require('./templates/row.html'),

  modelEvents: {
    'change:newlyAdded': 'render'
  },

  onRender: function() {
    if (this.model.get('newlyAdded')) {
      this.$el.addClass('success');
    }
    else {
      this.$el.removeClass('success');
    }
  }
});

var TableView = Marionette.LayoutView.extend({
  behaviors: {
    successHighlight: {
      behaviorClass: SuccessBehavior
    }
  },

  tagName: 'table',
  className: 'table',
  template: require('./templates/table.html')
});
```

This behavior can now, when a new item is added to our collection, highlight the
table row of the newly added model, like so:

```javascript
var table = new TableView({
  collection: new Collection([
    {name: 'Scott', language: 'English', handle: 'scott-w'}
  ])
});
table.add({name: 'Joanne', language: 'English', handle: 'jdaudier'});
```


## Adding options to our Behavior



[views]: ../views/README.md
