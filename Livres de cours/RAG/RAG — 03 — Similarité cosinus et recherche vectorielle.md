---
schema_version: 1
uid: 01M1BQ6276JMTTZW8HPCZN98S2
titre: "RAG — 03 — Similarité cosinus et recherche vectorielle"
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
resume: "Chapitre 3 sur 12 du livre « RAG » : Similarité cosinus et recherche vectorielle. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « RAG » — chapitre 3/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 02 — Les embeddings représenter le sens sous forme de vecteur|← 02 — Les embeddings représenter le sens sous forme de vecteur]] · [[RAG — 04 — Construire un RAG standard|04 — Construire un RAG standard →]]

# Chapitre 3 — Similarité cosinus et recherche vectorielle
## 3.1. Introduction

Dans le chapitre précédent, nous avons étudié les **embeddings**. Nous avons vu qu’un embedding transforme un texte en vecteur numérique, de façon à placer des textes proches en sens dans des zones proches d’un espace vectoriel.

Nous savons maintenant représenter une phrase, un paragraphe ou un morceau de document sous forme de vecteur.

Mais une nouvelle question apparaît :

```text
Comment mesurer concrètement la proximité entre deux embeddings ?
```

Si nous avons :

```text
embedding(question)
embedding(chunk_1)
embedding(chunk_2)
embedding(chunk_3)
```

nous devons être capables de dire quel chunk est le plus proche de la question.

C’est là qu’intervient la **similarité cosinus**.

Dans ce chapitre, nous allons comprendre :

```text
ce qu’est la similarité cosinus ;
pourquoi elle est très utilisée en NLP et en RAG ;
comment elle permet de comparer deux embeddings ;
comment fonctionne une recherche vectorielle ;
ce qu’est une recherche top-k ;
quelles sont les limites des scores de similarité.
```

Nous allons donc passer de l’idée générale :

```text
deux textes proches ont des vecteurs proches
```

à un mécanisme concret :

```text
nous calculons un score de proximité entre vecteurs,
puis nous récupérons les passages les mieux classés.
```

---

## 3.2. Rappel : un texte devient un vecteur

Prenons une question utilisateur :

```text
Comment réinitialiser mon mot de passe ?
```

Un modèle d’embedding transforme cette question en vecteur :

```text
q = [0.12, -0.45, 0.88, ..., 0.03]
```

Dans notre base documentaire, nous avons plusieurs chunks :

```text
chunk_1 :
La procédure de réinitialisation du mot de passe nécessite une validation sécurité.

chunk_2 :
Le cluster-3 sera en maintenance vendredi soir.

chunk_3 :
Les factures sont payables sous trente jours.

chunk_4 :
L’API payments est utilisée par le service checkout.
```

Chaque chunk est lui aussi transformé en vecteur :

```text
v1 = embedding(chunk_1)
v2 = embedding(chunk_2)
v3 = embedding(chunk_3)
v4 = embedding(chunk_4)
```

Le problème devient alors mathématique :

```text
Quel vecteur vi est le plus proche du vecteur q ?
```

Si le modèle d’embedding fonctionne correctement, nous nous attendons à ce que :

```text
v1
```

soit le plus proche de :

```text
q
```

car `chunk_1` parle bien de réinitialisation de mot de passe.

---

## 3.3. Intuition géométrique

Pour comprendre la similarité cosinus, imaginons d’abord un espace à deux dimensions.

Chaque texte est représenté par un point ou un vecteur dans cet espace.

```text
                         sécurité / compte
                               ↑
                               |
          mot de passe         |      reset identifiants
                x              |             x
                               |
                               |
                               |
                               |
facture x                      |
                               |
-------------------------------→ infrastructure
                               |
                               |        cluster maintenance
                               |               x
```

Dans cet exemple, les textes sur le mot de passe sont proches les uns des autres. Les textes sur les factures ou l’infrastructure sont plus éloignés.

En réalité, les embeddings ne vivent pas dans un espace à deux dimensions, mais dans un espace à plusieurs centaines ou milliers de dimensions.

Cependant, l’intuition reste identique :

```text
nous voulons mesurer la proximité géométrique entre vecteurs.
```

---

## 3.4. Distance euclidienne ou similarité cosinus ?

Pour comparer deux vecteurs, nous pourrions utiliser la distance euclidienne.

La distance euclidienne mesure la distance directe entre deux points.

Mais en NLP, on utilise très souvent la **similarité cosinus**, qui mesure plutôt l’orientation des vecteurs.

