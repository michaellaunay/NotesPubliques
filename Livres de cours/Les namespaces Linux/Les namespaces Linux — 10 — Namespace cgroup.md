---
schema_version: 1
uid: 01M1BQ624HM8DFYD1NPYCW7BKM
titre: "Les namespaces Linux — 10 — Namespace cgroup"
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
resume: "Chapitre 10 sur 16 du livre « Les namespaces Linux » : Namespace cgroup. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 10/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 09 — Namespace user|← 09 — Namespace user]] · [[Les namespaces Linux — 11 — Namespace time|11 — Namespace time →]]

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

## 10.1. Rappel : que sont les cgroups ?

## 10.1.1. Définition générale

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

## 10.1.2. Exemple simple

Un processus peut être placé dans un cgroup qui limite sa mémoire à 512 Mio.

Dans ce cas, même si la machine possède 32 Gio de RAM, le processus ne peut pas dépasser la limite imposée par son cgroup.

S’il dépasse cette limite, il peut être tué par le noyau avec un événement de type OOM.

Nous voyons donc que les cgroups répondent à une question différente des namespaces :

```text
Namespaces : que voit le processus ?
Cgroups    : que peut consommer le processus ?
```

---

## 10.2. Pourquoi un namespace cgroup ?

## 10.2.1. Le problème de la vue des cgroups

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

## 10.2.2. Ce que le namespace cgroup isole

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

## 10.2.3. Ce que le namespace cgroup ne fait pas

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

## 10.3. Observer le namespace cgroup courant

## 10.3.1. Avec `/proc/<PID>/ns/cgroup`

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
cgroup:
```

Nous pouvons comparer avec le PID 1 :

```bash
readlink /proc/1/ns/cgroup
```

Si les valeurs sont identiques, notre shell partage le même namespace cgroup que le PID 1.

---

## 10.3.2. Comparer deux processus

Nous pouvons comparer :

```bash
echo "Shell courant : $(readlink /proc/$$/ns/cgroup)"
echo "PID 1         : $(readlink /proc/1/ns/cgroup)"
```

Même valeur :

```text
Shell courant : cgroup:
PID 1         : cgroup:
```

Les deux processus partagent le même namespace cgroup.

Valeurs différentes :

```text
Shell courant : cgroup:
PID 1         : cgroup:
```

Les deux processus ont une vue cgroup différente.

---

## 10.4. Lire `/proc/self/cgroup`

## 10.4.1. Commande de base

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

## 10.4.2. Format avec cgroups v1

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

## 10.4.3. Interprétation

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

## 10.5. Différence entre cgroups et namespace cgroup

## 10.5.1. Les cgroups contrôlent les ressources

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

## 10.5.2. Le namespace cgroup contrôle la vue

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

## 10.5.3. Analogie

Nous pouvons faire une analogie avec le namespace mount.

Le namespace mount ne crée pas les fichiers. Il change la manière dont les montages sont visibles.

De la même façon, le namespace cgroup ne crée pas les limites. Il change la manière dont l’arborescence cgroup est visible.

---

## 10.6. cgroups v1 et cgroups v2

## 10.6.1. Pourquoi cette distinction compte ?

Pour comprendre le namespace cgroup, nous devons savoir que Linux a connu deux grandes versions de cgroups :

```text
cgroups v1
cgroups v2
```

Les deux modèles organisent les ressources différemment.

Les systèmes modernes utilisent de plus en plus cgroups v2, notamment avec systemd, Docker récent et Kubernetes récent.

---

## 10.6.2. cgroups v1

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

## 10.6.3. cgroups v2

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

## 10.7. Observer les limites cgroups v2

## 10.7.1. Mémoire

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

## 10.7.2. CPU

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

## 10.7.3. Nombre de processus

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

## 10.8. Créer un namespace cgroup avec `unshare`

## 10.8.1. Expérience simple

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

## 10.8.2. Permissions

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

## 10.8.3. Ce que nous observons

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

## 10.9. Lien avec systemd

## 10.9.1. systemd organise les processus en cgroups

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

## 10.9.2. `systemd-cgls`

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

## 10.9.3. `systemd-cgtop`

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

## 10.9.4. Limites systemd

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

## 10.10. Lien avec Docker

## 10.10.1. Docker et les cgroups

Docker utilise les cgroups pour limiter les ressources des conteneurs.

Exemples :

```bash
docker run --memory=512m image
docker run --cpus=2 image
docker run --pids-limit=256 image
```

Ces options configurent des cgroups.

Elles ne créent pas seulement des namespaces.

---

## 10.10.2. Observer depuis le conteneur

Dans un conteneur :

```bash
cat /proc/self/cgroup
```

Nous pouvons voir une vue simplifiée.

Sur cgroups v2, nous voyons parfois :

```text
0::/
```

Cela signifie que, dans son namespace cgroup, le conteneur voit son cgroup comme la racine.

Depuis l’hôte, le même processus peut être dans un chemin beaucoup plus long.

---

## 10.10.3. Observer depuis l’hôte

Nous lançons un conteneur :

```bash
docker run --rm -d --name cgroup-demo --memory=512m alpine sleep 1000
```

Nous récupérons son PID :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' cgroup-demo)
```

