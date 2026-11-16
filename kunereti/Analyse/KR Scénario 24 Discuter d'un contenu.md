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
titre: 'KR Scénario 24 : Discuter d''un contenu'
type: document
themes:
- genie-logiciel
- scenarios
- messagerie
- kuneagi
resume: 'Scénario de discussion : un message est un fichier signé qui cite l''objet dont il parle, et un fil est l''ensemble des messages qui citent le même objet.'
schema_version: 1
uid: 01M7AK88YTJ8FM1SMDK3AP7706
date_creation: 2026-11-13
langue: fr
date_modification: 2026-11-13
metadata_verifiees: false
---
# **Discuter d'un contenu**

## Résumé
Bob ne comprend pas le second paragraphe de la section « Financement ». Il écrit un message sur cette section ; Alice lui répond. Chaque message est un petit fichier signé qui dit de quoi il parle et à quoi il répond. Ce n'est pas une messagerie instantanée : un message part à la synchronisation de l'un et arrive à celle de l'autre.

## Acteurs en jeu
- **Bob** : l'auteur du message.
- **Alice** : celle qui répond.
- **Applications** de Bob et d'Alice.
- **Dépôt du groupe** : porte les messages.

## Prérequis
- Bob et Alice ont accès au dépôt du groupe.
- La section discutée existe.

## Étapes du scénario
1. Bob ouvre la section « Financement » et touche « Discuter ».
2. L'application affiche le fil : tous les messages qui citent cette section, dans l'ordre.
3. Bob écrit : « Qui paie la première année ? » Il peut joindre une image ou un lien.
4. L'application crée le message, le signe et l'envoie à la prochaine synchronisation. Deux messages écrits en même temps ne se gênent pas : chacun est un fichier distinct.
5. Le téléphone d'Alice se synchronise ; elle est notifiée parce qu'elle a rédigé cette section.
6. Alice répond. Son message cite celui de Bob.
7. À sa synchronisation suivante, Bob lit la réponse.
8. Le fil reste attaché à la section, y compris dans les copies du dépôt.

## Diagramme de séquence Mermaid
```mermaid
sequenceDiagram
    participant B as Bob
    participant AB as Application de Bob
    participant G as Dépôt du groupe
    participant AA as Application d'Alice
    participant Al as Alice
    B->>AB: Écrit un message sur la section
    AB->>AB: Signe le message
    AB->>G: Envoie à la synchronisation
    AA->>G: Synchronise
    AA->>Al: Notifie le nouveau message
    Al->>AA: Répond
    AA->>G: Envoie la réponse signée
    AB->>G: Synchronise
    AB->>B: Affiche la réponse
```

Ce diagramme illustre un débat au rythme du courrier, pas à celui du clavardage. Aucun canal de discussion n'est créé ni stocké : le fil est une simple requête sur ce que citent les messages.

Cas d'utilisation : « Discuter ». Retour à l'analyse : [[KR Analyse]].
