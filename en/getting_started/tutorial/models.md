# Storing user-entered data

We've just built a simple list of jobs that we need to complete but, as always,
we want to add more jobs to our list as they come in. In this chapter, we'll
build that functionality using some more of Marionette's functionality.


## Building our form

First things first, we need a form to enter data. To make this work in our
current application, we can swap out `CollectionView` for `CompositeView` in our
`driver.js` file like so:

```js
var Backbone = require('backbone');
var Marionette = require('backbone.marionette');


var ToDo = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require('./templates/todoitem.html')
});


var TodoList = Marionette.CompositeView.extend({
  el: '#app-hook',
  template: require('./templates/todolist.html'),

  childView: ToDo,
  childViewContainer: 'ul'
});

var todo = new TodoList({
  collection: new Backbone.Collection([
    {assignee: 'Scott', text: 'Write a book about Marionette'},
    {assignee: 'Andrew', text: 'Do some coding'}
  ])
});

todo.render();
```


A `CompositeView` is a `CollectionView` with its own template. We then define
where the `childView` items are to be attached to the template using
`childViewContainer` and pass in the template. In `templates/todolist.html` we
have:

```html
<ul></ul>
<form>
  <label for="id_text">Todo Text</label>
  <input type="text" name="text" id="id_text" />
  <label for="id_assignee">Assign to</label>
  <input type="text" name="assignee" id="id_assignee" />

  <button id="btn-add">Add Item</button>
</form>
```


We don't need to change the individual item template at all. Now when we refresh
the page, we'll have a form where we can enter data, click "Add Item" and...
nothing happens. Marionette doesn't know what elements to watch and what not to
watch, so we have to tell it.


## Binding to user input

We need to tell Marionette that it needs to listen to user input on the form and
how it should respond to that. We'll reopen `driver.js` and start building this
in:

```js
var Backbone = require('backbone');
var Marionette = require('backbone.marionette');


var ToDo = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require('./templates/todoitem.html')
});


var TodoList = Marionette.CompositeView.extend({
  el: '#app-hook',
  template: require('./templates/todolist.html'),

  childView: ToDo,
  childViewContainer: 'ul',

  ui: {  // 1
    assignee: '#id_assignee',
    form: 'form',
    text: '#id_text'
  },

  triggers: {  // 2
    'submit @ui.form': 'add:todo:item'
  },

  collectionEvents: {  // 3
    add: 'itemAdded'
  },

  onAddTodoItem: function() {  // 4
    this.collection.add({
      assignee: this.ui.assignee.val(),  // 5
      text: this.ui.text.val()
    });
  },

  itemAdded: function() {  // 6
    this.ui.assignee.val('');
    this.ui.text.val('');
  }
});

var todo = new TodoList({
  collection: new Backbone.Collection([
    {assignee: 'Scott', text: 'Write a book about Marionette'},
    {assignee: 'Andrew', text: 'Do some coding'}
  ])
});
todo.render();
```


We've added quite a bit of code here, so lets take a few minutes to break it
down:

  1. We've added a "ui hash" to our view. We can attach these to any view to
    create cached jQuery selectors to elements in our view's template.
  2. In the triggers hash, we can reference those ui keys and, when a jQuery
    event occurs, we can listen for it and fire a trigger.
  3. The collectionEvents hash allows us to listen to changes occurring on the
    attached `this.collection` attribute. The value must exist as a method on
    this view.
  4. This trigger is then converted to an `onEventName` method and called. This
    method need not exist and is very powerful. We'll cover it in more depth
    later in the book.
  5. We can also reference the ui hash inside our view and treat it just like a
    jQuery selector object.
  6. The method referenced in `collectionEvents` is called when the event is
    triggered.


Now, whenever we click on the "Add Item" button, a new job will be added to our
todo list and the form will be cleared. We've taken an opportunity to introduce
model and collection driven events as well. For a full list of events, see the
[Backbone documentation][eventlist].


## Validating input

A job shouldn't be added to the list unless it has some text and has been
assigned to someone. You'll notice that we don't really enforce that here but we
really should. There are a couple of ways we could go about this: we could use
the ui hash and validate the jQuery content, or we could use Backbone's Model
validation to do it for us.

Apart from looking nicer, the advantage of doing data validation in the Model is
that we can share that model class between views and not have to rewrite all our
validation logic every time.

We'll create a new file called `models/todo.js` which contains:

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


If we have no errors, our `validate` method must return nothing (`undefined`).
When we do have errors, we can return an object describing the errors. The
Marionette view will then see this validation error and let us handle it.

Back in our `driver.js` file we can do:

```js
var Backbone = require('backbone');
var Marionette = require('backbone.marionette');

var ToDoModel = require('./models/todo');


var ToDo = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require('./templates/todoitem.html')
});


var TodoList = Marionette.CompositeView.extend({
  el: '#app-hook',
  template: require('./templates/todolist.html'),

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

    this.ui.assignee.val('');
    this.ui.text.val('');
  }
});

var todo = new TodoList({
  collection: new Backbone.Collection([
    {assignee: 'Scott', text: 'Write a book about Marionette'},
    {assignee: 'Andrew', text: 'Do some coding'}
  ]),
  model: new ToDoModel()
});

todo.render();
```


With these changes, we can now refuse to add an item unless it passes
validation. We could also display error messages if validation fails by binding
the `invalid` event in `modelEvents`. You can see how we've set our fields to
blank fields on the model and also in the ui hash. If we start having more and
more fields, you can see how this would quickly become unmanageable. Surely we
can just clear the form when the model is cleared?


## Rendering from models

Back in our first chapter, we were able to render data based on the model
fields. We'll use this to handle our form as well. Firstly we'll open up our
`todolist.html` template:

```html
<ul></ul>
<form>
  <label for="id_text">Todo Text</label>
  <input type="text" name="text" id="id_text" value="<%- text %>" />
  <label for="id_assignee">Assign to</label>
  <input type="text" name="assignee" id="id_assignee" value="<%- assignee %>"/>

  <button id="btn-add">Add Item</button>
</form>
```


By adding in the model field values to our form, we'll be able to render the
data directly from our model fields. It also means that, when the model fields
are blank, these will contain no data. We just need to wire up our view to
handle this:

```js
var Backbone = require('backbone');
var Marionette = require('backbone.marionette');

var ToDoModel = require('./models/todo');


var ToDo = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require('./templates/todoitem.html')
});


var TodoList = Marionette.CompositeView.extend({
  el: '#app-hook',
  template: require('./templates/todolist.html'),

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

var todo = new TodoList({
  collection: new Backbone.Collection([
    {assignee: 'Scott', text: 'Write a book about Marionette'},
    {assignee: 'Andrew', text: 'Do some coding'}
  ]),
  model: new ToDoModel()
});

todo.render();
```


With these simple changes, the form will now re-render itself as an empty form
whenever the user clicks the "Add Item" button. However, there's one final thing
to note - the `render` method redraws the entire list as well. You can probably
imagine this will start to get really slow as the list grows in size. Ideally,
we just want to re-render the form itself and handle the list separately. We'll
go into this in our next chapter.


[eventlist]: http://backbonejs.org/#Events-catalog "Backbone Events"
