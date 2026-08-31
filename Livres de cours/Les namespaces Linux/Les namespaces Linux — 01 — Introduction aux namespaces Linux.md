---
schema_version: 1
uid: 01M1BQ6247K5KNFV9XY9TGQXJ3
titre: "Les namespaces Linux — 01 — Introduction aux namespaces Linux"
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
resume: "Chapitre 1 sur 16 du livre « Les namespaces Linux » : Introduction aux namespaces Linux. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 1/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — Sommaire|← Sommaire]] · [[Les namespaces Linux — 02 — Architecture générale des namespaces|02 — Architecture générale des namespaces →]]

# Chapitre 1 — Introduction aux namespaces Linux
## Objectifs du chapitre

Dans ce premier chapitre, nous posons les bases du cours sur les namespaces Linux.

Nous cherchons à comprendre pourquoi Linux a introduit des mécanismes d’isolation, en quoi ces mécanismes diffèrent de la virtualisation classique, et pourquoi ils sont devenus essentiels avec les conteneurs.

À la fin de ce chapitre, nous savons :

- expliquer ce qu’est un namespace Linux ;
    
- comprendre pourquoi nous isolons des processus ;
    
- distinguer isolation par namespaces et virtualisation par machine virtuelle ;
    
- expliquer le lien entre namespaces et conteneurs ;
    
- comprendre pourquoi un conteneur n’est pas une machine virtuelle ;
    
- identifier les principaux types de namespaces Linux.
    

---

## 1.1. Pourquoi isoler des processus ?

## 1.1.1. Le problème de départ

Sur un système Linux classique, tous les processus partagent le même noyau.

Ils ne voient pas nécessairement les mêmes droits, mais ils vivent dans le même environnement global :

- même table globale des processus ;
    
- même pile réseau ;
    
- même hostname ;
    
- même arborescence de montages ;
    
- mêmes mécanismes IPC ;
    
- même noyau ;
    
- mêmes ressources matérielles sous-jacentes.
    

Cela pose une question importante :

```text
Comment faire croire à un groupe de processus qu’il vit dans son propre système, sans lancer une machine virtuelle complète ?
```

C’est précisément l’un des rôles des namespaces.

---

## 1.1.2. Exemple simple : la vue des processus

Sur une machine Linux classique, nous pouvons afficher les processus :

```bash
ps aux
```

Nous voyons alors de nombreux processus du système.

Mais dans un conteneur Docker, si nous exécutons :

```bash
ps aux
```

nous pouvons ne voir qu’un ou quelques processus.

Pourtant, le conteneur tourne sur le même noyau que l’hôte.

Cela signifie que le noyau présente une vue différente selon le contexte du processus.

Cette idée est fondamentale :

```text
Un namespace ne crée pas forcément une nouvelle ressource.
Il crée une vue isolée d’une ressource existante.
```

---

## 1.1.3. Isoler pour organiser

Nous isolons des processus pour plusieurs raisons :

- éviter qu’un processus voie tous les autres processus ;
    
- donner un hostname différent à un environnement ;
    
- séparer les piles réseau ;
    
- donner une arborescence de fichiers différente ;
    
- limiter les interactions entre applications ;
    
- construire des environnements reproductibles ;
    
- renforcer la sécurité ;
    
- faire fonctionner plusieurs services avec des dépendances différentes ;
    
- exécuter des applications dans des conteneurs.
    

Les namespaces sont donc à la fois un outil d’isolation, d’organisation et de construction d’environnements applicatifs.

---

## 1.2. Qu’est-ce qu’un namespace Linux ?

## 1.2.1. Définition générale

Un namespace Linux est un mécanisme du noyau qui donne à un processus et à ses descendants une vue isolée d’une ressource système.

Autrement dit, deux processus peuvent utiliser le même noyau, mais ne pas voir la même chose.

Par exemple :

- dans un namespace PID, un processus peut voir son propre arbre de processus ;
    
