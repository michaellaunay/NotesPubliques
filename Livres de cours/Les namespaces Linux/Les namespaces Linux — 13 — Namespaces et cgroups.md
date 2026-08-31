---
schema_version: 1
uid: 01M1BQ624MYYRSPE1V56J6256S
titre: "Les namespaces Linux — 13 — Namespaces et cgroups"
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
resume: "Chapitre 13 sur 16 du livre « Les namespaces Linux » : Namespaces et cgroups. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 13/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 12 — Namespaces et proc|← 12 — Namespaces et proc]] · [[Les namespaces Linux — 14 — Namespaces, capabilities et sécurité|14 — Namespaces, capabilities et sécurité →]]

# Chapitre 13 — Namespaces et cgroups
## Objectifs du chapitre

Dans ce chapitre, nous clarifions la relation entre deux mécanismes fondamentaux de Linux :

```text
namespaces
cgroups
```

Ces deux mécanismes sont souvent associés aux conteneurs, mais ils ne jouent pas le même rôle.

Nous devons retenir dès le départ :

```text
Les namespaces isolent ce que les processus voient.
Les cgroups contrôlent ce que les processus consomment.
```

À la fin de ce chapitre, nous savons :

- distinguer clairement namespaces et cgroups ;
    
- expliquer pourquoi les deux sont nécessaires aux conteneurs ;
    
- comprendre les ressources contrôlées par les cgroups ;
    
- lire les limites CPU, mémoire et processus avec cgroups v2 ;
    
- comprendre la différence entre cgroups v1 et cgroups v2 ;
    
- faire le lien avec systemd ;
    
- faire le lien avec Docker, Podman et Kubernetes ;
    
- expliquer ce qu’est un `OOMKilled` ;
    
- diagnostiquer une limite de ressources dans un conteneur.
    

---

## 13.1. Pourquoi comparer namespaces et cgroups ?

## 13.1.1. Deux mécanismes souvent confondus

Quand nous parlons de conteneurs, nous disons souvent :

```text
Docker isole les processus.
Kubernetes limite les ressources.
Un conteneur a son propre réseau.
Un pod a une limite mémoire.
```

Derrière ces phrases, plusieurs mécanismes Linux interviennent.

Les namespaces permettent de créer des vues isolées :

```text
vue des processus
vue réseau
vue des montages
vue des utilisateurs
vue du hostname
vue des IPC
```

Les cgroups permettent de contrôler les ressources :

```text
CPU
mémoire
I/O
nombre de processus
périphériques
```

Un conteneur complet utilise donc les deux.

---

## 13.1.2. Exemple simple

Imaginons un processus dans un conteneur.

Grâce aux namespaces, il peut voir :

```text
PID 1
hostname app-1
interface eth0
racine / propre au conteneur
seulement ses propres processus
```

Mais sans cgroups, il pourrait potentiellement consommer trop de ressources :

```text
toute la mémoire disponible
trop de CPU
trop de processus
trop d'I/O disque
```

Les namespaces ne suffisent donc pas.

Inversement, avec seulement des cgroups, nous pourrions limiter la mémoire et le CPU, mais le processus verrait encore l’environnement de l’hôte.

Nous avons besoin des deux.

---

## 13.2. Synthèse fondamentale

## 13.2.1. Namespaces : isolation des vues

Les namespaces répondent à la question :

```text
Que voit le processus ?
```

Exemples :

|Namespace|Ce qu’il isole|
|---|---|
|`pid`|les identifiants et la vue des processus|
|`net`|interfaces, routes, sockets, ports|
|`mnt`|points de montage|
|`uts`|hostname et domainname|
|`ipc`|ressources IPC|
|`user`|UID, GID et capabilities|
|`cgroup`|vue des chemins cgroups|
|`time`|offsets de certaines horloges|

Les namespaces changent donc la perception du système.

---

## 13.2.2. Cgroups : contrôle des ressources

Les cgroups répondent à la question :

```text
Que peut consommer le processus ?
```

Ils permettent notamment de contrôler :

|Ressource|Exemple de contrôle|
|---|---|
|CPU|quota, poids, répartition|
|mémoire|limite maximale, consommation actuelle|
|I/O|limitation ou observation des accès disque|
|pids|nombre maximal de processus/threads|
|devices|accès à certains périphériques|
|pressure|observation de la pression CPU, mémoire, I/O|

