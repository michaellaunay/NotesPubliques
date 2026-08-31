---
schema_version: 1
uid: 01M1BQ627CDMBCSN0VCBDX3YMH
titre: "RAG — 09 — Graph RAG entités, relations et requêtes multi-hop"
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
resume: "Chapitre 9 sur 12 du livre « RAG » : Graph RAG : entités, relations et requêtes multi-hop. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « RAG » — chapitre 9/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 08 — Optimisation hybride, reranking et query rewriting|← 08 — Optimisation hybride, reranking et query rewriting]] · [[RAG — 10 — Agentic RAG agents, outils et orchestration dynamique|10 — Agentic RAG agents, outils et orchestration dynamique →]]

# Chapitre 9 — Graph RAG : entités, relations et requêtes multi-hop
## 9.1. Introduction

Dans les chapitres précédents, nous avons construit un système RAG standard, puis nous avons étudié ses limites et ses optimisations.

Nous avons vu qu’un RAG vectoriel fonctionne très bien lorsqu’une question peut être résolue en retrouvant un ou quelques passages proches de la requête.

Exemple :

```text
Question :
Quand aura lieu la maintenance du cluster-3 ?

Document :
Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.
```

Dans ce cas, la question et le document sont proches. La recherche vectorielle a de bonnes chances de récupérer le bon passage.

Mais certains problèmes sont plus complexes.

Ils ne demandent pas seulement de trouver un passage similaire. Ils demandent de relier plusieurs informations dispersées dans plusieurs documents.

Exemple :

```text
Le service checkout sera-t-il affecté par la maintenance de vendredi ?
```

Pour répondre, nous devons construire une chaîne :

```text
checkout
→ utilise payments API
→ payments API tourne sur cluster-3
→ cluster-3 est en maintenance vendredi
→ donc checkout peut être affecté
```

Un RAG vectoriel standard peut rater un fait intermédiaire, car ce fait ne ressemble pas directement à la question.

Le **Graph RAG** vise à résoudre ce type de problème en ajoutant une couche de représentation relationnelle.

Au lieu de stocker uniquement des morceaux de texte et leurs embeddings, nous allons aussi représenter :

```text
des entités ;
des relations ;
des chemins ;
des sous-graphes ;
des communautés ;
des dépendances.
```

L’objectif du chapitre est de comprendre comment un graphe de connaissances peut compléter le RAG vectoriel.

---

## 9.2. Limite du RAG vectoriel sur les questions relationnelles

Reprenons notre exemple.

Corpus :

```text
Document 1 :
Le service checkout utilise l’API payments.

Document 2 :
L’API payments est déployée sur le cluster-3.

Document 3 :
Le cluster-3 sera en maintenance vendredi soir.
```

Question :

```text
Le service checkout sera-t-il affecté par la maintenance de vendredi ?
```

Le RAG vectoriel calcule l’embedding de la question et cherche les chunks les plus proches.

Il peut facilement retrouver :

```text
Document 1 :
Le service checkout utilise l’API payments.
```

car la question contient `checkout`.

Il peut aussi retrouver :

```text
Document 3 :
Le cluster-3 sera en maintenance vendredi soir.
```

car la question contient l’idée de maintenance vendredi.

Mais il peut rater :

```text
Document 2 :
L’API payments est déployée sur le cluster-3.
```

Pourquoi ?

Parce que ce document ne contient ni :

```text
checkout ;
maintenance ;
vendredi ;
affecté.
```

Pourtant, ce document est indispensable. C’est lui qui relie `payments API` à `cluster-3`.

Nous avons donc un problème important :

```text
un passage peut être indispensable pour raisonner,
même s’il n’est pas directement similaire à la question.
```

Le Graph RAG cherche précisément à représenter ces liens.

---

## 9.3. Qu’est-ce qu’un graphe ?

Un graphe est une structure composée de :

```text
nœuds ;
arêtes.
```

Les nœuds représentent des objets.

Les arêtes représentent des relations entre ces objets.

Dans notre exemple :

```text
Nœuds :
- checkout service
- payments API
- cluster-3
- maintenance vendredi

Arêtes :
- checkout service utilise payments API
- payments API est déployée sur cluster-3
- cluster-3 a une maintenance vendredi
```

Nous pouvons représenter cela ainsi :

