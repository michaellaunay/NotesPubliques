---
schema_version: 1
uid: 01M02EX5CEF7YK9JF8DK4YJZS8
titre: /proc
aliases:
- /proc
- procfs
type: cours
statut: actif
para: ressource
domaines:
- enseignement
themes:
- informatique
- administration-systeme
- gnu-linux
- noyau
- procfs
resume: 'Cours approfondi et progressif sur procfs sous GNU/Linux : modèle de procfs, processus, mémoire, descripteurs, namespaces, réseau, sysctl, sécurité, PSI, conteneurs et diagnostic de production.'
niveau: avance
prerequis:
- '[[GNULinux]]'
- '[[Les namespaces Linux]]'
auteurs:
- Michaël Launay
langue: fr
date_creation: 2026-05-24
date_modification: '2026-08-31'
confidentialite: publique
publication:
- notes-publiques
rag: true
metadata_verifiees: true
---

> [!tip] Version longue
> Ce cours existe aussi sous forme de livre complet : [[proc — livre complet]].

# Le système de fichiers `/proc` sous GNU/Linux


> [!abstract] Référence du cours
> Ce cours décrit **procfs** comme une interface noyau vivante : les détails exacts peuvent dépendre de la version du noyau, de sa configuration, du contexte de namespaces et des mécanismes de sécurité actifs. Les exemples sont conçus pour des GNU/Linux récents et doivent être vérifiés avant toute écriture dans `/proc/sys`.


# Chapitre 1 — Comprendre procfs avant de le parcourir


## Objectifs du cours

À l'issue du cours, nous savons :

1. expliquer ce qu'est procfs et pourquoi il existe ;
2. distinguer `/proc`, `/sys`, `/dev`, `/run` et `/sys/fs/cgroup` ;
3. analyser un processus via `/proc/<PID>` ;
4. lire correctement `cmdline`, `environ`, `status`, `stat`, `fd`, `maps` et `smaps` ;
5. diagnostiquer des problèmes de CPU, mémoire, I/O et fichiers ouverts ;
6. comprendre la visibilité de `/proc` dans les namespaces et conteneurs ;
7. interpréter les champs de sécurité d'un processus ;
8. utiliser `/proc/sys` sans confondre configuration temporaire et persistante ;
9. exploiter PSI (`/proc/pressure`) ;
10. éviter les erreurs de parsing, les races et les hypothèses non portables ;
11. savoir quand préférer une API, `sysctl`, `ps`, `ss`, `lsof`, `systemd`, eBPF ou les fichiers cgroup ;
12. construire une méthode de diagnostic de production reproductible.

---


## Modèle mental : un pseudo-système de fichiers

### Un pseudo-système de fichiers

Nous pouvons vérifier le type du montage :

```bash
findmnt -T /proc
```

Exemple :

```text
TARGET SOURCE FSTYPE OPTIONS
/proc  proc   proc   rw,nosuid,nodev,noexec,relatime
```

Le type est `proc`.

Le noyau fournit les opérations nécessaires au VFS (*Virtual File System*) pour que les informations internes puissent être manipulées avec une interface de fichiers.

Ainsi :

```bash
cat /proc/uptime
```

ne lit généralement pas des octets persistés sur un SSD. Le noyau génère une représentation de son état au moment de la lecture.

### Taille apparente de zéro

De nombreuses entrées indiquent une taille nulle :

```bash
stat /proc/meminfo
```

Cela ne signifie pas que le fichier est vide.

La taille affichée par les métadonnées VFS n'est pas nécessairement une indication fiable de la quantité de texte qui sera retournée.

Conséquence :

> Nous ne devons pas écrire un parseur de procfs qui alloue son buffer uniquement à partir de `st_size`.

### Une photographie non transactionnelle

Les informations sont dynamiques.

Entre :

```bash
ls /proc/1234
```

et :

```bash
cat /proc/1234/status
```

le processus `1234` peut avoir disparu et son PID peut éventuellement être réutilisé plus tard.

Il faut donc accepter :

- `ENOENT` ;
- `EACCES` ;
- une information ayant changé entre deux lectures ;
- des races inévitables.

Pour des programmes sensibles à la réutilisation des PID, les **pidfds** sont souvent une primitive plus sûre que le simple stockage d'un entier PID.

### Une ABI utilisateur, pas une API de haut niveau

Une partie importante de procfs constitue une ABI entre le noyau et l'espace utilisateur.

Cela ne signifie pas que chaque fichier a un format idéal pour notre application.

Lorsque nous disposons d'une interface spécialisée, nous la préférons souvent :

```text
Besoin                     Interface souvent préférable
-------------------------  --------------------------------------
Processus                  ps / procps-ng / API dédiée
Sockets                    ss / netlink
Paramètres sysctl          sysctl
Montages                   findmnt / libmount
Limites cgroup             /sys/fs/cgroup
Journal système            journalctl
Événements noyau           tracefs / perf / eBPF selon le besoin
Infos matérielles          sysfs / lscpu / udev selon le besoin
```

`/proc` reste néanmoins indispensable pour comprendre ce que font ces outils.

---


## Situer `/proc` parmi les pseudo-systèmes de fichiers

### `/proc`

Procfs expose principalement :

- processus et threads ;
- informations globales du noyau ;
- statistiques ;
- paramètres `sysctl` sous `/proc/sys`.

### `/sys`

Sysfs expose surtout le **modèle des objets du noyau** :

- périphériques ;
- pilotes ;
- bus ;
- classes ;
- topologie ;
- paramètres liés aux objets noyau.

Exemple :

```bash
ls /sys/class/net
```

### `/dev`

`/dev` contient principalement les nœuds permettant de communiquer avec des périphériques ou pseudo-périphériques :

```text
/dev/null
/dev/zero
/dev/random
/dev/tty
/dev/nvme0n1
```

### `/run`

`/run` contient des **données volatiles de l'espace utilisateur** créées depuis le démarrage :

- PID files historiques ;
- sockets ;
- verrous ;
- état des services ;
- données temporaires de démons.

### `/sys/fs/cgroup`

Sur un système moderne en **cgroup v2**, les limites et compteurs de ressources d'un cgroup vivent principalement sous :

```text
/sys/fs/cgroup
```

C'est essentiel dans les conteneurs.

Par exemple, `/proc/meminfo` peut refléter une vue beaucoup plus large que la limite mémoire imposée au conteneur.

Pour connaître la limite cgroup réelle, nous devons inspecter les fichiers cgroup correspondants, par exemple :

```bash
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/memory.current
```

> [!warning]
> **Ne pas utiliser `/proc/meminfo` seul pour décider de la mémoire réellement disponible dans un conteneur.**

---


## Montage et options de procfs

### Montage normal

En système complet, `/proc` est généralement monté très tôt au boot.

Nous pouvons l'observer avec :

```bash
findmnt -t proc
```

Dans un environnement isolé pédagogique :

```bash
sudo mount -t proc proc /mnt/proc
```

### Options classiques

Des options génériques courantes sont :

```text
nosuid
nodev
noexec
```

Elles ne remplacent toutefois pas les contrôles propres à procfs.

### `hidepid`

Depuis Linux 5.8, les options de procfs sont propres à chaque instance de montage, et le noyau fournit plusieurs modes de visibilité des PID (formes numériques ou textuelles) :

```text
hidepid=0 / off         visibilité classique
hidepid=1 / noaccess    protège les informations des autres utilisateurs
hidepid=2 / invisible   masque en plus leurs répertoires PID
hidepid=4 / ptraceable  ne montre que les processus que l'appelant peut ptrace
```

Exemple conceptuel :

```bash
sudo mount -o remount,hidepid=2 /proc
```

Le résultat doit être évalué avec les outils locaux avant un déploiement en production, car des démons de supervision peuvent dépendre de la visibilité globale des processus.

### `gid=`

L'option `gid=` permet d'accorder une visibilité particulière à un groupe lorsqu'un `hidepid` restrictif est utilisé.

Nous devons limiter ce groupe aux outils qui ont réellement besoin de cette visibilité.

### `subset=pid`

Une instance de procfs montée avec :

```text
subset=pid
```

n'expose que la partie liée aux tâches/processus et masque les autres entrées top-level de procfs. Cette option existe depuis Linux 5.8.

Cette option est intéressante pour réduire la surface visible dans certains environnements isolés.

### `pidns=`

Depuis **Linux 6.18** (fin 2025), le noyau permet également de sélectionner explicitement le PID namespace utilisé par une instance de procfs via l'option `pidns=` ; sur les noyaux antérieurs, cette option n'existe pas.

Cette notion complète le lien historique :

```text
PID namespace
      ↓
vue des PID
      ↓
instance de procfs
```

Nous approfondissons les namespaces dans `[[Les namespaces Linux]]`.

---


## Structure générale de `/proc`

Nous pouvons classer les entrées en plusieurs groupes.

### Répertoires de processus

```text
/proc/1
/proc/42
/proc/12345
```

Chaque nombre correspond à un **TGID/PID de processus visible dans le PID namespace de cette instance de procfs**.

### Processus courant

```text
/proc/self
```

est un lien magique résolu vers le processus qui effectue l'accès.

Exemple :

```bash
readlink /proc/self
```

Le PID affiché correspond à `readlink`, pas nécessairement au shell parent.

### Thread courant

```text
/proc/thread-self
```

pointe vers la tâche/thread qui effectue l'accès.

### Informations globales

Quelques entrées majeures :

```text
/proc/cpuinfo
/proc/meminfo
/proc/stat
/proc/loadavg
/proc/uptime
/proc/interrupts
/proc/vmstat
/proc/pressure/
/proc/modules
/proc/filesystems
```

### Paramètres noyau

```text
/proc/sys/
```

fournit la représentation procfs du mécanisme `sysctl`.

---


# Chapitre 2 — Lire l’état global du système

## Vue d’ensemble des fichiers globaux

**Les fichiers globaux les plus utiles**

À la racine de `/proc`, nous trouvons de nombreux fichiers qui donnent une vue générale du système.

Nous rencontrons notamment :

```text
/proc/cpuinfo
/proc/meminfo
/proc/version
/proc/uptime
/proc/loadavg
/proc/cmdline
/proc/filesystems
/proc/mounts
/proc/swaps
/proc/partitions
/proc/modules
/proc/interrupts
/proc/stat
```

Ces fichiers ne décrivent pas un processus particulier. Ils exposent une partie de l’état global du noyau.

Nous pouvons commencer par les lister :

```bash
ls -lh /proc/cpuinfo /proc/meminfo /proc/version /proc/uptime /proc/loadavg
```

Nous observons encore une fois que beaucoup de ces fichiers affichent une taille de `0`, car leur contenu est généré dynamiquement.


**Première lecture rapide**

Nous pouvons obtenir un premier aperçu du système avec :

```bash
cat /proc/version
cat /proc/cpuinfo | head
cat /proc/meminfo | head
cat /proc/uptime
cat /proc/loadavg
```

Ces quelques commandes permettent déjà de répondre à des questions importantes :

- quelle version du noyau est utilisée ?

- quel processeur est visible ?

- combien de mémoire est disponible ?

- depuis combien de temps le système tourne-t-il ?

- quelle est la charge moyenne du système ?


Dans un diagnostic réel, ces informations constituent souvent la première couche d’observation.


## Informations CPU avec `/proc/cpuinfo`

**Lire `/proc/cpuinfo`**

Le fichier `/proc/cpuinfo` expose des informations sur les processeurs logiques visibles par le noyau.

Nous le lisons avec :

```bash
cat /proc/cpuinfo
```

Sur une machine classique, nous obtenons un bloc d’informations par processeur logique.

Un extrait peut ressembler à ceci :

```text
processor   : 0
vendor_id   : GenuineIntel
cpu family  : 6
model       : 154
model name  : Intel(R) Core(TM) i7-12700H
stepping    : 3
microcode   : 0xffffffff
cpu MHz     : 2300.000
cache size  : 24576 KB
physical id : 0
siblings    : 20
core id     : 0
cpu cores   : 14
apicid      : 0
flags       : fpu vme de pse tsc msr pae mce cx8 apic ...
```

Nous devons apprendre à distinguer les informations réellement utiles des détails très bas niveau.

**Nombre de CPU logiques**

Chaque entrée `processor` correspond à un CPU logique vu par Linux.

Pour les compter :

```bash
grep -c '^processor' /proc/cpuinfo
```

Exemple :

```text
16
```

Cela signifie que Linux voit 16 processeurs logiques.

Un processeur logique n’est pas forcément un cœur physique. Il peut s’agir d’un thread matériel, par exemple avec l’Hyper-Threading chez Intel ou SMT chez AMD.

Nous pouvons aussi utiliser :

```bash
nproc
```

Mais ici, notre objectif est de comprendre la source d’information brute.

**Modèle du processeur**

Pour afficher le nom du modèle :

```bash
grep 'model name' /proc/cpuinfo | head -1
```

Exemple :

```text
model name  : AMD Ryzen 9 5900X 12-Core Processor
```

Nous utilisons `head -1` car le nom est répété pour chaque processeur logique.

Nous pouvons extraire uniquement la valeur :

```bash
awk -F': ' '/model name/ { print $2; exit }' /proc/cpuinfo
```

**Cœurs physiques et threads logiques**

Dans `/proc/cpuinfo`, plusieurs champs peuvent nous aider :

```text
physical id
core id
cpu cores
siblings
processor
```

Nous devons distinguer :

- `processor` : identifiant du CPU logique ;

- `physical id` : identifiant du socket physique ;

- `core id` : identifiant du cœur dans un socket ;

- `cpu cores` : nombre de cœurs physiques par socket ;

- `siblings` : nombre de threads logiques par socket.


Exemple simplifié :

```text
1 socket physique
8 cœurs physiques
16 processeurs logiques
```

Cela signifie probablement que chaque cœur expose deux threads matériels.

Nous pouvons comparer avec :

```bash
lscpu
```

`lscpu` donne une vue plus lisible, mais s’appuie sur les informations fournies par le noyau.

**Flags CPU**

Le champ `flags` est très important.

Exemple :

```bash
grep -m1 '^flags' /proc/cpuinfo
```

Il peut contenir des indicateurs comme :

```text
sse
sse2
avx
avx2
avx512f
aes
vmx
svm
nx
lm
```

Ces flags indiquent des fonctionnalités supportées par le processeur.

Quelques exemples :

|Flag|Signification|
|---|---|
|`sse`, `sse2`, `avx`, `avx2`|extensions SIMD pour calcul vectoriel|
|`aes`|accélération matérielle AES|
|`vmx`|virtualisation Intel VT-x|
|`svm`|virtualisation AMD-V|
|`nx`|bit No-eXecute pour la protection mémoire|
|`lm`|long mode, support 64 bits|

Ces informations peuvent être utiles pour :

- vérifier si une machine supporte la virtualisation ;

- vérifier la disponibilité d’instructions cryptographiques ;

- comprendre pourquoi un binaire optimisé ne fonctionne pas ;

- diagnostiquer un problème de performance lié aux instructions CPU.

**Limites de `/proc/cpuinfo`**

Nous devons rester prudents.

Dans une machine virtuelle, `/proc/cpuinfo` décrit le processeur virtuel exposé par l’hyperviseur, pas nécessairement le processeur physique complet.

Dans un conteneur, `/proc/cpuinfo` peut montrer plus de CPU que ceux réellement alloués par les limites cgroups.

Par exemple, dans Docker ou Kubernetes, une application peut voir 16 CPU dans `/proc/cpuinfo`, mais être limitée à 2 CPU par cgroup.

Nous retenons donc :

```text
/proc/cpuinfo indique les CPU visibles, pas toujours les ressources réellement garanties.
```

Pour les conteneurs modernes, nous devons aussi regarder les cgroups, par exemple dans `/sys/fs/cgroup`.

## Mémoire système avec `/proc/meminfo`

**Lire `/proc/meminfo`**

Le fichier `/proc/meminfo` expose l’état détaillé de la mémoire.

Nous le lisons avec :

```bash
cat /proc/meminfo
```

Extrait possible :

```text
MemTotal:       16234568 kB
MemFree:         1234567 kB
MemAvailable:   9345672 kB
Buffers:          234567 kB
Cached:          4567890 kB
SwapCached:            0 kB
Active:          5432100 kB
Inactive:        3210000 kB
SwapTotal:       2097148 kB
SwapFree:        2097148 kB
Dirty:             12345 kB
Writeback:             0 kB
```

Ce fichier est beaucoup plus riche que la simple sortie de `free`.

**`MemTotal`**

`MemTotal` indique la quantité totale de mémoire RAM visible par le noyau.

Exemple :

```bash
grep '^MemTotal:' /proc/meminfo
```

Sortie :

```text
MemTotal:       16234568 kB
```

Nous pouvons convertir en Gio :

```bash
awk '/^MemTotal:/ { printf "%.2f GiB\n", $2/1024/1024 }' /proc/meminfo
```

Nous faisons attention : la valeur est exprimée en kilo-octets selon l’affichage de `/proc/meminfo`.

**`MemFree`**

`MemFree` indique la mémoire complètement inutilisée.

```bash
grep '^MemFree:' /proc/meminfo
```

Mais cette valeur ne suffit pas pour savoir si le système manque de mémoire.

Linux utilise volontairement la mémoire disponible pour le cache disque. Une mémoire “non libre” peut donc être facilement récupérable.

Nous devons éviter l’erreur classique :

```text
MemFree faible ≠ système forcément saturé
```

**`MemAvailable`**

`MemAvailable` est souvent l’indicateur le plus utile pour estimer la mémoire encore disponible pour lancer de nouveaux programmes.

```bash
grep '^MemAvailable:' /proc/meminfo
```

Il tient compte de la mémoire libre, mais aussi de certaines zones de cache récupérables.

Nous pouvons afficher la valeur en Mio :

```bash
awk '/^MemAvailable:/ { printf "%.0f MiB\n", $2/1024 }' /proc/meminfo
```

Dans un diagnostic simple, nous privilégions souvent `MemAvailable` à `MemFree`.

**`Buffers` et `Cached`**

Linux utilise la RAM pour accélérer les accès disque.

Deux champs importants sont :

```bash
grep -E '^(Buffers|Cached):' /proc/meminfo
```

- `Buffers` correspond à certaines métadonnées et buffers liés aux blocs ;

- `Cached` correspond au cache de fichiers.


Cette mémoire peut souvent être libérée si une application en a besoin.

C’est pourquoi une machine Linux peut afficher peu de mémoire libre tout en ayant beaucoup de mémoire disponible.

**`SwapTotal` et `SwapFree`**

Le swap est une extension de la mémoire sur disque.

Nous regardons :

```bash
grep -E '^(SwapTotal|SwapFree):' /proc/meminfo
```

Exemple :

```text
SwapTotal:       2097148 kB
SwapFree:        1048576 kB
```

Nous pouvons calculer le swap utilisé :

```bash
awk '
/^SwapTotal:/ { total=$2 }
/^SwapFree:/ { free=$2 }
END { printf "Swap utilisé : %.0f MiB\n", (total-free)/1024 }
' /proc/meminfo
```

Un peu de swap utilisé n’est pas forcément grave. En revanche, une utilisation massive et active du swap peut indiquer une pression mémoire importante.

**`Dirty` et `Writeback`**

Deux champs sont utiles pour comprendre les écritures disque :

```bash
grep -E '^(Dirty|Writeback):' /proc/meminfo
```

- `Dirty` : mémoire contenant des données modifiées mais pas encore écrites sur disque ;

- `Writeback` : mémoire en cours d’écriture vers le disque.


Si `Dirty` est très élevé, cela peut indiquer que le système accumule des écritures en attente.

Dans certains incidents, nous pouvons observer une forte latence disque associée à une quantité importante de pages dirty.

**Comparaison avec `free`**

La commande :

```bash
free -h
```

présente une synthèse lisible de la mémoire.

Mais elle s’appuie en grande partie sur les informations de `/proc/meminfo`.

Nous pouvons comparer :

```bash
free -h
cat /proc/meminfo | head
```

Notre objectif n’est pas de remplacer `free`, mais de comprendre ce qu’il résume.

**Limites dans les conteneurs**

Dans certains environnements conteneurisés, `/proc/meminfo` peut présenter la mémoire de l’hôte, tandis que le conteneur est limité par cgroup.

Cela dépend de la version du noyau, du runtime, de la configuration et du système de cgroups.

Nous devons donc être prudents dans Docker et Kubernetes.

Une application peut lire :

```text
MemTotal: 64 Go
```

alors que le conteneur est limité à :

```text
2 Go
```

Pour diagnostiquer un conteneur, nous devons regarder aussi les limites cgroups dans `/sys/fs/cgroup`.

## Version du noyau avec `/proc/version`

**Lire la version du noyau**

Le fichier `/proc/version` donne des informations sur le noyau courant.

```bash
cat /proc/version
```

Exemple :

```text
Linux version 6.8.0-31-generic (buildd@lcy02-amd64-...) (x86_64-linux-gnu-gcc-13 ...) #31-Ubuntu SMP PREEMPT_DYNAMIC ...
```

Nous pouvons y trouver :

- la version du noyau ;

- le nom de construction ;

- le compilateur utilisé ;

- la date ou les métadonnées de compilation ;

- des informations de distribution.

**Comparaison avec `uname`**

La commande classique est :

```bash
uname -a
```

Exemple :

```text
Linux machine 6.8.0-31-generic #31-Ubuntu SMP PREEMPT_DYNAMIC x86_64 GNU/Linux
```

Nous pouvons aussi utiliser :

```bash
uname -r
```

pour obtenir uniquement la version du noyau :

```text
6.8.0-31-generic
```

`/proc/version` donne souvent plus de détails sur la compilation.

**À ne pas confondre avec la version de distribution**

Le noyau Linux et la distribution sont deux choses différentes.

Pour connaître la distribution, nous utilisons plutôt :

```bash
cat /etc/os-release
```

Exemple :

```text
NAME="Ubuntu"
VERSION="24.04 LTS (Noble Numbat)"
```

Nous retenons :

```text
/proc/version → version du noyau
/etc/os-release → version de la distribution
```

Cette distinction est importante en support système.

## Temps de fonctionnement avec `/proc/uptime`

**Lire `/proc/uptime`**

Le fichier `/proc/uptime` contient deux valeurs :

```bash
cat /proc/uptime
```

Exemple :

```text
25420.15 98120.42
```

La première valeur représente le temps écoulé depuis le démarrage du système, en secondes.

La seconde représente le temps passé en idle par les CPU, cumulé.

**Interpréter la première valeur**

La première valeur est l’uptime.

Nous pouvons la convertir grossièrement :

```bash
awk '{ printf "Uptime : %.2f heures\n", $1/3600 }' /proc/uptime
```

Ou en jours :

```bash
awk '{ printf "Uptime : %.2f jours\n", $1/86400 }' /proc/uptime
```

Cette information permet de savoir depuis combien de temps le système n’a pas redémarré.

**Interpréter la seconde valeur**

La seconde valeur correspond au temps idle cumulé sur tous les CPU.

Sur une machine avec plusieurs CPU logiques, cette valeur peut augmenter plus vite que le temps réel.

Exemple :

- si le système a 4 CPU logiques ;

- si les 4 CPU restent inactifs pendant 1 seconde ;

- le temps idle cumulé augmente d’environ 4 secondes.


Nous devons donc éviter de comparer naïvement la seconde valeur à la première.

**Comparaison avec `uptime`**

La commande :

```bash
uptime
```

affiche une sortie plus lisible :

```text
15:42:01 up 2 days,  3:12,  2 users,  load average: 0.24, 0.32, 0.28
```

Elle combine notamment des informations issues de `/proc/uptime` et `/proc/loadavg`.

## Charge système avec `/proc/loadavg`

**Lire `/proc/loadavg`**

Nous lisons :

```bash
cat /proc/loadavg
```

Exemple :

```text
0.35 0.42 0.38 2/842 15321
```

Ce fichier contient cinq informations :

```text
load1 load5 load15 running/total last_pid
```

Dans l’exemple :

- `0.35` : charge moyenne sur 1 minute ;

- `0.42` : charge moyenne sur 5 minutes ;

- `0.38` : charge moyenne sur 15 minutes ;

- `2/842` : 2 tâches exécutables sur 842 tâches totales ;

- `15321` : dernier PID attribué.

**Que signifie la charge moyenne ?**

La charge moyenne mesure le nombre moyen de tâches :

- en cours d’exécution ;

- prêtes à s’exécuter ;

- parfois bloquées en attente non interruptible, notamment sur des entrées/sorties.


Ce n’est pas simplement un pourcentage CPU.

Une charge de `4.00` n’a pas le même sens sur une machine à 2 CPU logiques et sur une machine à 16 CPU logiques.

**Interprétation selon le nombre de CPU**

Supposons une machine avec 4 CPU logiques.

Une règle simplifiée :

|Load average|Interprétation approximative|
|---|---|
|`0.50`|machine peu chargée|
|`2.00`|charge modérée|
|`4.00`|CPU pleinement utilisés|
|`8.00`|file d’attente importante|

Mais cette interprétation dépend fortement du contexte.

Pour connaître le nombre de CPU logiques :

```bash
nproc
```

ou :

```bash
grep -c '^processor' /proc/cpuinfo
```

**Les trois valeurs : 1, 5 et 15 minutes**

Les trois valeurs permettent de voir la tendance.

Exemple :

```text
8.00 2.00 1.00
```

La charge est montée récemment.

Exemple inverse :

```text
1.00 4.00 8.00
```

La charge était élevée avant, mais elle redescend.

Nous devons donc lire les trois valeurs ensemble.

**Charge CPU ou attente I/O ?**

Une charge élevée ne signifie pas toujours que le CPU est saturé.

Des processus bloqués en attente disque peuvent faire monter la charge.

Pour différencier les causes, nous devons compléter avec :

```bash
top
htop
vmstat 1
iostat
pidstat
```

Mais `/proc/loadavg` nous donne le premier signal.

## Ligne de commande du noyau avec `/proc/cmdline`

**Lire `/proc/cmdline`**

Le fichier `/proc/cmdline` contient les paramètres passés au noyau au démarrage.

```bash
cat /proc/cmdline
```

Exemple :

```text
BOOT_IMAGE=/boot/vmlinuz-6.8.0-31-generic root=UUID=... ro quiet splash
```

Nous y trouvons souvent :

- l’image noyau démarrée ;

- la partition racine ;

