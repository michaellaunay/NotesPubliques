---
schema_version: 1
uid: "01M02EX5C743KQCWZ60WYSHSWN"
titre: "Solid (Social Linked Data)"
aliases:
  - "SOLID"
  - "Solid Project"
  - "Social Linked Data"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - web-semantique
  - decentralisation
  - solid
  - rdf
  - vie-privee
resume: "Cours complet sur Solid : Pods, WebID, RDF/Linked Data, protocole HTTP, Solid-OIDC, WAC/ACP, Community Solid Server, développement d'applications, notifications, sécurité et interopérabilité."
niveau: intermediaire
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-06-25
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Solid (Social Linked Data)

> [!abstract] Objectif
> Comprendre le projet Solid (Social Linked Data) : séparer les données des applications dans des pods personnels, décrire ces données en RDF et Linked Data, s'authentifier avec Solid-OIDC, contrôler l'accès avec WAC et ACP, puis développer une application cliente et déployer un serveur communautaire (Community Solid Server) en connaissant l'état réel de l'écosystème en 2026.

Voir aussi : [[HTTP]], [[OAuth OpenID]], [[SPARQL et RDF]], [[Javascript]], [[Identité numérique européenne]].

> [!important]
> Ce cours traite du projet **Solid** initié par Tim Berners-Lee autour du Web décentralisé et des données liées. Il ne faut pas le confondre avec les cinq principes de conception orientée objet **SOLID**, traités dans [[Principes SOLID en COO]].

## Sommaire

