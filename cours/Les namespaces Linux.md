---
schema_version: 1
uid: 01M02EX5BD9BVRTQDDF2WDT310
titre: Les namespaces Linux
aliases:
- namespaces
- espaces de noms Linux
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
resume: 'Cours approfondi et pratique sur les namespaces Linux : modèle du noyau, API clone/unshare/setns, UTS/PID/mount/network/IPC/user/cgroup/time, rootless, idmapped mounts, sécurité, conteneurs et diagnostic.'
niveau: avance
prerequis:
- '[[GNULinux]]'
- '[[proc]]'
auteurs:
- Michaël Launay
langue: fr
date_creation: 2026-05-24
date_modification: 2026-08-29
confidentialite: publique
publication:
- notes-publiques
rag: true
metadata_verifiees: true
---

# Les namespaces Linux

> [!abstract]
> Un namespace Linux ne crée pas une machine virtuelle : il modifie **la vue qu'un groupe de processus possède d'une ressource du noyau**. Les conteneurs modernes combinent plusieurs namespaces avec les cgroups, les capabilities, seccomp, les LSM, un système de fichiers et un runtime.

# Objectifs du cours

À la fin de ce cours, nous savons :

- expliquer précisément ce qu'isole chacun des huit grands namespaces Linux ;
- observer les namespaces d'un processus via `/proc/<PID>/ns` et `lsns` ;
- créer des namespaces avec `unshare` et comprendre `clone()` / `clone3()` ;
- rejoindre un namespace avec `nsenter` et comprendre `setns()` ;
- raisonner correctement sur les namespaces PID et leur processus PID 1 ;
- construire et diagnostiquer des namespaces réseau avec `ip netns` et des paires `veth` ;
- expliquer les mappings UID/GID d'un user namespace et le fonctionnement rootless ;
- distinguer namespace cgroup et contrôle réel des ressources avec cgroup v2 ;
- utiliser les time namespaces sans les confondre avec une virtualisation de l'heure civile ;
- comprendre les mount namespaces, la propagation des montages et les idmapped mounts ;
- expliquer pourquoi un namespace **n'est pas à lui seul une frontière de sécurité suffisante** ;
- relier les primitives du noyau à Docker, Podman, systemd et Kubernetes ;
- construire un environnement pseudo-conteneur minimal à des fins pédagogiques ;
- diagnostiquer une isolation défaillante de façon méthodique.

# Références de version

Ce cours est vérifié au **29 août 2026**.

Quelques repères :

- noyau Linux stable amont : **7.2.2** ;
- branche LTS récente : **6.18.x** ;
- util-linux stable : **2.42.2** ;
- les interfaces fondamentales des namespaces sont bien plus anciennes et restent compatibles ;
- la présence d'une fonctionnalité dépend aussi de la configuration du noyau et de la distribution.

Le but n'est donc pas de mémoriser un numéro de noyau, mais de comprendre les primitives et de savoir vérifier leur disponibilité.

Dans util-linux 2.42, plusieurs détails utiles à l'administration moderne ont encore évolué : `unshare --forward-signals` et `--owner`, ainsi que l'adressage `PID:INO` de `nsenter` pour viser plus précisément une instance de processus.

---

# Chapitre 1 — Modèle mental : isoler une vue du noyau

## 1.1. Le noyau reste partagé

Dans un conteneur Linux classique, l'hôte et le conteneur utilisent le **même noyau**.

Le conteneur n'embarque pas son propre noyau comme une machine virtuelle classique.

```text
Machine virtuelle
-----------------
application
OS invité
noyau invité
matériel virtuel
hyperviseur
noyau hôte
matériel

Conteneur Linux
---------------
application
bibliothèques / rootfs
namespaces + cgroups + sécurité
noyau hôte partagé
matériel
```

Cette différence explique à la fois :

- la légèreté des conteneurs ;
- leur démarrage rapide ;
- leur forte densité ;
- mais aussi le fait qu'une vulnérabilité noyau peut avoir un impact inter-conteneurs.

## 1.2. Ce qu'est réellement un namespace

Un namespace est un objet du noyau auquel des tâches sont rattachées.

Deux processus peuvent partager le même noyau et pourtant voir :

- des PID différents ;
- des interfaces réseau différentes ;
- des points de montage différents ;
- des UID/GID différents ;
- un hostname différent.

La bonne formule est :

```text
namespace = virtualisation d'une vue de ressource du noyau
```

et non :

```text
namespace = machine virtuelle miniature
```

## 1.3. Namespace ≠ cgroup

Les deux mécanismes répondent à des questions différentes.

| Mécanisme | Question principale |
|---|---|
| namespace | « Qu'est-ce que ce processus voit ? » |
| cgroup | « Quelles ressources ce groupe de processus peut-il consommer ? » |

Exemple : un PID namespace peut empêcher un processus de voir les PID de l'hôte, mais il ne l'empêche pas d'utiliser toute la mémoire disponible.

La limite mémoire est une fonction des cgroups, pas des PID namespaces.

Voir aussi [[Initialisation système et des services]] pour la gestion moderne des cgroups par systemd.

## 1.4. Namespace ≠ sandbox complète

Une vraie défense en profondeur peut combiner :

```text
namespaces
+ cgroups
+ capabilities minimales
+ seccomp
+ AppArmor / SELinux
+ Landlock selon le cas
+ système de fichiers read-only
+ no_new_privs
+ réseau filtré
+ utilisateur non privilégié
```

Voir [[Sécurité avancée sous Linux]].

---

# Chapitre 2 — Les huit grands types de namespaces

Linux expose aujourd'hui huit familles principales utilisées en espace utilisateur.

| Namespace | Flag | Isole principalement |
|---|---|---|
| mount | `CLONE_NEWNS` | points de montage |
| UTS | `CLONE_NEWUTS` | hostname et NIS domain name |
| IPC | `CLONE_NEWIPC` | IPC System V et files POSIX mqueue |
| PID | `CLONE_NEWPID` | identifiants de processus |
| network | `CLONE_NEWNET` | pile réseau |
| user | `CLONE_NEWUSER` | UID/GID, capabilities et identité de sécurité |
| cgroup | `CLONE_NEWCGROUP` | vue de la hiérarchie cgroup |
| time | `CLONE_NEWTIME` | offsets de certaines horloges monotoniques |

## 2.1. Ce qui n'est pas un namespace de processus

Le noyau contient aussi d'autres mécanismes appelés « namespaces » dans d'autres contextes, par exemple les **symbol namespaces** utilisés pour organiser des symboles exportés par des modules noyau.

Ils ne doivent pas être confondus avec les namespaces de processus étudiés ici.

## 2.2. Tous les namespaces ne sont pas équivalents

