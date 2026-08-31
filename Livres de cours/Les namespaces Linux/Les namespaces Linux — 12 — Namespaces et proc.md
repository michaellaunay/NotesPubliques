---
schema_version: 1
uid: 01M1BQ624K9EB2MNYK2Q2F30XV
titre: "Les namespaces Linux — 12 — Namespaces et proc"
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
  - namespaces
  - conteneurisation
resume: "Chapitre 12 sur 16 du livre « Les namespaces Linux » : Namespaces et /proc. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 12/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 11 — Namespace time|← 11 — Namespace time]] · [[Les namespaces Linux — 13 — Namespaces et cgroups|13 — Namespaces et cgroups →]]

# Chapitre 12 — Namespaces et `/proc`
## Objectifs du chapitre

Dans ce chapitre, nous faisons le lien entre deux notions centrales de Linux :

```text
les namespaces
/proc
```

Nous avons déjà vu que les namespaces isolent différentes vues du système : processus, réseau, montages, hostname, IPC, utilisateurs, cgroups et temps.

Nous avons aussi étudié `/proc` comme interface d’observation du noyau.

Nous allons maintenant comprendre que `/proc` est l’un des meilleurs moyens d’observer les namespaces, mais aussi que son contenu dépend souvent du namespace depuis lequel nous le lisons.

À la fin de ce chapitre, nous savons :

- observer les namespaces d’un processus avec `/proc/<PID>/ns` ;
    
- comparer les namespaces de deux processus ;
    
- comprendre pourquoi `/proc` change selon le namespace courant ;
    
- relier le namespace PID à `/proc/<PID>` ;
    
- relier le namespace network à `/proc/net` ;
    
- relier le namespace mount à `/proc/mounts` et `/proc/self/mountinfo` ;
    
- relier le namespace user à `uid_map` et `gid_map` ;
    
- relier le namespace cgroup à `/proc/self/cgroup` ;
    
- relier le namespace time à `/proc/self/timens_offsets` ;
    
- comprendre les implications de sécurité de `/proc`.
    

---

## 12.1. `/proc` comme interface d’observation des namespaces

## 12.1.1. Rappel sur `/proc`

`/proc` est un pseudo-système de fichiers.

Il ne contient pas des fichiers classiques stockés sur disque. Il expose des informations produites dynamiquement par le noyau.

Exemples :

```bash
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/loadavg
cat /proc/self/status
```

Dans le contexte des namespaces, `/proc` est particulièrement important, car il permet d’observer :

- les namespaces auxquels appartient un processus ;
    
- la vue des processus ;
    
- la vue réseau ;
    
- la vue des montages ;
    
- les mappings utilisateurs ;
    
- les cgroups ;
    
- certains offsets temporels.
    

---

## 12.1.2. Même chemin, contenu différent

Une idée importante revient souvent :

```text
Le chemin peut être identique, mais le contenu peut changer selon le namespace.
```

Exemples :

```bash
cat /proc/net/dev
cat /proc/sys/kernel/hostname
cat /proc/self/cgroup
cat /proc/mounts
```

Ces chemins peuvent exister dans plusieurs environnements, mais afficher des informations différentes selon le processus qui les lit.

Nous devons donc toujours poser la question :

```text
Depuis quel namespace lisons-nous ce fichier ?
```

---

## 12.2. `/proc/<PID>/ns`

## 12.2.1. Le répertoire principal

Chaque processus possède un répertoire :

```bash
/proc/<PID>/ns
```

Pour le shell courant :

```bash
ls -l /proc/$$/ns
```

Sortie possible :

```text
cgroup -> cgroup:
ipc    -> ipc:
mnt    -> mnt:
net    -> net:
pid    -> pid:
time   -> time:
user   -> user:
uts    -> uts:
```

Chaque lien symbolique représente le namespace du processus pour un type donné.

---

## 12.2.2. Lire un namespace précis

Nous pouvons lire un namespace avec `readlink`.

Exemples :

```bash
readlink /proc/$$/ns/pid
readlink /proc/$$/ns/net
readlink /proc/$$/ns/mnt
readlink /proc/$$/ns/user
```

Sorties possibles :

