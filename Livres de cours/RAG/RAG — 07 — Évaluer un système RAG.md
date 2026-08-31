---
schema_version: 1
uid: 01M1BQ627A8HB2RXXKAV3VVSCH
titre: "RAG — 07 — Évaluer un système RAG"
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
resume: "Chapitre 7 sur 12 du livre « RAG » : Évaluer un système RAG. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « RAG » — chapitre 7/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 06 — Améliorer le retrieval|← 06 — Améliorer le retrieval]] · [[RAG — 08 — Optimisation hybride, reranking et query rewriting|08 — Optimisation hybride, reranking et query rewriting →]]

# Chapitre 7 — Évaluer un système RAG
## 7.1. Introduction

Dans les chapitres précédents, nous avons construit un système RAG standard, puis nous avons étudié plusieurs façons d’améliorer le retrieval.

Nous avons vu que la qualité d’un RAG dépend de nombreux éléments :

```text
qualité des documents ;
qualité de l’extraction ;
qualité du chunking ;
qualité des embeddings ;
qualité du retrieval ;
qualité du prompt ;
qualité de la génération ;
qualité des citations ;
gestion des métadonnées ;
gestion des droits d’accès.
```

Mais une question reste centrale :

```text
Comment savons-nous que notre RAG fonctionne réellement ?
```

Il ne suffit pas de tester quelques questions à la main et de trouver les réponses “plutôt bonnes”. Un système RAG peut sembler impressionnant en démonstration, mais échouer sur des cas critiques.

Dans ce chapitre, nous allons apprendre à évaluer un système RAG de manière structurée.

L’idée principale est la suivante :

```text
Nous devons évaluer séparément le retrieval et la génération.
```

Pourquoi ?

Parce qu’un RAG peut échouer à plusieurs endroits.

Il peut récupérer les mauvais documents.

Il peut récupérer les bons documents, mais mal les utiliser.

Il peut produire une bonne réponse, mais sans citer correctement les sources.

Il peut répondre correctement sur les questions simples, mais échouer sur les questions multi-hop.

Nous devons donc construire une méthode d’évaluation qui permet de localiser les erreurs.

---

## 7.2. Pourquoi évaluer un RAG ?

Un RAG est une chaîne de traitement.

```text
question
   ↓
retrieval
   ↓
contexte
   ↓
génération
   ↓
réponse
```

Si la réponse finale est mauvaise, il faut savoir où se situe le problème.

### 7.2.1. Cas 1 : le retrieval échoue

Question :

```text
Le service checkout sera-t-il affecté vendredi ?
```

Documents nécessaires :

```text
checkout utilise payments API ;
payments API tourne sur cluster-3 ;
cluster-3 est en maintenance vendredi.
```

Si le retriever ne récupère que :

```text
checkout utilise payments API ;
cluster-3 est en maintenance vendredi.
```

il manque le lien intermédiaire.

Le LLM ne dispose pas de toutes les informations nécessaires.

Dans ce cas, le problème principal est le retrieval.

### 7.2.2. Cas 2 : le retrieval réussit, mais la génération échoue

Le retriever récupère :

```text
checkout utilise payments API ;
payments API tourne sur cluster-3 ;
cluster-3 est en maintenance vendredi.
```

Mais le LLM répond :

```text
Le service checkout sera indisponible vendredi.
```

Cette réponse est trop forte.

Les sources permettent de dire :

```text
checkout peut être affecté
```

mais elles ne permettent pas forcément de dire :

```text
checkout sera indisponible
```

Dans ce cas, le retrieval est correct, mais la génération surinterprète.

### 7.2.3. Cas 3 : la réponse est correcte, mais mal sourcée

Le modèle répond correctement :

```text
Le service checkout peut être affecté vendredi.
```

Mais il cite seulement :

```text
cluster-3 est en maintenance vendredi.
```

Il oublie de citer les sources qui relient checkout à payments API et payments API à cluster-3.

La réponse est plausible, mais elle n’est pas correctement justifiée.

Dans ce cas, le problème est la citation ou la traçabilité.

### 7.2.4. Cas 4 : la réponse est utile, mais trop vague

Le système répond :

```text
Oui, il y a un risque d’impact.
```

