---
schema_version: 1
uid: 01M1BQ624R78TPP2PCXN0DF9QP
titre: "Les namespaces Linux — 16 — Namespaces dans Kubernetes"
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
resume: "Chapitre 16 sur 16 du livre « Les namespaces Linux » : Namespaces dans Kubernetes. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 16/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 15 — Des namespaces aux conteneurs|← 15 — Des namespaces aux conteneurs]] · [[Les namespaces Linux — Compléments 2026|Compléments 2026 →]]

# Chapitre 16 — Namespaces dans Kubernetes
## Objectifs du chapitre

Dans ce chapitre, nous étudions l’usage des namespaces Linux dans Kubernetes.

Nous avons déjà compris que Docker, Podman et les runtimes OCI utilisent les namespaces pour isoler des processus. Kubernetes reprend ces mêmes mécanismes, mais à une échelle supérieure : il ne gère pas seulement des conteneurs isolés, il orchestre des pods sur un ensemble de nœuds.

Le point central est le suivant :

```text
Dans Kubernetes, l’unité fondamentale n’est pas le conteneur seul.
L’unité fondamentale d’exécution est le pod.
```

À la fin de ce chapitre, nous savons :

- expliquer le rôle du pod comme unité d’isolation ;
    
- comprendre quels namespaces sont partagés entre conteneurs d’un même pod ;
    
- comprendre le rôle du pause container ;
    
- expliquer pourquoi les conteneurs d’un même pod partagent souvent le réseau ;
    
- comprendre les options `hostNetwork`, `hostPID` et `hostIPC` ;
    
- faire le lien entre CNI et namespaces réseau ;
    
- comprendre le rôle du `securityContext` ;
    
- utiliser les ephemeral containers pour le debug ;
    
- relier Kubernetes aux notions de namespaces, cgroups, capabilities et sécurité.
    

---

## 16.1. Rappel : Kubernetes orchestre des conteneurs

## 16.1.1. Kubernetes ne remplace pas le noyau Linux

Kubernetes n’invente pas une nouvelle forme magique d’isolation.

Il s’appuie sur les mécanismes Linux que nous avons étudiés :

```text
namespaces
cgroups
capabilities
seccomp
AppArmor ou SELinux
mounts
réseau virtuel
runtimes de conteneurs
```

Kubernetes demande au runtime de conteneurs de créer les environnements d’exécution.

Selon les installations, ce runtime peut être par exemple :

```text
containerd
CRI-O
```

Ces runtimes utilisent ensuite des runtimes OCI comme `runc` ou d’autres implémentations compatibles.

---

## 16.1.2. Kubernetes travaille au niveau du pod

Dans Docker seul, nous pensons souvent en termes de conteneur.

Dans Kubernetes, nous pensons d’abord en termes de pod.

Un pod peut contenir :

- un seul conteneur ;
    
- plusieurs conteneurs applicatifs ;
    
- un conteneur principal et un ou plusieurs sidecars ;
    
- des init containers ;
    
- des ephemeral containers pour le debug.
    

Le pod est donc une enveloppe logique qui regroupe un ou plusieurs conteneurs partageant certaines ressources.

---

## 16.2. Qu’est-ce qu’un pod ?

## 16.2.1. Définition

Un pod est la plus petite unité déployable dans Kubernetes.

Il représente un groupe de conteneurs qui doivent fonctionner ensemble sur le même nœud.

Ces conteneurs partagent généralement :

```text
un namespace réseau
une adresse IP de pod
certains volumes
une durée de vie commune
un contexte de planification
```

Ils ne sont pas répartis sur plusieurs machines : tous les conteneurs d’un même pod s’exécutent sur le même nœud.

---

## 16.2.2. Exemple simple de pod à un conteneur

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-simple
spec:
  containers:
    - name: app
      image: nginx:alpine
      ports:
        - containerPort: 80
```

Ce pod contient un seul conteneur.

Même dans ce cas simple, Kubernetes crée un environnement réseau, des montages, des cgroups et des namespaces autour de ce conteneur.

---

## 16.2.3. Exemple de pod avec sidecar

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-sidecar
spec:
  containers:
    - name: app
      image: mon-app:latest
      ports:
        - containerPort: 8080

    - name: sidecar-logs
      image: fluent-bit:latest
```

Ici, le pod contient deux conteneurs :

```text
app
sidecar-logs
```

Ils vivent ensemble et peuvent partager certains volumes et le réseau du pod.

---

## 16.3. Namespaces Linux et namespaces Kubernetes

## 16.3.1. Attention à l’ambiguïté du mot namespace

Kubernetes utilise aussi le mot `namespace`.

Mais un namespace Kubernetes n’est pas un namespace Linux.

Nous devons distinguer :

|Terme|Signification|
|---|---|
|Namespace Linux|mécanisme noyau d’isolation des processus|
|Namespace Kubernetes|espace logique pour organiser les objets Kubernetes|

