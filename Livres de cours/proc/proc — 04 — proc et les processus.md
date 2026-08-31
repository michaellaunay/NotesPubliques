---
schema_version: 1
uid: 01M1BQ629PPQ1GRMXBK1CTKSWA
titre: "proc — 04 — proc et les processus"
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
resume: "Chapitre 4 sur 11 du livre « proc » : /proc et les processus. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « proc » — chapitre 4/11
> [[proc — Sommaire|Sommaire]] · [[proc — 03 — Informations globales sur le système|← 03 — Informations globales sur le système]] · [[proc — 05 — Fichiers ouverts et descripteurs|05 — Fichiers ouverts et descripteurs →]]

# Chapitre 4 — `/proc` et les processus
## Objectifs du chapitre

Dans ce chapitre, nous passons de la vision globale du système à l’observation détaillée des processus.

Nous avons vu que `/proc` expose des informations générales sur le noyau, la mémoire, le CPU ou la charge système. Nous étudions maintenant les répertoires de la forme :

```bash
/proc/<PID>
```

Chaque processus actif possède son propre répertoire dans `/proc`. Ce répertoire nous permet d’observer son état, sa ligne de commande, ses variables d’environnement, son utilisateur, ses threads, ses signaux, ses limites, ses fichiers ouverts et son contexte d’exécution.

À la fin de ce chapitre, nous savons :

- identifier les processus visibles dans `/proc` ;
    
- comprendre la structure générale de `/proc/<PID>` ;
    
- lire les fichiers `cmdline`, `comm`, `status` et `stat` ;
    
- interpréter les principaux états d’un processus ;
    
- lire l’environnement d’un processus avec `environ` ;
    
- comprendre les liens `cwd`, `exe` et `root` ;
    
- faire le lien entre `/proc` et les commandes `ps`, `top` et `htop` ;
    
- écrire des commandes simples pour analyser un processus.
    


## 4.1. Les répertoires par PID

### 4.1.1. Rappel : qu’est-ce qu’un PID ?

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


### 4.1.2. Lister les processus visibles via `/proc`

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


### 4.1.3. Observer le processus `1`

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

## 4.2. Structure générale de `/proc/<PID>`

### 4.2.1. Explorer un processus

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

### 4.2.2. Les catégories principales

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

## 4.3. `cmdline` : la ligne de commande du processus

### 4.3.1. Lire `/proc/<PID>/cmdline`

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

### 4.3.2. Affichage lisible de `cmdline`

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

### 4.3.3. Cas des processus noyau

Certains processus ont une ligne de commande vide.

Par exemple, certains threads noyau peuvent apparaître avec une `cmdline` vide.

Nous pouvons rencontrer :

```bash
cat /proc/<PID>/cmdline
```

qui ne produit aucune sortie.

Cela ne signifie pas nécessairement que le processus n’existe pas. Cela peut signifier qu’il ne possède pas de ligne de commande utilisateur classique.

### 4.3.4. Risque de sécurité

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

## 4.4. `comm` : le nom court du processus

### 4.4.1. Lire `/proc/<PID>/comm`

Le fichier `comm` contient le nom court du processus.

```bash
cat /proc/$pid/comm
```

Exemple :

```text
sleep
```

Ce nom est généralement court et ne contient pas toute la ligne de commande.

### 4.4.2. Différence entre `comm` et `cmdline`

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

## 4.5. `status` : état lisible du processus

### 4.5.1. Lire `/proc/<PID>/status`

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

### 4.5.2. Les champs d’identification

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


### 4.5.3. Lire seulement les champs utiles

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


### 4.5.4. Les champs mémoire dans `status`

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


## 4.6. Les états des processus

### 4.6.1. Lire l’état dans `status`

L’état du processus se lit dans :

```bash
grep '^State:' /proc/$pid/status
```

Exemple :

```text
State:  S (sleeping)
```

Le processus `sleep` est naturellement en état sleeping, car il attend l’écoulement du temps.


### 4.6.2. Principaux états

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


### 4.6.3. État `R`

Un processus en état `R` est soit en train de s’exécuter sur un CPU, soit prêt à s’exécuter.

Il est dans la file des tâches exécutables.

Exemple d’observation :

```bash
ps -eo pid,state,comm | awk '$2=="R"'
```

Cet état est normal si le processus travaille réellement.


### 4.6.4. État `S`

L’état `S` est très courant.

Il signifie que le processus dort, mais peut être réveillé par un signal ou un événement.

Exemples :

- un shell qui attend une commande ;
    
- un serveur qui attend une connexion ;
    
- un processus `sleep` ;
    
- une application qui attend une réponse réseau.
    

Cet état n’est pas inquiétant en soi.


### 4.6.5. État `D`

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


### 4.6.6. État `Z`

Un processus zombie est un processus terminé dont le parent n’a pas encore récupéré le code de sortie.

Il ne consomme presque plus de mémoire utile, mais il occupe encore une entrée dans la table des processus.

Nous pouvons les lister avec :

```bash
ps -eo pid,ppid,state,comm | awk '$3=="Z"'
```

