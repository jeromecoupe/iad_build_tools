# Asset pipeline et scripts NPM

## Asset pipeline

Le développement front-end est aujourd'hui relativement complexe et disposer d'automatisations fluidifie grandement le processus. On appelle asset pipeline la création d'opérations automatisées permettant de convertir une série de fichiers sources en des fichiers optimisés pour être déployés en production.

Ces opérations concernent les trois grands langages du web (HTML/CSS/JS) ainsi que les assets (images, videos, etc.).

- CSS: rassembler une série de petits fichiers Sass ou CSS en un seul fichier optimisé (bundling), éliminer en production les règles CSS non utilisées, valider le code CSS (linting), etc.
- JavaScript: rassembler plusieurs fichiers en un seul fichier optimisé (bundling), automatiser la transformation de code plus moderne vers du code pouvant être utilisé par des navigateurs plus anciens (transpiling), éliminer du code de production les parties de code qui ne sont pas utilisées (tree-shaking), valider le code JavaScript (linting), vérifier que votre code passe une série de tests (testing), etc.
- Images: optimisation et génération de formats, création de sprites, etc.

Nous nous concentrerons ici sur l'utilisation des scripts NPM qui permettent, via un fichier `package.json` et la commande `npm run [script name]`, de centraliser et d'exécuter scripts d'invite de commande.

Au niveau des avantages, les scripts NPM permettent d'utiliser l'ensemble du (très) large écosystème des packages NPM, de spécifier l'ordre d'exécution de vos différents scripts et commandes, et même d'en faire tourner certains en parallèle ou en série. Le fait de travailler avec des packages natifs est aussi un avantage, comparé à des solutions comme grunt ou gulp qui doivent passer par des wrappers propre à ces outils de build.

Si vous devez construire un build pipeline complexe, utiliser un ensemble de scripts en ligne de commande qui doivent être exécuté dans un ordre spécifique peut devenir plus difficile. Certains critiquent également le caractère peu lisible de ces scripts et la difficulté de commenter un fichier JSON tel que `package.json`.

## NPM scripts

### Création d'un package.json

Les scripts NPM sont donc essentiellement une façon de centraliser et d'exécuter des scripts d'invite de commande. Ils sont initialement prévu pour fonctionner dans un écosystème NodeJS mais peuvent également être utilisés dans le contexte plus large du développement web.

Pour commencer à travailler, nous devons [installer NodeJS et NPM](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) et créer un fichier `package.json` qui reprendra toutes nos dépendances Node et nous permettra d'écrire et d'exécuter des scripts.

```text
npm init -y
```

