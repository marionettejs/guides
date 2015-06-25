# Installation de _Marionette_

## Modularité

Une approche modulaire est à privilégier d'emblée pour la complexité future de votre application. Le Javascript ne proposent pas nativement de gestion de dépendances (quoique l'[ES6](https://leanpub.com/exploring-es6/read#ch_modules) en propose à présent), il est possible de faire appel soit
- aux modules _AMD_, via [require.js](http://requirejs.org/)
- aux modules _CommonJS_, grâce à [browserify](http://browserify.org/)  


## Workflow de publication

Dans les deux cas, il est important d'installer un environement [node.js](http://nodejs.org/) et d'établir un _workflow_ de publication automatisé, par exemple depuis les scripts [npm](https://www.npmjs.com/).

Suivez [ce guide complet](https://github.com/joyent/node/wiki/Installation) en choisissant votre plateforme de développement.


## Browserify

Nous optons pour l'emploi de la librarie _browserify_.
Pour cela, nous allons nous inspirer du [dossier du cookbook repository](https://github.com/MarionetteLabs/marionette-cookbook/tree/master/recipes/architectures/browserify).

- Définissez un nouveau projet dans un répertoire dédié
```sh
npm init
```
Suivez les étapes de la ligne de commande.
- Installez _browserify_ depuis le `package.json`
```sh
npm install --savedev browserify
```
- Installez les libraries depuis le `package.json`
```sh
npm install --save-dev
```
- Installez les dépendances
```sh
npm install underscore --save
npm install jquery --save
npm install backbone --save
npm install backbone.marionette --save
```
- Placez ce code dans la propriété `script` de `package.json`
```js
"build": "./node_modules/browserify/bin/cmd.js app.js -o build.js"
```
- Ajoutez un fichier `index.html`
```html
<html>
  <body></body>
  <script src="build.js"></script>
</html>
```
- Développez votre application depuis un fichier `app.js`
- Compilez votre code source dès que vous souhaitez le tester
```sh
npm run build
```
