---
schema_version: 1
uid: 01M1BQ62D4ECDH3MF5E88P34V6
titre: "Pyramid — 06 — Introduction à LDAP et OpenLDAP"
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
resume: "Chapitre 6 sur 10 du livre « Pyramid » : Introduction à LDAP et OpenLDAP. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « Pyramid » — chapitre 6/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 05 — Sécurisation des données|← 05 — Sécurisation des données]] · [[Pyramid — 07 — Authentification LDAP|07 — Authentification LDAP →]]

# 6. Introduction à LDAP et OpenLDAP
## 6.1 Qu'est-ce que LDAP ?

LDAP, pour Lightweight Directory Access Protocol, est un protocole standard ouvert, largement utilisé pour accéder et gérer les services d'annuaire sur Internet. Il est utilisé pour stocker, organiser et récupérer des informations complexes sur les utilisateurs et les ressources à partir d'un annuaire centralisé.

Dans le contexte de notre cours, nous pouvons imaginer LDAP comme un système pour gérer les informations d'authentification et d'autorisation de nos utilisateurs. Au lieu de stocker ces informations dans notre base de données Pyramid, nous pourrions les stocker dans un annuaire LDAP.

Dans LDAP, les données sont organisées de manière hiérarchique et structurée, ce qui facilite l'organisation et la recherche d'informations. Chaque entrée dans l'annuaire LDAP est identifiée de manière unique par son DN (Distinguished Name), qui suit une structure hiérarchique.

Quelques concepts clés à comprendre avec LDAP sont :

1. **Distinguished Name (DN)** : Il s'agit d'un nom unique qui identifie une entrée dans l'annuaire LDAP. Par exemple, `cn=John Doe,ou=users,dc=example,dc=com`.

2. **Common Name (CN)** : Il s'agit du nom de l'entrée, par exemple, "John Doe" dans l'exemple ci-dessus.

3. **Organizational Unit (OU)** : Il s'agit d'une subdivision au sein de l'organisation, par exemple, "users" dans l'exemple ci-dessus.

4. **Domain Component (DC)** : Il s'agit du domaine de niveau supérieur, par exemple, "example.com" dans l'exemple ci-dessus.

Nous explorerons plus en détail LDAP dans la note [[LDAP]] et son implémentation spécifique, OpenLDAP. 

Pour nos exercices de réflexion nous allons utiliser les données utilisateurs suivantes :
	- Noms,
	- Prénoms,
	- Date de naissance,
	- Nationalité,
	- Numéro de Coopérateur (attribué par la plate-forme)
	- Pseudonyme,
	- Adresse de courriel,
	- Langue d’interaction de rang 1
	- Langue d’interaction de rang 2
	- Prénom d’usage,
	- Nom d’usage,
	- Code postal,
	- Ville
	- pays de la résidence principale
	- Texte de profil utilisateur
	- Image de profil utilisateur / avatar

---
> [!info] Livre « Pyramid » — chapitre 6/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 05 — Sécurisation des données|← 05 — Sécurisation des données]] · [[Pyramid — 07 — Authentification LDAP|07 — Authentification LDAP →]]
