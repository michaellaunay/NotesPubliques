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
resume: 'Cours approfondi et progressif sur les namespaces Linux : modèle du noyau, API clone/clone3/unshare/setns, UTS/PID/mount/network/IPC/user/cgroup/time, /proc, rootless, idmapped mounts, sécurité, conteneurs et diagnostic.'
niveau: avance
prerequis:
- '[[GNULinux]]'
- '[[proc]]'
auteurs:
- Michaël Launay
langue: fr
date_creation: 2026-05-24
date_modification: '2026-08-31'
confidentialite: publique
publication:
- notes-publiques
rag: true
metadata_verifiees: true
---

> [!tip] Version longue
> Ce cours existe aussi sous forme de livre complet : [[Les namespaces Linux — livre complet]].

# Les namespaces Linux

## Pourquoi ce cours est important

Les namespaces sont l'une des briques qui permettent à Linux de faire cohabiter des environnements qui paraissent séparés alors qu'ils partagent le même noyau. Nous les rencontrons derrière Docker, Podman, containerd, systemd et Kubernetes, mais ils restent souvent cachés derrière des outils de plus haut niveau.

Le but de ce cours est de faire l'inverse : partir du noyau et reconstruire progressivement le modèle mental d'un conteneur. Nous allons observer les objets de namespace dans `/proc`, créer des espaces de noms avec `unshare`, rejoindre ceux d'un autre processus avec `nsenter`, construire un réseau avec des paires `veth`, comprendre les mappings UID/GID des conteneurs rootless, puis replacer ces mécanismes dans un modèle de sécurité complet.

Un point doit rester présent pendant tout le cours : **un namespace n'est pas une machine virtuelle et n'est pas, à lui seul, une sandbox complète**. Il virtualise une vue d'une ressource du noyau. La sécurité d'un conteneur repose sur la combinaison de plusieurs mécanismes.

## Objectifs

À la fin du cours, nous saurons notamment :

- expliquer précisément la fonction des namespaces UTS, PID, mount, network, IPC, user, cgroup et time ;
- lire `/proc/<PID>/ns`, `uid_map`, `gid_map`, `timens_offsets` et `/proc/<PID>/cgroup` ;
- utiliser `lsns`, `unshare`, `nsenter`, `ip netns`, `findmnt` et les outils de capabilities ;
- comprendre les différences entre `clone()`, `clone3()`, `unshare()` et `setns()` ;
- raisonner sur la hiérarchie des PID namespaces et le rôle particulier du PID 1 ;
- comprendre la propagation des montages et les idmapped mounts ;
- construire un réseau isolé avec `veth`, routage et bridge ;
- expliquer les user namespaces, les subordinate IDs et le rootless ;
- distinguer l'isolation par namespace du contrôle de ressources par cgroup v2 ;
- analyser les limites de sécurité des conteneurs et diagnostiquer un environnement existant.

## Repères de version

Ce cours est vérifié au **31 août 2026**. Le noyau stable amont est Linux **7.2.2**, publié le 28 août 2026, tandis que **6.18.x** est une branche LTS récente. Ces numéros sont des repères et non des prérequis : l'essentiel des API étudiées ici existe depuis bien plus longtemps et leur disponibilité concrète dépend du noyau et de la configuration de la distribution.

Les huit familles de namespaces exposées par l'API utilisateurs sont toujours UTS, PID, mount, network, IPC, user, cgroup et time.

---

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

# 1.1. Pourquoi isoler des processus ?

**Le problème de départ.**

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

**Exemple simple : la vue des processus.**

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

**Isoler pour organiser.**

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

# 1.2. Qu’est-ce qu’un namespace Linux ?

**Définition générale.**

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

**Un processus appartient à plusieurs namespaces.**

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

**Les namespaces ne sont pas des machines virtuelles.**

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

# 1.3. Virtualisation classique et isolation par namespaces

**Machine virtuelle.**

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

**Conteneur avec namespaces.**

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

**Comparaison synthétique.**

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

# 1.4. Lien entre namespaces et conteneurs

**Un conteneur n’est pas une magie Docker.**

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

**Ce que Docker automatise.**

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

**Pourquoi étudier les namespaces si Docker existe ?.**

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

# 1.5. Limites : un conteneur n’est pas une machine virtuelle

**Le noyau est partagé.**

Le point central est le suivant :

```text
Les conteneurs Linux partagent le noyau de l’hôte.
```

Cela signifie que si le noyau est vulnérable, l’isolation peut être affaiblie.

Cela signifie aussi qu’un conteneur Linux ne peut pas exécuter directement un noyau différent de celui de l’hôte.

Par exemple, un conteneur Ubuntu sur un hôte Debian utilise toujours le noyau de l’hôte Debian.

Il peut avoir des bibliothèques Ubuntu dans son root filesystem, mais pas un noyau Ubuntu séparé.

---

**Isolation forte ou isolation légère ?.**

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

**Les namespaces isolent la vue, pas toujours la consommation.**

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

# 1.6. Vue d’ensemble des principaux namespaces

**Namespace UTS.**

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

**Namespace PID.**

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

**Namespace mount.**

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

**Namespace network.**

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

**Namespace IPC.**

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

**Namespace user.**

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

**Namespace cgroup.**

Le namespace cgroup isole la vue des cgroups.

Il ne crée pas les limites de ressources lui-même.

Il permet plutôt à un processus de voir une racine cgroup différente.

Nous devons bien distinguer :

```text
cgroups : limitent et mesurent les ressources.
cgroup namespace : isole la vue des cgroups.
```

---

**Namespace time.**

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

# 1.7. Première observation avec `/proc`

**Observer les namespaces du shell courant.**

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

**Comparer avec le PID 1.**

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

**Observer avec `lsns`.**

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

# 1.8. Exemple intuitif : isoler le hostname

**Situation initiale.**

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

**Créer un namespace UTS.**

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

# 1.9. Les namespaces comme fondation des conteneurs

**Conteneur minimal conceptuel.**

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

**Ce que nous allons faire dans le cours.**

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

# 1.10. Vocabulaire essentiel

**Namespace.**

Un namespace est une vue isolée d’une ressource noyau.

Exemple :

```text
namespace PID → vue isolée des processus
namespace net → vue isolée du réseau
```

---

**Processus.**

Un processus est une instance d’un programme en cours d’exécution.

Un processus appartient à plusieurs namespaces.

---

**Namespace parent et enfant.**

Certains namespaces peuvent être hiérarchiques, notamment le namespace PID et le namespace user.

Un processus dans un namespace parent peut parfois voir des processus du namespace enfant, mais l’inverse n’est pas vrai.

---

**`unshare`.**

`unshare` permet de lancer un programme dans un ou plusieurs nouveaux namespaces.

Exemple :

```bash
unshare --uts bash
```

---

**`nsenter`.**

`nsenter` permet d’entrer dans les namespaces d’un processus existant.

Exemple :

```bash
nsenter --target <PID> --net bash
```

---

**`lsns`.**

`lsns` permet de lister les namespaces présents sur le système.

Exemple :

```bash
lsns -t net
```

---

# 1.11. Points d’attention dès le début

**Nous devons savoir où nous sommes.**

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

**`/proc` dépend des namespaces.**

Le contenu de `/proc` dépend du contexte.

Exemples :

- `/proc` dans un namespace PID ne montre pas forcément tous les processus de l’hôte ;

- `/proc/net` dépend du namespace réseau ;

- `/proc/mounts` dépend du namespace de montage ;

- `/proc/<PID>/uid_map` dépend du namespace user.


Nous faisons donc un lien direct avec le cours sur `/proc`.

---

**Isolation ne veut pas dire sécurité absolue.**

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

# 1.13. Ce que nous devons retenir

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

# Conclusion du chapitre 1

Nous avons introduit les namespaces Linux comme un mécanisme fondamental d’isolation.

Nous avons vu que les namespaces permettent à des processus de voir des réalités différentes tout en partageant le même noyau. Cette idée est au cœur des conteneurs modernes.

Nous avons aussi posé une distinction essentielle : les namespaces ne sont pas des machines virtuelles. Ils isolent certaines vues du système, mais ils ne remplacent ni les cgroups, ni les mécanismes de sécurité, ni une isolation complète par hyperviseur.

Dans le chapitre suivant, nous étudions l’architecture générale des namespaces : comment un processus appartient à plusieurs namespaces, comment nous les observons avec `/proc/<PID>/ns`, comment ils sont créés avec `clone` ou `unshare`, et comment nous pouvons entrer dans un namespace existant avec `setns` ou `nsenter`.

---

---

# Chapitre 2 — Architecture générale des namespaces

## Objectifs du chapitre

Dans ce chapitre, nous étudions la structure interne des namespaces Linux.

Dans le chapitre précédent, nous avons défini un namespace comme une vue isolée d’une ressource noyau. Nous allons maintenant comprendre comment ces vues sont attachées aux processus, comment les observer, comment les créer, comment y entrer, et comment elles vivent ou disparaissent.

À la fin de ce chapitre, nous savons :

- expliquer qu’un processus appartient simultanément à plusieurs namespaces ;

- observer les namespaces d’un processus via `/proc/<PID>/ns` ;

- comparer les namespaces de deux processus ;

- comprendre le rôle des appels système `clone`, `unshare` et `setns` ;

- utiliser les commandes `unshare`, `nsenter` et `lsns` ;

- comprendre la durée de vie d’un namespace ;

- expliquer comment un namespace peut rester vivant même sans processus actif apparent ;

- faire le lien entre namespaces, processus, `/proc` et conteneurs.


---

# 2.1. Un processus appartient à plusieurs namespaces

**Une appartenance multiple.**

Un processus Linux n’est pas “dans un namespace” au singulier.

Il appartient simultanément à plusieurs namespaces, chacun correspondant à une catégorie d’isolation.

Nous pouvons observer cela avec :

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

Chaque ligne correspond à un namespace différent.

Nous avons donc, pour le shell courant :

- un namespace `mnt` ;

- un namespace `pid` ;

- un namespace `net` ;

- un namespace `uts` ;

- un namespace `ipc` ;

- un namespace `user` ;

- un namespace `cgroup` ;

- un namespace `time`.


---

**Même processus, plusieurs vues.**

Chaque namespace contrôle une vue différente.

Par exemple :

```text
mnt   → vue des montages
pid   → vue des processus
net   → vue réseau
uts   → hostname et domainname
ipc   → ressources IPC
user  → UID, GID et capabilities
cgroup → vue des cgroups
time  → offsets de certaines horloges
```

Nous pouvons donc avoir deux processus qui partagent leur namespace réseau, mais pas leur namespace PID.

Ou inversement, deux processus peuvent partager leur namespace PID, mais avoir des namespaces mount différents.

C’est cette combinaison qui permet de construire des environnements très variés.

---

**Exemple conceptuel.**

Imaginons trois processus :

```text
Processus A
  mnt: 1
  pid: 1
  net: 1
  uts: 1

Processus B
  mnt: 2
  pid: 2
  net: 1
  uts: 2

Processus C
  mnt: 2
  pid: 2
  net: 3
  uts: 2
```

Nous pouvons dire :

- B et C partagent le même namespace mount ;

- B et C partagent le même namespace PID ;

- A et B partagent le même namespace réseau ;

- C a un namespace réseau différent ;

- B et C ont le même hostname isolé ;

- A a une autre vue UTS.


Cette granularité est fondamentale.

Un conteneur n’est donc pas seulement “un namespace”, mais une combinaison de plusieurs namespaces.

---

# 2.2. Observer les namespaces avec `/proc/<PID>/ns`

**Le répertoire `ns`.**

Chaque processus possède un répertoire :

```bash
/proc/<PID>/ns
```

Pour le shell courant :

```bash
ls -l /proc/$$/ns
```

Nous obtenons des liens symboliques spéciaux.

Exemple :

```text
mnt -> mnt:[4026531841]
pid -> pid:[4026531836]
net -> net:[4026531840]
uts -> uts:[4026531838]
```

Ces liens ne sont pas des fichiers ordinaires. Ils représentent les namespaces auxquels appartient le processus.

---

**Lire un namespace précis.**

Nous pouvons lire la cible d’un namespace avec `readlink`.

Exemple :

```bash
readlink /proc/$$/ns/pid
```

Sortie possible :

```text
pid:[4026531836]
```

Pour le namespace réseau :

```bash
readlink /proc/$$/ns/net
```

Sortie possible :

```text
net:[4026531840]
```

Le nombre entre crochets identifie l’instance du namespace.

---

**Comparer deux processus.**

Nous pouvons comparer le shell courant et le PID 1 :

```bash
readlink /proc/$$/ns/pid
readlink /proc/1/ns/pid
```

Si les deux valeurs sont identiques :

```text
pid:[4026531836]
pid:[4026531836]
```

les deux processus partagent le même namespace PID.

Si elles sont différentes :

```text
pid:[4026532501]
pid:[4026531836]
```

les deux processus ne sont pas dans le même namespace PID.

Même principe pour le réseau :

```bash
readlink /proc/$$/ns/net
readlink /proc/1/ns/net
```

---

**Comparaison complète.**

Nous pouvons comparer tous les namespaces de deux processus avec un petit script :

```bash
pid1=$$
pid2=1

for ns in cgroup ipc mnt net pid time user uts; do
    a=$(readlink "/proc/$pid1/ns/$ns" 2>/dev/null || echo "absent")
    b=$(readlink "/proc/$pid2/ns/$ns" 2>/dev/null || echo "absent")

    if [ "$a" = "$b" ]; then
        printf "%-8s partagé     %s\n" "$ns" "$a"
    else
        printf "%-8s différent   %s / %s\n" "$ns" "$a" "$b"
    fi
done
```

Ce type de comparaison est très utile quand nous analysons des conteneurs.

---

# 2.3. Identifiants symboliques des namespaces

**Que signifie `pid:[4026531836]` ?.**

Quand nous voyons :

```text
pid:[4026531836]
```

nous voyons une représentation symbolique d’un namespace PID.

Le nombre n’est pas un PID.

C’est un identifiant interne exposé par le noyau pour reconnaître une instance de namespace.

Deux processus qui affichent exactement la même valeur pour un type de namespace donné partagent ce namespace.

---

**Attention aux comparaisons.**

Nous comparons toujours les namespaces par type.

Par exemple :

```text
pid:[4026531836]
net:[4026531836]
```

Même si les nombres étaient identiques, cela ne signifierait pas que ce sont le même objet, car les types sont différents.

Nous comparons donc :

```text
pid avec pid
net avec net
mnt avec mnt
```

et pas un type avec un autre.

---

# 2.4. Création de namespaces : `clone` et `unshare`

**Deux idées différentes.**

Pour créer ou modifier l’appartenance d’un processus à des namespaces, Linux fournit notamment deux mécanismes :

```text
clone   → créer un nouveau processus, éventuellement dans de nouveaux namespaces
unshare → détacher le processus courant d’un ou plusieurs namespaces
```

En pratique, nous utilisons souvent la commande `unshare`, mais il faut comprendre les idées sous-jacentes.

---

**`clone`.**

L’appel système `clone` permet de créer un nouveau processus ou thread avec un contrôle fin sur ce qui est partagé avec le parent.

Avec certains flags, `clone` peut créer un nouveau namespace.

Exemples conceptuels de flags :

```text
CLONE_NEWNS     → nouveau namespace mount
CLONE_NEWPID    → nouveau namespace PID
CLONE_NEWNET    → nouveau namespace réseau
CLONE_NEWUTS    → nouveau namespace UTS
CLONE_NEWIPC    → nouveau namespace IPC
CLONE_NEWUSER   → nouveau namespace user
CLONE_NEWCGROUP → nouveau namespace cgroup
CLONE_NEWTIME   → nouveau namespace time
```

Nous n’appelons généralement pas `clone` directement en shell. Ce sont les runtimes de conteneurs, bibliothèques système ou programmes spécialisés qui l’utilisent.

---

**`unshare`.**

`unshare` permet à un processus de ne plus partager certains namespaces avec son parent.

La commande `unshare` est l’outil pédagogique principal.

Exemple :

```bash
unshare --uts bash
```

Nous lançons un nouveau shell avec un namespace UTS distinct.

Exemple plus complet :

```bash
unshare --fork --pid --mount-proc bash
```

Nous lançons un shell dans un nouveau namespace PID, avec un `/proc` monté pour refléter cette vue.

---

**Pourquoi `--fork` est important avec le namespace PID.**

Le namespace PID a une particularité : le processus qui demande la création du nouveau namespace PID ne devient pas lui-même PID 1 dans ce namespace.

Ce sont ses enfants qui entrent dans le nouveau namespace PID.

C’est pour cela que nous utilisons souvent :

```bash
unshare --fork --pid --mount-proc bash
```

Ici :

- `unshare` crée le nouveau namespace PID ;

- `--fork` lance un processus enfant dans ce namespace ;

- ce processus enfant devient PID 1 dans le nouveau namespace ;

- `--mount-proc` monte `/proc` pour afficher la bonne vue des processus.


Sans `--fork`, le comportement peut surprendre.

---

# 2.5. Entrer dans un namespace existant : `setns` et `nsenter`

**Le principe de `setns`.**

L’appel système `setns` permet à un processus d’entrer dans un namespace existant, en utilisant un descripteur vers un fichier de namespace, par exemple :

```text
/proc/<PID>/ns/net
```

C’est ce que font des outils comme `nsenter`.

---

**Utiliser `nsenter`.**

La commande `nsenter` permet d’entrer dans les namespaces d’un processus cible.

Exemple :

```bash
sudo nsenter --target <PID> --net bash
```

Nous lançons un shell dans le namespace réseau du processus `<PID>`.

Nous pouvons entrer dans plusieurs namespaces à la fois :

```bash
sudo nsenter --target <PID> --mount --uts --ipc --net --pid bash
```

Cela est très utile pour diagnostiquer un conteneur depuis l’hôte.

---

**Exemple avec un processus cible.**

Nous lançons un shell isolé :

```bash
unshare --uts bash
```

Dans ce shell, nous changeons le hostname :

```bash
hostname ns-test
echo $$
sleep 1000
```

Depuis un autre terminal, nous pouvons entrer dans le namespace UTS du processus :

```bash
sudo nsenter --target <PID> --uts bash
hostname
```

Nous devons voir :

```text
ns-test
```

Cela montre que nous sommes entrés dans le namespace UTS du processus cible.

---

# 2.6. Lister les namespaces avec `lsns`

**Utilisation de base.**

La commande `lsns` liste les namespaces visibles :

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

Les colonnes indiquent notamment :

- identifiant du namespace ;

- type ;

- nombre de processus ;

- PID associé ;

- utilisateur ;

- commande.


---

**Filtrer par type.**

Nous pouvons filtrer par type.

Namespaces PID :

```bash
lsns -t pid
```

Namespaces réseau :

```bash
lsns -t net
```

Namespaces mount :

```bash
lsns -t mnt
```

Namespaces UTS :

```bash
lsns -t uts
```

Cela permet de voir rapidement si des conteneurs ou environnements isolés existent sur la machine.

---

**Interpréter `NPROCS`.**

La colonne `NPROCS` indique le nombre de processus associés au namespace.

Mais nous devons rester prudents :

- certains namespaces peuvent être visibles grâce à des références particulières ;

- un namespace peut être maintenu vivant par un bind mount ;

- la visibilité dépend des permissions ;

- un processus peut disparaître pendant l’observation.


`lsns` donne une vue très utile, mais ce n’est pas une vérité absolue déconnectée du contexte.

---

# 2.7. Héritage des namespaces

**Héritage lors de la création d’un processus.**

Quand un processus crée un enfant, celui-ci hérite normalement des namespaces de son parent.

Exemple :

```bash
bash
```

Le nouveau shell partage les mêmes namespaces que le shell parent, sauf si nous demandons explicitement à en créer de nouveaux.

Nous pouvons vérifier :

```bash
readlink /proc/$$/ns/uts
bash -c 'readlink /proc/$$/ns/uts'
```

Les deux valeurs sont généralement identiques.

---

**Création explicite d’un nouveau namespace.**

Avec `unshare`, nous créons une rupture.

Exemple :

```bash
readlink /proc/$$/ns/uts
unshare --uts bash -c 'readlink /proc/$$/ns/uts'
```

Les deux valeurs doivent être différentes.

Cela signifie que le processus lancé par `unshare` appartient à un nouveau namespace UTS.

---

**Héritage dans un conteneur.**

Dans un conteneur, le processus principal est lancé dans un ensemble de namespaces.

Ses processus enfants héritent ensuite de ces namespaces.

C’est pourquoi tous les processus d’un même conteneur partagent généralement :

- le même namespace PID ;

- le même namespace mount ;

- le même namespace UTS ;

- le même namespace network ;

- le même namespace IPC.


Sauf configuration particulière.

---

# 2.8. Durée de vie d’un namespace

**Quand un namespace disparaît-il ?.**

Un namespace reste vivant tant qu’il existe au moins une référence vers lui.

La référence la plus évidente est un processus qui appartient à ce namespace.

Quand le dernier processus d’un namespace disparaît, le namespace peut être détruit.

Mais ce n’est pas la seule forme de référence.

---

**Namespace maintenu par un fichier ouvert.**

Les entrées de `/proc/<PID>/ns/*` peuvent être ouvertes comme des fichiers.

Si un processus garde un descripteur ouvert vers un namespace, il peut maintenir ce namespace vivant.

C’est une notion importante pour des outils avancés.

---

**Namespace maintenu par bind mount.**

Un namespace peut être maintenu vivant avec un bind mount vers son fichier de namespace.

Exemple conceptuel :

```bash
touch /tmp/ns-net
mount --bind /proc/<PID>/ns/net /tmp/ns-net
```

Même si le processus initial disparaît, le namespace peut rester référencé par le bind mount.

Nous pouvons ensuite entrer dans ce namespace via ce chemin, selon les outils et les permissions.

C’est notamment une idée utilisée par certains mécanismes de gestion de namespaces réseau.

---

# 2.9. Namespaces nommés avec `ip netns`

**Cas particulier des namespaces réseau.**

La commande `ip netns` fournit une interface pratique pour gérer des namespaces réseau nommés.

Créer un namespace réseau :

```bash
sudo ip netns add ns1
```

Lister :

```bash
ip netns list
```

Exécuter une commande dans ce namespace :

```bash
sudo ip netns exec ns1 ip addr
```

Supprimer :

```bash
sudo ip netns delete ns1
```

---

**Où sont stockées les références ?.**

Les namespaces réseau nommés sont généralement référencés sous :

```bash
/var/run/netns
```

ou :

```bash
/run/netns
```

Nous pouvons voir :

```bash
ls -l /run/netns
```

Ces fichiers sont des références vers des namespaces réseau.

Cela permet au namespace de survivre même sans processus actif permanent à l’intérieur.

---

**Pourquoi c’est utile ?.**

Les namespaces réseau nommés sont très utiles pour :

- créer des laboratoires réseau ;

- tester des routes ;

- simuler plusieurs machines ;

- comprendre Docker ou Kubernetes ;

- manipuler des paires `veth` ;

- créer des topologies réseau locales.


Nous les étudions plus en détail dans le chapitre consacré au namespace réseau.

---

# 2.10. Les namespaces et `/proc`

**`/proc` comme outil d’inspection.**

`/proc` est l’interface principale pour observer les namespaces d’un processus.

Nous utilisons :

```bash
/proc/<PID>/ns
```

mais aussi :

```bash
/proc/<PID>/status
/proc/<PID>/uid_map
/proc/<PID>/gid_map
/proc/<PID>/cgroup
/proc/<PID>/mountinfo
```

Selon le type de namespace, différentes parties de `/proc` deviennent utiles.

---

**PID namespace et `/proc`.**

Dans un namespace PID, `/proc` doit généralement être remonté pour refléter correctement les processus visibles.

Exemple :

```bash
unshare --fork --pid bash
ps aux
```

peut donner une vue confuse si `/proc` reste celui de l’hôte.

La forme plus correcte est :

```bash
unshare --fork --pid --mount-proc bash
ps aux
```

Nous voyons alors une vue cohérente du namespace PID.

---

**Network namespace et `/proc/net`.**

Le contenu de :

```bash
/proc/net
```

dépend du namespace réseau courant.

Exemple :

```bash
readlink /proc/$$/ns/net
cat /proc/net/dev
```

Dans un autre namespace réseau, nous pouvons voir d’autres interfaces.

---

**User namespace et UID/GID maps.**

Pour un user namespace, nous regardons :

```bash
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Ces fichiers montrent comment les identifiants utilisateurs et groupes sont mappés entre l’intérieur du namespace et l’extérieur.

Exemple :

```text
         0       1000          1
