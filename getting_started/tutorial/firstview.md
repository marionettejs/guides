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

```
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
  template: require('./templates/layout.html'),

  initialize: function() {
    this.model = new Backbone.Model({
      items: [
        {assignee: 'Scott', text: 'Write a book about Marionette'},
        {assignee: 'Andrew': text: 'Do some coding'}
      ]
    });
  }
});

var todo = new TodoList();
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
    <li><%- item.text %> &mdash; <%- item.assignee %>
  <% }) %>
</ul>
```

We're using the built-in [Underscore template engine][underscore] to render our
item list. You should have a list of two items detailing simple todo items.

It's perfectly possible to build complex applications with just a single view,
however you'll quickly run into limitations, the most pressing being how to only
redraw certain areas of your screen more easily.

With a single view, you'd become dependent on jQuery again, which we really
want to avoid if we're going to make our application easy to understand.

Our next section will deal with exactly that.


[installing]:[../installing_marionette.md]
[models]:[./models.md]
[underscore][./underscore.md]