Mais il ne donne pas le raisonnement, les sources ou les limites.

La réponse n’est pas fausse, mais elle est insuffisante pour un contexte professionnel.

Nous devons donc évaluer aussi l’utilité de la réponse.

---

## 7.3. Séparer les niveaux d’évaluation

Pour évaluer un RAG correctement, nous pouvons distinguer quatre niveaux.

```text
1. Évaluation du retrieval
2. Évaluation du contexte fourni au LLM
3. Évaluation de la génération
4. Évaluation de la réponse finale
```

### 7.3.1. Évaluation du retrieval

Question :

```text
Le système récupère-t-il les bons documents ?
```

Nous regardons :

```text
les chunks retrouvés ;
leur rang ;
leur score ;
leur pertinence ;
leur couverture ;
leur redondance.
```

### 7.3.2. Évaluation du contexte

Question :

```text
Le contexte donné au LLM contient-il assez d’information pour répondre ?
```

Même si les bons chunks ont été récupérés, le système peut les tronquer, les dédupliquer incorrectement ou en supprimer certains avant le prompt.

### 7.3.3. Évaluation de la génération

Question :

```text
Le LLM utilise-t-il correctement le contexte ?
```

Nous regardons :

```text
fidélité aux sources ;
absence d’hallucination ;
qualité de la synthèse ;
gestion des incertitudes ;
gestion des contradictions.
```

### 7.3.4. Évaluation de la réponse finale

Question :

```text
La réponse est-elle utile pour l’utilisateur ?
```

Nous regardons :

```text
clarté ;
complétude ;
actionnabilité ;
format ;
niveau de détail ;
citations ;
adaptation au besoin.
```

---

## 7.4. Construire un jeu d’évaluation

Avant de parler de métriques, nous avons besoin d’un jeu de test.

Un jeu d’évaluation RAG contient généralement :

```text
des questions ;
les documents ou passages attendus ;
éventuellement une réponse de référence ;
des critères d’évaluation ;
des cas difficiles.
```

Exemple :

```json
{
  "question": "Le service checkout sera-t-il affecté vendredi ?",
  "expected_sources": [
    "doc_checkout_dependencies",
    "doc_payments_deployment",
    "doc_cluster_maintenance"
  ],
  "reference_answer": "Le service checkout peut être affecté, car il utilise payments API, qui est déployée sur cluster-3, et cluster-3 sera en maintenance vendredi.",
  "expected_behavior": "répondre avec prudence et signaler l'absence d'information sur le failover"
}
```

Un bon jeu d’évaluation doit couvrir plusieurs types de questions.

---

## 7.5. Types de questions à inclure

Nous devons éviter d’évaluer seulement des questions faciles.

Un vrai benchmark RAG doit inclure plusieurs catégories.

### 7.5.1. Questions factuelles simples

Exemple :

```text
Quand aura lieu la maintenance du cluster-3 ?
```

La réponse se trouve dans un seul document.

Objectif :

```text
vérifier que le RAG retrouve un passage directement pertinent.
```

### 7.5.2. Questions de reformulation

Exemple :

```text
Je n’arrive plus à me connecter, que faire ?
```

Document attendu :

```text
Procédure de réinitialisation du mot de passe utilisateur.
```

Objectif :

```text
tester la recherche sémantique.
```

### 7.5.3. Questions avec termes exacts

Exemple :

```text
Que signifie ERR_MODULE_NOT_FOUND ?
```

Objectif :

```text
tester la recherche lexicale ou hybride.
```

### 7.5.4. Questions multi-hop

Exemple :

```text
Le service checkout sera-t-il affecté par la maintenance de vendredi ?
```

Objectif :

```text
tester la capacité à récupérer plusieurs faits reliés.
```

### 7.5.5. Questions larges

Exemple :

```text
Quels services dépendent de payments API ?
```

Objectif :

```text
tester la couverture du retrieval.
```

### 7.5.6. Questions impossibles

Exemple :

```text
Quel est le numéro personnel du responsable sécurité ?
```

Si cette information n’est pas dans les documents, le système doit refuser de répondre.

Objectif :

```text
tester la capacité à dire "je ne sais pas".
```

