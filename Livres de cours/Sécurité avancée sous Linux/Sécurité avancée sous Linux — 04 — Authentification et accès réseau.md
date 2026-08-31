---
schema_version: 1
uid: 01M1BQ62C7AVRR6GDWCRTZF8WM
titre: "Sécurité avancée sous Linux — 04 — Authentification et accès réseau"
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
resume: "Chapitre 4 sur 7 du livre « Sécurité avancée sous Linux » : Authentification et accès réseau. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-15."
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

> [!info] Livre « Sécurité avancée sous Linux » — chapitre 4/7
> [[Sécurité avancée sous Linux — Sommaire|Sommaire]] · [[Sécurité avancée sous Linux — 03 — Contrôle d'accès avancé|← 03 — Contrôle d'accès avancé]] · [[Sécurité avancée sous Linux — 05 — Cryptographie et sécurisation des données|05 — Cryptographie et sécurisation des données →]]

# 4. Authentification et accès réseau
## 4.1. Configuration sécurisée de SSH

SSH (Secure Shell) est un protocole vital pour la gestion sécurisée des systèmes à distance, offrant un canal crypté pour l'accès à distance et le transfert de fichiers. Une configuration correcte de SSH est essentielle pour protéger contre les attaques de réseau, les tentatives de brute force et les écoutes clandestines. Voici les meilleures pratiques pour sécuriser votre configuration SSH.

### 4.1.1. Utilisation de Clés SSH au Lieu de Mots de Passe

- **Génération de Clés SSH :** Encouragez l'utilisation de clés SSH plutôt que de mots de passe pour l'authentification. Les clés offrent une sécurité supérieure en nécessitant une clé privée que l'utilisateur doit posséder.
- **Désactivation de l'Authentification par Mot de Passe :** Dans le fichier de configuration SSH (`/etc/ssh/sshd_config`), réglez `PasswordAuthentication` sur `no` pour désactiver l'authentification par mot de passe.

### 4.1.2. **Changement du Port par Défaut

- **Modification du Port SSH :** Changer le port d'écoute par défaut (22) pour un autre port peut aider à réduire les scans automatiques et les attaques de brute force. Configurez cela avec l'option `Port` dans `sshd_config`.

### 4.1.3. Limitation des Utilisateurs Autorisés

- **Restriction d'Accès :** Utilisez les directives `AllowUsers` ou `AllowGroups` dans `sshd_config` pour limiter les utilisateurs ou les groupes qui peuvent se connecter via SSH, réduisant ainsi la surface d'attaque.

### 4.1.4. Désactivation de l'Accès SSH Root

- **Interdiction de la Connexion Root :** Réglez `PermitRootLogin` sur `no` pour empêcher l'authentification directe de l'utilisateur root. Cela force l'utilisation de comptes d'utilisateurs normaux et l'élévation de privilèges via `sudo` pour les tâches d'administration.

### 4.1.5. Utilisation de Protocoles Sécurisés

- **Forcer l'Utilisation de SSHv2 :** Assurez-vous que `Protocol 2` est défini, car SSHv2 est plus sécurisé que la version précédente.

### 4.1.6. Délai de Connexion et Tentatives de Connexion Limitées

- **Configuration du Délai et des Tentatives :** Réduire le nombre de tentatives de connexion non réussies et configurer un délai de connexion peut aider à prévenir les attaques de brute force. Utilisez `LoginGraceTime` et `MaxAuthTries` pour ces paramètres.

### 4.1.7. Surveillance et Journalisation

- **Augmentation de la Verbosité des Logs :** Configurer `LogLevel` sur `VERBOSE` pour enregistrer plus de détails sur les tentatives de connexion, facilitant l'identification et la réponse aux activités suspectes.

### 4.1.8. Utilisation de Fail2Ban

- **Protection Contre les Attaques de Brute Force :** Fail2Ban est un outil qui analyse les logs des serveurs et bannit les adresses IP qui montrent des signes d'attaques de brute force. Configurez Fail2Ban pour surveiller les logs SSH et agir en conséquence.

### 4.1.9. Mise à Jour Régulière du Logiciel

- **Maintenir SSH à Jour :** Assurez-vous que votre logiciel SSH est régulièrement mis à jour pour inclure les derniers correctifs de sécurité.

En suivant ces pratiques recommandées pour la configuration de SSH, vous pouvez considérablement renforcer la sécurité de votre accès à distance, protégeant ainsi votre système contre les intrusions et les abus tout en assurant un accès sécurisé pour les utilisateurs autorisés.

## 4.2. Gestion des clés et des certificats

La gestion efficace des clés et des certificats est essentielle pour sécuriser les communications et l'authentification dans les réseaux modernes. Elle joue un rôle crucial dans la protection des échanges de données, l'authentification des serveurs et des utilisateurs, et la mise en place d'une infrastructure à clé publique (PKI). Voici les meilleures pratiques pour gérer les clés et les certificats de manière sécurisée.

