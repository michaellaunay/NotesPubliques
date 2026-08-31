---
schema_version: 1
uid: 01M1BQ62754J1HSR3N887Y1Y4P
titre: "RAG — 02 — Les embeddings représenter le sens sous forme de vecteur"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - intelligence-artificielle
  - rag
  - recherche-vectorielle
  - llm
  - embeddings
resume: "Chapitre 2 sur 12 du livre « RAG » : Les embeddings : représenter le sens sous forme de vecteur. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
niveau: avance
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2026-06-03
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre « RAG » — chapitre 2/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 01 — Pourquoi le RAG|← 01 — Pourquoi le RAG]] · [[RAG — 03 — Similarité cosinus et recherche vectorielle|03 — Similarité cosinus et recherche vectorielle →]]

# Chapitre 2 — Les embeddings : représenter le sens sous forme de vecteur
## 2.1. Introduction

Dans le chapitre précédent, nous avons vu pourquoi le RAG existe. Nous avons compris qu’un LLM seul ne suffit pas toujours lorsqu’il doit répondre à partir de documents spécifiques, récents ou vérifiables.

Nous avons aussi vu qu’un système RAG repose sur une idée simple :

```text
Nous cherchons d’abord les passages pertinents,
puis nous demandons au LLM de répondre à partir de ces passages.
```

Mais une question centrale apparaît immédiatement :

```text
Comment retrouver automatiquement les passages pertinents dans une grande base documentaire ?
```

Si l’utilisateur demande :

```text
Comment réinitialiser mes identifiants ?
```

il est possible que le document pertinent contienne plutôt :

```text
Procédure de changement du mot de passe utilisateur.
```

Les mots ne sont pas exactement les mêmes, mais le sens est proche.

Une recherche classique par mots-clés peut rater certains résultats utiles. C’est là qu’interviennent les **embeddings**.

Un embedding permet de représenter un texte sous forme de vecteur numérique, de façon à ce que des textes proches en sens soient proches dans l’espace vectoriel.

Dans ce chapitre, nous allons comprendre :

```text
ce qu’est un embedding ;
pourquoi on transforme le texte en vecteurs ;
comment ces vecteurs permettent la recherche sémantique ;
quelle est la différence entre recherche lexicale et recherche vectorielle ;
quelles sont les limites des embeddings.
```

---

## 2.2. Pourquoi transformer du texte en nombres ?

Un ordinateur ne comprend pas directement le sens d’une phrase. Pour pouvoir comparer, classer ou rechercher des textes, nous devons les représenter dans une forme mathématique.

Un texte brut est une suite de caractères :

```text
"Le service checkout utilise l’API payments."
```

Pour effectuer des calculs, nous devons transformer cette phrase en nombres.

Un embedding est précisément cette transformation :

```text
texte → vecteur de nombres réels
```

Par exemple :

```text
"Le service checkout utilise l’API payments."
→ [0.12, -0.34, 0.87, 0.05, ..., -0.21]
```

Ce vecteur peut contenir plusieurs centaines ou plusieurs milliers de dimensions.

L’objectif n’est pas que chaque dimension soit facilement interprétable par un humain. L’objectif est que la position du vecteur dans l’espace encode une partie du sens du texte.

Autrement dit, nous voulons que deux textes proches en sens soient représentés par deux vecteurs proches.

---

## 2.3. Intuition générale d’un embedding

Prenons trois phrases :

```text
A. Le chien dort sur le canapé.
B. Un animal domestique se repose sur le sofa.
C. PostgreSQL permet de gérer des transactions SQL.
```

Pour nous, les phrases A et B sont proches.

Elles ne partagent pas exactement les mêmes mots :

```text
chien ≠ animal domestique
dort ≠ se repose
canapé ≠ sofa
```

Mais elles décrivent une situation similaire.

La phrase C, elle, parle d’un sujet complètement différent.

Un bon modèle d’embedding devrait donc produire quelque chose comme :

```text
embedding(A) proche de embedding(B)
embedding(A) éloigné de embedding(C)
embedding(B) éloigné de embedding(C)
```

