---
schema_version: 1
uid: 01M1BQ62C7HAE7CR8NX0N29D28
titre: "Sécurité avancée sous Linux — 05 — Cryptographie et sécurisation des données"
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
resume: "Chapitre 5 sur 7 du livre « Sécurité avancée sous Linux » : Cryptographie et sécurisation des données. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-15."
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

> [!info] Livre « Sécurité avancée sous Linux » — chapitre 5/7
> [[Sécurité avancée sous Linux — Sommaire|Sommaire]] · [[Sécurité avancée sous Linux — 04 — Authentification et accès réseau|← 04 — Authentification et accès réseau]] · [[Sécurité avancée sous Linux — 06 — Détection d'intrusions et réponse aux incidents|06 — Détection d'intrusions et réponse aux incidents →]]

# 5. Cryptographie et sécurisation des données
## 5.1. Principes de base de la cryptographie appliquée
### 5.1. Principes de Base de la Cryptographie Appliquée

La cryptographie est l'art et la science de sécuriser les communications pour les rendre inintelligibles aux observateurs non autorisés. Elle joue un rôle essentiel dans la protection de l'information dans le monde numérique d'aujourd'hui. Voici une introduction aux principes fondamentaux de la cryptographie appliquée, qui forment la base de la sécurisation des données et des communications.

#### **Chiffrement Symétrique et Asymétrique**

- **Chiffrement Symétrique :** Utilise la même clé pour chiffrer et déchiffrer les données. Bien qu'il soit rapide et efficace pour de grandes quantités de données, le défi majeur réside dans la distribution sécurisée de la clé. DES, AES et ChaCha20 sont des exemples d'algorithmes de chiffrement symétrique.

- **Chiffrement Asymétrique :** Utilise une paire de clés, une publique pour le chiffrement et une privée pour le déchiffrement. Cela résout le problème de la distribution de clés mais est plus lent que le chiffrement symétrique. RSA, ECC (Elliptic Curve Cryptography) et Diffie-Hellman sont des exemples de systèmes de chiffrement asymétrique.

#### **Hachage Cryptographique**

Les fonctions de hachage transforment les données d'entrée de n'importe quelle taille en une sortie de taille fixe (un hachage) de manière irréversible. Les hachages sont utilisés pour vérifier l'intégrité des données, les signatures numériques, et comme partie des processus d'authentification. SHA-256 et SHA-3 sont des exemples de fonctions de hachage cryptographique.

#### **Signatures Numériques**

Une signature numérique utilise le chiffrement asymétrique pour prouver l'origine et l'intégrité des données. Elle permet au destinataire de vérifier que les données n'ont pas été altérées et confirme l'identité de l'expéditeur.

#### **Gestion des Clés**

La gestion efficace des clés est cruciale pour la sécurité de la cryptographie. Cela comprend la création, la distribution, le stockage, l'usage et la destruction des clés. Une mauvaise gestion des clés peut compromettre la sécurité de tout le système cryptographique.

#### **Protocoles de Sécurité**

Les protocoles de sécurité, tels que TLS/SSL pour les communications web sécurisées, SSH pour l'accès sécurisé aux serveurs, et IPsec pour les VPN sécurisés, utilisent la cryptographie pour protéger les données en transit. La configuration correcte de ces protocoles est essentielle pour leur efficacité.

#### **Principes de Sécurité**

- **Confidentialité :** Assurer que seuls les destinataires autorisés peuvent accéder aux informations.
- **Intégrité :** Garantir que les données ne sont pas altérées, intentionnellement ou accidentellement, pendant la transmission ou le stockage.
- **Authentification :** Vérifier l'identité des parties dans une communication.
- **Non-répudiation :** Empêcher qu'une partie nie avoir envoyé ou reçu des données.

## 5.2. Chiffrement des disques et des fichiers avec LUKS et eCryptfs

Le chiffrement des disques et des fichiers est une méthode essentielle pour protéger les données contre l'accès non autorisé. Linux offre plusieurs outils pour le chiffrement, parmi lesquels LUKS (Linux Unified Key Setup) et eCryptfs sont particulièrement populaires pour leur efficacité et leur facilité d'utilisation.

### 5.2.1. LUKS (Linux Unified Key Setup)

LUKS est une norme de chiffrement pour Linux qui sécurise les disques ou partitions en utilisant des algorithmes de chiffrement robustes. LUKS est couramment utilisé pour le chiffrement complet des disques, offrant une couche de protection solide pour les données au repos.

- **Configuration de LUKS :**
  1. **Préparation du Disque :** Identifiez le disque ou la partition à chiffrer (par exemple, `/dev/sdaX`).
  2. **Création d'une Partition Chiffrée :** Utilisez `cryptsetup luksFormat /dev/sdaX` pour initialiser le chiffrement LUKS sur la partition. Vous serez invité à entrer une passphrase qui sera utilisée pour déverrouiller le disque.
  3. **Ouverture de la Partition Chiffrée :** Après le formatage, ouvrez la partition chiffrée avec `cryptsetup open /dev/sdaX nom_chiffré`. Cela crée un périphérique mappé dans `/dev/mapper/` que vous pouvez formater et monter comme n'importe quel autre système de fichiers.
  4. **Montage et Utilisation :** Formatez le périphérique mappé avec votre système de fichiers préféré, puis montez-le pour stocker des données sécurisées.

