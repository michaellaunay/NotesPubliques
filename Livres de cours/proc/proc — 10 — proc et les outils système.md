---
schema_version: 1
uid: 01M1BQ629WC2ZW590ZZAK9RGDE
titre: "proc — 10 — proc et les outils système"
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
resume: "Chapitre 10 sur 11 du livre « proc » : /proc et les outils système. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « proc » — chapitre 10/11
> [[proc — Sommaire|Sommaire]] · [[proc — 09 — Sécurité, permissions et isolation|← 09 — Sécurité, permissions et isolation]] · [[proc — 11 — Limites de proc|11 — Limites de proc →]]

# Chapitre 10 — `/proc` et les outils système
## Objectifs du chapitre

Dans ce chapitre, nous étudions le lien entre `/proc` et les outils système classiques.

Depuis le début du cours, nous lisons directement des fichiers comme :

```bash
/proc/cpuinfo
/proc/meminfo
/proc/loadavg
/proc/<PID>/status
/proc/<PID>/fd
/proc/net/tcp
```

Mais, en pratique, nous utilisons souvent des outils plus confortables :

```bash
ps
top
htop
free
uptime
vmstat
lsof
ss
pidstat
strace
```

Ces outils ne remplacent pas `/proc` : ils s’appuient souvent sur lui, ou sur des interfaces noyau proches, pour produire une vue plus lisible.

À la fin de ce chapitre, nous savons :

- comprendre le rôle de `/proc` comme source d’information pour les outils système ;
    
- relier `ps`, `top`, `htop`, `free`, `uptime`, `lsof`, `ss` et `vmstat` aux données noyau ;
    
- utiliser `strace` pour observer les fichiers ouverts par un outil ;
    
- construire un mini-outil système en Bash ;
    
- comprendre les limites de l’observation par `/proc`.
    


## 10.1. Pourquoi les outils système existent-ils ?

### 10.1.1. `/proc` est complet, mais brut

`/proc` expose énormément d’informations.

Mais ces informations sont souvent :

- nombreuses ;
    
- dispersées ;
    
- peu formatées ;
    
- parfois difficiles à parser ;
    
- parfois encodées ;
    
- dépendantes du noyau ;
    
- difficiles à interpréter directement.
    

Par exemple :

```bash
cat /proc/loadavg
```

donne :

```text
0.35 0.42 0.38 2/842 15321
```

C’est compact, mais peu explicite pour un débutant.

La commande :

```bash
uptime
```

affiche une information plus lisible :

```text
14:32:01 up 3 days,  2:15,  2 users,  load average: 0.35, 0.42, 0.38
```

Nous comprenons donc que les outils système ajoutent :

- du formatage ;
    
- de l’interprétation ;
    
- des calculs ;
    
- des filtres ;
    
- du tri ;
    
- une interface interactive ;
    
- parfois une agrégation de plusieurs sources.
    


### 10.1.2. Les outils comme couches d’abstraction

Nous pouvons représenter la relation ainsi :

```text
Noyau Linux
   |
   v
/proc, /sys, netlink, appels système, cgroups
   |
   v
outils système : ps, top, free, ss, lsof, vmstat...
   |
   v
administrateur, développeur, supervision
```

`/proc` est donc une interface bas niveau.

Les outils système sont des couches de lecture et de présentation.

Nous devons savoir utiliser les deux niveaux :

- les outils pour aller vite ;
    
- `/proc` pour comprendre, vérifier ou diagnostiquer finement.
    


## 10.2. `ps` et `/proc`

### 10.2.1. Rôle de `ps`

La commande `ps` affiche une photographie des processus.

Exemple :

```bash
ps aux
```

ou :

```bash
ps -eo pid,ppid,state,user,comm,args
```

Elle affiche notamment :

- PID ;
    
- utilisateur ;
    
- état ;
    
- consommation CPU ;
    
- consommation mémoire ;
    
- commande ;
    
- arguments ;
    
- parent ;
    
- terminal ;
    
- temps CPU.
    

Une grande partie de ces informations peut être retrouvée dans `/proc/<PID>`.


### 10.2.2. Correspondance entre `ps` et `/proc`

|Information affichée par `ps`|Source possible dans `/proc`|
|---|---|
|PID|nom du répertoire `/proc/<PID>`|
|PPID|`/proc/<PID>/status` ou `/proc/<PID>/stat`|
|état|`/proc/<PID>/status` ou `/proc/<PID>/stat`|
|commande courte|`/proc/<PID>/comm`|
|arguments|`/proc/<PID>/cmdline`|
|utilisateur|`/proc/<PID>/status`|
|mémoire RSS|`/proc/<PID>/status` ou `/proc/<PID>/statm`|
|temps CPU|`/proc/<PID>/stat`|
|terminal|`/proc/<PID>/stat` et informations associées|

`ps` lit donc des informations brutes, les assemble, puis les affiche sous forme tabulaire.


### 10.2.3. Reproduire une petite partie de `ps`

Nous pouvons écrire une mini-version très simplifiée :

```bash
printf "%8s %8s %-2s %-20s %s\n" "PID" "PPID" "S" "NAME" "CMDLINE"

for dir in /proc/[0-9]*; do
    [ -r "$dir/status" ] || continue

    pid=${dir#/proc/}

    name=$(awk '/^Name:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    state=$(awk '/^State:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    ppid=$(awk '/^PPid:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    cmd=$(tr '\0' ' ' < "$dir/cmdline" 2>/dev/null)

    printf "%8s %8s %-2s %-20s %s\n" "$pid" "$ppid" "$state" "$name" "$cmd"
done | head
```