- dans un namespace réseau, un processus peut voir ses propres interfaces réseau ;
    
- dans un namespace UTS, un processus peut voir un hostname différent ;
    
- dans un namespace mount, un processus peut voir une arborescence de montages différente.
    

Nous pouvons résumer ainsi :

```text
Les namespaces isolent ce que les processus voient.
```

---

## 1.2.2. Un processus appartient à plusieurs namespaces

Un processus Linux n’appartient pas à un seul namespace, mais à plusieurs namespaces en même temps.

Nous pouvons observer cela avec :

```bash
ls -l /proc/$$/ns
```

Exemple de sortie :

```text
cgroup -> cgroup:[4026531835]
ipc    -> ipc:[4026531839]
mnt    -> mnt:[4026531841]
net    -> net:[4026531840]
pid    -> pid:[4026531836]
time   -> time:[4026531834]
user   -> user:[4026531837]
uts    -> uts:[4026531838]
```

Chaque ligne représente un type de namespace auquel appartient notre shell courant.

Le nombre entre crochets identifie une instance de namespace.

Deux processus qui ont le même identifiant pour un type donné partagent ce namespace.

---

## 1.2.3. Les namespaces ne sont pas des machines virtuelles

Un namespace ne lance pas un nouveau noyau.

Il ne virtualise pas tout le matériel.

Il ne crée pas une machine complète.

Il isole certaines vues du système pour certains processus.

C’est une différence majeure avec les machines virtuelles.

Dans une machine virtuelle, nous avons généralement :

```text
application
système invité
noyau invité
matériel virtuel
hyperviseur
noyau hôte
matériel réel
```

Avec des namespaces, nous avons plutôt :

```text
application isolée
namespaces
noyau Linux commun
matériel réel
```

Les conteneurs sont donc plus légers que les machines virtuelles, mais ils partagent le noyau avec l’hôte.

---

## 1.3. Virtualisation classique et isolation par namespaces

## 1.3.1. Machine virtuelle

Une machine virtuelle exécute un système d’exploitation invité complet.

Par exemple, sur un hôte Linux, nous pouvons lancer :

- une VM Ubuntu ;
    
- une VM Debian ;
    
- une VM Windows ;
    
- une VM FreeBSD.
    

Chaque VM possède son propre noyau invité.

Cela apporte une isolation forte, mais avec un coût plus important :

- consommation mémoire plus élevée ;
    
- démarrage plus lent ;
    
- image système plus lourde ;
    
- couche d’hypervision ;
    
- administration plus proche d’une machine complète.
    

---

## 1.3.2. Conteneur avec namespaces

Un conteneur Linux n’exécute pas son propre noyau.

Il utilise le noyau de l’hôte, mais avec des vues isolées.

Par exemple, un conteneur peut avoir :

- son propre PID 1 ;
    
- son propre hostname ;
    
- ses propres interfaces réseau ;
    
- son propre système de fichiers apparent ;
    
- ses propres montages ;
    
- ses propres utilisateurs mappés ;
    
- ses propres limites de ressources via les cgroups.
    

Mais il partage toujours le noyau avec l’hôte.

Nous pouvons donc dire :

```text
La VM virtualise une machine.
Le conteneur isole des processus.
```

---

## 1.3.3. Comparaison synthétique

|Point comparé|Machine virtuelle|Conteneur avec namespaces|
|---|---|---|
|Noyau|noyau invité séparé|noyau hôte partagé|
|Démarrage|plus lent|plus rapide|
|Isolation|généralement plus forte|dépend de la configuration|
|Poids|plus lourd|plus léger|
|Usage|systèmes complets|applications et services|
|Sécurité|frontière plus nette|noyau partagé, vigilance nécessaire|
|Exemples|KVM, VMware, VirtualBox|Docker, Podman, LXC, Kubernetes|

Nous ne devons pas opposer naïvement les deux approches.

Elles répondent à des besoins différents.

---

## 1.4. Lien entre namespaces et conteneurs

