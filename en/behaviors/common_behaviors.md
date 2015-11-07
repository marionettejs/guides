# Common Behavior Patterns

There are a number of design patterns that can be made simpler with behaviors.
This section will outline some of the more common ones.


## Bootstrap Modal

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


[bootstrap]: http://getbootstrap.com
