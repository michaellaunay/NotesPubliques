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
titre: 'KR Scénario 26 : Synthétiser un fil de discussion'
type: document
themes:
- genie-logiciel
- scenarios
- agents-ia
- llm
- kuneagi
resume: 'Scénario de synthèse d''un long fil par un agent : positions en présence et points non couverts par un amendement, sans recommandation de vote.'
schema_version: 1
uid: 01M7AK88Z35DWBPCRAYKAM1GXP
date_creation: 2026-11-13
langue: fr
date_modification: 2026-11-20
metadata_verifiees: false
---
# **Synthétiser un fil de discussion**

## Résumé
Le fil sur le financement dépasse soixante messages. L'agent synthétiseur en rédige un résumé : les positions en présence, qui les tient, et ce qu'aucun amendement ne couvre encore. La synthèse est toujours affichée comme écrite par une machine. Elle n'arbitre pas et ne recommande aucun vote.

## Acteurs en jeu
- **Agent synthétiseur** : un modèle de langage dont le périmètre d'écriture est limité aux synthèses.
- **Exécutant** : fait tourner l'agent.
- **Dépôt du groupe**.
- **Vérificateur**.
- **Les participants** : lecteurs de la synthèse.

## Prérequis
- La fiche de l'agent existe dans le dépôt de communauté.
- Le fil a dépassé le seuil de messages fixé par le groupe, ou une itération s'achève.

## Étapes du scénario
1. L'exécutant lance l'agent sur le dépôt du groupe.
2. L'agent lit les messages du fil et les amendements de l'itération, comme des données.
3. Il dégage les positions et rattache à chacune les messages qui la portent.
4. Il relève les objections restées sans réponse et les points qu'aucun amendement ne traite.
5. Il rédige la synthèse, avec les références aux messages et sa provenance (modèle, version, date), et la signe de sa clé.
6. Il la propose au dossier des synthèses ; le vérificateur contrôle qu'il n'a écrit nulle part ailleurs.
7. L'application affiche la synthèse en tête du fil, sous l'avertissement « Synthèse rédigée par une machine ».
8. Un participant qui la trouve infidèle le dit dans le fil ; l'agent en tiendra compte à son prochain passage.

## Diagramme de séquence Mermaid
```mermaid
sequenceDiagram
    participant X as Exécutant
    participant S as Agent synthétiseur
    participant G as Dépôt du groupe
    participant V as Vérificateur
    participant M as Participants
    X->>S: Lance la synthèse du fil
    S->>G: Lit les messages et les amendements
    S->>S: Dégage les positions, relève les manques
    S->>G: Propose la synthèse signée
    G->>V: Demande le contrôle
    V-->>G: Écriture dans le périmètre de l'agent
    M->>G: Synchronisent
    G-->>M: Synthèse marquée comme écrite par une machine
```

Ce diagramme illustre la frontière posée aux agents : ils résument, ils n'arbitrent pas. Sans modèle de langage, le groupe perd les synthèses, pas la délibération.

Cas d'utilisation : « Discuter ». Retour à l'analyse : [[KR Analyse]].
