Plan de cours : Javascript - Des origines aux techniques avancées

1. **Introduction à JavaScript**
    - Définition et utilité du langage
    - Rôle dans le développement web
    - Place de JavaScript dans la pile technologique

2. **Historique de JavaScript**
    - Naissance et premières versions (LiveScript, ECMAScript)
    - Les principales évolutions et les étapes clés (ES3, ES5, ES6/ES2015…)
    - Navigateurs et compatibilité : la guerre des navigateurs, Node.js

3. **Concepts fondamentaux**
    - Variables, types et opérateurs
    - Fonctions, portées et closures
    - Objets et prototypes
    - Gestion des erreurs et exceptions

4. **Asynchronicité en JavaScript**
    - Callbacks et problèmes courants ("Callback Hell")
    - Promesses et l'API `Promise`
    - `async` et `await` : écrire du code asynchrone de manière synchrone

5. **Le DOM et l'interaction avec les pages web**
    - Sélection d'éléments et manipulation
    - Gestion des événements
    - Ajax et l'échange de données avec le serveur

6. **Frameworks et librairies modernes**
    - Introduction à jQuery : simplification des interactions DOM
    - Frameworks MVC/MVVM : Angular, React, Vue.js
    - Frameworks côté serveur : Node.js, Express.js
    - Outils de construction et bundlers : Webpack, Babel

7. **Techniques avancées et tendances modernes**
    - Programmation fonctionnelle avec JavaScript
    - Web Components et Shadow DOM
    - Service Workers et Progressive Web Apps (PWAs)
    - WebAssembly : exécuter du code d'autres langages dans le navigateur

8. **Sécurité en JavaScript**
    - Problèmes courants : Cross-Site Scripting (XSS), Cross-Site Request Forgery (CSRF)
    - Bonnes pratiques de sécurisation
    - Content Security Policy (CSP)

9. **Tests et assurance qualité**
    - Introduction aux tests unitaires : Jasmine, Mocha, Jest
    - Tests d'intégration et tests de bout en bout (E2E) : Protractor, Cypress
    - Linters et analyseurs statiques : ESLint, TSLint

10. **Projets réels et études de cas**
    - Analyse de projets open source populaires
    - Best practices en développement JavaScript moderne
    - Atelier : Création d'une application complète en utilisant les concepts abordés

11. **Conclusion et perspectives**
    - Les futurs standards : où va ECMAScript ?
    - Autres domaines d'application de JavaScript (IoT, applications desktop avec Electron, etc.)
    - Écosystème et ressources pour continuer à apprendre


# 1 Introduction à JavaScript

## 1.1 Origines de JavaScript

### 1.1.1 La création par Netscape et son introduction avec Netscape Navigato

Au milieu des années 90, le paysage du web était en pleine effervescence. Netscape, l'une des entreprises les plus influentes de cette époque, voyait le potentiel du web et cherchait des moyens de le rendre plus interactif et engageant. Nous devons reconnaître l'ingéniosité de Brendan Eich, un ingénieur chez Netscape, qui a développé le premier prototype de JavaScript en seulement dix jours en 1995. Il était alors connu sous le nom de Mocha, puis LiveScript avant d'être finalement baptisé JavaScript.

L'intégration de JavaScript dans le navigateur Netscape Navigator a changé la donne. Auparavant, les pages web étaient largement statiques, mais avec l'introduction de JavaScript, les développeurs ont eu la capacité de créer des pages dynamiques répondant aux actions des utilisateurs en temps réel.

### 1.1.2 Distinction entre Java et JavaScript

Si nous revenons sur ce choix de nom, "JavaScript", nous ne pouvons nous empêcher de relever sa ressemblance avec "Java", un autre langage de programmation qui gagnait en popularité à l'époque. Cette similitude n'est pas accidentelle. L'idée était en partie marketing : capitaliser sur la popularité croissante de Java pour gagner en traction. Cependant, cela a souvent conduit à la confusion, une confusion que nous rencontrons encore parfois aujourd'hui.

Il est primordial pour nous de clarifier : malgré leur nom similaire, Java et JavaScript sont fondamentalement différents à bien des égards :
  