Ce script montre que `ps` n’est pas magique.

Il fait cependant beaucoup plus que ce script : il gère les erreurs, les permissions, les threads, les formats, les tris et les colonnes complexes.


### 10.2.4. Pourquoi `ps` est plus fiable qu’un script naïf

Notre script pédagogique peut échouer dans plusieurs cas :

- un processus disparaît pendant la lecture ;
    
- un fichier devient inaccessible ;
    
- une ligne de commande contient des caractères particuliers ;
    
- un utilisateur n’a pas les permissions ;
    
- un processus noyau n’a pas de `cmdline` ;
    
- un champ attendu est absent ;
    
- un processus change d’état pendant l’analyse.
    

`ps` gère beaucoup de ces cas.

Nous retenons :

```text
Lire /proc nous aide à comprendre.
Utiliser ps nous aide à travailler efficacement.
```


## 10.3. `top`, `htop` et `/proc`

### 10.3.1. Rôle de `top`

La commande :

```bash
top
```

donne une vue dynamique du système.

Elle affiche notamment :

- charge moyenne ;
    
- nombre de tâches ;
    
- CPU utilisé ;
    
- mémoire ;
    
- swap ;
    
- liste des processus ;
    
- consommation CPU par processus ;
    
- consommation mémoire par processus ;
    
- état des processus.
    

Pour construire cette vue, `top` lit plusieurs sources d’information, notamment des fichiers sous `/proc`.


### 10.3.2. Informations globales utilisées

`top` peut exploiter des informations similaires à :

```bash
/proc/stat
/proc/meminfo
/proc/loadavg
/proc/uptime
```

Ces fichiers permettent de connaître :

- les temps CPU cumulés ;
    
- la mémoire totale et disponible ;
    
- la charge moyenne ;
    
- l’uptime ;
    
- le nombre de processus.
    


### 10.3.3. Informations par processus utilisées

Pour chaque processus, `top` peut lire des informations proches de :

```bash
/proc/<PID>/stat
/proc/<PID>/status
/proc/<PID>/cmdline
```

Il peut ensuite calculer :

- le pourcentage CPU ;
    
- la mémoire résidente ;
    
- l’état ;
    
- le temps CPU cumulé ;
    
- la commande ;
    
- l’utilisateur.
    

Le pourcentage CPU est calculé à partir d’une différence entre deux observations dans le temps.


### 10.3.4. Pourquoi `top` observe dans le temps

Un fichier comme `/proc/<PID>/stat` contient des compteurs cumulés.

Pour calculer une consommation CPU instantanée, il faut :

1. lire une première valeur ;
    
2. attendre un intervalle ;
    
3. lire une deuxième valeur ;
    
4. calculer la différence ;
    
5. rapporter cette différence au temps écoulé et au nombre de CPU.
    

C’est ce que font les outils interactifs.

Nous pouvons donc comprendre pourquoi une mesure CPU n’est pas simplement une valeur statique lue dans un fichier.


### 10.3.5. `htop`

`htop` est une interface plus confortable que `top`.

Il ajoute :

- navigation interactive ;
    
- arbre des processus ;
    
- recherche ;
    
- tri visuel ;
    
- barres CPU et mémoire ;
    
- affichage des threads ;
    
- envoi de signaux ;
    
- personnalisation des colonnes.
    

Mais conceptuellement, `htop` repose sur les mêmes grandes idées : lire l’état du noyau, le reformater, l’actualiser régulièrement.


## 10.4. `free` et `/proc/meminfo`

### 10.4.1. Rôle de `free`

La commande :

```bash
free -h
```

affiche une synthèse mémoire :

```text
               total        used        free      shared  buff/cache   available
Mem:            15Gi       4.2Gi       1.1Gi       320Mi       9.7Gi        10Gi
Swap:          2.0Gi          0B       2.0Gi
```

Elle rend lisible ce que `/proc/meminfo` expose en détail.


### 10.4.2. Source principale : `/proc/meminfo`

Nous pouvons comparer :

```bash
cat /proc/meminfo | head -20
free -h
```

`free` utilise notamment des champs comme :

```text
MemTotal
MemFree
MemAvailable
Buffers
Cached
SwapTotal
SwapFree
Shmem
```

Il les présente sous forme synthétique.


### 10.4.3. Interprétation correcte

Le point fondamental reste celui vu au chapitre 3 :

```text
MemFree faible ne signifie pas forcément manque de mémoire.
MemAvailable est souvent plus utile pour évaluer la mémoire réellement disponible.
```

`free` nous aide à éviter une mauvaise lecture en présentant une colonne `available`.

Mais nous devons savoir que cette colonne provient d’une interprétation des données noyau.


## 10.5. `uptime` et `/proc/loadavg`

### 10.5.1. Rôle de `uptime`

La commande :

```bash
uptime
```

affiche :

- l’heure courante ;
    
- le temps depuis le démarrage ;
    
- le nombre d’utilisateurs connectés ;
    
- la charge moyenne à 1, 5 et 15 minutes.
    

Exemple :

```text
14:42:10 up 5 days,  3:12,  2 users,  load average: 0.25, 0.31, 0.29
```


### 10.5.2. Sources liées dans `/proc`

Nous pouvons comparer :

