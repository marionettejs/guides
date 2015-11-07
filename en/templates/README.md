# Working with Templates

After we've constructed [views][views] from [models and collections][models], we
need to show this to our users. This section covers how to render dynamic data
and the different approaches we can take using Marionette to to this.

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
well-defined* cases. The cases are:

* When the output comes from a trusted source
* When the output has been processed by a [template helper][helpers]

In each of these cases, the output to be printed must contain HTML markup that
you want to render - otherwise we should just use `<%- %>` and get the same
effect.

#### Accessing undefined fields

Some template engines (such as Django templates) allow us to attempt to display
variables that haven't been defined - rendering an empty string. Underscore
templates don't have this property; attempting to access an undefined variable
causes an error.

For example, take the following model:

```javascript
{
  heading: 'Scott'
}
```

The following template can't render and will raise an error instead:

```html
<h1><%- heading %></h1>
<%- user_data %>
```


### Introducing JavaScript logic

Underscore templates allow us to execute JavaScript inline with the template
using `<% %>`. With this syntax we can conditionally display data or iterate
through an array.

Let's take the following models:

```javascript
{
  heading: 'Some header',
  data: [
    'string 1',
    'string 2'
  ]
}
```

and an empty model:

```javascript
{
  heading: null,
  data: []
}
```

```html
<% if (heading === null) { %>
<h1>New Item</h1>
<% } else { %>
<h1><%- heading %></h1>
<% } %>
```

For our populated model, the following is output:

```html
<h1>Some header</h1>
```

and our empty model outputs:

```html
<h1>New Item</h1>
```

We get access to the `_` namespace in our templates too, making it easier to
iterate:

```html
<ul>
<% _.each(data, function(item) { %>
  <li><%- item %></li>
<% }) %>
</ul>
```

Again, with our populated model, the output is:

```html
<ul>
  <li>string 1</li>
  <li>string 2</li>
</ul>
```

while our empty model simply looks like:

```html
<ul>
</ul>
```


## Template Helpers

As we can see, Underscore's template language is pretty basic and gives us
very little to work with. For example, setting variables or custom logic isn't
well supported, and attempting to access an undefined variable causes the entire
template to not render. In addition, if we want to start introducing some
complex logic, the template itself will quickly become unwieldy. We can mitigate
some of this complexity using [views and layouts][views] but, when we simply
want to validate data or format it, we turn to the `templateHelpers` attribute
on our view.

The `templateHelpers` attribute lets us assign an object - or function returning
and object - whose keys will be available in the template. Let's build a simple
template helper that outputs some information about a basket.

Our desired template is:

```html
<p>
  Your basket total is <%- toCurrency(total) %> and contains
  <%- count(items) %> items.
</p>
```

and our model is:

```javascript
{
  items: [
    {description: "An item", cost: "30.00"},
    {description: "Another item", cost: "5.00"}
  ],
  total: "35.00"
}
```

The output we'd want to get would be:

```html
<p>
  Your basket total is £35.00 and contains 2 items.
</p>
```

Let's define a simple template helper on our view:

```javascript
var BasketView = Marionette.LayoutView.extend({
  template: require('./basket.html'),

  templateHelpers: {
    count: function(items) {
      return items.length;
    },
    toCurrency: function(total) {
      var totalFloat = parseFloat(total);
      return '£' + totalFloat.toFixed(2);
    }
  }
});
```

Now, our template can see the keys `count` and `toCurrency` and can call these
functions directly. Another key advantage of this method is reusability - it's
simple to just import a function call and attach it to our `templateHelpers` as
such:

```javascript
var currencyFormatter = require('./helpers/currency');
var arrayHelpers = require('./helpers/array');

var BasketView = Marionette.LayoutView.extend({
  template: require('./basket.html'),

  templateHelpers: {
    count: arrayHelpers.count,
    toCurrency: currencyFormatter
  }
});
```

### Returning a function

The `templateHelpers` attribute can also take a function and call it for you.
This is especially useful in the case where we have potentially undefined
variables on our model:

```javascript
var currencyFormatter = require('./helpers/currency');

var BasketView = Marionette.LayoutView.extend({
  template: require('./basket.html'),

  templateHelpers: function() {
    var items = this.model.get('items');
    var total = this.model.get('total');

    var count = _.isUndefined(items) ? 0 : items.length;
    var cleanTotal = _.isUndefined(total) ? '0.00' : total;

    return {
      count: count,
      totalAsCurrency: currencyFormatter(cleanTotal)
    };
  }
});
```

We can modify our template slightly to look like:

```html
<p>
  Your basket total is <%- totalAsCurrency %> and contains
  <%- count %> items.
</p>
```

as our view has pre-processed the values for us.


[helpers]: #template-helpers
[models]: ../persisted_data/README.md
[views]: ../views/README.md
[underscore]: http://underscorejs.org/
