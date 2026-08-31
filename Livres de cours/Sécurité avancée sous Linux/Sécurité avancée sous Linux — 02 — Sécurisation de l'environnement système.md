---
schema_version: 1
uid: 01M1BQ62C664VTHK0JXP473WSZ
titre: "Sécurité avancée sous Linux — 02 — Sécurisation de l'environnement système"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - securite
  - administration-systeme
  - gnu-linux
  - cryptographie
resume: "Chapitre 2 sur 7 du livre « Sécurité avancée sous Linux » : Sécurisation de l'environnement système. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-15."
niveau: avance
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2024-03-11
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre « Sécurité avancée sous Linux » — chapitre 2/7
> [[Sécurité avancée sous Linux — Sommaire|Sommaire]] · [[Sécurité avancée sous Linux — 01 — Compréhension des bases de la sécurité Linux|← 01 — Compréhension des bases de la sécurité Linux]] · [[Sécurité avancée sous Linux — 03 — Contrôle d'accès avancé|03 — Contrôle d'accès avancé →]]

# 2. Sécurisation de l'environnement système
## 2.1. Installation et maintenance sécurisées

L'installation et la maintenance sécurisées d'un système Linux sont des étapes fondamentales pour assurer la robustesse et la résilience face aux menaces de sécurité. Cette approche commence dès le premier déploiement du système et se poursuit tout au long de son cycle de vie. Appliquer des pratiques d'installation et de maintenance sécurisées est crucial pour prévenir les vulnérabilités et protéger les données et les services contre les exploits et les intrusions.

### 2.1.1. **Installation Sécurisée**

- **Choix de la distribution :** Opter pour une distribution avec un bon historique de sécurité et un support actif. Certaines distributions sont spécifiquement conçues avec la sécurité en tête, comme Debian, CentOS (et son successeur Rocky Linux), ou Fedora.
  
- **Minimisation de l'installation :** Installer uniquement les paquets nécessaires à l'utilisation prévue du système. Moins il y a de composants installés, moins il y a de surface d'attaque pour les potentiels exploitants.

- **Partitionnement sécurisé :** Utiliser un schéma de partitionnement qui prend en compte la sécurité, tel que séparer `/home`, `/tmp`, et `/var` dans des partitions distinctes pour limiter les dommages en cas d'exploitation. Envisager le chiffrement des partitions sensibles, en particulier `/home` pour protéger les données utilisateur.

- **Configuration sécurisée du réseau :** Désactiver les services réseau non nécessaires et sécuriser ceux qui doivent être actifs. Appliquer des règles de pare-feu strictes dès le départ.

### 2.1.2. **Maintenance Sécurisée**

- **Mises à jour régulières :** Appliquer régulièrement les mises à jour de sécurité pour le système d'exploitation et tous les logiciels installés. Configurer si possible des mises à jour automatiques pour s'assurer que le système reste protégé contre les vulnérabilités récemment découvertes.

- **Gestion des droits d'accès :** Réexaminer périodiquement les droits d'accès aux fichiers et aux répertoires pour s'assurer qu'ils suivent le principe du moindre privilège. Utiliser des outils comme `find` pour identifier les fichiers avec des permissions potentiellement dangereuses.

- **Suivi des modifications de configuration :** Utiliser des outils de gestion de la configuration comme Ansible, Chef, ou Puppet pour automatiser et suivre les modifications apportées aux configurations système. Cela aide à maintenir la cohérence de la sécurité à travers l'environnement.

- **Utilisation de logiciels de sécurité :** Installer et configurer des outils de sécurité supplémentaires, tels que des systèmes de détection d'intrusion (IDS), des scanners de vulnérabilités et des outils d'audit de sécurité pour surveiller et évaluer régulièrement la sécurité du système.

- **Audits de sécurité :** Effectuer régulièrement des audits de sécurité pour identifier et corriger les vulnérabilités. Cela peut inclure des analyses de vulnérabilité, des tests de pénétration et la révision des politiques de sécurité.

- **Formation et sensibilisation :** Assurer que les utilisateurs et les administrateurs sont formés aux meilleures pratiques de sécurité et conscients des menaces actuelles. La sensibilisation à la sécurité est essentielle pour prévenir les erreurs humaines pouvant compromettre la sécurité du système.

L'installation et la maintenance sécurisées sont des processus continus qui exigent une attention et une adaptation constantes aux nouvelles menaces et vulnérabilités. En adoptant une approche proactive et en suivant les meilleures pratiques, il est possible de créer et de maintenir un environnement Linux sécurisé et résilient.
## 2.2. Gestion des packages et des mises à jour de sécurité
### 2.2. Gestion des Packages et des Mises à Jour de Sécurité