- des options de démarrage ;

- des paramètres de sécurité ;

- des paramètres de debug ;

- des options liées au matériel.

**Utilité pratique**

Ce fichier est utile pour vérifier :

- si le système démarre en mode normal ou debug ;

- quelle racine est utilisée ;

- si certaines mitigations CPU sont activées ou désactivées ;

- si des paramètres réseau, console ou mémoire ont été passés ;

- si des options spécifiques ont été configurées dans GRUB.


Exemple :

```bash
cat /proc/cmdline | tr ' ' '\n'
```

Cette commande affiche un paramètre par ligne.

## Systèmes de fichiers avec `/proc/filesystems`

**Lire les systèmes de fichiers supportés**

Nous lisons :

```bash
cat /proc/filesystems
```

Exemple :

```text
nodev   sysfs
nodev   tmpfs
nodev   proc
        ext4
        xfs
        btrfs
nodev   cgroup
nodev   devpts
```

Ce fichier indique les systèmes de fichiers connus par le noyau courant.

**Signification de `nodev`**

La mention `nodev` indique que le système de fichiers ne correspond pas à un périphérique bloc classique.

Par exemple :

```text
nodev proc
nodev sysfs
nodev tmpfs
```

`proc`, `sysfs` et `tmpfs` ne sont pas montés depuis une partition disque classique.

**Utilité pratique**

Ce fichier peut nous aider à vérifier si un type de système de fichiers est supporté.

Par exemple :

```bash
grep ext4 /proc/filesystems
grep btrfs /proc/filesystems
grep nfs /proc/filesystems
```

Si un système de fichiers n’apparaît pas, il peut être absent, non chargé ou disponible via un module non encore chargé.

## Points de montage avec `/proc/mounts`

**Lire `/proc/mounts`**

Le fichier `/proc/mounts` expose les points de montage vus par le processus courant.

```bash
cat /proc/mounts
```

Il peut être long. Nous pouvons le rendre plus lisible avec :

```bash
column -t /proc/mounts | less
```

ou utiliser directement :

```bash
findmnt
```

**Différence avec `/etc/fstab`**

`/etc/fstab` décrit les montages configurés de manière persistante.

`/proc/mounts` décrit les montages réellement actifs.

Nous retenons :

```text
/etc/fstab → intention de configuration
/proc/mounts → état réel du système
```

Il est possible qu’un montage actif ne soit pas dans `/etc/fstab`, et inversement qu’un montage dans `/etc/fstab` ne soit pas actuellement monté.

**Lien avec `/proc/self/mounts`**

Sur les systèmes modernes, `/proc/mounts` est souvent lié à :

```bash
/proc/self/mounts
```

Nous pouvons vérifier :

```bash
ls -l /proc/mounts
```

Cela signifie que les montages sont vus du point de vue du processus courant, ce qui est important avec les namespaces de montage dans les conteneurs.

## Swap avec `/proc/swaps`

**Lire les espaces de swap**

Nous lisons :

```bash
cat /proc/swaps
```

Exemple :

```text
Filename                Type        Size       Used    Priority
/swapfile               file        2097148    0       -2
```

Ce fichier indique :

- le fichier ou périphérique de swap ;

- son type ;

- sa taille ;

- la quantité utilisée ;

- sa priorité.

**Comparaison avec `swapon`**

La commande suivante donne une vue plus lisible :

```bash
swapon --show
```

Mais `/proc/swaps` est la source directe exposée par le noyau.


**Diagnostic rapide**

Si une machine est lente, nous pouvons vérifier :

```bash
cat /proc/swaps
grep -E '^(SwapTotal|SwapFree):' /proc/meminfo
```

Si beaucoup de swap est utilisé et que la mémoire disponible est faible, nous pouvons suspecter une pression mémoire.

Mais nous devons compléter avec d’autres outils pour savoir si le système est en train de swapper activement.

## Partitions avec `/proc/partitions`

**Lire les partitions connues**

Nous lisons :

```bash
cat /proc/partitions
```

Exemple :

```text
major minor  #blocks  name

   8        0  976762584 sda
   8        1     524288 sda1
   8        2  976236544 sda2
 259        0 1000204632 nvme0n1
 259        1     524288 nvme0n1p1
```

Nous voyons les périphériques bloc et leurs partitions.

**Limites**

Ce fichier donne une vue brute.

Pour une analyse plus lisible, nous utilisons souvent :

```bash
lsblk
blkid
fdisk -l
```

Mais `/proc/partitions` reste utile pour comprendre ce que le noyau voit.

## Modules noyau avec `/proc/modules`

**Lire les modules chargés**

Nous lisons :

```bash
cat /proc/modules
```

Extrait possible :

```text
iwlwifi 598016 1 iwlmvm, Live 0x0000000000000000
nvidia 56848384 125, Live 0x0000000000000000
vboxdrv 655360 3 vboxnetadp,vboxnetflt,vboxpci, Live 0x0000000000000000
```

Chaque ligne décrit un module noyau chargé.

**Comparaison avec `lsmod`**

La commande :

```bash
lsmod
```

affiche les modules de manière plus lisible.

En réalité, `lsmod` lit généralement `/proc/modules`.

Nous pouvons comparer :

```bash
lsmod | head
cat /proc/modules | head
```

**Utilité pratique**

Nous utilisons ces informations pour vérifier :

- si un pilote est chargé ;

- si un module réseau est présent ;

- si un module de virtualisation est actif ;

- si un module GPU est chargé ;

- si un module de système de fichiers est disponible.


Exemple :

```bash
grep -E 'nvidia|amdgpu|i915' /proc/modules
```

## Interruptions avec `/proc/interrupts`

**Lire les interruptions**

Le fichier `/proc/interrupts` expose les interruptions matérielles et leur répartition par CPU.

```bash
cat /proc/interrupts
```

Extrait simplifié :

```text
           CPU0       CPU1       CPU2       CPU3
  0:         12          0          0          0   IO-APIC   2-edge      timer
  1:        120          0          0          0   IO-APIC   1-edge      i8042
 16:      10234       9876      11234       9543   IO-APIC  16-fasteoi   eth0
```

Nous voyons combien d’interruptions chaque CPU a traité.

**Utilité avancée**

Ce fichier devient utile lorsque nous analysons :

- une carte réseau très sollicitée ;

- une répartition déséquilibrée des interruptions ;

- une latence système ;

- un problème de pilote ;

- une machine haute performance.


Dans un cours introductif à `/proc`, nous ne détaillons pas encore l’optimisation IRQ, mais nous retenons que `/proc/interrupts` donne une vue précieuse sur l’activité matérielle.

## Statistiques globales avec `/proc/stat`

**Lire `/proc/stat`**

Le fichier `/proc/stat` contient de nombreuses statistiques globales.

```bash
cat /proc/stat | head
```

Exemple :

```text
cpu  123456 789 34567 9876543 1234 0 567 0 0 0
cpu0 12345 67 3456 987654 123 0 56 0 0 0
cpu1 12346 68 3457 987655 124 0 57 0 0 0
intr 123456789 ...
ctxt 987654321
btime 1715860000
processes 123456
procs_running 2
procs_blocked 0
```

Ce fichier est utilisé par de nombreux outils de monitoring.

**Champs utiles**

Nous pouvons y trouver :

- temps CPU par mode ;

- nombre total d’interruptions ;

- nombre de changements de contexte ;

- heure de boot ;

- nombre de processus créés depuis le démarrage ;

- processus en cours d’exécution ;

- processus bloqués.


Exemple :

```bash
grep '^btime' /proc/stat
```

`btime` donne l’heure de démarrage du système sous forme de timestamp Unix.

Nous pouvons la convertir :

```bash
date -d "@$(awk '/^btime/ {print $2}' /proc/stat)"
```

**Pourquoi `/proc/stat` est plus difficile**

Contrairement à `/proc/meminfo`, `/proc/stat` est moins lisible directement.

Les champs CPU nécessitent une interprétation précise.

Nous ne les utilisons pas naïvement.

Pour comprendre l’utilisation CPU, nous préférons souvent :

```bash
top
mpstat
pidstat
vmstat
```

Mais ces outils reposent en partie sur ces statistiques.

## Construire une première synthèse système

**Objectif**

Nous voulons maintenant construire une commande qui produit une synthèse simple à partir de `/proc`.

Nous cherchons à afficher :

- version du noyau ;

- uptime ;

- CPU logiques ;

- mémoire totale ;

- mémoire disponible ;

- load average ;

- nombre de processus visibles.

**Script Bash simple**

Nous pouvons écrire :

```bash
##!/usr/bin/env bash

kernel=$(uname -r)
cpu_count=$(grep -c '^processor' /proc/cpuinfo)
mem_total=$(awk '/^MemTotal:/ { printf "%.1f GiB", $2/1024/1024 }' /proc/meminfo)
mem_available=$(awk '/^MemAvailable:/ { printf "%.1f GiB", $2/1024/1024 }' /proc/meminfo)
uptime_days=$(awk '{ printf "%.2f jours", $1/86400 }' /proc/uptime)
loadavg=$(awk '{ print $1 " " $2 " " $3 }' /proc/loadavg)
process_count=$(ls /proc | grep -E '^[0-9]+$' | wc -l)

echo "Kernel             : $kernel"
echo "CPU logiques       : $cpu_count"
echo "Mémoire totale     : $mem_total"
echo "Mémoire disponible : $mem_available"
echo "Uptime             : $uptime_days"
echo "Load average       : $loadavg"
echo "Processus visibles : $process_count"
```

Nous pouvons le sauvegarder :

```bash
nano proc-summary.sh
chmod +x proc-summary.sh
./proc-summary.sh
```

**Discussion sur la robustesse**

Ce script est volontairement simple.

Dans un environnement de production, nous devons améliorer plusieurs points :

- gérer les erreurs de lecture ;

- éviter de supposer que tous les champs existent ;

- tenir compte des conteneurs ;

- éviter certains parsings trop fragiles ;

- gérer les différences de noyau ;

- produire une sortie machine-readable, par exemple JSON.


Mais ce script montre déjà que `/proc` permet de construire rapidement un outil d’observation système.

## Étude de cas : première analyse d’un serveur

**Situation**

Nous nous connectons à un serveur qui semble lent.

Nous ne savons pas encore si le problème vient :

- du CPU ;

- de la mémoire ;

- du swap ;

- du disque ;

- du réseau ;

- d’un processus particulier.


Nous utilisons uniquement quelques fichiers globaux de `/proc` pour une première analyse.

**Étape 1 : version et uptime**

```bash
cat /proc/version
cat /proc/uptime
```

Nous cherchons à savoir :

- si le noyau est ancien ;

- si la machine vient de redémarrer ;

- si l’uptime est très long ;

- si un redémarrage récent peut expliquer un incident.

**Étape 2 : CPU visible**

```bash
grep -c '^processor' /proc/cpuinfo
grep 'model name' /proc/cpuinfo | head -1
```

Nous identifions la capacité CPU apparente.

Si la charge moyenne est de `8` sur une machine à `2` CPU logiques, la situation est plus préoccupante que sur une machine à `32` CPU logiques.

**Étape 3 : charge moyenne**

```bash
cat /proc/loadavg
```

Nous interprétons les trois valeurs.

Exemple :

```text
12.00 10.50 8.20
```

Si la machine a 4 CPU logiques, la charge est probablement élevée.

Mais nous ne concluons pas encore que le CPU est seul responsable.

**Étape 4 : mémoire disponible**

```bash
grep -E '^(MemTotal|MemFree|MemAvailable|Cached|SwapTotal|SwapFree):' /proc/meminfo
```

Nous observons :

- mémoire totale ;

- mémoire réellement disponible ;

- cache ;

- swap total ;

- swap libre.


Si `MemAvailable` est très bas et que le swap est largement utilisé, nous suspectons une pression mémoire.

**Étape 5 : hypothèse initiale**

À partir de ces informations, nous pouvons formuler une première hypothèse :

```text
La machine présente une charge élevée, avec peu de mémoire disponible et une utilisation importante du swap. Le ralentissement peut être lié à une pression mémoire provoquant des attentes I/O.
```

Ou au contraire :

```text
La charge est faible, la mémoire disponible est correcte, et le swap n’est pas utilisé. Le ralentissement perçu vient probablement d’un service spécifique, du réseau ou d’un stockage distant.
```

Nous ne cherchons pas encore à conclure définitivement. Nous construisons une première lecture du système.

## Limites des informations globales

**Une vue globale ne suffit pas toujours**

Les fichiers globaux nous donnent une vision d’ensemble, mais ils ne disent pas toujours quel processus est responsable.

Par exemple :

```bash
cat /proc/loadavg
```

peut indiquer une forte charge, mais ne donne pas directement le processus responsable.

```bash
cat /proc/meminfo
```

peut indiquer une pression mémoire, mais ne dit pas immédiatement quelle application consomme le plus.

Pour aller plus loin, nous devons ensuite explorer :

```text
/proc/<PID>
```

C’est l’objet des chapitres suivants.

**Attention aux moyennes**

Les moyennes peuvent masquer des pics.

La charge moyenne sur 15 minutes peut être faible alors qu’un pic violent vient de se produire.

Inversement, la charge sur 15 minutes peut rester élevée alors que le problème est déjà terminé.

Nous devons toujours croiser :

- 1 minute ;

- 5 minutes ;

- 15 minutes ;

- observation en temps réel ;

- logs ;

- métriques applicatives.

**Attention aux environnements virtualisés**

Dans une machine virtuelle ou un conteneur, les fichiers globaux peuvent ne pas représenter exactement les ressources physiques.

Nous devons identifier le contexte :

```bash
systemd-detect-virt
```

ou observer les cgroups :

```bash
cat /proc/1/cgroup
```

Dans Kubernetes, la lecture brute de `/proc` peut être trompeuse si nous oublions les limites CPU et mémoire du pod.

# Chapitre 3 — Explorer un processus avec `/proc/<PID>`

## Les répertoires par PID

**Rappel : qu’est-ce qu’un PID ?**

Un processus est une instance d’un programme en cours d’exécution.

Chaque processus possède un identifiant numérique appelé PID, pour Process Identifier.

Nous pouvons afficher le PID du shell courant avec :

```bash
echo $$
```

Exemple :

```text
18542
```

Cela signifie que notre shell a pour PID `18542`.

Dans `/proc`, nous pouvons alors observer :

```bash
ls /proc/18542
```

ou directement :

```bash
ls /proc/$$
```

Nous constatons que chaque processus actif possède un répertoire nommé par son PID.


**Lister les processus visibles via `/proc`**

Pour lister les répertoires numériques de `/proc`, nous utilisons :

```bash
ls /proc | grep -E '^[0-9]+$'
```

Chaque ligne correspond à un processus visible dans notre namespace PID.

Pour compter ces processus :

```bash
ls /proc | grep -E '^[0-9]+$' | wc -l
```

Nous obtenons une estimation du nombre de processus visibles.

Nous parlons d’estimation car l’état change pendant que nous observons le système : des processus peuvent apparaître ou disparaître entre deux commandes.


**Observer le processus `1`**

Le processus `1` occupe une place particulière.

Nous pouvons l’observer avec :

```bash
cat /proc/1/comm
```

Sur une distribution moderne avec systemd, nous obtenons souvent :

```text
systemd
```

Mais dans un conteneur, nous pouvons obtenir autre chose, par exemple :

```text
bash
node
python
tini
```

Le PID `1` est le premier processus visible dans le namespace courant.

Sur un système classique, il correspond au processus d’initialisation. Dans un conteneur, il correspond souvent au processus principal du conteneur.

Nous retenons donc :

```text
/proc/1 ne signifie pas toujours “le vrai PID 1 de la machine physique”.
Il signifie “le PID 1 dans le namespace PID courant”.
```

## Structure générale de `/proc/<PID>`

**Explorer un processus**

Nous choisissons d’abord un processus simple.

Nous lançons :

```bash
sleep 1000 &
```

Puis nous récupérons son PID :

```bash
pid=$!
echo "$pid"
```

Nous explorons ensuite son répertoire :

```bash
ls -l /proc/$pid
```

Nous trouvons de nombreuses entrées, par exemple :

```text
cmdline
comm
cwd
environ
exe
fd
limits
maps
root
stat
status
task
```

Chaque entrée expose une information différente.

**Les catégories principales**

Nous pouvons regrouper les entrées de `/proc/<PID>` en plusieurs catégories.

|Entrée|Rôle|
|---|---|
|`cmdline`|ligne de commande complète|
|`comm`|nom court du processus|
|`status`|état lisible du processus|
|`stat`|statistiques compactes bas niveau|
|`environ`|variables d’environnement|
|`cwd`|répertoire courant|
|`exe`|exécutable associé|
|`root`|racine vue par le processus|
|`fd`|descripteurs de fichiers ouverts|
|`maps`|cartographie mémoire|
|`limits`|limites de ressources|
|`task`|threads du processus|

Dans ce chapitre, nous nous concentrons surtout sur `cmdline`, `comm`, `status`, `stat`, `environ`, `cwd`, `exe` et `root`.

Les entrées `fd`, `maps`, `smaps` et `limits` sont approfondies dans les chapitres suivants.

## `cmdline` : la ligne de commande du processus

**Lire `/proc/<PID>/cmdline`**

Le fichier `cmdline` contient la ligne de commande utilisée pour lancer le processus.

Exemple :

```bash
cat /proc/$pid/cmdline
```

Pour notre processus `sleep`, la sortie peut sembler étrange :

```text
sleep1000
```

En réalité, les arguments ne sont pas séparés par des espaces, mais par des caractères nuls `\0`.

C’est pour cela que l’affichage brut avec `cat` n’est pas toujours lisible.

**Affichage lisible de `cmdline`**

Nous pouvons remplacer les caractères nuls par des espaces :

```bash
tr '\0' ' ' < /proc/$pid/cmdline
echo
```

Sortie attendue :

```text
sleep 1000
```

Nous pouvons aussi afficher un argument par ligne :

```bash
tr '\0' '\n' < /proc/$pid/cmdline
```

Sortie :

```text
sleep
1000
```

Cette représentation est plus fidèle : chaque ligne correspond à un argument.

**Cas des processus noyau**

Certains processus ont une ligne de commande vide.

Par exemple, certains threads noyau peuvent apparaître avec une `cmdline` vide.

Nous pouvons rencontrer :

```bash
cat /proc/<PID>/cmdline
```

qui ne produit aucune sortie.

Cela ne signifie pas nécessairement que le processus n’existe pas. Cela peut signifier qu’il ne possède pas de ligne de commande utilisateur classique.

**Risque de sécurité**

La ligne de commande peut contenir des informations sensibles.

Exemples de mauvaises pratiques :

```bash
mysql -u user -pMonMotDePasse
curl -H "Authorization: Bearer TOKEN_SECRET" https://example.org
python app.py --api-key SECRET
```

Si ces secrets sont passés en arguments, ils peuvent se retrouver visibles dans :

```bash
/proc/<PID>/cmdline
```

et parfois dans les sorties de :

```bash
ps aux
```

Nous retenons une règle importante :

```text
Nous évitons de passer des secrets en arguments de ligne de commande.
```

Nous privilégions des mécanismes plus sûrs : fichiers de secrets avec permissions strictes, gestionnaires de secrets, variables d’environnement maîtrisées ou sockets protégées selon le contexte.

## `comm` : le nom court du processus

**Lire `/proc/<PID>/comm`**

Le fichier `comm` contient le nom court du processus.

```bash
cat /proc/$pid/comm
```

Exemple :

```text
sleep
```

Ce nom est généralement court et ne contient pas toute la ligne de commande.

**Différence entre `comm` et `cmdline`**

Nous comparons :

```bash
cat /proc/$pid/comm
tr '\0' ' ' < /proc/$pid/cmdline
echo
```

Sortie possible :

```text
sleep
sleep 1000
```

`comm` donne le nom du processus.

`cmdline` donne la commande complète avec ses arguments.

La différence est importante.

Un processus Python peut avoir :

```text
comm = python
cmdline = python manage.py runserver 0.0.0.0:8000
```

Dans un diagnostic, `cmdline` est souvent plus utile pour identifier précisément le rôle du processus.

## `status` : état lisible du processus

**Lire `/proc/<PID>/status`**

Le fichier `status` est l’un des fichiers les plus utiles pour lire rapidement les informations d’un processus.

```bash
cat /proc/$pid/status
```

Extrait possible :

```text
Name:   sleep
Umask:  0022
State:  S (sleeping)
Tgid:   18590
Pid:    18590
PPid:   18420
Uid:    1000    1000    1000    1000
Gid:    1000    1000    1000    1000
FDSize: 64
Threads:        1
VmPeak:     5788 kB
VmSize:     5788 kB
VmRSS:       912 kB
SigQ:   0/30995
SigPnd: 0000000000000000
SigBlk: 0000000000000000
SigIgn: 0000000000000000
SigCgt: 0000000000000000
```

Ce fichier est plus lisible que `stat`, car les champs sont nommés.

**Les champs d’identification**

Nous trouvons notamment :

```text
Name
State
Tgid
Pid
PPid
Uid
Gid
Threads
```

Nous les interprétons ainsi :

|Champ|Signification|
|---|---|
|`Name`|nom court du processus|
|`State`|état courant|
|`Tgid`|identifiant du groupe de threads|
|`Pid`|identifiant du processus ou thread|
|`PPid`|PID du processus parent|
|`Uid`|identifiants utilisateur|
|`Gid`|identifiants groupe|
|`Threads`|nombre de threads|

Le champ `PPid` est particulièrement utile pour comprendre qui a lancé un processus.


**Lire seulement les champs utiles**

Nous pouvons extraire les lignes principales :

```bash
grep -E '^(Name|State|Pid|PPid|Uid|Gid|Threads):' /proc/$pid/status
```

Exemple :

```text
Name:   sleep
State:  S (sleeping)
Pid:    18590
PPid:   18420
Uid:    1000    1000    1000    1000
Gid:    1000    1000    1000    1000
Threads:        1
```

Cette commande est utile pour un premier diagnostic.


**Les champs mémoire dans `status`**

Le fichier `status` contient aussi des informations mémoire :

```bash
grep -E '^(VmPeak|VmSize|VmRSS|RssAnon|RssFile|RssShmem):' /proc/$pid/status
```

Nous pouvons obtenir :

```text
VmPeak:     5788 kB
VmSize:     5788 kB
VmRSS:       912 kB
RssAnon:      92 kB
RssFile:     820 kB
RssShmem:      0 kB
```

Nous retenons seulement une idée générale pour l’instant :

- `VmSize` représente la taille de l’espace mémoire virtuel ;

- `VmRSS` représente la mémoire résidente en RAM ;

- `RssAnon`, `RssFile` et `RssShmem` détaillent une partie de cette mémoire.


Nous approfondissons ces notions dans le chapitre sur la mémoire des processus.


## Les états des processus

**Lire l’état dans `status`**

L’état du processus se lit dans :

```bash
grep '^State:' /proc/$pid/status
```

Exemple :

```text
State:  S (sleeping)
```

Le processus `sleep` est naturellement en état sleeping, car il attend l’écoulement du temps.


**Principaux états**

Nous rencontrons principalement les états suivants :

|Code|Nom|Signification|
|---|---|---|
|`R`|running|processus en cours d’exécution ou prêt à s’exécuter|
|`S`|sleeping|sommeil interruptible, attente normale|
|`D`|uninterruptible sleep|attente non interruptible, souvent I/O|
|`Z`|zombie|processus terminé mais non récolté par son parent|
|`T`|stopped|processus arrêté, par exemple avec `SIGSTOP`|
|`t`|tracing stop|processus arrêté car tracé ou debuggué|
|`I`|idle|thread noyau idle|


**État `R`**

Un processus en état `R` est soit en train de s’exécuter sur un CPU, soit prêt à s’exécuter.

Il est dans la file des tâches exécutables.

Exemple d’observation :

```bash
ps -eo pid,state,comm | awk '$2=="R"'
```

Cet état est normal si le processus travaille réellement.


**État `S`**

L’état `S` est très courant.

Il signifie que le processus dort, mais peut être réveillé par un signal ou un événement.

Exemples :

- un shell qui attend une commande ;

- un serveur qui attend une connexion ;

- un processus `sleep` ;

- une application qui attend une réponse réseau.


Cet état n’est pas inquiétant en soi.


**État `D`**

L’état `D` signifie un sommeil non interruptible.

Il apparaît souvent lorsqu’un processus attend une opération d’entrée/sortie noyau : disque, réseau bas niveau, système de fichiers distant, périphérique bloqué.

Un processus en état `D` peut être difficile à tuer, car il attend que le noyau termine l’opération.

Si nous observons beaucoup de processus en état `D`, nous suspectons souvent :

- un disque lent ou bloqué ;

- un montage NFS problématique ;

- un périphérique défaillant ;

- une saturation I/O ;

- un bug pilote ;

- un problème de stockage distant.


Nous pouvons chercher ces processus avec :

```bash
ps -eo pid,state,comm | awk '$2=="D"'
```


**État `Z`**

Un processus zombie est un processus terminé dont le parent n’a pas encore récupéré le code de sortie.

Il ne consomme presque plus de mémoire utile, mais il occupe encore une entrée dans la table des processus.

Nous pouvons les lister avec :

```bash
ps -eo pid,ppid,state,comm | awk '$3=="Z"'
```

Dans `/proc`, un zombie possède encore un répertoire `/proc/<PID>`, mais beaucoup d’informations ne sont plus disponibles comme pour un processus vivant.

Un zombie isolé n’est pas forcément grave. Beaucoup de zombies persistants peuvent indiquer un bug dans le processus parent.


**État `T`**

L’état `T` indique qu’un processus est arrêté.

Nous pouvons produire cet état avec :

```bash
sleep 1000 &
pid=$!
kill -STOP $pid
grep '^State:' /proc/$pid/status
```

Sortie possible :

```text
State:  T (stopped)
```

Nous pouvons le reprendre avec :

```bash
kill -CONT $pid
grep '^State:' /proc/$pid/status
```

Puis le terminer :

```bash
kill $pid
```

Nous voyons ici le lien entre signaux Unix et états de processus.


## `stat` : statistiques compactes bas niveau

**Lire `/proc/<PID>/stat`**

Le fichier `stat` contient de nombreuses informations compactes sur le processus.

```bash
cat /proc/$pid/stat
```

Exemple simplifié :

