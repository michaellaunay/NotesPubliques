---
schema_version: 1
uid: 01M1BQ627G8V8JSFDWK9F5A134
titre: "RAG — Conclusion, travaux pratiques et annexes"
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
resume: "Matière finale du livre « RAG » : travaux pratiques, projet, progression, compétences et conclusion de la version longue du cours."
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

> [!info] Livre « RAG »
> [[RAG — Sommaire|Sommaire]] · [[RAG — Compléments 2026|← Compléments 2026]]

# Conclusion, travaux pratiques et annexes

## 13. Travaux pratiques proposés

## TP 1 — Construire un mini moteur de recherche sémantique

Nous construirons un petit moteur qui :

```text
charge des documents Markdown
les découpe en chunks
calcule les embeddings
stocke les vecteurs
retrouve les passages proches d’une question
```

Objectif : comprendre le retrieval sans encore générer de réponse.

## TP 2 — Construire un RAG simple

Nous ajouterons un LLM au moteur précédent.

Le système devra :

```text
retrouver les chunks pertinents
les injecter dans un prompt
produire une réponse
citer les sources
```

## TP 3 — Comparer recherche lexicale et vectorielle

Nous construirons des exemples où :

- BM25 fonctionne mieux ;
    
- l’embedding fonctionne mieux ;
    
- l’hybride fonctionne mieux.
    

Nous analyserons les résultats.

## TP 4 — Évaluer le RAG

Nous créerons un jeu de questions/réponses.

Nous mesurerons :

- recall@k ;
    
- pertinence des sources ;
    
- fidélité de la réponse ;
    
- hallucinations.
    

## TP 5 — Mini Graph RAG

Nous construirons un petit graphe à partir de documents techniques.

Exemple :

```text
service A dépend de service B
service B utilise Redis
Redis est déployé sur cluster-1
```

Puis nous poserons des questions multi-hop.

## TP 6 — Mini Agentic RAG

Nous construirons un agent simple capable de choisir entre :

```text
chercher dans les documents
interroger une base structurée
répondre directement
demander une vérification
```

L’objectif ne sera pas de faire un agent “magique”, mais de comprendre les risques d’orchestration.

---

## 14. Projet final possible

Nous proposerons un projet final sous forme de mini-produit RAG.

Par exemple :

```text
Construire un assistant de documentation technique pour un projet logiciel.
```

Le système devra :

- ingérer une documentation ;
    
- découper proprement les documents ;
    
- indexer les chunks ;
    
- répondre aux questions ;
    
- citer les sources ;
    
- gérer les cas où la réponse n’est pas dans les documents ;
    
- proposer une évaluation minimale ;
    
- documenter les limites.
    

Version avancée :

- ajouter BM25 ;
    
- ajouter un reranker ;
    
- ajouter un graphe de dépendances ;
    
- ajouter une interface web ;
    
- ajouter des métriques d’observabilité.
    

---

## 15. Progression pédagogique proposée

## Séance 1 — Introduction aux limites des LLM et au principe du RAG

Nous posons le problème : pourquoi un LLM seul ne suffit pas pour des données métier.

## Séance 2 — Embeddings et similarité cosinus

Nous comprenons comment représenter le texte sous forme de vecteurs et comparer les textes entre eux.

## Séance 3 — Bases vectorielles et retrieval

Nous construisons le cœur du système de recherche.

## Séance 4 — Chunking, métadonnées et ingestion documentaire

Nous voyons pourquoi la qualité du RAG dépend énormément de la préparation des documents.

## Séance 5 — Génération augmentée et prompts sourcés

Nous apprenons à injecter le contexte dans le prompt et à produire des réponses fiables.

## Séance 6 — Évaluation du RAG

Nous séparons l’évaluation du retrieval et celle de la génération.

## Séance 8 — Optimisation : hybride, reranking, query rewriting

Nous améliorons la pertinence des résultats.

## Séance 9 — Graph RAG

Nous abordons les requêtes multi-hop et les graphes de connaissances.

## Séance 10 — Agentic RAG

Nous abordons les agents, les outils et l’orchestration dynamique.

## Séance 11 — RAG multimodal et documents complexes

Nous traitons les PDF, tableaux, images, schémas et documents semi-structurés.

## Séance 12 — Production, sécurité, observabilité

Nous passons d’un prototype à une architecture exploitable.

## Séance 13 — Présentation des projets

Nous évaluons les projets étudiants selon la qualité technique, l’évaluation et la clarté des limites.

---

## 16. Compétences attendues à la fin du cours

À la fin du cours, nous devrons être capables de :

- expliquer clairement ce qu’est un RAG ;
    
- distinguer RAG standard, Graph RAG et Agentic RAG ;
    
- comprendre le rôle des embeddings ;
    
- utiliser la similarité cosinus ;
    
- construire un pipeline d’ingestion ;
    
- choisir une stratégie de chunking ;
    
- interroger une base vectorielle ;
    
- concevoir un prompt augmenté ;
    
- évaluer la qualité d’un système RAG ;
    
- identifier les limites d’un système RAG ;
    
- proposer une architecture de production raisonnable ;
    
- justifier le choix entre RAG standard, Graph RAG et Agentic RAG selon le besoin.
    

---

## 17. Message pédagogique central

Le fil conducteur du cours pourrait être le suivant :

> Nous n’allons pas apprendre à “mettre un chatbot sur des documents”. Nous allons apprendre à construire un système de recherche, de sélection, de justification et de génération contrôlée autour d’un LLM.

C’est cette distinction qui est fondamentale.

Un RAG n’est pas seulement un LLM avec une base vectorielle. C’est une architecture complète où la qualité dépend autant de l’ingestion, du retrieval, de l’évaluation, des métadonnées, de la sécurité et des sources que du modèle génératif lui-même.