Pourquoi ?

Parce que dans beaucoup de cas, ce qui nous intéresse n’est pas principalement la longueur du vecteur, mais sa direction dans l’espace sémantique.

Deux vecteurs peuvent avoir des longueurs différentes, mais pointer dans une direction similaire.

Exemple simplifié :

```text
u = [1, 1]
v = [10, 10]
```

Ces deux vecteurs n’ont pas la même longueur, mais ils pointent exactement dans la même direction.

La similarité cosinus considère qu’ils sont très similaires.

C’est utile car nous voulons surtout savoir si deux textes portent le même type de sens, pas si leurs vecteurs ont exactement la même amplitude.

---

## 3.5. Définition de la similarité cosinus

La similarité cosinus entre deux vecteurs `u` et `v` est définie par :

```text
cos(u, v) = (u · v) / (||u|| × ||v||)
```

où :

```text
u · v
```

est le produit scalaire entre les deux vecteurs, et :

```text
||u||
||v||
```

sont les normes des vecteurs.

Le produit scalaire mesure à quel point deux vecteurs vont dans la même direction.

La norme mesure la longueur d’un vecteur.

La division par les normes permet de normaliser le résultat.

---

## 3.6. Interprétation du score

La similarité cosinus donne une valeur comprise entre `-1` et `1`.

En simplifiant :

```text
1    → vecteurs dans la même direction
0    → vecteurs orthogonaux, donc peu liés
-1   → vecteurs dans des directions opposées
```

Dans beaucoup de systèmes d’embeddings modernes, nous rencontrons surtout des scores entre `0` et `1`.

Nous pouvons interpréter grossièrement :

```text
score proche de 1    → textes très proches
score autour de 0.5  → proximité moyenne ou incertaine
score proche de 0    → textes peu liés
```

Mais attention : ces seuils ne sont pas universels.

Un score de `0.82` peut être excellent avec un modèle et seulement moyen avec un autre. Il faut calibrer les seuils sur un corpus réel.

---

## 3.7. Exemple simple avec des vecteurs courts

Prenons deux vecteurs en deux dimensions :

```text
u = [1, 0]
v = [1, 0]
```

Ils pointent dans la même direction.

Leur similarité cosinus vaut :

```text
cos(u, v) = 1
```

Prenons maintenant :

```text
u = [1, 0]
v = [0, 1]
```

Ils sont perpendiculaires.

Leur similarité cosinus vaut :

```text
cos(u, v) = 0
```

Enfin :

```text
u = [1, 0]
v = [-1, 0]
```

Ils pointent dans des directions opposées.

Leur similarité cosinus vaut :

```text
cos(u, v) = -1
```

Dans un système RAG, les vecteurs sont beaucoup plus grands, mais le principe reste le même.

---

## 3.8. Exemple textuel

Imaginons une question :

```text
Question :
Comment réinitialiser mon mot de passe ?
```

Et trois chunks :

```text
Chunk A :
La procédure de changement du mot de passe utilisateur est décrite dans le guide sécurité.

Chunk B :
Le cluster Kubernetes sera mis à jour vendredi soir.

Chunk C :
Les factures doivent être réglées sous trente jours.
```

Après calcul des embeddings, nous pouvons obtenir des scores fictifs :

```text
similarité(question, chunk A) = 0.91
similarité(question, chunk B) = 0.34
similarité(question, chunk C) = 0.18
```

Le système récupère donc le chunk A.

C’est le mécanisme de base d’un RAG standard.

---

## 3.9. De la comparaison simple à la recherche vectorielle

Comparer une question avec trois chunks est facile.

Mais dans un vrai système, nous pouvons avoir :

```text
10 000 chunks ;
100 000 chunks ;
1 million de chunks ;
100 millions de chunks.
```

Nous ne voulons pas recalculer naïvement toutes les comparaisons de manière coûteuse à chaque question.

Nous utilisons donc une **base vectorielle** ou un **index vectoriel**.

Son rôle est de stocker les embeddings et de retrouver rapidement les vecteurs les plus proches d’une requête.

Pipeline :

```text
Documents
   ↓
Chunks
   ↓
Embeddings
   ↓
Index vectoriel
```

Lors d’une question :

```text
Question
   ↓
Embedding de la question
   ↓
Recherche dans l’index vectoriel
   ↓
Chunks les plus proches
```

La recherche vectorielle est donc le mécanisme qui permet de retrouver rapidement les documents pertinents à grande échelle.

