---
schema_version: 1
uid: "01M05DHB125ZSFYXVA05D73BFZ"
titre: "Index meetup"
type: index
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - meetup
resume: "Index thématique généré des notes du dossier meetup."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2026-08-16
date_modification: 2026-08-31
confidentialite: privee
publication: []
rag: false
metadata_verifiees: false
---

# meetup

Index régénéré par `make index` — ne pas éditer à la main. Vue graphique : [[_INDEX_meetup.excalidraw]].

## Analyse

- [[MU _Analyse]] — Index des documents d'analyse du meetup du 28 mars 2024.
- [[MU Analyse par chat GPT]] — Retranscription de la phase d'analyse menée conjointement avec ChatGPT selon la méthode 2TUP, de la génération du glossaire métier aux scénarios.
- [[MU Glossaire métier]] — Glossaire métier de l'étude de cas : Markdown, Obsidian et les autres termes du domaine.
- [[MU Scénario 1]] — Scénario de création d'une note Markdown dans Obsidian : acteurs, prérequis et déroulement.
- [[MU Scénario 2]] — Scénario de conversion de notes Markdown en HTML par un script Python.
- [[MU Scénario 3]] — Scénario d'ajout d'une ressource, image ou PDF, à une note existante.
- [[MU Scénario 4]] — Scénario de génération du site web complet à partir de l'ensemble des notes Markdown.
- [[MU Scénario 5]] — Scénario de consultation d'une note convertie en HTML sur le site généré.
- [[MU Scénario 6]] — Scénario de mise à jour d'une note dans Obsidian et de régénération automatique du site.
- [[MU Scénario 7]] — Scénario de gestion des erreurs rencontrées par le convertisseur lors du traitement d'un Markdown mal formé.

## Conception

- [[MU 05 Document de Conception]] — Document de conception de l'étude de cas : architecture du système et technologies retenues.

## Déroulé

- [[MU 00 Développement Logiciel avec l'IA]] — Support principal du meetup : potentiel de l'IA dans le développement logiciel, déroulé, prérequis et étude de cas.
- [[MU 01 Introduction]] — Introduction du meetup : clarification du terme intelligence artificielle et distinction entre IA générale et IA appliquée au développement.
- [[MU 02 Étude de cas]] — Présentation de l'étude de cas suivie de l'analyse au déploiement, en s'aidant au maximum des IA.
- [[MU 03 Sujet]] — Énoncé du sujet : développer un script Python publiant sur le web des notes Markdown issues d'Obsidian.
- [[MU 04 Cahier des charges]] — Cahier des charges de l'étude de cas de publication des notes Markdown.
- [[MU 70 RAG Retrieval-Augmented Generation]] — Introduction au RAG présentée lors du meetup : principe de la recherche documentaire préalable à la génération.
- [[MU 80 OpenAI API]] — Prise en main de l'API OpenAI : création d'une clé applicative et stockage sécurisé dans un fichier .env.
- [[MU 80 Prompt ChatGPT]] — Retranscription intégrale de la session de prompts ayant produit l'analyse, le glossaire et les scénarios de l'étude de cas.
- [[MU 90 Dialecte ChatGPT]] — Expérience et réflexion sur l'existence d'un dialecte propre à ChatGPT, à partir d'un prompt initial et de sa réponse.

## Développement

- [[MU 06 Organisation des développements]] — Génération assistée par IA de l'arborescence du projet, du README et des fichiers de dépendances.
- [[MU 07 Page HTML]] — Génération du HTML de l'interface à partir d'une maquette Excalidraw soumise à l'IA.
- [[MU 08 Fichiers Python]] — Génération des fichiers Python du projet selon le plan défini, puis reprise du travail dans VS Code.
