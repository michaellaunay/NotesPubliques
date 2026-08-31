---
schema_version: 1
uid: 01M1BQ624PRQPBQQ75CDFTXPK4
titre: "Les namespaces Linux — 15 — Des namespaces aux conteneurs"
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
resume: "Chapitre 15 sur 16 du livre « Les namespaces Linux » : Des namespaces aux conteneurs. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 15/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 14 — Namespaces, capabilities et sécurité|← 14 — Namespaces, capabilities et sécurité]] · [[Les namespaces Linux — 16 — Namespaces dans Kubernetes|16 — Namespaces dans Kubernetes →]]

# Chapitre 15 — Des namespaces aux conteneurs
## Objectifs du chapitre

Dans ce chapitre, nous relions les namespaces Linux au fonctionnement concret des conteneurs.

Jusqu’ici, nous avons étudié les briques séparément :

```text
namespace PID
namespace mount
namespace network
namespace UTS
namespace IPC
namespace user
namespace cgroup
namespace time
cgroups
capabilities
seccomp
AppArmor / SELinux
```

Nous allons maintenant comprendre comment un runtime de conteneur assemble ces mécanismes pour lancer une application dans un environnement isolé.

À la fin de ce chapitre, nous savons :

- expliquer ce qu’est réellement un conteneur Linux ;
    
- comprendre comment Docker combine les namespaces ;
    
- distinguer image, root filesystem et conteneur ;
    
- comprendre le rôle du runtime OCI ;
    
- expliquer le rôle de `runc`, `containerd` et Docker ;
    
- comprendre l’importance de l’entrypoint et du PID 1 ;
    
- comprendre les volumes et bind mounts ;
    
- comprendre le réseau bridge Docker ;
    
- observer un conteneur depuis l’hôte avec `/proc`, `lsns` et `nsenter`.
    

---

## 15.1. Un conteneur n’est pas une machine virtuelle

## 15.1.1. Rappel fondamental

Un conteneur Linux n’exécute pas son propre noyau.

Il utilise le noyau de l’hôte, mais avec des vues isolées.

Nous devons retenir :

```text
Machine virtuelle : noyau invité séparé.
Conteneur Linux   : noyau hôte partagé.
```

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

Dans un conteneur, nous avons plutôt :

```text
application
rootfs isolé
namespaces
cgroups
noyau Linux hôte
matériel réel
```

---

## 15.1.2. Pourquoi les conteneurs sont plus légers

Les conteneurs sont plus légers parce qu’ils ne démarrent pas un système d’exploitation complet avec son propre noyau.

Ils lancent simplement un ou plusieurs processus sur le noyau de l’hôte.

Ce qui donne l’impression d’un système séparé vient de la combinaison :

```text
namespaces
cgroups
root filesystem
capabilities
seccomp
profils de sécurité
configuration réseau
```

Nous ne devons donc pas voir Docker comme une technologie magique, mais comme un orchestrateur de mécanismes Linux.

---

## 15.2. Qu’est-ce qu’un conteneur Linux ?

## 15.2.1. Définition pratique

Un conteneur Linux est un ou plusieurs processus lancés avec :

- des namespaces isolés ;
    
- des limites cgroups ;
    
- un root filesystem spécifique ;
    
- des capabilities contrôlées ;
    
- des montages préparés ;
    
- une configuration réseau ;
    
- une politique de sécurité ;
    
- une commande de démarrage.
    

Autrement dit :

```text
Un conteneur est un processus isolé et configuré.
```

Il ne faut pas le confondre avec l’image.

---

## 15.2.2. Image et conteneur

Une image est un modèle immuable ou quasi immuable contenant :

- fichiers ;
    
- bibliothèques ;
    
- binaires ;
    
- métadonnées ;
    
- configuration par défaut ;
    
- entrypoint ;
    
- commande par défaut.
    

Un conteneur est une instance en cours d’exécution créée à partir d’une image.

Nous pouvons résumer :

```text
Image      = modèle de fichiers et de configuration.
Conteneur  = processus lancé à partir de cette image.
```

Exemple :

```bash
docker pull alpine
docker run --rm -it alpine sh
```

`alpine` est l’image.

Le shell lancé est un processus dans un conteneur créé à partir de cette image.

---

## 15.3. Ce que fait Docker au lancement d’un conteneur

## 15.3.1. Commande simple

Quand nous lançons :

```bash
docker run --rm -it alpine sh
```

Docker effectue beaucoup d’opérations.

Il ne fait pas seulement :

