# Sauvegarder les données utilisateur

Nous avons juste construit une liste simple de travaux qui a besoin d'être complétée mais, comme toujours, nous souhaitons ajouter de nouvelles taches, à mesure qu'elles arrivent.
Dans ce chapitre, nous allons définir cette fonctionen utilisant davantage de fonctionnalités de _Marionette_.


## Construire notre formulaire

Tout d'abord, nous avons besoin d'un formulaire pour saisir des données. Afin de faire fonctionner cela dans notre application actuelle, nous pouvons interchanger notre `CollectionView` par une  `CompositeView` au sein du fichier `driver.js` :

```js
// FIXME Expose underscore as global
_ = require( "underscore" );

var Backbone = require('backbone');
var Marionette = require('backbone.marionette');


var ToDo = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require( './templates/todoitem.html' )
});


var TodoList = Marionette.CompositeView.extend({  
  el: '#app-hook',
  template: require( './templates/todolist.html' ),

  childView: ToDo,
  childViewContainer: 'ul',

  initialize: function() {
    this.collection = new Backbone.Collection([
      {assignee: 'Scott', text: 'Write a book about Marionette'},
      {assignee: 'Andrew', text: 'Do some coding'}
    ]);
  }
});

var todo = new TodoList();
todo.render();
```


Une `CompositeView` est une `CollectionView` ayant son propre _template_. Nous définissons comment sa propriété `childView` doit être attaché au _template_ en utilisant `childViewContainer`, en lui passant le _template_.

Dans le fichier `templates/todolist.html` nous avons :

```html
<ul></ul>
<form>
  <label for="id_text">Todo Text</label>
  <input type="text" name="text" id="id_text" />
  <label for="id_assignee">Assign to</label>
  <input type="text" name="assignee" id="id_assignee" />

  <button id="btn-add">Add Item</button>
</form>
```


Nous n'avons pas besoin de modifier du tout le _template_ des entrées. Désormais, lorsque nous raffraichissons la page, nous aurons un formulaire à partir de quoi saisir des données, cliquez "Add Item" and... rien n'arrive ! _Marionette_ ne sait pas quel sont les éléments à observer, nous allons donc le  renseigner.


## Relier les saisies utilisateur

Nous avons besoin de dire à _Marionette_ qu'il a besoin d'écouter les saisies utilisateur du formulaire et comment il doit répondre à cela. Nous réouvrons `driver.js` et commençons à l'éditer ainsi :

```js
// FIXME Expose underscore as global
_ = require( "underscore" );

var Backbone = require('backbone');
var Marionette = require('backbone.marionette');


var ToDo = Marionette.LayoutView.extend({
  tagName: 'li',
  template: './templates/todoitem.html'
});


var TodoList = Marionette.CompositeView.extend({  
  el: '#app-hook',
  template: require('./templates/todolist.html'),

  childView: ToDo,
  childViewContainer: 'ul',

  ui: {  // 1
    assignee: '#id_assignee',
    form: 'form',
    text: '#id_text'
  },

  triggers: {  // 2
    'submit @ui.form': 'add:todo:item'
  },

  collectionEvents: {  // 3
    add: 'itemAdded'
  },

  initialize: function() {
    this.collection = new Backbone.Collection([
      {assignee: 'Scott', text: 'Write a book about Marionette'},
      {assignee: 'Andrew': text: 'Do some coding'}
    ]);
  },

  onAddTodoItem: function() {  // 4
    this.collection.add({
      assignee: this.ui.assignee.val(),  // 5
      text: this.ui.text.val()
    });
  },

  itemAdded: function() {  // 6
    this.ui.assignee.val('');
    this.ui.text.val('');
  }
});

var todo = new TodoList();
todo.render();
```


