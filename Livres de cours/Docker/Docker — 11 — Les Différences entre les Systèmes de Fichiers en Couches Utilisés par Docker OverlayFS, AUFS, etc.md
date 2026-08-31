---
schema_version: 1
uid: 01M1BQ62DWQXHFRAADP2JCE7HW
titre: "Docker — 11 — Les Différences entre les Systèmes de Fichiers en Couches Utilisés par Docker OverlayFS, AUFS, etc"
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
resume: "Chapitre 11 sur 17 du livre « Docker » : Les Différences entre les Systèmes de Fichiers en Couches Utilisés par Docker : OverlayFS, AUFS, etc. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Docker » — chapitre 11/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 10 — L'Impact des Couches sur la Sécurité des Images Docker|← 10 — L'Impact des Couches sur la Sécurité des Images Docker]] · [[Docker — 12 — Comment le Mécanisme de Couches Influence-t-il les Stratégies de Déploiement en Continu (CI CD)|12 — Comment le Mécanisme de Couches Influence-t-il les Stratégies de Déploiement en Continu (CI CD) →]]

# Les Différences entre les Systèmes de Fichiers en Couches Utilisés par Docker : OverlayFS, AUFS, etc.
## Introduction

Docker utilise des **systèmes de fichiers en couches** pour gérer efficacement les images et les conteneurs. Ces systèmes permettent de superposer plusieurs systèmes de fichiers pour créer un système de fichiers unique, optimisant ainsi l'utilisation de l'espace disque et facilitant la gestion des images. Parmi les systèmes de fichiers en couches utilisés par Docker, les plus courants sont **OverlayFS**, **AUFS**, **Device Mapper**, **Btrfs** et **ZFS**. Chacun de ces systèmes a ses propres caractéristiques, avantages et inconvénients. Dans cette section, nous allons explorer en détail les différences entre ces systèmes, en mettant l'accent sur OverlayFS et AUFS.

## Pourquoi Docker Utilise des Systèmes de Fichiers en Couches

Les systèmes de fichiers en couches offrent plusieurs avantages clés pour Docker :

- **Efficacité du Stockage** : Évitent la duplication des données en réutilisant les couches communes entre les images.
- **Rapidité de Déploiement** : Permettent de télécharger et de charger uniquement les couches modifiées ou manquantes.
- **Isolation** : Fournissent un système de fichiers isolé pour chaque conteneur, améliorant la sécurité et la stabilité.
- **Flexibilité** : Facilitent la création et la gestion des images en empilant des modifications successives.

## Présentation des Systèmes de Fichiers en Couches

### AUFS (Advanced Multi-Layered Unification Filesystem)