```text
lancer /bin/sh
```

Il prépare un environnement d’exécution complet.

---

## 15.3.2. Étapes conceptuelles

Docker ou son runtime prépare notamment :

1. le root filesystem issu de l’image ;
    
2. un namespace mount ;
    
3. un namespace PID ;
    
4. un namespace network ;
    
5. un namespace UTS ;
    
6. un namespace IPC ;
    
7. parfois un namespace user ;
    
8. une configuration cgroup ;
    
9. des capabilities ;
    
10. un profil seccomp ;
    
11. éventuellement un profil AppArmor ou SELinux ;
    
12. les fichiers `/etc/hosts`, `/etc/resolv.conf`, `/etc/hostname` ;
    
13. les volumes et bind mounts ;
    
14. l’interface réseau virtuelle ;
    
15. le processus initial.
    

Nous voyons que Docker assemble les chapitres précédents.

---

## 15.4. Les namespaces utilisés par un conteneur

## 15.4.1. Namespace PID

Le conteneur a souvent son propre namespace PID.

Dans le conteneur :

```sh
ps
```

nous pouvons voir :

```text
PID   USER     TIME  COMMAND
1     root     0:00  sh
```

Mais sur l’hôte, le même processus a un autre PID.

Nous pouvons vérifier :

```bash
docker run --rm -d --name pid-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' pid-demo)

ps -p "$pid" -o pid,ppid,comm,args
sudo grep NSpid /proc/$pid/status

docker exec pid-demo ps
docker rm -f pid-demo
```

Le namespace PID permet au processus principal d’être PID 1 dans le conteneur.

---

## 15.4.2. Namespace mount

Le conteneur a son propre namespace mount.

Il voit comme `/` le root filesystem de l’image.

Dans le conteneur :

```sh
ls /
```

nous voyons la racine de l’image, par exemple :

```text
bin
dev
etc
home
lib
proc
root
sys
tmp
usr
var
```

Ce n’est pas la racine de l’hôte.

Docker prépare cette vue avec :

- rootfs de l’image ;
    
- OverlayFS ou autre backend ;
    
- montage de `/proc` ;
    
- montage de `/sys` ;
    
- préparation de `/dev` ;
    
- volumes ;
    
- bind mounts.
    

---

## 15.4.3. Namespace network

Le conteneur a généralement son propre namespace réseau.

Dans le conteneur :

```sh
ip addr
```

nous voyons souvent :

```text
lo
eth0
```

`eth0` est souvent une extrémité d’une paire `veth`.

L’autre extrémité est côté hôte, connectée au bridge Docker, souvent `docker0`.

---

## 15.4.4. Namespace UTS

Docker donne souvent un hostname propre au conteneur.

Exemple :

```bash
docker run --rm alpine hostname
```

Sortie possible :

```text
a1b2c3d4e5f6
```

Nous pouvons définir le hostname :

```bash
docker run --rm --hostname mon-conteneur alpine hostname
```

Le namespace UTS permet cette isolation du hostname.

---

## 15.4.5. Namespace IPC

Docker crée généralement un namespace IPC propre au conteneur.

Nous pouvons comparer :

```bash
docker run --rm -d --name ipc-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' ipc-demo)

sudo readlink /proc/$pid/ns/ipc
readlink /proc/1/ns/ipc

docker rm -f ipc-demo
```

Si les valeurs diffèrent, le conteneur a son propre namespace IPC.

---

## 15.4.6. Namespace user

Le namespace user dépend de la configuration.

Dans un conteneur Docker classique, le conteneur peut être root à l’intérieur, mais pas nécessairement avec un mapping user namespace complet.

Avec les conteneurs rootless, ou certaines options de remapping, l’UID `0` interne peut correspondre à un UID non privilégié sur l’hôte.

Nous observons avec :

```bash
cat /proc/<PID>/uid_map
cat /proc/<PID>/gid_map
```

---

## 15.4.7. Namespace cgroup

Le namespace cgroup peut permettre au conteneur de voir une vue simplifiée de son rattachement cgroup.

Dans le conteneur :

```sh
cat /proc/self/cgroup
```

nous pouvons voir une vue plus simple que depuis l’hôte.

Mais les limites réelles sont appliquées par les cgroups eux-mêmes.

---

## 15.5. Le root filesystem du conteneur

## 15.5.1. Définition

Le root filesystem, ou rootfs, est l’arborescence vue comme `/` par le processus du conteneur.

Il contient les fichiers nécessaires à l’exécution de l’application :