Les cgroups ne changent pas forcément ce que le processus voit. Ils imposent des règles de consommation.

---

## 13.2.3. Résumé

Nous retenons :

```text
Namespaces = isolation des vues.
Cgroups = contrôle et comptabilité des ressources.
Capabilities = contrôle fin des privilèges.
Seccomp/AppArmor/SELinux = réduction de la surface d’attaque.
```

Un conteneur sérieux combine ces mécanismes.

---

## 13.3. Exemple : namespace PID sans cgroup pids

## 13.3.1. Situation

Nous créons un namespace PID :

```bash
unshare --fork --pid --mount-proc bash
```

Dans ce namespace, nous voyons peu de processus :

```bash
ps -ef
```

Nous pouvons avoir :

```text
UID   PID  PPID  CMD
user    1     0  bash
user    7     1  ps -ef
```

La vue est isolée.

---

## 13.3.2. Limite

Mais cela ne limite pas automatiquement le nombre de processus que nous pouvons créer.

Un processus mal écrit ou malveillant peut lancer de nombreux enfants :

```bash
:(){ :|:& };:
```

Cette commande est une fork bomb. Nous ne l’exécutons pas en vrai sur une machine non protégée.

Le namespace PID isole la vue des processus, mais ne limite pas le nombre de processus.

Pour limiter le nombre de processus, nous utilisons le contrôleur cgroup `pids`.

---

## 13.3.3. Contrôleur `pids`

Avec cgroups v2, nous pouvons lire :

```bash
cat /sys/fs/cgroup/pids.current
cat /sys/fs/cgroup/pids.max
```

Exemple :

```text
pids.current = 42
pids.max     = 512
```

Cela signifie que le cgroup contient actuellement 42 processus ou threads, avec une limite de 512.

Nous retenons :

```text
Namespace PID : ce que nous voyons.
Cgroup pids  : combien de processus nous pouvons créer.
```

---

## 13.4. Exemple : namespace network sans limite réseau

## 13.4.1. Situation

Nous créons un namespace réseau :

```bash
sudo ip netns add ns1
```

Nous lui donnons une interface `veth`, une IP et une route.

Le namespace réseau isole :

```text
interfaces
routes
sockets
ports
loopback
```

Un processus dans `ns1` ne voit pas directement les interfaces de l’hôte.

---

## 13.4.2. Limite

Le namespace réseau n’impose pas automatiquement :

- une limite de débit ;
    
- une limite de nombre de connexions ;
    
- une politique de qualité de service ;
    
- une interdiction d’émettre trop de trafic ;
    
- une règle de filtrage applicative.
    

Pour cela, nous avons besoin d’autres mécanismes :

```text
tc
nftables
iptables
eBPF
CNI Kubernetes
politiques réseau
cgroups selon les ressources concernées
```

Le namespace réseau isole la pile réseau. Il ne définit pas à lui seul toute la politique réseau.

---

## 13.5. Exemple : namespace mount sans quota disque

## 13.5.1. Situation

Le namespace mount permet à un processus de voir une arborescence de fichiers différente.

Dans un conteneur, nous pouvons avoir :

```text
/
├── bin
├── etc
├── app
├── proc
└── tmp
```

Cette racine n’est pas la racine réelle de l’hôte.

---

## 13.5.2. Limite

Le namespace mount ne limite pas automatiquement :

- la quantité de disque utilisée ;
    
- le nombre de fichiers créés ;
    
- les I/O disque ;
    
- la vitesse d’écriture ;
    
- l’espace consommé dans un volume.
    

Pour cela, nous avons besoin d’autres mécanismes :

```text
quotas filesystem
cgroups I/O
limites runtime
limites Kubernetes ephemeral-storage
politiques de stockage
```

Nous retenons :

```text
Namespace mount : quelle arborescence nous voyons.
Limites disque/I/O : combien nous pouvons écrire ou à quelle vitesse.
```

---

## 13.6. Les ressources contrôlées par les cgroups

## 13.6.1. CPU

Les cgroups peuvent contrôler l’usage CPU.

Avec cgroups v2 :

```bash
cat /sys/fs/cgroup/cpu.max
```

Sorties possibles :

```text
max 100000
```

Cela signifie qu’il n’y a pas de quota CPU explicite.

