# App Router Code

The code for the App Router tutorial can be used as a reference for how to build
and integrate a simple application that uses the AppRouter for its navigation.
Each application will have different requirements, so feel free to adapt this
layout for your own application design.


## Index file

`index.html`:

```html
<!DOCTYPE html>
<html>
  <head>
  </head>
  <body>
    <div id="blog-hook"></div>
    <script src="static/js/app.js"></script>
  </body>
</html>
```

As with all the appendix items here, this is a simplistic HTML file. Feel free
to utilize styling libraries such as Bootstrap to jazz it up.


## Driver file

`driver.js`:

```js
var Marionette = require('backbone.marionette');
var Router = require('./router');


var initialData = {
  posts: [
    {
      author: 'Scott',
      title: 'Why Marionette is amazing',
      content: '...',
      id: 42,
      comments: [
        {
          author: 'Steve',
          content: '...',
          id: 56
        }
      ]
    },
    {
      author: 'Andrew',
      title: 'How to use Routers',
      content: '...',
      id: 17
    }
  ]
};


var App = new Marionette.Application({
  onStart: function(options) {
    var router = new Router(options);

    /** Starts the URL handling framework */
    Backbone.history.start();
  }
});

app.start({initialData: initialData});
```


## Router and Controller

`router.js`:

```js
var LayoutView = require('./views/layout');
var BlogList = require('./collections/blog');


var Controller = Marionette.Object.extend({
  initialize: function() {
    /** The region manager gives us a consistent UI and event triggers across
        our different layouts.
    */
    this.options.regionManager = new Marionette.RegionManager({
      regions: {
        main: '#blog-hook'
      }
    });
    var initialData = this.getOption('initialData');

    var layout = new LayoutView({
      collection: new BlogList(initialData.posts)
    });

    this.getOption('regionManager').get('main').show(layout);

    /** We want easy access to our root view later */
    this.options.layout = layout;
  },

  /** List all blog entrys with a summary */
  blogList: function() {
    var layout = this.getOption('layout');
    layout.triggerMethod('show:blog:list');
  },

  /** List a named entry with its comments underneath */
  blogEntry: function(entry) {
    var layout = this.getOption('layout');
    layout.triggerMethod('show:blog:entry', entry);
  }
})

var Router = Marionette.AppRouter.extend({
  appRoutes: {
    'blog/': 'blogList',
    'blog/:entry': 'blogEntry'
  },

  /** Initialize our controller with the options passed into the application,
      such as the initial posts list.
  */
  initialize: function() {
    this.controller = new Controller({
      initialData: this.getOption('initialData');
    });
  }
});

module.exports = Router;
```


## Collections and Models

`models/blog.js`:

```js
module.exports = Backbone.Model.extend({
  /** Let us inject 0 comments in from the data set
  */
  defaults: function() {
    return {
      comments: []
    }
  }
});
```


`collections/blog.js`:

```js
var Blog = require('../models/blog');

module.exports = Backbone.Collection.extend({
  model: Blog
});
```

`models/comment.js`:

```js
module.exports = Backbone.Model.extend();
```

`collections/comment.js`:

```js
var Comment = require('../models/comment');


module.exports = Backbone.Collection.extend({
  model: Comment
});
```


## Views

`views/layout.js`:

```js
var List = require('./list');
var Blog = require('./blog');


var LayoutView = Marionette.LayoutView.extend({
  template: require('../templates/blog/layout.html'),

  regions: {
    layout: '.layout-hook'
  },

  onShowBlogList: function() {
    var list = new List({collection: this.collection});
    this.showChildView('layout', list);

    /*  Remember - this only sets the fragment, so we can safely call this as
        often as we like with no negative side-effects.
    */
    Backbone.history.navigate('blog/');

  },

  onShowBlogEntry: function(entry) {
    var model = this.collection.get(entry);
    this.showBlog(model);
  },

  onChildviewSelectEntry: function(child, model) {
    this.showBlog(model);
  },

  /** Child-initiated alias to onShowBlogList */
  onChildviewShowBlogList: function() {
    this.triggerMethod('show:blog:list');
  },

  /** Share some simple logic from our subviews */
  showBlog: function(blogModel) {
    var blog = new Blog({model: blogModel});
    this.showChildView('layout', blog);

    /*  Remember - this only sets the fragment, so we can safely call this as
        often as we like with no negative side-effects.
    */
    Backbone.history.navigate('blog/' + blog.id);
  }
});

module.exports = LayoutView;
```

`views/list.js`:

```js
var Entry = Marionette.LayoutView.extend({
  template: require('../templates/blog/item.html'),
  tagName: 'li',

  triggers: {
    click: 'select:entry'
  }
});


var BlogList = Marionette.CollectionView.extend({
  childView: Entry,
  tagName: 'ul',

  onChildviewSelectEntry: function(child, options) {
    this.triggerMethod('select:entry', child.model);
  }
});

module.exports = BlogList;
```

`views/blog.js`:

```js
var Comment = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require('../templates/blog/comment.html')
});

var CommentListView = Marionette.CollectionView.extend({
  tagName: 'ol',
  childView: Comment
});

var Blog = Marionette.LayoutView.extend({
  template: require('../templates/blog/blog.html'),

  regions: {
    comments: '.comment-hook'
  },

  ui: {
    back: '.back'
  },

  triggers: {
    'click @ui.back': 'show:blog:list'
  },

  onShow: function() {
    var comments = new CommentList(this.model.get('comments'));
    var commentView = new CommentListView({collection: comments});

    this.showChildView('comments', commentView);
  }
});

module.exports = Blog;
```


## Templates

`templates/blog/layout.html`:

```html
<h1>Marionette Blog</h1>
<div class="layout-hook"></div>
```

`templates/blog/item.html`:

```html
<a href="/blog/<%- id %>"> <!-- This won't actually link anything -->
  <div class="title"><%- title %></div>
  by
  <span class="author"><%- author %></span>
</a>
```

`templates/blog/blog.html`:

```html
<a href="/blog/" class="back">To Blog List</a>
<h2><%- title %></h2>
<div class="author">by <%- author %></div>

<%- content %>

<div class="comment-hook"></div>
```

`templates/blog/comment.html`:

```html
<%- content %>

<div class="author"><%- author %></div>
```
