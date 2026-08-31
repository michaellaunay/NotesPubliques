---
schema_version: 1
uid: 01M1BQ624CDXM2F149W0XC5HXM
titre: "Les namespaces Linux — 06 — Namespace mount"
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
resume: "Chapitre 6 sur 16 du livre « Les namespaces Linux » : Namespace mount. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 6/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 05 — Namespace PID|← 05 — Namespace PID]] · [[Les namespaces Linux — 07 — Namespace network|07 — Namespace network →]]

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

## 6.1. Pourquoi isoler les montages ?

## 6.1.1. Le problème de départ

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

## 6.1.2. Ce que le namespace mount isole

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

## 6.2. Arborescence de fichiers et points de montage

## 6.2.1. L’arborescence de fichiers

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

## 6.2.2. Le point de montage

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

## 6.2.3. Le rôle du namespace mount

Sans namespace mount, un nouveau montage peut être visible par les autres processus de l’hôte.

Avec un namespace mount isolé, nous pouvons monter quelque chose dans notre environnement sans le rendre visible ailleurs, à condition de gérer correctement la propagation des montages.

C’est ce qui permet à un conteneur d’avoir ses propres montages sans polluer l’hôte.

---

## 6.3. Observer le namespace mount courant

## 6.3.1. Avec `/proc/<PID>/ns/mnt`

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
mnt:
```

Pour le PID 1 :

```bash
readlink /proc/1/ns/mnt
```

Si les deux valeurs sont identiques, notre shell partage le même namespace mount que le PID 1.

---

## 6.3.2. Comparer deux processus

Nous pouvons comparer :

```bash
echo "Shell courant : $(readlink /proc/$$/ns/mnt)"
echo "PID 1         : $(readlink /proc/1/ns/mnt)"
```

Même valeur :

```text
Shell courant : mnt:
PID 1         : mnt:
```

Les deux processus partagent le même namespace mount.

Valeurs différentes :

```text
Shell courant : mnt:
PID 1         : mnt:
```

Les deux processus ont des vues de montages différentes.

---

## 6.3.3. Observer les montages visibles

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

## 6.4. Créer un namespace mount avec `unshare`

## 6.4.1. Première expérience

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

## 6.4.2. Monter un tmpfs dans le namespace

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

## 6.4.3. Vérifier depuis l’hôte

Depuis un autre terminal sur l’hôte :

```bash
findmnt /tmp/ns-mount-demo
ls /tmp/ns-mount-demo
```

Selon la propagation des montages de votre système, le montage peut être visible ou non depuis l’hôte.

C’est un point essentiel : créer un namespace mount ne suffit pas toujours à éviter la propagation. Il faut comprendre la propagation des montages.

---

## 6.5. Le piège de la propagation des montages

## 6.5.1. Pourquoi ce piège existe ?

Sur beaucoup de distributions modernes, certains montages sont configurés en mode `shared`.

Cela signifie qu’un montage créé dans un namespace peut être propagé vers un autre namespace qui partage le même groupe de propagation.

Nous pourrions croire :

```text
J’ai créé un namespace mount, donc mes montages sont automatiquement invisibles ailleurs.
```

Ce n’est pas toujours vrai.

La propagation des montages peut transmettre certains événements de montage entre namespaces.

---

## 6.5.2. Vérifier la propagation

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

## 6.5.3. Rendre les montages privés

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

## 6.6. Les types de propagation

## 6.6.1. Vue d’ensemble

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

## 6.6.2. `shared`

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

## 6.6.3. `private`

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

## 6.6.4. `slave`

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

## 6.6.5. `unbindable`

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

## 6.6.6. Synthèse

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

## 6.7. Bind mounts

## 6.7.1. Définition

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

## 6.7.2. Utilité dans les conteneurs

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

## 6.7.3. Bind mount en lecture seule

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

## 6.7.4. Risques des bind mounts

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

## 6.8. Monter `/proc` dans un namespace mount

## 6.8.1. Pourquoi monter `/proc` ?

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

## 6.8.2. `/proc` dans un conteneur

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

## 6.8.3. Risques liés à `/proc`

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

## 6.9. Monter `/sys` et `/dev`

## 6.9.1. `/sys`

`/sys` expose `sysfs`, qui donne accès à des informations sur :

- périphériques ;
    
- pilotes ;
    
- bus ;
    
- modules noyau ;
    
- classes de périphériques ;
    
- paramètres matériels.
    

Dans un conteneur, exposer `/sys` trop largement peut être dangereux.

Montage à éviter sans besoin fort :

```bash
docker run -v /sys:/sys image
```

Un conteneur standard devrait avoir une vue limitée et contrôlée.

---

## 6.9.2. `/dev`

`/dev` contient les fichiers de périphériques.

Exemples :

```text
/dev/null
/dev/zero
/dev/random
/dev/sda
/dev/tty
/dev/kvm
/dev/dri
```

Donner accès à certains périphériques peut donner beaucoup de pouvoir.

Exemples sensibles :

```text
/dev/sda     → disque brut
/dev/kvm     → virtualisation
/dev/mem     → mémoire physique, si présent
/dev/docker  → selon les cas
```

Les conteneurs doivent avoir un `/dev` minimal et contrôlé.

---

## 6.9.3. Conteneurs privilégiés

Un conteneur lancé avec :

```bash
docker run --privileged ...
```

peut recevoir beaucoup plus de droits, de capabilities et d’accès aux périphériques.

Cela réduit fortement l’isolation.

Nous retenons :

```text
Le namespace mount aide à isoler l’arborescence, mais les montages exposés déterminent une grande partie de la sécurité réelle.
```

---

## 6.10. `chroot` vs namespace mount

## 6.10.1. Qu’est-ce que `chroot` ?

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

## 6.10.2. Limites de `chroot`

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

## 6.10.3. Namespace mount et `chroot`

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

## 6.11. `pivot_root`

## 6.11.1. Rôle de `pivot_root`

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

## 6.11.2. Pourquoi ne pas utiliser seulement `chroot` ?

`pivot_root` est plus adapté à la construction d’un environnement complet car il agit au niveau des montages.

Il permet de remplacer réellement la racine dans le namespace mount.

Les runtimes de conteneurs utilisent plutôt des mécanismes de ce type, avec une préparation soigneuse des montages.

---

## 6.11.3. Attention

`pivot_root` est plus complexe à manipuler que `chroot`.

Il impose des contraintes :

- nouvelle racine sur un point de montage ;
    
- ancien root placé sous la nouvelle racine ;
    
- gestion correcte des répertoires courants ;
    
- démontage propre de l’ancien root ;
    
- permissions nécessaires.
    

Nous l’étudions surtout pour comprendre les conteneurs, pas comme commande de base à utiliser quotidiennement.

---

## 6.12. Root filesystem d’un conteneur

## 6.12.1. Qu’est-ce qu’un root filesystem ?

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

## 6.12.2. Images en couches

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

## 6.12.3. Volumes et bind mounts

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

## 6.13. Construire une mini-isolation de fichiers

## 6.13.1. Préparer un rootfs minimal

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

## 6.13.2. Entrer avec un namespace mount et `chroot`

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

## 6.13.3. Ajouter `/proc`

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

## 6.14. Namespace mount et Docker

## 6.14.1. Ce que Docker fait

Quand nous lançons :

```bash
docker run --rm -it alpine sh
```

Docker prépare un namespace mount dans lequel :

- le rootfs de l’image Alpine est visible comme `/` ;
    
- `/proc` est monté ;
    
- `/sys` peut être monté avec restrictions ;
    
- `/dev` est préparé ;
    
- certains fichiers comme `/etc/resolv.conf`, `/etc/hosts`, `/etc/hostname` sont montés ou générés ;
    
- les volumes demandés sont montés ;
    
- certains chemins peuvent être en lecture seule selon la configuration.
    

Tout cela repose largement sur le namespace mount.

---

## 6.14.2. Observer les montages d’un conteneur

Dans un conteneur :

```sh
mount
cat /proc/self/mountinfo
findmnt
```

Depuis l’hôte, nous pouvons trouver le PID du conteneur :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' <container>)
```