Autre sortie :

```text
200000 100000
```

Cela signifie que le cgroup peut utiliser 200000 microsecondes de CPU toutes les 100000 microsecondes.

Nous interprétons :

```text
200000 / 100000 = 2 CPU
```

---

## 13.6.2. Mémoire

Les cgroups peuvent mesurer et limiter la mémoire.

Avec cgroups v2 :

```bash
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
```

Exemple :

```text
memory.current = 120000000
memory.max     = 536870912
```

Cela signifie :

```text
consommation actuelle : environ 120 Mo
limite : 512 Mio
```

Si `memory.max` contient :

```text
max
```

cela signifie qu’aucune limite stricte n’est définie à ce niveau.

---

## 13.6.3. Nombre de processus

Le contrôleur `pids` limite le nombre de processus et threads.

```bash
cat /sys/fs/cgroup/pids.current
cat /sys/fs/cgroup/pids.max
```

Exemple :

```text
pids.current = 30
pids.max     = 256
```

Cela évite notamment qu’un processus crée un nombre illimité d’enfants.

---

## 13.6.4. I/O

Les cgroups peuvent aussi contrôler ou mesurer les I/O.

Avec cgroups v2, nous pouvons voir des fichiers comme :

```bash
cat /sys/fs/cgroup/io.stat
cat /sys/fs/cgroup/io.max
```

`io.stat` donne des statistiques d’I/O par périphérique.

`io.max` peut permettre de limiter certaines opérations selon les périphériques.

L’usage précis dépend du noyau, du contrôleur activé et du stockage utilisé.

---

## 13.6.5. Pression de ressources

Linux expose aussi des informations de pression, appelées PSI, _Pressure Stall Information_.

Exemples :

```bash
cat /proc/pressure/cpu
cat /proc/pressure/memory
cat /proc/pressure/io
```

Et parfois dans les cgroups :

```bash
cat /sys/fs/cgroup/cpu.pressure
cat /sys/fs/cgroup/memory.pressure
cat /sys/fs/cgroup/io.pressure
```

Ces informations indiquent si des tâches attendent des ressources.

Elles sont utiles pour diagnostiquer une saturation.

---

## 13.7. cgroups v1 et cgroups v2

## 13.7.1. Pourquoi deux versions ?

Les cgroups ont évolué.

Historiquement, Linux utilisait cgroups v1.

Aujourd’hui, les systèmes modernes utilisent de plus en plus cgroups v2.

Nous devons comprendre les deux, car nous pouvons encore rencontrer les deux modèles.

---

## 13.7.2. cgroups v1

Avec cgroups v1, chaque contrôleur peut avoir sa propre hiérarchie.

Exemple :

```text
memory
cpu,cpuacct
pids
blkio
devices
```

Dans `/proc/self/cgroup`, nous pouvons voir plusieurs lignes :

```text
12:memory:/user.slice
11:cpu,cpuacct:/user.slice
10:pids:/user.slice
```

Cela peut rendre l’analyse plus complexe.

---

## 13.7.3. cgroups v2

Avec cgroups v2, nous avons une hiérarchie unifiée.

Dans `/proc/self/cgroup`, nous voyons souvent une seule ligne :

```text
0::/user.slice/user-1000.slice/session-3.scope
```

Les fichiers de contrôle sont généralement dans :

```bash
/sys/fs/cgroup
```

Exemples :

```bash
cat /sys/fs/cgroup/cgroup.controllers
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/cpu.max
```

---

## 13.7.4. Vérifier la version utilisée

Nous pouvons vérifier le montage :

```bash
findmnt /sys/fs/cgroup
```

Si nous voyons :

```text
FSTYPE cgroup2
```

nous utilisons cgroups v2.

Nous pouvons aussi tester :

```bash
test -f /sys/fs/cgroup/cgroup.controllers && echo "cgroups v2"
```

---

## 13.8. systemd comme gestionnaire de cgroups

## 13.8.1. systemd organise les processus

Sur beaucoup de distributions modernes, systemd organise les processus en cgroups.

Nous pouvons voir l’arborescence :

```bash
systemd-cgls
```

Exemple conceptuel :

```text
Control group /:
├─user.slice
│ └─user-1000.slice
│   └─session-3.scope
└─system.slice
  ├─ssh.service
  ├─docker.service
  └─nginx.service
```

