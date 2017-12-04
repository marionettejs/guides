# Faire dancer votre application !

Bienvenue dans le guide de démarrage pour `Backbone Marionette`.


## Que vais-je découvrir dans ce tutoriel ?

Après avoir lu ce tutoriel, vous serez capable de débuter une application `Marionette` complète. Nous allons construire une simple liste de tâches, dans laquelle il est possible d'ajouter et de retirer des éléments.

Ce tutoriel vous apprendra comment
- associer des données à vos `Views`,
- architecturer votre application,
- conduire les utilisateurs via la barre d'_URL_
- synchroniser des données en provenance d'un serveur.


## A savoir

Avant de commencer, nous allons définir une structure simple afin de tout ordonner. Nous allons créer un répertoire dédié aux `Collection`, `Model`, `Router`, `template`  et `View`. La structure se compose ainsi :

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

A chaque fois que nous ferons référence à un fichier, nous réfèrerons à son nom et à son répertoire parent.

Notre `index.html` :

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

Si vous ne l'avez déjà fait, rendez-vous à la section [Installation de Marionette](../installing_marionette.md) pour des instructions sur la façon d'installer une application _Marionette_.

En option, utilisez un _framework CSS_ tel que [Bootstrap](https://getbootstrap.com) pour améliorer l'apparence de votre application.
Ce tutoriel conserve pour sa part une structure HTML la plus simple.

Ce tutoriel vous demandera de "refactorer" fréquemment le code des chapitres précédents. Tous les détails seront décrits pas à pas sur ce qui évolue, ainsi, si vous ne voyez pas apparaître un fichier, c'est qu'il n'a pas changé.

> **Note** Experimentation :
A travers ce tutoriel, sentez-vous libre d'experimenter à partir des exemples, afin d'appréhender ce qu'il se produit. Tentez différentes options, _templates_, classes, ou même des logiques personnalisées.