La gestion des packages et des mises à jour de sécurité est un aspect crucial de la maintenance d'un système Linux sécurisé. Les mises à jour de sécurité corrigent les vulnérabilités dans le système d'exploitation et les applications, réduisant ainsi le risque d'exploits et d'attaques. Une gestion efficace des packages et des mises à jour assure que le système reste à jour avec les dernières protections de sécurité.

#### **Stratégies de Mise à Jour**

- **Mises à jour automatiques :** Configurer le système pour appliquer automatiquement les mises à jour de sécurité peut aider à s'assurer que les corrections sont appliquées dès qu'elles sont disponibles. Cela est particulièrement important pour les systèmes critiques où le retard dans l'application d'une mise à jour peut exposer le système à des risques significatifs.

- **Planification des mises à jour :** Pour les environnements où les mises à jour automatiques ne sont pas souhaitables, établir un calendrier régulier de mises à jour et s'y tenir. Cela inclut la vérification des bulletins de sécurité et l'application des mises à jour dans un délai approprié.

- **Test des mises à jour :** Avant de déployer des mises à jour sur des systèmes en production, les tester dans un environnement de développement ou de test pour s'assurer qu'elles ne causent pas de problèmes de compatibilité ou de fonctionnement.

#### **Gestion des Packages**

- **Utiliser les dépôts officiels :** Télécharger les packages et les mises à jour uniquement à partir de dépôts officiels ou de sources fiables pour éviter les logiciels malveillants. Éviter l'utilisation de dépôts non officiels ou de packages téléchargés de sources inconnues.

- **Vérification de l'intégrité et de l'authenticité :** S'assurer que les mécanismes de vérification des signatures sont en place pour confirmer l'intégrité et l'authenticité des packages téléchargés. Cela aide à prévenir l'installation de logiciels modifiés ou malveillants.

- **Minimisation des logiciels installés :** Maintenir une politique de minimisation des logiciels en n'installant que les packages nécessaires pour les fonctions requises. Cela réduit la surface d'attaque potentielle du système.

#### **Outils et Commandes**

- **apt / apt-get (Debian, Ubuntu) :** Outils pour gérer les packages et les mises à jour sur les systèmes basés sur Debian. Utiliser `apt-get update` pour rafraîchir la liste des packages et `apt-get upgrade` pour mettre à jour les packages installés.

- **yum / dnf (Fedora, CentOS) :** Outils pour les systèmes basés sur RPM. Utiliser `yum update` ou `dnf update` pour appliquer les mises à jour disponibles.

- **zypper (openSUSE) :** Outil de gestion des packages pour openSUSE. Utiliser `zypper refresh` pour mettre à jour la liste des packages et `zypper update` pour mettre à jour les packages.

#### **Best Practices**

- **Surveillance des bulletins de sécurité :** Rester informé des derniers bulletins de sécurité pour les logiciels utilisés. Les distributions Linux et les développeurs de logiciels publient régulièrement des informations sur les vulnérabilités et les mises à jour de sécurité.

- **Formation et sensibilisation :** Sensibiliser les utilisateurs et les administrateurs aux meilleures pratiques de gestion des mises à jour et des packages, y compris l'importance de maintenir le système à jour.

La gestion proactive des packages et des mises à jour de sécurité est fondamentale pour protéger un système Linux contre les menaces existantes et émergentes. En adoptant une approche structurée et en suivant les meilleures pratiques, les administrateurs peuvent s'assurer que leur système est sécurisé, stable et à jour.

## 2.3. Sécurisation du bootloader et du kernel

La sécurisation du bootloader et du kernel est cruciale pour assurer l'intégrité et la sécurité globale d'un système Linux. Le bootloader, qui charge le système d'exploitation lors du démarrage de l'ordinateur, et le kernel, qui est le cœur du système d'exploitation, doivent être protégés contre les modifications non autorisées et les exploits. Voici comment renforcer la sécurité à ces niveaux critiques.

### 2.3.1. Sécurisation du Bootloader

Le bootloader, tel que GRUB (GRand Unified Bootloader), est souvent la première cible des attaquants car il s'exécute avant le système d'exploitation et a le potentiel de contourner les mesures de sécurité du système. Pour le sécuriser :

- **Mot de passe du bootloader :** Configurer un mot de passe pour le bootloader empêche les utilisateurs non autorisés de modifier les paramètres de démarrage ou d'accéder aux modes de récupération qui pourraient être utilisés pour compromettre le système. Dans GRUB, cela peut être accompli en ajoutant un mot de passe hashé dans le fichier de configuration `/etc/grub.d/40_custom` et en exécutant `update-grub`.

- **Utilisation de Secure Boot :** Secure Boot est une fonctionnalité de sécurité UEFI qui assure que seul le logiciel signé numériquement par une clé de confiance peut être exécuté au démarrage. Cela empêche l'exécution de logiciels malveillants au niveau du bootloader.

### 2.3.2. Sécurisation du Kernel

