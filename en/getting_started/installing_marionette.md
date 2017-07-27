# Installing Marionette

As with all JavaScript libraries, there are a number of ways to get started with
a Marionette application. In this section we'll cover the most common ways.


## Using NPM and Webpack

[Webpack][webpack] is a build tool that makes it easy to pull your dependencies
together into a single bundle to be delivered to your browser's `<script>` tag.
It works particularly well with Marionette and jQuery.

To install Marionette using NPM and Webpack:

  1. Install NPM following the advice from the [NPM blog][install-npm]
  2. Create a directory for your JavaScript application
  3. Inside that directory, run `npm init`, giving your application names
  4. Install [Webpack][webpack]: `npm install --save webpack`
  5. Install Marionette's dependencies:
    `npm install --save backbone@1.3.3 backbone.marionette@2.4.7 underscore@1.8.3
    backbone.wreqr backbone.babysitter underscore-template-loader`

Whether you bundle jQuery really depends on whether you have external
dependencies on jQuery. For example, if you're using Bootstrap's JavaScript as a
`<script>` tag, you probably don't want to bundle jQuery and you'll want to
depend on the global `window.$` variable.


### Bundled jQuery

Bundling jQuery lets you require `jquery` inside your application and keeps all
your dependencies under Webpack's management. To bundle jQuery, simply
`npm install --save jquery`. We'll move on to configuring our Webpack
application.


#### Configuring Webpack

Configuring Webpack with jQuery bundled in your application is relatively
straightforward. Let's assume that your application lives in a directory called
`app` and the main entry point (or driver) is called `driver.js`. In addition,
you want to send the resulting file to a directory called `static/js`.

Create a file called `webpack.config.js` with the following:

```javascript
var webpack = require('webpack');

module.exports = {
  entry: './app/driver.js',
  module: {
    loaders: [
      {
        test: /\.html$/,
        loader: 'underscore-template-loader'
      }
    ]
  },
  output: {
    path: __dirname + '/static/js',
    filename: 'bundle.js'
  },
  plugins: [
    new webpack.ProvidePlugin({
      _: 'underscore'
    })
  ],
  resolve: {
    modules: [__dirname + '/node_modules', __dirname + '/app']
  },
  resolveLoader: {
    modules: [__dirname + '/node_modules']
  }
};
```


### Global jQuery

If you're referencing the `window.$` variable created by including jQuery in a
`<script>` tag, download [jQuery][jquery] and place it in your static JS folder.


#### Configuring Webpack

Configuring Webpack for a global jQuery variable is only slightly more
complicated than a bundled jQuery. Let's assume that your application lives in a
directory called `app` and the main entry point (or driver) is called
`driver.js`. In addition, you want to send the resulting file to a directory
called `static/js`.

Create a file called `webpack.config.js` with the following:


```javascript
var webpack = require('webpack');

module.exports = {
  entry: './app/driver.js',

  externals: {
    'jquery': '$'
  },

  module: {
    loaders: [
      {
        test: /\.html$/,
        loader: 'underscore-template-loader'
      }
    ]
  },
  output: {
    path: __dirname + '/static/js',
    filename: 'bundle.js'
  },
  plugins: [
    new webpack.ProvidePlugin({
      _: 'underscore'
    })
  ],
  resolve: {
    modules: [__dirname + '/node_modules', __dirname + '/app']
  },
  resolveLoader: {
    modules: [__dirname + '/node_modules']
  }
};
```

Note the new `externals` key that tells Webpack to inject the global `window.$`
variable whenever Backbone or Marionette reference `require('jquery')` in their
imports.


### Building your application

With Webpack configured, you can build your application simply by doing:
`node_modules/.bin/webpack`.


### Serving your Application

We'll now create our `index.html` file to reference our new application and
start it.

```html
<script src="static/js/bundle.js"></script>
```

If you're using any other imported JavaScript (e.g. jQuery), make sure they're
loaded before our `bundle.js` file so anything that depends on them will be able
to see the globals they expose.


## Using NPM and Brunch

[Brunch][brunch] is a builder. Not a generic task runner, but a specialized tool
focusing on the production of a small number of deployment-ready files
from a large number of heterogenous development files or trees.

To install Marionette using NPM and Brunch:

  1. Install NPM following the advice from the [NPM blog][install-npm]
  2. Install Brunch: `sudo npm install -g brunch`
  3. Run `brunch new our_directory_name -s marionettejs`. Brunch will create simple skeleton
and install all needed dependencies.

Simple skeleton is placed in `our_directory_name`.
If we want to change our configuration file, we should look at `brunch-config.js`
inside our folder.

[Here](https://github.com/brunch/brunch/blob/master/docs/config.md)
more information about how to configurate `brunch`.

### Building your Application

When we want to compile our application, we'll run inside our folder:

`brunch build --production` — builds minified project for production.

### Serving your Application

When we want to run our application, we'll run inside our folder:

`brunch watch --server`. Brunch will watch the project with continuous rebuild.

Try opening `http://localhost:3333/` we'll see our application working.


## Using NPM and Browserify

[Browserify][browserify] is a build tool that makes it easy to bundle NPM
modules into your application, so you can `require` them as you would import
dependencies in any other language.

To setup Browserify and Marionette, we must install a few dependencies and then
install Marionette itself.

  1. Install NPM following the advice from the [NPM blog][install-npm]
  2. Create a directory for your JavaScript application
  3. Inside that directory, run `npm init`, giving your application names
  4. Install Browserify: `sudo npm install -g browserify`
  5. Install Underscore: `npm install --save underscore@1.8.3`
  6. Install Backbone: `npm install --save backbone@1.3.3`
  7. Install Marionette: `npm install --save backbone.marionette@2.4.7`
  8. Install the Underscore plugin for browserify:
    `npm install --save node-underscorify`
  9. Install jquery `npm install --save jquery`
  10. Download [jQuery][jquery] and place it in a static folder

Now we have our basics in place, we can setup a standard `setup.js` file that
will ensure Marionette can be accessed:

```javascript
window._ = require('underscore'); // Backbone can't see it otherwise

var Backbone = require('backbone');
Backbone.$ = window.$; // Use the jQuery from the script tag
Backbone.Marionette = require('backbone.marionette');
```

When building our applications, we'll commonly use a `driver.js` file that will
start by importing this `setup.js` file: `require('./setup.js')`.


### Building your Application

When we want to compile our application, we'll use Browserify:

```bash
browserify driver.js -t node-underscorify -o static_folder/app.js
```

where `static_folder` is the directory to the static directory that your web
server provides.


### Serving your Application

In the `index.html` file, or base template (if you're using Django, Rails,
etc.) place the following at the bottom of your `body` tag:

```html
<script src="static_folder/jquery.min.js"></script>
<script src="static_folder/app.js"></script>
```

It's important to load jQuery first, so your `setup.js` file sees it.

[install-npm]: http://blog.npmjs.org/post/85484771375/how-to-install-npm
[jquery]: https://jquery.org/
[browserify]: http://browserify.org/
[webpack]: https://webpack.github.io/
[brunch]: http://brunch.io/
