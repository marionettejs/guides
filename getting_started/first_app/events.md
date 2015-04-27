# Responding to user input with events

Backbone implements a simple, but powerful, event framework that lets us listen
to and trigger events based on different conditions.

Marionette takes this a step further with triggers that let views easily listen
to and act on events. If you're not familiar with event handling, this tutorial
should give you a good idea of how it works in practice.

## A quick example

Let's start with an easy example to demonstrate how events work. Using the
final result from [collections and advanced views](./collections.md), we will
attach an event to each table row that will pop up an alert box. Open up
`personlist.js` and go to our `PersonView`:

```js
var PersonView = Marionette.LayoutView.extend({
  tagName: 'tr',
  template: require('./templates/person.html'),

  events: {
    'click': 'alertItem'
  },

  alertItem: function() {
    window.alert('You clicked on ' + this.model.get('first_name'));
  }
});
```

Build and refresh the page and whenever you click on an item, you should get an
alert box telling you what you clicked on.

## More advanced behavior with triggers

Events are good as they let you bind a function to a user action. However, it's
difficult to pass the information to other views. A better way to handle this
sort of behavior is a trigger. Let's have a look:

```js
var PersonView = Marionette.LayoutView.extend({
  tagName: 'tr',
  template: require('./templates/person.html'),

  triggers: {
    'click': 'click:item'
  },

  onClickItem: function(options) {
    window.alert('You clicked on ' + options.model.get('first_name'));
  }
});
```

The first thing you'll notice is the method name now resembles the style of
`onShow` and `onRender` from our [layout views](./layouts.md). This is because
both of those methods listen to implicit triggers called `show` and `render`.

You'll also see we referenced the model from `options` instead of `this`. This
is just illustrative, but it should give you a hint as to what the major
advantage of triggers over events are - you can pass the view, model, or
collection to other parts of the system in a reusable way.

### Really advanced trigger behavior

Triggers are also implicitly passed up to parent views for us as a trigger with
a `childview:` prefix. Let's build up a slightly more complex app which counts
the number of times a row was clicked and the total number of clicks on all
rows.

Start by opening up our `models/person.js` to set a default clicked value on
`Person`:

```js
var Backbone = require('backbone');

var Person = Backbone.Model.extend({
  defaults: {
    first_name: '',
    last_name: '',
    nickname: 'Nobody',
    click_count: 0
  }
});

module.exports = Person;
```

Now we'll modify our event handler in `personlist.js`. We'll also update our
`CompositeView` so we also handle the total clicks overall:

```js
var PersonView = Marionette.LayoutView.extend({
  tagName: 'tr',
  template: require('./templates/person.html'),

  triggers: {
    'click': 'click:item'
  },

  onClickItem: function(options) {
    var counter = options.model.get('click_count');
    options.model.set(counter + 1);
  }
});


var PersonList = Marionette.CompositeView.extend({
  tagName: 'table',
  template: require('./templates/personlist.html'),
  childView: PersonView,
  childViewContainer: 'tbody',

  // Note the lower case v in Childview below
  onChildviewClickItem: function(options) {
    var counter = this.model.get('click_count');
    this.model.set(counter + 1);
  }
});

module.exports = PersonList;
```

Now we're responding to user input but we have no way to display it. Let's
modify our tables so we display the counters as a third column in
`templates/personlist.html`:

```html
<thead>
  <tr>
    <th>First Name</th>
    <th>Last Name</th>
    <th>Times clicked</th>
  </tr>
</thead>
<tbody></tbody>
<tfoot>
  <tr>
    <th scope="row">Common first name</th>
    <td><%- first_name %></td>
    <td class="click-count"><%- click_count %></td>
  </tr>
</tfoot>
```

We'll also update our `templates/person.html`:

```html
<td><%- first_name %></td>
<td><%- last_name %></td>
<td class="click-count"><%- click_count %></td>
```

Now when you build and refresh you can click on the table rows and... nothing
happens! I thought views were bound to their models!

### Binding views to model events

We need to listen to the `modelEvents` if we want our view to react to changes
on models. The full list of events can be found at
[the Backbone website](http://backbonejs.org/#Events-catalog). We're interested
in listening to the `change` event. More specifically, when we change the
`click_count` attribute. Reopen our `person.js` view file:

```js
var PersonView = Marionette.LayoutView.extend({
  tagName: 'tr',
  template: require('./templates/person.html'),

  modelEvents: {
    'change:click_count': 'render'
  },

  triggers: {
    'click': 'click:item'
  },

  onClickItem: function(options) {
    var counter = options.model.get('click_count');
    options.model.set(counter + 1);
  }
});


var PersonList = Marionette.CompositeView.extend({
  tagName: 'table',
  template: require('./templates/personlist.html'),
  childView: PersonView,
  childViewContainer: 'tbody',

  modelEvents: {
    'change:click_count': 'render'
  },

  onChildviewClickItem: function(options) {
    var counter = this.model.get('click_count');
    this.model.set(counter + 1);
  }
});

module.exports = PersonList;
```

Now, whenever we click on a row, that row and the table will be re-rendered.

### A detour for more efficiency

Re-rendering the entire table for just two rows doesn't seem very efficient.
Ideally we'd be able to set a region just for the table footer (or even just the
`td` cell) and just have that view listen for model changes. Marionette 3 will
do exactly this but for now we'll rely on a little hack:

```js
var PersonList = Marionette.CompositeView.extend({
  tagName: 'table',
  template: require('./templates/personlist.html'),
  childView: PersonView,
  childViewContainer: 'tbody',

  modelEvents: {
    'change:click_count': 'updateClickCount'
  },

  ui: {
    click_count: '.click-count'
  }

  onChildviewClickItem: function(options) {
    var counter = this.model.get('click_count');
    this.model.set(counter + 1);
  },

  updateClickCount: function(model, value, options) {
    this.ui.click_count.text(value);
  }
});
```

We've added two concepts here:
  1. We've attached the `modelEvents` to a custom method call
  2. We added a ui hash

#### What about collections?

Marionette also listens to a hash called `collectionEvents` if we attach a
collection to the view. It works exactly the same way as `modelEvents`.

#### The ui hash

The `ui` hash is a named mapping to jQuery selectors in the view. We can
reference the `click-count` class we set earlier from `this.ui` and we can
treat it as any other jQuery selector by calling methods on it. In this case,
we set the text attribute to the value we just set on the model.

## More specific triggers

So far we've attached the triggers to the view's `el`, but what if we want to
respond to a button click? Let's give it a try and start by updating our
`person.html` template:

```html
<td><%- first_name %></td>
<td><%- last_name %></td>
<td class="click-count">
  <%- click_count %>

  <button class="btn-click" type="button">Click Here</button>
</td>
```

Let's open up our `PersonView` again and modify the trigger hash:

```js
var PersonView = Marionette.LayoutView.extend({
  tagName: 'tr',
  template: require('./templates/person.html'),

  modelEvents: {
    'change:click_count': 'render'
  },

  ui: {
    button: '.btn-click'
  },

  triggers: {
    'click @ui.button': 'click:item'
  },

  onClickItem: function(options) {
    var counter = options.model.get('click_count');
    options.model.set(counter + 1);
  }
```

First we've bound a `ui` key to the button we're interested in and now we've
referenced that key in our triggers. This allows us to hide the details of what
`button` actually is and just focus on the logic of what we want to do with it.

## What next?

Now that you can compose some pretty complex event handling logic, let's move on
to one of the key parts of building a single-page app -
[the AppRouter](./routers.md).