```text
/bin
/etc
/lib
/usr
/app
/var
/tmp
```

Dans Docker, ce rootfs provient de l’image.

---

## 15.5.2. Images en couches

Les images Docker sont composées de couches.

Exemple conceptuel :

```text
couche 1 : base Alpine
couche 2 : installation de paquets
couche 3 : copie de l’application
couche 4 : configuration
```

Ces couches sont généralement en lecture seule.

Quand nous lançons un conteneur, Docker ajoute une couche writable propre au conteneur.

Nous obtenons :

```text
couches image en lecture seule
+
couche writable du conteneur
=
rootfs visible dans le conteneur
```

---

## 15.5.3. OverlayFS

Sur beaucoup de systèmes, Docker utilise OverlayFS.

Nous pouvons conceptualiser :

```text
lowerdir : couches en lecture seule
upperdir : couche writable du conteneur
merged   : vue finale présentée au conteneur
```

Le namespace mount présente ensuite `merged` comme racine `/` du conteneur.

---

## 15.5.4. Conséquence pratique

Si nous créons un fichier dans un conteneur :

```sh
echo test > /tmp/fichier
```

ce fichier est écrit dans la couche writable du conteneur.

Si nous supprimons le conteneur, cette couche disparaît, sauf si le fichier est dans un volume.

Nous retenons :

```text
Le conteneur est éphémère.
Les volumes servent à conserver des données.
```

---

## 15.6. Volumes et bind mounts

## 15.6.1. Pourquoi les volumes existent

Un conteneur est souvent jetable.

Mais certaines données doivent persister :

- base de données ;
    
- fichiers uploadés ;
    
- cache durable ;
    
- configuration ;
    
- logs selon architecture ;
    
- artefacts produits.
    

Nous utilisons alors des volumes ou bind mounts.

---

## 15.6.2. Bind mount

Un bind mount monte un chemin de l’hôte dans le conteneur.

Exemple :

```bash
docker run --rm -it -v "$PWD":/app alpine sh
```

Dans le conteneur :

```sh
ls /app
```

Nous voyons le répertoire courant de l’hôte.

Le conteneur et l’hôte accèdent au même contenu.

---

## 15.6.3. Volume Docker

Un volume Docker est géré par Docker.

Exemple :

```bash
docker volume create data-demo
docker run --rm -it -v data-demo:/data alpine sh
```

Les données sont stockées dans l’espace géré par Docker.

Nous pouvons lister les volumes :

```bash
docker volume ls
```

Et inspecter :

```bash
docker volume inspect data-demo
```

---

## 15.6.4. Lecture seule

Nous pouvons monter en lecture seule :

```bash
docker run --rm -it -v "$PWD":/app:ro alpine sh
```

Dans le conteneur, l’écriture dans `/app` doit échouer.

C’est une bonne pratique lorsque le conteneur n’a besoin que de lire des fichiers.

---

## 15.6.5. Risques

Les montages peuvent casser l’isolation.

Montages dangereux :

```bash
-v /:/host
-v /proc:/host/proc
-v /sys:/host/sys
-v /dev:/host/dev
-v /var/run/docker.sock:/var/run/docker.sock
```

Nous ne les utilisons pas sans justification forte.

---

## 15.7. Le réseau Docker bridge

## 15.7.1. Bridge par défaut

Docker crée souvent un bridge nommé :

```text
docker0
```

Nous pouvons l’observer :

```bash
ip addr show docker0
```

Les conteneurs connectés au réseau bridge par défaut reçoivent une interface `eth0` dans leur namespace réseau.

Côté hôte, une paire `veth` connecte le conteneur au bridge.

---

## 15.7.2. Schéma conceptuel

```text
conteneur
  eth0
   |
 paire veth
   |
hôte
  vethXXXX
   |
 docker0
   |
 réseau extérieur via routage/NAT
```

Le namespace réseau isole la pile réseau du conteneur.

Le bridge permet la communication entre conteneurs et avec l’hôte.

Le NAT permet souvent la sortie vers Internet.

---

## 15.7.3. Ports publiés

Quand nous écrivons :

```bash
docker run -p 8080:80 image
```

nous demandons :

```text
port 8080 de l’hôte → port 80 du conteneur
```

Le conteneur peut écouter sur `80` dans son namespace réseau.

L’hôte expose `8080` et redirige vers le conteneur.

Nous distinguons donc :

```text
port interne du conteneur
port exposé sur l’hôte
```

---

## 15.7.4. Mode host network