### 7.5.7. Questions avec sources contradictoires

Exemple :

```text
Sur quel cluster tourne payments API ?
```

Sources :

```text
document ancien : cluster-1 ;
document récent : cluster-3.
```

Objectif :

```text
tester la gestion des contradictions et de la fraîcheur documentaire.
```

---

## 7.6. Évaluation du retrieval

L’évaluation du retrieval consiste à vérifier si les bons documents ont été récupérés.

Pour cela, nous avons besoin de connaître les documents attendus pour chaque question.

Exemple :

```text
Question :
Le service checkout sera-t-il affecté vendredi ?

Sources attendues :
doc_1 : checkout utilise payments API
doc_2 : payments API tourne sur cluster-3
doc_3 : cluster-3 est en maintenance vendredi
```

Nous lançons le retriever et nous regardons si ces documents apparaissent dans les résultats.

---

## 7.7. Recall@k

Le **recall@k** mesure si les documents attendus apparaissent dans les `k` premiers résultats.

Exemple :

```text
Sources attendues :
A, B, C

Résultats top 5 :
A, D, B, E, F
```

Nous avons récupéré A et B, mais pas C.

```text
recall@5 = 2 / 3
```

Soit :

```text
recall@5 = 0,67
```

### 7.7.1. Interprétation

Un recall élevé signifie que le système récupère bien les documents utiles.

C’est très important en RAG, car si un document nécessaire n’est pas récupéré, le LLM ne peut pas l’utiliser.

### 7.7.2. Exemple

Question :

```text
Le service checkout sera-t-il affecté vendredi ?
```

Sources attendues :

```text
doc_checkout_payments
doc_payments_cluster
doc_cluster_maintenance
```

Résultats top 3 :

```text
doc_checkout_payments
doc_cluster_maintenance
doc_checkout_api_rest
```

Le recall@3 vaut :

```text
2 / 3 = 0,67
```

Le système a raté le lien payments → cluster-3.

---

## 7.8. Precision@k

La **precision@k** mesure la proportion de résultats pertinents parmi les `k` premiers résultats.

Exemple :

```text
Résultats top 5 :
A, D, B, E, F

Résultats pertinents :
A, B
```

```text
precision@5 = 2 / 5 = 0,4
```

### 7.8.1. Interprétation

Une précision élevée signifie que le système récupère peu de bruit.

C’est important parce que le LLM peut être perturbé par des documents inutiles ou contradictoires.

### 7.8.2. Rappel vs précision

Le rappel et la précision sont souvent en tension.

Si nous augmentons `top_k`, le rappel peut augmenter, car nous récupérons plus de documents.

Mais la précision peut baisser, car nous récupérons aussi plus de bruit.

Exemple :

```text
top_k = 3 → peu de bruit, mais source manquante
top_k = 20 → source retrouvée, mais beaucoup de bruit
```

Un bon système cherche un équilibre.

---

## 7.9. MRR : Mean Reciprocal Rank

Le **MRR**, pour _Mean Reciprocal Rank_, mesure le rang du premier résultat pertinent.

Si le premier résultat pertinent est en position 1 :

```text
score = 1 / 1 = 1
```

S’il est en position 3 :

```text
score = 1 / 3 ≈ 0,33
```

S’il est en position 10 :

```text
score = 1 / 10 = 0,1
```

Le MRR est utile lorsque nous voulons savoir si le système place rapidement un bon résultat en haut de liste.

### 7.9.1. Exemple

Question :

```text
Comment réinitialiser un mot de passe administrateur ?
```

Résultats :

```text
1. Guide général sécurité
2. Politique des comptes
3. Procédure reset-admin-password
```

Le premier résultat vraiment pertinent est en position 3.

```text
reciprocal rank = 1 / 3
```

Sur plusieurs questions, nous faisons la moyenne.

---

## 7.10. nDCG : qualité du classement

Le **nDCG**, pour _normalized Discounted Cumulative Gain_, mesure la qualité du classement en tenant compte du degré de pertinence des résultats.

Contrairement à precision@k, qui considère souvent qu’un document est pertinent ou non pertinent, nDCG peut prendre en compte des niveaux :

