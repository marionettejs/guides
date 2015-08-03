# Structuring your App

A key component of your Marionette toolkit is the `LayoutView`. This view allows
you to break your application's screen into regions that are implemented
separately and pieced together through a parent `LayoutView`.


## The Layout view

Let's demonstrate a simple layout - we'll construct a simple report with a
summary region. We'll start with our `layout.html` template:

```html
<div class="summary"></div>
<div class="report"></div>
```

We've created two empty elements with hooks that we want to attach our regions
to. Next, we'll define our view:

```javascript
var Summary = require('./summary');
var Table = require('./table');


var MyLayout = Marionette.LayoutView.extend({
  template: require('./layout.html'),

  regions: {
    summary: '.summary',
    report: '.report'
  }
});

module.exports = MyLayout;
```

The `LayoutView` takes a `regions` object that maps our keys to jQuery selector
strings in the template. The next step is to tell Marionette what views to
display when it is itself displayed:

```javascript
var Summary = require('./summary');
var Table = require('./table');


var MyLayout = Marionette.LayoutView.extend({
  template: require('./layout.html'),

  regions: {
    summary: '.summary',
    report: '.report'
  },

  onShow: function() {
    var summary = new Summary({model: new Backbone.Model});
    var table = new Table({collection: new Backbone.Collection});

    this.showChildView('summary', summary);
    this.showChildView('table', table);
  }
});

module.exports = MyLayout;
```

We've added our `onShow` handler to our layout. When we show the `MyLayout` view
in a parent region, the `show` trigger is fired, upon which it will then create
and show our summary and table. Each of these views will be rendered and their
`show` triggers will be fired. If these contain `LayoutView`s, we could keep
listening to this trigger and show more views down the tree.


## Layouts as mediators

The `LayoutView` is a great medium for transferring messages between one view
and another, particularly if they're on the same level. For example, we could
contain a form which, whenever the user saves it, adds a new record to a
collection elsewhere in the application. To make this easier, we'll use triggers
on the form. Triggers are [covered more in-depth in another chapter][events] so
we'll stick to the parts that are relevant for our layouts.

We'll gloss over the templates and stick to just the views, keeping everything
in a single file for simplicity:

```javascript
var FormView = Marionette.LayoutView.extend({
  tagName: 'form',
  template: require('./form.html'),

  ui: {
    save: '.save-button'
  },

  triggers: {
    'click @ui.save': 'save:model'
  }
});

var Item = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require('./item.html')
});

var ListView = Marionette.CollectionView.extend({
  tagName: 'ul',

  childView: Item
});

var Layout = Marionette.LayoutView.extend({
  template: require('./layout.html'),

  regions: {
    list: '.list',
    form: '.form'
  },

  onShow: function() {
    this.showChildView(
      'form',
      new FormView({model: new Backbone.Model()})
    );
    this.showChildView(
      'list',
      new ListView({collection: new Backbone.Collection})
    );
  }
});

module.exports = Layout;
```

Now that we have our skeleton, we need to figure out how to get the
`'save:model'`trigger fired on the `FormView` to be recognized from its parent
`Layout`. Luckily, Marionette gives us just a tool for that - the
`childview:` trigger prefix. Every time a trigger is fired on a view, any parent
views can see it as well. Taking just our `Layout`, we can do the following:

```javascript
var Layout = Marionette.LayoutView.extend({
  template: require('./layout.html'),

  regions: {
    list: '.list',
    form: '.form'
  },

  onShow: function() {
    this.showChildView(
      'form',
      new FormView({model: new Backbone.Model()})
    );
    this.showChildView(
      'list',
      new ListView({collection: new Backbone.Collection})
    );
  },

  onChildviewSaveModel: function(child, model) {
    var list = this.getChildView('list');
    var newModel = model.clone();
    list.collection.add(newModel);
    model.clear();
  }
});
```

The layout view knows about all its children, as it should, but the individual
views don't know about each other. This means each view can continue operating
independently and manage its own state - when that state needs to be shared, we
need only start looking up the view hierarchy to see when, and how, that state
gets shared between views.


[events]: ../messaging/README.md