## 1.4.1. Un conteneur n’est pas une magie Docker

Docker, Podman ou Kubernetes ne créent pas l’isolation à partir de rien.

Ils orchestrent plusieurs mécanismes du noyau Linux, notamment :

- namespaces ;
    
- cgroups ;
    
- capabilities ;
    
- seccomp ;
    
- AppArmor ou SELinux ;
    
- systèmes de fichiers en couches ;
    
- bridges réseau ;
    
- règles de pare-feu ;
    
- runtime OCI comme `runc`.
    

Les namespaces constituent l’un des fondements techniques des conteneurs.

---

## 1.4.2. Ce que Docker automatise

Quand nous lançons :

```bash
docker run --rm -it alpine sh
```

Docker prépare beaucoup de choses pour nous.

Il peut notamment :

- créer un namespace PID ;
    
- créer un namespace mount ;
    
- créer un namespace UTS ;
    
- créer un namespace network ;
    
- créer un namespace IPC ;
    
- configurer des cgroups ;
    
- monter un root filesystem ;
    
- configurer `/proc` ;
    
- créer une interface réseau virtuelle ;
    
- appliquer des capabilities ;
    
- lancer le processus initial.
    

Nous pourrions faire une partie de cela à la main avec `unshare`, `mount`, `ip`, `chroot` ou `pivot_root`, mais Docker automatise le tout.

---

## 1.4.3. Pourquoi étudier les namespaces si Docker existe ?

Nous étudions les namespaces parce qu’un bon ingénieur ne doit pas utiliser les conteneurs comme une boîte noire.

Comprendre les namespaces permet de répondre à des questions importantes :

- pourquoi le PID 1 d’un conteneur est-il spécial ?
    
- pourquoi deux conteneurs ne voient-ils pas les mêmes interfaces réseau ?
    
- pourquoi un conteneur peut-il voir un hostname différent ?
    
- pourquoi `/proc` dans un conteneur ne montre pas tous les processus de l’hôte ?
    
- pourquoi `hostNetwork`, `hostPID` ou `privileged` changent fortement l’isolation ?
    
- pourquoi un conteneur rootless peut fonctionner ?
    
- pourquoi les cgroups ne sont pas la même chose que les namespaces ?
    

---

## 1.5. Limites : un conteneur n’est pas une machine virtuelle

## 1.5.1. Le noyau est partagé

Le point central est le suivant :

```text
Les conteneurs Linux partagent le noyau de l’hôte.
```

Cela signifie que si le noyau est vulnérable, l’isolation peut être affaiblie.

Cela signifie aussi qu’un conteneur Linux ne peut pas exécuter directement un noyau différent de celui de l’hôte.

Par exemple, un conteneur Ubuntu sur un hôte Debian utilise toujours le noyau de l’hôte Debian.

Il peut avoir des bibliothèques Ubuntu dans son root filesystem, mais pas un noyau Ubuntu séparé.

---

## 1.5.2. Isolation forte ou isolation légère ?

Les namespaces fournissent une isolation très utile, mais ce n’est pas la même barrière qu’une machine virtuelle.

Un conteneur mal configuré peut être dangereux.

Exemples de configurations sensibles :

```bash
docker run --privileged ...
docker run -v /:/host ...
docker run -v /proc:/host/proc ...
docker run --pid=host ...
docker run --network=host ...
```

Ces options réduisent fortement l’isolation.

Nous devons donc éviter l’idée :

```text
C’est dans un conteneur, donc c’est forcément isolé.
```

La vraie question est :

```text
Quels namespaces sont isolés ?
Quelles capabilities sont accordées ?
Quels montages sont exposés ?
Quels cgroups limitent les ressources ?
Quelles politiques de sécurité sont actives ?
```

---

## 1.5.3. Les namespaces isolent la vue, pas toujours la consommation

Les namespaces répondent surtout à la question :

```text
Que voit le processus ?
```

