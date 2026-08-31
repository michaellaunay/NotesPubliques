---
schema_version: 1
uid: "01M02EX5CBAEZ0AZM5AMV0J7W7"
titre: "WSL2"
aliases:
  - "WSL 2"
  - "Windows Subsystem for Linux"
  - "Sous-système Windows pour Linux"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - gnu-linux
  - windows
  - wsl
  - virtualisation
  - developpement
  - systemd
  - reseau
resume: "Cours complet sur WSL 2 : architecture, installation et cycle de vie des distributions, systemd, WSLg, interopérabilité Windows/Linux, systèmes de fichiers, réseau NAT et mirrored, configuration, sauvegarde, conteneurs, GPU, USB, sécurité et dépannage."
niveau: intermediaire
prerequis:
  - "[[GNULinux]]"
  - "[[Installation Ubuntu]]"
  - "[[Les namespaces Linux]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-03-07
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Refonte
> Cours développé le 31 août 2026 (d'environ 621 à 7 926 mots  remplacement complet du texte précédent)  vérifié le même jour : schéma  titres  liens et affirmations datées contrôlés. La version précédente reste dans l'historique git du dépôt.

# Windows Subsystem for Linux 2 — WSL 2

> [!abstract] Objectif
> Comprendre ce qu'est réellement **WSL 2**, savoir l'installer et l'administrer proprement, choisir entre WSL 1 et WSL 2, organiser les fichiers sans perdre de performances, utiliser `systemd` et WSLg, exposer des services réseau, sauvegarder une distribution et diagnostiquer les problèmes les plus fréquents.

Voir aussi : [[GNULinux]], [[Installation Ubuntu]], [[Les namespaces Linux]], [[Docker]], [[Visual studio code]], [[Sécurité avancée sous Linux]], [[Watchdog espace disque]].

> [!important] Idée centrale
> WSL 2 n'est ni « un terminal Linux pour Windows », ni une machine virtuelle classique que l'utilisateur doit administrer. Il exécute un **vrai noyau Linux** dans une **machine virtuelle utilitaire légère gérée par Windows**, puis y fait fonctionner une ou plusieurs distributions Linux avec une forte intégration Windows/Linux.

# 1. Pourquoi WSL existe

Un poste de développement Windows rencontre souvent un besoin paradoxal :

- conserver les applications, pilotes et outils Windows ;
- disposer en même temps d'un environnement Linux crédible ;
- utiliser Bash, SSH, Git, Python, GCC, systemd, Docker, les outils cloud ou Kubernetes ;
- éviter la lourdeur d'une machine virtuelle traditionnelle ;
- partager facilement des fichiers, des ports et le presse-papiers ;
- pouvoir lancer ponctuellement une application graphique Linux.

Historiquement, plusieurs solutions étaient possibles :

```text
Dual boot
   │
   ├── Linux natif
   └── Windows natif

Machine virtuelle classique
   │
   └── Linux complet dans Hyper-V / VMware / VirtualBox

WSL
   │
   ├── intégration très forte avec Windows
   ├── démarrage rapide
   └── environnement Linux destiné surtout au développement
```

WSL vise principalement le troisième cas.

# 2. WSL 1 et WSL 2 ne sont pas la même architecture

Le nom « WSL » recouvre deux architectures.

## 2.1 WSL 1

WSL 1 traduit les appels système Linux vers des mécanismes Windows.

Il ne fait pas fonctionner un noyau Linux complet.

Schématiquement :

```text
programme Linux
      │
      ▼
appels système Linux
      │
      ▼
couche de traduction WSL 1
      │
      ▼
noyau Windows
```

Avantage historique important : l'accès croisé aux fichiers Windows peut être rapide.

Limites :

- compatibilité noyau incomplète ;
- absence de vrai noyau Linux ;
- incompatibilités avec certains logiciels système ;
- pas de prise en charge normale de `systemd`.

## 2.2 WSL 2

WSL 2 utilise un véritable noyau Linux dans une machine virtuelle utilitaire légère.

```text
Windows
└── VM utilitaire WSL 2
    ├── noyau Linux
    ├── distribution Ubuntu
    ├── distribution Debian
    └── autre distribution
```

La machine virtuelle est administrée automatiquement par WSL.

Nous n'avons donc normalement pas à :

- créer la VM ;
- choisir un disque virtuel à l'installation ;
- gérer un firmware virtuel ;
- installer manuellement le noyau ;
- configurer une carte réseau virtuelle dans Hyper-V Manager.

## 2.3 WSL 2 est aujourd'hui le choix par défaut

Une nouvelle distribution installée avec :

```powershell
wsl --install
```

utilise WSL 2 par défaut.

Microsoft recommande WSL 2 pour la majorité des usages.

## 2.4 Quand WSL 1 peut encore avoir un intérêt

WSL 1 reste pertinent dans quelques cas spécifiques, par exemple lorsqu'un projet doit absolument rester sur le système de fichiers Windows et être accédé intensivement depuis Linux.

Avant de choisir WSL 1 uniquement pour contourner un problème de performances, vérifier d'abord que le dépôt n'est pas simplement mal placé sous `/mnt/c`.

# 3. WSL 2 n'est pas exactement une VM classique

Dire « WSL 2 utilise une VM » est correct, mais insuffisant.

Dans une VM classique :

```text
VM Ubuntu A -> noyau Linux A
VM Debian B -> noyau Linux B
```

Avec WSL 2, plusieurs distributions utilisent l'infrastructure WSL gérée par Windows.

Elles partagent notamment des ressources fournies par la VM utilitaire, tout en conservant leurs propres espaces de processus, montages et utilisateurs.

Cette architecture explique plusieurs comportements :

- `wsl --shutdown` arrête l'ensemble de l'environnement WSL ;
- `wsl --terminate Ubuntu` n'arrête qu'une distribution ;
- une configuration globale `.wslconfig` affecte la VM WSL 2 ;
- `/etc/wsl.conf` configure une distribution individuellement.

# 4. Ce que WSL est — et ce qu'il n'est pas

## 4.1 WSL est excellent pour

- développement Python ;
- Node.js ;
- Rust ;
- Go ;
- C/C++ ;
- Git ;
- SSH ;
- Ansible ;
- outils cloud ;
- CLI Kubernetes ;
- conteneurs ;
- compilation Linux ;
- data science ;
- outils IA ;
- scripts Bash ;
- services de développement locaux.

## 4.2 WSL n'est pas un remplacement universel d'un serveur Linux

Il ne faut pas déduire :

```text
« mon application marche sous WSL »
                ↓
« mon architecture de production Linux est validée »
```

WSL ajoute des particularités :

- couche Windows ;
- réseau spécifique ;
- cycle de vie piloté depuis Windows ;
- stockage VHDX ;
- intégration automatique des chemins et exécutables Windows ;
- WSLg ;
- comportement spécifique pour le matériel.

Pour valider une production Linux, tester également sur une vraie machine Linux, une VM Linux ou un environnement de CI équivalent.

# 5. Prérequis

Les commandes modernes d'installation simplifiée nécessitent :

- Windows 11 ; ou
- Windows 10 version 2004, build 19041 ou ultérieure.

Pour un poste neuf, préférer Windows 11 à jour afin de bénéficier des fonctions WSL les plus récentes, notamment les fonctions réseau modernes.

## 5.1 Vérifier Windows

Dans Windows :

```text
Win + R
```

puis :

```text
winver
```

## 5.2 Vérifier la virtualisation matérielle

WSL 2 repose sur la virtualisation.

Dans le Gestionnaire des tâches :

```text
Performances
└── Processeur
    └── Virtualisation : Activée
```

Si elle est désactivée, vérifier UEFI/BIOS.

Les noms varient selon les fabricants :

- Intel VT-x ;
- Intel Virtualization Technology ;
- AMD-V ;
- SVM Mode.

# 6. Installer WSL de la manière moderne

> [!warning] Ancienne méthode
> Il n'est plus nécessaire, dans le cas général, d'activer manuellement plusieurs fonctionnalités Windows par l'interface « Fonctionnalités facultatives », d'installer un paquet de noyau séparé puis de récupérer Ubuntu dans le Store. `wsl --install` automatise le chemin normal.

Ouvrir **PowerShell en administrateur** puis exécuter :

```powershell
wsl --install
```

Cette commande installe WSL et Ubuntu par défaut.

Redémarrer Windows si cela est demandé.

Au premier lancement, la distribution initialise son système de fichiers puis demande la création de l'utilisateur Linux.

# 7. Installer une distribution précise

Lister les distributions disponibles :

```powershell
wsl --list --online
```

Version courte :

```powershell
wsl -l -o
```

Installer, par exemple, Debian :

```powershell
wsl --install -d Debian
```

Installer sans lancer immédiatement la distribution :

```powershell
wsl --install -d Debian --no-launch
```

Si le Microsoft Store n'est pas utilisable :

```powershell
wsl --install -d Debian --web-download
```

WSL sait également installer une distribution dans un emplacement choisi grâce à l'option `--location` sur les versions récentes.

Toujours vérifier les options effectivement prises en charge par la version installée :

```powershell
wsl --help
```

# 8. Installer WSL sans distribution

Dans certains environnements administrés, nous pouvons vouloir installer le moteur WSL mais gérer les distributions séparément :

```powershell
wsl --install --no-distribution
```

Cela peut être utile pour :

- automatiser un parc ;
- importer une image interne ;
- maîtriser précisément la distribution installée.

# 9. Vérifier l'installation

## 9.1 Version de WSL

```powershell
wsl --version
```

Cette commande affiche les versions des composants WSL.

## 9.2 État général

```powershell
wsl --status
```

## 9.3 Distributions installées

```powershell
wsl --list --verbose
```

ou :

```powershell
wsl -l -v
```

Exemple conceptuel :

```text
  NAME      STATE           VERSION
* Ubuntu    Running         2
  Debian    Stopped         2
```

# 10. Mettre WSL à jour

WSL moderne est distribué et maintenu indépendamment du rythme historique des grandes versions de Windows.

Mettre à jour :

```powershell
wsl --update
```

Si l'accès au Store pose problème :

```powershell
wsl --update --web-download
```

Après certaines mises à jour :

```powershell
wsl --shutdown
```

puis relancer une distribution.

# 11. Définir WSL 2 comme architecture par défaut

```powershell
wsl --set-default-version 2
```

Les nouvelles distributions utiliseront alors WSL 2.

# 12. Convertir une distribution WSL 1 vers WSL 2

Vérifier d'abord :

```powershell
wsl -l -v
```

Puis :

```powershell
wsl --set-version Ubuntu 2
```

Conversion inverse :

```powershell
wsl --set-version Ubuntu 1
```

> [!warning]
> Une conversion peut être longue pour une distribution volumineuse. Faire une sauvegarde avant une conversion importante.

# 13. Choisir la distribution par défaut

```powershell
wsl --set-default Ubuntu
```

ou :

```powershell
wsl -s Ubuntu
```

Ensuite :

```powershell
wsl
```

ouvre cette distribution.

# 14. Démarrer une distribution précise

```powershell
wsl -d Debian
```

Démarrer directement dans le home :

```powershell
wsl ~
```

Exécuter une commande sans ouvrir de shell interactif :

```powershell
wsl -d Ubuntu -- uname -a
```

Depuis Linux, lorsqu'il faut appeler l'exécutable Windows :

```bash
wsl.exe -l -v
```

# 15. Cycle de vie

## 15.1 Arrêter une distribution

```powershell
wsl --terminate Ubuntu
```

## 15.2 Arrêter tout WSL

```powershell
wsl --shutdown
```

Cette commande arrête :

- les distributions en cours ;
- la VM utilitaire WSL 2.

Elle est particulièrement utile après une modification de `.wslconfig`.

## 15.3 Pourquoi fermer le terminal ne suffit pas toujours

Fermer une fenêtre de terminal ne signifie pas forcément que tous les processus Linux s'arrêtent immédiatement.

Un service `systemd`, un serveur ou une autre session peut maintenir la distribution active.

Lister les distributions actives :

```powershell
wsl --list --running
```

# 16. Première configuration Ubuntu

Une distribution WSL reste une distribution Linux.

Sous Ubuntu :

```bash
sudo apt update
sudo apt full-upgrade
```

Installer une base de travail :

```bash
sudo apt install \
    build-essential \
    curl \
    git \
    ca-certificates \
    unzip \
    jq \
    openssh-client
```

Vérifier :

```bash
cat /etc/os-release
uname -a
```

> [!note]
> Le noyau affiché est le noyau WSL fourni par Microsoft, pas un noyau Ubuntu classique installé par le paquet `linux-generic`.

# 17. Les utilisateurs Linux restent distincts des comptes Windows

Le premier démarrage crée un utilisateur Linux.

Par exemple :

```text
Windows : alice@example
Linux   : alice
```

Les deux identités ne sont pas le même compte.

Le mot de passe demandé par `sudo` est le mot de passe **Linux**, pas le mot de passe Windows.

## 17.1 Exécuter en root depuis Windows

```powershell
wsl -d Ubuntu -u root
```

À utiliser pour réparer un compte, pas comme mode normal de travail.

# 18. Activer et comprendre systemd

WSL 2 moderne prend en charge `systemd`.

C'est une évolution majeure par rapport aux premières versions de WSL.

## 18.1 Vérifier PID 1

```bash
ps -p 1 -o comm=
```

Résultat attendu sur une distribution configurée avec systemd :

```text
systemd
```

Autre vérification :

```bash
systemctl status
```

## 18.2 Ubuntu récent installé avec WSL

Pour les versions actuelles d'Ubuntu installées avec `wsl --install`, systemd est normalement configuré par défaut.

## 18.3 Activer systemd sur une autre distribution

Vérifier d'abord la version WSL :

```powershell
wsl --version
```

WSL doit être suffisamment récent.

Dans Linux, éditer :

```bash
sudo nano /etc/wsl.conf
```

Ajouter :

```ini
[boot]
systemd=true
```

Puis côté Windows :

```powershell
wsl --shutdown
```

Relancer la distribution.

## 18.4 Utiliser les services normalement

```bash
systemctl status ssh
```

```bash
sudo systemctl enable --now ssh
```

```bash
journalctl -u ssh
```

Cela rapproche fortement WSL d'un environnement Linux classique.

# 19. `/etc/wsl.conf` : configuration d'une distribution

`/etc/wsl.conf` appartient à **une distribution**.

Exemple :

```ini
[boot]
systemd=true

[user]
default=alice

[automount]
enabled=true
mountFsTab=true
```

Les catégories de réglages incluent notamment :

- démarrage ;
- utilisateur par défaut ;
- montage automatique ;
- génération de `/etc/resolv.conf` ;
- interopérabilité Windows.

Après modification importante :

```powershell
wsl --shutdown
```

# 20. `.wslconfig` : configuration globale WSL 2

Ne pas confondre :

```text
/etc/wsl.conf
    └── une distribution

%UserProfile%\.wslconfig
    └── VM WSL 2 globale
```

Sous Windows, le fichier se trouve typiquement ici :

```text
C:\Users\alice\.wslconfig
```

Sur les versions récentes, Microsoft propose également une interface graphique **WSL Settings** ; elle est à privilégier lorsque disponible pour les réglages qu'elle expose.

# 21. Limiter CPU, RAM et swap

Exemple pédagogique :

```ini
[wsl2]
memory=8GB
processors=4
swap=4GB
```

Puis :

```powershell
wsl --shutdown
```

> [!warning]
> Ces valeurs ne constituent pas une recommandation universelle. Une machine de 16 Gio et une station de 128 Gio n'ont évidemment pas les mêmes besoins.

## 21.1 Valeurs par défaut

La documentation WSL actuelle indique notamment :

- mémoire : jusqu'à environ 50 % de la RAM Windows par défaut ;
- processeurs : tous les processeurs logiques disponibles ;
- swap : valeur dérivée de la mémoire Windows.

Il est donc inutile de fixer des limites uniquement « parce qu'il faut configurer WSL ».

Limiter surtout si :

- des workloads Linux monopolisent la machine ;
- Docker consomme trop de mémoire ;
- plusieurs IDE lourds tournent simultanément ;
- le poste est partagé avec des traitements Windows exigeants.

# 22. Où stocker les projets

C'est l'un des points les plus importants pour les performances.

## 22.1 Projet utilisé principalement avec des outils Linux

Préférer :

```text
/home/alice/projects/myapp
```

par exemple :

```bash
mkdir -p ~/projects
cd ~/projects
git clone https://example.com/project.git
```

## 22.2 Éviter pour les builds Linux intensifs

```text
/mnt/c/Users/alice/project
```

L'accès croisé au système de fichiers Windows ajoute une surcharge.

Les opérations particulièrement sensibles sont :

- `git status` sur de gros dépôts ;
- `npm install` ;
- `node_modules` ;
- compilation C/C++ ;
- environnements Python riches en petits fichiers ;
- Rust/Cargo ;
- bases de données de développement.

## 22.3 Règle pratique

```text
outils Linux majoritaires
    -> fichiers dans Linux

outils Windows majoritaires
    -> fichiers dans Windows
```

# 23. Accéder aux fichiers Windows depuis Linux

Le disque `C:` est monté par défaut sous :

```text
/mnt/c
```

Exemple :

```bash
ls /mnt/c/Users
```

Le disque `D:` devient généralement :

```text
/mnt/d
```

# 24. Accéder aux fichiers Linux depuis Windows

Depuis l'Explorateur :

```text
\\wsl$\Ubuntu\home\alice
```

ou :

```text
\\wsl.localhost\Ubuntu\home\alice
```

Depuis Linux, ouvrir le dossier courant dans l'Explorateur Windows :

```bash
explorer.exe .
```

# 25. Convertir les chemins

WSL fournit `wslpath`.

Windows vers Linux :

```bash
wslpath 'C:\Users\alice\Documents'
```

Linux vers Windows :

```bash
wslpath -w /home/alice/project
```

Format avec `/` :

```bash
wslpath -m /mnt/c/Users/alice/project
```

# 26. Interopérabilité : lancer Windows depuis Linux

Depuis Bash :

```bash
notepad.exe notes.txt
```

```bash
powershell.exe -Command 'Get-Date'
```

```bash
cmd.exe /c ver
```

Le suffixe `.exe` permet d'appeler directement des exécutables Windows accessibles par le `PATH` intégré.

# 27. Interopérabilité : lancer Linux depuis Windows

Depuis PowerShell :

```powershell
wsl ls -la
```

```powershell
wsl -d Ubuntu -- bash -lc 'uname -a && id'
```

Cette capacité est très utile pour :

- scripts hybrides ;
- tâches de build ;
- automatisation locale ;
- lancement d'outils Linux depuis un IDE Windows.

# 28. Attention au mélange des environnements

Le fait que Windows et Linux puissent s'appeler mutuellement ne signifie pas qu'ils doivent partager tous leurs runtimes.

Éviter par exemple :

```text
Python Windows
    + venv Linux
```

ou :

```text
Node Windows
    + node_modules Linux
```

Choisir un environnement cohérent par projet.

# 29. Git sous WSL

Pour un projet Linux :

```bash
sudo apt install git
```

Configurer l'identité :

```bash
git config --global user.name "Alice Doe"
git config --global user.email "alice@example.com"
```

Conserver le dépôt dans :

```text
~/projects
```

plutôt que `/mnt/c/...` si Git est exécuté depuis Linux.

## 29.1 Fins de lignes

Git peut devenir délicat lorsqu'un même dépôt est manipulé tantôt avec Git Windows, tantôt avec Git Linux.

Éviter de multiplier les configurations contradictoires `core.autocrlf`.

Pour les projets modernes, préférer un `.gitattributes` explicite, par exemple :

```gitattributes
* text=auto eol=lf
*.bat text eol=crlf
*.cmd text eol=crlf
```

# 30. VS Code et développement distant

Une approche très efficace consiste à garder :

```text
VS Code : interface Windows
Projet  : système de fichiers Linux
Outils  : Linux WSL
```

L'extension WSL de VS Code permet d'ouvrir le projet « dans » la distribution.

Depuis WSL, dans un projet :

```bash
code .
```

L'interface reste intégrée à Windows mais :

- le terminal tourne sous Linux ;
- les extensions serveur adaptées tournent côté WSL ;
- Python/Git/Node peuvent être ceux de Linux ;
- le projet reste dans le système de fichiers Linux.

Voir [[Visual studio code]].

# 31. Applications graphiques Linux : WSLg

> [!important] Correction de l'ancienne procédure
> Sur WSL 2 moderne, il n'est normalement **plus nécessaire d'installer un serveur X Windows, XFCE et VNC** uniquement pour lancer une application graphique Linux. WSLg intègre directement les applications Linux X11 et Wayland dans le bureau Windows.

Prérequis documentés par Microsoft :

- Windows 10 build 19044 ou plus récent, ou Windows 11 ;
- WSL 2 ;
- pilotes graphiques adaptés pour l'accélération GPU.

Mettre WSL à jour :

```powershell
wsl --update
wsl --shutdown
```

## 31.1 Exemple avec une application X11

Sous Ubuntu :

```bash
sudo apt update
sudo apt install x11-apps
```

Puis :

```bash
xclock
```

La fenêtre apparaît sur le bureau Windows.

## 31.2 Exemple avec GIMP

```bash
sudo apt install gimp
```

Puis :

```bash
gimp
```

## 31.3 Ce que WSLg fournit

Les applications peuvent notamment :

- apparaître comme des fenêtres Windows ;
- être épinglées à la barre des tâches ;
- participer à Alt+Tab ;
- utiliser le presse-papiers partagé ;
- utiliser X11 ou Wayland via l'infrastructure WSLg.

## 31.4 Ce que WSLg ne cherche pas à fournir

WSLg n'est pas destiné à transformer WSL en session de bureau Linux complète.

Pour apprendre ou tester GNOME/KDE en environnement complet, une VM traditionnelle ou une machine Linux est souvent plus adaptée.

# 32. Réseau : le modèle par défaut NAT

WSL 2 utilise historiquement un réseau NAT.

Schéma simplifié :

```text
Internet / LAN
      │
      ▼
Windows
      │
      ▼
NAT WSL
      │
      ▼
Linux
```

Dans ce mode, Windows et WSL peuvent avoir des adresses IP différentes.

Adresse de WSL vue depuis Windows :

```powershell
wsl hostname -I
```

Adresse de Windows vue depuis Linux :

```bash
ip route show | grep -i default
```

# 33. Accéder depuis Windows à un serveur lancé sous WSL

Exemple :

```bash
python3 -m http.server 8000
```

Dans les configurations usuelles, Windows peut accéder au service via :

```text
http://localhost:8000
```

La redirection localhost est activée par défaut en mode NAT dans WSL 2.

# 34. Ne pas confondre écoute locale et écoute réseau

Un serveur lié à :

```text
127.0.0.1
```

n'accepte normalement que les connexions locales dans son contexte réseau.

Un serveur lié à :

```text
0.0.0.0
```

écoute sur toutes les interfaces IPv4 disponibles.

Cela ne signifie pas pour autant qu'il est automatiquement accessible depuis tout le LAN :

- le mode réseau WSL compte ;
- le pare-feu Windows compte ;
- le pare-feu Hyper-V peut compter ;
- les règles de l'application comptent.

# 35. Mode réseau mirrored

Sur Windows 11 22H2 et versions ultérieures, WSL propose :

```ini
[wsl2]
networkingMode=mirrored
```

Microsoft recommande d'essayer ce mode pour bénéficier des fonctions réseau modernes.

Le principe consiste à mieux refléter les interfaces réseau Windows dans Linux.

Bénéfices documentés :

- IPv6 ;
- meilleure compatibilité VPN ;
- multicast ;
- accès plus naturel à Windows depuis Linux via `localhost` ;
- scénarios LAN améliorés.

Après modification :

```powershell
wsl --shutdown
```

# 36. Pare-feu WSL

Sur les versions Windows 11/WSL récentes, les règles de pare-feu Windows et Hyper-V participent au filtrage du trafic WSL.

Le réglage global est notamment :

```ini
[wsl2]
firewall=true
```

Il est activé par défaut dans les configurations modernes prises en charge.

> [!warning]
> Passer un service de développement de `localhost` à une exposition LAN change son modèle de menace. Ne pas désactiver le pare-feu globalement pour « faire marcher » un port.

Préférer une règle ciblée.

# 37. DNS tunneling

WSL moderne propose :

```ini
[wsl2]
dnsTunneling=true
```

Sur Windows 11 22H2 et ultérieur, cette fonction est activée par défaut dans les versions actuelles prises en charge.

Elle vise notamment à améliorer la compatibilité avec :

- VPN ;
- politiques DNS complexes ;
- configurations d'entreprise.

Lorsque DNS fonctionne mal sous WSL, ne pas commencer par écraser arbitrairement `/etc/resolv.conf` avec `8.8.8.8`.

Diagnostiquer d'abord :

```bash
cat /etc/resolv.conf
```

```bash
getent hosts example.com
```

```bash
resolvectl status
```

si `systemd-resolved` est utilisé.

# 38. Proxy automatique

WSL peut reprendre le proxy HTTP de Windows :

```ini
[wsl2]
autoProxy=true
```

C'est particulièrement utile en entreprise.

Vérifier ensuite dans Linux :

```bash
env | grep -i proxy
```

# 39. Exemple raisonnable de `.wslconfig`

Pour un poste Windows 11 récent :

```ini
[wsl2]
memory=8GB
processors=4
swap=4GB
networkingMode=mirrored
firewall=true
dnsTunneling=true
autoProxy=true
```

> [!warning]
> Cet exemple montre la syntaxe. Les valeurs CPU/RAM doivent être adaptées à la machine et au workload.

Ne pas recopier mécaniquement une configuration trouvée sur Internet.

# 40. Réseau : stratégie de diagnostic

Quand « Internet ne marche plus dans WSL » :

## Étape 1 — tester l'interface

```bash
ip addr
```

## Étape 2 — tester le routage

```bash
ip route
```

## Étape 3 — tester une IP

```bash
ping -c 2 1.1.1.1
```

## Étape 4 — tester le DNS

```bash
getent hosts example.com
```

## Étape 5 — vérifier le proxy

```bash
env | grep -i proxy
```

## Étape 6 — comparer VPN actif/inactif

Certains clients VPN modifient fortement routes, DNS et filtrage.

## Étape 7 — redémarrer proprement WSL

```powershell
wsl --shutdown
```

## Étape 8 — vérifier la configuration

```powershell
wsl --version
wsl --status
```

Puis inspecter :

```text
%UserProfile%\.wslconfig
```

# 41. Monter des disques dans WSL 2

WSL peut attacher un disque physique à Linux.

Lister les disques Windows :

```powershell
Get-CimInstance -Query "SELECT * from Win32_DiskDrive"
```

Puis, selon le disque :

```powershell
wsl --mount \\.\PHYSICALDRIVE1
```

Options possibles :

```powershell
wsl --mount \\.\PHYSICALDRIVE1 --bare
```

ou avec un type de système de fichiers adapté.

Démonter :

```powershell
wsl --unmount \\.\PHYSICALDRIVE1
```

> [!danger]
> Manipuler des disques physiques peut entraîner une perte de données. Identifier sans ambiguïté le disque et les partitions avant toute opération.

# 42. Le système de fichiers WSL 2 est stocké dans un VHDX

Chaque distribution WSL 2 utilise un disque virtuel, généralement basé sur `ext4` et représenté côté Windows par un fichier `.vhdx`.

Conceptuellement :

```text
Windows NTFS
└── ext4.vhdx
    └── système de fichiers Linux ext4
        ├── /etc
        ├── /home
        ├── /usr
        └── /var
```

Le fichier VHDX grandit selon les besoins jusqu'à une limite maximale.

La documentation actuelle indique une taille maximale par défaut de **1 To** pour les nouvelles distributions WSL 2.

Cela ne veut pas dire que 1 To est immédiatement occupé sur le disque Windows.

# 43. Vérifier l'espace disponible

Dans Linux :

```bash
df -h /
```

Inodes :

```bash
df -i /
```

Répertoires volumineux :

```bash
sudo du -xhd1 / 2>/dev/null | sort -h
```

Voir [[Watchdog espace disque]].

# 44. Agrandir le VHD WSL 2

Sur WSL récent, Microsoft documente :

```powershell
wsl --shutdown
```

puis :

```powershell
wsl --manage Ubuntu --resize 2TB
```

La commande `--manage --resize` nécessite une version WSL récente.

Toujours vérifier :

```powershell
wsl --help
```

et :

```powershell
wsl --version
```

avant d'utiliser une commande apparue récemment.

# 45. Pourquoi supprimer des fichiers Linux ne réduit pas toujours immédiatement le VHDX Windows

Un disque virtuel dynamique peut grandir lorsque Linux écrit des données, sans nécessairement rendre immédiatement toute la place au fichier hôte après suppression.

Il faut distinguer :

```text
espace libre dans ext4
        ≠
taille du fichier VHDX sur NTFS
```

Avant toute opération de compactage :

1. sauvegarder ;
2. arrêter WSL ;
3. utiliser une méthode documentée pour la version installée ;
4. éviter les scripts anciens manipulant directement le VHDX sans compréhension.

# 46. Sauvegarder une distribution

C'est une compétence essentielle.

## 46.1 Export TAR

```powershell
wsl --export Ubuntu ubuntu-backup.tar
```

Vérifier le fichier produit.

## 46.2 Export VHDX

Pour WSL 2 :

```powershell
wsl --export Ubuntu ubuntu-backup.vhdx --vhd
```

## 46.3 Pourquoi l'export est préférable à la copie sauvage du VHDX

Copier directement un VHDX pendant que la distribution est active peut capturer un état incohérent.

Préférer `wsl --export`.

# 47. Importer une distribution

Exemple :

```powershell
wsl --import Ubuntu-Lab D:\WSL\Ubuntu-Lab .\ubuntu-backup.tar --version 2
```

Puis :

```powershell
wsl -l -v
```

Démarrer :

```powershell
wsl -d Ubuntu-Lab
```

# 48. Importer un VHDX sur place

Pour un VHDX ext4 adapté :

```powershell
wsl --import-in-place Ubuntu-Lab D:\WSL\Ubuntu-Lab\ext4.vhdx
```

Cette méthode est utile dans certains scénarios de migration, mais elle ne remplace pas une stratégie de sauvegarde.

# 49. `wsl --unregister` est destructif

```powershell
wsl --unregister Ubuntu
```

supprime l'enregistrement de la distribution **et ses données associées**.

> [!danger]
> Ne jamais utiliser `wsl --unregister` comme une simple commande « reset » sans sauvegarde. Vérifier deux fois le nom de la distribution.

Avant :

```powershell
wsl -l -v
wsl --export Ubuntu ubuntu-before-unregister.tar
```

# 50. Sauvegarder les projets indépendamment de la distribution

L'export WSL est utile, mais le code doit aussi être sauvegardé normalement :

- Git ;
- remote Git ;
- sauvegarde des données ;
- gestion externe des secrets ;
- fichiers de configuration reproductibles.

Une distribution WSL ne doit pas être le seul endroit où existe un projet important.

# 51. Docker sous WSL 2

WSL 2 fournit la compatibilité noyau nécessaire aux conteneurs Linux.

Deux grandes stratégies existent.

## 51.1 Docker Desktop avec intégration WSL

Architecture conceptuelle :

```text
Windows
├── Docker Desktop
└── WSL 2
    └── distribution de développement
```

Avantages :

- intégration Windows ;
- expérience utilisateur simple ;
- Kubernetes et outils Desktop selon configuration ;
- accès Docker depuis plusieurs distributions.

## 51.2 Docker Engine directement dans une distribution WSL

Avec systemd, il est aussi possible d'exploiter un moteur Docker comme dans un Linux classique.

Dans ce cas :

```bash
systemctl status docker
```

est possible si Docker a été installé avec la méthode officielle adaptée à la distribution.

> [!important]
> Ne pas installer plusieurs moteurs Docker sans savoir lequel fournit `/var/run/docker.sock` et quel contexte `docker` est utilisé.

Voir [[Docker]].

# 52. Où placer les volumes et les sources Docker

Pour de bonnes performances Linux :

```text
~/projects/myapp
```

est généralement préférable à :

```text
/mnt/c/Users/alice/myapp
```

pour :

- bind mounts intensifs ;
- bases de données ;
- compilation ;
- `node_modules`.

# 53. GPU et calcul

WSL 2 peut exposer des capacités GPU au monde Linux avec des pilotes Windows compatibles.

Des scénarios courants incluent :

- CUDA avec GPU NVIDIA ;
- frameworks de machine learning ;
- rendu accéléré WSLg ;
- conteneurs GPU.

Vérifier côté Linux :

```bash
ls -l /dev/dxg
```

Pour NVIDIA, selon le pilote :

```bash
nvidia-smi
```

> [!warning]
> Suivre la documentation GPU actuelle du constructeur. Ne pas installer arbitrairement un pilote Linux complet dans WSL alors que la couche GPU dépend d'abord du pilote Windows compatible WSL.

# 54. USB

WSL 2 ne donne pas automatiquement à Linux un accès natif à tous les périphériques USB.

Microsoft documente l'usage du projet open source `usbipd-win`.

Installation possible côté Windows :

```powershell
winget install --interactive --exact dorssel.usbipd-win
```

Lister :

```powershell
usbipd list
```

Partager un périphérique — en administrateur :

```powershell
usbipd bind --busid 4-4
```

Attacher :

```powershell
usbipd attach --wsl --busid 4-4
```

Dans Linux :

```bash
lsusb
```

Détacher :

```powershell
usbipd detach --busid 4-4
```

Applications typiques :

- Arduino ;
- microcontrôleurs ;
- lecteurs de cartes ;
- outils embarqués.

# 55. Sécurité : WSL n'est pas une frontière magique

WSL exécute du code Linux sur le poste de travail de l'utilisateur.

Un programme malveillant lancé dans WSL peut potentiellement :

- lire les fichiers accessibles sous `/mnt/c` avec les droits de l'utilisateur ;
- invoquer certains programmes Windows via l'interopérabilité ;
- utiliser le réseau ;
- accéder aux secrets présents dans le home Linux ;
- compromettre les projets de développement.

Ne pas considérer WSL comme une sandbox de sécurité pour exécuter du code non fiable.

# 56. Réduire l'exposition croisée Windows/Linux

La forte interopérabilité est utile, mais elle augmente aussi les interactions entre environnements.

Bonnes pratiques :

- ne pas stocker des secrets en clair ;
- protéger les clés SSH ;
- limiter les ports exposés ;
- maintenir Windows et WSL à jour ;
- maintenir la distribution à jour ;
- éviter d'exécuter des scripts Internet directement avec `curl | sh` sans vérification ;
- contrôler les extensions IDE ;
- auditer les conteneurs et dépendances.

Voir [[Sécurité avancée sous Linux]].

# 57. SSH dans WSL

Pour utiliser le client :

```bash
ssh user@example.com
```

Les clés Linux résident typiquement dans :

```text
~/.ssh/
```

Permissions :

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
```

## 57.1 Serveur SSH dans WSL

Installer :

```bash
sudo apt install openssh-server
```

Avec systemd :

```bash
sudo systemctl enable --now ssh
```

Vérifier :

```bash
ss -ltnp | grep ':22'
```

L'accessibilité depuis Windows ou le LAN dépend ensuite du mode réseau et des pare-feu.

# 58. Ne pas réutiliser aveuglément les clés Windows

Il est possible de partager certains secrets entre Windows et Linux, mais ce n'est pas toujours souhaitable.

Choisir explicitement :

```text
une clé par environnement
```

ou :

```text
agent / gestionnaire de secrets commun maîtrisé
```

Éviter :

```text
copie permanente de clés privées dans plusieurs emplacements
```

# 59. Horloge et reprise de veille

Les machines virtuelles, les longues veilles et certains anciens builds WSL ont parfois provoqué des décalages d'horloge.

Diagnostic :

```bash
date --iso-8601=seconds
```

Comparer avec Windows.

Si l'environnement est dans un état étrange :

```powershell
wsl --shutdown
```

puis relancer.

Avec systemd :

```bash
timedatectl
```

# 60. Processus et consommation mémoire

Depuis Linux :

```bash
free -h
```

```bash
top
```

```bash
ps aux --sort=-%mem | head
```

Depuis Windows, la VM WSL peut apparaître via les composants de virtualisation WSL.

Ne pas conclure uniquement à partir de la mémoire apparente d'un processus hôte : inspecter aussi le workload Linux.

# 61. Récupération automatique de mémoire

Les versions modernes de WSL disposent de mécanismes de récupération de mémoire améliorés.

La configuration expérimentale documentée inclut notamment :

```ini
[experimental]
autoMemoryReclaim=dropCache
```

Les valeurs et leur statut peuvent évoluer.

Avant de modifier :

```powershell
wsl --version
```

puis consulter la documentation correspondant à cette version.

> [!note]
> Le fait qu'une option soit dans `[experimental]` est une indication importante : ne pas construire une procédure d'exploitation critique en supposant qu'elle restera inchangée.

# 62. VHD sparse

WSL documente également :

```ini
[experimental]
sparseVhd=true
```

Cette option concerne les nouveaux VHD.

Là encore, vérifier le statut dans la version installée avant de l'utiliser dans une procédure pérenne.

# 63. Personnaliser le montage des lecteurs Windows

Dans `/etc/wsl.conf` :

```ini
[automount]
enabled=true
root=/mnt/
mountFsTab=true
```

Puis :

```powershell
wsl --shutdown
```

Les lecteurs Windows sont alors accessibles suivant cette politique.

# 64. `/etc/fstab` sous WSL

Si :

```ini
[automount]
mountFsTab=true
```

WSL traite `/etc/fstab` au démarrage.

Cela permet par exemple de déclarer certains systèmes de fichiers ou partages.

Tester une modification avant redémarrage :

```bash
sudo mount -a
```

> [!warning]
> Une entrée `fstab` invalide peut dégrader le démarrage ou les montages. Toujours tester.

# 65. Comprendre les permissions sur `/mnt/c`

Les fichiers Windows montés via DrvFs n'ont pas exactement la même sémantique que de vrais fichiers ext4.

Ne pas supposer que :

```bash
chmod 600 secret
```

sur un fichier Windows produit exactement le même modèle de sécurité qu'un fichier natif Linux sous ext4.

Pour des fichiers sensibles utilisés exclusivement par Linux :

```text
~/.ssh
~/.config
~/.local
```

préférer le système de fichiers Linux.

# 66. Cas d'un environnement Python

Créer le projet dans Linux :

```bash
mkdir -p ~/projects/demo
cd ~/projects/demo
```

Installer Python :

```bash
sudo apt install python3 python3-venv
```

Créer un venv :

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Vérifier :

```bash
which python
python --version
```

Attendu : un chemin Linux sous le projet.

Éviter de créer le `venv` sous Windows puis de l'utiliser depuis Linux.

# 67. Cas d'un environnement Node.js

Même principe :

```text
code + node_modules + runtime
```

restent de préférence du même côté.

Si Node tourne sous WSL :

```text
~/projects/app
```

plutôt que :

```text
/mnt/c/Users/alice/app
```

# 68. Cas d'une base de données de développement

PostgreSQL, MariaDB ou Redis peuvent fonctionner sous WSL 2.

Avec systemd :

```bash
sudo systemctl status postgresql
```

Cependant :

- WSL n'est pas une stratégie de production ;
- sauvegarder les données ;
- connaître le cycle de vie de la distribution ;
- éviter de placer les données de base sur un montage Windows pour gagner artificiellement en partage de fichiers.

# 69. Conteneur, WSL ou VM : comment choisir

## WSL 2

Choisir pour :

- poste de développement Windows ;
- intégration Linux/Windows ;
- CLI Linux ;
- compilation ;
- Docker ;
- IDE Windows + outils Linux.

## Conteneur

Choisir pour :

- isoler une application ;
- reproduire ses dépendances ;
- CI/CD ;
- environnement jetable.

## VM complète

Choisir pour :

- noyau Linux indépendant ;
- tests réseau complexes ;
- appliance ;
- environnement fortement isolé ;
- simulation serveur proche de la production ;
- bureau Linux complet.

# 70. WSL n'est pas un conteneur Docker

Cette confusion est fréquente.

```text
WSL 2
    -> environnement Linux intégré à Windows

Docker container
    -> isolation d'un processus/application via namespaces/cgroups
```

Docker peut fonctionner **dans ou avec** WSL 2.

Les deux niveaux sont différents.

# 71. WSL et les namespaces Linux

WSL 2 exécute un vrai noyau Linux et peut donc utiliser les primitives du noyau :

- namespaces ;
- cgroups ;
- seccomp selon contexte ;
- overlayfs ;
- fonctionnalités nécessaires aux conteneurs.

Voir [[Les namespaces Linux]].

# 72. Diagnostic général : cinq couches

Quand WSL ne fonctionne pas, raisonner en couches.

```text
1. Windows
2. moteur WSL
3. distribution
4. Linux / systemd
5. application
```

Exemple : un serveur Web inaccessible peut être dû à :

```text
application non lancée
        │
        ├── service systemd arrêté
        ├── port mal lié
        ├── IP/réseau WSL
        ├── pare-feu Hyper-V/Windows
        └── VPN
```

# 73. Diagnostic côté Windows

```powershell
wsl --status
```

```powershell
wsl --version
```

```powershell
wsl -l -v
```

```powershell
wsl --list --running
```

Puis éventuellement :

```powershell
wsl --shutdown
```

# 74. Diagnostic côté Linux

Système :

```bash
uname -a
```

Distribution :

```bash
cat /etc/os-release
```

PID 1 :

```bash
ps -p 1 -o pid,comm,args
```

Services :

```bash
systemctl --failed
```

Journal :

```bash
journalctl -b -p warning
```

Réseau :

```bash
ip addr
ip route
ss -ltnup
```

DNS :

```bash
getent hosts example.com
```

Disque :

```bash
df -h
df -i
```

Mémoire :

```bash
free -h
```

# 75. Problème : `wsl --version` n'existe pas

Une très vieille installation WSL peut afficher une erreur du type option inconnue.

Mettre à jour WSL :

```powershell
wsl --update
```

Si nécessaire, utiliser le chemin de mise à jour Microsoft Store ou `--web-download` selon le contexte.

# 76. Problème : une distribution est encore en WSL 1

```powershell
wsl -l -v
```

Si `VERSION` vaut `1` :

```powershell
wsl --set-version Ubuntu 2
```

Sauvegarder auparavant pour une distribution importante.

# 77. Problème : `systemctl` indique que systemd n'est pas PID 1

Vérifier :

```bash
ps -p 1 -o comm=
```

Puis `/etc/wsl.conf` :

```ini
[boot]
systemd=true
```

Et :

```powershell
wsl --shutdown
```

Relancer.

# 78. Problème : une application graphique ne s'ouvre pas

Vérifier d'abord :

```powershell
wsl --update
wsl --shutdown
```

Puis que la distribution est WSL 2 :

```powershell
wsl -l -v
```

Sous Linux :

```bash
printf 'DISPLAY=%s\n' "$DISPLAY"
printf 'WAYLAND_DISPLAY=%s\n' "$WAYLAND_DISPLAY"
```

Ne pas réinstaller immédiatement Xorg/VNC comme première solution.

# 79. Problème : Git ou npm est très lent

Vérifier :

```bash
pwd
```

Si le projet est ici :

```text
/mnt/c/Users/...
```

et que les outils sont Linux, déplacer ou recloner le dépôt vers :

```text
~/projects/...
```

C'est souvent la correction la plus importante.

# 80. Problème : plus d'espace disque

```bash
df -h /
```

Puis :

```bash
sudo du -xhd1 / 2>/dev/null | sort -h
```

et :

```bash
df -i /
```

Si le système de fichiers atteint réellement sa taille maximale VHD :

```powershell
wsl --shutdown
wsl --manage Ubuntu --resize 2TB
```

sur une version WSL prenant cette commande en charge.

# 81. Problème : VPN et DNS

Comparer :

```bash
ip route
cat /etc/resolv.conf
getent hosts example.com
```

avec et sans VPN.

Sur Windows 11 récent, tester le réseau `mirrored` et conserver `dnsTunneling=true` est souvent plus pertinent que d'appliquer de vieilles recettes basées sur un `resolv.conf` statique.

Consulter toutefois les problèmes connus du client VPN utilisé.

# 82. Problème : port accessible depuis Windows mais pas depuis le LAN

Vérifier successivement :

1. application liée à `0.0.0.0` ou interface voulue ;
2. port réellement ouvert avec `ss` ;
3. mode NAT ou mirrored ;
4. pare-feu Windows ;
5. pare-feu Hyper-V ;
6. politique réseau d'entreprise ;
7. route du client LAN.

Ne pas créer un portproxy permanent avant d'avoir identifié le modèle réseau utilisé.

# 83. Anciennes recettes à éviter

## 83.1 Installer Xorg + VNC juste pour lancer une application GUI

Ancienne recette :

```text
xorg + xfce + tigervnc
```

Aujourd'hui :

```text
WSLg
```

pour le cas normal d'application graphique Linux intégrée au bureau Windows.

## 83.2 Modifier manuellement les fonctionnalités Windows pour toute installation neuve

Ancien parcours long.

Aujourd'hui :

```powershell
wsl --install
```

est le point de départ normal.

## 83.3 Déclarer que Windows Home ne permet pas les partages WSL

Cette affirmation est obsolète et mélangeait plusieurs mécanismes Windows.

WSL moderne fonctionne sur les éditions prises en charge, y compris Windows Home, sous réserve des prérequis de version et de virtualisation.

## 83.4 Traiter WSL 2 comme une VM à IP fixe

En mode NAT, l'adresse peut changer ; en mode mirrored, le modèle est différent.

Éviter d'inscrire une IP WSL éphémère en dur dans des configurations.

## 83.5 Travailler systématiquement sous `/mnt/c`

Cette pratique est commode mais peut dégrader fortement les performances des workloads Linux.

# 84. Configuration recommandée pour un poste de développement

Approche type :

```text
Windows 11
├── WSL à jour
├── Windows Terminal
├── VS Code
└── Ubuntu WSL 2
    ├── systemd
    ├── ~/projects
    ├── Git Linux
    ├── Python/Node/etc. Linux
    └── Docker selon stratégie choisie
```

Réseau :

```text
mirrored
```

si compatible avec le contexte et le VPN utilisé.

Sauvegardes :

```text
Git remote + wsl --export pour l'environnement
```

# 85. Exemple complet d'installation propre

## 85.1 PowerShell administrateur

```powershell
wsl --install -d Ubuntu
```

Redémarrer si demandé.

## 85.2 Premier lancement Ubuntu

Créer l'utilisateur.

Puis :

```bash
sudo apt update
sudo apt full-upgrade
sudo apt install build-essential git curl ca-certificates
```

## 85.3 Vérifier systemd

```bash
ps -p 1 -o comm=
```

## 85.4 Créer les projets dans Linux

```bash
mkdir -p ~/projects
cd ~/projects
```

## 85.5 Depuis Windows

```powershell
wsl --version
wsl -l -v
```

## 85.6 Sauvegarde initiale optionnelle

Après configuration :

```powershell
wsl --shutdown
wsl --export Ubuntu ubuntu-clean.tar
```

# 86. TP 1 — Installer et auditer WSL 2

## Objectif

Installer une distribution et produire une fiche d'audit.

## Travail demandé

1. afficher la version de Windows ;
2. afficher la version WSL ;
3. lister les distributions ;
4. confirmer WSL 2 ;
5. identifier la distribution Linux ;
6. identifier le noyau ;
7. identifier PID 1 ;
8. mesurer RAM et disque ;
9. relever l'adresse IP ;
10. confirmer la résolution DNS.

Commandes possibles :

```powershell
wsl --version
wsl --status
wsl -l -v
```

Puis :

```bash
cat /etc/os-release
uname -a
ps -p 1 -o comm=
free -h
df -h /
ip addr
getent hosts example.com
```

## Livrable

Un tableau :

| Élément | Valeur | Interprétation |
|---|---|---|
| Windows | ... | ... |
| WSL | ... | ... |
| Distribution | ... | ... |
| Kernel | ... | ... |
| PID 1 | ... | ... |
| Stockage | ... | ... |
| Réseau | ... | ... |

# 87. TP 2 — Mesurer l'impact de `/mnt/c`

## Objectif

Comprendre le coût des accès croisés de système de fichiers.

Créer deux répertoires :

```text
~/bench-wsl
/mnt/c/Users/<user>/bench-win
```

Créer le même petit projet dans les deux emplacements.

Mesurer par exemple :

```bash
time git status
```

sur un dépôt significatif, ou :

```bash
time find . -type f | wc -l
```

avec de nombreux fichiers.

Ne pas chercher un « benchmark universel » : constater le comportement de la machine réelle.

## Question

Pourquoi un workload constitué de milliers de petits fichiers amplifie-t-il le coût de l'accès inter-système ?

# 88. TP 3 — Service systemd

## Objectif

Vérifier que WSL peut exécuter un service Linux moderne.

Créer :

```text
/etc/systemd/system/hello-wsl.service
```

Contenu :

```ini
[Unit]
Description=Exemple de service WSL

[Service]
Type=oneshot
ExecStart=/usr/bin/logger "hello from WSL systemd"

[Install]
WantedBy=multi-user.target
```

Puis :

```bash
sudo systemctl daemon-reload
sudo systemctl start hello-wsl.service
journalctl -u hello-wsl.service --no-pager
```

Vérifier le code et les journaux.

# 89. TP 4 — Serveur réseau local

Sous Linux :

```bash
mkdir -p ~/projects/http-test
cd ~/projects/http-test
printf 'WSL works\n' > index.html
python3 -m http.server 8000 --bind 0.0.0.0
```

Depuis Windows :

```text
http://localhost:8000
```

Puis analyser :

```bash
ss -ltnp | grep 8000
```

Questions :

1. quelle interface écoute ?
2. quel processus possède le port ?
3. le service est-il accessible depuis le LAN ?
4. quelle règle de pare-feu s'applique ?
5. quel mode réseau WSL est actif ?

# 90. TP 5 — Sauvegarde et clone d'une distribution

Exporter :

```powershell
wsl --export Ubuntu ubuntu-lab.tar
```

Importer sous un nouveau nom :

```powershell
wsl --import Ubuntu-Clone D:\WSL\Ubuntu-Clone .\ubuntu-lab.tar --version 2
```

Lister :

```powershell
wsl -l -v
```

Démarrer :

```powershell
wsl -d Ubuntu-Clone
```

Comparer :

```bash
id
hostname
cat /etc/os-release
```

Après le TP, exporter ce qui doit être conservé puis supprimer uniquement la distribution clone :

```powershell
wsl --unregister Ubuntu-Clone
```

> [!danger]
> Vérifier impérativement le nom avant `--unregister`.

# 91. TP 6 — WSLg

Sous Ubuntu :

```bash
sudo apt update
sudo apt install x11-apps
```

Lancer :

```bash
xclock
```

Observer :

- fenêtre intégrée au bureau Windows ;
- absence de serveur VNC à configurer ;
- absence de bureau XFCE complet nécessaire ;
- variables d'environnement graphiques.

Inspecter :

```bash
env | grep -E 'DISPLAY|WAYLAND|PULSE'
```

# 92. Tableau de commandes essentielles

| Besoin | Commande Windows |
|---|---|
| Installer WSL | `wsl --install` |
| Lister les distributions disponibles | `wsl -l -o` |
| Installer Debian | `wsl --install -d Debian` |
| Lister les distributions installées | `wsl -l -v` |
| Mettre WSL à jour | `wsl --update` |
| Afficher la version | `wsl --version` |
| Afficher l'état | `wsl --status` |
| Lancer Ubuntu | `wsl -d Ubuntu` |
| Exécuter une commande | `wsl -d Ubuntu -- uname -a` |
| Terminer Ubuntu | `wsl --terminate Ubuntu` |
| Arrêter tout WSL | `wsl --shutdown` |
| Exporter | `wsl --export Ubuntu backup.tar` |
| Importer | `wsl --import Nom D:\WSL\Nom backup.tar --version 2` |
| Supprimer une distribution | `wsl --unregister Nom` |

# 93. Tableau des fichiers de configuration

| Fichier | Côté | Portée |
|---|---|---|
| `%UserProfile%\.wslconfig` | Windows | toutes les distributions WSL 2 |
| `/etc/wsl.conf` | Linux | distribution courante |
| `/etc/fstab` | Linux | montages de la distribution |
| `/etc/resolv.conf` | Linux | résolution DNS, souvent générée |

# 94. Tableau des choix techniques

| Besoin | Choix conseillé |
|---|---|
| Développement Linux sur poste Windows | WSL 2 |
| Applications GUI Linux ponctuelles | WSLg |
| Bureau Linux complet | VM ou machine Linux |
| Fichiers utilisés par outils Linux | système de fichiers Linux |
| Fichiers utilisés principalement par outils Windows | NTFS Windows |
| Réseau Windows 11 moderne | tester `mirrored` |
| Service Linux | systemd |
| Sauvegarde distribution | `wsl --export` |
| Code source | Git + remote |
| USB | `usbipd-win` selon besoin |

# 95. Ce qu'il faut retenir

1. **WSL 2 utilise un vrai noyau Linux** dans une VM utilitaire gérée par Windows.
2. **WSL 2 est l'architecture par défaut** pour les nouvelles distributions.
3. L'installation moderne commence par :

```powershell
wsl --install
```

4. Maintenir WSL avec :

```powershell
wsl --update
```

5. `systemd` est pris en charge par WSL 2 moderne.
6. WSLg remplace, pour les applications graphiques ordinaires, les anciennes recettes Xorg + VNC.
7. Pour des outils Linux, conserver les projets dans :

```text
~/projects
```

plutôt que `/mnt/c/...`.
8. `/etc/wsl.conf` et `.wslconfig` n'ont pas la même portée.
9. Le réseau moderne WSL inclut NAT, mirrored, DNS tunneling, auto proxy et intégration au pare-feu Windows/Hyper-V.
10. Une distribution importante doit être sauvegardée avec :

```powershell
wsl --export
```

11. `wsl --unregister` est destructif.
12. WSL est un excellent environnement de développement, mais **pas une preuve que la production Linux se comportera exactement de la même manière**.

# 96. Sources et documentation à maintenir

> [!tip]
> WSL évolue plus vite que les anciennes fonctionnalités Windows intégrées au système. Pour une commande d'administration ou un réglage de `.wslconfig`, vérifier la documentation actuelle et `wsl --help` plutôt que de conserver une recette vieille de plusieurs années.

Documentation Microsoft officielle :

- WSL : <https://learn.microsoft.com/windows/wsl/>
- Installation : <https://learn.microsoft.com/windows/wsl/install>
- Commandes : <https://learn.microsoft.com/windows/wsl/basic-commands>
- Comparaison WSL 1 / WSL 2 : <https://learn.microsoft.com/windows/wsl/compare-versions>
- Configuration : <https://learn.microsoft.com/windows/wsl/wsl-config>
- systemd : <https://learn.microsoft.com/windows/wsl/systemd>
- Fichiers : <https://learn.microsoft.com/windows/wsl/filesystems>
- Réseau : <https://learn.microsoft.com/windows/wsl/networking>
- WSLg : <https://learn.microsoft.com/windows/wsl/tutorials/gui-apps>
- Gestion de l'espace disque : <https://learn.microsoft.com/windows/wsl/disk-space>
- USB : <https://learn.microsoft.com/windows/wsl/connect-usb>
- Dépannage : <https://learn.microsoft.com/windows/wsl/troubleshooting>

# 97. Fiche de révision

## Architecture

```text
WSL 1 = traduction des appels système
WSL 2 = vrai noyau Linux + VM utilitaire légère
```

## Installation

```powershell
wsl --install
```

## Mise à jour

```powershell
wsl --update
```

## Inspection

```powershell
wsl --version
wsl --status
wsl -l -v
```

## Redémarrage complet

```powershell
wsl --shutdown
```

## systemd

```bash
ps -p 1 -o comm=
systemctl status
```

## Projet Linux

```text
~/projects
```

## Windows depuis Linux

```bash
explorer.exe .
powershell.exe
```

## Linux depuis Windows

```powershell
wsl -d Ubuntu -- <commande>
```

## Sauvegarde

```powershell
wsl --export Ubuntu backup.tar
```

## Suppression destructive

```powershell
wsl --unregister Ubuntu
```

## GUI

```text
WSLg
```

## Réseau moderne

```ini
[wsl2]
networkingMode=mirrored
dnsTunneling=true
autoProxy=true
firewall=true
```