1. [[#1. Comprendre Solid]]
2. [[#2. État du projet et des spécifications en 2026]]
3. [[#3. Architecture générale]]
4. [[#4. RDF, Linked Data et vocabulaires]]
5. [[#5. Le protocole Solid et les ressources HTTP]]
6. [[#6. WebID et identité]]
7. [[#7. Authentification avec Solid-OIDC]]
8. [[#8. Autorisation : WAC et ACP]]
9. [[#9. Installer un serveur Solid moderne]]
10. [[#10. Manipuler un Pod avec HTTP]]
11. [[#11. Développer une application Solid en JavaScript]]
12. [[#12. Structurer les données d'une application]]
13. [[#13. Notifications et synchronisation]]
14. [[#14. Sécurité, vie privée et modèle de menace]]
15. [[#15. Interopérabilité et portabilité]]
16. [[#16. Déploiement et exploitation]]
17. [[#17. Limites, maturité et choix d'architecture]]
18. [[#18. Cas d'usage]]
19. [[#19. Travaux pratiques]]
20. [[#20. Projet final]]
21. [[#21. Checklist]]
22. [[#22. Glossaire]]
23. [[#23. Sources et références]]

---

# 1. Comprendre Solid

## 1.1 Qu'est-ce que Solid ?

**Solid** est un projet de Web décentralisé visant à séparer :

- les **applications** ;
- les **données** ;
- l'**identité** ;
- et les **règles d'accès**.

Dans une application Web classique, le fournisseur de l'application contrôle généralement en même temps :

```text
Application
    |
    +-- interface
    +-- logique métier
    +-- base de données
    +-- compte utilisateur
    +-- autorisations
```

L'utilisateur peut éventuellement exporter ses données, mais elles restent généralement attachées au fournisseur.

L'idée de Solid est différente :

```text
                  +-------------------+
                  |   Application A   |
                  +---------+---------+
                            |
                            | HTTP + autorisation
                            v
+-------------+      +------+-------+      +-------------+
| Application | ---> |     Pod      | <--- | Application |
|      B      |      | de l'utilisateur    |      C      |
+-------------+      +------+-------+      +-------------+
                            |
                            v
                     données contrôlées
                     indépendamment
                     des applications
```

Une application Solid doit pouvoir travailler avec des données hébergées ailleurs que chez son éditeur.

## 1.2 Que signifie « Solid » ?

Le nom est historiquement associé à **Social Linked Data**.

Dans la documentation moderne, le projet est généralement écrit **Solid** plutôt que comme un acronyme en capitales.

L'objectif n'est pas de créer un « nouvel Internet », mais d'étendre les principes du Web avec :

- des identifiants Web ;
- des espaces de stockage contrôlés indépendamment des applications ;
- des données liées ;
- des mécanismes d'authentification ;
- des mécanismes d'autorisation interopérables.

## 1.3 Les objectifs

Solid cherche notamment à améliorer :

### Contrôle des données

L'utilisateur choisit où ses données sont hébergées.

### Portabilité

Changer d'application ne devrait pas obliger à abandonner ses données.

### Interopérabilité

Plusieurs applications peuvent, en théorie, comprendre et manipuler les mêmes données grâce à des vocabulaires partagés.

### Découplage

L'application n'est pas nécessairement le propriétaire de la base de données.

### Identité décentralisée

L'identifiant de l'utilisateur peut être indépendant d'une application particulière.

### Vie privée

Le modèle facilite une architecture dans laquelle les données ne sont pas automatiquement centralisées chez chaque fournisseur d'application.

> [!warning]
> Solid ne garantit pas à lui seul la vie privée. Un mauvais serveur, une application trop permissive, une fuite de jeton, une mauvaise ACL ou un vocabulaire contenant trop d'informations peuvent toujours exposer des données.

## 1.4 Solid n'est pas une blockchain

Solid ne repose pas sur une blockchain.

Les propriétés essentielles sont fondées sur des technologies Web :

- HTTP ;
- URI/IRI ;
- RDF ;
- Linked Data ;
- OpenID Connect ;
- OAuth ;
- contrôles d'accès ;
- Web Linking.

Il n'y a pas besoin :

- de consensus distribué ;
- de preuve de travail ;
- de jeton crypto ;
- de registre global immuable.

## 1.5 Solid n'est pas un simple cloud personnel

Un Pod ressemble en partie à un espace de stockage Web personnel, mais le modèle va plus loin.

Les ressources peuvent être :

- adressables par URI ;
- décrites en RDF ;
- liées entre elles ;
- partagées avec des agents ou applications ;
- découvertes à partir d'une identité Web.

Le Pod est donc un élément d'une architecture **Linked Data**, et pas uniquement un disque distant.

---

# 2. État du projet et des spécifications en 2026

## 2.1 Un point essentiel : Solid n'est pas un standard W3C final

Au 29 août 2026, les principaux documents Solid sont publiés par le **Solid Community Group du W3C**.

Ils ne doivent pas être présentés comme des **W3C Recommendations** finalisées.

Le **Solid Protocol** est notamment :

```text
Version : 0.11.0
Statut  : Community Group Draft
```

Une spécification de Community Group peut être très utile et implémentée, mais son statut est différent de celui d'une recommandation normative du W3C.

## 2.2 Principaux documents

Parmi les documents importants :

| Élément | Rôle |
|---|---|
| Solid Protocol | comportement des serveurs et clients Solid |
| Solid WebID Profile | profil associé à un WebID |
| Solid-OIDC | authentification basée sur OpenID Connect |
| Web Access Control (WAC) | modèle d'ACL RDF |
| Access Control Policy (ACP) | modèle d'autorisation plus riche |
| Solid Notifications Protocol | notifications de modification |
| Solid Application Interoperability | travail sur l'interopérabilité applicative |

## 2.3 Versions et maturité

Il faut toujours distinguer :

```text
standard Web stable
        +
spécification Solid communautaire
        +
implémentation serveur
        +
bibliothèque cliente
```

Par exemple :

- HTTP est un standard IETF mature ;
- RDF est un standard W3C mature ;
- OpenID Connect est largement déployé ;
- Solid Protocol reste un Community Group Draft ;
- un serveur Solid peut ne supporter qu'un sous-ensemble de fonctionnalités optionnelles.

## 2.4 Ne pas confondre Solid Project et SolidJS

Il existe également un framework JavaScript nommé **SolidJS**.

```text
Solid Project    -> données décentralisées / Pods / WebID
SolidJS          -> framework d'interface JavaScript
```

Ce sont deux projets différents.

---

# 3. Architecture générale

## 3.1 Les acteurs

Une architecture Solid fait intervenir plusieurs rôles.

### Agent

Une personne, une organisation, un programme ou une entité qui agit sur les ressources.

### WebID

Identifiant Web de l'agent.

Exemple :

```text
https://alice.example/profile/card#me
```

### Pod

Espace de stockage contenant les ressources de l'utilisateur.

### Resource Server

Serveur HTTP qui héberge les ressources Solid.

### Identity Provider / OpenID Provider

Service qui authentifie l'utilisateur et participe au mécanisme Solid-OIDC.

### Client

Application qui accède au Pod.

## 3.2 Découplage identité, stockage et application

Conceptuellement :

```text
+----------------+
|     WebID      |
| identité Web   |
+--------+-------+
         |
         | référence
         v
+----------------+          +----------------+
| Identity       |          |      Pod       |
| Provider       |          | Resource Server|
+-------+--------+          +-------+--------+
        ^                           ^
        | OIDC                      | HTTP
        |                           |
        +------------+--------------+
                     |
              +------+------+
              | Application |
              +-------------+
```

Ces composants peuvent être fournis par le même opérateur, mais le modèle ne l'exige pas conceptuellement.

## 3.3 Ressources et conteneurs

Solid suit un modèle de ressources Web.

Un Pod peut ressembler à :

```text
https://pod.example/alice/
├── profile/
│   └── card
├── private/
│   ├── notes/
│   └── finance/
├── public/
│   └── about
└── contacts/
    ├── bob
    └── claire
```

Un chemin se terminant par `/` représente un **conteneur**.

Exemple :

```text
https://pod.example/alice/contacts/
```

Une ressource :

```text
https://pod.example/alice/contacts/bob
```

## 3.4 Linked Data Platform

Solid reprend de nombreux principes de **Linked Data Platform (LDP)**.

Un conteneur peut décrire ses membres via RDF.

Conceptuellement :

```turtle
@prefix ldp: <http://www.w3.org/ns/ldp#>.

<> a ldp:Container ;
   ldp:contains <bob>, <claire> .
```

> [!important]
> Il ne faut pas modifier manuellement les triples de containment d'un conteneur comme s'il s'agissait d'une ressource RDF ordinaire. Leur gestion revient au serveur.

---

# 4. RDF, Linked Data et vocabulaires

Voir aussi les cours consacrés au Web sémantique si présents dans les notes.

## 4.1 Pourquoi RDF ?

Solid ne cherche pas uniquement à stocker des fichiers.

Pour les données structurées, RDF permet d'exprimer des relations explicites :

```text
sujet -- prédicat --> objet
```

Exemple :

```text
Alice -- connaît --> Bob
```

En RDF :

```turtle
@prefix foaf: <http://xmlns.com/foaf/0.1/>.

<#me> foaf:knows <https://bob.example/profile/card#me> .
```

## 4.2 URI comme identifiants

Une valeur RDF peut référencer une autre ressource du Web.

Exemple :

```turtle
<#me>
    a foaf:Person ;
    foaf:name "Alice" ;
    foaf:knows <https://bob.example/profile/card#me> .
```

Le lien vers Bob n'est pas une simple chaîne opaque : c'est une IRI.

## 4.3 Turtle

Turtle est particulièrement pratique pour lire et écrire RDF à la main.

Exemple :

```turtle
@prefix schema: <https://schema.org/>.
@prefix foaf: <http://xmlns.com/foaf/0.1/>.

<#me>
    a schema:Person, foaf:Person ;
    schema:name "Alice Dupont" ;
    schema:email <mailto:alice@example.org> .
```

## 4.4 JSON-LD

La même idée peut être représentée en JSON-LD :

```json
{
  "@context": {
    "schema": "https://schema.org/"
  },
  "@id": "#me",
  "@type": "schema:Person",
  "schema:name": "Alice Dupont"
}
```

Le Solid Protocol prévoit notamment des représentations RDF en :

```text
text/turtle
application/ld+json
```

## 4.5 Vocabulaire

Un vocabulaire définit des termes réutilisables.

Exemples :

- RDF ;
- RDFS ;
- OWL ;
- FOAF ;
- Schema.org ;
- vCard ;
- Dublin Core ;
- ActivityStreams.

Un bon modèle évite d'inventer :

```text
https://mon-app.example/vocabulaire/name
```

si un terme interopérable adapté existe déjà.

## 4.6 Ne pas surutiliser les vocabulaires

L'interopérabilité ne consiste pas à empiler un maximum d'ontologies.

Il faut choisir :

- des termes stables ;
- compris par l'écosystème ;
- adaptés au besoin métier.

## 4.7 SPARQL : rôle réel dans Solid

SPARQL est le langage standard d'interrogation de graphes RDF.

Exemple :

```sparql
PREFIX schema: <https://schema.org/>

SELECT ?name
WHERE {
  ?person a schema:Person ;
          schema:name ?name .
}
```

Cependant, il faut corriger une simplification fréquente :

> [!important]
> **Un Pod Solid n'est pas nécessairement un endpoint SPARQL.**

L'API fondamentale de Solid est orientée **ressources HTTP**.

On récupère des documents :

```http
GET /contacts/bob
```

puis on traite leur RDF.

Certains serveurs, backends ou outils peuvent exposer SPARQL, mais une application portable ne doit pas supposer qu'un Pod fournit automatiquement :

```text
/sparql
```

## 4.8 N3 Patch

Pour modifier une ressource RDF, le Solid Protocol impose le support des **N3 Patches**.

Exemple :

```n3
@prefix solid: <http://www.w3.org/ns/solid/terms#>.
@prefix schema: <https://schema.org/>.

_:patch
    a solid:InsertDeletePatch ;
    solid:where {
        ?person schema:name "Alice" .
    } ;
    solid:deletes {
        ?person schema:name "Alice" .
    } ;
    solid:inserts {
        ?person schema:name "Alice Dupont" .
    } .
```

Requête :

```http
PATCH /profile/card HTTP/1.1
Content-Type: text/n3
```

N3 Patch reprend des motifs de triples proches de SPARQL, mais **N3 Patch et SPARQL Update ne sont pas la même interface**.

---

# 5. Le protocole Solid et les ressources HTTP

## 5.1 HTTP reste au centre

Une application Solid utilise les méthodes HTTP classiques :

| Méthode | Usage |
|---|---|
| `GET` | lire |
| `HEAD` | métadonnées sans corps |
| `OPTIONS` | découvrir certaines capacités |
| `POST` | créer dans un conteneur |
| `PUT` | créer/remplacer une ressource |
| `PATCH` | modifier une ressource |
| `DELETE` | supprimer |

## 5.2 Lire une ressource

```bash
curl \
  -H 'Accept: text/turtle' \
  https://pod.example/alice/public/profile
```

Réponse possible :

```http
HTTP/1.1 200 OK
Content-Type: text/turtle
ETag: "abc123"
```

## 5.3 Créer avec PUT

```bash
curl \
  -X PUT \
  -H 'Content-Type: text/turtle' \
  --data-binary '@note.ttl' \
  https://pod.example/alice/notes/note-001
```

## 5.4 Créer dans un conteneur avec POST

```bash
curl \
  -X POST \
  -H 'Content-Type: text/turtle' \
  -H 'Slug: note-001' \
  --data-binary '@note.ttl' \
  https://pod.example/alice/notes/
```

Le serveur reste libre de déterminer l'URI finale.

Il peut répondre par exemple :

```http
Location: https://pod.example/alice/notes/note-001
```

## 5.5 Mise à jour conditionnelle

Les mises à jour concurrentes sont un problème réel.

Client A :

```text
GET resource -> version 1
```

Client B :

```text
GET resource -> version 1
PUT resource -> version 2
```

Puis A :

```text
PUT copie de version 1
```

A risque d'écraser les changements de B.

Les mécanismes HTTP comme :

```text
ETag
If-Match
If-None-Match
```

permettent d'éviter une partie de ces conflits.

Exemple :

```http
If-Match: "abc123"
```

## 5.6 Découvrir les capacités

Une réponse Solid peut contenir différents `Link` headers.

Exemple conceptuel :

```http
Link: <http://www.w3.org/ns/ldp#Resource>; rel="type"
```

L'application doit apprendre à lire les métadonnées HTTP plutôt que coder des hypothèses sur le serveur.

## 5.7 Conteneurs

Un chemin finissant par `/` désigne un conteneur.

```text
/notes/       -> conteneur
/notes/123    -> ressource
```

Un serveur Solid doit maintenir la cohérence de la hiérarchie.

## 5.8 Ressources RDF et ressources binaires

Un Pod peut contenir :

### RDF

```text
profile
contacts
préférences
métadonnées
```

### Non-RDF

```text
image.jpg
archive.zip
document.pdf
model.safetensors
```

Solid n'oblige pas tout contenu à être converti en RDF.

Une bonne architecture peut stocker :

```text
/document.pdf
/document-metadata
```

avec les métadonnées en RDF et le contenu en binaire.

---

# 6. WebID et identité

## 6.1 Définition

Un **WebID** est une URI HTTP(S) identifiant un agent.

Exemple :

```text
https://alice.example/profile/card#me
```

Cette URI peut être déréférencée pour obtenir un document RDF.

## 6.2 Fragment `#me`

Dans :

```text
https://alice.example/profile/card#me
```

la ressource HTTP est :

```text
https://alice.example/profile/card
```

et le fragment RDF est :

```text
#me
```

Le document peut contenir :

```turtle
@prefix foaf: <http://xmlns.com/foaf/0.1/>.
@prefix solid: <http://www.w3.org/ns/solid/terms#>.

<#me>
    a foaf:Person ;
    foaf:name "Alice" ;
    solid:oidcIssuer <https://id.example/> .
```

## 6.3 WebID n'est pas un mot de passe

Un WebID est un **identifiant**, pas une preuve d'identité.

Connaître :

```text
https://alice.example/profile/card#me
```

ne permet pas de se faire passer pour Alice.

## 6.4 WebID et profil public

Le document de profil peut contenir des informations utiles à la découverte.

Il ne faut cependant pas publier sans réflexion :

- date de naissance ;
- adresse personnelle ;
- numéro de téléphone ;
- relations privées ;
- identifiants internes ;
- secrets.

## 6.5 Stabilité des URI

Une identité Web gagne à avoir une URI durable.

Changer de domaine ou de structure d'URI peut casser de nombreux liens.

Principe :

```text
Cool URIs don't change
```

---

# 7. Authentification avec Solid-OIDC

Voir aussi [[OAuth OpenID]].

## 7.1 Authentification et autorisation sont différentes

### Authentification

> Qui es-tu ?

### Autorisation

> As-tu le droit de lire ou modifier cette ressource ?

Solid sépare ces deux problèmes.

## 7.2 Solid-OIDC

Solid-OIDC étend les mécanismes OpenID Connect/OAuth pour l'environnement Solid.

Il utilise notamment :

- Authorization Code Flow ;
- PKCE ;
- WebID ;
- DPoP ;
- découverte de l'issuer.

## 7.3 Flux simplifié

```text
Application
    |
    | 1. login
    v
Identity Provider
    |
    | 2. authentification utilisateur
    |
    | 3. redirection
    v
Application
    |
    | 4. requête authentifiée
    v
Resource Server
```

## 7.4 DPoP

DPoP permet de lier un jeton à une clé cryptographique possédée par le client.

Cela réduit le risque qu'un jeton volé puisse être réutilisé directement comme un simple bearer token.

Conceptuellement :

```text
token
  +
preuve signée
  +
clé du client
```

## 7.5 Ne jamais fabriquer Solid-OIDC à la main sans raison

OAuth/OIDC contient de nombreux détails de sécurité :

- PKCE ;
- state ;
- nonce ;
- validation de l'issuer ;
- validation de l'audience ;
- rotation ;
- expiration ;
- preuve DPoP.

En application, utiliser une bibliothèque maintenue est préférable.

---

# 8. Autorisation : WAC et ACP

## 8.1 Deux modèles

L'écosystème Solid a notamment travaillé sur :

- **WAC** — Web Access Control ;
- **ACP** — Access Control Policy.

Ils ne doivent pas être mélangés comme s'il s'agissait du même format.

## 8.2 WAC

WAC repose sur des ACL décrites en RDF.

Exemple conceptuel :

```turtle
@prefix acl: <http://www.w3.org/ns/auth/acl#>.

<#alice>
    a acl:Authorization ;
    acl:agent <https://alice.example/profile/card#me> ;
    acl:accessTo <./document> ;
    acl:mode acl:Read, acl:Write .
```

## 8.3 Modes WAC

On rencontre notamment :

```text
acl:Read
acl:Write
acl:Append
acl:Control
```

## 8.4 ACL d'un conteneur

L'accès à un conteneur peut définir des règles pour :

- le conteneur lui-même ;
- ses membres ;
- les descendants selon les mécanismes applicables.

Il faut tester le comportement réel du serveur utilisé.

## 8.5 ACP

ACP propose un modèle plus riche autour de :

- Access Control Resources ;
- Access Controls ;
- Policies ;
- Matchers ;
- allow ;
- deny.

Conceptuellement :

```text
resource
   |
   v
Access Control
   |
   v
Policy
   |
   +-- allow Read
   +-- matcher agent = Alice
   +-- matcher client = Application X
```

## 8.6 Pourquoi ACP peut être utile

WAC exprime efficacement de nombreuses ACL classiques.

ACP cherche notamment à permettre des politiques plus composables et contextuelles.

Exemple conceptuel :

```text
autoriser lecture
SI
    agent = Alice
ET
    client = Application Notes
```

## 8.7 Ne pas coder le modèle d'accès en dur

Une application portable doit détecter les mécanismes supportés par le serveur ou utiliser une couche cliente adaptée.

Éviter :

```javascript
const aclUrl = resourceUrl + ".acl";
```

comme hypothèse universelle.

## 8.8 Principe du moindre privilège

Une application de notes n'a généralement pas besoin de :

```text
Control sur tout le Pod
```

Elle devrait demander uniquement les accès nécessaires.

Exemple :

```text
/notes/
    Read
    Write
```

plutôt que :

```text
/
    Read
    Write
    Control
```

---

# 9. Installer un serveur Solid moderne

## 9.1 Ancienne recommandation à abandonner

L'ancien cours proposait :

```bash
npm install -g solid-server
```

Cette commande correspond à l'ancien **Node Solid Server** et ne doit plus être la recommandation par défaut pour démarrer un nouveau laboratoire.

En 2026, un choix de référence pour apprendre et tester est le :

**Community Solid Server (CSS)**.

## 9.2 Version de référence

Au 29 août 2026 :

```text
Community Solid Server 7.2.0
```

La version 7.2.0 a été publiée le 22 juillet 2026.

## 9.3 Préparer Node.js

Le serveur demande Node.js 18 ou supérieur.

Pour un nouvel environnement, préférer une version Node.js LTS maintenue.

Vérifier :

```bash
node --version
npm --version
```

## 9.4 Lancer rapidement CSS

```bash
npx @solid/community-server
```

Puis :

```text
http://localhost:3000/
```

Le serveur par défaut peut utiliser un stockage mémoire.

> [!warning]
> Un stockage mémoire est adapté aux TP, pas à la conservation de données.

## 9.5 Stockage persistant sur fichiers

```bash
npx @solid/community-server \
  -c @css:config/file.json \
  -f .data
```

Les données sont alors conservées dans :

```text
.data/
```

## 9.6 Installer comme dépendance de projet

```bash
mkdir solid-lab
cd solid-lab
npm init -y
npm install @solid/community-server
```

Puis utiliser les scripts du projet.

Cela évite de dépendre d'une installation npm globale.

## 9.7 Docker

Une installation conteneurisée peut être utile pour :

- isoler le serveur ;
- reproduire les environnements ;
- automatiser les tests.

Conceptuellement :

```bash
docker run --rm \
  -p 3000:3000 \
  -v "$PWD/data:/data" \
  solidproject/community-server
```

> [!note]
> Les options exactes dépendent de la configuration de l'image et de la version utilisée. Toujours vérifier la documentation de la version déployée.

## 9.8 Configuration CSS

Le Community Solid Server est modulaire.

Il permet de modifier notamment :

- stockage ;
- authentification ;
- autorisation ;
- identité ;
- notifications ;
- logging ;
- routing.

Il utilise **Components.js** pour la composition de ses modules.

## 9.9 Ne jamais utiliser une configuration « unsafe » en production

Certains tutoriels montrent comment désactiver l'autorisation pour comprendre le serveur.

Cela est acceptable uniquement dans un laboratoire isolé.

```text
Internet
   |
   X  configuration sans autorisation
```

---

# 10. Manipuler un Pod avec HTTP

## 10.1 Commencer par HTTP brut

Avant une bibliothèque cliente, il est utile de comprendre le protocole.

## 10.2 OPTIONS

```bash
curl -i -X OPTIONS http://localhost:3000/
```

Observer :

- `Allow` ;
- `Accept-Patch` ;
- `Accept-Post` ;
- `Link`.

## 10.3 Créer une ressource RDF

Créer `note.ttl` :

```turtle
@prefix schema: <https://schema.org/>.

<#note>
    a schema:CreativeWork ;
    schema:name "Première note Solid" ;
    schema:text "Bonjour depuis mon Pod" .
```

Puis :

```bash
curl -i \
  -X PUT \
  -H 'Content-Type: text/turtle' \
  --data-binary '@note.ttl' \
  http://localhost:3000/notes/note-001
```

## 10.4 Lire

```bash
curl -i \
  -H 'Accept: text/turtle' \
  http://localhost:3000/notes/note-001
```

## 10.5 Supprimer

```bash
curl -i \
  -X DELETE \
  http://localhost:3000/notes/note-001
```

## 10.6 Authentification

Les exemples précédents sont simples lorsqu'une ressource est publique ou dans une configuration de laboratoire.

Pour une ressource protégée, il ne suffit pas d'ajouter arbitrairement :

```http
Authorization: Bearer ...
```

L'application doit suivre le mécanisme d'authentification du serveur, notamment Solid-OIDC lorsque celui-ci est utilisé.

---

# 11. Développer une application Solid en JavaScript

## 11.1 Bibliothèques Inrupt

L'écosystème Inrupt fournit plusieurs bibliothèques JavaScript utiles :

```text
@inrupt/solid-client
@inrupt/solid-client-authn-browser
@inrupt/solid-client-authn-node
@inrupt/solid-client-access-grants
@inrupt/solid-client-notifications
```

Ces bibliothèques constituent un choix pratique, mais **Solid n'est pas limité à Inrupt**.

## 11.2 Installer

```bash
npm install \
  @inrupt/solid-client \
  @inrupt/solid-client-authn-browser \
  @inrupt/vocab-common-rdf
```

## 11.3 Session navigateur

```javascript
import {
  getDefaultSession,
  handleIncomingRedirect,
  login,
  logout,
} from "@inrupt/solid-client-authn-browser";

const session = getDefaultSession();

await handleIncomingRedirect({
  restorePreviousSession: true,
});

if (!session.info.isLoggedIn) {
  await login({
    oidcIssuer: "https://idp.example/",
    redirectUrl: window.location.href,
    clientName: "Mon application Solid",
  });
}

console.log(session.info.webId);
```

## 11.4 Le `fetch` authentifié

Une fois la session ouverte :

```javascript
const authenticatedFetch = session.fetch;
```

C'est ce `fetch` qu'il faut transmettre aux fonctions qui accèdent aux ressources protégées.

## 11.5 Lire un dataset RDF

```javascript
import {
  getSolidDataset,
} from "@inrupt/solid-client";

const dataset = await getSolidDataset(
  "https://pod.example/alice/notes/note-001",
  {
    fetch: session.fetch,
  },
);
```

## 11.6 Lire un Thing

Un `Thing` représente un ensemble de propriétés RDF liées à un sujet.

```javascript
import {
  getThing,
  getStringNoLocale,
} from "@inrupt/solid-client";

const resourceUrl =
  "https://pod.example/alice/notes/note-001";

const note = getThing(
  dataset,
  `${resourceUrl}#note`,
);

const title = getStringNoLocale(
  note,
  "https://schema.org/name",
);

console.log(title);
```

> [!warning]
> `getThing()` peut ne rien retourner. Le code de production doit gérer ce cas.

## 11.7 Créer un dataset

```javascript
import {
  createSolidDataset,
  createThing,
  setStringNoLocale,
  setThing,
  saveSolidDatasetAt,
} from "@inrupt/solid-client";

const SCHEMA = "https://schema.org/";

let dataset = createSolidDataset();

let note = createThing({
  name: "note",
});

note = setStringNoLocale(
  note,
  `${SCHEMA}name`,
  "Cours Solid",
);

note = setStringNoLocale(
  note,
  `${SCHEMA}text`,
  "Une note stockée dans un Pod",
);

dataset = setThing(dataset, note);

await saveSolidDatasetAt(
  "https://pod.example/alice/notes/note-002",
  dataset,
  {
    fetch: session.fetch,
  },
);
```

## 11.8 Gestion des erreurs

```javascript
try {
  const dataset = await getSolidDataset(url, {
    fetch: session.fetch,
  });
} catch (error) {
  console.error("Impossible de lire la ressource", error);
}
```

En production, distinguer notamment :

```text
401 -> authentification nécessaire/incorrecte
403 -> accès refusé
404 -> absent ou éventuellement masqué
409 -> conflit
412 -> précondition échouée
5xx -> problème serveur
```

## 11.9 Ne pas concaténer les URI naïvement

Éviter :

```javascript
const url = pod + "/notes/" + id;
```

si `pod` peut déjà finir par `/`.

Utiliser :

```javascript
const url = new URL(`notes/${id}`, podUrl);
```

## 11.10 Déconnexion

```javascript
await logout();
```

Une application doit fournir une UX claire pour :

- l'état connecté ;
- l'identité courante ;
- le Pod courant ;
- la déconnexion.

---

# 12. Structurer les données d'une application

## 12.1 Le vrai défi : le modèle de données

Faire un `GET` ou un `PUT` est simple.

Le défi est de permettre à plusieurs applications de comprendre le même contenu.

## 12.2 Exemple de mauvaise modélisation

```turtle
@prefix app: <https://mon-app.example/ns#>.

<#note>
    app:t "Mon titre" ;
    app:c "Mon contenu" ;
    app:d "2026-08-29" .
```

Une autre application ne sait pas ce que signifient :

```text
app:t
app:c
app:d
```

## 12.3 Exemple plus interopérable

```turtle
@prefix schema: <https://schema.org/>.

<#note>
    a schema:CreativeWork ;
    schema:name "Mon titre" ;
    schema:text "Mon contenu" ;
    schema:dateCreated "2026-08-29" .
```

## 12.4 Séparer le domaine de l'application

Éviter de faire dépendre les données de l'interface.

Mauvais :

```turtle
<#task>
    app:displayInRed true .
```

Meilleur :

```turtle
<#task>
    schema:status "blocked" .
```

Puis l'application décide que :

```text
blocked -> rouge
```

## 12.5 URI stables

Une donnée peut être référencée depuis plusieurs documents.

Exemple :

```text
https://pod.example/alice/tasks/01#task
```

Changer cette URI sans redirection peut casser des références.

## 12.6 Répartition des documents

Deux stratégies :

### Un gros document

```text
/tasks
```

Avantages :

- moins de requêtes ;
- simple au départ.

Inconvénients :

- conflits plus fréquents ;
- autorisation moins granulaire ;
- gros document à recharger.

### Une ressource par objet

```text
/tasks/001
/tasks/002
/tasks/003
```

Avantages :

- partage granulaire ;
- mise à jour indépendante ;
- meilleure concurrence.

Inconvénients :

- davantage de requêtes ;
- découverte plus complexe.

## 12.7 Données privées et publiques

Éviter de mélanger :

```text
profil public
+
données médicales privées
```

dans le même document RDF si leurs règles d'accès sont différentes.

La frontière des ressources doit prendre en compte les frontières d'autorisation.

---

# 13. Notifications et synchronisation

## 13.1 Pourquoi des notifications ?

Une application pourrait recharger une ressource toutes les secondes :

```text
GET
GET
GET
GET
GET
```

C'est du polling et cela gaspille :

- bande passante ;
- CPU ;
- batterie ;
- quotas.

## 13.2 Solid Notifications Protocol

Le protocole de notifications définit une manière de :

1. découvrir les capacités ;
2. créer une souscription ;
3. établir un canal ;
4. recevoir les changements.

Le modèle est extensible et peut utiliser différents types de canal.

## 13.3 Une notification n'est pas nécessairement la donnée

Une notification peut simplement indiquer :

```text
la ressource X a changé
```

L'application récupère ensuite l'état actuel.

Cette séparation facilite :

- contrôle d'accès ;
- cohérence ;
- reprise après déconnexion.

## 13.4 Ne pas supposer WebSocket

Les anciennes implémentations Solid utilisaient beaucoup des mécanismes WebSocket spécifiques.

Une application moderne doit préférer la découverte des capacités du serveur plutôt que supposer :

```text
ws://pod.example
```

## 13.5 Stratégie robuste

```text
notification reçue
        |
        v
invalidation du cache
        |
        v
GET conditionnel
        |
        v
mise à jour UI
```

## 13.6 Déduplication

Les systèmes distribués peuvent produire :

- notifications dupliquées ;
- événements en retard ;
- reconnexions.

Le client doit être idempotent autant que possible.

---

# 14. Sécurité, vie privée et modèle de menace

## 14.1 Solid ne supprime pas le besoin de sécurité applicative

Le fait que l'utilisateur contrôle un Pod ne rend pas automatiquement les applications sûres.

Menaces classiques :

- XSS ;
- CSRF selon l'architecture ;
- injection ;
- fuite de jeton ;
- SSRF côté serveur ;
- dépendances compromises ;
- ACL trop permissives ;
- erreurs CORS ;
- redirections OAuth mal contrôlées ;
- journalisation de données sensibles.

## 14.2 XSS et jetons

Si du JavaScript malveillant s'exécute dans le contexte d'une application authentifiée, il peut potentiellement exploiter la session.

La défense contre XSS reste essentielle :

- échappement ;
- CSP ;
- pas d'HTML non fiable injecté ;
- dépendances contrôlées.

## 14.3 Principe du moindre privilège

Une application devrait demander :

```text
uniquement les conteneurs nécessaires
uniquement les modes nécessaires
uniquement pendant la durée nécessaire
```

## 14.4 Séparer les catégories de sensibilité

Exemple :

```text
/public/
    profil public

/shared/
    documents partagés

/private/
    notes personnelles

/health/
    données sensibles
```

Cela facilite les politiques d'accès.

## 14.5 Données RDF et inférences

Le risque n'est pas seulement le contenu explicite.

Des liens peuvent révéler des informations par corrélation.

Exemple :

```text
Alice -> membre de -> groupe X
Alice -> consulte -> clinique Y
```

Même sans diagnostic explicite, des inférences peuvent être sensibles.

## 14.6 Métadonnées

Les métadonnées peuvent révéler :

- horaires ;
- fréquence d'accès ;
- structure des ressources ;
- relations ;
- taille des fichiers.

Le chiffrement du transport ne masque pas tout au serveur.

## 14.7 HTTPS

En production :

```text
HTTP -> interdit pour les identités et données sensibles
HTTPS -> obligatoire
```

Le TLS protège le transport.

Il ne protège pas contre :

- serveur compromis ;
- application compromise ;
- utilisateur malveillant autorisé.

## 14.8 Sauvegarde

Le contrôle de ses données implique aussi la responsabilité de ne pas les perdre.

Prévoir :

- sauvegarde ;
- restauration testée ;
- versioning si nécessaire ;
- réplication adaptée ;
- chiffrement des backups.

## 14.9 Suppression

Une requête `DELETE` ne garantit pas nécessairement la disparition instantanée de :

- sauvegardes ;
- caches ;
- journaux ;
- copies tierces déjà autorisées.

Une politique de rétention reste nécessaire.

## 14.10 Applications tierces

Avant d'autoriser une application :

- identifier son éditeur ;
- comprendre les permissions ;
- limiter les ressources ;
- savoir comment révoquer l'accès.

---

# 15. Interopérabilité et portabilité

## 15.1 Portabilité syntaxique et sémantique

Deux applications peuvent lire le même Turtle sans réellement comprendre les mêmes concepts.

Il faut distinguer :

### Interopérabilité syntaxique

```text
les deux savent lire RDF
```

### Interopérabilité sémantique

```text
les deux comprennent la signification des termes
```

### Interopérabilité comportementale

```text
les deux interprètent correctement les workflows
```

## 15.2 Application Interoperability

Le travail **Solid Application Interoperability** cherche à améliorer la capacité des applications et agents sociaux à partager et comprendre les données d'un Pod.

En 2026, ce travail reste un **Draft Community Group Report** et ne doit pas être supposé universellement implémenté.

## 15.3 Découverte

Éviter de coder :

```javascript
const contacts = `${pod}/contacts/`;
```

comme vérité universelle.

Deux utilisateurs peuvent organiser leurs Pods différemment.

La découverte des ressources et conventions de données est un enjeu important.

## 15.4 Migration de fournisseur

Un objectif de Solid est de rendre possible :

```text
Pod Provider A
      |
      | migration
      v
Pod Provider B
```

Mais une migration réelle doit gérer :

- données ;
- URI ;
- WebID ;
- ACL/policies ;
- liens entrants ;
- clients autorisés ;
- notifications.

## 15.5 Les URI rendent la migration délicate

Si toutes les données utilisent :

```text
https://provider-a.example/alice/...
```

une migration vers :

```text
https://provider-b.example/alice/...
```

peut casser des liens.

D'où l'intérêt, selon l'architecture, de contrôler son domaine ou de prévoir une stratégie d'identifiants durables.

---

# 16. Déploiement et exploitation

## 16.1 Séparer laboratoire et production

Laboratoire :

```text
localhost
stockage mémoire
utilisateur de test
```

Production :

```text
HTTPS
stockage persistant
sauvegardes
monitoring
rotation des secrets
durcissement
mises à jour
```

## 16.2 Reverse proxy

Une architecture courante :

```text
Internet
   |
   v
Nginx / Caddy / Traefik
   |
   | HTTPS / reverse proxy
   v
Community Solid Server
   |
   v
stockage
```

## 16.3 Headers proxy

Vérifier :

- `Host` ;
- `X-Forwarded-Proto` ;
- adresse client ;
- taille maximale des requêtes ;
- timeouts.

Une mauvaise configuration peut produire :

- URLs incorrectes ;
- redirects HTTP ;
- erreurs OIDC ;
- failles de confiance proxy.

## 16.4 Sauvegardes

Une sauvegarde doit inclure les données nécessaires au fonctionnement réel :

- ressources Pod ;
- métadonnées ;
- configuration ;
- état d'identité selon l'architecture ;
- secrets séparément et de manière sécurisée.

## 16.5 Monitoring

Surveiller :

```text
latence HTTP
5xx
401/403 anormaux
espace disque
CPU
RAM
nombre de connexions
expiration certificats
backups
```

## 16.6 Journalisation

Ne pas loguer inutilement :

- Authorization headers ;
- tokens ;
- cookies ;
- données personnelles RDF ;
- URLs contenant des informations sensibles.

## 16.7 Mise à jour

Avant une mise à jour majeure :

1. lire les release notes ;
2. sauvegarder ;
3. tester sur préproduction ;
4. vérifier OIDC ;
5. vérifier WAC/ACP ;
6. vérifier les notifications ;
7. vérifier les clients ;
8. prévoir un rollback.

## 16.8 Community Solid Server 7.2.0

La branche 7.x reste active en 2026.

La 7.2.0 a notamment renforcé certains comportements de sécurité, y compris des contrôles liés aux risques SSRF dans la validation de propriété de jetons.

C'est une bonne raison de ne pas figer un serveur ancien simplement parce qu'il fonctionne.

---

# 17. Limites, maturité et choix d'architecture

## 17.1 Solid n'est pas une solution magique

Solid apporte des propriétés intéressantes, mais aussi de la complexité.

Une application classique :

```text
frontend -> API -> DB
```

peut être plus simple à construire et exploiter.

Une application Solid doit gérer :

- données distantes ;
- identités externes ;
- autorisation distribuée ;
- vocabulaires ;
- disponibilité du Pod ;
- compatibilité entre serveurs ;
- migrations.

## 17.2 Le coût de l'interopérabilité

L'interopérabilité impose de réfléchir à :

- modèles ;
- URI ;
- versions ;
- vocabulaires ;
- politiques ;
- conventions.

Ce coût existe dès la conception.

## 17.3 Disponibilité

Si le Pod de l'utilisateur est indisponible :

```text
application disponible
+
données indisponibles
=
fonctionnalité indisponible
```

Il faut décider :

- cache local ?
- mode hors-ligne ?
- réplication ?
- file de synchronisation ?
- reprise ?

## 17.4 Performances

Un graphe distribué sur beaucoup de documents peut nécessiter de nombreuses requêtes.

```text
GET profil
GET préférences
GET contacts
GET document A
GET document B
...
```

Solutions possibles :

- cache ;
- regroupement judicieux ;
- chargement progressif ;
- requêtes conditionnelles ;
- notifications.

## 17.5 Cohérence

Il n'existe pas nécessairement de transaction ACID distribuée entre plusieurs ressources et plusieurs Pods.

Exemple :

```text
PUT Pod Alice -> succès
PUT Pod Bob   -> échec
```

Le workflow doit savoir gérer les états partiels.

## 17.6 Quand Solid est pertinent

Solid est intéressant lorsque :

- le contrôle des données est central ;
- plusieurs applications doivent partager des données ;
- les utilisateurs doivent pouvoir choisir leur stockage ;
- l'organisation veut éviter un silo propriétaire ;
- le domaine bénéficie des Linked Data.

## 17.7 Quand une architecture classique peut suffire

Pour :

- application interne simple ;
- données strictement propres à une application ;
- workflow fortement transactionnel ;
- contraintes de latence très fortes ;
- équipe sans besoin d'interopérabilité ;

une architecture classique peut être plus appropriée.

---

# 18. Cas d'usage

## 18.1 Réseau social décentralisé

Le profil et certaines relations sont stockés hors de l'application.

Applications :

```text
Client A
Client B
Client C
```

peuvent théoriquement travailler sur les mêmes données autorisées.

## 18.2 Éducation

Un étudiant pourrait conserver :

- parcours ;
- compétences ;
- travaux ;
- badges ;
- préférences ;

dans un espace contrôlé indépendamment d'une plateforme pédagogique.

Les établissements auraient accès uniquement aux éléments nécessaires.

## 18.3 Santé

Un Pod peut être envisagé pour faciliter le contrôle des données par le patient.

Mais le domaine médical impose :

- réglementation ;
- audit ;
- chiffrement ;
- disponibilité ;
- traçabilité ;
- gouvernance ;
- responsabilité clinique.

Solid ne remplace aucune de ces exigences.

## 18.4 Recherche scientifique

Solid peut être intéressant pour :

- consentement ;
- partage contrôlé ;
- provenance ;
- données liées ;
- collaborations multi-organisations.

## 18.5 Entreprises

Possibilités :

- espace de données personnel des salariés ;
- partage inter-organisations ;
- profils professionnels ;
- contrôle d'accès fédéré.

## 18.6 Applications personnelles

Exemples de TP réalistes :

- notes ;
- bookmarks ;
- contacts ;
- tâches ;
- bibliothèque de livres ;
- recettes ;
- journal sportif.

---

# 19. Travaux pratiques

# TP 1 — Lire et écrire du Turtle

## Objectif

Comprendre RDF indépendamment de Solid.

Créer :

```text
person.ttl
```

contenant :

```turtle
@prefix schema: <https://schema.org/>.

<#me>
    a schema:Person ;
    schema:name "Alice" ;
    schema:knows <https://example.org/bob#me> .
```

Identifier :

- sujet ;
- prédicats ;
- littéral ;
- IRI.

---

# TP 2 — Lancer Community Solid Server

## Objectif

Lancer un serveur local moderne.

```bash
mkdir solid-lab
cd solid-lab

npx @solid/community-server
```

Vérifier :

```text
http://localhost:3000/
```

Puis relancer avec stockage persistant :

```bash
npx @solid/community-server \
  -c @css:config/file.json \
  -f .data
```

Arrêter et redémarrer.

Vérifier que les données persistent.

---

# TP 3 — Explorer HTTP avec curl

Tester :

```bash
curl -i -X OPTIONS http://localhost:3000/
```

Puis :

```bash
curl -i \
  -H 'Accept: text/turtle' \
  http://localhost:3000/
```

Relever :

- status ;
- `Content-Type` ;
- `Allow` ;
- `Link` ;
- `Accept-Patch`.

---

# TP 4 — Créer un mini espace de notes

Créer :

```text
/notes/
```

puis :

```text
/notes/001
/notes/002
```

Chaque note doit être en RDF.

Exemple :

```turtle
@prefix schema: <https://schema.org/>.

<#it>
    a schema:CreativeWork ;
    schema:name "Note 1" ;
    schema:text "Contenu" .
```

---

# TP 5 — WebID

Créer ou utiliser un WebID de test.

Identifier :

```text
document du profil
fragment du WebID
OIDC issuer
storage
```

Dessiner le graphe RDF du profil.

---

# TP 6 — Authentification navigateur

Créer une petite application JavaScript.

Installer :

```bash
npm install \
  @inrupt/solid-client \
  @inrupt/solid-client-authn-browser
```

Implémenter :

- bouton login ;
- affichage du WebID ;
- logout ;
- état de session.

> [!important]
> Utiliser un Pod et un Identity Provider de laboratoire. Ne pas développer le flux OAuth/OIDC à la main.

---

# TP 7 — Application de notes

Implémenter :

```text
liste
création
lecture
modification
suppression
```

Les données doivent rester dans le Pod.

L'application ne possède pas sa propre base de notes.

---

# TP 8 — Autorisation

Créer deux identités de test :

```text
Alice
Bob
```

Objectif :

```text
Alice -> contrôle
Bob   -> lecture seule
```

Tester effectivement :

- `GET` ;
- `PUT` ;
- `DELETE`.

Documenter si le serveur utilise WAC ou ACP dans la configuration choisie.

---

# TP 9 — Concurrence

Scénario :

1. lire une ressource ;
2. conserver son `ETag` ;
3. modifier la ressource depuis un second client ;
4. tenter une mise à jour conditionnelle avec le premier.

Comprendre :

```text
If-Match
412 Precondition Failed
```

---

# TP 10 — Sécurité

Effectuer un mini audit :

- serveur uniquement local ou HTTPS ;
- permissions minimales ;
- aucun token dans les logs ;
- aucun secret dans Git ;
- données privées séparées ;
- dépendances à jour ;
- configuration de production différente du laboratoire.

---

# 20. Projet final

## 20.1 Sujet

Développer une application **Todo Solid**.

L'utilisateur doit pouvoir choisir son Pod et conserver ses tâches hors de l'application.

## 20.2 Modèle

Une tâche :

```turtle
@prefix schema: <https://schema.org/>.

<#task>
    a schema:Action ;
    schema:name "Relire le cours Solid" ;
    schema:description "Valider le chapitre OIDC" ;
    schema:startTime "2026-08-29" ;
    schema:actionStatus schema:PotentialActionStatus .
```

## 20.3 Architecture

```text
+-------------------------+
|      Todo Solid         |
| navigateur              |
+------------+------------+
             |
             | Solid-OIDC
             v
+-------------------------+
|     Identity Provider   |
+-------------------------+

             |
             | authenticated fetch
             v

+-------------------------+
|          Pod            |
| /todo/                  |
|   001                   |
|   002                   |
|   003                   |
+-------------------------+
```

## 20.4 Fonctions

Obligatoires :

- connexion ;
- affichage du WebID ;
- choix du conteneur ;
- création ;
- lecture ;
- modification ;
- suppression ;
- déconnexion ;
- affichage des erreurs.

## 20.5 Contraintes

L'application ne doit pas :

- stocker les tâches dans une base propriétaire ;
- demander le contrôle de tout le Pod ;
- supposer un domaine de Pod fixe ;
- insérer du HTML non fiable ;
- stocker des jetons dans le dépôt Git.

## 20.6 Bonus

- mode hors-ligne ;
- cache avec ETag ;
- notifications ;
- partage d'une tâche ;
- vocabulaire documenté ;
- tests automatisés ;
- Docker ;
- CI.

## 20.7 Évaluation

| Critère | Points |
|---|---:|
| Compréhension de RDF | 15 |
| Utilisation correcte de HTTP/Solid | 15 |
| Authentification | 15 |
| Autorisation | 15 |
| Modèle de données | 15 |
| Sécurité | 10 |
| Qualité du code | 10 |
| Documentation | 5 |

---

# 21. Checklist

## Architecture

- [ ] Les données sont séparées de l'application.
- [ ] Le Pod n'est pas supposé appartenir au fournisseur de l'application.
- [ ] Les URI sont traitées comme des URI, pas comme de simples chemins.
- [ ] Les ressources sensibles sont séparées.

## RDF

- [ ] Les vocabulaires existants sont privilégiés.
- [ ] Les URI sont stables.
- [ ] Les littéraux ont les types nécessaires.
- [ ] Le modèle est documenté.
- [ ] L'application ne suppose pas un endpoint SPARQL universel.

## HTTP

- [ ] `Content-Type` est correct.
- [ ] `Accept` est utilisé.
- [ ] Les erreurs HTTP sont traitées.
- [ ] Les mises à jour concurrentes sont prises en compte.
- [ ] Les capacités sont découvertes lorsque nécessaire.

## Identité

- [ ] Le WebID est traité comme un identifiant.
- [ ] Solid-OIDC est implémenté avec une bibliothèque maintenue.
- [ ] L'issuer est correctement utilisé.
- [ ] Les sessions peuvent être révoquées/déconnectées.

## Autorisation

- [ ] Le mécanisme du serveur est identifié.
- [ ] Les permissions suivent le moindre privilège.
- [ ] L'application ne demande pas `Control` sans raison.
- [ ] Le code ne construit pas les URLs d'ACL par hypothèse fragile.

## Sécurité

- [ ] HTTPS en production.
- [ ] Pas de jetons dans les logs.
- [ ] Pas de secrets dans le frontend ou Git.
- [ ] CSP et prévention XSS.
- [ ] Dépendances maintenues.
- [ ] Sauvegardes testées.
- [ ] Monitoring.
- [ ] Procédure de mise à jour.

## Exploitation

- [ ] Stockage persistant.
- [ ] Certificats surveillés.
- [ ] Reverse proxy correctement configuré.
- [ ] Backups chiffrés si nécessaire.
- [ ] Rollback testé.
- [ ] Ressources et identité documentées.

---

# 22. Glossaire

**ACP**
Access Control Policy. Modèle de politiques d'autorisation RDF.

**Agent**
Entité qui agit sur des ressources : personne, application ou autre acteur.

**Community Solid Server (CSS)**
Serveur Solid open source modulaire, particulièrement utilisé pour le développement, la recherche et les tests.

**DPoP**
Demonstrating Proof of Possession. Mécanisme OAuth liant un jeton à une clé.

**IRI**
Internationalized Resource Identifier. Généralisation des URI utilisée dans RDF.

**LDP**
Linked Data Platform.

**Linked Data**
Principes permettant de publier et relier des données identifiées sur le Web.

**N3 Patch**
Format de patch RDF supporté par le Solid Protocol pour les opérations `PATCH`.

**OIDC**
OpenID Connect.

**Pod**
Espace de stockage de données Solid.

**RDF**
Resource Description Framework.

**Resource Server**
Serveur HTTP hébergeant des ressources protégées.

**Solid-OIDC**
Profil OIDC destiné à l'environnement Solid.

**Thing**
Terme utilisé par certaines bibliothèques clientes pour représenter un sujet RDF et ses propriétés.

**Turtle**
Syntaxe textuelle RDF.

**WAC**
Web Access Control.

**WebID**
Identifiant Web HTTP(S) d'un agent, déréférençable vers un profil RDF.

---

# 23. Sources et références

## Solid Project

- https://solidproject.org/
- https://solidproject.org/faq
- https://solidproject.org/TR/
- https://solidproject.org/for_developers/getting_started

## Spécifications Solid

- Solid Protocol : https://solidproject.org/TR/protocol
- Web Access Control : https://solidproject.org/TR/wac
- Access Control Policy : https://solidproject.org/TR/acp
- Solid-OIDC : https://solidproject.org/TR/oidc
- Solid Notifications Protocol : https://solidproject.org/TR/notifications-protocol
- Solid Application Interoperability : https://solidproject.org/TR/sai

## Community Solid Server

- https://github.com/CommunitySolidServer/CommunitySolidServer
- https://communitysolidserver.github.io/CommunitySolidServer/latest/docs/
- https://github.com/CommunitySolidServer/tutorials

## Bibliothèques JavaScript

- https://docs.inrupt.com/sdk/javascript-sdk
- https://docs.inrupt.com/guides/authentication-in-solid/authentication-from-browser

## Standards Web connexes

- RDF : https://www.w3.org/TR/rdf11-concepts/
- Turtle : https://www.w3.org/TR/turtle/
- JSON-LD : https://www.w3.org/TR/json-ld11/
- Linked Data Platform : https://www.w3.org/TR/ldp/
- HTTP : https://www.rfc-editor.org/rfc/rfc9110
- OAuth 2.0 : https://www.rfc-editor.org/rfc/rfc6749
- OpenID Connect : https://openid.net/specs/openid-connect-core-1_0.html
- DPoP : https://www.rfc-editor.org/rfc/rfc9449

---

# Conclusion

Solid propose une idée structurante :

> **les applications ne devraient pas nécessairement posséder les données de leurs utilisateurs.**

Son architecture repose sur des technologies Web classiques et interopérables :

```text
HTTP
+
RDF / Linked Data
+
WebID
+
OIDC
+
contrôle d'accès
+
Pods
```

La difficulté n'est pas seulement de lire et écrire un document distant. Le véritable enjeu est de construire un système où :

- les identités restent portables ;
- les données restent compréhensibles ;
- les permissions restent maîtrisées ;
- les URI restent durables ;
- plusieurs applications peuvent réellement interopérer ;
- l'utilisateur conserve un contrôle effectif sur ses données.

En 2026, Solid reste un écosystème actif et techniquement intéressant, mais ses principales spécifications sont encore des travaux de Community Group. Une architecture Solid doit donc être conçue avec une attention particulière à la **maturité des spécifications, à la compatibilité des serveurs et à l'interopérabilité réelle**, plutôt qu'en supposant que toutes les fonctionnalités sont uniformément implémentées.
