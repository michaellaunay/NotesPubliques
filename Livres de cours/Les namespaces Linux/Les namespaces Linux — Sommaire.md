---
schema_version: 1
uid: 01M1BQ6258ZDKVB1G7P6MNR2DS
titre: "Les namespaces Linux — Sommaire"
aliases:
  - "Livre Les namespaces Linux"
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
resume: "Sommaire du livre « Les namespaces Linux » : 16 chapitres (68 691 mots), compléments 2026 et conclusion ; version longue du cours [[Les namespaces Linux]], découpée le 31 août 2026."
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
> [!info] Livre de cours
> Version longue du cours [[Les namespaces Linux]], découpée en chapitres le 31 août 2026 à partir de l'état du dépôt au commit `d6d76bd` (2026-08-29, environ 70 869 mots). La version condensée et actualisée reste dans `cours/` ; ses apports propres sont réunis dans [[Les namespaces Linux — Compléments 2026]].


# Les namespaces Linux — Sommaire


À la fin du cours, nous savons expliquer le rôle des namespaces Linux, comprendre leur place dans l’isolation des processus, manipuler les principaux types de namespaces, et faire le lien avec les conteneurs, Docker, Kubernetes, systemd, `/proc`, les cgroups et la sécurité.


## Chapitres

1. [[Les namespaces Linux — 01 — Introduction aux namespaces Linux|Introduction aux namespaces Linux]] — 3 725 mots
2. [[Les namespaces Linux — 02 — Architecture générale des namespaces|Architecture générale des namespaces]] — 3 564 mots
3. [[Les namespaces Linux — 03 — Les outils de manipulation des namespaces|Les outils de manipulation des namespaces]] — 3 945 mots
4. [[Les namespaces Linux — 04 — Namespace UTS|Namespace UTS]] — 3 243 mots
5. [[Les namespaces Linux — 05 — Namespace PID|Namespace PID]] — 3 397 mots
6. [[Les namespaces Linux — 06 — Namespace mount|Namespace mount]] — 5 314 mots
7. [[Les namespaces Linux — 07 — Namespace network|Namespace network]] — 5 945 mots
8. [[Les namespaces Linux — 08 — Namespace IPC|Namespace IPC]] — 3 936 mots
9. [[Les namespaces Linux — 09 — Namespace user|Namespace user]] — 4 251 mots
10. [[Les namespaces Linux — 10 — Namespace cgroup|Namespace cgroup]] — 3 980 mots
11. [[Les namespaces Linux — 11 — Namespace time|Namespace time]] — 3 977 mots
12. [[Les namespaces Linux — 12 — Namespaces et proc|Namespaces et /proc]] — 4 051 mots
13. [[Les namespaces Linux — 13 — Namespaces et cgroups|Namespaces et cgroups]] — 4 372 mots
14. [[Les namespaces Linux — 14 — Namespaces, capabilities et sécurité|Namespaces, capabilities et sécurité]] — 4 992 mots
15. [[Les namespaces Linux — 15 — Des namespaces aux conteneurs|Des namespaces aux conteneurs]] — 4 745 mots
16. [[Les namespaces Linux — 16 — Namespaces dans Kubernetes|Namespaces dans Kubernetes]] — 5 254 mots

## Compléments et conclusion

- [[Les namespaces Linux — Compléments 2026]] — sections propres à la version condensée de 2026
- Version condensée et actualisée : [[Les namespaces Linux]]

## Plan détaillé (résumés de chapitres de la version longue)

### Chapitre 2 — Architecture générale des namespaces
- Notion d’espace de noms
    
- Un processus appartient à plusieurs namespaces
    
- Héritage des namespaces lors de `fork`
    
- Changement de namespace avec `setns`
    
- Création avec `clone` et `unshare`
    
- Observation des namespaces via `/proc/<PID>/ns`
    
- Identifiants symboliques des namespaces
    
- Durée de vie d’un namespace
    
- Namespace sans processus mais maintenu par un bind mount
    

Commandes principales :

```bash
ls -l /proc/$$/ns
readlink /proc/$$/ns/pid
readlink /proc/1/ns/pid
```

---

### Chapitre 3 — Les outils de manipulation
- `unshare`
    
- `nsenter`
    
- `lsns`
    
- `ip netns`
    
- `mount`
    
- `findmnt`
    
- `ps`
    
- `ip`
    
- `hostname`
    
- `capsh`
    
- `newuidmap` et `newgidmap`
    

Objectif pratique :