- **Origine** : AUFS est un système de fichiers en couches pour Linux, initialement développé pour le projet Knoppix.
- **Caractéristiques** :
  - Permet de superposer plusieurs systèmes de fichiers en lecture seule avec un système de fichiers en lecture-écriture.
  - Supporte un grand nombre de couches (généralement jusqu'à 127 ou plus).
- **Support du Noyau** :
  - **Non inclus** dans le noyau Linux officiel.
  - Nécessite des patchs ou des modules externes pour être utilisé.
- **Utilisation dans Docker** :
  - Était le système de fichiers par défaut dans les premières versions de Docker.
  - Offrait une flexibilité et une compatibilité avec les noyaux plus anciens.

### OverlayFS

- **Origine** : OverlayFS est un système de fichiers en couches plus récent, conçu pour être simple et efficace.
- **Caractéristiques** :
  - Superpose une couche supérieure en lecture-écriture sur une couche inférieure en lecture seule.
  - Initialement limité à deux couches, mais le support pour plus de couches a été ajouté dans les versions ultérieures.
- **Support du Noyau** :
  - **Inclus** dans le noyau Linux principal depuis la version **3.18**.
  - Bénéficie du support de la communauté du noyau Linux.
- **Utilisation dans Docker** :
  - Est devenu le pilote de stockage par défaut dans les versions plus récentes de Docker.
  - Préféré en raison de sa performance et de sa simplicité.

## Comparaison entre AUFS et OverlayFS

### Support et Compatibilité

- **AUFS** :
  - Non intégré dans le noyau Linux officiel.
  - Peut nécessiter des distributions spécifiques ou des configurations personnalisées.
  - Moins de support à long terme en raison de sa dépendance aux mainteneurs externes.

- **OverlayFS** :
  - Intégré dans le noyau Linux officiel depuis la version 3.18.
  - Largement disponible sur les distributions modernes.
  - Bénéficie des mises à jour et du support de la communauté Linux.

### Performance

- **AUFS** :
  - Bonne performance, mais peut être affecté par sa complexité.
  - La gestion d'un grand nombre de couches peut entraîner une légère dégradation des performances.

- **OverlayFS** :
  - Performance généralement supérieure en raison de sa simplicité et de son intégration native.
  - Meilleure gestion du cache, ce qui améliore les opérations de lecture/écriture.

### Complexité et Maintenance

- **AUFS** :
  - Plus complexe à configurer et à maintenir.
  - La nécessité de patcher le noyau peut entraîner des problèmes de compatibilité.

- **OverlayFS** :
  - Simplicité de configuration.
  - Maintenance facilitée grâce à son inclusion dans le noyau principal.

### Nombre de Couches Supportées

- **AUFS** :
  - Supporte un grand nombre de couches (127 ou plus), ce qui peut être avantageux pour des images complexes.

- **OverlayFS** :
  - Initialement limité à deux couches, mais ce nombre a été augmenté dans les versions ultérieures.
  - Docker utilise des techniques pour combiner les couches et surmonter cette limitation.

### Disponibilité et Adoption

- **AUFS** :
  - Moins courant sur les distributions récentes.
  - Utilisé principalement lorsqu'OverlayFS n'est pas disponible.

- **OverlayFS** :
  - Devenu le standard de facto pour Docker sur Linux.
  - Large adoption dans la communauté en raison de ses avantages.

## Autres Systèmes de Fichiers en Couches

### Device Mapper

- **Fonctionnement** :
  - Utilise le sous-système Device Mapper du noyau Linux.
  - Crée des volumes logiques en utilisant LVM (Logical Volume Manager).

- **Avantages** :
  - Peut être utilisé sur des systèmes de fichiers qui ne supportent pas AUFS ou OverlayFS.
  - Offre des fonctionnalités avancées comme la gestion fine des volumes.

- **Inconvénients** :
  - Configuration plus complexe.
  - Performances moins bonnes dans certains cas par rapport à OverlayFS.

### Btrfs

- **Fonctionnement** :
  - Utilise les sous-volumes et les instantanés de Btrfs pour gérer les couches.

- **Avantages** :
  - Fonctionnalités avancées comme la compression, la déduplication, les instantanés.

- **Inconvénients** :
  - Stabilité variable selon les versions du noyau.
  - Nécessite que le système de fichiers de l'hôte soit Btrfs.

### ZFS

- **Fonctionnement** :
  - Utilise les capacités de ZFS pour la gestion des couches.

- **Avantages** :
  - Haute fiabilité, intégrité des données, fonctionnalités avancées.

- **Inconvénients** :
  - Non inclus dans le noyau Linux principal pour des raisons de licence.
  - Installation et configuration supplémentaires nécessaires.

## Historique et Évolution

- **Début avec AUFS** :
  - Docker a initialement choisi AUFS en raison de son support des systèmes de fichiers en couches et de sa disponibilité à l'époque.
  - Les limitations du noyau Linux ont rendu AUFS nécessaire malgré son absence du noyau principal.

- **Transition vers OverlayFS** :
  - Avec l'intégration d'OverlayFS dans le noyau Linux 3.18, Docker a commencé à le supporter.
  - OverlayFS est devenu le choix par défaut en raison de ses avantages en termes de performance et de maintenance.

- **Évolution des Besoins** :
  - Les améliorations apportées à OverlayFS ont permis de lever les limitations initiales (comme le nombre de couches).
  - La communauté a largement adopté OverlayFS, ce qui a renforcé son support et son développement.

## Facteurs à Considérer lors du Choix d'un Système de Fichiers

- **Compatibilité du Noyau** :
  - Assurons-nous que notre version du noyau Linux supporte le système de fichiers choisi.

- **Performance** :
  - OverlayFS offre généralement de meilleures performances pour la plupart des cas d'utilisation.

- **Complexité de Configuration** :
  - OverlayFS est plus simple à configurer que AUFS ou Device Mapper.

- **Fonctionnalités Spécifiques** :
  - Si nous avons besoin de fonctionnalités avancées (compression, déduplication), Btrfs ou ZFS peuvent être appropriés.

- **Stabilité et Support à Long Terme** :
  - Préférez les systèmes de fichiers inclus dans le noyau principal pour bénéficier du support continu.

## Recommandations Actuelles

- **OverlayFS** :
  - Recommandé pour la plupart des déploiements Docker sur Linux modernes.
  - Offre un bon équilibre entre performance, simplicité et support.

- **AUFS** :
  - Peut être utilisé sur des systèmes plus anciens où OverlayFS n'est pas disponible.
  - Moins recommandé en raison de sa complexité et de son manque de support dans le noyau principal.

- **Device Mapper** :
  - Utile dans des environnements spécifiques ou sur des systèmes de fichiers non supportés par OverlayFS.
  - Nécessite une configuration plus avancée.

- **Btrfs et ZFS** :
  - À considérer si nous avons des besoins spécifiques en termes de fonctionnalités de systèmes de fichiers.
  - Exigent que le système de fichiers de l'hôte soit compatible.

## Conclusion

Les différences entre les systèmes de fichiers en couches utilisés par Docker, tels que OverlayFS, AUFS, Device Mapper, Btrfs et ZFS, résident principalement dans leur intégration au noyau Linux, leurs performances, leur complexité et leurs fonctionnalités. **OverlayFS** est généralement préféré aujourd'hui en raison de sa simplicité, de ses bonnes performances et de son inclusion dans le noyau Linux principal. **AUFS**, bien qu'ayant été crucial dans les premières étapes de Docker, est moins utilisé aujourd'hui.

Le choix du système de fichiers dépendra de plusieurs facteurs :

- **Version du Noyau Linux** : Assurons-nous que notre noyau supporte le système de fichiers souhaité.
- **Besoins en Performance** : OverlayFS est performant pour la plupart des applications.
- **Fonctionnalités Requises** : Si nous avons besoin de fonctionnalités spécifiques, d'autres systèmes de fichiers peuvent être plus appropriés.
- **Complexité Acceptable** : Considérez la complexité de configuration et de maintenance que nous sommes prêt à gérer.

Il est important de tester et de valider le système de fichiers en couches dans notre environnement spécifique pour nous assurer qu'il répond à nos besoins en termes de performance, de stabilité et de fonctionnalités.

---

**Points Clés :**

- **OverlayFS** est le système de fichiers en couches par défaut et recommandé pour Docker sur les distributions Linux modernes.
- **AUFS** n'est plus le choix privilégié en raison de son absence du noyau principal et de sa complexité.
- **Le support dans le noyau Linux** est un facteur déterminant pour la stabilité et la maintenance à long terme.
- **Les performances** et la **simplicité** d'OverlayFS en font le choix idéal pour la plupart des utilisateurs.
- **Des alternatives** comme Device Mapper, Btrfs et ZFS sont disponibles pour des cas d'utilisation spécifiques mais nécessitent une expertise supplémentaire.

---
> [!info] Livre « Docker » — chapitre 11/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 10 — L'Impact des Couches sur la Sécurité des Images Docker|← 10 — L'Impact des Couches sur la Sécurité des Images Docker]] · [[Docker — 12 — Comment le Mécanisme de Couches Influence-t-il les Stratégies de Déploiement en Continu (CI CD)|12 — Comment le Mécanisme de Couches Influence-t-il les Stratégies de Déploiement en Continu (CI CD) →]]
