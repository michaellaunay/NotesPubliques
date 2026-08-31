---
schema_version: 1
uid: 01M1BQ629MS7J91656157TF131
titre: "proc — 03 — Informations globales sur le système"
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
resume: "Chapitre 3 sur 11 du livre « proc » : Informations globales sur le système. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
niveau: avance
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2026-05-24
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre « proc » — chapitre 3/11
> [[proc — Sommaire|Sommaire]] · [[proc — 02 — Architecture interne de proc|← 02 — Architecture interne de proc]] · [[proc — 04 — proc et les processus|04 — proc et les processus →]]

# Chapitre 3 — Informations globales sur le système
## Objectifs du chapitre

Dans ce chapitre, nous étudions les fichiers globaux de `/proc`.

Contrairement aux répertoires `/proc/<PID>`, qui décrivent un processus précis, les fichiers globaux exposent des informations sur l’ensemble du système : processeur, mémoire, noyau, charge, temps de fonctionnement, systèmes de fichiers, swap, modules, interruptions, disques ou encore ligne de commande du noyau.

À la fin de ce chapitre, nous savons :

- lire les principales informations système depuis `/proc` ;
    
- interpréter `/proc/cpuinfo` ;
    
- comprendre `/proc/meminfo` ;
    
- lire la version du noyau ;
    
- interpréter `/proc/uptime` et `/proc/loadavg` ;
    
- distinguer mémoire libre, mémoire disponible, cache et swap ;
    
- comprendre les limites de ces fichiers dans les machines virtuelles et les conteneurs ;
    
- utiliser ces informations pour établir un premier diagnostic système.
    


## 3.1. Vue d’ensemble des fichiers globaux

### 3.1.1. Les fichiers globaux les plus utiles

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


### 3.1.2. Première lecture rapide

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


## 3.2. Informations CPU avec `/proc/cpuinfo`

### 3.2.1. Lire `/proc/cpuinfo`

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

### 3.2.2. Nombre de CPU logiques

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

### 3.2.3. Modèle du processeur

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

### 3.2.4. Cœurs physiques et threads logiques

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

### 3.2.5. Flags CPU

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

### 3.2.6. Limites de `/proc/cpuinfo`

Nous devons rester prudents.

Dans une machine virtuelle, `/proc/cpuinfo` décrit le processeur virtuel exposé par l’hyperviseur, pas nécessairement le processeur physique complet.

Dans un conteneur, `/proc/cpuinfo` peut montrer plus de CPU que ceux réellement alloués par les limites cgroups.

Par exemple, dans Docker ou Kubernetes, une application peut voir 16 CPU dans `/proc/cpuinfo`, mais être limitée à 2 CPU par cgroup.

Nous retenons donc :

```text
/proc/cpuinfo indique les CPU visibles, pas toujours les ressources réellement garanties.
```

Pour les conteneurs modernes, nous devons aussi regarder les cgroups, par exemple dans `/sys/fs/cgroup`.

## 3.3. Mémoire système avec `/proc/meminfo`

### 3.3.1. Lire `/proc/meminfo`

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

### 3.3.2. `MemTotal`

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

### 3.3.3. `MemFree`

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

### 3.3.4. `MemAvailable`

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

### 3.3.5. `Buffers` et `Cached`

Linux utilise la RAM pour accélérer les accès disque.

Deux champs importants sont :

```bash
grep -E '^(Buffers|Cached):' /proc/meminfo
```

- `Buffers` correspond à certaines métadonnées et buffers liés aux blocs ;
    
- `Cached` correspond au cache de fichiers.
    

Cette mémoire peut souvent être libérée si une application en a besoin.

C’est pourquoi une machine Linux peut afficher peu de mémoire libre tout en ayant beaucoup de mémoire disponible.

### 3.3.6. `SwapTotal` et `SwapFree`

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

### 3.3.7. `Dirty` et `Writeback`

Deux champs sont utiles pour comprendre les écritures disque :

```bash
grep -E '^(Dirty|Writeback):' /proc/meminfo
```

- `Dirty` : mémoire contenant des données modifiées mais pas encore écrites sur disque ;
    
- `Writeback` : mémoire en cours d’écriture vers le disque.
    

Si `Dirty` est très élevé, cela peut indiquer que le système accumule des écritures en attente.

Dans certains incidents, nous pouvons observer une forte latence disque associée à une quantité importante de pages dirty.

### 3.3.8. Comparaison avec `free`

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