Nous observons :

```bash
sudo readlink /proc/$pid/ns/cgroup
readlink /proc/1/ns/cgroup

sudo cat /proc/$pid/cgroup
```

Nous pouvons comparer avec :

```bash
docker exec cgroup-demo cat /proc/self/cgroup
```

Nous voyons que la vue depuis le conteneur et la vue depuis l’hôte peuvent être différentes.

---

## 10.10.4. Nettoyage

```bash
docker rm -f cgroup-demo
```

---

## 10.11. Lien avec Kubernetes

## 10.11.1. Pods et cgroups

Kubernetes utilise les cgroups pour appliquer les `requests` et `limits`.

Exemple :

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "1"
```

Ces valeurs se traduisent au niveau du nœud par une configuration de cgroups.

---

## 10.11.2. OOMKilled

Si un conteneur dépasse sa limite mémoire, Kubernetes peut indiquer :

```text
OOMKilled
```

Mais l’événement vient du noyau et des cgroups : le processus dépasse la limite mémoire de son cgroup.

Nous pouvons diagnostiquer avec :

```bash
kubectl describe pod <pod>
kubectl top pod
kubectl logs <pod>
```

Et, sur le nœud, avec les fichiers cgroup si nous avons accès au système.

---

## 10.11.3. Chemins cgroup Kubernetes

Depuis l’hôte, un processus de pod peut apparaître dans un chemin du type :

```text
/kubepods.slice/kubepods-burstable.slice/...
```

ou autre selon :

- runtime utilisé ;
    
- systemd ou cgroupfs ;
    
- cgroups v1 ou v2 ;
    
- distribution ;
    
- configuration du cluster.
    

Le namespace cgroup peut masquer cette complexité depuis l’intérieur du conteneur.

---

## 10.12. Namespace cgroup et `/proc`

## 10.12.1. `/proc/<PID>/cgroup`

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

## 10.12.2. `/proc/<PID>/ns/cgroup`

Nous lisons :

```bash
readlink /proc/<PID>/ns/cgroup
```

Cela nous dit dans quel namespace cgroup se trouve le processus.

Deux processus qui ont le même lien symbolique partagent la même vue cgroup.

---

## 10.12.3. `/sys/fs/cgroup`

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

## 10.13. Cas pratique : comprendre une limite mémoire

## 10.13.1. Lancement d’un conteneur limité

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

## 10.13.2. Interprétation

Même si le conteneur voit parfois une vue cgroup simplifiée, la limite est bien appliquée par le noyau.

Nous distinguons donc :

```text
Vue cgroup dans /proc/self/cgroup
Limite effective dans les fichiers cgroup
```

Le namespace cgroup peut changer la première, mais pas supprimer la seconde.

---

## 10.14. Cas pratique : comparer hôte et conteneur

## 10.14.1. Côté hôte

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

## 10.14.2. Côté conteneur

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

## 10.14.3. Nettoyage

```bash
docker rm -f cgroup-demo
```

---

## 10.15. Pièges classiques

## 10.15.1. Confondre cgroup namespace et cgroups

C’est le piège principal.

Nous devons retenir :

```text
Le namespace cgroup isole la vue.
Les cgroups contrôlent les ressources.
```

Créer un namespace cgroup ne limite pas automatiquement la mémoire, le CPU ou les processus.

---

## 10.15.2. Croire que `/proc/self/cgroup` donne toujours le chemin réel de l’hôte

Dans un conteneur, `/proc/self/cgroup` peut être volontairement simplifié.

Nous ne devons pas supposer que ce chemin correspond au chemin complet de l’hôte.

Pour le voir, nous devons observer depuis l’hôte.

---

## 10.15.3. Croire que `/sys/fs/cgroup` est toujours complet

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

## 10.15.4. Confondre limites Kubernetes et namespace cgroup

Dans Kubernetes, les limites CPU/mémoire viennent des cgroups.

Le namespace cgroup peut masquer le chemin exact.

Mais si un pod est `OOMKilled`, c’est bien une limite cgroup qui est en jeu, pas le namespace cgroup lui-même.

---

## 10.16. Diagnostic cgroup

## 10.16.1. Questions à se poser

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

## 10.16.2. Commandes utiles

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

## 10.17. Sécurité et namespace cgroup

## 10.17.1. Pourquoi masquer la vue cgroup ?

Masquer ou simplifier les chemins cgroups peut éviter de révéler :

- organisation de l’hôte ;
    
- noms de services ;
    
- identifiants de conteneurs ;
    
- structure Kubernetes ;
    
- chemins systemd ;
    
- informations sur le runtime.
    

Ce n’est pas une barrière de sécurité absolue, mais cela réduit l’exposition d’informations internes.

---

## 10.17.2. Écriture dans `/sys/fs/cgroup`

Permettre à un conteneur d’écrire librement dans `/sys/fs/cgroup` peut être dangereux.

Un processus pourrait tenter de :

- modifier ses limites ;
    
- déplacer des processus ;
    
- créer des sous-cgroups ;
    
- perturber l’organisation attendue.
    

C’est pourquoi `/sys/fs/cgroup` est souvent monté en lecture seule ou limité dans les conteneurs.

---

## 10.17.3. Le namespace cgroup ne remplace pas les politiques de sécurité

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

## 10.18. Exercices

## Exercice 1 — Observer le namespace cgroup courant

Nous exécutons :

```bash
readlink /proc/$$/ns/cgroup
readlink /proc/1/ns/cgroup
cat /proc/self/cgroup
```

Nous répondons :

1. Le shell courant partage-t-il le namespace cgroup du PID 1 ?
    
2. Que contient `/proc/self/cgroup` ?
    
3. Sommes-nous probablement dans le namespace cgroup initial ?
    

---

## Exercice 2 — Identifier cgroups v1 ou v2

Nous exécutons :

```bash
findmnt /sys/fs/cgroup
cat /proc/filesystems | grep cgroup
cat /sys/fs/cgroup/cgroup.controllers 2>/dev/null
```

Nous répondons :

1. Le système utilise-t-il cgroups v2 ?
    
2. `/sys/fs/cgroup` est-il une hiérarchie unifiée ?
    
3. Quels contrôleurs sont disponibles ?
    

---

## Exercice 3 — Lire les limites cgroups v2

Si cgroups v2 est disponible :

```bash
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/pids.current
cat /sys/fs/cgroup/pids.max
```

Nous répondons :

1. Quelle mémoire est consommée ?
    
2. Existe-t-il une limite mémoire ?
    
3. Quelle est la limite CPU ?
    
4. Quelle est la limite de processus ?
    

---

## Exercice 4 — Créer un namespace cgroup

Nous exécutons :

```bash
unshare --cgroup bash
readlink /proc/$$/ns/cgroup
cat /proc/self/cgroup
exit
```

Si la commande échoue, nous essayons en environnement de test :

```bash
sudo unshare --cgroup bash
```

Nous répondons :

1. L’identifiant du namespace cgroup a-t-il changé ?
    
2. Le contenu de `/proc/self/cgroup` a-t-il changé ?
    
3. Avons-nous créé une limite mémoire ou CPU ?
    
4. Pourquoi ?
    

---

## Exercice 5 — Observer un service systemd

Nous choisissons un service :

```bash
systemctl status ssh
```

ou :

```bash
systemctl status nginx
```

Puis :

```bash
systemd-cgls
systemd-cgtop
```

Nous répondons :

1. Dans quel cgroup le service apparaît-il ?
    
2. systemd organise-t-il les processus par service ?
    
3. Quelle différence faisons-nous entre service systemd et processus Linux ?
    

---

## Exercice 6 — Observer un conteneur Docker

Si Docker est disponible :

```bash
docker run --rm -d --name cgroup-demo --memory=512m alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' cgroup-demo)

