---
schema_version: 1
uid: 01M1BQ62C6M56XA0EX8HYK56MX
titre: "Sécurité avancée sous Linux — 03 — Contrôle d'accès avancé"
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
resume: "Chapitre 3 sur 7 du livre « Sécurité avancée sous Linux » : Contrôle d'accès avancé. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-15."
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

> [!info] Livre « Sécurité avancée sous Linux » — chapitre 3/7
> [[Sécurité avancée sous Linux — Sommaire|Sommaire]] · [[Sécurité avancée sous Linux — 02 — Sécurisation de l'environnement système|← 02 — Sécurisation de l'environnement système]] · [[Sécurité avancée sous Linux — 04 — Authentification et accès réseau|04 — Authentification et accès réseau →]]

# 3. Contrôle d'accès avancé
## 3.1. Introduction à SELinux et AppArmor
### 3.1. Introduction à SELinux et AppArmor

SELinux (Security-Enhanced Linux) et AppArmor sont deux des principaux systèmes de contrôle d'accès obligatoire (MAC, Mandatory Access Control) utilisés dans les environnements Linux pour renforcer la sécurité en limitant les capacités des applications et des processus selon des politiques de sécurité définies. Bien que servant des objectifs similaires, SELinux et AppArmor adoptent des approches légèrement différentes pour la mise en œuvre du MAC, chacun avec ses propres avantages et particularités.

### 3.1.2. **SELinux**

SELinux a été développé par la National Security Agency (NSA) des États-Unis et intégré dans le noyau Linux pour fournir un mécanisme robuste de contrôle d'accès basé sur des politiques. Il fonctionne en attribuant des étiquettes de sécurité, ou contextes, aux utilisateurs, processus, fichiers, et autres objets du système, et en définissant des politiques qui dictent les interactions possibles entre ces étiquettes. SELinux est conçu pour être extrêmement flexible et configurable, permettant une granularité fine dans la définition des politiques de sécurité.

- **Modes de fonctionnement :** SELinux peut opérer en trois modes : Enforcing, Permissive, et Disabled. En mode Enforcing, les politiques de sécurité sont appliquées et les violations sont bloquées et enregistrées. En mode Permissive, les violations sont enregistrées mais pas bloquées, permettant le dépannage et la configuration. En mode Disabled, SELinux est désactivé.

- **Politiques :** SELinux utilise des politiques complexes qui peuvent être adaptées aux besoins spécifiques de sécurité d'un système. Les politiques les plus couramment utilisées sont la politique ciblée, qui protège les services sensibles sans restreindre l'ensemble du système, et la politique MLS (Multi-Level Security), qui offre des contrôles d'accès basés sur des niveaux de classification.

### 3.1.2 AppArmor

AppArmor est un autre système de contrôle d'accès obligatoire, souvent utilisé dans les distributions Linux comme Ubuntu. Contrairement à SELinux, qui se base sur des étiquettes de sécurité, AppArmor utilise des profils pour les applications, définissant les fichiers auxquels elles peuvent accéder et les opérations qu'elles peuvent effectuer. AppArmor est réputé pour sa facilité de configuration et d'utilisation, rendant le contrôle d'accès avancé plus accessible aux administrateurs de systèmes moins expérimentés.

- **Profils :** Les profils AppArmor sont écrits dans un langage relativement simple et peuvent être en mode Enforce ou Complain. En mode Enforce, les actions non autorisées par le profil sont bloquées, tandis qu'en mode Complain, elles sont seulement enregistrées pour faciliter le développement et la mise au point des profils.

- **Utilisation :** AppArmor est particulièrement apprécié pour sa simplicité d'approche, avec des profils spécifiques à chaque application qui peuvent être activés, désactivés ou modifiés individuellement sans nécessiter une connaissance approfondie du système de contrôle d'accès sous-jacent.

### Histoire et évolution
### Comment AppArmor se distingue-t-il des autres systèmes de sécurité Linux (SELinux, etc.) ?
### Concepts fondamentaux d'AppArmor
 
 #### Politiques de sécurité et profils
 
 #### Modes d'enforcement : enforcing, complain, et disable
  
  #### Structure d'un profil AppArmor

### Installation et configuration d'AppArmor
#### Installation d'AppArmor et des utilitaires associés

 ### Commandes apt pour l'installation
 
 ### Activation et désactivation d'AppArmor
#### Mise à jour des politiques AppArmor

### Gestion des profils AppArmor
- **Localisation et structure des profils AppArmor**
    - /etc/apparmor.d/
- **Création et édition de profils**
    - Utilisation de `aa-genprof` et `aa-autodep`
    - Écriture manuelle de profils
