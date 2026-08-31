---
schema_version: 1
uid: 01M1BQ62C6CGWR2A8YP0SN8NXX
titre: "Sécurité avancée sous Linux — 01 — Compréhension des bases de la sécurité Linux"
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
resume: "Chapitre 1 sur 7 du livre « Sécurité avancée sous Linux » : Compréhension des bases de la sécurité Linux. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-15."
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

> [!info] Livre « Sécurité avancée sous Linux » — chapitre 1/7
> [[Sécurité avancée sous Linux — Sommaire|Sommaire]] · [[Sécurité avancée sous Linux — Sommaire|← Sommaire]] · [[Sécurité avancée sous Linux — 02 — Sécurisation de l'environnement système|02 — Sécurisation de l'environnement système →]]

# 1. Compréhension des bases de la sécurité Linux
## 1.1. Structure et Architecture de Sécurité Linux

La structure et l'architecture de sécurité de Linux sont conçues pour offrir une protection robuste à tous les niveaux du système. Cette conception repose sur une série de mécanismes et de stratégies intégrés qui travaillent ensemble pour prévenir, détecter et répondre aux menaces et vulnérabilités. Voici les composantes clés de la structure et de l'architecture de sécurité sous Linux :

### 1.1.1 Noyau Linux et Modules de Sécurité

Le cœur de la sécurité sous Linux réside dans son noyau, qui gère l'accès aux ressources matérielles et logicielles et isole les processus les uns des autres pour prévenir les interférences malveillantes. Le noyau Linux inclut des modules de sécurité tels que SELinux (Security-Enhanced Linux), AppArmor, et le sous-système de sécurité Tomoyo, qui fournissent un contrôle d'accès obligatoire (MAC) au-delà du modèle traditionnel de contrôle d'accès discrétionnaire (DAC) basé sur les permissions.

### 1.1.2. Gestion des Utilisateurs et des Permissions

Linux adopte un modèle d'utilisateur multi-utilisateurs, où chaque utilisateur a un identifiant unique (UID) et appartient à un ou plusieurs groupes (GID). Cette structure permet une séparation claire des privilèges et des accès, où les permissions sont définies pour lire, écrire, et exécuter des fichiers et des répertoires. Cela limite l'accès aux ressources critiques et aux fonctionnalités du système aux seuls utilisateurs et groupes autorisés.

### 1.1.3. Système de Fichiers et Sécurité

Le système de fichiers sous Linux est structuré de manière hiérarchique, avec des points de montage et des partitions pouvant être sécurisés individuellement. Des fonctionnalités telles que les attributs étendus (Extended Attributes) permettent d'appliquer des politiques de sécurité spécifiques, comme les listes de contrôle d'accès (ACL) pour un contrôle d'accès plus granulaire que les permissions standards.

### 1.1.4. PAM (Pluggable Authentication Modules)

Linux utilise PAM, un mécanisme flexible pour l'authentification des utilisateurs, qui offre une méthode de configuration centralisée pour l'authentification des applications. PAM permet de définir des politiques de sécurité pour l'authentification, la gestion des comptes, les sessions et les mots de passe, supportant ainsi une variété de schémas d'authentification, y compris l'authentification à deux facteurs et l'intégration avec des systèmes d'authentification externes.

### 1.1.5. Sécurité Réseau

Linux intègre une pile réseau sophistiquée qui supporte des règles de filtrage de paquets, des pare-feux, et des fonctionnalités de réseau virtuel privé (VPN). Des outils tels qu'iptables, nftables, et Firewalld permettent aux administrateurs de configurer des politiques de sécurité réseau pour contrôler le trafic entrant et sortant, protégeant ainsi le système contre les accès non autorisés et les attaques de réseau.

### 1.1.6. Mécanismes de Cryptographie

Le noyau Linux et l'espace utilisateur intègrent des bibliothèques et des outils de cryptographie pour sécuriser les données en transit et au repos. Des fonctionnalités comme dm-crypt fournissent le chiffrement des volumes, tandis que des outils comme OpenSSL et GnuPG permettent le chiffrement des communications réseau et des fichiers.

En résumé, la structure et l'architecture de sécurité de Linux sont fondées sur une approche multicouche et modulaire, permettant une personnalisation et une adaptation à divers environnements et exigences de sécurité. Cette flexibilité, combinée à une communauté active et à une évolution constante, fait de Linux un système d'exploitation puissant et sécurisé pour les applications critiques.

