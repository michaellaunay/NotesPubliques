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
titre: 'KR Scénario 43 : Suivre un groupe en observateur'
type: document
themes:
- genie-logiciel
- scenarios
- workflow
- kuneagi
resume: 'Scénario de l''observateur : un membre suit un groupe de travail sans y participer, lit le texte et les amendements, apporte des arguments au débat, sans voter.'
schema_version: 1
uid: 01M7AQA2F81BC8NFCY0JQCE0WC
date_creation: 2026-11-13
langue: fr
date_modification: 2026-11-13
metadata_verifiees: false
---
# **Suivre un groupe en observateur**

## Résumé
Émile s'intéresse à la proposition « Des vélos partagés » mais ne veut pas, ou ne peut pas, rejoindre le groupe. Il le suit en observateur : il lit le texte, les amendements et les discussions, apporte des arguments que le groupe prend en compte librement, et est alerté à chaque nouvelle version. Il ne vote pas.

## Acteurs en jeu
- **Émile** : l'observateur.
- **Application mobile**.
- **Dépôt du groupe**.
- **Les participants** du groupe.

## Prérequis
- Émile est membre.
- Le dépôt du groupe lui est lisible (groupe ouvert à la communauté).
- Les observateurs sont en nombre illimité ; toute personne en liste d'attente est observatrice.

## Étapes du scénario
1. Émile ouvre la proposition et touche « Suivre ».
2. Son application ajoute le dépôt du groupe à ses dépôts suivis. Ce choix n'est inscrit nulle part ailleurs.
3. Il lit le texte en cours, les amendements soumis et leur justification.
4. Il écrit un message sur une section : une information que le groupe n'avait pas. Son message est signé ; les applications le présentent comme venant d'un observateur.
5. Les participants en tiennent compte, ou non.
6. Une fois la proposition soumise à la communauté, Émile peut donner un retour plus structuré : dire quelle modification le ferait soutenir, ou retirer son soutien.
7. Il est alerté à chaque nouvelle version et à l'ouverture de chaque scrutin, auquel il ne participe pas.
8. S'il veut peser, il demande à participer (scénario 15).

## Diagramme de séquence Mermaid
```mermaid
sequenceDiagram
    participant E as Émile
    participant A as Application mobile
    participant G as Dépôt du groupe
    participant M as Participants
    E->>A: Touche Suivre
    A->>G: Ajoute le dépôt aux dépôts suivis
    G-->>A: Texte, amendements, discussions
    E->>A: Écrit un argument sur une section
    A->>G: Envoie le message signé
    M->>G: Lisent le message de l'observateur
    G-->>A: Nouvelle version
    A->>E: Alerte
```

Ce diagramme montre le rôle d'observateur des spécifications de KuneAgi. Il règle une tension du petit groupe : douze personnes décident, mais tous peuvent voir et nourrir le débat.

Cas d'utilisation : « Co-écrire une proposition en groupe de travail ». Retour à l'analyse : [[KR Analyse]].