Certaines propriétés sont hiérarchiques :

- PID namespaces ;
- user namespaces.

D'autres isolent surtout une vue :

- UTS ;
- network ;
- mount ;
- IPC.

Le cgroup namespace est particulièrement trompeur : il ne limite rien directement ; il **virtualise la vue de la hiérarchie cgroup**.

---

# Chapitre 3 — Observer les namespaces d'un processus

## 3.1. `/proc/<PID>/ns`

Chaque processus expose des liens magiques dans :

```bash
ls -l /proc/$$/ns
```

Exemple :

```text
cgroup -> cgroup:[4026531835]
ipc    -> ipc:[4026531839]
mnt    -> mnt:[4026531841]
net    -> net:[4026531840]
pid    -> pid:[4026531836]
pid_for_children -> pid:[4026531836]
time   -> time:[4026531834]
time_for_children -> time:[4026531834]
user   -> user:[4026531837]
uts    -> uts:[4026531838]
```

Deux processus dont un lien donne le même couple `type:[inode]` partagent la même instance de namespace pour ce type.

## 3.2. Comparer deux processus

```bash
pid_a=$$
pid_b=1

for ns in cgroup ipc mnt net pid time user uts; do
    printf '%-7s %-24s %-24s\n' \
        "$ns" \
        "$(readlink /proc/$pid_a/ns/$ns)" \
        "$(readlink /proc/$pid_b/ns/$ns)"
done
```

Ce type de comparaison est plus fiable que de deviner l'isolation à partir du nom d'un processus.

## 3.3. `lsns`

`lsns`, fourni par util-linux, est l'outil d'inventaire principal :

```bash
lsns
```

Limiter à un type :

```bash
lsns --type net
lsns --type user
lsns --type pid
```

Afficher les namespaces liés à un processus :

```bash
lsns --task $$
```

> [!tip]
> Ne figeons pas dans des scripts la liste ou l'ordre des colonnes par défaut de `lsns`. Demandons explicitement les colonnes nécessaires avec `--output`.

## 3.4. Un namespace peut survivre sans processus membre

Un namespace vit tant qu'une référence noyau le maintient.

Un descripteur de fichier ouvert sur `/proc/<PID>/ns/<type>` peut conserver cette référence.

Il est également possible de rendre certains namespaces persistants au moyen d'un bind mount :

```bash
sudo touch /run/my-uts-ns
sudo unshare --uts=/run/my-uts-ns /bin/true
```

Puis :

```bash
sudo nsenter --uts=/run/my-uts-ns hostname
```

Pour supprimer la référence persistante :

```bash
sudo umount /run/my-uts-ns
sudo rm /run/my-uts-ns
```

Les PID namespaces ont des contraintes supplémentaires autour de leur processus init.

---

# Chapitre 4 — API noyau : `clone`, `clone3`, `unshare` et `setns`

## 4.1. Les quatre opérations mentales

| Primitive | Idée |
|---|---|
| `clone()` | créer une tâche avec contrôle fin du partage |
| `clone3()` | API plus moderne/extensible pour créer une tâche |
| `unshare()` | arrêter de partager certains contextes |
| `setns()` | rejoindre un namespace existant |

Les outils `unshare(1)` et `nsenter(1)` sont des interfaces en espace utilisateur autour de ces concepts.

## 4.2. `clone()` / `clone3()`

Un runtime de conteneur peut demander la création de plusieurs namespaces lors de la création d'un processus.

Conceptuellement :

```text
clone3(... CLONE_NEWPID | CLONE_NEWNS | CLONE_NEWUTS ...)
```

Dans un cours d'administration, nous n'avons généralement pas besoin d'appeler directement ces syscalls.

## 4.3. `unshare`

Créer un nouveau UTS namespace :

```bash
sudo unshare --uts bash
```

Créer plusieurs namespaces :

```bash
sudo unshare \
    --fork \
    --pid \
    --mount \
    --uts \
    --ipc \
    --mount-proc \
    bash
```

Avec util-linux moderne, `unshare` sait aussi gérer :

- des mappings UID/GID complexes ;
- des namespaces persistants ;
- `--kill-child` ;
- `--forward-signals` ;
- des offsets de time namespace ;
- la propagation mount.

## 4.4. `setns()`

`setns()` peut recevoir :

- un FD ouvert sur `/proc/<PID>/ns/<type>` ;
- ou, pour plusieurs usages, un **PID file descriptor** obtenu avec `pidfd_open()`.

Le cas pidfd permet notamment à une application de raisonner sur un processus sans s'appuyer uniquement sur un numéro de PID réutilisable.

## 4.5. `nsenter`

Exemple :

```bash
sudo nsenter \
    --target 1234 \
    --mount \
    --uts \
    --ipc \
    --net \
    --pid \
    --fork \
    bash
```

`--fork` est important lors de l'entrée dans un PID namespace : le nouveau programme enfant peut alors être créé dans la vue PID cible.

`nsenter` moderne permet également :

```bash
sudo nsenter --target 1234 --join-cgroup --all bash
```

selon la version et les permissions disponibles.

> [!warning]
> Entrer dans le **cgroup namespace** et rejoindre le **cgroup** d'un processus sont deux opérations différentes.

---

# Chapitre 5 — UTS namespace

## 5.1. Ce qu'il isole

UTS signifie historiquement UNIX Time-sharing System.

Le namespace isole principalement :

- `hostname` ;
- NIS domain name.

Créer un shell isolé :

```bash
sudo unshare --uts bash
```

Puis :

```bash
hostname lab-ns
hostname
```

Dans un autre terminal sur l'hôte :

```bash
hostname
```

Le hostname de l'hôte n'a pas changé.

## 5.2. Ce qu'il n'isole pas

Changer le hostname ne crée pas :

- un nouveau DNS ;
- une nouvelle pile réseau ;
- une nouvelle identité cryptographique ;
- un nouveau `/etc/hosts`.

Le UTS namespace ne doit donc pas être confondu avec une identité réseau complète.

## 5.3. Usage conteneur

Les runtimes l'utilisent pour donner un nom d'hôte propre à un conteneur ou un pod.

C'est l'un des namespaces les plus simples pour commencer les expérimentations.

---

# Chapitre 6 — PID namespaces

## 6.1. Vue hiérarchique

Un PID namespace est hiérarchique.

Un processus peut avoir plusieurs PID selon le namespace depuis lequel nous l'observons.

```text
hôte : PID 42100
        |
        +-- namespace enfant : PID 1
```

Le parent peut voir les processus de ses namespaces descendants.

L'inverse n'est pas vrai : un processus dans un PID namespace enfant ne voit pas les processus du namespace ancêtre.

## 6.2. Créer correctement un PID namespace