Puis observer :

```bash
sudo readlink /proc/$pid/ns/mnt
sudo cat /proc/$pid/mountinfo | head
```

Nous pouvons aussi entrer dans son namespace mount :

```bash
sudo nsenter --target "$pid" --mount bash
```

Puis :

```bash
findmnt
```

---

## 6.14.3. Bind mounts Docker

Exemple :

```bash
docker run --rm -it -v "$PWD":/app alpine sh
```

Dans le conteneur :

```sh
ls /app
```

Nous voyons le contenu du répertoire courant de l’hôte.

C’est un bind mount.

Si nous voulons le rendre en lecture seule :

```bash
docker run --rm -it -v "$PWD":/app:ro alpine sh
```

Dans le conteneur, une tentative d’écriture dans `/app` doit échouer.

---

## 6.15. Namespace mount et Kubernetes

## 6.15.1. Volumes Kubernetes

Kubernetes utilise aussi les montages.

Dans un pod, nous pouvons définir :

```yaml
volumes:
  - name: config
    configMap:
      name: app-config

containers:
  - name: app
    image: example/app
    volumeMounts:
      - name: config
        mountPath: /etc/app/config
```

Le conteneur voit un montage à :

```text
/etc/app/config
```

Ce montage est préparé par Kubernetes et le runtime.