Le flag `-y` permet de créer un fichier `package.json` par défaut. Vous pourrez éditer votre fichier par la suite. Voici la [documentation complète](https://docs.npmjs.com/files/package.json) pour vous aider à comprendre quels sont les paramètres et options disponibles.

### Principales commandes

Nous devrons également installer un certain nombre de packages NPM utilisés par nos scripts:

- `npm install [package(s)]` ou `npm i [package(s)]`: installe les packages nécessaires en production
- `npm install [package(s)] --save-dev` ou `npm i -D [package(s)]`: installe les packages nécessaires uniquement en développement local et pour des tests

_Exemple: la commande `NPM i -D browser-sync` installera BrowserSync comme dépendance de développement et spécifiera une plage des versions de ce package pouvant être utilisées dans le fichier `package.json`._

- `npm uninstall [package(s)]`: vous permet de désinstaller un ou plusieurs packages NPM.
- `npm outdated` ou `npm outdated [package(s)]`: vous permet de voir quelle est la dernière version stable d'un ou plusieurs packages installés

### Scripts NPM

La propriété [`scripts`](https://docs.npmjs.com/cli/v7/using-npm/scripts) du fichier `package.json` permet de créer des scripts qui vont pouvoir être exécutés grâce aux commandes `npm run-script [key]` ou `npm run [key]`.

```json
{
  "scripts": {
    "styles": "sass src/assets/scss/main.scss dist/assets/css/main.css"
  }
}
```

_Exemple: le script ci-dessus pourra être exécuté avec la commande `npm run styles` et aura pour effet de compiler le contenu du fichier source `main.scss` vers le fichier de destination `main.css`_

NPM offre des commandes courtes pour certains scripts très utilisés:

- `npm start` équivaut à `npm run start`
- `npm test` équivaut à `npm run test`

Les prefixes `pre` et `post` placé avant le nom d'un script existant ou d'un script par défaut permettent de créer de nouveaux scripts qui seront exécuté automatiquement avant ou après le script existant.

```json
{
  "scripts": {
    "prestyles": "script exécuté automatiquement avant le script 'styles'",
    "styles": "sass src/assets/scss/main.scss dist/assets/css/main.css",
    "poststyles": "script exécuté automatiquement après le script 'styles'"
  }
}
```

Il existe également des packages NPM comme [`npm-run-all`](https://www.npmjs.com/package/npm-run-all) qui permettent d'exécuter des scripts soit en série, soit en parallèle.

### Développement et production

Il se peut que vous ayez besoin de deux version différentes du même script suivant que vous soyez en développement ou en production. Par exemple, vous souhaitez avec des sourcemaps pour votre compilation CSS en développement mais pas en production

Pour cela, j'utilise personnellement la notation suivante:

```json
{
  "name": "npm-scripts",
  "version": "1.0.0",
  "description": "Learning NPM scripts",
  "scripts": {
    "styles:dev": "sass --watch --embed-source-map --source-map-urls=absolute \"src/scss/main.scss\" \"dist/css/styles.css\"",
    "styles:prod": "sass --no-source-map \"src/scss/main.scss\" \"dist/css/styles.css\""
  },
  "keywords": ["NPM Scripts", "asset pipeline", "build", "watch"],
  "author": "Jérôme Coupé",
  "license": "MIT",
  "devDependencies": {
    "sass": "^1.43.4"
  }
}
```

Typiquement, `styles:prod` sera exécuté lors du passage en production, tandis que `styles:dev` sera pour sa part utilisé en développement.

## Exemple concret

Nous allons maintenant créer ensemble un asset pipeline assez classique qui devra remplir les missions suivantes:

- Compiler du code [Sass](https://sass-lang.com) en un seul fichier CSS, utiliser [postCSS](https://postcss.org/) et les plugins [autoprefixer](https://github.com/postcss/autoprefixer) et [cssnano](https://github.com/cssnano/cssnano) pour ajouter les vendor prefixes et minifier la css finale.
- Bundler nos modules JavaScript en un seul fichier et transpiler le tout vers de l'ES6 avec [esbuild](https://esbuild.github.io/)
- Optimiser nos images et utiliser un script externe et la librairie [Sharp](https://github.com/lovell/sharp) pour générer des thumbnails aux dimensions et formats voulus.

### Pipeline Sass / PostCSS

Voyons maintenant comment ajouter un pipeline Sass à notre projet:

Installer les packages NPM Sass et PostCSS, ainsi que les plugins autoprefixer et cssnano:

- `npm install --save-dev sass postcss-cli autoprefixer cssnano`
- `npm i -D sass sass postcss-cli autoprefixer cssnano`

En développement, nous devons compiler notre code Scss et obtenir une css avec sourcemaps.

En production, nous devons également compiler notre code Scss et obtenir un fichier CSS sans sourcemaps. Nous devons ensuite ajouter nos vendor prefixes et compresser notre fichier CSS avec [PostCSS et postCSS CLI](https://github.com/postcss/postcss), autoprefixer et cssnano.

```json
"scripts": {
  "styles:dev": "sass --embed-source-map --source-map-urls=absolute src/scss/main.scss dist/css/styles.css",
  "styles:prod": "sass --no-source-map src/scss/main.scss dist/css/styles.css",
  "poststyles:prod": "postcss \"dist/css/styles.css\" --replace --no-map --use autoprefixer cssnano",
},
```

Autoprefixer ayant besoin de savoir quels navigateurs et quelles versions supporter, nous pouvons ajouter une [query browserslist](https://github.com/browserslist/browserslist) à notre `package.json`. Voici celle que j'utilise habituellement.

```json
"scripts": {
  "styles:dev": "sass --embed-source-map --source-map-urls=absolute src/scss/main.scss dist/css/styles.css",
  "styles:prod": "sass --no-source-map src/scss/main.scss dist/css/styles.css",
  "poststyles:prod": "postcss \"dist/css/styles.css\" --replace --no-map --use autoprefixer cssnano",
},
"browserslist": [
  "> 0.5%",
  "not IE 11",
  "not dead"
],
```

### Pipeline JS

Même si les modules JS commencent à être bien supportés et même si HTTP/2 aide également, il est encore utile de bundler (rassembler en une seul fichier) et de minifier votre code JavaScript.

Même si il est assez facile d'utiliser [webpack](https://webpack.js.org/) et un fichier de configuration, [esbuild](https://esbuild.github.io/) (et ES6) sont des alternatives intéressantes pour la plupart de vos projets, spécialement si vous les utilisez uniquement pour JavaScript.

```json
"scripts": {
  "styles:dev": "sass --embed-source-map --source-map-urls=absolute \"src/scss/main.scss\" \"dist/css/styles.css\"",
  "styles:prod": "sass --no-source-map \"src/scss/main.scss\" \"dist/css/styles.css\"",
  "poststyles:prod": "postcss \"dist/css/styles.css\" --replace --no-map --use autoprefixer cssnano",
  "scripts:prod": "esbuild \"src/js/main.js\" --target=\"es6\" --bundle --minify --outfile=\"dist/js/main.bundle.js\"",
  "scripts:dev": "esbuild \"src/js/main.js\" --target=\"es6\" --bundle --outfile=\"dist/js/main.bundle.js\""
},
"browserslist": [
  "> 0.5%",
  "not IE 11",
  "not dead"
],
```

## Clear, build, watch et rechargement automatique

En phase de développement, les divers fichiers de destination (CSS, JavaScript et assets) sont générés automatiquement à chaque fois une modification des divers fichiers est détectée. C'est ce que l'on appelle les opération de watch. Certains packages disposent d'un flag `--watch` et d'autres pas forcément. Nous utiliserons [`onchange`](https://www.npmjs.com/package/onchange) pour monitorer certains dossiers et exécuter certains scripts en conséquence.

Ces tâches et scripts peuvent être activés séquentiellement ou en parallèle. Nous utiliserons [`npm-run-all`](https://www.npmjs.com/package/npm-run-all) pour cela.

Il est également courant de disposer d'un petit serveur local ([Browsersync](https://browsersync.io/) dans ce cas-ci) permettant de recharger automatiquement les pages dans le navigateur à chaque modification des fichiers sources et de tester ses projets sur plusieurs plateformes simultanément.

Vous aurez souvent besoin de copier des fichiers (fonts, etc) depuis un dossier source vers une dossier de destination. [`copyfiles`](https://www.npmjs.com/package/copyfiles) est un outil très efficace pour cela.

Avant la phase de build, il est courant de supprimer l'ensemble des fichiers de destination. Nous utiliserons [rimraf](https://www.npmjs.com/package/rimraf) pour exécuter cette tâche.

```json
{
  "name": "npm-scripts",
  "version": "1.0.0",
  "description": "Learning NPM scripts",
  "author": "Jérôme Coupé",
  "license": "MIT",
  "scripts": {
    "clear": "rimraf \"dist/\"",
    "serve": "browser-sync start --server --no-open --files \"dist\" \"*.html\"",
    "styles:dev": "sass --embed-source-map --source-map-urls=absolute \"src/scss/main.scss\" \"dist/css/styles.css\"",
    "styles:prod": "sass --no-source-map \"src/scss/main.scss\" \"dist/css/styles.css\"",
    "poststyles:prod": "postcss \"dist/css/styles.css\" --replace --no-map --use autoprefixer cssnano",
    "scripts:prod": "esbuild \"src/js/main.js\" --target=es6 --bundle --minify --outfile=\"dist/js/main.bundle.js\"",
    "scripts:dev": "esbuild \"src/js/main.js\" --target=es6 --bundle --outfile=\"dist/js/main.bundle.js\"",
    "watch:styles": "onchange \"src/scss/**/*.scss\" -- npm run styles:dev",
    "watch:scripts": "onchange \"src/js/**/*.js\" -- npm run scripts:dev",
    "copy:fonts": "copyfiles -u 1 \"src/fonts/*\" \"dist/\"",
    "copy": "npm-run-all --parallel copy:*",
    "watch": "npm-run-all --parallel serve watch:*",
    "build": "npm-run-all clear --parallel styles:prod scripts:prod images copy",
    "dev": "npm run watch"
  },
  "browserslist": ["> 0.5%", "not IE 11", "not dead"],
  "devDependencies": {
    "autoprefixer": "^10.4.0",
    "browser-sync": "^2.27.5",
    "copyfiles": "^2.4.1",
    "cssnano": "^5.0.9",
    "esbuild": "^0.13.12",
    "glob": "^7.2.0",
    "npm-run-all": "^4.1.5",
    "onchange": "^7.1.0",
    "postcss-cli": "^9.0.1",
    "rimraf": "^3.0.2",
    "sass": "^1.43.4",
    "sharp": "^0.29.2"
  }
}
```

## Scripts externes

Il est aussi possible de faire tourner des scripts Node externes à l'aide de scripts NPM.

```json
"scripts": {
  "external": "node \"build_tasks/myscript.js\"",
}
```

De tels scripts externes peuvent par exemple être utilisés en combinaison avec des Static Sites Generators pour [créer différentes versions optimisées de vos images](https://github.com/jeromecoupe/webstoemp/blob/master/build_tasks/transform-images.js) lors du build.