```text
checkout service
      |
    utilise
      |
payments API
      |
 est déployée sur
      |
cluster-3
      |
 a une maintenance
      |
vendredi
```

Cette structure rend explicite ce que le RAG vectoriel laisse implicite.

Le système peut maintenant suivre un chemin :

```text
checkout service → payments API → cluster-3 → maintenance vendredi
```

---

## 9.4. Graphe de connaissances

Dans le contexte du RAG, nous parlons souvent de **graphe de connaissances**, ou _knowledge graph_.

Un graphe de connaissances représente des informations sous forme d’entités et de relations.

Une information textuelle comme :

```text
Le service checkout utilise l’API payments.
```

peut être convertie en triplet :

```text
(checkout service) --utilise--> (payments API)
```

Un autre exemple :

```text
L’API payments est déployée sur le cluster-3.
```

devient :

```text
(payments API) --est_déployée_sur--> (cluster-3)
```

Un troisième :

```text
Le cluster-3 sera en maintenance vendredi.
```

devient :

```text
(cluster-3) --a_maintenance--> (vendredi)
```

Nous obtenons alors un graphe interrogeable.

---

## 9.5. Les triplets sujet-relation-objet

Une manière simple de représenter les relations consiste à utiliser des triplets :

```text
sujet — relation — objet
```

Exemples :

```text
checkout service — utilise — payments API

payments API — est déployée sur — cluster-3

cluster-3 — a une maintenance — vendredi

Redis — est utilisé par — worker notification

worker notification — publie dans — queue email

API billing — dépend de — PostgreSQL
```

Ces triplets sont plus structurés que du texte libre.

Ils permettent de poser des questions comme :

```text
Quels services dépendent de Redis ?
Quels composants tournent sur cluster-3 ?
Quels services sont affectés par une maintenance de cluster-3 ?
Quelles APIs sont utilisées par checkout ?
```

Le graphe permet donc de naviguer dans les dépendances.

---

## 9.6. Indexation dans un Graph RAG

Dans un RAG standard, l’indexation ressemble à ceci :

```text
documents
   ↓
chunks
   ↓
embeddings
   ↓
index vectoriel
```

Dans un Graph RAG, nous ajoutons une étape :

```text
documents
   ↓
chunks
   ↓
extraction d’entités et de relations
   ↓
graphe de connaissances
   ↓
embeddings et index vectoriel
```

Le système peut donc stocker deux types de mémoire :

```text
mémoire textuelle vectorielle ;
mémoire relationnelle graphique.
```

La première sert à retrouver des passages proches.

La seconde sert à retrouver des liens entre entités.

---

## 9.7. Extraction d’entités

La première étape consiste à identifier les entités importantes dans les documents.

Exemples d’entités techniques :

```text
services ;
APIs ;
bases de données ;
clusters ;
queues ;
workers ;
équipes ;
environnements ;
dépôts Git ;
fichiers ;
fonctions ;
incidents ;
dates ;
versions.
```

Dans une phrase :

```text
L’API payments est déployée sur le cluster-3.
```

nous extrayons :

```text
API payments ;
cluster-3.
```

Dans :

```text
Le worker notification consomme la queue email.
```

nous extrayons :

```text
worker notification ;
queue email.
```

L’extraction peut être faite :

```text
par règles ;
par reconnaissance d’entités nommées ;
par LLM ;
par analyse statique pour le code ;
par parsing de fichiers structurés ;
par combinaison de plusieurs méthodes.
```

---

## 9.8. Extraction de relations

Une fois les entités identifiées, nous devons extraire les relations.

Exemple :

```text
Le service checkout utilise l’API payments.
```

Entités :

```text
checkout service ;
payments API.
```

Relation :

```text
utilise.
```

Triplet :

```text
checkout service — utilise — payments API
```

Autres exemples :

```text
payments API — est déployée sur — cluster-3
cluster-3 — a une maintenance — vendredi
service billing — dépend de — PostgreSQL
worker invoice — consomme — queue invoice
```

Les relations peuvent être typées.

Exemples :

```text
uses ;
depends_on ;
runs_on ;
owned_by ;
calls ;
reads_from ;
writes_to ;
deployed_on ;
scheduled_for ;
part_of ;
replaces ;
deprecated_by.
```

Le choix des types de relations dépend du domaine.

Pour une architecture logicielle, les relations seront différentes de celles d’un corpus juridique ou scientifique.

---