```text
pid:
net:
mnt:
user:
```

Le nombre entre crochets identifie une instance de namespace.

Deux processus partagent un namespace donné si la valeur est identique pour ce type de namespace.

---

## 12.2.3. Comparer avec le PID 1

Nous comparons notre shell avec le PID 1 :

```bash
for ns in cgroup ipc mnt net pid time user uts; do
    printf "%-8s shell=%-24s pid1=%s\n" \
        "$ns" \
        "$(readlink /proc/$$/ns/$ns 2>/dev/null)" \
        "$(readlink /proc/1/ns/$ns 2>/dev/null)"
done
```

Exemple :

```text
cgroup  shell=cgroup:   pid1=cgroup:
ipc     shell=ipc:      pid1=ipc:
mnt     shell=mnt:      pid1=mnt:
net     shell=net:      pid1=net:
pid     shell=pid:      pid1=pid:
time    shell=time:     pid1=time:
user    shell=user:     pid1=user:
uts     shell=uts:      pid1=uts:
```

Si toutes les valeurs sont identiques, nous sommes probablement dans le même ensemble de namespaces que le PID 1 du système courant.

Si certaines valeurs diffèrent, nous sommes dans un environnement partiellement isolé.

---

## 12.3. Comparer deux processus quelconques

## 12.3.1. Script de comparaison

Nous pouvons écrire une fonction simple :

```bash
compare_ns() {
    pid_a="$1"
    pid_b="$2"

    for ns in cgroup ipc mnt net pid time user uts; do
        a=$(readlink "/proc/$pid_a/ns/$ns" 2>/dev/null || echo "absent")
        b=$(readlink "/proc/$pid_b/ns/$ns" 2>/dev/null || echo "absent")

        if [ "$a" = "$b" ]; then
            printf "%-8s partagé   %s\n" "$ns" "$a"
        else
            printf "%-8s différent %s / %s\n" "$ns" "$a" "$b"
        fi
    done
}
```

Utilisation :

```bash
compare_ns $$ 1
```

Ou pour comparer deux processus :

```bash
compare_ns <PID_A> <PID_B>
```

---

## 12.3.2. Interpréter les différences

Si seul `uts` diffère, les processus peuvent avoir un hostname différent.

Si `net` diffère, ils ne voient pas la même pile réseau.

Si `pid` diffère, ils ne voient pas la même table de processus.

Si `mnt` diffère, ils ne voient pas la même table de montages.

Si `user` diffère, ils n’interprètent pas forcément les UID/GID de la même manière.

Nous pouvons donc diagnostiquer rapidement quel type d’isolation est en place.

---

## 12.4. Namespace PID et `/proc`

## 12.4.1. `/proc` comme vue des processus

Le namespace PID influence la vue des processus.

Dans un namespace PID isolé, `/proc` doit refléter les processus visibles dans ce namespace.

Nous créons un namespace PID propre :

```bash
unshare --fork --pid --mount-proc bash
```

Dans ce shell :

```bash
ps -ef
ls /proc | grep -E '^[0-9]+$'
cat /proc/1/comm
```

Nous voyons une table de processus réduite.

---

## 12.4.2. Pourquoi `--mount-proc` est essentiel

Si nous créons un namespace PID sans remonter `/proc` :

```bash
unshare --fork --pid bash
```

puis :

```bash
ps -ef
```

nous pouvons obtenir une vue incohérente.

Le processus est bien dans un nouveau namespace PID, mais `/proc` peut encore être celui du namespace mount parent.

Nous retenons :

```text
Namespace PID isolé + /proc non adapté = observation trompeuse.
```

C’est pour cela que nous utilisons :

```bash
unshare --fork --pid --mount-proc bash
```

---

## 12.4.3. Observer les PID imbriqués avec `NSpid`

Depuis l’hôte, nous pouvons observer un processus dans un namespace PID enfant :

```bash
grep NSpid /proc/<PID_HOTE>/status
```

Exemple :

```text
NSpid:  24531  1
```

Cela signifie que le même processus est vu :

- comme PID `24531` dans le namespace parent ;
    
- comme PID `1` dans le namespace enfant.
    

