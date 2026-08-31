---
schema_version: 1
uid: 01M1BQ627916HB5566J7GCB3Q6
titre: "RAG — 06 — Améliorer le retrieval"
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
resume: "Chapitre 6 sur 12 du livre « RAG » : Améliorer le retrieval. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « RAG » — chapitre 6/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 05 — Prompt augmenté et génération sourcée|← 05 — Prompt augmenté et génération sourcée]] · [[RAG — 07 — Évaluer un système RAG|07 — Évaluer un système RAG →]]

# Chapitre 6 — Améliorer le retrieval
## 6.1. Introduction

Dans les chapitres précédents, nous avons construit les bases d’un système RAG standard.

Nous avons vu :

```text
documents
   ↓
chunks
   ↓
embeddings
   ↓
index vectoriel
   ↓
retrieval
   ↓
prompt augmenté
   ↓
réponse du LLM
```

Nous avons aussi compris une idée essentielle : la génération ne peut être fiable que si le contexte fourni au LLM est pertinent.

Autrement dit, si le système récupère les mauvais documents, le LLM risque de produire une réponse fausse, incomplète ou trop vague.

Le **retrieval** est donc le cœur du RAG.

Dans ce chapitre, nous allons nous concentrer sur une question centrale :

```text
Comment améliorer la récupération des bons passages avant la génération ?
```

Nous verrons que le retrieval ne se limite pas à une recherche vectorielle naïve. Un système robuste peut combiner plusieurs techniques :

```text
recherche vectorielle ;
recherche lexicale ;
recherche hybride ;
filtres par métadonnées ;
query rewriting ;
query expansion ;
multi-query retrieval ;
reranking ;
parent-child retrieval ;
contextual retrieval ;
retrieval multi-hop.
```

L’objectif n’est pas d’empiler toutes les techniques, mais de comprendre quel problème chacune résout.

---

## 6.2. Pourquoi le retrieval est critique

Dans un RAG, le LLM répond à partir des documents récupérés.

Nous pouvons donc avoir trois situations.

### 6.2.1. Les bons documents sont récupérés

Dans ce cas, le LLM peut produire une bonne réponse.

Exemple :

```text
Question :
Comment réinitialiser un mot de passe administrateur ?

Chunks récupérés :
- La réinitialisation nécessite une validation sécurité.
- Après validation, il faut utiliser reset-admin-password.sh.
```

Le modèle dispose des informations nécessaires.

### 6.2.2. Les documents récupérés sont proches mais insuffisants

Exemple :

```text
Question :
Le service checkout sera-t-il affecté par la maintenance de vendredi ?

Chunks récupérés :
- Le service checkout utilise l’API payments.
- Le cluster-3 sera en maintenance vendredi.
```

Il manque le lien :

```text
L’API payments est déployée sur le cluster-3.
```

Le LLM peut alors hésiter, inventer ou conclure trop vite.

### 6.2.3. Les documents récupérés sont mauvais

Exemple :

```text
Question :
Comment corriger l’erreur Prisma P2025 ?

Chunks récupérés :
- Introduction générale à Prisma.
- Comparaison entre ORM et requêtes SQL.
```

Les documents parlent du bon domaine, mais pas du problème précis.

Le LLM peut alors produire une réponse générique, voire fausse.

Nous devons donc retenir :

```text
La qualité du retrieval fixe un plafond à la qualité de la réponse.
```

Même un excellent LLM ne peut pas compenser durablement un mauvais retrieval.

---

## 6.3. Les deux objectifs du retrieval

Le retrieval doit équilibrer deux objectifs classiques en recherche d’information.

### 6.3.1. Le rappel

Le rappel consiste à récupérer tous les documents utiles.

Dans notre contexte :

```text
Avons-nous récupéré tous les passages nécessaires pour répondre ?
```

Exemple :

```text
checkout utilise payments API ;
payments API tourne sur cluster-3 ;
cluster-3 est en maintenance vendredi.
```

Pour répondre correctement, nous devons idéalement récupérer les trois faits.

Si nous en récupérons seulement deux, le rappel est insuffisant.

### 6.3.2. La précision

La précision consiste à éviter de récupérer des documents inutiles.

Dans notre contexte :

```text
Les passages récupérés sont-ils réellement utiles pour répondre ?
```

Si nous récupérons vingt chunks, dont trois utiles et dix-sept hors sujet, le LLM risque d’être perturbé.

Le bon retrieval cherche donc un compromis :