En simplifiant, nous pouvons imaginer un espace en deux dimensions :

```text
                         informatique
                              ↑
                              |
                              |        PostgreSQL
                              |             x
                              |
                              |
                              |
chien / canapé       x   x
                              |
------------------------------→ animal / domestique
```

Dans la réalité, l’espace n’a pas deux dimensions, mais souvent plusieurs centaines ou milliers. Cependant, l’idée reste la même : les textes sont placés dans un espace où la proximité géométrique doit refléter une proximité de sens.

---

## 2.4. Embedding de mot, de phrase et de document

Il existe plusieurs niveaux d’embeddings.

### 2.4.1. Embedding de mot

Historiquement, beaucoup de modèles représentaient chaque mot par un vecteur.

Exemple :

```text
chien → vecteur
chat → vecteur
voiture → vecteur
ordinateur → vecteur
```

Un bon modèle place les mots proches dans des zones proches.

Par exemple :

```text
chien
chat
animal
vétérinaire
```

devraient être relativement proches.

Tandis que :

```text
SQL
PostgreSQL
transaction
index
```

devraient être proches entre eux, mais éloignés du vocabulaire animalier.

### 2.4.2. Embedding de phrase

Pour un RAG, nous voulons souvent représenter des phrases ou des paragraphes entiers.

Exemple :

```text
"Le cluster-3 sera en maintenance vendredi."
→ vecteur
```

Cela permet de comparer une question utilisateur avec un passage documentaire.

### 2.4.3. Embedding de document

Nous pouvons aussi créer un embedding pour un document entier.

Mais dans un système RAG, nous évitons souvent d’indexer seulement des documents entiers, car ils sont trop longs et trop généraux.

Si un document de vingt pages contient une seule information pertinente, l’embedding global du document risque de diluer cette information.

C’est pourquoi nous découpons généralement les documents en morceaux plus petits, appelés **chunks**.

Nous y reviendrons dans un chapitre dédié.

---

## 2.5. Recherche lexicale et recherche sémantique

Pour comprendre l’intérêt des embeddings, comparons deux manières de chercher de l’information.

### 2.5.1. Recherche lexicale

La recherche lexicale repose sur les mots présents dans la requête.

Si nous cherchons :

```text
mot de passe
```

le moteur cherche des documents contenant :

```text
mot
passe
mot de passe
```

Cette approche est simple et très efficace dans beaucoup de cas.

Elle fonctionne particulièrement bien pour :

```text
noms exacts ;
identifiants ;
codes d’erreur ;
noms de fichiers ;
noms de classes ;
références ;
numéros de tickets ;
termes rares.
```

Exemple :

```text
Erreur : ERR_MODULE_NOT_FOUND
```

Dans ce cas, une recherche lexicale est souvent excellente, car nous voulons retrouver exactement cette chaîne.

### 2.5.2. Recherche sémantique

La recherche sémantique ne cherche pas seulement les mêmes mots. Elle cherche des textes proches en sens.

Exemple :

```text
Question :
Comment changer mon mot de passe ?

Document :
Procédure de réinitialisation des identifiants utilisateur.
```

Une recherche lexicale peut avoir du mal, car les mots ne sont pas identiques.

Une recherche sémantique peut comprendre que :

```text
changer mot de passe
≈ réinitialisation des identifiants
```

Elle peut donc retrouver le bon document.

### 2.5.3. Les deux approches sont complémentaires

Il ne faut pas opposer naïvement recherche lexicale et recherche sémantique.

La recherche vectorielle n’est pas toujours meilleure.

Exemple :

```text
Question :
Que signifie l’erreur P2025 dans Prisma ?
```

Ici, le terme `P2025` est très important. Une recherche lexicale ou hybride sera souvent plus fiable qu’une recherche purement sémantique.

Autre exemple :

```text
Question :
Où est utilisé le fichier docker-compose.prod.yml ?
```

Le nom exact du fichier doit être retrouvé.

En production, beaucoup de systèmes RAG sérieux utilisent une recherche hybride :

