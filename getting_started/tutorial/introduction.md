# Making your applications dance!

Welcome to the getting started guide for Backbone Marionette.


## What will I get from this tutorial?

After reading this tutorial you will be able to write your own Marionette
applications from scratch. We will build a very simple todo list that we can add
items to and remove items from.

This tutorial will teach you how to render your data with views, structure your
application layout, route your user from the address bar, and integrate with
data stored in the server.


## Things to know

Before we start the tutorial, we'll set up a basic directory structure to keep
everything in order. We'll create a separate folder each for collections,
models, routers, templates, and views. The folder structure is:

```
|-- index.html
|-- app/
     |-- driver.js
     |-- collections/
     |-- models/
     |-- routers/
     |-- templates/
     |-- views/
```

Whenever we reference a file, we will typically refer to the filename and the
type of file corresponding to a directory in the above structure.

Our `index.html` file will be:

```html
<!DOCTYPE html>
<html>
  <head>
  </head>
  <body>
    <div id="app-hook"></div>
    <script src="static/js/app.js"></script>
  </body>
</html>
```

If you haven't already, go through the
[Installing Marionette](installing_marionette.md) section for instructions in
how to setup and install a Marionette application.

You can optionally use a CSS framework such as
[Bootstrap](https://getbootstrap.com) to improve how your application looks.
This tutorial will stick to a simpler to understand structure and HTML.

This tutorial will regularly ask you to restructure code from previous
chapters. We will detail every change, so if you can't see a file, assume that
it remains unchanged.


## Experimentation

Throughout this tutorial you should feel free to experiment with the code
samples to see if you can understand what's happening. Try different options,
templates, classes, or even write some custom logic.
