---
schema_version: 1
uid: 01M02JG1VF3CAAN45WM2GXPJ0K
titre: Présentation Nova-Ideo
aliases:
- Nova-Ideo
- KuneAgi
- Thèse d'Amen Souissi
type: fiche
statut: actif
para: ressource
domaines:
- enseignement
themes:
- informatique
- democratie-participative
- ingenierie-dirigee-par-les-modeles
- pyramid
- logiciel-libre
resume: "Présentation de Nova-Ideo, logiciel libre de démocratie participative issu de la thèse CIFRE d'Amen Souissi chez Ecréall (2013), de sa méthode de génération de portails à partir des processus métier, de ses déploiements (KuneAgi pour la coopérative Cosmopolitical, Engie), et de sa situation en 2026 : reprise par Logikascium et modernisation en cours."
auteurs:
- Michaël Launay
langue: fr
date_creation: 2023-04-16
date_modification: 2026-08-29
confidentialite: publique
publication:
- notes-publiques
rag: true
metadata_verifiees: false
---
# Présentation Nova-Ideo

> [!abstract] Objectif
> Retracer d'où vient Nova-Ideo — une thèse sur la génération de portails collaboratifs à partir des processus métier, puis un logiciel libre de démocratie participative — et situer son état en 2026, après la fermeture d'Ecréall et la reprise du code par Logikascium sous le nom de son déploiement historique, KuneAgi.

> [!info] État de la fiche
> Complétée le 29 août 2026 : corrections de forme (CIFRE, Nova-Ideo), ajout de la chronologie et de la situation actuelle. Les détails d'exploitation et les travaux en cours pour les clients sont dans les notes de prestation, non publiées.

Voir aussi : [[Pyramid]], [[ZC.Buildout]], [[UML Ecore EMF Plantuml QVT Mermaid PyEcore]] (génération à partir de modèles), [[OAuth OpenID]].

# 1. Origine : la thèse d'Amen Souissi (2013)

Amen Souissi, alors associé d'Ecréall, a soutenu en 2013 à l'Université Lille 1 une thèse CIFRE intitulée **« Modélisation centrée sur les processus métier pour la génération complète de portails collaboratifs »** (<https://hal.science/tel-00935324v1>). L'idée : décrire formellement les processus métier d'une organisation, puis **générer** l'intégralité du portail collaboratif — interfaces, workflows, formulaires, rapports — à partir de ces modèles, dans l'esprit de l'ingénierie dirigée par les modèles.

La preuve de concept fut une boîte à idées collaborative, première version de **KuneAgi** (« agir ensemble » en espéranto), fonctionnelle mais peu ergonomique. Ecréall en tira une leçon de méthode : plutôt que d'améliorer les générateurs jusqu'à obtenir une bonne application, écrire d'abord à la main l'application « parfaite » en Python/Pyramid, puis remonter les chaînes de transformation pour qu'elles produisent ce code — la génération suit l'application, et non l'inverse.

# 2. Nova-Ideo, logiciel libre de démocratie participative (2014-2023)

La boîte à idées réécrite a été généralisée en plateforme de démocratie participative sous le nom **Nova-Ideo** : dépôt d'idées, questions, propositions travaillées collectivement, amendements, votes, groupes de travail, notifications, modération, tout cela piloté par des **workflows** explicites.

Architecture : Python et **Pyramid**, persistance **ZODB**, moteur de workflows **dace** (*Data-Centric Engine*, issu de la thèse), bibliothèque d'interface **pontus**, formulaires Deform, déploiement par Buildout ; le code est publié sous licence libre sur GitHub (organisation `ecreall`).

Déploiements : plusieurs clients dont **Engie** (démarche participative interne), et **KuneAgi**, version spécifique pour le client historique, la coopérative **Cosmopolitical**, dont l'instance publique de délibération sur les politiques publiques est restée accessible. Le cœur de Nova-Ideo a aussi servi à une nouvelle version du portail de déclaration des spectacles de Sortir.eu, sans qu'un accord contractuel ait pu aboutir.

# 3. Situation en 2026

- **Ecréall** a cessé son activité en 2023 ; la propriété intellectuelle de Nova-Ideo, dace et pontus a été reprise en 2024 par **Logikascium**.
- **KuneAgi** reste le déploiement de référence ; une **modernisation** est engagée depuis 2025 (Python 3, Pyramid 2, mise à jour de dace et pontus, refonte du déploiement et de l'authentification), par étapes, en maintenant le service.
- Les principes de la thèse restent d'actualité : workflows explicites et données centrées sur les processus sont exactement ce que réclament aujourd'hui les plateformes de délibération et les agents logiciels ; la génération complète, elle, a été supplantée par les cadres applicatifs modernes et, depuis 2024, par la génération de code assistée.

# 4. Ce que Nova-Ideo enseigne

- Une application est un **ensemble de processus** avant d'être un ensemble d'écrans : modéliser le cycle de vie d'une proposition (dépôt, examen, amendement, vote, publication) fixe le reste.
- La **génération** à partir de modèles vaut pour les parties régulières (formulaires, transitions, permissions) et se paie en lisibilité pour les parties singulières : d'où le renversement méthodologique de 2013.
- Un logiciel libre survit à l'entreprise qui l'a créé si son code, ses données et ses déploiements sont documentés — c'est le sens des notes de ce coffre.

# 5. Sources

- Thèse d'Amen Souissi (2013) sur HAL : <https://hal.science/tel-00935324v1>
- Dépôts Nova-Ideo, dace et pontus : <https://github.com/ecreall>
- Coopérative Cosmopolitical : <https://www.cosmopolitical.coop/>