Un namespace Kubernetes sert à organiser des objets comme :

```text
pods
services
deployments
configmaps
secrets
roles
```

Exemples :

```bash
kubectl get namespaces
kubectl get pods -n default
kubectl get pods -n kube-system
```

Cela n’a pas le même sens que :

```bash
readlink /proc/$$/ns/net
```

---

## 16.3.2. Exemple de namespace Kubernetes

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

Ce namespace Kubernetes organise les ressources nommées `production`.

Il ne crée pas directement un namespace Linux.

Un pod dans le namespace Kubernetes `production` aura néanmoins ses propres namespaces Linux au moment de son exécution sur un nœud.

---

## 16.3.3. Règle de vocabulaire

Nous retenons :

```text
Namespace Kubernetes = organisation logique du cluster.
Namespace Linux = isolation noyau d’un processus.
```

Dans ce chapitre, quand nous parlons de namespaces sans précision, nous parlons des namespaces Linux utilisés par les pods et conteneurs.

---

## 16.4. Les namespaces Linux dans un pod

## 16.4.1. Vue d’ensemble

Un pod Kubernetes utilise plusieurs namespaces Linux.

Selon la configuration, il peut utiliser :

```text
network namespace
PID namespace
mount namespace
UTS namespace
IPC namespace
user namespace
cgroup namespace
time namespace selon runtime et noyau
```

Mais tous ne sont pas nécessairement partagés de la même manière entre les conteneurs du pod.

---

## 16.4.2. Partage classique

Dans un pod classique :

|Namespace Linux|Comportement typique|
|---|---|
|Network|partagé entre les conteneurs du pod|
|Mount|propre à chaque conteneur, avec volumes communs montés|
|PID|souvent séparé par conteneur, sauf option de partage|
|IPC|généralement isolé du host, modalités selon runtime|
|UTS|identité liée au pod/conteneur selon runtime|
|User|dépend de la configuration et du support|
|Cgroup|organisé par pod et conteneur|
|Time|rarement manipulé directement|

Le point le plus important est le partage du namespace réseau.

---

## 16.5. Le namespace réseau du pod

## 16.5.1. Une IP par pod

Dans Kubernetes, chaque pod reçoit généralement sa propre adresse IP.

Cette IP appartient au pod, pas à chaque conteneur individuellement.

Exemple :

```bash
kubectl get pod -o wide
```

Sortie possible :

```text
NAME        READY   STATUS    IP           NODE
web-abc     1/1     Running   10.244.1.23  node-1
```

L’adresse `10.244.1.23` est l’IP du pod.

---

## 16.5.2. Les conteneurs d’un même pod partagent le réseau

Si un pod contient deux conteneurs, ils partagent généralement le même namespace réseau.

Cela signifie qu’ils partagent :

```text
la même IP
les mêmes interfaces
les mêmes routes
le même loopback
les mêmes ports
la même pile réseau
```

Conséquence importante :

```text
Deux conteneurs d’un même pod peuvent communiquer via localhost.
```

---

## 16.5.3. Exemple : app et sidecar

Imaginons un pod :

```text
Pod
├── app écoute sur 127.0.0.1:8080
└── sidecar proxy
```

Le sidecar peut joindre l’application avec :

```text
http://127.0.0.1:8080
```

car il partage le même namespace réseau.

C’est la base de nombreux modèles sidecar :

- proxy service mesh ;
    
- collecteur de logs ;
    
- agent de supervision ;
    
- proxy de sécurité ;
    
- adaptateur réseau local.
    

---

## 16.5.4. Conséquence sur les ports

Comme les conteneurs d’un pod partagent le même namespace réseau, ils ne peuvent pas écouter sur le même port et la même adresse en même temps.

Exemple problématique :

```text
container A écoute sur 0.0.0.0:8080
container B essaie aussi d’écouter sur 0.0.0.0:8080
```

Le second échoue généralement avec :

```text
Address already in use
```

C’est normal : dans le pod, le port appartient au namespace réseau partagé.

---

## 16.6. Le pause container

## 16.6.1. Pourquoi existe-t-il ?

Kubernetes utilise généralement un conteneur spécial appelé `pause container`, ou conteneur sandbox.

Son rôle est de maintenir certains namespaces du pod, en particulier le namespace réseau.

Conceptuellement :

```text
pause container
  détient le namespace réseau du pod

container app
  rejoint ce namespace réseau

container sidecar
  rejoint ce même namespace réseau
```

Le pause container est très minimal. Il ne fait presque rien, mais il sert d’ancrage aux namespaces du pod.

---

## 16.6.2. Pourquoi ne pas attacher le réseau au conteneur applicatif ?

Si le namespace réseau était attaché uniquement au premier conteneur applicatif, que se passerait-il si ce conteneur redémarrait ?

Kubernetes veut que le pod conserve une identité réseau stable pendant la durée de vie du pod.