```bash
cat /proc/uptime
cat /proc/loadavg
uptime
```

`/proc/uptime` donne le temps depuis le démarrage.

`/proc/loadavg` donne les charges moyennes.

`uptime` reformate ces informations et les combine avec d’autres informations système.


### 10.5.3. Lire la tendance de charge

Nous utilisons `uptime` pour aller vite, mais nous devons interpréter correctement les trois valeurs :

```text
load average: 8.00, 4.00, 2.00
```

La charge monte.

```text
load average: 2.00, 4.00, 8.00
```

La charge descend.

Nous devons aussi comparer ces valeurs au nombre de CPU logiques :

```bash
nproc
```

ou :

```bash
grep -c '^processor' /proc/cpuinfo
```


## 10.6. `vmstat` et `/proc`

### 10.6.1. Rôle de `vmstat`

La commande :

```bash
vmstat 1
```

affiche périodiquement des statistiques sur :

- processus en attente ;
    
- mémoire ;
    
- swap ;
    
- entrées/sorties ;
    
- interruptions ;
    
- changements de contexte ;
    
- CPU.
    

Exemple :

```text
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 900000 120000 8000000  0    0     1     2  200  500  5  2 92  1  0
```


### 10.6.2. Sources d’information

`vmstat` agrège des informations qui peuvent venir de sources comme :

```text
/proc/stat
/proc/meminfo
/proc/vmstat
/proc/diskstats
```

Selon les versions et les implémentations, l’outil peut utiliser plusieurs interfaces noyau.


### 10.6.3. Lire `/proc/vmstat`

Nous pouvons observer :

```bash
cat /proc/vmstat | head
```

Ce fichier contient de nombreux compteurs liés à la mémoire virtuelle.

Exemples de champs possibles :

```text
pgpgin
pgpgout
pswpin
pswpout
pgfault
pgmajfault
```

Ces compteurs sont cumulés.

Un outil comme `vmstat` calcule des différences entre deux lectures pour produire des valeurs par seconde.


### 10.6.4. Ce que `vmstat` apporte

`vmstat` est très utile pour savoir si une machine souffre de :

- pression mémoire ;
    
- swap actif ;
    
- attente I/O ;
    
- activité disque ;
    
- charge CPU ;
    
- nombreux changements de contexte ;
    
- processus bloqués.
    

`/proc` contient les compteurs.

`vmstat` les met en forme et les rend exploitables en temps réel.


## 10.7. `lsof` et `/proc/<PID>/fd`

### 10.7.1. Rôle de `lsof`

La commande :

```bash
lsof
```

signifie “list open files”.

Elle affiche les fichiers ouverts par les processus.

Exemple :

```bash
sudo lsof -p <PID>
```

Elle peut afficher :

```text
COMMAND PID USER FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   932 root cwd  DIR  259,2     4096  128 /
nginx   932 root txt  REG  259,2  1260032  456 /usr/sbin/nginx
nginx   932 root 1w   REG  259,2   123456  789 /var/log/nginx/access.log
nginx   932 root 2w   REG  259,2    23456  790 /var/log/nginx/error.log
nginx   932 root 6u  IPv4 123456      0t0  TCP *:80 (LISTEN)
```


### 10.7.2. Lien avec `/proc/<PID>/fd`

Nous avons vu au chapitre 5 que :

```bash
ls -l /proc/<PID>/fd
```

affiche les descripteurs ouverts.

`lsof` va plus loin :

- il identifie le type de ressource ;
    
- il indique les modes d’ouverture ;
    
- il résout des noms ;
    
- il associe des sockets réseau ;
    
- il affiche l’utilisateur ;
    
- il permet de filtrer.
    


### 10.7.3. Exemples utiles

Voir les fichiers ouverts par un processus :

```bash
sudo lsof -p <PID>
```

Voir les processus qui utilisent un fichier :

```bash
sudo lsof /chemin/du/fichier
```

Voir les processus qui utilisent un répertoire ou un montage :

```bash
sudo lsof +D /mnt/data
```

Voir les sockets réseau :

```bash
sudo lsof -i -P -n
```

Voir les processus écoutant sur un port :

```bash
sudo lsof -i :8080 -P -n
```


### 10.7.4. Quand préférons-nous `/proc` ?

Nous utilisons directement `/proc` lorsque :

- `lsof` n’est pas installé ;
    
- l’environnement est minimal ;
    
- nous voulons inspecter un descripteur précis ;
    
- nous voulons comprendre la source brute ;
    
- nous cherchons un fichier supprimé encore ouvert ;
    
- nous écrivons un script très léger.
    

Nous utilisons `lsof` lorsque nous voulons une vue lisible et rapide.


## 10.8. `ss` et `/proc/net`

### 10.8.1. Rôle de `ss`

La commande :

```bash
ss
```

permet d’observer les sockets.

Elle remplace largement l’ancien outil `netstat`.

Exemples :

```bash
ss -tan
ss -ltn
sudo ss -tulpen
ss -x
```

Elle affiche :

- sockets TCP ;
    
- sockets UDP ;
    
- sockets Unix ;
    
- ports en écoute ;
    
- connexions établies ;
    
- processus associés ;
    
- files d’attente.
    


### 10.8.2. Comparaison avec `/proc/net`

Nous pouvons comparer :

```bash
cat /proc/net/tcp
ss -tan
```

`/proc/net/tcp` affiche des adresses hexadécimales et des codes d’état.

