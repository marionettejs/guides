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
  template: './templates/todoitem.html'
});


var TodoList = Marionette.CompositeView.extend({  
  el: '#app-hook',
  template: require('./templates/todolist.html'),

  childView: ToDo,
  childViewContainer: 'ul',

  initialize: function() {
    this.collection = new Backbone.Collection([
      {assignee: 'Scott', text: 'Write a book about Marionette'},
      {assignee: 'Andrew': text: 'Do some coding'}
    ]);
  }
});

var todo = new TodoList();
todo.render();
```


A `CompositeView` is a `CollectionView` with its own template. We then define
where the `childView` items are to be attached to the template using
`childViewContainer` and pass in the template. In `templates/todolist.html` we
have:

```
<ul></ul>
<form>
  <input type="text" name="text" id="id_text" />
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
  template: './templates/todoitem.html'
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

  initialize: function() {
    this.collection = new Backbone.Collection([
      {assignee: 'Scott', text: 'Write a book about Marionette'},
      {assignee: 'Andrew': text: 'Do some coding'}
    ]);
  },

  onAddTodoItem: function() {  // 3
    this.collection.add({
      assignee: this.ui.assignee.val(),  // 4
      text: this.ui.text.val()
    });
    this.ui.assignee.val('');
    this.ui.text.val('');
  }
});

var todo = new TodoList();
todo.render();
```


We've added quite a bit of code here, so lets take a few minutes to break it
down:

  1. We've added a "ui hash" to our view. We can attach these to any view to
    create cached jQuery selectors to elements in our view's template.
  2. In the triggers hash, we can reference those ui keys and, when a jQuery
    event occurs, we can listen for it and fire a trigger.
  3. This trigger is then converted to an `onEventName` method and called. This
    method need not exist and is very powerful. We'll cover it in more depth
    later in the book.
  4. We can also reference the ui hash inside our view and treat it just like a
    jQuery selector object.


Now, whenever we click on the "Add Item" button, a new job will be added to our
todo list and the form will be cleared.