Avec :

```bash
docker run --network=host image
```

le conteneur partage le namespace réseau de l’hôte.

Il n’a plus d’isolation réseau classique.

Conséquences :

- mêmes interfaces que l’hôte ;
    
- mêmes ports ;
    
- conflits possibles ;
    
- exposition plus directe ;
    
- diagnostic différent.
    

Nous utilisons ce mode avec prudence.

---

## 15.8. Entrypoint, command et PID 1

## 15.8.1. Le processus initial

Un conteneur démarre avec un processus initial.

Ce processus devient généralement PID 1 dans le namespace PID du conteneur.

Il peut venir de :

```text
ENTRYPOINT
CMD
commande passée à docker run
```

Exemple :

```bash
docker run --rm alpine sleep 1000
```

Ici, `sleep 1000` devient le processus principal du conteneur.

---

## 15.8.2. `ENTRYPOINT` et `CMD`

Dans un Dockerfile :

```dockerfile
ENTRYPOINT ["python3"]
CMD ["app.py"]
```

La commande finale est :

```text
python3 app.py
```

Autre exemple :

```dockerfile
ENTRYPOINT ["./server"]
CMD ["--port", "8080"]
```

`ENTRYPOINT` définit souvent le binaire principal.

`CMD` définit les arguments par défaut.

---

## 15.8.3. PID 1 et signaux

Le processus PID 1 doit gérer correctement les signaux.

Quand nous faisons :

```bash
docker stop <container>
```

Docker envoie d’abord généralement `SIGTERM`.

Si le processus ne s’arrête pas, Docker finit par envoyer `SIGKILL`.

Une application mal conçue comme PID 1 peut ignorer ou mal gérer `SIGTERM`.

---

## 15.8.4. Processus zombies

Le PID 1 doit aussi récolter les processus enfants terminés.

Si l’application lance des sous-processus et ne les récolte pas, des zombies peuvent s’accumuler.

Nous pouvons utiliser un init minimal :

```bash
docker run --init image
```

Docker ajoute alors un petit init, souvent `tini`, qui aide à gérer les signaux et les zombies.

---

## 15.9. Rôle de `runc`, containerd et Docker

## 15.9.1. Docker n’est pas seul

Quand nous utilisons Docker, plusieurs composants interviennent.

Schéma simplifié :

```text
docker CLI
   |
dockerd
   |
containerd
   |
runc
   |
noyau Linux
```

Chaque composant a un rôle.

---

## 15.9.2. Docker CLI

La commande :

```bash
docker run ...
```

est envoyée au démon Docker.

Le client Docker n’est qu’une interface de commande.

---

## 15.9.3. `dockerd`

Le démon Docker gère :

- images ;
    
- conteneurs ;
    
- réseaux ;
    
- volumes ;
    
- API Docker ;
    
- intégration avec containerd ;
    
- configuration globale Docker.
    

---

## 15.9.4. `containerd`

`containerd` est un runtime de plus bas niveau qui gère le cycle de vie des conteneurs :

- création ;
    
- démarrage ;
    
- arrêt ;
    
- supervision ;
    
- images ;
    
- snapshots ;
    
- interaction avec le runtime OCI.
    

Kubernetes utilise souvent `containerd` directement comme runtime CRI.

---

## 15.9.5. `runc`

`runc` est un runtime OCI bas niveau.

Il est responsable de créer concrètement le conteneur au niveau Linux :

- namespaces ;
    
- cgroups ;
    
- capabilities ;
    
- montages ;
    
- rootfs ;
    
- processus initial.
    

`runc` applique une configuration conforme à la spécification OCI.

---

## 15.10. OCI : Open Container Initiative

## 15.10.1. Pourquoi une spécification ?

Avant la standardisation, chaque outil pouvait définir ses propres formats et comportements.

L’OCI, Open Container Initiative, définit notamment :

- une spécification d’image ;
    
- une spécification de runtime ;
    
- un format de configuration.
    

Cela permet à différents outils d’être compatibles.

---

## 15.10.2. Runtime OCI

Un runtime OCI reçoit une configuration et lance le conteneur selon cette configuration.

Cette configuration décrit notamment :

- commande à exécuter ;
    
- environnement ;
    
- rootfs ;
    
- montages ;
    
- namespaces ;
    
- capabilities ;
    
- cgroups ;
    
- utilisateur ;
    
- limites ;
    
- hooks éventuels.
    

`runc` est l’implémentation de référence historique.

---

