# Installing Marionette

As with all JavaScript libraries, there are a number of ways to get started with
a Marionette application. In this section we'll cover the most common ways.


## Using NPM and Browserify

To install Marionette using NPM, we must install a few dependencies and then
install Marionette itself.

  1. Install NPM following the advice from the [NPM blog][install-npm]
  2. Create a directory for your JavaScript application
  3. Inside that directory, run `npm init`, giving your application names
  4. Install Browserify: `sudo npm install -g browserify`
  5. Install Underscore: `npm install --save underscore@1.7.0`
  6. Install Backbone: `npm install --save backbone`
  7. Install Marionette: `npm install --save backbone.marionette`
  8. Install the Underscore plugin for browserify:
    `npm install --save node-underscorify`
  9. Download [jQuery][jquery] and place it in a static folder

Now we have our basics in place, we can setup a standard `setup.js` file that
will ensure Marionette can be accessed:

```javascript
window._ = require('underscore'); // Backbone can't see it otherwise

var Backbone = require('backbone');
Backbone.$ = window.$; // Use the jQuery from the script tag
Backbone.Marionette = require('backbone.marionette');
```

When building our applications, we'll commonly use a `driver.js` file that will
start by importing this `setup.js` file: `require('setup.js')`.

When we want to compile our application, we'll use Browserify
`browserify driver.js -o static_folder/app.js` where `static_folder` is the
directory to the static directory that your web server provides.

Now, in the `index.html` file, or base template (if you're using Django, Rails,
etc.) place the following at the bottom of your `body` tag:

```html
<script src="static_folder/jquery.min.js"></script>
<script src="static_folder/app.js"></script>
```

It's important to load jQuery first, so your `setup.js` file sees it.


[install-npm]: http://blog.npmjs.org/post/85484771375/how-to-install-npm
[jquery]: https://jquery.org/
