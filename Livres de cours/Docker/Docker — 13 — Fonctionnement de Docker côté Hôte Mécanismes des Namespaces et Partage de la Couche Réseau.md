---
schema_version: 1
uid: 01M1BQ62DWK79XQ98CV2DQA5ND
titre: "Docker — 13 — Fonctionnement de Docker côté Hôte Mécanismes des Namespaces et Partage de la Couche Réseau"
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
resume: "Chapitre 13 sur 17 du livre « Docker » : Fonctionnement de Docker côté Hôte : Mécanismes des Namespaces et Partage de la Couche Réseau. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Docker » — chapitre 13/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 12 — Comment le Mécanisme de Couches Influence-t-il les Stratégies de Déploiement en Continu (CI CD)|← 12 — Comment le Mécanisme de Couches Influence-t-il les Stratégies de Déploiement en Continu (CI CD)]] · [[Docker — 14 — Utilisation de Docker Compose|14 — Utilisation de Docker Compose →]]

# Fonctionnement de Docker côté Hôte : Mécanismes des Namespaces et Partage de la Couche Réseau
Docker est une plateforme de virtualisation légère qui utilise des fonctionnalités du noyau Linux pour isoler les applications dans des conteneurs. Ces conteneurs fonctionnent comme des instances isolées de systèmes d'exploitation, mais sans la surcharge d'une machine virtuelle complète. Le fonctionnement interne de Docker repose principalement sur deux mécanismes clés du noyau Linux :

1. **Les Namespaces** : Pour l'isolation des ressources.
2. **Les Groupes de Contrôle (cgroups)** : Pour la limitation et la surveillance des ressources.

Dans cette section, nous allons explorer en détail comment Docker utilise ces mécanismes, en mettant l'accent sur les namespaces et le partage de la couche réseau.

## Introduction aux Namespaces Linux

Les **namespaces** sont une fonctionnalité du noyau Linux qui permet d'isoler et de virtualiser les ressources du système pour un ensemble de processus. Chaque namespace crée une vue indépendante d'une ressource, ce qui signifie que les processus dans un namespace ne peuvent pas voir ou interagir directement avec les ressources en dehors de ce namespace.

### Types de Namespaces

Il existe plusieurs types de namespaces, chacun isolant un aspect spécifique du système :

1. **Mount Namespace (`mnt`)** : Isolent le système de fichiers montés.
2. **Process ID Namespace (`pid`)** : Isolent l'espace des identifiants de processus.
3. **Network Namespace (`net`)** : Isolent la pile réseau.
4. **Inter-Process Communication Namespace (`ipc`)** : Isolent les ressources IPC (mémoires partagées, sémaphores).
5. **UTS Namespace (`uts`)** : Isolent les noms de domaine et les noms d'hôte.
6. **User Namespace (`user`)** : Isolent les identifiants d'utilisateurs et de groupes.
7. **Cgroup Namespace (`cgroup`)** : Isolent la vue des cgroups.

Docker utilise ces namespaces pour créer des environnements isolés pour chaque conteneur.

## Fonctionnement des Namespaces dans Docker

### Mount Namespace

- **Isolation du Système de Fichiers** : Chaque conteneur voit un système de fichiers indépendant. Les changements dans le système de fichiers d'un conteneur n'affectent pas les autres conteneurs ou l'hôte.
- **Utilisation des Couches (Layers)** : Docker utilise un système de fichiers en couches pour construire le système de fichiers des conteneurs, comme expliqué précédemment.

### PID Namespace

- **Isolation des Processus** : Les processus à l'intérieur d'un conteneur ne voient que les processus de ce conteneur. Le PID 1 dans le conteneur est le processus principal du conteneur.
- **Avantages** :
  - Empêche les processus d'un conteneur d'interagir avec ceux d'un autre conteneur ou de l'hôte.
  - Facilite la gestion des processus au sein du conteneur.

### Network Namespace

- **Isolation Réseau** : Chaque conteneur possède sa propre pile réseau, avec ses propres interfaces réseau, tables de routage, ports, etc.
- **Communication Interne** : Les conteneurs ne peuvent pas communiquer directement avec les interfaces réseau de l'hôte ou d'autres conteneurs sans configuration explicite.
- **Avantages** :
  - Permet d'attribuer des adresses IP distinctes aux conteneurs.
  - Contrôle précis sur les règles de pare-feu et les politiques de routage.

### IPC Namespace

- **Isolation des Mécanismes IPC** : Les mécanismes de communication inter-processus (comme les mémoires partagées) sont isolés.
- **Sécurité** : Empêche les processus d'un conteneur d'accéder aux ressources IPC d'un autre conteneur ou de l'hôte.

### UTS Namespace

- **Isolation du Nom d'Hôte et du Domaine** : Chaque conteneur peut avoir son propre nom d'hôte et de domaine.
- **Personnalisation** : Utile pour les applications qui dépendent du nom d'hôte pour leur configuration ou leur fonctionnement.