## 15.11. Observer un conteneur depuis l’hôte

## 15.11.1. Lancer un conteneur de test

Nous lançons :

```bash
docker run --rm -d --name ns-demo alpine sleep 1000
```

Nous récupérons le PID hôte :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' ns-demo)
echo "$pid"
```

---

## 15.11.2. Observer ses namespaces

Depuis l’hôte :

```bash
sudo ls -l /proc/$pid/ns
```

Nous comparons avec l’hôte :

```bash
ls -l /proc/1/ns
```

Nous pouvons voir quels namespaces sont différents.

---

## 15.11.3. Observer les PID

```bash
sudo grep NSpid /proc/$pid/status
```

Exemple :

```text
NSpid:  24531  1
```

Le même processus est PID `24531` sur l’hôte et PID `1` dans le conteneur.

---

## 15.11.4. Observer les cgroups

```bash
sudo cat /proc/$pid/cgroup
```

Et dans le conteneur :

```bash
docker exec ns-demo cat /proc/self/cgroup
```

Nous comparons la vue depuis l’hôte et la vue interne.

---

## 15.11.5. Observer les montages

```bash
sudo cat /proc/$pid/mountinfo | head
```

Ou :

```bash
sudo nsenter --target "$pid" --mount findmnt | head
```

Nous voyons les montages du conteneur.

---

## 15.11.6. Observer le réseau

```bash
sudo nsenter --target "$pid" --net ip addr
sudo nsenter --target "$pid" --net ip route
```

Si `ip` n’est pas disponible dans le conteneur, nous pouvons l’utiliser depuis l’hôte avec `nsenter`.

---

## 15.11.7. Nettoyage

```bash
docker rm -f ns-demo
```

---

## 15.12. `docker exec` et `nsenter`

## 15.12.1. `docker exec`

Quand nous utilisons :

```bash
docker exec -it ns-demo sh
```

Docker lance un nouveau processus dans le conteneur existant.

Ce nouveau processus rejoint les namespaces du conteneur.

Il partage donc généralement :

- namespace PID ;
    
- namespace mount ;
    
- namespace network ;
    
- namespace UTS ;
    
- namespace IPC ;
    
- cgroups du conteneur.
    

---

## 15.12.2. `nsenter`

Avec `nsenter`, nous pouvons entrer dans les namespaces du processus principal depuis l’hôte :

```bash
sudo nsenter --target "$pid" \
  --mount \
  --uts \
  --ipc \
  --net \
  --pid \
  --fork \
  sh
