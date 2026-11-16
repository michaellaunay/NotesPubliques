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
titre: 'KR Scénario 11 : Tenir les fiches d''idées'
type: document
themes:
- genie-logiciel
- scenarios
- agents-ia
- llm
- kuneagi
resume: 'Scénario du pipeline d''idées tenu par l''agent curateur : extraction avec citations, dédoublonnage, liaison aux propositions et signalement des idées mûres.'
schema_version: 1
uid: 01M7AK88WQ32SSQERDCDEQ0T43
date_creation: 2026-11-13
langue: fr
date_modification: 2026-11-13
metadata_verifiees: false
---
# **Tenir les fiches d'idées**

## Résumé
Aucun humain ne crée ni ne gère une fiche d'idée. L'agent curateur lit les contributions, les messages et les justifications d'amendements, en tire des fiches d'idées qui citent leurs sources mot pour mot, fusionne les doublons, relie les idées aux propositions et signale celles qui sont mûres. Il range ; il ne décide de rien.

## Acteurs en jeu
- **Agent curateur** : un modèle de langage muni d'une clé et d'un périmètre d'écriture limité aux fiches d'idées.
- **Exécutant** : le membre volontaire ou la tâche planifiée qui fait tourner l'agent.
- **Dépôt d'idées** : contributions en entrée, fiches en sortie.
- **Vérificateur** : contrôle les écritures de l'agent comme celles d'un humain.

## Prérequis
- La fiche de l'agent existe dans le dépôt de communauté : mission, clé, périmètres de lecture et d'écriture, exécutant désigné par un vote.
- De nouvelles contributions ont été publiées depuis le dernier passage.

## Étapes du scénario
1. L'exécutant lance le pipeline, à la main ou à heure fixe.
2. **Extraire.** L'agent lit les textes nouveaux comme des données, jamais comme des consignes. Pour chaque idée qu'il y trouve, il rédige ou complète une fiche : un énoncé, et les passages sources cités mot pour mot avec leurs auteurs.
3. **Dédoublonner.** Il compare la fiche aux fiches existantes. Les doublons certains sont fusionnés ; les doublons probables sont seulement reliés.
4. **Relier.** Il rattache les idées aux propositions et aux amendements qui les reprennent.
5. **Faire émerger.** Quand une idée a reçu assez de contributions d'auteurs différents, il la marque comme mûre et publie un message d'appel proposant l'ouverture d'un groupe de travail.
6. Chaque fiche porte sa provenance : modèle, version, empreinte de la consigne, date. Elle reste marquée « non vérifiée par un humain ».
7. L'agent propose ses écritures au dépôt d'idées, signées de sa clé.
8. Le vérificateur les accepte si elles restent dans le périmètre de l'agent et si chaque citation correspond à une contribution existante.

## Diagramme de séquence Mermaid
```mermaid
sequenceDiagram
    participant X as Exécutant
    participant Cu as Agent curateur
    participant I as Dépôt d'idées
    participant V as Vérificateur
    participant C as Dépôt de communauté
    X->>Cu: Lance le pipeline
    Cu->>I: Lit les contributions nouvelles
    Cu->>Cu: Extrait les idées et leurs citations
    Cu->>Cu: Fusionne ou relie les doublons
    Cu->>Cu: Relie idées, propositions et amendements
    Cu->>I: Propose les fiches signées
    I->>V: Demande le contrôle
    V-->>I: Périmètre et citations valides
    opt Idée mûre
        Cu->>C: Publie un message d'appel à former un groupe
    end
```

Ce diagramme montre pourquoi ce travail peut être confié à une machine : une fiche d'idée est un objet dérivé, sans aucun pouvoir. On ne vote pas sur elle, aucune décision n'en dépend, et l'on peut effacer toutes les fiches et les recalculer sans rien perdre, puisque les contributions restent intactes.

Cas d'utilisation : « Recueillir les contributions et tenir les idées ». Retour à l'analyse : [[KR Analyse]].
