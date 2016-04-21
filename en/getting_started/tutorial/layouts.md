# Applications and regions

In the previous chapter we built our ToDo list application with the ability to
add new jobs to our list. So far, everything has been built into a single
view. This led to an issue causing everything to get re-rendered whenever we
added an item to our list.

The `CompositeView` itself knows how to only add a single item to its list, but
our `change` hook was causing everything to be re-rendered anyway. In this
chapter we're going to dig into the `LayoutView` a little more and how we can
use it to display multiple views side-by-side and manage them independently.


## Applications

Before we do that though, we'll take a slight diversion into the `Application`
in Marionette. The Application is an object that lets us manage our ToDo list
and its interaction with the surrounding page.


### Creating an Application

We'll be using the layout [described in the introduction][introduction], so
let's move the bulk of our view code into `views/layout.js` and rejig it a
little:

```js
var Backbone = require('backbone');
var Marionette = require('backbone.marionette');

var ToDoModel = require('../models/todo');


var ToDo = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require('../templates/todoitem.html')
});


var TodoList = Marionette.CompositeView.extend({
  el: '#app-hook',
  template: require('../templates/todolist.html'),

  childView: ToDo,
  childViewContainer: 'ul',

  ui: {
    assignee: '#id_assignee',
    form: 'form',
    text: '#id_text'
  },

  triggers: {
    'submit @ui.form': 'add:todo:item'
  },

  collectionEvents: {
    add: 'itemAdded'
  },

  modelEvents: {
    change: 'render'
  },

  onAddTodoItem: function() {
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
});


module.exports = TodoList;
```


We now need a way to render this view when our application loads. Marionette
gives us the `Application` class for just this case. An `Application` sits
between your pre-rendered page and your application. Commons tasks for an
`Application` are:

  1. Take pre-defined data from your page and feed it into your application
  2. Render your initial views
  3. Start the `Backbone.history` and initialize your application's `Router`
    (more on this later)

Let's put this knowledge into practice by rewriting our `driver.js` file to look
like:

```js
var Marionette = require('backbone.marionette');
var TodoView = require('./views/layout');

var initialData = [
  {assignee: 'Scott', text: 'Write a book about Marionette'},
  {assignee: 'Andrew', text: 'Do some coding'}
];

var app = new Marionette.Application({
  onStart: function(options) {
    var todo = new TodoView({
      collection: new Backbone.Collection(options.initialData),
      model: new ToDoModel()
    });
    todo.render();
    todo.triggerMethod('show');
  }
});

app.start({initialData: initialData});
```


With an Application object, we now have an obvious starting point for our
application. We've passed our initial data in from the `driver.js` file instead
of the individual view file itself. Now, if our `index.html` file gets generated
by a web server e.g. Django, Rails, PHP; we can generate a different list for
each user, attach it to an object inside `index.html` and reference it from our
JavaScript application.


## Layouts

With that little diversion out the way, we can now start breaking up our
application's layout so we have different views for different purposes. The
first thing we need to do is add an extra layout. First, we're going to break up
our existing `views/layout.js` into `views/list.js` and `views/form.js` which
look like:

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


You can see straight away how much simpler these views are. They only deal with
their own data management and are completely unaware of each other. We've also
turned the list back into a `CollectionView` because we no longer need to attach
a model or a form template. We can safely remove the `todolist.html` template.
The `todoitem.html` template is unchanged but we have a new `form.html`
template:

```html
<label for="id_text">Todo Text</label>
<input type="text" name="text" id="id_text" value="<%- text %>" />
<label for="id_assignee">Assign to</label>
<input type="text" name="assignee" id="id_assignee" value="<%- assignee %>"/>

<button id="btn-add">Add Item</button>
```


We no longer need the wrapping form because the `LayoutView` will generate that
for us.


Our `views/layout.js` file now handles the management of the two separate views:

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


The major changes here are that some of the form rendering logic is pushed into
the form view itself, while logic that links the form to the list is kept in
this `LayoutView`. We also have an `onShow` handler that renders the views into
the `jQuery` selectors referenced by the `regions` hash. Finally, a `LayoutView`
can see events occurring on its children by prepending its event handler with
`Childview` as in `onChildviewAddTodoItem`. The `Childview` triggers always send
a copy of their view as the first argument, allowing us to see the `ui` object.

The individual views don't directly interact with each other, instead
interacting with the model and letting the view event handlers recognize when
they need to do something.

Finally, this layout solves the issue identified before. Now only the form
itself will be re-rendered when data changes. The list is able to manage the
individual items being attached and render only what needs to be rendered.

For completeness, the `layout.html` template is detailed below:

```html
<div class="list"></div>
<div class="form"></div>
```


As a template, the layout has been relegated to just an overarching frame that
delegates most of its rendering responsibilities to its subordinate views.

[introduction]: ./README.md "Marionette introduction"