```text
18590 (sleep) S 18420 18590 18420 34816 18590 4194304 ...
```

Ce fichier est moins lisible que `status`, mais il est utilisé par de nombreux outils système.


**Les premiers champs**

Les premiers champs sont particulièrement importants :

|Position|Champ|Signification|
|---|---|---|
|1|`pid`|PID|
|2|`comm`|nom du processus entre parenthèses|
|3|`state`|état|
|4|`ppid`|PID du parent|
|5|`pgrp`|groupe de processus|
|6|`session`|session|

Pour les extraire rapidement, nous pouvons faire :

```bash
awk '{ print "pid=" $1, "comm=" $2, "state=" $3, "ppid=" $4 }' /proc/$pid/stat
```

Exemple :

```text
pid=18590 comm=(sleep) state=S ppid=18420
```


**Attention au parsing de `stat`**

Le parsing de `/proc/<PID>/stat` est piégeux.

Le champ `comm` est entre parenthèses et peut contenir des espaces ou des caractères particuliers.

Une méthode naïve qui découpe simplement sur les espaces peut être incorrecte dans certains cas.

Pour un cours d’introduction, nous retenons :

```text
Nous utilisons /proc/<PID>/status pour la lecture humaine.
Nous utilisons /proc/<PID>/stat avec prudence pour les outils bas niveau.
```


## `environ` : environnement du processus

**Lire `/proc/<PID>/environ`**

Le fichier `environ` contient les variables d’environnement du processus.

```bash
cat /proc/$pid/environ
```

Comme pour `cmdline`, les entrées sont séparées par des caractères nuls.

Nous les rendons lisibles avec :

```bash
tr '\0' '\n' < /proc/$pid/environ
```

Exemple :

```text
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOME=/home/gregoire
LANG=fr_FR.UTF-8
SHELL=/bin/bash
```


**À quoi servent les variables d’environnement ?**

Les variables d’environnement servent à transmettre des paramètres au processus.

Exemples courants :

```text
PATH
HOME
USER
LANG
SHELL
PWD
JAVA_HOME
NODE_ENV
PYTHONPATH
DATABASE_URL
```

Elles influencent le comportement des programmes.

Par exemple :

```bash
NODE_ENV=production node server.js
```

ou :

```bash
LANG=C sort fichier.txt
```


**Implications de sécurité**

`environ` peut contenir des secrets.

Exemples :

```text
DATABASE_URL=postgresql://user:password@host/db
AWS_SECRET_ACCESS_KEY=...
GITHUB_TOKEN=...
JWT_SECRET=...
```

Si un utilisateur non autorisé peut lire l’environnement d’un processus, il peut récupérer des informations sensibles.

C’est pourquoi les permissions, les options de montage de `/proc` et les pratiques de gestion des secrets sont importantes.

Nous retenons :

```text
Les variables d’environnement sont pratiques, mais elles ne sont pas magiquement secrètes.
```


**Permissions**

La lecture de `environ` peut échouer :

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

Erreur possible :

```text
Permission denied
```

Cela dépend :

- de l’utilisateur propriétaire du processus ;

- des permissions ;

- de la configuration du noyau ;

- des options de montage de `/proc` ;

- de mécanismes de sécurité comme Yama, AppArmor ou SELinux selon le système.


Nous devons interpréter cette erreur comme une protection normale, pas comme un dysfonctionnement.


## `cwd`, `exe` et `root`

**Présentation**

Dans `/proc/<PID>`, nous trouvons trois liens symboliques très importants :

```bash
ls -l /proc/$pid/cwd
ls -l /proc/$pid/exe
ls -l /proc/$pid/root
```

Ils nous renseignent sur le contexte d’exécution du processus.

|Lien|Signification|
|---|---|
|`cwd`|répertoire de travail courant|
|`exe`|exécutable associé au processus|
|`root`|racine du système de fichiers vue par le processus|


**`cwd` : répertoire courant**

Le lien `cwd` pointe vers le répertoire courant du processus.

Exemple :

```bash
ls -l /proc/$$/cwd
```

Sortie possible :

```text
/proc/18542/cwd -> /home/gregoire
```

Si nous changeons de répertoire :

```bash
cd /tmp
ls -l /proc/$$/cwd
```

Nous voyons que `cwd` change.

Cette information est utile lorsqu’un processus semble utiliser des chemins relatifs ou écrire dans un emplacement inattendu.


**`exe` : exécutable lancé**

Le lien `exe` pointe vers le binaire exécuté.

```bash
ls -l /proc/$pid/exe
```

Exemple :

```text
/proc/18590/exe -> /usr/bin/sleep
```

C’est très utile pour savoir quel binaire réel est exécuté.

Deux processus peuvent avoir le même nom court, mais ne pas venir du même chemin.

Exemple :

```text
/usr/bin/python3
/home/user/venv/bin/python
/opt/app/custom-python
```

Le lien `exe` aide à lever cette ambiguïté.


**Cas d’un exécutable supprimé**

Il arrive qu’un processus continue d’exécuter un binaire qui a été supprimé ou remplacé sur disque.

Nous pouvons alors voir :

```text
/proc/<PID>/exe -> /usr/bin/app (deleted)
```

Cela signifie que le processus utilise encore l’ancien exécutable chargé en mémoire.

Ce cas est fréquent après une mise à jour : un service doit être redémarré pour utiliser la nouvelle version du binaire.


**`root` : racine vue par le processus**

Le lien `root` pointe vers la racine du système de fichiers vue par le processus.

```bash
ls -l /proc/$pid/root
```

Sur un système normal, nous obtenons souvent :

```text
/proc/18590/root -> /
```

Mais dans un `chroot` ou un conteneur, ce lien peut donner une vue différente.

Ce lien est très important pour comprendre l’isolation.

Un processus peut croire que sa racine est `/`, alors que cette racine correspond en réalité à un sous-arbre spécifique du système hôte.


**Lien avec les conteneurs**

Dans un conteneur, `root` permet de comprendre quelle arborescence le processus voit.

Depuis l’hôte, si nous avons les permissions nécessaires, nous pouvons parfois inspecter :

```bash
ls /proc/<PID_CONTENEUR>/root
```

Cela donne accès à la racine du conteneur telle qu’elle est vue par le processus.

Cette capacité est puissante pour le diagnostic, mais elle doit être comprise du point de vue sécurité.


## `task` : les threads du processus

**Observer les threads**

Le dossier `task` contient les threads du processus.

```bash
ls /proc/$pid/task
```

Pour un processus simple comme `sleep`, nous obtenons généralement un seul identifiant :

```text
18590
```

Pour un processus multi-thread, nous pouvons obtenir plusieurs entrées :

```text
20001
20002
20003
20004
```

Chaque entrée correspond à un thread.


**Processus et threads dans Linux**

Linux représente les threads comme des tâches partageant certaines ressources.

Chaque thread possède un identifiant appelé TID.

Dans `/proc`, nous trouvons :

```text
/proc/<PID>/task/<TID>
```

Chaque thread possède son propre fichier `status`, `stat`, etc.

Exemple :

```bash
for t in /proc/$pid/task/*; do
    echo "Thread ${t##*/}"
    grep '^State:' "$t/status"
done
```


**Utilité**

Cette observation est utile pour :

- analyser une application multi-thread ;

- comprendre le nombre réel de threads ;

- observer des threads bloqués ;

- diagnostiquer une saturation ;

- vérifier le comportement d’un serveur Java, Node.js, Python, PostgreSQL ou autre.


Nous approfondissons cette dimension lorsque nous parlons du scheduling, des performances et de la mémoire.


## Lien entre `/proc` et `ps`

**Observer avec `ps`**

La commande `ps` affiche les processus.

Exemple :

```bash
ps -p $pid -o pid,ppid,state,comm,args
```

Sortie possible :

```text
    PID    PPID S COMMAND         COMMAND
  18590   18420 S sleep           sleep 1000
```

Nous pouvons retrouver ces informations dans `/proc`.

|Information `ps`|Source possible dans `/proc`|
|---|---|
|PID|nom du répertoire `/proc/<PID>`|
|PPID|`/proc/<PID>/status` ou `stat`|
|état|`/proc/<PID>/status` ou `stat`|
|commande|`/proc/<PID>/comm`|
|arguments|`/proc/<PID>/cmdline`|


**Reproduire une mini-version de `ps`**

Nous pouvons écrire une commande simple :

```bash
for dir in /proc/[0-9]*; do
    pid=${dir#/proc/}
    [ -r "$dir/status" ] || continue

    name=$(awk '/^Name:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    state=$(awk '/^State:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    ppid=$(awk '/^PPid:/ {print $2}' "$dir/status" 2>/dev/null) || continue

    printf "%8s %8s %2s %s\n" "$pid" "$ppid" "$state" "$name"
done | head
```

Sortie possible :

```text
       1        0  S systemd
     532        1  S systemd-journal
     610        1  S NetworkManager
    18542     1840 S bash
    18590    18542 S sleep
```

Nous venons de reproduire une version très simplifiée de `ps`.


**Pourquoi `ps` est plus robuste**

Notre script est pédagogique, mais `ps` est beaucoup plus robuste.

Il gère :

- les processus qui disparaissent pendant la lecture ;

- les formats précis ;

- les permissions ;

- les threads ;

- les options de tri ;

- les colonnes ;

- les différences de noyau ;

- les performances.


L’objectif n’est donc pas de remplacer `ps`, mais de comprendre d’où viennent ses informations.


## Lien entre `/proc`, `top` et `htop`

**`top`**

La commande :

```bash
top
```

affiche dynamiquement :

- les processus ;

- leur consommation CPU ;

- leur consommation mémoire ;

- leur état ;

- la charge système ;

- le nombre de tâches ;

- le swap ;

- la mémoire.


Une grande partie de ces informations provient de `/proc`, notamment :

```text
/proc/stat
/proc/meminfo
/proc/loadavg
/proc/<PID>/stat
/proc/<PID>/status
```


**`htop`**

`htop` offre une interface interactive plus confortable.

Il présente :

- une barre CPU ;

- une barre mémoire ;

- une liste des processus ;

- les arbres de processus ;

- les threads ;

- les commandes complètes ;

- les signaux envoyables depuis l’interface.


Mais conceptuellement, il reste lié aux informations exposées par le noyau, souvent via `/proc`.


**Observer les lectures avec `strace`**

Nous pouvons observer ce que lit une commande avec `strace`.

Exemple :

```bash
strace -e openat ps -p $pid -o pid,ppid,state,comm,args
```

Nous pouvons voir des ouvertures de fichiers sous `/proc`.

Cette observation montre que les outils haut niveau ne sont pas magiques : ils lisent, agrègent et formattent des informations.


## Cas pratiques

**Identifier précisément un processus**

Nous voulons savoir ce qu’est un processus.

Nous utilisons :

```bash
pid=1234

cat /proc/$pid/comm
tr '\0' ' ' < /proc/$pid/cmdline; echo
grep -E '^(Name|State|Pid|PPid|Uid|Gid|Threads):' /proc/$pid/status
ls -l /proc/$pid/exe
ls -l /proc/$pid/cwd
```

Nous obtenons :

- son nom court ;

- sa commande complète ;

- son état ;

- son parent ;

- son utilisateur ;

- son nombre de threads ;

- son exécutable ;

- son répertoire courant.


C’est une base de diagnostic très solide.


**Trouver le parent d’un processus**

Nous lisons :

```bash
grep '^PPid:' /proc/$pid/status
```

Exemple :

```text
PPid:   18420
```

Puis :

```bash
cat /proc/18420/comm
tr '\0' ' ' < /proc/18420/cmdline; echo
```

Nous comprenons alors qui a lancé le processus.

Nous pouvons aussi utiliser :

```bash
pstree -p
```

mais l’objectif est ici de comprendre la source dans `/proc`.


**Identifier un processus bloqué**

Nous cherchons les processus en état `D` :

```bash
for dir in /proc/[0-9]*; do
    [ -r "$dir/status" ] || continue
    state=$(awk '/^State:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    if [ "$state" = "D" ]; then
        pid=${dir#/proc/}
        name=$(awk '/^Name:/ {print $2}' "$dir/status" 2>/dev/null)
        echo "$pid $state $name"
    fi
done
```

Si nous en trouvons beaucoup, nous suspectons une attente I/O ou un problème noyau/périphérique.


**Voir les processus d’un utilisateur**

Nous pouvons lire les UID dans `status`.

Exemple :

```bash
for dir in /proc/[0-9]*; do
    [ -r "$dir/status" ] || continue
    pid=${dir#/proc/}
    uid=$(awk '/^Uid:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    name=$(awk '/^Name:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    printf "%8s %8s %s\n" "$pid" "$uid" "$name"
done | head
```

Le premier UID affiché est l’UID réel du processus.

Pour convertir un UID en nom d’utilisateur :

```bash
getent passwd 1000
```


## Pièges et limites

**Le processus peut disparaître**

Entre deux lectures, un processus peut se terminer.

Exemple :

```bash
cat /proc/$pid/status
```

peut échouer avec :

```text
No such file or directory
```

Cela signifie souvent que le processus n’existe plus.

Un script robuste doit l’accepter.


**Les permissions peuvent bloquer la lecture**

Certains fichiers peuvent être illisibles :

```bash
cat /proc/$pid/environ
```

peut produire :

```text
Permission denied
```

C’est normal si nous n’avons pas les droits nécessaires.

Nous devons distinguer :

- processus inexistant ;

- processus existant mais inaccessible ;

- fichier absent ;

- fichier lisible mais vide.



**Les conteneurs changent la vue**

Dans un conteneur, les PID visibles peuvent être différents de ceux de l’hôte.

Un processus peut avoir :

- un PID dans le conteneur ;

- un autre PID sur l’hôte.


Le répertoire `/proc` reflète le namespace PID du processus qui le lit.

Nous devons donc toujours demander :

```text
Depuis quel contexte observons-nous /proc ?
Depuis l’hôte ?
Depuis un conteneur ?
Depuis un chroot ?
Depuis un namespace particulier ?
```


**Ne pas confondre nom et exécutable**

Le nom affiché dans `comm` ne suffit pas toujours.

Un processus peut s’appeler :

```text
python
node
java
bash
```

mais ce nom ne dit pas quelle application est réellement lancée.

Nous devons regarder :

```bash
/proc/<PID>/cmdline
/proc/<PID>/exe
/proc/<PID>/cwd
```

pour comprendre précisément le processus.

# Chapitre 4 — Fichiers ouverts, descripteurs et ressources

## Notion de descripteur de fichier

**Qu’est-ce qu’un descripteur de fichier ?**

Un descripteur de fichier est un entier utilisé par un processus pour désigner une ressource ouverte.

Lorsqu’un programme ouvre un fichier avec un appel système comme :

```c
open()
```

le noyau retourne un entier.

Exemple conceptuel :

```text
open("/var/log/app.log") → 3
```

Le processus utilise ensuite ce nombre pour lire ou écrire :

```c
read(3, ...)
write(3, ...)
close(3)
```

Le processus ne manipule donc pas directement le chemin `/var/log/app.log` à chaque opération. Il manipule le descripteur `3`.


**Pourquoi cette abstraction est importante ?**

Cette abstraction permet au noyau d’unifier plusieurs types de ressources.

Le même mécanisme peut représenter :

```text
fichier classique
socket TCP
socket Unix
pipe
terminal
périphérique
pseudo-fichier
```

C’est une application directe de la philosophie Unix : beaucoup de ressources peuvent être manipulées comme des fichiers.

Ainsi, pour un processus, écrire dans un fichier, écrire dans un terminal ou écrire dans une socket repose sur une logique proche : il écrit dans un descripteur.


**Exemple simple**

Nous lançons un processus :

```bash
sleep 1000 &
pid=$!
```

Puis nous observons ses descripteurs :

```bash
ls -l /proc/$pid/fd
```

Sortie possible :

```text
total 0
lrwx------ 1 user user 64 mai 16 16:00 0 -> /dev/pts/2
lrwx------ 1 user user 64 mai 16 16:00 1 -> /dev/pts/2
lrwx------ 1 user user 64 mai 16 16:00 2 -> /dev/pts/2
```

Nous voyons trois descripteurs ouverts : `0`, `1` et `2`.


## Les descripteurs standards : `0`, `1`, `2`

**Présentation**

Par convention, chaque processus possède généralement trois descripteurs ouverts au démarrage :

|Descripteur|Nom|Rôle|
|--:|---|---|
|`0`|entrée standard|lire les données d’entrée|
|`1`|sortie standard|écrire la sortie normale|
|`2`|sortie d’erreur|écrire les messages d’erreur|

Nous les appelons souvent :

```text
stdin
stdout
stderr
```


**Entrée standard : `0`**

Le descripteur `0` correspond à l’entrée standard.

Pour un programme interactif, elle pointe souvent vers le terminal.

Exemple :

```bash
ls -l /proc/$$/fd/0
```

Sortie possible :

```text
0 -> /dev/pts/2
```

Cela signifie que notre shell lit son entrée depuis le terminal `/dev/pts/2`.


**Sortie standard : `1`**

Le descripteur `1` correspond à la sortie standard.

Exemple :

```bash
ls -l /proc/$$/fd/1
```

Sortie possible :

```text
1 -> /dev/pts/2
```

Lorsque nous exécutons :

```bash
echo "bonjour"
```

le texte est écrit dans le descripteur `1`.


**Sortie d’erreur : `2`**

Le descripteur `2` correspond à la sortie d’erreur.

Exemple :

```bash
ls -l /proc/$$/fd/2
```

Sortie possible :

```text
2 -> /dev/pts/2
```

Les erreurs sont écrites séparément de la sortie normale.

C’est ce qui permet de rediriger la sortie standard et la sortie d’erreur différemment.


**Redirections shell**

Nous pouvons observer l’effet des redirections.

Exemple :

```bash
bash -c 'echo $$; sleep 1000' > /tmp/sortie.txt 2> /tmp/erreur.txt &
pid=$!
```

Puis :

```bash
ls -l /proc/$pid/fd
```

Nous pouvons voir :

```text
0 -> /dev/pts/2
1 -> /tmp/sortie.txt
2 -> /tmp/erreur.txt
```

Nous comprenons alors que les redirections shell ne sont pas magiques : elles changent les fichiers associés aux descripteurs du processus.


## Le dossier `/proc/<PID>/fd`

**Rôle du dossier `fd`**

Le dossier :

```bash
/proc/<PID>/fd
```

contient un lien symbolique par descripteur ouvert.

Chaque lien porte le numéro du descripteur.

Exemple :

```bash
ls -l /proc/$pid/fd
```

Sortie possible :

```text
0 -> /dev/null
1 -> /var/log/app.log
2 -> /var/log/app-error.log
3 -> socket:[123456]
4 -> /tmp/data.txt
5 -> pipe:[654321]
```

Nous pouvons ainsi voir ce que le processus garde ouvert.


**Lire un lien de descripteur**

Pour lire la cible d’un descripteur :

```bash
readlink /proc/$pid/fd/1
```

Exemple :

```text
/var/log/app.log
```

Ou pour tous les descripteurs :

```bash
for fd in /proc/$pid/fd/*; do
    printf "%s -> %s\n" "${fd##*/}" "$(readlink "$fd")"
done
```

Cette commande affiche chaque numéro de descripteur avec sa cible.


**Permissions**

L’accès à `/proc/<PID>/fd` dépend des permissions.

Si le processus appartient à un autre utilisateur, nous pouvons rencontrer :

```text
Permission denied
```

ou ne pas voir toutes les informations.

Avec les droits administrateur, nous avons généralement une meilleure visibilité :

```bash
sudo ls -l /proc/<PID>/fd
```

Nous devons toutefois éviter d’utiliser `sudo` sans raison : l’objectif est de comprendre les permissions normales du système.


**Processus qui disparaît**

Comme toujours avec `/proc`, le processus peut se terminer pendant notre observation.

Une commande comme :

```bash
ls -l /proc/$pid/fd
```

peut échouer avec :

```text
No such file or directory
```

Cela signifie souvent que le processus n’existe plus.

Dans un script, nous devons gérer ce cas.


## Types de ressources visibles dans `fd`

**Fichiers classiques**

Un descripteur peut pointer vers un fichier classique :

```text
4 -> /home/user/data.txt
```

Cela signifie que le processus a ouvert ce fichier.

Il peut l’avoir ouvert :

- en lecture ;

- en écriture ;

- en lecture-écriture ;

- avec un verrou ;

- en mode append ;

- via une bibliothèque.


Le lien dans `/proc/<PID>/fd` ne suffit pas toujours à connaître le mode exact d’ouverture. Pour cela, nous pouvons regarder `fdinfo`.


**Sockets réseau**

Un descripteur peut pointer vers une socket :

```text
3 -> socket:[123456]
```

Cela signifie que le processus possède une socket ouverte.

Le nombre entre crochets est un identifiant interne, souvent un inode de socket.

Nous pouvons chercher cette socket dans `/proc/net/tcp`, `/proc/net/udp` ou avec des outils comme :

```bash
ss -p
```

ou :

```bash
sudo lsof -i -P -n
```

Nous détaillons davantage le réseau dans le chapitre 7.


**Pipes**

Un descripteur peut pointer vers un pipe :

```text
5 -> pipe:[654321]
```

Un pipe permet de faire communiquer deux processus.

Exemple :

```bash
cat fichier.txt | grep erreur
```

Ici, la sortie de `cat` est reliée à l’entrée de `grep` par un pipe.

Dans `/proc/<PID>/fd`, nous pouvons voir ces connexions sous forme de `pipe:[...]`.


**Terminaux**

Un descripteur peut pointer vers un terminal :

```text
0 -> /dev/pts/2
1 -> /dev/pts/2
2 -> /dev/pts/2
```

`/dev/pts/2` représente ici un pseudo-terminal, souvent utilisé par un terminal graphique ou une connexion SSH.

Cela signifie que le processus est attaché à une session interactive.


**Périphériques**

Un descripteur peut pointer vers un périphérique :

```text
0 -> /dev/null
1 -> /dev/null
2 -> /dev/null
```

ou :

```text
3 -> /dev/ttyUSB0
```

ou encore :

```text
4 -> /dev/random
```

Cela peut être très utile pour diagnostiquer des services, des programmes embarqués, des outils série ou des programmes qui lisent des sources pseudo-aléatoires.


**Fichiers anonymes et ressources spéciales**

Nous pouvons aussi voir des ressources particulières :

```text
anon_inode:[eventfd]
anon_inode:[timerfd]
anon_inode:[eventpoll]
```

Ces entrées correspondent à des objets noyau anonymes utilisés pour des mécanismes avancés :

- événements ;

- timers ;

- epoll ;

- signalfd ;

- io_uring selon les cas.


Nous ne les approfondissons pas ici, mais nous devons savoir les reconnaître.


## Le dossier `/proc/<PID>/fdinfo`

**Rôle de `fdinfo`**

À côté de `fd`, nous trouvons souvent :

```bash
/proc/<PID>/fdinfo
```

Ce dossier contient un fichier par descripteur.

Exemple :

```bash
ls /proc/$pid/fdinfo
```

Sortie possible :

```text
0  1  2  3
```

Pour lire les informations d’un descripteur :

```bash
cat /proc/$pid/fdinfo/3
```


**Exemple de contenu**

Sortie possible :

```text
pos:    128
flags:  0100002
mnt_id: 35
ino:    1234567
```

Nous trouvons notamment :

|Champ|Signification|
|---|---|
|`pos`|position courante dans le fichier|
|`flags`|flags d’ouverture|
|`mnt_id`|identifiant du montage|
|`ino`|inode associé|

Pour un fichier classique, `pos` indique où le processus se trouve dans le fichier.


**Utilité de `pos`**

Supposons qu’un processus lise progressivement un fichier.

Nous pouvons observer la position :

```bash
watch -n 1 "cat /proc/$pid/fdinfo/3"
```

Si `pos` augmente, le processus avance dans sa lecture ou son écriture.

Cela peut aider à savoir si un programme progresse ou reste bloqué.


**Interpréter les flags**

Les flags sont affichés sous forme numérique, souvent en octal.

Ils correspondent aux flags d’ouverture du fichier, par exemple :

- lecture seule ;

- écriture seule ;

- lecture-écriture ;

- append ;

- non-blocking ;

- close-on-exec.


L’interprétation exacte demande de connaître les constantes du noyau et de la libc.

Pour un diagnostic avancé, nous préférons souvent utiliser :

```bash
lsof
```

ou :

```bash
strace
```

mais `fdinfo` nous donne la source brute.


## Cas classique : fichier supprimé mais encore ouvert

**Le problème**

Un problème très fréquent en administration système est le suivant :

1. un fichier volumineux est supprimé ;

2. `ls` ne le montre plus ;

3. pourtant l’espace disque n’est pas libéré ;

4. le disque reste plein ;

5. un processus garde encore le fichier ouvert.


Cela arrive souvent avec des fichiers de logs.

Exemple :

```text
/var/log/app.log
```

est supprimé, mais le service continue d’écrire dedans.


**Pourquoi l’espace n’est pas libéré ?**

Sous Linux, supprimer un fichier avec :

```bash
rm fichier.log
```

supprime le nom du fichier dans le répertoire.

Mais si un processus garde le fichier ouvert, les données restent associées à l’inode tant que le descripteur n’est pas fermé.

Le fichier n’a plus de nom visible dans l’arborescence, mais il existe encore tant qu’un processus le référence.

Nous pouvons résumer :

```text
Nom supprimé ≠ données immédiatement libérées si le fichier reste ouvert.
```

L’espace est libéré quand le dernier descripteur vers ce fichier est fermé.


**Reproduire le cas en TP**

Nous créons un fichier :

```bash
dd if=/dev/zero of=/tmp/gros-fichier.log bs=1M count=100
```

Nous lançons un processus qui garde le fichier ouvert :

```bash
tail -f /tmp/gros-fichier.log &
pid=$!
```

Nous supprimons le fichier :

```bash
rm /tmp/gros-fichier.log
```

Le fichier n’apparaît plus :

```bash
ls -lh /tmp/gros-fichier.log
```

Mais le processus le garde ouvert.

Nous observons :

```bash
ls -l /proc/$pid/fd | grep deleted
```

Sortie possible :

```text
3 -> /tmp/gros-fichier.log (deleted)
```


**Retrouver les fichiers supprimés encore ouverts**

Pour chercher globalement :

```bash
sudo find /proc/*/fd -lname '*deleted*' 2>/dev/null
```

