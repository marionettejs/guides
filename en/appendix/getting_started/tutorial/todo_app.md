# Todo App Code

The code for the [Todo app tutorial][todotutorial] can be found below as a
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
    {assignee: 'Andrew', text: 'Do some coding'}
  ]
};


var App = new Marionette.Application({
  onStart: function(options) {
    var todo = new TodoView({
      collection: new Backbone.Collection(options.initialData.items),
      model: new ToDoModel()
    });
    todo.render();
    todo.triggerMethod('show');
  }
});

App.start({initialData: initialData});
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

  onShow: function() {
    var formView = new FormView({model: this.model});
    var listView = new ListView({collection: this.collection});

    this.showChildView('form', formView);
    this.showChildView('list', listView);
  },

  onChildviewAddTodoItem: function(child) {
    this.model.set({
      assignee: child.ui.assignee.val(),
      text: child.ui.text.val()
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
});

module.exports = Layout;
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
  template: require('../templates/todoitem.html')
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

# Todo App Code - ES6 Version

The following code will load in latest version of Google Chrome with Marionette 2.4.7 without the need to compile back to ES5. Older browsers may require compiling with Babel or similar.

## Driver file

`driver.js`:

```js
import Backbone from 'backbone';
import Marionette from 'backbone.marionette';

import TodoView from './views/layout';
import TodoModel from './models/todo';

const initialData = [
  {assignee: 'Scott', text: 'Write a book about Marionette'},
  {assignee: 'Andrew', text: 'Do some coding'}
];

export class TodoApp extends Marionette.Application
{
  onStart(options)
  {
    const todoView = new TodoView({
      collection: new Backbone.Collection(options.initialData),
      model: new TodoModel()
    });
    todoView.render();
  }
}

window.app = new TodoApp;
window.app.start({initialData: initialData});
```

## Views

Everything in this section is contained in the `views/` directory.

`layout.js`:

```js
import Marionette from 'backbone.marionette';

import FormView from './form';
import ListView from './list';
import LayoutTemplate from '../templates/layout.html';

export default class TodoView extends Marionette.LayoutView
{
  constructor(options)
  {
    options.template = LayoutTemplate;
    options.el = '#app-hook';
    options.regions = {
      form: '.form',
      list: '.list'
    };

    super(options);
  }

  onRender()
  {
    const formView = new FormView({model: this.model});
    const listView = new ListView({collection: this.collection});
    this.showChildView('form', formView);
    this.showChildView('list', listView);
  }

  collectionEvents()
  {
    return { add: 'itemAdded' };
  }

  onChildviewAddTodoItem(child)
  {
    this.model.set({
      assignee: child.ui.assignee.val(),
      text: child.ui.text.val()
    }, {validate: true});

    if (this.model.isValid()) {
      const items = this.model.pick('assignee', 'text');
      this.collection.add(items);
    }
  }

  itemAdded()
  {
    this.model.set({
      assignee: '',
      text: ''
    });
  }
}
```

`form.js`:

```js
import Marionette from 'backbone.marionette';
import FormTemplate from '../templates/form.html';

export default class FormView extends Marionette.LayoutView
{
  constructor(options)
  {
    options.template = FormTemplate;
    options.tagName = 'form';

    super(options);
  }

  triggers()
  {
    return {
      submit: 'add:todo:item'
    };
  }

  modelEvents()
  {
    return {
      change: 'render'
    };
  }

  ui()
  {
    return {
      assignee: '#id_assignee',
      text: '#id_text'
    };
  }
}
```

`list.js`:

```js
import Marionette from 'backbone.marionette';
import TodoItemTemplate from '../templates/todoitem.html';

class Todo extends Marionette.LayoutView
{
  constructor(options)
  {
    options.template = TodoItemTemplate;
    options.tagName = 'li';

    super(options);
  }
}

export default class ListView extends Marionette.CollectionView
{
  constructor(options)
  {
    options.tagName = 'ul';
    options.childView = Todo;

    super(options);
  }
}
```

## Templates

Everything here is stored in the `templates` directory. All template files are the same as the ES5 version.

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
import Backbone from 'backbone';

export default class TodoModel extends Backbone.Model
{
  defaults()
  {
    return {
      assignee: '',
      text: ''
    };
  }

  validate(attrs)
  {
    const errors = {};
    let hasError = false;
    if (!attrs.assignee)
    {
      errors.assignee = 'assignee must be set';
      hasError = true;
    }
    if (!attrs.text)
    {
      errors.text = 'text must be set';
      hasError = true;
    }
    if (hasError) return errors;
  }
}
```

[todotutorial]: ../../../getting_started/tutorial/README.md
