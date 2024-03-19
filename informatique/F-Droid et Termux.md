# F-Droid
F-Droid est un magasin d'applications pour les appareils Android, similaire au Google Play Store, mais avec quelques différences clés qui le rendent unique. Voici les principaux aspects de F-Droid :

### Open Source
- **Entièrement Open Source** : Toutes les applications disponibles sur F-Droid sont open source, ce qui signifie que leur code source est accessible et modifiable par quiconque. Cela favorise la transparence, la sécurité et la possibilité pour les utilisateurs de contribuer aux applications qu'ils utilisent.

### Sécurité et Vie Privée
- **Respect de la vie privée** : F-Droid accorde une grande importance à la vie privée et à la sécurité. Il ne suit pas les utilisateurs, ne requiert pas un compte pour télécharger des applications, et fournit des informations détaillées sur les autorisations et les trackers éventuels dans les applications.
- **Vérification des applications** : Les applications soumises à F-Droid sont vérifiées et compilées à partir de leurs sources par les équipes de F-Droid, ce qui réduit les risques de malware comparé aux APK téléchargés à partir de sources inconnues.

### Communauté et Développement
- **Communauté** : F-Droid est soutenu par une communauté de développeurs et d'utilisateurs qui contribuent non seulement au développement des applications mais aussi à celui de la plateforme F-Droid elle-même.
- **Pas de centralisation** : Contrairement au Google Play Store, F-Droid permet aux utilisateurs de configurer et d'ajouter des dépôts (repositories) tiers, favorisant ainsi un écosystème moins centralisé.

### Gratuité
- **Applications gratuites** : Toutes les applications sur F-Droid sont gratuites. Bien que des dons aux développeurs soient encouragés, nous n'aurons pas à payer pour télécharger des applications.

### Pourquoi utiliser F-Droid ?
- **Vie privée et sécurité** : Si nous sommes préoccupé par votre vie privée et sécurité, F-Droid fournit une plateforme où nous pouvons télécharger des applications en étant moins inquiet sur le suivi et le malware.
- **Soutien au logiciel libre** : En utilisant F-Droid, nous soutenons le mouvement du logiciel libre et open source, en contribuant à un écosystème qui valorise la transparence et la communauté.
- **Accès à des applications uniques** : Certaines applications sont disponibles uniquement sur F-Droid et pas sur d'autres plateformes de téléchargement d'applications, souvent en raison de leur engagement fort envers l'open source et la vie privée.
# Termux

Termux est une application de terminal pour Android qui offre un environnement de ligne de commande puissant directement sur votre appareil mobile. Elle permet aux utilisateurs d'accéder à un large éventail d'outils Linux, de faire du développement de logiciels, de gérer des fichiers, et d'effectuer diverses tâches de programmation et de réseautage sans avoir besoin de root ou de configuration supplémentaire. Voici quelques points clés concernant Termux :

### Environnement Linux complet
- Termux fournit un environnement Linux qui fonctionne directement sur votre appareil Android, sans nécessiter d'accès root. Cela signifie que nous pouvons utiliser des commandes Unix standard et installer des centaines de paquets disponibles via son gestionnaire de paquets `pkg` ou `apt`.

### Outils de développement
- Avec Termux, nous pouvons écrire, compiler et exécuter du code dans plusieurs langages de programmation, tels que Python, PHP, Ruby, et bien d'autres. Il est particulièrement apprécié des développeurs et des passionnés de technologie pour la flexibilité et la liberté qu'il offre pour coder et tester des scripts directement sur leurs appareils Android.

### Personnalisation et scripts
- Les utilisateurs peuvent personnaliser leur environnement Termux, écrire et exécuter des scripts, et automatiser des tâches grâce à des outils de script comme Bash, Zsh ou Fish. Cela permet une personnalisation poussée et la création de workflows efficaces.

### Accès au système de fichiers
- Termux offre un accès au système de fichiers de l'appareil, permettant ainsi la gestion des fichiers et dossiers. Bien qu'il n'ait pas besoin de droits superutilisateur pour fonctionner, Termux fournit des fonctionnalités puissantes à ses utilisateurs tout en respectant les limites de sécurité d'Android.

### Connectivité réseau
- L'application inclut des outils pour l'inspection réseau et la communication, tels que ssh, scp, curl, et wget. Cela en fait un outil précieux pour les administrateurs système et les professionnels de la sécurité informatique pour effectuer des diagnostics et gérer des systèmes à distance.

### Extensible via des add-ons
- Termux peut être étendu avec des add-ons, permettant par exemple d'accéder à l'API Android, de contrôler votre appareil Android via des commandes, ou d'ajouter une interface graphique à certains programmes.

### Communauté et open source
- Étant un projet open source, Termux bénéficie du soutien d'une communauté active de développeurs et d'utilisateurs qui contribuent à son amélioration continue. Il a une documentation riche et des forums où les utilisateurs peuvent obtenir de l'aide et partager leurs connaissances.
# Installation
Installer Termux via F-Droid sur un appareil Android nécessite plusieurs étapes, notamment l'activation du mode développeur et la confiance envers l'application F-Droid. Voici un guide étape par étape pour nous aider à le faire :

### 1. Activer le mode développeur
1. **Ouvrir les paramètres** de notre appareil Android.
2. Faire défiler vers le bas et sélectionner **À propos du téléphone**.
3. Trouver **Numéro de build** ou **Version du logiciel** et **taper dessus 7 fois**. Nous devrons peut-être entrer notre code PIN ou mot de passe.
4. Un message apparaîtra indiquant que nous sommes maintenant des développeurs.

### 2. Activer les sources inconnues (optionnel selon la version d'Android)
Dans les versions d'Android antérieures à Android 8 (Oreo), il est nécessaire d'autoriser l'installation d'applications en dehors du Google Play Store :
1. Revenir aux **Paramètres** et sélectionner **Sécurité**.
2. Activer l'option **Sources inconnues** pour permettre l'installation d'applications en dehors du Google Play Store.

Sur Android 8 (Oreo) et versions ultérieures, cette option est gérée au cas par cas lors de l'installation d'une application.

### 3. Installer F-Droid
1. Sur notre appareil Android, ouvrir un navigateur web et aller sur le site officiel de F-Droid ([https://f-droid.org/](https://f-droid.org/)).
2. Télécharger l'**APK de F-Droid**.
3. Une fois le téléchargement terminé, ouvrir le fichier APK. Nous devrons peut-être autoriser notre navigateur à installer des applications de sources inconnues si nous ne l'avons pas déjà fait.
4. Suivre les instructions à l'écran pour installer F-Droid.

### 4. Installer Termux via F-Droid
1. Ouvrir **F-Droid** une fois installé.
2. Utiliser la fonction de recherche pour trouver **Termux**.
3. Sélectionner Termux dans les résultats de recherche et cliquer sur **Installer**.

### 5. Mettre à jour les paquets de Termux (optionnel mais recommandé)
Après avoir installé Termux, il est bon de mettre à jour les paquets :
1. Ouvrir Termux.
2. Taper `pkg update` et appuyer sur Entrée pour mettre à jour la liste des paquets.
3. Taper `pkg upgrade` et appuyer sur Entrée pour mettre à jour les paquets installés.

Ceci nous permettra d'avoir la dernière version de Termux et de tous les outils ou paquets que nous souhaitons utiliser. Notez que la disponibilité de Termux sur F-Droid peut varier et qu'il est toujours conseillé de vérifier les sources officielles pour obtenir les informations les plus à jour.

# Installation de VS Code sur Android
https://www.codewithharry.com/blogpost/install-vs-code-in-android/