Le kernel, en tant que noyau du système d'exploitation, doit également être sécurisé pour prévenir les exploits qui pourraient compromettre l'ensemble du système.

- **Mises à jour régulières :** Appliquer les mises à jour du kernel dès qu'elles sont disponibles. Les mises à jour du kernel corrigent souvent des vulnérabilités critiques qui, si elles sont exploitées, pourraient permettre à un attaquant de prendre le contrôle du système.

- **Paramètres de sécurité du kernel :** Utiliser des fonctionnalités comme SELinux, AppArmor, ou seccomp pour limiter les actions que les processus peuvent effectuer, réduisant ainsi la surface d'attaque. Configurer correctement ces outils peut prévenir de nombreux types d'exploits.

- **Désactivation des modules du kernel non nécessaires :** Limiter les modules du kernel chargés au démarrage à ceux strictement nécessaires pour le fonctionnement du système. Cela réduit la surface d'attaque potentielle. La liste des modules chargés peut être consultée avec `lsmod`, et leur chargement peut être désactivé en modifiant `/etc/modprobe.d/`.

- **Kernel Hardening :** Des patches de hardening, comme ceux fournis par le projet Grsecurity, offrent des mesures de sécurité supplémentaires pour le kernel. Bien que Grsecurity ne soit plus disponible publiquement pour les dernières versions du kernel, explorer des alternatives ou des pratiques similaires peut renforcer la sécurité.

- **Utilisation de systèmes de détection d'intrusion :** Des outils comme AIDE (Advanced Intrusion Detection Environment) ou Samhain peuvent être utilisés pour surveiller l'intégrité du système de fichiers, y compris les fichiers du kernel, et alerter les administrateurs en cas de modifications suspectes.
## 2.4. Configuration et sécurisation des services système
### 2.4. Configuration et Sécurisation des Services Système

Les services système, qui fournissent diverses fonctionnalités et permettent au système d'exploitation de communiquer avec le réseau, les applications et les utilisateurs, sont essentiels pour le fonctionnement d'un système Linux. Cependant, chaque service actif augmente la surface d'attaque potentielle, rendant cruciale la configuration et la sécurisation appropriées de ces services pour maintenir la sécurité globale du système.

#### **Identification et Gestion des Services**

- **Inventaire des services actifs :** Commencez par identifier tous les services en cours d'exécution sur le système avec des commandes comme `systemctl list-units --type=service --state=running` sur les systèmes utilisant `systemd`, ou `service --status-all` sur d'autres systèmes. Cet inventaire permet d'analyser quels services sont nécessaires et lesquels peuvent être désactivés.

- **Désactivation des services inutiles :** Pour chaque service non nécessaire identifié, le désactiver pour empêcher son démarrage automatique. Cela peut être accompli avec `systemctl disable service_name` sur les systèmes `systemd`. Réduire le nombre de services en cours d'exécution minimise les vecteurs d'attaque disponibles.

#### **Sécurisation des Services Nécessaires**

- **Configuration selon le principe de moindre privilège :** Configurer les services pour qu'ils s'exécutent avec le minimum de privilèges nécessaires. Par exemple, éviter d'exécuter des services en tant que root lorsque cela n'est pas nécessaire et envisager l'utilisation de `Capabilities` pour accorder uniquement les droits spécifiques requis par le service.

- **Mise à jour et Patching :** Assurer que tous les services et applications en cours d'exécution sont à jour avec les derniers patches de sécurité. Les vulnérabilités dans les logiciels tiers peuvent être exploitées par des attaquants pour compromettre le système.

- **Sécurisation des configurations :** Suivre les guides de sécurisation et les benchmarks pour chaque service, tels que ceux fournis par le Center for Internet Security (CIS). Cela inclut la modification des configurations par défaut, le durcissement des protocoles de communication et la limitation des accès réseau au strict nécessaire.

- **Utilisation de pare-feu et de règles de filtrage :** Configurer le pare-feu pour restreindre l'accès aux services, en autorisant uniquement le trafic réseau nécessaire et en bloquant les autres. Les outils comme `iptables`, `nftables` ou `firewalld` peuvent être utilisés pour gérer ces règles.

- **Surveillance et logs :** Activer la journalisation pour tous les services critiques et configurer des systèmes de surveillance pour alerter en cas d'activité suspecte. L'analyse régulière des logs peut aider à détecter des tentatives d'intrusion ou des configurations incorrectes.

#### **Isolation et Contrôle d'Accès**

- **Isolation des services :** Utiliser des technologies d'isolation comme les conteneurs Docker ou les machines virtuelles pour exécuter des services dans des environnements isolés. Cela limite les dommages potentiels en cas de compromission d'un service.

- **Contrôle d'accès réseau :** Restreindre l'accès aux services en fonction de l'adresse IP, du domaine ou d'autres critères d'authentification pour réduire le risque d'accès non autorisé.

