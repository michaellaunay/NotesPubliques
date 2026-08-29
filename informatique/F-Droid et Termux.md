---
schema_version: 1
uid: "01M02JG1VCS6ME1Q0VWGXQGM2C"
titre: "F-Droid et Termux"
aliases:
  - "Termux"
  - "F-Droid"
  - "Linux sur Android"
type: procedure
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - android
  - termux
  - logiciel-libre
  - ligne-de-commande
resume: "Procédure pour installer un environnement de développement Linux sur Android en 2026 : F-Droid (magasin libre, dépôts, alternatives), Termux (sources officielles F-Droid et GitHub, différences de la version Google Play, versions 0.118.3 et 0.119 bêta), installation pas à pas, premiers réglages (pkg, stockage, SSH, Python, git), dépôt TUR et code-server pour VS Code, limites d'Android 12 à 16 et vérification des développeurs annoncée par Google."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2024-03-18
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# F-Droid et Termux

> [!abstract] Objectif
> Transformer un téléphone ou une tablette Android en poste de travail Linux d'appoint, sans accès root : installer **F-Droid** (magasin d'applications libres), puis **Termux** depuis une source officielle, configurer l'environnement (paquets, stockage, SSH, Python, git), ajouter un éditeur (VS Code par code-server) et connaître les limites imposées par Android.

> [!info] État de la procédure
> Vérifiée le 29 août 2026 sur la documentation de Termux et de F-Droid : Termux 0.118.3 (mai 2025) en version stable sur F-Droid et GitHub, 0.119.0 en bêta, version Google Play distincte (2026.01.07) ; client F-Droid 1.23. La procédure de 2024 demandait d'activer le mode développeur, ce qui n'est pas nécessaire pour installer une application ; l'étape a été retirée.

Voir aussi : [[git]] (chapitre 26, Git sur Android avec Termux), [[Visual studio code]], [[Sécurité avancée sous Linux]], [[Python]].

# 1. F-Droid

**F-Droid** est un magasin d'applications Android libres : le client lui-même, le serveur et les quelque 3 800 applications sont sous licences libres, compilées de façon reproductible par l'équipe F-Droid à partir des sources, sans compte utilisateur, sans traceur et avec la liste des *anti-fonctionnalités* (publicité, dépendance à un service non libre) affichée pour chaque application.

- **Dépôts** : le dépôt principal, plus des dépôts tiers ajoutés dans les paramètres (IzzyOnDroid pour les binaires publiés par les développeurs, dépôts d'un projet ou d'une organisation) ; c'est le mécanisme de **décentralisation** de F-Droid.
- **F-Droid Basic** : variante allégée du client, sans le partage de proximité, qui gère les mises à jour automatiques sur Android 12+.
- **Obtainium** (disponible sur F-Droid) installe et met à jour des applications directement depuis leurs pages GitHub, GitLab ou leur site : utile quand une application est publiée hors F-Droid.

> [!warning] Vérification des développeurs par Google
> Google a annoncé qu'à partir de septembre 2026, d'abord dans quelques pays, les applications installées sur les appareils Android certifiés devront provenir de développeurs vérifiés auprès de Google, y compris hors Play Store ; F-Droid a répondu par une lettre ouverte (février 2026, *Keep Android Open*). Selon la mise en œuvre, l'installation d'applications libres non enregistrées pourrait être bloquée ou limitée : suivre f-droid.org et keepandroidopen.org avant d'équiper une flotte d'appareils.

# 2. Termux

**Termux** est un émulateur de terminal qui installe un environnement Linux en espace utilisateur (bibliothèques et outils recompilés pour Android, chemin `/data/data/com.termux/files/usr`) : bash ou zsh, `pkg`/`apt` avec plus de 1 000 paquets (Python, Node.js, Git, SSH, GCC/Clang, Rust, Go, Vim, tmux…), sans root. Les greffons **Termux:API** (accès aux capteurs, notifications, presse-papiers, SMS), **Termux:Boot**, **Termux:Widget**, **Termux:Styling** et **Termux:X11** l'étendent.

