---
schema_version: 1
uid: 01M1BQ624982F93P45PA4PY0JW
titre: "Les namespaces Linux — 03 — Les outils de manipulation des namespaces"
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
resume: "Chapitre 3 sur 16 du livre « Les namespaces Linux » : Les outils de manipulation des namespaces. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 3/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 02 — Architecture générale des namespaces|← 02 — Architecture générale des namespaces]] · [[Les namespaces Linux — 04 — Namespace UTS|04 — Namespace UTS →]]

# Chapitre 3 — Les outils de manipulation des namespaces
## Objectifs du chapitre

Dans ce chapitre, nous étudions les outils pratiques qui permettent de créer, observer et rejoindre des namespaces Linux.

Dans le chapitre précédent, nous avons vu qu’un processus appartient simultanément à plusieurs namespaces, observables dans :

```bash
/proc/<PID>/ns
```

Nous allons maintenant manipuler ces namespaces concrètement avec des commandes système.

À la fin de ce chapitre, nous savons :

- utiliser `unshare` pour créer de nouveaux namespaces ;
    
- utiliser `nsenter` pour entrer dans les namespaces d’un processus existant ;
    
- utiliser `lsns` pour lister les namespaces actifs ;
    
- utiliser `ip netns` pour manipuler des namespaces réseau nommés ;
    
- comprendre le rôle de `/proc/<PID>/ns` ;
    
- éviter les pièges classiques, notamment avec le namespace PID et `/proc` ;
    
- faire le lien entre ces outils et les conteneurs.
    

---

## 3.1. Vue d’ensemble des outils

## 3.1.1. Pourquoi avons-nous besoin d’outils ?

Les namespaces sont des objets du noyau. Nous ne les manipulons pas directement comme des fichiers classiques.

Pour les utiliser, nous avons besoin d’outils qui appellent les bons mécanismes noyau :

- `clone` pour créer un nouveau processus dans un ou plusieurs nouveaux namespaces ;
    
- `unshare` pour détacher un processus de certains namespaces ;
    
- `setns` pour rejoindre un namespace existant.
    

En pratique, nous utilisons surtout des commandes déjà disponibles sur la plupart des distributions Linux.

---

## 3.1.2. Les outils principaux

Nous allons étudier les outils suivants :

|Outil|Rôle|
|---|---|
|`unshare`|créer un shell ou une commande dans de nouveaux namespaces|
|`nsenter`|entrer dans les namespaces d’un processus existant|
|`lsns`|lister les namespaces visibles|
|`ip netns`|créer et gérer des namespaces réseau nommés|
|`ip`|configurer les interfaces, routes et namespaces réseau|
|`mount`|manipuler les montages dans un namespace mount|
|`findmnt`|observer les points de montage|
|`ps`|observer les processus dans un namespace PID|
|`capsh`|observer les capabilities|
|`newuidmap` / `newgidmap`|configurer des mappings UID/GID pour les user namespaces|

Ces outils ne sont pas tous au même niveau : certains créent des namespaces, d’autres les observent ou les configurent.

---

## 3.2. `unshare`

## 3.2.1. Rôle de `unshare`

La commande `unshare` lance un programme après avoir créé un ou plusieurs nouveaux namespaces.

Sa forme générale est :

```bash
unshare [options] commande
```

Exemple :

```bash
unshare --uts bash
```

Cette commande lance un shell `bash` dans un nouveau namespace UTS.

Dans ce shell, nous pouvons changer le hostname sans changer celui de l’hôte.

---

## 3.2.2. Options principales

Les options les plus importantes sont :

|Option|Namespace créé|
|---|---|
|`--uts`|namespace UTS|
|`--pid`|namespace PID|
|`--mount`|namespace mount|
|`--net`|namespace network|
|`--ipc`|namespace IPC|
|`--user`|namespace user|
|`--cgroup`|namespace cgroup|
|`--time`|namespace time|
|`--fork`|lance la commande dans un processus enfant|
|`--mount-proc`|monte `/proc` dans le nouveau namespace|
|`--map-root-user`|mappe l’utilisateur courant comme root dans le user namespace|

Nous utilisons souvent plusieurs options ensemble.

---

## 3.2.3. Créer un namespace UTS

Exemple simple :

```bash
hostname
unshare --uts bash
hostname ns-demo
hostname
exit
hostname
```