```text
0 = non pertinent
1 = partiellement pertinent
2 = pertinent
3 = très pertinent
```

L’idée est de récompenser les systèmes qui mettent les résultats les plus pertinents en haut.

### 7.10.1. Pourquoi c’est utile ?

Dans un RAG, tous les documents pertinents ne se valent pas.

Exemple :

```text
Question :
Comment corriger l’erreur P2025 ?

Document A :
Prisma est un ORM TypeScript.
Pertinence : 1

Document B :
P2025 signifie qu’un enregistrement attendu est introuvable.
Pertinence : 3

Document C :
Exemple de correction d’une erreur P2025 dans notre projet.
Pertinence : 3
```

Le système doit idéalement placer B et C avant A.

---

## 7.11. Évaluer la couverture du contexte

Même si les documents sont récupérés, nous devons vérifier que le contexte final donné au LLM contient toutes les informations nécessaires.

Exemple :

Retrieval initial :

```text
doc_1 : checkout utilise payments API
doc_2 : payments API tourne sur cluster-3
doc_3 : cluster-3 est en maintenance vendredi
```

Mais lors de la construction du prompt, le système tronque le contexte et supprime `doc_2`.

Le retrieval initial était bon, mais le contexte final est incomplet.

Nous devons donc évaluer :

```text
les résultats bruts du retriever ;
les résultats après filtrage ;
les résultats après reranking ;
le contexte final envoyé au LLM.
```

Cette distinction est très importante en production.

---

## 7.12. Évaluation de la génération

Une fois le bon contexte fourni, nous devons évaluer la réponse générée.

Une bonne réponse RAG doit être :

```text
fidèle aux sources ;
correcte ;
complète ;
claire ;
sourcée ;
prudente quand nécessaire ;
utile pour l’utilisateur.
```

Nous pouvons donc définir plusieurs critères.

---

## 7.13. Fidélité aux sources

La fidélité mesure si la réponse est soutenue par les documents fournis.

Exemple :

Contexte :

```text
Le cluster-3 sera en maintenance vendredi soir.
```

Réponse fidèle :

```text
Le cluster-3 sera en maintenance vendredi soir.
```

Réponse non fidèle :

```text
Le cluster-3 sera indisponible tout le week-end.
```

La deuxième réponse ajoute une information absente.

### 7.13.1. Questions à poser

```text
Chaque affirmation importante est-elle présente dans les sources ?
La réponse ajoute-t-elle une date absente ?
Ajoute-t-elle une cause absente ?
Ajoute-t-elle une conséquence non justifiée ?
Transforme-t-elle une possibilité en certitude ?
```

---

## 7.14. Exactitude

L’exactitude mesure si la réponse est correcte par rapport à la vérité attendue.

Dans un RAG, l’exactitude est souvent évaluée par rapport :

```text
aux documents ;
à une réponse de référence ;
à un expert humain ;
à un jeu de données validé.
```

Une réponse peut être fidèle aux sources, mais inexacte si les sources sont obsolètes ou contradictoires.

C’est pourquoi il faut distinguer :

```text
fidélité au contexte ;
exactitude métier.
```

Exemple :

Contexte obsolète :

```text
checkout tourne sur cluster-1.
```

Réponse :

```text
checkout tourne sur cluster-1.
```

La réponse est fidèle au contexte, mais inexacte si la vérité actuelle est cluster-3.

Dans ce cas, le problème vient du corpus ou du retrieval, pas forcément de la génération.

---

## 7.15. Complétude

La complétude mesure si la réponse couvre tous les éléments nécessaires.

Question :

```text
Comment réinitialiser un mot de passe administrateur ?
```

Sources :

```text
validation sécurité nécessaire ;
utilisation du script reset-admin-password.sh ;
journalisation obligatoire de l’opération.
```

Réponse incomplète :

```text
Il faut utiliser le script reset-admin-password.sh.
```

Elle oublie la validation et la journalisation.

Une réponse complète devrait inclure les trois étapes.

---

## 7.16. Utilité

Une réponse peut être correcte mais peu utile.

Exemple :

```text
La procédure est décrite dans la documentation.
```

Cette réponse est vraie, mais elle n’aide pas l’utilisateur.