```bash
sudo unshare --fork --pid --mount-proc bash
```

Puis :

```bash
echo $$
ps -ef
```

Nous devons voir un espace PID cohérent avec un processus PID 1.

## 6.3. Pourquoi `--fork` ?

Le nouveau PID namespace créé par `unshare()` définit le namespace dans lequel seront créés les **enfants suivants**.

Créer un enfant permet donc d'obtenir le PID 1 attendu dans le nouveau namespace.

## 6.4. Pourquoi remonter `/proc` ?

`ps` lit largement `/proc`.

Si nous changeons de PID namespace tout en gardant un ancien montage de `/proc`, nous pouvons obtenir une vue incohérente.

D'où :

```bash
unshare --fork --pid --mount-proc ...
```

## 6.5. Le PID 1 est spécial

Dans un PID namespace, son processus init possède des responsabilités importantes :

- adopter certains processus orphelins ;
- récolter les zombies ;
- recevoir et traiter correctement des signaux.

Un simple script shell n'est pas toujours un excellent PID 1.

C'est pourquoi les runtimes peuvent utiliser un petit init (`tini`, `catatonit`, etc.) lorsque c'est pertinent.

## 6.6. Cycle de vie

Si le processus init d'un PID namespace meurt, le noyau termine les autres processus du namespace.

Pour des expériences contrôlées avec `unshare`, l'option suivante est utile :

```bash
sudo unshare \
    --pid \
    --fork \
    --mount-proc \
    --kill-child \
    bash
```

## 6.7. PID visibles dans `/proc`

Depuis l'hôte :

```bash
ps -o pid,ppid,user,cmd -p <PID_HOTE>
```

Dans le namespace :

```bash
ps -o pid,ppid,user,cmd
```

Pour diagnostiquer un conteneur, comparons toujours :

- PID hôte ;
- PID vu dans le conteneur ;
- lien `/proc/<PID>/ns/pid` ;
- cgroup du processus.

---

# Chapitre 7 — Mount namespaces

## 7.1. Ce qu'ils isolent

Un mount namespace isole la **table des montages**, pas automatiquement le contenu de tout le stockage.

```bash
sudo unshare --mount bash
```

Dans le namespace :

```bash
mount -t tmpfs tmpfs /mnt
findmnt /mnt
```

Sur l'hôte, selon la propagation configurée, ce montage ne doit pas apparaître.

## 7.2. Propagation des montages

Un mount namespace ne signifie pas automatiquement « aucune propagation ».

Il faut connaître quatre modes :

| Mode | Idée |
|---|---|
| shared | propage certains événements dans le groupe de partage |
| private | aucune propagation vers/depuis les pairs |
| slave | reçoit du maître, ne renvoie pas vers lui |
| unbindable | private + ne peut pas être bind-mounté |

Inspection :

```bash
findmnt -o TARGET,PROPAGATION
```

`unshare(1)` moderne rend par défaut les montages privés dans un nouveau mount namespace, sauf demande `--propagation unchanged`.

## 7.3. Bind mounts

Un bind mount réexpose une arborescence existante :

```bash
mount --bind /source /destination
```

Le contenu reste le même objet sous-jacent.

Le bind mount modifie la vue du VFS, pas les données du disque.

## 7.4. `pivot_root` vs `chroot`

`chroot()` change le répertoire racine visible d'un processus mais n'est pas une primitive complète d'isolation.

Un runtime de conteneur moderne construit généralement un rootfs avec :

- un mount namespace ;
- des bind mounts ;
- éventuellement OverlayFS ;
- `pivot_root()` ou une stratégie équivalente ;
- restrictions complémentaires.

> [!warning]
> `chroot` seul n'est pas une sandbox de sécurité.

## 7.5. Nouvelle API mount

Les noyaux modernes disposent d'une API plus structurée autour notamment de :

- `open_tree()` ;
- `move_mount()` ;
- `fsopen()` / `fsconfig()` / `fsmount()` ;
- `mount_setattr()`.

Ces primitives réduisent certains problèmes liés à l'ancienne API `mount(2)` orientée chaînes de caractères et facilitent des opérations atomiques ou FD-based.

## 7.6. Idmapped mounts

Un **idmapped mount** permet d'exposer les mêmes inodes avec un mapping d'UID/GID différent **pour ce montage**, sans `chown -R` massif.

Conceptuellement :

```text
fichier sur disque : uid 100000
        |
idmapped mount
        |
vue du conteneur : uid 0
```

Le noyau associe un user namespace au montage via `mount_setattr()` et `MOUNT_ATTR_IDMAP`.

Usages :

- rootless containers ;
- partage de volumes avec ownership adapté ;
- images/rootfs réutilisables ;
- éviter de modifier physiquement des millions d'inodes.

---

# Chapitre 8 — Network namespaces

## 8.1. Ce qu'ils isolent

Un network namespace possède notamment sa propre vue de :

- interfaces réseau ;
- adresses IP ;
- routes ;
- sockets ;
- ports ;
- règles netfilter ;
- paramètres réseau sous `/proc/sys/net` ;
- loopback.

## 8.2. `unshare --net`

```bash
sudo unshare --net bash
```

Puis :

```bash
ip link
```

L'interface loopback peut exister mais être DOWN.

```bash
ip link set lo up
```

## 8.3. `ip netns`

Pour des labs réseau persistants, `ip netns` est souvent plus pratique.

```bash
sudo ip netns add ns-a
sudo ip netns exec ns-a ip link set lo up
sudo ip netns exec ns-a ip addr
```

Supprimer :

```bash
sudo ip netns del ns-a
```

`ip netns` maintient des références nommées sous `/run/netns`.

## 8.4. Connecter deux namespaces avec `veth`

Créer les namespaces :

```bash
sudo ip netns add ns-a
sudo ip netns add ns-b
```

Créer la paire :

```bash
sudo ip link add veth-a type veth peer name veth-b
```

Déplacer chaque extrémité :

```bash
sudo ip link set veth-a netns ns-a
sudo ip link set veth-b netns ns-b
```

Configurer :

```bash
sudo ip -n ns-a addr add 10.42.0.1/24 dev veth-a
sudo ip -n ns-b addr add 10.42.0.2/24 dev veth-b
sudo ip -n ns-a link set lo up
sudo ip -n ns-b link set lo up
sudo ip -n ns-a link set veth-a up
sudo ip -n ns-b link set veth-b up
```

Tester :

```bash
sudo ip netns exec ns-a ping -c 2 10.42.0.2
```

Nettoyage :

```bash
sudo ip netns del ns-a
sudo ip netns del ns-b
```

## 8.5. Bridge et conteneurs

Pour plusieurs conteneurs, un runtime crée souvent :

