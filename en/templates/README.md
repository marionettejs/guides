# Working with Templates

After we've constructed [views][views] from [models and collections][models], we
need to show this to our users. This section covers how to render dynamic data
and the different approaches you can take using Marionette to to this.

## A simple template

With a `LayoutView` we can render a template quite easily:

```javascript
var MyLayout = Marionette.LayoutView.extend({
  template: require('mylayouttemplate.html')
});
```

With our template `mylayouttemplate.html` as something like:

```html
<h1>Hello, world</h1>

<p>We have something to talk about</p>
```

When we `show` or `render` our view, we can see the template that we've just
created. This isn't very interesting, however, so let's take it a step further
using some model data:

```javascript
var mymodel = new Backbone.Model({
  name: 'Scott'
});
var layout = new MyLayout({model: mymodel});
layout.render();
```

Let's change our template before compiling this:

```html
<h1>Hello, <%- name %></h1>
<p>I now know your name</p>
```

Our page is now a little more interesting - we can change it based on the model
data we pass into our view. With just this knowledge we can, and will, build
plenty of complex web applications.

## Template syntax

This syntax comes from [Underscore's][underscore] template engine; an extremely
simple template engine that does most of what we need. We'll quickly go
through its syntax here.

### Output data

There are two basic ways to output data in an underscore template: escaped and
unescaped.

#### Escaped data

The default syntax we'll use is the familiar `<%- %>` syntax from above. This
will cause Underscore to escape HTML strings to make them safe. This should be
your default goto, especially for user-entered data:

```html
<h1><%- heading %></h1>
<p><%- user_data %></p>
```

Let's say our user has the following model:

```javascript
{
  heading: 'Some text',
  user_data: '<script>alert("test")</script>'
}
```

Once rendered, our HTML would be:

```html
<h1>Some text</h1>
<p>&lt;script&gt;alert("test")&lt;/script&gt;</p>
```

The escaping has protected us from potential cross-site scripting (XSS) attacks.

#### Unescaped data

On occasion, we need to output data exactly as it is entered into our model.
For example, we may have pre-escaped our output, or we explicitly want to output
HTML that the browser will render (for example, rendered Markdown).

In this case, we can tell Underscore to output our data as-is with `<%= %>`.

```html
<h1><%- heading %></h1>
<%= user_data %>
```

With the same model as above, the raw output will be:

```html
<h1>Some text</h1>
<script>alert("test")</script>
```

Causing the `alert` code to execute.


#### When to escape and when not to

By default, we should prefer `<%- %>` escaping over `<%= %>` except in *rare,
well-defined* cases.


[models]: ../persisted_data/README.md
[views]: ../views/README.md
[underscore]: http://underscorejs.org/