```

Cela est utile lorsque :

- l’image ne contient pas de shell ;
    
- `docker exec` ne fonctionne pas ;
    
- nous voulons utiliser des outils de l’hôte ;
    
- nous voulons diagnostiquer un namespace précis.
    

---

## 15.12.3. Différence pratique

`docker exec` passe par Docker.

`nsenter` passe directement par les namespaces exposés dans `/proc/<PID>/ns`.

Les deux approches sont utiles, mais elles n’ont pas exactement le même niveau d’abstraction.

---

## 15.13. Variables d’environnement

## 15.13.1. Définition

Un conteneur reçoit souvent des variables d’environnement.

Exemple :

```bash
docker run --rm -e APP_ENV=production alpine env
```

Elles peuvent configurer :

- environnement ;
    
- URL de base de données ;
    
- chemins ;
    
- options applicatives ;
    
- mode debug ;
    
- secrets parfois, avec prudence.
    

---

## 15.13.2. Observation

Dans un conteneur :

```sh
env
```

Depuis l’hôte, selon les permissions :

```bash
sudo tr '\0' '\n' < /proc/$pid/environ
```

Nous devons nous souvenir que les variables d’environnement peuvent être visibles via `/proc`.

---

## 15.13.3. Prudence avec les secrets

Passer des secrets en variables d’environnement est courant, mais pas parfait.

Risques :

- visibilité via `/proc` ;
    
- fuite dans des dumps ;
    
- affichage dans logs ;
    
- exposition par erreur dans une page debug ;
    
- transmission à des processus enfants.
    

En production, nous préférons des mécanismes de secrets adaptés et des permissions strictes.

---

## 15.14. Capabilities et sécurité dans un conteneur

## 15.14.1. Root dans le conteneur

Dans un conteneur :

```sh
id
```

peut afficher :

```text
uid=0(root) gid=0(root)
```

Mais cela ne signifie pas que le processus a tous les droits sur l’hôte.

Nous devons vérifier :

```sh
grep Cap /proc/$$/status
```

---

## 15.14.2. Réduire les capabilities

Bonne pratique :

```bash
docker run --cap-drop=ALL image
```

Puis ajouter seulement ce qui est nécessaire :

```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE image
```

Nous appliquons le principe du moindre privilège.

---

## 15.14.3. Éviter `--privileged`

`--privileged` donne beaucoup trop de droits.

```bash
docker run --privileged image
```

Nous l’évitons sauf cas très spécifique.

Nous préférons identifier le besoin précis :

```text
capability manquante ?
périphérique précis ?
montage précis ?
profil seccomp trop strict ?
```

---

## 15.15. Seccomp, AppArmor et SELinux dans les conteneurs

## 15.15.1. Seccomp

Docker applique généralement un profil seccomp par défaut.

Nous évitons :

```bash
docker run --security-opt seccomp=unconfined image
```

sauf diagnostic ou besoin justifié.

---

## 15.15.2. AppArmor

Sur Ubuntu/Debian, Docker peut utiliser un profil AppArmor comme :

```text
docker-default
```

Nous pouvons observer :

```bash
cat /proc/<PID>/attr/current
```

---

## 15.15.3. SELinux

Sur Fedora, RHEL, Rocky, AlmaLinux ou CentOS Stream, SELinux peut confiner les conteneurs.

Un problème de permission peut venir de SELinux même si les permissions Unix semblent correctes.

Nous vérifions :

```bash
getenforce
ls -Z
ps -eZ
```

---

## 15.16. Construire mentalement un conteneur

## 15.16.1. Les briques

Pour construire un conteneur, nous devons assembler :

```text
rootfs
namespace mount
namespace PID
namespace UTS
namespace IPC
namespace network
namespace user éventuellement
cgroups
capabilities
seccomp
AppArmor/SELinux
processus initial
```

Ce n’est pas une seule technologie.

C’est une composition.

---

## 15.16.2. Mini-conteneur conceptuel

À la main, nous pourrions faire :

```bash
unshare --fork --pid --mount --uts --ipc bash
```

Puis :

```bash
mount --make-rprivate /
mount -t proc proc /proc
hostname mini-container
```

Mais il manquerait encore :

- un rootfs propre ;
    
- un pivot_root ou chroot propre ;
    
- un réseau isolé configuré ;
    
- des cgroups ;
    
- des capabilities réduites ;
    
- une politique seccomp ;
    
- une gestion correcte du PID 1 ;
    
- un nettoyage fiable.
    

Docker automatise tout cela.

---

## 15.17. Exemple d’analyse complète d’un conteneur

## 15.17.1. Lancement

Nous lançons :

```bash
docker run --rm -d \
  --name analyse-demo \
  --hostname demo \
  --memory=256m \
  --pids-limit=128 \
  alpine sleep 1000
```

Nous récupérons le PID :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' analyse-demo)
```

---

## 15.17.2. Namespaces

```bash
sudo ls -l /proc/$pid/ns
```

Nous comparons avec :

```bash
ls -l /proc/1/ns
```

Nous identifions les namespaces différents.

---

## 15.17.3. PID

```bash
sudo grep NSpid /proc/$pid/status
docker exec analyse-demo ps
```

Nous observons le PID hôte et le PID interne.

---

## 15.17.4. Hostname

```bash
docker exec analyse-demo hostname
sudo nsenter --target "$pid" --uts hostname
```

Nous vérifions le namespace UTS.

---

## 15.17.5. Réseau

```bash
sudo nsenter --target "$pid" --net ip addr
sudo nsenter --target "$pid" --net ip route
```

Nous observons la pile réseau du conteneur.

---

## 15.17.6. Montages

```bash
sudo nsenter --target "$pid" --mount findmnt | head
sudo cat /proc/$pid/mountinfo | head
```

Nous observons le rootfs et les montages spéciaux.

---

## 15.17.7. Cgroups

```bash
docker exec analyse-demo sh -c 'cat /proc/self/cgroup'
docker exec analyse-demo sh -c 'cat /sys/fs/cgroup/memory.max 2>/dev/null || true'
docker exec analyse-demo sh -c 'cat /sys/fs/cgroup/pids.max 2>/dev/null || true'
```

Nous vérifions les limites.

---

## 15.17.8. Capabilities

```bash
docker exec analyse-demo sh -c 'grep Cap /proc/$$/status'
```

Nous observons les capabilities.

---

## 15.17.9. Nettoyage

```bash
docker rm -f analyse-demo
```

