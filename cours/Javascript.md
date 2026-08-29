---
schema_version: 1
uid: "01M02EX5B759XA95C0Z8BPNRMS"
titre: "Javascript"
aliases:
  - "JavaScript"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - developpement-web
  - javascript
resume: "Cours complet de JavaScript moderne : ECMAScript, types, fonctions, objets, modules, asynchronisme, DOM, Web APIs, Node.js, tests, sécurité, performance et pratiques de développement actuelles."
niveau: debutant
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-08-18
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# JavaScript moderne — des fondamentaux aux applications robustes

> [!abstract] Objectif
> Comprendre le langage JavaScript lui-même avant les frameworks : syntaxe et types, fonctions et closures, objets et classes, modules, asynchronisme et event loop, DOM et Web APIs, Node.js, gestion des erreurs, sécurité, tests et outillage, jusqu'aux évolutions d'ECMAScript 2026.

Voir aussi : [[HTML]], [[CSS]], [[HTTP]], [[Selenium]], [[Python]].

> [!info] Version du cours
> Ce cours est vérifié en août 2026. Il s'appuie sur **ECMAScript 2026**, sur les Web APIs modernes et sur les branches maintenues de Node.js. Les fonctionnalités proposées par TC39 mais non encore intégrées au standard ne sont pas présentées comme acquises.

JavaScript est le langage de programmation standard du Web. Il s'exécute dans les navigateurs, mais également côté serveur avec Node.js, dans des outils en ligne de commande, des applications de bureau, des applications mobiles et de nombreux environnements embarqués.

L'objectif de ce cours est de comprendre le langage lui-même avant d'aborder les frameworks. React, Vue, Angular, Express ou Electron changent avec le temps ; les fondamentaux JavaScript restent la base commune.

## Plan du cours