Cette ligne est très utile pour comprendre les conteneurs.

---

## 12.5. Namespace network et `/proc/net`

## 12.5.1. `/proc/net` dépend du namespace réseau

Le répertoire :

```bash
/proc/net
```

donne des informations sur la pile réseau.

Mais son contenu dépend du namespace réseau du processus qui lit ces fichiers.

Exemples :

```bash
cat /proc/net/dev
cat /proc/net/tcp
cat /proc/net/udp
cat /proc/net/unix
```

Dans deux namespaces réseau différents, ces fichiers peuvent afficher des informations différentes.

---

## 12.5.2. Démonstration avec `ip netns`

Nous créons un namespace réseau :

```bash
sudo ip netns add ns1
sudo ip netns exec ns1 ip link set lo up
```

Nous comparons :

```bash
cat /proc/net/dev
sudo ip netns exec ns1 cat /proc/net/dev
```

Sur l’hôte, nous pouvons voir :

```text
lo
eth0
docker0
wlan0
```

Dans `ns1`, nous pouvons voir seulement :

```text
lo
```

Même chemin, contenu différent.

---

## 12.5.3. Comparer les sockets

Sur l’hôte :

```bash
ss -tulpen
```

Dans le namespace :

```bash
sudo ip netns exec ns1 ss -tulpen
```

Les ports en écoute ne sont pas les mêmes.

Un serveur qui écoute dans `ns1` n’apparaît pas nécessairement dans la vue réseau de l’hôte comme un socket local classique.

---

## 12.5.4. Nettoyage

```bash
sudo ip netns delete ns1
```

---

## 12.6. Namespace UTS et `/proc/sys/kernel/hostname`

## 12.6.1. Le hostname via `/proc`

Le hostname actif est visible dans :

```bash
cat /proc/sys/kernel/hostname
```

Mais ce fichier dépend du namespace UTS.

Nous testons :

```bash
hostname
cat /proc/sys/kernel/hostname
```

Puis :

```bash
unshare --uts bash
hostname ns-proc-uts
hostname
cat /proc/sys/kernel/hostname
exit
```

Après `exit` :

```bash
hostname
cat /proc/sys/kernel/hostname
```

Nous retrouvons le hostname de l’hôte.

---

## 12.6.2. Même fichier, valeur différente

Dans le namespace UTS isolé :

```text
/proc/sys/kernel/hostname → ns-proc-uts
```

Sur l’hôte :

```text
/proc/sys/kernel/hostname → machine-hote
```

Le chemin est identique, mais la valeur dépend du namespace UTS.

C’est une excellente illustration du rôle de `/proc`.

---

## 12.7. Namespace mount et `/proc/mounts`

## 12.7.1. Observer les montages

La vue des montages est visible avec :

```bash
cat /proc/self/mounts
cat /proc/self/mountinfo
findmnt
```

Ces informations dépendent du namespace mount.

Un processus dans un namespace mount isolé peut voir des montages que l’hôte ne voit pas, ou inversement.

---

## 12.7.2. Démonstration

Nous créons un namespace mount :

```bash
unshare --mount bash
mount --make-rprivate /
mkdir -p /tmp/proc-mnt-demo
mount -t tmpfs tmpfs /tmp/proc-mnt-demo
```

Dans le shell isolé :

```bash
findmnt /tmp/proc-mnt-demo
grep proc-mnt-demo /proc/self/mountinfo
```

Depuis l’hôte :

```bash
findmnt /tmp/proc-mnt-demo
grep proc-mnt-demo /proc/self/mountinfo
```

Le montage ne doit normalement pas apparaître depuis l’hôte si la propagation a été rendue privée.

---

## 12.7.3. `/proc/<PID>/mountinfo`

Depuis l’hôte, nous pouvons lire les montages vus par un processus :

```bash
cat /proc/<PID>/mountinfo
```

C’est très utile pour analyser un conteneur.

Nous pouvons aussi entrer dans son namespace mount :

```bash
sudo nsenter --target <PID> --mount findmnt
```

Nous comparons alors :

```text
vue des montages depuis l’hôte
vue des montages depuis le namespace du processus
```

---