- **Activation et désactivation de profils**
    - Utilisation de `aa-enforce`, `aa-complain`, et `aa-disable`
- **Gestion des modes d'un profil**
    - Comprendre les logs d'AppArmor pour ajuster les politiques

### Utilisation avancée d'AppArmor
- **Création de politiques de sécurité personnalisées**
    - Bonnes pratiques pour l'écriture de profils sécurisés
    - Exemples de configuration pour des applications spécifiques (serveurs web, bases de données)
- **Dépannage et audit**
    - Utilisation de `aa-logprof` et `aa-notify`
    - Interprétation des logs pour résoudre les violations de politiques
- **Intégration d'AppArmor avec d'autres outils de sécurité**
    - Complémentarité avec le pare-feu, SELinux, etc.

### Ressources complémentaires et travaux pratiques
- **Ressources en ligne** : documentation officielle, forums, articles spécialisés

### 3.1.3. Conclusion

SELinux et AppArmor représentent deux approches puissantes pour renforcer la sécurité des systèmes Linux en appliquant des politiques de contrôle d'accès obligatoire. Le choix entre SELinux et AppArmor dépendra souvent de la distribution Linux utilisée, des besoins spécifiques en matière de sécurité, et de la préférence personnelle en termes de complexité de configuration et de gestion. En comprenant les capacités et les cas d'utilisation de chacun, les administrateurs peuvent mieux protéger leurs systèmes contre les accès et les modifications non autorisés.
## 3.2. Concepts de politique de sécurité et de contrôle d'accès obligatoire (MAC)

Le Contrôle d'Accès Obligatoire (MAC, Mandatory Access Control) est un paradigme de sécurité qui diffère significativement des approches traditionnelles basées sur les permissions. Contrairement au Contrôle d'Accès Discrétionnaire (DAC, Discretionary Access Control), où les utilisateurs ont la liberté de contrôler l'accès à leurs propres ressources, le MAC impose des politiques de sécurité centralisées qui régissent les autorisations d'accès pour tous les utilisateurs et processus, indépendamment de leurs préférences individuelles. Cette section explore les concepts fondamentaux des politiques de sécurité et du MAC, et leur importance pour renforcer la posture de sécurité d'un système Linux.

## 3.2.1 Politiques de Sécurité

Une politique de sécurité est un ensemble de règles qui définissent comment les informations et les ressources d'un système doivent être protégées. Ces politiques couvrent divers aspects de la sécurité, y compris, mais sans s'y limiter, qui peut accéder à quelles données, comment les données doivent être protégées pendant le transfert et au repos, et comment les tentatives d'accès sont enregistrées et traitées. Dans le contexte du MAC, les politiques de sécurité spécifient des règles obligatoires qui s'appliquent uniformément à tous les éléments du système, assurant une gestion cohérente et sécurisée des accès.

### 3.3.2. Contrôle d'Accès Obligatoire (MAC)

Le MAC est caractérisé par plusieurs principes clés :

- **Politiques Centralisées :** Les politiques de MAC sont définies et gérées de manière centralisée, souvent par des administrateurs de sécurité, garantissant que les contrôles d'accès sont appliqués uniformément à travers le système.

- **Séparation des Privileges :** Le MAC permet une séparation stricte des privilèges en limitant l'accès des utilisateurs et des applications uniquement aux ressources nécessaires à leurs fonctions.

- **Contraintes d'Intégrité :** Certaines politiques de MAC, comme celles basées sur les modèles Biba ou Clark-Wilson, mettent l'accent sur le maintien de l'intégrité des données en contrôlant strictement qui peut modifier les informations.

- **Contrôles Basés sur les Rôles et les Responsabilités :** Le MAC peut implémenter des Contrôles d'Accès Basés sur les Rôles (RBAC, Role-Based Access Control) ou Basés sur les Attributs (ABAC, Attribute-Based Access Control) pour définir les autorisations en fonction des rôles professionnels ou des attributs spécifiques.

### 3.3.3. Application du MAC avec SELinux et AppArmor

- **SELinux :** Utilise des politiques de sécurité complexes qui associent des étiquettes de sécurité aux utilisateurs, processus et objets (fichiers, sockets, etc.), permettant un contrôle d'accès granulaire basé sur ces étiquettes. Les politiques de SELinux peuvent être configurées pour appliquer différents niveaux de contrôle, de la restriction des actions d'un service spécifique à la mise en œuvre de politiques de sécurité multi-niveaux pour la protection des données classifiées.