- **Nature et utilisation** : Java est un langage de programmation orienté objet qui peut être utilisé pour créer des applications indépendantes, des applets ou des applications serveur. JavaScript, quant à lui, a été conçu principalement pour dynamiser les pages web.

- **Environnement d'exécution** : Java nécessite une machine virtuelle (JVM) pour exécuter son code, tandis que JavaScript est généralement exécuté directement par les navigateurs web.

- **Syntaxe et design** : Bien que les deux langages partagent certaines similarités syntaxiques, leurs paradigmes et leurs mécanismes sous-jacents, comme la gestion de la mémoire ou le système de types, diffèrent grandement.

## 1.2 Rôle principal de JavaScript

Lorsque nous parlons de développement web, il est presque impossible de ne pas mentionner JavaScript. C'est l'un des langages les plus influents qui a façonné et continue de définir l'expérience en ligne telle que nous la connaissons aujourd'hui. Examinons ensemble le rôle principal de ce langage.

### 1.2.1 Langage de script pour le Web

L'une des premières choses que nous devons comprendre est que JavaScript est un *langage de script*. Mais qu'est-ce que cela signifie exactement pour nous?

Un langage de script est un type de langage de programmation qui automatise l'exécution de tâches qui seraient autrement effectuées étape par étape manuellement. Dans le contexte du web, cela signifie que JavaScript est utilisé pour donner des instructions aux navigateurs sur comment et quand effectuer des actions spécifiques sans nécessiter de rechargement complet de la page.

Si nous regardons sous le capot d'un site web typique, nous constatons que la structure est définie par le HTML, le style par le CSS, et le comportement ou l'interactivité par JavaScript. En fait, JavaScript est la couche qui donne vie à une page web, la rendant réactive aux actions de l'utilisateur.

### 1.2.2 Dynamisation des pages HTML 

#### 1.2.2.1 Animation

Avec JavaScript, nous pouvons animer des éléments sur une page, que ce soit des éléments qui se déplacent, changent de couleur, se redimensionnent ou même se transforment de manière complexe. Pensez à des carrousels d'images, des effets de parallaxe ou des transitions douces entre les états.

#### 1.2.2.2 Interactivité

Imaginez une galerie d'images où chaque clic sur une image la fait s'agrandir sans avoir besoin de charger une nouvelle page. Ou peut-être un menu déroulant qui apparaît lorsque l'utilisateur passe la souris dessus. Tout cela est rendu possible grâce à JavaScript.

#### 1.2.2.3 Validation

Lorsque nous remplissons un formulaire en ligne, comme une inscription à un bulletin d'information ou un formulaire de contact, JavaScript peut vérifier instantanément la validité des informations saisies. Il peut s'assurer que l'adresse e-mail est dans un format correct, que tous les champs obligatoires sont remplis, et bien plus encore.

#### 1.2.2.4 Chargement dynamique de contenu

Vous êtes-vous déjà demandé comment les flux sociaux ou les sections "articles recommandés" sur certains sites web peuvent charger de nouveaux éléments sans recharger toute la page? Avec l'aide de JavaScript et des techniques AJAX, ces éléments sont ajoutés dynamiquement, offrant une expérience fluide pour l'utilisateur.


JavaScript est un *langage de script*, ainsi il automatise l'exécution de tâches qui seraient autrement effectuées étape par étape manuellement. JavaScript permet de donner des instructions aux navigateurs sur comment et quand effectuer des actions spécifiques sans nécessiter de rechargement complet de la page.

Si nous regardons sous le capot d'un site web typique, nous constatons que la structure est définie par le HTML, le style par le CSS, et le comportement ou l'interactivité par JavaScript. En fait, JavaScript est la couche qui donne vie à une page web, la rendant réactive aux actions de l'utilisateur.


### 1.2.3 Single Page Applications (SPA)

Les SPAs sont des applications ou des sites web qui interagissent avec l'utilisateur en modifiant dynamiquement la page actuelle, plutôt qu'en rechargeant de nouvelles pages depuis le serveur. Cela offre une expérience plus fluide et rapide.

- **Concept et avantage des SPAs** : L'idée derrière les SPAs est de rendre l'expérience utilisateur aussi fluide que possible, en évitant les rechargements de page inutiles. Cette architecture offre une navigation plus rapide et une consommation réduite de données, car seuls les contenus nécessaires sont chargés à la demande.

