# Common Behavior Patterns

There are a number of design patterns that can be made simpler with behaviors.
This section will outline some of the more common ones.


## Data Manipulation

### Forms

The form behavior is one of the most common behaviors for interactive
applications. The form here will be a simple behavior that you can build on and
customize for your specific case:

```javascript
var FormBehavior = Marionette.Behavior.extend({
  defaults: {
    formSelector: 'form',
    submitSelector: '.submit-form'
  },

  ui: {
    return {
      form: this.getOption('formSelector'),
      submit: this.getOption('submitSelector')
    };
  },

  events: {
    'submit @ui.form': 'saveForm',
    'click @ui.submit': 'saveForm'
  },

  saveForm: function() {
    this.view.triggerMethod('before:form:save', this.view.model);
    this.view.model.save();
    this.view.triggerMethod('form:save', this.view.model);
  }
})
```

We've provided `before:form:save` and `form:save` handlers that the view can
listen for to provide extra hooks. Some possible ways to extend this behavior
could be to add field selectors that could be used to automatically bind the
field data to the model before saving it.


## Bootstrap

A number of Bootstrap's JavaScript widgets involve repetitive rendering tasks
that can be handled using behaviors.


### Modal

The [Bootstrap CSS framework][bootstrap] provides a modal dialog that we can use
for confirmation boxes, extra information, or even pop-up forms. This behavior
wraps a view's region in a modal that can be rendered anywhere, as well as
providing the handlers to tear it back down again.

```javascript
var ModalBehavior = Marionette.Behavior.extend({
  defaults: {
    modalClasses: '',
    modalOptions: null
  },

  ui: {
    close: '.close-modal'
  },

  events: {
    'hidden.bs.modal': 'triggerFinish',
  },

  triggers: {
    'click @ui.close': 'close:modal'
  },

  onRender: function() {
    this.view.$el.addClass('modal ' + this.getOption('modalClasses'));
  },

  onAttach: function() {
    this.view.$el.modal(this.getOption('modalOptions') || {});
  },

  onCloseModal: function() {
    this.view.$el.modal('hide');
  },

  triggerFinish: function() {
    this.view.triggerMethod('destroy:modal');
  }
});
```

We provide a `destroy:modal` handler that a parent view can listen to so it
knows when it's safe to empty the region. For example:

```javascript
var ModalView = Marionette.LayoutView.extend({
  behaviors: {
    modal: {
      behaviorClass: ModalBehavior
    }
  },
  template: require('./templates/view_with_modal.html')
});


var Parent = Marionette.LayoutView.extend({
  regions: {
    modalRegion: '.modal-hook'
  },

  onShow: function() {
    this.showChildView('modalRegion', new ModalView());
  },

  onChildviewDestroyModal: function() {
    this.getRegion('modalRegion').empty();
  }
})
```


### Tooltip

The tooltip behavior is a lot easier than the modal, as we'll see:

```javascript
var TooltipBehavior = Marionette.Behavior.extend({
  defaults: {
    tooltipSelector: '.has-tooltip'
  },

  ui: function() {
    return {
      tooltip: this.getOption('tooltipSelector');
    }
  },

  onRender: function() {
    this.ui.tooltip.tooltip();
  }
})
```


[bootstrap]: http://getbootstrap.com