## 1.2. Principes de Moindre Privilège et Séparation des Devoirs

Les principes de moindre privilège et de séparation des devoirs sont fondamentaux dans la sécurisation des systèmes Linux. Ces principes visent à minimiser les risques de sécurité en limitant l'accès aux ressources et les capacités d'action dans le système au strict nécessaire pour chaque utilisateur ou processus. En appliquant ces principes, on réduit l'impact potentiel des failles de sécurité et on augmente la résilience du système face aux attaques.

### 1.2.1 Moindre Privilège

Le principe de moindre privilège stipule qu'un utilisateur, une application ou un service doit avoir uniquement les droits nécessaires pour effectuer ses tâches, rien de plus. Cela signifie que chaque processus s'exécute avec l'ensemble minimal de privilèges nécessaires pour sa fonction, réduisant ainsi la surface d'attaque disponible pour les acteurs malveillants.

**Application pratique sous Linux :**

- **Utilisateurs non privilégiés pour les tâches courantes :** Les utilisateurs doivent opérer avec des comptes non privilégiés pour l'exécution des tâches quotidiennes et ne recourir à des comptes administratifs que pour les opérations nécessitant des privilèges élevés.
- **Droits minimal sur les fichiers et services :** Attribuer des permissions strictes sur les fichiers, les répertoires et les services pour garantir que seul le personnel autorisé puisse y accéder.
- **Utilisation de sudo :** Sudo permet aux utilisateurs autorisés d'exécuter certaines commandes en tant qu'autre utilisateur (généralement le superutilisateur), selon des règles précises définies dans le fichier `/etc/sudoers`, ce qui aide à limiter l'usage des privilèges élevés.

### 1.2.2 Séparation des Devoirs

La séparation des devoirs est un concept complémentaire qui consiste à diviser les responsabilités parmi plusieurs entités (personnes, groupes, services, etc.) pour éviter qu'une seule entité ne possède un contrôle total sur un système ou un processus. Ce principe aide à prévenir les abus de pouvoir et les fraudes, tout en facilitant la détection des erreurs ou des activités malveillantes.

**Application pratique sous Linux :**

- **Rôles et groupes distincts :** Configurer des groupes avec des droits spécifiques et y affecter les utilisateurs selon leur rôle dans l'organisation, assurant ainsi que les tâches administratives, de développement et d'exploitation soient clairement séparées.
- **Services avec des utilisateurs dédiés :** Faire tourner les services avec des comptes utilisateurs dédiés qui ont uniquement les privilèges nécessaires pour exécuter le service en question. Cela limite les dommages potentiels en cas de compromission du service.
- **Audit et logs :** Mettre en place des procédures d'audit et de journalisation qui enregistrent les actions des utilisateurs et des services. Cela facilite le suivi des activités et la détection d'éventuels comportements inappropriés ou malveillants.

En intégrant les principes de moindre privilège et de séparation des devoirs dans la gestion de la sécurité des systèmes Linux, les organisations peuvent construire des environnements informatiques plus sûrs et résilients, où les risques de compromission et leurs impacts potentiels sont significativement réduits.

## 1.3. Gestion des Utilisateurs et des Groupes

La gestion des utilisateurs et des groupes est une composante fondamentale de la sécurité sous Linux, permettant de contrôler l'accès aux ressources et de limiter les actions possibles sur le système en fonction des rôles et des responsabilités de chaque utilisateur. Cette section explore comment Linux gère ces concepts et propose des pratiques recommandées pour maintenir un système sécurisé.

### 1.3.1 Utilisateurs sous Linux

Chaque utilisateur sous Linux est identifié par un nom d'utilisateur unique et un identifiant numérique (UID). Le système utilise ces identifiants pour appliquer les politiques de sécurité, telles que les permissions de fichiers et d'exécution de programmes. Il existe deux types principaux d'utilisateurs :

- **L'utilisateur root :** L'utilisateur administrateur qui possède tous les privilèges sur le système. Il a le pouvoir de modifier n'importe quelle configuration et d'accéder à toutes les commandes et fichiers.
- **Utilisateurs non privilégiés :** Les comptes utilisés pour des tâches spécifiques sans nécessiter un accès complet au système. Ces comptes sont limités dans leurs capacités pour augmenter la sécurité.

### 1.3.2 Groupes sous Linux

