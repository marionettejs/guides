# Multiple Application Starters

Another pattern for web applications is to use multiple applications and
starters. This is useful when integrating with an existing web service that
mixes JavaScript with regular web pages and forms.


## Dynamic Initializer

The key to achieving this is by attaching initializers on the application. To
do this, we can use the `addInitializer` method like so:

```js
var Marionette = require('backbone.marionette');

var initializerFunction = require(window.initializer);

var application = new Marionette.Application();
application.addInitializer(initializerFunction);
app.start({initialData: window.initialData});
```

We can then reference the initializer function in our application template:

```html
<script>
  window.initializer = 'initializers/root';
  window.initialData = {};
</script>
<script src="/static/js/app.js"></script>
```

The value of `initializer` is then set by the template created by the server.


### Browserify Integration

If you use [Browserify][browserify], you'll notice that this doesn't work at
all. The reason for this is because Browserify tracks the strings in your
`require()` calls to determine the modules to import and doesn't support dynamic
imports.

To get round this, we can use a `includes.js` file to map all the locations to
keys:

```js
module.exports = {
  root: require('./initializers/root'),
  guides: require('./initializers/guides')
};
```

Now, `driver.js` looks like:

```js
var Marionette = require('backbone.marionette');
var includes = require('./includes');

var initializerFunction = includes[window.initializer];

var application = new Marionette.Application();
application.addInitializer(initializerFunction);
app.start({initialData: window.initialData});
```

Finally, we modify our template to output:

```html
<script>
  window.initializer = 'root';
  window.initialData = {};
</script>
<script src="/static/js/app.js"></script>
```

Now Browserify can follow the imports from `includes.js` and will automatically
import all the modules.


## Purpose

Why go to all this trouble? There are plenty of valid reasons an application
may use multiple starters. It can reduce the amount of JS code transferred to
the client in one go - the server will implement some of the logic itself. It
also allows us to use Marionette for certain screens; a major benefit in
particularly large legacy applications that would take a long time to completely
port over to a JS app which may not be worth the cost.

[browserify]: https://browserify.org