Ou avec affichage plus lisible :

```bash
sudo sh -c '
for fd in /proc/[0-9]*/fd/*; do
    target=$(readlink "$fd" 2>/dev/null) || continue
    case "$target" in
        *"(deleted)"*) echo "$fd -> $target" ;;
    esac
done
'
```

Nous pouvons ensuite identifier le PID à partir du chemin :

```text
/proc/1234/fd/3 -> /var/log/app.log (deleted)
```

Ici, le PID est `1234` et le descripteur est `3`.


**Libérer l’espace**

La solution propre consiste généralement à redémarrer ou recharger le service concerné.

Exemple :

```bash
sudo systemctl restart mon-service
```

ou :

```bash
sudo systemctl reload mon-service
```

selon le service.

Lorsque le processus ferme le descripteur, l’espace disque est libéré.

Dans certains cas, nous pouvons tronquer le descripteur directement, mais cette pratique doit être utilisée avec prudence :

```bash
sudo truncate -s 0 /proc/<PID>/fd/<FD>
```

Nous ne l’utilisons que si nous comprenons parfaitement ce que nous faisons, car nous modifions la ressource encore ouverte par le processus.


## Diagnostic : “quel fichier ce processus écrit-il ?”

**Observer les descripteurs**

Supposons qu’un processus écrive beaucoup sur disque.

Nous voulons identifier ses fichiers ouverts.

Nous commençons par :

```bash
ls -l /proc/<PID>/fd
```

Nous cherchons des fichiers classiques :

```text
1 -> /var/log/app.log
4 -> /data/output.dat
5 -> /tmp/cache.tmp
```


**Observer les positions avec `fdinfo`**

Pour un fichier donné, nous regardons :

```bash
cat /proc/<PID>/fdinfo/<FD>
```

Puis nous observons dans le temps :

```bash
watch -n 1 'cat /proc/<PID>/fdinfo/<FD>'
```

Si `pos` augmente rapidement, le processus lit ou écrit dans ce fichier.


**Croiser avec la taille du fichier**

Nous pouvons regarder la taille du fichier :

```bash
ls -lh /proc/<PID>/fd/<FD>
```

ou directement la cible :

```bash
readlink /proc/<PID>/fd/<FD>
```

Puis :

```bash
ls -lh "$(readlink /proc/<PID>/fd/<FD>)"
```

Attention : cela ne fonctionne pas si le fichier est supprimé ou si le chemin contient des cas particuliers.


## Diagnostic : “pourquoi mon service ne démarre-t-il pas ?”

**Descripteurs et logs**

Un service peut échouer parce qu’il ne peut pas ouvrir :

- un fichier de configuration ;

- un fichier de log ;

- une socket ;

- un certificat ;

- un périphérique ;

- un fichier temporaire.


Lorsque le service est encore actif, `/proc/<PID>/fd` peut nous montrer ce qu’il a réussi à ouvrir.

Exemple :

```bash
sudo ls -l /proc/<PID>/fd
```

Nous pouvons vérifier si le fichier attendu est bien ouvert.


**Complément avec `strace`**

Si le processus échoue immédiatement, `/proc/<PID>/fd` peut ne pas suffire, car le processus disparaît trop vite.

Dans ce cas, nous utilisons plutôt :

```bash
strace -e openat,close,read,write -f commande
```

`strace` permet d’observer les ouvertures de fichiers en temps réel.

Mais `/proc/<PID>/fd` reste utile pour les processus vivants.


## Diagnostic : trop de fichiers ouverts

**Symptôme**

Un service peut échouer avec une erreur du type :

```text
Too many open files
```

Cela signifie qu’il a atteint une limite de descripteurs ouverts.

Nous pouvons compter les descripteurs ouverts :

```bash
ls /proc/<PID>/fd | wc -l
```

Exemple :

```text
1024
```


**Lire les limites du processus**

Nous regardons :

```bash
cat /proc/<PID>/limits
```

Nous cherchons :

```text
Max open files
```

Exemple :

```text
Max open files            1024                 1048576              files
```

Nous interprétons :

- limite souple ;

- limite dure ;

- unité.


Nous étudierons `limits` plus en détail au chapitre suivant, mais il est déjà utile ici.


**Trouver une fuite de descripteurs**

Si le nombre de descripteurs augmente continuellement, nous suspectons une fuite de descripteurs.

Nous pouvons observer :

```bash
watch -n 1 "ls /proc/<PID>/fd | wc -l"
```

Si le nombre monte sans redescendre, l’application ouvre des fichiers, sockets ou pipes sans les fermer correctement.

Nous pouvons inspecter :

```bash
ls -l /proc/<PID>/fd
```

pour voir quel type de ressource s’accumule.

Exemples :

```text
socket:[...]
pipe:[...]
/tmp/...
```

Cela oriente le diagnostic.


## Sockets et correspondance avec le réseau

**Identifier une socket**

Dans `fd`, nous pouvons voir :

```text
7 -> socket:[4026532761]
```

Ce numéro correspond à une socket.

Pour savoir à quelle connexion elle correspond, nous utilisons généralement :

```bash
ss -tulpen
```

ou :

```bash
sudo ss -p
```


**Utiliser `lsof`**

Nous pouvons aussi utiliser :

```bash
sudo lsof -p <PID>
```

ou pour le réseau :

```bash
sudo lsof -Pan -p <PID> -i
```

Cela nous donne une vue plus lisible des sockets.

Mais il est important de comprendre que ces outils agrègent des informations que le noyau expose notamment via `/proc`.


## Comparaison avec `lsof`

**Rôle de `lsof`**

`lsof` signifie “list open files”.

Il liste les fichiers ouverts par les processus.

Exemple :

```bash
sudo lsof -p <PID>
```

Sortie possible :

```text
COMMAND PID USER FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   932 root cwd  DIR  259,2     4096  128 /
nginx   932 root txt  REG  259,2  1260032  456 /usr/sbin/nginx
nginx   932 root 1w   REG  259,2   123456  789 /var/log/nginx/access.log
nginx   932 root 2w   REG  259,2    23456  790 /var/log/nginx/error.log
nginx   932 root 6u  IPv4 123456      0t0  TCP *:80 (LISTEN)
```


**Ce que `lsof` apporte**

`lsof` apporte une lecture plus riche :

- nom de commande ;

- PID ;

- utilisateur ;

- numéro de descripteur ;

- mode d’ouverture ;

- type de ressource ;

- périphérique ;

- inode ;

- nom lisible ;

- sockets réseau interprétées.


`/proc/<PID>/fd` est plus brut, mais disponible sur presque tous les systèmes Linux sans installer d’outil supplémentaire.


**Quand utiliser `/proc` plutôt que `lsof` ?**

Nous utilisons directement `/proc` lorsque :

- `lsof` n’est pas installé ;

- nous voulons comprendre la source brute ;

- nous écrivons un script léger ;

- nous diagnostiquons dans un conteneur minimal ;

- nous voulons observer un descripteur précis ;

- nous voulons accéder à un fichier supprimé mais encore ouvert.


Nous utilisons `lsof` lorsque :

- nous voulons une vue synthétique ;

- nous voulons filtrer rapidement ;

- nous voulons interpréter les sockets ;

- nous voulons une sortie plus humaine ;

- nous faisons un diagnostic interactif.


Les deux approches sont complémentaires.


## Accéder au contenu via `/proc/<PID>/fd/<FD>`

**Principe**

Un lien comme :

```bash
/proc/<PID>/fd/3
```

peut parfois être utilisé comme chemin d’accès au fichier ouvert.

Exemple :

```bash
cat /proc/<PID>/fd/3
```

Cela lit le contenu du fichier associé au descripteur `3`, si nous avons les droits nécessaires.


**Cas du fichier supprimé**

C’est particulièrement utile lorsqu’un fichier a été supprimé mais reste ouvert.

Si nous avons :

```text
/proc/1234/fd/3 -> /tmp/data.log (deleted)
```

nous pouvons parfois récupérer son contenu :

```bash
cp /proc/1234/fd/3 /tmp/data-recupered.log
```

Cela peut sauver un fichier supprimé accidentellement, tant que le processus le garde ouvert.


**Précautions**

Nous devons être prudents.

Lire un descripteur peut avoir des effets selon le type de ressource :

- lire un pipe peut consommer les données ;

- lire une socket peut perturber le protocole ;

- lire un terminal peut interférer avec l’entrée ;

- écrire dans un descripteur peut modifier le comportement du processus.


Nous ne manipulons donc pas aveuglément `/proc/<PID>/fd/<FD>` en production.


## Redirections et héritage des descripteurs

**Héritage à la création d’un processus**

Lorsqu’un processus crée un enfant avec `fork()` puis `exec()`, les descripteurs ouverts peuvent être hérités.

C’est pourquoi un service peut garder ouvert un fichier ou une socket qui vient de son parent.

Exemple :

```bash
bash -c 'sleep 1000' > /tmp/out.txt &
pid=$!
ls -l /proc/$pid/fd
```

Nous voyons que `sleep` hérite de la sortie standard redirigée vers `/tmp/out.txt`.


**Close-on-exec**

Pour éviter d’hériter de descripteurs non nécessaires, les programmes peuvent utiliser le flag `close-on-exec`.

Ce flag indique que le descripteur doit être fermé lors d’un `exec()`.

En diagnostic, une accumulation de descripteurs hérités peut expliquer des comportements inattendus :

- fichier impossible à démonter ;

- socket gardée ouverte ;

- log supprimé mais non libéré ;

- service enfant gardant des ressources du parent.



## Relation entre fichiers ouverts et démontage

**Point de montage occupé**

Un montage peut refuser de se démonter :

```bash
sudo umount /mnt/data
```

Erreur possible :

```text
target is busy
```

Cela signifie souvent qu’un processus utilise encore un fichier ou un répertoire sur ce montage.


**Identifier les processus concernés**

Nous pouvons utiliser :

```bash
sudo lsof +f -- /mnt/data
```

ou explorer `/proc` :

```bash
sudo sh -c '
for fd in /proc/[0-9]*/fd/*; do
    target=$(readlink "$fd" 2>/dev/null) || continue
    case "$target" in
        /mnt/data/*) echo "$fd -> $target" ;;
    esac
done
'
```

Nous pouvons aussi vérifier les répertoires courants :

```bash
sudo ls -l /proc/[0-9]*/cwd 2>/dev/null | grep '/mnt/data'
```

Un processus peut empêcher le démontage simplement parce que son répertoire courant se trouve dans le point de montage.


## Cas pratique complet : disque plein à cause d’un log supprimé

**Situation**

Nous avons un serveur avec une partition pleine.

La commande :

```bash
df -h
```

indique que `/var` est à 100 %.

Nous cherchons les gros fichiers :

```bash
sudo du -h /var | sort -h | tail
```

Mais nous ne trouvons pas de fichier assez gros pour expliquer l’usage disque.

Nous suspectons alors un fichier supprimé mais encore ouvert.


**Recherche dans `/proc`**

Nous cherchons les descripteurs supprimés :

```bash
sudo sh -c '
for fd in /proc/[0-9]*/fd/*; do
    target=$(readlink "$fd" 2>/dev/null) || continue
    case "$target" in
        *"(deleted)"*) echo "$fd -> $target" ;;
    esac
done
'
```

Sortie possible :

```text
/proc/2451/fd/7 -> /var/log/app/app.log (deleted)
```

Nous identifions :

```text
PID = 2451
FD  = 7
fichier = /var/log/app/app.log
```


**Identifier le processus**

Nous lisons :

```bash
cat /proc/2451/comm
tr '\0' ' ' < /proc/2451/cmdline; echo
grep -E '^(Name|State|Pid|PPid|Uid):' /proc/2451/status
```

Nous trouvons par exemple :

```text
app-server
/usr/bin/app-server --config /etc/app/config.yml
```

Nous savons maintenant quel service garde le log ouvert.


**Correction**

La correction propre consiste à faire rouvrir les logs par le service.

Selon le service :

```bash
sudo systemctl reload app-server
```

ou :

```bash
sudo systemctl restart app-server
```

Puis nous vérifions :

```bash
df -h /var
```

et :

```bash
sudo ls -l /proc/2451/fd 2>/dev/null | grep deleted
```

Si le processus a été redémarré, son ancien PID n’existe plus et l’espace doit être libéré.


## Scripts utiles

**Lister les fichiers ouverts par un PID**

```bash
pid="$1"

if [ -z "$pid" ]; then
    echo "Usage: $0 <PID>" >&2
    exit 1
fi

if [ ! -d "/proc/$pid/fd" ]; then
    echo "Processus introuvable ou inaccessible : $pid" >&2
    exit 1
fi

for fd in /proc/$pid/fd/*; do
    target=$(readlink "$fd" 2>/dev/null) || continue
    printf "%5s -> %s\n" "${fd##*/}" "$target"
done
```

Nous pouvons sauvegarder ce script sous :

```bash
list-fd.sh
```

Puis :

```bash
chmod +x list-fd.sh
./list-fd.sh 1234
```


**Compter les descripteurs ouverts par processus**

```bash
for dir in /proc/[0-9]*; do
    pid=${dir#/proc/}
    [ -d "$dir/fd" ] || continue

    count=$(ls "$dir/fd" 2>/dev/null | wc -l) || continue
    name=$(cat "$dir/comm" 2>/dev/null || echo "?")

    printf "%8s %8s %s\n" "$count" "$pid" "$name"
done | sort -n | tail
```

Cette commande permet de trouver les processus ayant le plus de descripteurs ouverts.


**Trouver les fichiers supprimés encore ouverts**

```bash
sudo sh -c '
for fd in /proc/[0-9]*/fd/*; do
    target=$(readlink "$fd" 2>/dev/null) || continue
    case "$target" in
        *"(deleted)"*)
            pid=$(echo "$fd" | cut -d/ -f3)
            comm=$(cat "/proc/$pid/comm" 2>/dev/null || echo "?")
            echo "PID=$pid COMM=$comm FD=${fd##*/} TARGET=$target"
            ;;
    esac
done
'
```

Sortie possible :

```text
PID=2451 COMM=app-server FD=7 TARGET=/var/log/app/app.log (deleted)
```


## Pièges et limites

**Un lien `fd` n’est pas toujours un fichier classique**

Dans `/proc/<PID>/fd`, nous pouvons voir :

```text
socket:[...]
pipe:[...]
anon_inode:[...]
```

Nous ne devons pas supposer que chaque descripteur pointe vers un chemin de fichier classique.


**Lire un descripteur peut avoir un effet**

Accéder à :

```bash
/proc/<PID>/fd/<FD>
```

n’est pas toujours neutre.

Pour un fichier classique, lire est souvent sans danger.

Pour une socket, un pipe ou un terminal, lire peut consommer des données ou perturber le processus.

Nous devons donc être prudents.


**Les permissions peuvent masquer des informations**

Selon les droits, nous pouvons ne pas voir certains descripteurs.

C’est normal.

Nous devons éviter de conclure trop vite qu’un processus n’a rien ouvert si nous n’avons pas les permissions nécessaires.


**Les processus changent pendant l’observation**

Un processus peut ouvrir ou fermer des fichiers pendant que nous lisons `/proc/<PID>/fd`.

Un descripteur peut disparaître entre :

```bash
ls /proc/<PID>/fd
```

et :

```bash
readlink /proc/<PID>/fd/<FD>
```

Les scripts robustes doivent tolérer ces erreurs.

# Chapitre 5 — Comprendre la mémoire d’un processus

## Mémoire virtuelle et mémoire physique

**Pourquoi la mémoire d’un processus est difficile à comprendre ?**

Lorsque nous disons qu’un processus “consomme de la mémoire”, nous simplifions fortement la réalité.

Sous Linux, un processus ne voit pas directement la RAM physique. Il manipule un espace d’adressage virtuel.

Cet espace virtuel est ensuite traduit par le noyau et le matériel, notamment par la MMU, en pages mémoire physiques.

Nous devons donc distinguer plusieurs notions :

|Notion|Signification|
|---|---|
|Mémoire virtuelle|espace d’adressage vu par le processus|
|Mémoire résidente|partie réellement présente en RAM|
|Mémoire partagée|mémoire utilisée par plusieurs processus|
|Mémoire privée|mémoire propre à un processus|
|Mémoire anonyme|mémoire non directement associée à un fichier|
|Mémoire mappée|mémoire associée à un fichier ou à une ressource|

Cette distinction est essentielle pour éviter de mauvaises conclusions.


**Exemple classique : `VmSize` n’est pas la RAM consommée**

Dans `/proc/<PID>/status`, nous pouvons lire :

```bash
grep -E '^(VmSize|VmRSS):' /proc/<PID>/status
```

Exemple :

```text
VmSize:   1250000 kB
VmRSS:      85000 kB
```

Nous pourrions croire que le processus consomme 1,25 Go de RAM. Ce serait faux.

`VmSize` représente l’espace mémoire virtuel total du processus.

`VmRSS` représente la mémoire résidente, c’est-à-dire la partie actuellement présente en RAM.

Nous retenons :

```text
VmSize élevé ≠ consommation RAM élevée.
VmRSS est souvent plus proche de la mémoire réellement présente en RAM.
```

Mais même `VmRSS` doit être interprété avec prudence, car il inclut de la mémoire partagée.


## Préparer un processus d’exemple

**Lancer un processus simple**

Pour observer la mémoire, nous lançons un processus simple :

```bash
sleep 1000 &
pid=$!
echo "$pid"
```

Puis nous observons ses informations mémoire :

```bash
grep -E '^(Name|State|VmPeak|VmSize|VmRSS|RssAnon|RssFile|RssShmem):' /proc/$pid/status
```

Nous obtenons une première vue.

Cependant, `sleep` est très simple. Pour observer plus de mappings, nous pouvons utiliser un programme plus riche, par exemple notre shell :

```bash
pid=$$
```

ou un processus Python :

```bash
python3 -c 'import time; data = bytearray(100 * 1024 * 1024); time.sleep(1000)' &
pid=$!
```

Ce processus alloue environ 100 Mio de mémoire et reste actif.


**Nettoyage à la fin**

Lorsque nous lançons un processus de test en arrière-plan, nous devons penser à le terminer :

```bash
kill $pid
```

Si nécessaire :

```bash
kill -9 $pid
```

Nous évitons de laisser des processus de test tourner inutilement.


## `/proc/<PID>/maps` : cartographie mémoire

**Lire `maps`**

Le fichier principal pour comprendre l’organisation mémoire d’un processus est :

```bash
cat /proc/<PID>/maps
```

Exemple :

```bash
cat /proc/$pid/maps | head
```

Sortie possible :

```text
55b1f3b2d000-55b1f3b2f000 r--p 00000000 08:02 393324 /usr/bin/sleep
55b1f3b2f000-55b1f3b33000 r-xp 00002000 08:02 393324 /usr/bin/sleep
55b1f3b33000-55b1f3b35000 r--p 00006000 08:02 393324 /usr/bin/sleep
55b1f3b35000-55b1f3b36000 r--p 00007000 08:02 393324 /usr/bin/sleep
55b1f3b36000-55b1f3b37000 rw-p 00008000 08:02 393324 /usr/bin/sleep
55b1f4f4f000-55b1f4f70000 rw-p 00000000 00:00 0      [heap]
7f83d5c00000-7f83d5c21000 rw-p 00000000 00:00 0
7f83d5c21000-7f83d6000000 ---p 00000000 00:00 0
```

Chaque ligne décrit une zone mémoire du processus.


**Structure d’une ligne de `maps`**

Une ligne de `/proc/<PID>/maps` suit généralement cette structure :

```text
adresse-début-adresse-fin permissions offset périphérique inode chemin
```

Exemple :

```text
55b1f3b2f000-55b1f3b33000 r-xp 00002000 08:02 393324 /usr/bin/sleep
```

Nous pouvons découper :

|Champ|Exemple|Signification|
|---|---|---|
|Plage d’adresses|`55b1f3b2f000-55b1f3b33000`|zone mémoire virtuelle|
|Permissions|`r-xp`|droits de lecture, écriture, exécution, partage|
|Offset|`00002000`|décalage dans le fichier mappé|
|Périphérique|`08:02`|périphérique contenant le fichier|
|Inode|`393324`|inode du fichier|
|Chemin|`/usr/bin/sleep`|fichier ou zone spéciale|


**Plages d’adresses**

La première colonne indique la plage d’adresses virtuelles.

Exemple :

```text
55b1f3b2f000-55b1f3b33000
```

Cela signifie que la zone commence à l’adresse virtuelle :

```text
55b1f3b2f000
```

et se termine à :

```text
55b1f3b33000
```

Nous pouvons calculer la taille de cette zone :

```text
0x55b1f3b33000 - 0x55b1f3b2f000 = 0x4000
```

Soit 16 Kio.

Nous pouvons automatiser ce calcul, mais l’idée principale est que `maps` expose des plages d’adresses virtuelles, pas directement des adresses physiques.


**Permissions mémoire**

La deuxième colonne indique les permissions.

Exemples :

```text
r--p
r-xp
rw-p
---p
r--s
rw-s
```

Nous interprétons les caractères :

|Caractère|Signification|
|---|---|
|`r`|lecture autorisée|
|`w`|écriture autorisée|
|`x`|exécution autorisée|
|`-`|permission absente|
|`p`|mapping privé|
|`s`|mapping partagé|

Exemple :

```text
r-xp
```

signifie :

```text
lisible, exécutable, non modifiable, privé
```

C’est typiquement une zone de code.

Exemple :

```text
rw-p
```

signifie :

```text
lisible, modifiable, non exécutable, privé
```

C’est typiquement une zone de données ou de tas.


## Les zones mémoire importantes

**L’exécutable principal**

Dans `maps`, nous voyons souvent plusieurs lignes associées à l’exécutable.

Exemple :

```text
55b1f3b2d000-55b1f3b2f000 r--p ... /usr/bin/sleep
55b1f3b2f000-55b1f3b33000 r-xp ... /usr/bin/sleep
55b1f3b33000-55b1f3b35000 r--p ... /usr/bin/sleep
55b1f3b36000-55b1f3b37000 rw-p ... /usr/bin/sleep
```

Nous observons que le même fichier peut être mappé plusieurs fois avec des permissions différentes.

Cela correspond aux différentes sections du binaire :

- code exécutable ;

- données en lecture seule ;

- données modifiables ;

- métadonnées.


Nous pouvons faire le lien avec les formats exécutables comme ELF.


**Le tas : `[heap]`**

Le tas est une zone mémoire utilisée pour les allocations dynamiques.

Dans `maps`, il apparaît souvent ainsi :

```text
55b1f4f4f000-55b1f4f70000 rw-p 00000000 00:00 0 [heap]
```

Le tas est utilisé par des fonctions comme :

```c
malloc()
calloc()
realloc()
free()
```

En Python, JavaScript, Java, Ruby ou Go, le runtime gère lui-même les allocations, mais repose finalement sur des mécanismes mémoire du système.

Nous pouvons chercher le tas :

```bash
grep '\[heap\]' /proc/$pid/maps
```

Un processus peut avoir un tas plus ou moins grand selon son comportement.


**La pile : `[stack]`**

La pile du thread principal apparaît souvent ainsi :

```text
7ffcc7b4d000-7ffcc7b6e000 rw-p 00000000 00:00 0 [stack]
```

Nous pouvons la chercher :

```bash
grep '\[stack\]' /proc/$pid/maps
```

La pile contient notamment :

- les variables locales ;

- les adresses de retour ;

- les frames d’appel ;

- certaines données temporaires.


Chaque thread peut avoir sa propre pile.

Dans les processus multi-thread, nous pouvons observer les stacks par thread dans :

```bash
/proc/<PID>/task/<TID>/maps
```


**Bibliothèques partagées**

Les bibliothèques partagées apparaissent avec leur chemin.

Exemples :

```text
/usr/lib/x86_64-linux-gnu/libc.so.6
/usr/lib/x86_64-linux-gnu/libpthread.so.0
/usr/lib/x86_64-linux-gnu/libm.so.6
```

Nous pouvons les lister :

```bash
awk '{print $6}' /proc/$pid/maps 2>/dev/null | grep '^/' | sort -u
```

Cela nous permet de voir les bibliothèques chargées par le processus.

C’est utile pour diagnostiquer :

- une mauvaise version de bibliothèque ;

- un conflit de dépendances ;

- un binaire qui charge une bibliothèque inattendue ;

- une application qui utilise beaucoup de modules natifs.



**Mémoire anonyme**

Certaines lignes n’ont pas de chemin.

Exemple :

```text
7f83d5c00000-7f83d5c21000 rw-p 00000000 00:00 0
```

Nous parlons souvent de mémoire anonyme.

Elle n’est pas directement associée à un fichier.

Elle peut correspondre à :

- des allocations dynamiques ;

- des zones réservées par le runtime ;

- des piles de threads ;

- des buffers ;

- des zones internes de bibliothèques.


Une forte quantité de mémoire anonyme peut être normale pour certaines applications, mais elle peut aussi signaler une fuite mémoire.


**Mappings de fichiers**

Un processus peut mapper un fichier en mémoire.

Cela peut être fait avec l’appel système :

```c
mmap()
```

Un fichier mappé peut être lu comme s’il était en mémoire.

Cela sert notamment pour :

- charger un exécutable ;

- charger des bibliothèques partagées ;

- accéder efficacement à de gros fichiers ;

- partager de la mémoire entre processus ;

- implémenter certaines bases de données ;

- manipuler des fichiers volumineux.


Dans `maps`, ces mappings apparaissent avec le chemin du fichier.


## Comprendre `maps` par l’exemple

**Afficher une vue simplifiée**

Nous pouvons afficher les zones mémoire avec leurs permissions et chemins :

```bash
awk '{print $1, $2, $6}' /proc/$pid/maps | head -30
```

Attention : toutes les lignes n’ont pas forcément de sixième champ.

Nous pouvons écrire une version plus lisible :

```bash
awk '
{
  path = $6
  if (path == "") path = "[anonymous]"
  print $1, $2, path
}
' /proc/$pid/maps | head -30
```


**Lister les zones exécutables**

Les zones exécutables ont la permission `x`.

```bash
awk '$2 ~ /x/ { print }' /proc/$pid/maps
```

Nous y trouvons généralement :

- le binaire principal ;

- les bibliothèques partagées ;

- le linker dynamique ;

- parfois des zones JIT pour certains runtimes.