Dans `/proc`, un zombie possède encore un répertoire `/proc/<PID>`, mais beaucoup d’informations ne sont plus disponibles comme pour un processus vivant.

Un zombie isolé n’est pas forcément grave. Beaucoup de zombies persistants peuvent indiquer un bug dans le processus parent.


### 4.6.7. État `T`

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


## 4.7. `stat` : statistiques compactes bas niveau

### 4.7.1. Lire `/proc/<PID>/stat`

Le fichier `stat` contient de nombreuses informations compactes sur le processus.

```bash
cat /proc/$pid/stat
```

Exemple simplifié :

```text
18590 (sleep) S 18420 18590 18420 34816 18590 4194304 ...
```

Ce fichier est moins lisible que `status`, mais il est utilisé par de nombreux outils système.


### 4.7.2. Les premiers champs

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


### 4.7.3. Attention au parsing de `stat`

Le parsing de `/proc/<PID>/stat` est piégeux.

Le champ `comm` est entre parenthèses et peut contenir des espaces ou des caractères particuliers.

Une méthode naïve qui découpe simplement sur les espaces peut être incorrecte dans certains cas.

Pour un cours d’introduction, nous retenons :

```text
Nous utilisons /proc/<PID>/status pour la lecture humaine.
Nous utilisons /proc/<PID>/stat avec prudence pour les outils bas niveau.
```


## 4.8. `environ` : environnement du processus

### 4.8.1. Lire `/proc/<PID>/environ`

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


### 4.8.2. À quoi servent les variables d’environnement ?

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


### 4.8.3. Implications de sécurité

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


### 4.8.4. Permissions

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


## 4.9. `cwd`, `exe` et `root`

### 4.9.1. Présentation

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


### 4.9.2. `cwd` : répertoire courant

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


### 4.9.3. `exe` : exécutable lancé

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


### 4.9.4. Cas d’un exécutable supprimé

Il arrive qu’un processus continue d’exécuter un binaire qui a été supprimé ou remplacé sur disque.

Nous pouvons alors voir :

```text
/proc/<PID>/exe -> /usr/bin/app (deleted)
```

Cela signifie que le processus utilise encore l’ancien exécutable chargé en mémoire.

Ce cas est fréquent après une mise à jour : un service doit être redémarré pour utiliser la nouvelle version du binaire.


### 4.9.5. `root` : racine vue par le processus

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


### 4.9.6. Lien avec les conteneurs

Dans un conteneur, `root` permet de comprendre quelle arborescence le processus voit.

Depuis l’hôte, si nous avons les permissions nécessaires, nous pouvons parfois inspecter :

```bash
ls /proc/<PID_CONTENEUR>/root
```

Cela donne accès à la racine du conteneur telle qu’elle est vue par le processus.

Cette capacité est puissante pour le diagnostic, mais elle doit être comprise du point de vue sécurité.


## 4.10. `task` : les threads du processus

### 4.10.1. Observer les threads

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


### 4.10.2. Processus et threads dans Linux

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


### 4.10.3. Utilité

Cette observation est utile pour :

- analyser une application multi-thread ;
    
- comprendre le nombre réel de threads ;
    
- observer des threads bloqués ;
    
- diagnostiquer une saturation ;
    
- vérifier le comportement d’un serveur Java, Node.js, Python, PostgreSQL ou autre.
    

Nous approfondissons cette dimension lorsque nous parlons du scheduling, des performances et de la mémoire.


## 4.11. Lien entre `/proc` et `ps`

### 4.11.1. Observer avec `ps`

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


### 4.11.2. Reproduire une mini-version de `ps`

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


### 4.11.3. Pourquoi `ps` est plus robuste

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


## 4.12. Lien entre `/proc`, `top` et `htop`

### 4.12.1. `top`

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


### 4.12.2. `htop`

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


### 4.12.3. Observer les lectures avec `strace`

Nous pouvons observer ce que lit une commande avec `strace`.

Exemple :

```bash
strace -e openat ps -p $pid -o pid,ppid,state,comm,args
```

Nous pouvons voir des ouvertures de fichiers sous `/proc`.

Cette observation montre que les outils haut niveau ne sont pas magiques : ils lisent, agrègent et formatent des informations.


## 4.13. Cas pratiques

### 4.13.1. Identifier précisément un processus

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


### 4.13.2. Trouver le parent d’un processus

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


### 4.13.3. Identifier un processus bloqué

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


### 4.13.4. Voir les processus d’un utilisateur

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


## 4.14. Pièges et limites

### 4.14.1. Le processus peut disparaître

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


### 4.14.2. Les permissions peuvent bloquer la lecture

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
    


### 4.14.3. Les conteneurs changent la vue

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


### 4.14.4. Ne pas confondre nom et exécutable

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


## 4.15. Exercices

### Exercice 1 — Observer le shell courant

Nous exécutons :

```bash
echo $$
cat /proc/$$/comm
tr '\0' ' ' < /proc/$$/cmdline; echo
grep -E '^(Name|State|Pid|PPid|Uid|Gid|Threads):' /proc/$$/status
ls -l /proc/$$/cwd
ls -l /proc/$$/exe
ls -l /proc/$$/root
```