Interprétation :

1. Nous affichons le hostname de l’hôte.
    
2. Nous lançons un shell dans un nouveau namespace UTS.
    
3. Nous changeons le hostname dans ce namespace.
    
4. Nous vérifions que le hostname a changé dans le shell isolé.
    
5. Nous quittons.
    
6. Nous vérifions que le hostname de l’hôte n’a pas changé.
    

Le namespace UTS est donc un excellent premier exemple, car il est simple et visible.

---

## 3.2.4. Créer un namespace PID

Le namespace PID demande une attention particulière.

Commande correcte :

```bash
unshare --fork --pid --mount-proc bash
```

Dans le shell obtenu :

```bash
echo $$
ps aux
cat /proc/1/comm
```

Nous observons généralement que le shell lancé est PID 1 dans le nouveau namespace.

Pourquoi utilisons-nous `--fork` ?

Parce que le nouveau namespace PID s’applique aux enfants du processus qui le crée. Le processus `unshare` lui-même ne devient pas directement PID 1 dans ce nouveau namespace.

Pourquoi utilisons-nous `--mount-proc` ?

Parce que `/proc` doit refléter le namespace PID courant. Sans remontage propre de `/proc`, les commandes comme `ps` peuvent afficher une vue incohérente.

Nous retenons :

```text
Pour manipuler un namespace PID proprement avec un shell :
unshare --fork --pid --mount-proc bash
```

---

## 3.2.5. Créer un namespace mount

Nous pouvons créer un namespace de montage :

```bash
unshare --mount bash
```

Dans ce shell, nous pouvons monter un système de fichiers temporaire :

```bash
mkdir -p /tmp/ns-mount-test
mount -t tmpfs tmpfs /tmp/ns-mount-test
findmnt /tmp/ns-mount-test
```

Puis nous quittons :

```bash
exit
```

Depuis l’hôte :

```bash
findmnt /tmp/ns-mount-test
```

Selon la propagation des montages, le montage peut ne pas être visible hors du namespace.

Cependant, sur certaines distributions, la propagation des montages peut créer des surprises. C’est pourquoi nous étudierons les notions `shared`, `private`, `slave` et `unbindable` dans le chapitre sur le namespace mount.

---

## 3.2.6. Créer un namespace réseau

Nous lançons :

```bash
unshare --net bash
```

Dans ce shell :

```bash
ip addr
```

Nous voyons généralement seulement l’interface loopback :

```text
lo
```

Elle peut être désactivée. Nous pouvons l’activer :

```bash
ip link set lo up
ip addr
```

Ce namespace réseau est isolé :

- il a ses propres interfaces ;
    
- il a ses propres routes ;
    
- il a ses propres sockets ;
    
- il a sa propre table réseau.
    

Mais il n’a pas automatiquement accès au réseau extérieur. Pour cela, nous devons configurer une paire `veth`, des routes et éventuellement du NAT.

---

## 3.2.7. Créer un namespace user

Nous pouvons créer un user namespace :

```bash
unshare --user --map-root-user bash
```

Dans ce shell :

```bash
id
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Nous pouvons voir :

```text
uid=0(root) gid=0(root)
```

Mais cela ne veut pas dire que nous sommes root sur l’hôte.

Nous sommes root dans le namespace utilisateur.

Le mapping peut ressembler à :

```text
         0       1000          1