sudo readlink /proc/$pid/ns/cgroup
readlink /proc/1/ns/cgroup

sudo cat /proc/$pid/cgroup
docker exec cgroup-demo cat /proc/self/cgroup
docker exec cgroup-demo sh -c 'cat /sys/fs/cgroup/memory.max 2>/dev/null || true'

docker rm -f cgroup-demo
```

Nous répondons :

1. Le conteneur partage-t-il le namespace cgroup de l’hôte ?
    
2. La vue de `/proc/self/cgroup` est-elle identique dedans et dehors ?
    
3. La limite mémoire est-elle visible ?
    
4. Quelle différence faisons-nous entre vue et limite ?
    

---

## Exercice 7 — Analyse Kubernetes

Nous étudions cette configuration :

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

1. Quelle limite mémoire est demandée ?
    
2. Quelle limite CPU est demandée ?
    
3. Quel mécanisme Linux applique ces limites ?
    
4. Quel rôle joue éventuellement le namespace cgroup ?
    
5. Que signifie un pod `OOMKilled` ?
    

---

## 10.19. Ce que nous devons retenir

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

## Conclusion du chapitre 10

Nous avons étudié le namespace cgroup et clarifié sa relation avec les cgroups.

Nous savons maintenant que les cgroups sont le mécanisme qui contrôle et mesure les ressources, tandis que le namespace cgroup isole la vue qu’un processus a de son rattachement dans l’arborescence cgroup.

Cette distinction est essentielle. Quand un conteneur est limité à 512 Mio de RAM, ce n’est pas le namespace cgroup qui impose cette limite : ce sont les cgroups. Le namespace cgroup peut seulement faire apparaître le chemin du conteneur comme une racine plus propre ou plus isolée dans `/proc/self/cgroup`.

Nous avons également relié ce mécanisme à systemd, Docker et Kubernetes. Nous comprenons désormais pourquoi un service systemd, un conteneur Docker ou un pod Kubernetes apparaissent dans des cgroups, et pourquoi la vue observée depuis l’intérieur d’un conteneur peut différer de celle observée depuis l’hôte.

Dans le chapitre suivant, nous étudions le namespace time, qui permet d’isoler certains offsets d’horloges comme `CLOCK_MONOTONIC` et `CLOCK_BOOTTIME`.

---
> [!info] Livre « Les namespaces Linux » — chapitre 10/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 09 — Namespace user|← 09 — Namespace user]] · [[Les namespaces Linux — 11 — Namespace time|11 — Namespace time →]]
