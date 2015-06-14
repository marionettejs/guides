# Implementing Routing

One of the main benefits to building a JavaScript application is showing/hiding
views based on user input without lengthy calls to the server. For example, when
choosing an item from a list, we can just render that item based on information
stored in a `Collection` without making any lengthy calls to the server to fetch
data we have in our browser already. Over time this method has been successfully
used in many applications, reducing server load and increasing the
responsiveness of our web applications.

However, a major downside of this method has historically been the inability to
get back to that page. You'll undoubtedly remember the frustration of
accidentally clicking back, refreshing a page, or closing your browser only to
get sent to the home page and have to find your way back to the screen you were
working on, or that interesting article you were reading.

Over the years different methods were tried, including having the server try to
remember your position in an application, many with poor results, bugs,
complex code, inability to bookmark or share a post, and many other issues you
have likely encountered. What we really wanted was to just use the trusted
browser URL in our applications.

Twitter popularized the use of the fragment - the portion of the URL after `#` -
to store information about the current location. This part of the URL isn't read
by the server but is understood by web browsers to jump to certain  sections of
a page. It is also the part of the `window.location` object that we can change
without causing the browser to change the whole page from underneath us. When we
display some information we may want to retrieve later, we could simply do:
`window.location.hashCode = 'newLocation';` and the fragment would be updated.
When we want to retrieve the data on page load, we can read the value of
`window.location.hashCode` and map it to the code we want to execute to retrieve
and render the information referenced by the fragment.

The Marionette `AppRouter` is an attempt to abstract over this behavior and
provide a URL routing engine that mirrors those found in the more mature
server-side web frameworks such as those provided by Django, Flask, and Ruby on
Rails.


## Router and Controller

Routing in Marionette is split into two connected parts: a router and a
controller. The router takes a map of URL fragments to listen for and maps them
to method names on an attached controller to call. The controller is a simple
object with matching methods to be called by the router.

The Marionette class used is called the `AppRouter`. To use it we simply attach
an `appRoutes` object to map the routes to methods:

```js
var Router = Marionette.AppRouter.extend({
  appRoutes: {
    'blog/': 'blogList',
    'blog/:id': 'blogEntry',
    'blog/:id/comments/:id': 'blogComment'
  }
});

module.exports = Router;
```

The methods referenced in `appRoutes` must exist on an attached controller. We
can attach this controller in a number of ways which we'll explore below.

### Attaching a Controller

The easiest way to attach a controller is to simple attach an object to the
`controller` key when extending `AppRouter` as such:

```js
var Router = Marionette.AppRouter.extend({
  appRoutes: {
    'blog/': 'blogList',
    'blog/:entry': 'blogEntry',
    'blog/:entry/comments/:comment': 'blogComment'
  },

  controller: {
    blogList: function() {
      // ...
    },
    blogEntry: function(entry) {
      // ...
    },
    blogComment: function(entry, comment) {
      // ...
    }
  }
});

module.exports = Router;
```


## Browser History API
More recently, browser vendors recognized the benefits of this pattern and began
working on the Browser History and Push State APIs. These combined the benefits
of using the fragment with more natural looking URLs that could be recognized by
the server too.
