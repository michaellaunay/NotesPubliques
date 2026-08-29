---
schema_version: 1
uid: "01M02EX5C7BG2HNVQYHHEH1VE8"
titre: "SEO"
aliases:
  - "Référencement naturel"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
  - communication
themes:
  - marketing
  - referencement
  - seo
  - developpement-web
resume: "Cours complet sur le référencement naturel moderne : fonctionnement des moteurs de recherche, stratégie éditoriale, SEO on-page et technique, indexation, performances, données structurées, popularité, mesure et visibilité dans les fonctionnalités de recherche générative."
niveau: intermediaire
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-08-09
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Optimisation pour les moteurs de recherche — SEO

> [!abstract] Objectif
> Rendre un site découvrable, compréhensible et digne de confiance pour les moteurs de recherche et les réponses génératives (AI Overviews, AI Mode) : fondamentaux, intention de recherche, contenu, SEO technique, performance, données structurées, mesure et audit.

Voir aussi : [[HTML]], [[HTTP]], [[Javascript]], [[LLM]].

> [!NOTE]
> Ce cours est actualisé pour l'état du Web et de Google Search en août 2026. Le SEO évolue continuellement : les principes durables sont distingués des fonctionnalités susceptibles de changer rapidement.

Le **SEO** (*Search Engine Optimization*, ou référencement naturel) regroupe les méthodes permettant d'améliorer la **découvrabilité**, la **compréhension**, l'**indexabilité** et la **visibilité** d'un contenu dans les moteurs de recherche.

Le SEO moderne ne consiste pas à « tromper l'algorithme » ni à répéter des mots-clés. Il consiste surtout à :

- publier un contenu réellement utile à une audience identifiée ;
- rendre ce contenu accessible aux moteurs de recherche ;
- fournir une architecture et des signaux techniques cohérents ;
- offrir une bonne expérience aux utilisateurs ;
- construire une réputation et une autorité réelles ;
- mesurer les résultats et corriger les problèmes ;
- rester conforme aux règles antispam des moteurs.

Le référencement concerne désormais plusieurs formes de résultats :

- liens Web classiques ;
- résultats enrichis (*rich results*) ;
- images et vidéos ;
- résultats locaux ;
- produits ;
- actualités ;
- extraits et réponses directes ;
- fonctionnalités de recherche utilisant l'IA générative, notamment **AI Overviews** et **AI Mode** chez Google.

---

## Sommaire

