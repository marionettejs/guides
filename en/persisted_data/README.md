# Handling persisted data

To do any interesting work, a web application usually needs to interact with a
web server. Historically, this would have been managed by using jQuery's
`$.ajax` function, and storing the result in a JavaScript object. As our
applications grow in size, this starts to become more and more difficult to
manage unless we introduce a structure.

Another drawback of this method is that updating our UI in response to data
changes gets very complex very quickly. Take an application like Evernote -
there are usually two versions of a note: the one in the list and the one you're
currently editing. How do you easily keep both versions in sync? If we want to
get an updated version of the note from our server, how can we easily make sure
both versions are updated simultaneously?

Backbone gives us the [Model][backbone-model] and
[Collection][backbone-collection] classes for just this purpose. In short, the
Model represents a single object/resource/record with attributes that can be
synchronized with a server. A Collection is simply a list of Models, with extra
helper methods, that can also be synchronized with our server. With these two
classes, it becomes very easy to simply list a Collection, change the individual
Model instances, and have those changes simultaneously propagated across
different sections of your application.

Sound too easy? Let's look at how to make it all happen.


## Models

We'll start by looking at the `Backbone.Model` and how to use it. Let's create
a simple note with a timestamp, content, and title.

```javascript
var Note = Backbone.Model.extend({
  defaults: {
    timestamp: '',
    content: '',
    title: ''
  }
});

var note = new Note({
  timestamp: '2015-09-02 11:00:00',
  content: "I'm writing a book!",
  title: 'Doing something'
});
```

Now we have our note, we can read its fields:

```javascript
console.log(note.get('content'));
// Doing something
```

If we have some new data to put in this note, we can update it like so:

```javascript
note.set('content', 'New content');
```

Or, if we wanted to update multiple fields at a time:

```javascript
note.set({
  content: 'New content',
  title: 'Updated title too'
});
```

Besides brevity, there's a very good reason we'd want to update multiple fields
at once, as we'll see later.


## Collections


[backbone-model]: http://backbonejs.org/#Model
[backbone-collection]: http://backbonejs.org/#Collection