### 3.3.9. Limites dans les conteneurs

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

## 3.4. Version du noyau avec `/proc/version`

### 3.4.1. Lire la version du noyau

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

### 3.4.2. Comparaison avec `uname`

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

### 3.4.3. À ne pas confondre avec la version de distribution

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

## 3.5. Temps de fonctionnement avec `/proc/uptime`

### 3.5.1. Lire `/proc/uptime`

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

### 3.5.2. Interpréter la première valeur

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

### 3.5.3. Interpréter la seconde valeur

La seconde valeur correspond au temps idle cumulé sur tous les CPU.

Sur une machine avec plusieurs CPU logiques, cette valeur peut augmenter plus vite que le temps réel.

Exemple :

- si le système a 4 CPU logiques ;
    
- si les 4 CPU restent inactifs pendant 1 seconde ;
    
- le temps idle cumulé augmente d’environ 4 secondes.
    

Nous devons donc éviter de comparer naïvement la seconde valeur à la première.

### 3.5.4. Comparaison avec `uptime`

La commande :

```bash
uptime
```

affiche une sortie plus lisible :

```text
15:42:01 up 2 days,  3:12,  2 users,  load average: 0.24, 0.32, 0.28
```

Elle combine notamment des informations issues de `/proc/uptime` et `/proc/loadavg`.

## 3.6. Charge système avec `/proc/loadavg`

### 3.6.1. Lire `/proc/loadavg`

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

### 3.6.2. Que signifie la charge moyenne ?

La charge moyenne mesure le nombre moyen de tâches :

- en cours d’exécution ;
    
- prêtes à s’exécuter ;
    
- parfois bloquées en attente non interruptible, notamment sur des entrées/sorties.
    

Ce n’est pas simplement un pourcentage CPU.

Une charge de `4.00` n’a pas le même sens sur une machine à 2 CPU logiques et sur une machine à 16 CPU logiques.

### 3.6.3. Interprétation selon le nombre de CPU

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

### 3.6.4. Les trois valeurs : 1, 5 et 15 minutes

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

### 3.6.5. Charge CPU ou attente I/O ?

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

## 3.7. Ligne de commande du noyau avec `/proc/cmdline`

### 3.7.1. Lire `/proc/cmdline`

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

### 3.7.2. Utilité pratique

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

## 3.8. Systèmes de fichiers avec `/proc/filesystems`

### 3.8.1. Lire les systèmes de fichiers supportés

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

### 3.8.2. Signification de `nodev`

La mention `nodev` indique que le système de fichiers ne correspond pas à un périphérique bloc classique.

Par exemple :

```text
nodev proc
nodev sysfs
nodev tmpfs
```

`proc`, `sysfs` et `tmpfs` ne sont pas montés depuis une partition disque classique.

### 3.8.3. Utilité pratique

Ce fichier peut nous aider à vérifier si un type de système de fichiers est supporté.

Par exemple :

```bash
grep ext4 /proc/filesystems
grep btrfs /proc/filesystems
grep nfs /proc/filesystems
```

Si un système de fichiers n’apparaît pas, il peut être absent, non chargé ou disponible via un module non encore chargé.

## 3.9. Points de montage avec `/proc/mounts`

### 3.9.1. Lire `/proc/mounts`

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

### 3.9.2. Différence avec `/etc/fstab`

`/etc/fstab` décrit les montages configurés de manière persistante.

`/proc/mounts` décrit les montages réellement actifs.

Nous retenons :

```text
/etc/fstab → intention de configuration
/proc/mounts → état réel du système
```

Il est possible qu’un montage actif ne soit pas dans `/etc/fstab`, et inversement qu’un montage dans `/etc/fstab` ne soit pas actuellement monté.

### 3.9.3. Lien avec `/proc/self/mounts`

Sur les systèmes modernes, `/proc/mounts` est souvent lié à :

```bash
/proc/self/mounts
```

Nous pouvons vérifier :

```bash
ls -l /proc/mounts
```

Cela signifie que les montages sont vus du point de vue du processus courant, ce qui est important avec les namespaces de montage dans les conteneurs.

## 3.10. Swap avec `/proc/swaps`

### 3.10.1. Lire les espaces de swap

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

### 3.10.2. Comparaison avec `swapon`

La commande suivante donne une vue plus lisible :

```bash
swapon --show
```

Mais `/proc/swaps` est la source directe exposée par le noyau.