Le pause container aide à maintenir cette stabilité.

Ainsi, les autres conteneurs peuvent redémarrer sans nécessairement recréer toute la sandbox réseau du pod.

---

## 16.6.3. Observer le pause container

Sur un nœud, avec les outils du runtime, nous pouvons parfois voir des conteneurs pause.

Avec `crictl`, par exemple :

```bash
sudo crictl ps -a
```

Nous pouvons voir des images ou noms liés à `pause`.

Selon le runtime et la distribution Kubernetes, les détails peuvent varier.

---

## 16.7. CNI et namespace réseau

## 16.7.1. Qu’est-ce que CNI ?

CNI signifie Container Network Interface.

C’est une spécification qui permet aux runtimes de déléguer la configuration réseau des pods à des plugins.

Kubernetes ne configure pas directement toute la connectivité réseau. Il appelle un plugin CNI.

Exemples de solutions CNI :

```text
Calico
Cilium
Flannel
Canal
Antrea
Weave Net
```

---

## 16.7.2. Ce que fait un plugin CNI

Un plugin CNI configure généralement :

- le namespace réseau du pod ;
    
- une interface dans le pod, souvent `eth0` ;
    
- une paire `veth` ;
    
- les routes ;
    
- la connectivité entre nœuds ;
    
- les règles de pare-feu ;
    
- éventuellement eBPF ;
    
- les NetworkPolicies.
    

Schéma conceptuel :

```text
pod network namespace
  eth0
   |
veth
   |
nœud
   |
CNI / bridge / routage / eBPF / overlay
   |
réseau cluster
```

---

## 16.7.3. Observer depuis un pod

Dans un pod :

```bash
ip addr
ip route
cat /proc/net/dev
```

Nous voyons la pile réseau du pod.

Mais dans une image minimale, `ip` peut manquer. Dans ce cas, nous pouvons utiliser un conteneur de debug ou entrer depuis le nœud avec `nsenter`.

---

## 16.7.4. NetworkPolicy

Les NetworkPolicies Kubernetes permettent de définir quelles communications sont autorisées.

Elles ne sont pas appliquées par les namespaces Linux seuls.

Elles dépendent du plugin CNI.

Nous retenons :

```text
Namespace réseau = isolation de la pile réseau.
CNI = configuration de la connectivité.
NetworkPolicy = politique de communication, si le CNI la supporte.
```

---

## 16.8. PID namespace dans Kubernetes

## 16.8.1. Vue par défaut

Par défaut, les processus d’un conteneur sont généralement isolés des autres conteneurs et de l’hôte au niveau PID.

Dans un conteneur :

```bash
ps
```

nous voyons souvent seulement les processus du conteneur.

Mais Kubernetes peut configurer le partage de processus au niveau du pod.

---

## 16.8.2. `shareProcessNamespace`

Kubernetes permet :

```yaml
spec:
  shareProcessNamespace: true
```

Cela permet aux conteneurs d’un même pod de voir leurs processus respectifs.

Exemple :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pid-shared-demo
spec:
  shareProcessNamespace: true
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "sleep 3600"]

    - name: debug
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
```

Dans le conteneur `debug`, nous pouvons voir les processus du conteneur `app`.

---

## 16.8.3. Pourquoi partager les processus ?

Cela peut être utile pour :

- debug ;
    
- sidecar de supervision ;
    
- agents de diagnostic ;
    
- collecte de traces ;
    
- gestion de processus entre conteneurs du même pod.
    

Mais cela réduit l’isolation entre conteneurs du pod.

Nous l’activons seulement si le besoin est clair.

---

## 16.8.4. `hostPID`

Kubernetes permet aussi :

```yaml
spec:
  hostPID: true
```

Dans ce cas, le pod partage le namespace PID de l’hôte.

C’est beaucoup plus sensible.

Conséquences :

- le pod voit les processus du nœud ;
    
- il peut exposer des informations sensibles ;
    
- il peut interagir avec certains processus selon permissions ;
    
- il réduit fortement l’isolation.
    

Nous évitons `hostPID: true` sauf cas spécifique, par exemple certains agents système.

---

## 16.9. Namespace network partagé avec l’hôte : `hostNetwork`

## 16.9.1. Principe

Kubernetes permet :

```yaml
spec:
  hostNetwork: true
```

Dans ce cas, le pod partage le namespace réseau de l’hôte.

Il ne reçoit pas une IP de pod classique de la même manière.

Il utilise directement la pile réseau du nœud.

---

## 16.9.2. Conséquences

Avec `hostNetwork: true` :

- le pod voit les interfaces réseau du nœud ;
    
- il partage les ports du nœud ;
    
- il peut écouter directement sur les adresses du nœud ;
    
- il peut entrer en conflit avec d’autres services du nœud ;
    
- il contourne une partie de l’isolation réseau habituelle ;
    
- le diagnostic réseau change.
    

Exemple :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostnetwork-demo
spec:
  hostNetwork: true
  containers:
    - name: app
      image: nginx:alpine
```