Une réponse utile doit être adaptée à la question.

Pour une question opérationnelle, elle peut proposer :

```text
les étapes à suivre ;
les prérequis ;
les risques ;
les limites ;
les sources.
```

Pour une question conceptuelle, elle peut expliquer :

```text
le principe ;
un exemple ;
les exceptions ;
les erreurs fréquentes.
```

L’utilité est plus subjective que le recall@k, mais elle est essentielle.

---

## 7.17. Qualité des citations

Un RAG doit être évalué sur ses citations.

Nous devons vérifier :

```text
les sources citées existent-elles ?
les sources citées soutiennent-elles réellement les affirmations ?
les citations sont-elles assez précises ?
le modèle cite-t-il toutes les affirmations importantes ?
cite-t-il des sources inutiles ?
oublie-t-il des sources essentielles ?
```

### 7.17.1. Citation correcte

Réponse :

```text
checkout utilise payments API [Source 1].
payments API est déployée sur cluster-3 [Source 2].
cluster-3 sera en maintenance vendredi [Source 3].
```

Chaque affirmation est soutenue.

### 7.17.2. Citation décorative

Réponse :

```text
checkout sera affecté vendredi [Source 3].
```

Source 3 dit seulement :

```text
cluster-3 sera en maintenance vendredi.
```

Elle ne suffit pas à justifier l’impact sur checkout.

La citation est donc décorative ou insuffisante.

---

## 7.18. Gestion des incertitudes

Nous devons évaluer si le système sait répondre avec prudence.

Exemple :

Sources :

```text
checkout utilise payments API ;
payments API tourne sur cluster-3 ;
cluster-3 sera en maintenance vendredi.
```

Réponse correcte :

```text
checkout peut être affecté vendredi.
Les sources ne précisent pas s’il existe un mécanisme de redondance.
```

Réponse trop forte :

```text
checkout sera indisponible vendredi.
```

Une bonne évaluation doit pénaliser les réponses qui transforment une possibilité en certitude.

---

## 7.19. Gestion des cas sans réponse

Un RAG doit savoir dire :

```text
Je ne trouve pas cette information dans les documents.
```

Nous devons donc inclure dans le benchmark des questions sans réponse.

Exemple :

```text
Quel est le numéro de téléphone personnel du responsable sécurité ?
```

Si cette information n’est pas dans le corpus, une bonne réponse est :

```text
Je ne trouve pas cette information dans les documents disponibles.
```

Une mauvaise réponse serait d’inventer un numéro ou de donner une réponse générique.

### 7.19.1. Métrique possible

Nous pouvons mesurer le taux de refus correct :

```text
nombre de questions impossibles correctement refusées
/
nombre total de questions impossibles
```

Mais il faut aussi éviter le refus excessif.

Si le système dit “je ne sais pas” alors que les sources permettent de répondre, c’est aussi une erreur.

---

## 7.20. Évaluation humaine

L’évaluation humaine reste souvent indispensable, surtout au début.

Nous pouvons demander à des évaluateurs de noter chaque réponse selon plusieurs critères :

```text
pertinence ;
exactitude ;
fidélité aux sources ;
complétude ;
clarté ;
utilité ;
qualité des citations ;
gestion de l’incertitude.
```

Exemple de grille :

```text
0 = mauvais
1 = insuffisant
2 = acceptable
3 = bon
4 = excellent
```

Pour chaque réponse, l’évaluateur peut indiquer :

```text
la réponse est correcte ;
la réponse est partiellement correcte ;
la réponse hallucine ;
la réponse manque une source ;
la réponse est trop vague ;
la réponse est trop catégorique.
```

L’évaluation humaine est coûteuse, mais elle permet de construire un premier jeu de référence.

---

## 7.21. Évaluation automatique avec un LLM juge

Nous pouvons aussi utiliser un LLM comme évaluateur.

Principe :

```text
question + contexte + réponse générée
        ↓
LLM juge
        ↓
score et commentaire
```

Exemple de prompt d’évaluation :

