# Todo App Code

The code for the [Todo app tutorial][todotutorial] can be found before as a
comparison with your own application. This code is for demonstration purposes -
it is not necessarily the best way to build such applications and it certainly
isn't the only way.


## Driver file

`driver.js`:

```js
var Marionette = require('backbone.marionette');
var TodoView = require('./views/layout');

var initialData = {
  items: [
    {assignee: 'Scott', text: 'Write a book about Marionette'},
    {assignee: 'Andrew': text: 'Do some coding'}
  ]
};

var App = new Marionette.Application({
  onStart: function(options) {
    var todo = new TodoView(options);
    todo.render();
    todo.triggerMethod('show');
  }
});

var app = new App();
app.start({initialData: initialData});
```

## Views

Everything in this section is contained in the `views/` directory.

`layout.js`:

```js
var Backbone = require('backbone');
var Marionette = require('backbone.marionette');
var ToDoModel = require('../models/todo');

var FormView = require('./form');
var ListView = require('./list');


var Layout = Marionette.LayoutView.extend({
  el: '#app-hook',

  template: require('../templates/layout.html'),

  regions: {
    form: '.form',
    list: '.list'
  },

  collectionEvents: {
    add: 'itemAdded'
  },

  initialize: function() {
    this.collection = new Backbone.Collection([
      {assignee: 'Scott', text: 'Write a book about Marionette'},
      {assignee: 'Andrew': text: 'Do some coding'}
    ]);
    this.model = new ToDoModel();
  },

  onShow: function() {
    var formView = new FormView({model: this.model});
    var listView = new ListView({collection: this.collection});

    this.showChildView('form', formView);
    this.showChildView('list', listView);
  },

  onChildviewAddTodoItem: function() {
    this.model.set({
      assignee: this.ui.assignee.val(),
      text: this.ui.text.val()
    }, {validate: true});

    var items = this.model.pick('assignee', 'text');
    this.collection.add(items);
  },

  itemAdded: function() {
    this.model.set({
      assignee: '',
      text: ''
    });
  }
})
```

`form.js`:

```js
// views/form.js
var Marionette = require('backbone.marionette');


var FormView = Marionette.LayoutView.extend({
  tagName: 'form',
  template: require('../templates/form.html'),

  triggers: {
    submit: 'add:todo:item'
  },

  modelEvents: {
    change: 'render'
  },

  ui: {
    assignee: '#id_assignee',
    text: '#id_text'
  }
});


module.exports = FormView;
```

`list.js`:

```js
// views/list.js
var Marionette = require('backbone.marionette');

var ToDo = Marionette.LayoutView.extend({
  tagName: 'li',
  template: '../templates/todoitem.html'
});


var TodoList = Marionette.CollectionView.extend({
  tagName: 'ul',
  childView: ToDo
});


module.exports = TodoList;
```

## Templates

Everything here is stored in the `templates` directory.

`layout.html`:

```html
<div class="list"></div>
<div class="form"></div>
```

`form.html`:

```html
<label for="id_text">Todo Text</label>
<input type="text" name="text" id="id_text" value="<%- text %>" />
<label for="id_assignee">Assign to</label>
<input type="text" name="assignee" id="id_assignee" value="<%- assignee %>"/>

<button id="btn-add">Add Item</button>
```

`todoitem.html`:

```html
<%- item.text %> &mdash; <%- item.assignee %>
```

## Models

Everything here is under the `models` directory:

`todo.js`

```js
var Backbone = require('backbone');


var ToDo = Backbone.Model.extend({
  defaults: {
    assignee: '',
    text: ''
  },

  validate: function(attrs) {
    var errors = {};
    var hasError = false;
    if (!attrs.assignee) {
      errors.assignee = 'assignee must be set';
      hasError = true;
    }
    if (!attrs.text) {
      errors.text = 'text must be set';
      hasError = true;
    }

    if (hasError) {
      return errors;
    }
  }
});


module.exports = ToDo;
```

[todotutorial]:(../../../getting_started/tutorial/introduction.md)