1. [[#1. Introduction à JavaScript|Introduction à JavaScript]]
2. [[#2. Historique, ECMAScript et environnements d'exécution|Historique, ECMAScript et environnements d'exécution]]
3. [[#3. Syntaxe, variables, types et opérateurs|Syntaxe, variables, types et opérateurs]]
4. [[#4. Contrôle du flux et fonctions|Contrôle du flux et fonctions]]
5. [[#5. Portées, closures et modèle d'exécution|Portées, closures et modèle d'exécution]]
6. [[#6. Objets, prototypes et classes|Objets, prototypes et classes]]
7. [[#7. Collections, itération et programmation fonctionnelle|Collections, itération et programmation fonctionnelle]]
8. [[#8. Modules JavaScript|Modules JavaScript]]
9. [[#9. Asynchronisme, Promises et event loop|Asynchronisme, Promises et event loop]]
10. [[#10. DOM, événements et formulaires|DOM, événements et formulaires]]
11. [[#11. Réseau et Web APIs|Réseau et Web APIs]]
12. [[#12. JavaScript côté serveur avec Node.js|JavaScript côté serveur avec Node.js]]
13. [[#13. Gestion des erreurs et robustesse|Gestion des erreurs et robustesse]]
14. [[#14. Sécurité|Sécurité]]
15. [[#15. Tests et qualité|Tests et qualité]]
16. [[#16. Tooling, paquets et construction|Tooling, paquets et construction]]
17. [[#17. Performance et mémoire|Performance et mémoire]]
18. [[#18. APIs avancées du navigateur|APIs avancées du navigateur]]
19. [[#19. Frameworks et architecture applicative|Frameworks et architecture applicative]]
20. [[#20. JavaScript moderne et évolutions ECMAScript|JavaScript moderne et évolutions ECMAScript]]
21. [[#21. Travaux pratiques|Travaux pratiques]]
22. [[#22. Checklist de bonnes pratiques|Checklist de bonnes pratiques]]
23. [[#23. Références|Références]]

---

# 1. Introduction à JavaScript

## 1.1 Qu'est-ce que JavaScript ?

JavaScript est un langage de programmation :

- **dynamique** : le type d'une valeur est connu à l'exécution ;
- **à ramasse-miettes** (*garbage collected*) : la mémoire des objets devenus inaccessibles est récupérée automatiquement ;
- **multi-paradigme** : impératif, fonctionnel et orienté objet ;
- **à prototypes** : l'héritage natif repose sur des chaînes de prototypes ;
- **standardisé** par ECMA International sous le nom **ECMAScript** ;
- doté d'un environnement hôte qui fournit des APIs supplémentaires.

JavaScript ne se réduit donc pas au navigateur. Le langage ECMAScript et les APIs de l'environnement doivent être distingués.

```js
// JavaScript / ECMAScript
const total = [10, 20, 30].reduce((somme, valeur) => somme + valeur, 0);
console.log(total); // 60
```

Dans un navigateur, `document`, `window`, `fetch`, `localStorage` ou `crypto` sont des **Web APIs** fournies par l'environnement, pas par le cœur ECMAScript.

Dans Node.js, `process`, `Buffer`, `fs` ou `http` sont des APIs de Node.js.

## 1.2 JavaScript, HTML et CSS

Sur le Web, on résume souvent la répartition ainsi :

| Technologie | Rôle principal |
|---|---|
| HTML | structure et sémantique du document |
| CSS | présentation et mise en page |
| JavaScript | comportement, logique et interaction |

Cette séparation n'est pas absolue, mais elle reste une bonne règle architecturale.

```html
<button id="saluer">Saluer</button>
<p id="message"></p>

<script type="module">
  const bouton = document.querySelector("#saluer");
  const message = document.querySelector("#message");

  bouton.addEventListener("click", () => {
    message.textContent = "Bonjour !";
  });
</script>
```

## 1.3 Java n'est pas JavaScript

Le nom JavaScript a une origine essentiellement historique et marketing. Java et JavaScript sont deux langages distincts.

| Java | JavaScript |
|---|---|
| typage statique | typage dynamique |
| compilation en bytecode JVM | compilation/interprétation par moteur JS |
| classes au cœur du modèle objet | prototypes au cœur du modèle objet |
| JVM | navigateur, Node.js, Deno, Bun, etc. |

## 1.4 Où JavaScript est-il utilisé ?

- applications web classiques ;
- Single Page Applications et applications hybrides ;
- serveurs et APIs avec Node.js ;
- outils CLI ;
- applications de bureau avec Electron ou Tauri côté interface ;
- applications mobiles via des frameworks spécialisés ;
- scripts d'automatisation ;
- extensions de navigateurs ;
- runtimes edge/serverless ;
- interaction avec WebAssembly.

## 1.5 JavaScript et TypeScript

**TypeScript** est un sur-ensemble syntaxique de JavaScript qui ajoute un système de types statiques et des outils de compilation. Le code TypeScript est transformé en JavaScript avant son exécution dans les environnements qui n'interprètent pas TypeScript nativement.

Pour apprendre correctement TypeScript, il faut d'abord comprendre JavaScript : portée, prototypes, objets, promesses, modules et modèle asynchrone restent les mêmes concepts fondamentaux.

---

# 2. Historique, ECMAScript et environnements d'exécution

## 2.1 Naissance du langage

JavaScript a été créé en 1995 par Brendan Eich chez Netscape. Son premier nom interne était **Mocha**, puis **LiveScript**, avant de devenir JavaScript.

En 1997, le langage est standardisé par Ecma International : la spécification porte le nom **ECMAScript**.

## 2.2 Quelques jalons

| Version | Année | Exemples d'évolutions |
|---|---:|---|
| ES3 | 1999 | expressions régulières, `try/catch` |
| ES5 | 2009 | strict mode, JSON, méthodes de tableaux |
| ES2015 / ES6 | 2015 | `let`, `const`, classes, modules, Promises, générateurs |
| ES2017 | 2017 | `async` / `await` |
| ES2020 | 2020 | optional chaining, nullish coalescing, `BigInt`, import dynamique |
| ES2021 | 2021 | opérateurs d'affectation logique, `Promise.any` |
| ES2022 | 2022 | champs privés de classe, top-level `await` |
| ES2023 | 2023 | méthodes non mutantes `toSorted`, `toReversed`, `with`... |
| ES2024 | 2024 | `Object.groupBy`, `Map.groupBy`, `Promise.withResolvers`... |
| ES2025 | 2025 | Iterator Helpers, Set methods, import attributes, `RegExp.escape`, `Promise.try`... |
| ES2026 | 2026 | édition annuelle courante du standard |

Depuis 2015, ECMAScript suit un rythme annuel. Une fonctionnalité ne doit pas être considérée comme standard uniquement parce qu'elle apparaît dans une proposition TC39.

## 2.3 TC39 et les propositions

Le comité **TC39** fait évoluer ECMAScript. Les propositions passent par plusieurs étapes avant leur intégration au standard.

Pour un projet de production :

1. vérifier le niveau de standardisation ;
2. vérifier la prise en charge des moteurs ciblés ;
3. utiliser un transpileur ou un polyfill seulement si nécessaire ;
4. ne pas confondre fonctionnalité expérimentale et standard publié.

## 2.4 Les moteurs JavaScript

Un moteur JavaScript analyse, compile et exécute le code.

Exemples :

- **V8** : Chromium, Chrome, Node.js ;
- **SpiderMonkey** : Firefox ;
- **JavaScriptCore** : Safari.

Le moteur n'est pas le navigateur. Le navigateur ajoute le DOM, le réseau, le stockage, le rendu, etc.

## 2.5 Node.js

Node.js, créé en 2009, permet d'utiliser JavaScript hors du navigateur. En août 2026 :

- Node.js 24 est une branche **LTS** ;
- Node.js 26 est la branche **Current**.

En production, on privilégie généralement une version LTS maintenue, sauf besoin explicite d'une version Current.

---

# 3. Syntaxe, variables, types et opérateurs

## 3.1 Exécuter du JavaScript

Dans un navigateur :

```html
<script type="module" src="main.js"></script>
```

Dans Node.js :

```bash
node main.js
```

Pour expérimenter rapidement, la console des outils de développement du navigateur est très pratique.

## 3.2 Commentaires

```js
// Commentaire sur une ligne

/*
  Commentaire
  sur plusieurs lignes
*/
```

## 3.3 `const`, `let` et `var`

En JavaScript moderne :

- utiliser `const` par défaut ;
- utiliser `let` lorsqu'une réaffectation est nécessaire ;
- éviter `var` dans le nouveau code.

```js
const nom = "Ada";
let compteur = 0;

compteur += 1;
```

`const` interdit la **réaffectation de la variable**, mais ne rend pas l'objet immuable :

```js
const utilisateur = { nom: "Ada" };
utilisateur.nom = "Grace"; // autorisé

// utilisateur = {}; // TypeError / affectation interdite
```

Pour empêcher certaines mutations :

```js
const configuration = Object.freeze({ mode: "production" });
```

`Object.freeze()` n'effectue toutefois qu'un gel superficiel.

## 3.4 Les types primitifs

JavaScript possède sept types primitifs :

- `string` ;
- `number` ;
- `bigint` ;
- `boolean` ;
- `undefined` ;
- `symbol` ;
- `null`.

Les primitives **ne sont pas des objets**. Le langage peut cependant les envelopper temporairement afin de permettre l'accès à des méthodes.

```js
const texte = "bonjour";
console.log(texte.toUpperCase());
```

Tout ce qui n'est pas une primitive est un **objet**, y compris les tableaux, fonctions, dates, maps et sets.

## 3.5 `typeof`

```js
console.log(typeof "bonjour");  // "string"
console.log(typeof 42);         // "number"
console.log(typeof 42n);        // "bigint"
console.log(typeof true);       // "boolean"
console.log(typeof undefined);  // "undefined"
console.log(typeof Symbol());   // "symbol"
console.log(typeof {});         // "object"
console.log(typeof (() => {})); // "function"
```

Cas historique :

```js
console.log(typeof null); // "object"
```

Il s'agit d'une bizarrerie historique. Pour tester `null`, utiliser :

```js
if (valeur === null) {
  // ...
}
```

## 3.6 Nombres

Le type `number` représente les nombres IEEE-754 en double précision.

```js
const entier = 42;
const reel = 3.14;
const infini = Infinity;
const invalide = NaN;
```

Les nombres flottants ont les limites habituelles du binaire :

```js
console.log(0.1 + 0.2 === 0.3); // false
```

Pour comparer des flottants, on utilise souvent une tolérance :

```js
const egal = Math.abs((0.1 + 0.2) - 0.3) < Number.EPSILON;
```

Pour tester `NaN` :

```js
Number.isNaN(Number("abc")); // true
```

## 3.7 `BigInt`

`BigInt` représente des entiers arbitrairement grands :

```js
const grand = 9007199254740993n;
```

On ne mélange pas directement `number` et `bigint` :

```js
// 1n + 1; // TypeError
1n + BigInt(1);
```

## 3.8 Chaînes de caractères

```js
const simple = 'bonjour';
const double = "bonjour";
const nom = "Ada";
const modele = `Bonjour ${nom}`;
```

Les *template literals* permettent également les chaînes multilignes.

## 3.9 `undefined` et `null`

- `undefined` signifie généralement « valeur non définie » ;
- `null` est une valeur explicite souvent utilisée pour représenter « absence volontaire de valeur ».

```js
let resultat;        // undefined
const parent = null; // absence explicitement choisie
```

## 3.10 Tableaux

```js
const couleurs = ["rouge", "vert", "bleu"];

console.log(couleurs[0]);
console.log(couleurs.length);
```

Un tableau est un objet spécialisé :

```js
typeof couleurs;          // "object"
Array.isArray(couleurs);  // true
```

## 3.11 Objets littéraux

```js
const personne = {
  nom: "Ada",
  age: 36,
  langages: ["Ada", "SQL"],
};

console.log(personne.nom);
console.log(personne["age"]);
```

Clé calculée :

```js
const cle = "role";
const utilisateur = {
  [cle]: "admin",
};
```

## 3.12 Destructuration

```js
const utilisateur = { nom: "Ada", role: "admin" };
const { nom, role } = utilisateur;

const coordonnees = [48.85, 2.35];
const [latitude, longitude] = coordonnees;
```

Avec valeur par défaut :

```js
const { theme = "clair" } = {};
```

## 3.13 Spread et rest

Spread :

```js
const a = [1, 2];
const b = [...a, 3, 4];

const base = { actif: true };
const user = { ...base, nom: "Ada" };
```

Rest :

```js
function somme(...nombres) {
  return nombres.reduce((total, n) => total + n, 0);
}
```

Le spread d'un objet est superficiel : les objets imbriqués restent partagés.

## 3.14 Comparaisons

Préférer `===` et `!==` :

```js
0 == "0";   // true  : coercition
0 === "0";  // false : types différents
```

`Object.is()` diffère sur quelques cas :

```js
Object.is(NaN, NaN); // true
Object.is(0, -0);    // false
```

## 3.15 Valeurs truthy et falsy

Valeurs falsy importantes :

```text
false
0
-0
0n
""
null
undefined
NaN
```

Les tableaux et objets vides sont truthy :

```js
Boolean([]); // true
Boolean({}); // true
```

## 3.16 Optional chaining et nullish coalescing

```js
const ville = utilisateur.adresse?.ville;
```

`?.` arrête l'accès si la valeur précédente vaut `null` ou `undefined`.

```js
const limite = configuration.limite ?? 100;
```

`??` utilise la valeur de droite uniquement si la gauche vaut `null` ou `undefined`.

Différence avec `||` :

```js
const a = 0 || 10;  // 10
const b = 0 ?? 10;  // 0
```

## 3.17 Conversions explicites

```js
Number("42");      // 42
String(42);         // "42"
Boolean(1);         // true
parseInt("42px", 10); // 42
```

Préférer les conversions explicites à une dépendance involontaire à la coercition.

---

# 4. Contrôle du flux et fonctions

## 4.1 Conditions

```js
if (age >= 18) {
  console.log("majeur");
} else {
  console.log("mineur");
}
```

Ternaire :

```js
const statut = age >= 18 ? "majeur" : "mineur";
```

## 4.2 `switch`

```js
switch (role) {
  case "admin":
    console.log("Administration");
    break;
  case "user":
    console.log("Utilisateur");
    break;
  default:
    console.log("Rôle inconnu");
}
```

## 4.3 Boucles

```js
for (let i = 0; i < 3; i += 1) {
  console.log(i);
}
```

Pour parcourir les **valeurs d'un itérable** :

```js
for (const couleur of couleurs) {
  console.log(couleur);
}
```

Pour parcourir les **clés énumérables** d'un objet :

```js
for (const cle in personne) {
  console.log(cle);
}
```

Ne pas confondre `for...of` et `for...in`.

## 4.4 Déclaration de fonction

```js
function addition(a, b) {
  return a + b;
}
```

Une déclaration de fonction est *hoisted* : son binding est créé avant l'exécution des instructions du bloc concerné.

## 4.5 Expression de fonction

```js
const multiplier = function (a, b) {
  return a * b;
};
```

## 4.6 Fonctions fléchées

```js
const carre = (n) => n * n;
```

Une fonction fléchée :

- ne crée pas son propre `this` ;
- n'a pas son propre `arguments` ;
- ne peut pas être utilisée comme constructeur avec `new`.

Elle est très pratique pour les callbacks, mais n'est pas un remplacement universel de `function`.

## 4.7 Paramètres par défaut

```js
function saluer(nom = "inconnu") {
  return `Bonjour ${nom}`;
}
```

## 4.8 Fonctions de premier ordre

Les fonctions sont des valeurs :

```js
function appliquer(operation, a, b) {
  return operation(a, b);
}

const resultat = appliquer((x, y) => x + y, 2, 3);
```

On peut les stocker, les passer en argument et les retourner.

## 4.9 Fonctions pures

Une fonction pure :

- retourne le même résultat pour les mêmes entrées ;
- ne modifie pas un état extérieur observable.

```js
function ajouterTva(prix, taux) {
  return prix * (1 + taux);
}
```

Les fonctions pures facilitent les tests et le raisonnement.

---

# 5. Portées, closures et modèle d'exécution

## 5.1 Portée lexicale

JavaScript utilise une portée lexicale : la visibilité dépend de l'emplacement du code source.

```js
const globale = "G";

function exemple() {
  const locale = "L";
  console.log(globale, locale);
}
```

## 5.2 Portée de bloc

`let`, `const` et `class` respectent les blocs :

```js
if (true) {
  const interne = 42;
}

// console.log(interne); // ReferenceError
```

`var` possède une portée de fonction et a des règles de hoisting différentes ; cela explique pourquoi on l'évite généralement dans le nouveau code.

## 5.3 Temporal Dead Zone

Une variable `let` ou `const` existe lexicalement avant sa déclaration, mais ne peut pas être utilisée avant son initialisation :

```js
// console.log(valeur); // ReferenceError
const valeur = 10;
```

Cette période est appelée **Temporal Dead Zone**.

## 5.4 Closures

Une closure associe une fonction à son environnement lexical.

```js
function creerCompteur() {
  let valeur = 0;

  return () => {
    valeur += 1;
    return valeur;
  };
}

const compteur = creerCompteur();
console.log(compteur()); // 1
console.log(compteur()); // 2
```

La variable `valeur` reste accessible à la fonction retournée même après la fin de `creerCompteur()`.

Applications :

- encapsulation ;
- fonctions génératrices ;
- callbacks ;
- memoization ;
- création de factories.

## 5.5 Pile d'appels

Lorsqu'une fonction appelle une autre fonction, les contextes d'exécution sont placés dans une pile.

```js
function c() {
  console.trace();
}

function b() {
  c();
}

function a() {
  b();
}

a();
```

Une récursion infinie finit généralement par provoquer une erreur de dépassement de pile.

## 5.6 `this`

La valeur de `this` dépend du **mode d'appel**, sauf pour les fonctions fléchées qui capturent lexicalement le `this` environnant.

```js
const personne = {
  nom: "Ada",
  afficherNom() {
    console.log(this.nom);
  },
};

personne.afficherNom();
```

Extraire une méthode peut perdre son receveur :

```js
const afficher = personne.afficherNom;
// afficher(); // `this` n'est plus `personne`
```

On peut fixer explicitement `this` avec `bind` :

```js
const afficherAda = personne.afficherNom.bind(personne);
afficherAda();
```

## 5.7 `call`, `apply` et `bind`

```js
function saluer(prefixe) {
  return `${prefixe} ${this.nom}`;
}

const contexte = { nom: "Ada" };

saluer.call(contexte, "Bonjour");
saluer.apply(contexte, ["Bonjour"]);
const liee = saluer.bind(contexte, "Bonjour");
```

---

# 6. Objets, prototypes et classes

## 6.1 Le modèle objet

JavaScript repose sur l'héritage prototypal.

```js
const animal = {
  respirer() {
    return "respiration";
  },
};

const chat = Object.create(animal);
chat.nom = "Mina";

chat.respirer();
```

Si une propriété n'existe pas directement sur `chat`, JavaScript remonte sa chaîne de prototypes.

## 6.2 Propriété propre et propriété héritée

```js
Object.hasOwn(chat, "nom");      // true
Object.hasOwn(chat, "respirer"); // false
```

Pour du nouveau code, `Object.hasOwn()` est généralement préférable à l'appel direct de `obj.hasOwnProperty()` sur un objet potentiellement non fiable.

## 6.3 Descripteurs de propriétés

```js
Object.defineProperty(personne, "id", {
  value: 123,
  writable: false,
  enumerable: true,
  configurable: false,
});
```

Les propriétés possèdent notamment les attributs :

- `writable` ;
- `enumerable` ;
- `configurable`.

## 6.4 Getters et setters

```js
const rectangle = {
  largeur: 4,
  hauteur: 3,
  get aire() {
    return this.largeur * this.hauteur;
  },
};

console.log(rectangle.aire);
```

Éviter les getters qui effectuent des opérations très coûteuses ou possèdent des effets de bord surprenants.

## 6.5 Classes

Les classes offrent une syntaxe plus lisible par-dessus le modèle prototypal.

```js
class Compte {
  constructor(titulaire, solde = 0) {
    this.titulaire = titulaire;
    this.solde = solde;
  }

  crediter(montant) {
    this.solde += montant;
  }
}
```

## 6.6 Héritage de classe

```js
class CompteEpargne extends Compte {
  constructor(titulaire, solde, taux) {
    super(titulaire, solde);
    this.taux = taux;
  }
}
```

Préférer souvent la **composition** à des hiérarchies d'héritage profondes.

## 6.7 Champs privés

```js
class Coffre {
  #secret;

  constructor(secret) {
    this.#secret = secret;
  }

  lire() {
    return this.#secret;
  }
}
```

Les champs privés `#nom` sont réellement privés au niveau syntaxique ; ils ne sont pas équivalents à une convention `_nom`.

## 6.8 Méthodes statiques

```js
class Temperature {
  static celsiusVersKelvin(celsius) {
    return celsius + 273.15;
  }
}
```

## 6.9 Copie d'objets

Copie superficielle :

```js
const copie = { ...original };
```

Copie structurée profonde pour les types pris en charge :

```js
const copieProfonde = structuredClone(original);
```

`structuredClone()` est une API de la plateforme disponible dans les navigateurs modernes et Node.js ; ce n'est pas une fonction permettant de cloner n'importe quelle instance arbitraire avec ses méthodes/prototypes personnalisés.

---

# 7. Collections, itération et programmation fonctionnelle

## 7.1 Méthodes de tableau

### `map`

```js
const doubles = [1, 2, 3].map((n) => n * 2);
```

### `filter`

```js
const pairs = [1, 2, 3, 4].filter((n) => n % 2 === 0);
```

### `reduce`

```js
const total = [10, 20, 30].reduce((acc, n) => acc + n, 0);
```

### `find` et `findIndex`

```js
const utilisateur = utilisateurs.find((u) => u.id === 42);
```

### `some` et `every`

```js
const contientAdmin = utilisateurs.some((u) => u.role === "admin");
const tousActifs = utilisateurs.every((u) => u.actif);
```

## 7.2 Méthodes mutantes et non mutantes

`sort()` modifie le tableau :

```js
const nombres = [3, 1, 2];
nombres.sort((a, b) => a - b);
```

Pour conserver l'original, JavaScript moderne fournit notamment :

```js
const tries = nombres.toSorted((a, b) => a - b);
const inverse = nombres.toReversed();
const remplace = nombres.with(0, 99);
```

Les méthodes non mutantes simplifient la programmation fonctionnelle et la gestion d'état.

## 7.3 `Map`

```js
const droits = new Map();
droits.set("alice", "admin");
droits.set("bob", "user");

console.log(droits.get("alice"));
```

Contrairement aux clés d'un objet ordinaire, une clé de `Map` peut être une valeur de n'importe quel type.

## 7.4 `Set`

```js
const uniques = new Set([1, 1, 2, 3]);
console.log([...uniques]); // [1, 2, 3]
```

Les versions modernes d'ECMAScript proposent également des opérations d'ensemble telles que `union`, `intersection`, `difference` ou `isSubsetOf` lorsque l'environnement les prend en charge.

## 7.5 WeakMap et WeakSet

`WeakMap` et `WeakSet` conservent des références faibles vers leurs clés/éléments objets. Ils sont utiles lorsqu'on ne veut pas empêcher le ramasse-miettes de libérer l'objet.

Ils ne sont pas énumérables.

## 7.6 Itérables

Un objet itérable implémente le protocole associé à `Symbol.iterator`.

```js
const texte = "ABC";

for (const lettre of texte) {
  console.log(lettre);
}
```

## 7.7 Générateurs

```js
function* compteur(max) {
  for (let i = 0; i < max; i += 1) {
    yield i;
  }
}

for (const valeur of compteur(3)) {
  console.log(valeur);
}
```

Un générateur produit ses valeurs à la demande.

## 7.8 Iterator Helpers

Les éditions récentes d'ECMAScript ajoutent des méthodes sur les itérateurs, par exemple :

```js
const resultat = [1, 2, 3, 4]
  .values()
  .filter((n) => n % 2 === 0)
  .map((n) => n * 10)
  .toArray();
```

Toujours vérifier la compatibilité avec les moteurs ciblés avant d'utiliser une fonctionnalité récente sans transpilation/polyfill.

## 7.9 Groupement

```js
const produits = [
  { nom: "A", categorie: "livre" },
  { nom: "B", categorie: "jeu" },
  { nom: "C", categorie: "livre" },
];

const parCategorie = Object.groupBy(
  produits,
  (produit) => produit.categorie,
);
```

---

# 8. Modules JavaScript

## 8.1 Pourquoi utiliser des modules ?

Les modules permettent :

- d'éviter un espace global partagé ;
- de définir des APIs explicites ;
- de séparer les responsabilités ;
- de faciliter les tests ;
- d'analyser statiquement les dépendances.

## 8.2 Export nommé

```js
// math.js
export function addition(a, b) {
  return a + b;
}

export const PI = Math.PI;
```

```js
// main.js
import { addition, PI } from "./math.js";
```

## 8.3 Export par défaut

```js
export default function demarrer() {
  // ...
}
```

```js
import demarrer from "./app.js";
```

Les exports nommés sont souvent plus faciles à refactorer dans les gros projets.

## 8.4 Renommage

```js
import { addition as somme } from "./math.js";
```

## 8.5 `import()` dynamique

```js
const module = await import("./fonctionnalite-lourde.js");
module.demarrer();
```

Cela permet notamment le chargement conditionnel et le *code splitting*.

## 8.6 Top-level `await`

Dans un module :

```js
const configuration = await fetch("/config.json").then((r) => r.json());
```

À utiliser avec discernement : un top-level `await` influence l'évaluation des modules dépendants.

## 8.7 Import maps

Dans le navigateur, une import map permet de résoudre des noms de modules :

```html
<script type="importmap">
{
  "imports": {
    "lib/": "/vendor/lib/"
  }
}
</script>

<script type="module">
  import { outil } from "lib/outils.js";
</script>
```

Les import maps ne remplacent pas automatiquement un gestionnaire de paquets, mais elles réduisent certains besoins de réécriture d'URL dans les navigateurs modernes.

## 8.8 Import attributes

La syntaxe actuelle utilise `with`, et non l'ancienne syntaxe expérimentale `assert` :

```js
import configuration from "./config.json" with { type: "json" };
```

Pour les ressources non JavaScript, la disponibilité dépend de l'environnement et du type de module.

## 8.9 ESM et CommonJS dans Node.js

Deux systèmes coexistent encore :

- ECMAScript Modules : `import` / `export` ;
- CommonJS historique : `require()` / `module.exports`.

Pour un nouveau projet, ESM est généralement un bon choix lorsque les dépendances et outils utilisés le permettent.

Exemple `package.json` :

```json
{
  "type": "module"
}
```

---

# 9. Asynchronisme, Promises et event loop

## 9.1 Pourquoi l'asynchronisme ?

Une opération réseau, un timer ou certaines opérations d'entrée/sortie prennent du temps. Bloquer le thread JavaScript pendant toute cette durée rendrait l'application inutilisable.

JavaScript coordonne donc des tâches asynchrones avec l'environnement hôte et une **event loop**.

## 9.2 Callback

```js
setTimeout(() => {
  console.log("exécuté plus tard");
}, 1000);
```

Les callbacks sont fondamentaux, mais une imbrication excessive peut rendre le code difficile à maintenir.

## 9.3 Promise

Une Promise représente un résultat futur :

- `pending` ;
- `fulfilled` ;
- `rejected`.

```js
const promesse = fetch("/api/users");

promesse
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

## 9.4 `async` / `await`

```js
async function chargerUtilisateurs() {
  const response = await fetch("/api/users");

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  return response.json();
}
```

Une fonction `async` retourne toujours une Promise.

## 9.5 Concurrence avec `Promise.all`

Mauvais si les appels sont indépendants :

```js
const a = await chargerA();
const b = await chargerB();
```

Plus concurrent :

```js
const [a, b] = await Promise.all([
  chargerA(),
  chargerB(),
]);
```

`Promise.all()` échoue dès qu'une des promesses rejette.

## 9.6 Autres combinateurs

```js
await Promise.allSettled(promesses);
await Promise.any(promesses);
await Promise.race(promesses);
```

Choisir selon la sémantique souhaitée.

## 9.7 `Promise.withResolvers()`

```js
const { promise, resolve, reject } = Promise.withResolvers();
```

Cette API évite de capturer manuellement les fonctions `resolve` et `reject` depuis le constructeur de Promise.

## 9.8 `Promise.try()`

ECMAScript moderne fournit `Promise.try()` pour transformer uniformément une fonction susceptible de retourner une valeur, une Promise ou de lever synchroniquement une exception :

```js
const resultat = await Promise.try(() => operationPeutEchouer());
```

## 9.9 Event loop : tâches et microtasks

Considérons :

```js
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

console.log("D");
```

Sortie typique :

```text
A
D
C
B
```

Pourquoi ?

1. le code synchrone s'exécute ;
2. les microtasks, dont les réactions de Promise, sont vidées ;
3. une nouvelle tâche de la file d'événements peut être exécutée.

Comprendre cette distinction est essentiel pour diagnostiquer des problèmes d'ordre d'exécution.

## 9.10 Annulation avec AbortController

```js
const controller = new AbortController();

const requete = fetch("/api/longue", {
  signal: controller.signal,
});

controller.abort();
```

Il est préférable de concevoir explicitement l'annulation plutôt que de laisser des opérations inutiles continuer.

## 9.11 Timeout de requête

Une approche moderne :

```js
const response = await fetch("/api/data", {
  signal: AbortSignal.timeout(5000),
});
```

Vérifier la compatibilité avec les environnements ciblés.

## 9.12 Async iterators

```js
for await (const morceau of flux) {
  console.log(morceau);
}
```

Pour matérialiser un itérable asynchrone :

```js
const valeurs = await Array.fromAsync(flux);
```

Attention : `Array.fromAsync()` traite séquentiellement les valeurs de l'itérable, ce qui diffère d'un `Promise.all()` sur une liste déjà disponible.

---

# 10. DOM, événements et formulaires

## 10.1 Sélectionner des éléments

```js
const titre = document.querySelector("h1");
const boutons = document.querySelectorAll("button.action");
```

`querySelectorAll()` retourne une `NodeList` statique.

## 10.2 Modifier le contenu

```js
const message = document.querySelector("#message");
message.textContent = "Bonjour";
```

Pour du texte non fiable, préférer `textContent` à `innerHTML` afin de ne pas interpréter le contenu comme HTML.

## 10.3 Créer des éléments

```js
const li = document.createElement("li");
li.textContent = "Élément";

document.querySelector("ul").append(li);
```

## 10.4 Classes CSS

```js
const panneau = document.querySelector(".panneau");
panneau.classList.add("visible");
panneau.classList.toggle("compact");
```

## 10.5 Attributs et propriétés

```js
const image = document.querySelector("img");
image.setAttribute("alt", "Description");

const caseACocher = document.querySelector("input[type=checkbox]");
console.log(caseACocher.checked);
```

Les attributs HTML et propriétés DOM ne sont pas toujours équivalents.

## 10.6 Événements

```js
const bouton = document.querySelector("button");

bouton.addEventListener("click", (event) => {
  console.log(event.currentTarget);
});
```

Préférer `addEventListener()` aux attributs HTML `onclick="..."`.

## 10.7 Propagation

Un événement traverse typiquement :

1. capture ;
2. cible ;
3. bubbling.

```js
conteneur.addEventListener("click", (event) => {
  console.log(event.target);
});
```

## 10.8 Délégation d'événements

```js
liste.addEventListener("click", (event) => {
  const bouton = event.target.closest("button[data-id]");
  if (!bouton) return;

  supprimerElement(bouton.dataset.id);
});
```

La délégation évite de créer un listener par élément dynamique.

## 10.9 Formulaires

```js
formulaire.addEventListener("submit", (event) => {
  event.preventDefault();

  const data = new FormData(formulaire);
  console.log(Object.fromEntries(data));
});
```

La validation côté navigateur améliore l'expérience utilisateur, mais **ne remplace jamais la validation serveur**.

## 10.10 Accessibilité

Un composant interactif doit préserver :

- la navigation clavier ;
- les éléments HTML sémantiques ;
- le focus visible ;
- les labels de formulaire ;
- les états ARIA lorsque nécessaires.

Ne pas remplacer un `<button>` par un `<div>` cliquable sans raison.

---

# 11. Réseau et Web APIs

## 11.1 `fetch()`

```js
const response = await fetch("/api/users");

if (!response.ok) {
  throw new Error(`Erreur HTTP ${response.status}`);
}

const utilisateurs = await response.json();
```

Important : `fetch()` ne rejette pas une Promise simplement parce que le serveur répond `404` ou `500`. Il faut examiner `response.ok` ou `response.status`.

## 11.2 Requête POST JSON

```js
const response = await fetch("/api/users", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ nom: "Ada" }),
});
```

## 11.3 URL et URLSearchParams

```js
const url = new URL("https://example.org/recherche");
url.searchParams.set("q", "javascript");
url.searchParams.set("page", "2");
```

Éviter de concaténer manuellement des query strings complexes.

## 11.4 CORS

CORS est une politique du navigateur contrôlant certains accès cross-origin.

Le serveur doit envoyer les en-têtes appropriés. Ajouter arbitrairement `mode: "no-cors"` n'est pas une solution générale : la réponse devient souvent opaque et inutilisable par le script.

## 11.5 Stockage navigateur

### `localStorage`

```js
localStorage.setItem("theme", "dark");
const theme = localStorage.getItem("theme");
```

`localStorage` :

- stocke des chaînes ;
- est synchrone ;
- est accessible au JavaScript de la même origine ;
- ne convient pas au stockage de secrets sensibles exposés aux risques XSS.

### IndexedDB

Pour de gros volumes structurés ou un usage hors ligne, IndexedDB est mieux adapté.

## 11.6 Cookies

Les cookies sont également gérés par le navigateur et peuvent être protégés avec :

- `HttpOnly` ;
- `Secure` ;
- `SameSite`.

Un cookie de session sensible est souvent plus sûr lorsqu'il est `HttpOnly`, car le JavaScript de la page ne peut alors pas le lire directement.

## 11.7 WebSocket

```js
const socket = new WebSocket("wss://example.org/socket");

socket.addEventListener("message", (event) => {
  console.log(event.data);
});
```

WebSocket permet une communication bidirectionnelle persistante.

## 11.8 Server-Sent Events

```js
const source = new EventSource("/events");
source.addEventListener("message", (event) => {
  console.log(event.data);
});
```

SSE est adapté lorsque le serveur pousse des événements vers le client sans besoin d'un canal bidirectionnel complet.

---

# 12. JavaScript côté serveur avec Node.js

## 12.1 Initialiser un projet

```bash
mkdir mon-projet
cd mon-projet
npm init -y
```

Exemple minimal :

```json
{
  "name": "mon-projet",
  "private": true,
  "type": "module",
  "engines": {
    "node": ">=24"
  }
}
```

En août 2026, Node.js 24 est LTS et Node.js 26 Current. Pour un cours, les exemples restent volontairement compatibles avec une branche LTS moderne.

## 12.2 `process`

```js
console.log(process.version);
console.log(process.argv);
console.log(process.env.NODE_ENV);
```

Ne pas journaliser tout `process.env` : il peut contenir des secrets.

## 12.3 Lire un fichier

```js
import { readFile } from "node:fs/promises";

const contenu = await readFile("./config.json", "utf8");
console.log(contenu);
```

Préférer les APIs asynchrones dans un serveur lorsqu'une opération bloquante limiterait la concurrence.

## 12.4 Chemins

```js
import path from "node:path";

const fichier = path.join("data", "utilisateurs.json");
```

Éviter de construire des chemins avec des concaténations dépendantes du système d'exploitation.

## 12.5 Serveur HTTP minimal

```js
import { createServer } from "node:http";

const serveur = createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" });
  res.end(JSON.stringify({ ok: true }));
});

serveur.listen(3000);
```

En pratique, des frameworks comme Express, Fastify, Hono ou d'autres abstractions peuvent faciliter le routage, la validation et les middlewares. Il faut cependant comprendre les primitives de base.

## 12.6 Variables d'environnement

```js
const port = Number(process.env.PORT ?? 3000);
```

Bonnes pratiques :

- valider les variables au démarrage ;
- ne pas commit les secrets ;
- utiliser un gestionnaire de secrets en production ;
- séparer configuration et code.

## 12.7 Streams

Les streams évitent de charger de très gros fichiers entièrement en mémoire.

```js
import { createReadStream } from "node:fs";

const flux = createReadStream("./gros-fichier.log", {
  encoding: "utf8",
});

for await (const chunk of flux) {
  traiter(chunk);
}
```

## 12.8 Processus et concurrence

JavaScript s'exécute principalement sur un thread pour le code utilisateur, mais Node.js délègue plusieurs opérations à son environnement et peut utiliser :

- pool de threads pour certaines opérations ;
- `worker_threads` pour du calcul CPU ;
- plusieurs processus/conteneurs pour le parallélisme horizontal.

Ne pas confondre « JavaScript mono-thread » avec « Node.js ne fait jamais rien en parallèle ».

---

# 13. Gestion des erreurs et robustesse

## 13.1 `throw`

```js
function diviser(a, b) {
  if (b === 0) {
    throw new RangeError("Division par zéro");
  }

  return a / b;
}
```

## 13.2 `try`, `catch`, `finally`

```js
try {
  await operation();
} catch (error) {
  console.error("Échec", error);
} finally {
  libererRessource();
}
```

`finally` s'exécute que l'opération réussisse ou échoue, sauf interruption brutale de l'environnement.

## 13.3 Classes d'erreur personnalisées

```js
class ValidationError extends Error {
  constructor(message, champ) {
    super(message);
    this.name = "ValidationError";
    this.champ = champ;
  }
}
```

## 13.4 Erreurs avec cause

```js
try {
  await connexion();
} catch (error) {
  throw new Error("Impossible d'initialiser le service", {
    cause: error,
  });
}
```

Cela conserve la chaîne causale sans perdre le contexte métier.

## 13.5 Ne pas avaler les erreurs

À éviter :

```js
try {
  await operation();
} catch {
  // rien
}
```

Si l'erreur est volontairement ignorée, documenter pourquoi.

## 13.6 Validation aux frontières

Les données externes doivent être considérées comme non fiables :

- corps HTTP ;
- paramètres d'URL ;
- fichiers ;
- messages de queue ;
- données d'une API tierce ;
- stockage persistant.

Valider au plus tôt permet de garder un cœur applicatif cohérent.

---

# 14. Sécurité

## 14.1 Principe général

Le JavaScript exécuté dans le navigateur est **contrôlé par l'utilisateur**. Il ne peut pas être considéré comme une frontière de sécurité.

Une règle métier critique ou une autorisation doit être contrôlée côté serveur.

## 14.2 XSS

Une vulnérabilité XSS permet l'exécution de code dans le contexte d'une page.

À éviter avec des données non fiables :

```js
message.innerHTML = contenuUtilisateur;
```

Préférer :

```js
message.textContent = contenuUtilisateur;
```

Si l'application doit accepter du HTML utilisateur, utiliser une stratégie de sanitization robuste adaptée au contexte et ne pas inventer un filtre artisanal à base de regex.

## 14.3 Injection de code

Éviter avec des données non fiables :

```js
// eval(contenuExterne);
// new Function(contenuExterne)();
```

`eval()` complique également l'optimisation, l'analyse statique et les politiques CSP.

## 14.4 CSP

**Content Security Policy** limite les sources autorisées pour scripts, styles, frames, etc.

Une CSP robuste permet notamment de réduire l'impact d'une injection HTML.

Éviter de dépendre d'`unsafe-inline` ou `unsafe-eval` lorsque ce n'est pas nécessaire.

## 14.5 CSRF

CSRF vise principalement les requêtes authentifiées automatiquement par le navigateur, typiquement via cookies.

Mesures fréquentes :

- cookies `SameSite` appropriés ;
- token CSRF ;
- vérification de l'origine selon le contexte ;
- méthodes HTTP correctes ;
- absence d'effet de bord sur GET.

## 14.6 Secrets

Un secret inclus dans du JavaScript envoyé au navigateur n'est plus secret.

```text
Mauvaise idée :
clé API privée -> bundle JavaScript -> navigateur
```

Les secrets privés doivent rester côté serveur ou dans un composant de confiance.

## 14.7 Dépendances

Risques :

- paquet compromis ;
- typosquatting ;
- dépendance abandonnée ;
- script d'installation malveillant ;
- attaque sur la chaîne d'approvisionnement.

Bonnes pratiques :

- limiter les dépendances ;
- conserver le lockfile ;
- auditer les mises à jour ;
- utiliser des versions maintenues ;
- appliquer le principe du moindre privilège aux tokens CI ;
- produire si nécessaire un SBOM.

## 14.8 Pollution de prototype

Des fusions naïves d'objets non fiables peuvent permettre une pollution de prototype dans certaines bibliothèques ou implémentations.

Ne pas copier aveuglément des propriétés arbitraires vers des objets sensibles. Valider les clés attendues.

## 14.9 DOM clobbering

Des éléments HTML contrôlés peuvent, dans certains contextes, entrer en conflit avec des propriétés globales ou des noms supposés par du code ancien.

Éviter de dépendre implicitement des éléments HTML comme variables globales et utiliser des sélecteurs explicites.

## 14.10 Open redirect

Ne pas rediriger vers une URL fournie directement par l'utilisateur sans validation.

```js
const destination = new URL(parametre, location.origin);

if (destination.origin !== location.origin) {
  throw new Error("Destination interdite");
}
```

La règle exacte dépend du besoin métier.

---

# 15. Tests et qualité

## 15.1 Pyramide de tests

Un projet peut combiner :

- tests unitaires ;
- tests d'intégration ;
- tests de contrat ;
- tests end-to-end.

Les outils doivent être choisis selon le projet, pas par habitude.

## 15.2 Tests natifs Node.js

Node.js fournit un runner de tests intégré :

```js
import test from "node:test";
import assert from "node:assert/strict";

function addition(a, b) {
  return a + b;
}

test("additionne deux nombres", () => {
  assert.equal(addition(2, 3), 5);
});
```

Exécution :

```bash
node --test
```

## 15.3 Frameworks de tests

Selon le contexte, on rencontre notamment :

- Vitest ;
- Jest ;
- Mocha ;
- Playwright ;
- Cypress.

**Protractor** et **TSLint**, cités dans d'anciens supports, ne sont plus de bonnes recommandations pour de nouveaux projets JavaScript modernes.

## 15.4 Test d'une fonction pure

```js
export function prixTtc(prixHt, taux) {
  return prixHt * (1 + taux);
}
```

```js
import test from "node:test";
import assert from "node:assert/strict";
import { prixTtc } from "./prix.js";

test("calcule le prix TTC", () => {
  assert.equal(prixTtc(100, 0.2), 120);
});
```

## 15.5 Tester l'asynchrone

```js
test("charge une ressource", async () => {
  const resultat = await charger();
  assert.equal(resultat.ok, true);
});
```

## 15.6 Mocking : avec parcimonie

Trop de mocks peuvent tester une reproduction imaginaire du système plutôt que son comportement réel.

Privilégier :

- unités pures pour la logique ;
- fakes contrôlés ;
- tests d'intégration sur les frontières importantes ;
- E2E pour les parcours critiques.

## 15.7 ESLint

ESLint analyse statiquement le code JavaScript.

Configuration moderne : `eslint.config.js`.

```js
export default [
  {
    files: ["**/*.js"],
    rules: {
      "no-unused-vars": "error",
      "no-undef": "error",
      eqeqeq: "error",
    },
  },
];
```

L'ancien format `.eslintrc.*` subsiste dans beaucoup de projets historiques, mais le **flat config** est le format moderne à privilégier.

## 15.8 Formatage

Un formatter comme Prettier peut uniformiser la présentation. Il ne remplace pas un linter :

- formatter : style de présentation ;
- linter : erreurs potentielles, conventions et règles de code.

## 15.9 Type checking avec JSDoc

Même en JavaScript pur, on peut documenter des types :

```js
/**
 * @param {number} prix
 * @param {number} taux
 * @returns {number}
 */
function prixTtc(prix, taux) {
  return prix * (1 + taux);
}
```

Des éditeurs et outils peuvent exploiter ces annotations.

---

# 16. Tooling, paquets et construction

## 16.1 `package.json`

```json
{
  "name": "exemple",
  "private": true,
  "type": "module",
  "scripts": {
    "test": "node --test",
    "lint": "eslint ."
  }
}
```

## 16.2 Gestionnaires de paquets

On rencontre principalement :

- npm ;
- pnpm ;
- Yarn.

Le choix dépend de l'écosystème et des contraintes du projet. Éviter de mélanger plusieurs lockfiles dans un même dépôt.

## 16.3 Lockfile

Le lockfile :

- fige la résolution exacte des dépendances ;
- améliore la reproductibilité ;
- doit généralement être versionné dans une application.

Exemple npm :

```bash
npm ci
```

`npm ci` utilise le lockfile et convient bien aux environnements CI reproductibles.

## 16.4 `dependencies` et `devDependencies`

```bash
npm install express
npm install --save-dev eslint
```

- `dependencies` : nécessaires au fonctionnement du paquet/application selon son modèle de déploiement ;
- `devDependencies` : outils de développement, tests, lint, build, etc.

## 16.5 Bundlers et outils de build

Outils possibles :

- Vite ;
- Rollup ;
- esbuild ;
- Webpack ;
- outils intégrés de frameworks.

Un bundler peut :

- assembler les modules ;
- effectuer du tree shaking ;
- découper le code ;
- transformer des syntaxes ;
- gérer des assets ;
- optimiser la production.

Mais un petit projet destiné à des navigateurs modernes peut parfois utiliser directement les modules natifs.

## 16.6 Babel

Babel transforme du JavaScript vers une syntaxe compatible avec des cibles plus anciennes.

Il faut distinguer :

- **transpilation de syntaxe** ;
- **polyfill d'une API manquante**.

Transformer `?.` en syntaxe ancienne ne crée pas automatiquement une API inexistante comme une méthode récente de `Set`.

## 16.7 Sourcemaps

Les sourcemaps relient le code produit au code source, ce qui facilite le debugging.

En production, réfléchir à leur exposition : elles peuvent révéler des détails du code source selon leur mode de publication.

## 16.8 Scripts npm

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "test": "node --test",
    "lint": "eslint ."
  }
}
```

Les scripts constituent une interface simple et documentable pour les tâches courantes.

---

# 17. Performance et mémoire

## 17.1 Mesurer avant d'optimiser

Éviter les micro-optimisations intuitives. Utiliser :

- DevTools Performance ;
- profiler Node.js ;
- métriques applicatives ;
- benchmarks représentatifs.

## 17.2 Complexité algorithmique

```js
// Recherche O(n)
const trouve = utilisateurs.find((u) => u.id === id);
```

Si la recherche est extrêmement fréquente, une `Map` indexée peut être plus appropriée :

```js
const parId = new Map(utilisateurs.map((u) => [u.id, u]));
const utilisateur = parId.get(id);
```

Choisir une structure de données selon les opérations dominantes.

## 17.3 Éviter de bloquer le main thread

Dans le navigateur, un calcul CPU long bloque :

- affichage ;
- saisie utilisateur ;
- événements ;
- animations.

Pour du calcul lourd, envisager un **Web Worker**.

## 17.4 Layout et rendu

De nombreuses lectures/écritures DOM alternées peuvent provoquer des recalculs de style/layout coûteux.

Préférer regrouper les opérations lorsque cela améliore le rendu.

## 17.5 Fuites mémoire

Causes possibles :

- listeners jamais retirés ;
- timers persistants ;
- closures retenant de gros objets ;
- caches non bornés ;
- références globales ;
- DOM détaché encore référencé.

## 17.6 Nettoyer les ressources

```js
const controller = new AbortController();

bouton.addEventListener("click", gestionnaire, {
  signal: controller.signal,
});

// À la destruction du composant :
controller.abort();
```

Ceci permet notamment de retirer automatiquement les listeners associés au signal.

## 17.7 Code splitting

```js
const { editeur } = await import("./editeur.js");
```

Charger une fonctionnalité lourde uniquement lorsqu'elle est nécessaire peut réduire le JavaScript initial.

## 17.8 Ne pas surcharger le navigateur

Une application moderne n'a pas besoin d'envoyer tout son code au client. Le rendu serveur, le streaming, les composants serveur de certains frameworks ou de simples pages multi-pages peuvent parfois être plus performants qu'une SPA totale.

---

# 18. APIs avancées du navigateur

## 18.1 Web Workers

```js
const worker = new Worker("./worker.js", { type: "module" });

worker.postMessage({ operation: "calcul", donnees });
worker.addEventListener("message", (event) => {
  console.log(event.data);
});
```

Les workers n'accèdent pas directement au DOM.

## 18.2 Service Workers

Un Service Worker agit comme intermédiaire entre l'application web et le réseau. Il permet notamment :

- stratégie de cache ;
- fonctionnement hors ligne ;
- interception de requêtes ;
- notifications push selon l'environnement.

Il s'exécute dans un contexte distinct du document.

## 18.3 Progressive Web Apps

Une PWA combine plusieurs capacités Web pour se rapprocher d'une application installable :

- manifest ;
- HTTPS ;
- service worker lorsque nécessaire ;
- design responsive ;
- expérience hors ligne ou résiliente.

Une PWA n'est pas un framework JavaScript.

## 18.4 Web Components

Trois briques fréquentes :

- Custom Elements ;
- Shadow DOM ;
- templates HTML.

```js
class BonjourElement extends HTMLElement {
  connectedCallback() {
    this.textContent = "Bonjour";
  }
}

customElements.define("bonjour-element", BonjourElement);
```

## 18.5 Shadow DOM

```js
const shadow = element.attachShadow({ mode: "open" });
shadow.innerHTML = `<p>Contenu encapsulé</p>`;
```

Le Shadow DOM fournit une encapsulation DOM/CSS, pas une frontière de sécurité.

## 18.6 WebAssembly

WebAssembly permet d'exécuter un format binaire portable dans le navigateur et d'autres runtimes.

JavaScript sert souvent à :

- charger le module Wasm ;
- lui fournir des données ;
- piloter l'UI ;
- orchestrer les appels.

WebAssembly ne « remplace » pas JavaScript pour l'ensemble des applications Web.

## 18.7 Web Crypto

Pour la cryptographie dans le navigateur, utiliser les primitives fournies par la plateforme plutôt que coder soi-même des algorithmes cryptographiques.

```js
const octets = crypto.getRandomValues(new Uint8Array(32));
```

`Math.random()` ne convient pas à la génération de secrets cryptographiques.

---

# 19. Frameworks et architecture applicative

## 19.1 Un framework n'est pas le langage

Il est utile de séparer :

```text
JavaScript / ECMAScript
        ↓
Web APIs ou Node APIs
        ↓
framework / bibliothèque
        ↓
application
```

Cette distinction facilite les migrations et évite de confondre une API de framework avec une capacité native du langage.

## 19.2 React, Vue et Angular

Très schématiquement :

- **React** : bibliothèque d'interface reposant sur des composants ;
- **Vue** : framework progressif orienté composants ;
- **Angular** : framework complet disposant d'un écosystème intégré.

Le choix dépend :

- de l'équipe ;
- de l'architecture ;
- du besoin de SSR/SSG ;
- de l'écosystème ;
- des contraintes de performance ;
- de la durée de vie du projet.

## 19.3 jQuery aujourd'hui

jQuery a joué un rôle historique majeur en masquant les incompatibilités entre navigateurs et en simplifiant le DOM/Ajax.

Il reste présent dans de nombreux projets existants, mais les APIs natives modernes couvrent aujourd'hui une grande partie des besoins qui justifiaient son adoption systématique.

Pour un nouveau projet, ne pas l'ajouter par réflexe.

## 19.4 Architecture en couches

Exemple :

```text
UI / contrôleurs
       ↓
cas d'usage / services
       ↓
domaine
       ↓
adaptateurs HTTP / stockage
```

La logique métier devrait être aussi indépendante que possible du DOM ou du framework.

## 19.5 Séparer les effets de bord

```js
// Domaine pur
export function calculerTotal(lignes) {
  return lignes.reduce(
    (total, ligne) => total + ligne.prix * ligne.quantite,
    0,
  );
}
```

Puis l'adaptateur UI appelle cette logique. Cela facilite les tests et la réutilisation.

## 19.6 SPA, MPA, SSR, SSG

Une SPA n'est pas automatiquement la meilleure architecture.

| Architecture | Idée |
|---|---|
| MPA | navigation principalement gérée par le serveur |
| SPA | application client longue durée |
| SSR | HTML produit côté serveur à la requête |
| SSG | pages générées au build |
| hybride | combinaison selon les routes et besoins |

Les frameworks modernes mélangent souvent ces modèles.

---

# 20. JavaScript moderne et évolutions ECMAScript

## 20.1 Compatibilité avant nouveauté

Une fonctionnalité standard n'est utile que si les environnements ciblés la prennent en charge ou si une stratégie de compatibilité est prévue.

Avant d'utiliser une nouveauté :

1. définir les cibles ;
2. vérifier la compatibilité ;
3. évaluer transpilation/polyfill ;
4. mesurer le coût ;
5. documenter la décision.

## 20.2 Quelques APIs modernes importantes

### `Object.groupBy()`

```js
const groupes = Object.groupBy(items, (item) => item.type);
```

### `Map.groupBy()`

À utiliser lorsque les clés de groupement ne sont pas naturellement des chaînes/propriétés d'objet.

### `Promise.withResolvers()`

```js
const { promise, resolve, reject } = Promise.withResolvers();
```

### `RegExp.escape()`

Pour incorporer une chaîne littérale dans une expression régulière :

```js
const recherche = RegExp.escape(saisieUtilisateur);
const regex = new RegExp(recherche, "i");
```

Éviter de construire manuellement une fonction d'échappement incomplète si l'environnement fournit l'API standard.

### Méthodes de `Set`

```js
const a = new Set([1, 2, 3]);
const b = new Set([3, 4]);

const communs = a.intersection(b);
```

### Iterator Helpers

```js
const valeurs = grosTableau
  .values()
  .filter((x) => x > 0)
  .map((x) => x * 2)
  .take(10)
  .toArray();
```

Ils permettent des pipelines paresseux et peuvent éviter des tableaux intermédiaires.

### Import attributes

```js
import donnees from "./donnees.json" with { type: "json" };
```

L'ancienne variante avec `assert` ne doit pas être enseignée comme syntaxe standard actuelle.

## 20.3 Temporal et autres propositions

Certaines fonctionnalités très visibles peuvent être disponibles dans certains moteurs avant d'être intégrées à une édition publiée d'ECMAScript.

Ne pas annoncer une proposition comme faisant partie d'ES2026 sans vérifier la spécification publiée. C'est notamment important pour les APIs de dates/temps ou les nouvelles syntaxes encore en évolution.

## 20.4 JavaScript et IA générative

Les assistants de programmation peuvent produire rapidement du JavaScript, mais le développeur reste responsable de :

- vérifier les APIs réellement existantes ;
- supprimer les dépendances inventées ;
- vérifier les versions ;
- contrôler XSS, SSRF côté serveur, injections et secrets ;
- écrire/exécuter les tests ;
- lire le diff ;
- mesurer les performances.

Un code plausible n'est pas nécessairement un code correct.

---

# 21. Travaux pratiques

## TP 1 — Manipuler les types et collections

Créer un tableau :

```js
const produits = [
  { id: 1, nom: "Livre", prix: 15, stock: 3 },
  { id: 2, nom: "Stylo", prix: 2, stock: 0 },
  { id: 3, nom: "Cahier", prix: 5, stock: 12 },
];
```

Objectifs :

1. filtrer les produits en stock ;
2. calculer leur prix TTC ;
3. calculer la valeur totale du stock ;
4. construire une `Map` indexée par `id` ;
5. trier sans modifier le tableau original.

Solution possible :

```js
const disponibles = produits.filter((produit) => produit.stock > 0);

const avecTva = disponibles.map((produit) => ({
  ...produit,
  prixTtc: produit.prix * 1.2,
}));

const valeurStock = produits.reduce(
  (total, produit) => total + produit.prix * produit.stock,
  0,
);

const parId = new Map(produits.map((produit) => [produit.id, produit]));

const tries = produits.toSorted((a, b) => a.prix - b.prix);
```

## TP 2 — Closure

Écrire une fonction `creerLimiteur(max)` qui retourne une fonction. Chaque appel à cette fonction doit retourner `true` tant que le nombre maximum d'appels n'est pas dépassé, puis `false`.

```js
function creerLimiteur(max) {
  let appels = 0;

  return () => {
    if (appels >= max) {
      return false;
    }

    appels += 1;
    return true;
  };
}
```

## TP 3 — Modules

Créer :

```text
src/
├── calcul.js
├── format.js
└── main.js
```

Contraintes :

- `calcul.js` exporte des fonctions de calcul ;
- `format.js` exporte un formatter ;
- `main.js` importe uniquement les fonctions nécessaires ;
- aucun symbole métier inutile dans `globalThis`.

## TP 4 — API asynchrone

Créer une fonction :

```js
async function chargerJson(url, { timeout = 5000 } = {}) {
  const response = await fetch(url, {
    signal: AbortSignal.timeout(timeout),
  });

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  return response.json();
}
```

Puis :

1. gérer une erreur HTTP ;
2. gérer un timeout ;
3. lancer trois requêtes indépendantes avec `Promise.all()` ;
4. expliquer pourquoi trois `await` successifs sont différents.

## TP 5 — Todo List DOM

Construire une Todo List sans framework :

- formulaire d'ajout ;
- liste de tâches ;
- bouton de suppression ;
- délégation d'événements ;
- persistance locale ;
- aucun `innerHTML` avec du contenu utilisateur.

Objectif : distinguer logique métier et manipulation DOM.

## TP 6 — Tests

Créer une fonction :

```js
export function appliquerRemise(total, pourcentage) {
  if (total < 0) {
    throw new RangeError("Le total doit être positif");
  }

  if (pourcentage < 0 || pourcentage > 100) {
    throw new RangeError("Remise invalide");
  }

  return total * (1 - pourcentage / 100);
}
```

Écrire avec `node:test` des tests pour :

- 0 % ;
- 20 % ;
- 100 % ;
- total négatif ;
- pourcentage supérieur à 100.

## TP 7 — Mini API Node.js

Créer une API HTTP minimale qui expose :

```text
GET  /health
GET  /api/items
POST /api/items
```

Contraintes :

- JSON valide ;
- contrôle des erreurs ;
- validation des entrées ;
- aucun secret dans le dépôt ;
- tests des fonctions métier séparément du transport HTTP.

## TP 8 — Sécurité DOM

On fournit :

```js
const commentaire = new URLSearchParams(location.search).get("commentaire");
resultat.innerHTML = commentaire;
```

Travail :

1. identifier la vulnérabilité ;
2. proposer un payload pédagogique non destructif ;
3. corriger avec `textContent` ;
4. expliquer pourquoi CSP constitue une défense supplémentaire et non un remplacement de l'encodage/sanitization.

## TP 9 — Performance

À partir d'une liste de 100 000 objets :

1. comparer des recherches répétées avec `Array.find()` ;
2. créer une `Map` par identifiant ;
3. mesurer avant/après ;
4. expliquer le coût de construction de l'index ;
5. déterminer dans quel scénario l'indexation est pertinente.

## TP 10 — Projet final

Construire une petite application complète avec :

- modules ESM ;
- logique métier pure ;
- DOM ou API Node.js ;
- appels asynchrones ;
- validation ;
- gestion d'erreurs ;
- tests ;
- ESLint ;
- documentation ;
- revue de sécurité ;
- README avec commandes de développement.

Livrables :

```text
.
├── package.json
├── package-lock.json
├── eslint.config.js
├── README.md
├── src/
└── test/
```

---

# 22. Checklist de bonnes pratiques

## Langage

- [ ] utiliser `const` par défaut et `let` si réaffectation ;
- [ ] éviter `var` dans le nouveau code ;
- [ ] utiliser `===` / `!==` par défaut ;
- [ ] distinguer `null` et `undefined` ;
- [ ] ne pas dire que « tout est objet » ;
- [ ] ne pas supposer que `const` rend un objet immuable ;
- [ ] préférer des conversions explicites lorsque la coercition est ambiguë ;
- [ ] utiliser `?.` et `??` lorsque leur sémantique correspond au besoin.

## Fonctions et architecture

- [ ] garder les fonctions petites et cohésives ;
- [ ] isoler les effets de bord ;
- [ ] préférer la composition aux hiérarchies complexes ;
- [ ] éviter un état global inutile ;
- [ ] exposer une API de module explicite.

## Asynchronisme

- [ ] gérer les rejets ;
- [ ] contrôler `response.ok` avec `fetch()` ;
- [ ] paralléliser les opérations indépendantes avec discernement ;
- [ ] prévoir annulation et timeout pour les opérations longues ;
- [ ] comprendre tâches et microtasks.

## Navigateur

- [ ] utiliser les éléments HTML sémantiques ;
- [ ] préférer `textContent` pour du texte non fiable ;
- [ ] nettoyer listeners/timers si leur cycle de vie est limité ;
- [ ] ne pas stocker de secrets dans le bundle ou `localStorage` ;
- [ ] valider côté serveur même si le formulaire valide côté client.

## Node.js

- [ ] utiliser une branche maintenue ;
- [ ] valider les variables d'environnement ;
- [ ] ne pas bloquer inutilement l'event loop ;
- [ ] utiliser les streams pour les gros flux ;
- [ ] journaliser sans exposer de secrets.

## Qualité

- [ ] tests automatisés ;
- [ ] linter ;
- [ ] lockfile versionné ;
- [ ] CI reproductible ;
- [ ] dépendances minimales et maintenues ;
- [ ] revue du diff ;
- [ ] mesures avant optimisation.

## Sécurité

- [ ] aucune confiance dans les entrées externes ;
- [ ] aucun `eval()` sur des données externes ;
- [ ] protection XSS ;
- [ ] stratégie CSRF si authentification par cookie ;
- [ ] CSP adaptée ;
- [ ] aucun secret côté client ;
- [ ] dépendances auditées ;
- [ ] validation des redirects et URLs externes.

---

# 23. Références

Références de travail à privilégier pour vérifier une API ou une fonctionnalité :

- **ECMA-262 / TC39** : spécification normative ECMAScript ;
- **MDN Web Docs** : documentation JavaScript et Web APIs, avec tableaux de compatibilité ;
- **Node.js Documentation** : runtime et APIs serveur ;
- **WHATWG HTML / Fetch / DOM** : standards de la plateforme Web ;
- **web.dev** : pratiques Web, performance et compatibilité ;
- **OWASP** : sécurité des applications Web et JavaScript.

> [!warning] Vérifier la compatibilité
> JavaScript et les Web APIs évoluent continuellement. Avant d'utiliser une fonctionnalité récente, vérifier à la fois son statut dans le standard et sa disponibilité dans les navigateurs/runtimes réellement ciblés.

---

# Conclusion

Maîtriser JavaScript ne consiste pas à mémoriser un framework. Les compétences durables sont :

1. comprendre les types, valeurs et coercitions ;
2. maîtriser fonctions, portée, closures et `this` ;
3. comprendre prototypes et objets ;
4. utiliser correctement modules et collections ;
5. raisonner sur l'event loop et les Promises ;
6. distinguer ECMAScript des APIs du navigateur et de Node.js ;
7. valider les données et traiter les erreurs ;
8. écrire des tests ;
9. penser sécurité et performance ;
10. vérifier la compatibilité avant d'adopter une nouveauté.

Avec ces bases, apprendre un framework JavaScript devient principalement une question d'API et d'architecture plutôt qu'une découverte permanente du langage lui-même.