```

Cela signifie que l’UID `0` à l’intérieur correspond à l’UID `1000` à l’extérieur, pour une plage de taille `1`.

---

# 2.11. Relation avec les conteneurs

**Un conteneur est un assemblage de namespaces.**

Un conteneur typique combine plusieurs namespaces :

```text
mnt
pid
net
uts
ipc
user parfois
cgroup
time parfois
```

Le runtime crée ces namespaces, configure les montages, les cgroups, les capabilities et lance le processus initial.

---

**Observer un conteneur depuis l’hôte.**

Nous pouvons récupérer le PID d’un conteneur, puis observer ses namespaces.

Avec Docker, par exemple :

```bash
docker inspect --format '{{.State.Pid}}' <container>
```

Puis :

```bash
sudo ls -l /proc/<PID>/ns
```

Nous pouvons comparer avec l’hôte :

```bash
ls -l /proc/1/ns
```

Nous voyons quels namespaces sont isolés.

---

**Entrer dans les namespaces d’un conteneur.**

Depuis l’hôte :

```bash
sudo nsenter --target <PID> --mount --uts --ipc --net --pid bash
```

Nous entrons dans plusieurs namespaces du conteneur.

C’est une technique utile pour diagnostiquer un conteneur même si aucun shell n’est disponible dans son image.

---

# 2.12. Pièges classiques

**Confondre PID hôte et PID conteneur.**

Un processus peut avoir un PID sur l’hôte et un autre PID dans un namespace PID.

Dans le conteneur :

```bash
echo $$
```

peut afficher :

```text
1
```

Sur l’hôte, le même processus peut être :

```text
24531
```

Nous devons toujours préciser depuis quel namespace nous observons.

---

**Oublier de remonter `/proc`.**

Quand nous créons un namespace PID à la main, nous devons monter `/proc` correctement.

Sinon, `ps` et `/proc` peuvent ne pas refléter la vue attendue.

Bonne pratique pédagogique :

```bash
unshare --fork --pid --mount-proc bash
```

---

**Croire qu’un namespace limite les ressources.**

Un namespace isole une vue.

Il ne limite pas automatiquement la consommation de ressources.

Pour les ressources, nous devons utiliser les cgroups.

Nous retenons :

```text
Namespaces = isolation des vues.
Cgroups = contrôle et comptabilité des ressources.
```

---

**Confondre user namespace et privilèges réels.**

Dans un user namespace, nous pouvons être `root` à l’intérieur.

Mais cela ne signifie pas que nous avons les privilèges root complets sur l’hôte.

Exemple :

```bash
unshare --user --map-root-user bash
id
```

Nous pouvons voir :

```text
uid=0(root)
```

Mais cette racine est limitée au namespace.

C’est une distinction centrale pour les conteneurs rootless.

---

# 2.14. Ce que nous devons retenir

Nous retenons les points suivants :

1. Un processus appartient simultanément à plusieurs namespaces.

2. Les namespaces d’un processus sont visibles dans `/proc/<PID>/ns`.

3. Deux processus partagent un namespace si le lien symbolique correspondant a la même valeur.

4. Les namespaces s’héritent normalement lors de la création de processus.

5. `clone` peut créer un processus dans de nouveaux namespaces.

6. `unshare` permet de détacher un processus de certains namespaces.

7. `setns` permet d’entrer dans un namespace existant.

8. `nsenter` est l’outil pratique qui exploite cette logique.

9. `lsns` permet de lister les namespaces visibles sur le système.

10. Un namespace vit tant qu’il existe une référence vers lui.

11. Un namespace peut être maintenu vivant par un processus, un fichier ouvert ou un bind mount.

12. Les namespaces réseau nommés avec `ip netns` reposent sur des références dans `/run/netns`.

13. `/proc` est essentiel pour inspecter les namespaces.

14. Dans un namespace PID, il faut remonter `/proc` pour obtenir une vue cohérente.

15. Les namespaces isolent les vues, mais ne limitent pas automatiquement les ressources.


---

# Conclusion du chapitre 2

Nous avons étudié l’architecture générale des namespaces Linux.

Nous savons maintenant qu’un processus appartient à plusieurs namespaces en même temps, et que nous pouvons observer ces appartenances avec `/proc/<PID>/ns`. Nous avons compris que les namespaces sont hérités par défaut, mais qu’ils peuvent être créés ou modifiés avec des mécanismes comme `clone`, `unshare` et `setns`.

Nous avons également introduit les outils pratiques `unshare`, `nsenter`, `lsns` et `ip netns`, qui nous permettent de créer, lister et rejoindre des namespaces.

Dans le chapitre suivant, nous détaillons précisément les outils de manipulation des namespaces. Nous verrons comment les utiliser proprement, quelles options connaître, et comment éviter les erreurs classiques lors des manipulations manuelles.

---

## Complément 2026 — `clone3()`, FDs de namespace et `pidfd`

La présentation historique de l'API reste correcte, mais les outils modernes poussent davantage à raisonner avec des **descripteurs de fichiers** qu'avec de simples PID. `clone3()` fournit une interface structurée et extensible à la création de processus. `setns()` travaille sur un descripteur qui référence un namespace, et les pidfds permettent de désigner un processus sans dépendre uniquement d'un numéro de PID susceptible d'être réutilisé.

Pour l'administration quotidienne, nous utilisons surtout `unshare(1)`, `nsenter(1)` et `lsns(8)`. Pour un runtime ou un outil bas niveau, comprendre les FDs de namespace, `nsfs` et les pidfds permet d'éviter plusieurs courses classiques liées à `/proc/<PID>`.

---

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

# 3.1. Vue d’ensemble des outils

**Pourquoi avons-nous besoin d’outils ?.**

Les namespaces sont des objets du noyau. Nous ne les manipulons pas directement comme des fichiers classiques.

Pour les utiliser, nous avons besoin d’outils qui appellent les bons mécanismes noyau :

- `clone` pour créer un nouveau processus dans un ou plusieurs nouveaux namespaces ;

- `unshare` pour détacher un processus de certains namespaces ;

- `setns` pour rejoindre un namespace existant.


En pratique, nous utilisons surtout des commandes déjà disponibles sur la plupart des distributions Linux.

---

**Les outils principaux.**

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

# 3.2. `unshare`

**Rôle de `unshare`.**

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

**Options principales.**

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

**Créer un namespace UTS.**

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

**Créer un namespace PID.**

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

**Créer un namespace mount.**

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

**Créer un namespace réseau.**

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

**Créer un namespace user.**

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

**Combiner plusieurs namespaces.**

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

# 3.3. `nsenter`

**Rôle de `nsenter`.**

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

**Options principales.**

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

**Entrer dans un namespace réseau.**

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

**Entrer dans plusieurs namespaces.**

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

**Utilisation avec les conteneurs Docker.**

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

# 3.4. `lsns`

**Rôle de `lsns`.**

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

**Colonnes principales.**

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

**Filtrer par type.**

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

**Voir les namespaces d’un processus précis.**

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

# 3.5. `ip netns`

**Rôle de `ip netns`.**

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

**Où sont stockés les namespaces nommés ?.**

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

**Créer un namespace réseau et activer loopback.**

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

**Différence entre `unshare --net` et `ip netns`.**

`unshare --net` crée un namespace réseau pour un processus lancé.

`ip netns add` crée un namespace réseau nommé, avec une référence persistante dans `/run/netns`.

Nous pouvons résumer :

```text
unshare --net  → pratique pour lancer un shell ou une commande isolée
ip netns       → pratique pour créer des namespaces réseau nommés et réutilisables
```

Pour des laboratoires réseau, `ip netns` est souvent plus confortable.

---

# 3.6. `ip` pour configurer le réseau des namespaces

**Pourquoi `ip` est indispensable ?.**

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

**Créer une paire `veth`.**

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

**Placer une interface dans un namespace.**

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

# 3.7. `mount` et `findmnt`

**Rôle de `mount`.**

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

**Rôle de `findmnt`.**

`findmnt` permet d’observer les montages de manière plus lisible :

```bash
findmnt
findmnt /tmp/mnt-test
findmnt /proc
```

Nous l’utilisons souvent pour vérifier si un montage existe dans le namespace courant.

---

**Propagation des montages.**

Un piège classique est la propagation des montages.

Selon la configuration, un montage créé dans un namespace peut être propagé ailleurs.

Pour éviter cela en TP, nous pouvons rendre les montages privés :

```bash
mount --make-rprivate /
```

Nous détaillerons ce point dans le chapitre sur le namespace mount.

Pour l’instant, nous retenons que le namespace mount ne se comprend pas correctement sans la notion de propagation.

---

# 3.8. `ps` et le namespace PID

**Pourquoi `ps` peut être trompeur.**

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

**Vérifier le PID 1.**

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

# 3.9. `capsh` et les capabilities

**Pourquoi parler des capabilities dans un chapitre sur les outils ?.**

Les namespaces ne suffisent pas à comprendre les privilèges.

Un processus peut être dans un namespace isolé, mais ses permissions dépendent aussi des capabilities.

Nous pouvons afficher les capabilities avec :

```bash
capsh --print
```

Si `capsh` n’est pas installé, il peut être fourni par un paquet comme `libcap2-bin` sur Debian/Ubuntu.

---

**Lire les capabilities dans `/proc`.**

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

# 3.10. `newuidmap` et `newgidmap`

**Pourquoi ces outils existent-ils ?.**

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

**Observer les mappings.**

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

**Pourquoi c’est important ?.**

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

# 3.11. Chaîne d’outils typique pour diagnostiquer un conteneur

**Trouver le processus.**

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

**Observer ses namespaces.**

```bash
sudo ls -l /proc/<PID>/ns
```

Comparer avec l’hôte :

```bash
ls -l /proc/1/ns
```

Nous voyons quels namespaces sont différents.

---

**Entrer dans ses namespaces.**

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

**Vérifier les limites de ressources.**

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

# 3.12. Pièges classiques avec les outils

**Oublier les droits.**

Certaines commandes nécessitent des droits administrateur :

```bash
unshare --net bash
ip link add ...
nsenter --target <PID> --net bash
mount -t tmpfs ...
```

Selon la distribution et la configuration, un utilisateur non privilégié peut créer certains namespaces, notamment user namespaces, mais pas tous.

---

**Oublier `--fork` avec PID namespace.**

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

**Oublier de monter `/proc`.**

Sans `/proc` cohérent, les outils comme `ps` peuvent mentir ou afficher la vue de l’hôte.

Nous utilisons :

```bash
--mount-proc
```

avec `unshare`.

---

**Confondre namespace réseau nommé et processus vivant.**

Un namespace réseau créé par :

```bash
sudo ip netns add ns1
```

peut exister grâce à une référence dans `/run/netns`, même si aucun processus ne tourne dedans.

C’est différent d’un namespace créé uniquement pour un processus avec `unshare --net`.

---

**Croire que `nsenter` crée une isolation.**

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

# 3.14. Ce que nous devons retenir

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

# Conclusion du chapitre 3

Nous avons étudié les principaux outils qui permettent de manipuler les namespaces Linux.

Nous savons maintenant créer des namespaces avec `unshare`, les observer avec `lsns` et `/proc/<PID>/ns`, les rejoindre avec `nsenter`, et manipuler les namespaces réseau avec `ip netns`.

Nous avons aussi vu plusieurs pièges importants : le rôle de `--fork` avec le namespace PID, la nécessité de remonter `/proc`, la différence entre créer et rejoindre un namespace, et la différence entre namespace réseau temporaire et namespace réseau nommé.

Dans le chapitre suivant, nous commençons l’étude détaillée des namespaces un par un avec le namespace UTS, qui isole le hostname et le domainname.

---

# Chapitre 4 — Namespace UTS

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace UTS.

Le namespace UTS est l’un des namespaces les plus simples à comprendre. Il permet d’isoler deux informations du système :

- le hostname ;

- le domainname, aussi appelé nom de domaine NIS dans certains contextes historiques.


À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace UTS ;

- comprendre pourquoi le hostname peut être différent dans un conteneur ;

- créer un namespace UTS avec `unshare` ;

- modifier le hostname dans un namespace isolé ;

- comparer les namespaces UTS de deux processus ;

- faire le lien avec Docker, Podman et Kubernetes ;

- comprendre les limites de cette isolation.


---

# 4.1. Qu’est-ce que le namespace UTS ?

**Définition.**

Le namespace UTS isole l’identité système visible par un processus.

Concrètement, il isole principalement :

```text
hostname
domainname
```

Le hostname correspond au nom de la machine tel qu’il est vu par les processus.

Nous pouvons le lire avec :

```bash
hostname
```

ou :

```bash
cat /proc/sys/kernel/hostname
```

Sur une machine classique, cela peut donner :

```text
serveur-prod-01
```

Dans un conteneur, cela peut donner autre chose, par exemple :

```text
b7f3c2a91d4e
```

Cela ne signifie pas que le conteneur a son propre noyau. Cela signifie qu’il a une vue UTS différente.

---

**Origine du nom UTS.**

UTS vient historiquement de “UNIX Time-Sharing System”.

Aujourd’hui, dans le contexte des namespaces Linux, nous retenons surtout que le namespace UTS isole le nom de la machine.

Il ne faut pas le confondre avec le namespace time, qui concerne certaines horloges.

Le namespace UTS ne sert pas à gérer le temps. Il sert à isoler l’identité système visible.

---

# 4.2. Le hostname

**Lire le hostname.**

Nous pouvons lire le hostname avec :

```bash
hostname
```

Exemple :

```text
machine-hote
```

Nous pouvons aussi lire :

```bash
cat /proc/sys/kernel/hostname
```

Ces deux commandes donnent normalement le hostname actif du namespace UTS courant.

Cela signifie que si nous sommes dans un namespace UTS isolé, nous voyons le hostname de ce namespace, pas forcément celui de l’hôte.

---

**Modifier le hostname.**

Sur un système classique, le hostname peut être modifié avec :

```bash
sudo hostname nouveau-nom
```

ou, sur un système avec systemd :

```bash
sudo hostnamectl set-hostname nouveau-nom
```

Mais dans un namespace UTS isolé, modifier le hostname ne modifie que la vue du namespace.

C’est précisément ce que nous voulons expérimenter.

---

**Hostname actif et configuration persistante.**

Nous distinguons deux choses :

```text
hostname actif
configuration persistante du hostname
```

Le hostname actif peut être lu dans :

```bash
cat /proc/sys/kernel/hostname
```

La configuration persistante dépend de la distribution. Sur beaucoup de distributions, elle se trouve dans :

```bash
/etc/hostname
```

Mais dans un conteneur, cette distinction peut être différente, car le hostname est souvent injecté au lancement par le runtime.

Nous retenons :

```text
Le namespace UTS isole le hostname actif vu par les processus.
Il ne crée pas à lui seul une configuration persistante complète.
```

---

# 4.3. Le domainname

**Lire le domainname.**

Le namespace UTS isole aussi le domainname.

Nous pouvons le lire avec :

```bash
domainname
```

ou :

```bash
cat /proc/sys/kernel/domainname
```

Sur beaucoup de machines modernes, nous pouvons obtenir :

```text
(none)
```

ou une valeur vide selon les outils et distributions.

Le domainname ici est historiquement lié au domaine NIS/YP, et non nécessairement au nom DNS complet de la machine.

---

**À ne pas confondre avec DNS.**

Le domainname du namespace UTS n’est pas forcément le domaine DNS utilisé pour résoudre les noms.

Par exemple, une machine peut avoir :

```text
hostname : serveur1
domainname UTS : (none)
nom DNS complet : serveur1.example.org
```

La résolution DNS dépend plutôt de fichiers et services comme :

```text
/etc/hosts
/etc/resolv.conf
systemd-resolved
DNS externe
```

Le namespace UTS ne suffit donc pas à isoler toute la configuration de résolution de noms.

---

# 4.4. Observer le namespace UTS d’un processus

**Avec `/proc/<PID>/ns/uts`.**

Comme les autres namespaces, le namespace UTS d’un processus est visible dans :

```bash
/proc/<PID>/ns/uts
```

Pour notre shell courant :

```bash
readlink /proc/$$/ns/uts
```

Sortie possible :

```text
uts:[4026531838]
```

Pour le PID 1 :

```bash
readlink /proc/1/ns/uts
```

Si les deux valeurs sont identiques, notre shell et le PID 1 partagent le même namespace UTS.

---

**Comparer deux processus.**

Nous pouvons comparer :

```bash
echo "Shell courant : $(readlink /proc/$$/ns/uts)"
echo "PID 1         : $(readlink /proc/1/ns/uts)"
```

Exemple :

```text
Shell courant : uts:[4026531838]
PID 1         : uts:[4026531838]
```

Cela signifie que les deux processus partagent le même namespace UTS.

Si nous voyons :

```text
Shell courant : uts:[4026532500]
PID 1         : uts:[4026531838]
```

cela signifie que le shell courant est dans un namespace UTS différent de celui du PID 1.

---

# 4.5. Créer un namespace UTS avec `unshare`

**Expérience simple.**

Nous commençons par afficher le hostname de l’hôte :

```bash
hostname
readlink /proc/$$/ns/uts
```

Exemple :

```text
machine-hote
uts:[4026531838]
```

Nous créons ensuite un nouveau namespace UTS :

```bash
unshare --uts bash
```

Dans ce nouveau shell :

```bash
readlink /proc/$$/ns/uts
hostname
```

Nous devons voir un identifiant UTS différent, mais le hostname initial peut encore être identique.

Le namespace est différent, mais il a été initialisé avec une copie de la valeur existante.

---

**Modifier le hostname dans le namespace.**

Dans le shell isolé :

```bash
hostname ns-uts-demo
hostname
cat /proc/sys/kernel/hostname
```

Nous obtenons :

```text
ns-uts-demo
```

Nous quittons :

```bash
exit
```

Puis, sur l’hôte :

```bash
hostname
cat /proc/sys/kernel/hostname
```

Nous retrouvons le hostname de l’hôte.

Conclusion :

```text
Le hostname a changé dans le namespace UTS isolé.
Le hostname de l’hôte n’a pas été modifié.
```

---

**Pourquoi cela fonctionne-t-il ?.**

Quand nous faisons :

```bash
unshare --uts bash
```

le nouveau shell ne partage plus le namespace UTS de son parent.

Il reçoit une nouvelle instance de namespace UTS.

Cette instance contient initialement les mêmes valeurs, mais elles peuvent ensuite évoluer séparément.

Nous avons donc :

```text
avant unshare :
shell et hôte partagent le même namespace UTS

après unshare :
shell isolé et hôte ont deux namespaces UTS différents
```

---

# 4.6. Permissions nécessaires

**Création d’un namespace UTS.**

Selon la distribution et la configuration du noyau, créer un namespace UTS peut nécessiter des privilèges.

La commande suivante peut échouer :

```bash
unshare --uts bash
```

Erreur possible :

```text
unshare: unshare failed: Operation not permitted
```

Dans ce cas, nous pouvons avoir besoin de droits administrateur :

```bash
sudo unshare --uts bash
```

Ou bien nous pouvons utiliser un user namespace combiné, selon la configuration :

```bash
unshare --user --map-root-user --uts bash
```

---

**Modifier le hostname.**

Modifier le hostname demande normalement la capacité :

```text
CAP_SYS_ADMIN
```

dans le namespace utilisateur qui gouverne le namespace UTS.

Cela signifie qu’un utilisateur non privilégié ne peut pas toujours modifier le hostname directement.

Mais dans un user namespace avec mapping root :

```bash
unshare --user --map-root-user --uts bash
```

nous pouvons parfois être `root` dans le namespace et disposer des capabilities nécessaires à l’intérieur de ce contexte.

Nous vérifions :

```bash
id
hostname ns-demo
```

Si cela fonctionne, le changement reste limité au namespace UTS créé.

---

# 4.9. Lien avec `/proc`

**Lire le hostname via `/proc`.**

Le hostname actif du namespace UTS courant est visible dans :

```bash
cat /proc/sys/kernel/hostname
```

Dans un namespace UTS isolé :

```bash
unshare --uts bash
hostname ns-proc-demo
cat /proc/sys/kernel/hostname
```

Nous voyons :

```text
ns-proc-demo
```

Après `exit`, sur l’hôte :

```bash
cat /proc/sys/kernel/hostname
```

Nous retrouvons le hostname de l’hôte.

---

**Lire le namespace UTS via `/proc/<PID>/ns`.**

Nous pouvons voir le namespace UTS courant :

```bash
readlink /proc/$$/ns/uts
```

Dans deux terminaux différents, nous pouvons comparer les valeurs.

Dans un namespace UTS créé avec `unshare`, l’identifiant change.

---

**`/proc/sys/kernel/hostname` dépend du namespace.**

C’est un point important.

Le chemin :

```bash
/proc/sys/kernel/hostname
```

est le même, mais son contenu dépend du namespace UTS du processus qui le lit.

Nous pouvons donc avoir :

```text
même chemin
contenu différent selon le namespace
```

C’est une excellente illustration de l’idée de namespace.

---

# 4.11. Limites du namespace UTS

**Il n’isole pas le réseau.**

Changer le hostname ne crée pas une nouvelle pile réseau.

Pour isoler les interfaces, les routes, les ports et les sockets, nous avons besoin du namespace réseau.

Le namespace UTS ne fait que modifier l’identité système visible.

---

**Il n’isole pas le système de fichiers.**

Le namespace UTS ne change pas l’arborescence de fichiers.

Même si le hostname est différent, les processus peuvent encore voir les mêmes fichiers si le namespace mount n’est pas isolé.

Pour isoler les montages, nous avons besoin du namespace mount.

---

**Il n’isole pas les processus.**

Le namespace UTS ne modifie pas la table des processus visible.

Pour isoler les PID, nous avons besoin du namespace PID.

Exemple :

```bash
unshare --uts bash
ps aux
```

Nous voyons encore les processus du même namespace PID que le parent.

---

**Il ne limite aucune ressource.**

Le namespace UTS ne limite pas :

- CPU ;

- mémoire ;

- nombre de processus ;

- I/O disque ;

- réseau.


Pour limiter les ressources, nous avons besoin des cgroups.

---

# 4.12. Cas d’usage du namespace UTS

**Donner une identité à un conteneur.**

Le cas d’usage principal est simple : donner un hostname propre à un environnement isolé.

Un conteneur peut ainsi croire qu’il s’appelle :

```text
app-frontend-1
```

alors que l’hôte s’appelle :

```text
node-prod-42
```

Cela aide les applications, logs et outils système à avoir une identité cohérente.

---

**Tests et environnements temporaires.**

Nous pouvons tester des applications qui lisent le hostname sans modifier l’hôte.

Exemple :

```bash
unshare --uts bash
hostname test-env
./application
```

L’application voit `test-env`, mais l’hôte n’est pas modifié.

---

**Formation et démonstration.**

Le namespace UTS est idéal pour introduire les namespaces, car :

- il est simple ;

- il produit un effet visible immédiatement ;

- il ne demande pas de configuration réseau ;

- il illustre bien la notion de vue isolée.


---

# 4.13. Pièges classiques

**Croire que le hostname DNS change automatiquement.**

Changer le hostname avec :

```bash
hostname ns-demo
```

ne modifie pas forcément :

- `/etc/hosts` ;

- `/etc/resolv.conf` ;

- les entrées DNS ;

- les services Kubernetes ;

- les certificats TLS ;

- les configurations applicatives.


Le hostname système est une chose. La résolution DNS en est une autre.

---

**Croire que le conteneur a son propre noyau.**

Un hostname différent ne signifie pas que nous avons un noyau différent.

Dans un conteneur :

```sh
hostname
uname -a
```

Le hostname peut être propre au conteneur, mais `uname -a` affiche le noyau de l’hôte.

Nous retenons :

```text
Le namespace UTS isole l’identité visible.
Il ne virtualise pas le noyau.
```

---

**Oublier les permissions.**

Si :

```bash
unshare --uts bash
```

ou :

```bash
hostname ns-demo
```

échoue avec :

```text
Operation not permitted
```

cela indique un problème de permissions ou de capabilities.

Nous pouvons tester avec :

```bash
sudo unshare --uts bash
```

ou, selon la configuration :

```bash
unshare --user --map-root-user --uts bash
```

---

# 4.15. Ce que nous devons retenir

Nous retenons les points suivants :

1. Le namespace UTS isole le hostname et le domainname.

2. Il ne crée pas un nouveau noyau.

3. Il ne modifie pas à lui seul le réseau, les processus ou les montages.

4. Le hostname actif est visible avec `hostname` et `/proc/sys/kernel/hostname`.

5. Le namespace UTS d’un processus est visible avec `/proc/<PID>/ns/uts`.

6. Deux processus partagent le même namespace UTS si `readlink /proc/<PID>/ns/uts` donne la même valeur.

7. `unshare --uts bash` permet de créer un nouveau namespace UTS.

8. Modifier le hostname dans ce namespace ne modifie pas celui de l’hôte.

9. Docker et Podman utilisent le namespace UTS pour donner un hostname propre aux conteneurs.

10. Kubernetes configure également une identité de pod, mais ajoute une couche DNS et orchestration.

11. Changer le hostname ne change pas automatiquement la résolution DNS.

12. Les permissions et capabilities peuvent empêcher la création ou la modification du namespace UTS.

13. Le namespace UTS est simple, mais il illustre parfaitement l’idée de vue isolée.


---

# Conclusion du chapitre 4

Nous avons étudié le namespace UTS, l’un des namespaces les plus simples et les plus pédagogiques.

Nous savons maintenant qu’il permet d’isoler le hostname et le domainname d’un groupe de processus. Nous avons vu comment créer un namespace UTS avec `unshare`, comment observer son identifiant avec `/proc/<PID>/ns/uts`, comment modifier le hostname dans une vue isolée, et comment entrer dans ce namespace avec `nsenter`.

Nous retenons surtout que le namespace UTS ne crée pas une machine virtuelle. Il ne change ni le noyau, ni la pile réseau, ni les processus visibles, ni les ressources disponibles. Il isole seulement une partie de l’identité système.

Dans le chapitre suivant, nous étudions le namespace PID, beaucoup plus central pour les conteneurs, car il permet à un processus d’être PID 1 dans son propre environnement.

---

# Chapitre 5 — Namespace PID

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace PID.

Le namespace PID est l’un des namespaces les plus importants pour comprendre les conteneurs. Il permet d’isoler la numérotation et la visibilité des processus. Grâce à lui, un processus peut être vu comme `PID 1` à l’intérieur d’un environnement isolé, tout en ayant un autre PID sur l’hôte.

À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace PID ;

- comprendre qu’un processus peut avoir plusieurs PID selon le namespace d’observation ;

- créer un namespace PID avec `unshare` ;

- comprendre pourquoi `--fork` est nécessaire ;

- comprendre pourquoi nous devons remonter `/proc` ;

- expliquer le rôle particulier du PID 1 ;

- identifier les problèmes liés aux signaux et aux processus zombies ;

- faire le lien avec Docker, Podman et Kubernetes.


---

# 5.1. Pourquoi isoler les PID ?

**Le problème de la visibilité des processus.**

Sur une machine Linux classique, nous pouvons lister les processus avec :

```bash
ps aux
```

Nous voyons alors les processus du système :

```text
root           1  ...  /sbin/init
root         512  ...  systemd-journald
user        2042  ...  bash
user        2101  ...  firefox
```

Dans un conteneur, nous ne voulons généralement pas que l’application voie tous les processus de l’hôte.

Nous voulons plutôt qu’elle voie seulement les processus de son environnement.

Le namespace PID permet cette isolation.

---

**Le principe.**

Le namespace PID isole la vue des identifiants de processus.

Cela signifie qu’un processus peut avoir :

- un PID dans le namespace de l’hôte ;

- un autre PID dans un namespace enfant ;

- éventuellement encore un autre PID dans un namespace plus profond.


Nous pouvons donc avoir une situation comme :

```text
Vue depuis l’hôte :
PID 24531 -> processus node

Vue depuis le conteneur :
PID 1 -> même processus node
```

C’est le même processus réel, mais il est vu avec des identifiants différents selon le namespace depuis lequel nous l’observons.

---

# 5.2. Observer le namespace PID courant

**Avec `/proc/<PID>/ns/pid`.**

Nous pouvons observer le namespace PID de notre shell courant :

```bash
readlink /proc/$$/ns/pid
```

Exemple :

```text
pid:[4026531836]
```

Nous pouvons comparer avec le processus 1 :

```bash
readlink /proc/1/ns/pid
```

Si les valeurs sont identiques, notre shell et le PID 1 partagent le même namespace PID.

---

**Comparer plusieurs processus.**

Nous pouvons comparer deux processus :

```bash
pid_a=$$
pid_b=1

readlink /proc/$pid_a/ns/pid
readlink /proc/$pid_b/ns/pid
```

Même valeur :

```text
pid:[4026531836]
pid:[4026531836]
```

Les processus partagent le même namespace PID.

Valeurs différentes :

```text
pid:[4026532501]
pid:[4026531836]
```

Les processus sont dans des namespaces PID différents.

---

# 5.3. Créer un namespace PID avec `unshare`

**Première commande correcte.**

Pour créer un namespace PID de manière pédagogique, nous utilisons :

```bash
unshare --fork --pid --mount-proc bash
```

Dans le shell obtenu, nous exécutons :

```bash
echo $$
ps aux
cat /proc/1/comm
readlink /proc/$$/ns/pid
```

Nous devons observer que notre shell est vu comme `PID 1` dans ce nouveau namespace.

---

**Pourquoi utiliser `--fork` ?.**

Le namespace PID a une particularité importante.

Quand un processus crée un nouveau namespace PID, ce n’est pas ce processus lui-même qui devient PID 1 dans le nouveau namespace. Ce sont ses enfants qui entrent dans ce nouveau namespace.

C’est pour cela que nous utilisons :

```bash
--fork
```

Avec :

```bash
unshare --fork --pid bash
```

`unshare` crée un enfant, et cet enfant devient le premier processus du nouveau namespace PID.

Nous retenons :

```text
Pour créer un namespace PID utilisable en shell, nous utilisons presque toujours --fork.
```

---

**Pourquoi utiliser `--mount-proc` ?.**

Les commandes comme `ps` lisent les informations dans `/proc`.

Si nous créons un nouveau namespace PID sans remonter `/proc`, nous pouvons obtenir une vue incohérente.

Exemple à éviter pour débuter :

```bash
unshare --fork --pid bash
ps aux
```

Le shell est dans un nouveau namespace PID, mais `/proc` peut encore refléter l’ancien environnement de montage.

La forme correcte est :

```bash
unshare --fork --pid --mount-proc bash
```

`--mount-proc` monte un `/proc` adapté au nouveau namespace PID.

Nous retenons :

```text
Namespace PID isolé + /proc non remonté = diagnostic souvent trompeur.
```

---

# 5.4. Le PID 1 dans un namespace PID

**Un processus particulier.**

Dans chaque namespace PID, il existe un processus vu comme `PID 1`.

Ce processus a un rôle particulier.

Sur un système classique, le PID 1 est souvent :

```text
systemd
```

ou un autre système d’initialisation.

Dans un conteneur, le PID 1 peut être :

```text
node
python
nginx
bash
tini
```

Nous pouvons observer cela avec :

```bash
cat /proc/1/comm
```

---

**Pourquoi le PID 1 est important ?.**

Le PID 1 a des responsabilités particulières :

- il reçoit certains signaux différemment ;

- il doit récolter les processus enfants terminés ;

- il devient le parent adoptif de certains processus orphelins ;

- il influence l’arrêt propre d’un conteneur ;

- il peut accumuler des zombies s’il ne gère pas correctement ses enfants.


Dans un conteneur, si notre application est directement PID 1, elle doit gérer correctement ces aspects.

---

**Exemple simple.**

Nous lançons :

```bash
unshare --fork --pid --mount-proc bash
```

Dans le shell :

```bash
echo $$
cat /proc/1/comm
ps -ef
```

Nous pouvons voir que `bash` est PID 1.

Exemple :

```text
UID        PID  PPID  CMD
root         1     0  bash
root        10     1  ps -ef
```

Ici, `bash` est le premier processus du namespace.

---

# 5.5. Processus zombies et rôle du PID 1

**Qu’est-ce qu’un processus zombie ?.**

Un processus zombie est un processus terminé dont le parent n’a pas encore récupéré le code de retour.

Nous pouvons en voir avec :

```bash
ps -eo pid,ppid,state,comm | awk '$3=="Z"'
```

Un zombie ne consomme presque plus de mémoire, mais il occupe encore une entrée dans la table des processus.

---

**Pourquoi cela concerne les conteneurs ?.**

Dans un conteneur, si le processus PID 1 lance des enfants et ne les récolte pas correctement, les zombies peuvent s’accumuler.

Cela arrive notamment quand une application n’est pas conçue pour jouer le rôle de processus d’init.

Exemple conceptuel :

```text
PID 1 : application
  ├── enfant A terminé mais non récolté -> zombie
  ├── enfant B terminé mais non récolté -> zombie
  └── enfant C terminé mais non récolté -> zombie