---

## 6.15.2. Secrets et ConfigMaps

Les Secrets et ConfigMaps Kubernetes sont souvent exposés sous forme de fichiers montés dans le conteneur.

Exemple :

```text
/etc/secrets/password
/etc/config/application.yaml
```

C’est pratique, mais cela signifie que les secrets existent dans l’arborescence de fichiers du conteneur.

Nous devons gérer :

- permissions ;
    
- accès applicatif ;
    
- rotation ;
    
- logs ;
    
- sauvegardes ;
    
- exposition accidentelle.
    

---

## 6.15.3. `hostPath`

Kubernetes permet de monter un chemin de l’hôte dans un pod avec `hostPath`.

Exemple :

```yaml
volumes:
  - name: host-logs
    hostPath:
      path: /var/log
```

C’est puissant, mais risqué.

Un `hostPath` mal choisi peut exposer l’hôte au conteneur.

Nous retenons :

```text
hostPath est l’équivalent Kubernetes d’un bind mount sensible depuis l’hôte.
```

---

## 6.16. Sécurité du namespace mount

## 6.16.1. Le namespace mount ne suffit pas

Un namespace mount peut isoler la vue des montages, mais la sécurité dépend de ce qui est monté.

Si nous montons toute la racine de l’hôte dans le conteneur :

```bash
docker run -v /:/host image
```

l’isolation du rootfs est largement compromise.

Même si le processus a une racine différente, il peut accéder à `/host`.

---

## 6.16.2. Montages sensibles

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

## 6.16.3. Lecture seule

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

## 6.16.4. Root filesystem en lecture seule

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

## 6.17. Diagnostic avec le namespace mount

## 6.17.1. Quel montage voit ce processus ?

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

## 6.17.2. Pourquoi mon fichier n’est-il pas visible ?

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

## 6.17.3. Pourquoi mon montage apparaît sur l’hôte ?

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

## 6.18. Expérience complète guidée

## 6.18.1. Créer un namespace mount isolé

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

## 6.18.2. Créer un montage visible seulement dans le namespace

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

## 6.18.3. Observer depuis l’hôte avec `nsenter`

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

## 6.18.4. Nettoyage

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

## 6.19. Pièges classiques

## 6.19.1. Croire que `unshare --mount` isole tout automatiquement

`unshare --mount` crée un nouveau namespace mount, mais la propagation peut encore produire des effets inattendus.

Pour un TP propre :

```bash
mount --make-rprivate /
```

---

## 6.19.2. Confondre fichiers et montages

Deux processus peuvent voir le même fichier sur disque, mais avec des montages différents.

Inversement, un même chemin peut pointer vers des contenus différents selon le namespace mount.

Exemple :

```text
/tmp/data dans le namespace A → tmpfs A
/tmp/data dans le namespace B → répertoire réel de l’hôte
```

Même chemin, réalité différente.

---

## 6.19.3. Monter par-dessus un répertoire existant

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

## 6.19.4. Confondre `chroot` et conteneur

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

## 6.19.5. Exposer le socket Docker

Le montage suivant est extrêmement sensible :

```bash
-v /var/run/docker.sock:/var/run/docker.sock
```

Un processus dans le conteneur peut alors contrôler Docker sur l’hôte, ce qui revient souvent à obtenir un pouvoir très important sur la machine.

Nous évitons ce montage sauf besoin très précis et parfaitement compris.

---

## 6.20. Exercices

## Exercice 1 — Observer le namespace mount courant

Nous exécutons :

```bash
readlink /proc/$$/ns/mnt
readlink /proc/1/ns/mnt
findmnt | head
cat /proc/self/mountinfo | head
```

Nous répondons :

1. Le shell partage-t-il le namespace mount du PID 1 ?
    
2. Quels sont les premiers montages visibles ?
    
3. Quelle différence voyons-nous entre `findmnt` et `mountinfo` ?
    

---

## Exercice 2 — Créer un montage isolé

Nous exécutons :

```bash
unshare --mount bash
mount --make-rprivate /
mkdir -p /tmp/ns-mount-demo
mount -t tmpfs tmpfs /tmp/ns-mount-demo
echo "test namespace mount" > /tmp/ns-mount-demo/test.txt
findmnt /tmp/ns-mount-demo
```

Depuis l’hôte :