```text
conteneur A -- veth --+
                      |
                    bridge -- réseau hôte
                      |
conteneur B -- veth --+
```

Voir [[Les protocoles de communications]] et [[Docker]].

## 8.6. Namespace réseau ≠ politique réseau

Le namespace sépare les piles réseau, mais la politique de sécurité exige encore :

- filtrage ;
- routes maîtrisées ;
- règles firewall ;
- contrôle des capabilities ;
- éventuellement des politiques Kubernetes/CNI.

---

# Chapitre 9 — IPC namespace

## 9.1. Ressources isolées

Les IPC namespaces isolent notamment :

- IPC System V : shared memory, sémaphores, message queues ;
- POSIX message queues.

```bash
sudo unshare --ipc bash
```

Inspection :

```bash
ipcs
```

## 9.2. Ce qui n'est pas automatiquement isolé

Une socket Unix basée sur un **pathname de filesystem** dépend de la vue filesystem/mount et des permissions, pas uniquement de l'IPC namespace.

C'est un piège classique lorsque l'on réduit « IPC » à « toutes les communications locales ».

## 9.3. Cycle de vie

Quand un IPC namespace disparaît, les objets IPC qu'il contient sont détruits.

Cela rend ce namespace utile pour éviter les collisions entre applications historiques utilisant des identifiants IPC globaux.

---

# Chapitre 10 — User namespaces

## 10.1. Pourquoi ils sont particuliers

Les user namespaces virtualisent notamment :

- UID ;
- GID ;
- certaines notions de credentials ;
- l'étendue des capabilities.

Un utilisateur non root sur l'hôte peut devenir UID 0 **dans un user namespace** sans devenir root sur l'hôte.

```text
hôte           user namespace
UID 1000  ---> UID 0
```

## 10.2. Expérience minimale

```bash
unshare --user --map-root-user bash
```

Puis :

```bash
id
cat /proc/self/uid_map
cat /proc/self/gid_map
```

Typiquement :

```text
inside     outside     length
0          1000        1
```

## 10.3. Root dans un namespace n'est pas root sur l'hôte

C'est l'idée centrale.

Les capabilities sont évaluées relativement au user namespace qui possède la ressource ou le namespace concerné.

Un processus peut donc avoir `CAP_SYS_ADMIN` dans son user namespace tout en étant incapable :

- de charger un module noyau sur l'hôte ;
- de changer l'heure civile globale ;
- de monter un filesystem bloc arbitraire ;
- de modifier des ressources appartenant au user namespace initial.

## 10.4. `/etc/subuid` et `/etc/subgid`

Pour mapper davantage qu'un seul UID, Linux s'appuie souvent sur des plages subordonnées :

```text
/etc/subuid
/etc/subgid
```

Exemple :

```text
alice:100000:65536
```

Util-linux moderne peut automatiser ces mappings :

```bash
unshare --user --map-auto --map-root-user bash
```

Selon la configuration système, cela utilise `newuidmap` et `newgidmap`.

## 10.5. `setgroups`

Le mapping GID possède des règles de sécurité particulières.

Pour un utilisateur non privilégié, le noyau impose généralement de désactiver `setgroups()` avant d'autoriser certains mappings GID.

Les outils modernes gèrent cela pour nous dans les cas courants.

## 10.6. Rootless containers

Les user namespaces sont une fondation majeure des conteneurs rootless.

Un runtime rootless combine :

- user namespace ;
- mount namespace ;
- PID namespace ;
- network namespace ou solution réseau userspace ;
- cgroups selon les capacités du système ;
- idmapped mounts ou mécanismes de décalage d'UID ;
- capabilities confinées.

## 10.7. Sécurité des user namespaces

Les user namespaces augmentent fortement les possibilités offertes à un utilisateur non privilégié et donc la surface noyau accessible.

Le noyau Linux recommande d'associer leur usage à un **contrôle de ressources**, notamment avec les memory cgroups, lorsque les utilisateurs ou leurs workloads ne sont pas considérés comme fiables.

Une distribution peut aussi ajouter sa propre politique de restriction.

> [!warning]
> Ne concluons jamais « rootless = aucun risque ». Cela réduit les privilèges de l'orchestrateur et des processus vis-à-vis de l'hôte, mais le noyau reste partagé.

---

# Chapitre 11 — Cgroup namespace

## 11.1. Ce qu'il fait réellement

Le cgroup namespace virtualise la **vue de la hiérarchie cgroup**.

Il peut notamment modifier ce que le processus voit dans :

```bash
cat /proc/self/cgroup
```

Il ne crée pas de limite CPU ou mémoire.

## 11.2. Cgroup namespace vs cgroup v2

```text
cgroup namespace
    ↓
masque / relativise la vue

cgroup v2
    ↓
contrôle CPU, mémoire, PIDs, IO, etc.
```

Créer un cgroup namespace :

```bash
sudo unshare --cgroup bash
```

Lister :

```bash
lsns --type cgroup
```

## 11.3. `setns()` ne déplace pas le processus dans un cgroup

Rejoindre un cgroup namespace change la vue, mais **ne change pas automatiquement l'appartenance aux cgroups**.

C'est pourquoi `nsenter` propose séparément `--join-cgroup` pour certains scénarios.

---

# Chapitre 12 — Time namespaces

## 12.1. Horloges virtualisées

Les time namespaces virtualisent des offsets de :

- `CLOCK_MONOTONIC` ;
- `CLOCK_BOOTTIME` ;
- variantes associées.

Ils **ne virtualisent pas `CLOCK_REALTIME`**.

Donc un time namespace n'est pas un mécanisme pour donner arbitrairement une date civile différente à un conteneur.

## 12.2. Motivation

Ils sont particulièrement utiles pour :

- checkpoint/restore ;
- migration de conteneurs ;
- conserver une cohérence de temps monotone après restauration.

## 12.3. Observation

```bash
cat /proc/self/timens_offsets
```

Avec util-linux moderne, `unshare` peut créer un time namespace et spécifier des offsets.

Exemple pédagogique :

```bash
sudo unshare --time --fork --monotonic 3600 bash
```

La syntaxe exacte disponible doit être vérifiée avec :

```bash
unshare --help
```

car elle dépend de la version util-linux de la distribution.

## 12.4. `time` vs `time_for_children`

Comme pour PID, le namespace actif d'un processus et celui assigné à ses futurs enfants peuvent être distingués dans `/proc/<PID>/ns`.

C'est pourquoi nous pouvons voir :

```text
time
time_for_children
```

---

# Chapitre 13 — `/proc`, namespace FDs, pidfds et inspection avancée

## 13.1. Les liens de namespace sont des objets spéciaux

Les entrées `/proc/<PID>/ns/*` ne sont pas de simples symlinks ordinaires.