## 12.7.4. Nettoyage

Dans le namespace isolé :

```bash
umount /tmp/proc-mnt-demo
exit
```

---

## 12.8. Namespace user et UID/GID maps

## 12.8.1. Fichiers principaux

Le namespace user est observable avec :

```bash
readlink /proc/$$/ns/user
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Ces fichiers indiquent comment les UID et GID sont traduits entre l’intérieur et l’extérieur du namespace.

---

## 12.8.2. Exemple avec `--map-root-user`

Nous lançons :

```bash
unshare --user --map-root-user bash
```

Dans le shell :

```bash
id
readlink /proc/$$/ns/user
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Sortie possible :

```text
uid=0(root) gid=0(root)
```

Puis :

```text
         0       1000          1
```

Cela signifie que l’UID `0` dans le namespace correspond à l’UID `1000` dans le namespace parent.

---

## 12.8.3. Diagnostiquer les permissions

Quand un fichier a des propriétaires inattendus dans un conteneur rootless, nous devons comparer :

Dans le conteneur ou namespace :

```bash
id
ls -ln /chemin
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Sur l’hôte :

```bash
ls -ln /chemin
```

Nous devons raisonner avec les UID/GID numériques.

Les noms comme `root`, `www-data` ou `user` peuvent être trompeurs, car ils dépendent de `/etc/passwd` et `/etc/group` dans la vue courante.

---

## 12.9. Namespace cgroup et `/proc/self/cgroup`

## 12.9.1. Fichiers importants

Pour le namespace cgroup :

```bash
readlink /proc/$$/ns/cgroup
cat /proc/self/cgroup
```

Le fichier :

```bash
/proc/self/cgroup
```

indique la position du processus dans l’arborescence cgroup telle qu’elle est vue depuis le namespace cgroup courant.

---

## 12.9.2. Vue simplifiée dans les conteneurs

Dans un conteneur, nous pouvons voir :

```text
0::/
```

Depuis l’hôte, pour le même processus, nous pouvons voir :

```text
0::/system.slice/docker-abcdef.scope
```

ou, en Kubernetes :

```text
0::/kubepods.slice/kubepods-burstable.slice/...
```

Le namespace cgroup peut donc masquer une partie du chemin réel de l’hôte.

---

## 12.9.3. Ne pas confondre vue et limite

Le namespace cgroup ne limite pas directement les ressources.

Pour voir les limites effectives en cgroups v2, nous lisons plutôt :

```bash
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/pids.max
```

Nous retenons :

```text
/proc/self/cgroup montre une position vue.
/sys/fs/cgroup expose les contrôles et mesures.
```

---

## 12.10. Namespace IPC et `/proc`

## 12.10.1. Observer le namespace IPC

Nous lisons :

```bash
readlink /proc/$$/ns/ipc
```

Pour comparer avec le PID 1 :

```bash
readlink /proc/1/ns/ipc
```

Le namespace IPC influence la visibilité de certaines ressources IPC.

---

## 12.10.2. `ipcs`

La commande :

```bash
ipcs
```

affiche les ressources IPC System V visibles dans le namespace IPC courant.

Nous pouvons comparer :

```bash
ipcs
unshare --ipc bash
ipcs
exit
```

Les ressources visibles peuvent différer.

---

## 12.10.3. `/dev/shm`

Pour la mémoire partagée POSIX, nous regardons souvent :

```bash
df -h /dev/shm
ls -lh /dev/shm
findmnt /dev/shm
```

Mais `/dev/shm` dépend aussi du namespace mount.

C’est un bon exemple d’interaction entre namespaces :

```text
IPC namespace → ressources IPC visibles
mount namespace → montage /dev/shm visible
```

---

## 12.11. Namespace time et `/proc/self/timens_offsets`

## 12.11.1. Observer le namespace time

Nous lisons :

```bash
readlink /proc/$$/ns/time
readlink /proc/1/ns/time
```

Nous pouvons aussi voir :

```bash
ls -l /proc/$$/ns/time_for_children
```

selon le noyau.

---

## 12.11.2. Lire les offsets

Le fichier principal est :

```bash
cat /proc/self/timens_offsets
```

Sortie possible :

```text
monotonic           0         0
boottime            0         0
```

Ces valeurs indiquent les offsets appliqués à certaines horloges.

---

## 12.11.3. Attention à l’interprétation

Le namespace time ne signifie pas que :

```bash
date
```

affiche forcément une autre date.

Il concerne surtout des offsets pour certaines horloges monotones ou liées au boottime.

Nous devons donc distinguer :

- heure réelle ;
    
- fuseau horaire ;
    
- uptime ;
    
- temps monotone ;
    
- offsets de namespace time.
    

---

## 12.12. `/proc/self`, `/proc/thread-self` et namespaces

## 12.12.1. `/proc/self`

Le lien :

```bash
/proc/self
```

pointe toujours vers le processus courant.

Exemple :

```bash
readlink /proc/self
```

Sortie possible :

```text
12345
```

Dans un namespace PID, ce PID est interprété selon la vue courante.

`/proc/self` est donc très pratique pour écrire des commandes indépendantes du PID numérique.

---

## 12.12.2. `/proc/thread-self`

Le lien :

```bash
/proc/thread-self
```

pointe vers le thread courant.

Il est utile pour les programmes multi-threads.

Dans les manipulations de namespaces de base, nous utilisons surtout `/proc/self`, mais il est important de savoir que `/proc` peut aussi exposer la granularité des threads.

---

## 12.13. Entrer dans les namespaces d’un processus avec `/proc`

## 12.13.1. `nsenter` utilise les références de `/proc`

La commande `nsenter` s’appuie sur les namespaces d’un processus cible.

Exemple :

```bash
sudo nsenter --target <PID> --net bash
```

Elle rejoint le namespace réseau exposé par :

```bash
/proc/<PID>/ns/net
```

Même principe pour :

```bash
sudo nsenter --target <PID> --mount --uts --ipc --net --pid --fork bash
```

---

## 12.13.2. Entrer dans plusieurs namespaces

Pour diagnostiquer un conteneur depuis l’hôte, nous pouvons faire :

```bash
pid=<PID_DU_CONTENEUR>