- **Le rôle crucial de JavaScript** : Dans une SPA, c'est JavaScript qui gère la logique de la page, le chargement du contenu, la navigation entre les différentes vues, etc. Grâce à des frameworks comme React, Angular ou Vue.js, nous pouvons construire des SPAs robustes et efficaces.

## 1.3 Place de JavaScript dans la pile technologique**

- **Côté client vs. côté serveur** :
  - Utilisation traditionnelle de JavaScript côté client.
  - Introduction de Node.js : permettant l'exécution de JavaScript côté serveur.

- **APIs et échanges avec d'autres technologies** :
  - Comment JavaScript peut communiquer avec des bases de données, d'autres langages et des API externes.

- **Environnements d'exécution** :
  - Navigateurs web : Chrome, Firefox, Safari, Edge.
  - Serveurs : Node.js.
  - Applications mobiles : React Native, Ionic.
  - Applications de bureau : Electron.

# 2. Historique de JavaScript

## 2.1 Naissance et premières versions (LiveScript, ECMAScript)

Le voyage de JavaScript a commencé en 1995 avec Netscape. La société recherchait un moyen de dynamiser le contenu des pages web et d'offrir une expérience plus interactive aux utilisateurs de son navigateur, Netscape Navigator.

- **LiveScript** : À l'origine, le langage fut nommé "Mocha", puis renommé "LiveScript". C'est Brendan Eich qui a été chargé de sa conception, et il l'a créé en seulement dix jours ! 
- **Renommage en JavaScript** : Peu de temps après, en raison d'un partenariat stratégique avec Sun Microsystems et la popularité croissante de Java, LiveScript a été renommé "JavaScript", un choix marketing qui, malheureusement, a prêté à confusion car Java et JavaScript sont deux langages distincts.
- **ECMAScript** : En 1997, pour standardiser le langage et assurer une uniformité entre différents navigateurs, le langage a été officialisé sous le nom d'ECMAScript (ES) par l'organisation ECMA International. Le nom "JavaScript" est resté en tant que nom commercial, mais les spécifications officielles sont publiées sous le nom d'ECMAScript.

## 2.2 Les principales évolutions et les étapes clés (ES3, ES5, ES6/ES2015…)

JavaScript, comme tout bon langage de programmation, a connu plusieurs mises à jour et évolutions majeures au fil des ans.

- **ES3 (1999)** : Cette version a apporté de nombreuses améliorations à la syntaxe et introduit des concepts tels que les expressions régulières et les try/catch pour la gestion des erreurs.
- **ES5 (2009)** : Après une longue période sans mise à jour majeure, ES5 est arrivé avec des méthodes d'objet supplémentaires, la prise en charge stricte ("strict mode") et les getters/setters.
- **ES6/ES2015** : Publiée en 2015, cette version a été un tournant pour JavaScript, introduisant des fonctionnalités comme les classes, les promesses, les modules, le mot-clé `let` pour la déclaration de variables et bien d'autres encore. Les mises à jour annuelles ont commencé à partir de cette version, apportant régulièrement de nouvelles fonctionnalités et améliorations.

## 2.3 Navigateurs et compatibilité : la guerre des navigateurs, Node.js

L'évolution de JavaScript ne peut être complètement comprise sans évoquer la "guerre des navigateurs". À la fin des années 1990 et au début des années 2000, les principaux navigateurs (principalement Netscape Navigator et Internet Explorer) ont tenté d'ajouter leurs propres fonctionnalités exclusives à JavaScript, ce qui a entraîné des incompatibilités entre les navigateurs.

- **La guerre des navigateurs** : Cette période a été marquée par des différences notables dans la manière dont les navigateurs implémentaient les fonctionnalités, obligeant les développeurs à écrire du code spécifique à chaque navigateur. La situation s'est stabilisée avec la normalisation croissante des spécifications ECMAScript et la montée en puissance d'autres navigateurs comme Firefox, Chrome et Safari.
- **Node.js** : L'introduction de Node.js en 2009 a élargi les horizons de JavaScript, lui permettant de s'exécuter côté serveur. Cela a également conduit à la création de l'écosystème npm, un élément crucial pour la communauté JavaScript d'aujourd'hui.