```

Dans un système classique, `systemd` ou un init joue ce rôle. Dans un conteneur minimal, ce rôle peut manquer.

---

**Utiliser un init minimal.**

Pour éviter ce problème, nous utilisons parfois un init minimal comme :

```text
tini
dumb-init
```

Avec Docker, nous pouvons utiliser :

```bash
docker run --init ...
```

Docker ajoute alors un petit processus d’init comme PID 1 du conteneur.

Ce processus s’occupe notamment de transmettre les signaux et de récolter les processus enfants.

---

# 5.6. Signaux et PID 1

**Les signaux Unix.**

Les signaux permettent de demander à un processus de réagir.

Exemples :

|Signal|Rôle courant|
|---|---|
|`SIGTERM`|demander un arrêt propre|
|`SIGKILL`|tuer immédiatement|
|`SIGINT`|interruption clavier|
|`SIGHUP`|rechargement ou fermeture terminal|
|`SIGCHLD`|notification de terminaison d’un enfant|

Nous envoyons un signal avec :

```bash
kill -TERM <PID>
```

ou :

```bash
kill <PID>
```

qui envoie par défaut `SIGTERM`.

---

**PID 1 et gestion des signaux.**

Le PID 1 a un comportement particulier : certains signaux qui tueraient normalement un processus peuvent être ignorés si le processus PID 1 n’a pas explicitement prévu de gestionnaire.

Dans un conteneur, cela peut provoquer un problème :

```text
docker stop envoie SIGTERM
l’application PID 1 ne le gère pas correctement
le conteneur ne s’arrête pas proprement
Docker finit par envoyer SIGKILL
```

Nous devons donc comprendre que le rôle de PID 1 n’est pas neutre.

---

**Conséquence pour les applications.**

Une application lancée comme PID 1 doit :

- gérer `SIGTERM` ;

- arrêter proprement ses workers ;

- fermer ses sockets ;

- vider ses buffers ;

- attendre ses enfants ;

- éviter les zombies.


Si elle ne le fait pas, nous devons utiliser un init minimal ou ajuster l’entrypoint.

---

# 5.7. PID namespace et hiérarchie

**Namespaces PID imbriqués.**

Les namespaces PID peuvent être hiérarchiques.

Un namespace PID peut avoir un namespace enfant.

Un processus dans le namespace parent peut voir les processus du namespace enfant, mais l’inverse n’est pas vrai.

Exemple conceptuel :

```text
Namespace hôte
  PID 24531 -> processus visible comme PID 1 dans le conteneur

Namespace conteneur
  PID 1 -> même processus
```

Depuis l’hôte, nous voyons le processus avec son PID hôte.

Depuis le conteneur, nous le voyons avec son PID conteneur.

---

**Observer les PID avec `NSpid`.**

Dans `/proc/<PID>/status`, nous pouvons parfois voir une ligne :

```bash
grep NSpid /proc/<PID>/status
```

Exemple :

```text
NSpid:  24531  1
```

Cela signifie que le processus a plusieurs identifiants selon les niveaux de namespaces PID.

Ici :

- `24531` est le PID dans le namespace parent ;

- `1` est le PID dans le namespace enfant.


Cette ligne est très utile pour comprendre les conteneurs.

---

# 5.8. Lien avec `/proc`

**`/proc` dépend du namespace PID.**

Dans un namespace PID, `/proc` doit refléter la vue des processus de ce namespace.

Nous testons :

```bash
unshare --fork --pid --mount-proc bash
```

Puis :

```bash
ls /proc | grep -E '^[0-9]+$'
ps aux
```

Nous voyons seulement les processus visibles dans le namespace PID.

---

**Sans `--mount-proc`.**

Si nous lançons :

```bash
unshare --fork --pid bash
```

puis :

```bash
ps aux
```

la sortie peut être incohérente, car `/proc` n’a pas forcément été remonté.

C’est un piège très courant.

Nous retenons :

```text
Le namespace PID contrôle la numérotation et la visibilité.
Mais les outils comme ps dépendent de /proc.
Il faut donc que /proc corresponde au namespace PID courant.
```

---

**Comparer `/proc/1` dedans et dehors.**

Dans le namespace :

```bash
cat /proc/1/comm
readlink /proc/1/ns/pid
```

Sur l’hôte :

```bash
cat /proc/1/comm
readlink /proc/1/ns/pid
```

Nous pouvons voir deux réalités différentes.

Dans le namespace, `/proc/1` désigne le PID 1 du namespace.

Sur l’hôte, `/proc/1` désigne le PID 1 de l’hôte.

---

# 5.11. Entrer dans un namespace PID avec `nsenter`

**Depuis l’hôte vers un processus isolé.**

Si nous avons le PID hôte d’un processus dans un namespace PID, nous pouvons entrer dans ce namespace :

```bash
sudo nsenter --target <PID> --pid --fork bash
```

Mais en pratique, pour obtenir une vue cohérente, nous entrons souvent aussi dans le namespace mount :

```bash
sudo nsenter --target <PID> --mount --pid --fork bash
```

Puis :

```bash
ps aux
cat /proc/1/comm
```

---

**Entrer dans plusieurs namespaces d’un conteneur.**

Pour diagnostiquer un conteneur depuis l’hôte :

```bash
sudo nsenter --target <PID> --mount --uts --ipc --net --pid --fork bash
```

Nous obtenons une vue très proche de celle du conteneur.

Nous pouvons ensuite exécuter :

```bash
ps aux
hostname
ip addr
mount
```

C’est souvent plus puissant qu’un simple `docker exec`, notamment si l’image ne contient pas de shell ou d’outils de diagnostic.

---

# 5.12. Expérience complète guidée

**Créer un namespace PID.**

Nous lançons :

```bash
unshare --fork --pid --mount-proc bash
```

Dans ce shell :

```bash
echo "PID du shell : $$"
cat /proc/1/comm
ps -ef
readlink /proc/$$/ns/pid
```

Nous observons que notre shell est PID 1.

---

**Lancer un processus enfant.**

Dans le namespace :

```bash
sleep 1000 &
ps -ef
```

Nous pouvons voir :

```text
UID   PID  PPID  CMD
root    1     0  bash
root   12     1  sleep 1000
root   13     1  ps -ef
```

Le processus `sleep` a un PID dans le namespace.

---

**Observer depuis l’hôte.**

Dans un autre terminal sur l’hôte :

```bash
ps aux | grep sleep
```

Nous trouvons le PID hôte de `sleep`.

Puis :

```bash
grep NSpid /proc/<PID_HOTE>/status
```

Nous pouvons voir plusieurs PID :

```text
NSpid:  30142  12
```

Cela signifie que le processus est vu comme `30142` depuis l’hôte et `12` dans le namespace enfant.

---

**Nettoyage.**

Dans le shell isolé :

```bash
kill %1
exit
```

Quand le PID 1 du namespace se termine, les autres processus du namespace sont généralement terminés également.

---

# 5.13. Pièges classiques

**Oublier `--fork`.**

Commande problématique :

```bash
unshare --pid bash
```

Nous risquons de ne pas obtenir le comportement attendu.

Commande recommandée :

```bash
unshare --fork --pid --mount-proc bash
```

---

**Oublier `--mount-proc`.**

Commande incomplète :

```bash
unshare --fork --pid bash
```

Nous créons bien un namespace PID, mais les outils qui lisent `/proc` peuvent afficher une vue incorrecte.

Commande recommandée :

```bash
unshare --fork --pid --mount-proc bash
```

---

**Croire que PID 1 est un processus ordinaire.**

Dans un namespace PID, le PID 1 doit gérer correctement :

- signaux ;

- enfants ;

- zombies ;

- arrêt propre.


Une application lancée directement comme PID 1 peut se comporter différemment que lorsqu’elle est lancée sous un shell ou un superviseur.

---

**Confondre PID hôte et PID interne.**

Quand nous lisons un log ou un message d’erreur, nous devons savoir depuis quel namespace le PID est exprimé.

Un PID affiché dans le conteneur peut ne pas exister avec le même numéro sur l’hôte.

Nous utilisons :

```bash
grep NSpid /proc/<PID>/status
```

pour comprendre les correspondances.

---

# 5.15. Ce que nous devons retenir

Nous retenons les points suivants :

1. Le namespace PID isole la numérotation et la visibilité des processus.

2. Un même processus peut avoir plusieurs PID selon le namespace d’observation.

3. `/proc/<PID>/ns/pid` permet d’identifier le namespace PID d’un processus.

4. `unshare --fork --pid --mount-proc bash` est la commande pédagogique de base.

5. `--fork` est nécessaire car les enfants entrent dans le nouveau namespace PID.

6. `--mount-proc` est nécessaire pour que `/proc` reflète correctement le namespace PID.

7. Le PID 1 d’un namespace a un rôle particulier.

8. Le PID 1 doit gérer les signaux et récolter les processus enfants.

9. Une application conteneurisée lancée directement comme PID 1 peut avoir des comportements inattendus.

10. La ligne `NSpid` dans `/proc/<PID>/status` permet de voir les PID selon les namespaces.

11. Docker, Podman et Kubernetes utilisent le namespace PID pour isoler les processus.

12. Les options comme `--pid=host` ou `hostPID: true` réduisent fortement l’isolation.

13. Le namespace PID isole la vue, mais ne limite pas à lui seul le nombre de processus. Pour cela, nous utilisons les cgroups.


---

# Conclusion du chapitre 5

Nous avons étudié le namespace PID, un mécanisme central de l’isolation Linux.

Nous savons maintenant qu’un processus peut être vu avec des PID différents selon le namespace depuis lequel nous l’observons. Nous avons compris pourquoi un conteneur peut avoir son propre PID 1, pourquoi `/proc` doit être remonté dans un namespace PID, et pourquoi le processus PID 1 a des responsabilités particulières.

Nous avons aussi vu que cette isolation peut être réduite ou supprimée avec des options comme `--pid=host` ou `hostPID: true`, ce qui doit être réservé à des cas particuliers.

Dans le chapitre suivant, nous étudions le namespace mount, qui permet à un processus de voir une arborescence de montages différente de celle de l’hôte.

---

# Chapitre 6 — Namespace mount

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace mount, aussi appelé namespace de montage.

Le namespace mount permet à un processus de voir une arborescence de points de montage différente de celle de l’hôte. C’est un mécanisme fondamental des conteneurs : il permet de donner à une application l’impression qu’elle possède son propre système de fichiers, alors qu’elle partage toujours le noyau avec l’hôte.

À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace mount ;

- créer un namespace mount avec `unshare` ;

- comprendre la différence entre arborescence de fichiers et points de montage ;

- manipuler des montages isolés ;

- comprendre la propagation des montages ;

- distinguer `shared`, `private`, `slave` et `unbindable` ;

- expliquer le rôle des bind mounts ;

- comprendre la différence entre `chroot`, namespace mount et `pivot_root` ;

- comprendre comment un conteneur obtient son root filesystem ;

- identifier les pièges de sécurité liés aux montages.


---

# 6.1. Pourquoi isoler les montages ?

**Le problème de départ.**

Sur un système Linux classique, les processus voient une arborescence de fichiers commune.

Nous pouvons afficher les montages avec :

```bash
findmnt
```

ou :

```bash
mount
```

Nous voyons par exemple :

```text
/
├─ /proc
├─ /sys
├─ /dev
├─ /run
├─ /home
└─ /mnt/data
```

Si tous les processus partagent exactement la même vue des montages, il devient difficile de créer un environnement isolé.

Un conteneur a besoin d’une vue différente :

```text
/
├─ /bin
├─ /etc
├─ /lib
├─ /proc
├─ /tmp
└─ /app
```

Cette racine n’est pas forcément la vraie racine de l’hôte. C’est une racine apparente construite pour le processus isolé.

Le namespace mount permet précisément cela.

---

**Ce que le namespace mount isole.**

Le namespace mount isole la table des montages visible par un processus.

Cela signifie qu’un processus peut avoir :

- ses propres montages ;

- ses propres bind mounts ;

- son propre `/proc` ;

- son propre `/sys` ;

- son propre `/tmp` ;

- une racine apparente différente ;

- des volumes montés à des emplacements spécifiques ;

- des montages en lecture seule ou lecture-écriture selon les besoins.


Nous retenons :

```text
Le namespace mount isole la vue des points de montage.
```

Il ne copie pas automatiquement tous les fichiers. Il isole la manière dont les systèmes de fichiers sont montés et visibles.

---

# 6.2. Arborescence de fichiers et points de montage

**L’arborescence de fichiers.**

Sous Linux, tout est présenté dans une arborescence unique qui commence à :

```text
/
```

Nous pouvons naviguer avec :

```bash
cd /
ls
```

Nous voyons par exemple :

```text
bin
boot
dev
etc
home
proc
sys
tmp
usr
var
```

Mais cette arborescence est composée de plusieurs systèmes de fichiers montés à différents endroits.

---

**Le point de montage.**

Un point de montage est un répertoire sur lequel nous attachons un système de fichiers.

Exemple :

```bash
mount -t tmpfs tmpfs /tmp/test-mount
```

Ici :

```text
/tmp/test-mount
```

devient un point de montage.

Le contenu visible à cet endroit ne vient plus du répertoire ordinaire initial, mais du système de fichiers monté.

Nous pouvons vérifier :

```bash
findmnt /tmp/test-mount
```

---

**Le rôle du namespace mount.**

Sans namespace mount, un nouveau montage peut être visible par les autres processus de l’hôte.

Avec un namespace mount isolé, nous pouvons monter quelque chose dans notre environnement sans le rendre visible ailleurs, à condition de gérer correctement la propagation des montages.

C’est ce qui permet à un conteneur d’avoir ses propres montages sans polluer l’hôte.

---

# 6.3. Observer le namespace mount courant

**Avec `/proc/<PID>/ns/mnt`.**

Comme les autres namespaces, le namespace mount d’un processus est visible dans :

```bash
/proc/<PID>/ns/mnt
```

Pour notre shell courant :

```bash
readlink /proc/$$/ns/mnt
```

Exemple :

```text
mnt:[4026531841]
```

Pour le PID 1 :

```bash
readlink /proc/1/ns/mnt
```

Si les deux valeurs sont identiques, notre shell partage le même namespace mount que le PID 1.

---

**Comparer deux processus.**

Nous pouvons comparer :

```bash
echo "Shell courant : $(readlink /proc/$$/ns/mnt)"
echo "PID 1         : $(readlink /proc/1/ns/mnt)"
```

Même valeur :

```text
Shell courant : mnt:[4026531841]
PID 1         : mnt:[4026531841]
```

Les deux processus partagent le même namespace mount.

Valeurs différentes :

```text
Shell courant : mnt:[4026533000]
PID 1         : mnt:[4026531841]
```

Les deux processus ont des vues de montages différentes.

---

**Observer les montages visibles.**

Nous pouvons observer les montages du processus courant avec :

```bash
findmnt
```

ou :

```bash
cat /proc/self/mounts
```

ou encore :

```bash
cat /proc/self/mountinfo
```

Différence importante :

```text
/proc/self/mounts     → vue simplifiée des montages
/proc/self/mountinfo  → vue plus complète, avec propagation et identifiants
```

Pour l’étude du namespace mount, `mountinfo` devient très utile.

---

# 6.4. Créer un namespace mount avec `unshare`

**Première expérience.**

Nous créons un namespace mount :

```bash
unshare --mount bash
```

Dans le shell isolé, nous vérifions :

```bash
readlink /proc/$$/ns/mnt
findmnt | head
```

Dans un autre terminal, nous pouvons comparer :

```bash
readlink /proc/<PID_DU_SHELL_ISOLE>/ns/mnt
readlink /proc/1/ns/mnt
```

Nous devons voir que le shell isolé possède un namespace mount différent.

---

**Monter un tmpfs dans le namespace.**

Dans le shell isolé :

```bash
mkdir -p /tmp/ns-mount-demo
mount -t tmpfs tmpfs /tmp/ns-mount-demo
findmnt /tmp/ns-mount-demo
```

Nous créons un fichier :

```bash
echo "visible dans le namespace" > /tmp/ns-mount-demo/test.txt
cat /tmp/ns-mount-demo/test.txt
```

Nous avons monté un système de fichiers temporaire dans le namespace courant.

---

**Vérifier depuis l’hôte.**

Depuis un autre terminal sur l’hôte :

```bash
findmnt /tmp/ns-mount-demo
ls /tmp/ns-mount-demo
```

Selon la propagation des montages de votre système, le montage peut être visible ou non depuis l’hôte.

C’est un point essentiel : créer un namespace mount ne suffit pas toujours à éviter la propagation. Il faut comprendre la propagation des montages.

---

# 6.5. Le piège de la propagation des montages

**Pourquoi ce piège existe ?.**

Sur beaucoup de distributions modernes, certains montages sont configurés en mode `shared`.

Cela signifie qu’un montage créé dans un namespace peut être propagé vers un autre namespace qui partage le même groupe de propagation.

Nous pourrions croire :

```text
J’ai créé un namespace mount, donc mes montages sont automatiquement invisibles ailleurs.
```

Ce n’est pas toujours vrai.

La propagation des montages peut transmettre certains événements de montage entre namespaces.

---

**Vérifier la propagation.**

Nous pouvons inspecter :

```bash
findmnt -o TARGET,PROPAGATION /
```

Sortie possible :

```text
TARGET PROPAGATION
/      shared
```

ou :

```text
TARGET PROPAGATION
/      private
```

Nous pouvons aussi regarder :

```bash
cat /proc/self/mountinfo | head
```

Les mentions comme :

```text
shared:1
master:2
```

indiquent des relations de propagation.

---

**Rendre les montages privés.**

Dans un namespace de test, nous pouvons rendre l’arborescence privée :

```bash
mount --make-rprivate /
```

Ensuite, les nouveaux montages faits dans ce namespace ne se propagent normalement plus vers l’extérieur.

Commande pédagogique recommandée :

```bash
unshare --mount bash
mount --make-rprivate /
```

Puis :

```bash
mkdir -p /tmp/ns-mount-demo
mount -t tmpfs tmpfs /tmp/ns-mount-demo
findmnt /tmp/ns-mount-demo
```

Depuis l’hôte, le montage ne doit normalement pas apparaître.

Nous retenons :

```text
Pour un TP propre sur les namespaces mount, nous rendons souvent / récursivement privé avec mount --make-rprivate /.
```

---

# 6.6. Les types de propagation

**Vue d’ensemble.**

Linux définit plusieurs modes de propagation des montages :

```text
shared
private
slave
unbindable
```

Ces modes déterminent comment les événements de montage sont propagés entre namespaces.

C’est une notion avancée, mais indispensable pour comprendre Docker, Kubernetes et systemd.

---

**`shared`.**

Un montage `shared` propage les événements de montage à d’autres montages du même groupe de propagation.

Si nous montons un système de fichiers sous un point `shared`, ce montage peut apparaître dans d’autres namespaces liés.

Nous pouvons voir ce mode avec :

```bash
findmnt -o TARGET,PROPAGATION /
```

Sortie :

```text
/ shared
```

Ce mode est pratique pour certains usages système, mais peut surprendre dans un contexte d’isolation.

---

**`private`.**

Un montage `private` ne propage pas les événements et ne reçoit pas les événements des autres.

Nous pouvons rendre un montage privé :

```bash
mount --make-private /chemin
```

Ou récursivement :

```bash
mount --make-rprivate /chemin
```

Pour un conteneur, nous voulons souvent une base privée afin d’éviter que les montages internes fuient vers l’hôte.

---

**`slave`.**

Un montage `slave` reçoit les événements de son maître, mais ne renvoie pas ses propres événements vers lui.

Nous pouvons le voir comme une propagation à sens unique.

Cela peut être utile lorsqu’un environnement isolé doit voir certains montages de l’hôte, mais ne doit pas propager ses propres montages vers l’hôte.

Commande :

```bash
mount --make-slave /chemin
```

Récursif :

```bash
mount --make-rslave /chemin
```

---

**`unbindable`.**

Un montage `unbindable` ne peut pas être bind-mounté.

C’est utile pour éviter certaines copies ou propagations de montages sensibles.

Commande :

```bash
mount --make-unbindable /chemin
```

Récursif :

```bash
mount --make-runbindable /chemin
```

Ce mode est plus spécialisé, mais nous devons savoir qu’il existe.

---

**Synthèse.**

|Mode|Reçoit des événements|Propage ses événements|Usage typique|
|---|--:|--:|---|
|`shared`|oui|oui|propagation entre environnements liés|
|`private`|non|non|isolation forte des montages|
|`slave`|oui|non|recevoir de l’hôte sans propager vers lui|
|`unbindable`|non|non|empêcher certains bind mounts|

Nous retenons surtout :

```text
shared peut propager.
private isole.
slave reçoit sans renvoyer.
unbindable empêche certains bind mounts.
```

---

# 6.7. Bind mounts

**Définition.**

Un bind mount permet de remonter un chemin existant à un autre endroit de l’arborescence.

Exemple :

```bash
mkdir -p /tmp/source /tmp/destination
echo "test" > /tmp/source/fichier.txt

mount --bind /tmp/source /tmp/destination
ls /tmp/destination
```

Nous voyons :

```text
fichier.txt
```

Le même contenu est visible à deux emplacements.

---

**Utilité dans les conteneurs.**

Les bind mounts sont essentiels pour les conteneurs.

Quand nous faisons avec Docker :

```bash
docker run -v /home/user/app:/app image
```

Docker crée un bind mount entre :

```text
/home/user/app sur l’hôte
/app dans le conteneur
```

Cela permet de rendre un dossier de l’hôte visible dans le conteneur.

---

**Bind mount en lecture seule.**

Nous pouvons remonter un bind mount en lecture seule.

Exemple :

```bash
mount --bind /tmp/source /tmp/destination
mount -o remount,ro,bind /tmp/destination
```

Nous vérifions :

```bash
findmnt /tmp/destination
```

Dans les conteneurs, cela correspond à des montages de type :

```bash
docker run -v /host/path:/container/path:ro image
```

Le mode lecture seule réduit les risques de modification de l’hôte.

---

**Risques des bind mounts.**

Un bind mount peut exposer des chemins sensibles.

Exemples dangereux :

```bash
docker run -v /:/host image
docker run -v /var/run/docker.sock:/var/run/docker.sock image
docker run -v /proc:/host/proc image
docker run -v /sys:/host/sys image
```

Ces montages peuvent donner au conteneur une visibilité ou un contrôle excessif sur l’hôte.

Nous retenons :

```text
Un bind mount est une passerelle directe entre deux vues de l’arborescence.
Il doit être choisi avec prudence.
```

---

# 6.8. Monter `/proc` dans un namespace mount

**Pourquoi monter `/proc` ?.**

Dans un namespace PID, les outils comme `ps` dépendent de `/proc`.

Si nous construisons un environnement isolé, nous devons souvent monter un `/proc` adapté.

Exemple :

```bash
unshare --fork --pid --mount bash
mount -t proc proc /proc
ps aux
```

Mais attention : monter directement sur `/proc` dans un environnement non préparé peut être risqué ou perturber la vue du shell.

C’est pourquoi `unshare` propose :

```bash
unshare --fork --pid --mount-proc bash
```

---

**`/proc` dans un conteneur.**

Dans un conteneur, `/proc` est généralement monté pour refléter le namespace PID du conteneur.

Ainsi, dans le conteneur :

```bash
ps aux
ls /proc
cat /proc/1/comm
```

nous voyons la vue des processus du conteneur, pas forcément celle de l’hôte.

C’est une combinaison entre :

```text
namespace PID
namespace mount
montage de procfs
```

---

**Risques liés à `/proc`.**

Monter le `/proc` de l’hôte dans un conteneur est dangereux.

Exemple à éviter :

```bash
docker run -v /proc:/host/proc image
```

Selon les droits du conteneur, cela peut exposer :

- processus de l’hôte ;

- informations sensibles ;

- variables d’environnement ;

- descripteurs ;

- paramètres noyau ;

- informations réseau.


Nous devons toujours distinguer :

```text
/proc du conteneur
/proc de l’hôte
```

---

# 6.10. `chroot` vs namespace mount

**Qu’est-ce que `chroot` ?.**

`chroot` change la racine apparente d’un processus.

Exemple :

```bash
chroot /chemin/nouvelle-racine /bin/sh
```

Le processus voit alors :

```text
/chemin/nouvelle-racine
```

comme `/`.

Historiquement, `chroot` a été utilisé pour isoler partiellement des environnements.

---

**Limites de `chroot`.**

`chroot` seul n’est pas un mécanisme d’isolation complet.

Il n’isole pas :

- les PID ;

- le réseau ;

- les IPC ;

- le hostname ;

- les cgroups ;

- les capabilities ;

- les montages au sens complet ;

- le noyau.


De plus, avec des privilèges suffisants, il existe des manières de s’échapper d’un `chroot` mal configuré.

Nous retenons :

```text
chroot change une racine apparente.
Il ne constitue pas à lui seul un conteneur sécurisé.
```

---

**Namespace mount et `chroot`.**

Un namespace mount est plus puissant car il permet de construire une table de montages propre.

Nous pouvons combiner :

```text
namespace mount
bind mounts
montage de /proc
montage de /dev minimal
chroot ou pivot_root
```

C’est cette combinaison qui permet de construire un environnement proche d’un conteneur.

---

# 6.11. `pivot_root`

**Rôle de `pivot_root`.**

`pivot_root` permet de changer la racine d’un processus en remplaçant l’ancienne racine par une nouvelle, tout en plaçant l’ancienne racine dans un sous-répertoire de la nouvelle.

C’est un mécanisme utilisé dans la construction de systèmes isolés et de conteneurs.

Schéma conceptuel :

```text
Avant :
/                  → racine de l’hôte
/newroot           → futur rootfs