sudo nsenter --target "$pid" \
    --mount \
    --uts \
    --ipc \
    --net \
    --pid \
    --fork \
    bash
```

Puis :

```bash
hostname
ps -ef
ip addr
findmnt
cat /proc/self/cgroup
```

Nous observons une vue proche de celle du conteneur.

---

## 12.13.3. Pourquoi `--fork` avec `--pid`

Quand nous entrons dans un namespace PID avec `nsenter`, nous utilisons souvent :

```bash
--fork
```

Cela permet de lancer un enfant dans le namespace PID cible et d’obtenir une vue plus cohérente.

Sans cela, certaines observations peuvent être surprenantes.

---

## 12.14. Analyse d’un conteneur avec `/proc`

## 12.14.1. Récupérer le PID hôte

Avec Docker :

```bash
docker inspect --format '{{.State.Pid}}' <container>
```

Exemple :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' mon-conteneur)
```

---

## 12.14.2. Observer les namespaces du conteneur

Depuis l’hôte :

```bash
sudo ls -l /proc/$pid/ns
```

Nous comparons avec l’hôte :

```bash
ls -l /proc/1/ns
```

Nous pouvons identifier quels namespaces sont isolés.

---

## 12.14.3. Observer la vue PID

```bash
sudo grep NSpid /proc/$pid/status
```

Exemple :

```text
NSpid:  24531  1
```

Nous comprenons le PID hôte et le PID interne.

---

## 12.14.4. Observer les montages

```bash
sudo cat /proc/$pid/mountinfo | head
```

Ou :

```bash
sudo nsenter --target "$pid" --mount findmnt | head
```

Nous voyons le rootfs, `/proc`, `/sys`, `/dev`, les volumes et les bind mounts.

---

## 12.14.5. Observer le réseau

```bash
sudo nsenter --target "$pid" --net ip addr
sudo nsenter --target "$pid" --net ip route
sudo nsenter --target "$pid" --net ss -tulpen
```

Nous observons les interfaces, les routes et les sockets du conteneur.

---

## 12.14.6. Observer les cgroups