Mais ils ne répondent pas entièrement à la question :

```text
Combien de ressources peut-il consommer ?
```

Pour limiter les ressources, nous avons besoin des cgroups.

Exemple :

- le namespace PID peut limiter la vue des processus ;
    
- mais le cgroup `pids` peut limiter le nombre de processus créables ;
    
- le namespace network isole la pile réseau ;
    
- mais les limites de débit ou politiques réseau demandent d’autres mécanismes ;
    
- le namespace mount isole les montages ;
    
- mais les quotas disque demandent encore d’autres mécanismes.
    

Nous retenons :

```text
Namespaces = isolation des vues.
Cgroups = limitation et comptabilité des ressources.
```

---

## 1.6. Vue d’ensemble des principaux namespaces

## 1.6.1. Namespace UTS

Le namespace UTS isole :

- le hostname ;
    
- le domainname.
    

Il permet à un conteneur d’avoir son propre nom de machine.

Exemple :

```bash
unshare --uts bash
hostname test-namespace
hostname
```

Dans ce shell isolé, le hostname peut être différent de celui de l’hôte.

---

## 1.6.2. Namespace PID

Le namespace PID isole la numérotation des processus.

Dans un namespace PID, un processus peut voir son propre PID 1.

Exemple :

```bash
unshare --fork --pid --mount-proc bash
ps aux
cat /proc/1/comm
```

Ce namespace est fondamental pour les conteneurs.

Il permet qu’un processus soit PID 1 dans le conteneur, même s’il a un autre PID sur l’hôte.

---

## 1.6.3. Namespace mount

Le namespace mount isole les points de montage.

Il permet à un processus de voir une arborescence de fichiers différente.

Exemple :

```bash
unshare --mount bash
mount -t tmpfs tmpfs /tmp/test-mnt
```

Le montage peut exister dans le namespace courant sans apparaître dans l’environnement parent, selon la propagation des montages.

Ce namespace est essentiel pour donner à un conteneur son propre root filesystem.

---

## 1.6.4. Namespace network

Le namespace network isole la pile réseau.

Il fournit une vue séparée de :

- interfaces réseau ;
    
- adresses IP ;
    
- routes ;
    
- règles de firewall ;
    
- sockets ;
    
- ports en écoute ;
    
- table ARP ;
    
- loopback.
    

Exemple :

```bash
unshare --net bash
ip addr
```

Dans un nouveau namespace réseau, nous voyons généralement seulement l’interface loopback, souvent désactivée au départ.

---

## 1.6.5. Namespace IPC

Le namespace IPC isole certains mécanismes de communication inter-processus.

Il concerne notamment :

- files de messages System V ;
    
- sémaphores ;
    
- mémoire partagée System V ;
    
- certaines ressources IPC POSIX.
    

Exemple :

```bash
ipcs
unshare --ipc bash
ipcs
```

Nous pouvons voir une vue différente des ressources IPC.

---

## 1.6.6. Namespace user

Le namespace user isole les identifiants utilisateurs et groupes.

Il permet de mapper des UID/GID différents entre l’intérieur du namespace et l’hôte.

C’est ce qui permet, par exemple, d’être `root` dans un namespace sans être réellement `root` sur l’hôte.

Exemple :