## 2.1 D'où l'installer

| Source | État en 2026 | Remarques |
|---|---|---|
| **F-Droid** (`com.termux`) | source officielle, version stable 0.118.3 | APK universel (~180 Mo installé) ; les greffons doivent venir de la même source |
| **GitHub Releases** | source officielle, 0.118.3 stable et 0.119.0 bêta | APK par architecture (`arm64-v8a` pour presque tous les appareils actuels), variantes `apt-android-7` pour Android 7+ ; certains constructeurs refusent ces APK « conçus pour une ancienne version d'Android » : prendre alors F-Droid |
| **Google Play** | version distincte maintenue par le créateur de Termux dans l'organisation `termux-play-store` (2026.01.07), Android 11+ | clé de signature différente : **incompatible** avec les installations F-Droid/GitHub (désinstaller avant de changer de source) ; Termux:Boot et Widget intégrés à l'application ; l'équipe principale ne la considère pas comme officielle |

Règle : **F-Droid** pour la simplicité, **GitHub** pour choisir l'architecture ou tester la bêta, **Google Play** seulement si l'appareil interdit tout autre canal — et ne jamais mélanger les sources.

## 2.2 Ce qu'Android impose

- **Pas de root** : Termux ne modifie pas le système ; les outils qui l'exigent (`ping` brut, capture réseau, montage) ne fonctionnent pas ou passent par Termux:API.
- **Tueur de processus fantômes** (Android 12+) : le système peut tuer les processus en arrière-plan d'une application au-delà d'un plafond ; les tâches longues (serveur, compilation) doivent tourner dans une session visible avec un verrou de réveil (`termux-wake-lock`) ; le plafond se relève par `adb` sur un ordinateur (documentation Termux, « Phantom processes »).
- **Stockage** : l'accès aux fichiers de l'utilisateur passe par `termux-setup-storage`, qui crée `~/storage/` (Android 11+ demande l'autorisation « Accès à tous les fichiers » pour certains dossiers).
- **Exécution** depuis Android 10 : seuls les binaires installés par `pkg` ou compilés dans le préfixe Termux s'exécutent ; un binaire téléchargé dans `~/storage/downloads` doit être copié dans `$PREFIX/bin` ou `~`.
- **Pages de 16 Kio** (Android 15+ sur certains appareils) : prises en charge par les versions de 2025 et suivantes ; mettre à jour l'application et les paquets.

# 3. Installer

## 3.1 F-Droid

1. Sur l'appareil, ouvrir <https://f-droid.org/> dans le navigateur et télécharger l'APK du client (ou de F-Droid Basic).
2. Ouvrir le fichier téléchargé ; Android 8+ demande d'autoriser **ce navigateur** à installer des applications inconnues — autorisation donnée par application, pas globalement.
3. Lancer F-Droid, laisser l'index se télécharger (la première synchronisation prend une minute), puis dans les paramètres activer les mises à jour automatiques si l'appareil le permet.

Aucun mode développeur n'est requis : il ne sert qu'au débogage par `adb`.

## 3.2 Termux

1. Dans F-Droid, rechercher **Termux**, installer ; installer de la même façon **Termux:API** si l'on veut piloter l'appareil depuis le shell.
2. Ouvrir Termux : le système de base se déploie (quelques dizaines de secondes).
3. Mettre à jour :

```bash
pkg update && pkg upgrade -y
```

4. Donner accès au stockage, puis vérifier :

```bash
termux-setup-storage           # crée ~/storage/{shared,downloads,dcim,...}
ls ~/storage/shared
```

5. Installer l'essentiel :

```bash
pkg install -y git openssh python nodejs vim tmux termux-api
python --version && git --version
```

## 3.3 Réglages utiles

- **Clavier** : la barre de touches supplémentaires (Échap, Tab, Ctrl, flèches) se règle dans `~/.termux/termux.properties` (`extra-keys`) ; un clavier Bluetooth ou USB fonctionne directement.
- **SSH vers l'appareil** : `sshd` écoute sur le port **8022** ; définir un mot de passe (`passwd`) ou copier une clé dans `~/.ssh/authorized_keys`, puis `ssh -p 8022 utilisateur@adresse` depuis l'ordinateur (`whoami` donne le nom, `ifconfig` l'adresse).
- **Sauvegarde** : `termux-backup ~/storage/downloads/termux.tar.xz` et `termux-restore` archivent tout l'environnement.
- **Dépôts supplémentaires** : `pkg install tur-repo` ajoute le **Termux User Repository** (paquets communautaires, dont code-server) ; `pkg install x11-repo` les applications graphiques pour Termux:X11.
- **Distribution complète** : `pkg install proot-distro && proot-distro install debian && proot-distro login debian` donne un Debian ou Ubuntu émulé (plus lent, mais avec `apt` classique) pour les logiciels absents de Termux.

# 4. VS Code sur Android

Deux voies, sans passer par des tutoriels tiers :

**code-server dans Termux** — l'éditeur VS Code servi par un navigateur, sur l'appareil lui-même :

```bash
pkg install tur-repo
pkg install code-server
code-server --auth none --bind-addr 127.0.0.1:8080
```

puis ouvrir <http://127.0.0.1:8080> dans le navigateur (Chrome, Firefox, ou un navigateur en mode « application » pour le plein écran). Les extensions viennent de la place de marché Open VSX ; l'accès distant d'un autre appareil impose `--auth password` et un tunnel SSH.

**Développement distant** — l'inverse : l'appareil Android sert de terminal et de client, le code s'exécute ailleurs. Termux fournit `ssh` et `mosh` ; VS Code sur l'ordinateur se connecte à l'appareil par **Remote-SSH** sur le port 8022 pour éditer les fichiers de Termux avec l'éditeur complet (voir [[Visual studio code]], chapitre sur le développement distant).

# 5. Premier projet : Python et git

```bash
mkdir -p ~/projets/demo && cd ~/projets/demo
git init && git config user.name "Prénom Nom" && git config user.email "prenom.nom@example.org"
python -m venv .venv && . .venv/bin/activate
pip install requests
cat > main.py <<'PY'
import platform, requests
print(platform.machine(), requests.__version__)
PY
python main.py
git add . && git commit -m "Premier commit depuis Termux"
```

Les dépôts distants se clonent et se poussent avec `ssh` ou `https` exactement comme sur un ordinateur ; les particularités (agent SSH, clés, `safe.directory`) sont dans [[git]], chapitre 26.

# 6. Dépannage

| Symptôme | Cause | Remède |
|---|---|---|
| « Application non installée » ou « conçue pour une ancienne version d'Android » | APK GitHub refusé par le constructeur, ou source différente déjà installée | installer depuis F-Droid ; désinstaller l'ancienne source avant d'en changer |
| `pkg` échoue sur les miroirs | miroir hors ligne | `termux-change-repo` pour en choisir un autre |
| Processus tué en arrière-plan | tueur de processus fantômes | `termux-wake-lock`, session au premier plan, plafond relevé par `adb` |
| `Permission denied` sur `~/storage` | autorisation non accordée | relancer `termux-setup-storage`, vérifier « Fichiers et médias » dans les paramètres de l'application |
| Binaire téléchargé non exécutable | restriction d'exécution Android 10+ | le placer dans `$PREFIX/bin` ou `~`, `chmod +x` |
| Google Play propose une mise à jour d'un Termux F-Droid | versions confondues | désactiver la mise à jour automatique de Termux dans Play, ou ignorer |

# 7. Sources

- Termux, documentation et installation : <https://termux.dev/> ; dépôt <https://github.com/termux/termux-app> ; wiki <https://wiki.termux.com/>
- Termux sur Google Play (organisation `termux-play-store`) et journal des différences : <https://github.com/termux-play-store>
- Termux User Repository : <https://github.com/termux-user-repository/tur>
- F-Droid : <https://f-droid.org/> ; lettre ouverte *Keep Android Open* : <https://keepandroidopen.org/>
- code-server : <https://coder.com/docs/code-server>
- proot-distro : <https://github.com/termux/proot-distro>