`ss` affiche :

```text
State      Recv-Q Send-Q Local Address:Port   Peer Address:Port
LISTEN     0      4096   127.0.0.1:8080      0.0.0.0:*
ESTAB      0      0      192.168.1.10:443    192.168.1.20:51322
```

Nous voyons que `ss` décode et présente les informations.


### 10.8.3. Voir les ports en écoute

Commande pratique :

```bash
sudo ss -tulpen
```

Options :

|Option|Rôle|
|---|---|
|`-t`|TCP|
|`-u`|UDP|
|`-l`|sockets en écoute|
|`-p`|processus|
|`-e`|informations étendues|
|`-n`|pas de résolution DNS/service|

Cette commande est souvent la première à utiliser pour savoir quels services écoutent sur une machine.


### 10.8.4. Pourquoi `ss` est préférable à la lecture brute

Lire `/proc/net/tcp` directement est utile pédagogiquement.

Mais en production, `ss` est préférable parce qu’il :

- décode les adresses ;
    
- affiche les ports lisiblement ;
    
- traduit les états TCP ;
    
- associe les processus ;
    
- gère IPv4 et IPv6 ;
    
- affiche TCP, UDP et Unix ;
    
- fournit des filtres puissants.
    


## 10.9. `pidstat` et les statistiques par processus

### 10.9.1. Rôle de `pidstat`

La commande :

```bash
pidstat
```

fait partie du paquet `sysstat`.

Elle permet d’observer l’activité des processus dans le temps.

Exemples :

```bash
pidstat 1
pidstat -r 1
pidstat -d 1
pidstat -u 1
```

Elle peut afficher :

- CPU par processus ;
    
- mémoire ;
    
- fautes de page ;
    
- I/O ;
    
- threads ;
    
- changements de contexte.
    


### 10.9.2. Lien avec `/proc`

`pidstat` exploite des informations provenant de `/proc`, par exemple :

```text
/proc/<PID>/stat
/proc/<PID>/status
/proc/<PID>/io
/proc/stat
```

Comme `top`, il calcule des différences entre lectures successives.


### 10.9.3. Utilité pratique

`pidstat` est très utile lorsqu’un problème est intermittent.

Exemple :

```bash
pidstat -u -r -d 1
```

Nous pouvons observer en temps réel :

- quel processus consomme du CPU ;
    
- quel processus fait des lectures/écritures ;
    
- quel processus voit sa mémoire augmenter ;
    
- quel processus change de comportement.
    

`/proc` fournit les compteurs.

`pidstat` produit une observation temporelle claire.


## 10.10. `strace` pour voir les accès à `/proc`

### 10.10.1. Rôle de `strace`

`strace` permet d’observer les appels système effectués par un programme.

Nous pouvons l’utiliser pour voir quels fichiers un outil ouvre.

Exemple :

```bash
strace -e openat ps -p $$
```

Nous voyons des appels comme :

```text
openat(AT_FDCWD, "/proc/12345/stat", O_RDONLY) = 3
openat(AT_FDCWD, "/proc/12345/status", O_RDONLY) = 3
openat(AT_FDCWD, "/proc/12345/cmdline", O_RDONLY) = 3
```

Cela montre que `ps` lit bien des informations dans `/proc`.


### 10.10.2. Observer `free`

Nous pouvons tester :

```bash
strace -e openat free -h
```

Nous pouvons voir une ouverture de :

```text
/proc/meminfo
```

Cela confirme le lien entre `free` et `/proc/meminfo`.


### 10.10.3. Observer `uptime`

Nous pouvons tester :

```bash
strace -e openat uptime
```

Nous pouvons voir des accès à des fichiers comme :

```text
/proc/uptime
/proc/loadavg
```

selon l’implémentation.


### 10.10.4. Observer `ss`

Nous pouvons utiliser :

```bash
strace -e openat ss -tan
```

Selon les versions, `ss` peut utiliser netlink plutôt que lire uniquement `/proc/net/tcp`.

C’est un point important : tous les outils modernes ne lisent pas exclusivement `/proc`.

Ils peuvent utiliser :

- `/proc` ;
    
- netlink ;
    
- appels système ;
    
- `/sys` ;
    
- cgroups ;
    
- bibliothèques système.
    

Nous retenons donc :

```text
/proc est une source majeure, mais pas la seule interface noyau.
```


## 10.11. Construire un mini-outil système en Bash

### 10.11.1. Objectif

Nous voulons construire un petit outil qui affiche une synthèse système à partir de `/proc`.

Il doit afficher :

- hostname ;
    
- version du noyau ;
    
- uptime ;
    
- charge moyenne ;
    
- CPU logiques ;
    
- mémoire totale ;
    
- mémoire disponible ;
    
- swap utilisé ;
    
- nombre de processus ;
    
- nombre de processus par état.
    


### 10.11.2. Première version simple

Nous créons un fichier :

```bash
nano mini-proc-status.sh
```

Contenu :