---

## 3.10. Recherche top-k

Un retriever vectoriel renvoie souvent les `k` résultats les plus proches.

C’est ce qu’on appelle une recherche **top-k**.

Exemple :

```text
top_k = 5
```

Le système retourne les cinq chunks ayant les meilleurs scores.

Exemple fictif :

```text
1. chunk_17   score = 0.91
2. chunk_42   score = 0.87
3. chunk_08   score = 0.79
4. chunk_95   score = 0.72
5. chunk_11   score = 0.69
```

Ces chunks seront ensuite injectés dans le prompt du LLM.

Le choix de `k` est important.

Si `k` est trop petit, nous risquons de manquer une information nécessaire.

Si `k` est trop grand, nous risquons de donner trop de bruit au modèle.

---

## 3.11. Le compromis du top-k

Prenons une question simple :

```text
Quel est le délai de paiement des factures ?
```

Si le bon chunk est très proche, un `top_k = 3` peut suffire.

Mais pour une question complexe :

```text
Le service checkout sera-t-il affecté par la maintenance de vendredi ?
```

il faut peut-être récupérer plusieurs passages :

```text
checkout utilise payments API ;
payments API tourne sur cluster-3 ;
cluster-3 est en maintenance vendredi.
```

Si nous mettons :

```text
top_k = 1
```

nous récupérerons peut-être seulement :

```text
checkout utilise payments API.
```

et la réponse sera incomplète.

Si nous mettons :

```text
top_k = 20
```

nous récupérerons peut-être les bons passages, mais aussi beaucoup de bruit documentaire.

Le `top_k` est donc un paramètre d’équilibre entre :

```text
rappel : récupérer tous les éléments utiles ;
précision : éviter de récupérer des éléments inutiles.
```

---

## 3.12. Score de similarité et seuil minimal

En plus du top-k, nous pouvons utiliser un seuil minimal.

Par exemple :

```text
ne garder que les chunks dont le score est supérieur à 0.75
```

Cela évite de donner au LLM des documents trop éloignés.

Exemple :

```text
chunk_17   score = 0.91 → conservé
chunk_42   score = 0.87 → conservé
chunk_08   score = 0.79 → conservé
chunk_95   score = 0.52 → rejeté
chunk_11   score = 0.41 → rejeté
```

Mais là encore, attention : un seuil fixe peut être dangereux.

Certaines questions ont naturellement des scores faibles parce qu’elles sont formulées de manière vague ou parce que les documents sont différents lexicalement.

D’autres questions peuvent avoir des scores élevés sur des documents proches mais inutiles.

Nous devons donc tester ces seuils sur des cas réels.

---

## 3.13. Similarité élevée ne signifie pas réponse correcte

C’est une limite fondamentale.

Un chunk peut être très proche de la question, sans contenir la réponse.

Exemple :

```text
Question :
Comment corriger l’erreur Prisma P2025 ?

Chunk récupéré :
Prisma est un ORM moderne pour Node.js et TypeScript.
```

Le chunk parle bien de Prisma, donc il peut être proche.

Mais il ne répond pas à la question.

Autre exemple :

```text
Question :
Le service checkout sera-t-il affecté vendredi ?

Chunk récupéré :
Le service checkout gère la validation des paniers.
```

Le chunk parle bien de checkout, mais il ne dit rien sur vendredi, ni sur les dépendances, ni sur une maintenance.

Nous devons donc distinguer :

```text
similarité sémantique
```

et :

```text
pertinence pour répondre.
```

Ce sont deux notions proches, mais pas identiques.

---

## 3.14. Similarité faible ne signifie pas inutilité

À l’inverse, un chunk peut avoir une similarité modérée ou faible avec la question, tout en étant indispensable.

Reprenons l’exemple :

```text
Question :
Le service checkout sera-t-il affecté par la maintenance de vendredi ?
```

Faits disponibles :

```text
1. Le service checkout utilise l’API payments.
2. L’API payments tourne sur le cluster-3.
3. Le cluster-3 sera en maintenance vendredi.
```

Le fait 2 :

```text
L’API payments tourne sur le cluster-3.
```

peut avoir un score plus faible, parce qu’il ne contient pas les mots :

```text
checkout
maintenance
vendredi
affecté
```

Pourtant, il est indispensable.

La recherche vectorielle standard peut donc échouer sur les questions multi-hop (plusieurs sauts).

Cela ne signifie pas que les embeddings sont inutiles. Cela signifie qu’ils ne suffisent pas toujours.