## 9.9. Normalisation des entités

Un problème difficile est la normalisation.

Dans les documents, une même entité peut être nommée de plusieurs façons.

Exemple :

```text
payments API ;
API payments ;
payment-api ;
PaymentService ;
service de paiement ;
payments-service.
```

Ces noms peuvent désigner la même entité.

Si le système les traite comme cinq nœuds différents, le graphe sera fragmenté.

Nous devons donc faire de la résolution d’entités.

Objectif :

```text
identifier quand plusieurs mentions désignent le même objet réel.
```

Cela peut se faire avec :

```text
règles de normalisation ;
dictionnaires ;
métadonnées ;
similarité de noms ;
LLM ;
validation humaine ;
identifiants techniques stables.
```

Dans un système technique, il est préférable d’utiliser des identifiants canoniques.

Exemple :

```text
service_id = payments-api
display_name = Payments API
aliases = ["payment-api", "API payments", "PaymentService"]
```

---

## 9.10. Construction du graphe

Une fois les entités et relations extraites, nous construisons le graphe.

Exemple :

```text
checkout service — uses — payments API
payments API — runs_on — cluster-3
cluster-3 — maintenance_on — Friday
```

Nous stockons chaque nœud avec des propriétés :

```json
{
  "id": "payments-api",
  "type": "service",
  "name": "Payments API",
  "aliases": ["API payments", "payment-api"],
  "environment": "production"
}
```

Et chaque arête avec des propriétés :

```json
{
  "source": "checkout-service",
  "relation": "uses",
  "target": "payments-api",
  "source_document": "doc_checkout_dependencies",
  "confidence": 0.91,
  "updated_at": "2026-05-12"
}
```

Ces propriétés sont importantes pour la traçabilité.

Nous devons savoir d’où vient une relation.

---

## 9.11. Graphe et sources documentaires

Un graphe ne doit pas faire disparaître les documents originaux.

Chaque relation doit idéalement pointer vers une source.

Exemple :

```text
Relation :
checkout service — uses — payments API

Source :
Runbook Checkout, section "Dépendances"
```

Pourquoi ?

Parce que le graphe est une représentation extraite. Il peut contenir des erreurs.

Pour qu’une réponse soit vérifiable, nous devons pouvoir revenir au texte source.

Le Graph RAG doit donc combiner :

```text
nœuds et relations ;
passages documentaires ;
métadonnées ;
citations.
```

Une réponse fondée uniquement sur un graphe sans source serait difficile à auditer.

---

## 9.12. Retrieval dans un Graph RAG

Dans un RAG vectoriel, le retrieval consiste à chercher les chunks proches de la question.

Dans un Graph RAG, le retrieval peut inclure :

```text
identifier les entités de la question ;
trouver les nœuds correspondants ;
parcourir les relations ;
extraire un sous-graphe pertinent ;
retrouver les documents associés ;
construire un contexte pour le LLM.
```

Pipeline :

```text
Question utilisateur
   ↓
Extraction des entités de la question
   ↓
Recherche des nœuds correspondants
   ↓
Parcours du graphe
   ↓
Sélection du sous-graphe pertinent
   ↓
Récupération des sources textuelles
   ↓
Prompt augmenté
   ↓
Réponse du LLM
```

---

## 9.13. Exemple de retrieval par graphe

Question :

```text
Le service checkout sera-t-il affecté par la maintenance de vendredi ?
```

Étape 1 : entités détectées.

```text
checkout service ;
maintenance vendredi.
```

Étape 2 : nœuds correspondants.

```text
checkout-service ;
maintenance-friday.
```

Étape 3 : parcours du graphe.

```text
checkout-service
  --uses-->
payments-api
  --runs_on-->
cluster-3
  --maintenance_on-->
friday
```

Étape 4 : récupération des sources.

```text
Source 1 :
checkout utilise payments API.

Source 2 :
payments API tourne sur cluster-3.

Source 3 :
cluster-3 est en maintenance vendredi.
```

Étape 5 : génération.

```text
Le service checkout peut être affecté vendredi, car il utilise payments API,
qui tourne sur cluster-3, et cluster-3 sera en maintenance vendredi.
```

Le graphe permet donc de récupérer le fait intermédiaire qui aurait pu être manqué par la recherche vectorielle.

---

## 9.14. Requêtes multi-hop

Une requête multi-hop est une question qui nécessite plusieurs étapes de liaison.