```bash
sudo cat /proc/$pid/cgroup
```

Et depuis le conteneur :

```bash
docker exec <container> cat /proc/self/cgroup
```

Nous comparons la vue de l’hôte et la vue interne.

---

## 12.15. Sécurité de `/proc` avec les namespaces

## 12.15.1. Informations sensibles

`/proc` peut exposer des informations sensibles :

```text
/proc/<PID>/cmdline
/proc/<PID>/environ
/proc/<PID>/fd
/proc/<PID>/maps
/proc/<PID>/mountinfo
/proc/<PID>/cgroup
/proc/net/*
```

Ces fichiers peuvent révéler :

- arguments de lancement ;
    
- tokens passés par erreur en ligne de commande ;
    
- variables d’environnement ;
    
- fichiers ouverts ;
    
- chemins internes ;
    
- sockets ;
    
- structure de déploiement ;
    
- informations cgroup ou conteneur.
    

---

## 12.15.2. Exemples dangereux

Secrets en ligne de commande :

```bash
app --token secret123
```

Visible potentiellement dans :

```bash
tr '\0' ' ' < /proc/<PID>/cmdline
```

Secrets en environnement :

```bash
DATABASE_URL=postgres://user:password@host/db
```

Visible potentiellement dans :

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

Fichiers ouverts :

```bash
ls -l /proc/<PID>/fd
```

peut révéler des fichiers sensibles.

---

## 12.15.3. `hidepid`

Sur un serveur multi-utilisateurs, nous pouvons limiter la visibilité de `/proc` avec l’option de montage `hidepid`.

Vérifier :

```bash
findmnt /proc
```

Exemple de configuration :

```fstab
proc /proc proc defaults,nosuid,nodev,noexec,hidepid=2 0 0
```

`hidepid=2` masque davantage les processus des autres utilisateurs.

Cela ne remplace pas les autres protections, mais réduit l’exposition.

---

## 12.15.4. `/proc` dans les conteneurs

Dans un conteneur, `/proc` doit être monté avec prudence.

Monter le `/proc` de l’hôte dans un conteneur est dangereux :

```bash
docker run -v /proc:/host/proc image
```

Cela peut exposer des informations de l’hôte.

Nous préférons laisser le runtime monter un `/proc` adapté au namespace PID du conteneur.

---

## 12.16. Interactions entre namespaces

## 12.16.1. Aucun namespace ne suffit seul

Nous avons vu que plusieurs vues de `/proc` dépendent de plusieurs namespaces à la fois.

Exemple :

```text
ps dépend de /proc
/proc dépend du namespace mount
la vue des PID dépend du namespace PID
```

Donc, pour que `ps` soit cohérent dans un conteneur, il faut :

- un namespace PID ;
    
- un namespace mount adapté ;
    
- un montage correct de procfs.
    

---

## 12.16.2. Exemple avec `/dev/shm`

Pour `/dev/shm` :

```text
namespace IPC → ressources IPC
namespace mount → montage /dev/shm
cgroups → limite mémoire éventuelle
```

Un problème de mémoire partagée en conteneur peut donc venir de plusieurs couches.

---

## 12.16.3. Exemple avec le réseau

Pour le réseau :

```text
namespace network → interfaces, routes, sockets
namespace mount → fichiers de configuration visibles
/proc/net → vue réseau
cgroups → limites éventuelles
capabilities → droit de configurer le réseau
```

Un diagnostic correct doit croiser les informations.

---

## 12.17. Méthode de diagnostic avec `/proc`

## 12.17.1. Étape 1 : identifier le processus

Nous commençons par trouver le PID :

```bash
ps aux | grep <nom>
```

ou, pour Docker :

```bash
docker inspect --format '{{.State.Pid}}' <container>
```

---

## 12.17.2. Étape 2 : observer les namespaces

```bash
sudo ls -l /proc/<PID>/ns
```

Puis comparer avec l’hôte :

```bash
ls -l /proc/1/ns
```

Nous identifions les namespaces partagés et différents.

---

## 12.17.3. Étape 3 : observer chaque vue utile

Pour les PID :

```bash
sudo grep NSpid /proc/<PID>/status
```