Les groupes sont des collections d'utilisateurs partageant des droits d'accès communs à certains fichiers et répertoires. Chaque utilisateur appartient à un groupe primaire et peut appartenir à plusieurs groupes secondaires. Les groupes sont utilisés pour simplifier la gestion des permissions en attribuant des droits à plusieurs utilisateurs simultanément.

### 1.3.3 Pratiques Recommandées

1. **Utilisation minimale du compte root :** Éviter l'utilisation du compte root pour des tâches quotidiennes. Utiliser `sudo` pour exécuter des commandes nécessitant des privilèges élevés.
   
2. **Création de comptes spécifiques pour les tâches :** Pour chaque service ou tâche nécessitant des privilèges, créer un compte utilisateur spécifique qui possède uniquement les droits nécessaires à cette tâche. Cela limite les risques en cas de compromission du compte.

3. **Gestion stricte des membres des groupes :** S'assurer que les utilisateurs sont uniquement membres des groupes nécessaires à leurs tâches. Limiter l'appartenance aux groupes sensibles comme `wheel` ou `sudo`, qui confèrent des droits d'accès élevés.

4. **Sécurisation des mots de passe :** Utiliser des mots de passe forts et uniques pour tous les comptes et envisager l'utilisation d'un gestionnaire de mots de passe. Configurer des politiques de mot de passe strictes, telles que des expirations régulières et des critères de complexité.

5. **Utilisation de l'authentification à deux facteurs (2FA) :** Renforcer la sécurité en exigeant une seconde forme d'authentification en plus du mot de passe pour accéder aux comptes critiques.

6. **Audit et révision réguliers des comptes :** Examiner régulièrement les comptes utilisateurs et les appartenance aux groupes pour s'assurer qu'ils correspondent toujours aux besoins actuels et aux politiques de sécurité.

7. **Configuration de l'expiration des comptes :** Définir des dates d'expiration pour les comptes temporaires ou ceux associés à des employés partant, pour s'assurer qu'ils ne restent pas actifs indéfiniment.

## 1.4. Permissions et Droits d'Accès aux Fichiers

La gestion des permissions et des droits d'accès aux fichiers est une pierre angulaire de la sécurité sous Linux. Ce système permet de définir qui peut lire, écrire ou exécuter des fichiers et des répertoires, offrant ainsi un contrôle granulaire sur l'accès aux données et aux ressources du système. Comprendre et appliquer correctement ces permissions est crucial pour maintenir l'intégrité, la confidentialité et la disponibilité des systèmes Linux.

### 1.4.1 Types de Permissions

Sous Linux, chaque fichier et répertoire est associé à un ensemble de permissions divisées en trois catégories :