- **AppArmor :** Applique le MAC en utilisant des profils pour chaque programme, spécifiant les fichiers auxquels le programme peut accéder et les opérations qu'il peut effectuer. Les profils AppArmor sont relativement simples à créer et à gérer, rendant l'application de politiques de sécurité MAC plus accessible.

### 3.3.4. Conclusion

Les concepts de politique de sécurité et le Contrôle d'Accès Obligatoire sont fondamentaux pour la construction d'un système Linux sécurisé. En imposant des politiques de sécurité centralisées et rigoureuses à travers le MAC, les organisations peuvent mieux protéger leurs ressources contre les accès non autorisés et les compromissions, tout en maintenant un équilibre entre la sécurité et la fonctionnalité. Les systèmes comme SELinux et AppArmor offrent des outils puissants pour la mise en œuvre de ces politiques, permettant aux administrateurs de tirer parti des avantages du MAC tout en gérant la complexité inhérente à ces systèmes.
## 3.3. Configuration et administration de SELinux/AppArmor

La configuration et l'administration efficaces de SELinux et AppArmor sont essentielles pour sécuriser les systèmes Linux en appliquant des politiques de contrôle d'accès obligatoire (MAC). Ces outils permettent d'affiner la sécurité en définissant précisément comment les applications et les services interagissent avec le système et entre eux. Voici comment administrer et configurer SELinux et AppArmor pour renforcer la sécurité de votre système.

### 3.3.1. SELinux

- **Modes de Fonctionnement :** SELinux opère en trois modes : Enforcing, Permissive et Disabled. Le mode Enforcing applique les politiques de sécurité et bloque les actions non autorisées, le mode Permissive enregistre les violations sans les bloquer (utile pour le débogage), et le mode Disabled désactive SELinux. Changez de mode avec `setenforce Enforcing` ou `setenforce Permissive`, et vérifiez le mode actuel avec `getenforce`.

- **Gestion des Politiques :** Les politiques SELinux définissent les règles de contrôle d'accès. Utilisez les commandes `semanage`, `semodule` et `sepolgen` pour gérer les politiques, modules et règles personnalisées. Pour installer ou supprimer des modules de politique, utilisez `semodule -i module.pp` ou `semodule -r module`.

- **Étiquetage des Fichiers :** L'étiquetage correct des fichiers et des ressources est crucial pour SELinux. Utilisez `chcon` pour changer temporairement les contextes de sécurité des fichiers ou `restorecon` pour appliquer le contexte par défaut basé sur les politiques actuelles. Pour des modifications permanentes, utilisez `semanage fcontext`.

- **Outils d'Analyse et de Dépannage :** Des outils comme `ausearch` et `audit2why` peuvent aider à analyser les logs d'audit SELinux pour comprendre les violations de politique. `setroubleshoot-server` fournit des explications détaillées et des suggestions pour résoudre les problèmes de sécurité SELinux.

### 3.3.2. AppArmor

- **Activation et Désactivation des Profils :** AppArmor utilise des profils pour définir les permissions des applications. Utilisez `aa-enforce` pour placer un profil en mode Enforce et `aa-complain` pour le placer en mode Complain. Vérifiez le statut d'un profil avec `aa-status`.

- **Création et Modification des Profils :** Les profils AppArmor sont stockés dans `/etc/apparmor.d/`. Pour créer ou modifier un profil, éditez le fichier correspondant dans ce répertoire. Utilisez `aa-genprof` et `aa-autodep` pour générer automatiquement des profils de base pour les applications.

- **Gestion des Profils :** Chargez, déchargez ou rechargez des profils modifiés avec `apparmor_parser`. Par exemple, `apparmor_parser -r /etc/apparmor.d/profile.name` recharge un profil modifié.

- **Journalisation et Dépannage :** Les violations de profil AppArmor sont enregistrées dans le syslog. Utilisez les outils de visualisation de logs habituels pour examiner ces entrées et diagnostiquer les problèmes de permission.

### 3.3.3. **Bonnes Pratiques**

- **Audit Régulier :** Effectuez régulièrement des audits de sécurité pour vous assurer que les politiques SELinux et AppArmor sont correctement appliquées et ne bloquent pas indûment les fonctionnalités applicatives.

- **Formation et Documentation :** La complexité de SELinux et AppArmor peut représenter un défi. Investissez dans la formation des administrateurs et maintenez une documentation à jour sur les politiques et configurations spécifiques à votre environnement.

- **Communauté et Support :** Tirez parti des ressources disponibles en ligne, y compris la documentation officielle, les forums et les groupes d'utilisateurs pour obtenir de l'aide et partager les meilleures pratiques.