# 3. Concepts fondamentaux

## 3.1 Variables

En JavaScript, une variable est un conteneur pour stocker des données. Avec l'aide des mots-clés `var` (ancienne méthode), `let` (introduit dans ES6), `const` (aussi introduit dans ES6 pour les valeurs immuables), nous pouvons déclarer des variables.

La création d'une variable sans affectation assigne automatiquement la valeur `undefined` 
```js
>> let a
undefined
>> a == undefined
true
```

Création d'un tableau :
```js
>> t = ["un","deux","trois"]
Array(3) [ "un", "deux", "trois" ]
>> t[0]
"un"
>> t[1]
"deux"
```
La création d'un  dictionnaire :
```js
>> d = {un:1, "deux":2, "plein de nombres":[0,1,2,3]}
Object { un: 1, deux: 2, "plein de nombres": (4) […] }
>> d.un
1
>> d.deux
2
>> d["deux"]
2
>> d["plein de nombres"]
Array(4) [ 0, 1, 2, 3 ]
```
Remarque, l'accès aux valeurs d'un dictionnaire peut être fait directement comme si c'était un attribut lorsque la clé est simple, sinon avec les crochets.
Une clé d'un dictionnaire peut être la valeur d'une variable lors de sa création si l'on entoure le nom de la variable de crochets (clé dynamique):
```js
>> a = "aA"
"A"
>> d2 = {[a]:"Un A"}
Object { aA: "Un A" }
```

**Remarque :**
Dans un même bloc de code, il n'est pas possible de déclarer deux fois la même variable.
## 3.2 Types

JavaScript est un langage à typage dynamique, ce qui signifie que vous n'avez pas besoin de déclarer le type de variable lors de sa création. Les types primitifs comprennent `Number`, `String`, `Boolean`, `Undefined`, `Null`, `BigInt`, et `Symbol`.

Pour connaitre le type d'un objet, nous utilisons `typeof` :
```js
a_string = "hello"
typeof a_string
```

### Conversions automatiques

Attention aux conversions automatiques :
```js
>> 4 + "2"
'42'
>> 4 * "2"
'8'
>> "cou"*2
NaN
```
## 3.3 Opérateurs

Ces symboles sont utilisés pour effectuer des opérations. Nous avons des opérateurs arithmétiques (comme +, -, *, /), des opérateurs de comparaison (comme ==, !=, ===, !==), des opérateurs logiques (comme &&, ||, !) et d'autres.

## 3.3 Fonctions

Une fonction est un bloc de code conçu pour effectuer une tâche particulière. Elle est définie avec le mot-clé `function` et peut être appelée par son nom.

## 3.4 Portées

La portée détermine la visibilité d'une variable dans le code. En JavaScript, nous avons la portée globale et la portée locale (fonction). Avec l'introduction de `let` et `const` dans ES6, nous avons également la notion de portée de bloc.

## 3.5 Closures

C'est un concept fondamental en JavaScript qui permet à une fonction d'accéder à toutes les variables de son environnement lexical. Cela signifie qu'une fonction peut "se souvenir" des variables de son environnement externe, même si elle est exécutée en dehors de cet environnement.

## 3.6 Objets

Tout en JavaScript est un objet. Un objet est une collection de paires clé-valeur. Les objets peuvent être créés à l'aide de littéraux d'objets ou du mot-clé `new`.

## 3.7 Prototypes

Chaque objet en JavaScript a un prototype, qui est aussi un objet. Les prototypes permettent à JavaScript d'implémenter un type spécial d'héritage, appelé héritage prototypal. C'est la manière dont les propriétés et les méthodes sont partagées entre les objets.

## 3.8 try...catch**

Le bloc `try...catch` nous permet de tester un bloc de code pour des erreurs. Si une erreur se produit dans le bloc `try`, le code peut "attraper" cette erreur et la gérer à l'aide du bloc `catch`.

## 3.9 throw

Si nous voulons générer une erreur manuellement en fonction de certaines conditions, nous pouvons utiliser le mot-clé `throw`.

## 3.10 finally

C'est un bloc qui sera exécuté quelle que soit l'issue du bloc `try...catch`, que ce soit une erreur ou non.