### 3.10.3. Diagnostic rapide

Si une machine est lente, nous pouvons vérifier :

```bash
cat /proc/swaps
grep -E '^(SwapTotal|SwapFree):' /proc/meminfo
```

Si beaucoup de swap est utilisé et que la mémoire disponible est faible, nous pouvons suspecter une pression mémoire.

Mais nous devons compléter avec d’autres outils pour savoir si le système est en train de swapper activement.

## 3.11. Partitions avec `/proc/partitions`

### 3.11.1. Lire les partitions connues

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

### 3.11.2. Limites

Ce fichier donne une vue brute.

Pour une analyse plus lisible, nous utilisons souvent :

```bash
lsblk
blkid
fdisk -l
```

Mais `/proc/partitions` reste utile pour comprendre ce que le noyau voit.

## 3.12. Modules noyau avec `/proc/modules`

### 3.12.1. Lire les modules chargés

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

### 3.12.2. Comparaison avec `lsmod`

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

### 3.12.3. Utilité pratique

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

## 3.13. Interruptions avec `/proc/interrupts`

### 3.13.1. Lire les interruptions

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

### 3.13.2. Utilité avancée

Ce fichier devient utile lorsque nous analysons :

- une carte réseau très sollicitée ;
    
- une répartition déséquilibrée des interruptions ;
    
- une latence système ;
    
- un problème de pilote ;
    
- une machine haute performance.
    

Dans un cours introductif à `/proc`, nous ne détaillons pas encore l’optimisation IRQ, mais nous retenons que `/proc/interrupts` donne une vue précieuse sur l’activité matérielle.

## 3.14. Statistiques globales avec `/proc/stat`

### 3.14.1. Lire `/proc/stat`

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

### 3.14.2. Champs utiles

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

### 3.14.3. Pourquoi `/proc/stat` est plus difficile

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

## 3.15. Construire une première synthèse système

### 3.15.1. Objectif

Nous voulons maintenant construire une commande qui produit une synthèse simple à partir de `/proc`.

Nous cherchons à afficher :

- version du noyau ;
    
- uptime ;
    
- CPU logiques ;
    
- mémoire totale ;
    
- mémoire disponible ;
    
- load average ;
    
- nombre de processus visibles.

### 3.15.2. Script Bash simple

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

### 3.15.3. Discussion sur la robustesse

Ce script est volontairement simple.

Dans un environnement de production, nous devons améliorer plusieurs points :

- gérer les erreurs de lecture ;
    
- éviter de supposer que tous les champs existent ;
    
- tenir compte des conteneurs ;
    
- éviter certains parsings trop fragiles ;
    
- gérer les différences de noyau ;
    
- produire une sortie machine-readable, par exemple JSON.
    

Mais ce script montre déjà que `/proc` permet de construire rapidement un outil d’observation système.

## 3.16. Étude de cas : première analyse d’un serveur

### 3.16.1. Situation

Nous nous connectons à un serveur qui semble lent.

Nous ne savons pas encore si le problème vient :

- du CPU ;
    
- de la mémoire ;
    
- du swap ;
    
- du disque ;
    
- du réseau ;
    
- d’un processus particulier.
    

Nous utilisons uniquement quelques fichiers globaux de `/proc` pour une première analyse.

### 3.16.2. Étape 1 : version et uptime

```bash
cat /proc/version
cat /proc/uptime
```

Nous cherchons à savoir :

- si le noyau est ancien ;
    
- si la machine vient de redémarrer ;
    
- si l’uptime est très long ;
    
- si un redémarrage récent peut expliquer un incident.

### 3.16.3. Étape 2 : CPU visible

```bash
grep -c '^processor' /proc/cpuinfo
grep 'model name' /proc/cpuinfo | head -1
```

Nous identifions la capacité CPU apparente.

Si la charge moyenne est de `8` sur une machine à `2` CPU logiques, la situation est plus préoccupante que sur une machine à `32` CPU logiques.

### 3.16.4. Étape 3 : charge moyenne

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

### 3.16.5. Étape 4 : mémoire disponible

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

### 3.16.6. Étape 5 : hypothèse initiale

À partir de ces informations, nous pouvons formuler une première hypothèse :

```text
La machine présente une charge élevée, avec peu de mémoire disponible et une utilisation importante du swap. Le ralentissement peut être lié à une pression mémoire provoquant des attentes I/O.
```

Ou au contraire :

