# Built-in Triggers

Marionette views include a set of built-in triggers that are fired throughout
the View creations and destruction lifecycle. These triggers are designed to
give you more fine-grained control over your view, for example being able to
modify your template before it gets attached to your page, or even before the
template gets rendered.

It's also possible to [write your own custom triggers and handlers][events] to
provide hooks for specific actions that aren't provided by Marionette.

## Listening for triggers

To listen to a trigger, we can attach a handler to our view:

```javascript
// Listen to the 'render' trigger
view = new Marionette.LayoutView();
view.listenTo('render', function() {
  console.log('View was rendered');
});

// Listen to the 'before:attach' trigger
view.listenTo('before:attach', function() {
  console.log('View is about to be attached to the page');
});
```

Most triggers have `before:action` and `action` hooks that fire before and after
`action` has occurred. A common example could be `before:render` and `render`
which fire before and after the template is rendered.

### Short-hand

All triggers that get fired can also be handled by defining methods in a
specific style on our views: the `onTriggerFired` format e.g. `onBeforeRender`.
To know what name our handler method requires, we simply take the trigger name
(`before:render` in this case), prefix `on` and uppercase the first letter and
every first letter following a `:` in the name. So `render` becomes `onRender`,
`before:render` becomes `onBeforeRender`, `attach` becomes `onAttach`, etc.

An example should help explain:

```javascript
var MyLayout = Marionette.LayoutView.extend({
  onBeforeRender: function() {
    console.log('before:render was fired');
  },

  onRender: function() {
    console.log('render was fired');
  }
});
```

For simplicity, this reference will use the `trigger:fired` format, which you
should now be able to convert to the method name using the formula described
above.


## List of View Triggers

This reference lists all the triggers that can be fired in alphabetical order,
with the `before:<trigger>` as part of the `<trigger>` section. We'll list the
view types that can fire each trigger (or all), and what each trigger means to
you. Where appropriate, we'll provide an example of common use of the trigger.

If you just want to know what triggers a view can fire, skip to the bottom of
this reference to find the list of views and the triggers they can each fire.


### `add:child` and `before:add:child`

This trigger fires whenever a child view gets rendered and attached to the
`CollectionView`. We will get an argument referencing the child that has just
been added. This can be useful if you need to do some extra work on the child
view where you need information from the parent.


#### Example


#### On Views

  - `CollectionView`
  - `CompositeView`


### `attach` and `before:attach`

This trigger fires once our view gets attached to the actual DOM i.e. once the
view is completely rendered and viewable in the browser window. This is usually
used to perform actions that require the view to be completely rendered and
visible - for instance displaying a modal or datepicker widget. We also use this
as a safe point where we know that the view is finished and active for the user.


#### Example