Pour les montages :

```bash
sudo cat /proc/<PID>/mountinfo | head
```

Pour le réseau :

```bash
sudo nsenter --target <PID> --net ip addr
sudo nsenter --target <PID> --net ss -tulpen
```

Pour les utilisateurs :

```bash
sudo cat /proc/<PID>/uid_map
sudo cat /proc/<PID>/gid_map
```

Pour les cgroups :

```bash
sudo cat /proc/<PID>/cgroup
```

---

## 12.17.4. Étape 4 : entrer dans le contexte si nécessaire

```bash
sudo nsenter --target <PID> \
    --mount \
    --uts \
    --ipc \
    --net \
    --pid \
    --fork \
    bash
```

Puis :

```bash
hostname
ps -ef
ip addr
findmnt
cat /proc/self/cgroup
id
```

Nous obtenons une vue proche de celle du processus cible.

---

## 12.18. Exercices

## Exercice 1 — Observer tous les namespaces du shell

Nous exécutons :

```bash
ls -l /proc/$$/ns
```

Nous répondons :

1. Quels namespaces voyons-nous ?
    
2. Quel est l’identifiant du namespace PID ?
    
3. Quel est l’identifiant du namespace réseau ?
    
4. Quel est l’identifiant du namespace mount ?
    
5. Ces identifiants sont-ils les mêmes que ceux du PID 1 ?
    

---

## Exercice 2 — Comparer deux processus

Nous lançons :

```bash
unshare --uts bash
hostname ns-test
echo $$
sleep 1000
```

Dans un autre terminal :

```bash
for ns in cgroup ipc mnt net pid time user uts; do
    echo "$ns"
    readlink /proc/<PID>/ns/$ns 2>/dev/null
    readlink /proc/1/ns/$ns 2>/dev/null
    echo
done
```

Nous répondons :

1. Quel namespace diffère ?
    
2. Pourquoi ?
    
3. Les autres namespaces sont-ils partagés ?
    
4. Comment isoler aussi le namespace PID ?
    
5. Comment isoler aussi le namespace réseau ?
    

---

## Exercice 3 — PID namespace et `/proc`

Nous exécutons :

```bash
unshare --fork --pid --mount-proc bash
```

Dans le shell :

```bash
ps -ef
cat /proc/1/comm
ls /proc | grep -E '^[0-9]+$'
```

Nous répondons :

1. Quels processus voyons-nous ?
    
2. Pourquoi `/proc/1` ne désigne-t-il pas le PID 1 de l’hôte ?
    
3. Pourquoi `--mount-proc` est-il nécessaire ?
    

---

## Exercice 4 — Network namespace et `/proc/net`

Nous exécutons :

```bash
sudo ip netns add ns1
sudo ip netns exec ns1 ip link set lo up

cat /proc/net/dev
sudo ip netns exec ns1 cat /proc/net/dev

sudo ip netns delete ns1
```

Nous répondons :

1. Les interfaces visibles sont-elles les mêmes ?
    
2. Pourquoi `/proc/net/dev` change-t-il ?
    
3. Quel namespace influence ce fichier ?
    

---

## Exercice 5 — Mount namespace et `/proc/self/mountinfo`

Nous exécutons :

```bash
unshare --mount bash
mount --make-rprivate /
mkdir -p /tmp/ns-proc-mount
mount -t tmpfs tmpfs /tmp/ns-proc-mount
grep ns-proc-mount /proc/self/mountinfo
```

Depuis l’hôte :

```bash
grep ns-proc-mount /proc/self/mountinfo
```

Nous répondons :

1. Le montage est-il visible dans les deux contextes ?
    
2. Quel namespace explique cette différence ?
    
3. Pourquoi la propagation des montages compte-t-elle ?
    

---

## Exercice 6 — User namespace et mappings

Nous exécutons :

```bash
unshare --user --map-root-user bash
id
cat /proc/$$/uid_map
cat /proc/$$/gid_map
exit
```

Nous répondons :

1. Quel UID voyons-nous dans le namespace ?
    
2. Vers quel UID externe est-il mappé ?
    
