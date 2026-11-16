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
titre: 'KR Scénario 02 : Rejoindre une communauté'
type: document
themes:
- genie-logiciel
- scenarios
- cryptographie
- kuneagi
resume: 'Scénario d''entrée dans une communauté Kunereti : scan du QR code, vérification de la clé racine, création des clés et dépôt de la candidature.'
schema_version: 1
uid: 01M7AK88VB6361YB6VX9252C6Y
date_creation: 2026-11-13
langue: fr
date_modification: 2026-11-13
metadata_verifiees: false
---
# **Rejoindre une communauté**

## Résumé
Chloé a reçu le QR code de la Coopérative des Tilleuls. Elle le scanne avec l'application mobile, qui clone le dépôt de communauté, vérifie l'empreinte de la clé racine, crée ses clés et dépose sa candidature. Chloé peut lire tout de suite ; elle ne pourra écrire qu'une fois admise.

## Acteurs en jeu
- **Chloé** : la candidate.
- **Application mobile** : cache entièrement Git et les clés.
- **Dépôt de communauté** : reçoit la candidature.

## Prérequis
- Chloé a installé l'application Kunereti sur son téléphone.
- Elle dispose du QR code de la communauté (affiche, courriel, écran d'un membre).
- La communauté accepte les candidatures (règle de la charte).

## Étapes du scénario
1. Chloé ouvre l'application et choisit « Rejoindre une communauté ».
2. Elle scanne le QR code. L'application y lit l'adresse du dépôt et l'empreinte de la clé racine.
3. L'application clone le dépôt de communauté et vérifie que la notice est bien signée par la clé dont l'empreinte figure dans le QR code. Si ce n'est pas le cas, elle s'arrête et le dit.
4. Chloé lit la charte et l'accepte.
5. Elle choisit un pseudonyme. L'application génère ses deux paires de clés et les range dans le coffre du téléphone ; elle lui fait noter une phrase de secours.
6. L'application crée la fiche de membre de Chloé (pseudonyme et clés publiques, aucune donnée d'identité), à l'état `applied`, la signe et la propose au dépôt de communauté par une demande de fusion.
7. Le vérificateur contrôle le schéma et la signature ; la candidature est fusionnée.
8. L'application affiche « Candidature déposée, en attente de vérification » et ouvre l'accueil en lecture seule.

## Diagramme de séquence Mermaid
```mermaid
sequenceDiagram
    participant Ch as Chloé
    participant A as Application mobile
    participant C as Dépôt de communauté
    participant V as Vérificateur
    Ch->>A: Scanne le QR code
    A->>C: Clone le dépôt
    A->>A: Vérifie la signature de la notice (empreinte de la clé racine)
    A->>Ch: Affiche la charte
    Ch->>A: Accepte, choisit un pseudonyme
    A->>A: Génère les clés, les range dans le coffre
    A->>C: Propose la fiche de membre signée (applied)
    C->>V: Demande le contrôle
    V-->>C: Schéma et signature valides
    A->>Ch: Candidature déposée, lecture ouverte
```

Ce diagramme illustre le remplacement de l'installation et de l'inscription de 2017 par un seul geste : scanner, cloner, vérifier. La communauté ne reçoit qu'un pseudonyme et des clés publiques. La suite, l'admission par les pairs, fait l'objet du scénario 3.

Cas d'utilisation : « Faire vivre la communauté ». Retour à l'analyse : [[KR Analyse]].