systemd ne se contente pas de lancer des services. Il les place dans des groupes de contrôle.

---

## 13.8.2. Observer la consommation

Nous pouvons utiliser :

```bash
systemd-cgtop
```

Cela donne une vue dynamique de la consommation des cgroups :

```text
Control Group           Tasks   %CPU   Memory
/                         250    8.0    2.3G
/system.slice              80    2.0    800M
/user.slice               120    5.5    1.2G
```

C’est une vue orientée groupes de processus, souvent plus pertinente qu’un simple `top`.

---

## 13.8.3. Limiter un service systemd

Dans une unité systemd, nous pouvons définir :

```ini
[Service]
MemoryMax=512M
CPUQuota=200%
TasksMax=512
```

Ces paramètres se traduisent en configuration cgroup.

Le service reste un ensemble de processus Linux, mais systemd organise et limite cet ensemble.

---

## 13.8.4. Lien avec les namespaces

systemd peut aussi lancer des services avec certaines formes d’isolation.

Exemples de directives :

```ini
PrivateTmp=true
PrivateNetwork=true
ProtectSystem=strict
ReadWritePaths=/var/lib/app
```

Certaines utilisent des namespaces ou des mécanismes associés.

Nous voyons donc que systemd combine aussi plusieurs briques du noyau.

---

## 13.9. Docker, Podman et les cgroups

## 13.9.1. Options Docker courantes

Docker utilise les cgroups pour appliquer des limites.

Exemples :

```bash
docker run --memory=512m image
docker run --cpus=2 image
docker run --pids-limit=256 image
```

Ces options ne relèvent pas principalement des namespaces.

Elles configurent des cgroups.

---

## 13.9.2. Exemple mémoire

Nous lançons :

```bash
docker run --rm -it --memory=256m alpine sh
```

Dans le conteneur :

```sh
cat /sys/fs/cgroup/memory.max 2>/dev/null
cat /sys/fs/cgroup/memory.current 2>/dev/null
```

Avec cgroups v2, nous pouvons voir :

```text
268435456
```

Cela correspond à 256 Mio.

---

## 13.9.3. Exemple CPU

Nous lançons :

```bash
docker run --rm -it --cpus=2 alpine sh
```

Dans le conteneur :

```sh
cat /sys/fs/cgroup/cpu.max 2>/dev/null
```

Nous pouvons voir une valeur proche de :

```text
200000 100000
```

Cela correspond à environ 2 CPU.

---

## 13.9.4. Exemple pids

Nous lançons :

```bash
docker run --rm -it --pids-limit=128 alpine sh
```

Dans le conteneur :

```sh
cat /sys/fs/cgroup/pids.max 2>/dev/null
cat /sys/fs/cgroup/pids.current 2>/dev/null
```

Nous voyons la limite de processus.

---

## 13.9.5. Podman rootless

Podman rootless utilise aussi les cgroups, mais le fonctionnement dépend de :

- cgroups v2 ;
    
- systemd utilisateur ;
    
- droits de l’utilisateur ;
    
- configuration de la distribution ;
    
- runtime OCI ;
    
- version de Podman.
    

Nous devons donc diagnostiquer avec :

```bash
podman info
cat /proc/self/cgroup
systemd-cgls --user
```

selon le contexte.

---

## 13.10. Kubernetes requests et limits

## 13.10.1. Requests

Dans Kubernetes, une `request` indique la quantité de ressources demandée pour planifier le pod.

Exemple :

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
```

Cela signifie :

```text
Nous demandons au scheduler de prévoir 256 Mio de mémoire.
Nous demandons 0,5 CPU.
```

La request influence la planification.

---

## 13.10.2. Limits

Une `limit` définit une limite maximale.

Exemple :

```yaml
resources:
  limits:
    memory: "512Mi"
    cpu: "1"
```

Cela signifie :

```text
Le conteneur ne doit pas dépasser 512 Mio de mémoire.
Le conteneur peut utiliser jusqu’à 1 CPU.
```

Ces limites sont appliquées via les cgroups sur le nœud.

---

## 13.10.3. Exemple complet

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "1"
```

Interprétation :

```text
Le pod demande 256 Mio et 0,5 CPU pour la planification.
Le pod est limité à 512 Mio et 1 CPU à l’exécution.
```