3. Pourquoi cela ne donne-t-il pas les droits root sur l’hôte ?
    

---

## Exercice 7 — Cgroup namespace et `/proc/self/cgroup`

Nous exécutons :

```bash
cat /proc/self/cgroup
readlink /proc/$$/ns/cgroup

unshare --cgroup bash
cat /proc/self/cgroup
readlink /proc/$$/ns/cgroup
exit
```

Nous répondons :

1. Le namespace cgroup a-t-il changé ?
    
2. La vue de `/proc/self/cgroup` a-t-elle changé ?
    
3. Cela crée-t-il une limite mémoire ou CPU ?
    
4. Pourquoi faut-il distinguer cgroup namespace et cgroups ?
    

---

## Exercice 8 — Analyse d’un conteneur Docker

Si Docker est disponible :

```bash
docker run --rm -d --name proc-ns-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' proc-ns-demo)

sudo ls -l /proc/$pid/ns
sudo grep NSpid /proc/$pid/status
sudo cat /proc/$pid/cgroup
sudo cat /proc/$pid/mountinfo | head
sudo nsenter --target "$pid" --net ip addr

docker rm -f proc-ns-demo
```

Nous répondons :

1. Quels namespaces sont différents de l’hôte ?
    
2. Quel est le PID du processus dans le conteneur ?
    
3. Quelle vue cgroup voyons-nous depuis l’hôte ?
    
4. Quelles interfaces réseau voit le conteneur ?
    
5. Pourquoi `/proc` est-il indispensable au diagnostic ?
    

---

## 12.19. Ce que nous devons retenir

Nous retenons les points suivants :

1. `/proc` est l’interface principale pour observer les namespaces d’un processus.
    
2. Les namespaces sont visibles dans `/proc/<PID>/ns`.
    
3. Deux processus partagent un namespace si le lien symbolique correspondant est identique.
    
4. Le contenu de certains fichiers `/proc` dépend du namespace courant.
    
5. Le namespace PID influence la vue des processus dans `/proc`.
    
6. Le namespace network influence `/proc/net`.
    
7. Le namespace UTS influence `/proc/sys/kernel/hostname`.
    
8. Le namespace mount influence `/proc/self/mounts` et `/proc/self/mountinfo`.
    
9. Le namespace user est observable avec `uid_map` et `gid_map`.
    
10. Le namespace cgroup influence la vue de `/proc/self/cgroup`.
    
11. Le namespace time expose ses offsets dans `/proc/self/timens_offsets`.
    
12. `nsenter` s’appuie sur les namespaces exposés via `/proc/<PID>/ns`.
    
13. Dans un conteneur, `/proc` doit être monté correctement pour refléter la bonne vue.
    
14. `/proc` peut exposer des informations sensibles.
    
15. La sécurité de `/proc` dépend des permissions, des options de montage, des namespaces et des politiques de sécurité.
    
16. Un diagnostic sérieux croise toujours namespaces, `/proc`, cgroups, montages, capabilities et contexte d’exécution.
    

---

## Conclusion du chapitre 12

Nous avons relié les namespaces à `/proc`.

Nous savons maintenant que `/proc` n’est pas seulement une source d’informations système générales. C’est aussi une interface centrale pour observer, comparer et comprendre les namespaces d’un processus.

Nous avons vu que `/proc/<PID>/ns` permet d’identifier les namespaces, que `/proc/net` dépend du namespace réseau, que `/proc/self/mountinfo` dépend du namespace mount, que `uid_map` et `gid_map` expliquent les traductions du namespace user, et que `/proc/self/cgroup` peut être influencé par le namespace cgroup.

Nous retenons surtout que `/proc` est contextuel : ce que nous lisons dépend souvent du processus qui lit et du namespace dans lequel il se trouve.

Dans le chapitre suivant, nous étudions plus précisément la relation entre namespaces et cgroups, afin de clarifier la différence entre isolation des vues et limitation des ressources.

---
> [!info] Livre « Les namespaces Linux » — chapitre 12/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 11 — Namespace time|← 11 — Namespace time]] · [[Les namespaces Linux — 13 — Namespaces et cgroups|13 — Namespaces et cgroups →]]