Les ouvrir donne un descripteur maintenant une référence vers le namespace.

Cela permet :

- `setns()` ;
- la persistance par bind mount ;
- des introspections via ioctls spécialisés.

## 13.2. `pid_for_children` et `time_for_children`

Ces entrées représentent le namespace qui sera appliqué aux enfants futurs.

Elles sont essentielles pour comprendre pourquoi un appel `unshare(CLONE_NEWPID)` ne transforme pas magiquement le processus courant en PID 1 du nouveau namespace.

## 13.3. pidfds

Un numéro de PID peut être réutilisé après la mort d'un processus. Util-linux 2.42 ajoute aussi à `nsenter` une convention `PID:INO`, où l'inode permet de préciser l'instance de processus visée.

Un **pidfd** fournit une référence FD à un processus précis.

Des APIs modernes peuvent l'utiliser pour :

- signaler un processus de manière moins sujette aux races ;
- attendre sa terminaison ;
- rejoindre plusieurs de ses namespaces via `setns()` selon les flags et permissions.

Ce modèle FD-based est une tendance importante des API Linux modernes.

## 13.4. `/proc` dépend du contexte

Beaucoup de fichiers `/proc` exposent une vue dépendant des namespaces :

- `/proc/<PID>` et le PID namespace ;
- `/proc/net` et le network namespace ;
- `/proc/self/cgroup` et le cgroup namespace ;
- UID/GID maps et user namespace.

Pour diagnostiquer, il faut toujours se demander :

```text
Depuis quel namespace suis-je en train de lire /proc ?
```

---

# Chapitre 14 — Namespaces, capabilities, seccomp et LSM

## 14.1. Capabilities

Linux découpe une partie des privilèges root en capabilities :

```bash
capsh --print
```

ou :

```bash
grep '^Cap' /proc/$$/status
```

Un user namespace modifie le contexte dans lequel ces capabilities sont valides.

## 14.2. Le piège `CAP_SYS_ADMIN`

`CAP_SYS_ADMIN` reste une capability extrêmement large.

Un processus qui possède cette capability dans le user namespace initial est très puissant.

Dans un user namespace enfant, elle est plus confinée, mais peut permettre de nombreuses opérations sur les namespaces possédés par ce user namespace.

## 14.3. `no_new_privs`

`no_new_privs` garantit qu'un `execve()` ne pourra pas accroître les privilèges du processus.

C'est une brique importante des sandboxes modernes.

## 14.4. seccomp

Seccomp filtre les appels système autorisés.

```text
namespace : limite la vue
seccomp   : limite les syscalls
```

Un conteneur peut donc voir peu de choses mais encore disposer d'une grande surface d'appels système si aucun filtrage n'est appliqué.

## 14.5. AppArmor et SELinux

Les LSM peuvent imposer une politique indépendante des namespaces.

Ils constituent une autre couche de défense.

## 14.6. Landlock

Landlock est un LSM empilable permettant à un processus, y compris non privilégié, de **se restreindre lui-même** sur certaines ressources filesystem et réseau selon la version de l'ABI.

Cela illustre bien l'approche moderne :

```text
isolation par namespaces
+ réduction de privilèges
+ confinement LSM
```

---

# Chapitre 15 — Des namespaces aux conteneurs

## 15.1. Un conteneur est un assemblage

Un runtime OCI construit typiquement :

```text
image/rootfs
+
mount namespace
+
PID namespace
+
network namespace
+
UTS namespace
+
IPC namespace
+
user namespace éventuellement
+
cgroup namespace
+
cgroups
+
capabilities réduites
+
seccomp
+
LSM
```

Voir [[Docker]].

## 15.2. Conteneurs rootful et rootless

### Rootful

Le daemon/runtime possède des privilèges élevés sur l'hôte.

Avantages : compatibilité maximale.

Risques : une compromission du runtime ou une mauvaise configuration a potentiellement plus d'impact.

### Rootless

L'orchestration fonctionne sous un utilisateur non privilégié et exploite notamment les user namespaces.

Avantages : réduction du blast radius de nombreux scénarios.

Limites : certaines opérations réseau, stockage ou périphériques peuvent demander des adaptations.

## 15.3. Docker et Podman

Observer un conteneur :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' <conteneur>)
sudo lsns --task "$pid"
```

Entrer dans ses namespaces :

```bash
sudo nsenter --target "$pid" --all --fork bash
```

> [!warning]
> `docker exec` et `nsenter` n'ont pas exactement la même sémantique. `docker exec` passe par le runtime, alors que `nsenter` rejoint directement des namespaces du processus cible.

## 15.4. systemd

systemd exploite plusieurs primitives d'isolation :

```ini
[Service]
PrivateTmp=yes
PrivateDevices=yes
ProtectSystem=strict
ProtectHome=yes
PrivateNetwork=yes
NoNewPrivileges=yes
```

Certaines options construisent ou utilisent des namespaces sous le capot.

Le bon outil pour analyser une unité reste :

```bash
systemd-analyze security mon-service.service
```

Voir [[Initialisation système et des services]].

## 15.5. Kubernetes

Dans Kubernetes, plusieurs conteneurs d'un Pod partagent typiquement certains namespaces, en particulier le réseau.

Conceptuellement :

```text
Pod
├── conteneur A
├── conteneur B
└── namespace réseau partagé
```

Cela explique pourquoi les conteneurs d'un même Pod peuvent communiquer via `localhost`.

Le partage de PID dépend de la configuration (`shareProcessNamespace`).

Les détails exacts sont implémentés par le runtime OCI/CRI et les plugins réseau.

---

# Chapitre 16 — Construire un mini-conteneur pédagogique

> [!warning]
> Ce chapitre sert à comprendre les primitives. Ce n'est **pas** un remplacement d'un runtime OCI audité.

## 16.1. Objectif

Nous voulons obtenir :

- nouveau user namespace ;
- nouveau UTS ;
- nouveau PID namespace ;
- nouveau mount namespace ;
- `/proc` cohérent ;
- rootfs minimal ;
- réseau éventuellement séparé.

## 16.2. Préparer un rootfs

Nous pouvons utiliser un rootfs minimal déjà construit par une distribution ou un outil dédié.

Évitons de copier arbitrairement `/bin`, `/lib` et `/usr` à la main : les dépendances dynamiques, NSS, certificats et fichiers de configuration rendent cette approche fragile.

## 16.3. Créer l'environnement de namespaces

Exemple conceptuel :

```bash
sudo unshare \
    --fork \
    --pid \
    --mount \
    --uts \
    --ipc \
    --mount-proc \
    --kill-child \
    bash