---

## 3.15. Recherche vectorielle exacte et approximative

Lorsque nous avons peu de vecteurs, nous pouvons comparer la question à tous les chunks.

C’est une recherche exacte.

Mais à grande échelle, cette approche peut être coûteuse.

Les bases vectorielles utilisent souvent des algorithmes de recherche approximative des plus proches voisins, appelés **ANN**, pour _Approximate Nearest Neighbors_.

L’idée est d’accepter parfois une approximation pour gagner beaucoup en performance.

Nous cherchons donc :

```text
les vecteurs probablement les plus proches
```

plutôt que :

```text
les vecteurs mathématiquement les plus proches avec certitude absolue.
```

En pratique, cela permet de faire des recherches rapides dans de très grands corpus.

Mais cela ajoute un compromis :

```text
rapidité
vs
exactitude de la recherche
```

---

## 3.16. Index vectoriel : intuition

Un index vectoriel organise les vecteurs pour éviter de comparer la requête à tous les vecteurs un par un.

Il peut utiliser différentes structures ou techniques, comme :

```text
graphes de proximité ;
quantification ;
partitionnement ;
arbres ;
clustering ;
HNSW ;
IVF ;
PQ.
```

Nous n’avons pas besoin de maîtriser tous ces algorithmes dès maintenant, mais nous devons comprendre l’idée :

```text
l’index prépare l’espace vectoriel pour rendre les recherches rapides.
```

Sans index, chercher dans un million de chunks peut être coûteux.

Avec un index bien construit, la recherche peut être beaucoup plus rapide.

---

## 3.17. Métadonnées et filtrage

Une recherche vectorielle pure peut retourner des chunks proches, mais pas forcément autorisés, récents ou pertinents pour le contexte métier.

Nous devons donc utiliser les métadonnées.

Exemple de métadonnées :

```text
source = "documentation production"
type = "runbook"
date = "2026-05-10"
environment = "prod"
service = "checkout"
permissions = ["team-payments", "sre"]
```

Une recherche peut alors combiner :

```text
similarité vectorielle
+
filtres par métadonnées
```

Exemple :

```text
Question :
Comment redémarrer le service checkout en production ?

Filtre :
environment = "prod"
type = "runbook"
service = "checkout"
```

Cela évite de récupérer un document de test, une ancienne procédure ou une note non validée.

---

## 3.18. Recherche vectorielle et droits d’accès

Dans un système professionnel, les droits d’accès sont critiques.

Un utilisateur ne doit jamais recevoir une réponse fondée sur un document qu’il n’a pas le droit de lire.

Il ne suffit donc pas de faire :

```text
chercher les chunks les plus proches
```

Il faut faire :

```text
chercher les chunks les plus proches parmi ceux que l’utilisateur a le droit de consulter
```

Cela peut être géré :

```text
avant la recherche ;
pendant la recherche ;
après la recherche ;
ou par séparation des index.
```

Mais dans tous les cas, la règle est claire :

```text
le RAG ne doit pas devenir un moyen de contourner les permissions documentaires.
```

---

## 3.19. Recherche vectorielle et documents obsolètes

Un autre problème important est la fraîcheur des documents.

Supposons que nous ayons deux documents :

```text
Document A, 2023 :
Le service checkout tourne sur cluster-1.

Document B, 2026 :
Le service checkout tourne désormais sur cluster-3.
```

Les deux documents peuvent être proches de la question :

```text
Sur quel cluster tourne checkout ?
```

Si nous ne tenons pas compte de la date ou de la version, le système peut récupérer l’ancien document.

Les métadonnées permettent de gérer cela :

```text
date de mise à jour ;
version ;
statut du document ;
document archivé ou actif ;
environnement concerné.
```

Un bon RAG ne doit pas seulement retrouver un document sémantiquement proche. Il doit retrouver le bon document dans le bon contexte.

---

## 3.20. Exemple complet de recherche vectorielle

Nous construisons un mini corpus :

```text
doc_1 :
La procédure de réinitialisation du mot de passe nécessite une validation sécurité.

doc_2 :
Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.

doc_3 :
Les factures doivent être payées sous trente jours.

doc_4 :
Le service checkout utilise l’API payments.

doc_5 :
L’API payments est déployée sur le cluster-3.
```

Question :

```text
Le service checkout sera-t-il touché par la maintenance de vendredi ?
```

Résultats possibles d’une recherche vectorielle :