```

Cela signifie :

```text
UID 0 dans le namespace → UID 1000 sur l’hôte
taille de plage : 1
```

Ce mécanisme est essentiel pour les conteneurs rootless.

---

## 3.2.8. Combiner plusieurs namespaces

Nous pouvons combiner plusieurs isolations :

```bash
unshare --fork --pid --mount --uts --ipc --mount-proc bash
```

Dans ce shell, nous pouvons vérifier :

```bash
ls -l /proc/$$/ns
hostname
ps aux
```

Nous avons un environnement déjà plus proche d’un conteneur, même s’il manque encore :

- un vrai root filesystem isolé ;
    
- une configuration réseau ;
    
- des cgroups ;
    
- une gestion fine des capabilities ;
    
- des restrictions seccomp ;
    
- une politique de sécurité.
    

---

## 3.3. `nsenter`

## 3.3.1. Rôle de `nsenter`

`nsenter` permet d’entrer dans les namespaces d’un processus existant.

La forme générale est :

```bash
nsenter --target <PID> [options] commande
```

Exemple :

```bash
sudo nsenter --target 1234 --net bash
```

Cette commande lance un shell dans le namespace réseau du processus `1234`.

---

## 3.3.2. Options principales

|Option|Effet|
|---|---|
|`--mount`|entrer dans le namespace mount|
|`--uts`|entrer dans le namespace UTS|
|`--ipc`|entrer dans le namespace IPC|
|`--net`|entrer dans le namespace réseau|
|`--pid`|entrer dans le namespace PID|
|`--user`|entrer dans le namespace user|
|`--cgroup`|entrer dans le namespace cgroup|
|`--time`|entrer dans le namespace time|
|`--target <PID>`|processus dont nous voulons rejoindre les namespaces|

Nous pouvons entrer dans plusieurs namespaces en même temps.

---

## 3.3.3. Entrer dans un namespace réseau

Supposons qu’un processus `1234` soit dans un namespace réseau isolé.

Nous pouvons faire :

```bash
sudo nsenter --target 1234 --net bash
```

Dans ce shell :

```bash
ip addr
ip route
ss -tulpen
```

Nous observons la pile réseau vue par le processus cible.

C’est très utile pour diagnostiquer un conteneur ou un service isolé.

---

## 3.3.4. Entrer dans plusieurs namespaces

Pour entrer dans un environnement plus complet :

```bash
sudo nsenter --target 1234 --mount --uts --ipc --net --pid bash
```

Dans ce shell, nous voyons une partie de l’environnement du processus cible.

Attention : entrer dans le namespace PID avec `nsenter` ne change pas toujours immédiatement la perception de tous les PID pour le shell déjà lancé. Le comportement exact dépend de l’utilisation de `--fork` et du processus enfant lancé. En pratique, pour une expérience plus cohérente, nous pouvons utiliser :

```bash
sudo nsenter --target 1234 --mount --uts --ipc --net --pid --fork bash
```

---

## 3.3.5. Utilisation avec les conteneurs Docker

Pour trouver le PID principal d’un conteneur Docker :

```bash
docker inspect --format '{{.State.Pid}}' <nom_ou_id_conteneur>
```

Puis :

```bash
sudo nsenter --target <PID> --mount --uts --ipc --net --pid --fork bash
```

Cela permet d’entrer dans l’environnement du conteneur, même si l’image ne contient pas forcément les outils de debug habituels.

Nous utilisons cette technique avec prudence, car nous intervenons depuis l’hôte dans l’environnement du conteneur.

---

## 3.4. `lsns`

## 3.4.1. Rôle de `lsns`

`lsns` liste les namespaces visibles sur le système.

Commande simple :

```bash
lsns
```

Sortie typique :

```text
        NS TYPE   NPROCS   PID USER COMMAND
