# Hello, world


We're going to start by building a very simple view that simply displays some
text to the world.

## Creating our view

It's time to create our first JavaScript-rendered view! Open up `driver.js` and
enter the following:

```js
require('backbone').$ = require('jquery');  // 1.

var Marionette = require('backbone.marionette');  // 2.


var HelloWorld = Marionette.LayoutView.extend({  // 3.
  el: '#app-hook',  // 4.
  template: require('./templates/layout.html')  // 5.
});

var hello = new HelloWorld();  // 6.
hello.render();  // 7.
```

We then create a file in `templates` called `layout.html` and set it up as such:

```html
<p>Hello, world!</p>
```

### What does this all mean?

The template file itself is pretty straightforward, so let's focus on
`driver.js`:

  1. This is necessary for Browserify, we just need to point Backbone at the
    correct jQuery reference.
  2. Import Marionette
  3. Create a new type of view called `HelloWorld` that borrows from the
    standard Marionette LayoutView. We'll go into more depth in that shortly.
  4. We direct the view to the element we want to attach it to. This is a
    jQuery selector and we can use any valid jQuery selector here.
  5. We must set a template to display to our users.
  6. We must create an instance of our `HelloWorld` class before we can do
    anything useful with it.
  7. Now the fun stuff begins and we call `render()` to display the template
    on the screen.

### Running it all

After compiling the file, navigate to `index.html` in your browser to see it
render on your screen. You've just built your first JavaScript application with
Marionette!