```text
Tu es un évaluateur de système RAG.
Évalue si la réponse est fidèle au contexte.

Question :
...

Contexte :
...

Réponse :
...

Critères :
- La réponse est-elle soutenue par le contexte ?
- Contient-elle une information non présente ?
- Transforme-t-elle une possibilité en certitude ?
- Les citations sont-elles correctes ?

Retourne un score de 0 à 4 et une justification.
```

### 7.21.1. Avantages

```text
rapide ;
peu coûteux par rapport à une évaluation humaine massive ;
utile pour comparer des versions ;
peut produire des commentaires détaillés.
```

### 7.21.2. Limites

Un LLM juge peut se tromper.

Il peut être trop indulgent, trop sévère ou instable.

Nous devons donc utiliser cette méthode avec prudence, idéalement en la calibrant sur des évaluations humaines.

---

## 7.22. Tests de non-régression

Un système RAG évolue.

Nous pouvons changer :

```text
le modèle d’embedding ;
la taille des chunks ;
le top-k ;
le reranker ;
le prompt ;
le modèle génératif ;
la stratégie hybride ;
les filtres ;
le corpus.
```

Chaque changement peut améliorer certains cas et en dégrader d’autres.

Nous devons donc construire des tests de non-régression.

Exemple :

```text
Avant modification :
recall@5 = 0,78
fidélité moyenne = 3,4 / 4
taux d’hallucination = 6 %

Après modification :
recall@5 = 0,83
fidélité moyenne = 3,1 / 4
taux d’hallucination = 11 %
```

Ici, le retrieval s’est amélioré, mais la génération s’est dégradée.

Les tests de non-régression permettent de détecter ce type de compromis.

---

## 7.23. Logs nécessaires à l’évaluation

Pour évaluer un RAG, nous devons logger les bonnes informations.

À chaque question, nous pouvons enregistrer :

```text
question utilisateur ;
question reformulée éventuelle ;
résultats du retrieval ;
scores ;
métadonnées des chunks ;
chunks finalement injectés ;
prompt final ;
réponse générée ;
sources citées ;
feedback utilisateur ;
latence ;
coût ;
modèle utilisé ;
version de l’index ;
version du prompt.
```

Sans logs, il est très difficile de comprendre pourquoi une réponse est mauvaise.

Les logs permettent aussi de construire progressivement un jeu d’évaluation à partir des usages réels.

---

## 7.24. Analyse d’erreurs

L’évaluation ne sert pas seulement à produire un score.

Elle sert surtout à comprendre les erreurs.

Nous pouvons classer les erreurs en catégories.

### 7.24.1. Erreur d’ingestion

Le document source est mal extrait.

Exemple :

```text
tableau PDF cassé ;
colonnes mélangées ;
texte OCR bruité.
```

### 7.24.2. Erreur de chunking

L’information est coupée entre plusieurs chunks.

Exemple :

```text
la condition est dans un chunk ;
l’action est dans le chunk suivant.
```

### 7.24.3. Erreur d’embedding

Le modèle ne rapproche pas correctement la question et le document.

### 7.24.4. Erreur de retrieval

Le bon document existe mais n’est pas récupéré.

### 24.5. Erreur de reranking

Le bon document est récupéré en candidat, mais supprimé ou mal classé par le reranker.

### 7.24.6. Erreur de prompt

Le contexte est bon, mais le prompt ne contraint pas assez le modèle.

### 7.24.7. Erreur de génération

Le modèle hallucine, surinterprète ou ignore une source.

### 7.24.8. Erreur de citation

La réponse est correcte, mais les sources sont absentes ou incorrectes.

### 7.24.9. Erreur de corpus

Les documents eux-mêmes sont obsolètes, contradictoires ou incomplets.

Cette classification aide à choisir la bonne correction.

---

## 7.25. Exemple d’analyse d’erreur

Question :

```text
Le service checkout sera-t-il affecté vendredi ?
```

Réponse du système :

```text
Oui, checkout sera indisponible vendredi à 22 h.
```

Sources récupérées :

```text
Source 1 :
checkout utilise payments API.

Source 2 :
payments API est déployée sur cluster-3.

Source 3 :
cluster-3 sera en maintenance vendredi à 22 h.
```

Analyse :

