---
schema_version: 1
uid: 01M1BQ62DVY57GNR6D8KTJAXZ8
titre: "Docker — 04 — Terminologie"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - administration-systeme
  - conteneurisation
  - docker
  - devops
resume: "Chapitre 4 sur 17 du livre « Docker » : Terminologie. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
niveau: intermediaire
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-01-27
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre « Docker » — chapitre 4/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 03 — Installation|← 03 — Installation]] · [[Docker — 05 — Les commandes de bases|05 — Les commandes de bases →]]

# Terminologie
Dans le contexte de Docker, la terminologie "conteneur" et "image" est essentielle pour comprendre le fonctionnement des applications virtualisées. Voici une explication détaillée de ces deux concepts :

## Image Docker
Une **image** Docker est un package léger, autonome et exécutable qui inclut tout ce qui est nécessaire pour faire fonctionner un logiciel, y compris le code, les bibliothèques, les dépendances et les fichiers de configuration. En termes simples, une image est un modèle de base à partir duquel un conteneur peut être lancé.

**Caractéristiques d'une image :**
- **Immutable** : Une image est en lecture seule. Une fois créée, elle ne peut pas être modifiée. Si des modifications sont nécessaires, une nouvelle image doit être construite.
- **Empilable** : Les images Docker sont construites à partir de couches empilées. Chaque couche correspond à une modification ou à une instruction dans le `Dockerfile`, le script utilisé pour créer l'image.
- **Portable** : Une image peut être déplacée d'un environnement à un autre (comme un PC de développement, un serveur de production ou un cloud) et exécutée de manière cohérente.

## Conteneur Docker
Un **conteneur** Docker est une instance d'une image Docker. Lorsqu'une image est exécutée, elle devient un conteneur, qui est un environnement isolé où l'application peut s'exécuter de manière autonome.

**Caractéristiques d'un conteneur :**
- **Éphémère et modifiable** : Contrairement aux images, les conteneurs peuvent être modifiés en cours d'exécution. Cependant, les modifications apportées aux conteneurs ne persistent pas après leur suppression, sauf si elles sont sauvegardées dans une image mise à jour ou si des volumes externes sont utilisés.
- **Isolé** : Chaque conteneur fonctionne dans un espace isolé qui contient tout ce dont l'application a besoin, y compris un système de fichiers, des variables d'environnement et des bibliothèques. Cela permet d'éviter les conflits entre les applications qui tournent sur le même host.
- **Léger** : Les conteneurs partagent le noyau du système d'exploitation de l'hôte, ce qui les rend beaucoup plus légers que les machines virtuelles classiques. Cela leur permet de démarrer rapidement et d'utiliser moins de ressources.

## Différences entre Conteneur et Image
- **Image** : C’est un modèle statique et immuable qui définit comment un conteneur sera construit et quelles ressources il aura.
- **Conteneur** : C’est une instance active, exécutable et isolée de l’image. On peut créer plusieurs conteneurs à partir d'une seule image.
## Analogie
Imaginez qu'une image soit comme une classe dans un programme orienté objet, et qu'un conteneur soit comme un objet créé à partir de cette classe :
- **Image** = la classe (la définition de ce que sera l'objet).
- **Conteneur** = l'objet (une instance de la classe, qui peut être manipulée et utilisée).

---
> [!info] Livre « Docker » — chapitre 4/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 03 — Installation|← 03 — Installation]] · [[Docker — 05 — Les commandes de bases|05 — Les commandes de bases →]]