---

## 15.18. Pièges classiques

## 15.18.1. Confondre image et conteneur

Une image n’est pas un processus.

Un conteneur est une instance lancée à partir d’une image.

Nous pouvons avoir plusieurs conteneurs basés sur la même image.

---

## 15.18.2. Croire que le conteneur a son propre noyau

Dans un conteneur :

```sh
uname -a
```

affiche le noyau de l’hôte.

Le conteneur n’a pas son propre noyau.

---

## 15.18.3. Oublier le rôle du PID 1

Si l’application est PID 1, elle doit gérer :

- signaux ;
    
- arrêt propre ;
    
- processus enfants ;
    
- zombies.
    

Sinon, nous utilisons un init minimal :

```bash
docker run --init ...
```

---

## 15.18.4. Confondre `EXPOSE` et publication de port

Dans un Dockerfile :

```dockerfile
EXPOSE 8080
```

documente le port prévu.

Mais cela ne publie pas automatiquement le port sur l’hôte.

Pour publier :

```bash
docker run -p 8080:8080 image
```

---

## 15.18.5. Croire qu’un volume est une copie

Un bind mount n’est pas une copie.

Si nous montons :

```bash
-v "$PWD":/app
```

le conteneur et l’hôte voient le même contenu.

Une modification dans le conteneur peut modifier l’hôte.

---

## 15.18.6. Utiliser `--privileged` pour résoudre trop vite

Si quelque chose ne marche pas, nous ne lançons pas directement :

```bash
docker run --privileged ...
```

Nous identifions le mécanisme en cause :

- permissions fichiers ;
    
- UID/GID ;
    
- capability manquante ;
    
- seccomp ;
    
- AppArmor/SELinux ;
    
- montage absent ;
    
- périphérique nécessaire ;
    
- cgroup trop restrictif.
    

---

## 15.19. Exercices

## Exercice 1 — Observer les namespaces d’un conteneur

Nous lançons :

```bash
docker run --rm -d --name ns-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' ns-demo)

sudo ls -l /proc/$pid/ns
ls -l /proc/1/ns

docker rm -f ns-demo
```

Nous répondons :

1. Quels namespaces sont différents de ceux de l’hôte ?
    
2. Quels namespaces sont éventuellement partagés ?
    
3. Pourquoi un conteneur a-t-il besoin de plusieurs namespaces ?
    

---

## Exercice 2 — Observer le PID interne et externe

Nous lançons :

```bash
docker run --rm -d --name pid-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' pid-demo)

ps -p "$pid" -o pid,ppid,comm,args
sudo grep NSpid /proc/$pid/status
docker exec pid-demo ps

docker rm -f pid-demo
```

Nous répondons :

1. Quel est le PID du processus sur l’hôte ?
    
2. Quel est son PID dans le conteneur ?
    
3. Pourquoi les deux valeurs diffèrent-elles ?
    

---

## Exercice 3 — Observer le réseau du conteneur

Nous lançons :

```bash
docker run --rm -d --name net-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' net-demo)

sudo nsenter --target "$pid" --net ip addr
sudo nsenter --target "$pid" --net ip route

docker rm -f net-demo
```

Nous répondons :

1. Quelles interfaces le conteneur voit-il ?
    
2. Quelle est sa route par défaut ?
    
3. Quel rôle joue le bridge Docker ?
    

---

## Exercice 4 — Observer les montages du conteneur

Nous lançons :

```bash
docker run --rm -d --name mnt-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' mnt-demo)

sudo nsenter --target "$pid" --mount findmnt | head
sudo cat /proc/$pid/mountinfo | head

docker rm -f mnt-demo
```

Nous répondons :

1. Que représente `/` dans le conteneur ?
    
2. Voyons-nous `/proc`, `/sys` et `/dev` ?
    
3. Pourquoi le namespace mount est-il fondamental ?
    

---

## Exercice 5 — Comparer volume et couche writable

Nous lançons :

```bash
mkdir -p /tmp/docker-volume-test
echo "depuis hote" > /tmp/docker-volume-test/fichier.txt

docker run --rm -it -v /tmp/docker-volume-test:/data alpine sh
```

Dans le conteneur :

```sh
cat /data/fichier.txt
echo "depuis conteneur" > /data/autre.txt
exit
```

Sur l’hôte :

```bash
ls -l /tmp/docker-volume-test
cat /tmp/docker-volume-test/autre.txt
```

Nous répondons :

1. Le fichier créé dans le conteneur est-il visible sur l’hôte ?
    