Nous devons distinguer planification et limite réelle.

---

## 13.10.4. Qualité de service Kubernetes

Kubernetes classe les pods en QoS selon les requests et limits :

```text
Guaranteed
Burstable
BestEffort
```

Résumé :

|Classe|Condition générale|
|---|---|
|`Guaranteed`|requests = limits pour CPU et mémoire sur tous les conteneurs|
|`Burstable`|au moins une request ou limit, mais pas Guaranteed|
|`BestEffort`|aucune request ni limit|

Cette classe influence notamment les décisions d’éviction en cas de pression sur le nœud.

---

## 13.11. OOMKilled

## 13.11.1. Définition

`OOMKilled` signifie que le processus a été tué à cause d’un dépassement mémoire.

OOM signifie :

```text
Out Of Memory
```

Dans Kubernetes, nous pouvons voir :

```bash
kubectl describe pod <pod>
```

avec un état :

```text
Reason: OOMKilled
Exit Code: 137
```

Cela indique généralement que le conteneur a dépassé sa limite mémoire.

---

## 13.11.2. Ce qui se passe côté Linux

Si un processus dépasse la limite `memory.max` de son cgroup, le noyau peut tuer un processus dans ce cgroup.

C’est une action du noyau liée au contrôleur mémoire des cgroups.

Ce n’est pas un namespace qui tue le processus.

Nous retenons :

```text
OOMKilled = limite mémoire cgroup dépassée.
```

---

## 13.11.3. Diagnostiquer un OOMKilled

Nous vérifions :

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl top pod
kubectl get events
```

Sur le nœud, si nous avons accès :

```bash
dmesg | grep -i oom
journalctl -k | grep -i oom
```

Et dans certains cas :

```bash
cat /sys/fs/cgroup/memory.events
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
```

---

## 13.11.4. Solutions possibles

Selon le cas :

- augmenter la limite mémoire ;
    
- réduire la consommation applicative ;
    
- corriger une fuite mémoire ;
    
- ajuster le nombre de workers ;
    
- limiter les caches ;
    
- améliorer les paramètres JVM, Node.js, Python, etc. ;
    
- définir des requests plus réalistes ;
    
- surveiller la mémoire avec Prometheus.
    

Nous ne répondons pas simplement “ajoutons de la mémoire” sans analyser le comportement.

---

## 13.12. Exemple complet : conteneur limité en mémoire

## 13.12.1. Lancer le conteneur

Nous lançons :

```bash
docker run --rm -it --name mem-demo --memory=256m alpine sh
```

Dans le conteneur :

```sh
cat /proc/self/cgroup
cat /sys/fs/cgroup/memory.max 2>/dev/null
cat /sys/fs/cgroup/memory.current 2>/dev/null
```

Nous observons la limite.

---

## 13.12.2. Depuis l’hôte

Dans un autre terminal :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' mem-demo)
sudo cat /proc/$pid/cgroup
sudo readlink /proc/$pid/ns/cgroup
```

Nous pouvons comparer :

```text
vue depuis l’hôte
vue depuis le conteneur
```

Le namespace cgroup peut changer la vue du chemin, mais la limite est effective.

---

## 13.12.3. Nettoyage

Dans le conteneur :

```sh
exit
```

Ou depuis l’hôte :

```bash
docker rm -f mem-demo
```

---

## 13.13. Exemple complet : limite de processus

## 13.13.1. Lancer un conteneur avec limite pids

Nous lançons :

```bash
docker run --rm -it --pids-limit=64 alpine sh
```

Dans le conteneur :

```sh
cat /sys/fs/cgroup/pids.max 2>/dev/null
cat /sys/fs/cgroup/pids.current 2>/dev/null
```

Nous voyons la limite.

---

## 13.13.2. Pourquoi c’est utile ?

Cela protège contre :

- fork bomb ;
    
- bugs créant trop de threads ;
    
- runaway process ;
    
- mauvaise configuration de workers ;
    
- épuisement de la table des processus.
    

Le namespace PID ne suffit pas à cela.

---

## 13.14. Exemple complet : CPU limité

## 13.14.1. Lancer un conteneur avec CPU limité

```bash
docker run --rm -it --cpus=1 alpine sh
```

Dans le conteneur :