```bash
findmnt /tmp/ns-mount-demo
ls /tmp/ns-mount-demo
```

Nous répondons :

1. Le montage est-il visible depuis l’hôte ?
    
2. Le fichier est-il visible depuis l’hôte ?
    
3. Pourquoi avons-nous utilisé `mount --make-rprivate /` ?
    

---

## Exercice 3 — Observer la propagation

Nous exécutons :

```bash
findmnt -o TARGET,PROPAGATION /
cat /proc/self/mountinfo | grep -E 'shared|master' | head
```

Nous répondons :

1. La racine est-elle `shared`, `private` ou autre ?
    
2. Voyons-nous des groupes `shared` ?
    
3. Pourquoi cela peut-il influencer les namespaces mount ?
    

---

## Exercice 4 — Tester un bind mount

Nous exécutons :

```bash
mkdir -p /tmp/source-bind /tmp/dest-bind
echo "contenu bind" > /tmp/source-bind/fichier.txt

unshare --mount bash
mount --make-rprivate /
mount --bind /tmp/source-bind /tmp/dest-bind
cat /tmp/dest-bind/fichier.txt
findmnt /tmp/dest-bind
umount /tmp/dest-bind
exit
```

Nous répondons :

1. Que fait `mount --bind` ?
    
2. Le contenu est-il copié ?
    
3. Pourquoi les bind mounts sont-ils importants pour Docker ?
    

---

## Exercice 5 — Masquage par montage

Nous exécutons :

```bash
mkdir -p /tmp/mask-demo
echo "avant montage" > /tmp/mask-demo/fichier.txt
ls /tmp/mask-demo

unshare --mount bash
mount --make-rprivate /
mount -t tmpfs tmpfs /tmp/mask-demo
ls /tmp/mask-demo
echo "dans tmpfs" > /tmp/mask-demo/autre.txt
ls /tmp/mask-demo
umount /tmp/mask-demo
ls /tmp/mask-demo
exit
```

Nous répondons :

1. Le fichier initial a-t-il été supprimé ?
    
2. Pourquoi disparaît-il pendant le montage ?
    
3. Pourquoi réapparaît-il après `umount` ?
    

---

## Exercice 6 — Comparer `chroot` et namespace mount

Nous réfléchissons à la commande :

```bash
chroot /tmp/rootfs /bin/sh
```

Nous répondons :

1. Que change `chroot` ?
    
2. Que ne change-t-il pas ?
    
3. Pourquoi `chroot` seul ne suffit-il pas pour créer un conteneur ?
    
4. Quel rôle joue le namespace mount en plus ?
    

---

## Exercice 7 — Observer un conteneur Docker

Si Docker est disponible :

```bash
docker run --rm -d --name mnt-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' mnt-demo)

sudo readlink /proc/$pid/ns/mnt
readlink /proc/1/ns/mnt

sudo nsenter --target "$pid" --mount findmnt | head
sudo cat /proc/$pid/mountinfo | head

docker rm -f mnt-demo
```

Nous répondons :

1. Le conteneur partage-t-il le namespace mount de l’hôte ?
    
2. Quels montages voyons-nous dans le conteneur ?
    
3. Que représente le rootfs du conteneur ?
    
4. Pourquoi Docker utilise-t-il un namespace mount ?
    

---

## 6.21. Ce que nous devons retenir

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

## Conclusion du chapitre 6

Nous avons étudié le namespace mount, qui est l’un des mécanismes fondamentaux des conteneurs Linux.

Nous savons maintenant qu’il permet d’isoler la vue des montages, de présenter un root filesystem spécifique à un processus, de monter `/proc`, `/sys`, `/dev`, des volumes ou des bind mounts, et de construire une arborescence différente de celle de l’hôte.

Nous avons aussi vu que ce namespace demande une grande prudence. La propagation des montages peut surprendre, les bind mounts peuvent exposer l’hôte, et des montages sensibles comme `/proc`, `/sys`, `/dev` ou le socket Docker peuvent réduire fortement l’isolation.

Nous retenons surtout que le namespace mount ne se limite pas à “changer des fichiers visibles”. Il définit une partie essentielle de la frontière entre l’hôte et l’environnement isolé.

Dans le chapitre suivant, nous étudions le namespace network, qui permet à un processus d’avoir sa propre pile réseau, ses propres interfaces, ses routes, ses ports et ses sockets.

---
> [!info] Livre « Les namespaces Linux » — chapitre 6/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 05 — Namespace PID|← 05 — Namespace PID]] · [[Les namespaces Linux — 07 — Namespace network|07 — Namespace network →]]
