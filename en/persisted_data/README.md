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
at once, as we'll get to later.

There's no reason we have to stick to the fields defined by defaults (we don't
even need to define them), so we can add some ad-hoc data:

```javascript
note.set('reminder', '2015-09-04 11:01:00');
```

### Server synchronization

This is all well and good but it still takes us no closer to pushing and pulling
data to a server. First, we'll need to define a pretend server. Let's give it
the URL `http://example.com/note/1` which, when we GET it, returns:

```javascript
{
  "id": 1,
  "title": "My note",
  "content": "Some content saved online",
  "timestamp": "2015-09-02 11:01:02",
  "reminder": null
}
```

First, we'll define a new type of Note:

```javascript
var Note = Backbone.Model.extend({
  urlRoot: 'http://example.com/note/'
});

var note = new Note({
  id: 1
});
note.fetch();
```

The `Model.fetch` method knows how to construct a URL from its `urlRoot` and
`id` properties - namely appending `id` to `urlRoot`. Like most web calls in
JavaScript, `fetch` is asynchronous - execution will continue before the web
request completes.

If we wanted to perform an action on the data once the fetch method returns, we
can attach a `success` callback, as in jQuery:

```javascript
note.fetch({
  success: function(response, model) {
    // Do something
  }
});
```

When we're done modifying our data and want to save it, we'll call
`Model.save()` and Backbone will save the data back to the server:

```javascript
note.set('title', 'New title');
note.save();
```

We could also modify data and save in one call:

```javascript
note.save({
  title: 'New title'
});
```

Again, like fetch, we can use the success callback to execute based on the
result of the call:

```javascript
note.save(
  {
    title: 'New title'
  },
  {
    success: function(response, model) {
    // Do something
  }
);
```

This dependency on the `success` and `error` callbacks doesn't help us when our
model is attached to multiple [views][views] - what happens if one view updates
the model but another one needs to be changed? Do both views need to know about
each other? Let's find out.


### Events

Models use events to signal that something has happened that another object may
be interested in. For example, a model can signal that fields have changed, it
has successfully saved its data (or failed), or that it has fetched a new set
of data from the server. These events can then be listened to by
[views][view-event], or any other object that knows about the model. Using this,
our models can affect multiple parts of an application without needing to be
explicitly told about them. Let's look at some examples:

```javascript
note.set('title', 'New title');
// Fires the 'change' and 'change:title' events

note.set('content', 'New content');
// Fires another 'change' event and 'change:content' event

note.set({
  content: 'Newer content',
  title: 'Newer title'
});
// Fires only one 'change' event, 'change:content', and 'change:title'

note.save();
// Fires the 'request' event, then either 'sync' or 'error' depending on the
// server response

note.fetch();
// Like save, fires `request`, then `sync` or `error` depending on the response
```

A full, up-to-date, list of events can always be found on the
[Backbone documentation][backbone-event], with a description of when each event
fires. As we can see in the example, `change` is fired every time we
successfully `set` a field - if we want to only fire a single `change` event,
we must pass an object into `set` with all the fields we want to update.


#### Listening to Events

For these events to be useful, we need to attach listeners that act when the
event is fired. When the title is updated, let's listen for it like so:

```javascript
var titleUpdated = function(model, value) {
  console.log('title is now ' + value);
};

note.on('change:title', titleUpdated);
note.set('title', 'Changed Title');
// Outputs 'title is now Changed Title'
```

We can also listen to events from our [views][view-event] by using the
`modelEvents` attribute. When defining our view:

```javascript
var NoteView = Marionette.LayoutView.extend({
  modelEvents: {
    'change:title': 'updateTitle'
  },

  updateTitle: function(model, value, options) {
    console.log('title for NoteView is now ' + value);
  }
});
```

This will trigger the `updateTitle` method on our `NoteView` whenever `title`
changes.

#### Custom Events

When you start building your apps, you'll notice that Backbone doesn't always
give you the events you need. As a common example, there's no way to distinguish
between a successful save and a successful data pull - `sync` covers both cases.

Luckily, we can fire custom events on models, and Marionette views are capable
of binding to them. Let's see an example of this now:


```javascript
var MyView = Marionette.LayoutView.extend({
  modelEvents: {
    saved: 'saveComplete'
  },

  triggers: {
    'click .save-button': 'save:note'
  },

  onSaveNote: function() {
    this.model.save(
      {
        content: 'New content',
        title: 'New title'
      },
      {
        success: function() {
          note.trigger('save', note);
        }
      }
    );
  },

  saveComplete: function(model) {
    console.log('Note saved');
  }
});
```

Now, when `save` succeeds, our `saveComplete` method gets called and
`Note saved` makes it to the log. Whilst useful, this example has a flaw in that
only `NoteView` will trigger the `saved` event, and so it's not much better than
just executing the code in `success` directly. This could be acceptable if our
`save` is only called in this view - other views can still happily listen to the
`saved` event, even though they don't fire it.


## Collections

Managing a single model is good, and we can do a lot of interesting things with
just this knowledge. When it comes to building applications, we will normally
operate on collections of data to render lists, draw charts, and otherwise
aggregate data.

The `Backbone.Collection` class is used to model and act on multiple models at
the same time. Let's take our note example and see how we could build up a list
of notes that we'd like to draw later:

```javascript
var NoteCollection = Backbone.Collection.extend({
});

var noteList = new NoteCollection([
  new Note({title: 'Note1', content: 'Content1'}),
  new Note({title: 'Note2', content: 'Content2'})
]);
```

This will store the list of notes. After creating our collection, we can add new
notes using the `add` method, like so:

```javascript
noteList.add(new Note({title: 'Note3', content: 'Content3'}));
```

Since we only have a single type in our list, let's set this constraint in the
definition:

```javascript
var NoteCollection = Backbone.Collection.extend({
  model: Note
});

var noteList = new NoteCollection([
  {title: 'Note1', content: 'Content1'},
  {title: 'Note2', content: 'Content2'}
]);

noteList.add({title: 'Note3', content: 'Content3'});
```

Now Backbone will convert our raw JavaScript objects into `Note` objects, both
on creation and when adding new objects. It will also enforce this:

```javascript
var NotANote = Backbone.Model.extend();

noteList.add(new NotANote({something: 'Some Value'}));
// ERROR - This is not a Note object
```


### Synchronizing data

Collections are used to pull lists of data from our server and build an
abstraction we can operate on. Assuming that our note-taking app has a server
endpoint `/note/` that returns a JSON list in the form:

```javascript
[
  {
    "content": "Note Content",
    "title": "Note title",
    "reminder": null,
    "timestamp": "2015-09-01 12:01:00",
    "id": 1
  },
  {
    "content": "Another note with some content",
    "title": "Note title2",
    "reminder": "2015-09-02 09:00:00",
    "timestamp": "2015-09-01 12:51:00",
    "id": 2
  },
  {
    "content": "Yet another note",
    "title": "Note title 3",
    "reminder": null,
    "timestamp": "2015-02-01 12:01:00",
    "id": 3
  }
]
```

Now let's define our collection that references the URL endpoint:

```javascript
var NoteCollection = Backbone.Collection.extend({
  url: '/note/',
  model: Note
});

var noteList = new NoteCollection();
noteList.fetch();
```

Like in `Model`, we use the `fetch` method to pull our data from the server and
insert it into our collection.


[backbone-model]: http://backbonejs.org/#Model
[backbone-collection]: http://backbonejs.org/#Collection
[backbone-event]: http://backbonejs.org/#Events-catalog
[views]: ../views/README.md
[view-event]: ../views/events.md