Exemple simple :

```text
A dépend de B.
B dépend de C.
C est affecté.
Donc A peut être affecté.
```

Dans notre cas :

```text
checkout → payments API → cluster-3 → maintenance vendredi
```

Chaque flèche est un “hop”.

Une question single-hop :

```text
Sur quel cluster tourne payments API ?
```

Réponse :

```text
payments API → cluster-3
```

Une question multi-hop :

```text
Quels services seront affectés par la maintenance de cluster-3 ?
```

Réponse :

```text
cluster-3
← payments API
← checkout service
```

Le Graph RAG est particulièrement adapté à ces questions.

---

## 9.15. Parcours de graphe

Le parcours de graphe consiste à explorer les relations autour d’un nœud.

Exemple :

```text
Point de départ :
cluster-3
```

Nous pouvons chercher :

```text
tous les services déployés sur cluster-3 ;
toutes les APIs utilisées par ces services ;
tous les clients dépendants de ces APIs ;
tous les incidents liés à cluster-3.
```

Le parcours peut être limité par :

```text
profondeur maximale ;
types de relations autorisés ;
direction des relations ;
filtres temporels ;
filtres de permissions ;
score de confiance ;
type d’entité.
```

Exemple :

```text
trouver tous les services à distance ≤ 2 de cluster-3
en suivant uniquement les relations "runs_on" et "uses".
```

---

## 9.16. Direction des relations

Dans un graphe, la direction des relations est importante.

Exemple :

```text
checkout service --uses--> payments API
```

Cela signifie :

```text
checkout dépend de payments.
```

Mais la relation inverse est différente :

```text
payments API --is_used_by--> checkout service
```

Pour répondre à certaines questions, nous devons suivre les relations dans un sens ou dans l’autre.

Question :

```text
De quoi dépend checkout ?
```

Nous suivons :

```text
checkout --uses--> ...
```

Question :

```text
Quels services dépendent de payments API ?
```

Nous suivons l’inverse :

```text
... --uses--> payments API
```

Le modèle de graphe doit donc être conçu soigneusement.

---

## 9.17. Types de relations et schéma du graphe

Un graphe peut être très libre, mais en production il est souvent préférable de définir un schéma.

Exemple de types de nœuds :

```text
Service ;
API ;
Database ;
Cluster ;
Queue ;
Team ;
Incident ;
Document ;
Environment ;
Version.
```

Exemple de types de relations :

```text
uses ;
calls ;
runs_on ;
owned_by ;
depends_on ;
reads_from ;
writes_to ;
deployed_in ;
has_incident ;
scheduled_for ;
documented_by.
```

Un schéma aide à :

```text
contrôler l’extraction ;
éviter les relations incohérentes ;
écrire des requêtes fiables ;
faciliter l’évaluation ;
expliquer le graphe aux utilisateurs.
```

Sans schéma, le graphe peut devenir difficile à maintenir.

---

## 9.18. Graph RAG et RAG vectoriel : complémentarité

Graph RAG ne remplace pas forcément le RAG vectoriel.

Les deux approches sont complémentaires.

La recherche vectorielle est bonne pour :

```text
retrouver des passages proches en sens ;
gérer les reformulations ;
rechercher dans du texte libre ;
retrouver des explications ;
trouver des sections pertinentes.
```

Le graphe est bon pour :

```text
suivre des relations ;
répondre à des questions de dépendance ;
faire du multi-hop ;
détecter des chemins ;
regrouper des entités ;
analyser des réseaux.
```

Une architecture réaliste peut donc combiner :

```text
vector search ;
BM25 ;
knowledge graph ;
reranker ;
LLM.
```

---

## 9.19. Exemple d’architecture hybride Graph RAG

Pipeline possible :

```text
Question utilisateur
   ↓
Classification de la question
   ↓
Extraction des entités
   ↓
Recherche vectorielle initiale
   ↓
Recherche dans le graphe
   ↓
Fusion des résultats
   ↓
Récupération des sources textuelles
   ↓
Reranking
   ↓
Prompt augmenté
   ↓
Réponse sourcée
```

Exemple :

```text
Question :
Quels services risquent d’être affectés par la maintenance de cluster-3 ?
```

Le graphe trouve :

```text
payments API runs_on cluster-3
checkout uses payments API
billing uses payments API
```

