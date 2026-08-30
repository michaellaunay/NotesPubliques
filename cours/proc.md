---
schema_version: 1
uid: "01M02EX5CEF7YK9JF8DK4YJZS8"
titre: "/proc"
aliases:
  - "/proc"
  - "procfs"
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
resume: "Cours approfondi et pratique sur procfs sous GNU/Linux : processus, mémoire, fichiers ouverts, namespaces, sécurité, PSI, sysctl, conteneurs et diagnostic de production."
niveau: avance
prerequis:
  - "[[GNULinux]]"
  - "[[Les namespaces Linux]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2026-05-24
date_modification: 2026-08-30
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Refonte
> Cours condensé le 30 août 2026 (d'environ 51 000 à 7 800 mots) puis vérifié le même jour : options de montage, PSI, sécurité et études de cas contrôlées. La version longue — exercices détaillés par chapitre, cas pratiques longs, tableaux comparatifs — reste disponible dans l'historique git du dépôt.

# Le système de fichiers `/proc` sous GNU/Linux

> [!abstract] Référence du cours
> Ce cours prend comme référence **Linux 7.2.2**, version stable publiée le 28 août 2026. La plupart des interfaces décrites existent depuis longtemps et sont également disponibles sur les noyaux LTS récents. Leur présence exacte dépend toutefois de la version du noyau, de sa configuration et des mécanismes de sécurité activés.

`/proc` est l'une des interfaces les plus importantes entre l'espace utilisateur et le noyau Linux.

Ce n'est ni un répertoire de fichiers ordinaires, ni une base de données stable à parser arbitrairement. C'est un **pseudo-système de fichiers dynamique**, appelé **procfs**, dont une grande partie du contenu est fabriquée par le noyau au moment de la lecture.

Il permet notamment d'observer :

- les processus et leurs threads ;
- les espaces d'adressage mémoire ;
- les descripteurs de fichiers ;
- les namespaces ;
- les capabilities et l'état seccomp ;
- les informations CPU et mémoire ;
- la pression CPU, mémoire et I/O ;
- une partie des statistiques réseau ;
- de nombreux paramètres du noyau via `/proc/sys`.

Il faut toutefois garder une idée centrale :

> **`/proc` montre une vue du système depuis le contexte du processus qui l'observe.**

Avec les namespaces, les conteneurs, les cgroups et les options de montage de procfs, cette vue peut être volontairement partielle.

---

# 1. Objectifs du cours

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

# 2. Procfs : modèle mental

## 2.1. Un pseudo-système de fichiers

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

## 2.2. Taille apparente de zéro

De nombreuses entrées indiquent une taille nulle :

```bash
stat /proc/meminfo
```

Cela ne signifie pas que le fichier est vide.

La taille affichée par les métadonnées VFS n'est pas nécessairement une indication fiable de la quantité de texte qui sera retournée.

Conséquence :

> Nous ne devons pas écrire un parseur de procfs qui alloue son buffer uniquement à partir de `st_size`.

## 2.3. Une photographie non transactionnelle

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

## 2.4. Une ABI utilisateur, pas une API de haut niveau

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

# 3. `/proc`, `/sys`, `/dev`, `/run` et cgroupfs

## 3.1. `/proc`

Procfs expose principalement :

- processus et threads ;
- informations globales du noyau ;
- statistiques ;
- paramètres `sysctl` sous `/proc/sys`.

## 3.2. `/sys`

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

## 3.3. `/dev`

`/dev` contient principalement les nœuds permettant de communiquer avec des périphériques ou pseudo-périphériques :

```text
/dev/null
/dev/zero
/dev/random
/dev/tty
/dev/nvme0n1
```

## 3.4. `/run`

`/run` contient des **données volatiles de l'espace utilisateur** créées depuis le démarrage :

- PID files historiques ;
- sockets ;
- verrous ;
- état des services ;
- données temporaires de démons.

## 3.5. `/sys/fs/cgroup`

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

# 4. Monter et inspecter procfs

## 4.1. Montage normal

En système complet, `/proc` est généralement monté très tôt au boot.

Nous pouvons l'observer avec :

```bash
findmnt -t proc
```

Dans un environnement isolé pédagogique :

```bash
sudo mount -t proc proc /mnt/proc
```

## 4.2. Options classiques

Des options génériques courantes sont :

```text
nosuid
nodev
noexec
```

Elles ne remplacent toutefois pas les contrôles propres à procfs.

## 4.3. `hidepid`

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

## 4.4. `gid=`

L'option `gid=` permet d'accorder une visibilité particulière à un groupe lorsqu'un `hidepid` restrictif est utilisé.

Nous devons limiter ce groupe aux outils qui ont réellement besoin de cette visibilité.

## 4.5. `subset=pid`

Une instance de procfs montée avec :

```text
subset=pid
```

n'expose que la partie liée aux tâches/processus et masque les autres entrées top-level de procfs. Cette option existe depuis Linux 5.8.

Cette option est intéressante pour réduire la surface visible dans certains environnements isolés.

## 4.6. `pidns=`

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

# 5. Structure générale de `/proc`

Nous pouvons classer les entrées en plusieurs groupes.

## 5.1. Répertoires de processus

```text
/proc/1
/proc/42
/proc/12345
```

Chaque nombre correspond à un **TGID/PID de processus visible dans le PID namespace de cette instance de procfs**.

## 5.2. Processus courant

```text
/proc/self
```

est un lien magique résolu vers le processus qui effectue l'accès.

Exemple :

```bash
readlink /proc/self
```

Le PID affiché correspond à `readlink`, pas nécessairement au shell parent.

## 5.3. Thread courant

```text
/proc/thread-self
```

pointe vers la tâche/thread qui effectue l'accès.

## 5.4. Informations globales

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

## 5.5. Paramètres noyau

```text
/proc/sys/
```

fournit la représentation procfs du mécanisme `sysctl`.

---

# 6. Explorer `/proc/<PID>`

Prenons le shell courant :

```bash
pid=$$
ls -la "/proc/$pid"
```

Parmi les entrées les plus utiles :

```text
cmdline
comm
status
stat
statm
environ
fd/
fdinfo/
limits
maps
smaps
smaps_rollup
mountinfo
cgroup
ns/
task/
exe
cwd
root
io
sched
wchan
```

---

# 7. `cmdline`, `comm` et `exe`

## 7.1. `cmdline`

```bash
cat /proc/$$/cmdline
```

peut produire une sortie peu lisible, car les arguments sont séparés par des octets NUL (`\0`).

Pour les visualiser :

```bash
tr '\0' '\n' < /proc/$$/cmdline
```

ou :

```bash
xargs -0 -n1 < /proc/$$/cmdline
```

> [!warning]
> Les arguments de ligne de commande peuvent contenir des informations sensibles si une application a été mal conçue. Nous ne passons pas des mots de passe ou tokens dans les arguments lorsque nous pouvons l'éviter.

## 7.2. `comm`

```bash
cat /proc/$$/comm
```

fournit un nom court de tâche.

Ce nom ne remplace pas la ligne de commande complète.

## 7.3. `exe`

```bash
readlink -f /proc/$$/exe
```

permet d'identifier l'exécutable associé au processus lorsque les permissions le permettent.

Un suffixe `(deleted)` peut apparaître si le binaire a été supprimé après son ouverture.

---

# 8. `status` : la fiche lisible d'un processus

`/proc/<PID>/status` est l'une des interfaces les plus utiles :

```bash
cat /proc/$$/status
```

## 8.1. Identité

Les champs importants incluent :

```text
Name
State
Tgid
Pid
PPid
Uid
Gid
Groups
Threads
```

## 8.2. PID namespaces

Nous trouvons notamment :

```text
NStgid
NSpid
NSpgid
NSsid
```

Ils permettent d'observer les identifiants d'une tâche dans les différents PID namespaces imbriqués.

Exemple :

```bash
grep '^NSpid:' /proc/1/status
```

Dans un conteneur, cette information permet souvent de faire le lien entre PID vu par l'hôte et PID vu depuis le conteneur.

## 8.3. Mémoire

Quelques champs :

```text
VmPeak
VmSize
VmHWM
VmRSS
RssAnon
RssFile
RssShmem
VmSwap
```

`VmRSS` est une mesure de mémoire résidente, mais elle ne suffit pas à attribuer correctement la mémoire partagée entre processus.

Pour une analyse plus fine, nous utilisons `smaps` ou `smaps_rollup`.

## 8.4. Capabilities

Les champs :

```text
CapInh
CapPrm
CapEff
CapBnd
CapAmb
```

sont des masques hexadécimaux représentant respectivement les ensembles inheritable, permitted, effective, bounding et ambient.

Nous pouvons les décoder avec :

```bash
capsh --decode="$(awk '/^CapEff:/ {print $2}' /proc/$$/status)"
```

## 8.5. `NoNewPrivs`

```bash
grep '^NoNewPrivs:' /proc/$$/status
```

`1` indique que le bit `no_new_privs` est actif : le processus et ses descendants ne peuvent plus obtenir de nouveaux privilèges par certains mécanismes comme un exécutable setuid.

## 8.6. `Seccomp`

```bash
grep -E '^Seccomp|^Seccomp_filters' /proc/$$/status
```

Valeurs principales :

```text
0  seccomp désactivé
1  mode strict
2  filtres seccomp
```

Le nombre de filtres empilés peut être exposé via `Seccomp_filters`.

Ces champs sont très utiles pour auditer un conteneur ou un service systemd durci.

---

# 9. `/proc/<PID>/stat` : puissant mais piégeux

`stat` fournit de nombreux compteurs dans un format compact :

```bash
cat /proc/$$/stat
```

Il alimente de nombreux outils de supervision.

Nous y trouvons notamment :

- PID ;
- `comm` ;
- état ;
- PPID ;
- fautes de page ;
- temps CPU utilisateur/noyau ;
- priorité/nice ;
- nombre de threads ;
- timestamp de démarrage ;
- mémoire virtuelle ;
- RSS.

## 9.1. Ne pas faire `split(" ")` naïvement

Le champ `comm` est placé entre parenthèses et peut contenir des espaces ou caractères surprenants.

Un parseur naïf :

```python
fields = open(path).read().split()
```

peut donc associer les mauvais numéros de champs.

Nous préférons :

- une bibliothèque éprouvée ;
- les fichiers plus structurés lorsque disponibles ;
- ou un parseur explicitement conçu selon la documentation `proc_pid_stat(5)`.

## 9.2. Unités en ticks

Les temps `utime` et `stime` sont exprimés en **clock ticks**, pas directement en secondes.

Nous pouvons obtenir la fréquence avec :

```bash
getconf CLK_TCK
```

---

# 10. Threads et `/proc/<PID>/task`

Linux représente les threads comme des tâches partageant plusieurs ressources.

Pour un processus :

```bash
ls /proc/$$/task
```

Chaque TID possède son propre sous-répertoire :

```text
/proc/<TGID>/task/<TID>/
```

Nous pouvons inspecter le thread courant avec :

```bash
cat /proc/thread-self/status
```

Cette distinction est essentielle avec :

- signaux ;
- affinité CPU ;
- scheduling ;
- debugging ;
- profilage ;
- PID namespaces.

---

# 11. Descripteurs de fichiers : `fd` et `fdinfo`

## 11.1. `/proc/<PID>/fd`

```bash
ls -l /proc/$$/fd
```

Les descripteurs standards sont généralement :

```text
0  stdin
1  stdout
2  stderr
```

Les liens peuvent pointer vers :

```text
fichier régulier
socket:[inode]
pipe:[inode]
anon_inode:[eventpoll]
memfd:...
```

## 11.2. Fichier supprimé mais toujours ouvert

Un cas de diagnostic classique :

```text
/var/log/app.log (deleted)
```

Le fichier n'est plus dans le répertoire, mais l'espace disque reste occupé tant qu'un processus garde le descripteur ouvert.

Recherche :

```bash
sudo lsof +L1
```

ou via `/proc` :

```bash
sudo find /proc/[0-9]*/fd -lname '* (deleted)' -print 2>/dev/null
```

## 11.3. `fdinfo`

```bash
cat /proc/$$/fdinfo/1
```

peut exposer :

- position ;
- flags ;
- mount ID ;
- informations spécifiques au type de FD.

## 11.4. Permissions

La possibilité de dereferencer les liens sous `fd` dépend de contrôles de type ptrace et des credentials.

Il ne faut donc pas supposer que :

```bash
ls -l /proc/$pid/fd
```

fonctionnera pour n'importe quel processus.

---

# 12. Environnement : `/proc/<PID>/environ`

Les variables sont séparées par NUL :

```bash
tr '\0' '\n' < /proc/$$/environ
```

> [!danger] Secrets
> Les variables d'environnement peuvent contenir des tokens, mots de passe, URLs signées ou credentials cloud. Leur exposition est une raison de plus pour appliquer le moindre privilège et éviter de stocker des secrets sensibles dans l'environnement lorsque l'architecture permet mieux.

La lecture d'`environ` n'est pas une API de gestion de secrets.

---

# 13. `cwd`, `root` et vue du système de fichiers

## 13.1. Répertoire courant

```bash
readlink /proc/$$/cwd
```

## 13.2. Racine vue par le processus

```bash
readlink /proc/$$/root
```

Avec un conteneur, un `chroot`, un mount namespace ou `pivot_root`, cette racine peut différer de celle de l'observateur.

Nous pouvons explorer prudemment :

```bash
sudo ls "/proc/$pid/root/"
```

Cette technique est utile pour le diagnostic de conteneurs minimalistes ne contenant pas d'outils de debugging.

---

# 14. Namespaces : `/proc/<PID>/ns`

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

# 15. Cgroups vus depuis `/proc`

## 15.1. `/proc/<PID>/cgroup`

```bash
cat /proc/$$/cgroup
```

Avec une hiérarchie cgroup v2 unifiée, une sortie typique est :

```text
0::/user.slice/user-1000.slice/session-2.scope
```

Ce fichier permet de déterminer **dans quel cgroup** se trouve le processus.

## 15.2. Les limites ne sont pas dans `/proc/<PID>/cgroup`

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

# 16. Mémoire virtuelle : `maps`

```bash
cat /proc/$$/maps
```

Chaque ligne décrit un mapping virtuel :

```text
adresse-début-adresse-fin perms offset dev inode chemin
```

Exemple conceptuel :

```text
55e2...-55e2... r-xp ... /usr/bin/bash
7f3a...-7f3a... r--p ... libc.so.6
7ffc...-7ffc... rw-p ... [stack]
```

## 16.1. Permissions

```text
r  lecture
w  écriture
x  exécution
p  private
s  shared
```

## 16.2. Utilisations

`maps` est utile pour :

- comprendre ASLR ;
- repérer les bibliothèques chargées ;
- trouver pile et heap ;
- comprendre `mmap()` ;
- analyser les permissions W^X ;
- diagnostiquer des mappings volumineux.

---

# 17. `smaps` et `smaps_rollup`

## 17.1. `smaps`

```bash
cat /proc/$pid/smaps
```

ajoute des statistiques détaillées à chaque mapping :

```text
Size
Rss
Pss
Shared_Clean
Shared_Dirty
Private_Clean
Private_Dirty
Swap
AnonHugePages
VmFlags
```

Sa lecture peut être coûteuse, car le noyau doit parcourir davantage d'informations de tables de pages.

Nous ne le pollons pas inutilement à haute fréquence.

## 17.2. PSS

La **Proportional Set Size** répartit le coût d'une page partagée entre ses utilisateurs.

Elle est souvent plus pertinente que RSS pour estimer le coût mémoire imputable à un processus.

## 17.3. `smaps_rollup`

Pour obtenir un résumé :

```bash
cat /proc/$pid/smaps_rollup
```

C'est généralement plus pratique pour une observation ponctuelle de la mémoire du processus.

---

# 18. `/proc/<PID>/io`

```bash
cat /proc/$$/io
```

Des compteurs possibles :

```text
rchar
wchar
syscr
syscw
read_bytes
write_bytes
cancelled_write_bytes
```

Nous devons distinguer :

- octets manipulés par les appels système ;
- octets réellement attribués aux I/O de stockage ;
- effet du page cache ;
- compteurs pouvant changer rapidement.

Pour une analyse globale de stockage, nous complétons avec :

```bash
iostat
pidstat -d
```

ou des outils de tracing adaptés.

---

# 19. Limites de ressources

```bash
cat /proc/$$/limits
```

Nous pouvons y observer les limites souples et dures de type `RLIMIT` :

```text
open files
processes
stack size
locked memory
core file size
```

Pour un service systemd, certaines limites sont configurables directement dans l'unité :

```ini
[Service]
LimitNOFILE=65536
```

Nous préférons la configuration déclarative du service aux modifications ad hoc dans un shell.

---

# 20. Affinité, scheduling et contexte CPU

Plusieurs entrées peuvent aider :

```text
/proc/<PID>/status
/proc/<PID>/sched
/proc/<PID>/schedstat
/proc/<PID>/wchan
```

Dans `status` :

```text
Cpus_allowed
Cpus_allowed_list
Mems_allowed
Mems_allowed_list
```

Nous pouvons comparer avec :

```bash
taskset -pc "$pid"
```

`wchan` peut indiquer une fonction noyau dans laquelle une tâche dort, mais sa visibilité peut être restreinte.

Pour un diagnostic de performance sérieux, nous utilisons généralement également :

- `perf` ;
- `pidstat` ;
- `strace` ;
- PSI ;
- eBPF selon le contexte.

---

# 21. Informations CPU globales

## 21.1. `/proc/cpuinfo`

```bash
cat /proc/cpuinfo
```

fournit des informations dépendantes de l'architecture.

Sur x86, nous pouvons notamment y voir :

- vendor ;
- family/model ;
- modèle commercial ;
- flags CPU ;
- topologie partielle.

Pour une présentation structurée, nous préférons souvent :

```bash
lscpu
```

## 21.2. Flags ≠ permission d'utiliser aveuglément une instruction

Une feature CPU peut dépendre :

- du matériel ;
- du noyau ;
- de l'état du système ;
- du runtime ;
- de la virtualisation.

Nous laissons les bibliothèques spécialisées effectuer leur propre détection lorsque possible.

---

# 22. `/proc/stat` et utilisation CPU

```bash
head /proc/stat
```

Exemple :

```text
cpu  123 4 56 7890 12 0 3 0 0 0
cpu0 ...
cpu1 ...
```

Les champs sont des temps cumulés exprimés en unités liées à `USER_HZ`.

Pour calculer un pourcentage CPU, nous devons prendre **deux snapshots** et travailler sur les différences.

Pseudo-algorithme :

```text
snapshot A
attendre Δt
snapshot B

busy = Δbusy
idle = Δidle
CPU% = busy / (busy + idle)
```

Il ne faut pas interpréter une valeur cumulative comme un pourcentage instantané.

---

# 23. Charge système : `/proc/loadavg`

```bash
cat /proc/loadavg
```

Exemple :

```text
1.42 1.20 0.98 3/1245 5021
```

Les trois premières valeurs sont les moyennes de charge à environ :

```text
1 minute
5 minutes
15 minutes
```

Sous Linux, la charge inclut notamment les tâches exécutables et certaines tâches en attente non interruptible, souvent liées à l'I/O.

Ainsi :

> **load average élevé ≠ CPU forcément saturé.**

Nous croisons avec :

```bash
mpstat
vmstat
pidstat
cat /proc/pressure/cpu
cat /proc/pressure/io
```

---

# 24. Mémoire globale : `/proc/meminfo`

```bash
cat /proc/meminfo
```

Champs utiles :

```text
MemTotal
MemFree
MemAvailable
Buffers
Cached
SwapCached
Active
Inactive
Dirty
Writeback
AnonPages
Mapped
Slab
SReclaimable
SUnreclaim
PageTables
SwapTotal
SwapFree
HugePages_*
AnonHugePages
```

## 24.1. `MemFree` n'est pas « la mémoire disponible »

Linux utilise la RAM disponible pour les caches.

Pour une approximation utilisateur de la mémoire que le système peut récupérer sans swap agressif, `MemAvailable` est généralement plus utile que `MemFree`.

## 24.2. Conteneurs

Encore une fois :

```text
/proc/meminfo
```

ne remplace pas :

```text
/sys/fs/cgroup/memory.current
/sys/fs/cgroup/memory.max
```

pour connaître la situation d'un cgroup.

---

# 25. `/proc/vmstat`

```bash
cat /proc/vmstat
```

Ce fichier expose de nombreux compteurs de VM :

- allocation/libération de pages ;
- page faults ;
- swap ;
- reclaim ;
- dirty/writeback ;
- NUMA selon le système.

Il alimente ou complète des outils comme `vmstat`.

Pour un diagnostic, nous observons souvent les **deltas** plutôt que les valeurs absolues cumulatives.

---

# 26. PSI : Pressure Stall Information

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

## 26.1. `some`

`some` mesure le temps durant lequel **au moins une tâche** est bloquée par manque de la ressource considérée.

## 26.2. `full`

Pour les ressources qui l'exposent, `full` mesure les périodes où **toutes les tâches non-idle concernées** sont simultanément bloquées.

C'est un signal fort de contention.

## 26.3. Pourquoi PSI est meilleur qu'un simple pourcentage

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

## 26.4. PSI cgroup

Cgroup v2 peut également exposer la pression par cgroup sous `/sys/fs/cgroup`.

Pour un conteneur, cette vue est souvent plus pertinente que la pression globale de l'hôte.

---

# 27. Interruptions et softirqs

## 27.1. Interruptions

```bash
cat /proc/interrupts
```

permet d'observer la distribution des interruptions entre CPU.

Applications :

- diagnostic réseau ;
- IRQ imbalance ;
- stockage ;
- affinité IRQ ;
- problèmes de latence.

## 27.2. Softirqs

```bash
cat /proc/softirqs
```

permet d'observer les interruptions logicielles.

Un volume élevé dans `NET_RX` ou `NET_TX` peut être intéressant lors d'un diagnostic réseau, mais doit être interprété dans le contexte de la charge réelle.

---

# 28. Fichiers systèmes utiles

## 28.1. Noyau

```bash
cat /proc/version
uname -a
```

`/proc/version` ne doit pas être utilisé pour déduire aveuglément la distribution utilisateur.

## 28.2. Ligne de commande du noyau

```bash
cat /proc/cmdline
```

Très utile pour diagnostiquer :

- IOMMU ;
- paramètres de boot ;
- console ;
- mitigations ;
- root filesystem ;
- options de debug.

## 28.3. Modules

```bash
cat /proc/modules
```

Une présentation plus pratique peut être obtenue avec :

```bash
lsmod
```

## 28.4. Systèmes de fichiers supportés

```bash
cat /proc/filesystems
```

La présence de `nodev` indique qu'un filesystem n'est pas associé directement à un périphérique bloc.

---

# 29. Montages : préférer `mountinfo`

## 29.1. `/proc/self/mounts`

```bash
cat /proc/self/mounts
```

fournit une vue compatible avec le format historique des tables de montage.

## 29.2. `/proc/self/mountinfo`

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

## 29.3. Utiliser `findmnt`

Pour un outil humain :

```bash
findmnt
findmnt -T /chemin
```

est souvent préférable au parsing manuel.

---

# 30. Réseau : comprendre la place de `/proc/net`

## 30.1. Vue par network namespace

Sur Linux moderne :

```text
/proc/net
```

reflète le network namespace du processus courant et est lié à la vue de `/proc/self/net`.

Ainsi, deux conteneurs peuvent obtenir un contenu différent pour :

```bash
cat /proc/net/dev
```

## 30.2. Interfaces

```bash
cat /proc/net/dev
```

fournit des compteurs par interface.

Pour une interface utilisateur structurée :

```bash
ip -s link
```

## 30.3. TCP/UDP

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

## 30.4. ARP et routes

Les fichiers :

```text
/proc/net/arp
/proc/net/route
```

existent toujours, mais `ip neigh` et `ip route` sont préférables pour l'administration moderne.

---

# 31. `/proc/sys` et `sysctl`

`/proc/sys` expose des paramètres du noyau.

Exemple :

```bash
cat /proc/sys/kernel/hostname
```

équivaut conceptuellement à :

```bash
sysctl kernel.hostname
```

## 31.1. Lire

```bash
sysctl vm.swappiness
```

ou :

```bash
cat /proc/sys/vm/swappiness
```

## 31.2. Modifier temporairement

```bash
sudo sysctl -w vm.swappiness=10
```

ou, selon l'entrée :

```bash
echo 10 | sudo tee /proc/sys/vm/swappiness
```

La modification n'est normalement pas une configuration persistante à travers un reboot.

## 31.3. Configuration persistante

Nous utilisons plutôt :

```text
/etc/sysctl.conf
/etc/sysctl.d/*.conf
```

Exemple :

```ini
# /etc/sysctl.d/60-local.conf
vm.swappiness = 10
```

Puis :

```bash
sudo sysctl --system
```

## 31.4. Ne pas appliquer des recettes sysctl aveuglément

Le noyau documente explicitement que certains paramètres peuvent dégrader fortement ou casser le comportement d'un système.

Nous devons pour chaque réglage connaître :

1. le problème mesuré ;
2. la valeur actuelle ;
3. la documentation du paramètre ;
4. la métrique de succès ;
5. la procédure de rollback.

---

# 32. Quelques paramètres noyau importants

Cette section n'est **pas** une liste de valeurs recommandées universelles.

## 32.1. `kernel.pid_max`

```bash
sysctl kernel.pid_max
```

fixe la borne supérieure de l'espace des PID selon l'architecture et le noyau.

Modifier cette valeur sans besoin concret n'apporte généralement rien.

## 32.2. `kernel.core_pattern`

```bash
sysctl kernel.core_pattern
```

contrôle la destination ou le handler des core dumps.

Sur un système systemd, la valeur peut diriger les dumps vers `systemd-coredump`.

Nous utilisons alors :

```bash
coredumpctl list
coredumpctl info PID_OU_EXE
```

## 32.3. `kernel.randomize_va_space`

```bash
sysctl kernel.randomize_va_space
```

contrôle certains aspects d'ASLR.

Le désactiver pour « simplifier un debug » doit rester exceptionnel et limité à un environnement de laboratoire.

## 32.4. `fs.file-max`

```bash
sysctl fs.file-max
```

est une limite système de file handles.

Elle ne remplace pas les limites par processus/cgroup et ne doit pas être augmentée avant d'avoir identifié la vraie cause d'une exhaustion de FDs.

---

# 33. Sécurité de procfs

Procfs contient beaucoup d'informations utiles à un attaquant local.

## 33.1. Informations sensibles potentielles

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

## 33.2. Contrôles ptrace

De nombreuses entrées sensibles utilisent des vérifications proches des permissions `ptrace`.

Les mécanismes pouvant intervenir incluent :

- UID/GID ;
- dumpability ;
- capabilities ;
- Yama ;
- LSM ;
- user namespaces ;
- options `hidepid`.

## 33.3. Yama

Selon la distribution :

```bash
sysctl kernel.yama.ptrace_scope
```

peut restreindre les possibilités de ptrace entre processus.

Nous devons comprendre la politique de la distribution avant de la modifier.

---

# 34. `kallsyms`, `kcore` et informations noyau sensibles

## 34.1. `/proc/kallsyms`

Cette interface expose la table de symboles du noyau, sous réserve des restrictions de sécurité.

L'affichage d'adresses peut être masqué selon :

```bash
sysctl kernel.kptr_restrict
```

## 34.2. `/proc/kcore`

`/proc/kcore` représente une vue ELF de la mémoire virtuelle du noyau.

C'est une interface très sensible, généralement inaccessible à un utilisateur ordinaire.

Nous ne la copions jamais « pour voir » sur un système de production.

## 34.3. Principe

> Plus une interface permet d'observer l'état interne du noyau, plus nous devons considérer sa lecture comme une opération privilégiée et potentiellement sensible.

---

# 35. `/proc/<PID>/mem` et `pagemap`

## 35.1. `mem`

`/proc/<PID>/mem` donne accès à l'espace d'adressage d'un processus sous des contrôles stricts.

Ce n'est pas une interface d'observation ordinaire.

Pour du debugging :

- `gdb` ;
- `ptrace` ;
- `process_vm_readv()` ;
- outils de profiling dédiés

sont généralement de meilleures primitives.

## 35.2. `pagemap`

```text
/proc/<PID>/pagemap
```

fournit des informations bas niveau sur les pages virtuelles.

L'accès à certaines informations comme les PFN a été volontairement restreint pour limiter des attaques matérielles ou par canaux auxiliaires.

Nous ne supposons donc pas qu'un parseur `pagemap` fonctionnera sans privilèges sur tous les systèmes.

---

# 36. Conteneurs et `/proc`

Un conteneur n'obtient pas un « faux `/proc` » au sens simple : son runtime organise namespaces, montages et cgroups afin de lui présenter une vue cohérente.

## 36.1. PID namespace

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

## 36.2. Network namespace

```text
/proc/net
```

reflète le réseau vu depuis le namespace du lecteur.

## 36.3. Mount namespace

```text
/proc/self/mountinfo
```

montre la topologie des montages vue par le processus.

## 36.4. Cgroups

Le runtime peut limiter CPU/RAM/PIDs via cgroup v2, mais les limites sont principalement observées sous :

```text
/sys/fs/cgroup
```

## 36.5. Pourquoi `/proc` ne suffit pas pour « détecter un conteneur »

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

# 37. systemd et `/proc`

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

# 38. Comment les outils utilisent `/proc`

## 38.1. `ps`

`ps` lit plusieurs informations par processus et applique sa propre logique de présentation.

Nous ne devons pas reconstruire `ps` en shell pour une application de production si `procps-ng` fait déjà le travail.

## 38.2. `top` / `htop`

Ces outils prennent des snapshots successifs pour calculer des taux.

## 38.3. `free`

`free` interprète notamment `/proc/meminfo` plutôt que d'afficher naïvement `MemFree`.

## 38.4. `lsof`

`lsof` combine plusieurs sources et abstrait la complexité de `fd`, sockets, mappings et permissions.

## 38.5. `ss`

Pour les sockets, `ss` est généralement préférable au décodage manuel de `/proc/net/tcp`.

---

# 39. Parsing robuste de procfs

## 39.1. Accepter la disparition d'un processus

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

## 39.2. Parser les formats clé/valeur

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

## 39.3. NUL dans `cmdline`

```python
from pathlib import Path

raw = Path("/proc/self/cmdline").read_bytes()
argv = [part.decode(errors="replace") for part in raw.split(b"\0") if part]
print(argv)
```

## 39.4. Ne pas scraper la sortie humaine si une API existe

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

# 40. Pièges courants

## 40.1. Confondre RSS et consommation réelle

Une bibliothèque partagée peut être comptée dans le RSS de plusieurs processus.

Utiliser PSS lorsque l'attribution proportionnelle est importante.

## 40.2. Additionner les RSS

```text
RSS(P1) + RSS(P2) + ...
```

surestime souvent la consommation réelle à cause des pages partagées.

## 40.3. Utiliser `MemFree`

`MemAvailable` est généralement plus pertinent pour comprendre la marge globale.

## 40.4. Utiliser `/proc/meminfo` comme limite conteneur

Inspecter cgroup v2.

## 40.5. Parser `/proc/<PID>/stat` avec un split simple

Le champ `comm` rend cette approche incorrecte.

## 40.6. Stocker seulement un PID longtemps

Un PID peut être réutilisé.

Pour les applications sensibles, envisager pidfd ou valider l'identité du processus.

## 40.7. Déduire un état stable de plusieurs fichiers

Deux lectures successives ne constituent pas une transaction.

## 40.8. Écrire dans `/proc/sys` sans rollback

Toujours connaître la valeur précédente et documenter l'effet attendu.

## 40.9. Poller `smaps` très fréquemment

La précision a un coût.

## 40.10. Confondre namespace et cgroup

Namespace : visibilité/isolation de ressources.

Cgroup : contrôle/comptabilité de ressources.

---

# 41. Méthode de diagnostic avec `/proc`

Nous utilisons une méthode en couches.

## 41.1. Étape 1 — Identifier le contexte

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

## 41.2. Étape 2 — Santé globale

```bash
cat /proc/loadavg
head /proc/meminfo
cat /proc/pressure/cpu
cat /proc/pressure/memory
cat /proc/pressure/io
```

## 41.3. Étape 3 — Identifier le processus

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

## 41.4. Étape 4 — Examiner les ressources

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

## 41.5. Étape 5 — Corréler avec les événements

```bash
journalctl
journalctl -k
```

Procfs donne un état ; les journaux donnent souvent la chronologie.

---

# 42. Étude de cas — Service lent sans CPU saturé

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

# 43. Étude de cas — Conteneur tué pour OOM malgré `MemAvailable`

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

# 44. Étude de cas — Disque plein mais aucun gros fichier visible

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

# 45. Étude de cas — Vérifier le durcissement d'un service

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

# 46. TP 1 — Cartographier procfs

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

# 47. TP 2 — Analyser notre propre processus

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

# 48. TP 3 — PID namespaces

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

# 49. TP 4 — Détecter un fichier supprimé ouvert

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

# 50. TP 5 — RSS vs PSS

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

# 51. TP 6 — Mesurer un taux CPU à partir de `/proc/stat`

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

# 52. TP 7 — PSI

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

# 53. TP 8 — Sysctl temporaire vs persistant

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

# 54. TP 9 — Audit sécurité d'un processus

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

# 55. TP 10 — Vue cgroup d'un conteneur

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

# 56. TP 11 — Parser `/proc/self/status` en Python

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

# 57. TP 12 — Diagnostic complet

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

# 58. Projet final — `proc-inspect`

Construire un petit outil de diagnostic en Python.

## 58.1. Commande

```bash
proc-inspect PID
```

## 58.2. Sortie minimale

```text
Identity
--------
PID             1234
PPID            1
Name            api
Threads         12

Security
--------
NoNewPrivs      yes
Seccomp         filter
Effective caps  ...

Memory
------
RSS             ...
PSS             ...
Swap            ...

Isolation
---------
PID namespace   ...
NET namespace   ...
Cgroup          ...

Files
-----
Open FDs        ...
Deleted open    ...
```

## 58.3. Exigences

Le programme doit :

- accepter qu'un processus disparaisse pendant l'analyse ;
- gérer les permissions insuffisantes ;
- ne jamais modifier `/proc` ;
- ne jamais afficher le contenu d'`environ` par défaut ;
- documenter les champs utilisés ;
- avoir des tests ;
- fournir une sortie JSON optionnelle ;
- fonctionner sans privilèges lorsque les informations sont disponibles.

## 58.4. Bonus

Ajouter :

- comparaison de deux snapshots ;
- PSI global ;
- lecture des limites cgroup v2 ;
- détection des FDs supprimés ;
- export Markdown pour incident report.

---

# 59. Checklist de diagnostic `/proc`

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

# 60. Aide-mémoire

```bash
# Montage procfs
findmnt -T /proc

# Processus courant
cat /proc/self/status
tr '\0' '\n' < /proc/self/cmdline

# PID namespaces
cat /proc/self/status | grep '^NSpid:'

# Namespaces
ls -l /proc/self/ns

# Cgroup
cat /proc/self/cgroup

# FDs
ls -l /proc/self/fd

# Mémoire processus
cat /proc/self/smaps_rollup

# Limites
cat /proc/self/limits

# CPU global
head /proc/stat

# Mémoire globale
grep -E '^(MemTotal|MemAvailable|SwapTotal|SwapFree):' /proc/meminfo

# Charge
cat /proc/loadavg

# PSI
cat /proc/pressure/cpu
cat /proc/pressure/memory
cat /proc/pressure/io

# Montages
cat /proc/self/mountinfo

# Sysctl
sysctl kernel.hostname
sysctl vm.swappiness

# Sécurité processus
grep -E '^(Cap|NoNewPrivs|Seccomp)' /proc/self/status
```

---

# 61. Questions de révision

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

# 62. Glossaire

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

# 63. Références

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

# Conclusion

`/proc` est une interface extraordinaire pour comprendre Linux, mais son bon usage exige de dépasser le réflexe :

```bash
cat /proc/quelque-chose
```

Nous devons toujours nous demander :

1. **quelle vue du système observons-nous ?**
2. **dans quel namespace sommes-nous ?**
3. **la valeur est-elle cumulative, instantanée ou calculée ?**
4. **la donnée est-elle globale ou limitée par un cgroup ?**
5. **la lecture est-elle autorisée et sûre ?**
6. **une interface de plus haut niveau est-elle préférable ?**
7. **pouvons-nous corréler cette information avec une seconde source ?**

La maîtrise de procfs fournit une excellente passerelle entre l'administration GNU/Linux, la programmation système, l'observabilité, la sécurité et les conteneurs.

Voir également :

- `[[GNULinux]]` ;
- `[[Les namespaces Linux]]` ;
- `[[Initialisation système et des services]]` ;
- `[[Sécurité avancée sous Linux]]` ;
- `[[Docker]]` ;
- `[[Les protocoles de communications]]`.
