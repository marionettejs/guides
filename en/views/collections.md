# Lists of Data

A key component of views is being able to handle lists (or collections) of data.
The `CollectionView` and `CompositeView` are specifically designed to deal with
these lists by binding to the `Backbone.Collection`.

When you bind a collection to your collection view, it will iterate over each
item in the list, rendering a new view for each one. Let's start with an example
`CollectionView` with our items using a `item.html` template called 'list.js':

```html
<a href="<%- url %>"><%- text %></a>
```

```javascript
var Item = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require('./item.html')
});

var List = Marionette.CollectionView.extend({
  tagName: 'ul',
  childView: Item
});

module.exports = List;
```

Now let's create an instance of this with our collection:

```javascript
var List = require('./list');

var collection = new Backbone.Collection([
  {text: 'Some Text', url: '/items/1'},
  {text: 'Some other text', url: '/items/4'}
]);

var view = new List({
  collection: collection
});
view.render();
```

The output from this will look like:

```html
<ul>
  <li><a href="/items/1">Some Text</a></li>
  <li><a href="/items/2">Some other text</a></li>
</ul>
```

If we want to add an item to this list, we simply do:

```javascript
collection.add({
  text: 'New Item',
  url: '/items/8'
});
```

The `CollectionView` will recognize the item has been added and inject the item
into the HTML template at the right location.


### Events on Collections

Just like we can bind `modelEvents` we can also bind `collectionEvents` to our
views. The full list of events can be found in the
[Backbone documentation][backbone]. This includes events for `add` and `remove`
(which is what Marionette listens to internally). Let's see an example:

```javascript
var List = Marionette.CollectionView.extend({
  tagName: 'ul',
  childView: Item,

  collectionEvents: {
    add: 'itemAdded'
  },

  itemAdded: function(collection) {
    alert('New item added');
  }
});
```

Now, whenever an item gets added, no matter how, an alert box will be displayed
for each one.


## Rendering Tables

You'll notice the `CollectionView` doesn't define its own template. This makes
it unsuitable for more complex listed layouts such as tables. To solve this
problem, we have the `CompositeView` - a view that lets us assign a template to
be rendered.

Let's take two templates making up a table: `row.html` and `table.html`

```html
<thead>
  <tr>
    <th>Name</th>
    <th>Nationality</th>
    <th>Gender</th>
  </tr>
</thead>
<tbody></tbody>
```

```html
<td><%- name %></td>
<td><%- nationality %></td>
<td><%- gender %></td>
```

We'll now use the `CompositeView` to build this:

```javascript
var Row = Marionette.LayoutView.extend({
  tagName: 'tr',
  template: require('row.html')
});

var Table = Marionette.CompositeView.extend({
  tagName: 'table',
  template: require('table.html'),

  childView: Row,
  childViewContainer: 'tbody'
});

module.exports = Table;
```

The `CompositeView` takes two extra required attributes: the familiar `template`
and `childViewContainer` - a jQuery selector to the element in our template to
attach the `childView` elements. Using the `CompositeView` is identical to the
`CollectionView`:

```javascript
var Table = require('./table');

var collection = new Backbone.Collection([
  {name: 'John Smith', gender: 'male', nationality: 'UK', url: '/items/1'},
  {name: 'Jane Doe', gender: 'female', nationality: 'USA', url: '/items/4'}
]);

var view = new Table({
  collection: collection
});
view.render();
```

Adding and removing items works exactly as you'd expect - the new views are
injected at the correct locations in the template.


## Binding Models

The `CompositeView` has another advantage over `CollectionView` - it can take
and additional `model` argument and render based on the contents of the model.
Let's say we wanted to know how many people were in the final list above, we'll
modify our `table.html`:

```html
<thead>
  <tr>
    <th>Name</th>
    <th>Nationality</th>
    <th>Gender</th>
  </tr>
</thead>
<tbody></tbody>
<tfoot>
  <tr>
    <th>Total</th>
    <td colspan="2"><%- total %></td>
  </tr>
</tfoot>
```

This total needs to come from a model which we'll pass when creating the table:

```javascript
var Table = require('./table');

var collection = new Backbone.Collection([
  {name: 'John Smith', gender: 'male', nationality: 'UK', url: '/items/1'},
  {name: 'Jane Doe', gender: 'female', nationality: 'USA', url: '/items/4'}
]);

var model = new Backbone.Model({
  total: 30
});

var view = new Table({
  collection: collection,
  model: model
});
view.render();
```

And that's it, our table now has access to the `total` field in the model!


### Events

Our CompositeView can now listen to both `collectionEvents` and `modelEvents` at
the same time. One good use for this is a report table with a summary that is
calculated and fetched separately:

```javascript
var Table = Marionette.CompositeView.extend({
  tagName: 'table',
  template: require('table.html'),

  childView: Row,
  childViewContainer: 'tbody',

  modelEvents: {
    sync: 'render'
  },

  collectionEvents: {
    update: 'checkStatus'
  },

  checkStatus: function(collection) {
    collection.each(function(model) {
      // Do something
    });
  }
});
```

With this example, when our model is fetched from the server, we'll re-render
the table. When our collection is modified, we run another handler that, in this
case, does some form of checking/modification for the collection.


[backbone]: http://backbonejs.org/#Events-catalog