Pour des environnements comme JavaScript, Java, .NET ou certains moteurs de base de données, la présence de mémoire exécutable générée dynamiquement peut être normale.


**Lister les zones modifiables**

Les zones en écriture ont la permission `w`.

```bash
awk '$2 ~ /w/ { print }' /proc/$pid/maps | head
```

Nous y trouvons souvent :

- le tas ;

- la pile ;

- des zones anonymes ;

- des zones de données modifiables ;

- certaines zones partagées.



**Chercher les fichiers supprimés mappés**

Comme pour les descripteurs, un processus peut encore mapper un fichier supprimé.

Nous pouvons chercher :

```bash
grep deleted /proc/$pid/maps
```

Exemple :

```text
7f1234000000-7f1234200000 r-xp ... /usr/lib/libold.so (deleted)
```

Cela peut arriver après une mise à jour de bibliothèque.

Le processus continue d’utiliser l’ancienne version tant qu’il n’est pas redémarré.


## `/proc/<PID>/smaps` : statistiques détaillées par mapping

**Lire `smaps`**

Le fichier :

```bash
/proc/<PID>/smaps
```

donne des statistiques détaillées pour chaque zone mémoire.

Nous pouvons lire :

```bash
cat /proc/$pid/smaps | head -60
```

La sortie ressemble à `maps`, mais chaque mapping est suivi de nombreuses lignes.

Extrait :

```text
55b1f3b2f000-55b1f3b33000 r-xp 00002000 08:02 393324 /usr/bin/sleep
Size:                  16 kB
KernelPageSize:         4 kB
MMUPageSize:            4 kB
Rss:                   16 kB
Pss:                    4 kB
Shared_Clean:          16 kB
Shared_Dirty:           0 kB
Private_Clean:          0 kB
Private_Dirty:          0 kB
Referenced:            16 kB
Anonymous:              0 kB
Swap:                   0 kB
```

`smaps` est beaucoup plus détaillé que `maps`.


**Coût de lecture de `smaps`**

Lire `smaps` peut coûter plus cher que lire `maps`.

Sur un gros processus avec beaucoup de mappings, `smaps` peut être volumineux et demander un travail réel au noyau.

Nous évitons donc de lire `smaps` en boucle très fréquente en production sans raison.

Pour une synthèse plus légère, nous préférons parfois :

```bash
/proc/<PID>/smaps_rollup
```

que nous étudions ensuite.


**Champs importants de `smaps`**

Les champs les plus utiles sont :

|Champ|Signification|
|---|---|
|`Size`|taille virtuelle du mapping|
|`Rss`|mémoire résidente en RAM|
|`Pss`|part proportionnelle de mémoire|
|`Shared_Clean`|mémoire partagée propre|
|`Shared_Dirty`|mémoire partagée modifiée|
|`Private_Clean`|mémoire privée propre|
|`Private_Dirty`|mémoire privée modifiée|
|`Referenced`|mémoire récemment référencée|
|`Anonymous`|mémoire anonyme|
|`Swap`|mémoire de ce mapping partie en swap|

Nous n’avons pas besoin de tous les champs pour un premier diagnostic, mais nous devons comprendre les principaux.


## RSS, PSS, mémoire partagée et mémoire privée

**RSS : Resident Set Size**

Le RSS représente la mémoire du processus actuellement présente en RAM.

Dans `status`, nous avons :

```bash
grep '^VmRSS:' /proc/$pid/status
```

Dans `smaps`, chaque mapping a une ligne :

```text
Rss: 16 kB
```

Le RSS est utile, mais il peut être trompeur.

Pourquoi ? Parce qu’il compte la mémoire partagée dans chaque processus qui l’utilise.


**Exemple de limite du RSS**

Supposons qu’une bibliothèque partagée de 10 Mio soit utilisée par 20 processus.

Le RSS de chaque processus peut inclure ces 10 Mio.

Si nous additionnons naïvement les RSS des 20 processus, nous comptons potentiellement plusieurs fois la même mémoire.

Nous obtenons alors une surestimation.

C’est pourquoi le RSS est utile pour observer un processus, mais dangereux pour additionner la mémoire d’un ensemble de processus.


**PSS : Proportional Set Size**

Le PSS répartit la mémoire partagée entre les processus qui l’utilisent.

Si une page mémoire est partagée par 4 processus, chaque processus se voit attribuer un quart de cette page dans son PSS.

Cela donne une estimation plus juste de la contribution mémoire réelle d’un processus.

Dans `smaps`, nous voyons :

```text
Pss: 4 kB
```

Pour additionner la mémoire de plusieurs processus, le PSS est souvent plus pertinent que le RSS.


**Mémoire privée**

La mémoire privée est propre à un processus.

Dans `smaps`, elle apparaît notamment sous :

```text
Private_Clean
Private_Dirty
```

- `Private_Clean` : mémoire privée propre, potentiellement relisible depuis un fichier ;

- `Private_Dirty` : mémoire privée modifiée, réellement propre au processus.


La mémoire `Private_Dirty` est souvent intéressante pour détecter la mémoire réellement consommée par une application.


**Mémoire partagée**

La mémoire partagée apparaît sous :

```text
Shared_Clean
Shared_Dirty
```

Elle peut correspondre à :

- bibliothèques partagées ;

- pages partagées entre processus ;

- mémoire partagée explicite ;

- fichiers mappés partagés.


Une forte mémoire partagée n’est pas forcément problématique.

Au contraire, elle peut être un mécanisme d’efficacité.


## `/proc/<PID>/smaps_rollup` : synthèse mémoire

**Lire `smaps_rollup`**

Sur les noyaux modernes, nous pouvons lire :

```bash
cat /proc/$pid/smaps_rollup
```

Exemple :

```text
55b1f3b2d000-7ffcc7b6e000 ---p 00000000 00:00 0 [rollup]
Rss:                1216 kB
Pss:                 180 kB
Pss_Dirty:           104 kB
Shared_Clean:       1112 kB
Shared_Dirty:          0 kB
Private_Clean:         0 kB
Private_Dirty:       104 kB
Referenced:         1216 kB
Anonymous:           104 kB
Swap:                  0 kB
```

Ce fichier agrège les informations de `smaps` pour l’ensemble du processus.

Il est très pratique pour obtenir une vue synthétique.


**Extraire les champs clés**

Nous pouvons lire seulement quelques champs :

```bash
grep -E '^(Rss|Pss|Private_Dirty|Shared_Clean|Swap):' /proc/$pid/smaps_rollup
```

Exemple :

```text
Rss:                1216 kB
Pss:                 180 kB
Shared_Clean:       1112 kB
Private_Dirty:       104 kB
Swap:                  0 kB
```

Ces informations donnent une vision plus fine que `VmRSS`.


**Quand préférons-nous `smaps_rollup` ?**

Nous préférons `smaps_rollup` lorsque :

- nous voulons une synthèse mémoire du processus ;

- nous ne voulons pas analyser mapping par mapping ;

- nous voulons une estimation plus fine que `status` ;

- nous voulons observer `Pss` ;

- nous cherchons une contribution mémoire plus réaliste.


Nous préférons `smaps` lorsque :

- nous voulons comprendre quelle zone mémoire consomme ;

- nous cherchons une bibliothèque ou un mapping particulier ;

- nous analysons une fuite mémoire fine ;

- nous voulons distinguer précisément les zones anonymes, fichiers mappés, heap, stack.



## Calculer une synthèse mémoire avec `awk`

**Additionner les RSS dans `smaps`**

Nous pouvons additionner les lignes `Rss` :

```bash
awk '/^Rss:/ { total += $2 } END { printf "RSS total: %.2f MiB\n", total/1024 }' /proc/$pid/smaps
```

Cela doit être proche de `VmRSS`.


**Additionner les PSS**

```bash
awk '/^Pss:/ { total += $2 } END { printf "PSS total: %.2f MiB\n", total/1024 }' /proc/$pid/smaps
```

Le PSS est souvent plus intéressant pour comprendre la part réelle du processus.


**Additionner la mémoire privée dirty**

```bash
awk '/^Private_Dirty:/ { total += $2 } END { printf "Private_Dirty: %.2f MiB\n", total/1024 }' /proc/$pid/smaps
```

Cette valeur est utile pour savoir quelle mémoire modifiée appartient vraiment au processus.


**Regrouper par fichier mappé**

Nous pouvons produire une vue approximative par chemin :

```bash
awk '
/^[0-9a-f]+-[0-9a-f]+/ {
    path=$6
    if (path=="") path="[anonymous]"
}
/^Rss:/ {
    rss[path]+=$2
}
END {
    for (p in rss) {
        printf "%10.2f MiB  %s\n", rss[p]/1024, p
    }
}
' /proc/$pid/smaps | sort -n | tail
```

Cela permet d’identifier les chemins ou catégories qui contribuent le plus au RSS.

Nous restons prudents : ce script est pédagogique et peut être amélioré.


## `/proc/<PID>/limits` : limites de ressources

**Lire les limites**

Le fichier :

```bash
cat /proc/<PID>/limits
```

affiche les limites de ressources du processus.

Exemple :

```text
Limit                     Soft Limit           Hard Limit           Units
Max cpu time              unlimited            unlimited            seconds
Max file size             unlimited            unlimited            bytes
Max data size             unlimited            unlimited            bytes
Max stack size            8388608              unlimited            bytes
Max core file size        0                    unlimited            bytes
Max resident set          unlimited            unlimited            bytes
Max processes             30995                30995                processes
Max open files            1024                 1048576              files
Max locked memory         65536                65536                bytes
Max address space         unlimited            unlimited            bytes
```

Ces limites correspondent aux limites de ressources applicables au processus.


**Limite souple et limite dure**

Nous distinguons :

|Type|Signification|
|---|---|
|Soft limit|limite actuellement appliquée|
|Hard limit|limite maximale que le processus peut généralement se fixer sans privilèges supplémentaires|

Un processus peut souvent augmenter sa limite souple jusqu’à la limite dure.

Pour aller au-delà de la limite dure, il faut généralement des droits supplémentaires.


**Limites importantes pour la mémoire**

Plusieurs limites peuvent impacter la mémoire :

|Limite|Rôle|
|---|---|
|`Max data size`|taille maximale du segment de données|
|`Max stack size`|taille maximale de pile|
|`Max resident set`|limite RSS, rarement utilisée comme contrainte stricte moderne|
|`Max locked memory`|mémoire verrouillable en RAM|
|`Max address space`|taille maximale de l’espace d’adressage virtuel|

La limite la plus visible dans de nombreux cas est `Max stack size`.

Exemple :

```bash
grep 'Max stack size' /proc/$pid/limits
```

Sortie possible :

```text
Max stack size            8388608              unlimited            bytes
```

Cela correspond à une pile de 8 Mio par défaut dans beaucoup de configurations.


**Lien avec `ulimit`**

Dans le shell, nous pouvons afficher les limites avec :

```bash
ulimit -a
```

Les processus enfants héritent généralement des limites du shell ou du service qui les lance.

Exemple :

```bash
ulimit -n
```

affiche la limite du nombre de fichiers ouverts.

Nous faisons le lien avec le chapitre précédent :

```text
Une erreur “Too many open files” est souvent liée à la limite Max open files.
```


**Limites systemd**

Pour un service lancé par systemd, les limites peuvent être définies dans l’unité.

Exemples :

```ini
LimitNOFILE=65536
LimitNPROC=4096
LimitSTACK=8M
```

Après modification, nous rechargeons systemd :

```bash
sudo systemctl daemon-reload
sudo systemctl restart mon-service
```

Puis nous vérifions dans :

```bash
cat /proc/<PID>/limits
```

Nous ne nous contentons pas de lire le fichier de configuration : nous vérifions la limite réellement appliquée au processus.


## Diagnostiquer une fuite mémoire

**Symptôme**

Une fuite mémoire se manifeste souvent par une croissance continue de la mémoire consommée par un processus.

Nous pouvons observer :

```bash
watch -n 2 "grep -E '^(VmSize|VmRSS|RssAnon):' /proc/$pid/status"
```

Si `VmRSS` ou `RssAnon` augmente continuellement sans redescendre, nous suspectons une fuite ou une accumulation de données.


**Observer `smaps_rollup`**

Nous pouvons aussi observer :

```bash
watch -n 2 "grep -E '^(Rss|Pss|Private_Dirty|Anonymous|Swap):' /proc/$pid/smaps_rollup"
```

Les champs intéressants sont :

- `Rss` ;

- `Pss` ;

- `Private_Dirty` ;

- `Anonymous` ;

- `Swap`.


Si `Private_Dirty` et `Anonymous` augmentent continuellement, cela peut indiquer que le processus accumule de la mémoire propre.


**Différencier fuite et cache applicatif**

Une croissance mémoire n’est pas toujours une fuite.

Elle peut correspondre à :

- un cache applicatif volontaire ;

- un warm-up de runtime ;

- un JIT ;

- un pool de connexions ;

- un chargement de données ;

- une file d’attente interne ;

- une montée en charge normale.


Nous devons croiser avec :

- les logs applicatifs ;

- les métriques métier ;

- le trafic ;

- le nombre de requêtes ;

- les tailles de files ;

- les métriques du runtime.


`/proc` donne des symptômes, pas toujours la cause applicative exacte.


## Cas particulier des runtimes modernes

**Java**

Un processus Java peut réserver un espace mémoire virtuel important.

Nous pouvons voir un `VmSize` très élevé sans que la RAM soit réellement utilisée.

Il faut croiser avec :

```bash
jcmd
jstat
jmap
```

et les options de JVM :

```text
-Xms
-Xmx
-XX:MaxRAMPercentage
```

`/proc` nous donne la vue noyau, mais la JVM a sa propre gestion mémoire.


**Node.js**

Node.js utilise le moteur V8.

La mémoire observée dans `/proc` inclut :

- le heap JavaScript ;

- les buffers natifs ;

- les bibliothèques ;

- le code JIT ;

- les allocations C++ natives ;

- les modules natifs.


Nous ne pouvons pas conclure uniquement avec `VmRSS`.

Nous devons parfois compléter avec :

```bash
node --inspect
process.memoryUsage()
```

ou des dumps mémoire.


**Python**

Python peut conserver de la mémoire via son allocateur interne.

Après libération d’objets Python, la mémoire peut ne pas être immédiatement rendue au système.

Nous pouvons donc observer un RSS stable ou élevé même si l’application a libéré des objets.

Nous complétons avec :

- `tracemalloc` ;

- `objgraph` ;

- profils mémoire ;

- métriques applicatives.



**Go**

Go utilise son propre garbage collector.

Le runtime peut réserver de la mémoire, la réutiliser, et ne pas la rendre immédiatement au système.

Nous complétons avec :

- `pprof` ;

- métriques runtime ;

- `GOMEMLIMIT` ;

- `GOGC`.


Encore une fois, `/proc` est indispensable, mais il ne remplace pas les outils spécifiques du runtime.


## Mémoire et conteneurs

**Limite du point de vue `/proc`**

Dans un conteneur, `/proc/<PID>/status`, `maps` et `smaps` décrivent le processus.

Mais la limite mémoire réelle peut être imposée par les cgroups.

Un processus peut voir beaucoup de mémoire système dans `/proc/meminfo`, mais être limité à 512 Mio par son conteneur.

Nous devons donc croiser avec :

```bash
cat /proc/1/cgroup
```

et les fichiers de cgroups dans :

```bash
/sys/fs/cgroup
```


**OOM dans les conteneurs**

Lorsqu’un conteneur dépasse sa limite mémoire, il peut être tué par le mécanisme OOM.

Depuis l’application, nous pouvons voir :

- une croissance de RSS ;

- une pression mémoire ;

- puis une terminaison brutale.


Dans Kubernetes, cela apparaît souvent comme :

```text
OOMKilled
```

Pour diagnostiquer, nous croisons :

- `/proc/<PID>/status` ;

- `/proc/<PID>/smaps_rollup` ;

- les métriques cgroups ;

- les événements Kubernetes ;

- les logs du pod.



## Sécurité et mémoire

**Informations sensibles dans la mémoire**

La mémoire d’un processus peut contenir :

- mots de passe ;

- tokens ;

- clés privées ;

- secrets applicatifs ;

- données utilisateurs ;

- contenu de requêtes ;

- données temporaires.


Certains fichiers de `/proc` peuvent exposer des informations sensibles si les permissions sont trop ouvertes.


**`/proc/<PID>/mem`**

Il existe un fichier :

```bash
/proc/<PID>/mem
```

qui représente la mémoire du processus.

Nous ne l’utilisons pas dans ce cours comme outil de lecture directe.

Son accès est fortement contrôlé, car il permettrait de lire ou modifier la mémoire d’un processus.

Nous retenons simplement :

```text
/proc/<PID>/mem est très sensible et ne doit pas être manipulé sans cadre précis.
```


**Dumps mémoire**

Un core dump peut contenir toute la mémoire d’un processus au moment d’un crash.

Cela peut aider au debugging, mais cela peut aussi exposer des secrets.

La limite liée aux core dumps apparaît dans :

```bash
grep 'Max core file size' /proc/$pid/limits
```

Si la limite est `0`, les core dumps sont désactivés pour ce processus.

Nous devons traiter les dumps mémoire comme des données sensibles.


## Cas pratique complet : analyser la mémoire d’un processus

**Situation**

Nous avons un processus applicatif suspecté de consommer trop de mémoire.

Nous avons son PID :

```bash
pid=1234
```

Nous voulons établir un premier diagnostic.


**Identifier le processus**

Nous commençons par :

```bash
cat /proc/$pid/comm
tr '\0' ' ' < /proc/$pid/cmdline
echo
grep -E '^(Name|State|Pid|PPid|Threads):' /proc/$pid/status
```

Nous vérifions que nous analysons le bon processus.


**Lire la mémoire globale du processus**

Nous lisons :

```bash
grep -E '^(VmPeak|VmSize|VmRSS|RssAnon|RssFile|RssShmem):' /proc/$pid/status
```

Nous interprétons :

- `VmSize` : espace virtuel ;

- `VmRSS` : mémoire résidente ;

- `RssAnon` : mémoire anonyme ;

- `RssFile` : mémoire liée à des fichiers ;

- `RssShmem` : mémoire partagée.



**Lire la synthèse fine**

Si disponible :

```bash
grep -E '^(Rss|Pss|Private_Dirty|Shared_Clean|Anonymous|Swap):' /proc/$pid/smaps_rollup
```

Nous obtenons une vision plus fine.

Nous cherchons notamment :

- PSS élevé ;

- Private_Dirty élevé ;

- Anonymous élevé ;

- Swap non nul.



**Identifier les mappings importants**

Nous regardons les plus gros contributeurs :

```bash
awk '
/^[0-9a-f]+-[0-9a-f]+/ {
    path=$6
    if (path=="") path="[anonymous]"
}
/^Rss:/ {
    rss[path]+=$2
}
END {
    for (p in rss) {
        printf "%10.2f MiB  %s\n", rss[p]/1024, p
    }
}
' /proc/$pid/smaps | sort -n | tail -20
```

Nous cherchons :

- mémoire anonyme importante ;

- gros fichiers mappés ;

- bibliothèques inattendues ;

- segments heap importants ;

- fichiers supprimés encore mappés.



**Observer l’évolution**

Nous observons dans le temps :

```bash
watch -n 5 "grep -E '^(VmRSS|RssAnon):' /proc/$pid/status"
```

ou :

```bash
watch -n 5 "grep -E '^(Rss|Pss|Private_Dirty|Anonymous|Swap):' /proc/$pid/smaps_rollup"
```

Si la mémoire augmente continuellement, nous formulons une hypothèse de fuite ou d’accumulation.


**Formuler une conclusion**

Exemple de conclusion prudente :

```text
Nous observons que le processus possède un VmRSS élevé, mais qu’une partie importante est partagée. Le PSS est plus modéré. La mémoire privée dirty reste stable. Nous ne concluons donc pas à une fuite mémoire immédiate.
```

Autre exemple :

```text
Nous observons une croissance continue de RssAnon et Private_Dirty. Cette évolution suggère une accumulation de mémoire propre au processus. Nous devons compléter l’analyse avec les outils du runtime applicatif.
```


## Scripts utiles

**Synthèse mémoire d’un PID**

```bash
##!/usr/bin/env bash

pid="$1"

if [ -z "$pid" ]; then
    echo "Usage: $0 <PID>" >&2
    exit 1
fi

if [ ! -d "/proc/$pid" ]; then
    echo "Processus introuvable : $pid" >&2
    exit 1
fi

echo "Processus"
echo "---------"
cat "/proc/$pid/comm" 2>/dev/null || true
tr '\0' ' ' < "/proc/$pid/cmdline" 2>/dev/null
echo
echo

echo "Status mémoire"
echo "--------------"
grep -E '^(VmPeak|VmSize|VmRSS|RssAnon|RssFile|RssShmem):' "/proc/$pid/status" 2>/dev/null || true
echo

if [ -r "/proc/$pid/smaps_rollup" ]; then
    echo "smaps_rollup"
    echo "-------------"
    grep -E '^(Rss|Pss|Private_Dirty|Shared_Clean|Anonymous|Swap):' "/proc/$pid/smaps_rollup"
else
    echo "smaps_rollup indisponible"
fi
```

Nous pouvons le sauvegarder sous :

```bash
mem-summary.sh
```

Puis :

```bash
chmod +x mem-summary.sh
./mem-summary.sh 1234
```


**Surveiller la mémoire d’un processus**

```bash
watch -n 2 "grep -E '^(VmSize|VmRSS|RssAnon|RssFile|RssShmem):' /proc/<PID>/status"
```

Nous remplaçons `<PID>` par le PID réel.


**Afficher les bibliothèques chargées**

```bash
awk '{print $6}' /proc/<PID>/maps 2>/dev/null | grep '^/' | sort -u
```

Cette commande liste les fichiers mappés depuis le disque.


**Chercher les fichiers mappés supprimés**

```bash
grep deleted /proc/<PID>/maps
```

Si nous trouvons des bibliothèques ou fichiers supprimés, nous pouvons envisager de redémarrer le processus pour qu’il recharge les versions actuelles.


## Pièges et limites

**Ne pas confondre virtuel et physique**

Un très grand `VmSize` peut être normal.

Certains programmes réservent beaucoup d’espace virtuel sans utiliser réellement autant de RAM.

Nous regardons aussi :

- `VmRSS` ;

- `RssAnon` ;

- `Pss` ;

- `Private_Dirty`.



**Ne pas additionner naïvement les RSS**

Additionner les RSS de plusieurs processus peut surestimer la mémoire utilisée, car la mémoire partagée est comptée plusieurs fois.

Pour une estimation plus juste, nous préférons le PSS.


**`smaps` peut être coûteux**

Lire `smaps` trop fréquemment sur beaucoup de processus peut coûter cher.

Pour une supervision régulière, nous préférons des outils adaptés ou des métriques déjà agrégées.


**Les runtimes ont leur propre logique**

Java, Go, Python, Node.js et d’autres runtimes gèrent leur mémoire de manière spécifique.

`/proc` donne la vue du noyau, mais pas toujours la cause applicative.

Nous complétons avec les outils du langage.


**Les conteneurs ajoutent une couche**

Dans un conteneur, nous devons croiser `/proc` avec les cgroups.

Sinon, nous risquons de mal interpréter les limites mémoire.

# Chapitre 6 — Namespaces, cgroups, PSI, montages et réseau


## Namespaces vus depuis `/proc/<PID>/ns`

```bash
ls -l /proc/$$/ns
```

Nous pouvons obtenir des entrées telles que :

```text
cgroup
ipc
mnt
net
pid
pid_for_children
time
time_for_children
user
uts
```

Les liens utilisent une identité du type :

```text
net:[4026531840]
```

Deux processus dont les liens vers un type de namespace ont la même identité partagent cette instance.

Exemple :

```bash
readlink /proc/$$/ns/net
readlink /proc/1/ns/net
```

Pour rejoindre un namespace, l'outil privilégié est :

```bash
nsenter
```

Voir `[[Les namespaces Linux]]`.

---


## Cgroups vus depuis `/proc`

### `/proc/<PID>/cgroup`

```bash
cat /proc/$$/cgroup
```

Avec une hiérarchie cgroup v2 unifiée, une sortie typique est :

```text
0::/user.slice/user-1000.slice/session-2.scope
```

Ce fichier permet de déterminer **dans quel cgroup** se trouve le processus.

### Les limites ne sont pas dans `/proc/<PID>/cgroup`

Pour connaître les limites réelles :

```bash
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/pids.max
```

La distinction est donc :

```text
/proc/<PID>/cgroup    appartenance
/sys/fs/cgroup       contrôles + compteurs cgroup
```

---


## PSI : mesurer la pression sur les ressources

PSI est l'une des interfaces modernes les plus utiles de procfs.

Nous trouvons :

```bash
cat /proc/pressure/cpu
cat /proc/pressure/memory
cat /proc/pressure/io
```

Exemple :

```text
some avg10=1.20 avg60=0.42 avg300=0.10 total=123456
full avg10=0.00 avg60=0.00 avg300=0.00 total=789
```

### `some`

`some` mesure le temps durant lequel **au moins une tâche** est bloquée par manque de la ressource considérée.

### `full`

Pour les ressources qui l'exposent, `full` mesure les périodes où **toutes les tâches non-idle concernées** sont simultanément bloquées.

C'est un signal fort de contention.

### Pourquoi PSI est meilleur qu'un simple pourcentage

Exemple :

```text
CPU à 100 %
```

peut être parfaitement normal pour un batch parallèle.

À l'inverse :

```text
memory full élevé
```

montre directement une perte de temps liée à la pression mémoire.

PSI peut servir à :

- autoscaling ;
- admission control ;
- détection de saturation ;
- déclenchement d'actions avant OOM ;
- comparaison de workloads.

### PSI cgroup

Cgroup v2 peut également exposer la pression par cgroup sous `/sys/fs/cgroup`.

Pour un conteneur, cette vue est souvent plus pertinente que la pression globale de l'hôte.

---


## Montages : pourquoi préférer `mountinfo`

### `/proc/self/mounts`

```bash
cat /proc/self/mounts
```

fournit une vue compatible avec le format historique des tables de montage.

### `/proc/self/mountinfo`

Pour l'analyse moderne :

```bash
cat /proc/self/mountinfo
```

est généralement plus riche.

Nous y trouvons notamment :