```bash
unshare --fork --pid --mount-proc bash
nsenter --target <PID> --pid --mount --uts --ipc --net bash
lsns
```

Nous apprenons à créer, entrer et inspecter des namespaces sans passer par Docker.

---

### Chapitre 4 — Namespace UTS
- Rôle du namespace UTS
    
- Isolation du hostname
    
- Isolation du domainname
    
- Différence entre hostname global et hostname dans un conteneur
    
- Expérience avec `unshare --uts`
    
- Lien avec Docker `--hostname`
    

Commandes :

```bash
hostname
unshare --uts bash
hostname conteneur-test
hostname
exit
hostname
```

Objectif : comprendre qu’un processus peut voir un hostname différent de celui de l’hôte.

---

### Chapitre 5 — Namespace PID
- Rôle du namespace PID
    
- PID vus depuis l’hôte et depuis le namespace
    
- Le processus PID 1 dans un namespace
    
- Responsabilités particulières du PID 1
    
- Gestion des signaux
    
- Processus zombies et réapage
    
- `/proc` dans un namespace PID
    
- Pourquoi un conteneur a souvent son propre PID 1
    

Commandes :

```bash
unshare --fork --pid --mount-proc bash
ps aux
echo $$
cat /proc/1/comm
```

Points essentiels :

- un même processus peut avoir plusieurs PID selon le namespace ;
    
- `/proc` doit être remonté pour refléter correctement le namespace PID ;
    
- le PID 1 a un comportement particulier vis-à-vis des signaux et des zombies.
    

---

### Chapitre 6 — Namespace mount
- Rôle du namespace de montage
    
- Isolation des points de montage
    
- Propagation des montages
    
- `private`, `shared`, `slave`, `unbindable`
    
- `pivot_root`
    
- `chroot` vs namespace mount
    
- Montage de `/proc`
    
- Montage de `/sys`
    
- Bind mounts
    
- Root filesystem d’un conteneur
    

Commandes :

```bash
unshare --mount bash
mount --make-rprivate /
mkdir /tmp/test-mnt
mount -t tmpfs tmpfs /tmp/test-mnt
findmnt /tmp/test-mnt
exit
findmnt /tmp/test-mnt
```

Objectif : comprendre qu’un processus peut voir une arborescence de montages différente de celle de l’hôte.

---

### Chapitre 7 — Namespace network
- Rôle du namespace réseau
    
- Interfaces réseau isolées
    
- Loopback propre au namespace
    
- Tables de routage séparées
    
- Firewall et règles réseau par namespace
    
- Sockets isolées
    
- Création d’une paire `veth`
    
- Pont réseau Linux
    
- NAT
    
- Lien avec Docker bridge
    
- Lien avec Kubernetes CNI
    

Commandes :

```bash
unshare --net bash
ip addr
ip link set lo up
ip addr
```

TP plus avancé :

```bash
ip netns add ns1
ip link add veth-host type veth peer name veth-ns
ip link set veth-ns netns ns1
ip addr add 10.0.0.1/24 dev veth-host
ip link set veth-host up
ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns
ip netns exec ns1 ip link set veth-ns up
ip netns exec ns1 ip link set lo up
ping 10.0.0.2
```

---

### Chapitre 8 — Namespace IPC
- Rôle du namespace IPC
    
- Isolation des mécanismes System V IPC
    
- Files de messages
    
- Sémaphores
    
- Mémoire partagée
    
- IPC POSIX
    
- Cas d’usage en conteneurs
    
- Risques de partage IPC avec l’hôte
    
- Option Docker `--ipc`
    

Commandes :

```bash
ipcs
unshare --ipc bash
ipcs
```

Objectif : comprendre que plusieurs processus peuvent être isolés au niveau de leurs mécanismes de communication inter-processus.

---

### Chapitre 9 — Namespace user
- Rôle du namespace user
    
- Mapping UID/GID
    
- Être root dans un namespace sans être root sur l’hôte
    
- `/proc/<PID>/uid_map`
    
- `/proc/<PID>/gid_map`
    
- Capabilities dans un user namespace
    
- Rootless containers
    
- Sécurité et risques
    
- `newuidmap`, `newgidmap`
    
- `/etc/subuid` et `/etc/subgid`
    

Commandes :