La configuration et l'administration adéquates de SELinux et AppArmor nécessitent une compréhension approfondie de vos besoins de sécurité et des fonctionnalités de votre système. En appliquant des politiques de contrôle d'accès rigoureuses et en exploitant les outils de dépannage disponibles, vous pouvez sécuriser efficacement vos systèmes Linux contre les accès non autorisés et les activités malveillantes.
## 3.4. Dépannage et audit des politiques SELinux/AppArmor

Les systèmes de contrôle d'accès obligatoire comme SELinux et AppArmor augmentent significativement la sécurité des systèmes Linux, mais leur complexité peut également introduire des défis en matière de dépannage. Les administrateurs doivent être équipés pour identifier et résoudre les problèmes liés aux politiques SELinux/AppArmor pour maintenir la fonctionnalité du système tout en assurant sa sécurité.

### 3.4.1 Dépannage SELinux

- **Identifier les Violations de Politique :** Utilisez `ausearch -m avc -ts recent` pour rechercher les violations de politiques SELinux récentes dans les logs d'audit. `audit2why` peut ensuite être utilisé pour analyser ces entrées et fournir des explications sur les violations et des suggestions pour les résoudre.

- **Gestion Temporaire des Problèmes :** En cas de blocage par SELinux, envisagez de passer temporairement en mode Permissive avec `setenforce 0`, permettant au système de fonctionner normalement tout en enregistrant les violations, facilitant ainsi le dépannage.

- **Ajustement des Politiques :** Si une application nécessite des accès non couverts par la politique SELinux en vigueur, utilisez `audit2allow` pour générer une politique personnalisée qui accorde l'accès nécessaire. Soyez prudent avec cette approche pour éviter d'affaiblir la sécurité du système.

- **Utilisation des Booleans SELinux :** SELinux offre des booleans qui permettent d'activer ou de désactiver certains comportements de politique de manière dynamique. Utilisez `getsebool -a` pour lister tous les booleans et `setsebool` pour ajuster leurs valeurs en fonction de vos besoins.

### 3.4.2. Dépannage AppArmor

- **Analyse des Logs :** Les violations de politique AppArmor sont enregistrées dans le syslog. Utilisez des commandes comme `grep apparmor /var/log/syslog` pour identifier les problèmes. Les messages fournissent des détails sur le profil violé et l'action bloquée.

- **Passage en Mode Complain :** Si un profil AppArmor bloque indûment une application, mettez le profil en mode Complain avec `aa-complain /etc/apparmor.d/name_of_profile`, qui enregistrera les actions bloquées sans les empêcher.

- **Modification des Profils :** Editez directement les profils AppArmor dans `/etc/apparmor.d/` pour ajuster les permissions. Utilisez `aa-genprof` pour faciliter la création et la modification des profils en surveillant l'exécution de l'application et en générant les règles nécessaires.

- **Rechargement des Profils :** Après modification, rechargez le profil avec `apparmor_parser -r /etc/apparmor.d/name_of_profile` pour appliquer les changements sans redémarrer le service ou le système.

### 3.4.3 Audit et Amélioration Continue

- **Audit Régulier des Politiques :** Réalisez des audits réguliers de vos politiques SELinux et AppArmor pour vous assurer qu'elles sont à jour avec les besoins de sécurité et de fonctionnalité du système. L'objectif est d'équilibrer sécurité et fonctionnalité sans introduire de restrictions inutiles.

- **Documentation et Revue :** Maintenez une documentation détaillée sur les personnalisations de politique pour faciliter le dépannage et les audits futurs. La revue périodique des politiques avec des experts en sécurité peut aider à identifier les améliorations potentielles.

- **Formation :** Investissez dans la formation continue pour les administrateurs système sur SELinux et AppArmor. Une meilleure compréhension de ces outils peut réduire les erreurs de configuration et améliorer la réponse aux incidents.

Le dépannage et l'audit des politiques SELinux et AppArmor sont des compétences essentielles pour les administrateurs de systèmes Linux souhaitant maintenir une posture de sécurité forte tout en assurant le bon fonctionnement des services. Une approche proactive, combinée à une connaissance approfondie de ces systèmes de contrôle d'accès, permettra d'identifier et de résoudre efficacement les problèmes tout en tirant parti des avantages en matière de sécurité qu'ils offrent.

---
> [!info] Livre « Sécurité avancée sous Linux » — chapitre 3/7
> [[Sécurité avancée sous Linux — Sommaire|Sommaire]] · [[Sécurité avancée sous Linux — 02 — Sécurisation de l'environnement système|← 02 — Sécurisation de l'environnement système]] · [[Sécurité avancée sous Linux — 04 — Authentification et accès réseau|04 — Authentification et accès réseau →]]
