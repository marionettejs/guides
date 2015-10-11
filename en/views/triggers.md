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


### `attach` and `before:attach`

This trigger fires once our view gets attached to the actual DOM i.e. once the
view is completely rendered and viewable in the browser window. This is usually
used to perform actions that require the view to be completely rendered and
visible - for instance displaying a modal. We also use this as a safe point
where we know that the view is finished and active for the user.


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

  * **All**


[events]: (./events.md)