1. [[#1. Fondamentaux du SEO]]
2. [[#2. Fonctionnement d'un moteur de recherche]]
3. [[#3. Stratégie SEO et intention de recherche]]
4. [[#4. SEO on-page et contenu]]
5. [[#5. Architecture du site et maillage interne]]
6. [[#6. SEO technique : exploration, indexation et canonicalisation]]
7. [[#7. Performance, expérience utilisateur et Core Web Vitals]]
8. [[#8. SEO JavaScript et applications Web modernes]]
9. [[#9. Données structurées et apparence dans les résultats]]
10. [[#10. SEO off-page, liens et réputation]]
11. [[#11. SEO local, international et e-commerce]]
12. [[#12. SEO, IA générative, AI Overviews et AI Mode]]
13. [[#13. Mesure, outils et indicateurs]]
14. [[#14. Méthodologie d'audit SEO]]
15. [[#15. Migrations et changements d'URL]]
16. [[#16. Spam, erreurs fréquentes et pratiques à éviter]]
17. [[#17. Cas pratique : construire une stratégie SEO]]
18. [[#18. Checklist opérationnelle]]
19. [[#19. Conclusion]]
20. [[#20. Sources et références]]

---

# 1. Fondamentaux du SEO

## 1.1 Définition

Le SEO vise à améliorer la présence d'un site dans les résultats **non publicitaires** des moteurs de recherche.

Il faut distinguer :

| Terme | Signification | Exemple |
|---|---|---|
| **SEO** | Référencement naturel | Optimiser une page pour une requête |
| **SEA** | Publicité sur les moteurs | Google Ads, Microsoft Advertising |
| **SEM** | Ensemble des actions marketing liées aux moteurs | SEO + SEA au sens courant |
| **SMO** | Optimisation pour les médias sociaux | LinkedIn, Mastodon, Instagram, etc. |

Le trafic SEO n'est pas réellement « gratuit » : l'acquisition d'un clic n'entraîne généralement pas un paiement au moteur, mais le contenu, la technique, les outils et le travail humain ont un coût.

## 1.2 Les trois grands piliers

On peut regrouper le SEO en trois familles complémentaires.

### Technique

Le moteur doit pouvoir :

1. découvrir les URL ;
2. les explorer ;
3. récupérer leurs ressources importantes ;
4. comprendre et rendre le contenu ;
5. déterminer les URL canoniques ;
6. indexer les pages utiles.

### Contenu

La page doit :

- répondre à une intention réelle ;
- être claire et structurée ;
- apporter une valeur propre ;
- être à jour lorsque le sujet le nécessite ;
- rendre explicites ses auteurs, ses sources et son contexte quand cela améliore la confiance.

### Popularité et réputation

Le Web est un graphe. Les liens, citations et mentions permettent notamment de découvrir des contenus et d'estimer leur importance ou leur réputation.

> [!IMPORTANT]
> La popularité ne se résume pas au nombre de backlinks. Un petit nombre de liens éditoriaux réellement pertinents peut être bien plus utile qu'un grand volume de liens artificiels.

## 1.3 Ce que le SEO ne garantit pas

Même une page techniquement parfaite et conforme aux recommandations :

- n'est pas garantie d'être explorée immédiatement ;
- n'est pas garantie d'être indexée ;
- n'est pas garantie d'obtenir une position particulière ;
- n'est pas garantie d'obtenir un résultat enrichi ;
- n'est pas garantie d'être citée par une fonctionnalité d'IA.

Les moteurs sélectionnent les résultats selon la requête, le contexte, la concurrence, la qualité perçue et de nombreux autres signaux.

## 1.4 Une discipline centrée sur l'utilisateur

Une règle de décision simple est :

> « Cette modification rend-elle la page plus utile, plus compréhensible ou plus accessible pour l'utilisateur ? »

Si une action n'a d'autre justification que la manipulation supposée d'un algorithme, elle est généralement fragile à long terme.

---

# 2. Fonctionnement d'un moteur de recherche

Le fonctionnement exact d'un moteur est propriétaire, mais on peut distinguer cinq étapes conceptuelles.

```text
Découverte -> Exploration -> Rendu -> Indexation -> Classement / diffusion
```

## 2.1 Découverte des URL

Un moteur découvre des URL par exemple grâce :

- aux liens HTML ;
- aux sitemaps XML ;
- aux URL déjà connues ;
- aux redirections ;
- aux flux ou autres sources propres au moteur.

Un lien HTML simple reste la forme la plus robuste :

```html
<a href="/documentation/seo">Cours de SEO</a>
```

## 2.2 Exploration (*crawling*)

Un robot comme **Googlebot** télécharge des ressources accessibles sur le Web.

L'exploration dépend notamment :

- de la disponibilité du serveur ;
- de `robots.txt` ;
- du nombre et de la qualité des URL découvertes ;
- de l'utilité estimée de leur nouvelle exploration ;
- de la capacité du site à répondre correctement.

Sur les très grands sites, on parle parfois de **budget d'exploration** (*crawl budget*). Pour un petit site bien conçu, ce problème est généralement secondaire.

## 2.3 Rendu (*rendering*)

Pour une page simple, le HTML reçu contient déjà le contenu.

Pour une application JavaScript, le moteur peut devoir :

1. télécharger le document HTML ;
2. télécharger le JavaScript et les ressources nécessaires ;
3. exécuter le code ;
4. analyser le DOM résultant.

Le rendu JavaScript ajoute de la complexité et peut retarder ou empêcher la découverte de certains contenus si l'application est mal conçue.

## 2.4 Indexation

L'indexation consiste à analyser et stocker des informations sur une page :

- texte principal ;
- langue ;
- titres ;
- images ;
- liens ;
- données structurées ;
- relations avec d'autres URL ;
- page canonique ;
- signaux de qualité ou de spam.

Une page explorée n'est donc pas nécessairement indexée.

## 2.5 Classement et diffusion

Lors d'une recherche, le moteur doit identifier les documents susceptibles de répondre à la requête, puis les ordonner.

Les systèmes de classement cherchent notamment à apprécier :

- la pertinence vis-à-vis de la requête ;
- la qualité et l'utilité du contenu ;
- le contexte géographique ou linguistique ;
- la fraîcheur lorsque la requête l'exige ;
- l'autorité ou la réputation des sources ;
- l'expérience globale proposée par la page.

Il n'existe pas de formule publique du type :

```text
score = 30 % mots-clés + 20 % backlinks + 10 % vitesse + ...
```

Les listes de « 200 facteurs de classement » doivent donc être considérées avec prudence.

## 2.6 Les moteurs de recherche

### Google

Google domine de nombreux marchés et dispose d'un écosystème comprenant notamment Search, Images, Maps, Discover, Shopping et les fonctionnalités génératives.

### Microsoft Bing

Bing est le moteur de Microsoft et alimente ou contribue à différents produits et services de recherche.

### DuckDuckGo

DuckDuckGo met particulièrement en avant la protection de la vie privée et s'appuie sur plusieurs sources pour produire ses résultats.

### Baidu, Naver et autres moteurs régionaux

Une stratégie internationale doit tenir compte du moteur réellement utilisé dans le pays ciblé. Les pratiques et outils disponibles ne sont pas nécessairement identiques à ceux de Google.

---

# 3. Stratégie SEO et intention de recherche

## 3.1 Partir des utilisateurs, pas uniquement des mots-clés

Une stratégie SEO commence par comprendre :

- qui recherche ;
- quel problème cette personne cherche à résoudre ;
- à quel moment de son parcours elle se trouve ;
- quelle forme de réponse serait la plus utile.

Exemple : les requêtes suivantes parlent toutes d'un même produit mais expriment des besoins différents.

```text
chaussures randonnée
meilleures chaussures randonnée pluie
chaussures randonnée gore-tex homme 44
réparer semelle chaussure randonnée
```

## 3.2 Intentions de recherche

On distingue souvent quatre grandes intentions.

| Intention | Objectif | Exemple |
|---|---|---|
| Informationnelle | Apprendre | `comment fonctionne un sitemap` |
| Navigationnelle | Trouver un site précis | `github documentation` |
| Commerciale | Comparer avant décision | `meilleur hébergeur python` |
| Transactionnelle | Effectuer une action | `acheter ssd 4 to` |

Une même requête peut combiner plusieurs intentions.

## 3.3 Recherche de sujets et de requêtes

Les sources utiles comprennent :

- Search Console ;
- suggestions et recherches associées des moteurs ;
- échanges avec les utilisateurs ;
- support client ;
- documentation interne ;
- outils SEO tiers ;
- analyse des résultats réellement affichés pour une requête.

La recherche de mots-clés sert à comprendre la demande. Elle ne doit pas mener à créer une page artificielle pour chaque variante syntaxique.

## 3.4 Regrouper par sujet

Plutôt que :

```text
/page/chaussure-randonnee
/page/chaussures-randonnee
/page/chaussure-de-randonnee
/page/meilleure-chaussure-randonnee
```

on peut souvent créer une ressource forte qui couvre correctement le besoin principal et ses sous-questions.

## 3.5 Analyser la SERP

La page de résultats elle-même renseigne sur ce que le moteur estime utile :

- articles ;
- pages catégories ;
- fiches produit ;
- vidéos ;
- cartes locales ;
- images ;
- résultats enrichis ;
- fonctionnalités génératives.

L'objectif n'est pas de copier les concurrents, mais de comprendre le format attendu puis d'apporter davantage de valeur.

## 3.6 Prioriser

Une opportunité SEO peut être évaluée par exemple selon :

```text
Priorité ≈ valeur métier × demande × adéquation × probabilité de réussite / coût
```

Ce n'est pas une formule de classement Google : c'est un outil interne de décision.

---

# 4. SEO on-page et contenu

## 4.1 Le contenu principal

Une page doit répondre rapidement à son sujet principal puis développer les éléments nécessaires.

Un bon contenu peut inclure :

- explications ;
- données originales ;
- exemples ;
- démonstrations ;
- retours d'expérience ;
- comparaisons ;
- images ou vidéos ;
- références vers des sources primaires.

La longueur n'est pas une finalité. Une réponse de 300 mots peut être meilleure qu'un texte de 3 000 mots rempli de répétitions.

## 4.2 Contenu *people-first*

Un contenu pensé d'abord pour l'utilisateur :

- a une audience identifiable ;
- démontre une connaissance réelle du sujet ;
- répond à la question annoncée ;
- évite les paragraphes ajoutés uniquement pour « faire du SEO » ;
- permet au lecteur d'agir ou de comprendre après lecture.

## 4.3 E-E-A-T

**E-E-A-T** signifie :

- **Experience** : expérience concrète ;
- **Expertise** : compétence ;
- **Authoritativeness** : légitimité / autorité ;
- **Trustworthiness** : fiabilité.

> [!IMPORTANT]
> E-E-A-T n'est pas un unique « facteur de classement » possédant un score SEO public. C'est un cadre utilisé notamment dans les consignes de qualité et utile pour réfléchir à la confiance accordée au contenu.

La fiabilité est particulièrement importante pour les sujets **YMYL** (*Your Money or Your Life*) pouvant affecter par exemple :

- la santé ;
- la sécurité ;
- les finances ;
- les droits ;
- le bien-être de la société.

Pour ces sujets, il est particulièrement utile de présenter clairement :

- l'auteur ;
- ses compétences pertinentes ;
- la date de mise à jour ;
- les sources ;
- les limites ou incertitudes.

## 4.4 Le titre HTML

Le `<title>` décrit le contenu de la page :

```html
<title>Comprendre robots.txt : règles et exemples | Exemple</title>
```

Bonnes pratiques :

- un titre spécifique à la page ;
- descriptif et compréhensible ;
- éviter le bourrage de mots-clés ;
- éviter les titres identiques sur toutes les pages.

Google peut générer le **title link** affiché dans les résultats à partir de plusieurs signaux et n'est pas obligé d'afficher exactement le `<title>` fourni.

## 4.5 Meta description

```html
<meta
  name="description"
  content="Comprendre le rôle de robots.txt, ses limites et les erreurs courantes avec des exemples pratiques."
>
```

La meta description :

- peut être utilisée comme extrait dans les résultats ;
- aide à présenter la page ;
- n'est pas une garantie de l'extrait affiché ;
- n'est pas à traiter comme un bouton magique de classement.

Le moteur peut générer un extrait différent à partir du contenu de la page selon la requête.

## 4.6 Titres H1 à H6

Exemple :

```html
<h1>Guide du référencement technique</h1>
<h2>Exploration</h2>
<h3>robots.txt</h3>
<h3>Sitemaps</h3>
<h2>Indexation</h2>
```

Les titres servent surtout à structurer le document pour les lecteurs et les technologies d'assistance.

Éviter :

- d'utiliser un titre uniquement pour sa taille visuelle ;
- une hiérarchie incompréhensible ;
- des dizaines de titres répétant la même expression.

## 4.7 URL

Une URL doit être stable, simple à comprendre et aussi durable que possible.

Préférer :

```text
https://example.org/guides/seo-technique
```

à :

```text
https://example.org/index.php?id=4827&cat=12&session=abc
```

Les paramètres ne sont pas interdits, mais il faut éviter les combinaisons inutiles produisant un grand nombre d'URL équivalentes.

## 4.8 Images

Exemple :

```html
<img
  src="/images/schema-crawl-indexation.webp"
  alt="Étapes entre exploration, rendu et indexation d'une page"
  width="1200"
  height="700"
>
```

L'attribut `alt` doit décrire l'image lorsque celle-ci apporte une information utile.

Une image purement décorative peut utiliser :

```html
alt=""
```

Il faut également :

- compresser correctement les images ;
- utiliser des dimensions adaptées ;
- renseigner `width` et `height` pour limiter les décalages de mise en page ;
- employer des formats modernes lorsque pertinents ;
- éviter de placer une information essentielle uniquement dans une image.

## 4.9 Fraîcheur du contenu

Tous les contenus ne nécessitent pas une date récente.

Une définition mathématique peut rester correcte pendant des décennies, alors qu'un cours sur une API ou une réglementation doit être vérifié régulièrement.

Mettre artificiellement à jour une date sans modifier le contenu n'améliore pas sa qualité.

## 4.10 Contenu dupliqué

La duplication n'est pas automatiquement une pénalité. Elle devient problématique lorsque :

- de nombreuses URL représentent le même contenu ;
- les signaux sont divisés entre plusieurs URL ;
- l'exploration est gaspillée ;
- l'utilisateur ne sait pas quelle version utiliser.

La canonicalisation, les redirections et une architecture cohérente permettent de consolider les variantes.

---

# 5. Architecture du site et maillage interne

## 5.1 Une architecture compréhensible

Une organisation simple facilite l'usage et l'exploration.

```text
/
├── cours/
│   ├── web/
│   ├── python/
│   └── securite/
├── articles/
└── a-propos/
```

Une page importante ne devrait généralement pas être accessible uniquement via un formulaire de recherche interne.

## 5.2 Maillage interne

Les liens internes :

- permettent aux utilisateurs de naviguer ;
- aident les robots à découvrir les pages ;
- transmettent du contexte ;
- révèlent la structure du site.

Préférer :

```html
<a href="/cours/http">cours sur HTTP</a>
```

à :

```html
<a href="/cours/http">cliquez ici</a>
```

quand le contexte le permet.

## 5.3 Pages orphelines

Une **page orpheline** n'est reliée à aucune autre page pertinente du site.

Elle peut être :

- difficile à découvrir ;
- peu contextualisée ;
- oubliée lors des mises à jour.

Un crawl interne permet de les identifier en comparant :

- URL du sitemap ;
- URL connues par Analytics/Search Console ;
- URL trouvées via les liens internes.

## 5.4 Profondeur

La profondeur correspond au nombre de clics nécessaires depuis une page d'entrée importante.

Il n'existe pas de règle universelle « toutes les pages en trois clics », mais une page stratégique profondément enfouie mérite une justification.

## 5.5 Pagination et chargement progressif

Une interface peut charger progressivement des éléments, mais le moteur doit pouvoir découvrir des URL persistantes correspondant aux contenus à indexer.

Un bouton JavaScript qui ne crée jamais d'URL explorables peut empêcher la découverte complète d'un catalogue.

---

# 6. SEO technique : exploration, indexation et canonicalisation

## 6.1 Codes HTTP

Les codes d'état ont un sens important.

| Code | Signification | Usage SEO courant |
|---:|---|---|
| `200` | Succès | Page valide |
| `301` | Redirection permanente | URL déplacée durablement |
| `302` | Redirection temporaire | Déplacement temporaire |
| `307` | Redirection temporaire conservant la méthode | Cas HTTP approprié |
| `308` | Redirection permanente conservant la méthode | Alternative permanente |
| `404` | Ressource introuvable | URL inexistante |
| `410` | Ressource supprimée | Suppression volontaire |
| `429` | Trop de requêtes | Limitation temporaire |
| `5xx` | Erreur serveur | Problème côté serveur |

Tester :

```bash
curl -I https://example.org/page
```

## 6.2 `robots.txt`

Le fichier se place classiquement à la racine :

```text
https://example.org/robots.txt
```

Exemple :

```text
User-agent: *
Disallow: /admin/
Disallow: /recherche-interne/

Sitemap: https://example.org/sitemap.xml
```

> [!WARNING]
> `robots.txt` contrôle principalement **l'exploration**, pas l'indexation. Une URL bloquée peut malgré tout être connue et éventuellement apparaître sans contenu détaillé si d'autres pages pointent vers elle.

Il ne faut jamais utiliser `robots.txt` pour protéger une information confidentielle. Utiliser une véritable authentification et des contrôles d'accès.

## 6.3 `noindex`

Pour demander qu'une page ne soit pas indexée :

```html
<meta name="robots" content="noindex">
```

ou via HTTP :

```http
X-Robots-Tag: noindex
```

Le robot doit pouvoir explorer la page pour voir la directive. Bloquer simultanément l'URL dans `robots.txt` peut donc empêcher le moteur de lire `noindex`.

## 6.4 `nofollow`

Exemple :

```html
<a href="https://example.net" rel="nofollow">Lien</a>
```

Pour les liens publicitaires ou sponsorisés :

```html
rel="sponsored"
```

Pour certains contenus générés par des utilisateurs :

```html
rel="ugc"
```

Ces attributs servent à qualifier la relation avec la cible, notamment lorsqu'un lien ne doit pas être interprété comme une recommandation éditoriale ordinaire.

## 6.5 Sitemap XML

Exemple minimal :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.org/</loc>
    <lastmod>2026-08-29</lastmod>
  </url>
  <url>
    <loc>https://example.org/cours/seo</loc>
    <lastmod>2026-08-29</lastmod>
  </url>
</urlset>
```

Un sitemap doit surtout contenir les URL **canoniques que l'on souhaite voir indexées**.

Pour Google, un fichier sitemap est limité à :

- **50 000 URL** ;
- **50 Mo non compressés**.

Au-delà, utiliser plusieurs sitemaps et éventuellement un index de sitemaps.

> [!NOTE]
> Un sitemap facilite la découverte et transmet des indications. Il ne garantit pas l'indexation.

## 6.6 URL canonique

Exemple :

```html
<link rel="canonical" href="https://example.org/produits/clavier">
```

La canonicalisation permet de signaler la version préférée parmi des URL très similaires.

Exemple :

```text
/produits/clavier
/produits/clavier?utm_source=newsletter
/produits/clavier?sort=popularite
```

Le `rel="canonical"` est un **signal**, pas une instruction absolue. Le moteur peut choisir une autre URL s'il reçoit des signaux contradictoires.

Bonnes pratiques :

- canonical auto-référente sur la page principale ;
- liens internes vers l'URL canonique ;
- sitemap cohérent avec les canonicals ;
- redirection si une variante ne doit plus exister ;
- ne pas utiliser `robots.txt` comme mécanisme de canonicalisation.

## 6.7 Redirections

Pour un déplacement permanent :

```text
ancienne URL -> 301/308 -> nouvelle URL
```

Éviter les chaînes :

```text
A -> B -> C -> D
```

Préférer :

```text
A -> D
B -> D
C -> D
```

Une redirection doit pointer vers une destination réellement équivalente, pas systématiquement vers la page d'accueil.

## 6.8 Pages 404

Une bonne page 404 :

- renvoie réellement un statut `404` ;
- explique que la ressource est absente ;
- propose une navigation utile ;
- ne se fait pas passer pour une page valide avec un `200`.

Le dernier cas est parfois appelé **soft 404**.

## 6.9 HTTPS

HTTPS est indispensable pour la sécurité moderne du Web.

Après une migration HTTP -> HTTPS :

- rediriger toutes les anciennes URL ;
- utiliser les URL HTTPS dans les liens internes ;
- mettre à jour canonical et sitemap ;
- vérifier les ressources mixtes ;
- conserver les certificats et la configuration TLS à jour.

Voir également [[HTTP]].

## 6.10 Internationalisation et `hreflang`

Exemple :

```html
<link rel="alternate" hreflang="fr" href="https://example.org/fr/guide">
<link rel="alternate" hreflang="en" href="https://example.org/en/guide">
<link rel="alternate" hreflang="x-default" href="https://example.org/guide">
```

Les annotations doivent être cohérentes et réciproques.

`hreflang` ne remplace pas une bonne canonicalisation.

---

# 7. Performance, expérience utilisateur et Core Web Vitals

## 7.1 Performance et SEO

La performance est importante d'abord parce qu'elle affecte l'utilisateur :

- perception de rapidité ;
- capacité à interagir ;
- stabilité de la page ;
- conversion ;
- accessibilité sur mobile et réseaux lents.

Les performances ne remplacent pas la pertinence du contenu.

## 7.2 Core Web Vitals

Les trois métriques principales sont :

| Métrique | Mesure | Bon niveau |
|---|---|---:|
| **LCP** | affichage du principal élément visible | `<= 2,5 s` |
| **INP** | réactivité aux interactions | `<= 200 ms` |
| **CLS** | stabilité visuelle | `<= 0,1` |

Les seuils sont évalués sur l'expérience réelle des utilisateurs, généralement au **75e percentile**.

### LCP — Largest Contentful Paint

Pour améliorer le LCP :

- réduire le délai du serveur ;
- ne pas retarder la ressource principale ;
- optimiser les images héro ;
- éviter les dépendances de rendu inutiles ;
- utiliser cache et CDN lorsque pertinents.

### INP — Interaction to Next Paint

Pour améliorer l'INP :

- réduire les longues tâches JavaScript ;
- diviser le travail ;
- limiter les scripts tiers ;
- éviter les handlers très coûteux ;
- ne pas bloquer inutilement le thread principal.

### CLS — Cumulative Layout Shift

Pour améliorer le CLS :

- réserver l'espace des images et vidéos ;
- éviter d'insérer des blocs au-dessus d'un contenu déjà affiché ;
- maîtriser le chargement des polices ;
- dimensionner les publicités et embeds.

## 7.3 Données de terrain et données de laboratoire

Il faut distinguer :

- **field data** : mesures d'utilisateurs réels ;
- **lab data** : simulation reproductible dans un environnement contrôlé.

Outils typiques :

- Chrome User Experience Report ;
- Search Console ;
- PageSpeed Insights ;
- Lighthouse ;
- outils RUM (*Real User Monitoring*).

## 7.4 Mobile

Google utilise une approche **mobile-first** de l'indexation.

Les contenus et métadonnées importants doivent donc être disponibles sur la version mobile.

Une interface mobile de qualité implique notamment :

- texte lisible ;
- éléments cliquables utilisables ;
- navigation cohérente ;
- contenu principal non amputé ;
- performance acceptable.

---

# 8. SEO JavaScript et applications Web modernes

Voir également [[Javascript]].

## 8.1 Google peut exécuter JavaScript, mais...

Google utilise un moteur de rendu basé sur Chromium et sait traiter de nombreux sites JavaScript.

Cela ne signifie pas que toutes les architectures sont équivalentes.

Une application JavaScript ajoute des risques :

- contenu absent du HTML initial ;
- API indisponible lors du rendu ;
- ressources bloquées ;
- erreurs JavaScript ;
- liens non explorables ;
- métadonnées incohérentes ;
- coût de rendu supérieur.

## 8.2 SSR, SSG et CSR

| Mode | Principe | SEO |
|---|---|---|
| **SSR** | HTML produit côté serveur à la requête | robuste pour le contenu indexable |
| **SSG** | HTML pré-généré | robuste et performant pour contenu stable |
| **CSR** | rendu principalement dans le navigateur | possible, mais plus complexe |

Il n'est pas nécessaire d'abandonner une SPA, mais il faut s'assurer que le contenu essentiel et la navigation sont effectivement accessibles au moteur.

## 8.3 Liens explorables

Préférer :

```html
<a href="/produits/42">Produit 42</a>
```

Éviter comme seul mécanisme de navigation :

```html
<span onclick="openProduct(42)">Produit 42</span>
```

## 8.4 Métadonnées

Dans une application client-side, vérifier pour chaque route indexable :

- `<title>` ;
- meta description ;
- canonical ;
- meta robots ;
- données structurées ;
- statut HTTP ou comportement équivalent côté serveur.

## 8.5 Tester ce que reçoit le moteur

Utiliser notamment :

- inspection d'URL dans Search Console ;
- rendu du HTML côté serveur ;
- navigateur sans JavaScript pour comprendre les dépendances ;
- logs serveur ;
- outils de crawl capables de rendre JavaScript.

---

# 9. Données structurées et apparence dans les résultats

## 9.1 À quoi servent les données structurées ?

Les données structurées décrivent explicitement des entités et leurs propriétés.

Google s'appuie notamment sur **Schema.org** pour certaines fonctionnalités enrichies.

Exemples de types :

- `Article` ;
- `BreadcrumbList` ;
- `Product` ;
- `Organization` ;
- `LocalBusiness` ;
- `VideoObject` ;
- `Recipe`.

## 9.2 JSON-LD

Exemple :

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Comprendre le SEO technique",
  "datePublished": "2026-08-29",
  "dateModified": "2026-08-29",
  "author": {
    "@type": "Person",
    "name": "Michaël Launay"
  }
}
</script>
```

Le balisage doit correspondre au contenu réellement visible et respecter les règles du type concerné.

> [!IMPORTANT]
> Des données structurées valides rendent une page **éligible** à certaines présentations. Elles ne garantissent pas qu'un résultat enrichi sera affiché.

## 9.3 Fil d'Ariane

Exemple visible :

```text
Accueil > Cours > Web > SEO
```

Un fil d'Ariane aide :

- l'utilisateur ;
- la compréhension de la structure ;
- la navigation interne.

Il peut aussi être accompagné d'un balisage `BreadcrumbList` approprié.

## 9.4 Extraits et contrôle de prévisualisation

Des directives permettent de contrôler certains extraits :

```html
<meta name="robots" content="max-snippet:120">
```

ou :

```html
<meta name="robots" content="nosnippet">
```

Pour exclure uniquement un fragment d'un extrait :

```html
<span data-nosnippet>Texte à ne pas utiliser dans le snippet</span>
```

Ces contrôles ont aussi des conséquences sur l'utilisation du contenu dans certaines fonctionnalités de recherche fondées sur l'IA.

---

# 10. SEO off-page, liens et réputation

## 10.1 Backlinks

Un backlink est un lien provenant d'un autre site.

Les bons liens sont généralement obtenus parce que la cible :

- constitue une source ;
- apporte une donnée ;
- fournit un outil ;
- publie une analyse utile ;
- mérite d'être recommandée.

## 10.2 Qualité d'un lien

On peut examiner :

- pertinence thématique ;
- contexte éditorial ;
- qualité de la page source ;
- caractère naturel du lien ;
- trafic ou audience réelle ;
- position du lien dans le document ;
- texte d'ancre.

Un lien n'est pas « bon » uniquement parce qu'un outil tiers lui attribue un score élevé.

## 10.3 DA, DR et métriques similaires

**Domain Authority**, **Domain Rating** et d'autres scores sont des métriques créées par des entreprises tierces.

> [!WARNING]
> Elles ne sont pas des métriques internes de Google et ne constituent pas directement des facteurs de classement Google.

Elles peuvent être utiles pour comparer approximativement des profils de liens, mais ne doivent pas devenir un objectif en soi.

## 10.4 Acquisition de liens saine

Méthodes durables :

- études ou données originales ;
- logiciels ou ressources ouvertes ;
- documentation de référence ;
- partenariats éditoriaux légitimes ;
- relations presse ;
- conférences ;
- contributions expertes ;
- correction de liens cassés lorsqu'une vraie ressource de remplacement existe.

## 10.5 Liens payants

Un lien publicitaire doit être correctement qualifié, par exemple :

```html
<a href="https://sponsor.example" rel="sponsored">Partenaire</a>
```

L'achat ou la vente de liens dans le but de manipuler le classement est contraire aux règles de spam de Google.

## 10.6 Désaveu de liens

Le désaveu n'est pas un outil de maintenance courante à utiliser par peur de chaque backlink étrange.

La priorité est d'éviter de créer ou acheter soi-même des schémas de liens artificiels.

---

# 11. SEO local, international et e-commerce

## 11.1 SEO local

Pour une entreprise locale, les éléments importants comprennent :

- fiche d'établissement complète et à jour ;
- nom, adresse et téléphone cohérents ;
- horaires ;
- catégorie pertinente ;
- avis authentiques ;
- pages locales réellement utiles ;
- données structurées adaptées lorsque pertinentes.

Éviter de créer des centaines de pages quasi identiques pour chaque ville sans valeur locale réelle.

## 11.2 International

Une stratégie internationale doit choisir clairement :

- les langues ;
- les pays ciblés ;
- la structure des URL ;
- les traductions ;
- les annotations `hreflang` ;
- la gestion des contenus régionaux similaires.

Exemple de structures possibles :

```text
example.fr/
example.com/fr/
fr.example.com/
```

Aucune structure ne remplace une traduction et une localisation de qualité.

## 11.3 E-commerce

Pour une boutique :

- catégories explorables ;
- fiches produits uniques ;
- URLs stables ;
- variantes correctement gérées ;
- données `Product` lorsqu'elles sont conformes ;
- prix et disponibilité cohérents ;
- pagination et filtres maîtrisés ;
- images de qualité ;
- Merchant Center lorsque pertinent.

## 11.4 Navigation à facettes

Des filtres peuvent produire énormément d'URL :

```text
?couleur=noir
?taille=42
?couleur=noir&taille=42
?tri=prix
...
```

Il faut déterminer quelles combinaisons :

- répondent réellement à une demande ;
- doivent être explorables ;
- doivent être indexables ;
- doivent avoir une URL canonique propre.

Une stratégie incorrecte peut créer un espace d'URL pratiquement infini.

---

# 12. SEO, IA générative, AI Overviews et AI Mode

## 12.1 Le SEO n'a pas disparu

L'arrivée de réponses génératives transforme l'interface de recherche, mais les fondamentaux restent essentiels.

En 2026, Google indique explicitement que les bonnes pratiques SEO restent applicables aux fonctionnalités génératives telles que :

- **AI Overviews** ;
- **AI Mode**.

Pour être éligible comme source, une page doit notamment être :

- indexée ;
- éligible à l'affichage dans Search avec un extrait ;
- accessible techniquement selon les règles habituelles.

Il n'existe pas de balisage Schema.org spécial obligatoire pour « entrer dans AI Overviews ».

## 12.2 AEO et GEO

Les termes suivants sont utilisés dans l'industrie :

- **AEO** : *Answer Engine Optimization* ;
- **GEO** : *Generative Engine Optimization*.

Ils peuvent être utiles pour nommer une préoccupation métier, mais Google considère que l'optimisation de ses expériences génératives repose sur les mêmes bases que le SEO.

Il faut donc se méfier des promesses de « hack GEO » ou de fichiers secrets supposés garantir une citation.

## 12.3 Comment les fonctionnalités génératives trouvent des sources

Google décrit notamment :

- des mécanismes de **RAG** (*Retrieval-Augmented Generation*) ;
- le **query fan-out**, où le système lance plusieurs recherches connexes pour traiter une question complexe.

Conséquence pratique : une page utile n'a pas besoin de répéter mot pour mot toutes les variantes imaginables d'une requête.

Elle doit plutôt :

- couvrir clairement son sujet ;
- fournir des informations distinctives ;
- être structurée ;
- présenter des faits vérifiables ;
- proposer des médias de qualité lorsque pertinents.

Voir également [[RAG]].

## 12.4 Contenu non générique

Avec les modèles génératifs, reproduire une synthèse déjà disponible partout apporte peu de valeur.

Un contenu plus différenciant peut apporter :

- une expérience réelle ;
- une mesure originale ;
- un corpus ou jeu de données ;
- une enquête ;
- des tests ;
- une analyse experte ;
- un outil ;
- une comparaison reproductible ;
- une documentation de première main.

## 12.5 Contenu généré par IA

Google ne considère pas qu'un texte est automatiquement du spam parce qu'une IA a participé à sa création.

Le problème apparaît lorsque l'automatisation sert principalement à produire à grande échelle du contenu :

- non original ;
- peu utile ;
- destiné à manipuler les résultats.

Une bonne utilisation de l'IA peut inclure :

- assistance à la recherche ;
- structuration ;
- reformulation ;
- extraction ;
- traduction avec contrôle ;
- génération d'ébauches ensuite vérifiées et enrichies.

La responsabilité éditoriale reste humaine ou organisationnelle : les faits doivent être contrôlés.

## 12.6 Contrôler l'utilisation dans les fonctionnalités de Search

Les contrôles d'extrait classiques restent pertinents :

- `nosnippet` ;
- `data-nosnippet` ;
- `max-snippet` ;
- `noindex`.

Google distingue ces contrôles de **Google-Extended**, qui concerne certains usages d'entraînement et de grounding dans d'autres systèmes Google et ne constitue pas le mécanisme de contrôle de Google Search lui-même.

## 12.7 Mesurer la visibilité générative

En 2026, Google a commencé à proposer dans Search Console des rapports spécifiques sur la visibilité dans les fonctionnalités de recherche générative, notamment AI Overviews et AI Mode, en complément des données de performance générales.

Pour mesurer l'impact, observer :

- impressions ;
- clics ;
- conversions ;
- engagement post-clic ;
- requêtes ;
- pages de destination ;
- évolution avant/après changements de SERP.

Ne pas se limiter au trafic brut : une diminution du nombre de clics peut parfois coexister avec une amélioration du taux de conversion ou de la qualité des visites.

## 12.8 Agents IA et Web

Les agents peuvent effectuer des tâches pour leurs utilisateurs :

- comparer des offres ;
- préparer un achat ;
- réserver ;
- collecter des informations ;
- interagir avec des interfaces.

Les bonnes pratiques de base restent :

- contenu accessible ;
- informations fiables et à jour ;
- structure technique claire ;
- données produit/locales cohérentes ;
- interfaces compréhensibles et sécurisées.

---

# 13. Mesure, outils et indicateurs

## 13.1 Google Search Console

Search Console permet notamment de suivre :

- clics ;
- impressions ;
- CTR ;
- position moyenne ;
- requêtes ;
- pages ;
- pays et appareils ;
- état d'indexation ;
- sitemaps ;
- Core Web Vitals ;
- résultats enrichis ;
- actions manuelles et problèmes de sécurité selon les rapports disponibles.

## 13.2 Google Analytics et outils analytics

Un outil analytics mesure ce qui se passe **après l'arrivée sur le site** :

- sessions ;
- événements ;
- conversions ;
- revenus ;
- engagement ;
- parcours utilisateurs.

Search Console et Analytics répondent donc à des questions différentes et leurs chiffres ne doivent pas nécessairement être identiques.

## 13.3 Logs serveur

Les logs permettent de savoir ce qui a réellement été demandé au serveur.

Exemple simplifié :

```text
66.249.x.x - - [29/Aug/2026:10:42:31 +0200] "GET /cours/seo HTTP/2" 200 18452
```

On peut analyser :

- fréquence de crawl ;
- erreurs ;
- redirections ;
- ressources très explorées ;
- URL inutiles ;
- comportement de robots déclarés.

Attention : l'User-Agent seul ne suffit pas toujours à prouver l'identité d'un robot.

## 13.4 Outils de crawl

Exemples :

- Screaming Frog SEO Spider ;
- Sitebulb ;
- crawlers internes ;
- scripts personnalisés.

Ils permettent de détecter :

- liens cassés ;
- redirections ;
- titres absents ou dupliqués ;
- canonicals ;
- profondeur ;
- pages orphelines ;
- directives robots ;
- données structurées.

## 13.5 Outils tiers

Ahrefs, Semrush, Moz et d'autres peuvent fournir :

- estimations de mots-clés ;
- visibilité concurrentielle ;
- backlinks connus ;
- suivi de positions ;
- audits.

Leurs chiffres sont des estimations issues de leurs propres bases. Aucun outil tiers ne connaît exactement les algorithmes internes de Google.

## 13.6 KPI pertinents

Un bon KPI dépend de l'objectif.

### Site éditorial

- impressions ;
- clics ;
- lecteurs récurrents ;
- inscriptions ;
- profondeur de lecture.

### E-commerce

- revenus organiques ;
- transactions ;
- marge ;
- taux de conversion ;
- visibilité des catégories stratégiques.

### SaaS

- demandes de démonstration ;
- essais ;
- inscriptions ;
- activation ;
- revenu attribuable.

> [!IMPORTANT]
> « Être premier sur Google » n'est pas un objectif métier suffisant. Une requête peut générer beaucoup d'impressions et aucune valeur réelle.

## 13.7 CTR et position moyenne

Le CTR varie selon :

- requête ;
- position ;
- type de résultat ;
- marque ;
- présence d'annonces ;
- vidéos, cartes ou résultats enrichis ;
- fonctionnalités génératives.

Il faut donc éviter d'utiliser une courbe universelle « position -> CTR » comme vérité absolue.

---

# 14. Méthodologie d'audit SEO

Un audit doit aboutir à des **actions priorisées**, pas à une liste de centaines d'avertissements sans contexte.

## 14.1 Étape 1 — Comprendre le site

Identifier :

- objectifs métier ;
- audiences ;
- technologies ;
- historique ;
- marchés ;
- pages stratégiques ;
- contraintes légales ou techniques.

## 14.2 Étape 2 — Explorer le site

Relever :

- URL ;
- statuts HTTP ;
- profondeur ;
- liens entrants internes ;
- titles ;
- descriptions ;
- H1 ;
- canonical ;
- indexabilité ;
- tailles de pages ;
- pagination ;
- hreflang ;
- données structurées.

## 14.3 Étape 3 — Vérifier l'indexation

Comparer :

```text
URL théoriques
URL crawlables
URL du sitemap
URL indexables
URL effectivement indexées
URL qui reçoivent des impressions
```

Les écarts sont souvent très instructifs.

## 14.4 Étape 4 — Inspecter la technique

Vérifier :

- `robots.txt` ;
- sitemaps ;
- canonicals ;
- redirections ;
- HTTP/HTTPS ;
- erreurs serveur ;
- rendu JavaScript ;
- performance ;
- mobile ;
- pages 404 ;
- facettes ;
- pagination.

## 14.5 Étape 5 — Auditer le contenu

Pour chaque groupe de pages :

- intention satisfaite ?
- contenu unique ?
- expertise ou expérience démontrée ?
- information à jour ?
- page trop faible ou artificielle ?
- cannibalisation éventuelle ?
- maillage interne suffisant ?

## 14.6 Étape 6 — Étudier les requêtes

Search Console permet d'identifier :

- pages avec fortes impressions et CTR faible ;
- requêtes en progression ;
- pertes soudaines ;
- contenus découvrant des requêtes inattendues ;
- pages proches de la première page ;
- différences par pays ou appareil.

## 14.7 Étape 7 — Étudier les liens et la réputation

Chercher notamment :

- liens naturels importants ;
- mentions ;
- pages historiquement très liées ;
- anciens liens cassés après migration ;
- schémas artificiels éventuellement créés par le site lui-même.

## 14.8 Étape 8 — Prioriser

Une matrice simple :

| Impact | Effort | Priorité |
|---|---|---|
| Fort | Faible | immédiate |
| Fort | Fort | planifier |
| Faible | Faible | opportuniste |
| Faible | Fort | souvent éviter |

Ajouter la notion de **risque** : une migration de 100 000 URL n'a pas le même niveau de risque qu'une correction de title.

## 14.9 Étape 9 — Mesurer après modification

Noter :

- date ;
- pages concernées ;
- hypothèse ;
- métriques attendues ;
- période de comparaison.

Cela évite d'attribuer arbitrairement chaque variation à la dernière modification effectuée.

---

# 15. Migrations et changements d'URL

Les migrations sont parmi les opérations SEO les plus risquées.

Exemples :

- HTTP -> HTTPS ;
- changement de domaine ;
- changement de CMS ;
- nouveau framework ;
- réorganisation de l'arborescence ;
- fusion de sites.

## 15.1 Principe : limiter les changements simultanés

Si possible, ne modifier pas en même temps :

- domaine ;
- design ;
- contenu ;
- URLs ;
- architecture ;
- moteur de rendu.

Plus il y a de variables, plus le diagnostic devient difficile.

## 15.2 Mapping des URL

Préparer un tableau :

```text
ancienne URL -> nouvelle URL
```

Chaque page importante doit avoir une destination logique.

Éviter :

```text
10 000 anciennes pages -> page d'accueil
```

## 15.3 Avant la migration

Sauvegarder :

- crawl complet ;
- titles ;
- canonicals ;
- hreflang ;
- statuts ;
- sitemaps ;
- trafic par URL ;
- requêtes principales ;
- backlinks importants.

## 15.4 Après la migration

Contrôler :

- redirections ;
- absence de boucles ;
- absence de chaînes inutiles ;
- liens internes ;
- canonical ;
- sitemap ;
- indexabilité ;
- logs ;
- Search Console ;
- trafic et conversions.

Conserver les redirections suffisamment longtemps : les anciens liens et favoris peuvent continuer à être utilisés bien après la migration.

---

# 16. Spam, erreurs fréquentes et pratiques à éviter

## 16.1 Bourrage de mots-clés

Mauvais exemple :

```text
Notre agence SEO Lille est la meilleure agence SEO Lille pour votre SEO Lille...
```

Le résultat est médiocre pour le lecteur et peut être considéré comme une tentative de manipulation.

## 16.2 Contenu à grande échelle sans valeur

Créer automatiquement des milliers de pages peu originales dans le but principal de capter des requêtes constitue un risque important.

Cela peut être produit :

- par IA générative ;
- par templates ;
- par traduction automatique ;
- par assemblage de flux ;
- manuellement.

Le moyen de production n'est pas le cœur du problème : c'est l'objectif manipulateur et l'absence de valeur.

## 16.3 Cloaking

Le **cloaking** consiste à présenter volontairement des contenus substantiellement différents au moteur et à l'utilisateur afin de manipuler les résultats.

Il ne faut pas le confondre avec des adaptations légitimes d'interface ou d'accessibilité.

## 16.4 Pages satellites (*doorway pages*)

Exemple :

```text
/plombier-lille
/plombier-roubaix
/plombier-tourcoing
...
```

si toutes les pages sont quasi identiques, sans information locale spécifique, uniquement créées pour rediriger vers le même service.

## 16.5 Réseaux de liens et achat de liens

Pratiques à risque :

- fermes de liens ;
- PBN destinés à manipuler le classement ;
- échanges massifs ;
- ancres artificiellement optimisées ;
- articles sponsorisés non signalés dans l'unique but de transmettre des signaux de classement.

## 16.6 Abus de réputation de site

Publier des contenus tiers principalement pour exploiter la réputation d'un domaine hôte, sans supervision ou valeur éditoriale suffisante, peut entrer dans les politiques d'abus de réputation.

Le fait qu'un contenu soit hébergé sur un domaine puissant ne le rend pas automatiquement légitime.

## 16.7 Domaines expirés

Racheter un domaine expiré peut être légitime.

L'abus consiste à exploiter son historique ou sa réputation principalement pour manipuler le classement avec du contenu sans rapport ou peu utile.

## 16.8 « Autorité de domaine » comme objectif

Erreur classique :

```text
Objectif : passer le DA de 42 à 50
```

Mieux :

```text
Objectif : obtenir des citations éditoriales de sources pertinentes et augmenter les conversions organiques sur nos pages stratégiques.
```

## 16.9 `robots.txt` pour cacher un secret

Incorrect :

```text
Disallow: /admin-secret/
```

Le chemin est visible publiquement dans le fichier et n'est pas protégé.

Pour un secret : authentification, autorisation et contrôle serveur.

## 16.10 Obsession du score 100/100

Un score Lighthouse ou d'outil SEO parfait n'est pas un objectif métier.

Corriger en priorité ce qui :

- bloque l'exploration ;
- empêche l'indexation ;
- dégrade fortement l'expérience ;
- empêche les utilisateurs de trouver une information ;
- affecte une page importante.

---

# 17. Cas pratique : construire une stratégie SEO

Imaginons un nouveau site proposant des formations Python.

## 17.1 Objectif

Objectif métier :

> obtenir des inscriptions qualifiées à des formations Python avancées.

## 17.2 Identifier les publics

Exemples :

- développeurs débutants ;
- développeurs confirmés ;
- responsables techniques ;
- entreprises cherchant une formation interne.

## 17.3 Cartographier les besoins

```text
Découverte
  -> qu'est-ce qu'un décorateur Python ?

Apprentissage
  -> tutoriel décorateurs Python

Comparaison
  -> formation Python avancée en ligne

Décision
  -> prix formation Python entreprise
```

## 17.4 Architecture

```text
/
├── formations/
│   ├── python-debutant/
│   ├── python-avance/
│   └── python-entreprise/
├── cours/
│   ├── decorateurs-python/
│   ├── async-python/
│   └── tests-python/
└── a-propos/
```

Les articles pédagogiques peuvent naturellement lier les formations lorsqu'elles sont pertinentes.

## 17.5 Production de contenu

Plutôt que 100 textes génériques, produire :

- exemples exécutables ;
- benchmarks ;
- exercices ;
- dépôts Git ;
- explications issues d'une expérience d'enseignement ;
- corrections détaillées ;
- comparaison entre approches.

## 17.6 Technique

Avant lancement :

- HTTPS ;
- sitemap ;
- robots.txt ;
- canonical ;
- pages d'erreur ;
- responsive ;
- Core Web Vitals ;
- données structurées pertinentes ;
- Search Console ;
- analytics respectueux du cadre légal applicable.

## 17.7 Acquisition de réputation

Exemples :

- publier un projet libre ;
- intervenir dans une conférence Python ;
- contribuer à une documentation ;
- produire une étude originale ;
- être cité naturellement par des communautés techniques.

## 17.8 Mesure

Suivre :

- requêtes non-brand ;
- inscriptions ;
- conversions par landing page ;
- pages générant des prospects ;
- évolution des sujets stratégiques ;
- erreurs d'indexation ;
- visibilité dans les résultats classiques et génératifs.

---

# 18. Checklist opérationnelle

## 18.1 Avant publication d'une page

- [ ] L'intention utilisateur est claire.
- [ ] Le contenu apporte une valeur propre.
- [ ] Le `<title>` décrit précisément la page.
- [ ] Le H1 et les sous-titres structurent le contenu.
- [ ] La meta description est utile.
- [ ] L'URL est stable et lisible.
- [ ] Les images importantes ont un `alt` adapté.
- [ ] Les liens internes importants sont présents.
- [ ] La page répond en `200`.
- [ ] Elle n'est pas accidentellement `noindex`.
- [ ] La canonical est cohérente.
- [ ] Les données structurées correspondent au contenu visible.
- [ ] La page est utilisable sur mobile.
- [ ] Les performances sont acceptables.

## 18.2 Pour un site entier

- [ ] HTTPS est généralisé.
- [ ] `robots.txt` est valide.
- [ ] Les sitemaps ne contiennent que les URL pertinentes.
- [ ] Les redirections sont maîtrisées.
- [ ] Les pages 404 renvoient un vrai `404`.
- [ ] Les facettes ne créent pas une infinité d'URL inutiles.
- [ ] Le maillage interne reflète les priorités du site.
- [ ] Les pages stratégiques ne sont pas orphelines.
- [ ] Search Console est configurée.
- [ ] Les conversions sont mesurables.
- [ ] Les logs et erreurs serveur sont surveillés.
- [ ] Les changements SEO importants sont documentés.

## 18.3 Pour une mise à jour de contenu

- [ ] Les faits sont toujours exacts.
- [ ] Les sources sont toujours valides.
- [ ] Les captures et exemples correspondent aux versions actuelles.
- [ ] Les liens cassés sont corrigés.
- [ ] La date de modification n'est changée que si une vraie modification a été faite.
- [ ] Les anciennes sections inutiles sont supprimées plutôt que conservées artificiellement.

---

# 19. Conclusion

Le SEO moderne est une discipline d'ingénierie, de contenu et de stratégie.

Les principes les plus durables sont :

1. **rendre le contenu accessible** aux moteurs et aux utilisateurs ;
2. **répondre précisément à une intention** ;
3. **apporter une valeur que les pages génériques n'apportent pas** ;
4. **maintenir une architecture technique cohérente** ;
5. **développer une réputation réelle plutôt que fabriquer des signaux** ;
6. **mesurer l'impact métier**, pas uniquement des positions ;
7. **adapter la stratégie aux nouvelles interfaces de recherche sans abandonner les fondamentaux**.

L'IA générative ne supprime donc pas le SEO. Elle renforce au contraire l'importance de contenus techniquement accessibles, fiables, structurés et suffisamment originaux pour mériter d'être utilisés comme source.

---

# 20. Sources et références

## Références officielles principales

- Google Search Central — SEO Starter Guide : <https://developers.google.com/search/docs/fundamentals/seo-starter-guide>
- Google Search Essentials : <https://developers.google.com/search/docs/essentials>
- Fonctionnement de Google Search : <https://developers.google.com/search/docs/fundamentals/how-search-works>
- Contenu utile, fiable et *people-first* : <https://developers.google.com/search/docs/fundamentals/creating-helpful-content>
- Exploration et indexation : <https://developers.google.com/search/docs/crawling-indexing>
- Canonicalisation : <https://developers.google.com/search/docs/crawling-indexing/canonicalization>
- Sitemaps : <https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap>
- SEO JavaScript : <https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics>
- Core Web Vitals : <https://developers.google.com/search/docs/appearance/core-web-vitals>
- Données structurées : <https://developers.google.com/search/docs/appearance/structured-data/sd-policies>
- Règles antispam : <https://developers.google.com/search/docs/essentials/spam-policies>
- Contenu généré avec l'IA : <https://developers.google.com/search/docs/fundamentals/using-gen-ai-content>
- Fonctionnalités d'IA et site Web : <https://developers.google.com/search/docs/appearance/ai-features>
- Optimisation pour les fonctionnalités d'IA générative : <https://developers.google.com/search/docs/fundamentals/ai-optimization-guide>
- Search Console : <https://search.google.com/search-console/about>
- Schema.org : <https://schema.org/>

## Notes de vigilance

Les documents officiels des moteurs doivent être privilégiés pour les règles techniques et les politiques. Les outils tiers sont utiles pour l'analyse et la comparaison, mais leurs métriques et leurs estimations ne constituent pas des métriques internes des moteurs de recherche.