2. Pourquoi ?
    
3. Quelle différence avec un fichier créé hors volume dans le conteneur ?
    

---

## Exercice 6 — Observer les limites cgroups

Nous lançons :

```bash
docker run --rm -d \
  --name limits-demo \
  --memory=256m \
  --pids-limit=64 \
  alpine sleep 1000

docker exec limits-demo sh -c 'cat /sys/fs/cgroup/memory.max 2>/dev/null || true'
docker exec limits-demo sh -c 'cat /sys/fs/cgroup/pids.max 2>/dev/null || true'

docker rm -f limits-demo
```

Nous répondons :

1. Quelle limite mémoire est visible ?
    
2. Quelle limite de processus est visible ?
    
3. Quel mécanisme applique ces limites ?
    

---

## Exercice 7 — Analyser les capabilities

Nous comparons :

```bash
docker run --rm alpine sh -c 'grep Cap /proc/$$/status'
docker run --rm --cap-drop=ALL alpine sh -c 'grep Cap /proc/$$/status'
```

Nous répondons :

1. Les capabilities changent-elles ?
    
2. Pourquoi est-ce important ?
    
3. Pourquoi être root dans le conteneur ne suffit-il pas à comprendre les droits ?
    

---

## 15.20. Ce que nous devons retenir

Nous retenons les points suivants :

1. Un conteneur Linux est un processus isolé, pas une machine virtuelle.
    
2. Le conteneur partage le noyau de l’hôte.
    
3. Docker assemble des namespaces, cgroups, capabilities, montages et politiques de sécurité.
    
4. Une image est un modèle, un conteneur est une instance en cours d’exécution.
    
5. Le rootfs d’un conteneur vient des couches de l’image et d’une couche writable.
    
6. Le namespace mount présente ce rootfs comme `/`.
    
7. Le namespace PID permet au processus principal d’être PID 1 dans le conteneur.
    
8. Le PID 1 doit gérer les signaux et les processus enfants.
    
9. Le namespace network donne au conteneur sa propre pile réseau.
    
10. Le réseau bridge Docker repose sur des paires `veth`, un bridge et souvent du NAT.
    
11. Les volumes et bind mounts ajoutent des montages dans le conteneur.
    
12. Les bind mounts peuvent exposer l’hôte.
    
13. Les cgroups appliquent les limites CPU, mémoire et pids.
    
14. Les capabilities contrôlent les privilèges fins.
    
15. Seccomp, AppArmor et SELinux ajoutent des couches de sécurité.
    
16. `runc` crée concrètement les namespaces, montages, cgroups et processus.
    
17. `containerd` gère le cycle de vie des conteneurs à plus bas niveau.
    
18. Docker fournit une interface plus haut niveau.
    
19. `docker exec` lance un processus dans les namespaces du conteneur.
    
20. `nsenter` permet d’entrer directement dans les namespaces depuis l’hôte.
    
21. Pour diagnostiquer un conteneur, nous croisons `/proc`, `nsenter`, cgroups, montages, réseau et capabilities.
    

---

## Conclusion du chapitre 15

Nous avons relié les namespaces au fonctionnement concret des conteneurs.

Nous savons maintenant qu’un conteneur n’est pas une machine virtuelle, mais un processus lancé dans un environnement soigneusement construit. Docker, Podman, containerd ou `runc` assemblent des namespaces, des cgroups, des montages, des capabilities et des politiques de sécurité pour produire cette isolation.

Nous avons aussi compris la différence entre image, root filesystem et conteneur. L’image fournit les fichiers, le namespace mount présente ces fichiers comme racine, le namespace PID donne une vue de processus isolée, le namespace network fournit une pile réseau séparée, et les cgroups limitent les ressources.

Nous retenons surtout qu’un conteneur est une composition de mécanismes Linux. Pour diagnostiquer ou sécuriser un conteneur, nous devons donc raisonner couche par couche : processus, montages, réseau, utilisateurs, cgroups, capabilities et sécurité.

Dans le chapitre suivant, nous étudions les namespaces dans Kubernetes, où ces mêmes mécanismes sont utilisés à l’échelle de pods, de nœuds, de workloads et de clusters.

---
> [!info] Livre « Les namespaces Linux » — chapitre 15/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 14 — Namespaces, capabilities et sécurité|← 14 — Namespaces, capabilities et sécurité]] · [[Les namespaces Linux — 16 — Namespaces dans Kubernetes|16 — Namespaces dans Kubernetes →]]