```text
recherche lexicale
+
recherche vectorielle
+
filtrage par métadonnées
+
reranking
```

---

## 2.6. Comment un modèle produit-il un embedding ?

Un modèle d’embedding est un modèle entraîné pour transformer du texte en vecteur.

Nous pouvons le voir comme une fonction :

```text
f(texte) = vecteur
```

Par exemple :

```text
f("Le cluster-3 est en maintenance vendredi")
=
[0.18, -0.22, 0.74, ..., 0.09]
```

Le modèle a appris, pendant son entraînement, à placer des textes proches dans des régions proches de l’espace vectoriel.

Il ne s’agit pas d’un dictionnaire manuel. Le modèle apprend statistiquement à partir de grands volumes de textes.

### 2.6.1. Exemple conceptuel

Supposons que le modèle ait vu beaucoup de textes où les mots suivants apparaissent dans des contextes similaires :

```text
mot de passe
password
identifiants
authentification
connexion
réinitialisation
compte utilisateur
```

Il apprend progressivement que ces concepts sont liés.

Donc une question comme :

```text
Je n’arrive plus à me connecter à mon compte.
```

peut être rapprochée d’un document intitulé :

```text
Procédure de réinitialisation du mot de passe.
```

Même si la question ne contient pas explicitement “mot de passe”.

### 2.6.2. Les dimensions ne sont pas directement lisibles

Un embedding peut ressembler à ceci :

```text
[0.018, -0.104, 0.582, -0.331, ..., 0.047]
```

Nous ne pouvons généralement pas dire :

```text
la dimension 12 représente la sécurité ;
la dimension 53 représente Kubernetes ;
la dimension 118 représente le droit.
```

Les dimensions sont des représentations distribuées. Le sens est encodé dans des combinaisons de dimensions, pas dans une seule coordonnée facilement lisible.

C’est une différence importante avec des systèmes symboliques où chaque relation est explicitement nommée.

---

## 2.7. Exemple dans un système RAG

Prenons un mini corpus documentaire :

```text
Doc 1 :
La procédure de réinitialisation du mot de passe nécessite une validation par le responsable sécurité.

Doc 2 :
Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.

Doc 3 :
Les factures sont payables sous trente jours après émission.

Doc 4 :
L’API payments est utilisée par le service checkout.
```

Pendant l’indexation, nous transformons chaque document ou chunk en vecteur :

```text
embedding(Doc 1)
embedding(Doc 2)
embedding(Doc 3)
embedding(Doc 4)
```

Puis un utilisateur demande :

```text
Comment changer un mot de passe administrateur ?
```

Nous transformons aussi cette question en vecteur :

```text
embedding(question)
```

Ensuite, nous comparons l’embedding de la question avec les embeddings des documents.

Le document 1 devrait être le plus proche, car il parle de réinitialisation de mot de passe.

Le système récupère donc ce passage et le transmet au LLM.

Pipeline :

```text
Question utilisateur
        ↓
embedding de la question
        ↓
comparaison avec les embeddings des chunks
        ↓
récupération des chunks les plus proches
        ↓
injection dans le prompt
        ↓
réponse du LLM
```

---

## 2.8. Les embeddings ne sont pas des résumés

Il est important de ne pas confondre embedding et résumé.

Un résumé est lisible par un humain :

```text
Ce document explique la procédure de réinitialisation d’un mot de passe.
```

Un embedding ne l’est pas :

```text
[0.18, -0.22, 0.74, ..., 0.09]
```

Un embedding est une représentation mathématique destinée à être comparée avec d’autres représentations mathématiques.

Nous ne lisons pas un embedding. Nous le comparons.

C’est exactement ce qui le rend utile pour la recherche à grande échelle.

---

## 2.9. Les embeddings ne sont pas une preuve de vérité

Un embedding indique une proximité sémantique, pas une vérité.

Si deux textes sont proches dans l’espace vectoriel, cela signifie qu’ils parlent probablement de sujets proches. Cela ne signifie pas que l’un répond correctement à l’autre.

Exemple :

```text
Question :
Est-ce que le service checkout est affecté par la maintenance de vendredi ?

Chunk récupéré :
Le service checkout gère la validation des paniers.
```