This example uses [Bootstrap's Modal JavaScript][bootstrap-modal] which requires
the HTML to be attached to the DOM to work.

```javascript
var ModalLayout = Marionette.Layout.extend({
  ui: {
    wrapper: '.modal-wrapper'
  },

  /** Use Bootstrap's modal JavaScript. This assumes that the JS files are
  *   loaded and will work.
  */
  onAttach: function() {
    this.ui.wrapper.modal('show');
  }
});
```


#### On Views

  - `View`
  - `ItemView`
  - `LayoutView`
  - `CollectionView`
  - `CompositeView`


### `destroy` and `before:destroy`

Fired by the region manager to perform any extra clean up on the view after it
has been destroyed. This can be things like executing extra JS on plugins like
modals and datepickers.


#### Example

In this example, we'll use the `before:destroy` trigger to close a modal before
it gets removed from the DOM. We're using something similar to
[Bootstrap's modal JavaScript][bootstrap-modal].

```javascript
var ModalView = Marionette.LayoutView.extend({
  ui: {
    wrapper: '.modal-wrapper'
  },

  onBeforeDestroy: function() {
    this.ui.wrapper.modal('hide');
  }
});
```

As an aside, this wouldn't work in Bootstrap as the `modal` method returns
immediately, causing the view to be destroyed before the modal can be hidden.


#### On Views

  - `View`
  - `ItemView`
  - `LayoutView`
  - `CollectionView`
  - `CompositeView`


### `dom:refresh`

Only fired when a view is re-rendered. This does not get fired on the first
render, only on subsequent rendering.


#### Example

This example demonstrates when the `dom:refresh` trigger is fired.

```javascript
var MyLayout = Marionette.LayoutView.extend({
  onDomRefresh: function() {
    console.log('dom:refresh');
  },

  onRender: function() {
    console.log('render');
  },

  onShow: function() {
    console.log('show');
  }
});

var view = new MyLayout();

regionManager.get('layout').show(view);
// 'render'
// 'show'

view.render();

// 'render'
// 'dom:refresh'
```


#### On Views

  - `View`
  - `ItemView`
  - `LayoutView`
  - `CollectionView`
  - `CompositeView`


### `render` and `before:render`

The `render` event fires when the view's HTML template has been rendered
(the string has been built) but before it gets attached to the browser DOM. We
commonly use this hook to operate on the HTML but before the user can see it.


#### Example

A common example is to check for data and add/remove classes on the view's
element every time it gets rendered. This example checks to see if a table row
was selected for a form element and attaches the appropriate class.

```javascript
var RowView = Marionette.LayoutView.extend({
  tagName: 'tr',

  onRender: function() {
    if (this.model.get('selected')) {
      this.$el.addClass('selected');
    }
    else {
      this.$el.removeClass('selected');
    }
  }
});


var TableView = Marionette.CompositeView.extend({
  tagName: 'table',

  childView: RowView,
  childViewContainer: 'tbody'
});
```


#### On Views

  - `ItemView`
  - `LayoutView`
  - `CollectionView`
  - `CompositeView`


### `render:collection` and `before:render:collection`

Fired after the collection has been rendered and attached to the view. This will
only fire if the view's collection is _not_ empty. In other words, if the view
attached to `emptyView` is displayed, this trigger won't be fired.


#### Example


#### On Views

  - `CollectionView`
  - `CompositeView`


### `render:template` and `before:render:template`

The `render:template` trigger is fired when the wrapper template for a
`CompositeView` is rendered but before the collection items have started
rendering - `childViewContainer` will be empty while this trigger is first
fired.


#### Example


#### On Views

  - `CompositeView`


### `reorder` and `before:reorder`

Gets fired whenever we reorder the underlying collection attached to a view.


#### Example


#### On Views

  - `CollectionView`
  - `CompositeView`


### `remove:child` and `before:remove:child`

The `remove:child` trigger gets fired when a collection item is removed,
typically when `this.collection.remove(id)` is called. Unlike the collection
event, this fires after the view has been removed from the DOM.


#### Example


#### On Views

  - `CollectionView`
  - `CompositeView`


### `show` and `before:show`

The `show` event fires when `region.show(view)` - or
`layoutView.showChildView(view)` - is complete. We'll commonly use this trigger
to build up our nested layout hierarchy and show a layout's sub-views.


#### Example

Our example is a Marionette pattern that is commonly used to build up our view
hierarchy by chaining `show` handlers.

```javascript
var MyLayout = Marionette.LayoutView.extend({
  regions: {
    todoList: '.todo-hook'
  },

  onShow: function() {
    console.log('regionManager has show this view');

    this.showChildView('todoList', new CompositeView({
      collection: this.collection
    }); // This will trigger CompositeView's 'before:show' and 'show' handlers
  }
})
// Assuming regionManager has been defined with a region 'layout'
regionManager.get('layout').show(new MyLayout({collection: todoList}));
```

#### On views

  - `View`
  - `ItemView`
  - `LayoutView`
  - `CollectionView`
  - `CompositeView`


## List of Views and their Triggers

&nbsp;      |  `View`  | `ItemView` | `LayoutView` | `CollectionView` | `CompositeView`
------------|:--------:|:----------:|:------------:|:----------------:|:--------------:
`add:child` |          |            |              |     &#10003;     |    &#10003;
`attach`    |  &#10003;|  &#10003;  |    &#10003;  |     &#10003;     |    &#10003;
`destroy`   | &#10003; |  &#10003;  |    &#10003;  |     &#10003;     |    &#10003;
`dom:refresh` | &#10003; | &#10003; |    &#10003;  |     &#10003;     |    &#10003;
`render`    |  &#10003;|  &#10003;  |    &#10003;  |     &#10003;     |    &#10003;
`render:collection` |  |            |              |     &#10003;     |    &#10003;
`render:template` |    |            |              |                  |    &#10003;
`reorder`   |          |            |              |     &#10003;     |    &#10003;
`remove:child` |       |            |              |     &#10003;     |    &#10003;
`show`      |  &#10003;|  &#10003;  |    &#10003;  |     &#10003;     |    &#10003;


## Standard Trigger Lifecycles

The Marionette view lifecycle involves firing a number of triggers at each stage
of its creation. The exact lifecycle depends on how the view is displayed and
what type of view it is. We'll cover the most common view types and


### Show a View from a Region

This code is run when we show a view using regions:

```javascript
this.showChildView('myRegion', childView);
```

![Marionette Lifecycle](images/marionette-region-show-lifecycle.png)

[bootstrap-modal]: http://getbootstrap.com/javascript/#modals-methods
[events]: ./events.md