```text
1. doc_4 : score 0.84
   Le service checkout utilise l’API payments.

2. doc_2 : score 0.81
   Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.

3. doc_5 : score 0.58
   L’API payments est déployée sur le cluster-3.

4. doc_1 : score 0.22
   La procédure de réinitialisation du mot de passe nécessite une validation sécurité.

5. doc_3 : score 0.15
   Les factures doivent être payées sous trente jours.
```

Si nous prenons :

```text
top_k = 2
```

nous récupérons `doc_4` et `doc_2`, mais nous ratons `doc_5`.

Le LLM reçoit alors :

```text
checkout utilise payments API ;
cluster-3 est en maintenance vendredi.
```

Mais il ne reçoit pas le lien :

```text
payments API est déployée sur cluster-3.
```

Il ne peut donc pas conclure de manière rigoureuse.

Si nous prenons :

```text
top_k = 3
```

nous récupérons les trois documents nécessaires.

Cet exemple montre que la configuration du retrieval influence directement la qualité de la réponse.

---

## 3.21. Le rôle du reranking

Une recherche vectorielle est souvent une première étape.

Elle peut récupérer un ensemble assez large de documents :

```text
top_k = 30
```

Puis un modèle plus coûteux mais plus précis peut les reclasser.

C’est le **reranking**.

Pipeline :

```text
Question
   ↓
Recherche vectorielle large
   ↓
30 chunks candidats
   ↓
Reranker
   ↓
5 meilleurs chunks
   ↓
LLM
```

Le retriever vectoriel sert à chercher rapidement.

Le reranker sert à juger plus finement la pertinence de chaque chunk par rapport à la question.

Nous étudierons cette étape plus en détail dans un chapitre ultérieur.

---

## 3.22. Recherche vectorielle vs recherche hybride

La recherche vectorielle est très utile, mais elle n’est pas toujours suffisante.

Exemple :

```text
Question :
Que signifie l’erreur ERR_MODULE_NOT_FOUND ?
```

Le terme exact `ERR_MODULE_NOT_FOUND` est déterminant.

Une recherche lexicale peut être plus fiable qu’une recherche vectorielle.

À l’inverse :

```text
Question :
Je n’arrive plus à me connecter à mon compte, que faire ?
```

La recherche vectorielle peut retrouver un document qui parle de :

```text
réinitialisation du mot de passe
```

même si la question ne contient pas ces mots.

En production, une bonne stratégie est souvent hybride :

```text
score final = score lexical + score vectoriel + score de reranking
```

Cela permet de combiner :

```text
la précision des mots exacts ;
la souplesse de la recherche sémantique ;
la finesse d’un reranker.
```

---

## 3.23. Les erreurs fréquentes dans l’interprétation des scores

### 3.23.1. Croire qu’un score élevé garantit une bonne réponse

Un score élevé indique une proximité, pas une réponse suffisante.

### 3.23.2. Croire qu’un seuil universel existe

Il n’existe pas de seuil magique valable pour tous les modèles et tous les corpus.

### 3.23.3. Comparer les scores de modèles différents

Les scores produits par deux modèles d’embedding différents ne sont pas directement comparables.

### 3.23.4. Oublier les métadonnées

Un document proche mais obsolète, non autorisé ou hors contexte peut être dangereux.

### 3.23.5. Confondre retrieval et raisonnement

La recherche vectorielle sélectionne du contexte. Elle ne fait pas, à elle seule, le raisonnement complet.

---

## 3.24. Mini-TP : calculer une similarité cosinus

### Objectif

Nous voulons comprendre concrètement comment comparer deux vecteurs.

### Données

Nous prenons deux vecteurs simples :

```text
u = [1, 2, 3]
v = [2, 4, 6]
```

Ces deux vecteurs pointent dans la même direction, car :

```text
v = 2u
```

Nous nous attendons donc à une similarité cosinus de `1`.

Puis nous prenons :

```text
u = [1, 0]
v = [0, 1]
```

Ces vecteurs sont orthogonaux. Nous nous attendons à une similarité de `0`.

### Code Python

```python
import numpy as np

def cosine_similarity(u, v):
    u = np.array(u)
    v = np.array(v)

    dot_product = np.dot(u, v)
    norm_u = np.linalg.norm(u)
    norm_v = np.linalg.norm(v)

    return dot_product / (norm_u * norm_v)

print(cosine_similarity([1, 2, 3], [2, 4, 6]))
print(cosine_similarity([1, 0], [0, 1]))
print(cosine_similarity([1, 0], [-1, 0]))
```