Ce chunk est proche de la question, car il parle du service checkout.

Mais il ne répond pas à la question.

Il manque les informations sur :

```text
les dépendances du service checkout ;
l’API payments ;
le cluster-3 ;
la maintenance de vendredi.
```

Nous devons donc retenir une limite essentielle :

```text
similarité sémantique ≠ pertinence suffisante pour répondre
```

Un passage peut être proche mais incomplet.

À l’inverse, un passage peut être indispensable mais peu proche lexicalement ou sémantiquement de la question.

---

## 2.10. Le problème des faits intermédiaires

Reprenons l’exemple vu dans le chapitre précédent :

```text
Fait 1 :
Le service checkout utilise l’API payments.

Fait 2 :
L’API payments tourne sur le cluster-3.

Fait 3 :
Le cluster-3 sera en maintenance vendredi.
```

Question :

```text
Le service checkout sera-t-il affecté par la maintenance de vendredi ?
```

Le système vectoriel peut facilement retrouver le fait 1, car il contient :

```text
checkout
```

Il peut aussi retrouver le fait 3, car il contient :

```text
maintenance
vendredi
```

Mais il peut rater le fait 2 :

```text
L’API payments tourne sur le cluster-3.
```

Pourquoi ?

Parce que ce passage ne contient ni :

```text
checkout
maintenance
vendredi
affecté
```

Pourtant, il est indispensable pour construire la chaîne de raisonnement :

```text
checkout
→ payments API
→ cluster-3
→ maintenance vendredi
```

C’est une limite classique du RAG standard.

La recherche vectorielle retrouve des textes proches de la question. Elle ne garantit pas qu’elle retrouve tous les liens nécessaires au raisonnement.

C’est une des raisons pour lesquelles nous étudierons plus tard :

```text
Graph RAG ;
retrieval multi-hop ;
query expansion ;
agentic retrieval ;
recherche hybride.
```

---

## 2.11. Embeddings et granularité documentaire

Un embedding dépend fortement de ce que nous lui donnons à encoder.

Comparons trois niveaux.

### 2.11.1. Phrase seule

```text
L’API payments tourne sur le cluster-3.
```

L’embedding sera précis, mais le contexte est limité.

### 2.11.2. Paragraphe

```text
Le service checkout utilise l’API payments pour déclencher les paiements.
L’API payments tourne sur le cluster-3.
Le cluster-3 sera en maintenance vendredi.
```

L’embedding contient davantage de contexte et permet peut-être de retrouver plus facilement la relation.

### 2.11.3. Document entier

Si nous encodons un document entier de vingt pages, l’information spécifique risque d’être diluée.

Le vecteur représentera une moyenne sémantique approximative de nombreux sujets.

C’est pourquoi le **chunking** est fondamental.

Nous devons choisir une granularité qui préserve assez de contexte sans produire des morceaux trop gros.

---

## 2.12. Embeddings et métadonnées

Un embedding représente le contenu textuel, mais un bon système RAG ne stocke pas seulement un vecteur.

Il stocke aussi des métadonnées.

Par exemple :

```text
texte du chunk ;
embedding ;
document source ;
titre du document ;
section ;
page ;
date de mise à jour ;
auteur ;
type de document ;
droits d’accès ;
URL ;
version ;
tags métier.
```

Ces métadonnées sont essentielles.

Elles permettent :

```text
de citer les sources ;
de filtrer les résultats ;
de respecter les droits d’accès ;
de favoriser les documents récents ;
de limiter la recherche à un périmètre ;
d’expliquer la réponse.
```

Exemple :

```text
Question :
Quelle est la procédure actuelle de déploiement en production ?

Filtre :
type_document = "runbook"
environnement = "production"
date_version > 2025-01-01
```

Sans métadonnées, le système peut retrouver un ancien document obsolète mais sémantiquement très proche.

---

## 2.13. Modèles d’embedding généralistes et spécialisés

Tous les modèles d’embedding ne se valent pas pour tous les usages.