- mount ID ;
- parent mount ID ;
- major:minor ;
- racine du montage ;
- point de montage ;
- options ;
- informations de propagation ;
- type du système de fichiers ;
- source.

Cela permet de comprendre les mount namespaces et les montages bind avec beaucoup plus de précision.

### Utiliser `findmnt`

Pour un outil humain :

```bash
findmnt
findmnt -T /chemin
```

est souvent préférable au parsing manuel.

---


## Réseau et `/proc/net`

### Vue par network namespace

Sur Linux moderne :

```text
/proc/net
```

reflète le network namespace du processus courant et est lié à la vue de `/proc/self/net`.

Ainsi, deux conteneurs peuvent obtenir un contenu différent pour :

```bash
cat /proc/net/dev
```

### Interfaces

```bash
cat /proc/net/dev
```

fournit des compteurs par interface.

Pour une interface utilisateur structurée :

```bash
ip -s link
```

### TCP/UDP

Des fichiers historiques existent :

```text
/proc/net/tcp
/proc/net/tcp6
/proc/net/udp
/proc/net/udp6
/proc/net/unix
```

Leur format est bas niveau.

Nous préférons généralement :

```bash
ss -lntup
```

qui utilise les interfaces noyau appropriées et fournit une présentation robuste.

### ARP et routes

Les fichiers :

```text
/proc/net/arp
/proc/net/route
```

existent toujours, mais `ip neigh` et `ip route` sont préférables pour l'administration moderne.

---


# Chapitre 7 — Paramètres noyau avec `/proc/sys` et `sysctl`

## Présentation générale de `/proc/sys`

**Une interface de configuration du noyau**

Le répertoire `/proc/sys` expose des paramètres internes du noyau Linux.

Nous pouvons le parcourir avec :

```bash
ls /proc/sys
```

Sortie possible :

```text
abi
debug
dev
fs
kernel
net
sunrpc
user
vm
```

Ces sous-répertoires organisent les paramètres par domaine.

Contrairement à beaucoup de fichiers de `/proc`, certains fichiers de `/proc/sys` sont modifiables. Quand nous écrivons dedans, nous ne modifions pas un fichier texte ordinaire : nous demandons au noyau de changer un paramètre actif.

Exemple :

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Si la valeur est :

```text
0
```

le routage IPv4 n’est pas activé.

Si nous écrivons :

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

nous activons temporairement le forwarding IPv4.


**Attention : nous agissons sur le noyau**

Nous devons être très prudents.

Lire un paramètre est généralement sans risque :

```bash
cat /proc/sys/vm/swappiness
```

Modifier un paramètre peut avoir des conséquences immédiates :

```bash
echo 10 | sudo tee /proc/sys/vm/swappiness
```

Selon le paramètre, nous pouvons affecter :

- le réseau ;

- la mémoire ;

- la sécurité ;

- les limites système ;

- le comportement des fichiers ;

- les performances ;

- la stabilité.


Nous retenons donc :

```text
/proc/sys est une interface de configuration active du noyau.
Nous ne modifions pas ces paramètres sans comprendre leur effet.
```


## Organisation de `/proc/sys`

**Les grands domaines**

Les répertoires principaux correspondent à de grands domaines du noyau.

|Répertoire|Rôle général|
|---|---|
|`/proc/sys/kernel`|paramètres généraux du noyau|
|`/proc/sys/vm`|gestion mémoire virtuelle|
|`/proc/sys/net`|pile réseau|
|`/proc/sys/fs`|systèmes de fichiers et limites associées|
|`/proc/sys/user`|limites liées aux namespaces et ressources utilisateur|
|`/proc/sys/debug`|paramètres de debug|
|`/proc/sys/dev`|certains périphériques|
|`/proc/sys/abi`|compatibilité ABI|
|`/proc/sys/sunrpc`|paramètres liés à SunRPC/NFS|

Nous pouvons explorer :

```bash
find /proc/sys -maxdepth 2 -type f | head
```

ou :

```bash
tree -L 2 /proc/sys 2>/dev/null
```

si `tree` est installé.


**Fichiers simples, effets complexes**

Beaucoup de paramètres sont représentés par des fichiers contenant une valeur simple.

Exemple :

```bash
cat /proc/sys/kernel/hostname
```

Sortie :

```text
machine-test
```

Autre exemple :

```bash
cat /proc/sys/vm/swappiness
```

Sortie :

```text
60
```

La valeur est simple, mais son interprétation peut être complexe.

Nous devons donc distinguer :

```text
facilité de lecture ≠ simplicité du comportement noyau
```


## Lecture des paramètres noyau

**Lire le nom d’hôte actif**

Nous pouvons lire le nom d’hôte actif :

```bash
cat /proc/sys/kernel/hostname
```

Ce nom correspond au hostname courant du noyau.

Nous pouvons comparer avec :

```bash
hostname
```

Le fichier `/proc/sys/kernel/hostname` donne l’état actif, tandis que des fichiers comme `/etc/hostname` servent à la configuration persistante selon la distribution.


**Lire la valeur de `swappiness`**

Le paramètre `swappiness` influence la tendance du noyau à utiliser le swap.

```bash
cat /proc/sys/vm/swappiness
```

Valeur possible :

```text
60
```

Une valeur plus basse tend à réduire l’usage du swap, sans l’interdire totalement.

Nous ne devons pas simplifier abusivement :

```text
swappiness faible ≠ swap désactivé
swappiness élevé ≠ système forcément lent
```

C’est un paramètre d’arbitrage, pas un interrupteur absolu.


**Lire le forwarding IPv4**

Nous lisons :

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Valeurs possibles :

```text
0
```

ou :

```text
1
```

Interprétation :

|Valeur|Signification|
|---|---|
|`0`|le noyau ne route pas les paquets IPv4 entre interfaces|
|`1`|le noyau peut router les paquets IPv4 entre interfaces|

Ce paramètre est important pour :

- routeurs Linux ;

- machines faisant du NAT ;

- VPN ;

- conteneurs ;

- Kubernetes ;

- passerelles réseau.



**Lire les limites de fichiers**

Nous pouvons lire :

```bash
cat /proc/sys/fs/file-max
```

Ce paramètre indique le nombre maximal de fichiers ouverts à l’échelle du système.

Nous pouvons aussi observer l’état courant avec :

```bash
cat /proc/sys/fs/file-nr
```

Sortie possible :

```text
12032   0   9223372036854775807
```

Selon le noyau, ces champs indiquent notamment le nombre de handles de fichiers alloués et la limite maximale.

Nous faisons le lien avec le chapitre 5 : un processus a ses propres descripteurs, mais le système possède aussi des limites globales.


## Correspondance entre `/proc/sys` et `sysctl`

**La commande `sysctl`**

La commande `sysctl` permet de lire et modifier les paramètres exposés via `/proc/sys`.

Exemple :

```bash
sysctl vm.swappiness
```

Sortie :

```text
vm.swappiness = 60
```

Ce paramètre correspond au fichier :

```bash
/proc/sys/vm/swappiness
```

La correspondance est simple :

```text
/proc/sys/vm/swappiness
↔
vm.swappiness
```

Les `/` deviennent des `.`.


**Exemples de correspondance**

|Fichier `/proc/sys`|Nom `sysctl`|
|---|---|
|`/proc/sys/kernel/hostname`|`kernel.hostname`|
|`/proc/sys/vm/swappiness`|`vm.swappiness`|
|`/proc/sys/net/ipv4/ip_forward`|`net.ipv4.ip_forward`|
|`/proc/sys/fs/file-max`|`fs.file-max`|
|`/proc/sys/kernel/pid_max`|`kernel.pid_max`|

Nous pouvons donc lire :

```bash
cat /proc/sys/kernel/pid_max
```

ou :

```bash
sysctl kernel.pid_max
```

Les deux donnent la même information sous une forme différente.


**Lister les paramètres disponibles**

Nous pouvons lister tous les paramètres connus par `sysctl` :

```bash
sysctl -a
```

Cette commande peut produire beaucoup de lignes.

Nous filtrons souvent avec `grep` :

```bash
sysctl -a | grep swappiness
sysctl -a | grep ip_forward
sysctl -a | grep '^net.ipv4'
```

Nous pouvons aussi explorer directement `/proc/sys`.


## Modifier temporairement un paramètre

**Modification par écriture directe**

Certains paramètres peuvent être modifiés en écrivant dans le fichier correspondant.

Exemple avec `ip_forward` :

```bash
cat /proc/sys/net/ipv4/ip_forward
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
cat /proc/sys/net/ipv4/ip_forward
```

Nous modifions ici l’état actif du noyau.

Cette modification est temporaire : elle est perdue au redémarrage, sauf si elle est aussi configurée de manière persistante.


**Modification avec `sysctl -w`**

La méthode équivalente avec `sysctl` est :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Sortie possible :

```text
net.ipv4.ip_forward = 1
```

Pour `swappiness` :

```bash
sudo sysctl -w vm.swappiness=10
```

Cela modifie la valeur active.


**Pourquoi préférer `sysctl` ?**

Nous pouvons modifier directement `/proc/sys`, mais `sysctl` présente plusieurs avantages :

- syntaxe standard ;

- affichage clair ;

- meilleure intégration avec les fichiers de configuration ;

- habitudes administrateur ;

- moins de risque d’écrire dans le mauvais fichier ;

- possibilité de charger une configuration complète.


Nous utilisons donc souvent `sysctl` en administration système.


## Rendre une configuration persistante

**Le problème des modifications temporaires**

Si nous faisons :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

la modification est active immédiatement.

Mais après redémarrage, elle peut disparaître.

Pour rendre une configuration persistante, nous devons l’écrire dans un fichier de configuration.


**`/etc/sysctl.conf`**

Historiquement, nous pouvons utiliser :

```bash
/etc/sysctl.conf
```

Exemple :

```conf
net.ipv4.ip_forward = 1
vm.swappiness = 10
```

Puis nous appliquons :

```bash
sudo sysctl -p
```

Cette commande charge par défaut `/etc/sysctl.conf`.


**`/etc/sysctl.d/*.conf`**

Sur les distributions modernes, nous utilisons souvent :

```bash
/etc/sysctl.d/
```

Exemple :

```bash
sudo nano /etc/sysctl.d/99-custom.conf
```

Contenu :

```conf
net.ipv4.ip_forward = 1
vm.swappiness = 10
```

Puis :

```bash
sudo sysctl --system
```

Cette commande charge les configurations depuis plusieurs emplacements, dont `/etc/sysctl.d`.

Nous préférons souvent créer un fichier dédié plutôt que modifier directement `/etc/sysctl.conf`, car c’est plus propre et plus maintenable.


**Vérifier la valeur réellement appliquée**

Après configuration, nous vérifions toujours :

```bash
sysctl net.ipv4.ip_forward
sysctl vm.swappiness
```

ou :

```bash
cat /proc/sys/net/ipv4/ip_forward
cat /proc/sys/vm/swappiness
```

Nous ne supposons pas que la configuration est appliquée : nous la vérifions.


## Domaine `kernel`

**Présentation**

Le répertoire :

```bash
/proc/sys/kernel
```

contient des paramètres généraux du noyau.

Nous pouvons lister :

```bash
ls /proc/sys/kernel | head
```

Exemples de paramètres :

```text
hostname
domainname
pid_max
threads-max
panic
randomize_va_space
dmesg_restrict
kptr_restrict
```


**`kernel.hostname`**

Nous lisons :

```bash
cat /proc/sys/kernel/hostname
```

ou :

```bash
sysctl kernel.hostname
```

Nous pouvons temporairement modifier le hostname actif :

```bash
echo "machine-test" | sudo tee /proc/sys/kernel/hostname
```

ou :

```bash
sudo sysctl -w kernel.hostname=machine-test
```

En pratique, pour modifier durablement le nom d’hôte, nous utilisons plutôt les outils de la distribution, par exemple :

```bash
sudo hostnamectl set-hostname machine-test
```


**`kernel.pid_max`**

Nous lisons :

```bash
cat /proc/sys/kernel/pid_max
```

Ce paramètre indique la valeur maximale des PID attribuables.

Exemple :

```text
4194304
```

Le noyau attribue des PID jusqu’à cette limite, puis réutilise des valeurs disponibles.

Nous ne modifions pas ce paramètre sans raison précise.


**`kernel.threads-max`**

Nous lisons :

```bash
cat /proc/sys/kernel/threads-max
```

Ce paramètre limite le nombre total de threads que le système peut créer.

Il est lié à la capacité mémoire et aux contraintes du noyau.

Nous faisons le lien avec :

```bash
cat /proc/<PID>/status | grep Threads
```

et avec les limites de processus utilisateur.


**`kernel.randomize_va_space`**

Nous lisons :

```bash
cat /proc/sys/kernel/randomize_va_space
```

Ce paramètre concerne l’ASLR, Address Space Layout Randomization.

Valeurs courantes :

|Valeur|Signification approximative|
|---|---|
|`0`|désactivation|
|`1`|randomisation partielle|
|`2`|randomisation plus complète|

L’ASLR est une mesure de sécurité qui rend plus difficile l’exploitation de certaines vulnérabilités mémoire.

Nous ne désactivons pas ce paramètre en production.


**`kernel.dmesg_restrict` et `kernel.kptr_restrict`**

Ces paramètres concernent l’exposition d’informations sensibles.

```bash
cat /proc/sys/kernel/dmesg_restrict
cat /proc/sys/kernel/kptr_restrict
```

Ils peuvent limiter :

- l’accès aux logs noyau ;

- l’exposition d’adresses noyau ;

- certaines informations utiles à un attaquant.


Nous les étudions aussi dans le chapitre sécurité.


## Domaine `vm`

**Présentation**

Le répertoire :

```bash
/proc/sys/vm
```

contient des paramètres liés à la mémoire virtuelle.

Nous pouvons lister :

```bash
ls /proc/sys/vm | head
```

Exemples :

```text
swappiness
dirty_ratio
dirty_background_ratio
overcommit_memory
overcommit_ratio
drop_caches
max_map_count
```

Ces paramètres peuvent avoir un impact important sur les performances.


**`vm.swappiness`**

Nous avons déjà vu :

```bash
cat /proc/sys/vm/swappiness
```

La valeur indique la tendance du noyau à déplacer des pages mémoire vers le swap.

Exemples de modification temporaire :

```bash
sudo sysctl -w vm.swappiness=10
```

ou :

```bash
echo 10 | sudo tee /proc/sys/vm/swappiness
```

Nous devons éviter les recettes universelles du type “mettre toujours 1” ou “mettre toujours 10”.

La bonne valeur dépend :

- du type de charge ;

- de la quantité de RAM ;

- de la présence de swap ;

- du type de stockage ;

- des exigences de latence ;

- du comportement applicatif.



**`vm.dirty_ratio` et `vm.dirty_background_ratio`**

Ces paramètres contrôlent le comportement des pages mémoire modifiées non encore écrites sur disque.

Nous lisons :

```bash
cat /proc/sys/vm/dirty_ratio
cat /proc/sys/vm/dirty_background_ratio
```

Idée générale :

- `dirty_background_ratio` : seuil à partir duquel le noyau commence à écrire en arrière-plan ;

- `dirty_ratio` : seuil plus élevé à partir duquel les processus peuvent être ralentis pour forcer l’écriture.


Ces paramètres peuvent influencer les performances d’écriture disque.

Nous les modifions avec prudence, surtout sur des serveurs de bases de données ou de stockage.


**`vm.overcommit_memory`**

Nous lisons :

```bash
cat /proc/sys/vm/overcommit_memory
```

Ce paramètre contrôle la stratégie d’allocation mémoire virtuelle.

Valeurs courantes :

|Valeur|Comportement|
|---|---|
|`0`|heuristique noyau|
|`1`|overcommit toujours autorisé|
|`2`|overcommit strict selon une limite calculée|

Ce paramètre est important pour certaines applications sensibles à l’allocation mémoire, notamment bases de données, calcul scientifique ou systèmes embarqués.


**`vm.max_map_count`**

Nous lisons :

```bash
cat /proc/sys/vm/max_map_count
```

Ce paramètre limite le nombre de mappings mémoire qu’un processus peut avoir.

Il est connu dans le monde Elasticsearch, OpenSearch et certaines bases de données, car ces systèmes peuvent nécessiter une valeur élevée.

Modification temporaire :

```bash
sudo sysctl -w vm.max_map_count=262144
```

Configuration persistante :

```conf
vm.max_map_count = 262144
```

dans un fichier de `/etc/sysctl.d/`.


**`vm.drop_caches`**

Le fichier :

```bash
/proc/sys/vm/drop_caches
```

permet de demander au noyau de libérer certains caches.

Exemple souvent vu :

```bash
sync
echo 3 | sudo tee /proc/sys/vm/drop_caches
```

Nous devons être très prudents avec ce paramètre.

Il ne sert pas à “réparer” la mémoire dans un usage normal. Linux utilise le cache pour améliorer les performances. Vider les caches peut au contraire ralentir le système.

Nous l’utilisons uniquement dans des contextes de test ou de diagnostic précis.


## Domaine `net`

**Présentation**

Le répertoire :

```bash
/proc/sys/net
```

contient les paramètres de la pile réseau.

Nous pouvons explorer :

```bash
ls /proc/sys/net
```

Sortie possible :

```text
core
ipv4
ipv6
netfilter
unix
```

Les paramètres réseau sont nombreux et sensibles.


**`net.ipv4.ip_forward`**

Nous lisons :

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Modification temporaire :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Configuration persistante :

```conf
net.ipv4.ip_forward = 1
```

Ce paramètre est nécessaire si la machine doit router des paquets IPv4 entre interfaces.

Exemples :

- passerelle Linux ;

- routeur ;

- serveur VPN ;

- NAT ;

- certaines configurations Docker ou Kubernetes.



**Paramètres ICMP**

Exemple :

```bash
cat /proc/sys/net/ipv4/icmp_echo_ignore_all
```

Si nous mettons :

```bash
sudo sysctl -w net.ipv4.icmp_echo_ignore_all=1
```

la machine ignore les requêtes ping ICMP echo.

Nous ne le faisons pas sans raison, car `ping` est utile pour le diagnostic.


**Paramètres TCP**

Nous trouvons de nombreux paramètres TCP :

```bash
ls /proc/sys/net/ipv4/tcp_*
```

Exemples :

```text
tcp_fin_timeout
tcp_keepalive_time
tcp_syncookies
tcp_max_syn_backlog
```

Ces paramètres peuvent influencer :

- la résistance à certaines attaques ;

- la gestion des connexions ;

- les timeouts ;

- les performances réseau ;

- le comportement serveur sous forte charge.


Nous ne modifions pas ces paramètres avec des recettes copiées sans analyse.


**`net.ipv4.tcp_syncookies`**

Nous lisons :

```bash
cat /proc/sys/net/ipv4/tcp_syncookies
```

Les SYN cookies sont une protection contre certaines attaques SYN flood.

Valeurs courantes :

```text
0
1
```

Nous gardons généralement cette protection activée sauf cas très particulier.


**IPv6**

Les paramètres IPv6 se trouvent dans :

```bash
/proc/sys/net/ipv6
```

Exemple :

```bash
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
```

Nous pouvons aussi voir des paramètres par interface :

```bash
ls /proc/sys/net/ipv6/conf
```

Sortie possible :

```text
all
default
eth0
lo
```

Cela signifie que certaines options peuvent être globales ou spécifiques à une interface.


## Domaine `fs`

**Présentation**

Le répertoire :

```bash
/proc/sys/fs
```

contient des paramètres liés aux fichiers, aux inodes, aux descripteurs, aux montages et à certains comportements de systèmes de fichiers.

Nous pouvons lister :

```bash
ls /proc/sys/fs | head
```

Exemples :

```text
file-max
file-nr
inode-nr
inotify
protected_hardlinks
protected_symlinks
```


**`fs.file-max`**

Nous lisons :

```bash
cat /proc/sys/fs/file-max
```

Ce paramètre indique la limite globale du nombre de fichiers ouverts au niveau du système.

Nous le distinguons de :

```bash
cat /proc/<PID>/limits
```

qui donne les limites d’un processus particulier.


**`fs.file-nr`**

Nous lisons :

```bash
cat /proc/sys/fs/file-nr
```

Sortie possible :

```text
12032   0   9223372036854775807
```

Ce fichier donne des informations sur les handles de fichiers au niveau système.

Nous l’utilisons pour détecter une pression globale sur les fichiers ouverts.


**Inotify**

Les paramètres inotify sont dans :

```bash
/proc/sys/fs/inotify
```

Exemples :

```bash
cat /proc/sys/fs/inotify/max_user_watches
cat /proc/sys/fs/inotify/max_user_instances
cat /proc/sys/fs/inotify/max_queued_events
```

Ces paramètres sont importants pour :

- IDE ;

- outils de développement ;

- watchers JavaScript ;

- Docker bind mounts ;

- synchronisation de fichiers ;

- outils comme `webpack`, `vite`, `watchexec`.


Si un outil dit :

```text
ENOSPC: System limit for number of file watchers reached
```

nous devons souvent regarder :

```bash
cat /proc/sys/fs/inotify/max_user_watches
```

Modification temporaire :

```bash
sudo sysctl -w fs.inotify.max_user_watches=524288
```

Configuration persistante :

```conf
fs.inotify.max_user_watches = 524288
```


**`protected_symlinks` et `protected_hardlinks`**

Nous lisons :

```bash
cat /proc/sys/fs/protected_symlinks
cat /proc/sys/fs/protected_hardlinks
```

Ces paramètres renforcent la sécurité contre certaines attaques utilisant des liens symboliques ou hard links, notamment dans des répertoires partagés comme `/tmp`.

Nous les laissons généralement activés.


## Domaine `user`

**Présentation**

Le répertoire :

```bash
/proc/sys/user
```

contient des limites liées aux namespaces et ressources utilisateur.

Nous pouvons lister :

```bash
ls /proc/sys/user
```

Exemples possibles :

```text
max_user_namespaces
max_pid_namespaces
max_net_namespaces
max_mnt_namespaces
```

Ces paramètres sont importants pour les conteneurs et les mécanismes d’isolation.


**`user.max_user_namespaces`**

Nous lisons :

```bash
cat /proc/sys/user/max_user_namespaces
```

Ce paramètre limite le nombre de namespaces utilisateur.

Il peut influencer :

- Docker rootless ;

- Podman ;

- certains sandboxes ;

- outils d’isolation ;

- navigateurs ;

- environnements de build.


Une valeur trop basse peut empêcher certains outils de fonctionner.

Une valeur trop permissive peut augmenter la surface d’attaque si le système n’est pas correctement durci.


## Domaine `debug`

**Présentation**

Le répertoire :

```bash
/proc/sys/debug
```

expose certains paramètres de debug selon le noyau et la configuration.

Tous les systèmes n’ont pas les mêmes entrées.

Nous l’explorons avec :

```bash
ls /proc/sys/debug
```

Nous ne modifions généralement pas ces paramètres dans un cours introductif sans objectif précis.


## Permissions et erreurs fréquentes

**Permission refusée**

Si nous essayons de modifier un paramètre sans privilèges :

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

nous pouvons obtenir :

```text
Permission denied
```

Même avec `sudo`, cette forme peut échouer :

```bash
sudo echo 1 > /proc/sys/net/ipv4/ip_forward
```

Pourquoi ? Parce que la redirection `>` est effectuée par le shell courant, avant ou en dehors de l’effet attendu de `sudo`.


**Utiliser `tee`**

La bonne forme est :

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

Ici, `tee` s’exécute avec les droits administrateur et écrit dans le fichier.