```sh
cat /sys/fs/cgroup/cpu.max 2>/dev/null
```

Nous voyons généralement un quota correspondant à 1 CPU.

---

## 13.14.2. Interprétation

Avec cgroups v2 :

```text
100000 100000
```

signifie environ 1 CPU.

```text
50000 100000
```

signifie environ 0,5 CPU.

```text
200000 100000
```

signifie environ 2 CPU.

Nous devons toutefois tenir compte du scheduler, de la charge globale et de la configuration du runtime.

---

## 13.15. Observabilité des cgroups

## 13.15.1. Avec les outils système

Nous pouvons observer les cgroups avec :

```bash
systemd-cgls
systemd-cgtop
cat /proc/self/cgroup
cat /sys/fs/cgroup/*
```

Pour Docker :

```bash
docker stats
```

Pour Kubernetes :

```bash
kubectl top pod
kubectl top node
```

---

## 13.15.2. Avec Prometheus

Dans un cluster Kubernetes, les métriques cgroups sont souvent collectées via :

```text
kubelet
cAdvisor
node-exporter
Prometheus
```

Prometheus permet ensuite d’historiser :

- CPU consommé ;
    
- mémoire consommée ;
    
- redémarrages ;
    
- OOMKilled ;
    
- limites ;
    
- requests ;
    
- saturation nœud.
    

---

## 13.15.3. Pourquoi `/proc` et `/sys` ne suffisent pas en production

`/proc` et `/sys/fs/cgroup` donnent une vue locale et instantanée.

En production, nous avons besoin :

- d’historique ;
    
- de graphes ;
    
- d’alertes ;
    
- d’agrégation multi-nœuds ;
    
- de corrélation avec les logs ;
    
- de lien avec les déploiements ;
    
- de métriques applicatives.
    

Les cgroups fournissent les compteurs bruts. L’observabilité les transforme en information exploitable.

---

## 13.16. Sécurité et isolation

## 13.16.1. Pourquoi les cgroups renforcent la sécurité

Les cgroups limitent l’impact d’un processus qui consomme trop.

Exemples :

- une fuite mémoire ne doit pas tuer tout le nœud ;
    
- une fork bomb doit être limitée ;
    
- un service CPU-bound ne doit pas monopoliser toute la machine ;
    
- un conteneur ne doit pas saturer les I/O disque ;
    
- un pod ne doit pas faire tomber les autres pods.
    

Les cgroups participent donc à la stabilité et à la sécurité opérationnelle.

---

## 13.16.2. Ce que les cgroups ne font pas

Les cgroups ne masquent pas les processus.

Ils ne donnent pas un hostname différent.

Ils ne créent pas de pile réseau séparée.

Ils ne changent pas les UID/GID.

Ils ne remplacent pas les permissions Unix.

Ils ne filtrent pas les appels système.

Ils ne remplacent pas AppArmor, SELinux ou seccomp.

Nous devons donc les combiner avec les namespaces et les mécanismes de sécurité.

---

## 13.16.3. Mauvaise configuration

Un conteneur sans limites peut poser problème.

Exemple Kubernetes :

```yaml
resources: {}
```

Cela peut produire un pod `BestEffort`.

En cas de pression mémoire, ce pod est plus facilement évincé.

À l’inverse, une limite trop basse peut provoquer des `OOMKilled` répétés.

Nous devons donc dimensionner correctement.

---

## 13.17. Méthode de diagnostic : ressource saturée dans un conteneur

## 13.17.1. Étape 1 : identifier le symptôme

Nous commençons par identifier :

```text
CPU élevé
mémoire élevée
OOMKilled
trop de processus
latence I/O
pod évincé
conteneur redémarre
```

---

## 13.17.2. Étape 2 : vérifier les limites

Dans Docker :

```bash
docker inspect <container>
docker stats
```

Dans Kubernetes :

```bash
kubectl describe pod <pod>
kubectl top pod
kubectl get events
```

Dans le conteneur ou sur le nœud :

```bash
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/pids.max
cat /sys/fs/cgroup/pids.current
```

---

## 13.17.3. Étape 3 : distinguer vue et limite

Nous ne confondons pas :

```text
/proc/self/cgroup
```

avec :

```text
/sys/fs/cgroup/memory.max
/sys/fs/cgroup/cpu.max
/sys/fs/cgroup/pids.max
```