- **Avantages :** LUKS offre un chiffrement fort et flexible, supporte plusieurs clés de déchiffrement et permet de changer facilement les passphrases sans rechiffrer l'intégralité du disque.

### 5.2.2. eCryptfs

eCryptfs est un système de fichiers chiffré qui fonctionne au niveau des fichiers plutôt que des disques entiers. Cela permet le chiffrement transparent de dossiers spécifiques, avec des fichiers stockés sous forme chiffrée mais automatiquement déchiffrés lors de l'accès par un utilisateur autorisé.

- **Configuration d'eCryptfs :**
  1. **Installation :** Assurez-vous que le paquet eCryptfs est installé sur votre système.
  2. **Chiffrement d'un Répertoire :** Utilisez `ecryptfs-setup-private` pour créer un répertoire sécurisé dans votre répertoire personnel. Vous serez invité à entrer une passphrase qui chiffrera votre répertoire privé.
  3. **Accès aux Fichiers :** Les fichiers placés dans le répertoire chiffré sont automatiquement chiffrés. Lorsque vous vous connectez, le répertoire est monté automatiquement et les fichiers sont déchiffrés à la volée lors de l'accès.

- **Avantages :** eCryptfs est particulièrement utile pour le chiffrement sélectif de dossiers et de fichiers, permettant une flexibilité accrue et la possibilité de partager des fichiers chiffrés entre systèmes.

### 5.2.3. Bonnes Pratiques

- **Sauvegarde des En-têtes LUKS :** Après avoir configuré LUKS, sauvegardez l'en-tête du disque chiffré avec `cryptsetup luksHeaderBackup`. Cela peut être crucial pour la récupération des données en cas de corruption de l'en-tête.
- **Gestion des Passphrases :** Utilisez des passphrases fortes et envisagez d'utiliser un gestionnaire de mots de passe pour les stocker en toute sécurité.
- **Sauvegardes Régulières :** Maintenez des sauvegardes régulières de vos données chiffrées pour prévenir la perte de données en cas de défaillance du matériel ou de corruption des données.

Le choix entre LUKS et eCryptfs dépend des besoins spécifiques en matière de sécurité et de la commodité. LUKS est idéal pour le chiffrement complet des disques, tandis qu'eCryptfs convient mieux au chiffrement de dossiers et de fichiers spécifiques, offrant une solution flexible.
## 5.3. Gestion sécurisée des clés de chiffrement

La gestion sécurisée des clés de chiffrement est un pilier fondamental de la protection des données. Une gestion efficace assure que les clés de chiffrement sont à la fois accessibles aux utilisateurs autorisés et protégées contre l'accès non autorisé. Voici les meilleures pratiques et stratégies pour gérer de manière sécurisée les clés de chiffrement dans un environnement Linux.

### 5.3.1. Principes Fondamentaux

- **Séparation des Clés :** Gardez une séparation claire entre les clés de chiffrement et les données chiffrées. Les clés ne doivent jamais être stockées à côté des données qu'elles protègent ou dans le même système ou réseau où les données sont accessibles.

- **Chiffrement des Clés :** Les clés de chiffrement elles-mêmes doivent être chiffrées lorsqu'elles sont stockées ou transmises. Utilisez des clés maîtresses robustes pour chiffrer les clés de chiffrement secondaires.

### 5.3.2. Gestion du Cycle de Vie des Clés

- **Génération de Clés :** Générez des clés en utilisant des sources d'entropie fortes pour assurer leur robustesse. Les outils de gestion de clés ou les modules de sécurité matérielle (HSM) peuvent fournir des fonctionnalités de génération de clés sécurisées.

- **Distribution de Clés :** La distribution des clés aux utilisateurs et systèmes autorisés doit se faire de manière sécurisée, en évitant les canaux de communication non chiffrés. 

- **Stockage de Clés :** Stockez les clés de manière sécurisée, en utilisant des coffres-forts de clés ou des HSMs qui offrent à la fois la sécurité physique et logique contre les accès non autorisés.

- **Rotation de Clés :** Implémentez une politique de rotation des clés pour remplacer régulièrement les clés par de nouvelles. Cela aide à limiter la durée pendant laquelle une clé compromise peut être utilisée pour accéder à des données chiffrées.

- **Révocation et Expiration de Clés :** Mettez en place des mécanismes pour révoquer rapidement les clés compromises et définissez une date d'expiration pour les clés pour forcer leur renouvellement.

### 5.3.3. Accès et Autorisations

- **Contrôle d'Accès :** Assurez-vous que seuls les utilisateurs et les processus autorisés ont accès aux clés de chiffrement. Utilisez des listes de contrôle d'accès (ACLs) et des politiques basées sur les rôles pour gérer les autorisations.