Après pivot_root :
/                  → ancien /newroot
/oldroot           → ancienne racine de l’hôte
```

Ensuite, on peut démonter `/oldroot` pour couper l’accès à l’ancienne racine.

---

**Pourquoi ne pas utiliser seulement `chroot` ?.**

`pivot_root` est plus adapté à la construction d’un environnement complet car il agit au niveau des montages.

Il permet de remplacer réellement la racine dans le namespace mount.

Les runtimes de conteneurs utilisent plutôt des mécanismes de ce type, avec une préparation soigneuse des montages.

---

**Attention.**

`pivot_root` est plus complexe à manipuler que `chroot`.

Il impose des contraintes :

- nouvelle racine sur un point de montage ;

- ancien root placé sous la nouvelle racine ;

- gestion correcte des répertoires courants ;

- démontage propre de l’ancien root ;

- permissions nécessaires.


Nous l’étudions surtout pour comprendre les conteneurs, pas comme commande de base à utiliser quotidiennement.

---

# 6.12. Root filesystem d’un conteneur

**Qu’est-ce qu’un root filesystem ?.**

Le root filesystem, ou rootfs, est l’arborescence de fichiers vue comme `/` par le conteneur.

Il contient par exemple :

```text
/bin
/etc
/lib
/usr
/var
/tmp
/app
```

Dans Docker, il provient de l’image du conteneur.

Exemple :

```bash
docker run --rm alpine ls /
```

Sortie typique :

```text
bin
dev
etc
home
lib
media
mnt
proc
root
run
sbin
sys
tmp
usr
var
```

Ce n’est pas la racine de l’hôte. C’est la racine de l’image Alpine, montée dans un namespace.

---

**Images en couches.**

Docker et les runtimes modernes utilisent souvent des systèmes de fichiers en couches.

Exemples :

```text
overlayfs
aufs anciennement
btrfs
zfs
devicemapper anciennement
```

Le principe courant avec OverlayFS :

```text
couches image en lecture seule
+
couche writable du conteneur
=
rootfs final visible dans le conteneur
```

Le namespace mount permet ensuite de présenter ce rootfs comme `/` au processus du conteneur.

---

**Volumes et bind mounts.**

Les volumes Docker ou Kubernetes sont ajoutés comme montages supplémentaires dans le namespace mount du conteneur.

Exemples :

```bash
docker run -v /data:/data image
```

ou en Kubernetes :

```yaml
volumeMounts:
  - name: data
    mountPath: /data
```

Le conteneur voit `/data`, mais la source peut venir :

- d’un dossier de l’hôte ;

- d’un volume Docker ;

- d’un volume Kubernetes ;

- d’un PVC ;

- d’un secret ;

- d’une ConfigMap ;

- d’un tmpfs ;

- d’un stockage réseau.


---

# 6.13. Construire une mini-isolation de fichiers

**Préparer un rootfs minimal.**

Pour un TP avancé, nous pouvons utiliser un rootfs minimal, par exemple avec BusyBox.

Sur Debian/Ubuntu, une approche simple est d’utiliser un répertoire de test :

```bash
mkdir -p /tmp/rootfs-demo/{bin,proc,sys,dev,tmp}
```

Nous pouvons copier un binaire statique comme BusyBox si disponible :

```bash
cp /bin/busybox /tmp/rootfs-demo/bin/
```

Puis créer quelques liens :

```bash
ln -s busybox /tmp/rootfs-demo/bin/sh
ln -s busybox /tmp/rootfs-demo/bin/ls
ln -s busybox /tmp/rootfs-demo/bin/mount
ln -s busybox /tmp/rootfs-demo/bin/ps
```

Cette partie dépend fortement de la distribution et des binaires disponibles.

---

**Entrer avec un namespace mount et `chroot`.**

Exemple conceptuel :

```bash
sudo unshare --mount --uts --fork bash
mount --make-rprivate /
chroot /tmp/rootfs-demo /bin/sh
```

Dans ce shell, nous voyons une racine différente.

Mais ce n’est pas encore un conteneur complet :

- pas forcément de namespace PID ;

- pas forcément de réseau isolé ;

- `/proc` pas forcément monté ;

- `/dev` minimal absent ;

- pas de cgroups ;

- pas de politique de sécurité.


---

**Ajouter `/proc`.**

Dans un environnement isolé bien préparé, nous pouvons monter :

```bash
mount -t proc proc /proc
```

Puis :

```bash
ps
```

Mais nous devons être prudents : monter `/proc` doit être fait dans le bon namespace, avec le bon PID namespace si nous voulons une vue cohérente.

---

# 6.16. Sécurité du namespace mount

**Le namespace mount ne suffit pas.**

Un namespace mount peut isoler la vue des montages, mais la sécurité dépend de ce qui est monté.

Si nous montons toute la racine de l’hôte dans le conteneur :

```bash
docker run -v /:/host image
```

l’isolation du rootfs est largement compromise.

Même si le processus a une racine différente, il peut accéder à `/host`.

---

**Montages sensibles.**

Chemins sensibles :

```text
/
/proc
/sys
/dev
/run
/var/run/docker.sock
/etc
/root
/home
/var/lib/docker
/var/lib/kubelet
```

Nous ne les montons pas dans un conteneur sans justification forte.

---

**Lecture seule.**

Quand un montage est nécessaire, nous privilégions souvent la lecture seule :

```bash
docker run -v /host/path:/container/path:ro image
```

En Kubernetes :

```yaml
volumeMounts:
  - name: data
    mountPath: /data
    readOnly: true
```

Cela ne supprime pas tous les risques, mais réduit les possibilités de modification.

---

**Root filesystem en lecture seule.**

Pour durcir un conteneur, nous pouvons rendre le root filesystem en lecture seule.

Docker :

```bash
docker run --read-only image
```

Kubernetes :

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

Il faut alors prévoir des emplacements temporaires explicitement montés, par exemple `/tmp` ou `/run`.

---

# 6.17. Diagnostic avec le namespace mount

**Quel montage voit ce processus ?.**

Pour un PID donné :

```bash
cat /proc/<PID>/mountinfo
```

ou :

```bash
sudo nsenter --target <PID> --mount findmnt
```

Nous pouvons alors comprendre :

- quel rootfs est vu ;

- quels volumes sont montés ;

- quels chemins viennent de l’hôte ;

- quels montages sont en lecture seule ;

- si `/proc`, `/sys`, `/dev` sont présents ;

- quelles propagations sont configurées.


---

**Pourquoi mon fichier n’est-il pas visible ?.**

Question fréquente :

```text
J’ai créé un fichier sur l’hôte, pourquoi le conteneur ne le voit-il pas ?
```

Causes possibles :

- le chemin n’est pas monté dans le conteneur ;

- un autre montage masque le répertoire ;

- le conteneur a un rootfs différent ;

- le fichier est créé dans une couche writable différente ;

- mauvais chemin hôte ;

- permissions insuffisantes ;

- montage en lecture seule ;

- propagation non adaptée.


Le namespace mount est souvent au cœur de ce diagnostic.

---

**Pourquoi mon montage apparaît sur l’hôte ?.**

Cause fréquente :

```text
propagation shared
```

Nous vérifions :

```bash
findmnt -o TARGET,PROPAGATION /
```

ou dans le namespace concerné :

```bash
findmnt -o TARGET,PROPAGATION
```

Nous pouvons corriger dans un TP avec :

```bash
mount --make-rprivate /
```

Dans un système réel, nous devons comprendre l’impact avant de changer la propagation.

---

# 6.18. Expérience complète guidée

**Créer un namespace mount isolé.**

Nous lançons :

```bash
unshare --mount bash
```

Dans le shell :

```bash
echo "Namespace mount : $(readlink /proc/$$/ns/mnt)"
findmnt -o TARGET,PROPAGATION / | cat
```

Nous rendons la propagation privée pour le TP :

```bash
mount --make-rprivate /
```

---

**Créer un montage visible seulement dans le namespace.**

Dans le shell isolé :

```bash
mkdir -p /tmp/mnt-ns-demo
mount -t tmpfs tmpfs /tmp/mnt-ns-demo
echo "bonjour depuis le namespace mount" > /tmp/mnt-ns-demo/message.txt
cat /tmp/mnt-ns-demo/message.txt
findmnt /tmp/mnt-ns-demo
```

Dans un autre terminal sur l’hôte :

```bash
findmnt /tmp/mnt-ns-demo
ls /tmp/mnt-ns-demo
```

Normalement, le montage ne doit pas être visible comme montage depuis l’hôte, si la propagation a été rendue privée correctement.

---

**Observer depuis l’hôte avec `nsenter`.**

Nous récupérons le PID du shell isolé :

```bash
echo $$
```

Depuis l’hôte :

```bash
sudo nsenter --target <PID> --mount findmnt /tmp/mnt-ns-demo
sudo nsenter --target <PID> --mount cat /tmp/mnt-ns-demo/message.txt
```

Nous voyons le montage depuis le namespace du processus.

---

**Nettoyage.**

Dans le shell isolé :

```bash
umount /tmp/mnt-ns-demo
exit
```

Si nécessaire depuis l’hôte :

```bash
sudo umount /tmp/mnt-ns-demo 2>/dev/null
```

---

# 6.19. Pièges classiques

**Croire que `unshare --mount` isole tout automatiquement.**

`unshare --mount` crée un nouveau namespace mount, mais la propagation peut encore produire des effets inattendus.

Pour un TP propre :

```bash
mount --make-rprivate /
```

---

**Confondre fichiers et montages.**

Deux processus peuvent voir le même fichier sur disque, mais avec des montages différents.

Inversement, un même chemin peut pointer vers des contenus différents selon le namespace mount.

Exemple :

```text
/tmp/data dans le namespace A → tmpfs A
/tmp/data dans le namespace B → répertoire réel de l’hôte
```

Même chemin, réalité différente.

---

**Monter par-dessus un répertoire existant.**

Si nous montons un système de fichiers sur un répertoire qui contient déjà des fichiers, ces fichiers ne sont pas supprimés, mais ils sont masqués tant que le montage est actif.

Exemple :

```bash
mkdir -p /tmp/demo
echo "avant" > /tmp/demo/fichier.txt
mount -t tmpfs tmpfs /tmp/demo
ls /tmp/demo
```

Le fichier `fichier.txt` semble disparaître, mais il réapparaît après :

```bash
umount /tmp/demo
ls /tmp/demo
```

Nous devons comprendre cette notion de masquage.

---

**Confondre `chroot` et conteneur.**

`chroot` ne suffit pas à créer un conteneur.

Il faut au minimum réfléchir à :

- namespace mount ;

- namespace PID ;

- namespace UTS ;

- namespace réseau ;

- user namespace ;

- cgroups ;

- capabilities ;

- `/proc`, `/sys`, `/dev` ;

- sécurité des montages.


---

**Exposer le socket Docker.**

Le montage suivant est extrêmement sensible :

```bash
-v /var/run/docker.sock:/var/run/docker.sock
```

Un processus dans le conteneur peut alors contrôler Docker sur l’hôte, ce qui revient souvent à obtenir un pouvoir très important sur la machine.

Nous évitons ce montage sauf besoin très précis et parfaitement compris.

---

# 6.21. Ce que nous devons retenir

Nous retenons les points suivants :

1. Le namespace mount isole la vue des points de montage.

2. Il permet à un processus de voir une arborescence de montages différente de celle de l’hôte.

3. Le namespace mount est visible avec `/proc/<PID>/ns/mnt`.

4. Les montages visibles sont consultables avec `findmnt`, `/proc/self/mounts` et `/proc/self/mountinfo`.

5. `unshare --mount bash` crée un nouveau namespace mount.

6. La propagation des montages peut produire des effets inattendus.

7. `mount --make-rprivate /` est souvent utile en TP pour éviter les propagations.

8. Les modes principaux sont `shared`, `private`, `slave` et `unbindable`.

9. Un bind mount remonte un chemin existant ailleurs dans l’arborescence.

10. Les bind mounts sont essentiels pour les volumes Docker et Kubernetes.

11. Monter `/proc` correctement est indispensable dans un namespace PID.

12. Monter `/proc`, `/sys` ou `/dev` de l’hôte dans un conteneur peut être dangereux.

13. `chroot` change la racine apparente, mais ne suffit pas à créer une isolation complète.

14. `pivot_root` est plus adapté à la construction d’un vrai rootfs isolé.

15. Le root filesystem d’un conteneur est présenté grâce au namespace mount.

16. La sécurité dépend autant de ce qui est monté que du namespace lui-même.


---

# Conclusion du chapitre 6

Nous avons étudié le namespace mount, qui est l’un des mécanismes fondamentaux des conteneurs Linux.

Nous savons maintenant qu’il permet d’isoler la vue des montages, de présenter un root filesystem spécifique à un processus, de monter `/proc`, `/sys`, `/dev`, des volumes ou des bind mounts, et de construire une arborescence différente de celle de l’hôte.

Nous avons aussi vu que ce namespace demande une grande prudence. La propagation des montages peut surprendre, les bind mounts peuvent exposer l’hôte, et des montages sensibles comme `/proc`, `/sys`, `/dev` ou le socket Docker peuvent réduire fortement l’isolation.

Nous retenons surtout que le namespace mount ne se limite pas à “changer des fichiers visibles”. Il définit une partie essentielle de la frontière entre l’hôte et l’environnement isolé.

Dans le chapitre suivant, nous étudions le namespace network, qui permet à un processus d’avoir sa propre pile réseau, ses propres interfaces, ses routes, ses ports et ses sockets.

---

## Complément 2026 — nouvelle API mount et idmapped mounts

Les mount namespaces ne se limitent plus à l'ancienne séquence `mount()` / bind mount / `pivot_root()`. Le noyau dispose d'une API de montage plus structurée (`open_tree()`, `move_mount()`, `fsopen()`, `fsmount()`, `mount_setattr()`), utilisée par des outils modernes pour manipuler des arbres de montage avec moins de courses.

Les **idmapped mounts** sont particulièrement importants pour les conteneurs rootless. Au lieu de réécrire récursivement le propriétaire de milliers de fichiers, un montage peut présenter des UID/GID remappés dans un contexte donné. Le stockage reste le même sur disque ; c'est l'interprétation des identités au niveau du montage qui change. Cette technique complète les user namespaces et résout une partie des difficultés historiques de partage de volumes avec des UID décalés.

---

# Chapitre 7 — Namespace network

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace network, ou namespace réseau.

Le namespace réseau permet à un processus de disposer de sa propre pile réseau. Cela signifie qu’il peut avoir ses propres interfaces, ses propres adresses IP, ses propres routes, ses propres sockets, ses propres ports en écoute et ses propres règles de filtrage réseau.

C’est un mécanisme fondamental des conteneurs. Sans namespace réseau, les conteneurs partageraient directement les interfaces et les ports de l’hôte.

À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace réseau ;

- créer un namespace réseau avec `unshare` ;

- créer un namespace réseau nommé avec `ip netns` ;

- comprendre pourquoi l’interface `lo` est isolée ;

- connecter un namespace réseau à l’hôte avec une paire `veth` ;

- configurer des adresses IP et des routes ;

- comprendre le rôle d’un bridge Linux ;

- comprendre le principe du NAT ;

- faire le lien avec Docker bridge ;

- faire le lien avec Kubernetes et les plugins CNI ;

- identifier les risques de sécurité liés au partage du réseau de l’hôte.


---

# 7.1. Pourquoi isoler le réseau ?

**Le problème de départ.**

Sur une machine Linux classique, les processus utilisent la pile réseau de l’hôte.

Ils partagent :

```text
interfaces réseau
adresses IP
routes
ports en écoute
sockets
règles de pare-feu
table ARP
configuration IPv6
```

Cela signifie que deux processus qui veulent écouter sur le même port de la même adresse ne peuvent pas le faire en même temps.

Exemple :

```bash
python3 -m http.server 8080
```

Si un autre processus essaie d’écouter aussi sur `0.0.0.0:8080`, il obtient généralement une erreur :

```text
Address already in use
```

Avec des namespaces réseau séparés, deux processus peuvent écouter sur le même port, car ils n’appartiennent pas à la même pile réseau.

---

**Ce que nous voulons obtenir.**

Dans un environnement conteneurisé, nous voulons souvent que chaque conteneur ait :

- ses propres interfaces ;

- sa propre adresse IP ;

- sa propre table de routage ;

- ses propres ports ;

- ses propres sockets ;

- ses propres règles réseau ;

- éventuellement sa propre pile loopback.


Ainsi, un conteneur peut écouter sur le port `80` sans empêcher un autre conteneur d’écouter lui aussi sur son propre port `80`.

Nous obtenons cette isolation grâce au namespace network.

---

# 7.2. Qu’est-ce que le namespace network ?

**Définition.**

Un namespace network isole la pile réseau visible par un groupe de processus.

Il isole notamment :

```text
interfaces réseau
adresses IP
routes
sockets TCP/UDP
ports en écoute
règles netfilter/nftables selon contexte
table ARP/voisinage
/proc/net
/sys/class/net
```

Un processus dans un namespace réseau ne voit pas forcément les interfaces réseau de l’hôte.

Il voit seulement les interfaces qui appartiennent à son namespace.

---

**Exemple conceptuel.**

Sur l’hôte, nous avons :

```text
lo
eth0
docker0
wlan0
```

Dans un namespace réseau isolé, nous pouvons avoir seulement :

```text
lo
```

Puis, si nous ajoutons une interface virtuelle, nous pouvons avoir :

```text
lo
veth-ns
```

Ce namespace a alors sa propre mini-pile réseau.

---

**Même port, namespaces différents.**

Deux processus peuvent écouter sur le même port s’ils sont dans deux namespaces réseau différents.

Exemple conceptuel :

```text
Namespace réseau A :
  nginx écoute sur 0.0.0.0:80

Namespace réseau B :
  nginx écoute aussi sur 0.0.0.0:80
```

Il n’y a pas de conflit, car les sockets appartiennent à deux piles réseau séparées.

C’est l’un des principes qui rendent les conteneurs pratiques.

---

# 7.3. Observer le namespace réseau courant

**Avec `/proc/<PID>/ns/net`.**

Nous pouvons observer le namespace réseau de notre shell courant :

```bash
readlink /proc/$$/ns/net
```

Exemple :

```text
net:[4026531840]
```

Nous pouvons comparer avec le PID 1 :

```bash
readlink /proc/1/ns/net
```

Si les valeurs sont identiques, notre shell partage le namespace réseau du PID 1.

---

**Observer les interfaces.**

Nous affichons les interfaces réseau visibles :

```bash
ip addr
```

ou :

```bash
ip link
```

Nous pouvons aussi regarder :

```bash
ls /sys/class/net
```

et :

```bash
cat /proc/net/dev
```

Ces chemins dépendent du namespace réseau courant.

Dans un namespace réseau différent, leur contenu peut changer.

---

**Observer les routes.**

Nous affichons la table de routage :

```bash
ip route
```

Dans le namespace réseau de l’hôte, nous voyons souvent une route par défaut :

```text
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.10
```

Dans un namespace réseau nouvellement créé, nous pouvons ne voir aucune route.

---

**Observer les sockets.**

Nous pouvons afficher les sockets TCP et UDP :

```bash
ss -tulpen
```

ou :

```bash
ss -tan
```

Le résultat dépend du namespace réseau courant.

Un port en écoute dans l’hôte n’apparaît pas forcément dans un namespace réseau isolé.

---

# 7.4. Créer un namespace réseau avec `unshare`

**Première expérience.**

Nous créons un nouveau namespace réseau :

```bash
unshare --net bash
```

Dans ce shell, nous observons :

```bash
readlink /proc/$$/ns/net
ip addr
ip route
ss -tulpen
```

Nous constatons généralement que :

- le namespace réseau est différent ;

- seule l’interface `lo` existe ;

- `lo` peut être désactivée ;

- aucune route par défaut n’est configurée ;

- aucune connexion réseau extérieure n’est disponible.


---

**Activer l’interface loopback.**

Dans un namespace réseau neuf, l’interface loopback existe, mais elle peut être désactivée.

Nous vérifions :

```bash
ip link
```

Nous pouvons voir :

```text
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
```

Nous l’activons :

```bash
ip link set lo up
```

Puis :

```bash
ip addr
```

Nous voyons généralement :

```text
lo    127.0.0.1/8
```

Le loopback est maintenant actif dans ce namespace.

---

**Tester le loopback.**

Nous pouvons lancer un serveur local :

```bash
python3 -m http.server 8080 &
```

Puis tester :

```bash
curl http://127.0.0.1:8080
```

Cela fonctionne dans le namespace si `lo` est actif.

Mais depuis l’hôte, ce serveur n’est pas accessible directement sur `127.0.0.1:8080`, car le loopback du namespace est séparé du loopback de l’hôte.

Nous retenons :

```text
Chaque namespace réseau possède son propre loopback.
127.0.0.1 ne désigne pas toujours la même pile réseau.
```

---

# 7.5. Namespace réseau nommé avec `ip netns`

**Pourquoi utiliser `ip netns` ?.**

`unshare --net` est pratique pour lancer un shell isolé.

Mais pour construire des laboratoires réseau, nous préférons souvent créer des namespaces réseau nommés.

Nous utilisons :

```bash
sudo ip netns add ns1
```

Puis :

```bash
ip netns list
```

Sortie :

```text
ns1
```

---

**Exécuter une commande dans un namespace nommé.**

Nous exécutons :

```bash
sudo ip netns exec ns1 ip addr
```

Nous pouvons activer `lo` :

```bash
sudo ip netns exec ns1 ip link set lo up
```

Puis vérifier :

```bash
sudo ip netns exec ns1 ip addr
```

---

**Où est stockée la référence ?.**

Les namespaces réseau nommés sont généralement référencés dans :

```bash
/run/netns
```

Nous pouvons vérifier :

```bash
ls -l /run/netns
```

Le fichier `ns1` n’est pas un fichier ordinaire au sens classique. Il sert de référence vers le namespace réseau.

Tant que cette référence existe, le namespace peut rester vivant même sans processus permanent à l’intérieur.

---

**Supprimer le namespace.**

Nous supprimons :

```bash
sudo ip netns delete ns1
```

Puis :

```bash
ip netns list
```

Le namespace disparaît si plus aucune référence ne le maintient vivant.

---

# 7.6. Connecter un namespace réseau à l’hôte avec une paire `veth`

**Principe d’une paire `veth`.**

Une paire `veth` fonctionne comme un câble Ethernet virtuel.

Elle possède deux extrémités :

```text
veth-host  <── lien virtuel ──>  veth-ns
```

Ce qui entre par une extrémité ressort par l’autre.

Nous plaçons une extrémité dans le namespace de l’hôte et l’autre dans le namespace isolé.

---

**Créer le namespace.**

Nous créons un namespace réseau nommé :

```bash
sudo ip netns add ns1
```

Nous activons le loopback :

```bash
sudo ip netns exec ns1 ip link set lo up
```

---

**Créer la paire `veth`.**

Nous créons la paire :

```bash
sudo ip link add veth-host type veth peer name veth-ns
```

Nous vérifions côté hôte :

```bash
ip link show veth-host
ip link show veth-ns
```

Pour l’instant, les deux interfaces sont dans le namespace réseau de l’hôte.

---

**Déplacer une extrémité dans le namespace.**

Nous déplaçons `veth-ns` dans `ns1` :

```bash
sudo ip link set veth-ns netns ns1
```

Maintenant :

- `veth-host` reste côté hôte ;

- `veth-ns` est dans le namespace `ns1`.


Nous vérifions :

```bash
ip link show veth-host
sudo ip netns exec ns1 ip link show veth-ns
```

---

**Configurer les adresses IP.**

Côté hôte :

```bash
sudo ip addr add 10.0.0.1/24 dev veth-host
sudo ip link set veth-host up
```

Côté namespace :

```bash
sudo ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns
sudo ip netns exec ns1 ip link set veth-ns up
```

Nous vérifions :

```bash
ip addr show veth-host
sudo ip netns exec ns1 ip addr show veth-ns
```

---

**Tester la connectivité.**

Depuis l’hôte :

```bash
ping -c 3 10.0.0.2
```

Depuis le namespace :

```bash
sudo ip netns exec ns1 ping -c 3 10.0.0.1
```

Si tout est correct, les deux côtés communiquent.

Nous avons créé un lien réseau entre l’hôte et un namespace isolé.

---

**Nettoyage.**

Nous supprimons le namespace :

```bash
sudo ip netns delete ns1
```

Puis, si l’interface côté hôte existe encore :

```bash
sudo ip link delete veth-host 2>/dev/null
```

Quand nous supprimons une extrémité d’une paire `veth`, l’autre disparaît généralement aussi.

---

# 7.7. Ajouter une route par défaut

**Situation actuelle.**

Après la configuration précédente, le namespace `ns1` peut joindre l’hôte via `10.0.0.1`.

Mais il ne sait pas forcément joindre Internet.

Dans `ns1`, nous vérifions :

```bash
sudo ip netns exec ns1 ip route
```

Nous voyons probablement :

```text
10.0.0.0/24 dev veth-ns proto kernel scope link src 10.0.0.2
```

Il manque une route par défaut.

---

**Ajouter une route par défaut.**

Nous ajoutons dans `ns1` :

```bash
sudo ip netns exec ns1 ip route add default via 10.0.0.1
```

Nous vérifions :

```bash
sudo ip netns exec ns1 ip route
```

Nous voyons :

```text
default via 10.0.0.1 dev veth-ns
10.0.0.0/24 dev veth-ns proto kernel scope link src 10.0.0.2
```

Le namespace envoie maintenant son trafic externe vers l’hôte.

---

**Pourquoi cela ne suffit pas toujours ?.**

Même avec une route par défaut, l’accès Internet peut ne pas fonctionner.

Il faut aussi que l’hôte :

- accepte de router les paquets ;

- fasse éventuellement du NAT ;

- autorise le trafic dans son pare-feu ;

- ait une route retour ou traduise les adresses.


Nous devons donc configurer le forwarding et le NAT si nous voulons sortir vers Internet.

---

# 7.8. Activer le forwarding IPv4

**Lire l’état actuel.**

Sur l’hôte :

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Valeurs :

```text
0 → forwarding désactivé
1 → forwarding activé
```

Nous pouvons aussi utiliser :

```bash
sysctl net.ipv4.ip_forward
```

---

**Activation temporaire.**

Pour activer temporairement :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

ou :

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

Cette modification est active immédiatement, mais elle n’est pas forcément persistante après redémarrage.

---

**Persistance.**

Pour rendre la configuration persistante, nous pouvons créer un fichier :

```bash
sudo nano /etc/sysctl.d/99-forwarding.conf
```

Contenu :

```conf
net.ipv4.ip_forward = 1
```

Puis appliquer :

```bash
sudo sysctl --system
```

Dans un TP, nous pouvons nous contenter d’une activation temporaire.

---

# 7.9. NAT avec nftables ou iptables

**Pourquoi avons-nous besoin de NAT ?.**

Notre namespace `ns1` utilise l’adresse :

```text
10.0.0.2
```

Si nous envoyons un paquet vers Internet, la machine distante ne sait probablement pas retourner vers `10.0.0.2`, car c’est une adresse privée de notre laboratoire local.

Nous avons donc besoin de traduire l’adresse source en adresse de l’hôte.

C’est le rôle du NAT, souvent sous forme de masquerading.

---

**Exemple avec iptables.**

Supposons que l’interface de sortie de l’hôte soit `eth0`.

Nous pouvons faire :

```bash
sudo iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
```

Puis dans le namespace :

```bash
sudo ip netns exec ns1 ping -c 3 1.1.1.1
```

Si la route, le forwarding et le NAT sont corrects, le ping peut fonctionner.

Pour DNS, il faut aussi une configuration de résolution de noms.

---

**Exemple avec nftables.**

Sur les systèmes modernes, `nftables` remplace souvent `iptables`.

Exemple indicatif :

```bash
sudo nft add table ip nat
sudo nft add chain ip nat postrouting '{ type nat hook postrouting priority 100 ; }'
sudo nft add rule ip nat postrouting ip saddr 10.0.0.0/24 oifname "eth0" masquerade
```

La syntaxe exacte dépend de la configuration existante du système.

En cours, nous insistons surtout sur le principe :

```text
route par défaut dans le namespace
+ forwarding sur l’hôte
+ NAT sur l’hôte
= accès possible vers l’extérieur
```

---

**Nettoyage des règles.**

En TP, nous devons nettoyer les règles ajoutées.

Avec `iptables`, nous pouvons supprimer la règle si nous la connaissons :

```bash
sudo iptables -t nat -D POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
```

Avec `nftables`, nous devons supprimer la règle ou la table créée.

Nous devons éviter de laisser des règles réseau de test sur une machine de production.

---

# 7.10. DNS dans un namespace réseau

**Ping IP et résolution DNS.**

Si ceci fonctionne :

```bash
sudo ip netns exec ns1 ping -c 3 1.1.1.1
```

mais ceci échoue :

```bash
sudo ip netns exec ns1 ping -c 3 example.org
```

le problème est probablement lié au DNS.

Le namespace réseau isole la pile réseau, mais le fichier de configuration DNS dépend aussi de la vue du système de fichiers.

Nous devons regarder :

```bash
cat /etc/resolv.conf
```

dans l’environnement concerné.

---

**Avec `ip netns exec`.**

Quand nous utilisons `ip netns exec`, la commande s’exécute dans le namespace réseau, mais elle garde souvent la même vue de fichiers que l’hôte.

Donc `/etc/resolv.conf` peut rester celui de l’hôte.

Cependant, certaines distributions ou outils peuvent gérer des fichiers par namespace, par exemple sous :

```text
/etc/netns/<nom>/resolv.conf
```

Pour un namespace nommé `ns1`, nous pouvons créer :

```bash
sudo mkdir -p /etc/netns/ns1
echo "nameserver 1.1.1.1" | sudo tee /etc/netns/ns1/resolv.conf
```

Puis :

```bash
sudo ip netns exec ns1 ping -c 3 example.org
```

Selon la configuration, `ip netns exec` peut utiliser ce fichier.

---

# 7.11. Bridge Linux

**Pourquoi utiliser un bridge ?.**

Si nous avons plusieurs namespaces réseau, nous pouvons connecter chacun à l’hôte avec une paire `veth`.

Mais pour connecter plusieurs namespaces entre eux proprement, nous utilisons souvent un bridge Linux.

Un bridge fonctionne comme un switch virtuel.

Schéma conceptuel :

```text
          bridge br0
          /       \
   veth-ns1     veth-ns2
      |            |
     ns1          ns2
