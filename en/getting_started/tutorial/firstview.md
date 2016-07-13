# Hello, world

We're going to start by building a very simple view that simply displays some
text to the world.


## Creating our view

Let's get started! Open up `driver.js` and enter the following:

```js
var Marionette = require('backbone.marionette');  // 1


var HelloWorld = Marionette.LayoutView.extend({  // 2
  el: '#app-hook',  // 3
  template: require('./templates/layout.html')  // 4
});

var hello = new HelloWorld();  // 5

hello.render();  // 6
```

We then create a file in `templates` called `layout.html` and set it up as such:

```html
<p>Hello, world!</p>
```


### What does this all mean?

The template file itself is pretty straightforward, so let's focus on
`driver.js`:

  1. Import Marionette
  2. Create a new type of view called `HelloWorld` that borrows from the
    standard Marionette LayoutView. We'll go into more depth in that shortly.
  3. We direct the view to the element we want to attach it to. This is a
    jQuery selector and we can use any valid jQuery selector here.
  4. We must set a template to display to our users.
  5. We must create an instance of our `HelloWorld` class before we can do
    anything useful with it.
  6. Now the fun stuff begins and we call `render()` to display the template
    on the screen.


### Running it all

After [compiling the file][installing], navigate to `index.html` in your browser
to see it render on your screen. You've just built your first JavaScript
application with Marionette!


## Building a ToDo application

Now that we have a view, let's do something a little more interesting and render
a pre-defined list of ToDo items. We'll need a to use a
[`Backbone Model`][models] to store the list in a way that our `LayoutView` can
see it. Reopen our `driver.js` file:

```js
var Backbone = require('backbone');
var Marionette = require('backbone.marionette');


var TodoList = Marionette.LayoutView.extend({
  el: '#app-hook',
  template: require('./templates/layout.html')
});

var todo = new TodoList({
  model: new Backbone.Model({
    items: [
      {assignee: 'Scott', text: 'Write a book about Marionette'},
      {assignee: 'Andrew', text: 'Do some coding'}
    ]
  })
});

todo.render();
```


The major point of note is that we're wrapping our object in a `Backbone.Model`
instance. Backbone models cleanly integrate with templates and make it easy to
access their data. We'll see in a few chapters how powerful models can be for
our views.

Next we'll modify our `templates/layout.html` file to take advantage of this:

```js
<ul>
  <% _.each(items, function(item) { %>
    <li><%- item.text %> &mdash; <%- item.assignee %></li>
  <% }) %>
</ul>
```

We're using the built-in [Underscore template engine][underscore] to render our
item list. You should have a list of two items detailing simple todo items. We
now have a list of items being rendered on the page but it all feels a little
clunky. What happens if we want to add or remove items in this list?


## Collections and CollectionViews

Our list of items is really a collection, so we'll use the `Backbone.Collection`
to model it. We'll also use a view that specializes in rendering lists of data -
the `CollectionView`. Back in our `driver.js` file, we're going to use a couple
of views:

```js
var Backbone = require('backbone');
var Marionette = require('backbone.marionette');


var ToDo = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require('./templates/todoitem.html')
});


var TodoList = Marionette.CollectionView.extend({
  el: '#app-hook',
  tagName: 'ul',

  childView: ToDo
});

var todo = new TodoList({
  collection: new Backbone.Collection([
    {assignee: 'Scott', text: 'Write a book about Marionette'},
    {assignee: 'Andrew', text: 'Do some coding'}
  ])
});

todo.render();
```


The first thing we've done is add another view and attached it to our
`CollectionView` using the `childView` attribute. We also changed out
`this.model` for `this.collection` in the `initialize` method of our newly
minted `CollectionView` and removed the wrapping `items` key.

The `CollectionView` is a view that goes through `this.collection` and renders
an instance of its `childView` attribute for each item that it finds. We've
removed the `template` attribute as `CollectionView` has no template of its own.

In `templates/todoitem.html` we simply have:

```html
<%- text %> &mdash; <%- assignee %>
```


That's it! Marionette knows how to build up the surrounding `ul` and `li` tags
and inserts the iteration for us. Even better, if we wanted to add/remove items
in the collection, `CollectionView` sees that and automatically re-renders
itself for us.

In the next chapter, we're going to add new items to this collection so we can
keep track of our job list.

[installing]: ../installing_marionette.md "Installing Marionette"
[models]: ./models.md "Storing user-entered data"
[underscore]: http://underscorejs.org/#template "Underscore.js"