```

À l'intérieur :

```bash
hostname mini-container
mount --make-rprivate /
```

Puis nous mettons en place le rootfs de laboratoire avec les techniques de mount vues précédemment.

## 16.4. Ce qui manque encore

Même si l'environnement ressemble à un conteneur, il manque potentiellement :

- cgroups ;
- rootless mapping complet ;
- seccomp ;
- profile AppArmor/SELinux ;
- gestion robuste des devices ;
- rootfs OCI ;
- lifecycle et signal handling ;
- hooks ;
- logs ;
- réseau CNI ;
- vérifications de sécurité.

C'est précisément le travail d'un runtime.

---

# Chapitre 17 — Idmapped mounts et stockage rootless moderne

## 17.1. Le problème historique

Un conteneur rootless peut mapper :

```text
UID 0 conteneur -> UID 100000 hôte
```

Si nous voulons monter un répertoire de l'hôte, les ownerships peuvent devenir difficiles à gérer.

Une ancienne solution consiste à faire :

```bash
chown -R 100000:100000 gros-volume/
```

Sur des millions de fichiers, c'est coûteux et modifie réellement les métadonnées.

## 17.2. Idmapped mount

Un idmapped mount applique une traduction de propriété **au niveau du montage**.

Le disque reste inchangé.

```text
inode uid=1000
       |
       +-- montage normal  -> uid 1000
       |
       +-- montage idmapped -> uid 0 dans la vue cible
```

## 17.3. Pourquoi c'est important

Cas d'usage :

- volumes rootless ;
- images partagées ;
- répertoires homes ;
- VM/conteneurs avec mappings différents ;
- réduction des `chown` massifs.

## 17.4. Namespace et idmapping

L'idmapping utilisé par un mount est dérivé d'un user namespace.

Cela montre une relation importante : le user namespace n'est pas seulement utilisé pour « devenir root dans un conteneur » ; il sert aussi comme **objet de mapping d'identités** pour d'autres primitives du noyau.

---

# Chapitre 18 — Diagnostic méthodique

## 18.1. Symptôme : « le conteneur voit trop de processus »

Vérifier :

```bash
readlink /proc/$$/ns/pid
mount | grep ' on /proc '
ps -ef
```

Questions :

1. Sommes-nous réellement dans un nouveau PID namespace ?
2. `/proc` a-t-il été remonté pour ce namespace ?
3. Le processus attendu est-il PID 1 ?

## 18.2. Symptôme : « le montage apparaît sur l'hôte »

```bash
findmnt -o TARGET,PROPAGATION
cat /proc/self/mountinfo
```

Vérifier shared/private/slave.

## 18.3. Symptôme : « le réseau ne fonctionne pas »

```bash
ip addr
ip route
ip link
ss -lntup
cat /etc/resolv.conf
```

Puis depuis l'hôte :

```bash
ip link
bridge link
```

Distinguer :

- loopback DOWN ;
- interface DOWN ;
- adresse manquante ;
- route manquante ;
- forwarding absent ;
- firewall ;
- DNS.

## 18.4. Symptôme : « je suis root mais l'opération est refusée »

```bash
id
cat /proc/self/uid_map
grep '^Cap' /proc/self/status
readlink /proc/self/ns/user
```

La question est :

```text
root / capability dans quel user namespace ?
```

## 18.5. Symptôme : « la limite mémoire n'est pas respectée »

Ne pas chercher dans le namespace.

Vérifier le cgroup :

```bash
cat /proc/self/cgroup
cat /sys/fs/cgroup/memory.max 2>/dev/null || true
```

## 18.6. Symptôme : « nsenter ne reproduit pas exactement le conteneur »

`nsenter` rejoint des namespaces, mais ne recrée pas nécessairement :

- environnement ;
- cwd ;
- rootfs/chroot ;
- cgroup ;
- LSM context ;
- credentials ;
- variables du runtime.

`nsenter` moderne possède des options comme `--root`, `--wd`, `--wdns`, `--env`, `--join-cgroup` et gestion des credentials : utilisons-les seulement lorsque nous savons ce que nous voulons reproduire.

---

# Chapitre 19 — API C minimale

## 19.1. Rejoindre un UTS namespace avec `setns()`

```c
#define _GNU_SOURCE
#include <fcntl.h>
#include <sched.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char **argv) {
    if (argc != 2) {
        fprintf(stderr, "usage: %s /proc/PID/ns/uts\n", argv[0]);
        return EXIT_FAILURE;
    }

    int fd = open(argv[1], O_RDONLY | O_CLOEXEC);
    if (fd == -1) {
        perror("open");
        return EXIT_FAILURE;
    }

    if (setns(fd, CLONE_NEWUTS) == -1) {
        perror("setns");
        close(fd);
        return EXIT_FAILURE;
    }

    close(fd);
    execlp("hostname", "hostname", NULL);
    perror("execlp");
    return EXIT_FAILURE;
}
```

Compilation :

```bash
cc -Wall -Wextra -O2 setns_uts.c -o setns_uts
```

Exécution selon les droits :

```bash
sudo ./setns_uts /proc/<PID>/ns/uts
```

## 19.2. Pourquoi préférer les FDs

Une API FD-based permet d'éviter plusieurs classes de races basées sur des noms ou PID réutilisés.

Linux moderne tend à fournir davantage d'APIs de ce type :

- pidfds ;
- nouvelle API mount ;
- namespace FDs.

---

# Chapitre 20 — Sécurité : modèle de menace

## 20.1. Ce que les namespaces protègent bien

Ils réduisent :

- la visibilité ;
- les collisions ;
- les interactions accidentelles ;
- l'exposition de certaines ressources globales.

## 20.2. Ce qu'ils ne protègent pas seuls

Ils ne protègent pas automatiquement contre :

- vulnérabilité noyau ;
- syscall dangereux autorisé ;
- capability excessive ;
- device sensible exposé ;
- bind mount dangereux ;
- socket Docker exposée ;
- secret injecté ;
- déni de service sans cgroup ;
- réseau trop permissif.

## 20.3. Checklist conteneur

Avant de considérer un workload isolé :

- [ ] user non root si possible ;
- [ ] user namespace/rootless lorsque pertinent ;
- [ ] capabilities supprimées par défaut ;
- [ ] pas de `--privileged` sans justification forte ;
- [ ] seccomp actif ;
- [ ] AppArmor/SELinux actif lorsque disponible ;
- [ ] `no_new_privs` ;
- [ ] rootfs read-only si possible ;
- [ ] `/proc` et `/sys` correctement restreints ;
- [ ] aucun bind mount sensible ;
- [ ] pas de socket Docker dans un workload non fiable ;
- [ ] limites cgroup ;
- [ ] politique réseau ;
- [ ] secrets hors image ;
- [ ] noyau maintenu et patché.

---

# Chapitre 21 — Bonnes pratiques d'administration

## 21.1. Vérifier les fonctions disponibles

```bash
uname -r
unshare --version
nsenter --version
lsns --version
```

Puis :

```bash
unshare --help
nsenter --help
```

Ne supposons pas qu'une option util-linux récente existe sur une distribution LTS plus ancienne.

## 21.2. Privilégier les outils de haut niveau en production

Pour un workload de production :

```text
runtime OCI / systemd sandboxing / orchestration
                ↓
