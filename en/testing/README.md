# Testing Marionette Applications

When working with JavaScript, there are always multiple ways of testing your
application. In this guide, we will cover the basics of testing, different
strategies you can use to test your applications, and provide a simple test
framework configuration you can use to test your own apps.

## What is Testing?

A key part of building applications is ensuring they're fit for use before we
release them to our customers. Traditionally, we would manually test our
systems after building them. This process is very time-consuming and
error-prone, requiring previous bugs to be constantly re-tested lest regressions
creep back into our code.

To improve this situation, the principles of test-driven development were
designed. This places a heavy emphasis on building up automated tests that can
be run against your code base constantly. Achieving a good coverage of your
application's use-cases and potential problem-areas helps ensure that you are
providing a high-quality system to your users at all times.

## Testing a Model

Let's look at a simple test case with a custom model method. We will use a basic
assertion framework:

```javascript
var assert = require('assert');
var ToDo = require('./models/todo');

describe('ToDo', function() {
  var model;

  beforeEach(function() {
    model = new ToDo();
  });

  afterEach(function() {
    model = null;
  });

  it('can save a note', function() {
    model.makeNote('Scott', 'Get the shopping in for later');

    assert.ok(model.get('created')); // Assert that created was set
  });
});
```

Our `models/todo.js` file contains:

```javascript
var Backbone = require('backbone');
var moment = require('moment');

module.exports = Backbone.Model.extend({
  defaults: {
    author: '',
    text: '',
    created: ''
  },

  makeNote: function(author, text) {
    this.set({
      author: author,
      text: text,
      created: moment().format('YYYY-MM-DDThh:mm:ss')
    });
  }
})
```

This gives us a basic outline for testing our model methods behave as expected.

## Configuring a Test Framework

## When to Test