```text
récupérer assez large pour ne pas manquer l’information ;
récupérer assez précis pour ne pas noyer le modèle.
```

---

## 6.4. Le problème du top-k

Dans un RAG standard, on récupère souvent les `k` meilleurs résultats.

Exemple :

```text
top_k = 5
```

Cela signifie que nous donnons au LLM les cinq chunks les plus proches de la question.

Mais le choix de `k` est délicat.

### 6.4.1. Top-k trop faible

Si `k` est trop faible, nous risquons de manquer une information importante.

Exemple :

```text
Question :
Le service checkout sera-t-il affecté vendredi ?

Résultats :
1. checkout utilise payments API
2. cluster-3 est en maintenance vendredi
3. payments API tourne sur cluster-3
```

Avec :

```text
top_k = 2
```

nous ratons le troisième résultat, qui est indispensable.

### 6.4.2. Top-k trop élevé

Si `k` est trop élevé, nous récupérons trop de bruit.

Exemple :

```text
top_k = 30
```

Le LLM reçoit de nombreux passages, dont certains sont peu pertinents.

Cela peut produire :

```text
une réponse confuse ;
une mauvaise hiérarchisation de l’information ;
des contradictions inutiles ;
un coût plus élevé ;
une latence plus forte.
```

### 6.4.3. Top-k dynamique

Une amélioration consiste à adapter `k` selon la question.

Question simple :

```text
Quel est le délai de paiement ?
```

Un petit `k` peut suffire.

Question relationnelle :

```text
Quels services seront impactés par la maintenance du cluster-3 ?
```

Un `k` plus grand peut être nécessaire.

Nous pouvons donc imaginer un système qui classe d’abord la question :

```text
question factuelle simple → top_k faible
question large ou relationnelle → top_k plus élevé
question exploratoire → retrieval plus large + reranking
```

---

## 6.5. Les seuils de similarité

En plus du top-k, nous pouvons utiliser un seuil minimal.

Exemple :

```text
score >= 0.75
```

Cela permet de rejeter des chunks trop éloignés.

Mais ce mécanisme est délicat.

Un seuil trop haut peut supprimer des passages nécessaires.

Un seuil trop bas peut laisser entrer trop de bruit.

Il n’existe pas de seuil universel.

Un score dépend :

```text
du modèle d’embedding ;
du corpus ;
de la langue ;
du type de document ;
de la longueur des chunks ;
de la formulation des questions ;
de la métrique utilisée.
```

Nous devons donc calibrer les seuils empiriquement, sur un jeu de questions représentatif.

---

## 6.6. Recherche vectorielle : forces et limites

La recherche vectorielle est très utile, car elle permet de retrouver des passages proches en sens.

Elle est forte pour :

```text
les reformulations ;
les synonymes ;
les questions naturelles ;
les contenus explicatifs ;
les documents où les mots ne correspondent pas exactement.
```

Exemple :

```text
Question :
Je n’arrive plus à me connecter à mon compte.

Document :
Procédure de réinitialisation du mot de passe utilisateur.
```

La recherche vectorielle peut retrouver ce document, même si les mots sont différents.

Mais elle a des limites.

Elle peut être moins bonne pour :

```text
les identifiants exacts ;
les noms de fichiers ;
les codes d’erreur ;
les noms de classes ;
les acronymes rares ;
les références juridiques ;
les numéros de tickets ;
les versions logicielles.
```

Exemple :

```text
ERR_MODULE_NOT_FOUND
P2025
docker-compose.prod.yml
Article L.221-1
UserRepositoryImpl
```

Dans ces cas, une recherche lexicale peut être meilleure.

---

## 6.7. Recherche lexicale : BM25 et mots exacts

La recherche lexicale repose sur les mots présents dans la requête et les documents.

Un algorithme classique est **BM25**.

BM25 favorise les documents qui contiennent les termes de la requête, en tenant compte de leur fréquence et de leur rareté dans le corpus.

La recherche lexicale est très efficace lorsque les mots exacts sont importants.

Exemple :

```text
Question :
Que signifie l’erreur Prisma P2025 ?
```

Ici, `P2025` est un signal très fort.

Une recherche vectorielle pourrait retrouver des documents généraux sur Prisma, mais une recherche lexicale retrouvera plus directement les passages contenant `P2025`.

Autre exemple :

```text
Question :
Où est utilisé docker-compose.prod.yml ?
```

Le nom exact du fichier doit être conservé.