---

## 16.9.3. Cas d’usage possibles

`hostNetwork` peut être utilisé pour :

- agents réseau ;
    
- composants de monitoring bas niveau ;
    
- certains DaemonSets ;
    
- composants CNI ;
    
- outils qui doivent écouter directement sur le réseau du nœud.
    

Mais nous ne l’utilisons pas pour une application standard sans raison forte.

---

## 16.10. Namespace IPC dans Kubernetes

## 16.10.1. Isolation IPC

Les pods utilisent généralement une isolation IPC par rapport à l’hôte.

Les ressources IPC de l’hôte ne sont pas directement visibles depuis les conteneurs ordinaires.

---

## 16.10.2. `hostIPC`

Kubernetes permet :

```yaml
spec:
  hostIPC: true
```

Dans ce cas, le pod partage le namespace IPC de l’hôte.

C’est sensible.

Conséquences :

- visibilité sur certaines ressources IPC de l’hôte ;
    
- interaction possible avec des segments de mémoire partagée, sémaphores ou files de messages selon permissions ;
    
- réduction de l’isolation.
    

Nous l’évitons sauf besoin système très spécifique.

---

## 16.10.3. `/dev/shm`

Certaines applications ont besoin d’un `/dev/shm` plus grand.

Nous pouvons utiliser un volume `emptyDir` en mémoire :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shm-demo
spec:
  volumes:
    - name: dshm
      emptyDir:
        medium: Memory
        sizeLimit: 1Gi
  containers:
    - name: app
      image: mon-app:latest
      volumeMounts:
        - name: dshm
          mountPath: /dev/shm
```

Cela évite d’utiliser `hostIPC` pour un simple problème de mémoire partagée.

---

## 16.11. Mount namespaces et volumes Kubernetes

## 16.11.1. Chaque conteneur a sa vue de montages

Chaque conteneur d’un pod a généralement son propre namespace mount.

Cela signifie que chaque conteneur peut avoir une vue de fichiers adaptée.

Mais Kubernetes peut monter les mêmes volumes dans plusieurs conteneurs.

---

## 16.11.2. Exemple de volume partagé

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-shared-demo
spec:
  volumes:
    - name: shared-data
      emptyDir: {}
  containers:
    - name: producer
      image: busybox
      command: ["sh", "-c", "echo hello > /data/message.txt && sleep 3600"]
      volumeMounts:
        - name: shared-data
          mountPath: /data

    - name: consumer
      image: busybox
      command: ["sh", "-c", "cat /data/message.txt; sleep 3600"]
      volumeMounts:
        - name: shared-data
          mountPath: /data
```

Les deux conteneurs ont chacun leur namespace mount, mais ils voient le même volume à `/data`.

---

## 16.11.3. Types de volumes

Kubernetes peut monter différents types de volumes :

```text
emptyDir
configMap
secret
persistentVolumeClaim
hostPath
projected
downwardAPI
CSI volumes
```

Chaque volume est intégré dans la vue de montage du conteneur.

---

## 16.11.4. `hostPath`

`hostPath` monte un chemin du nœud dans le pod.

Exemple :

```yaml
volumes:
  - name: host-logs
    hostPath:
      path: /var/log
```

C’est puissant, mais sensible.

Risques :

- exposition de fichiers du nœud ;
    
- modification accidentelle ;
    
- dépendance au nœud ;
    
- réduction de portabilité ;
    
- risque de sécurité si le chemin est critique.
    

Nous évitons `hostPath` sauf besoin bien justifié.

---

## 16.12. Cgroups dans Kubernetes

## 16.12.1. Requests et limits

Kubernetes utilise les cgroups pour appliquer les limites.

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

Nous interprétons :

```text
request memory = mémoire demandée pour la planification
limit memory   = mémoire maximale autorisée
request cpu    = CPU demandé
limit cpu      = CPU maximal autorisé
```

---

## 16.12.2. Organisation par pod et conteneur

Sur le nœud, les processus des pods sont organisés dans une hiérarchie cgroup.

Nous pouvons voir des chemins liés à :

```text
kubepods
burstable
besteffort
guaranteed
pod UID
container ID
```

Les détails dépendent de :

- cgroups v1 ou v2 ;
    
- systemd ou cgroupfs ;
    
- runtime ;
    
- distribution ;
    
- configuration du cluster.
    

---

## 16.12.3. OOMKilled

Si un conteneur dépasse sa limite mémoire, le noyau peut tuer le processus.

Kubernetes affiche alors souvent :

```text
OOMKilled
```