```

Chaque namespace est connecté au bridge par une paire `veth`.

---

**Créer un bridge.**

Nous créons le bridge :

```bash
sudo ip link add br0 type bridge
sudo ip addr add 10.10.0.1/24 dev br0
sudo ip link set br0 up
```

Le bridge a l’adresse `10.10.0.1`.

---

**Connecter un namespace au bridge.**

Nous créons un namespace :

```bash
sudo ip netns add ns1
```

Nous créons une paire `veth` :

```bash
sudo ip link add veth1-host type veth peer name veth1-ns
```

Nous plaçons une extrémité dans le namespace :

```bash
sudo ip link set veth1-ns netns ns1
```

Nous connectons l’autre au bridge :

```bash
sudo ip link set veth1-host master br0
sudo ip link set veth1-host up
```

Nous configurons le namespace :

```bash
sudo ip netns exec ns1 ip addr add 10.10.0.2/24 dev veth1-ns
sudo ip netns exec ns1 ip link set veth1-ns up
sudo ip netns exec ns1 ip link set lo up
sudo ip netns exec ns1 ip route add default via 10.10.0.1
```

---

**Ajouter un second namespace.**

Nous répétons avec `ns2` :

```bash
sudo ip netns add ns2
sudo ip link add veth2-host type veth peer name veth2-ns
sudo ip link set veth2-ns netns ns2

sudo ip link set veth2-host master br0
sudo ip link set veth2-host up

sudo ip netns exec ns2 ip addr add 10.10.0.3/24 dev veth2-ns
sudo ip netns exec ns2 ip link set veth2-ns up
sudo ip netns exec ns2 ip link set lo up
sudo ip netns exec ns2 ip route add default via 10.10.0.1
```

Nous testons :

```bash
sudo ip netns exec ns1 ping -c 3 10.10.0.3
sudo ip netns exec ns2 ping -c 3 10.10.0.2
```

Les deux namespaces communiquent via le bridge.

---

**Nettoyage du bridge.**

Nous supprimons :

```bash
sudo ip netns delete ns1
sudo ip netns delete ns2
sudo ip link delete br0
```

Si des interfaces restent :

```bash
ip link | grep veth
```

Nous pouvons supprimer les interfaces résiduelles avec prudence.

---

# 7.14. `/proc/net` et namespace réseau

**`/proc/net` dépend du namespace courant.**

Nous avons vu dans le cours sur `/proc` que :

```bash
cat /proc/net/dev
cat /proc/net/tcp
cat /proc/net/udp
cat /proc/net/unix
```

affichent des informations réseau.

Ces informations dépendent du namespace réseau courant.

Dans un namespace isolé :

```bash
sudo ip netns exec ns1 cat /proc/net/dev
```

nous ne voyons que les interfaces de `ns1`.

---

**Comparer hôte et namespace.**

Sur l’hôte :

```bash
cat /proc/net/dev
```

Dans `ns1` :

```bash
sudo ip netns exec ns1 cat /proc/net/dev
```

Nous comparons les interfaces.

Nous pouvons aussi comparer les sockets :

```bash
ss -tulpen
sudo ip netns exec ns1 ss -tulpen
```

Les ports visibles ne sont pas les mêmes.

---

**Même chemin, contenu différent.**

Le chemin :

```text
/proc/net/dev
```

est le même, mais son contenu dépend du namespace réseau du processus qui le lit.

C’est une idée importante :

```text
même chemin
namespace différent
contenu différent
```

Comme pour `/proc/sys/kernel/hostname` avec le namespace UTS, `/proc/net` illustre parfaitement la logique des namespaces.

---

# 7.15. Pare-feu et règles réseau

**Netfilter et namespaces.**

Linux utilise netfilter pour le filtrage, le NAT et d’autres traitements réseau.

Les règles peuvent dépendre du namespace réseau.

Chaque namespace réseau possède sa propre pile réseau, et donc sa propre vue de certaines règles et tables.

Nous pouvons tester dans un namespace :

```bash
sudo ip netns exec ns1 nft list ruleset
```

ou, si `iptables` est utilisé :

```bash
sudo ip netns exec ns1 iptables -L
```

Selon les droits et l’outillage disponible, les résultats peuvent varier.

---

**Pare-feu sur l’hôte.**

Même si un namespace a ses propres règles, le trafic passe souvent par l’hôte pour sortir.

Le pare-feu de l’hôte peut donc influencer :

- la connectivité du namespace ;

- le NAT ;

- les communications entre namespaces ;

- les communications vers Internet.


Dans un diagnostic réseau, nous devons regarder :

```bash
ip route
nft list ruleset
iptables -L -n -v
iptables -t nat -L -n -v
```

selon le système.

---

# 7.16. Sécurité du namespace réseau

**Ce que le namespace réseau protège.**

Le namespace réseau limite la visibilité et l’accès aux éléments réseau de l’hôte.

Un processus isolé ne voit pas directement :

- les interfaces de l’hôte ;

- les ports en écoute de l’hôte ;

- les sockets de l’hôte ;

- les routes de l’hôte ;

- certains états réseau de l’hôte.


C’est une protection importante.

---

**Ce qu’il ne protège pas seul.**

Le namespace réseau ne suffit pas à lui seul.

Nous devons aussi considérer :

- capabilities réseau ;

- accès à `/proc` et `/sys` ;

- montages sensibles ;

- règles de pare-feu ;

- cgroups ;

- politiques Kubernetes ;

- seccomp ;

- AppArmor ou SELinux.


Un conteneur avec `CAP_NET_ADMIN` peut modifier beaucoup de choses dans son namespace réseau.

S’il partage le namespace réseau de l’hôte, `CAP_NET_ADMIN` devient encore plus sensible.

---

**Options dangereuses.**

Docker :

```bash
docker run --network=host ...
```

Kubernetes :

```yaml
hostNetwork: true
```

Ces options réduisent l’isolation réseau.

Elles peuvent être justifiées pour certains agents système, mais elles doivent être utilisées avec prudence.

---

# 7.17. Diagnostic réseau avec namespaces

**Questions à se poser.**

Lorsqu’un conteneur ou un namespace réseau a un problème, nous posons les questions suivantes :

```text
Dans quel namespace suis-je ?
Quelles interfaces sont visibles ?
L’interface lo est-elle up ?
L’interface veth est-elle up ?
Quelle adresse IP est configurée ?
Quelle route par défaut existe ?
Le forwarding est-il activé ?
Le NAT est-il configuré ?
Le DNS fonctionne-t-il ?
Le pare-feu bloque-t-il ?
Le processus écoute-t-il dans le bon namespace ?
```

---

**Commandes utiles.**

Dans le namespace :

```bash
ip addr
ip link
ip route
ss -tulpen
cat /proc/net/dev
cat /etc/resolv.conf
```

Depuis l’hôte vers un namespace nommé :

```bash
sudo ip netns exec ns1 ip addr
sudo ip netns exec ns1 ip route
sudo ip netns exec ns1 ss -tulpen
```

Depuis l’hôte vers un processus :

```bash
sudo nsenter --target <PID> --net ip addr
sudo nsenter --target <PID> --net ip route
sudo nsenter --target <PID> --net ss -tulpen
```

---

**Exemple : un serveur écoute, mais il est inaccessible.**

Nous lançons dans `ns1` :

```bash
sudo ip netns exec ns1 python3 -m http.server 8080
```

Depuis l’hôte, nous testons :

```bash
curl http://10.0.0.2:8080
```

Si cela échoue, nous vérifions :

```bash
sudo ip netns exec ns1 ip addr
sudo ip netns exec ns1 ss -ltnp
ip addr show veth-host
ping -c 3 10.0.0.2
```

Nous cherchons :

- le serveur écoute-t-il sur `127.0.0.1` seulement ?

- `veth-ns` est-elle up ?

- `veth-host` est-elle up ?

- les IP sont-elles correctes ?

- le pare-feu bloque-t-il ?

- la route est-elle correcte ?


---

# 7.18. Expérience complète guidée

**Créer un namespace réseau connecté à l’hôte.**

Nous exécutons :

```bash
sudo ip netns add ns1
sudo ip netns exec ns1 ip link set lo up

sudo ip link add veth-host type veth peer name veth-ns
sudo ip link set veth-ns netns ns1

sudo ip addr add 10.0.0.1/24 dev veth-host
sudo ip link set veth-host up

sudo ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns
sudo ip netns exec ns1 ip link set veth-ns up
```

Nous testons :

```bash
ping -c 3 10.0.0.2
sudo ip netns exec ns1 ping -c 3 10.0.0.1
```

---

**Lancer un service dans le namespace.**

Nous lançons :

```bash
sudo ip netns exec ns1 python3 -m http.server 8080
```

Dans un autre terminal, depuis l’hôte :

```bash
curl http://10.0.0.2:8080
```

Nous avons un service réseau isolé dans `ns1`, accessible via l’interface `veth`.

---

**Observer les sockets.**

Dans `ns1` :

```bash
sudo ip netns exec ns1 ss -ltnp
```

Sur l’hôte :

```bash
ss -ltnp | grep 8080
```

Le service peut être visible différemment selon le contexte.

Nous comprenons ainsi que les sockets appartiennent à un namespace réseau.

---

**Nettoyage.**

Nous arrêtons le serveur avec `Ctrl+C`.

Puis :

```bash
sudo ip netns delete ns1
sudo ip link delete veth-host 2>/dev/null
```

Nous vérifions :

```bash
ip netns list
ip link | grep veth
```

---

# 7.19. Pièges classiques

**Oublier d’activer `lo`.**

Dans un namespace réseau neuf :

```bash
ip link set lo up
```

est souvent nécessaire.

Sans cela, même `127.0.0.1` peut ne pas fonctionner correctement.

---

**Oublier de monter ou d’installer les outils.**

Dans un conteneur minimal, les commandes suivantes peuvent manquer :

```text
ip
ss
ping
curl
iptables
nft
```

Le namespace réseau existe, mais l’image ne contient pas forcément les outils de diagnostic.

Nous pouvons diagnostiquer depuis l’hôte avec `nsenter`.

---

**Confondre `127.0.0.1` de l’hôte et du conteneur.**

Dans un conteneur :

```text
127.0.0.1
```

désigne le loopback du conteneur, pas celui de l’hôte.

Dans un pod Kubernetes, deux conteneurs du même pod partagent souvent le même `127.0.0.1`.

Nous devons toujours préciser le namespace réseau.

---

**Oublier la route par défaut.**

Un namespace peut avoir une interface et une IP, mais aucune route par défaut.

Nous vérifions :

```bash
ip route
```

ou :

```bash
sudo ip netns exec ns1 ip route
```

---

**Oublier le NAT.**

Si le namespace doit accéder à Internet via l’hôte, il faut souvent :

```text
route par défaut
forwarding
NAT
pare-feu adapté
DNS
```

Une IP configurée ne suffit pas.

---

**Utiliser `hostNetwork` sans comprendre.**

`--network=host` ou `hostNetwork: true` supprime une partie importante de l’isolation réseau.

Cela peut provoquer :

- conflits de ports ;

- exposition de services ;

- confusion dans les diagnostics ;

- risques de sécurité accrus.


---

# 7.21. Ce que nous devons retenir

Nous retenons les points suivants :

1. Le namespace network isole la pile réseau.

2. Il isole les interfaces, adresses IP, routes, sockets et ports.

3. `/proc/<PID>/ns/net` permet d’identifier le namespace réseau d’un processus.

4. `/proc/net` dépend du namespace réseau courant.

5. Un namespace réseau neuf contient souvent seulement `lo`, parfois désactivée.

6. Nous devons activer `lo` avec `ip link set lo up`.

7. Une paire `veth` permet de connecter deux namespaces réseau.

8. `ip netns` permet de créer des namespaces réseau nommés.

9. Un bridge Linux fonctionne comme un switch virtuel.

10. Pour sortir vers Internet, il faut souvent route par défaut, forwarding, NAT et DNS.

11. Docker utilise des namespaces réseau, des paires `veth`, des bridges et des règles NAT.

12. Dans Kubernetes, les conteneurs d’un même pod partagent généralement le même namespace réseau.

13. Le pause container maintient souvent le namespace réseau du pod.

14. Les plugins CNI configurent le réseau des pods.

15. `--network=host` et `hostNetwork: true` réduisent fortement l’isolation.

16. `127.0.0.1` désigne le loopback du namespace courant, pas forcément celui de l’hôte.

17. Le namespace réseau est puissant, mais il doit être compris avec les routes, le pare-feu, le NAT et les politiques de sécurité.


---

# Conclusion du chapitre 7

Nous avons étudié le namespace network, qui est l’un des namespaces les plus importants pour les conteneurs et l’isolation applicative.

Nous savons maintenant créer un namespace réseau, activer son loopback, le connecter à l’hôte avec une paire `veth`, configurer des adresses IP, ajouter des routes, comprendre le forwarding, le NAT et le rôle d’un bridge Linux.

Nous avons aussi relié ces notions aux environnements réels : Docker utilise des bridges, des paires `veth` et du NAT ; Kubernetes confie la configuration réseau aux plugins CNI et fait généralement partager le namespace réseau aux conteneurs d’un même pod.

Nous retenons surtout qu’un problème réseau dans un conteneur ne se diagnostique pas comme un problème réseau classique. Nous devons toujours nous demander dans quel namespace nous observons les interfaces, les routes, les sockets et les ports.

Dans le chapitre suivant, nous étudions le namespace IPC, qui isole certains mécanismes de communication inter-processus comme les files de messages, les sémaphores et la mémoire partagée.

---

# Chapitre 8 — Namespace IPC

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace IPC.

Le namespace IPC permet d’isoler certains mécanismes de communication inter-processus. IPC signifie _Inter-Process Communication_, c’est-à-dire communication entre processus.

Sous Linux, plusieurs mécanismes permettent à des processus de communiquer localement :

- files de messages System V ;

- sémaphores System V ;

- mémoire partagée System V ;

- mémoire partagée POSIX ;

- files de messages POSIX ;

- certains objets IPC nommés.


Le namespace IPC permet de séparer ces ressources entre groupes de processus. C’est utile pour éviter qu’un processus dans un conteneur voie ou manipule des ressources IPC appartenant à l’hôte ou à un autre conteneur.

À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace IPC ;

- comprendre ce que sont les mécanismes IPC ;

- observer les ressources IPC avec `ipcs` ;

- créer un namespace IPC avec `unshare`;

- comparer les ressources IPC entre l’hôte et un namespace isolé ;

- comprendre le lien avec Docker, Podman et Kubernetes ;

- identifier les risques liés au partage du namespace IPC avec l’hôte.


---

# 8.1. Pourquoi isoler les communications inter-processus ?

**Le problème de départ.**

Sur un système Linux, plusieurs processus peuvent communiquer entre eux sans passer par le réseau.

Ils peuvent utiliser des mécanismes IPC pour :

- partager une zone mémoire ;

- synchroniser des accès concurrents ;

- envoyer des messages ;

- coordonner plusieurs processus d’une même application ;

- communiquer entre services locaux.


Cela peut être très utile, mais cela pose une question d’isolation.

Si tous les processus voient les mêmes ressources IPC, alors un processus non souhaité pourrait potentiellement :

- voir l’existence de ressources IPC d’une autre application ;

- interférer avec des sémaphores ;

- lire ou modifier une mémoire partagée mal protégée ;

- envoyer des messages à une file IPC ;

- provoquer des conflits de noms ;

- perturber une application.


Le namespace IPC permet de donner à un groupe de processus sa propre vue des ressources IPC.

---

**Exemple conceptuel.**

Sans namespace IPC isolé :

```text
Hôte
├── application A
│   └── mémoire partagée id=123
├── application B
│   └── sémaphore id=456
└── conteneur C
    └── voit potentiellement certaines ressources IPC globales
```

Avec namespace IPC isolé :

```text
Namespace IPC de l’hôte
├── application A
└── application B

Namespace IPC du conteneur C
└── ressources IPC propres au conteneur
```

Le conteneur ne voit pas les ressources IPC de l’hôte.

---

# 8.2. Qu’est-ce que l’IPC ?

**Définition générale.**

IPC signifie _Inter-Process Communication_.

Nous parlons d’IPC lorsqu’un processus communique avec un autre processus sur la même machine.

Il existe plusieurs familles de mécanismes IPC :

```text
pipes
sockets Unix
signaux
files de messages
sémaphores
mémoire partagée
futex
eventfd
```

Tous ces mécanismes ne sont pas isolés de la même façon par le namespace IPC.

Dans ce chapitre, nous nous concentrons surtout sur les IPC System V et POSIX, car ce sont eux qui sont traditionnellement associés au namespace IPC.

---

**IPC System V.**

Les IPC System V sont des mécanismes historiques d’Unix.

Ils comprennent :

- files de messages ;

- sémaphores ;

- segments de mémoire partagée.


Nous les observons avec :

```bash
ipcs
```

Sortie typique :

```text
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status

------ Semaphore Arrays --------
key        semid      owner      perms      nsems
```

---

**IPC POSIX.**

Les IPC POSIX sont des mécanismes plus modernes ou différents, par exemple :

- mémoire partagée POSIX ;

- sémaphores POSIX ;

- files de messages POSIX.


Ils peuvent apparaître dans des emplacements comme :

```text
/dev/shm
```

ou être gérés par le noyau avec des objets nommés.

Le namespace IPC peut également influencer la visibilité de certaines ressources POSIX IPC.

---

# 8.3. Observer le namespace IPC courant

**Avec `/proc/<PID>/ns/ipc`.**

Nous pouvons observer le namespace IPC de notre shell courant :

```bash
readlink /proc/$$/ns/ipc
```

Exemple :

```text
ipc:[4026531839]
```

Nous comparons avec le PID 1 :

```bash
readlink /proc/1/ns/ipc
```

Si les deux valeurs sont identiques, notre shell partage le namespace IPC du PID 1.

---

**Comparer deux processus.**

Nous pouvons comparer deux processus :

```bash
echo "Shell courant : $(readlink /proc/$$/ns/ipc)"
echo "PID 1         : $(readlink /proc/1/ns/ipc)"
```

Même valeur :

```text
Shell courant : ipc:[4026531839]
PID 1         : ipc:[4026531839]
```

Les deux processus partagent le même namespace IPC.

Valeurs différentes :

```text
Shell courant : ipc:[4026533000]
PID 1         : ipc:[4026531839]
```

Les deux processus ont des vues IPC différentes.

---

# 8.4. Observer les ressources IPC avec `ipcs`

**La commande `ipcs`.**

La commande principale pour observer les ressources IPC System V est :

```bash
ipcs
```

Elle affiche :

- files de messages ;

- segments de mémoire partagée ;

- tableaux de sémaphores.


Nous pouvons aussi filtrer :

```bash
ipcs -q
ipcs -m
ipcs -s
```

Avec :

|Option|Ressource|
|---|---|
|`-q`|files de messages|
|`-m`|mémoire partagée|
|`-s`|sémaphores|

---

**Exemple de sortie.**

```text
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0x12345678 0          user       600        0            0

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x87654321 1          user       600        4096       1

------ Semaphore Arrays --------
key        semid      owner      perms      nsems
0x11111111 2          user       600        1
```

Nous voyons que chaque ressource possède :

- une clé ;

- un identifiant ;

- un propriétaire ;

- des permissions ;

- des informations propres au type de ressource.


---

**Pourquoi `ipcs` dépend du namespace IPC.**

La commande `ipcs` affiche les ressources IPC visibles dans le namespace IPC courant.

Donc :

```bash
ipcs
```

sur l’hôte et :

```bash
unshare --ipc bash
ipcs
```

dans un namespace isolé peuvent afficher des résultats différents.

C’est le même outil, mais pas la même vue.

---

# 8.5. Créer un namespace IPC avec `unshare`

**Expérience simple.**

Nous créons un nouveau namespace IPC :

```bash
unshare --ipc bash
```

Dans ce shell, nous observons :

```bash
readlink /proc/$$/ns/ipc
ipcs
```

Nous pouvons comparer avec l’hôte dans un autre terminal :

```bash
readlink /proc/1/ns/ipc
ipcs
```

Le namespace IPC est différent.

Les ressources IPC visibles peuvent aussi être différentes.

---

**Avec les droits nécessaires.**

Selon la configuration du système, la commande peut échouer :

```bash
unshare --ipc bash
```

Erreur possible :

```text
unshare: unshare failed: Operation not permitted
```

Dans ce cas, nous pouvons utiliser :

```bash
sudo unshare --ipc bash
```

Ou, selon la configuration des user namespaces :

```bash
unshare --user --map-root-user --ipc bash
```

Nous retenons que la création de certains namespaces dépend des permissions et des capacités disponibles.

---

# 8.6. Créer une ressource IPC pour observer l’isolation

**Problème pratique.**

Observer l’isolation IPC est moins spectaculaire que changer le hostname ou créer une interface réseau.

Si aucune ressource IPC n’existe, `ipcs` peut afficher une sortie vide dans l’hôte et dans le namespace isolé.

Nous avons donc besoin de créer une ressource IPC de test.

---

**Créer une mémoire partagée avec Python.**

Nous pouvons créer une mémoire partagée POSIX avec Python 3 :

```bash
python3 - <<'PY'
from multiprocessing import shared_memory
import time

shm = shared_memory.SharedMemory(create=True, size=1024, name="demo_ipc_namespace")
print("Mémoire partagée créée :", shm.name)
print("PID:", __import__("os").getpid())
try:
    time.sleep(300)
finally:
    shm.close()
    shm.unlink()
PY
```

Selon le système, cette ressource peut être visible via `/dev/shm`.

Dans un autre terminal :

```bash
ls -l /dev/shm | grep demo_ipc_namespace
```

Attention : l’observation dépend aussi du namespace mount, car `/dev/shm` est un montage.

Cela montre que les namespaces peuvent interagir : IPC et mount peuvent tous les deux influencer ce que nous voyons.

---

**Créer une ressource System V.**

Les ressources System V sont plus directement observables avec `ipcs`.

Selon les outils installés, nous pouvons utiliser des programmes dédiés ou écrire un petit programme C/Python. Pour un cours général, nous pouvons retenir l’idée suivante :

```text
Créer une ressource IPC System V dans un namespace IPC
→ elle apparaît dans ipcs dans ce namespace
→ elle n’apparaît pas dans ipcs depuis un autre namespace IPC
```

L’objectif pédagogique n’est pas ici d’apprendre toute l’API System V, mais de comprendre l’isolation.

---

# 8.7. Mémoire partagée et `/dev/shm`

**Rôle de `/dev/shm`.**

Sur beaucoup de systèmes Linux, `/dev/shm` est un `tmpfs` utilisé pour la mémoire partagée POSIX.

Nous pouvons observer :

```bash
findmnt /dev/shm
```

Sortie possible :

```text
TARGET   SOURCE FSTYPE OPTIONS
/dev/shm tmpfs  tmpfs  rw,nosuid,nodev
```

Les objets de mémoire partagée POSIX peuvent apparaître comme des fichiers dans `/dev/shm`.

---

**Interaction entre IPC namespace et mount namespace.**

Il faut être prudent : `/dev/shm` dépend aussi des montages.

Deux processus peuvent avoir :

- le même namespace IPC mais des namespaces mount différents ;

- des namespaces IPC différents mais un `/dev/shm` monté de manière surprenante ;

- un `/dev/shm` partagé par erreur entre conteneurs.


Dans les conteneurs, `/dev/shm` est souvent monté spécifiquement.

Docker permet par exemple de contrôler sa taille avec :

```bash
docker run --shm-size=256m ...
```

Nous voyons donc que le diagnostic IPC ne se limite pas au namespace IPC seul.

---

**`/dev/shm` dans Docker.**

Dans un conteneur Docker :

```bash
df -h /dev/shm
mount | grep shm
```

Nous pouvons voir un `tmpfs` dédié.

Par défaut, Docker donne souvent une taille limitée à `/dev/shm`.

Cela peut provoquer des erreurs pour certaines applications utilisant beaucoup de mémoire partagée, par exemple :

- navigateurs ;

- bases de données ;

- calcul scientifique ;

- traitement vidéo ;

- tests headless ;

- Chrome/Chromium en conteneur.


---

# 8.8. Sémaphores

**Qu’est-ce qu’un sémaphore ?.**

Un sémaphore est un mécanisme de synchronisation.

Il permet à plusieurs processus de coordonner l’accès à une ressource partagée.

Exemple conceptuel :

```text
Un seul processus à la fois peut écrire dans une zone critique.
Le sémaphore sert à contrôler l’accès.
```

Les sémaphores peuvent être utilisés par des bases de données, des services système ou des applications multi-processus.

---

**Observer les sémaphores System V.**

Nous utilisons :

```bash
ipcs -s
```

Sortie possible :

```text
------ Semaphore Arrays --------
key        semid      owner      perms      nsems
0x00000000 3          user       600        1
```

Dans un namespace IPC isolé, nous ne voyons que les sémaphores de ce namespace.

---

# 8.9. Files de messages

**Rôle des files de messages.**

Les files de messages IPC permettent à des processus d’échanger des messages structurés via le noyau.

Elles sont moins utilisées dans beaucoup d’applications modernes que les sockets Unix ou les files applicatives, mais elles existent toujours.

---

**Observer les files de messages.**

Nous utilisons :

```bash
ipcs -q
```

Sortie possible :

```text
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
```

Si aucune file n’existe, la table est vide.

Dans un namespace IPC isolé, les files de messages de l’hôte ne sont pas visibles.

---

# 8.12. Namespace IPC et sécurité

**Ce que le namespace IPC protège.**

Le namespace IPC évite qu’un processus voie ou manipule directement certaines ressources IPC d’un autre environnement.

Il protège notamment contre :

- observation de segments de mémoire partagée ;

- interaction avec des sémaphores ;

- collisions de ressources IPC ;

- accès involontaire à des files de messages ;

- perturbation entre applications.


---

**Ce qu’il ne protège pas seul.**

Le namespace IPC ne suffit pas à sécuriser un conteneur.

Nous devons aussi considérer :

- user namespace ;

- mount namespace ;

- permissions Unix ;

- capabilities ;

- seccomp ;

- AppArmor ou SELinux ;

- exposition de `/dev/shm` ;

- options Docker/Kubernetes ;

- volumes montés ;

- droits sur les objets IPC.


Comme souvent, l’isolation réelle repose sur plusieurs couches.

---

**Risques de `--ipc=host` et `hostIPC`.**

Les options suivantes réduisent fortement l’isolation :

```bash
docker run --ipc=host ...
```

et :

```yaml
hostIPC: true
```

Elles peuvent permettre à un processus conteneurisé de voir ou d’interagir avec des ressources IPC de l’hôte.

Nous les évitons sauf cas justifié.

---

# 8.13. Diagnostic IPC

**Questions à se poser.**

Lorsqu’une application a un problème lié à l’IPC, nous demandons :

```text
Dans quel namespace IPC tourne-t-elle ?
Voit-elle les ressources IPC attendues ?
Utilise-t-elle System V IPC ou POSIX IPC ?
Utilise-t-elle /dev/shm ?
La taille de /dev/shm est-elle suffisante ?
Les permissions sont-elles correctes ?
Le conteneur partage-t-il l’IPC de l’hôte ?
Un autre processus doit-il partager le même namespace IPC ?
```

---

**Commandes utiles.**

Observer le namespace IPC :

```bash
readlink /proc/<PID>/ns/ipc
```

Observer les ressources IPC System V :

```bash
ipcs
ipcs -m
ipcs -s
ipcs -q
```

Observer `/dev/shm` :

```bash
df -h /dev/shm
ls -lh /dev/shm
findmnt /dev/shm
```

Entrer dans le namespace IPC d’un processus :

```bash
sudo nsenter --target <PID> --ipc bash
```

Ou avec d’autres namespaces utiles :

```bash
sudo nsenter --target <PID> --ipc --mount --pid --fork bash
```

---

**Exemple : application Chrome en conteneur.**

Certaines applications comme Chrome/Chromium en mode headless peuvent utiliser beaucoup `/dev/shm`.

Symptôme possible :

```text
crash du navigateur
erreurs de mémoire partagée
problèmes aléatoires en tests end-to-end
```

Diagnostic :

```bash
df -h /dev/shm
```

Solution possible avec Docker :

```bash
docker run --shm-size=1g ...
```

Solution possible avec Kubernetes :

```yaml
emptyDir:
  medium: Memory