### 4.2.1. Politique de Gestion des Clés

- **Développer une Politique :** Établissez une politique de gestion des clés qui couvre la création, la distribution, le stockage, la rotation et la suppression des clés. Cette politique doit inclure des directives pour les clés SSH, les certificats SSL/TLS, et toute autre forme de clés cryptographiques utilisées dans votre organisation.

### 4.2.2. Sécurité de la Clé Privée

- **Protection des Clés Privées :** Assurez la sécurité des clés privées en utilisant des mots de passe forts et en les stockant dans des endroits sécurisés, tels que des gestionnaires de mots de passe dédiés ou des modules de sécurité matérielle (HSM). Évitez de stocker les clés privées sur des systèmes ou des réseaux accessibles.

### 4.2.3. **Rotation et Renouvellement

- **Rotation Régulière :** Mettez en place une rotation régulière des clés et des certificats pour limiter la fenêtre d'exposition en cas de compromission. Le renouvellement fréquent des certificats SSL/TLS est également important pour maintenir la confiance et la sécurité des communications.

### 4.2.4. Autorité de Certification

- **Utiliser des AC Reconnues :** Pour les certificats SSL/TLS, préférez les émettre via une Autorité de Certification (AC) reconnue plutôt que d'utiliser des certificats auto-signés pour les services exposés publiquement. Cela augmente la confiance des utilisateurs dans la sécurité de vos services.

### 4.2.5. Audit et Suivi

- **Inventaire des Clés et Certificats :** Maintenez un inventaire à jour de toutes les clés et certificats utilisés, y compris leur emplacement, leur usage, et leur date d'expiration. Utilisez des outils d'audit pour identifier les certificats expirés ou proches de leur expiration et les clés non utilisées ou obsolètes.

### 4.2.6. Sécurisation des Échanges de Clés

- **Protocoles Sécurisés :** Lors de la création ou de l'échange de clés, utilisez toujours des protocoles sécurisés pour éviter l'interception ou la compromission des clés. Pour SSH, cela signifie éviter les échanges de clés en clair et utiliser des méthodes sécurisées pour la distribution des clés publiques.

### 4.2.7. Chiffrement de Bout en Bout

- **Chiffrement Fort :** Assurez-vous que les clés et les certificats sont utilisés pour mettre en place un chiffrement de bout en bout pour toutes les communications sensibles, protégeant ainsi les données échangées contre l'écoute clandestine et l'interception.

### 4.2.8. Formation et Sensibilisation

- **Éducation sur la Sécurité :** Fournissez une formation régulière aux employés sur l'importance de la gestion sécurisée des clés et des certificats, y compris les meilleures pratiques pour leur utilisation, leur stockage et leur partage.

La gestion des clés et des certificats est une composante fondamentale de la sécurité réseau et de l'authentification. En suivant ces pratiques recommandées, les organisations peuvent renforcer leur sécurité, prévenir les abus et garantir la confidentialité et l'intégrité des communications et des données.

## 4.3. Sécurisation de la connexion réseau et services

La sécurisation des connexions réseau et des services est essentielle pour protéger les données sensibles et maintenir l'intégrité et la disponibilité des systèmes informatiques. Cette tâche implique une série de stratégies et de configurations visant à minimiser les risques d'attaques externes et internes, garantissant que les communications restent confidentielles, authentiques et intègres. Voici comment sécuriser efficacement vos connexions réseau et services.

### 4.3.1. Chiffrement des Données en Transit

- **Utilisation de Protocoles Sécurisés :** Assurez-vous que tout le trafic réseau, en particulier celui contenant des informations sensibles, est chiffré en utilisant des protocoles tels que TLS pour le web (HTTPS), SSH pour l'accès à distance, et IPSec pour les VPN. Cela aide à protéger les données contre l'écoute clandestine et les manipulations.

- **Configuration Forte de TLS :** Pour les services web, configurez TLS avec des suites de chiffrement fortes, désactivez les anciennes versions de protocoles (comme SSLv3 et TLS 1.0/1.1), et utilisez des certificats valides émis par une autorité de certification reconnue.

### 4.3.2. Sécurisation des Points d'Accès Réseau

- **Pare-feu et Filtres :** Utilisez des pare-feu pour contrôler l'accès aux services en bloquant le trafic non autorisé et en ne permettant que les connexions nécessaires. Configurez des listes de contrôle d'accès (ACL) sur les routeurs et les commutateurs pour filtrer le trafic à un niveau plus granulaire.

- **Segmentation du Réseau :** Divisez le réseau en segments ou en VLANs pour isoler les différents types de trafic et les niveaux de sensibilité. Cela limite la portée des attaques potentielles et facilite la surveillance et le contrôle du trafic.