Nous diagnostiquons avec :

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl top pod
kubectl get events
```

Le mécanisme bas niveau est le contrôleur mémoire des cgroups.

---

## 16.13. SecurityContext

## 16.13.1. Rôle

Le `securityContext` permet de configurer des paramètres de sécurité au niveau du pod ou du conteneur.

Il peut définir notamment :

- utilisateur d’exécution ;
    
- groupe ;
    
- capabilities ;
    
- mode privilégié ;
    
- seccomp ;
    
- root filesystem en lecture seule ;
    
- élévation de privilèges ;
    
- SELinux ;
    
- AppArmor selon configuration.
    

---

## 16.13.2. Exemple de conteneur durci

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-demo
spec:
  containers:
    - name: app
      image: nginx:alpine
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
        seccompProfile:
          type: RuntimeDefault
```

Cette configuration applique plusieurs bonnes pratiques :

```text
non-root
pas d’élévation de privilèges
rootfs en lecture seule
capabilities supprimées
seccomp par défaut du runtime
```

---

## 16.13.3. `privileged`

Kubernetes permet :

```yaml
securityContext:
  privileged: true
```

C’est très sensible.

Un conteneur privilégié reçoit des droits très larges.

Nous ne l’utilisons pas pour contourner rapidement un problème de permission. Nous cherchons d’abord le besoin exact :

```text
capability manquante
périphérique nécessaire
volume nécessaire
profil seccomp trop restrictif
AppArmor ou SELinux
```

---

## 16.13.4. Capabilities

Nous pouvons ajouter ou retirer des capabilities :

```yaml
securityContext:
  capabilities:
    drop:
      - ALL
    add:
      - NET_BIND_SERVICE
```

Nous préférons supprimer toutes les capabilities puis ajouter uniquement celles nécessaires.

---

## 16.14. User namespaces dans Kubernetes

## 16.14.1. Pourquoi c’est utile

Le user namespace permet de mapper l’UID `0` dans le conteneur vers un UID non privilégié sur l’hôte.

Cela réduit l’impact potentiel d’une compromission.

Dans Kubernetes, l’usage des user namespaces dépend du runtime, de la version du cluster, de la configuration des nœuds et des fonctionnalités activées.

---

## 16.14.2. Différence avec `runAsUser`

`runAsUser` change l’utilisateur utilisé dans le conteneur.

Exemple :

```yaml
securityContext:
  runAsUser: 10001
```

Mais cela ne crée pas nécessairement un user namespace.

Nous devons distinguer :

```text
runAsUser    = UID utilisé par le processus dans le conteneur
user namespace = mapping UID/GID entre conteneur et hôte
```

Les deux mécanismes peuvent se compléter, mais ils ne sont pas identiques.

---

## 16.14.3. Problèmes avec les volumes

Les user namespaces peuvent compliquer les permissions sur les volumes.

Un fichier peut apparaître avec un UID différent selon :

- le mapping UID/GID ;
    
- le type de volume ;
    
- le système de fichiers ;
    
- le runtime ;
    
- la politique de sécurité ;
    
- l’utilisateur d’exécution.
    

Nous diagnostiquons avec :

```bash
id
ls -ln /chemin
cat /proc/self/uid_map
cat /proc/self/gid_map
```

---

## 16.15. Ephemeral containers pour le debug

## 16.15.1. Problème des images minimales

Les images de production sont souvent minimales.

Elles peuvent ne pas contenir :

```text
sh
bash
ip
ss
ps
curl
dig
tcpdump
strace
```

Cela complique le diagnostic.

---

## 16.15.2. Principe des ephemeral containers

Les ephemeral containers permettent d’ajouter temporairement un conteneur de debug dans un pod existant.

Nous pouvons utiliser :

```bash
kubectl debug -it <pod> --image=busybox --target=<container>
```

Selon le cluster et les permissions, cela ajoute un conteneur de debug qui peut partager certains namespaces avec le pod ou cibler un conteneur existant.

---

## 16.15.3. Intérêt

Cela permet de diagnostiquer :

- réseau du pod ;
    
- DNS ;
    
- montages ;
    
- processus selon configuration ;
    
- fichiers visibles ;
    
- environnement d’exécution.
    

Nous évitons ainsi de mettre des outils de debug dans les images de production.

---

## 16.15.4. Limites

Les ephemeral containers :

- nécessitent des droits Kubernetes ;
    
- ne sont pas faits pour exécuter une charge durable ;
    
- peuvent ne pas partager tous les namespaces selon configuration ;
    
- doivent respecter les politiques de sécurité du cluster ;
    
- ne résolvent pas tous les problèmes d’observabilité.
    

---

## 16.16. Debug réseau dans Kubernetes

## 16.16.1. Questions à se poser

Quand un pod ne communique pas, nous demandons :

```text
Le pod a-t-il une IP ?
L’interface eth0 est-elle présente ?
La route par défaut existe-t-elle ?
Le DNS fonctionne-t-il ?
Le service Kubernetes pointe-t-il vers les bons endpoints ?
La NetworkPolicy autorise-t-elle le trafic ?
Le CNI fonctionne-t-il ?
Le problème vient-il du pod, du service, du nœud ou du cluster ?
```