namespaces configurés de manière cohérente
```

Créer manuellement des namespaces est excellent pour apprendre et diagnostiquer, mais rarement le bon niveau d'abstraction pour exploiter une flotte.

## 21.3. Ne pas automatiser avec des PID seuls

Préférer, quand l'API le permet :

- runtime/container ID ;
- cgroup ;
- pidfd ;
- références namespace persistantes contrôlées.

Un PID numérique est réutilisable.

## 21.4. Journaliser le contexte

Pour diagnostiquer un incident, capturons :

```bash
uname -a
lsns
cat /proc/<PID>/status
cat /proc/<PID>/cgroup
ls -l /proc/<PID>/ns
cat /proc/<PID>/uid_map
cat /proc/<PID>/gid_map
```

ainsi que :

- commande de lancement ;
- cgroup ;
- runtime ;
- image/digest ;
- politique seccomp/LSM ;
- mounts ;
- réseau.

---

# Chapitre 22 — Erreurs classiques

## Erreur 1 — « un namespace est un conteneur »

Faux.

Un conteneur est un assemblage de plusieurs mécanismes.

## Erreur 2 — « root dans un user namespace = root hôte »

Faux.

Les privilèges sont bornés par la hiérarchie des user namespaces et la propriété des ressources.

## Erreur 3 — « le cgroup namespace limite CPU/RAM »

Faux.

Il virtualise une vue ; les contrôleurs cgroup imposent les limites.

## Erreur 4 — « time namespace change la date système »

Faux.

`CLOCK_REALTIME` n'est pas virtualisée par les time namespaces.

## Erreur 5 — « mount namespace = fichiers privés »

Faux.

Les processus peuvent voir des mounts différents tout en accédant aux mêmes objets sous-jacents.

## Erreur 6 — « network namespace = firewall »

Faux.

Il sépare les piles réseau, mais il faut encore contrôler routes, forwarding et filtrage.

## Erreur 7 — « `chroot` suffit à isoler un filesystem »

Faux.

Il ne remplace pas un mount namespace et les autres couches de confinement.

## Erreur 8 — « `nsenter --all` reproduit exactement le conteneur »

Faux.

Namespace membership n'est qu'une partie du contexte d'exécution.

## Erreur 9 — oublier la propagation des mounts

Peut provoquer des montages visibles là où nous ne l'attendions pas.

## Erreur 10 — oublier PID 1 et la récolte des zombies

Un workload conteneurisé doit gérer correctement le rôle init.

---

# Chapitre 23 — Travaux pratiques

## TP 1 — Inventorier les namespaces de la machine

Objectifs :

1. lancer `lsns` ;
2. compter les namespaces de chaque type ;
3. identifier ceux du PID 1 ;
4. comparer avec notre shell ;
5. documenter les différences.

## TP 2 — UTS namespace

Créer un UTS namespace, changer le hostname et prouver que l'hôte n'est pas modifié.

## TP 3 — PID namespace

Créer :

```bash
sudo unshare --fork --pid --mount-proc --kill-child bash
```

Observer PID 1, créer deux processus enfants, puis étudier leur visibilité depuis l'hôte.

## TP 4 — Mount namespace et propagation

1. créer un mount namespace ;
2. monter un `tmpfs` ;
3. comparer `findmnt` dans et hors namespace ;
4. étudier `private`, `shared` et `slave` sur un environnement de lab.

## TP 5 — Deux network namespaces

Construire :

```text
ns-a 10.42.0.1/24
   |
  veth
   |
ns-b 10.42.0.2/24
```

Tester ping et observer les sockets.

## TP 6 — User namespace

Avec un compte de test :

```bash
unshare --user --map-root-user bash
```

Comparer :

```bash
id
cat /proc/self/uid_map
cat /proc/self/status | grep '^Cap'
```

## TP 7 — Subordinate IDs

Si la machine de lab le permet :

1. inspecter `/etc/subuid` et `/etc/subgid` ;
2. créer un mapping automatique ;
3. créer un fichier appartenant à un UID interne différent ;
4. observer l'UID vu depuis l'hôte.

## TP 8 — Cgroup namespace

Comparer `/proc/self/cgroup` :

- avant ;
- après `unshare --cgroup` ;
- dans un conteneur réel.

Expliquer pourquoi cela ne change pas `memory.max` à lui seul.

## TP 9 — Time namespace

Vérifier :

```bash
cat /proc/self/timens_offsets
```

Créer un time namespace dans un lab compatible et observer les horloges monotonic/boottime.

Démontrer que la date civile globale n'est pas simplement modifiée.

## TP 10 — Diagnostiquer un conteneur Docker

Pour un conteneur de test :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' <conteneur>)
ls -l /proc/$pid/ns
lsns --task "$pid"
cat /proc/$pid/cgroup
```

Comparer avec `docker exec` et `nsenter`.

## TP 11 — Sandboxing systemd

Créer une unité de lab avec :

```ini
PrivateTmp=yes
PrivateNetwork=yes
ProtectSystem=strict
NoNewPrivileges=yes
```

Analyser :

```bash
systemd-analyze security lab.service
```

Observer les namespaces créés.

## TP 12 — Mini-conteneur documenté

Construire un pseudo-conteneur pédagogique avec :

- PID ;
- mount ;
- UTS ;
- IPC ;
- user namespace si possible ;
- `/proc` cohérent ;
- rootfs minimal.

Produire ensuite un rapport répondant à :

1. quelles ressources sont réellement isolées ?
2. lesquelles sont encore partagées ?
3. quelles capabilities restent disponibles ?
4. quelles limites de ressources existent ?
5. quelle frontière de sécurité manque encore ?

---

# Chapitre 24 — Projet final : expliquer un conteneur à partir du noyau

## Objectif

Choisir un conteneur réel de laboratoire et reconstruire son modèle d'isolation à partir de l'hôte.

## Livrables

### 1. Carte des namespaces

```text
PID hôte
├── mnt : ...
├── uts : ...
├── ipc : ...
├── pid : ...
├── net : ...
├── user: ...
├── cgroup: ...
└── time: ...
```

### 2. Cgroups

Documenter :

- chemin cgroup ;
- `memory.max` ;
- CPU ;
- `pids.max` ;
- IO si pertinent.

### 3. Filesystem