### 2.13.1. Modèle généraliste

Un modèle généraliste fonctionne correctement sur beaucoup de sujets :

```text
questions générales ;
documentation courante ;
FAQ ;
articles ;
emails ;
contenus métiers variés.
```

Il est souvent suffisant pour un premier prototype.

### 2.13.2. Modèle spécialisé

Certains domaines nécessitent des embeddings plus adaptés :

```text
biomédecine ;
droit ;
code source ;
mathématiques ;
documentation technique ;
langues spécifiques ;
documents multilingues.
```

Par exemple, un modèle généraliste peut être moins performant pour comparer des extraits de code, des logs techniques ou des clauses juridiques complexes.

### 2.13.3. Modèle multilingue

Si le corpus contient plusieurs langues, nous devons vérifier que le modèle gère correctement le multilingue.

Exemple :

```text
Question en français :
Comment réinitialiser mon mot de passe ?

Document en anglais :
Password reset procedure for administrator accounts.
```

Un bon modèle multilingue peut rapprocher ces deux textes.

Un modèle monolingue ou mal adapté peut échouer.

---

## 2.14. Embeddings pour du code source

Dans un contexte informatique, nous pouvons aussi créer des embeddings pour du code.

Exemple :

```typescript
async function resetPassword(userId: string) {
  await securityService.validateReset(userId);
  return authService.resetPassword(userId);
}
```

Une question pourrait être :

```text
Où est validée la réinitialisation du mot de passe ?
```

Un embedding de code peut aider à retrouver cette fonction.

Cependant, le code pose des problèmes spécifiques :

```text
les noms exacts sont très importants ;
la structure syntaxique compte ;
les appels de fonctions créent des dépendances ;
les types donnent du sens ;
les imports sont importants ;
le graphe d’appel peut être plus pertinent que la proximité sémantique.
```

Pour le code, une approche purement vectorielle doit souvent être complétée par :

```text
recherche lexicale ;
analyse statique ;
graphe d’appels ;
indexation des symboles ;
LSP (Language Server Protocol);
AST (Abstract Syntaxt Tree);
métadonnées Git.
```

C’est un bon exemple de limite des embeddings seuls.

Ici :
**LSP** signifie **Language Server Protocol**.

C’est un protocole utilisé par les IDE comme VS Code, IntelliJ, Cursor, etc., pour interroger un “serveur de langage” capable de comprendre un langage de programmation.

Par exemple, grâce au LSP, on peut savoir :

- où une fonction est définie ;
- où elle est utilisée ;
- quel est le type d’une variable ;
- quelles erreurs de compilation existent ;
- quels symboles existent dans un fichier ;
- quelles méthodes appartiennent à une classe ;
- quelle documentation est associée à une fonction.

Dans un RAG pour du code, le LSP permet donc d’avoir une compréhension plus “IDE-like” du projet.

Exemple :

```
userService.createUser(...)
```

Avec le LSP, le système peut retrouver directement la définition de `createUser`, même si elle est dans un autre fichier.

**AST** signifie **Abstract Syntax Tree**, en français **arbre syntaxique abstrait**.

C’est une représentation structurée du code sous forme d’arbre.

Par exemple, ce code :

```
function add(a: number, b: number) {  return a + b;}
```

peut être transformé en une structure du genre :

```
FunctionDeclaration ├── name: add ├── parameters │    ├── a: number │    └── b: number └── body      └── ReturnStatement           └── BinaryExpression: a + b
```

L’AST ne voit pas le code comme du texte brut, mais comme une structure logique.

Dans un RAG pour du code, l’AST sert à découper intelligemment le code :

- par fonction ;
- par classe ;
- par méthode ;
- par import/export ;
- par bloc logique ;
- par dépendances entre éléments.

C’est beaucoup mieux que de découper naïvement tous les 500 tokens, car une fonction reste entière et cohérente.

---

## 2.15. Embeddings et bases vectorielles

Les embeddings sont souvent stockés dans une base vectorielle.

Une base vectorielle permet de chercher rapidement les vecteurs les plus proches d’un vecteur de requête.