```bash
##!/usr/bin/env bash

hostname=$(cat /proc/sys/kernel/hostname 2>/dev/null || hostname)
kernel=$(uname -r)
cpu_count=$(grep -c '^processor' /proc/cpuinfo 2>/dev/null)

uptime_days=$(awk '{ printf "%.2f jours", $1/86400 }' /proc/uptime)
loadavg=$(awk '{ print $1 " " $2 " " $3 }' /proc/loadavg)

mem_total=$(awk '/^MemTotal:/ { printf "%.1f GiB", $2/1024/1024 }' /proc/meminfo)
mem_available=$(awk '/^MemAvailable:/ { printf "%.1f GiB", $2/1024/1024 }' /proc/meminfo)

swap_used=$(awk '
/^SwapTotal:/ { total=$2 }
/^SwapFree:/ { free=$2 }
END { printf "%.1f MiB", (total-free)/1024 }
' /proc/meminfo)

process_count=$(find /proc -maxdepth 1 -regex '/proc/[0-9]+' 2>/dev/null | wc -l)

echo "Hostname           : $hostname"
echo "Kernel             : $kernel"
echo "CPU logiques       : $cpu_count"
echo "Uptime             : $uptime_days"
echo "Load average       : $loadavg"
echo "Mémoire totale     : $mem_total"
echo "Mémoire disponible : $mem_available"
echo "Swap utilisé       : $swap_used"
echo "Processus visibles : $process_count"
```

Nous rendons le script exécutable :

```bash
chmod +x mini-proc-status.sh
```

Puis nous exécutons :

```bash
./mini-proc-status.sh
```


### 10.11.3. Ajouter les états des processus

Nous pouvons ajouter un comptage des états.

```bash
echo
echo "États des processus"
echo "-------------------"

for dir in /proc/[0-9]*; do
    [ -r "$dir/status" ] || continue
    awk '/^State:/ { print $2 }' "$dir/status" 2>/dev/null
done | sort | uniq -c
```

Sortie possible :

```text
      2 D
      3 R
    180 S
      1 Z
```

Nous pouvons intégrer cela au script.


### 10.11.4. Version plus complète

```bash
##!/usr/bin/env bash

set -u

read_file() {
    local file="$1"
    [ -r "$file" ] && cat "$file"
}

hostname=$(read_file /proc/sys/kernel/hostname || hostname)
kernel=$(uname -r)
cpu_count=$(grep -c '^processor' /proc/cpuinfo 2>/dev/null || echo "?")

uptime_days=$(awk '{ printf "%.2f jours", $1/86400 }' /proc/uptime 2>/dev/null || echo "?")
loadavg=$(awk '{ print $1 " " $2 " " $3 }' /proc/loadavg 2>/dev/null || echo "?")

mem_total=$(awk '/^MemTotal:/ { printf "%.1f GiB", $2/1024/1024 }' /proc/meminfo 2>/dev/null || echo "?")
mem_available=$(awk '/^MemAvailable:/ { printf "%.1f GiB", $2/1024/1024 }' /proc/meminfo 2>/dev/null || echo "?")

swap_used=$(awk '
/^SwapTotal:/ { total=$2 }
/^SwapFree:/ { free=$2 }
END {
    if (total == "") print "?"
    else printf "%.1f MiB", (total-free)/1024
}
' /proc/meminfo 2>/dev/null)

process_count=$(find /proc -maxdepth 1 -regex '/proc/[0-9]+' 2>/dev/null | wc -l)

echo "Synthèse système"
echo "================"
echo "Hostname           : $hostname"
echo "Kernel             : $kernel"
echo "CPU logiques       : $cpu_count"
echo "Uptime             : $uptime_days"
echo "Load average       : $loadavg"
echo "Mémoire totale     : $mem_total"
echo "Mémoire disponible : $mem_available"
echo "Swap utilisé       : $swap_used"
echo "Processus visibles : $process_count"

echo
echo "États des processus"
echo "-------------------"

for dir in /proc/[0-9]*; do
    [ -r "$dir/status" ] || continue
    awk '/^State:/ { print $2 }' "$dir/status" 2>/dev/null
done | sort | uniq -c
```

Ce script reste simple, mais il montre comment bâtir un outil d’observation avec `/proc`.


## 10.12. Construire un mini-outil en Python

### 10.12.1. Objectif

Nous pouvons aussi écrire un outil Python.

Python est plus adapté si nous voulons :

- structurer les données ;
    
- gérer les erreurs proprement ;
    
- produire du JSON ;
    
- construire une API ;
    
- intégrer l’outil à une supervision ;
    
- faire des calculs plus complexes.
    


### 10.12.2. Exemple simple

```python
##!/usr/bin/env python3

from pathlib import Path
import os
import json

PROC = Path("/proc")


def read_text(path: Path) -> str | None:
    try:
        return path.read_text().strip()
    except (FileNotFoundError, PermissionError, ProcessLookupError):
        return None


def parse_meminfo() -> dict[str, int]:
    data: dict[str, int] = {}
    content = read_text(PROC / "meminfo")
    if content is None:
        return data

    for line in content.splitlines():
        key, value = line.split(":", 1)
        parts = value.strip().split()
        if parts:
            data[key] = int(parts[0])
    return data


def count_processes() -> int:
    return sum(1 for p in PROC.iterdir() if p.name.isdigit())


def process_states() -> dict[str, int]:
    states: dict[str, int] = {}

    for p in PROC.iterdir():
        if not p.name.isdigit():
            continue

        status = read_text(p / "status")
        if status is None:
            continue

        for line in status.splitlines():
            if line.startswith("State:"):
                state = line.split()[1]
                states[state] = states.get(state, 0) + 1
                break

    return states


def main() -> None:
    mem = parse_meminfo()

    uptime_content = read_text(PROC / "uptime")
    uptime_seconds = None
    if uptime_content:
        uptime_seconds = float(uptime_content.split()[0])

    loadavg = read_text(PROC / "loadavg")

    result = {
        "hostname": read_text(PROC / "sys/kernel/hostname"),
        "kernel": os.uname().release,
        "cpu_logical": sum(
            1
            for line in (read_text(PROC / "cpuinfo") or "").splitlines()
            if line.startswith("processor")
        ),
        "uptime_seconds": uptime_seconds,
        "loadavg": loadavg.split()[:3] if loadavg else None,
        "memory": {
            "MemTotal_kB": mem.get("MemTotal"),
            "MemAvailable_kB": mem.get("MemAvailable"),
            "SwapTotal_kB": mem.get("SwapTotal"),
            "SwapFree_kB": mem.get("SwapFree"),
        },
        "process_count": count_processes(),
        "process_states": process_states(),
    }

    print(json.dumps(result, indent=2, ensure_ascii=False))


if __name__ == "__main__":
    main()
```