Nous devons donc éviter une opposition simpliste :

```text
recherche vectorielle = moderne
recherche lexicale = dépassée
```

En pratique, les deux sont complémentaires.

---

## 6.8. Recherche hybride

La recherche hybride combine recherche vectorielle et recherche lexicale.

L’idée est simple :

```text
recherche vectorielle → retrouve les passages proches en sens ;
recherche lexicale → retrouve les passages contenant les termes exacts.
```

Puis on fusionne les résultats.

Pipeline :

```text
Question
   ↓
Recherche vectorielle
   ↓
Résultats vectoriels

Question
   ↓
Recherche lexicale
   ↓
Résultats lexicaux

Fusion
   ↓
Liste de candidats
   ↓
Reranking éventuel
   ↓
Contexte final
```

### 6.8.1. Exemple

Question :

```text
Pourquoi mon build échoue avec ERR_MODULE_NOT_FOUND après compilation TypeScript ?
```

La recherche lexicale retrouve les documents contenant :

```text
ERR_MODULE_NOT_FOUND
TypeScript
build
```

La recherche vectorielle retrouve aussi des passages parlant de :

```text
imports cassés ;
modules introuvables ;
chemins relatifs après compilation ;
ESM ;
dist ;
tsconfig.
```

La combinaison des deux est souvent plus robuste.

### 6.8.2. Fusion des scores

Fusionner les résultats n’est pas toujours trivial.

Les scores vectoriels et lexicaux ne sont pas sur la même échelle.

On peut utiliser différentes stratégies :

```text
prendre l’union des résultats ;
normaliser les scores ;
pondérer lexical et vectoriel ;
utiliser Reciprocal Rank Fusion ;
envoyer les candidats à un reranker.
```

L’idée importante est que la recherche hybride augmente les chances de récupérer à la fois :

```text
les passages exacts ;
les passages sémantiquement proches.
```

---

## 6.9. Filtres par métadonnées

Les métadonnées sont indispensables pour améliorer le retrieval.

Elles permettent de limiter la recherche à un périmètre pertinent.

Exemples de métadonnées :

```text
type_document ;
langue ;
date ;
version ;
environnement ;
service ;
auteur ;
statut ;
source ;
droits d’accès ;
projet ;
client ;
```

### 6.9.1. Exemple technique

Question :

```text
Comment redémarrer checkout en production ?
```

Sans filtre, le système peut récupérer :

```text
une procédure de développement ;
une ancienne documentation ;
un brouillon ;
un document de staging ;
une note d’incident obsolète.
```

Avec filtres :

```text
service = checkout
environment = production
type = runbook
status = validé
```

nous réduisons fortement le bruit.

### 6.2. Exemple administratif

Question :

```text
Quelle est la procédure actuelle pour demander cette aide ?
```

Filtres utiles :

```text
date_version récente ;
document actif ;
territoire concerné ;
type = procédure ;
statut = validé.
```

La recherche sémantique seule peut retrouver un ancien document très proche. Les métadonnées permettent d’éviter cela.

---

## 6.10. Query rewriting

La question utilisateur n’est pas toujours formulée dans les meilleurs termes pour la recherche.

Exemple :

```text
Ça casse vendredi ?
```

Cette question est compréhensible dans une conversation, mais mauvaise pour un moteur de recherche.

Un LLM peut reformuler la question en requête plus explicite :

```text
Quels services seront affectés par la maintenance prévue vendredi ?
```

C’est le **query rewriting**.

### 6.10.1. Objectif

Nous ne cherchons pas à changer l’intention de l’utilisateur.

Nous cherchons à produire une requête plus adaptée au retrieval.

Exemples :

```text
"ça plante au build" 
→ "erreur de build après compilation TypeScript"

"je ne peux plus me connecter"
→ "problème de connexion compte utilisateur réinitialisation mot de passe"

"est-ce qu’on est impactés vendredi ?"
→ "impact de la maintenance vendredi sur les services dépendants"
```

### 6.10.2. Risque

Le query rewriting peut aussi déformer la question.

Exemple :

```text
Question :
Le service checkout sera-t-il ralenti vendredi ?

Mauvaise reformulation :
Le service checkout sera-t-il indisponible vendredi ?
```

Le sens a changé.

Nous devons donc contrôler cette étape.

Une bonne reformulation doit préserver :

```text
les entités ;
les contraintes ;
le niveau d’incertitude ;
la temporalité ;
le périmètre ;
l’intention.
```

