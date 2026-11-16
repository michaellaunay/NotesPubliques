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
titre: 'KR Scénario 27 : Envoyer un message privé'
type: document
themes:
- genie-logiciel
- scenarios
- messagerie
- cryptographie
- kuneagi
resume: 'Scénario de message privé : chiffrement pour les clés publiques des destinataires, dépôt dans un dépôt partagé qui n''en révèle que l''existence et la taille.'
schema_version: 1
uid: 01M7AK88Z8GE2TGWV3T2XC99NE
date_creation: 2026-11-13
langue: fr
date_modification: 2026-11-13
metadata_verifiees: false
---
# **Envoyer un message privé**

## Résumé
Alice veut dire à Bob, sans que le groupe le lise, qu'elle trouve son amendement prématuré. Son application chiffre le message pour la clé publique de Bob et pour la sienne, puis le dépose. Le dépôt ne révèle que l'existence du message et sa taille.

## Acteurs en jeu
- **Alice** : l'expéditrice.
- **Bob** : le destinataire.
- **Applications** d'Alice et de Bob.
- **Dépôt du groupe** ou dépôt privé partagé entre les deux.

## Prérequis
- Alice et Bob sont membres ; leurs clés publiques de chiffrement figurent dans leurs fiches.
- Ils partagent au moins un dépôt.

## Étapes du scénario
1. Alice ouvre la fiche de Bob et touche « Message privé ».
2. Elle écrit son message.
3. L'application lit la clé publique de chiffrement de Bob dans sa fiche de membre.
4. Elle chiffre le message pour Bob et pour Alice, signe l'ensemble et dépose le fichier.
5. Le téléphone de Bob se synchronise et repère un message chiffré qui lui est destiné.
6. Il le déchiffre avec la clé privée de Bob, qui n'a jamais quitté le téléphone.
7. Bob lit le message et répond de la même façon.
8. Les autres membres voient qu'un fichier chiffré existe ; ils ne peuvent pas le lire.

## Diagramme de séquence Mermaid
```mermaid
sequenceDiagram
    participant Al as Alice
    participant AA as Application d'Alice
    participant G as Dépôt partagé
    participant AB as Application de Bob
    participant B as Bob
    Al->>AA: Écrit un message privé pour Bob
    AA->>G: Lit la clé publique de Bob
    AA->>AA: Chiffre pour Bob et Alice, signe
    AA->>G: Dépose le fichier chiffré
    AB->>G: Synchronise
    AB->>AB: Déchiffre avec la clé privée de Bob
    AB->>B: Affiche le message
```

Ce diagramme montre que le secret d'un message ne dépend pas de l'hébergeur. Il a deux limites à dire aux membres : l'existence de l'échange est visible, et qui perd sa clé perd ses messages (scénario 4).

Cas d'utilisation : « Discuter ». Retour à l'analyse : [[KR Analyse]].
