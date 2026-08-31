---
schema_version: 1
uid: 01M1BQ627G7573QWANVB1K84TF
titre: "RAG — Compléments 2026"
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
resume: "Compléments apportés au livre « RAG » : sections de la version condensée du cours [[RAG]] (31 août 2026) dont le sujet est absent de la version longue."
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
> [[RAG — Sommaire|Sommaire]] · [[RAG — 12 — Production, sécurité et observabilité d’un système RAG|← 12 — Production, sécurité et observabilité d’un système RAG]] · [[RAG — Conclusion, travaux pratiques et annexes|Conclusion →]]

# Compléments 2026

> [!info] Origine
> Les sections ci-dessous proviennent de la version condensée et actualisée du cours [[RAG]] (31 août 2026). Elles traitent de sujets absents de la version longue et n'ont pas été fondues dans les chapitres ; pour les versions logicielles et l'état de l'art du moment, la version condensée fait foi.

## 2.2. Identité, version et provenance

Une erreur fréquente consiste à utiliser le chemin du fichier ou son titre comme seule identité. Or un document peut être renommé sans changer de contenu, ou être remplacé par une nouvelle version au même endroit.

Il est plus robuste de distinguer trois niveaux :

```text
document_id
→ identité logique du document

revision_id
→ version du document

chunk_id
→ unité dérivée et indexée
```

Exemple :

```json
{
  "document_id": "proc-incident-042",
  "revision_id": "2026-08-17T14:32:00Z",
  "source_uri": "kb://production/incidents/042",
  "title": "Procédure de bascule Redis",
  "owner": "platform-team",
  "status": "validated",
  "classification": "internal",
  "language": "fr"
}
```

Lorsqu'une réponse cite un passage, nous devons idéalement pouvoir remonter jusqu'à la source originale :

```text
réponse
  ↓
citation
  ↓
chunk_id
  ↓
document_id + revision_id
  ↓
source originale
```

Cette chaîne de provenance rend la réponse auditée et reproductible. Sans elle, nous pouvons afficher une citation sans être capables de prouver exactement quel document a été utilisé.

## 3.2. Chunking fixe, récursif et structurel

La stratégie la plus simple consiste à découper tous les N caractères ou tokens, souvent avec un overlap.

```text
chunk 1 : tokens 0–500
chunk 2 : tokens 400–900
chunk 3 : tokens 800–1300
```

Cette méthode est facile à mettre en œuvre et constitue une bonne baseline, mais elle ignore la structure du document.

Un chunking récursif essaie au contraire de couper d'abord sur les séparateurs les plus significatifs : chapitres, paragraphes, phrases, puis espaces. Un chunking structurel peut aller plus loin et respecter directement les titres Markdown, les sections HTML ou les fonctions d'un fichier source.

Pour une documentation technique, il est souvent plus pertinent de conserver :

```text
Titre de section
+ sous-section
+ paragraphes associés
```

que de couper au milieu d'une explication simplement parce qu'une limite de tokens a été atteinte.

## 5.8. Multi-représentations

Un document peut disposer de plusieurs représentations de recherche.

Pour une page de documentation, nous pouvons indexer :

```text
texte original
résumé
mots-clés
questions auxquelles la page répond
embedding de la section
```

Le document final cité reste le texte source, mais les représentations auxiliaires peuvent améliorer le retrieval.

Ce principe est particulièrement utile lorsque le texte réel est mal adapté à la formulation des utilisateurs : spécifications très formelles, tableaux ou longues procédures.

## 6.5. Lost in the middle

Les modèles ne traitent pas toujours de manière égale toutes les positions d'un long contexte. Une information importante placée au milieu d'une grande quantité de texte peut être moins bien utilisée.

Il est donc raisonnable de :

- placer les preuves les plus importantes en tête ;
- éviter les passages redondants ;
- regrouper les chunks d'une même source ;
- structurer clairement les séparateurs et identifiants.

L'ordre du contexte fait partie de l'architecture RAG.

## 7.6. Faithfulness et correctness

La **faithfulness**, ou groundedness, demande si les affirmations de la réponse sont réellement soutenues par le contexte.

La **correctness** demande si la réponse est correcte par rapport à une référence ou à la réalité attendue.

Ces deux notions ne sont pas identiques. Un modèle peut produire une vérité connue de lui-même alors que cette vérité n'apparaît pas dans les sources. La réponse serait correcte dans l'absolu mais non grounded dans le RAG.

À l'inverse, si le corpus contient une information fausse mais validée, le modèle peut être parfaitement fidèle au contexte tout en répétant une erreur documentaire.

Ragas expose notamment des métriques de Faithfulness, Context Precision et Context Recall : https://docs.ragas.io/en/latest/concepts/metrics/available_metrics/

## 7.9. LLM-as-a-judge

Un LLM peut servir de juge pour comparer des réponses, estimer la fidélité ou vérifier des citations. Cette approche est pratique à grande échelle, mais elle n'est pas une vérité absolue.

