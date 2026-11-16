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
titre: 'KR Scénario 18 : Préparer et soumettre un amendement'
type: document
themes:
- genie-logiciel
- scenarios
- agents-ia
- git
- kuneagi
resume: 'Scénario de préparation d''un amendement : modifications faites dans sa copie, regroupement par intention avec l''aide d''un assistant local, justification et signature par l''auteur.'
schema_version: 1
uid: 01M7AK88XTVH4ZZWD4AHW8EVEZ
date_creation: 2026-11-13
langue: fr
date_modification: 2026-11-13
metadata_verifiees: false
---
# **Préparer et soumettre un amendement**

## Résumé
Le groupe travaille en mode avec amendements. Alice a modifié la proposition dans sa copie pour élargir le service aux vélos cargo, et en a profité pour reformuler quelques phrases. Avant de soumettre, elle doit regrouper ses modifications par intention et justifier chacune. Un assistant, qui tourne sur son poste et ne lit que sa copie, lui propose un regroupement ; Alice corrige, décide et signe.

## Acteurs en jeu
- **Alice** : l'auteure des amendements.
- **Assistant d'amendement** : un modèle de langage local, sans droit d'écriture dans les dépôts partagés.
- **Copie personnelle d'Alice**.
- **Dépôt du groupe**.
- **Vérificateur**.

## Prérequis
- L'itération en cours est en mode avec amendements et n'est pas échue.
- Alice est participante.
- Elle a modifié une ou plusieurs sections dans sa copie personnelle.

## Étapes du scénario
1. Alice termine ses modifications et ouvre l'écran « Mes amendements ».
2. Le poste affiche la liste de ses modifications, calculée par comparaison avec la version du groupe.
3. Alice demande l'aide de l'assistant.
4. L'assistant propose deux regroupements : « Étendre aux vélos cargo », qui réunit trois modifications dispersées, et « Reformulations », qui réunit les autres. Il propose pour chacun un premier jet de justification et les idées auxquelles le rattacher.
5. Alice déplace une modification d'un groupe à l'autre et réécrit la justification du premier.
6. Pour chaque amendement, le poste crée une fiche : intention, justification, sections visées avec leur empreinte de départ, idées liées.
7. Alice signe chaque fiche. L'assistant ne signe rien et ne soumet rien.
8. Le poste envoie les amendements au dépôt du groupe ; le vérificateur contrôle le schéma, la signature et les empreintes de départ. Les amendements sont à l'état `submitted` et resteront modifiables par leur auteure jusqu'à l'échéance.

## Diagramme de séquence Mermaid
```mermaid
sequenceDiagram
    participant Al as Alice
    participant L as Assistant d'amendement
    participant F as Copie personnelle
    participant G as Dépôt du groupe
    participant V as Vérificateur
    Al->>F: Modifie plusieurs sections
    Al->>L: Demande de l'aide
    L->>F: Lit les modifications
    L-->>Al: Propose regroupements et justifications
    Al->>F: Corrige les regroupements, réécrit
    Al->>F: Signe chaque fiche d'amendement
    F->>G: Envoie les amendements
    G->>V: Demande le contrôle
    V-->>G: Schéma, signature, empreintes valides
    Note over G: Amendements à l'état submitted
```

Ce diagramme illustre l'écran qui était le plus difficile de Nova-Ideo, et celui où un modèle de langage aide le plus. La règle « un amendement, une intention » est tenue par l'auteure : la machine propose un découpage, elle n'engage personne.

Cas d'utilisation : « Co-écrire une proposition en groupe de travail ». Retour à l'analyse : [[KR Analyse]].