```bash
unshare --user --map-root-user bash
id
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Nous pouvons obtenir :

```text
uid=0(root) gid=0(root)
```

dans le namespace, alors que notre utilisateur réel sur l’hôte reste non privilégié.

Ce mécanisme est central pour les conteneurs rootless.

---

## 1.6.7. Namespace cgroup

Le namespace cgroup isole la vue des cgroups.

Il ne crée pas les limites de ressources lui-même.

Il permet plutôt à un processus de voir une racine cgroup différente.

Nous devons bien distinguer :

```text
cgroups : limitent et mesurent les ressources.
cgroup namespace : isole la vue des cgroups.
```

---

## 1.6.8. Namespace time

Le namespace time permet d’isoler certains offsets d’horloges, notamment pour :

- `CLOCK_MONOTONIC` ;
    
- `CLOCK_BOOTTIME`.
    

Il ne permet pas simplement de changer l’heure réelle globale du système pour un conteneur.

Il sert surtout à des cas plus spécialisés :

- tests ;
    
- checkpoint/restore ;
    
- environnements isolés avancés.
    

Nous pouvons observer :

```bash
ls -l /proc/$$/ns/time
cat /proc/self/timens_offsets
```

---

## 1.7. Première observation avec `/proc`

## 1.7.1. Observer les namespaces du shell courant

Nous pouvons commencer sans créer de namespace.

Nous observons les namespaces de notre shell :

```bash
ls -l /proc/$$/ns
```

Sortie possible :

```text
cgroup -> cgroup:[4026531835]
ipc    -> ipc:[4026531839]
mnt    -> mnt:[4026531841]
net    -> net:[4026531840]
pid    -> pid:[4026531836]
time   -> time:[4026531834]
user   -> user:[4026531837]
uts    -> uts:[4026531838]
```

Chaque lien symbolique représente un namespace.

---

## 1.7.2. Comparer avec le PID 1

Nous pouvons comparer avec le processus 1 :

```bash
ls -l /proc/1/ns
```

Si notre shell est sur l’hôte, il partage probablement plusieurs namespaces avec le PID 1.

Nous pouvons comparer précisément :

```bash
readlink /proc/$$/ns/pid
readlink /proc/1/ns/pid
```

Si les deux valeurs sont identiques, les deux processus sont dans le même namespace PID.

Si elles sont différentes, ils sont dans deux namespaces PID distincts.

---

## 1.7.3. Observer avec `lsns`

La commande `lsns` liste les namespaces du système :

```bash
lsns
```

Nous pouvons filtrer :

```bash
lsns -t pid
lsns -t net
lsns -t mnt
```

`lsns` permet de voir :

- le type de namespace ;
    
- son identifiant ;
    
- le nombre de processus ;
    
- un PID associé ;
    
- la commande associée.
    

C’est un outil très utile pour comprendre l’état des namespaces sur une machine.

---

## 1.8. Exemple intuitif : isoler le hostname

## 1.8.1. Situation initiale

Nous regardons le hostname de la machine :

```bash
hostname
```

Exemple :

```text
machine-hote
```

Si nous modifions le hostname sans namespace, nous modifions le hostname global du système.

Mais avec un namespace UTS, nous pouvons changer le hostname seulement dans une vue isolée.

---

## 1.8.2. Créer un namespace UTS

Nous lançons :

```bash
unshare --uts bash
```

Nous sommes maintenant dans un shell qui possède un namespace UTS distinct.

Nous modifions le hostname :

```bash
hostname namespace-test
hostname
```

Nous voyons :

```text
namespace-test
```

Nous quittons :

```bash
exit
```

Puis nous relisons :

```bash
hostname
```

Nous retrouvons le hostname de l’hôte.

Cette démonstration montre le principe des namespaces : une ressource semble différente depuis l’intérieur du namespace, sans changer la vue extérieure.

---

## 1.9. Les namespaces comme fondation des conteneurs

## 1.9.1. Conteneur minimal conceptuel

Un conteneur combine plusieurs isolations.

Un conteneur typique peut utiliser :

```text
UTS namespace   → hostname isolé
PID namespace   → arbre de processus isolé
mount namespace → système de fichiers isolé
net namespace   → réseau isolé
IPC namespace   → IPC isolé
user namespace  → utilisateurs mappés
cgroups         → ressources limitées
capabilities    → privilèges réduits
seccomp         → appels système filtrés
```

Docker, Podman ou Kubernetes automatisent cette combinaison.

---

## 1.9.2. Ce que nous allons faire dans le cours

Dans ce cours, nous n’allons pas seulement utiliser Docker.

Nous allons comprendre les briques de base.

Nous allons manipuler directement :

```bash
unshare
nsenter
lsns
ip netns
mount
ps
ip
```

Puis nous ferons le lien avec :

- Docker ;
    
- Kubernetes ;
    
- `/proc` ;
    
- cgroups ;
    
- systemd ;
    
- sécurité.
    

L’objectif est de pouvoir diagnostiquer des situations réelles, par exemple :

- un conteneur ne voit pas le bon réseau ;
    
- un processus est PID 1 et ne gère pas les signaux ;
    
- `/proc` ne montre pas les bons processus ;
    
- un conteneur rootless a des permissions inattendues ;
    
- un pod Kubernetes partage le réseau entre plusieurs containers ;
    
- un montage apparaît dans l’hôte alors qu’il devait rester isolé.
    

---

## 1.10. Vocabulaire essentiel

## 1.10.1. Namespace

Un namespace est une vue isolée d’une ressource noyau.

Exemple :

```text
namespace PID → vue isolée des processus
namespace net → vue isolée du réseau
```

---

## 1.10.2. Processus

Un processus est une instance d’un programme en cours d’exécution.

Un processus appartient à plusieurs namespaces.

---

## 1.10.3. Namespace parent et enfant

Certains namespaces peuvent être hiérarchiques, notamment le namespace PID et le namespace user.

Un processus dans un namespace parent peut parfois voir des processus du namespace enfant, mais l’inverse n’est pas vrai.

---

## 1.10.4. `unshare`

`unshare` permet de lancer un programme dans un ou plusieurs nouveaux namespaces.

Exemple :

```bash
unshare --uts bash
```

---

## 1.10.5. `nsenter`

`nsenter` permet d’entrer dans les namespaces d’un processus existant.

Exemple :

```bash
nsenter --target <PID> --net bash
```

---

## 1.10.6. `lsns`

`lsns` permet de lister les namespaces présents sur le système.

Exemple :

```bash
lsns -t net
```

---

## 1.11. Points d’attention dès le début

## 1.11.1. Nous devons savoir où nous sommes

Quand nous observons un système, nous devons toujours nous demander :

```text
Sommes-nous sur l’hôte ?
Dans un conteneur ?
Dans un namespace particulier ?
Dans un chroot ?
Dans un pod Kubernetes ?
```

La réponse change l’interprétation des commandes.

---

## 1.11.2. `/proc` dépend des namespaces

Le contenu de `/proc` dépend du contexte.

Exemples :

- `/proc` dans un namespace PID ne montre pas forcément tous les processus de l’hôte ;
    
- `/proc/net` dépend du namespace réseau ;
    
- `/proc/mounts` dépend du namespace de montage ;
    
- `/proc/<PID>/uid_map` dépend du namespace user.
    

Nous faisons donc un lien direct avec le cours sur `/proc`.

---

## 1.11.3. Isolation ne veut pas dire sécurité absolue

Les namespaces contribuent à la sécurité, mais ils ne suffisent pas seuls.

Nous devons aussi considérer :

- cgroups ;
    
- capabilities ;
    
- seccomp ;
    
- AppArmor ;
    
- SELinux ;
    
- droits Unix ;
    
- montages ;
    
- noyau partagé ;
    
- mises à jour de sécurité.
    

Un conteneur mal configuré peut être beaucoup moins isolé qu’on ne l’imagine.

---

## 1.12. Exercices

## Exercice 1 — Observer les namespaces du shell courant

Nous exécutons :

```bash
ls -l /proc/$$/ns
```

Nous répondons :

1. Quels types de namespaces voyons-nous ?
    
2. Quel est l’identifiant du namespace PID ?
    
3. Quel est l’identifiant du namespace réseau ?
    
4. Ces identifiants sont-ils les mêmes que ceux de `/proc/1/ns` ?
    

---

## Exercice 2 — Comparer deux processus

Nous exécutons :

```bash
readlink /proc/$$/ns/pid
readlink /proc/1/ns/pid
readlink /proc/$$/ns/net
readlink /proc/1/ns/net
```

Nous répondons :

1. Le shell courant partage-t-il le namespace PID avec le PID 1 ?
    
2. Partage-t-il le namespace réseau ?
    
3. Que signifie une valeur identique ?
    
4. Que signifie une valeur différente ?
    

---

## Exercice 3 — Utiliser `lsns`

Nous exécutons :

```bash
lsns
lsns -t pid
lsns -t net
lsns -t mnt
```

Nous répondons :

1. Combien de namespaces PID voyons-nous ?
    
2. Combien de namespaces réseau voyons-nous ?
    
3. Quels processus semblent associés à ces namespaces ?
    
4. Voyons-nous des namespaces liés à des conteneurs ?
    

---

## Exercice 4 — Créer un namespace UTS

Nous exécutons :

```bash
hostname
unshare --uts bash
hostname namespace-test
hostname
exit
hostname
```

Nous répondons :

1. Le hostname change-t-il dans le shell isolé ?
    
2. Le hostname de l’hôte change-t-il après `exit` ?
    
3. Quel namespace avons-nous isolé ?
    
4. Pourquoi cet exemple illustre-t-il bien le concept de namespace ?
    

---

## Exercice 5 — Réfléchir aux conteneurs

Nous lançons un conteneur si Docker ou Podman est disponible :

```bash
docker run --rm -it alpine sh
```

Dans le conteneur :

```sh
hostname
ps
ip addr
ls -l /proc/1/ns
```

Sur l’hôte, nous comparons avec :

```bash
hostname
ps aux
ip addr
lsns
```

Nous répondons :

1. Le conteneur voit-il les mêmes processus que l’hôte ?
    
2. Voit-il les mêmes interfaces réseau ?
    
3. A-t-il le même hostname ?
    
4. Quels namespaces semblent isolés ?
    

---

## 1.13. Ce que nous devons retenir

Nous retenons les points suivants :

1. Les namespaces sont des mécanismes du noyau Linux.
    
2. Ils isolent la vue qu’un processus a de certaines ressources.
    
3. Ils ne créent pas une machine virtuelle complète.
    
4. Un processus appartient à plusieurs namespaces en même temps.
    
5. Les conteneurs reposent largement sur les namespaces.
    
6. Les namespaces isolent les vues, tandis que les cgroups limitent les ressources.
    
7. Un conteneur partage le noyau avec l’hôte.
    
8. L’isolation dépend de la configuration réelle.
    
9. `/proc/<PID>/ns` permet d’observer les namespaces d’un processus.
    
10. `unshare`, `nsenter` et `lsns` sont les outils fondamentaux pour manipuler et observer les namespaces.
    
11. Comprendre les namespaces est indispensable pour comprendre Docker, Podman, Kubernetes et les conteneurs rootless.
    

---

## Conclusion du chapitre 1

Nous avons introduit les namespaces Linux comme un mécanisme fondamental d’isolation.

Nous avons vu que les namespaces permettent à des processus de voir des réalités différentes tout en partageant le même noyau. Cette idée est au cœur des conteneurs modernes.

Nous avons aussi posé une distinction essentielle : les namespaces ne sont pas des machines virtuelles. Ils isolent certaines vues du système, mais ils ne remplacent ni les cgroups, ni les mécanismes de sécurité, ni une isolation complète par hyperviseur.

Dans le chapitre suivant, nous étudions l’architecture générale des namespaces : comment un processus appartient à plusieurs namespaces, comment nous les observons avec `/proc/<PID>/ns`, comment ils sont créés avec `clone` ou `unshare`, et comment nous pouvons entrer dans un namespace existant avec `setns` ou `nsenter`.

---

---
> [!info] Livre « Les namespaces Linux » — chapitre 1/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — Sommaire|← Sommaire]] · [[Les namespaces Linux — 02 — Architecture générale des namespaces|02 — Architecture générale des namespaces →]]