Le premier montre une position ou une vue.

Les seconds montrent des limites ou mesures.

---

## 13.17.4. Étape 4 : relier à l’application

Nous analysons ensuite l’application :

- nombre de workers ;
    
- taille des caches ;
    
- fuite mémoire ;
    
- threads ;
    
- batchs trop gros ;
    
- requêtes lentes ;
    
- configuration JVM/Node/Python ;
    
- pool de connexions ;
    
- limites applicatives.
    

Les cgroups nous disent qu’une limite est atteinte. Ils ne nous disent pas toujours pourquoi l’application consomme autant.

---

## 13.18. Travaux pratiques

## TP 1 — Distinguer namespaces et cgroups

Nous exécutons :

```bash
readlink /proc/$$/ns/pid
readlink /proc/$$/ns/net
readlink /proc/$$/ns/mnt
cat /proc/self/cgroup
```

Nous répondons :

1. Quels namespaces voyons-nous ?
    
2. Que nous dit `/proc/self/cgroup` ?
    
3. Est-ce que les namespaces affichent des limites de ressources ?
    
4. Est-ce que `/proc/self/cgroup` isole les processus ?
    

---

## TP 2 — Identifier cgroups v1 ou v2

Nous exécutons :

```bash
findmnt /sys/fs/cgroup
test -f /sys/fs/cgroup/cgroup.controllers && echo "cgroups v2"
cat /proc/self/cgroup
```

Nous répondons :

1. Le système utilise-t-il cgroups v2 ?
    
2. Combien de lignes contient `/proc/self/cgroup` ?
    
3. Que signifie `0::/` en cgroups v2 ?
    

---

## TP 3 — Lire les limites cgroups v2

Si cgroups v2 est disponible :

```bash
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/pids.current
cat /sys/fs/cgroup/pids.max
```

Nous répondons :

1. Quelle est la consommation mémoire actuelle ?
    
2. Quelle est la limite mémoire ?
    
3. Quelle est la limite CPU ?
    
4. Quelle est la limite de processus ?
    

---

## TP 4 — Observer systemd

Nous exécutons :

```bash
systemd-cgls
systemd-cgtop
systemctl status ssh
```

ou un autre service disponible.

Nous répondons :

1. Où apparaît le service dans l’arborescence cgroup ?
    
2. Quelle consommation voyons-nous ?
    
3. Pourquoi systemd utilise-t-il les cgroups ?
    

---

## TP 5 — Docker avec limite mémoire

Si Docker est disponible :

```bash
docker run --rm -d --name mem-demo --memory=256m alpine sleep 1000
docker stats --no-stream mem-demo
docker exec mem-demo sh -c 'cat /sys/fs/cgroup/memory.max 2>/dev/null; cat /sys/fs/cgroup/memory.current 2>/dev/null'
docker rm -f mem-demo
```

Nous répondons :

1. Quelle limite mémoire est appliquée ?
    
2. Où la lisons-nous ?
    
3. Est-ce un namespace ou un cgroup qui applique cette limite ?
    

---

## TP 6 — Docker avec limite pids

Si Docker est disponible :

```bash
docker run --rm -d --name pids-demo --pids-limit=64 alpine sleep 1000
docker exec pids-demo sh -c 'cat /sys/fs/cgroup/pids.max 2>/dev/null; cat /sys/fs/cgroup/pids.current 2>/dev/null'
docker rm -f pids-demo
```

Nous répondons :

1. Quelle limite de processus est appliquée ?
    
2. Pourquoi cette limite est-elle utile ?
    
3. Pourquoi le namespace PID ne suffit-il pas ?
    

---

## TP 7 — Kubernetes requests et limits