4026531834 time      120     1 root /sbin/init
4026531835 cgroup    120     1 root /sbin/init
4026531836 pid       120     1 root /sbin/init
4026531837 user      120     1 root /sbin/init
4026531838 uts       120     1 root /sbin/init
4026531839 ipc       120     1 root /sbin/init
4026531840 net       120     1 root /sbin/init
4026531841 mnt       120     1 root /sbin/init
```

---

## 3.4.2. Colonnes principales

|Colonne|Signification|
|---|---|
|`NS`|identifiant du namespace|
|`TYPE`|type de namespace|
|`NPROCS`|nombre de processus associés|
|`PID`|un PID associé au namespace|
|`USER`|utilisateur du processus affiché|
|`COMMAND`|commande associée|

`lsns` permet de repérer rapidement les namespaces présents, notamment ceux créés par des conteneurs.

---

## 3.4.3. Filtrer par type

Nous pouvons filtrer par type :

```bash
lsns -t pid
lsns -t net
lsns -t mnt
lsns -t uts
lsns -t user
```

Exemple :

```bash
lsns -t net
```

permet de lister les namespaces réseau.

C’est très utile lorsque nous voulons comprendre combien d’environnements réseau isolés existent sur une machine.

---

## 3.4.4. Voir les namespaces d’un processus précis

Nous pouvons combiner `lsns` avec `/proc`.

Pour un PID donné :

```bash
ls -l /proc/<PID>/ns
```

Puis comparer avec :

```bash
lsns
```

L’objectif est de comprendre où se trouve le processus dans l’ensemble des namespaces du système.

---

## 3.5. `ip netns`

## 3.5.1. Rôle de `ip netns`

`ip netns` est spécialisé dans les namespaces réseau.

Il permet de créer des namespaces réseau nommés.

Créer un namespace :

```bash
sudo ip netns add ns1
```

Lister :

```bash
ip netns list
```

Exécuter une commande dans le namespace :

```bash
sudo ip netns exec ns1 ip addr
```

Supprimer :

```bash
sudo ip netns delete ns1
```

---

## 3.5.2. Où sont stockés les namespaces nommés ?

Les références sont généralement visibles dans :

```bash
/run/netns
```

Nous pouvons regarder :

```bash
ls -l /run/netns
```

Ces fichiers servent de références au namespace réseau.

Tant que cette référence existe, le namespace peut rester vivant.

---

## 3.5.3. Créer un namespace réseau et activer loopback

Nous créons :

```bash
sudo ip netns add ns1
```

Nous observons :

```bash
sudo ip netns exec ns1 ip addr
```

Nous activons loopback :

```bash
sudo ip netns exec ns1 ip link set lo up
```

Nous vérifions :

```bash
sudo ip netns exec ns1 ip addr
```

Puis nous supprimons :

```bash
sudo ip netns delete ns1
```

Cet exercice montre qu’un namespace réseau commence souvent avec une pile réseau minimale.

---

## 3.5.4. Différence entre `unshare --net` et `ip netns`

`unshare --net` crée un namespace réseau pour un processus lancé.

`ip netns add` crée un namespace réseau nommé, avec une référence persistante dans `/run/netns`.

Nous pouvons résumer :

```text
unshare --net  → pratique pour lancer un shell ou une commande isolée
ip netns       → pratique pour créer des namespaces réseau nommés et réutilisables
```

Pour des laboratoires réseau, `ip netns` est souvent plus confortable.

---

## 3.6. `ip` pour configurer le réseau des namespaces

## 3.6.1. Pourquoi `ip` est indispensable ?

Créer un namespace réseau ne suffit pas.

Il faut souvent configurer :

- interfaces ;
    
- adresses IP ;
    
- routes ;
    
- liens virtuels ;
    
- ponts ;
    
- NAT ;
    
- règles de firewall.
    

L’outil principal est :

```bash
ip
```

---

## 3.6.2. Créer une paire `veth`

Une paire `veth` est comme un câble virtuel entre deux interfaces.

Nous créons :

```bash
sudo ip link add veth-host type veth peer name veth-ns
```

Nous obtenons deux interfaces liées :

```text
veth-host <── câble virtuel ──> veth-ns
```

Ce qui entre d’un côté ressort de l’autre.

---

## 3.6.3. Placer une interface dans un namespace

Nous créons un namespace :

```bash
sudo ip netns add ns1
```

Nous plaçons une extrémité dedans :

```bash
sudo ip link set veth-ns netns ns1
```

Nous configurons l’hôte :

```bash
sudo ip addr add 10.0.0.1/24 dev veth-host
sudo ip link set veth-host up
```

Nous configurons le namespace :

```bash
sudo ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns
sudo ip netns exec ns1 ip link set veth-ns up
sudo ip netns exec ns1 ip link set lo up
```

Nous testons :

```bash
ping -c 3 10.0.0.2
```

Depuis le namespace :

```bash
sudo ip netns exec ns1 ping -c 3 10.0.0.1
```

Nous avons créé une communication réseau entre l’hôte et un namespace isolé.

---

## 3.7. `mount` et `findmnt`

## 3.7.1. Rôle de `mount`

Le namespace mount contrôle la vue des points de montage.

Nous utilisons `mount` pour créer ou modifier des montages.

Exemple :

```bash
unshare --mount bash
mkdir -p /tmp/mnt-test
mount -t tmpfs tmpfs /tmp/mnt-test
mount | grep mnt-test
```

---

## 3.7.2. Rôle de `findmnt`

`findmnt` permet d’observer les montages de manière plus lisible :

```bash
findmnt
findmnt /tmp/mnt-test
findmnt /proc
```

Nous l’utilisons souvent pour vérifier si un montage existe dans le namespace courant.

---

## 3.7.3. Propagation des montages

Un piège classique est la propagation des montages.

Selon la configuration, un montage créé dans un namespace peut être propagé ailleurs.

Pour éviter cela en TP, nous pouvons rendre les montages privés :

```bash
mount --make-rprivate /
```

Nous détaillerons ce point dans le chapitre sur le namespace mount.

Pour l’instant, nous retenons que le namespace mount ne se comprend pas correctement sans la notion de propagation.

---

## 3.8. `ps` et le namespace PID

## 3.8.1. Pourquoi `ps` peut être trompeur

`ps` lit les informations de `/proc`.

Si nous créons un namespace PID sans remonter `/proc`, `ps` peut afficher les processus de l’hôte au lieu de ceux du namespace.

Mauvais exemple pédagogique :

```bash
unshare --fork --pid bash
ps aux
```

Meilleur exemple :

```bash
unshare --fork --pid --mount-proc bash
ps aux
```

Nous voyons une liste cohérente avec le namespace PID.

---

## 3.8.2. Vérifier le PID 1

Dans le shell isolé :

```bash
echo $$
cat /proc/1/comm
ps aux
```

Nous devons comprendre que le PID 1 du namespace peut être notre shell.

Ce point est central pour comprendre les conteneurs.

Dans un conteneur, le processus principal est souvent PID 1 dans son namespace PID.

---

## 3.9. `capsh` et les capabilities

## 3.9.1. Pourquoi parler des capabilities dans un chapitre sur les outils ?

Les namespaces ne suffisent pas à comprendre les privilèges.

Un processus peut être dans un namespace isolé, mais ses permissions dépendent aussi des capabilities.

Nous pouvons afficher les capabilities avec :

```bash
capsh --print
```

Si `capsh` n’est pas installé, il peut être fourni par un paquet comme `libcap2-bin` sur Debian/Ubuntu.

---

## 3.9.2. Lire les capabilities dans `/proc`

Nous pouvons aussi lire :

```bash
grep Cap /proc/$$/status
```

Sortie possible :

```text
CapInh:  0000000000000000
CapPrm:  000001ffffffffff
CapEff:  000001ffffffffff
CapBnd:  000001ffffffffff
CapAmb:  0000000000000000
```

Ces valeurs sont en hexadécimal.

Elles décrivent les capabilities héritables, permises, effectives, bornées et ambiantes.

Nous approfondirons ce point dans le chapitre sur sécurité et capabilities.

---

## 3.10. `newuidmap` et `newgidmap`

## 3.10.1. Pourquoi ces outils existent-ils ?

Les user namespaces utilisent des mappings UID/GID.

Pour des mappings plus larges que le simple utilisateur courant, nous avons besoin de plages subordonnées définies dans :

```bash
/etc/subuid
/etc/subgid
```

Les outils :

```bash
newuidmap
newgidmap
```

permettent de configurer ces mappings de manière contrôlée.

Ils sont utilisés par des outils comme Podman rootless, Buildah ou certains runtimes rootless.

---

## 3.10.2. Observer les mappings

Dans un user namespace :

```bash
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Exemple simple :