La recherche vectorielle retrouve :

```text
runbook payments ;
planning maintenance ;
documentation checkout ;
documentation billing.
```

Le LLM reçoit ensuite un contexte structuré.

---

## 9.20. Sous-graphe pertinent

Au lieu de donner tout le graphe au LLM, nous devons extraire un sous-graphe pertinent.

Exemple :

```text
Question :
Quels services sont impactés par cluster-3 ?
```

Sous-graphe utile :

```text
cluster-3
← payments API
← checkout
← billing
```

Sous-graphe inutile :

```text
cluster-7 ;
ancienne documentation ;
incidents non liés ;
services de staging.
```

Le sous-graphe doit être :

```text
assez complet pour répondre ;
assez petit pour être compréhensible ;
sourcé ;
filtré par permissions ;
filtré par fraîcheur.
```

---

## 9.21. Transformer un sous-graphe en contexte LLM

Un LLM ne consomme pas directement un graphe comme une base de données.

Nous devons transformer le sous-graphe en contexte textuel.

Exemple :

```text
Relations pertinentes :
1. checkout service utilise payments API. Source : doc_checkout_dependencies.
2. billing service utilise payments API. Source : doc_billing_dependencies.
3. payments API est déployée sur cluster-3. Source : doc_payments_deployment.
4. cluster-3 sera en maintenance vendredi. Source : doc_maintenance.
```

Ou sous forme structurée :

```json
{
  "nodes": [
    {"id": "checkout", "type": "service"},
    {"id": "billing", "type": "service"},
    {"id": "payments-api", "type": "api"},
    {"id": "cluster-3", "type": "cluster"}
  ],
  "edges": [
    {"from": "checkout", "relation": "uses", "to": "payments-api"},
    {"from": "billing", "relation": "uses", "to": "payments-api"},
    {"from": "payments-api", "relation": "runs_on", "to": "cluster-3"}
  ]
}
```

Le format dépend du cas d’usage.

Pour une réponse pédagogique, le texte est souvent plus lisible.

Pour une API, le JSON peut être utile.

---

## 9.22. Graph RAG global : résumer un corpus

Le Graph RAG ne sert pas seulement aux dépendances techniques.

Il peut aussi aider à répondre à des questions globales sur un corpus.

Exemple :

```text
Quels sont les grands thèmes qui ressortent des rapports d’incident ?
```

Ou :

```text
Quels acteurs reviennent souvent dans ces documents ?
```

Ou :

```text
Quels risques sont liés à plusieurs projets ?
```

Dans ce cas, le système peut :

```text
extraire des entités ;
regrouper des communautés ;
résumer les clusters du graphe ;
identifier des relations fréquentes.
```

Cela permet de produire des réponses qui ne dépendent pas seulement d’un passage isolé.

---

## 9.23. Communautés dans un graphe

Dans un grand graphe, certaines entités forment des groupes fortement connectés.

Exemple :

```text
checkout ;
payments API ;
billing ;
invoices ;
PostgreSQL payments ;
team finance.
```

Ces nœuds forment peut-être une communauté autour du domaine paiement.

Une autre communauté peut concerner :

```text
auth ;
users ;
sessions ;
OAuth ;
Redis ;
team identity.
```

Détecter ces communautés peut aider à :

```text
résumer un domaine ;
orienter la recherche ;
identifier les dépendances ;
structurer la documentation ;
répondre à des questions larges.
```

---

## 9.24. Exemple dans un projet logiciel

Imaginons un projet avec plusieurs services :

```text
frontend ;
api ;
auth-service ;
payments-api ;
billing-worker ;
notification-worker ;
Redis ;
MariaDB ;
MongoDB ;
cluster-prod ;
cluster-staging.
```

Relations :

```text
frontend calls api ;
api calls auth-service ;
api calls payments-api ;
payments-api writes_to MariaDB ;
billing-worker consumes queue billing ;
notification-worker consumes queue email ;
queues run_on Redis ;
payments-api runs_on cluster-prod.
```

Question :

```text
Quels composants peuvent être affectés si Redis tombe ?
```

Un RAG vectoriel peut retrouver des passages sur Redis.

Mais un graphe peut parcourir :

```text
Redis
← queue billing
← billing-worker

Redis
← queue email
← notification-worker
```

Réponse :

