
# Modules, introduction

Au fur et à mesure que notre application grandit, nous souhaitons la scinder en plusieurs fichiers, appelés «modules». Un module contient généralement une classe ou une bibliothèque de fonctions.

Pendant longtemps, JavaScript n'avait pas de module. Ce n’était pas un problème, car au départ les scripts étaient petits et simples, il n’était donc pas nécessaire.

Mais les scripts sont devenus de plus en plus complexes et la communauté a donc inventé diverses méthodes pour organiser le code en modules, des bibliothèques spéciales pour charger des modules à la demande.

Par exemple:

- [AMD](https://en.wikipedia.org/wiki/Asynchronous_module_definition) -- un des systèmes de modules les plus anciens, initialement mis en œuvre par la bibliothèque [require.js](http://requirejs.org/).
- [CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1) -- le système de module créé pour Node.js
- [UMD](https://github.com/umdjs/umd) -- un système de module supplémentaire, proposé comme universel, compatible avec AMD et CommonJS

Maintenant, tous ces éléments deviennent lentement du passé, mais nous pouvons toujours les trouver dans d’anciens scripts.

Le système de modules natif apparu dans la norme en 2015, a progressivement évolué depuis et est maintenant pris en charge par tous les principaux navigateurs et dans Node.js. Nous allons donc l’étudier à partir de maintenant.

## Qu'est-ce qu'un module?

Un module est juste un fichier. Un script est un module.

Les modules peuvent se charger mutuellement et utiliser des directives spéciales, `export` et `import`, pour échanger des fonctionnalités, appeler les fonctions d’un module dans un autre:

- `export` permet d'indiquer les variables et fonctions devant être accessible en dehors du document.
- `import` permet d'importer les variables et fonctions d'autres modules.

Par exemple, si nous avons un fichier `sayHi.js` exportant une fonction

```js
// 📁 sayHi.js
export function sayHi(user) {
  alert(`Hello, ${user}!`);
}
```

...Un autre fichier peut l'importer et l'utiliser:

```js
// 📁 main.js
import {sayHi} from './sayHi.js';

alert(sayHi); // function...
sayHi('John'); // Hello, John!
```

La directive `import` charge le module qui a pour chemin `./sayHi.js` par rapport au fichier actuel et affecte la fonction exportée `sayHi` à la variable correspondante.

Lançons l’exemple dans le navigateur.

<<<<<<< HEAD
Comme les modules prennent en charge des mots-clés et des fonctionnalités spéciales, nous devons indiquer au navigateur qu'un script doit être traité comme un module, en utilisant l'attribut `<script type="module">`.
=======
As modules support special keywords and features, we must tell the browser that a script should be treated as a module, by using the attribute `<script type="module">`.
>>>>>>> 2b5ac971c1bd8abe7b17cdcf724afd84799b6cbd

Comme ça:

[codetabs src="say" height="140" current="index.html"]

Le navigateur extrait et évalue automatiquement le module importé (et, le cas échéant, ses importations), puis exécute le script.

## Caractéristiques du module de base

Qu'est-ce qui est différent dans les modules par rapport aux scripts "normaux"?

Il existe des fonctionnalités de base, valables à la fois pour le navigateur et le JavaScript côté serveur.

### Toujours en mode "use strict"

Les modules utilisent `use strict`, par défaut. Par exemple. assigner à une variable non déclarée donnera une erreur.

```html run
<script type="module">
  a = 5; // error
</script>
```

### Portée au niveau du module

Chaque module a sa propre portée globale. En d'autres termes, les variables et les fonctions globales d'un module ne sont pas visibles dans les autres scripts.

Dans l'exemple ci-dessous, deux scripts sont importés et `hello.js` essaie d'utiliser la variable `user` déclarée dans `user.js` et échoue:

[codetabs src="scopes" height="140" current="index.html"]

Les modules sont censés `export` ce qui doit être accessible de l'extérieur et `import` ce dont ils ont besoin.

Nous devons donc importer `user.js` dans `hello.js` et en tirer les fonctionnalités requises au lieu de nous fier à des variables globales.

Un exemple:

[codetabs src="scopes-working" height="140" current="hello.js"]

Dans le navigateur, un environnement indépendant existe également pour chaque `<script type ="module">`:

```html run
<script type="module">
  // La variable est uniquement visible dans ce module
  let user = "John";
</script>

<script type="module">
  *!*
  alert(user); // Error: user is not defined
  */!*
</script>
```

Si nous devons réellement créer une variable globale, nous pouvons l’affecter explicitement à `window` et y accéder en tant que `window.user`. Mais c’est une exception qui nécessite une bonne raison.

### Un code de module est chargé la première fois lorsqu'il est importé

Si le même module est importé dans plusieurs autres emplacements, son code n'est exécuté que la première fois, puis les exportations sont données à tous les importateurs.

Cela a des conséquences importantes. Voyons cela sur des exemples.

Premièrement, si exécuter un code de module entraîne des effets secondaires, comme afficher un message, l'importer plusieurs fois ne le déclenchera qu'une seule fois - la première fois:

```js
// 📁 alert.js
alert("Module is evaluated!");
```

```js
// Importer le même module à partir de fichiers différents

// 📁 1.js
import `./alert.js`; // le module est chargé

// 📁 2.js
import `./alert.js`; // (n'affiche rien)
```

En pratique, le code dans l'environnement global du module est principalement utilisé pour l'initialisation, la création de structures de données internes mais, si nous voulons que quelque chose soit réutilisable, exportez-le.

Maintenant, un exemple plus avancé.

Disons qu'un module exporte un objet:

```js
// 📁 admin.js
export let admin = {
  name: "John"
};
```

Si ce module est importé à partir de plusieurs fichiers, il n'est chargé que la première fois, un objet `admin` est créé, puis transmis à tous les autres importateurs.

Tous les importateurs obtiennent exactement le seul et unique objet `admin`:

```js
// 📁 1.js
import {admin} from './admin.js';
admin.name = "Pete";

// 📁 2.js
import {admin} from './admin.js';
alert(admin.name); // Pete

*!*
// 1.js et 2.js ont importé le même objet
// Les modifications apportées dans 1.js sont visibles dans 2.js
*/!*
```

Donc, répétons-le, le module n’est exécuté qu’une fois. Les exportations sont générées, puis partagées entre les importateurs. Par conséquent, si quelque chose change d'objet `admin`, les autres modules le verront.

Un tel comportement permet de configurer des modules lors de la première importation. Nous pouvons configurer ses propriétés une fois, puis dans les importations ultérieures, il est prêt.

<<<<<<< HEAD
Par exemple, le module `admin.js` peut fournir certaines fonctionnalités, mais attendez-vous à ce que les informations d'identification entrent dans l'objet `admin` de l'extérieur:
=======
For instance, the `admin.js` module may provide certain functionality, but expect the credentials to come into the `admin` object from outside:
>>>>>>> 2b5ac971c1bd8abe7b17cdcf724afd84799b6cbd

```js
// 📁 admin.js
export let admin = { };

export function sayHi() {
  alert(`Ready to serve, ${admin.name}!`);
}
```

Dans `init.js`, le premier script de notre application, nous définissons `admin.name`. Ensuite, tout le monde le verra, y compris les appels passés depuis `admin.js` lui-même:

```js
// 📁 init.js
import {admin} from './admin.js';
admin.name = "Pete";
```

Un autre module peut aussi voir `admin.name`:

```js
// 📁 other.js
import {admin, sayHi} from './admin.js';

alert(admin.name); // *!*Pete*/!*

sayHi(); // Prêt à être utilisé, *!*Pete*/!*!
```

### import.meta

L'objet `import.meta` contient les informations sur le module actuel.

Son contenu dépend de l'environnement. Dans le navigateur, il contient l'URL du script ou une URL de page Web actuelle si elle est en HTML:

```html run height=0
<script type="module">
  alert(import.meta.url); // URL du script (URL de la page HTML)
</script>
```

### Dans un module, "this" n'est pas défini

C’est un peu une caractéristique mineure, mais pour être complet, nous devrions le mentionner.

Dans un module, l'objet global `this` est indéfini.

Comparez-le à des scripts sans module, là où il est un object global:

```html run height=0
<script>
  alert(this); // window
</script>

<script type="module">
  alert(this); // undefined
</script>
```

## Fonctionnalités spécifiques au navigateur

Il existe également plusieurs différences de scripts spécifiques au navigateur avec `type="module"` par rapport aux scripts classiques.

Vous devriez peut-être ignorer cette section pour l'instant si vous lisez pour la première fois ou si vous n'utilisez pas JavaScript dans un navigateur.

### Les modules sont différés

Les modules sont *toujours* différés, avec le même effet que l'attribut `defer` (décrit dans le chapitre [](info:script-async-defer)), pour les scripts externes et intégrés.

<<<<<<< HEAD
En d'autres termes:
- télécharger des modules externe `<script type="module" src="...">` ne bloque pas le traitement HTML, ils se chargent en parallèle avec d’autres ressources.
- Les modules attendent que le document HTML soit complètement prêt (même s'ils sont minuscules et se chargent plus rapidement que HTML), puis s'exécutent.
- l'ordre relatif des scripts est maintenu: les scripts qui entrent en premier dans le document sont exécutés en premier.
=======
In other words:
- downloading external module scripts `<script type="module" src="...">` doesn't block HTML processing, they load in parallel with other resources.
- module scripts wait until the HTML document is fully ready (even if they are tiny and load faster than HTML), and then run.
- relative order of scripts is maintained: scripts that go first in the document, execute first.
>>>>>>> 2b5ac971c1bd8abe7b17cdcf724afd84799b6cbd

Comme effet secondaire, les modules "voient" toujours la page HTML entièrement chargée, y compris les éléments HTML situés en dessous.

Par exemple:

```html run
<script type="module">
*!*
  alert(typeof button); // object: le script peut 'voir' le bouton ci-dessous
*/!*
  // à mesure que les modules sont différés, le script s'exécute après le chargement de la page entière
</script>

Comparez au script habituel ci-dessous:

<script>
*!*
  alert(typeof button); // Error: button is undefined, le script ne peut pas voir les éléments ci-dessous
*/!*
  // les scripts normaux sont exécutés immédiatement, avant que le reste de la page ne soit traité
</script>

<button id="button">Button</button>
```

<<<<<<< HEAD
Remarque: le deuxième script fonctionne avant le premier! Nous verrons donc d'abord `undefined`, puis `object`.

C’est parce que les modules sont différés, nous attendons donc que le document soit traité. Les scripts réguliers s'exécutent immédiatement, nous avons donc vu son resultat en premier.
=======
Please note: the second script actually runs before the first! So we'll see `undefined` first, and then `object`.

That's because modules are deferred, so we wait for the document to be processed. The regular script runs immediately, so we see its output first.
>>>>>>> 2b5ac971c1bd8abe7b17cdcf724afd84799b6cbd

Lorsque vous utilisez des modules, vous devez savoir que la page HTML apparaît lors de son chargement et que les modules JavaScript s'exécutent par la suite, afin que l'utilisateur puisse voir la page avant que l'application JavaScript soit prête. Certaines fonctionnalités peuvent ne pas encore fonctionner. Nous devons définir des "indicateurs de chargement" ou veiller à ce que le visiteur ne soit pas confus.

### Async fonctionne sur les scripts en ligne

Pour les scripts non modulaires, l'attribut `async` ne fonctionne que sur les scripts externes. Les scripts asynchrones s'exécutent immédiatement lorsqu'ils sont prêts, indépendamment des autres scripts ou du document HTML.

<<<<<<< HEAD
Pour les modules, cela fonctionne sur tous les scripts.

Par exemple, le script ci-dessous est `async` et n’attend donc personne.

Il effectue l'importation (récupère `./analytics.js`) et s'exécute lorsqu'il est prêt, même si le document HTML n'est pas encore terminé ou si d'autres scripts sont toujours en attente.
=======
For module scripts, it works on inline scripts as well.

For example, the inline script below has `async`, so it doesn't wait for anything.

It performs the import (fetches `./analytics.js`) and runs when ready, even if the HTML document is not finished yet, or if other scripts are still pending.
>>>>>>> 2b5ac971c1bd8abe7b17cdcf724afd84799b6cbd

C’est bon pour une fonctionnalité qui ne dépend de rien, comme des compteurs, des annonces, des écouteurs d’événements au niveau du document.

```html
<!-- toutes les dépendances sont récupérées (analytics.js) et le script s'exécute -->
<!-- il n'attend pas le document ou d'autres balises <script> -->
<script *!*async*/!* type="module">
  import {counter} from './analytics.js';

  counter.count();
</script>
```

### Scripts externes

Les scripts externes de `type="module"` se distinguent sous deux aspects:

<<<<<<< HEAD
1. Les scripts externes avec le même `src` ne s'exécutent qu'une fois:
=======
1. External scripts with the same `src` run only once:
>>>>>>> 2b5ac971c1bd8abe7b17cdcf724afd84799b6cbd
    ```html
    <!-- le script my.js est récupéré et exécuté une seule fois -->
    <script type="module" src="my.js"></script>
    <script type="module" src="my.js"></script>
    ```

2. Les scripts externes extraits d’une autre origine (par exemple, un autre site) nécessitent [CORS](https://developer.mozilla.org/fr/docs/Web/HTTP/CORS) en-têtes, comme décrit dans le chapitre <info:fetch-crossorigin>. En d'autres termes, si un module est extrait d'une autre origine, le serveur distant doit fournir un en-tête `Access-Control-Allow-Origin` permettant l'extraction.
    ```html
    <!-- another-site.com doit fournir Access-Control-Allow-Origin -->
    <!-- sino, le script ne sera pas exécuté -->
    <script type="module" src="*!*http://another-site.com/their.js*/!*"></script>
    ```

    Cela garantit une meilleure sécurité par défaut.

### Aucun module "nu" autorisé

Dans le navigateur, `import` doit avoir une URL relative ou absolue. Les modules sans chemin sont appelés modules "nus". De tels modules ne sont pas autorisés lors de l'importation.

Par exemple, cette `import` n'est pas valide:
```js
import {sayHi} from 'sayHi'; // Error, "bare" module
// le module doit avoir un chemin, par exemple './sayHi.js'
```

<<<<<<< HEAD
Certains environnements, tels que Node.js ou les outils de bundle, autorisent les modules nus, sans chemin d'accès, car ils disposent de moyens propres de recherche de modules. Mais les navigateurs ne supportent pas encore les modules nus.
=======
Certain environments, like Node.js or bundle tools allow bare modules, without any path, as they have their own ways for finding modules and hooks to fine-tune them. But browsers do not support bare modules yet.
>>>>>>> 2b5ac971c1bd8abe7b17cdcf724afd84799b6cbd

### Compatibilité, “nomodule”

<<<<<<< HEAD
Les anciens navigateurs ne comprennent pas `type="module"`. Les scripts du type inconnu sont simplement ignorés. Pour eux, il est possible de fournir une solution de secours en utilisant l’attribut `nomodule`:
=======
Old browsers do not understand `type="module"`. Scripts of an unknown type are just ignored. For them, it's possible to provide a fallback using the `nomodule` attribute:
>>>>>>> 2b5ac971c1bd8abe7b17cdcf724afd84799b6cbd

```html run
<script type="module">
  alert("Runs in modern browsers");
</script>

<script nomodule>
  alert("Modern browsers know both type=module and nomodule, so skip this")
  alert("Old browsers ignore script with unknown type=module, but execute this.");
</script>
```

## Construire des outils

Dans la vie réelle, les modules de navigateur sont rarement utilisés sous leur forme "brute". Généralement, nous les regroupons avec un bundle tel que [Webpack](https://webpack.js.org/) et les déployons sur le serveur de production.

L'un des avantages de l'utilisation des bundles est -- qu'ils permettent de mieux contrôler la façon dont les modules sont résolus, permettant ainsi des modules nus et bien plus encore, comme les modules CSS / HTML.

Les outils de construction font ce qui suit:

<<<<<<< HEAD
1. Prenons un module «principal», celui qui est destiné à être placé dans `<script type="module">` en HTML.
2. Analyser ses dépendances: importations puis importations d'importations etc.
3. Construisez un seul fichier avec tous les modules (ou plusieurs fichiers réglables), en remplaçant les appels `import` natifs par des fonctions d’assemblage, pour que cela fonctionne. Les types de modules "spéciaux" tels que les modules HTML / CSS sont également pris en charge.
4. Dans le processus, d'autres transformations et optimisations peuvent être appliquées:
    - Code inaccessible supprimé.
    - Les exportations non utilisées ont été supprimées ("tree-shaking").
    - Les instructions spécifiques au développement telles que `console` et le `debugger` ont été supprimées.
    - La syntaxe JavaScript moderne et ultramoderne peut être transformée en une ancienne version dotée de fonctionnalités similaires avec [Babel](https://babeljs.io/).
    - Le fichier résultant est minifié (espaces supprimés, variables remplacées par des noms plus courts, etc.).
=======
1. Take a "main" module, the one intended to be put in `<script type="module">` in HTML.
2. Analyze its dependencies: imports and then imports of imports etc.
3. Build a single file with all modules (or multiple files, that's tunable), replacing native `import` calls with bundler functions, so that it works. "Special" module types like HTML/CSS modules are also supported.
4. In the process, other transformations and optimizations may be applied:
    - Unreachable code removed.
    - Unused exports removed ("tree-shaking").
    - Development-specific statements like `console` and `debugger` removed.
    - Modern, bleeding-edge JavaScript syntax may be transformed to older one with similar functionality using [Babel](https://babeljs.io/).
    - The resulting file is minified (spaces removed, variables replaced with shorter names, etc).
>>>>>>> 2b5ac971c1bd8abe7b17cdcf724afd84799b6cbd

Si nous utilisons des outils d'ensemble, alors que les scripts sont regroupés dans un seul fichier (ou quelques fichiers), les instructions `import/export` contenues dans ces scripts sont remplacées par des fonctions spéciales de regroupeur. Ainsi, le script "fourni" résultant ne contient aucune `import/export`, il ne nécessite pas `type="module"`, et nous pouvons le mettre dans un script standard:

```html
<!-- En supposant que nous ayons bundle.js d'un outil tel que Webpack -->
<script src="bundle.js"></script>
```

Cela dit, les modules natifs sont également utilisables. Nous n’utilisons donc pas Webpack ici: vous pourrez le configurer plus tard.

## Sommaire

Pour résumer, les concepts de base sont les suivants:

1. Un module est un fichier. Pour que `import/export` fonctionne, les navigateurs ont besoin de `<script type="module">`. Les modules ont plusieurs différences:
    - Différé par défaut.
    - Async fonctionne sur les scripts en ligne.
    - Pour charger des scripts externes d'une autre origine (domain/protocol/port), des en-têtes CORS sont nécessaires.
    - Les scripts externes en double sont ignorés.
2. Les modules ont leur propre portée globale et leurs fonctionnalités d’échange via `import/export`.
3. Les modules utilisent toujours `use strict`.
4. Le code des modules est exécuté une seule fois. Les exportations sont créées une fois et partagées entre les importateurs

<<<<<<< HEAD
Lorsque nous utilisons des modules, chaque module implémente la fonctionnalité et l'exporte. Nous utilisons ensuite import pour l’importer directement là où il le faut. Le navigateur charge et exécute les scripts automatiquement.
=======
When we use modules, each module implements the functionality and exports it. Then we use `import` to directly import it where it's needed. The browser loads and evaluates the scripts automatically.
>>>>>>> 2b5ac971c1bd8abe7b17cdcf724afd84799b6cbd

En production, les gens utilisent souvent des "bundlers" tels que [Webpack](https://webpack.js.org) qui regroupe des modules pour des raisons de performances ou pour d’autres raisons.

Dans le chapitre suivant, nous verrons plus d’exemples de modules et comment des choses peuvent être importé / exporté.