Le juge doit être calibré contre des annotations humaines et recevoir une grille explicite. Il faut également éviter qu'il soit influencé par la longueur, le style ou l'ordre des réponses plutôt que par leur contenu.

Les métriques automatiques doivent donc compléter, et non remplacer totalement, une validation humaine ciblée.

## 9.5. Search-only

Parfois, la meilleure interface n'est pas une réponse générée.

Pour une recherche juridique ou documentaire très sensible, nous pouvons préférer retourner :

```text
5 passages classés
+ titres
+ liens
+ extraits
```

et laisser l'utilisateur interpréter les sources.

Un LLM peut encore servir à reformuler la requête ou reranker les résultats, sans être autorisé à synthétiser une conclusion.

Le RAG est donc un spectre allant de la recherche classique à une génération fortement orchestrée.

---

## 10.4. Isolation multi-tenant

Dans un SaaS, deux clients peuvent partager la même infrastructure tout en devant rester totalement isolés.

Un filtre tel que :

```text
tenant_id = acme
```

ne doit pas être construit depuis une valeur proposée par le LLM. Il doit venir du jeton d'authentification ou de la session serveur.

Selon le niveau de risque, nous pouvons utiliser :

- des index séparés ;
- des namespaces ;
- des filtres obligatoires appliqués côté serveur ;
- des clés de chiffrement distinctes ;
- des politiques réseau séparées.

L'isolation logique doit être testée explicitement avec des scénarios de fuite cross-tenant.

## 10.9. Human-in-the-loop

Pour une action à fort impact, le RAG peut servir à **préparer** une décision sans être autorisé à l'exécuter seul.

Exemple :

```text
RAG
→ retrouve la procédure
→ propose les étapes
→ cite les sources
→ opérateur humain valide
→ outil d'administration exécute
```

Cette séparation réduit le risque qu'une erreur de retrieval ou une prompt injection se transforme directement en action irréversible.

---

## 12.4. Anti-patterns fréquents

### Choisir une base vectorielle avant de définir le corpus

La technologie ne compense pas une mauvaise source de vérité.

### Choisir une taille de chunk universelle

La bonne unité dépend du document, de la question et du modèle.

### Envoyer les top 20 résultats au LLM « parce que le contexte est grand »

Plus de contexte n'est pas synonyme de plus de preuve.

### Mesurer uniquement la satisfaction utilisateur

Une réponse fluide peut être incorrecte et mal sourcée.

### Laisser le LLM décider des permissions

La sécurité doit être appliquée par l'application.

### Ajouter GraphRAG ou un agent avant d'avoir une baseline

Sans baseline et jeu d'évaluation, il est impossible de savoir si l'architecture avancée apporte réellement quelque chose.

## Glossaire

**ANN — Approximate Nearest Neighbor**
Famille de méthodes permettant de rechercher rapidement des vecteurs proches sans comparer exhaustivement tous les vecteurs du corpus.

**BM25**
Fonction de classement lexicale très utilisée dans les moteurs de recherche textuels.

**Chunk**
Unité documentaire indexée et récupérable par le retriever.

**Contextual Retrieval**
Technique qui enrichit un chunk avec un contexte décrivant sa position ou son rôle dans le document avant indexation.

**Cross-encoder**
Modèle qui lit conjointement la requête et un candidat pour produire un score de pertinence.

**Dense retrieval**
Recherche fondée sur des représentations vectorielles denses produites par un modèle d'embedding.

**Embedding**
Vecteur représentant une entrée dans un espace numérique destiné notamment à comparer sa proximité avec d'autres entrées.

**Faithfulness / Groundedness**
Degré auquel les affirmations produites sont soutenues par les sources fournies.

**GraphRAG**
Famille d'architectures qui combinent retrieval et structures de graphe afin de représenter ou exploiter des relations entre entités.

**Hybrid retrieval**
Combinaison de plusieurs retrievers, souvent dense et lexical.

**HyDE**
Méthode dans laquelle un document hypothétique est généré depuis la question puis utilisé pour guider le retrieval.

**Late interaction**
Architecture de retrieval qui conserve plusieurs représentations par document et réalise l'interaction question-document plus tard qu'un embedding dense classique.

**MRR — Mean Reciprocal Rank**
Métrique valorisant la position du premier résultat pertinent.

**nDCG — normalized Discounted Cumulative Gain**
Métrique de classement tenant compte de plusieurs degrés de pertinence et de leur position.

**RAG — Retrieval-Augmented Generation**
Architecture dans laquelle des informations externes sont récupérées afin d'augmenter une génération ou un raisonnement.

**Recall@k**
Part des éléments pertinents présents dans les `k` premiers résultats.

**Reranker**
Modèle ou fonction qui reclasse une liste de candidats produite par un premier retriever.

**RRF — Reciprocal Rank Fusion**
Méthode fusionnant plusieurs classements à partir de leurs rangs plutôt que de scores bruts non comparables.

**Sparse retrieval**
Recherche reposant sur des représentations creuses, lexicales ou apprises.

---