```

Ici, le problème n’est pas seulement le namespace IPC, mais aussi le montage et la taille de `/dev/shm`.

---

# 8.14. Expérience complète guidée

**Comparer l’IPC de l’hôte et d’un namespace isolé.**

Sur l’hôte :

```bash
readlink /proc/$$/ns/ipc
ipcs
```

Nous créons un namespace IPC :

```bash
unshare --ipc bash
```

Dans le shell isolé :

```bash
readlink /proc/$$/ns/ipc
ipcs
```

Nous comparons les identifiants et les ressources visibles.

---

**Entrer dans le namespace IPC depuis un second terminal.**

Dans le shell isolé :

```bash
echo $$
sleep 1000
```

Dans un second terminal :

```bash
sudo nsenter --target <PID> --ipc bash
readlink /proc/$$/ns/ipc
ipcs
```

Nous voyons le même namespace IPC que le processus cible.

Nous quittons :

```bash
exit
```

---

**Nettoyage.**

Dans le shell isolé :

```bash
exit
```

Si un processus `sleep` a été lancé :

```bash
kill <PID>
```

Quand le dernier processus du namespace IPC disparaît, le namespace est détruit, sauf s’il est maintenu par une autre référence.

---

# 8.15. Pièges classiques

**Croire que toutes les communications locales passent par le namespace IPC.**

Le namespace IPC n’isole pas tous les mécanismes de communication.

Par exemple :

- les sockets réseau relèvent du namespace network ;

- les sockets Unix peuvent dépendre du système de fichiers et des montages ;

- les pipes anonymes dépendent surtout des relations entre processus ;

- les signaux dépendent des permissions et du namespace PID ;

- les fichiers partagés dépendent du namespace mount et des permissions.


Nous devons donc identifier le mécanisme de communication exact.

---

**Confondre `/dev/shm` et namespace IPC.**

`/dev/shm` est un montage `tmpfs`.

Il est souvent utilisé pour la mémoire partagée POSIX, mais son comportement dépend aussi du namespace mount.

Un problème de `/dev/shm` peut être lié :

- à sa taille ;

- à son montage ;

- aux permissions ;

- au namespace IPC ;

- au runtime de conteneur.


---

**Utiliser `--ipc=host` pour contourner un problème.**

Il peut être tentant de résoudre un problème en lançant :

```bash
docker run --ipc=host ...
```

Mais cela réduit l’isolation.

Nous devons d’abord comprendre le problème :

- manque de taille `/dev/shm` ?

- besoin réel de partager des IPC entre processus ?

- mauvaise configuration de volume ?

- problème de permissions ?


Souvent, une solution plus propre existe.

---

**Oublier que les ressources IPC peuvent persister.**

Certaines ressources IPC System V peuvent persister après la fin d’un processus si elles ne sont pas supprimées correctement.

Nous pouvons les voir avec :

```bash
ipcs
```

Et les supprimer avec :

```bash
ipcrm
```

Par exemple :

```bash
ipcrm -m <shmid>
ipcrm -s <semid>
ipcrm -q <msqid>
```

Nous utilisons `ipcrm` avec prudence, car supprimer une ressource IPC utilisée par une application peut la perturber.

---

# 8.17. Ce que nous devons retenir

Nous retenons les points suivants :

1. IPC signifie communication inter-processus.

2. Le namespace IPC isole certaines ressources IPC entre groupes de processus.

3. Il concerne notamment les IPC System V : files de messages, sémaphores et mémoire partagée.

4. Il influence aussi certains mécanismes POSIX IPC.

5. Le namespace IPC d’un processus est visible avec `/proc/<PID>/ns/ipc`.

6. La commande `ipcs` affiche les ressources IPC visibles dans le namespace courant.

7. `unshare --ipc bash` permet de créer un nouveau namespace IPC.

8. `nsenter --target <PID> --ipc bash` permet d’entrer dans le namespace IPC d’un processus.

9. `/dev/shm` est souvent utilisé pour la mémoire partagée POSIX, mais dépend aussi du namespace mount.

10. Docker crée généralement un namespace IPC isolé par conteneur.

11. `--ipc=host` fait partager le namespace IPC de l’hôte et réduit l’isolation.

12. Kubernetes propose `hostIPC: true`, option sensible à éviter sans justification forte.

13. Les problèmes IPC peuvent venir du namespace IPC, mais aussi de `/dev/shm`, des permissions, des montages ou de la taille disponible.

14. Certaines ressources IPC peuvent persister et nécessiter un nettoyage avec `ipcrm`.

15. Le namespace IPC est une couche d’isolation utile, mais il ne remplace pas les autres mécanismes de sécurité.


---

# Conclusion du chapitre 8

Nous avons étudié le namespace IPC, qui isole certains mécanismes de communication inter-processus.

Nous savons maintenant observer le namespace IPC d’un processus, utiliser `ipcs` pour voir les ressources IPC System V, créer un namespace IPC avec `unshare`, et entrer dans un namespace existant avec `nsenter`.

Nous avons aussi compris que l’IPC ne doit pas être analysé isolément. Dans les environnements conteneurisés, les problèmes de mémoire partagée impliquent souvent `/dev/shm`, donc aussi le namespace mount et la configuration du runtime.

Nous retenons surtout que le namespace IPC évite que des processus isolés voient ou manipulent les ressources IPC de l’hôte ou d’un autre conteneur. Comme toujours avec les namespaces, cette isolation est utile, mais elle ne suffit pas à elle seule : elle doit être combinée avec les permissions, les cgroups, les capabilities et les politiques de sécurité.

Dans le chapitre suivant, nous étudions le namespace user, l’un des namespaces les plus importants et les plus subtils, car il permet d’être `root` dans un namespace sans être réellement `root` sur l’hôte.

---

# Chapitre 9 — Namespace user

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace user.

C’est l’un des namespaces les plus importants, mais aussi l’un des plus subtils. Il permet d’isoler les identifiants utilisateurs et groupes, c’est-à-dire les UID et les GID.

Son idée centrale est la suivante :

```text
Nous pouvons être root dans un namespace sans être root sur l’hôte.
```

Cela paraît paradoxal au premier abord, mais c’est un mécanisme fondamental pour les conteneurs rootless et pour la réduction des privilèges.

À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace user ;

- comprendre les mappings UID/GID ;

- lire `/proc/<PID>/uid_map` et `/proc/<PID>/gid_map` ;

- créer un user namespace avec `unshare` ;

- comprendre ce que signifie être `root` dans un namespace ;

- expliquer le lien avec les capabilities ;

- comprendre le rôle de `/etc/subuid` et `/etc/subgid` ;

- comprendre le rôle de `newuidmap` et `newgidmap` ;

- faire le lien avec Docker, Podman et les conteneurs rootless ;

- identifier les limites et risques de sécurité.


---

# 9.1. Pourquoi isoler les utilisateurs ?

**Le problème de départ.**

Sur un système Linux classique, les utilisateurs sont identifiés par des UID et des GID.

Nous pouvons afficher notre identité avec :

```bash
id
```

Exemple :

```text
uid=1000(user) gid=1000(user) groupes=1000(user),27(sudo)
```

L’utilisateur `root` possède généralement l’UID `0`.

```text
uid=0(root)
```

Sur l’hôte, l’UID `0` est extrêmement puissant. Il peut administrer le système, modifier des fichiers sensibles, gérer des processus, monter des systèmes de fichiers, configurer le réseau, charger certains paramètres, etc.

Le problème est donc le suivant :

```text
Comment permettre à un processus d’être root dans un environnement isolé,
sans lui donner les droits root sur toute la machine ?
```

Le namespace user répond à cette question.

---

**L’idée fondamentale.**

Un namespace user permet de créer une correspondance entre les UID/GID vus à l’intérieur du namespace et les UID/GID réels à l’extérieur.

Exemple conceptuel :

```text
Dans le namespace :
UID 0 → root

Sur l’hôte :
UID 0 du namespace correspond à UID 1000
```

Le processus se croit root dans son namespace, mais le noyau sait qu’à l’extérieur, il correspond à un utilisateur non privilégié.

C’est le mécanisme de mapping UID/GID.

---

# 9.2. Comprendre UID, GID et root

**UID.**

L’UID identifie un utilisateur.

Exemples courants :

```text
UID 0     → root
UID 1000  → premier utilisateur classique sur beaucoup de distributions
UID 33    → www-data sur Debian/Ubuntu
```

Nous pouvons voir l’UID courant avec :

```bash
id -u
```

---

**GID.**

Le GID identifie un groupe.

Nous pouvons voir notre groupe principal avec :

```bash
id -g
```

Et tous nos groupes avec :

```bash
id
```

Les groupes permettent de donner des permissions à plusieurs utilisateurs.

---

**Root n’est pas seulement un nom.**

`root` n’est pas magique parce qu’il s’appelle `root`.

Il est spécial parce que son UID est `0`.

Nous pouvons le vérifier :

```bash
id root
```

Sortie typique :

```text
uid=0(root) gid=0(root) groupes=0(root)
```

Le namespace user permet justement de faire apparaître un UID `0` à l’intérieur, tout en le mappant vers un UID non-root à l’extérieur.

---

# 9.3. Observer le namespace user courant

**Avec `/proc/<PID>/ns/user`.**

Nous observons le namespace user de notre shell courant :

```bash
readlink /proc/$$/ns/user
```

Exemple :

```text
user:[4026531837]
```

Nous comparons avec le PID 1 :

```bash
readlink /proc/1/ns/user
```

Si les deux valeurs sont identiques, notre shell partage le même namespace user que le PID 1.

---

**Lire les mappings UID/GID.**

Les mappings du processus courant sont visibles dans :

```bash
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Sur un système classique, dans le namespace initial, nous pouvons voir :

```text
         0          0 4294967295
```

Cela signifie :

```text
UID interne 0 correspond à UID externe 0
pour une très grande plage d’UID.
```

Autrement dit, dans le namespace user initial, les identifiants sont les mêmes dedans et dehors.

---

# 9.4. Format de `uid_map` et `gid_map`

**Structure d’une ligne.**

Une ligne de `uid_map` ou `gid_map` a trois colonnes :

```text
ID_dans_le_namespace   ID_dans_le_parent   taille_de_la_plage
```

Exemple :

```text
         0       1000          1
```

Nous lisons :

```text
UID 0 dans le namespace
correspond à UID 1000 dans le namespace parent
pour une plage de taille 1.
```

Cela signifie que seul l’UID `0` interne est mappé vers l’UID `1000` externe.

---

**Exemple avec une plage plus large.**

Autre exemple :

```text
         0     100000      65536
```

Nous lisons :

```text
UID 0 à 65535 dans le namespace
correspondent à UID 100000 à 165535 dans le parent.
```

Cela permet à un conteneur d’avoir de nombreux utilisateurs internes sans utiliser directement les vrais UID de l’hôte.

---

**Pourquoi les mappings sont essentiels.**

Sans mapping, le noyau ne sait pas comment traduire les identifiants internes vers l’extérieur.

Un fichier créé par un processus dans le namespace doit avoir un propriétaire réel sur l’hôte.

Le mapping répond à cette question :

```text
Quand le processus dit “je suis UID 0”, quel UID réel utilise-t-on sur l’hôte ?
```

C’est une question centrale pour les permissions de fichiers, les bind mounts et les conteneurs rootless.

---

# 9.5. Créer un user namespace simple

**Avec `unshare --user`.**

Nous pouvons créer un user namespace :

```bash
unshare --user bash
```

Dans ce shell :

```bash
id
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Selon la configuration, nous pouvons voir une situation où les mappings sont vides ou limités.

Créer un user namespace brut n’est pas toujours très pratique.

Nous utilisons souvent une option plus pédagogique :

```bash
unshare --user --map-root-user bash
```

---

**Avec `--map-root-user`.**

Nous lançons :

```bash
unshare --user --map-root-user bash
```

Dans le shell :

```bash
id
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Sortie possible :

```text
uid=0(root) gid=0(root) groupes=0(root)
```

Puis :

```text
         0       1000          1
```

Cela signifie que nous sommes root à l’intérieur du namespace, mais que cet UID root est mappé vers notre UID réel sur l’hôte.

---

**Vérifier que nous ne sommes pas root sur l’hôte.**

Dans le namespace :

```bash
id
```

Nous voyons :

```text
uid=0(root)
```

Mais si nous essayons d’écrire dans un fichier sensible de l’hôte, par exemple :

```bash
touch /root/test-userns
```

nous obtenons généralement :

```text
Permission denied
```

Cela montre que notre root interne n’est pas root global sur l’hôte.

Nous sommes root dans le namespace, pas dans le système entier.

---

# 9.6. Root dans le namespace, utilisateur normal sur l’hôte

**Exemple avec un fichier.**

Sur l’hôte, nous créons un dossier de test :

```bash
mkdir -p /tmp/userns-demo
```

Nous lançons :

```bash
unshare --user --map-root-user bash
```

Dans le namespace :

```bash
id
touch /tmp/userns-demo/fichier.txt
ls -ln /tmp/userns-demo/fichier.txt
```

Nous pouvons voir à l’intérieur :

```text
-rw-r--r-- 1 0 0 ... fichier.txt
```

Depuis l’hôte, dans un autre terminal :

```bash
ls -ln /tmp/userns-demo/fichier.txt
```

Nous pouvons voir :

```text
-rw-r--r-- 1 1000 1000 ... fichier.txt
```

Le même fichier a une interprétation différente selon le mapping.

---

**Même fichier, deux lectures différentes.**

Dans le namespace :

```text
propriétaire : UID 0
```

Sur l’hôte :

```text
propriétaire : UID 1000
```

Ce n’est pas une contradiction.

C’est la conséquence du mapping UID.

Nous retenons :

```text
Le namespace user modifie la traduction des identifiants.
Il ne change pas magiquement les permissions réelles de l’hôte.
```

---

# 9.7. User namespace et capabilities

**Rappel sur les capabilities.**

Sous Linux, les privilèges de root sont divisés en capacités appelées capabilities.

Exemples :

```text
CAP_SYS_ADMIN
CAP_NET_ADMIN
CAP_CHOWN
CAP_SETUID
CAP_SETGID
CAP_KILL
CAP_DAC_OVERRIDE
```

Nous pouvons voir les capabilities du shell courant avec :

```bash
grep Cap /proc/$$/status
```

ou :

```bash
capsh --print
```

si l’outil est installé.

---

**Capabilities dans un user namespace.**

Quand nous sommes root dans un user namespace, nous pouvons avoir des capabilities à l’intérieur de ce namespace.

Mais ces capabilities ne valent pas nécessairement sur tout l’hôte.

Exemple :

```bash
unshare --user --map-root-user bash
id
grep Cap /proc/$$/status
```

Nous pouvons voir des capabilities effectives dans le namespace.

Mais cela ne veut pas dire que nous pouvons tout faire sur l’hôte.

---

**Exemple : monter un système de fichiers.**

Dans certains cas, être root dans un user namespace peut permettre certaines opérations qui seraient interdites autrement, par exemple monter certains types de systèmes de fichiers autorisés dans un namespace mount combiné.

Exemple pédagogique :

```bash
unshare --user --map-root-user --mount bash
mkdir -p /tmp/userns-mnt
mount -t tmpfs tmpfs /tmp/userns-mnt
```

Selon la distribution et la configuration du noyau, cela peut fonctionner.

Mais cela ne signifie pas que nous avons tous les droits root de l’hôte.

La portée des capabilities est limitée par le namespace user et par les règles du noyau.

---

**`CAP_SYS_ADMIN` : une capacité très large.**

`CAP_SYS_ADMIN` est souvent décrite comme une capability très puissante.

Elle couvre de nombreuses opérations système.

Dans un user namespace, avoir `CAP_SYS_ADMIN` à l’intérieur permet certaines actions dans les namespaces associés, mais pas nécessairement sur l’hôte.

Nous devons donc toujours demander :

```text
Dans quel user namespace cette capability est-elle effective ?
Sur quelles ressources s’applique-t-elle ?
Le namespace propriétaire de la ressource est-il le même ?
```

---

# 9.8. Mappings GID et difficulté avec les groupes

**Pourquoi le GID est parfois plus complexe.**

Le mapping des groupes peut être plus délicat que celui des utilisateurs.

Linux empêche certaines manipulations de groupes non autorisées, notamment pour éviter qu’un utilisateur obtienne des droits de groupe qu’il ne possède pas réellement.

C’est pour cela que l’écriture dans `gid_map` est encadrée.

---

**`setgroups`.**

Nous pouvons voir un fichier :

```bash
cat /proc/$$/setgroups
```

Dans certains cas, avant d’écrire dans `gid_map`, il faut désactiver `setgroups` :

```bash
echo deny > /proc/<PID>/setgroups
```

Cette règle évite qu’un processus non privilégié manipule des groupes de manière dangereuse.

Les outils comme `unshare --map-root-user` masquent souvent cette complexité pour les cas simples.

---

# 9.9. `/etc/subuid` et `/etc/subgid`

**Pourquoi avons-nous besoin de plages subordonnées ?.**

Pour un conteneur rootless complet, il ne suffit pas toujours de mapper uniquement :

```text
UID 0 interne → UID 1000 externe
```

Un conteneur peut contenir plusieurs utilisateurs internes :

```text
root
www-data
postgres
redis
app
```

Nous avons donc besoin d’une plage d’UID/GID externes réservés à l’utilisateur.

C’est le rôle de :

```bash
/etc/subuid
/etc/subgid
```

---

**Exemple de `/etc/subuid`.**

Nous pouvons lire :

```bash
cat /etc/subuid
```

Exemple :

```text
user:100000:65536
```

Cela signifie :

```text
L’utilisateur user peut utiliser la plage d’UID subordonnés
de 100000 à 165535
pour ses user namespaces.
```

Même principe pour :

```bash
cat /etc/subgid
```

---

**Mapping typique rootless.**

Un conteneur rootless peut utiliser un mapping comme :

```text
         0     100000      65536
```

Cela signifie :

```text
UID 0 dans le conteneur      → UID 100000 sur l’hôte
UID 1 dans le conteneur      → UID 100001 sur l’hôte
...
UID 65535 dans le conteneur  → UID 165535 sur l’hôte
```

Ainsi, les utilisateurs internes du conteneur correspondent à des UID non privilégiés sur l’hôte.

---

# 9.10. `newuidmap` et `newgidmap`

**Rôle de ces outils.**

Les outils :

```bash
newuidmap
newgidmap
```

permettent de configurer les mappings UID/GID d’un processus de manière contrôlée.

Ils vérifient notamment que l’utilisateur a le droit d’utiliser les plages indiquées dans :

```text
/etc/subuid
/etc/subgid
```

Ces outils sont utilisés par des systèmes comme :

- Podman rootless ;

- Buildah ;

- certains runtimes OCI rootless ;

- outils de sandboxing.


---

**Pourquoi ne pas écrire directement dans `uid_map` ?.**

Techniquement, les mappings sont exposés dans :

```bash
/proc/<PID>/uid_map
/proc/<PID>/gid_map
```

Mais l’écriture directe est soumise à des règles strictes.

Pour des mappings avancés, nous utilisons `newuidmap` et `newgidmap` afin d’éviter de donner des privilèges excessifs.

---

**Exemple conceptuel.**

Pour un processus cible `<PID>`, un outil rootless peut configurer :

```bash
newuidmap <PID> 0 100000 65536
newgidmap <PID> 0 100000 65536
```

Cela signifie :

```text
Dans le namespace du processus <PID>,
les UID/GID 0 à 65535 sont mappés vers 100000 à 165535 sur l’hôte.
```

En pratique, nous ne faisons pas toujours cela à la main. Les outils de conteneurs le font pour nous.

---

# 9.11. User namespace et conteneurs rootless

**Qu’est-ce qu’un conteneur rootless ?.**

Un conteneur rootless est un conteneur lancé sans privilèges root sur l’hôte.

L’utilisateur qui lance le conteneur est un utilisateur normal.

À l’intérieur du conteneur, le processus peut voir :

```text
uid=0(root)
```

Mais sur l’hôte, il correspond à un UID non privilégié ou à une plage subordonnée.

---

**Pourquoi c’est important ?.**

Les conteneurs rootless réduisent les risques.

Si un processus s’échappe partiellement de son conteneur, il n’arrive pas directement avec les droits root de l’hôte.

Cela ne rend pas le système invulnérable, mais cela réduit fortement certains impacts.

Nous retenons :

```text
Rootless ne veut pas dire sans risque.
Rootless veut dire sans root réel sur l’hôte.
```

---

**Podman rootless.**

Podman est souvent utilisé en mode rootless.

Quand nous lançons un conteneur rootless, Podman utilise notamment :

- user namespace ;

- mappings UID/GID ;

- cgroups adaptés ;

- réseau rootless ;

- stockage adapté à l’utilisateur ;

- runtime OCI.


Nous pouvons inspecter les mappings depuis l’hôte en retrouvant le PID du processus et en lisant :

```bash
cat /proc/<PID>/uid_map
cat /proc/<PID>/gid_map
```

---

**Docker rootless.**

Docker propose aussi un mode rootless.

Le principe reste similaire : le démon et les conteneurs fonctionnent sans privilèges root globaux.

Là encore, les user namespaces et les plages `/etc/subuid` et `/etc/subgid` sont essentiels.

---

# 9.12. User namespace et fichiers montés

**Le problème des bind mounts.**

Les user namespaces peuvent créer des surprises avec les volumes et bind mounts.

Exemple :

```text
Dans le conteneur :
un fichier appartient à root:root

Sur l’hôte :
le même fichier appartient à 100000:100000
```

Ou inversement :

```text
Sur l’hôte :
fichier appartient à UID 1000

Dans le conteneur :
ce fichier peut apparaître comme nobody ou comme un UID non mappé
```

Cela dépend du mapping.

---

**UID non mappés.**

Si un fichier appartient à un UID qui n’est pas mappé dans le namespace, il peut apparaître avec un identifiant spécial ou inattendu.

Dans certains cas, il peut apparaître comme :

```text
nobody
```

ou comme un UID numérique non résolu.

Cela peut provoquer des erreurs de permissions dans les conteneurs rootless.

---

**Diagnostic.**

Pour diagnostiquer, nous utilisons :

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

Nous comparons les UID/GID numériques.

Il faut raisonner numériquement, pas seulement avec les noms d’utilisateurs.

---

# 9.13. User namespace et sécurité

**Ce que le user namespace protège.**

Le user namespace permet de réduire les privilèges réels sur l’hôte.

Il permet notamment :

- d’avoir root dans le conteneur sans root sur l’hôte ;

- de limiter les effets d’une compromission ;

- de lancer des conteneurs sans démon root ;

- de déléguer certaines isolations à des utilisateurs non privilégiés.


C’est une brique importante de sécurité.

---

**Ce qu’il ne protège pas seul.**

Le user namespace ne suffit pas.

Nous devons aussi considérer :

- namespaces mount, PID, network, IPC ;

- cgroups ;

- capabilities ;

- seccomp ;

- AppArmor ou SELinux ;

- permissions de fichiers ;

- montages exposés ;

- vulnérabilités noyau ;

- configuration du runtime.


Un conteneur rootless avec un mauvais bind mount peut encore exposer des données sensibles.

---

**Risques historiques et surface d’attaque.**

Les user namespaces augmentent la quantité de code noyau accessible à des utilisateurs non privilégiés.

Certaines distributions ont longtemps désactivé ou restreint les user namespaces non privilégiés pour des raisons de sécurité.

Nous devons comprendre l’équilibre :

```text
Avantage : conteneurs rootless et isolation plus fine.
Risque : surface d’attaque noyau exposée aux utilisateurs non privilégiés.
```

La bonne décision dépend du contexte :

- poste développeur ;

- serveur mutualisé ;

- cluster Kubernetes ;

- environnement de production sensible ;

- machine personnelle ;

- système durci.


---

# 9.14. Interaction avec les autres namespaces

**User namespace comme socle de privilèges.**

Le user namespace est particulier, car il influence les droits sur les autres namespaces.

Créer certains namespaces ou effectuer certaines opérations peut être autorisé si nous avons les capabilities nécessaires dans le user namespace approprié.

Par exemple, nous pouvons combiner :

```bash
unshare --user --map-root-user --mount bash
```

Puis effectuer certains montages autorisés.

---

**Avec le namespace mount.**

Le user namespace permet parfois à un utilisateur non privilégié de créer un namespace mount et d’effectuer certains montages limités.

Exemple :

```bash
unshare --user --map-root-user --mount bash
mount -t tmpfs tmpfs /tmp/test
```

Selon la configuration, cela peut fonctionner sans `sudo`.

---

**Avec le namespace network.**

Créer un namespace réseau peut nécessiter des privilèges.

Mais combiné à un user namespace, certains environnements permettent :

```bash
unshare --user --map-root-user --net bash
```

Nous sommes alors root dans le user namespace et pouvons avoir certaines capabilities dans le namespace réseau créé.

Cependant, connecter ce namespace au réseau extérieur demande souvent des privilèges ou des outils spécifiques.

C’est pourquoi les conteneurs rootless utilisent des solutions particulières pour le réseau.

---

# 9.15. Expérience complète guidée

**Créer un user namespace root interne.**