```text
Retrieval :
correct, les trois sources nécessaires sont présentes.

Génération :
partiellement incorrecte, car le modèle conclut à une indisponibilité certaine.

Erreur :
surinterprétation.

Correction possible :
modifier le prompt pour distinguer faits, déductions et incertitudes ;
ajouter des exemples de réponses prudentes ;
évaluer la fidélité aux sources.
```

Autre cas :

Sources récupérées :

```text
Source 1 :
checkout utilise payments API.

Source 2 :
cluster-3 sera en maintenance vendredi.
```

Analyse :

```text
Retrieval :
incomplet, le lien payments API → cluster-3 manque.

Génération :
le modèle ne pouvait pas répondre rigoureusement.

Erreur :
retrieval multi-hop insuffisant.

Correction possible :
augmenter top-k ;
ajouter recherche hybride ;
faire du multi-query ;
introduire Graph RAG ou retrieval multi-hop.
```

---

## 7.26. Métriques techniques et métriques produit

Nous devons distinguer deux familles de métriques.

### 7.26.1. Métriques techniques

Elles évaluent le fonctionnement interne.

Exemples :

```text
recall@k ;
precision@k ;
MRR ;
nDCG ;
taux de sources attendues récupérées ;
taux d’hallucination ;
fidélité aux sources ;
exactitude ;
latence ;
coût par requête.
```

### 7.26.2. Métriques produit

Elles évaluent l’utilité pour les utilisateurs.

Exemples :

```text
taux de satisfaction ;
taux de clic sur les sources ;
taux de reformulation de question ;
taux d’escalade vers un humain ;
temps gagné ;
taux d’abandon ;
feedback positif ou négatif ;
questions sans réponse.
```

Un système peut avoir de bonnes métriques techniques mais être peu utile, par exemple s’il répond trop longuement ou s’il ne s’intègre pas bien dans le workflow utilisateur.

---

## 7.27. Évaluation en ligne et hors ligne

### 7.27.1. Évaluation hors ligne

Elle se fait sur un jeu de test fixe.

Avantages :

```text
reproductible ;
utile pour comparer deux versions ;
permet des tests de non-régression ;
contrôlable.
```

Limites :

```text
ne couvre jamais tous les usages réels ;
peut devenir obsolète ;
peut favoriser l’optimisation sur le benchmark.
```

### 7.27.2. Évaluation en ligne

Elle se fait pendant l’usage réel.

Exemples :

```text
feedback utilisateur ;
analyse des logs ;
A/B testing ;
mesure du taux de correction humaine ;
suivi des questions échouées.
```

Avantages :

```text
reflète les usages réels ;
permet de détecter des problèmes inattendus.
```

Limites :

```text
plus difficile à contrôler ;
dépend des utilisateurs ;
peut poser des enjeux de confidentialité.
```

Un bon système combine les deux.

---

## 7.28. Évaluer les coûts et la latence

Un RAG doit aussi être évalué comme un système informatique.

Nous devons mesurer :

```text
temps d’embedding de la question ;
temps de retrieval ;
temps de reranking ;
temps de construction du prompt ;
temps de génération ;
latence totale ;
coût du modèle d’embedding ;
coût du LLM ;
coût du reranker ;
taille du contexte ;
nombre d’appels outils.
```

Une amélioration de qualité peut être trop coûteuse pour certains usages.

Exemple :

```text
multi-query retrieval + reranking + gros modèle
```

peut être excellent, mais trop lent pour un chatbot interactif.

Nous devons donc arbitrer :

```text
qualité ;
latence ;
coût ;
robustesse.
```

---

## 7.29. Mini-TP : évaluer le retrieval

### Objectif

Nous voulons calculer recall@k et precision@k sur un petit exemple.

### Données

Question :

```text
Le service checkout sera-t-il affecté vendredi ?
```

Sources attendues :

```text
A : checkout utilise payments API
B : payments API tourne sur cluster-3
C : cluster-3 est en maintenance vendredi
```

Résultats du retriever top 5 :

```text
1. A
2. D : checkout expose une API REST
3. C
4. E : documentation générale Kubernetes
5. F : politique de monitoring
```

### Travail

Nous calculons :

```text
recall@5 ;
precision@5 ;
recall@3 ;
precision@3.
```

### Correction attendue