- **Audit et Suivi :** Tenez un journal des accès aux clés de chiffrement et des opérations effectuées avec, permettant de détecter et d'enquêter sur les accès non autorisés ou les utilisations anormales.

### 5.3.4. Sauvegardes et Récupération

- **Sauvegardes de Clés :** Gardez des sauvegardes sécurisées des clés de chiffrement pour assurer la récupérabilité en cas de perte. Les sauvegardes doivent être stockées dans un emplacement sécurisé, séparé des données qu'elles protègent.

- **Plan de Récupération :** Élaborez un plan de récupération de clés pour restaurer l'accès aux données chiffrées en cas de perte ou de compromission des clés.

La gestion sécurisée des clés de chiffrement requiert une attention constante et une mise en œuvre rigoureuse des politiques et des procédures. En suivant ces meilleures pratiques, les organisations peuvent assurer la sécurité et l'intégrité des clés de chiffrement, un élément essentiel pour la protection globale des données sensibles.
## 5.4. VPN et sécurisation des communications

L'utilisation d'un Réseau Privé Virtuel (VPN) est une méthode efficace pour sécuriser les communications sur des réseaux non fiables, comme Internet. Le VPN crée un tunnel crypté entre votre appareil et un serveur VPN distant, garantissant que les données transmises restent confidentielles et protégées contre les écoutes indiscrètes. Voici les principes et meilleures pratiques pour l'utilisation des VPN et la sécurisation des communications.

### 5.4.1. Choix du Protocole VPN

- **Protocoles Sécurisés :** Optez pour des protocoles VPN reconnus pour leur sécurité, tels que OpenVPN, IKEv2/IPSec, ou WireGuard. Évitez les protocoles obsolètes et moins sécurisés comme PPTP et L2TP/IPSec sans chiffrement fort.

- **Configuration Forte :** Assurez-vous que la configuration du VPN utilise des algorithmes de chiffrement robustes, des clés de chiffrement de longueur suffisante, et des mécanismes d'authentification sécurisés pour éviter les vulnérabilités.

### 5.4.2. Utilisation de VPN d'Entreprise

- **Serveurs VPN Propres :** Pour les organisations, l'implémentation de serveurs VPN internes contrôlés est recommandée. Cela permet une meilleure gestion des politiques d'accès et une assurance supplémentaire quant à la confidentialité des données.

- **Authentification et Autorisation :** Intégrez le système VPN avec des solutions d'authentification existantes (comme Active Directory ou LDAP) pour contrôler l'accès basé sur les identités utilisateurs et les politiques de sécurité de l'entreprise.

### 5.4.3. Sécurisation du Client VPN

- **Mise à Jour des Clients VPN :** Gardez les logiciels clients VPN à jour avec les dernières versions pour bénéficier des correctifs de sécurité et des améliorations de performances.

- **Politiques de Sécurité Locales :** Configurez les postes clients pour utiliser le VPN pour tout le trafic réseau ou établissez des règles de routage spécifiques pour diriger uniquement le trafic sensible à travers le VPN.

### 5.4.4. Sécurité Wi-Fi Publique

- **VPN sur Wi-Fi Public :** Utilisez systématiquement un VPN lors de la connexion à des réseaux Wi-Fi publics pour protéger vos données contre les risques d'interception sur ces réseaux non sécurisés.

### 5.4.5. VPN et Confidentialité

- **Choix du Fournisseur VPN :** Si vous utilisez un service VPN externe, sélectionnez un fournisseur réputé qui respecte une politique stricte de non-conservation des logs et qui est situé dans une juridiction protégeant la vie privée des utilisateurs.

- **Sensibilisation des Utilisateurs :** Éduquez les utilisateurs sur l'importance de l'utilisation du VPN, en particulier lors de l'accès à des informations sensibles ou de l'utilisation de réseaux non sécurisés.

### 5.4.6. Audit et Surveillance

- **Surveillance des Connexions VPN :** Surveillez les connexions VPN pour détecter les activités anormales qui pourraient indiquer des tentatives d'intrusion ou des abus du système VPN.

- **Tests de Pénétration :** Réalisez périodiquement des tests de pénétration sur votre infrastructure VPN pour identifier et corriger les vulnérabilités potentielles.

L'implémentation et la gestion correctes d'un VPN sont essentielles pour garantir la sécurité et la confidentialité des communications, particulièrement dans un contexte où les travailleurs accèdent fréquemment à des ressources d'entreprise à distance.

---
> [!info] Livre « Sécurité avancée sous Linux » — chapitre 5/7
> [[Sécurité avancée sous Linux — Sommaire|Sommaire]] · [[Sécurité avancée sous Linux — 04 — Authentification et accès réseau|← 04 — Authentification et accès réseau]] · [[Sécurité avancée sous Linux — 06 — Détection d'intrusions et réponse aux incidents|06 — Détection d'intrusions et réponse aux incidents →]]
