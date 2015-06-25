# Hello, world

Nous allons commencer par définir une `View` très simple qui se contente d'adresser nos salutations.


## Créons notre `View`

Commençons ! Ouvrez `driver.js` et ajoutez le code suivant :

```js
var Marionette = require('backbone.marionette');  // 1


var HelloWorld = Marionette.LayoutView.extend({  // 2
  el: '#app-hook',  // 3
  template: require('./templates/layout.html')  // 4
});

var hello = new HelloWorld();  // 5
hello.render();  // 6
```

Ensuite, nous créons un fichier dans le dossier `templates` called `layout.html` et le définissons ainsi :

```html
<p>Hello, world!</p>
```
> **todo** Ajoutez à _browserify_ dans `package.json`  une _transform_ pour les templates _underscore_
```sh
npm install --save-dev node-underscorify
```
```json
"build": "./node_modules/browserify/bin/cmd.js -t node-underscorify app/driver.js -o static/js/app.js"
```

### Que cela signifie t-il ?

Le fichier _template_ lui-même est plutôt compréhensible, intéressons-nous à `driver.js`:

  1. Import de _Marionette_
  2. Création d'une nouvelle typoe de `View` nommée `HelloWorld` qui provient d'une extension de _Marionette_ `LayoutView`. Nous entrerons dans plus de détails dans un moment.
  3. Nous attachons la vue à un élément du _DOM_. Il s'agit en fait d'un sélecteur jQuery, nous pouvons utiliser à cet endroit n'importe quel selecteur jQuery valide.
  4. Nous devons définir un _template_ à afficher à nos utilisateurs.
  5. Nous devons créer une instance de notre classe `HelloWorld` avant de pouvoir faire quoique ce soit d'utile avec.
  6. Désormais la parade commence ! Nous appelons la méthode `render()` pour afficher le _template_ à l'écran.


### Lancer la compilation

Après avoir [compilé le résultat][installing],  rendez-vous depuis votre navigateur sur l'URL `index.html`  pour observer l'affichage produit.
Vous venez de construire votre première application Javascript avec `Marionette` !


## Construire une application de taches

Désormais, nous avons une `View`, faisons quelque chose d'un peu plus intéressant en restituant une liste prédéfinie de taches. Nous allons avoir besoin d'un [`Backbone Model`][models] pour sauvegarder la liste de telle façon que que notre `LayoutView` puisse y accéder. Ré-ouvrez notre fichier `driver.js` :

```js
// FIXME Expose underscore as global
_ = require( "underscore" );

var Backbone = require('backbone');
var Marionette = require('backbone.marionette');


var TodoList = Marionette.LayoutView.extend({  
  el: '#app-hook',  
  template: require('./templates/layout.html'),

  initialize: function() {
    this.model = new Backbone.Model({
      items: [
        {assignee: 'Scott', text: 'Write a book about Marionette'},
        {assignee: 'Andrew', text: 'Do some coding'}
      ]
    });
  }
});

var todo = new TodoList();
todo.render();  
```

Le point central de cette modification tient l'insertion d'un objet en une instance de `Backbone.Model`. Les modèles _Backbone_ s'intègrent aisément avec les _templates_, conférant un accès simple à la source de données. Nous verrons dans plusieurs chapitres à quel point les modèles peuvent être puissants en ce qui concerne nos vues.

Modifions notre fichier `templates/layout.html` pour prendre avantage de ce cahngement :

```js
<ul>
  <% _.each(items, function(item) { %>
    <li><%- item.text %> &mdash; <%- item.assignee %>
  <% }) %>
</ul>
```

Nous utilisons [le moteur de templateUnderscore][underscore] pour restituer notre liste d'éléments. Vous devriez obtenir une liste de deux éléments.
Nous avons désormais une liste rendue dans la page, mais cela semble un peu maladroit. Qu'arrive t-il si nous souhaitons ajouter ou supprimer des entrées à cette liste ?


## `Collections` and `CollectionViews`

Notre liste d'entrées correspond vraiment à une collection, nous allons donc utiliser une  `Backbone.Collection` pour la modéliser. Nous allons également utiliser une vue se spécialisant dans la restituion de liste de données -
la `CollectionView`. Retour à notre fichier `driver.js`, nous allons utiliser plusieurs vues :

```js
// FIXME Expose underscore as global
_ = require( "underscore" );

var Backbone = require('backbone');
var Marionette = require('backbone.marionette');


var ToDo = Marionette.LayoutView.extend({
  tagName: 'li',
  template: './templates/todoitem.html'
});


var TodoList = Marionette.CollectionView.extend({  
  el: '#app-hook',
  tagName: 'ul',

  childView: ToDo,

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

La première chose que nous avoins fait, est d'ajouter une nouvelle vue et de la relier à une
`CollectionView` en utilisant l'attribut `childView`. Nous avons également modifié la propriété
`this.model` pour `this.collection` au sein de la méthode `initialize` de notre nouvelle `CollectionView` et supprimé la clé `items`.

La `CollectionView` est une vue qui parcoure `this.collection` et restitue une instance de son attribut `childView` pour chacune des entrées trouvées.  Nous avons supprimé la propriété  `template` car la `CollectionView` n'a plus de  _template_ en soi.

Dans `templates/todoitem.html` nous avons simplement :

```html
<%- text %> &mdash; <%- assignee %>
```


C'est tout ! _Marionette_ sait comment construire l'enveloppe des éléments `ul` et `li` et insère les données de l'itération pour nous. Encore mieux, si nous voulions ajouter/supprimer des entrées de la collecion, `CollectionView` le repère et le restitue automatiquement pour nous.

Dans le prochain chapitre, nous allons ajouter de nouvelles entrées à cette collection, de telle manière à garder l'état de notre liste de taches.

[installing]: ../installing_marionette.md "Installing Marionette"
[models]: ./models.md "Storing user-entered data"
[underscore]: http://underscorejs.org/#template "Underscore.js"