Nous pouvons l’enregistrer sous :

```bash
mini_proc_status.py
```

Puis :

```bash
chmod +x mini_proc_status.py
./mini_proc_status.py
```


### 10.12.3. Pourquoi gérer les exceptions ?

Lorsque nous parcourons `/proc`, plusieurs erreurs sont normales :

- le processus disparaît ;
    
- le fichier n’est pas lisible ;
    
- le fichier n’existe plus ;
    
- les permissions refusent l’accès ;
    
- le format diffère.
    

C’est pourquoi nous gérons :

```python
FileNotFoundError
PermissionError
ProcessLookupError
```

Un outil robuste ne considère pas ces cas comme des catastrophes. Il les traite comme des événements normaux lors de l’observation d’un système vivant.


## 10.13. Comparaison des outils

### 10.13.1. Tableau synthétique

|Besoin|Fichier `/proc`|Outil pratique|
|---|---|---|
|Voir les processus|`/proc/<PID>`|`ps`, `top`, `htop`|
|Voir la mémoire|`/proc/meminfo`|`free`, `vmstat`|
|Voir la charge|`/proc/loadavg`|`uptime`, `top`|
|Voir l’uptime|`/proc/uptime`|`uptime`|
|Voir les fichiers ouverts|`/proc/<PID>/fd`|`lsof`|
|Voir les sockets|`/proc/net/*`|`ss`, `lsof -i`|
|Voir les paramètres noyau|`/proc/sys`|`sysctl`|
|Voir les stats CPU|`/proc/stat`|`top`, `mpstat`, `vmstat`|
|Voir les I/O disque|`/proc/diskstats`, `/proc/<PID>/io`|`iostat`, `pidstat -d`|
|Observer les appels système|appels système|`strace`|


### 10.13.2. Quand lire directement `/proc` ?

Nous lisons directement `/proc` lorsque :

- nous voulons comprendre le fonctionnement interne ;
    
- nous sommes dans un environnement minimal ;
    
- un outil haut niveau n’est pas disponible ;
    
- nous voulons écrire un script simple ;
    
- nous voulons vérifier la source brute ;
    
- nous voulons diagnostiquer un cas très précis ;
    
- nous enseignons les systèmes d’exploitation.
    


### 10.13.3. Quand utiliser les outils ?

Nous utilisons les outils lorsque :

- nous voulons aller vite ;
    
- nous avons besoin d’une présentation lisible ;
    
- nous voulons trier ou filtrer ;
    
- nous voulons une vue temps réel ;
    
- nous voulons éviter les erreurs de parsing ;
    
- nous voulons une sortie connue par les administrateurs ;
    
- nous sommes en production.
    

Les deux approches se complètent.


## 10.14. Limites des outils fondés sur `/proc`

### 10.14.1. Les outils voient une photographie

`ps` donne une photographie à un instant donné.

Un processus peut apparaître puis disparaître entre deux observations.

Un état peut changer très rapidement.

Nous devons éviter de surinterpréter une seule lecture.


### 10.14.2. Les outils calculent des moyennes

`top`, `pidstat` ou `vmstat` calculent des valeurs sur des intervalles.

Le résultat dépend :

- de la durée d’échantillonnage ;
    
- de la fréquence ;
    
- du nombre de CPU ;
    
- de la charge ;
    
- du contexte virtualisé ou non.
    

Une mesure à une seconde peut ne pas refléter un pic très court.


### 10.14.3. Les conteneurs changent la vue

Dans un conteneur, les outils lisent souvent `/proc` depuis le point de vue du conteneur.

Cela signifie que :

- `ps` voit les processus du namespace PID ;
    
- `ss` voit le namespace réseau ;
    
- `free` peut parfois être trompeur selon la configuration ;
    
- `top` peut ne pas afficher correctement les limites cgroups ;
    
- les PID diffèrent entre hôte et conteneur.
    

Nous devons donc toujours demander :

```text
Depuis quel contexte l’outil est-il exécuté ?
```


### 10.14.4. Certains outils utilisent d’autres interfaces

Tous les outils ne lisent pas exclusivement `/proc`.

Certains utilisent aussi :

- netlink ;
    
- `/sys`;
    
- cgroups ;
    
- perf events ;
    
- eBPF ;
    
- appels système ;
    
- bibliothèques système.
    

C’est particulièrement vrai pour les outils réseau modernes ou les outils de performance avancés.


## 10.15. Cas pratique : diagnostic rapide avec outils et `/proc`

### 10.15.1. Situation