Nous répondons aux questions :

1. Quel est le PID du shell ?
    
2. Quel est son nom court ?
    
3. Quelle est sa ligne de commande ?
    
4. Quel est son processus parent ?
    
5. Quel est son répertoire courant ?
    
6. Quel binaire exécute-t-il ?
    


### Exercice 2 — Lancer et analyser un processus

Nous lançons :

```bash
sleep 300 &
pid=$!
```

Nous analysons :

```bash
cat /proc/$pid/comm
tr '\0' ' ' < /proc/$pid/cmdline; echo
grep -E '^(Name|State|Pid|PPid|Threads):' /proc/$pid/status
ls -l /proc/$pid/exe
```

Puis nous terminons :

```bash
kill $pid
```

Nous vérifions :

```bash
ls /proc/$pid
```

Nous expliquons pourquoi le répertoire disparaît.


### Exercice 3 — Observer les états

Nous lançons :

```bash
sleep 1000 &
pid=$!
```

Nous lisons son état :

```bash
grep '^State:' /proc/$pid/status
```

Puis nous l’arrêtons :

```bash
kill -STOP $pid
grep '^State:' /proc/$pid/status
```

Nous le reprenons :

```bash
kill -CONT $pid
grep '^State:' /proc/$pid/status
```

Puis nous le terminons :

```bash
kill $pid
```

Nous expliquons les états observés.


### Exercice 4 — Lire les variables d’environnement

Nous observons l’environnement du shell courant :

```bash
tr '\0' '\n' < /proc/$$/environ | head
```

Nous répondons aux questions :

1. Quelles variables reconnaissons-nous ?
    
2. À quoi servent `PATH`, `HOME`, `LANG` ?
    
3. Pourquoi peut-il être dangereux d’y stocker des secrets ?
    
4. Pourquoi cette lecture peut-elle être refusée pour certains processus ?
    


### Exercice 5 — Mini-version de `ps`

Nous écrivons un script `mini-ps.sh` :

```bash
##!/usr/bin/env bash

printf "%8s %8s %-2s %s\n" "PID" "PPID" "S" "NAME"

for dir in /proc/[0-9]*; do
    [ -r "$dir/status" ] || continue

    pid=${dir#/proc/}

    name=$(awk '/^Name:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    state=$(awk '/^State:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    ppid=$(awk '/^PPid:/ {print $2}' "$dir/status" 2>/dev/null) || continue

    printf "%8s %8s %-2s %s\n" "$pid" "$ppid" "$state" "$name"
done
```

Nous le rendons exécutable :

```bash
chmod +x mini-ps.sh
./mini-ps.sh | head
```

Nous comparons avec :

```bash
ps -eo pid,ppid,state,comm | head
```


## 4.16. Ce que nous devons retenir

Nous retenons les points suivants :

1. Chaque processus actif possède un répertoire `/proc/<PID>`.
    
2. Les répertoires numériques de `/proc` représentent les processus visibles dans le namespace courant.
    
3. `/proc/1` représente le PID 1 du namespace courant, pas forcément celui de l’hôte physique.
    
4. `cmdline` donne la ligne de commande complète, avec des arguments séparés par `\0`.
    
5. `comm` donne le nom court du processus.
    
6. `status` donne une vue lisible et très utile du processus.
    
7. `stat` donne une vue compacte et bas niveau, mais plus difficile à parser correctement.
    
8. Les états `R`, `S`, `D`, `Z` et `T` sont essentiels pour le diagnostic.
    
9. `environ` expose les variables d’environnement et peut contenir des informations sensibles.
    
10. `cwd`, `exe` et `root` décrivent le contexte d’exécution du processus.
    
11. `task` permet d’observer les threads.
    
12. Les outils `ps`, `top` et `htop` s’appuient en partie sur ces informations.
    
13. Les scripts qui lisent `/proc/<PID>` doivent gérer les processus qui disparaissent et les permissions refusées.
    
14. Les conteneurs et namespaces modifient la vue que nous avons des processus.
    


## Conclusion du chapitre 4

Nous savons maintenant utiliser `/proc` pour analyser les processus.

Nous passons d’une vision globale du système à une vision détaillée par processus. Cette capacité est fondamentale pour l’administration système, le diagnostic applicatif, la cybersécurité et la compréhension fine de Linux.

Nous savons identifier un processus, lire sa commande, comprendre son état, retrouver son parent, observer son environnement, vérifier son exécutable et inspecter son contexte d’exécution.

Dans le chapitre suivant, nous approfondissons un aspect central du diagnostic : les fichiers ouverts et les descripteurs via `/proc/<PID>/fd`.

---
> [!info] Livre « proc » — chapitre 4/11
> [[proc — Sommaire|Sommaire]] · [[proc — 03 — Informations globales sur le système|← 03 — Informations globales sur le système]] · [[proc — 05 — Fichiers ouverts et descripteurs|05 — Fichiers ouverts et descripteurs →]]
