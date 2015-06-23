# Dealing with Complex Persisted Data

If you are developing a Backbone / Marionette application, at some point you will very likely have to perform the following core operations:

1. Fetch data from the server.
2. Update your models with the server data.
3. Render the updated models.

The complexity and difficulty of this process largely depends on how closely the format of your server data matches that of your view(s).  If it's a perfect match, then the process is relatively straightforward -- you instantiate a collection with the appropriate URL property, call Backbone's `fetch` method on the collection, and pass that collection to your view of choice, e.g.   

**Server response.  Nicely formatted in a "JSON array of models," just like Backbone expects.**
    
    [
		{
			name: 'John Smith',
			age: 30,
			city: 'Los Angeles',
            date: 07/15/2014
		},
		{
			name: 'Bill Johnson',
			age: 33,
			city: 'San Francisco',
            date: 07/05/2014
		},
		{
			name: 'Ann Doe',
			age: 35,
			city: 'Boulder',
            date: 07/01/2014
		},
	]

**Collection, view, model, and templates.**
	
    MyModel = Backbone.Model.extend({
    	urlRoot : '/api/data'
    });
   
	MyCollection = Backbone.Collection.extend({
		url : '/api/data',
		model : MyModel
	});
    
    MyItemView = Marionette.ItemView.extend({
    	template : itemTemplate
    });
    
    MyCompositeView = Marionette.CompositeView.extend({
    	template : compositeTemplate,
        className : 'table-content'
        itemView : MyItemView
    });

	// compositeTemplate (written in Jade).
	div.header-cell NAME
	div.header-cell AGE
    div.header-cell CITY
    div.header-cell DATE

	// itemTemplate
    div.body-cell #{name}
    div.body-cell #{age}
    div.body-cell #{city}
    div.body-cell #{date}

**Code to get the server data and display it in your view.**
   
    myCollection = new MyCollection();
	myCollection.fetch();
	myCompositeView = new MyCompositeView({ collection: myCollection });
	// app is a Marionette application.  myRegion is a Marionette region.
	app.myRegion.show(myCompositeView);
   
However, if your server data is a far cry from what Backbone and/or Marionette expects, or from what you'd like to show end users in your views, then the process becomes a bit more nuanced.  For example, suppose  that instead of getting a nice array of models with clean attributes in your response, your get something that looks more like this:

**Server response.**
	
    {
		data: 
			
			[
				{
					name: 'John Smith',
					age: 30,
					address: {
                    	city: 'Los Angeles',
                        state: 'CA'
                    },
                    date: 1318781876
				},
				{
					name: 'Bill Johnson',
					age: 33,
					address: {
                        city: 'San Francisco',
                        state: 'CA'
                   },
                   date: 1318781876
				},
				{
					name: 'Ann Doe',
					age: 35,
					address: {
                        city: 'Boulder',
                        state: 'CO'
                    },
                    date: 1318781876
				},
			],
		
		someOtherKey: 'someOtherValue'
	}
    
Yikes.  Your array of models is now buried in an object under a "data" key, your "city" attribute is nested inside an "address" object, and your date is a Unix timestamp instead of a nicely formated "MM/DD/YY" string!  Given this data format, simply calling `fetch` won't work (if you do, you'll end up with only one "model" that has a "data" attribute and a blank view).  Instead, we need to use a few helper methods, Backbone's `parse` and Marionette's `serializeData`, to get our data where we want it to be.        