---

## 6.11. Query expansion

La **query expansion** consiste à ajouter des termes proches ou liés à la question.

Exemple :

```text
Question :
Comment changer mon mot de passe ?
```

Expansion possible :

```text
mot de passe ;
password ;
identifiants ;
connexion ;
authentification ;
réinitialisation ;
compte utilisateur.
```

Cela peut améliorer la recherche, notamment lexicale.

Dans un contexte technique :

```text
Question :
Erreur module introuvable après build.
```

Expansion :

```text
ERR_MODULE_NOT_FOUND ;
module not found ;
ESM ;
import ;
dist ;
tsconfig ;
path alias ;
compilation TypeScript.
```

### 6.11.1. Intérêt

La query expansion aide lorsque les documents utilisent un vocabulaire différent de celui de l’utilisateur.

### 6.11.2. Risque

Elle peut élargir trop fortement la recherche et récupérer du bruit.

Exemple :

```text
"connexion"
```

peut ramener des documents sur :

```text
connexion utilisateur ;
connexion base de données ;
connexion réseau ;
connexion SSH ;
connexion OAuth.
```

Il faut donc garder le contexte.

---

## 6.12. Multi-query retrieval

Le **multi-query retrieval** consiste à générer plusieurs variantes de la question, puis à lancer plusieurs recherches.

Exemple :

Question originale :

```text
Le checkout sera-t-il affecté vendredi ?
```

Variantes :

```text
Quels services dépendent de la maintenance de vendredi ?
Le service checkout dépend-il d’un composant en maintenance ?
Quelle est la chaîne de dépendance entre checkout et les clusters maintenus ?
Maintenance vendredi impact service checkout.
```

Chaque requête peut récupérer des passages différents.

On fusionne ensuite les résultats.

### 6.12.1. Avantage

Cela augmente le rappel.

Nous avons plus de chances de récupérer des documents complémentaires.

### 6.12.2. Limite

Cela augmente :

```text
le coût ;
la latence ;
le bruit ;
la complexité de fusion.
```

Le multi-query retrieval est donc utile pour des questions complexes, mais pas forcément nécessaire pour toutes les requêtes.

---

## 6.13. Reranking

Le **reranking** est une étape très importante.

L’idée est de séparer deux tâches :

```text
retrouver rapidement des candidats ;
trier finement ces candidats.
```

Le premier retriever peut récupérer largement :

```text
top_k = 30
```

Puis un reranker sélectionne les meilleurs passages :

```text
top_n = 5
```

Pipeline :

```text
Question
   ↓
Retriever rapide
   ↓
30 chunks candidats
   ↓
Reranker
   ↓
5 chunks les plus pertinents
   ↓
LLM
```

### 6.13.1. Pourquoi le reranking est utile

La recherche vectorielle classe les documents selon une proximité globale.

Un reranker peut comparer plus finement la question et chaque passage.

Il peut mieux distinguer :

```text
un passage qui parle du même sujet ;
un passage qui répond réellement à la question.
```

Exemple :

```text
Question :
Comment corriger Prisma P2025 ?

Chunk A :
Prisma est un ORM moderne pour TypeScript.

Chunk B :
L’erreur P2025 indique qu’un enregistrement attendu n’a pas été trouvé.

Chunk A parle de Prisma.
Chunk B répond à la question.
```

Le reranker doit favoriser le chunk B.

### 6.13.2. Coût

Un reranker est souvent plus coûteux qu’un simple calcul de similarité vectorielle.

C’est pourquoi on ne l’applique pas à tout le corpus, mais seulement à une liste de candidats.

---

## 6.14. Parent-child retrieval

Le découpage en chunks crée un problème : un petit chunk peut être très précis, mais manquer de contexte.

Le **parent-child retrieval** résout cela en séparant deux niveaux :

```text
child chunks : petits morceaux utilisés pour la recherche ;
parent chunks : blocs plus larges fournis au LLM.
```

Exemple :

```text
Document
  Section
    Paragraphe 1
    Paragraphe 2
    Paragraphe 3
```

On indexe les paragraphes pour avoir une recherche précise.

Mais si un paragraphe est retrouvé, on fournit au LLM toute la section.

### 6.14.1. Avantage

Nous obtenons :

```text
précision au retrieval ;
contexte plus riche à la génération.
```

### 6.14.2. Exemple

Question :

```text
Quelles précautions prendre avant de redémarrer checkout ?
```