```text
Les workers billing et notification peuvent être affectés, car ils consomment
des queues hébergées sur Redis. Les fonctionnalités d’envoi d’email et de
traitement de facturation peuvent donc être impactées.
```

---

## 9.25. Exemple dans un corpus juridique ou administratif

Graph RAG peut aussi s’appliquer à des documents non techniques.

Entités :

```text
personnes ;
institutions ;
procédures ;
aides ;
conditions ;
documents justificatifs ;
dates ;
territoires ;
articles ;
décisions.
```

Relations :

```text
personne demande aide ;
aide nécessite justificatif ;
procédure dépend de territoire ;
article encadre décision ;
document remplace document précédent.
```

Question :

```text
Quelles conditions doivent être réunies pour demander cette aide ?
```

Le graphe peut relier :

```text
aide → conditions → justificatifs → organisme → procédure.
```

Il faut cependant être très prudent dans les domaines juridiques ou administratifs, car les relations doivent être fidèles aux textes et aux versions applicables.

---

## 9.26. Graph RAG sur du code source

Pour le code, un graphe est souvent naturel.

Nous pouvons construire :

```text
graphe d’appels ;
graphe de dépendances ;
graphe d’imports ;
graphe de classes ;
graphe de modules ;
graphe de packages ;
graphe Git.
```

Exemple :

```text
UserController → UserService → UserRepository → PostgreSQL
```

Question :

```text
Quels composants sont impliqués dans la suppression d’un utilisateur ?
```

Le graphe peut suivre :

```text
route DELETE /users/:id
→ UserController.delete
→ UserService.deleteUser
→ UserRepository.delete
→ table users
```

La recherche vectorielle peut aider à retrouver des commentaires ou de la documentation, mais le graphe d’appels apporte une structure plus fiable.

---

## 9.27. Difficultés du Graph RAG

Graph RAG est puissant, mais il ajoute de la complexité.

### 9.27.1. Extraction imparfaite

Un LLM peut extraire une relation incorrecte.

Exemple :

```text
checkout utilise payments API.
```

Relation correcte :

```text
checkout --uses--> payments API
```

Relation incorrecte possible :

```text
payments API --uses--> checkout
```

La direction a été inversée.

### 9.27.2. Entités dupliquées

Exemple :

```text
payments API ;
payment-api ;
PaymentService.
```

Si elles ne sont pas fusionnées, le graphe est fragmenté.

### 9.27.3. Relations trop vagues

Exemple :

```text
A est lié à B.
```

Cette relation est peu utile.

Il vaut mieux une relation typée :

```text
A depends_on B ;
A runs_on B ;
A owned_by B.
```

### 9.27.4. Graphe trop dense

Si nous créons trop de relations, le graphe devient bruité.

Le parcours peut ramener trop de chemins.

### 9.27.5. Graphe obsolète

Si les documents changent, le graphe doit être mis à jour.

Une relation ancienne peut devenir fausse.

---

## 9.28. Évaluation d’un Graph RAG

Nous devons évaluer plusieurs éléments.

### 9.28.1. Qualité de l’extraction

Questions :

```text
Les bonnes entités sont-elles extraites ?
Les relations sont-elles correctes ?
Les directions sont-elles correctes ?
Les relations sont-elles sourcées ?
Les doublons sont-ils fusionnés ?
```

### 9.28.2. Qualité du graphe

Questions :

```text
Le graphe est-il trop fragmenté ?
Trop dense ?
Les types de relations sont-ils utiles ?
Les métadonnées sont-elles présentes ?
Les relations obsolètes sont-elles supprimées ?
```

### 9.28.3. Qualité du retrieval

Questions :

```text
Le système trouve-t-il le bon chemin ?
Le sous-graphe récupéré contient-il les relations nécessaires ?
Le système évite-t-il les chemins inutiles ?
```

### 9.28.4. Qualité de la réponse

Questions :

```text
Le LLM explique-t-il correctement le chemin ?
Cite-t-il les sources textuelles ?
Signale-t-il les incertitudes ?
Ne transforme-t-il pas une relation faible en certitude ?
```

---

## 9.29. Graph RAG et hallucinations

Graph RAG peut réduire certaines hallucinations, car il fournit une structure explicite.

Mais il peut aussi introduire de nouvelles erreurs.

Si le graphe contient une relation fausse, le système peut produire une réponse fausse avec beaucoup de confiance.

Exemple :

Relation incorrecte :