Nous lançons :

```bash
unshare --user --map-root-user bash
```

Dans le shell :

```bash
echo "Identité dans le namespace :"
id

echo "Namespace user :"
readlink /proc/$$/ns/user

echo "Mapping UID :"
cat /proc/$$/uid_map

echo "Mapping GID :"
cat /proc/$$/gid_map
```

Nous observons que nous sommes root dans le namespace.

---

**Tester les limites de ce root.**

Toujours dans le namespace :

```bash
touch /tmp/test-userns
ls -ln /tmp/test-userns
```

Puis :

```bash
touch /root/test-userns
```

Nous devons généralement obtenir :

```text
Permission denied
```

Nous sommes root dans le namespace, mais pas root sur l’hôte.

---

**Comparer depuis l’hôte.**

Dans un autre terminal :

```bash
ls -ln /tmp/test-userns
```

Nous observons le propriétaire réel du fichier sur l’hôte.

Si le mapping est :

```text
0 1000 1
```

le fichier créé comme root dans le namespace apparaît souvent comme appartenant à l’UID `1000` sur l’hôte.

---

**Quitter.**

Nous quittons :

```bash
exit
```

Quand le dernier processus du namespace disparaît, le namespace user disparaît aussi, sauf s’il est maintenu par une référence.

---

# 9.16. Cas pratique : comprendre un problème de permissions rootless

**Situation.**

Nous lançons un conteneur rootless avec un volume monté depuis l’hôte.

L’application dans le conteneur affiche :

```text
Permission denied
```

Pourtant, dans le conteneur, elle semble tourner comme root.

Pourquoi ?

---

**Analyse.**

Nous devons vérifier :

Dans le conteneur :

```bash
id
ls -ln /data
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Sur l’hôte :

```bash
ls -ln /chemin/hote
cat /etc/subuid
cat /etc/subgid
```

Nous comparons les UID/GID numériques.

---

**Explication possible.**

Le fichier sur l’hôte appartient à :

```text
UID 1000
```

Mais dans le conteneur rootless, l’UID interne attendu correspond peut-être à :

```text
UID 100000 sur l’hôte
```

Le conteneur voit donc un propriétaire qui ne correspond pas à l’utilisateur interne attendu.

Nous devons ajuster :

- les permissions ;

- le propriétaire ;

- le mapping ;

- le mode de montage ;

- la configuration du runtime ;

- ou l’utilisateur utilisé dans le conteneur.


---

# 9.17. Pièges classiques

**Croire que root dans le namespace est root sur l’hôte.**

C’est l’erreur principale.

Nous devons toujours distinguer :

```text
root interne
root externe
```

Le root interne a des droits dans le namespace, pas nécessairement sur tout l’hôte.

---

**Raisonner avec les noms au lieu des UID numériques.**

Les noms comme `root`, `user`, `www-data` viennent de fichiers comme :

```text
/etc/passwd
/etc/group
```

Mais le noyau raisonne surtout avec des nombres.

Nous devons donc utiliser :

```bash
ls -ln
id -u
id -g
```

pour comprendre réellement les permissions.

---

**Oublier `/etc/subuid` et `/etc/subgid`.**

Les conteneurs rootless ont besoin de plages subordonnées.

Si elles sont absentes ou incorrectes, nous pouvons rencontrer :

- erreurs de lancement ;

- mappings incomplets ;

- problèmes de permissions ;

- impossibilité d’utiliser certains utilisateurs internes.


---

**Confondre user namespace et cgroups.**

Le user namespace concerne les identifiants et les privilèges.

Les cgroups concernent les ressources.

Être root dans un user namespace ne signifie pas que nous pouvons consommer CPU et mémoire sans limite.

Pour cela, nous devons regarder les cgroups.

---

**Oublier les capabilities.**

Même si nous sommes UID 0 dans un namespace, les opérations autorisées dépendent aussi des capabilities.

Nous devons vérifier :

```bash
grep Cap /proc/$$/status
```

ou :

```bash
capsh --print
```

---

# 9.19. Ce que nous devons retenir

Nous retenons les points suivants :

1. Le namespace user isole les UID et GID.

2. Il permet d’être root dans un namespace sans être root sur l’hôte.

3. Le namespace user d’un processus est visible avec `/proc/<PID>/ns/user`.

4. Les mappings UID/GID sont visibles dans `/proc/<PID>/uid_map` et `/proc/<PID>/gid_map`.

5. Une ligne de mapping contient : ID interne, ID externe, taille de plage.

6. `unshare --user --map-root-user bash` permet de créer un user namespace simple.

7. Root interne ne signifie pas root externe.

8. Les fichiers peuvent apparaître avec des propriétaires différents selon le namespace.

9. Nous devons raisonner avec les UID/GID numériques.

10. Les capabilities sont interprétées dans le contexte d’un user namespace.

11. `/etc/subuid` et `/etc/subgid` définissent les plages subordonnées disponibles.

12. `newuidmap` et `newgidmap` servent à configurer des mappings avancés.

13. Les conteneurs rootless reposent fortement sur les user namespaces.

14. Les bind mounts peuvent créer des problèmes de permissions avec les mappings.

15. Le user namespace réduit certains risques, mais ne remplace pas les autres mécanismes de sécurité.

16. Les user namespaces augmentent aussi la surface d’attaque noyau accessible aux utilisateurs non privilégiés.

17. Nous devons toujours analyser le mapping, les capabilities, les montages et les cgroups ensemble.


---

# Conclusion du chapitre 9

Nous avons étudié le namespace user, l’un des mécanismes les plus puissants et les plus subtils de Linux.

Nous savons maintenant qu’il permet d’isoler les identifiants utilisateurs et groupes, de créer un environnement où un processus se voit comme `root`, tout en restant mappé vers un utilisateur non privilégié sur l’hôte.

Nous avons compris le rôle central des fichiers `uid_map` et `gid_map`, des plages `/etc/subuid` et `/etc/subgid`, ainsi que des outils `newuidmap` et `newgidmap`.

Nous retenons surtout que le namespace user est une brique essentielle des conteneurs rootless. Il améliore la sécurité en évitant de donner les droits root réels à un conteneur, mais il introduit aussi une complexité importante autour des permissions, des fichiers montés, des capabilities et des mappings.

Dans le chapitre suivant, nous étudions le namespace cgroup, qui isole la vue des cgroups, et nous clarifions la différence entre le namespace cgroup et les cgroups eux-mêmes.

---

## Complément 2026 — rootless et ownership des autres namespaces

Un user namespace est spécial parce que les vérifications de capabilities des autres namespaces sont reliées au **user namespace propriétaire** de ces objets. Lorsqu'un processus crée en même temps un user namespace et, par exemple, un UTS ou mount namespace, le noyau crée d'abord le user namespace puis rattache les autres namespaces à celui-ci. C'est ce mécanisme qui permet à un utilisateur non privilégié d'obtenir des capacités puissantes **dans son environnement isolé** sans devenir root dans le user namespace initial de l'hôte.

Cette différence explique le fonctionnement de nombreux conteneurs rootless. Elle explique aussi pourquoi « je suis UID 0 dans le conteneur » ne signifie jamais automatiquement « je suis root sur l'hôte ».

---

# Chapitre 10 — Namespace cgroup

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace cgroup.

Ce namespace est souvent mal compris, parce que son nom ressemble beaucoup aux cgroups eux-mêmes. Pourtant, nous devons distinguer deux choses :

```text
Les cgroups limitent, mesurent et organisent les ressources.

Le namespace cgroup isole la vue que les processus ont de leur position dans l’arborescence des cgroups.
```

Autrement dit, le namespace cgroup ne limite pas directement la mémoire, le CPU ou les processus. Il masque ou transforme la manière dont un processus voit son rattachement aux cgroups.

À la fin de ce chapitre, nous savons :

- expliquer ce qu’est le namespace cgroup ;

- distinguer cgroup namespace et cgroups ;

- lire `/proc/self/cgroup` ;

- comprendre pourquoi cette vue est importante dans les conteneurs ;

- faire le lien avec Docker, Podman, systemd et Kubernetes ;

- comprendre les limites du namespace cgroup ;

- diagnostiquer la position d’un processus dans les cgroups.


---

# 10.1. Rappel : que sont les cgroups ?

**Définition générale.**

Les cgroups, ou _control groups_, sont un mécanisme du noyau Linux qui permet de regrouper des processus afin de :

- limiter leur consommation de ressources ;

- mesurer leur consommation ;

- organiser les processus par service, utilisateur, conteneur ou pod ;

- appliquer des politiques de contrôle.


Les cgroups peuvent contrôler ou mesurer plusieurs types de ressources :

```text
CPU
mémoire
I/O disque
nombre de processus
périphériques
pression de ressources
```

Nous utilisons les cgroups dès que nous lançons des services systemd, des conteneurs Docker, des pods Kubernetes ou des environnements avec limites de ressources.

---

**Exemple simple.**

Un processus peut être placé dans un cgroup qui limite sa mémoire à 512 Mio.

Dans ce cas, même si la machine possède 32 Gio de RAM, le processus ne peut pas dépasser la limite imposée par son cgroup.

S’il dépasse cette limite, il peut être tué par le noyau avec un événement de type OOM.

Nous voyons donc que les cgroups répondent à une question différente des namespaces :

```text
Namespaces : que voit le processus ?
Cgroups    : que peut consommer le processus ?
```

---

# 10.2. Pourquoi un namespace cgroup ?

**Le problème de la vue des cgroups.**

Sans namespace cgroup, un processus peut voir son chemin complet dans l’arborescence des cgroups de l’hôte.

Nous pouvons lire :

```bash
cat /proc/self/cgroup
```

Exemple possible sur l’hôte :

```text
0::/user.slice/user-1000.slice/session-3.scope
```

Dans un conteneur, sans isolation de cette vue, nous pourrions voir un chemin très révélateur :

```text
0::/system.slice/docker-8c5f2a9c4e7d9f.scope
```

ou encore :

```text
0::/kubepods.slice/kubepods-burstable.slice/kubepods-burstable-pod123456.slice/cri-containerd-abcdef.scope
```

Cette information révèle l’organisation interne de l’hôte ou du cluster.

Le namespace cgroup permet de présenter une vue plus propre et plus isolée.

---

**Ce que le namespace cgroup isole.**

Le namespace cgroup isole principalement la vue des chemins cgroups visibles dans :

```bash
/proc/<PID>/cgroup
/proc/self/cgroup
```

Dans un namespace cgroup isolé, un processus peut voir sa position comme :

```text
0::/
```

alors que, depuis l’hôte, le même processus est réellement placé plus profondément dans l’arborescence :

```text
0::/system.slice/docker-xxxx.scope
```

Le namespace cgroup ne déplace pas forcément le processus. Il change la racine apparente depuis laquelle le processus voit son rattachement.

---

**Ce que le namespace cgroup ne fait pas.**

Le namespace cgroup ne définit pas directement les limites de ressources.

Il ne dit pas :

```text
Ce processus a droit à 512 Mio de RAM.
Ce processus a droit à 2 CPU.
Ce processus peut créer 100 processus au maximum.
```

Ces limites sont définies par les cgroups eux-mêmes, via les contrôleurs cgroup.

Le namespace cgroup répond plutôt à :

```text
À quoi ressemble l’arborescence cgroup depuis le point de vue du processus ?
```

---

# 10.3. Observer le namespace cgroup courant

**Avec `/proc/<PID>/ns/cgroup`.**

Comme les autres namespaces, le namespace cgroup d’un processus est visible dans :

```bash
/proc/<PID>/ns/cgroup
```

Pour notre shell courant :

```bash
readlink /proc/$$/ns/cgroup
```

Exemple :

```text
cgroup:[4026531835]
```

Nous pouvons comparer avec le PID 1 :

```bash
readlink /proc/1/ns/cgroup
```

Si les valeurs sont identiques, notre shell partage le même namespace cgroup que le PID 1.

---

**Comparer deux processus.**

Nous pouvons comparer :

```bash
echo "Shell courant : $(readlink /proc/$$/ns/cgroup)"
echo "PID 1         : $(readlink /proc/1/ns/cgroup)"
```

Même valeur :

```text
Shell courant : cgroup:[4026531835]
PID 1         : cgroup:[4026531835]
```

Les deux processus partagent le même namespace cgroup.

Valeurs différentes :

```text
Shell courant : cgroup:[4026533000]
PID 1         : cgroup:[4026531835]
```

Les deux processus ont une vue cgroup différente.

---

# 10.4. Lire `/proc/self/cgroup`

**Commande de base.**

Nous lisons :

```bash
cat /proc/self/cgroup
```

Sur un système utilisant cgroups v2, nous pouvons voir :

```text
0::/user.slice/user-1000.slice/session-3.scope
```

Le format cgroups v2 est souvent :

```text
0::/chemin/du/cgroup
```

La partie importante est le chemin après le second `:`.

---

**Format avec cgroups v1.**

Sur un système avec cgroups v1 ou un mode hybride, nous pouvons voir plusieurs lignes :

```text
12:memory:/user.slice
11:cpu,cpuacct:/user.slice
10:pids:/user.slice
9:devices:/user.slice
```

Le format général est :

```text
hiérarchie:contrôleurs:chemin
```

Exemple :

```text
11:cpu,cpuacct:/docker/abcdef
```

Cela signifie que, pour les contrôleurs `cpu` et `cpuacct`, le processus est dans le cgroup `/docker/abcdef`.

---

**Interprétation.**

`/proc/self/cgroup` nous dit où le processus courant se situe dans les hiérarchies cgroup.

Mais attention :

```text
Il nous donne une vue.
Cette vue peut être modifiée par le namespace cgroup.
```

Depuis l’intérieur d’un conteneur, nous pouvons voir :

```text
0::/
```

Alors que depuis l’hôte, le même processus est dans un chemin plus détaillé.

---

# 10.5. Différence entre cgroups et namespace cgroup

**Les cgroups contrôlent les ressources.**

Les cgroups sont actifs même sans namespace cgroup isolé.

Ils peuvent définir :

```text
memory.max
cpu.max
pids.max
io.max
```

Exemples avec cgroups v2 :

```bash
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/pids.max
```

Ces fichiers décrivent les limites et consommations du cgroup courant.

---

**Le namespace cgroup contrôle la vue.**

Le namespace cgroup ne crée pas ces limites.

Il influence plutôt la manière dont le processus voit son chemin dans `/proc/self/cgroup`.

Nous pouvons résumer :

|Mécanisme|Rôle|
|---|---|
|cgroups|contrôlent et mesurent les ressources|
|namespace cgroup|isole la vue des chemins cgroups|
|`/proc/self/cgroup`|affiche la position du processus dans les cgroups|
|`/sys/fs/cgroup`|expose les contrôleurs et fichiers de limites|

---

**Analogie.**

Nous pouvons faire une analogie avec le namespace mount.

Le namespace mount ne crée pas les fichiers. Il change la manière dont les montages sont visibles.

De la même façon, le namespace cgroup ne crée pas les limites. Il change la manière dont l’arborescence cgroup est visible.

---

# 10.6. cgroups v1 et cgroups v2

**Pourquoi cette distinction compte ?.**

Pour comprendre le namespace cgroup, nous devons savoir que Linux a connu deux grandes versions de cgroups :

```text
cgroups v1
cgroups v2
```

Les deux modèles organisent les ressources différemment.

Les systèmes modernes utilisent de plus en plus cgroups v2, notamment avec systemd, Docker récent et Kubernetes récent.

---

**cgroups v1.**

Avec cgroups v1, chaque contrôleur peut avoir sa propre hiérarchie.

Nous pouvons avoir :

```text
memory → une hiérarchie
cpu    → une autre hiérarchie
pids   → une autre hiérarchie
```

Cela rend l’analyse parfois complexe.

Dans `/proc/self/cgroup`, nous voyons souvent plusieurs lignes.

Exemple :

```text
12:memory:/docker/abcdef
11:cpu,cpuacct:/docker/abcdef
10:pids:/docker/abcdef
```

---

**cgroups v2.**

Avec cgroups v2, nous avons une hiérarchie unifiée.

Dans `/proc/self/cgroup`, nous voyons souvent une seule ligne :

```text
0::/user.slice/user-1000.slice/session-3.scope
```

Les fichiers de contrôle se trouvent dans la hiérarchie unifiée, souvent montée sur :

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

# 10.7. Observer les limites cgroups v2

**Mémoire.**

Nous pouvons lire la mémoire consommée par le cgroup courant :

```bash
cat /sys/fs/cgroup/memory.current
```

La valeur est en octets.

Nous pouvons lire la limite :

```bash
cat /sys/fs/cgroup/memory.max
```

Sortie possible :

```text
max
```

Cela signifie qu’il n’y a pas de limite explicite à ce niveau.

Ou bien :

```text
536870912
```

Cela correspond à 512 Mio.

---

**CPU.**

Nous lisons :

```bash
cat /sys/fs/cgroup/cpu.max
```

Sortie possible :

```text
max 100000
```

Cela signifie qu’il n’y a pas de quota CPU explicite.

Autre exemple :

```text
200000 100000
```

Cela signifie que le cgroup peut utiliser 200000 microsecondes de CPU par période de 100000 microsecondes, soit environ 2 CPU.

---

**Nombre de processus.**

Nous lisons :

```bash
cat /sys/fs/cgroup/pids.current
cat /sys/fs/cgroup/pids.max
```

Exemple :

```text
pids.current → 42
pids.max     → 512
```

Cela signifie que le cgroup contient actuellement 42 processus ou threads, avec une limite à 512.

---

# 10.8. Créer un namespace cgroup avec `unshare`

**Expérience simple.**

Nous pouvons créer un namespace cgroup avec :

```bash
unshare --cgroup bash
```

Dans le shell :

```bash
readlink /proc/$$/ns/cgroup
cat /proc/self/cgroup
```

Nous comparons avec l’hôte dans un autre terminal :

```bash
readlink /proc/<PID_DU_SHELL>/ns/cgroup
readlink /proc/1/ns/cgroup
```

Nous devons voir que le namespace cgroup peut être différent.

---

**Permissions.**

Selon la distribution et la configuration, cette commande peut échouer :

```bash
unshare --cgroup bash
```

Erreur possible :

```text
unshare: unshare failed: Operation not permitted
```

Nous pouvons tester avec :

```bash
sudo unshare --cgroup bash
```

Ou dans certains cas :

```bash
unshare --user --map-root-user --cgroup bash
```

Les permissions dépendent fortement du système, de la version du noyau et des règles de sécurité.

---

**Ce que nous observons.**

Dans un nouveau namespace cgroup, le contenu de :

```bash
cat /proc/self/cgroup
```

peut devenir plus court ou présenter une racine différente.

Mais nous devons bien comprendre :

```text
Nous avons isolé la vue.
Nous n’avons pas nécessairement créé une nouvelle limite mémoire ou CPU.
```

---

# 10.9. Lien avec systemd

**systemd organise les processus en cgroups.**

Sur beaucoup de distributions modernes, systemd organise les services et sessions dans des cgroups.

Exemples :

```text
system.slice
user.slice
machine.slice
```

Nous pouvons voir l’état d’un service :

```bash
systemctl status ssh
```

ou :

```bash
systemctl status nginx
```

systemd affiche souvent le cgroup du service.

---

**`systemd-cgls`.**

Nous pouvons afficher l’arborescence cgroup avec :

```bash
systemd-cgls
```

Sortie conceptuelle :

```text
Control group /:
├─user.slice
│ └─user-1000.slice
│   └─session-3.scope
└─system.slice
  ├─ssh.service
  └─docker.service
```

Cette vue est très utile pour comprendre où se trouvent les processus.

---

**`systemd-cgtop`.**

Nous pouvons voir une vue dynamique avec :

```bash
systemd-cgtop
```

Cela montre la consommation par cgroup :

- CPU ;

- mémoire ;

- tâches ;

- I/O selon configuration.


C’est une vue plus orientée services que `top`.

---

**Limites systemd.**

systemd peut définir des limites de ressources pour un service.

Exemple dans une unité systemd :

```ini
[Service]
MemoryMax=512M
CPUQuota=200%
TasksMax=512
```

Ces paramètres sont appliqués via les cgroups.

Le namespace cgroup peut masquer une partie du chemin vu par le processus, mais les limites restent effectives.

---

# 10.12. Namespace cgroup et `/proc`

**`/proc/<PID>/cgroup`.**

Pour un processus donné :

```bash
cat /proc/<PID>/cgroup
```

Nous voyons sa position cgroup depuis notre perspective.

Dans certains cas, nous devons comparer :

```bash
cat /proc/<PID>/cgroup
sudo nsenter --target <PID> --cgroup cat /proc/self/cgroup
```

Cela permet de comprendre si la vue change avec le namespace cgroup.

---

**`/proc/<PID>/ns/cgroup`.**

Nous lisons :

```bash
readlink /proc/<PID>/ns/cgroup
```

Cela nous dit dans quel namespace cgroup se trouve le processus.

Deux processus qui ont le même lien symbolique partagent la même vue cgroup.

---

**`/sys/fs/cgroup`.**

Le répertoire :

```bash
/sys/fs/cgroup
```

expose les fichiers de contrôle cgroup.

Il dépend aussi du namespace mount et de ce qui est monté.

Dans un conteneur, la vue de `/sys/fs/cgroup` peut être :

- complète ;

- partielle ;

- en lecture seule ;

- masquée ;

- adaptée au cgroup du conteneur.


Nous devons donc croiser trois choses :

```text
/proc/self/cgroup        → position vue par le processus
/proc/self/ns/cgroup     → namespace cgroup
/sys/fs/cgroup           → interface de contrôle montée
```

---

# 10.13. Cas pratique : comprendre une limite mémoire

**Lancement d’un conteneur limité.**

Nous lançons :

```bash
docker run --rm -it --memory=256m alpine sh
```

Dans le conteneur :

```sh
cat /proc/self/cgroup
cat /sys/fs/cgroup/memory.max 2>/dev/null
cat /sys/fs/cgroup/memory.current 2>/dev/null
```

Sur cgroups v2, nous pouvons voir :

```text
memory.max     → 268435456
memory.current → valeur courante
```

Cela correspond à 256 Mio.

---

**Interprétation.**

Même si le conteneur voit parfois une vue cgroup simplifiée, la limite est bien appliquée par le noyau.

Nous distinguons donc :

```text
Vue cgroup dans /proc/self/cgroup
Limite effective dans les fichiers cgroup
```

Le namespace cgroup peut changer la première, mais pas supprimer la seconde.

---

# 10.14. Cas pratique : comparer hôte et conteneur

**Côté hôte.**

Nous lançons :

```bash
docker run --rm -d --name cgroup-demo --memory=512m alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' cgroup-demo)
```

Sur l’hôte :

```bash
echo "Namespace cgroup du conteneur :"
sudo readlink /proc/$pid/ns/cgroup

echo "Namespace cgroup de l'hôte :"
readlink /proc/1/ns/cgroup