Le chunk enfant retrouvé :

```text
Ne pas redémarrer le service pendant une synchronisation de paiement.
```

Le parent fournit le contexte :

```text
Section : Redémarrage du service checkout
- conditions préalables ;
- vérifications ;
- risques ;
- procédure.
```

Le LLM peut alors répondre plus correctement.

---

## 6.15. Contextual retrieval

Le **contextual retrieval** consiste à enrichir les chunks avec un contexte avant indexation.

Un chunk brut peut être ambigu :

```text
Il ne doit pas être redémarré pendant cette période.
```

Nous pouvons l’enrichir :

```text
Document : Runbook Checkout Production
Section : Maintenance du service checkout
Chunk :
Il ne doit pas être redémarré pendant cette période.
```

Le texte indexé devient :

```text
Dans le Runbook Checkout Production, section Maintenance du service checkout :
il ne doit pas être redémarré pendant cette période.
```

Cela aide l’embedding à mieux représenter le sens du chunk.

### 6.15.1. Avantage

Le chunk devient plus compréhensible seul.

### 6.15.2. Limite

Si nous ajoutons trop de contexte répétitif, nous polluons les embeddings.

Il faut donc enrichir avec sobriété :

```text
titre du document ;
section ;
entité principale ;
date ou version si utile ;
périmètre métier.
```

---

## 6.16. Retrieval avec voisinage documentaire

Parfois, le chunk retrouvé ne suffit pas, mais ses voisins sont utiles.

Exemple :

```text
chunk_12 :
Après validation, exécuter le script reset-admin-password.sh.
```

Le chunk précédent contient :

```text
chunk_11 :
La réinitialisation d’un compte administrateur nécessite une validation sécurité.
```

Si nous récupérons seulement `chunk_12`, nous ratons une condition importante.

Une stratégie consiste donc à récupérer :

```text
le chunk trouvé ;
le chunk précédent ;
le chunk suivant.
```

Cela s’appelle parfois **window retrieval** ou récupération par fenêtre.

### 6.16.1. Avantage

Nous restaurons du contexte local.

### 6.16.2. Limite

Nous augmentons la quantité de texte injectée.

Il faut donc contrôler la taille de la fenêtre.

---

## 6.17. Retrieval multi-hop

Certaines questions nécessitent plusieurs étapes.

Exemple :

```text
Le checkout sera-t-il affecté vendredi ?
```

Pour répondre, il faut relier :

```text
checkout → payments API → cluster-3 → maintenance vendredi
```

Une recherche vectorielle simple peut rater le fait intermédiaire.

Le retrieval multi-hop consiste à faire plusieurs recherches successives.

### 6.17.1. Exemple

Première recherche :

```text
checkout service dependencies
```

Résultat :

```text
checkout utilise payments API.
```

Deuxième recherche à partir du résultat :

```text
payments API infrastructure deployment
```

Résultat :

```text
payments API tourne sur cluster-3.
```

Troisième recherche :

```text
cluster-3 maintenance
```

Résultat :

```text
cluster-3 est en maintenance vendredi.
```

Le système construit progressivement le chemin.

### 6.17.2. Limite

Le retrieval multi-hop est plus complexe.

Il nécessite :

```text
une stratégie de planification ;
une mémoire intermédiaire ;
une condition d’arrêt ;
une fusion des résultats ;
une gestion des erreurs.
```

C’est un pont vers Graph RAG et Agentic RAG.

---

## 6.18. Diversifier les résultats

Un problème fréquent est la redondance.

Le retriever peut récupérer cinq chunks presque identiques.

Exemple :

```text
chunk 1 : cluster-3 maintenance vendredi
chunk 2 : maintenance cluster-3 vendredi soir
chunk 3 : vendredi maintenance du cluster-3
```

Cela donne une impression de pertinence, mais n’apporte pas de nouvelles informations.

Nous pouvons améliorer cela en diversifiant les résultats.

Objectif :

```text
récupérer des passages pertinents mais non redondants.
```

On peut favoriser :

```text
des documents différents ;
des sections différentes ;
des types de sources différents ;
des étapes différentes du raisonnement.
```

Pour une question multi-hop, il vaut mieux récupérer :

```text
un chunk sur checkout ;
un chunk sur payments ;
un chunk sur cluster-3 ;
un chunk sur la maintenance ;
```

que quatre chunks disant la même chose sur la maintenance.

---

## 6.19. Déduplication

