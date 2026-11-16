---
statut: brouillon
para: projet
domaines:
- recherche
auteurs:
- Michaël Launay
confidentialite: publique
publication:
- notes-publiques
rag: true
titre: 'KR Scénario 42 : Rechercher et suivre ses centres d''intérêt'
type: document
themes:
- genie-logiciel
- scenarios
- metadonnees
- obsidian
- kuneagi
resume: 'Scénario de recherche : tri et filtres sur les contenus, requêtes mémorisées comme centres d''intérêt, alertes calculées localement quand un contenu nouveau y répond.'
schema_version: 1
uid: 01M7AQA2F4DX0QXQ9JFGPPVCXK
date_creation: 2026-11-13
langue: fr
date_modification: 2026-11-13
metadata_verifiees: false
---
# **Rechercher et suivre ses centres d'intérêt**

## Résumé
Bob cherche ce qui s'est dit sur le stationnement. Il interroge l'index que son application tient sur son téléphone, trie les résultats, les filtre, puis mémorise sa recherche comme centre d'intérêt. Désormais, son application l'alertera quand une idée, un appel ou un groupe de travail y répondra.

## Acteurs en jeu
- **Bob** : le membre.
- **Application mobile** : tient l'index local.
- **Dépôts suivis**.

## Prérequis
- Bob a synchronisé les dépôts de la communauté.
- L'index local a été construit à partir des en-têtes des fichiers.

## Étapes du scénario
1. Bob saisit « stationnement » dans la recherche.
2. L'application interroge son index local et affiche idées, propositions, questions et messages qui en parlent.
3. Bob trie : par date, par soutien, par titre.
4. Il filtre : par type de contenu, par période, par auteur, par appel à idées, par état.
5. Il enregistre la recherche comme centre d'intérêt, « Stationnement et vélos ».
6. Il choisit d'être alerté pour les nouveaux groupes de travail et les propositions soumises qui y répondent.
7. À chaque synchronisation, l'application rejoue ses centres d'intérêt sur les nouveautés et ajoute les résultats à ses notifications (scénario 25).
8. La recherche et les centres d'intérêt ne quittent pas le téléphone de Bob.

## Diagramme de séquence Mermaid
```mermaid
sequenceDiagram
    participant B as Bob
    participant A as Application mobile
    participant X as Index local
    participant R as Dépôts suivis
    B->>A: Recherche un mot
    A->>X: Interroge l'index
    X-->>A: Contenus correspondants
    B->>A: Trie et filtre
    B->>A: Mémorise la recherche comme centre d'intérêt
    A->>R: Synchronise, plus tard
    A->>X: Rejoue les centres d'intérêt sur les nouveautés
    A->>B: Alerte, un groupe de travail s'ouvre
```

Ce diagramme illustre les « centres d'intérêt » des spécifications de 2011, qui étaient déjà des requêtes mémorisées. La différence est leur emplacement : ils ne sont plus sur un serveur, donc personne ne peut dresser le portrait des curiosités de Bob.

Cas d'utilisation : « Recueillir les contributions et tenir les idées ». Retour à l'analyse : [[KR Analyse]].