Nous nous connectons à un serveur lent.

Nous voulons faire un diagnostic rapide.

Nous combinons outils haut niveau et vérification dans `/proc`.


### 10.15.2. Étape 1 : charge et uptime

Outil :

```bash
uptime
```

Source brute :

```bash
cat /proc/loadavg
cat /proc/uptime
```

Nous interprétons la charge par rapport au nombre de CPU :

```bash
nproc
```


### 10.15.3. Étape 2 : mémoire

Outil :

```bash
free -h
```

Source brute :

```bash
grep -E '^(MemTotal|MemFree|MemAvailable|Buffers|Cached|SwapTotal|SwapFree):' /proc/meminfo
```

Nous cherchons :

- mémoire disponible ;
    
- swap utilisé ;
    
- cache ;
    
- pression mémoire potentielle.
    


### 10.15.4. Étape 3 : processus consommateurs

Outil :

```bash
top
```

ou :

```bash
ps -eo pid,ppid,state,%cpu,%mem,comm,args --sort=-%cpu | head
```

Source brute pour un PID suspect :

```bash
cat /proc/<PID>/comm
tr '\0' ' ' < /proc/<PID>/cmdline; echo
grep -E '^(Name|State|Pid|PPid|Threads|VmRSS):' /proc/<PID>/status
```


### 10.15.5. Étape 4 : fichiers ouverts

Outil :

```bash
sudo lsof -p <PID>
```

Source brute :

```bash
sudo ls -l /proc/<PID>/fd
```

Nous cherchons :

- fichiers logs ;
    
- sockets ;
    
- fichiers supprimés ;
    
- beaucoup de descripteurs ;
    
- ressources inattendues.
    


### 10.15.6. Étape 5 : réseau

Outil :

```bash
sudo ss -tulpen
```

Source brute :

```bash
cat /proc/net/tcp
cat /proc/net/udp
cat /proc/net/unix
```

Nous utilisons directement `ss` pour la lisibilité.

Nous utilisons `/proc/net` pour comprendre la représentation brute ou diagnostiquer un environnement minimal.


### 10.15.7. Étape 6 : hypothèse

À partir de ces éléments, nous formulons une hypothèse prudente.

Exemple :

```text
Nous observons une charge élevée, peu de CPU réellement utilisé, plusieurs processus en état D, et une attente I/O élevée dans vmstat. Nous suspectons un problème de stockage ou de système de fichiers plutôt qu’un calcul CPU.
```

Autre exemple :

```text
Nous observons une croissance continue du RSS d’un processus applicatif, sans augmentation proportionnelle du trafic. Nous suspectons une fuite mémoire ou une accumulation de cache applicatif non maîtrisée.
```


## 10.16. Exercices

### Exercice 1 — Relier `ps` à `/proc`

Nous exécutons :

```bash
ps -p $$ -o pid,ppid,state,comm,args
```

Puis :

```bash
cat /proc/$$/comm
tr '\0' ' ' < /proc/$$/cmdline; echo
grep -E '^(State|Pid|PPid):' /proc/$$/status
```

Nous répondons :

1. Quelles informations de `ps` retrouvons-nous dans `/proc` ?
    
2. Quelle information est plus lisible avec `ps` ?
    
3. Quelle information est plus brute dans `/proc` ?
    


### Exercice 2 — Relier `free` à `/proc/meminfo`

Nous exécutons :

```bash
free -h
```

Puis :

```bash
grep -E '^(MemTotal|MemFree|MemAvailable|Buffers|Cached|SwapTotal|SwapFree):' /proc/meminfo
```

Nous répondons :

1. Quels champs de `/proc/meminfo` apparaissent dans `free` ?
    
2. Pourquoi `available` est-il important ?
    
3. Pourquoi `free` est-il plus lisible ?


### Exercice 3 — Relier `uptime` à `/proc`

Nous exécutons :

```bash
uptime
cat /proc/uptime
cat /proc/loadavg
```

Nous répondons :

1. Où retrouvons-nous le temps depuis le démarrage ?
    
2. Où retrouvons-nous la charge moyenne ?
    
3. Que signifient les trois valeurs de load average ?
    


### Exercice 4 — Observer un outil avec `strace`

Nous exécutons :

```bash
strace -e openat free -h
```

Puis :

```bash
strace -e openat uptime
```

Nous répondons :

1. Quels fichiers sous `/proc` sont ouverts ?
    
2. Que prouve cette observation ?
    
3. Pourquoi certains outils peuvent-ils aussi utiliser d’autres interfaces ?


### Exercice 5 — Comparer `/proc/<PID>/fd` et `lsof`

Nous lançons :

```bash
sleep 300 &
pid=$!
```

Puis :

```bash
ls -l /proc/$pid/fd
lsof -p $pid
```

Nous répondons :

1. Quelles informations sont visibles dans les deux cas ?
    
2. Que rajoute `lsof` ?
    
3. Pourquoi `/proc/<PID>/fd` reste utile ?
    
4. Pourquoi `lsof` est plus pratique en production ?
    

Nous terminons :

```bash
kill $pid
```


### Exercice 6 — Comparer `/proc/net/tcp` et `ss`

Nous lançons un serveur local :

```bash
python3 -m http.server 8080 &
pid=$!
```

Puis :

```bash
cat /proc/net/tcp
ss -ltnp 'sport = :8080'
```

Nous répondons :

1. Quel format est le plus lisible ?
    