La déduplication consiste à supprimer les résultats trop similaires entre eux.

Elle peut se faire :

```text
par identifiant de document ;
par similarité textuelle ;
par hash ;
par proximité d’embedding ;
par regroupement de chunks voisins.
```

Elle est particulièrement utile lorsque :

```text
les documents sont dupliqués ;
plusieurs versions coexistent ;
les chunks se chevauchent fortement ;
l’overlap est important ;
les mêmes informations apparaissent dans plusieurs exports.
```

La déduplication réduit le bruit et économise des tokens.

---

## 6.20. Gestion des documents obsolètes

Un bon retrieval doit éviter de privilégier des documents anciens lorsque des documents plus récents existent.

Exemple :

```text
Document 2024 :
checkout tourne sur cluster-1.

Document 2026 :
checkout tourne sur cluster-3.
```

Les deux peuvent être proches de la question :

```text
Sur quel cluster tourne checkout ?
```

Nous devons utiliser les métadonnées :

```text
date ;
version ;
statut ;
document actif ou archivé ;
document validé ou brouillon.
```

Stratégies possibles :

```text
filtrer les documents archivés ;
favoriser les documents récents ;
signaler les contradictions ;
conserver l’historique mais ne pas le prioriser ;
utiliser une source canonique.
```

La fraîcheur des sources est une dimension essentielle du retrieval.

---

## 6.21. Gestion des permissions

Le retrieval doit respecter les droits d’accès.

La règle est simple :

```text
un utilisateur ne doit récupérer que les chunks qu’il a le droit de consulter.
```

Il ne faut pas récupérer tous les documents puis demander au LLM de cacher ce qui est confidentiel.

Les documents non autorisés ne doivent pas entrer dans le prompt.

Les permissions peuvent être appliquées :

```text
au niveau de l’index ;
au niveau des filtres ;
au niveau des métadonnées ;
par séparation des corpus ;
par contrôle applicatif avant génération.
```

C’est un point critique pour les systèmes d’entreprise.

---

## 6.22. Retrieval orienté code source

Pour du code source, la recherche vectorielle seule est rarement suffisante.

Un développeur peut demander :

```text
Où est appelée la fonction sendInvoiceEmail ?
```

Un embedding peut aider, mais la réponse dépend surtout :

```text
des symboles ;
des appels de fonctions ;
des imports ;
des types ;
de l’arbre syntaxique ;
du graphe d’appels.
```

Pour du code, nous devons souvent combiner :

```text
recherche lexicale ;
indexation des symboles ;
LSP ;
AST ;
graphe d’appels ;
analyse statique ;
embeddings de code ;
métadonnées Git.
```

### 6.22.1. Exemple

Question :

```text
Quel code est responsable de l’erreur ERR_MODULE_NOT_FOUND après build ?
```

Recherche utile :

```text
ERR_MODULE_NOT_FOUND en lexical ;
imports TypeScript ;
fichiers tsconfig ;
scripts de build ;
chemins dist ;
extensions .js manquantes ;
historique Git récent.
```

La recherche vectorielle peut retrouver des explications générales, mais l’analyse statique permet de trouver les références exactes.

---

## 6.23. Retrieval orienté documents juridiques ou administratifs

Dans un corpus juridique ou administratif, les contraintes sont différentes.

Les mots exacts, les articles, les dates et les versions comptent beaucoup.

Exemples :

```text
article L.221-1 ;
décret du 12 mars 2024 ;
clause 7.2 ;
avenant n°3 ;
version consolidée ;
barème 2026.
```

Une recherche vectorielle peut aider à trouver des passages sémantiquement liés, mais elle doit être complétée par :

```text
recherche exacte ;
métadonnées temporelles ;
hiérarchie documentaire ;
versions ;
citations précises ;
contrôle des sources officielles.
```

Dans ce contexte, une réponse non sourcée est insuffisante.

---

## 6.24. Retrieval orienté PDF et documents complexes

Les PDF posent des problèmes particuliers.

Une extraction incorrecte peut casser le retrieval.

Exemples :

```text
tableaux mal linéarisés ;
colonnes mélangées ;
titres séparés du contenu ;
notes de bas de page confondues avec le texte ;
schémas ignorés ;
OCR bruité.
```

Améliorer le retrieval sur PDF suppose souvent d’améliorer d’abord l’ingestion :

```text
conserver les numéros de page ;
préserver les titres ;
extraire les tableaux proprement ;
ajouter des métadonnées de position ;
associer images et légendes ;
utiliser OCR ou modèles multimodaux si nécessaire.
```