- **Lire (r) :** Permet de visualiser le contenu d'un fichier ou de lister les fichiers et sous-répertoires d'un répertoire.
- **Écrire (w) :** Autorise la modification ou la suppression d'un fichier. Pour un répertoire, cette permission permet de créer, modifier ou supprimer des fichiers et des sous-répertoires.
- **Exécuter (x) :** Permet d'exécuter un fichier comme un programme. Pour un répertoire, elle permet d'accéder à son contenu et de le traverser (nécessaire pour accéder à des répertoires situés plus loin dans l'arborescence).

### 1.4.2 Gestion des Permissions

Les permissions sont définies pour trois types d'utilisateurs :

- **Propriétaire :** L'utilisateur qui possède le fichier ou le répertoire.
- **Groupe :** Le groupe auquel appartient le fichier. Tous les utilisateurs membres de ce groupe héritent des permissions accordées au groupe.
- **Autres :** Tous les autres utilisateurs du système.

Les permissions sont souvent affichées sous forme d'une chaîne de caractères, telle que `-rwxr-xr--`, ou en notation octale, comme `755`. Chaque caractère ou chiffre représente une permission différente pour le propriétaire, le groupe et les autres, respectivement.

### 1.4.3. Modification des Permissions

La commande `chmod` (change mode) est utilisée pour modifier les permissions d'un fichier ou d'un répertoire. Elle peut être utilisée avec la notation symbolique (r, w, x) ou la notation octale. Par exemple, `chmod 755 fichier` accorde au propriétaire toutes les permissions, au groupe et aux autres uniquement les permissions de lire et d'exécuter.

### 1.4.4. Changement de Propriétaire et de Groupe

Les commandes `chown` et `chgrp` permettent de changer respectivement le propriétaire et le groupe d'un fichier ou d'un répertoire. Cela est souvent nécessaire lors du déploiement d'applications ou de la restructuration des accès utilisateurs.

### 1.4.5. Pratiques Recommandées

- **Principe de moindre privilège :** Attribuer les permissions les plus restrictives possibles qui permettent encore d'accomplir les tâches nécessaires.
- **Vérification régulière des permissions :** Examiner régulièrement les permissions des fichiers et répertoires sensibles pour s'assurer qu'ils ne sont pas trop permissifs.
- **Utilisation de groupes pour gérer les permissions :** Utiliser des groupes pour gérer efficacement les permissions d'accès pour plusieurs utilisateurs, réduisant ainsi la nécessité de modifier les permissions sur une base individuelle.
- **Sécurité des répertoires sensibles :** Assurer une protection particulière aux répertoires contenant des informations sensibles, en limitant strictement les permissions d'accès.

## 1.5. Utilisation de sudo pour la Gestion des Privilèges

L'outil `sudo` (substitute user and do) est essentiel dans la gestion des privilèges sur les systèmes Linux, permettant aux utilisateurs autorisés d'exécuter des commandes avec les privilèges d'un autre utilisateur, typiquement le superutilisateur (root), selon des règles définies dans le fichier de configuration `/etc/sudoers`. Cet outil est central pour appliquer le principe de moindre privilège, en minimisant l'utilisation des comptes à haut privilège tout en permettant l'accomplissement de tâches nécessitant de tels privilèges.

### 1.5.1. Avantages de sudo

- **Sécurité accrue :** Limite l'accès root en ne l'octroyant que pour des commandes spécifiques, réduisant ainsi les risques d'erreurs ou d'exploitations malveillantes.
- **Traçabilité :** Chaque commande exécutée avec `sudo` est enregistrée, permettant un audit précis des actions effectuées avec des privilèges élevés.
- **Flexibilité :** Permet une configuration granulaire des privilèges, autorisant différents utilisateurs à exécuter certaines commandes en tant que root ou en tant qu'un autre utilisateur sans partager le mot de passe root.

### 1.5.2 Configuration de sudo

La configuration de `sudo` se fait dans le fichier `/etc/sudoers`, qui devrait toujours être édité avec la commande `visudo` pour éviter les erreurs de syntaxe. Ce fichier définit qui peut exécuter quoi, sur quelles machines et en tant que quel utilisateur. Il est possible de spécifier des groupes d'utilisateurs (précédés d'un `%`) et des alias pour simplifier la gestion des permissions.

### 1.5.3. Principales Directives dans sudoers

- **User specifications :** Détermine qui peut exécuter quoi. La syntaxe générale est `user host = (run-as) command`.
- **Runas specification :** Spécifie sous quelle identité l'utilisateur peut exécuter les commandes.
- **Cmnd alias :** Permet de créer des alias pour un ensemble de commandes, simplifiant la gestion des permissions pour des commandes similaires ou complexes.

### 1.5.4. Utilisation Responsable de sudo

- **Ne pas exécuter d'interface graphique avec sudo :** Lancer une interface graphique complète avec `sudo` peut créer des risques de sécurité en exposant le système à des modifications involontaires ou malveillantes.
- **Utiliser sudo pour des commandes précises :** Limiter l'utilisation de `sudo` à des commandes nécessaires, réduisant ainsi l'exposition du système à des erreurs potentielles ou à des abus.
- **Configurer des timeouts :** `sudo` peut être configuré pour oublier les privilèges après une période définie, forçant l'utilisateur à réauthentifier pour continuer à utiliser des privilèges élevés.

### 1.5.5. Best Practices

- **Utiliser `sudo` au lieu de se connecter en tant que root :** Cela garantit que les commandes privilégiées sont enregistrées et auditées.
- **Minimiser les permissions dans le fichier sudoers :** Accorder uniquement les permissions nécessaires pour les tâches requises, suivant le principe de moindre privilège.
- **Former les utilisateurs :** Assurer que les utilisateurs sachent comment utiliser `sudo` de manière responsable et comprendre les risques associés à l'exécution de commandes avec des privilèges élevés.

---
> [!info] Livre « Sécurité avancée sous Linux » — chapitre 1/7
> [[Sécurité avancée sous Linux — Sommaire|Sommaire]] · [[Sécurité avancée sous Linux — Sommaire|← Sommaire]] · [[Sécurité avancée sous Linux — 02 — Sécurisation de l'environnement système|02 — Sécurisation de l'environnement système →]]