```text
         0       1000          1
```

Exemple avec une plage plus large :

```text
         0     100000      65536
```

Cela signifie que les UID internes du namespace sont mappés vers une plage d’UID subordonnés sur l’hôte.

---

## 3.10.3. Pourquoi c’est important ?

Ce mécanisme permet d’avoir `root` dans le conteneur sans donner les privilèges root réels sur l’hôte.

C’est essentiel pour les conteneurs rootless.

Mais c’est aussi une source de complexité :

- permissions de fichiers ;
    
- volumes montés ;
    
- UID/GID inattendus ;
    
- problèmes avec les bind mounts ;
    
- interaction avec les systèmes de fichiers réseau.
    

Nous l’étudierons plus précisément dans le chapitre sur le user namespace.

---

## 3.11. Chaîne d’outils typique pour diagnostiquer un conteneur

## 3.11.1. Trouver le processus

Avec Docker :

```bash
docker ps
docker inspect --format '{{.State.Pid}}' <container>
```

Avec un processus quelconque :

```bash
ps aux | grep <nom>
```

---

## 3.11.2. Observer ses namespaces

```bash
sudo ls -l /proc/<PID>/ns
```

Comparer avec l’hôte :

```bash
ls -l /proc/1/ns
```

Nous voyons quels namespaces sont différents.