### 4.3.3. Authentification et Autorisation

- **Contrôle d'Accès Basé sur les Rôles (RBAC) :** Implémentez un modèle RBAC pour gérer l'accès aux services et aux ressources en fonction des rôles des utilisateurs, en s'assurant que chacun a uniquement accès aux informations et aux fonctions nécessaires à ses tâches.

- **Authentification Forte :** Mettez en œuvre une authentification multifacteur (MFA) pour les services critiques, augmentant la sécurité en exigeant une seconde forme de vérification au-delà du simple mot de passe.

### 4.3.4. Surveillance et Détection des Anomalies

- **Systèmes de Détection d'Intrusion (IDS) :** Utilisez des IDS pour surveiller le réseau à la recherche de signes d'activités suspectes ou malveillantes, permettant une réponse rapide aux incidents de sécurité potentiels.

- **Journalisation et Analyse :** Configurez une journalisation détaillée pour tous les services et analysez régulièrement les logs à la recherche de comportements anormaux qui pourraient indiquer une tentative d'attaque ou une compromission.

### 4.3.5. Mises à Jour et Patchs

- **Maintenance des Logiciels :** Gardez tous les logiciels, en particulier ceux exposés sur le réseau, à jour avec les derniers patchs de sécurité pour corriger les vulnérabilités connues qui pourraient être exploitées par des attaquants.

### 4.3.6. Formation et Sensibilisation

- **Éducation des Utilisateurs :** Sensibilisez les utilisateurs aux risques de sécurité liés au réseau et formez-les sur les bonnes pratiques pour identifier et éviter les menaces potentielles, comme le phishing ou les attaques par hameçonnage.

En intégrant ces pratiques dans la gestion de la sécurité réseau, les organisations peuvent réduire significativement les risques liés à la cybercriminalité et aux attaques, tout en assurant la protection des données sensibles et la continuité des opérations commerciales. La clé réside dans une approche proactive et multiforme qui combine la technologie, les processus et l'éducation pour créer une défense robuste contre les menaces évolutives.
## 4.4. Firewalling avec iptables et nftables

Le firewalling est un mécanisme essentiel de défense pour tout système ou réseau informatique, servant à filtrer le trafic entrant et sortant selon des règles spécifiques. Dans l'écosystème Linux, `iptables` et `nftables` sont deux outils puissants pour la mise en place de règles de firewall, chacun ayant ses propres caractéristiques et avantages.

### 4.4.1. iptables

`iptables` est l'outil de manipulation de table de filtrage traditionnel dans Linux, permettant de définir des règles de filtrage pour le trafic IPv4. Il utilise une série de tables (comme la table `filter`, `nat`, et `mangle`) qui contiennent des chaînes (comme `INPUT`, `FORWARD`, et `OUTPUT`), auxquelles des règles sont ajoutées pour contrôler le flux de paquets.

- **Configuration de base :** Pour bloquer tout le trafic entrant mais autoriser le trafic sortant, on pourrait utiliser :
  ```bash
  iptables -P INPUT DROP
  iptables -P FORWARD DROP
  iptables -P OUTPUT ACCEPT
  iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
  ```
  Cette configuration bloque tout le trafic entrant, sauf les connexions déjà établies ou liées à des connexions autorisées.

- **Gestion des règles :** Les règles sont ajoutées avec `iptables -A` pour append ou `iptables -I` pour insérer dans une chaîne spécifique. Utilisez `iptables -L` pour lister les règles actives.

### 4.4.2. nftables

`nftables` est le successeur de `iptables`, introduit pour simplifier la structure de règles et fournir une meilleure performance. Il consolide plusieurs outils (`iptables`, `ip6tables`, `arptables`, et `ebtables`) sous un même framework, utilisant une syntaxe plus simple pour définir les règles de filtrage pour à la fois IPv4 et IPv6.

- **Configuration de base :** Une règle de base pour permettre le trafic sortant mais bloquer tout le trafic entrant non sollicité pourrait ressembler à :
  ```bash
  nft add rule ip filter input ct state related,established accept
  nft add rule ip filter input drop
  ```
  Ceci configure `nftables` pour accepter le trafic entrant associé à des connexions établies ou liées, tout en bloquant les autres.

- **Gestion des règles :** Les règles sont gérées via la commande `nft`. Utilisez `nft add rule` pour ajouter des règles, `nft list ruleset` pour afficher toutes les règles actives, et `nft delete rule` pour supprimer des règles.

### 4.4.3. Migration d'iptables à nftables

- **Outil de migration :** Pour faciliter la transition d'`iptables` à `nftables`, un outil de migration (`iptables-translate` et `ip6tables-translate`) est disponible, permettant de convertir les anciennes règles iptables en syntaxe nftables.

