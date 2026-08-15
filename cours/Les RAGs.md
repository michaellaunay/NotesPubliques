---
schema_version: 1
uid: "01M02EX5BAMEN15GB8B2S2QY9C"
titre: "Les RAGs"
aliases:
  - "Graph RAG"
  - "Agentic RAG"
type: fiche
statut: actif
para: ressource
domaines:
  - enseignement
  - veille
themes:
  - informatique
  - intelligence-artificielle
  - rag
  - graph-rag
  - agentic-rag
resume: "Synthèse comparative des architectures RAG : RAG standard et ses limites, Graph RAG, Agentic RAG, requêtes single-hop et multi-hop, coûts et pièges de la similarité vectorielle."
niveau: intermediaire
prerequis:
  - "[[RAG]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2026-07-13
date_modification: 2026-06-03
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
Le RAG n’est pas une seule **architecture**, c’est une famille de méthodes pour donner à un LLM accès à une connaissance externe au moment de répondre. Le papier fondateur de 2020 décrit déjà cette idée : combiner la mémoire “paramétrique” du modèle, c’est-à-dire ce qu’il a appris pendant l’entraînement, avec une mémoire externe récupérée par recherche documentaire. ([arXiv](https://arxiv.org/abs/2005.11401?utm_source=chatgpt.com "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"))

## 1. Le problème de départ : pourquoi le RAG existe

Un LLM seul fonctionne comme un modèle qui a appris des régularités linguistiques et beaucoup de connaissances générales. Mais il a plusieurs limites :

Il ne connaît pas forcément tes documents internes.

Il peut être obsolète.

Il peut halluciner.

Il ne cite pas naturellement ses sources.

Il ne sait pas toujours sur quelle information fiable s’appuyer.

Le RAG, pour **Retrieval-Augmented Generation**, ajoute donc une étape avant la génération : on va chercher des documents pertinents, puis on les donne au LLM dans le prompt.

En très simplifié :

```text
Question utilisateur
        ↓
Recherche dans une base documentaire
        ↓
Récupération de passages pertinents
        ↓
Injection des passages dans le prompt
        ↓
Réponse du LLM basée sur ces passages
```

Donc le LLM ne répond plus seulement “de mémoire”. Il répond avec un contexte documentaire fourni à la volée.

## 2. Le RAG standard

Dans un RAG classique, les documents sont découpés en morceaux, souvent appelés **chunks**.

Exemple :

```text
Document A
  chunk 1
  chunk 2
  chunk 3

Document B
  chunk 1
  chunk 2
  chunk 3
```

Chaque chunk est transformé en **embedding**, c’est-à-dire un vecteur numérique qui représente son sens approximatif.

Par exemple :

```text
"La facture est payable sous 30 jours"
→ [0.12, -0.44, 0.91, ...]
```

La question utilisateur est elle aussi transformée en embedding.

Puis on cherche les chunks dont le vecteur est le plus proche de celui de la question.

```text
Question : "Quel est le délai de paiement ?"
        ↓
Embedding de la question
        ↓
Recherche de similarité
        ↓
Chunks proches :
  - "La facture est payable sous 30 jours"
  - "Le délai court à partir de la date d’émission"
```

Ensuite, ces chunks sont injectés dans le prompt :

```text
Tu es un assistant. Réponds uniquement avec les informations suivantes :

[chunk 1]
[chunk 2]
[chunk 3]

Question :
Quel est le délai de paiement ?
```

Ce mécanisme marche très bien pour des questions simples, dites **single-hop**.

Par exemple :

```text
Question : Quel est le cluster utilisé par l’API payments ?
Document : L’API payments tourne sur cluster-3.
Réponse : L’API payments tourne sur cluster-3.
```

La recherche vectorielle retrouve directement le bon passage.

## 3. La limite du RAG standard

La newsletter donne un bon exemple.

On a trois faits :

```text
Fait 1 : Le checkout service utilise payments API.
Fait 2 : Payments API tourne sur cluster-3.
Fait 3 : Cluster-3 est en maintenance vendredi.
```

Question :

```text
Le checkout service sera-t-il affecté par la maintenance de vendredi ?
```

Pour répondre correctement, il faut faire une chaîne logique :

```text
checkout service
→ utilise payments API
→ payments API tourne sur cluster-3
→ cluster-3 est en maintenance vendredi
→ donc checkout service peut être affecté
```

Le problème, c’est que la recherche vectorielle standard ne raisonne pas vraiment sur les relations. Elle cherche les chunks les plus proches de la question.

Elle peut retrouver :

```text
Fait 1 : Le checkout service utilise payments API.
```

parce que la question contient “checkout service”.

Elle peut retrouver :

```text
Fait 3 : Cluster-3 est en maintenance vendredi.
```

parce que la question parle de “maintenance vendredi”.

Mais elle peut rater :

```text
Fait 2 : Payments API tourne sur cluster-3.
```

Pourquoi ? Parce que ce chunk ne contient ni “checkout service”, ni “maintenance”, ni “vendredi”. Sémantiquement, il est au milieu du raisonnement, mais pas forcément proche de la question en espace vectoriel.

C’est ce que montre la première image : le système récupère le fait 1 et le fait 3, mais rate le fait 2, qui est pourtant le lien indispensable.

Le RAG standard récupère des morceaux pertinents **localement**, mais il ne garantit pas de récupérer le chemin logique complet.

## 4. Graph RAG : ajouter une couche relationnelle

Graph RAG ajoute une structure au-dessus des documents : un **graphe de connaissances**.

Au lieu de stocker uniquement des chunks dans une base vectorielle, on extrait aussi :

- des entités ;
    
- des relations ;
    
- parfois des types de relations ;
    
- parfois des communautés ou sous-graphes.
    

Exemple :

```text
Entités :
- checkout service
- payments API
- cluster-3
- Friday maintenance

Relations :
- checkout service --uses--> payments API
- payments API --runs_on--> cluster-3
- cluster-3 --scheduled_for--> Friday maintenance
```

On obtient alors un graphe :

```text
checkout service
       |
     uses
       |
payments API
       |
   runs_on
       |
cluster-3
       |
scheduled_for
       |
Friday maintenance
```

Quand l’utilisateur demande :

```text
Will the checkout service be affected by Friday’s maintenance?
```

le système peut partir de `checkout service`, parcourir les relations, trouver `payments API`, puis `cluster-3`, puis `Friday maintenance`.

Ce n’est plus seulement :

```text
Quels chunks ressemblent à ma question ?
```

C’est aussi :

```text
Quelles entités sont liées aux entités de ma question ?
Quel chemin relie ces concepts ?
Quel sous-graphe est pertinent ?
```

Microsoft Research décrit GraphRAG comme une technique combinant extraction de texte, analyse de réseau, prompting LLM et résumé, afin de mieux comprendre des corpus textuels. ([Microsoft](https://www.microsoft.com/en-us/research/project/graphrag/?utm_source=chatgpt.com "Project GraphRAG - Microsoft Research")) Leur papier présente GraphRAG comme une approche adaptée aux questions sur des corpus privés, notamment quand la question nécessite une compréhension globale ou relationnelle du corpus. ([arXiv](https://arxiv.org/abs/2404.16130?utm_source=chatgpt.com "A Graph RAG Approach to Query-Focused Summarization"))

## 5. Ce que fait concrètement un pipeline Graph RAG

La deuxième image montre le pipeline.

### Étape 1 : ingestion des documents

On part de documents bruts :

```text
docs internes
PDF
wiki
tickets
README
logs
notions d’architecture
```

### Étape 2 : extraction par LLM

Un LLM lit les documents et extrait des triplets du type :

```text
Sujet --relation--> Objet
```

Exemples :

```text
checkout service --uses--> payments API
payments API --runs_on--> cluster-3
cluster-3 --has_maintenance--> Friday
```

On peut aussi extraire des propriétés :

```text
cluster-3:
  type: Kubernetes cluster
  environment: production
  maintenance_date: Friday
```

### Étape 3 : construction du graphe

Les entités deviennent des nœuds.

Les relations deviennent des arêtes.

```text
(checkout service) --uses--> (payments API)
(payments API) --runs_on--> (cluster-3)
(cluster-3) --maintenance--> (Friday)
```

### Étape 4 : retrieval par parcours de graphe

Au moment de la question, le système identifie les entités de la question :

```text
checkout service
Friday maintenance
```

Puis il cherche les chemins entre elles.

```text
checkout service
→ payments API
→ cluster-3
→ Friday maintenance
```

### Étape 5 : génération

Le LLM reçoit un contexte structuré :

```text
Le checkout service utilise payments API.
Payments API tourne sur cluster-3.
Cluster-3 est en maintenance vendredi.
```

Et il peut répondre :

```text
Oui, le checkout service risque d’être affecté, car il dépend de payments API, qui tourne sur cluster-3, lequel est en maintenance vendredi.
```

Le point important : **Graph RAG rend explicites les relations que le RAG standard espère retrouver implicitement par similarité vectorielle**.

## 6. Graph RAG ne remplace pas forcément le RAG standard

Il ne faut pas comprendre :

```text
RAG standard = ancien
Graph RAG = meilleur
Agentic RAG = encore meilleur
```

Ce n’est pas une échelle de maturité. Ce sont des outils différents.

Le RAG standard est souvent meilleur pour :

```text
"Quelle est la procédure de remboursement ?"
"Quel est le montant indiqué dans ce contrat ?"
"Résume-moi cette note."
"Que dit la documentation sur cette API ?"
```

Graph RAG devient utile quand les questions ressemblent à :

```text
"Quels services seront affectés si ce cluster tombe ?"
"Quels clients dépendent indirectement de cette API ?"
"Quelles équipes sont liées à ce projet ?"
"Quels risques émergent de ces dépendances ?"
"Quels documents parlent du même acteur sous des noms différents ?"
```

Là, il faut relier plusieurs morceaux d’information.

## 7. Les coûts et difficultés du Graph RAG

Graph RAG est puissant, mais il a des contraintes.

D’abord, l’indexation est plus chère. Il faut faire passer les documents dans un LLM pour extraire les entités et relations.

Ensuite, l’extraction peut être imparfaite. Le LLM peut oublier une relation, inventer une relation, mal fusionner deux entités, ou créer des doublons.

Par exemple :

```text
"payments API"
"payment-api"
"PaymentService"
"service de paiement"
```

Ces quatre noms peuvent désigner la même chose, ou pas. Il faut donc gérer la résolution d’entités.

Autre difficulté : le graphe peut devenir très gros. Il faut donc savoir quels chemins parcourir, quelle profondeur autoriser, comment pondérer les relations, et comment éviter de ramener trop de contexte.

Enfin, Graph RAG ne dispense pas de garder les chunks originaux. En pratique, un bon système garde souvent :

```text
base vectorielle
+ graphe de connaissances
+ moteur lexical BM25
+ reranker
+ métadonnées
```

Le graphe aide à naviguer dans la structure, mais le texte source reste nécessaire pour justifier la réponse.

## 8. Agentic RAG : laisser un agent piloter la recherche

Agentic RAG est différent.

Dans un RAG classique, le pipeline est fixe :

```text
question
→ embedding
→ recherche vectorielle
→ top-k chunks
→ génération
```

Dans Graph RAG, le pipeline est souvent plus structuré :

```text
question
→ extraction d’entités
→ parcours du graphe
→ récupération de contexte
→ génération
```

Dans Agentic RAG, on laisse un **agent LLM** décider dynamiquement quoi faire.

L’agent peut se demander :

```text
Ai-je besoin de chercher dans la base documentaire ?
Ai-je besoin d’interroger une API ?
Ai-je besoin de chercher dans plusieurs sources ?
Ai-je besoin de reformuler la requête ?
Ai-je besoin de faire plusieurs étapes ?
Ai-je besoin de vérifier la réponse ?
```

LangChain décrit par exemple des retrieval agents capables de décider s’ils doivent récupérer du contexte depuis une vector store ou répondre directement. ([Documentation de LangChain](https://docs.langchain.com/oss/python/langgraph/agentic-rag?utm_source=chatgpt.com "Build a custom RAG agent with LangGraph")) Sa documentation distingue aussi le RAG “2-step”, où on récupère puis on répond, de l’Agentic RAG, où la récupération peut être exposée comme un outil utilisable par un agent. ([Documentation de LangChain](https://docs.langchain.com/oss/python/langchain/retrieval?utm_source=chatgpt.com "Retrieval - Docs by LangChain"))

## 9. Exemple d’Agentic RAG

Question :

```text
Est-ce que le service checkout sera affecté vendredi, et faut-il prévenir les clients premium ?
```

Un pipeline fixe pourrait chercher des chunks et répondre.

Un agent, lui, peut faire plusieurs actions :

```text
1. Chercher "checkout service" dans la documentation architecture.
2. Trouver qu’il dépend de payments API.
3. Chercher "payments API" dans l’inventaire infra.
4. Trouver qu’il tourne sur cluster-3.
5. Chercher les maintenances prévues.
6. Trouver que cluster-3 est en maintenance vendredi.
7. Interroger le CRM ou la base clients.
8. Identifier les clients premium utilisant checkout.
9. Proposer une réponse et une action.
```

Cela ressemble plus à un mini workflow autonome.

Le RAG standard répond à :

```text
"Retrouve-moi les passages pertinents."
```

Graph RAG répond à :

```text
"Retrouve-moi les relations pertinentes."
```

Agentic RAG répond à :

```text
"Décide toi-même quelles recherches et quels outils sont nécessaires pour accomplir la tâche."
```

## 10. Comparaison synthétique

|Architecture|Principe|Très bon pour|Limite principale|
|---|---|---|---|
|RAG standard|Recherche de chunks proches de la question|Questions factuelles simples, documentation, FAQ, contrats, procédures|Peut rater les faits intermédiaires|
|Graph RAG|Graphe d’entités et de relations|Raisonnement multi-hop, dépendances, liens entre documents|Indexation plus complexe, extraction imparfaite|
|Agentic RAG|Agent qui choisit outils et étapes|Tâches dynamiques, multi-sources, workflows|Plus coûteux, moins prévisible, plus difficile à évaluer|

## 11. Single-hop, multi-hop, multi-source

La newsletter utilise trois catégories utiles.

### Single-hop factual lookup

Une seule étape suffit.

```text
Question : Sur quel cluster tourne payments API ?
Fait : Payments API tourne sur cluster-3.
Réponse : cluster-3.
```

RAG standard suffit.

### Multi-hop relationship query

Il faut relier plusieurs faits.

```text
checkout service
→ payments API
→ cluster-3
→ maintenance vendredi
```

Graph RAG est adapté.

### Dynamic multi-source task

Il faut consulter plusieurs outils ou bases, dans un ordre qui dépend de ce qu’on découvre.

```text
docs architecture
+ monitoring
+ calendrier maintenance
+ CRM
+ tickets support
+ base clients
```

Agentic RAG est adapté.

## 12. Le piège : croire que la similarité vectorielle comprend tout

La recherche vectorielle est souvent présentée comme “sémantique”, et c’est vrai dans une certaine mesure. Mais elle ne raisonne pas comme un humain.

Elle sait dire :

```text
Ce passage ressemble à la question.
```

Elle ne sait pas toujours dire :

```text
Ce passage ne ressemble pas directement à la question, mais il est indispensable pour relier deux autres passages.
```

C’est exactement le problème du fait 2 dans la newsletter.

Le fait :

```text
Payments API runs on cluster-3.
```

ne ressemble pas directement à :

```text
Will checkout service be affected by Friday maintenance?
```

Pourtant, il est nécessaire.

C’est là que le graphe devient utile.

## 13. Agentic RAG n’est pas magique non plus

Agentic RAG donne plus de liberté au système, mais cette liberté a un coût.

Un agent peut :

- faire trop d’appels outils ;
    
- partir dans une mauvaise direction ;
    
- mal planifier ;
    
- boucler ;
    
- coûter plus cher ;
    
- être plus lent ;
    
- être plus difficile à tester ;
    
- produire des réponses moins déterministes.
    

Donc en production, on évite souvent de laisser un agent totalement libre.

On préfère souvent un agent avec :

```text
outils limités
budget d’itérations
traces d’exécution
validation intermédiaire
politiques de sécurité
timeouts
logs
évaluation automatique
```

En ingénierie, l’Agentic RAG est puissant, mais il faut le traiter comme un orchestrateur probabiliste, pas comme un simple appel de fonction fiable.

## 14. Une architecture réaliste en production

Dans une vraie architecture, on peut combiner les trois.

Par exemple :

```text
Question utilisateur
        ↓
Classification de la question
        ↓
┌───────────────────────────────┐
│ Question simple ?             │ → RAG standard
│ Question relationnelle ?      │ → Graph RAG
│ Tâche multi-source ?          │ → Agentic RAG
└───────────────────────────────┘
        ↓
Récupération du contexte
        ↓
Reranking / filtrage
        ↓
Génération
        ↓
Citations / sources / vérification
```

Autrement dit, on ne choisit pas forcément **un seul** modèle.

On peut avoir :

```text
Vector DB pour les passages
Graph DB pour les relations
SQL pour les données métier
API pour les données temps réel
Agent pour orchestrer certains cas complexes
```

## 15. Exemple avec ton contexte informatique

Imagine une base documentaire d’infrastructure Kubernetes.

Tu as :

```text
README de services
manifests Kubernetes
tickets d’incident
docs d’architecture
fichiers Terraform
logs d’exploitation
```

### RAG standard

Question :

```text
Quelle variable d’environnement configure Redis dans l’API ?
```

Le RAG standard suffit probablement. Il retrouve le chunk contenant la variable.

### Graph RAG

Question :

```text
Quels services seront impactés si Redis est indisponible ?
```

Là, il faut relier :

```text
services
→ dépendances
→ Redis
→ files BullMQ
→ workers
→ fonctionnalités utilisateur
```

Graph RAG est plus adapté.

### Agentic RAG

Question :

```text
Vérifie si la panne Redis d’hier peut expliquer les erreurs Sentry, puis propose un correctif.
```

Là, il faut peut-être :

```text
chercher dans les logs
chercher dans Sentry
lire la doc
consulter les manifests Kubernetes
regarder les métriques Redis
raisonner sur la chronologie
proposer un patch
```

C’est plutôt Agentic RAG.

## 16. Comment lire la newsletter en une phrase

La newsletter dit ceci :

Le **RAG standard** retrouve des passages similaires à la question ; le **Graph RAG** retrouve des chemins de relations entre entités ; l’**Agentic RAG** laisse un agent décider dynamiquement quelles recherches, sources et outils utiliser.

C’est une bonne synthèse.

Mais il faut ajouter la nuance d’ingénierie :

En production, ces architectures sont souvent combinées, et le vrai sujet n’est pas seulement “quelle architecture est la plus avancée”, mais “quel type de question dois-je servir, avec quel niveau de coût, de latence, de fiabilité et d’explicabilité ?”