---

## 3.11.3. Entrer dans ses namespaces

```bash
sudo nsenter --target <PID> --mount --uts --ipc --net --pid --fork bash
```

Puis diagnostiquer :

```bash
hostname
ps aux
ip addr
mount
cat /proc/1/cgroup
```

Nous retrouvons une vue proche de celle du conteneur.

---

## 3.11.4. Vérifier les limites de ressources

Les namespaces ne suffisent pas.

Nous regardons aussi les cgroups :

```bash
cat /proc/1/cgroup
```

Sur cgroups v2 :

```bash
cat /sys/fs/cgroup/memory.current 2>/dev/null
cat /sys/fs/cgroup/memory.max 2>/dev/null
cat /sys/fs/cgroup/cpu.max 2>/dev/null
```

Nous faisons le lien avec le cours sur les cgroups.

---

## 3.12. Pièges classiques avec les outils

## 3.12.1. Oublier les droits

Certaines commandes nécessitent des droits administrateur :

```bash
unshare --net bash
ip link add ...
nsenter --target <PID> --net bash
mount -t tmpfs ...
```

Selon la distribution et la configuration, un utilisateur non privilégié peut créer certains namespaces, notamment user namespaces, mais pas tous.

---

## 3.12.2. Oublier `--fork` avec PID namespace

Commande à éviter pour débuter :

```bash
unshare --pid bash
```

Commande recommandée :

```bash
unshare --fork --pid --mount-proc bash
```

Le namespace PID a une sémantique particulière : les enfants entrent dans le nouveau namespace.

---

## 3.12.3. Oublier de monter `/proc`

Sans `/proc` cohérent, les outils comme `ps` peuvent mentir ou afficher la vue de l’hôte.

Nous utilisons :

```bash
--mount-proc
```

avec `unshare`.

---

## 3.12.4. Confondre namespace réseau nommé et processus vivant

Un namespace réseau créé par :

```bash
sudo ip netns add ns1
```

peut exister grâce à une référence dans `/run/netns`, même si aucun processus ne tourne dedans.

C’est différent d’un namespace créé uniquement pour un processus avec `unshare --net`.

---

## 3.12.5. Croire que `nsenter` crée une isolation

`nsenter` ne crée pas un nouveau namespace.

Il entre dans un namespace existant.

Pour créer, nous utilisons plutôt :

```bash
unshare
ip netns add
```

Pour rejoindre, nous utilisons :

```bash
nsenter
ip netns exec
```

---

## 3.13. Exercices

## Exercice 1 — Créer un namespace UTS avec `unshare`

Nous exécutons :

```bash
hostname
readlink /proc/$$/ns/uts
unshare --uts bash
hostname ns-outil
hostname
readlink /proc/$$/ns/uts
exit
hostname
readlink /proc/$$/ns/uts
```

Nous répondons :

1. L’identifiant du namespace UTS change-t-il ?
    
2. Le hostname change-t-il dans le shell isolé ?
    
3. Le hostname de l’hôte est-il modifié après `exit` ?
    
4. Quel outil avons-nous utilisé ?
    

---

## Exercice 2 — Créer un namespace PID correctement

Nous exécutons :

```bash
unshare --fork --pid --mount-proc bash
```

Dans le shell :

```bash
echo $$
cat /proc/1/comm
ps aux
ls -l /proc/$$/ns/pid
```

Nous répondons :

1. Quel est le PID du shell ?
    
2. Quel est le processus PID 1 ?
    
3. Pourquoi `--fork` est-il nécessaire ?
    
4. Pourquoi `--mount-proc` est-il important ?
    

Nous quittons :

```bash
exit
```

---

## Exercice 3 — Lister les namespaces avec `lsns`

Nous exécutons :

```bash
lsns
lsns -t pid
lsns -t net
lsns -t uts
lsns -t mnt
```

Nous répondons :

1. Quels types de namespaces voyons-nous ?
    
2. Combien de namespaces réseau existent ?
    
3. Certains namespaces semblent-ils liés à des conteneurs ?
    
4. Que signifie la colonne `NPROCS` ?
    

---

## Exercice 4 — Créer un namespace réseau nommé