```text
La charge est faible, la mémoire disponible est correcte, et le swap n’est pas utilisé. Le ralentissement perçu vient probablement d’un service spécifique, du réseau ou d’un stockage distant.
```

Nous ne cherchons pas encore à conclure définitivement. Nous construisons une première lecture du système.

## 3.17. Limites des informations globales

### 3.17.1. Une vue globale ne suffit pas toujours

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

### 3.17.2. Attention aux moyennes

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

### 3.17.3. Attention aux environnements virtualisés

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

## 3.18. Exercices

### Exercice 1 — Identifier le CPU

Nous exécutons :

```bash
grep -c '^processor' /proc/cpuinfo
grep 'model name' /proc/cpuinfo | head -1
grep -m1 '^flags' /proc/cpuinfo
```

Nous répondons aux questions :

1. Combien de CPU logiques sont visibles ?
    
2. Quel est le modèle du processeur ?
    
3. Le processeur expose-t-il `vmx` ou `svm` ?
    
4. Le processeur expose-t-il `aes` ?
    
5. Quelle différence faisons-nous entre cœur physique et CPU logique ?
    


### Exercice 2 — Comprendre la mémoire

Nous exécutons :

```bash
grep -E '^(MemTotal|MemFree|MemAvailable|Buffers|Cached|SwapTotal|SwapFree):' /proc/meminfo
```

Nous répondons aux questions :

1. Quelle est la mémoire totale ?
    
2. Quelle est la mémoire libre ?
    
3. Quelle est la mémoire disponible ?
    
4. Pourquoi `MemFree` peut-il être beaucoup plus faible que `MemAvailable` ?
    
5. Le swap est-il activé ?
    
6. Quelle quantité de swap est utilisée ?
    


### Exercice 3 — Uptime et charge

Nous exécutons :

```bash
cat /proc/uptime
cat /proc/loadavg
nproc
```

Nous répondons aux questions :

1. Depuis combien de temps la machine est-elle démarrée ?
    
2. Quelle est la charge moyenne sur 1, 5 et 15 minutes ?
    
3. Cette charge est-elle élevée par rapport au nombre de CPU logiques ?
    
4. La charge semble-t-elle monter ou descendre ?
    


### Exercice 4 — Comparer fichiers bruts et commandes haut niveau

Nous comparons :

```bash
cat /proc/meminfo | head
free -h
```

Puis :

```bash
cat /proc/loadavg
uptime
```

Puis :

```bash
cat /proc/modules | head
lsmod | head
```

Nous expliquons ce que les commandes haut niveau ajoutent par rapport aux fichiers bruts.


### Exercice 5 — Construire une synthèse système

Nous écrivons un script qui affiche :

```text
Kernel
CPU logiques
Mémoire totale
Mémoire disponible
Swap utilisé
Uptime
Load average
Nombre de processus visibles
```

Nous utilisons uniquement `/proc` autant que possible.


## Conclusion du chapitre 3

Nous savons maintenant lire les principales informations globales exposées par `/proc`.

Nous avons étudié :

- `/proc/cpuinfo` pour le processeur ;
    
- `/proc/meminfo` pour la mémoire ;
    
- `/proc/version` pour le noyau ;
    
- `/proc/uptime` pour le temps de fonctionnement ;
    
- `/proc/loadavg` pour la charge système ;
    
- `/proc/cmdline` pour les paramètres de démarrage ;
    
- `/proc/filesystems` pour les systèmes de fichiers supportés ;
    
- `/proc/mounts` pour les montages actifs ;
    
- `/proc/swaps` pour le swap ;
    
- `/proc/partitions` pour les périphériques bloc ;
    
- `/proc/modules` pour les modules noyau ;
    
- `/proc/interrupts` et `/proc/stat` pour des statistiques plus avancées.
    

Nous retenons surtout que ces fichiers donnent une première lecture de l’état du système. Ils sont indispensables pour diagnostiquer rapidement une machine, mais ils doivent être interprétés avec prudence, surtout dans les environnements virtualisés ou conteneurisés.

Dans le chapitre suivant, nous passons de la vue globale à la vue par processus avec les répertoires `/proc/<PID>`.

---
> [!info] Livre « proc » — chapitre 3/11
> [[proc — Sommaire|Sommaire]] · [[proc — 02 — Architecture interne de proc|← 02 — Architecture interne de proc]] · [[proc — 04 — proc et les processus|04 — proc et les processus →]]