Pour chaque chunk, nous stockons :

```text
id du chunk ;
texte ;
embedding ;
métadonnées.
```

Quand l’utilisateur pose une question, nous calculons :

```text
embedding(question)
```

Puis nous cherchons :

```text
les k vecteurs les plus proches
```

C’est ce qu’on appelle souvent une recherche **top-k**.

Exemple :

```text
top_k = 5
```

Le système récupère les cinq chunks les plus proches.

Mais le choix de `k` est important.

Si `k` est trop petit, nous risquons de manquer un passage important.

Si `k` est trop grand, nous donnons trop de bruit au LLM.

---

## 2.16. Notion de score de similarité

La base vectorielle renvoie généralement des résultats avec un score.

Exemple :

```text
Chunk 1 : score 0.89
Chunk 2 : score 0.83
Chunk 3 : score 0.76
Chunk 4 : score 0.52
Chunk 5 : score 0.49
```

Ces scores indiquent une proximité entre la question et les chunks.

Mais attention : un score n’est pas universel.

Un score de 0.82 peut être excellent avec un modèle et moyen avec un autre.

Nous devons donc éviter les règles naïves du type :

```text
si score > 0.8 alors le document est forcément pertinent
```

En pratique, nous devons calibrer les seuils sur nos données.

---

## 2.17. Prétraitement du texte

Avant de créer les embeddings, nous devons parfois nettoyer le texte.

Exemples de nettoyage utile :

```text
supprimer le HTML inutile ;
corriger les problèmes d’encodage ;
retirer les menus répétés ;
supprimer les pieds de page répétitifs ;
normaliser les espaces ;
conserver les titres ;
conserver les tableaux sous une forme lisible ;
préserver les métadonnées importantes.
```

Mais il faut éviter de nettoyer trop brutalement.

Dans les anciens pipelines NLP, on supprimait souvent :

```text
stop words ;
ponctuation ;
accents ;
majuscules ;
formes grammaticales.
```

Avec les modèles modernes, ce n’est pas toujours souhaitable.

La ponctuation, les titres, les négations ou les formulations exactes peuvent avoir du sens.

Exemple :

```text
Le service doit être redémarré.
Le service ne doit pas être redémarré.
```

Si nous supprimons trop d’éléments, nous risquons de perdre une information critique.

---

## 2.18. Les limites générales des embeddings

Les embeddings sont puissants, mais ils ont plusieurs limites.

### 2.18.1. Ils capturent une proximité, pas un raisonnement

Un embedding rapproche des textes similaires, mais il ne construit pas nécessairement une chaîne logique.

### 2.18.2. Ils peuvent rater les termes exacts

Un identifiant rare, un code d’erreur ou un nom de fichier peut être mal traité par la recherche sémantique.

Exemple :

```text
ERR_MODULE_NOT_FOUND
P2025
docker-compose.prod.yml
UserRepositoryImpl
```

Pour ces cas, la recherche lexicale est souvent indispensable.

### 2.18.3. Ils peuvent rapprocher des textes trop généraux

Deux textes peuvent sembler proches parce qu’ils parlent du même domaine, sans que l’un réponde vraiment à l’autre.

Exemple :

```text
Question :
Comment corriger l’erreur Prisma P2025 ?

Chunk :
Prisma est un ORM pour TypeScript permettant d’interagir avec une base de données.
```

Le chunk parle de Prisma, mais il ne répond pas à la question.

### 2.18.4. Ils dépendent fortement du modèle

Changer de modèle d’embedding peut changer les résultats du retrieval.

Il faut donc considérer le modèle d’embedding comme un composant critique du système.

### 2.18.5. Ils vieillissent avec l’index

Si nous changeons de modèle d’embedding, nous devons généralement réindexer les documents.

On ne mélange pas proprement des vecteurs produits par plusieurs modèles différents dans le même espace.

---

## 2.19. Expérience pédagogique simple

Pour bien comprendre les embeddings, nous pouvons proposer une petite expérience.

Nous prenons les phrases suivantes :