Nous exécutons :

```bash
sudo ip netns add ns1
ip netns list
ls -l /run/netns
sudo ip netns exec ns1 ip addr
sudo ip netns exec ns1 ip link set lo up
sudo ip netns exec ns1 ip addr
sudo ip netns delete ns1
```

Nous répondons :

1. Où apparaît le namespace nommé ?
    
2. Quelle interface est visible au départ ?
    
3. Pourquoi activons-nous `lo` ?
    
4. Que fait `ip netns exec` ?
    

---

## Exercice 5 — Créer une paire `veth`

Nous exécutons :

```bash
sudo ip netns add ns1
sudo ip link add veth-host type veth peer name veth-ns
sudo ip link set veth-ns netns ns1

sudo ip addr add 10.0.0.1/24 dev veth-host
sudo ip link set veth-host up

sudo ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns
sudo ip netns exec ns1 ip link set veth-ns up
sudo ip netns exec ns1 ip link set lo up

ping -c 3 10.0.0.2
sudo ip netns exec ns1 ping -c 3 10.0.0.1
```

Nettoyage :

```bash
sudo ip netns delete ns1
sudo ip link delete veth-host 2>/dev/null
```

Nous répondons :

1. À quoi sert une paire `veth` ?
    
2. Quelle interface est côté hôte ?
    
3. Quelle interface est côté namespace ?
    
4. Pourquoi pouvons-nous faire un ping entre les deux ?
    

---

## Exercice 6 — Entrer dans les namespaces d’un processus

Dans un premier terminal :

```bash
unshare --uts bash
hostname nsenter-demo
echo $$
sleep 1000
```

Dans un second terminal, avec le PID du shell isolé :

```bash
sudo nsenter --target <PID> --uts bash
hostname
exit
```

Nous répondons :

1. Quel hostname voyons-nous après `nsenter` ?
    
2. Avons-nous créé un nouveau namespace ou rejoint un namespace existant ?
    
3. Quelle option de `nsenter` avons-nous utilisée ?
    

---

## 3.14. Ce que nous devons retenir

Nous retenons les points suivants :

1. `unshare` sert à créer de nouveaux namespaces pour une commande.
    
2. `nsenter` sert à entrer dans les namespaces d’un processus existant.
    
3. `lsns` sert à lister les namespaces visibles.
    
4. `ip netns` sert à manipuler des namespaces réseau nommés.
    
5. `ip` est indispensable pour configurer les interfaces et routes dans les namespaces réseau.
    
6. `mount` et `findmnt` sont essentiels pour comprendre les namespaces mount.
    
7. `ps` dépend de `/proc`, donc `/proc` doit être cohérent avec le namespace PID.
    
8. `capsh` et `/proc/<PID>/status` permettent d’observer les capabilities.
    
9. `newuidmap` et `newgidmap` servent aux mappings avancés des user namespaces.
    
10. Le namespace PID nécessite souvent `--fork`.
    
11. Pour un namespace PID, nous utilisons souvent `--mount-proc`.
    
12. `nsenter` ne crée pas une isolation, il rejoint une isolation existante.
    
13. Les namespaces réseau nommés persistent grâce à des références dans `/run/netns`.
    
14. Les outils de namespaces sont puissants, mais demandent une grande prudence.
    

---

## Conclusion du chapitre 3

Nous avons étudié les principaux outils qui permettent de manipuler les namespaces Linux.

Nous savons maintenant créer des namespaces avec `unshare`, les observer avec `lsns` et `/proc/<PID>/ns`, les rejoindre avec `nsenter`, et manipuler les namespaces réseau avec `ip netns`.

Nous avons aussi vu plusieurs pièges importants : le rôle de `--fork` avec le namespace PID, la nécessité de remonter `/proc`, la différence entre créer et rejoindre un namespace, et la différence entre namespace réseau temporaire et namespace réseau nommé.

Dans le chapitre suivant, nous commençons l’étude détaillée des namespaces un par un avec le namespace UTS, qui isole le hostname et le domainname.

---
> [!info] Livre « Les namespaces Linux » — chapitre 3/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 02 — Architecture générale des namespaces|← 02 — Architecture générale des namespaces]] · [[Les namespaces Linux — 04 — Namespace UTS|04 — Namespace UTS →]]