---

## 16.16.2. Commandes utiles

```bash
kubectl get pod -o wide
kubectl describe pod <pod>
kubectl get svc
kubectl get endpoints
kubectl exec -it <pod> -- ip addr
kubectl exec -it <pod> -- ip route
kubectl exec -it <pod> -- nslookup kubernetes.default
```

Si l’image ne contient pas les outils :

```bash
kubectl debug -it <pod> --image=nicolaka/netshoot
```

Selon les politiques du cluster, l’image de debug peut être différente.

---

## 16.16.3. Comprendre `localhost`

Dans un pod multi-conteneurs :

```text
localhost désigne le namespace réseau partagé du pod.
```

Donc si un conteneur `app` écoute sur `127.0.0.1:8080`, un sidecar du même pod peut le joindre avec :

```text
http://127.0.0.1:8080
```

Mais un autre pod ne peut pas utiliser ce `localhost`. Il doit utiliser l’IP du pod ou un Service Kubernetes.

---

## 16.17. Debug PID dans Kubernetes

## 16.17.1. Voir les processus

Dans un conteneur :

```bash
kubectl exec -it <pod> -- ps -ef
```

Nous voyons les processus visibles dans le namespace PID du conteneur ou du pod selon configuration.

---

## 16.17.2. Avec `shareProcessNamespace`

Si le pod a :

```yaml
shareProcessNamespace: true
```

un conteneur de debug peut voir les processus des autres conteneurs du pod.

Cela est utile pour :

- observer un processus ;
    
- envoyer un signal ;
    
- lire `/proc/<PID>` selon permissions ;
    
- diagnostiquer des zombies ;
    
- analyser le PID 1.
    

---

## 16.17.3. Avec `hostPID`

Si le pod a :

```yaml
hostPID: true
```

il voit les processus du nœud.

C’est utile pour certains agents système, mais sensible.

Nous devons éviter cette option pour les applications standards.

---

## 16.18. Debug des montages dans Kubernetes

## 16.18.1. Problèmes fréquents

Les problèmes de montage peuvent venir de :

- volume absent ;
    
- mauvais `mountPath` ;
    
- secret ou configMap non monté ;
    
- permissions incorrectes ;
    
- `readOnly` activé ;
    
- `subPath` mal utilisé ;
    
- PVC non attaché ;
    
- `hostPath` dépendant du nœud ;
    
- root filesystem en lecture seule.
    

---

## 16.18.2. Commandes utiles

```bash
kubectl describe pod <pod>
kubectl exec -it <pod> -- mount
kubectl exec -it <pod> -- findmnt
kubectl exec -it <pod> -- ls -la /chemin
kubectl exec -it <pod> -- id
kubectl exec -it <pod> -- ls -ln /chemin
```

Si `findmnt` n’est pas disponible, nous pouvons utiliser :

```bash
kubectl exec -it <pod> -- cat /proc/self/mountinfo
```

---

## 16.18.3. `readOnlyRootFilesystem`

Si nous activons :

```yaml
readOnlyRootFilesystem: true
```

l’application ne peut plus écrire partout.

Nous devons prévoir des volumes pour les chemins nécessaires :

```yaml
volumeMounts:
  - name: tmp
    mountPath: /tmp
volumes:
  - name: tmp
    emptyDir: {}
```

Nous devons donc connaître les besoins d’écriture de l’application.

---

## 16.19. Options sensibles dans Kubernetes

## 16.19.1. `hostNetwork`

```yaml
hostNetwork: true
```

Risque :

```text
partage du réseau du nœud
conflits de ports
exposition directe
réduction de l’isolation réseau
```

---

## 16.19.2. `hostPID`

```yaml
hostPID: true
```

Risque :

```text
visibilité sur les processus du nœud
fuite d’informations
interactions possibles selon permissions
```

---

## 16.19.3. `hostIPC`

```yaml
hostIPC: true
```

Risque :

```text
partage des IPC du nœud
exposition possible de mémoire partagée ou sémaphores
```

---

## 16.19.4. `privileged`

```yaml
securityContext:
  privileged: true
```

Risque :

```text
capabilities étendues
accès plus large aux périphériques
affaiblissement de seccomp/AppArmor/SELinux selon configuration
```

---

## 16.19.5. `hostPath`

```yaml
hostPath:
  path: /
```

Risque :

```text
exposition du système de fichiers du nœud
modifications accidentelles ou malveillantes
dépendance forte au nœud
```

Nous traitons ces options comme des exceptions, pas comme des paramètres ordinaires.

---

## 16.20. Méthode d’analyse d’un pod

## 16.20.1. Étape 1 : identifier le pod et le nœud

```bash
kubectl get pod <pod> -o wide
kubectl describe pod <pod>
```

Nous regardons :

- nœud ;
    
- IP du pod ;
    
- conteneurs ;
    