Le retrieval ne peut pas compenser entièrement une mauvaise extraction.

---

## 6.25. Évaluer les améliorations du retrieval

Nous ne devons pas améliorer le retrieval uniquement “à l’intuition”.

Chaque amélioration doit être évaluée.

Exemple de questions :

```text
Le passage attendu est-il dans le top 3 ?
Est-il dans le top 5 ?
Les résultats sont-ils redondants ?
Les documents obsolètes sont-ils écartés ?
Les permissions sont-elles respectées ?
La recherche hybride améliore-t-elle vraiment le rappel ?
Le reranker améliore-t-il la précision ?
```

Nous pouvons mesurer :

```text
recall@k ;
precision@k ;
MRR ;
nDCG ;
taux de récupération de la source attendue ;
taux de bruit ;
latence ;
coût.
```

Nous détaillerons ces métriques dans le chapitre consacré à l’évaluation.

Pour l’instant, retenons ceci :

```text
une amélioration du retrieval doit être mesurée sur des cas représentatifs.
```

---

## 6.26. Exemple complet : amélioration progressive

Prenons un corpus :

```text
Doc 1 :
Le service checkout utilise l’API payments.

Doc 2 :
L’API payments est déployée sur le cluster-3.

Doc 3 :
Le cluster-3 sera en maintenance vendredi soir.

Doc 4 :
Le service checkout expose une API REST pour le panier.

Doc 5 :
Ancienne documentation : checkout était déployé sur cluster-1 en 2023.
```

Question :

```text
Le service checkout sera-t-il affecté vendredi ?
```

### 6.26.1. Retrieval vectoriel simple

Résultats :

```text
1. Doc 1 — checkout utilise payments
2. Doc 4 — checkout expose une API REST
3. Doc 3 — cluster-3 maintenance vendredi
```

Problème :

```text
Doc 2 manque.
Doc 4 est proche mais peu utile.
```

### 6.26.2. Avec top-k plus grand

Résultats :

```text
1. Doc 1
2. Doc 4
3. Doc 3
4. Doc 2
5. Doc 5
```

Doc 2 apparaît, mais Doc 5 ajoute un document obsolète.

### 6.26.3. Avec métadonnées

Filtre :

```text
status = actif
```

Doc 5 est écarté.

### 6.26.4. Avec diversification

On évite de récupérer trop de chunks sur checkout uniquement.

On favorise :

```text
dépendance ;
infrastructure ;
maintenance.
```

### 6.26.5. Avec multi-hop

Le système suit :

```text
checkout → payments API → cluster-3 → maintenance
```

Résultat final :

```text
Doc 1
Doc 2
Doc 3
```

Le contexte devient suffisant pour répondre.

---

## 6.27. Pseudo-code d’un retrieval hybride avec reranking

Voici un pseudo-code simplifié.

```python
def retrieve(question, user, filters):
    # 1. Recherche vectorielle
    vector_results = vector_index.search(
        query=question,
        top_k=30,
        filters={
            **filters,
            "permissions": user.permissions
        }
    )

    # 2. Recherche lexicale
    lexical_results = bm25_index.search(
        query=question,
        top_k=30,
        filters={
            **filters,
            "permissions": user.permissions
        }
    )

    # 3. Fusion
    candidates = merge_results(
        vector_results,
        lexical_results
    )

    # 4. Déduplication
    candidates = deduplicate(candidates)

    # 5. Reranking
    reranked = reranker.rank(
        query=question,
        documents=candidates
    )

    # 6. Sélection finale
    final_chunks = reranked[:5]

    return final_chunks
```

Ce pseudo-code montre une architecture plus robuste qu’un simple :

```text
embedding(question) → top_k chunks
```

---

## 6.28. Mini-TP : comparer vectoriel, lexical et hybride

### Objectif

Nous voulons observer les différences entre trois stratégies :

```text
recherche vectorielle ;
recherche lexicale ;
recherche hybride.
```

### Corpus

```text
doc_1 : La procédure de réinitialisation du mot de passe nécessite une validation sécurité.
doc_2 : Pour changer vos identifiants, utilisez le portail utilisateur.
doc_3 : L’erreur ERR_MODULE_NOT_FOUND vient souvent d’un import invalide après build.
doc_4 : Le cluster-3 sera en maintenance vendredi soir.
doc_5 : Le service checkout utilise l’API payments.
doc_6 : L’API payments est déployée sur le cluster-3.
```