Documenter :

- rootfs ;
- OverlayFS éventuel ;
- bind mounts ;
- volumes ;
- propagation ;
- read-only/read-write.

### 4. Sécurité

Documenter :

- UID réel ;
- user namespace ;
- capabilities ;
- seccomp ;
- AppArmor/SELinux ;
- `no_new_privs` ;
- devices.

### 5. Réseau

Documenter :

- network namespace ;
- interfaces ;
- veth ;
- bridge ;
- routes ;
- DNS ;
- filtrage.

### 6. Conclusion

Répondre à la question :

```text
En quoi ce workload est-il isolé,
et quelle partie de cette isolation dépend encore du noyau hôte ?
```

---

# Chapitre 25 — Aide-mémoire

## Observer

```bash
lsns
lsns --task <PID>
ls -l /proc/<PID>/ns
readlink /proc/<PID>/ns/net
cat /proc/<PID>/cgroup
cat /proc/<PID>/uid_map
cat /proc/<PID>/gid_map
```

## Créer

```bash
unshare --uts bash
unshare --mount bash
unshare --net bash
unshare --ipc bash
unshare --user --map-root-user bash
unshare --fork --pid --mount-proc bash
unshare --cgroup bash
unshare --time --fork bash
```

## Rejoindre

```bash
nsenter --target <PID> --net bash
nsenter --target <PID> --mount bash
nsenter --target <PID> --all --fork bash
```

## Réseau

```bash
ip netns list
ip netns add ns-a
ip netns exec ns-a ip addr
ip link add veth-a type veth peer name veth-b
ip link set veth-a netns ns-a
```

## Mounts

```bash
findmnt
findmnt -o TARGET,PROPAGATION
cat /proc/self/mountinfo
```

## Capabilities

```bash
capsh --print
grep '^Cap' /proc/self/status
```

## Cgroups

```bash
cat /proc/self/cgroup
cat /sys/fs/cgroup/cgroup.controllers
```

---

# Chapitre 26 — Questions de révision

1. Quelle différence fondamentale existe entre un namespace et une VM ?
2. Quel mécanisme impose une limite mémoire : namespace ou cgroup ?
3. Pourquoi faut-il souvent utiliser `--fork` avec un PID namespace ?
4. Pourquoi faut-il remonter `/proc` dans un nouveau PID namespace ?
5. Qu'isole exactement un cgroup namespace ?
6. Pourquoi UID 0 dans un user namespace n'est-il pas root sur l'hôte ?
7. À quoi servent `/etc/subuid` et `/etc/subgid` ?
8. Qu'est-ce qu'un idmapped mount ?
9. Quelle différence existe entre un bind mount et une copie de fichier ?
10. Pourquoi la mount propagation est-elle importante ?
11. Quelles horloges un time namespace virtualise-t-il ?
12. Pourquoi `CLOCK_REALTIME` est-elle un contre-exemple important ?
13. Qu'est-ce qu'un pidfd apporte par rapport à un PID numérique ?
14. Quelle différence existe entre `nsenter --cgroup` et `--join-cgroup` ?
15. Quelles couches ajouter à des namespaces pour obtenir un confinement robuste ?
16. Pourquoi `chroot()` ne constitue-t-il pas une sandbox ?
17. Quelle relation existe entre user namespaces et capabilities ?
18. Pourquoi rootless réduit-il certains risques sans supprimer les risques noyau ?
19. Que partage typiquement un Pod Kubernetes entre ses conteneurs ?
20. Quelle commande utiliser pour comparer rapidement les namespaces de deux processus ?

---

# Conclusion

Les namespaces sont une des primitives fondamentales qui ont rendu possible la conteneurisation Linux moderne.

Le point le plus important n'est pourtant pas de retenir huit noms, mais de garder le modèle suivant :

```text
un processus
   |
   +-- voit certaines ressources via ses namespaces
   |
   +-- consomme des ressources via ses cgroups
   |
   +-- possède des droits via UID/GID + capabilities
   |
   +-- peut être filtré par seccomp
   |
   +-- peut être confiné par AppArmor/SELinux/Landlock
   |
   +-- partage toujours le noyau Linux de l'hôte
```

Comprendre ce modèle permet de raisonner correctement sur Docker, Podman, Kubernetes, systemd, les conteneurs rootless et de nombreux systèmes de sandboxing.

La compétence réellement utile consiste donc à savoir répondre, pour n'importe quel processus :

```text
Que voit-il ?
Que peut-il faire ?
Que peut-il consommer ?
Avec qui partage-t-il le noyau et les ressources ?
```

---

# Références

## Documentation noyau Linux

- Linux namespaces : <https://docs.kernel.org/admin-guide/namespaces/index.html>
- User namespaces et resource control : <https://docs.kernel.org/admin-guide/namespaces/resource-control.html>
- cgroup v2 : <https://docs.kernel.org/admin-guide/cgroup-v2.html>
- Idmapped mounts : <https://docs.kernel.org/filesystems/idmappings.html>
- Shared subtrees / mount propagation : <https://docs.kernel.org/filesystems/sharedsubtree.html>
- `/proc` : <https://docs.kernel.org/filesystems/proc.html>
- Landlock : <https://docs.kernel.org/userspace-api/landlock.html>

## Linux man-pages

- `namespaces(7)` : <https://man7.org/linux/man-pages/man7/namespaces.7.html>
- `user_namespaces(7)` : <https://man7.org/linux/man-pages/man7/user_namespaces.7.html>
- `pid_namespaces(7)` : <https://man7.org/linux/man-pages/man7/pid_namespaces.7.html>
- `mount_namespaces(7)` : <https://man7.org/linux/man-pages/man7/mount_namespaces.7.html>
- `network_namespaces(7)` : <https://man7.org/linux/man-pages/man7/network_namespaces.7.html>
- `ipc_namespaces(7)` : <https://man7.org/linux/man-pages/man7/ipc_namespaces.7.html>
- `cgroup_namespaces(7)` : <https://man7.org/linux/man-pages/man7/cgroup_namespaces.7.html>
- `time_namespaces(7)` : <https://man7.org/linux/man-pages/man7/time_namespaces.7.html>
- `clone(2)` : <https://man7.org/linux/man-pages/man2/clone.2.html>
- `setns(2)` : <https://man7.org/linux/man-pages/man2/setns.2.html>
- `unshare(2)` : <https://man7.org/linux/man-pages/man2/unshare.2.html>

## util-linux

- `unshare(1)` : <https://man7.org/linux/man-pages/man1/unshare.1.html>
- `nsenter(1)` : <https://man7.org/linux/man-pages/man1/nsenter.1.html>
- `lsns(8)` : <https://man7.org/linux/man-pages/man8/lsns.8.html>