echo "Vue cgroup depuis l'hôte :"
sudo cat /proc/$pid/cgroup
```

---

**Côté conteneur.**

Nous exécutons :

```bash
docker exec cgroup-demo sh -c '
echo "Vue depuis le conteneur :"
cat /proc/self/cgroup
echo
echo "Limite mémoire éventuelle :"
cat /sys/fs/cgroup/memory.max 2>/dev/null || true
'
```

Nous comparons les résultats.

Nous pouvons observer que :

- le chemin cgroup vu depuis l’hôte est plus détaillé ;

- le chemin vu depuis le conteneur peut être simplifié ;

- la limite mémoire reste visible ou appliquée selon le montage de `/sys/fs/cgroup`.


---

**Nettoyage.**

```bash
docker rm -f cgroup-demo
```

---

# 10.15. Pièges classiques

**Confondre cgroup namespace et cgroups.**

C’est le piège principal.

Nous devons retenir :

```text
Le namespace cgroup isole la vue.
Les cgroups contrôlent les ressources.
```

Créer un namespace cgroup ne limite pas automatiquement la mémoire, le CPU ou les processus.

---

**Croire que `/proc/self/cgroup` donne toujours le chemin réel de l’hôte.**

Dans un conteneur, `/proc/self/cgroup` peut être volontairement simplifié.

Nous ne devons pas supposer que ce chemin correspond au chemin complet de l’hôte.

Pour le voir, nous devons observer depuis l’hôte.

---

**Croire que `/sys/fs/cgroup` est toujours complet.**

Dans un conteneur, `/sys/fs/cgroup` peut être :

- absent ;

- partiellement monté ;

- en lecture seule ;

- limité au cgroup du conteneur ;

- différent selon cgroups v1 ou v2.


Nous devons donc toujours vérifier :

```bash
findmnt /sys/fs/cgroup
cat /proc/self/mountinfo | grep cgroup
```

---

**Confondre limites Kubernetes et namespace cgroup.**

Dans Kubernetes, les limites CPU/mémoire viennent des cgroups.

Le namespace cgroup peut masquer le chemin exact.

Mais si un pod est `OOMKilled`, c’est bien une limite cgroup qui est en jeu, pas le namespace cgroup lui-même.

---

# 10.16. Diagnostic cgroup

**Questions à se poser.**

Lorsque nous diagnostiquons un problème cgroup, nous demandons :

```text
Dans quel cgroup est le processus ?
Sommes-nous dans cgroups v1 ou v2 ?
Quelle est la limite mémoire ?
Quelle est la consommation mémoire courante ?
Quelle est la limite CPU ?
Quelle est la limite pids ?
La vue est-elle celle de l’hôte ou celle du conteneur ?
Le namespace cgroup masque-t-il le chemin réel ?
/sys/fs/cgroup est-il monté correctement ?
systemd organise-t-il ce processus ?
Kubernetes impose-t-il une limite ?
```

---

**Commandes utiles.**

Pour le namespace :

```bash
readlink /proc/<PID>/ns/cgroup
```

Pour la position cgroup :

```bash
cat /proc/<PID>/cgroup
cat /proc/self/cgroup
```

Pour cgroups v2 :

```bash
cat /sys/fs/cgroup/cgroup.controllers
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/pids.current
cat /sys/fs/cgroup/pids.max
```

Pour systemd :

```bash
systemd-cgls
systemd-cgtop
systemctl status <service>
```

Pour Docker :

```bash
docker stats
docker inspect <container>
```

Pour Kubernetes :

```bash
kubectl top pod
kubectl describe pod
kubectl get events
```

---

# 10.17. Sécurité et namespace cgroup

**Pourquoi masquer la vue cgroup ?.**

Masquer ou simplifier les chemins cgroups peut éviter de révéler :

- organisation de l’hôte ;

- noms de services ;

- identifiants de conteneurs ;

- structure Kubernetes ;

- chemins systemd ;

- informations sur le runtime.


Ce n’est pas une barrière de sécurité absolue, mais cela réduit l’exposition d’informations internes.

---

**Écriture dans `/sys/fs/cgroup`.**

Permettre à un conteneur d’écrire librement dans `/sys/fs/cgroup` peut être dangereux.

Un processus pourrait tenter de :

- modifier ses limites ;

- déplacer des processus ;

- créer des sous-cgroups ;

- perturber l’organisation attendue.


C’est pourquoi `/sys/fs/cgroup` est souvent monté en lecture seule ou limité dans les conteneurs.

---

**Le namespace cgroup ne remplace pas les politiques de sécurité.**

Nous devons toujours combiner :

```text
namespaces
cgroups
capabilities
seccomp
AppArmor ou SELinux
montages contrôlés
utilisateur non-root
root filesystem en lecture seule si possible
```

Le namespace cgroup est une couche parmi d’autres.

---

# 10.19. Ce que nous devons retenir

Nous retenons les points suivants :

1. Les cgroups contrôlent et mesurent les ressources.

2. Le namespace cgroup isole la vue des chemins cgroups.

3. Le namespace cgroup ne limite pas directement CPU, mémoire ou pids.

4. `/proc/<PID>/ns/cgroup` identifie le namespace cgroup d’un processus.

5. `/proc/self/cgroup` affiche la position cgroup vue par le processus.

6. Avec cgroups v2, nous avons souvent une hiérarchie unifiée.

7. Avec cgroups v1, nous pouvons avoir plusieurs hiérarchies par contrôleur.

8. `/sys/fs/cgroup` expose les fichiers de contrôle et de mesure.

9. systemd organise les services dans des cgroups.

10. Docker utilise les cgroups pour appliquer `--memory`, `--cpus` et `--pids-limit`.

11. Kubernetes traduit `requests` et `limits` en configuration cgroup sur les nœuds.

12. `OOMKilled` est lié au dépassement d’une limite mémoire cgroup.

13. Le namespace cgroup peut masquer le chemin réel de l’hôte.

14. Dans un conteneur, `/proc/self/cgroup` et `/sys/fs/cgroup` peuvent présenter une vue simplifiée ou limitée.

15. Nous devons toujours distinguer vue cgroup, limites effectives et montage de `/sys/fs/cgroup`.

16. Le namespace cgroup est utile pour l’isolation d’information, mais il ne remplace pas les cgroups eux-mêmes.


---

# Conclusion du chapitre 10

Nous avons étudié le namespace cgroup et clarifié sa relation avec les cgroups.

Nous savons maintenant que les cgroups sont le mécanisme qui contrôle et mesure les ressources, tandis que le namespace cgroup isole la vue qu’un processus a de son rattachement dans l’arborescence cgroup.

Cette distinction est essentielle. Quand un conteneur est limité à 512 Mio de RAM, ce n’est pas le namespace cgroup qui impose cette limite : ce sont les cgroups. Le namespace cgroup peut seulement faire apparaître le chemin du conteneur comme une racine plus propre ou plus isolée dans `/proc/self/cgroup`.

Nous avons également relié ce mécanisme à systemd, Docker et Kubernetes. Nous comprenons désormais pourquoi un service systemd, un conteneur Docker ou un pod Kubernetes apparaissent dans des cgroups, et pourquoi la vue observée depuis l’intérieur d’un conteneur peut différer de celle observée depuis l’hôte.

Dans le chapitre suivant, nous étudions le namespace time, qui permet d’isoler certains offsets d’horloges comme `CLOCK_MONOTONIC` et `CLOCK_BOOTTIME`.

---

# Chapitre 11 — Namespace time

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace time.

Le namespace time est plus récent et plus spécialisé que les namespaces UTS, PID, mount, network, IPC ou user. Il ne sert pas à donner une pile réseau ou une table de processus séparée. Il permet d’isoler certains décalages temporels vus par les processus.

Nous devons toutefois faire attention à une idée importante :

```text
Le namespace time ne permet pas simplement de changer l’heure globale du système.
```

Il concerne surtout certaines horloges monotones utilisées par les programmes pour mesurer des durées, des délais ou le temps écoulé depuis le démarrage.

À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace time ;

- distinguer heure réelle et horloges monotones ;

- comprendre les horloges `CLOCK_MONOTONIC` et `CLOCK_BOOTTIME` ;

- observer le namespace time avec `/proc/<PID>/ns/time` ;

- lire `/proc/self/timens_offsets` ;

- comprendre les cas d’usage : tests, checkpoint/restore, conteneurs avancés ;

- identifier les limites du namespace time.


---

# 11.1. Pourquoi isoler le temps ?

**Le temps comme dépendance système.**

Les programmes utilisent le temps en permanence.

Ils l’utilisent pour :

- mesurer une durée ;

- déclencher un timeout ;

- planifier une tâche ;

- calculer un délai d’expiration ;

- horodater un événement ;

- mesurer un temps de démarrage ;

- vérifier une période de validité ;

- gérer des caches ;

- calculer une latence.


Exemples :

```text
attendre 5 secondes
expirer une session après 30 minutes
mesurer une latence réseau
déterminer depuis combien de temps le système tourne
relancer un service après un délai
```

Dans un environnement isolé, il peut être utile qu’un groupe de processus voie certains temps de manière différente.

---

**Exemple de problème.**

Imaginons que nous voulions restaurer un processus qui a été figé puis relancé plus tard.

Le processus peut avoir enregistré des timers internes basés sur le temps monotone.

Si, lors de la restauration, le temps monotone vu par le processus a trop avancé, il peut croire que :

- tous ses timeouts ont expiré ;

- ses connexions sont trop anciennes ;

- ses tâches programmées sont en retard ;

- ses mesures internes sont incohérentes.


Le namespace time aide certains outils, notamment de checkpoint/restore, à fournir une vue temporelle plus cohérente.

---

# 11.2. Les différents types de temps sous Linux

**L’heure réelle.**

L’heure réelle correspond à l’heure du calendrier.

Nous pouvons l’afficher avec :

```bash
date
```

Exemple :

```text
dim. 24 mai 2026 22:35:12 CEST
```

Cette heure peut changer si :

- nous la réglons manuellement ;

- NTP la corrige ;

- la machine change de fuseau horaire ;

- l’horloge matérielle est ajustée.


Cette heure correspond à ce que les programmes utilisent souvent pour les dates humaines, les logs ou les certificats.

Le namespace time ne sert pas principalement à isoler cette heure réelle.

---

**Le temps monotone.**

Le temps monotone est une horloge qui ne recule pas.

Elle sert à mesurer des durées.

Par exemple, un programme peut dire :

```text
Je démarre un chronomètre maintenant.
Je relis l’horloge plus tard.
La différence me donne une durée.
```

Cette horloge est importante parce qu’elle n’est pas affectée de la même manière que l’heure réelle par les changements de date ou les corrections NTP.

Sous Linux, une horloge importante est :

```text
CLOCK_MONOTONIC
```

---

**Le temps depuis le démarrage.**

Une autre notion importante est le temps depuis le démarrage du système.

Nous pouvons lire :

```bash
cat /proc/uptime
```

Exemple :

```text
12345.67 89012.34
```

Le premier nombre indique le temps écoulé depuis le démarrage, en secondes.

Cette notion est proche des horloges monotones, mais plusieurs horloges existent avec des détails différents.

---

**`CLOCK_BOOTTIME`.**

`CLOCK_BOOTTIME` ressemble à `CLOCK_MONOTONIC`, mais elle inclut aussi le temps passé en suspension.

Elle est utile pour certains timers qui doivent tenir compte du temps réel écoulé même si la machine a été suspendue.

Nous retenons :

```text
CLOCK_MONOTONIC mesure un temps monotone.
CLOCK_BOOTTIME inclut aussi le temps passé en suspension.
```

Le namespace time permet d’appliquer des offsets à certaines de ces horloges.

---

# 11.3. Qu’est-ce que le namespace time ?

**Définition.**

Le namespace time permet d’isoler des offsets appliqués à certaines horloges vues par les processus.

Il concerne principalement :

```text
CLOCK_MONOTONIC
CLOCK_BOOTTIME
```

Cela signifie qu’un processus dans un namespace time peut voir une valeur monotone décalée par rapport à celle de l’hôte.

---

**Ce qu’il ne fait pas.**

Le namespace time ne permet pas simplement de faire :

```text
Dans ce conteneur, nous sommes le 1er janvier 2030.
Sur l’hôte, nous sommes le 24 mai 2026.
```

Pour l’heure réelle, les choses sont plus complexes et ne sont pas l’objectif principal du namespace time.

Nous ne devons donc pas le confondre avec :

- le fuseau horaire ;

- la commande `date`;

- le réglage NTP ;

- l’horloge matérielle ;

- les variables `TZ`;

- les bibliothèques qui simulent le temps.


---

**Pourquoi parler d’offset ?.**

Le namespace time ne crée pas une horloge entièrement indépendante.

Il applique un décalage, c’est-à-dire un offset, à certaines horloges.

Nous pouvons représenter cela ainsi :

```text
temps monotone vu dans le namespace
=
temps monotone de l’hôte
+
offset du namespace
```

L’offset peut être différent selon le namespace time.

---

# 11.4. Observer le namespace time courant

**Avec `/proc/<PID>/ns/time`.**

Comme les autres namespaces, le namespace time d’un processus est visible dans :

```bash
/proc/<PID>/ns/time
```

Pour notre shell courant :

```bash
readlink /proc/$$/ns/time
```

Exemple :

```text
time:[4026531834]
```

Nous comparons avec le PID 1 :

```bash
readlink /proc/1/ns/time
```

Si les deux valeurs sont identiques, notre shell partage le même namespace time que le PID 1.

---

**Namespace time pour les enfants.**

Sur les noyaux récents, nous pouvons aussi voir :

```bash
ls -l /proc/$$/ns/time_for_children
```

Exemple :

```text
time_for_children -> time:[4026531834]
```

Cette entrée indique le namespace time qui sera utilisé pour les processus enfants.

Ce détail est important parce que la création et la configuration du namespace time impliquent souvent les futurs enfants du processus.

---

# 11.5. Lire `/proc/self/timens_offsets`

**Le fichier `timens_offsets`.**

Le fichier principal pour observer les offsets du namespace time est :

```bash
/proc/self/timens_offsets
```

Nous pouvons lire :

```bash
cat /proc/self/timens_offsets
```

Sortie possible :

```text
monotonic           0         0
boottime            0         0
```

Les lignes correspondent aux horloges concernées.

---

**Interprétation.**

Une ligne ressemble à :

```text
monotonic           0         0
```

Elle indique un offset pour l’horloge monotone.

Les colonnes numériques correspondent au décalage en secondes et nanosecondes.

Par exemple, conceptuellement :

```text
monotonic           3600      0
```

signifierait que l’horloge monotone est décalée d’une heure pour les processus du namespace concerné.

---

**Lire les offsets du processus courant.**

Nous exécutons :

```bash
echo "Namespace time : $(readlink /proc/$$/ns/time)"
cat /proc/self/timens_offsets
```

Nous obtenons généralement des offsets nuls dans un shell normal.

---

# 11.6. Créer un namespace time

**Avec `unshare --time`.**

Nous pouvons tenter :

```bash
unshare --time bash
```

Dans le shell :

```bash
readlink /proc/$$/ns/time
cat /proc/self/timens_offsets
```

Selon le noyau, les permissions et la version de `util-linux`, cette commande peut fonctionner ou échouer.

Erreur possible :

```text
unshare: unshare failed: Operation not permitted
```

Dans ce cas, nous devons tester dans un environnement adapté ou avec des privilèges :

```bash
sudo unshare --time bash
```

---

**Avec user namespace.**

Sur certaines configurations, nous pouvons combiner avec un user namespace :

```bash
unshare --user --map-root-user --time bash
```

Puis :

```bash
id
readlink /proc/$$/ns/time
cat /proc/self/timens_offsets
```

Le comportement dépend fortement du noyau et de la configuration de sécurité.

Nous devons donc être plus prudents qu’avec les namespaces UTS ou PID.

---

**Pourquoi c’est moins visible ?.**

Changer le hostname dans un namespace UTS est immédiatement visible.

Créer un namespace PID est immédiatement visible avec `ps`.

Créer un namespace réseau est visible avec `ip addr`.

Le namespace time est plus subtil, car il ne change pas forcément une commande simple comme :

```bash
date
```

Son effet concerne certaines horloges utilisées par les programmes, pas forcément l’heure affichée à l’utilisateur.

---

# 11.7. Modifier les offsets temporels

**Écriture dans `timens_offsets`.**

Dans certains cas, les offsets peuvent être configurés en écrivant dans :

```bash
/proc/<PID>/timens_offsets
```

ou :

```bash
/proc/self/timens_offsets
```

Exemple conceptuel :

```bash
echo "monotonic 3600 0" > /proc/self/timens_offsets
```

ou :

```bash
echo "boottime 3600 0" > /proc/self/timens_offsets
```

Cela demande les permissions adéquates et doit être fait au bon moment.

---

**Attention au moment de configuration.**

Le namespace time a une contrainte importante : les offsets doivent généralement être configurés avant que le processus cible ou ses enfants n’utilisent réellement ce namespace de manière définitive.

En pratique, les outils spécialisés gèrent ce détail.

Pour un cours de niveau Master 2, nous devons surtout comprendre le principe :

```text
Le namespace time applique des offsets à certaines horloges.
La configuration est plus délicate que pour un simple hostname.
```

---

**Pourquoi nous ne faisons pas toujours la manipulation à la main.**

Selon le système, manipuler manuellement `timens_offsets` peut être difficile pour plusieurs raisons :

- noyau trop ancien ;

- commande `unshare` sans support complet ;

- permissions insuffisantes ;

- restrictions de sécurité ;

- moment d’écriture incorrect ;

- absence de besoin réel dans les environnements courants.


Nous étudions donc surtout le mécanisme et ses cas d’usage.

---

# 11.8. Mesurer les horloges en pratique

**Avec Python.**

Nous pouvons observer plusieurs notions temporelles avec Python :

```bash
python3 - <<'PY'
import time

print("time.time()        :", time.time())
print("time.monotonic()   :", time.monotonic())
print("time.perf_counter():", time.perf_counter())
PY
```

Interprétation :

- `time.time()` correspond à l’heure réelle Unix ;

- `time.monotonic()` utilise une horloge monotone ;

- `time.perf_counter()` sert à mesurer des performances et durées.


Le namespace time vise surtout les horloges monotones et boottime, pas simplement `time.time()`.

---

**Avec `/proc/uptime`.**

Nous lisons :

```bash
cat /proc/uptime
```

Puis :

```bash
sleep 2
cat /proc/uptime
```

Nous voyons que le premier nombre augmente.

Dans certains contextes avec namespace time, les valeurs liées au temps depuis le démarrage peuvent être influencées par les offsets.

---

**Comparaison hôte et namespace.**

Nous pouvons comparer :

Sur l’hôte :

```bash
readlink /proc/$$/ns/time
cat /proc/self/timens_offsets
cat /proc/uptime
```

Dans un namespace time :

```bash
unshare --time bash
readlink /proc/$$/ns/time
cat /proc/self/timens_offsets
cat /proc/uptime
```

Selon les offsets configurés, les valeurs peuvent être identiques ou décalées.

---

# 11.9. Cas d’usage : checkpoint/restore

**Qu’est-ce que checkpoint/restore ?.**

Le checkpoint/restore consiste à figer l’état d’un processus, puis à le restaurer plus tard.

Un outil connu dans cet écosystème est CRIU, _Checkpoint/Restore In Userspace_.

L’objectif est de pouvoir :

- migrer un processus ;

- sauvegarder son état ;

- restaurer un conteneur ;

- déplacer une charge ;

- suspendre puis reprendre une application.


---

**Pourquoi le temps pose problème ?.**

Un processus peut contenir des timers internes.

Exemple :

```text
timer A expire dans 10 secondes
connexion B expire dans 30 secondes
cache C expire dans 5 minutes
```

Si nous figeons le processus et le restaurons longtemps après, le temps monotone vu par le processus peut ne plus correspondre à ce qu’il attend.

Le namespace time permet d’ajuster la vue de certaines horloges pour rendre la restauration plus cohérente.

---

**Conteneurs migrables.**

Dans des scénarios avancés, nous pouvons vouloir migrer un conteneur d’un hôte à un autre.

Pour cela, il faut reconstruire une vue cohérente de plusieurs ressources :

- processus ;

- fichiers ;

- réseau ;

- mémoire ;

- cgroups ;

- temps.


Le namespace time ajoute une brique utile à cette cohérence.

---

# 11.10. Cas d’usage : tests

**Tester du code dépendant du temps.**

Beaucoup de logiciels dépendent du temps.

Exemples :

- expiration de session ;

- retry avec backoff ;

- timeout réseau ;

- expiration de cache ;

- calcul de durée ;

- ordonnanceur de tâches ;

- certificats ;

- tokens ;

- licences.


Pour tester ces comportements, nous avons parfois besoin de simuler un temps différent.

---

**Limite du namespace time pour les tests applicatifs.**

Le namespace time ne remplace pas toujours les bibliothèques de test temporel.

Par exemple, si nous voulons tester :

```text
Nous sommes le 1er janvier 2030.
```

le namespace time n’est pas forcément l’outil adapté.

Nous pouvons utiliser :

- injection d’horloge dans le code ;

- bibliothèques de mock du temps ;

- variables de configuration ;

- simulateurs ;

- outils comme `libfaketime` dans certains cas.


Le namespace time est plus bas niveau et concerne surtout certaines horloges noyau.

---

# 11.13. Sécurité du namespace time

**Pourquoi isoler le temps peut être sensible.**

Le temps influence beaucoup de comportements :

- expiration de tokens ;

- délais de sécurité ;

- logs ;

- timeouts ;

- mécanismes anti-rejeu ;

- supervision ;

- mesure de performance ;

- ordonnancement.


Une vue temporelle incorrecte peut perturber une application.

---

**Le namespace time n’est pas une barrière de sécurité suffisante.**

Comme les autres namespaces, il ne suffit pas seul.

Nous devons le combiner avec :

- user namespace ;

- mount namespace ;

- cgroups ;

- capabilities ;

- seccomp ;

- AppArmor ou SELinux ;

- configuration correcte du runtime.


---

**Attention aux logs et à l’observabilité.**

Si différents environnements voient des temps différents, les logs peuvent devenir plus difficiles à corréler.

Exemple :

```text
Hôte : événement à 10:00
Processus isolé : durée monotone décalée
Outil de monitoring : métrique interprétée différemment
```

Nous devons savoir quelles horloges sont utilisées par nos outils.

---

# 11.14. Différence avec le fuseau horaire

**Fuseau horaire.**

Le fuseau horaire est souvent contrôlé par :

```text
/etc/localtime
variable TZ
bibliothèques libc
configuration applicative
```

Exemple :

```bash
TZ=UTC date
TZ=Europe/Paris date
```

Cela change l’affichage local de l’heure, pas le namespace time.

---

**Ne pas confondre.**

Nous distinguons :

|Mécanisme|Rôle|
|---|---|
|Fuseau horaire|change l’affichage local de l’heure|
|`date` système|affiche ou règle l’heure réelle|
|NTP|synchronise l’heure réelle|
|namespace time|applique des offsets à certaines horloges monotonic/boottime|
|mock applicatif|simule le temps dans le code|

Cette distinction est essentielle pour éviter de choisir le mauvais outil.

---

# 11.15. Diagnostic autour du namespace time

**Questions à se poser.**

Lorsque nous analysons un problème lié au temps, nous demandons :

```text
L’application utilise-t-elle l’heure réelle ou une horloge monotone ?
Le problème concerne-t-il une date humaine ou une durée ?
Le processus est-il dans un namespace time différent ?
Quels offsets sont configurés ?
Le fuseau horaire est-il en cause ?
NTP corrige-t-il l’heure ?
Sommes-nous dans un conteneur ?
L’application utilise-t-elle des mocks ou une bibliothèque temporelle ?
```

---

**Commandes utiles.**

Observer le namespace time :

```bash
readlink /proc/$$/ns/time
readlink /proc/1/ns/time
```

Observer les offsets :

```bash
cat /proc/self/timens_offsets
```

Observer l’heure réelle :

```bash
date
timedatectl
```

Observer l’uptime :

```bash
cat /proc/uptime
uptime
```

Tester différentes horloges en Python :

```bash
python3 - <<'PY'
import time
print("time.time()     =", time.time())
print("time.monotonic()=", time.monotonic())
PY
```

---

# 11.16. Expérience guidée

**Observer le namespace time actuel.**

Nous exécutons :

```bash
echo "Namespace time courant :"
readlink /proc/$$/ns/time

echo "Namespace time du PID 1 :"
readlink /proc/1/ns/time

echo "Offsets temporels :"
cat /proc/self/timens_offsets
```

Nous répondons :

```text
Partageons-nous le même namespace time que le PID 1 ?
Les offsets sont-ils nuls ?
```

---

**Créer un namespace time.**

Nous testons :

```bash
unshare --time bash
```

Dans le shell :

```bash
readlink /proc/$$/ns/time
cat /proc/self/timens_offsets
date
cat /proc/uptime
```

Puis nous quittons :

```bash
exit
```

Si la commande échoue, nous notons l’erreur et nous testons éventuellement dans un environnement autorisé :

```bash
sudo unshare --time bash
```

---

**Comparer avec Python.**

Dans le shell normal :

```bash
python3 - <<'PY'
import time
print("time.time()     =", time.time())
print("time.monotonic()=", time.monotonic())
PY
```

Dans le namespace time :

```bash
unshare --time bash
python3 - <<'PY'
import time
print("time.time()     =", time.time())
print("time.monotonic()=", time.monotonic())
PY
exit
```

Si aucun offset n’est configuré, les différences ne seront pas spectaculaires.

L’intérêt est surtout de comprendre quelles horloges existent et lesquelles peuvent être concernées.

---

# 11.17. Limites du namespace time

**Il est moins général qu’on l’imagine.**

Le namespace time ne permet pas de tout simuler.

Il ne remplace pas :

- une horloge système indépendante ;

- une machine virtuelle ;

- un mock applicatif ;

- une configuration de fuseau horaire ;

- NTP ;

- une simulation complète du calendrier.


Il concerne surtout certains offsets d’horloges noyau.

---

**Il dépend du noyau.**

Le namespace time n’est disponible que sur des noyaux suffisamment récents.

Sur des systèmes anciens, il peut être absent.

Nous vérifions :

```bash
ls -l /proc/$$/ns/time
```

Si ce fichier n’existe pas, le système ne supporte probablement pas le namespace time.

---

**Il dépend des permissions.**

Même si le noyau le supporte, la création ou la modification des offsets peut être interdite.

Nous pouvons obtenir :

```text
Operation not permitted
```

C’est normal sur des systèmes durcis ou non configurés pour cela.

---

**Il est peu utilisé dans les usages courants.**

Un développeur Docker classique manipule souvent :

- mount ;

- PID ;

- network ;

- UTS ;

- IPC ;

- user.


Il manipule beaucoup plus rarement le namespace time.

Cela ne le rend pas inutile, mais il reste plus spécialisé.

---

# 11.18. Pièges classiques

**Croire que `date` change.**

Créer un namespace time ne signifie pas nécessairement que :

```bash
date
```

affiche une date différente.

Pour modifier l’affichage de `date`, nous parlons plutôt :

- heure réelle ;

- fuseau horaire ;

- variable `TZ`;

- configuration système ;

- mocks applicatifs.


---

**Confondre durée et date.**

Le namespace time est plus pertinent pour des durées et des horloges monotones que pour des dates humaines.

Nous devons distinguer :

```text
date humaine : 24 mai 2026
durée : 12,5 secondes
temps monotone : compteur qui ne recule pas
```

---

**Ignorer l’impact sur les logs.**

Si une application mélange plusieurs sources de temps, les logs et mesures peuvent devenir difficiles à interpréter.

Nous devons toujours savoir quelle horloge est utilisée.

---

# 11.20. Ce que nous devons retenir

Nous retenons les points suivants :

1. Le namespace time isole des offsets appliqués à certaines horloges.

2. Il concerne principalement `CLOCK_MONOTONIC` et `CLOCK_BOOTTIME`.

3. Il ne permet pas simplement de changer l’heure réelle du système.

4. L’heure réelle s’observe avec `date` et dépend d’autres mécanismes.

5. Le fuseau horaire est différent du namespace time.

6. Le namespace time d’un processus est visible avec `/proc/<PID>/ns/time`.

7. Les offsets sont visibles avec `/proc/self/timens_offsets`.

8. Les offsets sont exprimés en secondes et nanosecondes.

9. La manipulation du namespace time dépend du noyau et des permissions.

10. Le namespace time est particulièrement utile pour checkpoint/restore.

11. Il peut aussi servir à certains tests système avancés.

12. Il est moins utilisé dans les conteneurs courants que les namespaces PID, mount ou network.

13. Pour tester des dates applicatives, un mock d’horloge est souvent plus adapté.

14. Nous devons toujours distinguer date, durée, uptime, horloge monotone et fuseau horaire.

15. Le namespace time est une brique spécialisée d’isolation, pas une virtualisation complète du temps.


---

# Conclusion du chapitre 11

Nous avons étudié le namespace time, un namespace plus spécialisé que ceux vus précédemment.

Nous savons maintenant qu’il ne sert pas simplement à changer la date visible avec `date`, mais à appliquer des offsets à certaines horloges utilisées pour mesurer des durées, notamment `CLOCK_MONOTONIC` et `CLOCK_BOOTTIME`.

Nous avons compris pourquoi ce mécanisme est utile pour des scénarios avancés comme le checkpoint/restore, la migration de conteneurs ou certains tests système. Nous avons aussi vu ses limites : il dépend du noyau, des permissions, et ne remplace pas les mécanismes applicatifs de simulation du temps.

Dans le chapitre suivant, nous faisons le lien entre les namespaces et `/proc`, afin de comprendre comment `/proc/<PID>/ns`, `/proc/net`, `/proc/mounts`, `/proc/self/cgroup`, `uid_map` et `gid_map` reflètent les différentes vues isolées du système.

---

## Complément 2026 — `time` et `time_for_children`

Comme pour le PID namespace, `unshare --time` prépare le namespace temporel utilisé par les **enfants** créés ensuite ; le processus appelant ne change pas rétroactivement de vue. C'est pourquoi `/proc/<PID>/ns/time` et `/proc/<PID>/ns/time_for_children` peuvent être différents.

Le time namespace ne virtualise pas arbitrairement l'heure civile. Il permet principalement de décaler `CLOCK_MONOTONIC` et `CLOCK_BOOTTIME`, ce qui est utile pour le checkpoint/restore et certains tests reproductibles.

---

# Chapitre 12 — `/proc`, namespace FDs, pidfds et inspection avancée

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

---

# Chapitre 13 — Namespaces, capabilities, seccomp et LSM

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

---

# Chapitre 14 — Des namespaces aux conteneurs

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

---

# Chapitre 15 — Construire un mini-conteneur pédagogique

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

---

# Chapitre 16 — Idmapped mounts et stockage rootless moderne

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

---

# Chapitre 17 — Diagnostic méthodique

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

---

# Chapitre 18 — API C minimale

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

---

# Chapitre 19 — Sécurité : modèle de menace

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

---

# Chapitre 20 — Bonnes pratiques d'administration

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

---

# Chapitre 21 — Erreurs classiques

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

---

# Chapitre 22 — Travaux pratiques

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

---

# Chapitre 23 — Projet final : expliquer un conteneur à partir du noyau

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

---

# Chapitre 24 — Aide-mémoire

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

---

# Chapitre 25 — Questions de révision

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

---

# Conclusion générale

Les namespaces deviennent beaucoup plus simples lorsqu'on cesse de les imaginer comme des « mini-machines virtuelles ». Ils répondent à une question plus précise : **quelle vue d'une ressource du noyau ce processus reçoit-il ?**

Cette question se décline ensuite : quels PID voit-il ? quels montages ? quelle pile réseau ? quelles identités ? quelles ressources IPC ? quelle racine cgroup ? quelles horloges ? Une fois ces vues comprises, nous pouvons ajouter le second axe, celui des **limites** avec cgroup v2, puis le troisième, celui de la **réduction de privilèges** avec capabilities, seccomp, `no_new_privs` et LSM.

Un conteneur Linux est précisément l'assemblage cohérent de ces mécanismes autour d'un processus et d'un système de fichiers. Docker, Podman, systemd ou Kubernetes automatisent cet assemblage ; les namespaces restent les primitives qui permettent d'en comprendre le comportement et, surtout, de le diagnostiquer lorsqu'il ne correspond pas à ce que nous attendons.

# Références principales

- `namespaces(7)` et pages spécialisées de Linux man-pages ;
- `clone(2)`, `clone3(2)`, `unshare(2)`, `setns(2)` ;
- `unshare(1)`, `nsenter(1)`, `lsns(8)` dans util-linux ;
- documentation du noyau Linux sur les namespaces, les cgroups et les idmapped mounts ;
- documentation `ip-netns(8)` et `network_namespaces(7)` pour le réseau.