### Questions

```text
Je n’arrive plus à me connecter.
Que signifie ERR_MODULE_NOT_FOUND ?
Le service checkout sera-t-il affecté vendredi ?
```

### Analyse attendue

Pour :

```text
Je n’arrive plus à me connecter.
```

la recherche vectorielle peut être très utile.

Pour :

```text
ERR_MODULE_NOT_FOUND
```

la recherche lexicale est probablement indispensable.

Pour :

```text
checkout affecté vendredi
```

il faut plusieurs documents, donc une stratégie plus large ou multi-hop peut être nécessaire.

---

## 6.29. Mini-TP : tester l’effet du top-k

### Objectif

Nous voulons comprendre comment le choix de `top_k` influence la réponse.

### Corpus

```text
doc_1 : Le service checkout utilise l’API payments.
doc_2 : L’API payments est déployée sur le cluster-3.
doc_3 : Le cluster-3 sera en maintenance vendredi.
doc_4 : Le service checkout expose une API REST.
doc_5 : Les factures sont payables sous trente jours.
```

### Question

```text
Le service checkout sera-t-il affecté vendredi ?
```

### Travail

Nous testons :

```text
top_k = 1
top_k = 2
top_k = 3
top_k = 5
```

Nous observons :

```text
à partir de quel k les trois documents nécessaires sont récupérés ;
combien de bruit est ajouté ;
si le LLM répond avec certitude ou prudence ;
si la réponse est correctement justifiée.
```

---

## 6.30. Questions de compréhension

À la fin de ce chapitre, nous devons pouvoir répondre aux questions suivantes :

```text
Pourquoi le retrieval est-il central dans un système RAG ?
Quelle est la différence entre rappel et précision ?
Pourquoi le choix de top-k est-il important ?
Pourquoi un seuil de similarité fixe peut-il être dangereux ?
Dans quels cas la recherche lexicale est-elle meilleure que la recherche vectorielle ?
Pourquoi la recherche hybride est-elle souvent plus robuste ?
À quoi servent les métadonnées dans le retrieval ?
Qu’est-ce que le query rewriting ?
Quelle différence entre query rewriting et query expansion ?
Qu’est-ce que le multi-query retrieval ?
À quoi sert un reranker ?
Pourquoi récupérer les chunks voisins peut-il améliorer la réponse ?
Qu’est-ce que le parent-child retrieval ?
Pourquoi le retrieval multi-hop est-il nécessaire pour certaines questions ?
Comment gérer les documents obsolètes ?
Pourquoi les permissions doivent-elles être appliquées avant le prompt ?
```

---

## 6.31. Synthèse du chapitre

Dans ce chapitre, nous avons étudié comment améliorer le retrieval dans un système RAG.

Nous avons vu qu’un retrieval naïf basé uniquement sur la recherche vectorielle peut fonctionner pour des questions simples, mais échouer sur des cas plus réalistes.

Nous avons identifié plusieurs causes d’échec :

```text
top-k trop faible ;
chunks trop petits ou trop grands ;
documents obsolètes ;
absence de métadonnées ;
termes exacts mal récupérés ;
faits intermédiaires manquants ;
résultats redondants ;
documents non autorisés ;
requêtes utilisateur trop vagues.
```

Nous avons ensuite étudié plusieurs familles de solutions :

```text
recherche lexicale ;
recherche hybride ;
filtres par métadonnées ;
query rewriting ;
query expansion ;
multi-query retrieval ;
reranking ;
parent-child retrieval ;
contextual retrieval ;
récupération des chunks voisins ;
retrieval multi-hop ;
diversification ;
déduplication.
```

Le message principal du chapitre est le suivant :

```text
Améliorer le retrieval, ce n’est pas seulement chercher plus de documents.
C’est chercher les bons documents, dans le bon périmètre,
avec assez de contexte, peu de bruit, et des sources vérifiables.
```

Dans le chapitre suivant, nous étudierons l’évaluation du RAG. Nous apprendrons à mesurer séparément la qualité du retrieval, la qualité de la génération et la fidélité de la réponse aux sources.

---

---
> [!info] Livre « RAG » — chapitre 6/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 05 — Prompt augmenté et génération sourcée|← 05 — Prompt augmenté et génération sourcée]] · [[RAG — 07 — Évaluer un système RAG|07 — Évaluer un système RAG →]]