```text
checkout --runs_on--> cluster-3
```

alors que la relation correcte est :

```text
payments API --runs_on--> cluster-3
checkout --uses--> payments API
```

La réponse peut devenir :

```text
checkout tourne sur cluster-3.
```

Ce qui est faux.

Nous devons donc toujours conserver la traçabilité vers les sources et éviter de traiter le graphe comme une vérité absolue.

---

## 9.30. Quand utiliser Graph RAG ?

Graph RAG est pertinent lorsque les questions portent sur :

```text
des dépendances ;
des relations entre entités ;
des chaînes causales ;
des impacts indirects ;
des réseaux d’acteurs ;
des liens entre documents ;
des questions multi-hop ;
des synthèses globales de corpus ;
des architectures logicielles ;
des corpus scientifiques ou juridiques structurés.
```

Il est moins nécessaire pour :

```text
des FAQ simples ;
des questions factuelles directes ;
des petits corpus ;
des documents très bien structurés ;
des cas où la recherche vectorielle suffit.
```

Nous devons donc éviter de complexifier inutilement.

---

## 9.31. Comparaison RAG standard et Graph RAG

|Critère|RAG standard|Graph RAG|
|---|---|---|
|Représentation principale|Chunks + embeddings|Entités + relations + sources|
|Recherche|Similarité vectorielle|Parcours de graphe + retrieval|
|Très bon pour|Questions factuelles simples|Questions relationnelles et multi-hop|
|Limite principale|Peut rater les faits intermédiaires|Extraction et maintenance complexes|
|Coût d’indexation|Modéré|Plus élevé|
|Explicabilité|Sources textuelles|Chemins + sources|
|Risque|Chunks proches mais incomplets|Relations fausses ou obsolètes|

---

## 9.32. Pseudo-code simplifié d’un Graph RAG

### Indexation

```python
def index_documents(documents):
    for document in documents:
        chunks = chunk_document(document)

        for chunk in chunks:
            # Index vectoriel classique
            embedding = embedding_model.encode(chunk.text)
            vector_index.add(
                id=chunk.id,
                vector=embedding,
                text=chunk.text,
                metadata=chunk.metadata
            )

            # Extraction graphe
            entities = extract_entities(chunk.text)
            relations = extract_relations(chunk.text, entities)

            for entity in entities:
                graph.upsert_node(
                    id=normalize_entity(entity),
                    type=entity.type,
                    properties=entity.properties
                )

            for relation in relations:
                graph.upsert_edge(
                    source=normalize_entity(relation.source),
                    relation_type=relation.type,
                    target=normalize_entity(relation.target),
                    source_chunk=chunk.id,
                    confidence=relation.confidence
                )
```

### Requête

```python
def graph_rag_answer(question, user):
    question_entities = extract_entities(question)

    start_nodes = [
        graph.find_node(entity)
        for entity in question_entities
    ]

    subgraph = graph.traverse(
        start_nodes=start_nodes,
        max_depth=3,
        allowed_relations=["uses", "depends_on", "runs_on", "maintenance_on"],
        filters={"permissions": user.permissions}
    )

    source_chunks = get_chunks_from_edges(subgraph.edges)

    vector_chunks = vector_index.search(
        query=question,
        top_k=10,
        filters={"permissions": user.permissions}
    )

    context = merge_graph_and_vector_context(
        graph_context=subgraph,
        text_chunks=source_chunks + vector_chunks
    )

    prompt = build_prompt(question, context)

    return llm.generate(prompt)
```

Ce pseudo-code montre que Graph RAG n’exclut pas la recherche vectorielle. Il la complète.

---

## 9.33. Mini-TP : construire un graphe de dépendances

### Objectif

Nous voulons construire manuellement un petit graphe à partir de documents.

### Corpus

```text
doc_1 :
Le service checkout utilise l’API payments.

doc_2 :
L’API payments est déployée sur le cluster-3.

doc_3 :
Le cluster-3 sera en maintenance vendredi.

doc_4 :
Le service billing utilise aussi l’API payments.

doc_5 :
Le service profile utilise l’API users.
```

### Travail demandé

Nous devons extraire les triplets :

```text
checkout --uses--> payments API
payments API --runs_on--> cluster-3
cluster-3 --maintenance_on--> vendredi
billing --uses--> payments API
profile --uses--> users API
```

Puis répondre à la question :