Nous pouvons masquer l’affichage si nécessaire :

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward > /dev/null
```


**Utiliser `sysctl -w`**

La forme recommandée est souvent :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Elle évite le piège de la redirection shell.


**Fichier en lecture seule**

Certains paramètres ne sont pas modifiables.

Nous pouvons voir des permissions en lecture seule :

```bash
ls -l /proc/sys/kernel/ostype
```

Si nous essayons d’écrire dedans, le noyau refuse.

Ce n’est pas une erreur du système : tous les paramètres exposés ne sont pas configurables.


## Cas pratique 1 : activer temporairement le routage IPv4

**Situation**

Nous voulons transformer temporairement une machine Linux en routeur IPv4 entre deux interfaces.

Nous vérifions d’abord :

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Si la valeur est :

```text
0
```

le forwarding est désactivé.


**Activation temporaire**

Nous activons :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

ou :

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

Nous vérifions :

```bash
cat /proc/sys/net/ipv4/ip_forward
```


**Persistance**

Nous créons :

```bash
sudo nano /etc/sysctl.d/99-routing.conf
```

Contenu :

```conf
net.ipv4.ip_forward = 1
```

Puis :

```bash
sudo sysctl --system
```

Nous vérifions :

```bash
sysctl net.ipv4.ip_forward
```


**Attention : forwarding ne suffit pas**

Activer `ip_forward` ne suffit pas toujours.

Pour faire réellement du routage ou du NAT, nous devons aussi configurer :

- les routes ;

- le pare-feu ;

- éventuellement NAT avec `nftables` ou `iptables` ;

- les règles de filtrage ;

- les interfaces réseau.


Nous retenons donc :

```text
ip_forward autorise le noyau à router.
Il ne configure pas automatiquement toute la politique réseau.
```


## Cas pratique 2 : corriger une limite inotify

**Situation**

Nous lançons un outil de développement, par exemple un serveur Vite, Webpack, un IDE ou un watcher, et nous obtenons une erreur :

```text
ENOSPC: System limit for number of file watchers reached
```

Nous suspectons une limite inotify.


**Lire la valeur actuelle**

Nous lisons :

```bash
cat /proc/sys/fs/inotify/max_user_watches
```

ou :

```bash
sysctl fs.inotify.max_user_watches
```

Exemple :

```text
fs.inotify.max_user_watches = 8192
```

Cette valeur peut être trop basse pour de gros projets.


**Modification temporaire**

Nous pouvons tester :

```bash
sudo sysctl -w fs.inotify.max_user_watches=524288
```

Nous relançons ensuite l’outil concerné.


**Configuration persistante**

Nous créons :

```bash
sudo nano /etc/sysctl.d/99-inotify.conf
```

Contenu :

```conf
fs.inotify.max_user_watches = 524288
```

Puis :

```bash
sudo sysctl --system
```

Nous vérifions :

```bash
sysctl fs.inotify.max_user_watches
```


## Cas pratique 3 : observer sans casser la mémoire

**Mauvaise pratique fréquente**

Nous voyons parfois cette commande proposée :

```bash
sync
echo 3 | sudo tee /proc/sys/vm/drop_caches
```

Elle force le noyau à libérer certains caches.

Le problème est que beaucoup d’utilisateurs interprètent mal le cache Linux.

Ils voient une mémoire “utilisée” élevée et pensent qu’il faut “nettoyer” la RAM.

En réalité, le cache est utile : il accélère les accès disque et peut être libéré si une application a besoin de mémoire.


**Bonne lecture**

Nous regardons plutôt :

```bash
free -h
grep -E '^(MemTotal|MemFree|MemAvailable|Buffers|Cached):' /proc/meminfo
```

Nous privilégions `MemAvailable` pour comprendre la mémoire réellement disponible.

Nous ne vidons pas les caches pour “améliorer” le système.


**Quand utiliser `drop_caches` ?**

Nous pouvons l’utiliser dans des contextes de benchmark ou de test contrôlé, par exemple pour mesurer des lectures disque sans effet du cache.

Mais nous ne l’utilisons pas comme routine d’administration.


## Bonnes pratiques

**Lire avant de modifier**

Avant toute modification, nous lisons la valeur actuelle :

```bash
sysctl vm.swappiness
```

ou :

```bash
cat /proc/sys/vm/swappiness
```

Nous gardons une trace de la valeur initiale.


**Modifier temporairement avant de rendre persistant**

Nous testons d’abord temporairement :

```bash
sudo sysctl -w parametre=valeur
```

Puis nous observons les effets.

Si le comportement est correct, nous rendons la configuration persistante dans `/etc/sysctl.d`.


**Documenter la raison**

Dans un fichier de configuration, nous ajoutons un commentaire.

Exemple :

```conf
## Augmente la limite inotify pour les projets de développement avec beaucoup de fichiers.
fs.inotify.max_user_watches = 524288
```

Un futur administrateur doit comprendre pourquoi la valeur a été changée.


**Éviter les recettes universelles**

Nous évitons :

```text
Mets swappiness à 1, c’est toujours mieux.
Désactive IPv6, ça règle tout.
Vide les caches, Linux ira plus vite.
Augmente tous les buffers TCP au maximum.
```

Ces recettes peuvent dégrader le système.

Nous modifions un paramètre parce que nous avons :

- un problème identifié ;

- une hypothèse ;

- une mesure avant ;

- un test ;

- une mesure après ;

- une documentation.

# Chapitre 8 — Sécurité, permissions et isolation

## Pourquoi `/proc` pose des questions de sécurité

**Une interface très informative**

`/proc` est précieux parce qu’il rend visibles de nombreuses informations internes du système.

Nous pouvons y trouver :

```text
processus actifs
lignes de commande
variables d’environnement
fichiers ouverts
sockets réseau
mappings mémoire
paramètres noyau
statistiques système
informations CPU
informations mémoire
```

Cette visibilité est très utile pour l’administration système.

Mais du point de vue sécurité, une information utile à l’administrateur peut aussi être utile à un attaquant.


**Le problème de l’énumération**

Sur un système mal protégé, un utilisateur local peut parfois observer :

```bash
ps aux
ls /proc
cat /proc/<PID>/cmdline
tr '\0' '\n' < /proc/<PID>/environ
ls -l /proc/<PID>/fd
```

Cela peut lui permettre de découvrir :

- quels services tournent ;

- quels utilisateurs sont connectés ;

- quelles applications sont lancées ;

- quels fichiers sont ouverts ;

- quels ports ou sockets sont utilisés ;

- quels chemins applicatifs sont présents ;

- parfois des secrets mal protégés.


Nous comprenons donc que `/proc` peut faciliter la reconnaissance locale.


## Informations sensibles exposées par `/proc`

**La ligne de commande : `cmdline`**

Le fichier :

```bash
/proc/<PID>/cmdline
```

expose la ligne de commande utilisée pour lancer un processus.

Nous l’affichons de manière lisible avec :

```bash
tr '\0' ' ' < /proc/<PID>/cmdline
echo
```

Le problème apparaît lorsque des secrets sont passés en arguments.

Exemples de mauvaises pratiques :

```bash
mysql -u admin -pMonMotDePasse
curl -H "Authorization: Bearer TOKEN_SECRET" https://api.example.org
python app.py --password secret
backup --s3-secret-key ABCDEF
```

Ces informations peuvent être visibles dans `/proc/<PID>/cmdline`, mais aussi dans des outils comme `ps`.

Nous retenons :

```text
Nous ne passons pas de secrets en arguments de ligne de commande.
```


**Les variables d’environnement : `environ`**

Le fichier :

```bash
/proc/<PID>/environ
```

contient les variables d’environnement du processus.

Nous les lisons avec :

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

Les variables d’environnement peuvent contenir :

```text
DATABASE_URL
POSTGRES_PASSWORD
MYSQL_PASSWORD
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
GITHUB_TOKEN
JWT_SECRET
API_KEY
```

Elles sont pratiques, mais elles ne sont pas automatiquement secrètes.

Si les permissions permettent leur lecture, un autre utilisateur ou un processus compromis peut récupérer ces valeurs.

Nous retenons :

```text
Les variables d’environnement sont une convention pratique de configuration.
Elles ne remplacent pas un vrai mécanisme de gestion de secrets.
```


**Les fichiers ouverts : `fd`**

Le dossier :

```bash
/proc/<PID>/fd
```

expose les descripteurs ouverts.

Il peut révéler :

- fichiers de configuration ;

- fichiers temporaires ;

- logs ;

- sockets ;

- bases SQLite ;

- fichiers supprimés mais encore ouverts ;

- répertoires de travail ;

- chemins internes d’une application.


Exemple :

```bash
ls -l /proc/<PID>/fd
```

Sortie possible :

```text
0 -> /dev/null
1 -> /var/log/app/access.log
2 -> /var/log/app/error.log
3 -> /etc/app/secret.conf
4 -> socket:[123456]
5 -> /tmp/session-data
```

Cela peut donner beaucoup d’informations sur l’architecture d’un service.


**La cartographie mémoire : `maps`**

Le fichier :

```bash
/proc/<PID>/maps
```

expose les mappings mémoire du processus.

Il peut révéler :

- bibliothèques chargées ;

- chemins exacts des binaires ;

- versions de bibliothèques ;

- présence de modules natifs ;

- fichiers mappés ;

- exécutable supprimé mais encore chargé ;

- structure générale de l’espace mémoire.


Exemple :

```bash
cat /proc/<PID>/maps | head
```

Du point de vue d’un attaquant, ces informations peuvent aider à comprendre l’environnement d’exécution.

Les protections modernes, comme l’ASLR, limitent l’exploitation directe, mais l’exposition d’informations mémoire reste sensible.


**La mémoire brute : `mem`**

Le fichier :

```bash
/proc/<PID>/mem
```

représente la mémoire virtuelle du processus.

Son accès est fortement contrôlé.

Lire ou modifier ce fichier peut permettre :

- d’extraire des secrets en mémoire ;

- d’observer des données applicatives ;

- de modifier le comportement d’un processus ;

- de contourner certaines protections si les permissions sont insuffisantes.


Nous ne l’utilisons pas dans un usage courant.

Nous retenons :

```text
/proc/<PID>/mem est extrêmement sensible.
Son accès doit être strictement contrôlé.
```


**Les liens `cwd`, `exe` et `root`**

Les liens suivants peuvent aussi exposer des informations :

```bash
/proc/<PID>/cwd
/proc/<PID>/exe
/proc/<PID>/root
```

Ils permettent de savoir :

- d’où le processus travaille ;

- quel binaire il exécute ;

- quelle racine de système de fichiers il voit ;

- s’il tourne dans un conteneur ou un chroot ;

- s’il utilise un exécutable supprimé.


Exemple :

```bash
ls -l /proc/<PID>/exe
```

Sortie possible :

```text
/proc/1234/exe -> /opt/app/releases/2026-05-01/app-server
```

Ce type d’information peut révéler la stratégie de déploiement ou les chemins internes.


## Permissions dans `/proc`

**Observer les permissions**

Nous pouvons afficher les permissions d’un processus :

```bash
ls -ld /proc/<PID>
ls -l /proc/<PID> | head
```

Certaines entrées sont lisibles par tous, d’autres uniquement par le propriétaire du processus ou par `root`.

Exemple :

```bash
ls -l /proc/$$/environ
```

Nous observons les droits Unix classiques :

```text
-r-------- 1 user user 0 mai 19 10:00 environ
```

Le fichier semble avoir une taille `0`, mais il peut produire du contenu lors de la lecture.


**Le rôle de l’UID**

Les règles d’accès dépendent notamment :

- de l’utilisateur réel du processus ;

- de l’utilisateur qui lit ;

- des permissions du fichier ;

- des capacités Linux ;

- des options de montage de `/proc` ;

- de mécanismes de sécurité comme Yama, AppArmor ou SELinux.


Un utilisateur peut généralement lire plus facilement les informations de ses propres processus que celles des processus d’un autre utilisateur.


**Tester avec deux utilisateurs**

Sur une machine de test, nous pouvons créer deux utilisateurs :

```bash
sudo adduser alice
sudo adduser bob
```

Nous lançons un processus sous `alice` :

```bash
sudo -u alice sleep 1000 &
pid=$!
```

Puis nous essayons de lire depuis un autre utilisateur :

```bash
sudo -u bob cat /proc/$pid/status
sudo -u bob tr '\0' '\n' < /proc/$pid/environ
sudo -u bob ls -l /proc/$pid/fd
```

Nous observons ce qui est autorisé ou refusé.

Cet exercice montre que `/proc` applique des restrictions, mais que le niveau exact dépend de la configuration du système.


## Le mécanisme `ptrace` et Yama

**Pourquoi parler de `ptrace` ?**

Sous Linux, certains accès à `/proc/<PID>` sont liés aux règles de traçage et de debugging.

Le mécanisme `ptrace` permet à un processus d’en observer ou contrôler un autre, par exemple pour un débogueur comme `gdb`.

Certaines restrictions de `/proc` suivent une logique proche : si un processus n’a pas le droit d’en tracer un autre, il n’a pas forcément le droit de lire certaines informations sensibles.


**Le paramètre `ptrace_scope`**

Sur certaines distributions, le module de sécurité Yama expose :

```bash
cat /proc/sys/kernel/yama/ptrace_scope
```

Valeurs courantes :

|Valeur|Effet général|
|--:|---|
|`0`|comportement classique plus permissif|
|`1`|restrictions supplémentaires, souvent parent-enfant|
|`2`|seul un processus avec privilège peut tracer|
|`3`|traçage désactivé de manière stricte jusqu’au redémarrage|

La valeur exacte disponible dépend de la configuration du noyau.


**Effet sur le debugging**

Si `ptrace_scope` est restrictif, un utilisateur peut rencontrer des difficultés à attacher `gdb` à un processus qui ne descend pas directement de son shell.

Exemple :

```bash
gdb -p <PID>
```

peut échouer avec une erreur de permission.

Nous comprenons donc que les protections autour de `/proc` et du debugging sont liées.


## Option de montage `hidepid`

**Le problème sur les serveurs multi-utilisateurs**

Sur un serveur partagé, nous voulons parfois éviter qu’un utilisateur voie les processus des autres utilisateurs.

Sans restriction particulière, un utilisateur peut souvent lister les PID :

```bash
ls /proc | grep -E '^[0-9]+$'
```

et voir certaines informations via :

```bash
ps aux
```

Cela peut révéler :

- noms d’utilisateurs ;

- commandes lancées ;

- services personnels ;

- scripts ;

- chemins de travail ;

- arguments de ligne de commande.


L’option `hidepid` permet de réduire cette visibilité.


**Les valeurs de `hidepid`**

`hidepid` est une option de montage de `procfs`.

Les valeurs classiques sont :

|Valeur|Effet|
|--:|---|
|`hidepid=0`|comportement classique, visibilité large|
|`hidepid=1`|les utilisateurs voient les PID, mais pas les détails sensibles des processus des autres|
|`hidepid=2`|les processus des autres utilisateurs sont masqués|
|`hidepid=invisible`|forme équivalente ou moderne selon systèmes pour masquer davantage|
|`hidepid=ptraceable`|visibilité alignée sur les processus traçables, selon noyau|

Dans un cours général, nous retenons surtout `0`, `1` et `2`.


**Vérifier le montage actuel**

Nous vérifions les options de montage de `/proc` :

```bash
findmnt /proc
```

ou :

```bash
mount | grep ' /proc '
```

Sortie possible :

```text
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
```

Si `hidepid` est actif, nous pouvons voir :

```text
hidepid=2
```

dans les options.


**Monter `/proc` avec `hidepid`**

Sur une machine de test, nous pouvons remonter `/proc` avec :

```bash
sudo mount -o remount,hidepid=2 /proc
```

Puis nous testons avec un utilisateur non privilégié :

```bash
ls /proc
ps aux
```

Nous observons que les processus des autres utilisateurs deviennent moins visibles.

Pour revenir au comportement classique :

```bash
sudo mount -o remount,hidepid=0 /proc
```

Nous faisons cela uniquement dans un environnement de test ou avec une bonne compréhension des effets.


**Configuration persistante**

Une configuration persistante peut se faire via `/etc/fstab`.

Exemple indicatif :

```fstab
proc /proc proc defaults,nosuid,nodev,noexec,hidepid=2 0 0
```

Selon les distributions, systemd ou d’autres mécanismes peuvent influencer le montage de `/proc`.

Nous devons donc tester et vérifier après redémarrage :

```bash
findmnt /proc
```


**Groupe autorisé**

Dans certains environnements, nous voulons masquer les processus aux utilisateurs ordinaires, mais garder une visibilité pour un groupe d’administrateurs.

Des options de montage peuvent permettre d’indiquer un groupe autorisé, par exemple avec `gid`.

Le principe est :

```text
les utilisateurs ordinaires voient moins d’informations ;
les membres d’un groupe dédié gardent une visibilité d’administration.
```

La configuration exacte dépend de la distribution et de la version du noyau.


## `/proc` dans les conteneurs

**Pourquoi les conteneurs changent la lecture de `/proc`**

Un conteneur utilise des mécanismes d’isolation du noyau Linux, notamment les namespaces.

Un processus dans un conteneur peut voir :

- une liste de processus différente ;

- une pile réseau différente ;

- des montages différents ;

- une racine différente ;

- des limites cgroups différentes.


Cela signifie que `/proc` vu depuis un conteneur n’est pas forcément le même que `/proc` vu depuis l’hôte.


**Namespace PID**

Le namespace PID contrôle la vue des processus.

Dans un conteneur, le processus principal peut avoir le PID `1` dans le conteneur, mais un autre PID sur l’hôte.

Dans le conteneur :

```bash
cat /proc/1/comm
```

peut afficher :

```text
node
```

ou :

```text
python
```

ou :

```text
bash
```

Sur l’hôte, le même processus peut avoir le PID `24531`.

Nous retenons :

```text
Le PID dépend du namespace depuis lequel nous observons.
```


**Namespace réseau**

Nous avons vu au chapitre 7 que :

```bash
/proc/net
```

dépend du namespace réseau.

Dans un conteneur, nous pouvons voir :

```bash
cat /proc/net/dev
```

et obtenir seulement :

```text
lo
eth0
```

alors que l’hôte possède :

```text
lo
eno1
docker0
br-...
veth...
wg0
```

La vue réseau du conteneur est isolée.


**Namespace de montage**

Le namespace de montage contrôle les points de montage visibles.

Dans un conteneur :

```bash
cat /proc/mounts
```

peut montrer une arborescence différente de celle de l’hôte.

Le lien :

```bash
/proc/<PID>/root
```

est particulièrement intéressant.

Depuis l’hôte, si nous regardons un processus de conteneur :

```bash
sudo ls -l /proc/<PID>/root
```

nous pouvons accéder à la racine vue par ce processus.

C’est puissant pour le diagnostic, mais sensible du point de vue sécurité.


## Risques des conteneurs privilégiés

**Le danger d’un `/proc` trop permissif**

Dans un conteneur, monter `/proc` de manière trop permissive peut exposer des informations de l’hôte ou permettre des actions dangereuses.

Un conteneur privilégié peut avoir des capacités étendues.

Si nous combinons :

```text
conteneur privilégié
montage sensible de /proc
accès à /sys
accès à /dev
capabilities élevées
```

nous réduisons fortement l’isolation.


**Exemple de mauvaise pratique**

Un montage dangereux pourrait ressembler à :

```bash
docker run --privileged -v /proc:/host/proc image
```

Le conteneur peut alors observer des informations de l’hôte via `/host/proc`.

Selon les autres droits accordés, cela peut devenir très risqué.

Nous évitons ce type de montage sauf besoin exceptionnel et environnement maîtrisé.


**Paramètres noyau depuis un conteneur**

Certains fichiers de `/proc/sys` peuvent être visibles dans un conteneur.

Mais le noyau est partagé entre l’hôte et les conteneurs.

Modifier un paramètre noyau depuis un conteneur peut donc affecter l’hôte ou d’autres conteneurs, selon le paramètre et le namespace.

C’est pourquoi les runtimes de conteneurs limitent généralement les accès en écriture.

Nous retenons :

```text
Un conteneur partage le noyau avec l’hôte.
Les paramètres noyau ne sont pas des paramètres privés applicatifs ordinaires.
```


## `/proc/sys` et sécurité

**Paramètres de durcissement**

Certains paramètres de `/proc/sys` ont un rôle de sécurité.

Exemples :

```bash
cat /proc/sys/kernel/randomize_va_space
cat /proc/sys/kernel/dmesg_restrict
cat /proc/sys/kernel/kptr_restrict
cat /proc/sys/fs/protected_symlinks
cat /proc/sys/fs/protected_hardlinks
```

Ils influencent :

- la randomisation mémoire ;

- l’accès aux logs noyau ;

- l’exposition des pointeurs noyau ;

- la protection contre certaines attaques par liens symboliques ;

- la protection contre certains hard links abusifs.



**ASLR : `randomize_va_space`**

Nous lisons :

```bash
cat /proc/sys/kernel/randomize_va_space
```

Valeurs courantes :

```text
0 : désactivé
1 : randomisation partielle
2 : randomisation plus complète
```

En production, nous gardons généralement l’ASLR activé.

Désactiver l’ASLR facilite certains tests bas niveau, mais affaiblit la sécurité.


**Restriction de `dmesg`**

Nous lisons :

```bash
cat /proc/sys/kernel/dmesg_restrict
```

Si la valeur est `1`, l’accès non privilégié à `dmesg` est restreint.

Cela limite l’exposition d’informations noyau qui pourraient aider un attaquant.


**Restriction des pointeurs noyau**

Nous lisons :

```bash
cat /proc/sys/kernel/kptr_restrict
```

Ce paramètre limite l’exposition d’adresses noyau dans certaines interfaces.

C’est important car les adresses noyau peuvent faciliter des attaques exploitant des vulnérabilités bas niveau.


## Secrets et bonnes pratiques applicatives

**Ne pas mettre de secrets dans les arguments**

Nous évitons :

```bash
app --password secret
app --token abcdef
curl -H "Authorization: Bearer secret" ...
```

Nous préférons :

- gestionnaire de secrets ;

- fichier de configuration avec permissions strictes ;

- agent local ;

- socket sécurisée ;

- injection contrôlée par l’orchestrateur ;

- variable d’environnement seulement si le risque est compris et accepté.



**Variables d’environnement : pratique mais exposable**

Les variables d’environnement sont très utilisées dans Docker, Kubernetes, systemd et les applications cloud-native.

Mais elles peuvent être visibles :

- dans `/proc/<PID>/environ` ;

- dans certains dumps ;

- dans des interfaces de debug ;

- dans des logs si l’application les affiche ;

- dans des erreurs de configuration.


Nous ne devons pas croire qu’une variable d’environnement est forcément protégée.


**Fichiers de secrets**

Un fichier de secret peut être préférable si :

- les permissions sont strictes ;

- le propriétaire est correct ;

- le fichier n’est pas logué ;

- le fichier n’est pas inclus dans une image Docker ;

- le fichier n’est pas commité dans Git ;

- le secret peut être monté temporairement par l’orchestrateur.


Exemple de permissions :

```bash
chmod 600 /etc/app/secret.conf
chown appuser:appuser /etc/app/secret.conf
```

Mais là encore, si le processus ouvre ce fichier, il peut apparaître dans :

```bash
/proc/<PID>/fd
```

Nous ne supprimons pas le risque, nous le réduisons et nous le contrôlons.


## Serveurs multi-utilisateurs

**Risques spécifiques**

Sur un serveur utilisé par plusieurs personnes, `/proc` peut exposer des informations entre utilisateurs.

Exemples :

- un étudiant voit les processus d’un autre étudiant ;

- un utilisateur voit une commande contenant un token ;

- un utilisateur identifie un service vulnérable ;

- un utilisateur observe les chemins de travail d’un autre ;

- un utilisateur détecte les ports locaux ouverts.


Ces risques sont importants sur :

- serveurs universitaires ;

- machines de calcul partagées ;

- serveurs de formation ;

- bastions SSH ;

- plateformes mutualisées.



**Mesures possibles**

Nous pouvons renforcer :

```text
hidepid=2
Yama ptrace_scope
permissions strictes
pas de secrets en arguments
limites systemd
isolation par conteneur ou VM
journalisation contrôlée
groupes d’administration dédiés
```

Aucune mesure unique ne suffit.

Nous construisons une défense en profondeur.


## Audit rapide de sécurité autour de `/proc`

**Vérifier les options de montage**

```bash
findmnt /proc
```

Nous cherchons notamment :

```text
hidepid
nosuid
nodev
noexec
```

Options courantes :

```text
rw,nosuid,nodev,noexec,relatime
```


**Vérifier les paramètres sensibles**

```bash
sysctl kernel.randomize_va_space
sysctl kernel.dmesg_restrict
sysctl kernel.kptr_restrict
sysctl fs.protected_symlinks
sysctl fs.protected_hardlinks
```

Nous cherchons des valeurs cohérentes avec un système durci.


**Chercher des secrets en arguments**

En environnement de test ou d’audit autorisé :

```bash
ps aux | grep -Ei 'password|passwd|token|secret|key'
```

Nous pouvons aussi inspecter `/proc/<PID>/cmdline`.

L’objectif est de détecter les mauvaises pratiques.


**Chercher des environnements sensibles**

Avec les droits nécessaires et dans un cadre autorisé :

```bash
sudo sh -c '
for env in /proc/[0-9]*/environ; do
    pid=$(echo "$env" | cut -d/ -f3)
    tr "\0" "\n" < "$env" 2>/dev/null |
    grep -Ei "password|passwd|token|secret|key" |
    sed "s/^/PID=$pid /"