2. Comment le port 8080 apparaît-il dans `/proc/net/tcp` ?
    
3. Que rajoute `ss` ?
    
4. Pourquoi est-il utile de connaître les deux niveaux ?
    

Nous terminons :

```bash
kill $pid
```

### Exercice 7 — Construire un mini-outil

Nous écrivons un script Bash ou Python qui affiche :

```text
hostname
kernel
uptime
load average
CPU logiques
mémoire totale
mémoire disponible
swap utilisé
nombre de processus
états des processus
```

Nous utilisons `/proc` autant que possible.

Nous comparons ensuite notre sortie avec :

```bash
hostname
uname -r
uptime
free -h
ps
```


## 10.17. Ce que nous devons retenir

Nous retenons les points suivants :

1. `/proc` expose des informations brutes du noyau.
    
2. Les outils système rendent ces informations plus lisibles.
    
3. `ps` s’appuie largement sur `/proc/<PID>`.
    
4. `top` et `htop` observent le système dans le temps.
    
5. `free` synthétise `/proc/meminfo`.
    
6. `uptime` reformate `/proc/uptime` et `/proc/loadavg`.
    
7. `vmstat` exploite des compteurs système cumulés.
    
8. `lsof` enrichit la lecture de `/proc/<PID>/fd`.
    
9. `ss` rend lisibles les informations réseau, parfois via d’autres interfaces noyau modernes.
    
10. `pidstat` observe les processus dans le temps à partir de compteurs.
    
11. `strace` permet de vérifier quels fichiers et appels système un outil utilise.
    
12. Un outil haut niveau peut utiliser `/proc`, mais aussi netlink, `/sys`, cgroups ou d’autres interfaces.
    
13. Lire `/proc` nous aide à comprendre ce que font les outils.
    
14. Utiliser les outils nous aide à diagnostiquer plus vite.
    
15. Dans les conteneurs, les outils héritent des limites de visibilité imposées par les namespaces et cgroups.
    


## Conclusion du chapitre 10

Nous savons maintenant relier `/proc` aux outils système classiques.

Nous comprenons que des commandes comme `ps`, `top`, `free`, `uptime`, `lsof`, `ss`, `vmstat` ou `pidstat` ne sortent pas leurs informations de nulle part : elles lisent, agrègent, calculent et présentent des données fournies par le noyau.

Cette compréhension est importante pour un niveau Master 2. Elle nous permet de ne pas utiliser les outils comme des boîtes noires. Nous savons revenir aux sources brutes, vérifier une information, comprendre une incohérence et écrire nos propres outils d’observation.

Dans le chapitre suivant, nous passons à une série de travaux pratiques structurés pour consolider l’ensemble du cours sur `/proc`.

## 11. Travaux pratiques

## TP 1 — Explorer `/proc`

Nous devons répondre à une série de questions :

- Quelle est la version du noyau ?
- Combien de CPU logiques sont visibles ?
- Quelle quantité de mémoire est disponible ?
- Depuis combien de temps la machine est-elle démarrée ?
- Combien de processus sont actifs ?

Commandes de départ :

```
cat /proc/versioncat /proc/cpuinfocat /proc/meminfocat /proc/uptimels /proc | grep -E '^[0-9]+$' | wc -l
```

## TP 2 — Analyser un processus

Nous lançons un processus simple :

```
sleep 1000
```

Puis nous récupérons son PID et nous analysons :

```
cat /proc/<PID>/statuscat /proc/<PID>/cmdlinels -l /proc/<PID>/fdcat /proc/<PID>/limitscat /proc/<PID>/maps
```

Nous expliquons ce que nous observons.

## TP 3 — Retrouver un fichier supprimé mais encore ouvert

Nous créons un fichier, nous l’ouvrons dans un processus, puis nous le supprimons.

Nous observons que l’espace disque peut rester occupé tant que le processus garde le fichier ouvert.

Nous utilisons :

```
ls -l /proc/<PID>/fd
```

Nous discutons d’un cas réel : logs supprimés mais toujours ouverts par un service.

## TP 4 — Modifier temporairement un paramètre noyau

Nous lisons puis modifions un paramètre réseau ou mémoire.

Exemple :

```
cat /proc/sys/vm/swappinesssudo sysctl -w vm.swappiness=10cat /proc/sys/vm/swappiness
```

Nous expliquons pourquoi il faut être prudent avec ces modifications.

## 12. Étude de cas : diagnostic d’un serveur

Nous simulons un incident de production :

- charge système élevée
    
- mémoire presque saturée
    
- processus qui consomme trop de fichiers ouverts
    
- suspicion de fuite mémoire
    
- logs supprimés mais espace disque toujours occupé
    

Nous utilisons `/proc` pour enquêter.

Nous construisons une démarche :

1. Nous identifions les processus actifs.
    
2. Nous observons la charge système.
    
3. Nous analysons la mémoire globale.
    
4. Nous inspectons le processus suspect.
    
5. Nous regardons ses fichiers ouverts.
    
6. Nous examinons sa mémoire.
    
7. Nous formulons une hypothèse.
    
8. Nous proposons une action corrective.
    

---
> [!info] Livre « proc » — chapitre 10/11
> [[proc — Sommaire|Sommaire]] · [[proc — 09 — Sécurité, permissions et isolation|← 09 — Sécurité, permissions et isolation]] · [[proc — 11 — Limites de proc|11 — Limites de proc →]]
