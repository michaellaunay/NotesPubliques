---
schema_version: 1
uid: 01M1BQ62D9G9BJ52DW3NKPV5PE
titre: "Pyramid — Sommaire"
aliases:
  - "Livre Pyramid"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - developpement-web
  - python
  - pyramid
resume: "Sommaire du livre « Pyramid » : 10 chapitres (16 005 mots), compléments 2026 et conclusion ; version longue du cours [[Pyramid]], découpée le 31 août 2026."
niveau: avance
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-06-14
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
> [!info] Livre de cours
> Version longue du cours [[Pyramid]], découpée en chapitres le 31 août 2026 à partir de l'état du dépôt au commit `5a5a9b2` (2026-08-18, environ 16 470 mots). La version condensée et actualisée reste dans `cours/` ; ses apports propres sont réunis dans [[Pyramid — Compléments 2026]].


# Pyramid — Sommaire


**1. Introduction à Pyramid et au développement web Python**
- Introduction à Pyramid : qu'est-ce que c'est, pourquoi l'utiliser.
- Historique de Pyramid et du projet Pylons.
- Installation et configuration de Pyramid.
- Structure de base d'une application Pyramid.
- notre première application Pyramid.

**2. Les Routes et Vues dans Pyramid**
- Comprendre le mécanisme de routage dans Pyramid.
- Créer et utiliser des vues.
- Utilisation des templates Chameleon pour le rendu des vues.

**3. Gestion des requêtes et réponses**
- Comprendre les requêtes GET et POST.
- Manipulation des données de la requête.
- Envoyer des réponses : différences entre les types de réponses et quand les utiliser.
- Utilisation des cookies pour maintenir l'état de la session.

**4. Introduction à l'authentification**
- Qu'est-ce que l'authentification et pourquoi est-elle importante ?
- Les mécanismes d'authentification dans Pyramid.
- Créer une politique d'authentification simple.

**5. Sécurisation des données**
- Comment protéger les données des utilisateurs.
- Hashage des mots de passe.
- Utilisation de pyramid.csrf pour la prévention des attaques CSRF.
- Validation et assainissement des entrées des utilisateurs.

**6. Introduction à LDAP et OpenLDAP**
- Qu'est-ce que LDAP ?

**7. Authentification LDAP**
- Interagir avec un serveur LDAP en Python.
- Intégration d'OpenLDAP avec Pyramid pour l'authentification.
- Gestion des erreurs d'authentification.

**8. Déploiement de l'application**
- Introduction à Docker.
- Création d'un Dockerfile pour l'application Pyramid.
- Déploiement de l'application en utilisant Docker.
- Surveillance et dépannage de l'application déployée.

**9. Pyramide et Docker**
- Dockerfile pour Pyramid

**10. Exemple de code**
- Stockage d'une session en ZODB
- Coupler l'authentification avec OpenLDAP
- Vérification des certificats

**11.  Trucs et astuces **
- Débogage en ligne à travers le navigateur

Ce plan de cours vise à donner une compréhension complète de la création d'une application d'authentification en utilisant Pyramid et OpenLDAP, du développement à la mise en production.


## Chapitres

1. [[Pyramid — 01 — Introduction à Pyramid et au développement web Python|Introduction à Pyramid et au développement web Python]] — 2 760 mots
2. [[Pyramid — 02 — Les Routes et Vues dans Pyramid|Les Routes et Vues dans Pyramid]] — 2 105 mots
3. [[Pyramid — 03 — Gestion des requêtes et réponses|Gestion des requêtes et réponses]] — 2 896 mots
4. [[Pyramid — 04 — Introduction à l'authentification|Introduction à l'authentification]] — 1 785 mots
5. [[Pyramid — 05 — Sécurisation des données|Sécurisation des données]] — 2 583 mots
6. [[Pyramid — 06 — Introduction à LDAP et OpenLDAP|Introduction à LDAP et OpenLDAP]] — 310 mots
7. [[Pyramid — 07 — Authentification LDAP|Authentification LDAP]] — 688 mots
8. [[Pyramid — 08 — Déploiement de l'application Pyramid|Déploiement de l'application Pyramid]] — 1 010 mots
9. [[Pyramid — 09 — Pyramid et Docker|Pyramid et Docker]] — 321 mots
10. [[Pyramid — 10 — Exemples de code|Exemples de code]] — 1 547 mots

## Compléments et conclusion

- [[Pyramid — Compléments 2026]] — sections propres à la version condensée de 2026
- Version condensée et actualisée : [[Pyramid]]