done
'
```

Nous n’exécutons pas ce type de commande sans autorisation explicite, car elle peut exposer des secrets.


## Cas pratique : durcir un serveur partagé

**Situation**

Nous administrons un serveur Linux utilisé par plusieurs développeurs.

Chaque utilisateur peut se connecter en SSH.

Nous voulons éviter qu’un utilisateur puisse observer trop facilement les processus et arguments des autres.


**Diagnostic initial**

Nous vérifions :

```bash
findmnt /proc
```

Puis, depuis un utilisateur non privilégié :

```bash
ps aux | head
ls /proc | grep -E '^[0-9]+$' | head
```

Nous observons la visibilité actuelle.


**Mesure principale : `hidepid=2`**

Nous testons temporairement :

```bash
sudo mount -o remount,hidepid=2 /proc
```

Puis nous vérifions avec un utilisateur non privilégié :

```bash
ps aux
ls /proc
```

Nous observons que les processus des autres utilisateurs sont masqués ou moins visibles.


**Persistance**

Nous configurons `/etc/fstab` avec prudence.

Exemple :

```fstab
proc /proc proc defaults,nosuid,nodev,noexec,hidepid=2 0 0
```

Puis nous testons au redémarrage ou via un remount contrôlé.

Nous vérifions :

```bash
findmnt /proc
```


**Compléments**

Nous vérifions aussi :

```bash
sysctl kernel.yama.ptrace_scope
sysctl kernel.dmesg_restrict
sysctl kernel.kptr_restrict
```

Nous sensibilisons les utilisateurs :

```text
pas de mots de passe en ligne de commande ;
pas de tokens dans les scripts visibles ;
pas de secrets dans les logs ;
permissions strictes sur les fichiers de configuration.
```


## Cas pratique : analyser un conteneur

**Situation**

Nous sommes dans un conteneur et nous voulons comprendre ce que `/proc` représente.

Nous exécutons :

```bash
cat /proc/1/comm
ls /proc | grep -E '^[0-9]+$' | wc -l
cat /proc/net/dev
cat /proc/self/mounts | head
cat /proc/1/cgroup
```

Nous observons :

- le PID 1 du conteneur ;

- le nombre de processus visibles ;

- les interfaces réseau vues ;

- les montages visibles ;

- les cgroups.



**Comparaison avec l’hôte**

Depuis l’hôte, nous comparons :

```bash
ps aux
ip addr
findmnt
```

Nous constatons que les vues diffèrent.

Nous retenons :

```text
/proc n’est pas une vérité absolue indépendante du contexte.
C’est une vue dépendante des namespaces et des montages.
```


**Vérifier les montages sensibles**

Dans un conteneur, nous inspectons :

```bash
mount | grep proc
findmnt /proc
```

Nous cherchons à savoir si `/proc` est monté normalement, en lecture seule, ou avec des restrictions spécifiques.

Selon le runtime, certaines parties de `/proc` peuvent être masquées ou rendues non modifiables.


## Bonnes pratiques de sécurité

**Pour les développeurs**

Nous appliquons les règles suivantes :

```text
ne pas passer de secrets en arguments ;
éviter d’imprimer l’environnement dans les logs ;
ne pas stocker de secrets dans des fichiers lisibles par tous ;
limiter les permissions des fichiers de configuration ;
prévoir la rotation des secrets ;
utiliser un gestionnaire de secrets quand c’est possible.
```


**Pour les administrateurs système**

Nous vérifions :

```text
options de montage de /proc ;
paramètres sysctl de sécurité ;
permissions des services ;
limites systemd ;
séparation des utilisateurs ;
journalisation ;
durcissement SSH ;
conteneurs non privilégiés ;
absence de montages dangereux.
```


**Pour les environnements conteneurisés**

Nous évitons :

```text
--privileged sans nécessité ;
montage de /proc de l’hôte ;
montage de /sys de l’hôte ;
accès large à /dev ;
capabilities inutiles ;
secrets en variables d’environnement si l’exposition est inacceptable.
```

Nous préférons :

```text
capabilities minimales ;
root filesystem en lecture seule si possible ;
utilisateur non-root ;
seccomp ;
AppArmor ou SELinux ;
secrets gérés par l’orchestrateur ;
politiques réseau ;
limites cgroups.
```


## Pièges et limites

**Masquer `/proc` ne suffit pas**

`hidepid` réduit la visibilité, mais ne remplace pas :

- une bonne gestion des permissions ;

- une séparation correcte des utilisateurs ;

- une politique de secrets ;

- un durcissement système ;

- une supervision ;

- des mises à jour de sécurité.


C’est une couche parmi d’autres.


**Trop restreindre peut casser des outils**

Certaines restrictions peuvent perturber :

- outils de monitoring ;

- agents de supervision ;

- outils de debug ;

- scripts d’administration ;

- services qui inspectent les processus.


Avant de durcir, nous devons tester.


**Les conteneurs ne sont pas des machines virtuelles**

Un conteneur partage le noyau avec l’hôte.

Même si `/proc` est isolé par namespaces, une mauvaise configuration peut affaiblir l’isolation.

Nous ne devons pas considérer un conteneur privilégié comme une barrière de sécurité forte.


**Les secrets finissent souvent en mémoire**

Même si nous évitons les arguments et limitons les variables d’environnement, un secret utilisé par une application finit souvent en mémoire.

Nous devons donc contrôler :

- qui peut déboguer le processus ;

- qui peut lire ses dumps mémoire ;

- qui peut accéder à `/proc/<PID>/mem` ;

- qui peut lancer des outils de diagnostic ;

- comment les core dumps sont gérés.

# Chapitre 9 — Sécurité moderne, conteneurs et durcissement


## Réduire l’exposition de procfs

Procfs contient beaucoup d'informations utiles à un attaquant local.

### Informations sensibles potentielles

Exemples :

```text
cmdline
environ
maps
fd/
mountinfo
status
cgroup
ns/
```

Ces informations peuvent révéler :

- noms de services ;
- chemins ;
- secrets mal placés ;
- topologie ;
- protections activées ;
- bibliothèques ;
- fichiers ouverts.

### Contrôles ptrace

De nombreuses entrées sensibles utilisent des vérifications proches des permissions `ptrace`.

Les mécanismes pouvant intervenir incluent :

- UID/GID ;
- dumpability ;
- capabilities ;
- Yama ;
- LSM ;
- user namespaces ;
- options `hidepid`.

### Yama

Selon la distribution :

```bash
sysctl kernel.yama.ptrace_scope
```

peut restreindre les possibilités de ptrace entre processus.

Nous devons comprendre la politique de la distribution avant de la modifier.

---


## Informations noyau particulièrement sensibles

### `/proc/kallsyms`

Cette interface expose la table de symboles du noyau, sous réserve des restrictions de sécurité.

L'affichage d'adresses peut être masqué selon :

```bash
sysctl kernel.kptr_restrict
```

### `/proc/kcore`

`/proc/kcore` représente une vue ELF de la mémoire virtuelle du noyau.

C'est une interface très sensible, généralement inaccessible à un utilisateur ordinaire.

Nous ne la copions jamais « pour voir » sur un système de production.

### Principe

> Plus une interface permet d'observer l'état interne du noyau, plus nous devons considérer sa lecture comme une opération privilégiée et potentiellement sensible.

---


## Interfaces bas niveau : `mem` et `pagemap`

### `mem`

`/proc/<PID>/mem` donne accès à l'espace d'adressage d'un processus sous des contrôles stricts.

Ce n'est pas une interface d'observation ordinaire.

Pour du debugging :

- `gdb` ;
- `ptrace` ;
- `process_vm_readv()` ;
- outils de profiling dédiés

sont généralement de meilleures primitives.

### `pagemap`

```text
/proc/<PID>/pagemap
```

fournit des informations bas niveau sur les pages virtuelles.

L'accès à certaines informations comme les PFN a été volontairement restreint pour limiter des attaques matérielles ou par canaux auxiliaires.

Nous ne supposons donc pas qu'un parseur `pagemap` fonctionnera sans privilèges sur tous les systèmes.

---


## Ce que change un conteneur

Un conteneur n'obtient pas un « faux `/proc` » au sens simple : son runtime organise namespaces, montages et cgroups afin de lui présenter une vue cohérente.

### PID namespace

Dans un conteneur :

```bash
cat /proc/1/status
```

peut décrire le PID 1 **du namespace du conteneur**, et non le PID 1 de l'hôte.

Le champ :

```text
NSpid
```

peut aider à relier les niveaux de namespace.

### Network namespace

```text
/proc/net
```

reflète le réseau vu depuis le namespace du lecteur.

### Mount namespace

```text
/proc/self/mountinfo
```

montre la topologie des montages vue par le processus.

### Cgroups

Le runtime peut limiter CPU/RAM/PIDs via cgroup v2, mais les limites sont principalement observées sous :

```text
/sys/fs/cgroup
```

### Pourquoi `/proc` ne suffit pas pour « détecter un conteneur »

Il n'existe pas une unique propriété fiable disant :

```text
container=true
```

Nous devons raisonner sur le besoin réel :

- identifier notre namespace ;
- connaître les limites cgroup ;
- savoir quel filesystem est monté ;
- connaître l'orchestrateur ;
- comprendre le contexte de sécurité.

---


## Durcir un service systemd

Systemd expose une abstraction plus élevée de nombreux concepts noyau.

Exemples :

```bash
systemctl status ssh.service
systemctl show ssh.service
systemd-cgls
systemd-cgtop
```

Procfs reste utile pour vérifier la réalité bas niveau.

Exemple de démarche :

```text
systemctl status
      ↓
PID principal
      ↓
/proc/<PID>/status
/proc/<PID>/fd
/proc/<PID>/maps
/proc/<PID>/cgroup
      ↓
/sys/fs/cgroup/...
```

Voir `[[Initialisation système et des services]]`.

---


# Chapitre 10 — Parser et diagnostiquer sans se tromper


## Comment les outils de haut niveau utilisent `/proc`

### `ps`

`ps` lit plusieurs informations par processus et applique sa propre logique de présentation.

Nous ne devons pas reconstruire `ps` en shell pour une application de production si `procps-ng` fait déjà le travail.

### `top` / `htop`

Ces outils prennent des snapshots successifs pour calculer des taux.

### `free`

`free` interprète notamment `/proc/meminfo` plutôt que d'afficher naïvement `MemFree`.

### `lsof`

`lsof` combine plusieurs sources et abstrait la complexité de `fd`, sockets, mappings et permissions.

### `ss`

Pour les sockets, `ss` est généralement préférable au décodage manuel de `/proc/net/tcp`.

---


## Parsing robuste

### Accepter la disparition d'un processus

Python :

```python
from pathlib import Path

pid = 1234
path = Path(f"/proc/{pid}/status")

try:
    text = path.read_text()
except FileNotFoundError:
    print("Le processus a disparu")
except PermissionError:
    print("Accès refusé")
else:
    print(text)
```

### Parser les formats clé/valeur

Pour `status` :

```python
from pathlib import Path


def read_status(pid: int) -> dict[str, str]:
    values: dict[str, str] = {}
    for line in Path(f"/proc/{pid}/status").read_text().splitlines():
        if ":" not in line:
            continue
        key, value = line.split(":", 1)
        values[key] = value.strip()
    return values


status = read_status(1)
print(status.get("Name"))
print(status.get("NSpid"))
```

### NUL dans `cmdline`

```python
from pathlib import Path

raw = Path("/proc/self/cmdline").read_bytes()
argv = [part.decode(errors="replace") for part in raw.split(b"\0") if part]
print(argv)
```

### Ne pas scraper la sortie humaine si une API existe

Mauvais :

```text
outil → texte coloré localisé → grep fragile
```

Mieux :

```text
API structurée / bibliothèque → données typées
```

Procfs est utile, mais notre code doit documenter explicitement le format qu'il dépend de.

---


## Pièges courants

### Confondre RSS et consommation réelle

Une bibliothèque partagée peut être comptée dans le RSS de plusieurs processus.

Utiliser PSS lorsque l'attribution proportionnelle est importante.

### Additionner les RSS

```text
RSS(P1) + RSS(P2) + ...
```

surestime souvent la consommation réelle à cause des pages partagées.

### Utiliser `MemFree`

`MemAvailable` est généralement plus pertinent pour comprendre la marge globale.

### Utiliser `/proc/meminfo` comme limite conteneur

Inspecter cgroup v2.

### Parser `/proc/<PID>/stat` avec un split simple

Le champ `comm` rend cette approche incorrecte.

### Stocker seulement un PID longtemps

Un PID peut être réutilisé.

Pour les applications sensibles, envisager pidfd ou valider l'identité du processus.

### Déduire un état stable de plusieurs fichiers

Deux lectures successives ne constituent pas une transaction.

### Écrire dans `/proc/sys` sans rollback

Toujours connaître la valeur précédente et documenter l'effet attendu.

### Poller `smaps` très fréquemment

La précision a un coût.

### Confondre namespace et cgroup

Namespace : visibilité/isolation de ressources.

Cgroup : contrôle/comptabilité de ressources.

---


## Méthode de diagnostic

Nous utilisons une méthode en couches.

### Étape 1 — Identifier le contexte

```bash
uname -r
cat /proc/self/cgroup
ls -l /proc/self/ns
findmnt -T /proc
```

Questions :

- hôte ou conteneur ?
- quel PID namespace ?
- quel cgroup ?
- quelles restrictions procfs ?

### Étape 2 — Santé globale

```bash
cat /proc/loadavg
head /proc/meminfo
cat /proc/pressure/cpu
cat /proc/pressure/memory
cat /proc/pressure/io
```

### Étape 3 — Identifier le processus

```bash
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

Puis :

```bash
cat /proc/$pid/status
cat /proc/$pid/io
ls -l /proc/$pid/fd
```

### Étape 4 — Examiner les ressources

CPU :

```text
stat / sched / task / perf
```

Mémoire :

```text
status / smaps_rollup / cgroup memory.*
```

I/O :

```text
io / fd / PSI I/O / iostat
```

Réseau :

```text
ss / ip / namespace réseau
```

### Étape 5 — Corréler avec les événements

```bash
journalctl
journalctl -k
```

Procfs donne un état ; les journaux donnent souvent la chronologie.

---


## Cas : service lent sans CPU saturé

Symptôme :

```text
latence élevée
CPU moyen faible
```

## Étape 1 — Charge

```bash
cat /proc/loadavg
```

Charge anormalement élevée malgré CPU libre.

## Étape 2 — PSI

```bash
cat /proc/pressure/io
```

Nous observons une pression I/O élevée.

## Étape 3 — Processus

```bash
pidstat -d 1
```

puis :

```bash
cat /proc/$pid/io
ls -l /proc/$pid/fd
```

## Étape 4 — Stockage

```bash
iostat -xz 1
```

Nous confirmons la saturation du stockage.

## Conclusion

Le problème n'était pas « le CPU » mais du temps perdu en attente d'I/O.

PSI a permis d'identifier rapidement la nature de la contention.

---


## Cas : conteneur tué pour OOM malgré `MemAvailable`

Dans le conteneur :

```bash
grep MemAvailable /proc/meminfo
```

semble montrer plusieurs Go disponibles.

Mais :

```bash
cat /sys/fs/cgroup/memory.max
```

indique :

```text
1073741824
```

soit 1 Gio.

Et :

```bash
cat /sys/fs/cgroup/memory.current
```

est proche de cette limite.

Conclusion :

> La limite déterminante est celle du cgroup, pas la mémoire globale visible via `meminfo`.

---


## Cas : disque plein mais aucun gros fichier visible

```bash
df -h
```

montre un filesystem plein.

```bash
du -sh /*
```

ne retrouve pas l'espace.

Recherche :

```bash
sudo lsof +L1
```

ou :

```bash
sudo find /proc/[0-9]*/fd -lname '* (deleted)' -ls 2>/dev/null
```

Nous trouvons un ancien fichier de log supprimé mais encore ouvert par un processus.

Solution correcte :

- faire rouvrir proprement le fichier au service ;
- redémarrer le service si nécessaire ;
- corriger la rotation de logs.

Nous évitons de tronquer arbitrairement un FD sans comprendre le service.

---


## Cas : vérifier le durcissement d’un service

Trouvons le PID :

```bash
pid=$(systemctl show -p MainPID --value monservice.service)
```

Puis :

```bash
grep -E '^(Uid|Gid|Cap|NoNewPrivs|Seccomp|Seccomp_filters):' "/proc/$pid/status"
```

Namespaces :

```bash
ls -l "/proc/$pid/ns"
```

Cgroup :

```bash
cat "/proc/$pid/cgroup"
```

Montages :

```bash
head -n 20 "/proc/$pid/mountinfo"
```

Nous comparons ensuite à :

```bash
systemd-analyze security monservice.service
```

Procfs permet de vérifier l'**état réellement appliqué**, pas seulement la configuration déclarée.

---


# Chapitre 11 — Travaux pratiques et projet


## TP 1 — Cartographier procfs

Objectif : distinguer les grandes familles d'entrées.

```bash
findmnt -T /proc
ls /proc | head -50
ls -ld /proc/[0-9]* | head
```

À produire :

- type de filesystem ;
- options de montage ;
- cinq entrées globales ;
- cinq entrées par processus ;
- différence avec `/sys`.

---


## TP 2 — Analyser notre propre processus

```bash
pid=$$
cat /proc/$pid/status
tr '\0' '\n' < /proc/$pid/cmdline
ls -l /proc/$pid/fd
cat /proc/$pid/limits
```

Questions :

1. quel PPID ?
2. combien de threads ?
3. quelles capabilities effectives ?
4. seccomp est-il actif ?
5. quel est le nombre maximal de FDs ?

---


## TP 3 — PID namespaces

Avec `unshare` dans un environnement de laboratoire :

```bash
sudo unshare --pid --fork --mount-proc bash
```

Dans le shell isolé :

```bash
echo $$
cat /proc/self/status | grep -E '^(Pid|NSpid):'
ps aux
```

Comparer avec l'hôte.

Voir `[[Les namespaces Linux]]`.

---


## TP 4 — Détecter un fichier supprimé ouvert

Créer :

```bash
f=$(mktemp)
exec 9>"$f"
printf 'bonjour\n' >&9
rm "$f"
ls -l /proc/$$/fd/9
```

Observer `(deleted)`.

Fermer :

```bash
exec 9>&-
```

---


## TP 5 — RSS vs PSS

Choisir un processus :

```bash
pid=$$
grep -E '^(VmRSS|RssAnon|RssFile|RssShmem):' /proc/$pid/status
cat /proc/$pid/smaps_rollup
```

Comparer :

- RSS ;
- PSS ;
- Private ;
- Shared.

Expliquer pourquoi RSS et PSS diffèrent.

---


## TP 6 — Mesurer un taux CPU à partir de `/proc/stat`

Écrire un script qui :

1. lit la première ligne `cpu` ;
2. attend une seconde ;
3. relit la ligne ;
4. calcule les deltas ;
5. affiche une estimation du pourcentage occupé.

Comparer avec :

```bash
mpstat 1 2
```

---


## TP 7 — PSI

Créer une charge CPU contrôlée dans un environnement de test :

```bash
stress-ng --cpu "$(nproc)" --timeout 20s
```

Observer :

```bash
watch -n 0.5 cat /proc/pressure/cpu
```

Puis comparer avec :

```bash
uptime
mpstat 1
```

Questions :

- que mesure `avg10` ?
- différence entre utilisation CPU et pression CPU ?

---


## TP 8 — Sysctl temporaire vs persistant

Dans une VM de laboratoire uniquement :

```bash
sysctl vm.swappiness
```

Identifier :

```text
/proc/sys/vm/swappiness
```

Créer ensuite un fichier de configuration exemple sans nécessairement l'appliquer :

```ini
# /etc/sysctl.d/99-lab.conf
vm.swappiness = 20
```

Expliquer la différence entre lecture, écriture runtime et configuration persistante.

---


## TP 9 — Audit sécurité d'un processus

Choisir un service :

```bash
pid=$(systemctl show -p MainPID --value ssh.service)
```

Adapter le nom selon la distribution.

Observer :

```bash
grep -E '^(Uid|Gid|Cap|NoNewPrivs|Seccomp|Seccomp_filters):' /proc/$pid/status
ls -l /proc/$pid/ns
cat /proc/$pid/cgroup
```

Produire une courte analyse des frontières de sécurité visibles.

---


## TP 10 — Vue cgroup d'un conteneur

Lancer un conteneur avec une limite mémoire, par exemple avec Docker/Podman.

Dans le conteneur :

```bash
cat /proc/meminfo | head
cat /proc/self/cgroup
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/memory.current
```

Expliquer pourquoi les chiffres ne répondent pas tous à la même question.

---


## TP 11 — Parser `/proc/self/status` en Python

Écrire une fonction qui retourne :

```python
{
    "Name": "...",
    "Pid": 123,
    "PPid": 456,
    "Threads": 4,
    "NoNewPrivs": 1,
    "Seccomp": 2,
}
```

Contraintes :

- ne pas dépendre de l'ordre des lignes ;
- ignorer les champs inconnus ;
- gérer l'absence d'un champ ;
- tester avec pytest.

---


## TP 12 — Diagnostic complet

Scénario : une API devient lente dans un conteneur.

Notre rapport doit contenir :

1. contexte noyau ;
2. namespaces ;
3. cgroup ;
4. charge globale ;
5. PSI ;
6. processus principal ;
7. mémoire ;
8. I/O ;
9. FDs ;
10. sécurité ;
11. hypothèse ;
12. données manquantes avant conclusion.

Objectif : **ne pas diagnostiquer à partir d'une seule métrique**.

---


## Projet final — `proc-inspect`

Construire un petit outil de diagnostic en Python.

### Commande

```bash
proc-inspect PID
```

### Sortie minimale

```text
Identity
---
PID             1234
PPID            1
Name            api
Threads         12

Security
---
NoNewPrivs      yes
Seccomp         filter
Effective caps  ...

Memory
------
RSS             ...
PSS             ...
Swap            ...

Isolation
---
PID namespace   ...
NET namespace   ...
Cgroup          ...

Files
-----
Open FDs        ...
Deleted open    ...
```

### Exigences

Le programme doit :

- accepter qu'un processus disparaisse pendant l'analyse ;
- gérer les permissions insuffisantes ;
- ne jamais modifier `/proc` ;
- ne jamais afficher le contenu d'`environ` par défaut ;
- documenter les champs utilisés ;
- avoir des tests ;
- fournir une sortie JSON optionnelle ;
- fonctionner sans privilèges lorsque les informations sont disponibles.

### Bonus

Ajouter :

- comparaison de deux snapshots ;
- PSI global ;
- lecture des limites cgroup v2 ;
- détection des FDs supprimés ;
- export Markdown pour incident report.

---


# Chapitre 12 — Synthèse et révision


## Checklist de diagnostic

## Processus

- [ ] PID/TGID corrects ;
- [ ] risque de réutilisation du PID considéré ;
- [ ] `status` lu ;
- [ ] threads examinés si pertinent ;
- [ ] `cmdline` traité avec séparateurs NUL ;
- [ ] `environ` non exposé inutilement.

## Mémoire

- [ ] `MemAvailable` plutôt que seulement `MemFree` ;
- [ ] RSS/PSS distingués ;
- [ ] `smaps` non pollé excessivement ;
- [ ] cgroup v2 vérifié en conteneur ;
- [ ] PSI mémoire observé.

## CPU

- [ ] compteurs cumulés comparés sur un intervalle ;
- [ ] nombre de CPU pris en compte ;
- [ ] load average non confondu avec utilisation CPU ;
- [ ] PSI CPU observé si contention suspectée.

## I/O

- [ ] `/proc/<PID>/io` vérifié ;
- [ ] FDs analysés ;
- [ ] fichiers supprimés ouverts recherchés ;
- [ ] PSI I/O vérifié ;
- [ ] stockage analysé avec outil approprié.

## Isolation

- [ ] namespaces identifiés ;
- [ ] mountinfo examiné si besoin ;
- [ ] cgroup identifié ;
- [ ] limites cgroup lues sous `/sys/fs/cgroup`.

## Sécurité

- [ ] capabilities ;
- [ ] `NoNewPrivs` ;
- [ ] seccomp ;
- [ ] `hidepid`/permissions ;
- [ ] aucun secret affiché dans un rapport non protégé.

---


## Questions de révision

1. Pourquoi `/proc/meminfo` peut-il afficher une taille de fichier de zéro ?
2. Pourquoi `/proc` n'est-il pas une photographie transactionnelle du système ?
3. Différence entre PID et pidfd ?
4. Pourquoi `cmdline` ne doit-il pas être lu comme une ligne de texte ordinaire ?
5. Différence entre RSS et PSS ?
6. Pourquoi `/proc/meminfo` ne suffit-il pas dans un conteneur ?
7. Que signifie `NSpid` ?
8. Que signifie `Seccomp: 2` ?
9. Quel est le rôle de `NoNewPrivs` ?
10. Différence entre `/proc/<PID>/cgroup` et `/sys/fs/cgroup` ?
11. Que mesure PSI ?
12. Pourquoi `loadavg` peut-il être élevé sans saturation CPU ?
13. Pourquoi `smaps` peut-il être coûteux à lire ?
14. Pourquoi préférer `mountinfo` à `mounts` pour la topologie ?
15. Pourquoi ne faut-il pas parser `stat` avec un simple `split()` ?
16. Différence entre `hidepid=1`, `2` et `4` ?
17. Que fait `subset=pid` ?
18. Pourquoi `/proc/net` dépend-il du network namespace ?
19. Comment diagnostiquer un fichier supprimé encore ouvert ?
20. Pourquoi une modification sous `/proc/sys` n'est-elle pas nécessairement persistante ?

---


## Glossaire

**procfs**
Pseudo-système de fichiers Linux monté généralement sur `/proc`.

**VFS**
Couche d'abstraction du noyau fournissant une interface commune aux systèmes de fichiers.

**PID**
Identifiant numérique d'un processus dans un PID namespace donné.

**TGID**
Identifiant du groupe de threads ; correspond généralement au PID présenté comme processus.

**TID**
Identifiant d'une tâche/thread.

**pidfd**
Descripteur représentant un processus et évitant certaines races liées à la réutilisation des PID.

**RSS**
Resident Set Size : pages actuellement résidentes attribuées à un processus, avec partage potentiellement compté plusieurs fois.

**PSS**
Proportional Set Size : mémoire partagée répartie proportionnellement entre processus.

**PSI**
Pressure Stall Information : métriques de temps perdu à attendre CPU, mémoire ou I/O.

**sysctl**
Interface de lecture/modification de paramètres du noyau, dont beaucoup sont exposés via `/proc/sys`.

**namespace**
Mécanisme d'isolation d'une vue de certaines ressources noyau.

**cgroup**
Mécanisme de contrôle et de comptabilité des ressources.

**capability**
Découpage fin des privilèges traditionnellement associés à root.

**seccomp**
Mécanisme de filtrage des appels système.

**ASLR**
Randomisation de l'espace d'adressage.

**mountinfo**
Interface procfs riche décrivant les montages vus par un processus.

---


## Références

## Documentation noyau

- Linux kernel — *The `/proc` Filesystem* : <https://docs.kernel.org/filesystems/proc.html>
- Linux kernel — *Documentation for `/proc/sys/kernel/`* : <https://docs.kernel.org/admin-guide/sysctl/kernel.html>
- Linux kernel — *PSI — Pressure Stall Information* : <https://docs.kernel.org/accounting/psi.html>
- Linux kernel — cgroup v2 : <https://docs.kernel.org/admin-guide/cgroup-v2.html>

## Man-pages

- `proc(5)` ;
- `proc_pid_status(5)` ;
- `proc_pid_stat(5)` ;
- `proc_pid_smaps(5)` ;
- `proc_pid_mountinfo(5)` ;
- `proc_pid_fd(5)` ;
- `proc_pid_cgroup(5)` ;
- `proc_sys(5)` ;
- `namespaces(7)` ;
- `pid_namespaces(7)` ;
- `capabilities(7)` ;
- `seccomp(2)`.

Pages HTML : <https://man7.org/linux/man-pages/>

## Outils associés

- procps-ng ;
- util-linux ;
- iproute2 ;
- systemd ;
- perf ;
- strace ;
- bpftrace selon le besoin.

---


## Aide-mémoire opérationnel

```bash
### Montage procfs
findmnt -T /proc

### Processus courant
cat /proc/self/status
tr '\0' '\n' < /proc/self/cmdline

### PID namespaces
cat /proc/self/status | grep '^NSpid:'

### Namespaces
ls -l /proc/self/ns

### Cgroup
cat /proc/self/cgroup

### FDs
ls -l /proc/self/fd

### Mémoire processus
cat /proc/self/smaps_rollup

### Limites
cat /proc/self/limits

### CPU global
head /proc/stat

### Mémoire globale
grep -E '^(MemTotal|MemAvailable|SwapTotal|SwapFree):' /proc/meminfo

### Charge
cat /proc/loadavg

### PSI
cat /proc/pressure/cpu
cat /proc/pressure/memory
cat /proc/pressure/io

### Montages
cat /proc/self/mountinfo

### Sysctl
sysctl kernel.hostname
sysctl vm.swappiness

### Sécurité processus
grep -E '^(Cap|NoNewPrivs|Seccomp)' /proc/self/status
```

---


## Conclusion

`/proc` reste une interface fondamentale de Linux : elle permet de relier les outils d’administration aux structures du noyau, de comprendre ce qu’un processus voit réellement et d’effectuer un diagnostic local très précis. Sa puissance vient cependant avec deux contraintes : **le contexte** (namespaces, cgroups, conteneurs) et **la prudence** (formats mouvants, processus qui disparaissent, informations sensibles et paramètres noyau modifiables). La bonne pratique consiste donc à savoir lire procfs directement, tout en préférant les outils de haut niveau lorsqu’ils fournissent déjà une interprétation robuste de ces données.
