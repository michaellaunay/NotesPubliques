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
titre: 'KR Scénario 35 : Rejouer le pipeline d''idées et changer d''exécutant'
type: document
themes:
- genie-logiciel
- scenarios
- agents-ia
- gouvernance
- kuneagi
resume: 'Scénario de contrôle des agents : un membre rejoue le pipeline d''idées chez lui, publie l''écart constaté, et la communauté vote le changement d''exécutant ou de modèle.'
schema_version: 1
uid: 01M7AK890F34DH4449M1BDK6WA
date_creation: 2026-11-13
langue: fr
date_modification: 2026-11-13
metadata_verifiees: false
---
# **Rejouer le pipeline d'idées et changer d'exécutant**

## Résumé
Fatou soupçonne l'agent curateur de mal regrouper les idées depuis que son exécutant a changé de modèle. Le pipeline est rejouable par n'importe qui : elle le lance sur son poste, avec le modèle de son choix, compare ses fiches à celles qui sont publiées et publie l'écart. La communauté vote alors le changement d'exécutant.

## Acteurs en jeu
- **Fatou** : la membre qui contrôle.
- **Poste de Fatou** : exécute le pipeline localement.
- **Dépôt d'idées** et **dépôt de communauté**.
- **Les membres** : ils votent.

## Prérequis
- Fatou a cloné le dépôt d'idées.
- Elle dispose d'un modèle de langage, local ou distant.
- Les consignes du pipeline sont publiées dans le dépôt de communauté.

## Étapes du scénario
1. Fatou lance le pipeline sur les contributions publiées, sans rien écrire dans le dépôt partagé.
2. Son poste produit un jeu de fiches d'idées.
3. Il le compare au jeu publié : fiches regroupées différemment, citations manquantes, idées absentes.
4. Deux modèles ne produisent jamais les mêmes fiches ; Fatou cherche les écarts qui comptent. Elle en trouve un : douze contributions sur les vélos cargo ne sont citées par aucune fiche publiée.
5. Elle publie un message qui expose l'écart, avec la provenance de son propre calcul.
6. Elle demande un scrutin : désigner un autre exécutant, ou revenir au modèle précédent.
7. La communauté vote (scénario 5). La fiche de l'agent est mise à jour : nouvel exécutant, nouvelle clé, ancienne clé révoquée.
8. Le nouvel exécutant efface les fiches et les recalcule depuis les contributions, qui n'ont pas bougé.

## Diagramme de séquence Mermaid
```mermaid
sequenceDiagram
    participant Fa as Fatou
    participant P as Poste de Fatou
    participant I as Dépôt d'idées
    participant C as Dépôt de communauté
    participant M as Membres
    Fa->>P: Rejoue le pipeline localement
    P->>I: Lit les contributions
    P->>P: Produit ses fiches, les compare aux publiées
    P-->>Fa: Écarts constatés
    Fa->>C: Publie l'écart et demande un scrutin
    M->>C: Votent le changement d'exécutant
    C->>C: Met à jour la fiche de l'agent
    C-->>I: Fiches recalculées par le nouvel exécutant
```

Ce diagramme montre que changer de modèle ou d'exécutant est une décision votée, pas une migration. Il n'est tenable que parce que les fiches d'idées sont dérivées et sans poids : le jour où l'on voudrait voter sur une idée, tout ce raisonnement tomberait.

Cas d'utilisation : « Garantir la confiance sans serveur ». Retour à l'analyse : [[KR Analyse]].