- événements ;
    
- volumes ;
    
- securityContext ;
    
- ressources ;
    
- redémarrages.
    

---

## 16.20.2. Étape 2 : observer depuis le pod

```bash
kubectl exec -it <pod> -- hostname
kubectl exec -it <pod> -- id
kubectl exec -it <pod> -- cat /proc/self/cgroup
kubectl exec -it <pod> -- ls -l /proc/self/ns
kubectl exec -it <pod> -- ps -ef
kubectl exec -it <pod> -- cat /proc/self/mountinfo
```

Selon l’image, certaines commandes peuvent manquer.

---

## 16.20.3. Étape 3 : utiliser un conteneur de debug

```bash
kubectl debug -it <pod> --image=busybox
```

ou une image plus riche :

```bash
kubectl debug -it <pod> --image=nicolaka/netshoot
```

Nous utilisons ce conteneur pour vérifier :

- DNS ;
    
- réseau ;
    
- routes ;
    
- ports ;
    
- montages ;
    
- processus selon partage ;
    
- accès aux services.
    

---

## 16.20.4. Étape 4 : analyser les options sensibles

Nous cherchons dans le manifeste :

```yaml
hostNetwork: true
hostPID: true
hostIPC: true
securityContext:
  privileged: true
hostPath:
```

Si ces options sont présentes, nous évaluons leur justification et leur impact.

---

## 16.21. Exemple complet de pod multi-conteneurs

## 16.21.1. Manifeste

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-demo
spec:
  volumes:
    - name: shared
      emptyDir: {}

  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "echo hello > /shared/message.txt; nc -lk -p 8080"]
      volumeMounts:
        - name: shared
          mountPath: /shared

    - name: sidecar
      image: busybox
      command: ["sh", "-c", "while true; do cat /shared/message.txt; sleep 10; done"]
      volumeMounts:
        - name: shared
          mountPath: /shared
```

Ce pod illustre :

- volume partagé ;
    
- namespace réseau partagé ;
    
- conteneurs distincts ;
    
- durée de vie commune.
    

---

## 16.21.2. Observations

Nous pouvons tester :

```bash
kubectl apply -f pod.yaml
kubectl get pod multi-container-demo -o wide
kubectl exec -it multi-container-demo -c app -- hostname
kubectl exec -it multi-container-demo -c sidecar -- cat /shared/message.txt
```

Pour tester le réseau partagé, si les outils sont disponibles :

```bash
kubectl exec -it multi-container-demo -c sidecar -- wget -qO- http://127.0.0.1:8080
```

Nous observons que le sidecar rejoint le service exposé localement par `app`, car ils partagent le namespace réseau du pod.

---

## 16.22. Travaux pratiques

## TP 1 — Observer les namespaces d’un pod

Dans un pod disposant de `ls` :

```bash
kubectl exec -it <pod> -- ls -l /proc/self/ns
```

Nous répondons :

1. Quels namespaces voyons-nous ?
    
2. Le pod voit-il un namespace réseau ?
    
3. Le namespace PID est-il partagé avec les autres conteneurs ?
    
4. Que pouvons-nous déduire de `/proc/self/cgroup` ?
    

---

## TP 2 — Vérifier le partage réseau dans un pod

Nous créons un pod avec deux conteneurs :

- `app` écoute sur `127.0.0.1:8080` ;
    
- `debug` tente de joindre `127.0.0.1:8080`.
    

Nous répondons :

1. Pourquoi `localhost` fonctionne-t-il entre deux conteneurs du même pod ?
    
2. Pourquoi cela ne fonctionnerait-il pas depuis un autre pod ?
    
3. Quel namespace explique ce comportement ?
    

---

## TP 3 — Tester `shareProcessNamespace`

Nous créons un pod avec :

```yaml
shareProcessNamespace: true
```

Puis nous exécutons :

```bash
kubectl exec -it <pod> -c debug -- ps -ef
```

Nous répondons :

1. Voyons-nous les processus de l’autre conteneur ?
    
2. Quel namespace est partagé ?
    
3. Quels sont les avantages et risques ?
    

---

## TP 4 — Observer les volumes

Nous créons un volume `emptyDir` monté dans deux conteneurs.

Nous répondons :

1. Le fichier créé par un conteneur est-il visible par l’autre ?
    
2. Les conteneurs partagent-ils nécessairement le même namespace mount ?
    
3. Comment Kubernetes rend-il le volume visible dans les deux conteneurs ?
    

---

## TP 5 — Analyser un `securityContext`

Nous analysons :

```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL
  seccompProfile:
    type: RuntimeDefault
```

Nous répondons :

1. Quel risque réduit chaque option ?
    
2. Quelle option concerne les capabilities ?
    
3. Quelle option concerne seccomp ?
    
4. Que devons-nous prévoir si le rootfs est en lecture seule ?
    

---

## TP 6 — Identifier les options sensibles

Nous analysons un manifeste contenant :

```yaml
hostNetwork: true
hostPID: true
hostIPC: true
securityContext:
  privileged: true