Nous étudions :

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "1"
```

Nous répondons :

1. Quelle quantité de mémoire est demandée ?
    
2. Quelle quantité de mémoire est limitée ?
    
3. Quelle quantité de CPU est demandée ?
    
4. Quelle quantité de CPU est limitée ?
    
5. Quel mécanisme Linux applique ces limites ?
    
6. Que signifie `OOMKilled` ?
    

---

## 13.19. Pièges classiques

## 13.19.1. Croire que les namespaces limitent les ressources

Un namespace PID ne limite pas le nombre de processus.

Un namespace mount ne limite pas l’espace disque.

Un namespace network ne limite pas automatiquement le débit.

Un namespace user ne limite pas la mémoire.

Les namespaces isolent des vues.

---

## 13.19.2. Croire que les cgroups isolent les vues

Les cgroups limitent ou mesurent les ressources, mais ne créent pas à eux seuls :

- un hostname séparé ;
    
- une pile réseau séparée ;
    
- une table de processus séparée ;
    
- un root filesystem séparé.
    

Pour cela, nous avons besoin des namespaces.

---

## 13.19.3. Confondre cgroup namespace et cgroups

Le namespace cgroup isole la vue des chemins cgroups.

Les cgroups appliquent les limites.

Nous ne devons pas dire :

```text
Le namespace cgroup limite la mémoire.
```

Nous devons dire :

```text
Le cgroup limite la mémoire.
Le namespace cgroup peut masquer le chemin cgroup vu par le processus.
```

---

## 13.19.4. Interpréter naïvement `/proc/meminfo` dans un conteneur

Dans un conteneur, `/proc/meminfo` peut parfois refléter la mémoire du nœud plutôt que la limite réelle du cgroup, selon les environnements.

Pour connaître la limite, nous regardons plutôt :

```bash
cat /sys/fs/cgroup/memory.max
```

avec cgroups v2.

---

## 13.19.5. Oublier la QoS Kubernetes

Un pod sans requests ni limits peut être plus vulnérable aux évictions.

Un pod avec une limite trop basse peut être `OOMKilled`.

Un pod avec des requests trop hautes peut être difficile à planifier.

Les cgroups ne remplacent pas le dimensionnement applicatif.

---

## 13.20. Ce que nous devons retenir

Nous retenons les points suivants :

1. Les namespaces isolent les vues du système.
    
2. Les cgroups contrôlent et mesurent les ressources.
    
3. Les deux mécanismes sont complémentaires.
    
4. Un conteneur utilise généralement namespaces et cgroups.
    
5. Le namespace PID ne limite pas le nombre de processus.
    
6. Le contrôleur cgroup `pids` limite le nombre de processus ou threads.
    
7. Le namespace mount ne limite pas l’espace disque.
    
8. Les cgroups et quotas contrôlent les ressources.
    
9. Le namespace network n’impose pas seul une politique réseau complète.
    
10. Les cgroups peuvent limiter CPU, mémoire, I/O et pids.
    
11. cgroups v1 utilise souvent plusieurs hiérarchies.
    
12. cgroups v2 utilise une hiérarchie unifiée.
    
13. systemd organise les services en cgroups.
    
14. Docker et Podman utilisent les cgroups pour appliquer les limites de ressources.
    
15. Kubernetes traduit requests et limits en configuration cgroup.
    
16. `OOMKilled` correspond généralement à un dépassement de limite mémoire cgroup.
    
17. Le namespace cgroup isole la vue des chemins cgroups, pas les ressources elles-mêmes.
    
18. En diagnostic, nous devons distinguer vue, limite, consommation et contexte d’exécution.
    

---

## Conclusion du chapitre 13

Nous avons étudié la relation entre namespaces et cgroups.

Nous savons maintenant que ces deux mécanismes sont complémentaires, mais profondément différents. Les namespaces répondent à la question “que voit le processus ?”, tandis que les cgroups répondent à la question “que peut consommer le processus ?”.

Cette distinction est essentielle pour comprendre les conteneurs. Un conteneur a besoin de namespaces pour voir son propre environnement, mais il a aussi besoin de cgroups pour ne pas consommer toutes les ressources de l’hôte.

Nous avons aussi relié ces notions à systemd, Docker, Podman et Kubernetes. Nous savons désormais interpréter une limite mémoire, une limite CPU, une limite de processus, un `OOMKilled`, ainsi que la différence entre une vue cgroup simplifiée et une limite réellement appliquée.

Dans le chapitre suivant, nous étudions les capabilities et la sécurité, afin de comprendre pourquoi les namespaces et les cgroups ne suffisent pas à eux seuls pour construire une isolation robuste.

---
> [!info] Livre « Les namespaces Linux » — chapitre 13/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 12 — Namespaces et proc|← 12 — Namespaces et proc]] · [[Les namespaces Linux — 14 — Namespaces, capabilities et sécurité|14 — Namespaces, capabilities et sécurité →]]