Nous avons ajouter pas mal de code ici, alors tachons de découper cela :

  1. Nous avons ajouter un tableau associatif `ui` à notre vue. Nous pouvons ajouter cela à toute vue afin de créer des sélecteurs jQuery mis en cache de nos éléments  à soumettre au _template_ de la vue.
  2. A l'intérieur de notre tableau associatif `triggers`, nous pouvons obtenir une référence de des clés du _hash_ `ui` , lorsqu'un événement jQuery que nous écoutons se produit, le `trigger` le capture.
  3. L'attribut `collectionEvents` nous permet d'écouter les modifications qui se produisent sur la propriété `this.collection` reliée. La valeur doit exister en tant que méthode de la vue.
  4. Ce code de déclenchement `trigger` est ensuite convertit en une méthode `onEventName` qui est appelée. Cette méthode doit pré-exister et est vraiment très puissante. Nous couvrirons son usage plus en détails par la suite.
  5. Nous pouvons aussi référencé le _hash_ `ui` au sein de la vue et l'utiliser comme n'importe quel objet jQuery.
  6. La méthode référencée dans `collectionEvents` est notifiée lorsque l'événement est déclenché.

Maintenant, chaque fois que nous cliquons sur le bouton  "Add Item", une nouvelle tache sera ajoutée à notre liste et le formulaire sera ré-initialisé. Nous avions eu aussi l'opportunité de vous montrer aussi les événements provenant des collections aussi. Pöpur une liste complète des événements, veuillez vous rendre à la [documentation Backbone][eventlist].


## Valider la saisie

Un travail ne devrait pas pouvoir être ajouté à la liste, s'il contient ni texte, ni personne associée. Vous noterez que nous n'avons pas ajouter une telle contrainte, mais nous devrions le faire. Il y a une série de façon de le faire :
- Nous pouvons utiliser l'attribut `ui` et valider son contenu via jQuery
- ou nous pouvons utiliser la validation des `Model` de _Backbone_.

A part apparaître plus sympa, l'avantage d'effectuer la validation de données depuis le modèle, est que nous pourrons le partager cette référence entre plusieurs vues, sans avior à ré-écrire la logique de validation à chaque fois.

Nou sallons créer un enouveau fichier  nommé `models/todo.js` contenant :

```js
var Backbone = require('backbone');


var ToDo = Backbone.Model.extend({
  defaults: {
    assignee: '',
    text: ''
  },

  validate: function(attrs) {
    var errors = {};
    var hasError = false;
    if (!attrs.assignee) {
      errors.assignee = 'assignee must be set';
      hasError = true;
    }
    if (!attrs.text) {
      errors.text = 'text must be set';
      hasError = true;
    }

    if (hasError) {
      return errors;
    }
  }
});


module.exports = ToDo;
```

Lorsqu'aucune erreur de ne survient, notre méthode  `validate` ne doit rien retourner (`undefined`).
Lorsqu'une erreur aarive, nous poucvions retourner un objet la décrivant. La vue _Marionette_ verra ainsi l'erreur de validation et nous permettra de la manipuler.

De retour dans notre fichier `driver.js`  nous pouvons faire :

```js
// FIXME Expose underscore as global
_ = require( "underscore" );

var Backbone = require('backbone');
var Marionette = require('backbone.marionette');

var ToDoModel = require('./models/todo');


var ToDo = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require( './templates/todoitem.html' )
});


var TodoList = Marionette.CompositeView.extend({  
  el: '#app-hook',
  template: require('./templates/todolist.html'),

  childView: ToDo,
  childViewContainer: 'ul',

  ui: {
    assignee: '#id_assignee',
    form: 'form',
    text: '#id_text'
  },

  triggers: {
    'submit @ui.form': 'add:todo:item'
  },

  collectionEvents: {
    add: 'itemAdded'
  },

  initialize: function() {
    this.collection = new Backbone.Collection([
      {assignee: 'Scott', text: 'Write a book about Marionette'},
      {assignee: 'Andrew', text: 'Do some coding'}
    ]);
    this.model = new ToDoModel();
  },

  onAddTodoItem: function() {
    this.model.set({
      assignee: this.ui.assignee.val(),
      text: this.ui.text.val()
    }, {validate: true});

    var items = this.model.pick('assignee', 'text');
    this.collection.add(items);
  },

  itemAdded: function() {
    this.model.set({
      assignee: '',
      text: ''
    });

    this.ui.assignee.val('');
    this.ui.text.val('');
  }
});

var todo = new TodoList();
todo.render();
```