```text
1. Comment réinitialiser mon mot de passe ?
2. Procédure de changement des identifiants utilisateur.
3. Le cluster Kubernetes sera redémarré vendredi.
4. PostgreSQL utilise des transactions ACID.
5. J’ai oublié mon password administrateur.
6. Les factures sont payables sous trente jours.
```

Nous calculons les embeddings de chaque phrase.

Puis nous cherchons les phrases les plus proches de :

```text
Je n’arrive plus à me connecter à mon compte.
```

Nous devrions obtenir en tête :

```text
1. Comment réinitialiser mon mot de passe ?
2. Procédure de changement des identifiants utilisateur.
3. J’ai oublié mon password administrateur.
```

Ce type d’exercice permet de comprendre concrètement que la recherche ne dépend pas uniquement des mots exacts, mais aussi de la proximité sémantique.

---

## 2.20. Mini-TP proposé

### Objectif

Nous voulons construire une première expérience de recherche sémantique.

### Données

Nous créons un petit corpus :

```text
doc_1 : La procédure de réinitialisation du mot de passe nécessite une validation sécurité.
doc_2 : Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.
doc_3 : Les factures doivent être payées sous trente jours.
doc_4 : Le service checkout utilise l’API payments.
doc_5 : L’API payments est déployée sur le cluster-3.
```

### Étapes

Nous allons :

```text
1. Choisir un modèle d’embedding.
2. Vectoriser chaque document.
3. Vectoriser une question utilisateur.
4. Calculer la similarité entre la question et chaque document.
5. Trier les documents par score.
6. Observer les résultats.
```

### Questions à tester

```text
Comment changer un mot de passe ?
Le service checkout sera-t-il touché vendredi ?
Quand faut-il payer une facture ?
Quel service utilise l’API payments ?
```

### Analyse attendue

Nous devrons observer que certaines questions sont faciles :

```text
Quand faut-il payer une facture ?
```

devrait retrouver :

```text
Les factures doivent être payées sous trente jours.
```

Mais la question :

```text
Le service checkout sera-t-il touché vendredi ?
```

peut être plus difficile, car elle nécessite plusieurs documents :

```text
checkout utilise payments
payments est sur cluster-3
cluster-3 est en maintenance vendredi
```

Cette expérience prépare le terrain pour comprendre les limites du RAG standard.

---

## 2.21. Synthèse du chapitre

Dans ce chapitre, nous avons introduit les embeddings.

Nous avons vu qu’un embedding est une représentation vectorielle d’un texte.

Nous pouvons résumer ainsi :

```text
Un embedding transforme un texte en vecteur numérique.
Deux textes proches en sens doivent avoir des vecteurs proches.
```

Nous avons compris que les embeddings permettent la recherche sémantique, c’est-à-dire la recherche par proximité de sens plutôt que seulement par mots-clés.

Nous avons aussi vu que les embeddings sont au cœur du RAG standard :

```text
documents → chunks → embeddings → index vectoriel
question → embedding → recherche des chunks proches
```

Mais nous avons insisté sur leurs limites :

```text
ils ne prouvent pas la vérité ;
ils ne garantissent pas le raisonnement ;
ils peuvent rater les faits intermédiaires ;
ils peuvent être moins bons sur les identifiants exacts ;
ils dépendent du modèle utilisé ;
ils doivent être complétés par des métadonnées, du lexical, du reranking ou du graphe.
```

Le message principal de ce chapitre est donc le suivant :

```text
Les embeddings sont le mécanisme qui permet au RAG de retrouver des passages proches en sens.
Mais retrouver un passage proche ne signifie pas toujours retrouver le passage nécessaire pour répondre correctement.
```

Dans le prochain chapitre, nous étudierons plus précisément la **similarité cosinus** et la recherche vectorielle, c’est-à-dire la manière dont nous comparons mathématiquement ces embeddings.

---

---
> [!info] Livre « RAG » — chapitre 2/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 01 — Pourquoi le RAG|← 01 — Pourquoi le RAG]] · [[RAG — 03 — Similarité cosinus et recherche vectorielle|03 — Similarité cosinus et recherche vectorielle →]]