```text
Quels services peuvent être affectés par la maintenance de cluster-3 ?
```

### Réponse attendue

Nous partons de :

```text
cluster-3
```

Nous cherchons les services reliés :

```text
cluster-3
← payments API
← checkout

cluster-3
← payments API
← billing
```

Réponse :

```text
Les services checkout et billing peuvent être affectés, car ils utilisent
payments API, qui est déployée sur cluster-3.
```

---

## 9.34. Mini-TP : comparer vectoriel et graphe

### Objectif

Nous voulons comprendre pourquoi le graphe aide sur les questions multi-hop.

### Question

```text
Le service billing sera-t-il affecté par la maintenance de vendredi ?
```

### Documents

```text
billing utilise payments API.
payments API est déployée sur cluster-3.
cluster-3 est en maintenance vendredi.
```

### Étape 1 : retrieval vectoriel

Nous observons quels documents sont retrouvés par similarité.

Il est possible que le document :

```text
payments API est déployée sur cluster-3.
```

soit mal classé.

### Étape 2 : retrieval par graphe

Nous suivons :

```text
billing → payments API → cluster-3 → vendredi
```

### Analyse attendue

Le graphe rend explicite le chemin relationnel.

La recherche vectorielle peut fonctionner, mais elle ne garantit pas de retrouver tous les faits intermédiaires.

---

## 9.35. Questions de compréhension

À la fin de ce chapitre, nous devons pouvoir répondre aux questions suivantes :

```text
Pourquoi le RAG vectoriel peut-il échouer sur les questions multi-hop ?
Qu’est-ce qu’un graphe ?
Qu’est-ce qu’un graphe de connaissances ?
Qu’est-ce qu’un triplet sujet-relation-objet ?
Comment extrait-on des entités depuis un document ?
Comment extrait-on des relations ?
Pourquoi la normalisation des entités est-elle importante ?
Pourquoi faut-il sourcer les relations du graphe ?
Comment fonctionne le retrieval dans un Graph RAG ?
Qu’est-ce qu’un sous-graphe pertinent ?
Pourquoi Graph RAG et recherche vectorielle sont-ils complémentaires ?
Quels sont les risques d’un Graph RAG ?
Comment évaluer un Graph RAG ?
Dans quels cas Graph RAG est-il utile ?
Dans quels cas est-il probablement inutile ?
```

---

## 9.36. Synthèse du chapitre

Dans ce chapitre, nous avons étudié le Graph RAG.

Nous avons vu que le RAG vectoriel retrouve des passages proches de la question, mais qu’il peut échouer lorsque la réponse nécessite de relier plusieurs faits dispersés.

Le Graph RAG ajoute une représentation explicite :

```text
entités ;
relations ;
chemins ;
sous-graphes ;
sources.
```

Cela permet de répondre plus efficacement à des questions multi-hop comme :

```text
Quels services seront affectés par une maintenance ?
Quelles dépendances relient deux composants ?
Quels acteurs sont impliqués dans une procédure ?
Quels documents parlent d’une même entité ?
```

Nous avons aussi vu que Graph RAG ne remplace pas le RAG standard. Il le complète.

Un système robuste peut combiner :

```text
recherche vectorielle ;
recherche lexicale ;
graphe de connaissances ;
reranking ;
prompt augmenté ;
citations précises.
```

Mais Graph RAG introduit aussi de nouveaux défis :

```text
extraction d’entités ;
extraction de relations ;
résolution des doublons ;
maintenance du graphe ;
relations obsolètes ;
évaluation des chemins ;
traçabilité vers les sources.
```

Le message principal du chapitre est donc le suivant :

```text
Graph RAG devient utile lorsque la question ne demande pas seulement
de retrouver un passage pertinent, mais de suivre des relations entre
plusieurs entités pour construire une réponse.
```

Dans le prochain chapitre, nous étudierons l’**Agentic RAG**. Nous verrons comment un agent peut choisir dynamiquement les outils, les sources et les étapes nécessaires pour répondre à des questions complexes.

---

---
> [!info] Livre « RAG » — chapitre 9/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 08 — Optimisation hybride, reranking et query rewriting|← 08 — Optimisation hybride, reranking et query rewriting]] · [[RAG — 10 — Agentic RAG agents, outils et orchestration dynamique|10 — Agentic RAG agents, outils et orchestration dynamique →]]