### Résultat attendu

```text
1.0
0.0
-1.0
```

Ce TP montre que la similarité cosinus mesure bien l’orientation des vecteurs.

---

## 3.25. Mini-TP : simuler une recherche vectorielle

### Objectif

Nous voulons simuler le comportement d’un retriever.

### Corpus

```python
documents = [
    "La procédure de réinitialisation du mot de passe nécessite une validation sécurité.",
    "Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.",
    "Les factures doivent être payées sous trente jours.",
    "Le service checkout utilise l’API payments.",
    "L’API payments est déployée sur le cluster-3."
]
```

Dans un vrai système, nous utiliserions un modèle d’embedding.

Pour un exercice pédagogique, nous pouvons imaginer que nous avons déjà calculé les scores :

```python
scores = [
    0.22,
    0.81,
    0.15,
    0.84,
    0.58
]
```

Question :

```text
Le service checkout sera-t-il touché par la maintenance de vendredi ?
```

Nous voulons trier les documents par score.

### Code Python

```python
documents = [
    "La procédure de réinitialisation du mot de passe nécessite une validation sécurité.",
    "Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.",
    "Les factures doivent être payées sous trente jours.",
    "Le service checkout utilise l’API payments.",
    "L’API payments est déployée sur le cluster-3."
]

scores = [0.22, 0.81, 0.15, 0.84, 0.58]

ranked = sorted(
    zip(documents, scores),
    key=lambda item: item[1],
    reverse=True
)

for doc, score in ranked:
    print(score, "→", doc)
```

### Résultat attendu

```text
0.84 → Le service checkout utilise l’API payments.
0.81 → Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.
0.58 → L’API payments est déployée sur le cluster-3.
0.22 → La procédure de réinitialisation du mot de passe nécessite une validation sécurité.
0.15 → Les factures doivent être payées sous trente jours.
```

### Analyse

Nous voyons que les trois premiers documents permettent de construire la réponse complète.

Mais si nous avions choisi :

```text
top_k = 2
```

nous aurions raté le troisième document, pourtant indispensable.

Ce mini-TP montre concrètement l’importance du choix de `top_k`.

---

## 3.26. Questions de compréhension

À la fin de ce chapitre, nous devons être capables de répondre aux questions suivantes :

```text
Qu’est-ce que la similarité cosinus ?
Pourquoi l’utilise-t-on pour comparer des embeddings ?
Quelle est la différence entre distance et similarité ?
Que signifie un score proche de 1 ?
Pourquoi un score élevé ne garantit-il pas une bonne réponse ?
Qu’est-ce qu’une recherche top-k ?
Pourquoi le choix de k est-il important ?
Pourquoi faut-il utiliser des métadonnées ?
Pourquoi la recherche vectorielle peut-elle rater des faits intermédiaires ?
Pourquoi la recherche hybride est-elle souvent préférable en production ?
```

---

## 3.27. Synthèse du chapitre

Dans ce chapitre, nous avons étudié la similarité cosinus et la recherche vectorielle.

Nous avons vu que la similarité cosinus permet de comparer deux embeddings en mesurant l’orientation de leurs vecteurs.

Nous pouvons résumer ainsi :

```text
embedding(question)
        ↓
comparaison par similarité cosinus
        ↓
classement des chunks
        ↓
récupération des meilleurs résultats
```

Nous avons aussi vu que cette méthode est au cœur du RAG standard.

Cependant, nous avons insisté sur ses limites :

```text
un score élevé ne garantit pas une réponse correcte ;
un score faible ne signifie pas toujours que le passage est inutile ;
les seuils doivent être calibrés ;
le top-k influence fortement la réponse ;
les métadonnées sont indispensables ;
la recherche vectorielle peut rater les faits intermédiaires ;
la recherche hybride est souvent plus robuste.
```

Le message principal du chapitre est donc le suivant :

```text
La similarité cosinus permet de retrouver des textes proches en sens,
mais la proximité sémantique ne suffit pas toujours à garantir la pertinence documentaire.
```

Dans le chapitre suivant, nous construirons un **RAG standard complet** : ingestion, chunking, embeddings, indexation, retrieval, prompt augmenté et génération.

---

---
> [!info] Livre « RAG » — chapitre 3/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 02 — Les embeddings représenter le sens sous forme de vecteur|← 02 — Les embeddings représenter le sens sous forme de vecteur]] · [[RAG — 04 — Construire un RAG standard|04 — Construire un RAG standard →]]