### User Namespace

- **Isolation des Identifiants Utilisateurs** : Mappe les UID et GID du conteneur à ceux de l'hôte.
- **Sécurité Renforcée** : Permet aux processus dans le conteneur de s'exécuter en tant que root dans le conteneur, mais avec des privilèges limités sur l'hôte.

### Cgroup Namespace

- **Isolation des Cgroups** : Fournit une vue isolée des groupes de contrôle, améliorant la sécurité et la gestion des ressources.

## Les Groupes de Contrôle (cgroups)

En plus des namespaces, Docker utilise les **cgroups** pour limiter et surveiller l'utilisation des ressources par les conteneurs.

- **Limitation des Ressources** : CPU, mémoire, I/O disque, réseau.
- **Isolation** : Empêche un conteneur de consommer toutes les ressources de l'hôte.
- **Monitoring** : Permet de surveiller la consommation des ressources par chaque conteneur.

## Partage de la Couche Réseau

### Principe du Network Namespace

Par défaut, Docker crée un **network namespace** pour chaque conteneur, isolant ainsi la pile réseau du conteneur de celle de l'hôte et des autres conteneurs.

- **Interfaces Réseau Isolées** : Chaque conteneur a sa propre interface réseau virtuelle.
- **Tables de Routage Indépendantes** : Les règles de routage et les tables ARP sont propres au conteneur.

### Le Pont Réseau Docker (`docker0`)

- **Bridge Network par Défaut** : Docker crée un pont réseau virtuel nommé `docker0` sur l'hôte.
- **Connexions des Conteneurs** : Les interfaces réseau virtuelles des conteneurs sont connectées au pont `docker0`.
- **Fonctionnement** :
  - Lorsque nous lançons un conteneur, Docker crée une paire d'interfaces virtuelles (veth pairs).
  - Une extrémité est placée dans le network namespace du conteneur.
  - L'autre extrémité est attachée au pont `docker0` sur l'hôte.
- **Adresses IP** :
  - Docker attribue une adresse IP privée au conteneur sur le réseau du pont (`172.17.0.0/16` par défaut).
  - Les conteneurs peuvent communiquer entre eux via le pont.

### Port Mapping et NAT

- **Exposition des Ports** : Pour accéder à un service dans un conteneur depuis l'extérieur, nous devons mapper un port du conteneur à un port de l'hôte.
- **Attention à l'exposition des ports** : Si aucune adresse n'est précisée comme restriction, le port est accessible depuis l'extérieur de l'hôte.
- **Commande** :
```bash
docker run \
  -p <adresse_autorisée>:<port_hôte>:<port_conteneur> \
  --expose <port_exposé_aux_autres_conteneurs> \
  mon_image
```
- L'option `-p` expose un port sur l'interface réseau.
- L'option `--expose` partage un port avec les autres conteneurs.
- **NAT (Network Address Translation)** :
  - Docker configure des règles iptables pour effectuer la traduction d'adresses entre l'hôte et le conteneur.
  - Permet aux paquets entrants sur l'hôte d'être redirigés vers le conteneur.
  - `<adresse_autorisée>` est mis à 127.0.0.1 si l'on veut que seul l'hôte voit l'exposition du port du conteur.
### Redirection de port en interne du conteneur
Lorsque nous exposons les ports d'un conteneur, il peut être impératif de les router vers un autre port en interne du conteneur, sans passer par la redirection classique.


### Réseaux Personnalisés

- **Bridge Networks Personnalisés** :
  - Nous pouvons créer nos propres réseaux de ponts pour isoler des groupes de conteneurs.
  - Commande :
    ```bash
    docker network create mon_reseau
    ```
- **Avantages** :
  - Contrôle sur la configuration réseau (plage IP, DNS, etc.).
  - Isolation entre différents réseaux.

### Network Namespaces Partagés

- **Option `--network`** :
  - Nous pouvons configurer des conteneurs pour qu'ils partagent le même network namespace.
  - Permet à plusieurs conteneurs de partager la même pile réseau.
- **Cas d'Utilisation** :
  - Conteneurs légers qui doivent interagir étroitement.
  - Exécution de processus de surveillance ou de gestion du réseau.

### Modes Réseau Spéciaux

- **Host Networking** :
  - En utilisant `--network=host`, le conteneur partage le network namespace de l'hôte.
  - Pas d'isolation réseau : le conteneur utilise les interfaces réseau de l'hôte.
  - Avantage : Réduction de la latence réseau.
  - Inconvénient : Moins d'isolation, risques de sécurité accrus.

- **None Networking** :
  - En utilisant `--network=none`, le conteneur n'a pas de pile réseau.
  - Utile pour des conteneurs qui n'ont pas besoin de communiquer sur le réseau.

## Fonctionnement Global de Docker Côté Hôte

### Processus de Création d'un Conteneur