Let's start with Backbone's `parse`.  Per the [documentation](http://backbonejs.org/#Collection-parse), Backbone's `parse` function is "called by Backbone whenever a collection's models are returned by the server in `fetch`."  It takes the server response as an argument, and by default just passes it on through.  Now this isn't very helpful on its own, but we can override `parse` to meet our needs.  In this case, we should modify `parse` like so:      

	MyCollection = Backbone.Collection.extend({
		// Custom parse function.
		parse : function(response, options) {
			return response.data;
		}

	}); 

Here, we take the server response object, and return the value of the "data" key, which is array of models.  Backbone can now `set` the models properly.

But our work isn't finished.  We still need to deal with the embedded "city" attribute and the "date" attribute.  For this task, Marionette's `serializeData` is ideally suited.  `serializeData` is similar to `parse` in that the default function is essentially a no-op that you need to override (technically it's not a no-op -- it performs a shallow clone of the models' attributes via Backbone's `toJSON` but that's effectively just passing the attributes as-is on to the view / template).  It takes no arguments, and returns an object with the attributes formatted for display in your view.  This is key -- `serializeData` doesn't actually affect your model attributes themselves, it just packages them up for presentation, which is what we want.  Our `serializeData` function might look something like the following:

**Our ItemView with the new `serializeData` function**
    
    MyItemView = Marionette.ItemView.extend({
    	template : itemTemplate
    	
    	serializeData : function() {
    		// Get the relevant model attributes.
    		var addressObject = this.model.get('address');
    		var unixDate = this.model.get('date');
			
            // Return an object for your template.
    		return {
    			name : this.model.get('name'),
    			age : this.model.get('age'),
    			city : addressObject.city,
    			 // using moment.js library to format the date.
                date : moment.unix(unixDate).format('MM/DD/YYYY')
    		};
    	}
    });

**Nothing needs to change in our ItemView template**
	
    // itemTemplate
    div.body-cell #{name}
    div.body-cell #{age}
    div.body-cell #{city}
    div.body-cell #{date}

Now you might be saying to yourself, "`parse` and `serializeData` seem to be doing similar things...what's the difference?  When should I use `parse` and when should I use `serializeData`?"  Good question.  You actually could have used `parse` to accomplish everything we did in the above example -- access the relevant JSON array of models *and* clean up the city and date attributes.  But there's a good reason why we didn't do it that way: separation of concerns.  `parse` affects your data structure -- whatever you return from `parse` will be used to set your model attributes.  Using `parse` to clean up the "city" and "date" attributes would therefore be modifying your data structure to fit your view.  Not only does this create a tight coupling, but it also creates a disparity between the models on the server and the models on the client, which in turn makes syncing between client and server all the more difficult.  Not good.  That's why we used `serializeData` to clean up the city and date attributes.  `seralizeData` does *not* affect the models themselves, it only affects how the models' attributes are prepared for the view.  Make as many adjustments as you wish, your data structure remains intact.

So in summary:

+ Use Backbone's `collection.parse` and `model.parse` to translate server responses into proper data structures for Backbone (objects for models and arrays of objects for collections).
+ Use Marionette's `ItemView.serializeData` to modify Backbone model attributes for display in your views. 

Two additional things to keep in mind.  First, `parse` is both a model and a collection method and if you have parse defined for your collection and your model *both `parse` methods will be called anytime you return a collection of models from the server*.  Chances are you don't want both `model.parse` and `collection.parse` to be called whenever you get a collection back from the server, so you need to address this.  Fortunately, whenever a model is created as part of a collection, Backbone [passes a collection option to the model constructor](http://backbonejs.org/docs/backbone.html#section-117), which you can check for and accommodate accordingly:
		
		// Model parse function.
		parse : function(response, options) {
			// If model is part of a collection, just pass the response on through.
			if (options.collection) {
				return response;
			}

			// If model is on its own, execute your parse logic.
			else {
				return response.data
			}
		}

Full example can be found in [this](http://stackoverflow.com/questions/18652437/backbone-not-parse-each-model-in-collection-after-fetch) stackoverflow answer.

Second, even though it's currently the default option in Marionette (there are plans to change it soon), you really shouldn't use `toJSON` to serialize data for your views.  This is due to the fact that `toJSON` is *also* used [to prep data for the server](http://backbonejs.org/#Model-toJSON), and chances are you don't want to override your server prep methods with your view prep methods.  Does that mean you have to write a `serializeData` function for every single view to keep things straight?  Fortunately not.  Just override Marionette's `serializeModel` and `serializeCollection` methods:     

	Marionette.View.prototype.serializeModel = function(model) {
	  model = model || this.model;
	  return _.clone(model.attributes);
	};

	Marionette.ItemView.prototype.serializeCollection = function() {
	  return collection.map(function(model) {
	    return this.serializeModel(model);
	  }, this);
	};

That's it for now.  Happy coding!