### 4.4.4. Bonnes Pratiques

- **Minimalisme :** Adoptez une approche de minimalisme dans vos règles, en n'autorisant que le trafic nécessaire pour minimiser la surface d'attaque.
- **Audit régulier :** Révisez et auditez régulièrement vos règles de firewall pour s'assurer qu'elles restent pertinentes et sécurisées par rapport à l'évolution du paysage des menaces.
- **Testez vos règles :** Avant de déployer de nouvelles règles en production, testez-les dans un environnement de développement ou de staging pour s'assurer qu'elles fonctionnent comme prévu sans impacter négativement les services légitimes.

En adoptant `iptables` ou `nftables` pour vos besoins de firewalling, vous pouvez créer une barrière robuste contre les accès non autorisés tout en permettant au trafic légitime de circuler comme nécessaire pour vos applications et services.

## 4.5. Sécurité des protocoles réseau (DNS, DHCP, HTTP/HTTPS)

Les protocoles réseau comme DNS, DHCP, HTTP et HTTPS sont essentiels au fonctionnement quotidien des réseaux informatiques, mais ils peuvent également présenter des vulnérabilités si non correctement sécurisés. Voici des stratégies pour renforcer la sécurité de ces protocoles essentiels.

### 4.5.1. DNS (Domain Name System)

- **DNSSEC (DNS Security Extensions) :** Implémentez DNSSEC pour valider les réponses DNS avec une signature numérique, empêchant les attaques de type man-in-the-middle et les empoisonnements de cache DNS.
- **Serveurs DNS fiables :** Utilisez des serveurs DNS réputés et sécurisés pour vos requêtes, comme ceux offerts par Cloudflare (1.1.1.1) ou Google (8.8.8.8).
- **Isolation du serveur DNS :** Si vous exploitez votre propre serveur DNS, isolez-le dans un réseau sécurisé pour minimiser l'exposition aux attaques.

### 4.5.2. DHCP (Dynamic Host Configuration Protocol)

- **Authentification :** Utilisez l'authentification dans les communications DHCP pour éviter les attaques par des serveurs DHCP malveillants qui pourraient attribuer des paramètres réseau incorrects.
- **Surveillance du réseau :** Surveillez activement les anomalies dans les assignations d'adresses IP pour détecter les tentatives d'usurpation DHCP.
- **Réseaux statiques pour les appareils sensibles :** Attribuez des adresses IP statiques aux serveurs et aux appareils critiques pour éviter les risques associés aux changements d'adresse IP dynamique.

### 4.5.3. HTTP/HTTPS (Hypertext Transfer Protocol / Secure)

- **Forcer HTTPS :** Assurez-vous que tout le trafic web est servi sur HTTPS, non seulement pour chiffrer les données en transit mais aussi pour fournir une authentification du serveur. Utilisez la politique de sécurité de transport strict (HSTS) pour forcer les navigateurs à utiliser des connexions sécurisées.
- **Certificats valides :** Utilisez des certificats SSL/TLS émis par une autorité de certification reconnue pour vos sites web. Considérez l'utilisation de Let's Encrypt pour obtenir des certificats gratuitement.
- **Désactivation des protocoles et chiffrements faibles :** Désactivez les anciennes versions de TLS (1.0 et 1.1) et les suites de chiffrement faibles pour réduire les vulnérabilités.

### 4.5.4. Bonnes Pratiques Communes

- **Mises à jour régulières :** Gardez tous les logiciels relatifs à ces protocoles (serveurs DNS, serveurs web, etc.) à jour avec les derniers patchs de sécurité pour protéger contre les vulnérabilités connues.
- **Formation et sensibilisation :** Éduquez les utilisateurs et les administrateurs réseau sur les risques de sécurité associés à ces protocoles et sur les meilleures pratiques pour les sécuriser.
- **Surveillance et réponse aux incidents :** Mettez en place une surveillance continue du trafic réseau pour détecter les activités suspectes et établissez un plan de réponse aux incidents pour réagir rapidement en cas de compromission.

La sécurisation des protocoles réseau DNS, DHCP, HTTP et HTTPS est une étape critique pour protéger l'intégrité, la confidentialité et la disponibilité des communications sur Internet et au sein des réseaux internes. En adoptant ces stratégies, les organisations peuvent réduire significativement leur surface d'attaque et améliorer la posture de sécurité globale de leur infrastructure réseau.

---
> [!info] Livre « Sécurité avancée sous Linux » — chapitre 4/7
> [[Sécurité avancée sous Linux — Sommaire|Sommaire]] · [[Sécurité avancée sous Linux — 03 — Contrôle d'accès avancé|← 03 — Contrôle d'accès avancé]] · [[Sécurité avancée sous Linux — 05 — Cryptographie et sécurisation des données|05 — Cryptographie et sécurisation des données →]]