- **Authentification et chiffrement :** Pour les services nécessitant une authentification, utiliser des mécanismes robustes et envisager le chiffrement des communications, particulièrement pour les informations sensibles transitant sur des réseaux non sécurisés.

La configuration et la sécurisation des services système sont des tâches continues, nécessitant une vigilance constante et des ajustements réguliers pour répondre aux nouvelles menaces et aux meilleures pratiques de sécurité. En appliquant ces principes, les administrateurs système peuvent réduire significativement les risques de sécurité associés aux services système sur les serveurs Linux.
## 2.5. Audit et monitoring du système avec syslog et journalctl

L'audit et le monitoring sont des composantes essentielles de la sécurité d'un système Linux, permettant de détecter, enregistrer et analyser les activités système pour identifier des comportements anormaux ou malveillants. `syslog` et `journalctl` (partie de `systemd`) sont deux outils puissants utilisés pour la gestion des logs sous Linux, chacun offrant des fonctionnalités uniques pour l'audit et le monitoring du système.

### 2.5.1. Syslog : Le Système de Logging Traditionnel

- **Présentation :** `syslog` est un standard pour le message logging, offrant une architecture centralisée pour la collecte et le stockage des messages du système et des applications. Il permet de router les messages à différents fichiers de log, à des consoles, ou même à des serveurs de log distants, basés sur leur priorité et leur source.

- **Configuration :** Le fichier de configuration principal de `syslog` (`/etc/syslog.conf` ou `/etc/rsyslog.conf` pour rsyslog, une implémentation de `syslog`) permet de définir des règles pour la gestion des logs. Les administrateurs peuvent configurer la destination des logs en fonction de leur facilité (comme `auth`, `cron`, `daemon`, etc.) et de leur niveau de priorité (comme `info`, `warning`, `error`).

- **Sécurisation et Rotation des Logs :** Il est crucial de sécuriser l'accès aux fichiers de log pour éviter les modifications ou la suppression non autorisées. La rotation des logs, gérée par des outils comme `logrotate`, aide à contrôler la taille des fichiers de log, en les archivant et en les compressant périodiquement.

### 2.5.2. journalctl : Le Journal de systemd

- **Présentation :** Avec l'adoption de `systemd` comme système d'init pour de nombreuses distributions Linux, `journalctl` est devenu un outil central pour accéder aux logs du système. Le journal de `systemd` collecte non seulement les messages de syslog, mais aussi les logs du kernel, des démons d'init, et des applications, offrant ainsi une vue intégrée de tous les événements système.

- **Avantages :** `journalctl` offre des fonctionnalités avancées comme l'affichage des logs depuis un certain point dans le temps, le filtrage par priorité, unité `systemd`, ou même par processus. Les logs sont stockés dans un format binaire et indexé, facilitant des recherches rapides et efficaces.

- **Usage :** Pour visualiser les logs, `journalctl` peut être utilisé sans paramètres pour afficher tous les logs système, ou avec des options comme `-u nom_service.service` pour les logs d'un service spécifique, `--since` et `--until` pour une période donnée, ou `-p err` pour filtrer par priorité.

### 2.5.3 Best Practices pour l'Audit et le Monitoring

- **Analyse Régulière :** Examiner régulièrement les logs pour détecter des activités suspectes ou non autorisées. Des outils d'analyse de log automatisés peuvent aider à identifier les tendances et les anomalies.

- **Sécurité des Logs :** Assurer l'intégrité et la confidentialité des logs en les stockant dans des zones sécurisées, en utilisant le chiffrement si nécessaire, surtout pour les logs contenant des données sensibles.

- **Alertes en Temps Réel :** Configurer des alertes basées sur des événements critiques ou des indicateurs d'attaque pour une réponse rapide aux incidents de sécurité.

- **Intégration avec des Outils de SIEM :** Pour une vue plus complète et une analyse approfondie, intégrer les logs système avec des solutions de Security Information and Event Management (SIEM) qui peuvent corréler les données de log à travers différents systèmes et applications.

En utilisant `syslog` et `journalctl` efficacement pour l'audit et le monitoring, les administrateurs peuvent améliorer la posture de sécurité de leurs systèmes Linux, en identifiant et en répondant rapidement aux incidents de sécurité, tout en maintenant une trace précieuse des événements système pour l'analyse future.

---
> [!info] Livre « Sécurité avancée sous Linux » — chapitre 2/7
> [[Sécurité avancée sous Linux — Sommaire|Sommaire]] · [[Sécurité avancée sous Linux — 01 — Compréhension des bases de la sécurité Linux|← 01 — Compréhension des bases de la sécurité Linux]] · [[Sécurité avancée sous Linux — 03 — Contrôle d'accès avancé|03 — Contrôle d'accès avancé →]]