```bash
unshare --user --map-root-user bash
id
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Point central :

```text
Nous pouvons être UID 0 dans le namespace tout en restant un utilisateur non privilégié sur l’hôte.
```

C’est un mécanisme clé des conteneurs rootless.

---

### Chapitre 10 — Namespace cgroup
- Rôle du namespace cgroup
    
- Isolation de la vue des cgroups
    
- Différence entre cgroup namespace et mécanisme cgroups
    
- `/proc/self/cgroup`
    
- Intérêt pour les conteneurs
    
- Masquage partiel de l’organisation de l’hôte
    
- Lien avec systemd et Kubernetes
    

Commandes :

```bash
cat /proc/self/cgroup
lsns -t cgroup
```

Objectif : comprendre que le namespace cgroup isole la vue, mais que les limites de ressources sont définies par les cgroups eux-mêmes.

---

### Chapitre 11 — Namespace time
- Rôle du namespace time
    
- Isolation partielle des horloges
    
- `CLOCK_MONOTONIC`
    
- `CLOCK_BOOTTIME`
    
- Différence avec l’heure réelle du système
    
- Cas d’usage : tests, checkpoint/restore, environnements isolés
    
- Limites du namespace time
    

Commandes exploratoires :

```bash
ls -l /proc/$$/ns/time
cat /proc/self/timens_offsets
```

Objectif : comprendre que le time namespace ne permet pas simplement de changer l’heure système globale, mais d’ajuster certaines horloges vues par les processus.

---

### Chapitre 12 — Namespaces et `/proc`
- `/proc/<PID>/ns`
    
- Lire les namespaces d’un processus
    
- Comparer deux processus
    
- PID namespace et `/proc`
    
- Network namespace et `/proc/net`
    
- Mount namespace et `/proc/mounts`
    
- User namespace et UID/GID maps
    
- Sécurité des informations exposées dans `/proc`
    

Commandes :

```bash
ls -l /proc/<PID>/ns
readlink /proc/<PID>/ns/net
readlink /proc/<PID>/ns/pid
cat /proc/<PID>/uid_map
cat /proc/<PID>/gid_map
```

Ce chapitre fait le lien direct avec le cours sur `/proc`.

---

### Chapitre 13 — Namespaces et cgroups
- Différence entre isolation et limitation
    
- Namespaces : ce que le processus voit
    
- Cgroups : ce que le processus peut consommer
    
- CPU, mémoire, I/O, pids
    
- cgroups v1 vs cgroups v2
    
- systemd comme gestionnaire de cgroups
    
- Kubernetes requests et limits
    
- OOMKilled
    
- Exemple de conteneur avec namespace isolé mais ressources limitées
    

Synthèse :

```text
Namespaces = isolation des vues.
Cgroups = contrôle des ressources.
Capabilities = contrôle des privilèges.
Seccomp/AppArmor/SELinux = réduction de la surface d’attaque.
```

---

### Chapitre 14 — Namespaces, capabilities et sécurité
- Pourquoi les namespaces ne suffisent pas
    
- Capabilities Linux
    
- `CAP_SYS_ADMIN`
    
- Capabilities dans un user namespace
    
- Seccomp
    
- AppArmor
    
- SELinux
    
- Risques des conteneurs privilégiés
    
- Montage dangereux de `/proc`, `/sys`, `/dev`
    
- Évasion de conteneur : principes généraux sans exploitation
    
- Bonnes pratiques de durcissement
    

Commandes :

```bash
capsh --print
grep Cap /proc/$$/status
```

Objectif : comprendre que l’isolation repose sur plusieurs couches, pas uniquement sur les namespaces.

---

### Chapitre 15 — Des namespaces aux conteneurs
- Comment Docker combine les namespaces
    
- Ce que fait un runtime OCI
    
- Image, root filesystem et mount namespace
    
- Processus principal et PID namespace
    
- Réseau bridge et network namespace
    
- Volumes et bind mounts
    
- Variables d’environnement
    
- Entrypoint et PID 1
    
- Différence entre image et conteneur
    
- Rôle de `runc`, containerd, Docker
    

Démonstration :

```bash
docker run --rm -it alpine sh
ps
hostname
ip addr
mount
cat /proc/1/cgroup
```

Puis comparaison depuis l’hôte :

```bash
docker inspect <container>
lsns
ps aux | grep <process>
```

---

### Chapitre 16 — Namespaces dans Kubernetes
- Pod comme unité de partage de certains namespaces
    
- Containers d’un même pod
    
- Partage du network namespace dans un pod
    
- Pause container
    
- PID namespace dans Kubernetes
    
- Volumes et mount namespaces
    
- CNI et network namespaces
    
- SecurityContext
    
- Privileged containers
    
- hostNetwork, hostPID, hostIPC
    
- Debug avec ephemeral containers
    

Points importants :

```text
Dans un pod Kubernetes, plusieurs containers partagent généralement le même namespace réseau.
Ils ont donc la même IP et peuvent communiquer via localhost.
```

---

### Chapitre 17 — Travaux pratiques
## TP 1 — Observer les namespaces d’un processus

- Lister `/proc/$$/ns`
    
- Comparer avec `/proc/1/ns`
    
- Utiliser `lsns`
    
- Identifier les namespaces partagés ou distincts
    

## TP 2 — Créer un namespace UTS

- Créer un shell isolé
    
- Modifier le hostname
    
- Vérifier que l’hôte n’est pas modifié
    

## TP 3 — Créer un namespace PID

- Créer un PID namespace
    
- Monter `/proc`
    
- Observer le PID 1
    
- Comprendre les limites du PID 1
    

## TP 4 — Créer un namespace réseau

- Créer deux namespaces réseau
    
- Connecter avec une paire `veth`
    
- Configurer IP et routes
    
- Tester avec `ping`
    

## TP 5 — Créer un environnement pseudo-conteneur

- Namespace mount
    
- Namespace UTS
    
- Namespace PID
    
- Montage de `/proc`
    
- Root filesystem minimal
    
- Lancement d’un shell isolé
    

## TP 6 — User namespace rootless

- Créer un user namespace
    
- Mapper l’utilisateur courant vers root
    
- Observer `uid_map`
    
- Tester les limites de privilèges
    

---

### Chapitre 18 — Étude de cas : construire un mini-conteneur
Nous construisons progressivement un mini-conteneur à la main, sans Docker.

Étapes :

1. Préparer un root filesystem minimal.
    
2. Créer un namespace UTS.
    
3. Créer un namespace PID.
    
4. Créer un namespace mount.
    
5. Monter `/proc`.
    
6. Isoler le réseau.
    
7. Ajouter une paire `veth`.
    
8. Configurer une route.
    
9. Ajouter éventuellement un user namespace.
    
10. Lancer un processus isolé.
    
11. Comparer avec un vrai conteneur Docker.
    

Objectif : comprendre concrètement ce qu’un runtime de conteneur automatise.

---

### Chapitre 19 — Limites des namespaces
- Les namespaces isolent les vues, pas tout le système
    
- Le noyau reste partagé
    
- Risques liés aux capabilities
    
- Risques liés aux montages
    
- Risques liés aux périphériques
    
- Risques liés à `/proc` et `/sys`
    
- Complexité des user namespaces
    
- Problèmes de performance et d’observabilité
    
- Debug plus difficile
    
- Écart entre isolation conteneur et isolation VM
    
- Cas où une VM reste préférable
    

---

### Chapitre 20 — Évaluation proposée
## Partie théorique

Questions possibles :

- Qu’est-ce qu’un namespace Linux ?
    
- Quelle différence entre namespace et cgroup ?
    
- Pourquoi le PID 1 est-il particulier dans un namespace PID ?
    
- Comment un processus peut-il être root dans un user namespace sans être root sur l’hôte ?
    
- Pourquoi un pod Kubernetes partage-t-il souvent le namespace réseau ?
    
- Quels risques présente un conteneur privilégié ?
    

## Partie pratique

Sujet possible :

```text
Nous créons un environnement isolé à l’aide de unshare et nsenter.
Nous devons isoler le hostname, les PID, les montages et le réseau.
Nous devons ensuite expliquer quelles parties sont réellement isolées et lesquelles ne le sont pas.
```

## Mini-projet

Construire un script `mini-container.sh` capable de :

- créer un namespace UTS ;
    
- créer un namespace PID ;
    
- créer un namespace mount ;
    
- monter `/proc` ;
    
- lancer un shell dans un root filesystem minimal ;
    
- afficher les namespaces actifs ;
    
- nettoyer correctement à la fin.
    

---

# Progression pédagogique proposée

## Partie I — Concepts fondamentaux

- Chapitres 1 à 3
    
- Objectif : comprendre ce que sont les namespaces et comment les observer.
    

## Partie II — Étude des namespaces un par un

- Chapitres 4 à 11
    
- Objectif : comprendre UTS, PID, mount, network, IPC, user, cgroup et time.
    

## Partie III — Intégration système

- Chapitres 12 à 16
    
- Objectif : relier namespaces, `/proc`, cgroups, sécurité, Docker et Kubernetes.
    

## Partie IV — Travaux pratiques et mini-conteneur

- Chapitres 17 à 20
    
- Objectif : manipuler directement les namespaces et construire une compréhension concrète des conteneurs.