volumes:
  - name: host
    hostPath:
      path: /
```

Nous répondons :

1. Quelles isolations sont réduites ?
    
2. Quels risques apparaissent ?
    
3. Quelles alternatives plus ciblées proposons-nous ?
    

---

## 16.23. Pièges classiques

## 16.23.1. Confondre namespace Kubernetes et namespace Linux

Un namespace Kubernetes comme `production` n’est pas un namespace Linux.

Il organise les objets du cluster.

Les namespaces Linux isolent les processus sur les nœuds.

---

## 16.23.2. Croire qu’un conteneur d’un pod a sa propre IP

Dans Kubernetes, l’IP est généralement celle du pod.

Les conteneurs du même pod partagent le namespace réseau.

Ils partagent donc la même IP.

---

## 16.23.3. Oublier les conflits de ports dans un pod

Deux conteneurs d’un même pod ne peuvent pas écouter sur le même port et la même adresse en même temps.

Ils partagent le même namespace réseau.

---

## 16.23.4. Utiliser `hostNetwork` pour résoudre trop vite un problème réseau

Si un pod n’arrive pas à communiquer, nous ne devons pas passer immédiatement à :

```yaml
hostNetwork: true
```

Nous devons d’abord diagnostiquer :

- CNI ;
    
- DNS ;
    
- Service ;
    
- Endpoints ;
    
- NetworkPolicy ;
    
- routes ;
    
- ports ;
    
- application.
    

---

## 16.23.5. Utiliser `privileged` pour résoudre un problème de permission

Même logique :

```yaml
privileged: true
```

ne doit pas être une solution réflexe.

Nous cherchons le besoin précis :

- capability ;
    
- volume ;
    
- user ;
    
- groupe ;
    
- périphérique ;
    
- profil seccomp ;
    
- SELinux/AppArmor.
    

---

## 16.24. Ce que nous devons retenir

Nous retenons les points suivants :

1. Kubernetes utilise les namespaces Linux via les runtimes de conteneurs.
    
2. L’unité fondamentale d’exécution Kubernetes est le pod.
    
3. Un namespace Kubernetes n’est pas un namespace Linux.
    
4. Les conteneurs d’un même pod partagent généralement le namespace réseau.
    
5. Ils partagent donc la même IP et le même `localhost`.
    
6. Le pause container sert à maintenir certains namespaces du pod.
    
7. Le CNI configure le namespace réseau du pod et la connectivité cluster.
    
8. Les NetworkPolicies dépendent du support du plugin CNI.
    
9. Le namespace PID peut être partagé avec `shareProcessNamespace`.
    
10. `hostPID`, `hostNetwork` et `hostIPC` réduisent fortement l’isolation.
    
11. Les volumes Kubernetes sont montés dans les namespaces mount des conteneurs.
    
12. `hostPath` est puissant mais sensible.
    
13. Les cgroups appliquent les requests et limits.
    
14. `OOMKilled` vient généralement d’un dépassement de limite mémoire cgroup.
    
15. `securityContext` permet de configurer utilisateur, capabilities, seccomp et autres paramètres de sécurité.
    
16. Les ephemeral containers facilitent le debug sans modifier l’image applicative.
    
17. Pour diagnostiquer un pod, nous devons croiser namespaces, CNI, volumes, cgroups, securityContext et événements Kubernetes.
    

---

## Conclusion du chapitre 16

Nous avons étudié l’usage des namespaces Linux dans Kubernetes.

Nous savons maintenant que Kubernetes ne remplace pas les mécanismes Linux : il les orchestre. Le pod est l’unité centrale, et plusieurs conteneurs d’un même pod partagent généralement le même namespace réseau, donc la même IP et le même `localhost`.

Nous avons aussi compris le rôle du pause container, qui sert d’ancrage aux namespaces du pod, ainsi que le rôle du CNI, qui configure la connectivité réseau.

Nous avons relié les namespaces aux options Kubernetes sensibles comme `hostNetwork`, `hostPID`, `hostIPC`, `privileged` et `hostPath`. Ces options peuvent être nécessaires dans certains cas système, mais elles réduisent fortement l’isolation et doivent donc être justifiées.

Nous retenons surtout que diagnostiquer Kubernetes demande de raisonner à plusieurs niveaux : pod, conteneur, namespace Linux, cgroups, volumes, réseau CNI, sécurité et événements du cluster.

Dans le chapitre suivant, nous passons aux travaux pratiques pour manipuler directement les namespaces et consolider l’ensemble du cours.

---
> [!info] Livre « Les namespaces Linux » — chapitre 16/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 15 — Des namespaces aux conteneurs|← 15 — Des namespaces aux conteneurs]] · [[Les namespaces Linux — Compléments 2026|Compléments 2026 →]]