Avec ces changements, nous pouvons désormais refuser une nouvelle saisie, à moins qu'elle passe l'étape de validation. Nous pouvons aussi afficher un message d'erreur si la validation échoue, en reliant un événement `invalid` dans la propriété `modelEvents`.

Vous pouvez maintenant voir comment nous avons supprimer dans les champs de notre model et de notre hash `ui`.
> **Warning** Toutefois dès que nous commencerons à obtenir plus de champs, le tout devriendra assez rapidement difficilement gérable. A tous les coups, nous pouvons nettoyer le formulaire lorsque le model l'est aussi ?


## Rendu depuis les modèles

Révisant notre premier chapitre, nous étions capable d'afficher les données des champs d'un modèle. Nous utiliserons cela pour gérer notre formulaire également.
Tout d'abord, ouvrons notre _template_ `todolist.html` :

```html
<ul></ul>
<form>
  <label for="id_text">Todo Text</label>
  <input type="text" name="text" id="id_text" value="<%- text %>" />
  <label for="id_assignee">Assign to</label>
  <input type="text" name="assignee" id="id_assignee" value="<%- assignee %>"/>

  <button id="btn-add">Add Item</button>
</form>
```

En ajoutant les champs du modèle à notre formulaire, nous seront aptes à restituer les données directement deouis notre modèle. Cela signifie aussi que, lorsque les champs sont vides, ils ne contiendront pas de valeur.
Nous avons juste besoin de relier cela à notre vue :

```js
// FIXME Expose underscore as global
_ = require( "underscore" );
var Backbone = require('backbone');
var Marionette = require('backbone.marionette');

var ToDoModel = require('./models/todo');


var ToDo = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require( './templates/todoitem.html' )
});


var TodoList = Marionette.CompositeView.extend({  
  el: '#app-hook',
  template: require('./templates/todolist.html'),

  childView: ToDo,
  childViewContainer: 'ul',

  ui: {
    assignee: '#id_assignee',
    form: 'form',
    text: '#id_text'
  },

  triggers: {
    'submit @ui.form': 'add:todo:item'
  },

  collectionEvents: {
    add: 'itemAdded'
  },

  modelEvents: {
    change: 'render'
  },

  initialize: function() {
    this.collection = new Backbone.Collection([
      {assignee: 'Scott', text: 'Write a book about Marionette'},
      {assignee: 'Andrew', text: 'Do some coding'}
    ]);
    this.model = new ToDoModel();
  },

  onAddTodoItem: function() {
    this.model.set({
      assignee: this.ui.assignee.val(),
      text: this.ui.text.val()
    }, {validate: true});

    var items = this.model.pick('assignee', 'text');
    this.collection.add(items);
  },

  itemAdded: function() {
    this.model.set({
      assignee: '',
      text: ''
    });
  }
});

var todo = new TodoList();
todo.render();
```

Avec ces changements, le formulaire va se reinitialisé tout seul dès que l'utilisateur acrtive le bouton  "Add Item".
Toutefois, il y a une dernière chose à noter - la méthode `render` redesssine la totalité de la liste aussi. Vous pouvez probablement imaginer que cela va vite devenir un goulet d'étranglement, plus la liste grossira. Idéalement, nous souhaitons juste redessiné le formulaire et la liste séparement. Nous verrons cela au prochain chapitre.


[eventlist]: http://backbonejs.org/#Events-catalog "Backbone Events"