Résultats pertinents retrouvés dans le top 5 :

```text
A et C
```

Donc :

```text
recall@5 = 2 / 3
precision@5 = 2 / 5
```

Dans le top 3 :

```text
A et C
```

Donc :

```text
recall@3 = 2 / 3
precision@3 = 2 / 3
```

Le système a une précision correcte dans le top 3, mais il manque toujours la source B.

---

## 7.30. Mini-TP : évaluer une réponse générée

### Contexte

```text
[Source 1]
Le service checkout utilise l’API payments.

[Source 2]
L’API payments est déployée sur le cluster-3.

[Source 3]
Le cluster-3 sera en maintenance vendredi à partir de 22 h.
```

Question :

```text
Le service checkout sera-t-il affecté vendredi ?
```

Réponse du système :

```text
Oui, le service checkout sera indisponible vendredi à partir de 22 h.
```

### Travail

Nous évaluons :

```text
fidélité aux sources ;
exactitude ;
niveau d’incertitude ;
qualité des citations ;
complétude.
```

### Analyse attendue

La réponse est partiellement fondée, car les sources permettent de déduire un risque d’impact.

Mais elle est trop catégorique.

Les sources ne disent pas que checkout sera indisponible.

Elles ne précisent pas non plus l’existence ou non d’un failover.

Réponse préférable :

```text
Le service checkout peut être affecté vendredi à partir de 22 h, car il utilise
l’API payments, qui est déployée sur le cluster-3, et ce cluster sera en maintenance.
Les sources ne précisent pas si un mécanisme de redondance permet d’éviter
l’impact.
```

---

## 7.31. Questions de compréhension

À la fin de ce chapitre, nous devons pouvoir répondre aux questions suivantes :

```text
Pourquoi faut-il évaluer séparément retrieval et génération ?
Qu’est-ce que recall@k ?
Qu’est-ce que precision@k ?
Pourquoi le MRR est-il utile ?
À quoi sert nDCG ?
Quelle est la différence entre fidélité aux sources et exactitude ?
Pourquoi une réponse fidèle peut-elle être inexacte ?
Qu’est-ce qu’une hallucination dans un RAG ?
Qu’est-ce qu’une citation décorative ?
Pourquoi faut-il inclure des questions impossibles dans le benchmark ?
À quoi sert l’analyse d’erreurs ?
Pourquoi les logs sont-ils indispensables ?
Quelle différence entre évaluation hors ligne et évaluation en ligne ?
Pourquoi faut-il mesurer aussi la latence et le coût ?
```

---

## 7.32. Synthèse du chapitre

Dans ce chapitre, nous avons étudié l’évaluation d’un système RAG.

Nous avons vu qu’il ne suffit pas de juger la réponse finale de manière intuitive. Nous devons séparer plusieurs niveaux :

```text
retrieval ;
contexte ;
génération ;
réponse finale.
```

Nous avons introduit des métriques pour le retrieval :

```text
recall@k ;
precision@k ;
MRR ;
nDCG.
```

Nous avons aussi étudié les critères de génération :

```text
fidélité aux sources ;
exactitude ;
complétude ;
utilité ;
qualité des citations ;
gestion de l’incertitude ;
capacité à refuser quand l’information manque.
```

Nous avons insisté sur l’importance d’un jeu d’évaluation représentatif, contenant :

```text
questions simples ;
questions reformulées ;
questions avec termes exacts ;
questions multi-hop ;
questions larges ;
questions impossibles ;
questions avec sources contradictoires.
```

Le message principal du chapitre est le suivant :

```text
Évaluer un RAG, ce n’est pas seulement demander si la réponse semble bonne.
C’est identifier où la chaîne échoue : documents, retrieval, contexte,
génération, citations ou usage final.
```

Dans le chapitre suivant, nous étudierons les optimisations avancées : recherche hybride, reranking, query rewriting, query expansion et stratégies de retrieval plus robustes.

---

---
> [!info] Livre « RAG » — chapitre 7/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 06 — Améliorer le retrieval|← 06 — Améliorer le retrieval]] · [[RAG — 08 — Optimisation hybride, reranking et query rewriting|08 — Optimisation hybride, reranking et query rewriting →]]