1. **Création des Namespaces** :
   - Docker demande au noyau Linux de créer les différents namespaces pour le conteneur.
   - Isolent les ressources système (PID, réseau, système de fichiers, etc.).

2. **Configuration des Cgroups** :
   - Docker configure les cgroups pour limiter l'utilisation des ressources par le conteneur.
   - Paramètres comme la limite de mémoire, le pourcentage de CPU, etc.

3. **Configuration du Système de Fichiers** :
   - Montage du système de fichiers du conteneur en utilisant le système de fichiers en couches.
   - Application des changements spécifiques au conteneur.

4. **Configuration du Réseau** :
   - Création d'un network namespace pour le conteneur.
   - Configuration des interfaces réseau virtuelles et des adresses IP.
   - Ajout de règles iptables pour le NAT et le port mapping.

5. **Démarrage du Processus Principal** :
   - Exécution du processus spécifié (CMD ou ENTRYPOINT) dans le conteneur.
   - Le processus s'exécute avec l'UID et le GID spécifiés, généralement avec des privilèges réduits.

### Interaction avec le Noyau

- **Isolation vs. Partage** :
  - Les conteneurs sont isolés au niveau utilisateur, mais partagent le même noyau Linux.
  - Les appels système sont gérés par le noyau de l'hôte.

- **Sécurité** :
  - L'utilisation de namespaces et de cgroups renforce la sécurité en isolant les conteneurs.
  - Cependant, une vulnérabilité au niveau du noyau peut affecter tous les conteneurs.

## Avantages du Mécanisme des Namespaces

- **Isolation** : Fournit un environnement isolé pour chaque conteneur.
- **Sécurité** : Réduit les risques de fuite d'informations entre conteneurs.
- **Gestion des Ressources** : Les cgroups permettent de contrôler finement l'utilisation des ressources.
- **Flexibilité** : Les conteneurs peuvent être configurés pour partager ou isoler certaines ressources selon les besoins.

## Limitations et Considérations

- **Partage du Noyau** :
  - Tous les conteneurs partagent le même noyau, ce qui peut être une source de vulnérabilités.
  - Les mises à jour du noyau affectent tous les conteneurs.

- **Complexité Réseau** :
  - La configuration réseau peut devenir complexe avec de nombreux conteneurs et réseaux personnalisés.
  - Les règles iptables générées par Docker peuvent interférer avec les configurations manuelles.

- **Limitations des Namespaces** :
  - Certaines fonctionnalités ne sont pas entièrement isolées, comme le noyau lui-même.
  - Les conteneurs ne sont pas des machines virtuelles complètes.

## Conclusion sur Docker côté hôte

Docker utilise les fonctionnalités avancées du noyau Linux, en particulier les namespaces et les cgroups, pour fournir un environnement isolé et contrôlé pour les conteneurs. Le mécanisme des namespaces permet une isolation fine des ressources, tandis que les cgroups assurent le contrôle et la limitation de l'utilisation des ressources système.

Le partage de la couche réseau est géré via les network namespaces et les interfaces réseau virtuelles, offrant une grande flexibilité dans la configuration réseau des conteneurs. En comprenant ces mécanismes, il est possible de tirer pleinement parti de Docker pour déployer des applications de manière efficace, sécurisée et scalable.

---

**Points Clés :**

- **Namespaces** : Mécanisme d'isolation des ressources au niveau du noyau Linux.
- **Cgroups** : Mécanisme de limitation et de surveillance des ressources.
- **Network Namespaces** : Chaque conteneur a sa propre pile réseau isolée.
- **Pont Réseau Docker (`docker0`)** : Connecte les interfaces réseau virtuelles des conteneurs.
- **Port Mapping et NAT** : Permettent l'accès aux services des conteneurs depuis l'extérieur.
- **Flexibilité Réseau** : Possibilité de créer des réseaux personnalisés et de configurer le partage du network namespace.
- **Sécurité et Isolation** : Les mécanismes utilisés par Docker renforcent la sécurité, mais des considérations restent nécessaires, notamment en matière de partage du noyau.

---

**Questions pour Approfondir :**

- Comment les cgroups sont-ils utilisés pour surveiller et limiter les ressources spécifiques comme le CPU et la mémoire ?
- Quelles sont les implications de la partage du noyau Linux entre l'hôte et les conteneurs en termes de sécurité ?
- Comment Docker Compose et Kubernetes gèrent-ils les configurations réseau avancées avec les namespaces ?

---
> [!info] Livre « Docker » — chapitre 13/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 12 — Comment le Mécanisme de Couches Influence-t-il les Stratégies de Déploiement en Continu (CI CD)|← 12 — Comment le Mécanisme de Couches Influence-t-il les Stratégies de Déploiement en Continu (CI CD)]] · [[Docker — 14 — Utilisation de Docker Compose|14 — Utilisation de Docker Compose →]]
