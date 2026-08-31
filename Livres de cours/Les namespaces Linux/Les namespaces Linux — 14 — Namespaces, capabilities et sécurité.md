---
schema_version: 1
uid: 01M1BQ624NCQC2AXRZB330JQAX
titre: "Les namespaces Linux — 14 — Namespaces, capabilities et sécurité"
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
resume: "Chapitre 14 sur 16 du livre « Les namespaces Linux » : Namespaces, capabilities et sécurité. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 14/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 13 — Namespaces et cgroups|← 13 — Namespaces et cgroups]] · [[Les namespaces Linux — 15 — Des namespaces aux conteneurs|15 — Des namespaces aux conteneurs →]]

# Chapitre 14 — Namespaces, capabilities et sécurité
## Objectifs du chapitre

Dans ce chapitre, nous étudions la sécurité autour des namespaces Linux.

Jusqu’ici, nous avons vu que les namespaces permettent d’isoler différentes vues du système :

```text
PID
réseau
montages
utilisateurs
IPC
hostname
cgroups
temps
```

Nous avons aussi vu que les cgroups permettent de limiter les ressources.

Mais cela ne suffit pas à garantir une isolation robuste.

Un conteneur ou un processus isolé peut encore être dangereux si :

- il possède trop de privilèges ;
    
- il dispose de capabilities trop larges ;
    
- il accède à des montages sensibles ;
    
- il partage certains namespaces avec l’hôte ;
    
- il peut appeler trop d’appels système ;
    
- il n’est pas limité par AppArmor, SELinux ou seccomp ;
    
- il tourne en mode privilégié ;
    
- il a accès à `/proc`, `/sys`, `/dev` ou au socket Docker de l’hôte.
    

À la fin de ce chapitre, nous savons :

- expliquer pourquoi les namespaces ne suffisent pas à sécuriser un conteneur ;
    
- comprendre le rôle des capabilities Linux ;
    
- expliquer pourquoi `CAP_SYS_ADMIN` est particulièrement sensible ;
    
- observer les capabilities d’un processus ;
    
- comprendre le rôle de seccomp ;
    
- comprendre le rôle d’AppArmor et de SELinux ;
    
- identifier les risques des conteneurs privilégiés ;
    
- identifier les montages dangereux ;
    
- proposer des bonnes pratiques de durcissement.
    

---

## 14.1. Pourquoi les namespaces ne suffisent pas

## 14.1.1. Les namespaces isolent des vues

Nous avons vu que les namespaces permettent de modifier ce qu’un processus voit.

Par exemple :

```text
namespace PID     → le processus ne voit pas tous les PID
namespace network → le processus ne voit pas toutes les interfaces réseau
namespace mount   → le processus ne voit pas tous les montages
namespace UTS     → le processus voit un hostname isolé
namespace IPC     → le processus voit des IPC isolées
namespace user    → le processus peut voir des UID/GID mappés
```

Mais voir moins de choses ne signifie pas forcément être parfaitement sécurisé.

---

## 14.1.2. Le noyau reste partagé

Dans un conteneur Linux classique, le noyau est partagé avec l’hôte.

Cela signifie que les processus de l’hôte et les processus du conteneur utilisent le même noyau.

Nous devons donc retenir :

```text
Un conteneur Linux n’est pas une machine virtuelle.
Il ne possède pas son propre noyau.
```

Si un processus dans un conteneur exploite une vulnérabilité du noyau, il peut potentiellement sortir de l’isolation prévue.

C’est une différence fondamentale avec une machine virtuelle, où le noyau invité est séparé du noyau hôte par un hyperviseur.

---

## 14.1.3. L’isolation dépend de la configuration réelle

Nous ne devons jamais dire simplement :

```text
C’est dans un conteneur, donc c’est sécurisé.
```

Nous devons plutôt demander :

```text
Quels namespaces sont isolés ?
Quels namespaces sont partagés avec l’hôte ?
Quelles capabilities sont accordées ?
Quels montages sont exposés ?
Quels cgroups limitent les ressources ?
Quel profil seccomp est actif ?
Quel profil AppArmor ou SELinux est actif ?
Le conteneur tourne-t-il en root ?
Le conteneur est-il privilégié ?
```

La sécurité réelle dépend de l’ensemble de ces réponses.

---

## 14.2. Rappel : les trois grandes dimensions

## 14.2.1. Ce que le processus voit

C’est le rôle principal des namespaces.

```text
Namespaces = isolation des vues
```

Exemples :

- vue des processus ;
    
- vue réseau ;
    
- vue des montages ;
    
- vue des utilisateurs ;
    
- vue des IPC ;
    
- vue du hostname.
    

---

## 14.2.2. Ce que le processus consomme

C’est le rôle principal des cgroups.

```text
Cgroups = contrôle des ressources
```

Exemples :

- mémoire maximale ;
    
- quota CPU ;
    
- nombre de processus ;
    
- I/O ;
    
- accès à certains périphériques ;
    
- pression de ressources.
    

---

## 14.2.3. Ce que le processus a le droit de faire

C’est le rôle de plusieurs mécanismes combinés :

```text
capabilities
seccomp
AppArmor
SELinux
permissions Unix
montages
droits sur les périphériques
configuration du runtime
```

C’est cette troisième dimension qui nous intéresse particulièrement dans ce chapitre.

---

## 14.3. Les capabilities Linux

## 14.3.1. Pourquoi les capabilities existent-elles ?

Historiquement, Linux distinguait principalement :

```text
root
non-root
```

Le compte `root`, UID `0`, avait presque tous les privilèges.

Cette approche est simple, mais trop grossière.

Un programme peut avoir besoin d’un privilège précis, par exemple :

- ouvrir un port inférieur à 1024 ;
    
- changer le propriétaire d’un fichier ;
    
- configurer une interface réseau ;
    
- monter un système de fichiers ;
    
- envoyer un signal à un autre processus ;
    
- modifier certaines limites système.
    

Nous ne voulons pas forcément lui donner tous les droits root pour cela.

Les capabilities découpent donc les privilèges de root en unités plus fines.

---

## 14.3.2. Définition

Une capability est un droit spécifique accordé à un processus.

Nous pouvons résumer :

```text
Avant :
root a tous les privilèges.

Avec les capabilities :
un processus peut recevoir seulement certains privilèges précis.
```

Exemples de capabilities :

|Capability|Rôle simplifié|
|---|---|
|`CAP_CHOWN`|changer le propriétaire d’un fichier|
|`CAP_DAC_OVERRIDE`|contourner certaines permissions fichiers|
|`CAP_NET_BIND_SERVICE`|écouter sur un port inférieur à 1024|
|`CAP_NET_ADMIN`|administrer le réseau|
|`CAP_SYS_ADMIN`|nombreuses opérations système sensibles|
|`CAP_SYS_PTRACE`|tracer ou inspecter d’autres processus|
|`CAP_KILL`|envoyer des signaux à certains processus|
|`CAP_SETUID`|modifier l’UID d’un processus|
|`CAP_SETGID`|modifier le GID d’un processus|
|`CAP_MKNOD`|créer certains fichiers de périphériques|

---

## 14.4. Observer les capabilities d’un processus

## 14.4.1. Avec `/proc/<PID>/status`

Nous pouvons observer les capabilities du shell courant :

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

Elles représentent différents ensembles de capabilities.

---

## 14.4.2. Les principaux ensembles

Nous distinguons notamment :

|Champ|Signification|
|---|---|
|`CapInh`|capabilities héritables|
|`CapPrm`|capabilities permises|
|`CapEff`|capabilities effectives|
|`CapBnd`|capabilities maximales possibles après réduction|
|`CapAmb`|capabilities ambiantes transmissibles à certains exécutables|

Le champ le plus directement important pour l’action courante est souvent :

```text
CapEff
```

car il représente les capabilities effectivement actives.

---

## 14.4.3. Avec `capsh`

L’outil `capsh` rend la sortie plus lisible.

Commande :

```bash
capsh --print
```

Sur Debian/Ubuntu, il peut être fourni par :

```bash
sudo apt install libcap2-bin
```

Sortie typique :

```text
Current: cap_chown,cap_dac_override,cap_fowner,...
Bounding set = cap_chown,cap_dac_override,...
Ambient set =
```

Nous utilisons `capsh` pour comprendre rapidement quels droits possède un processus.

---

## 14.4.4. Décoder les capabilities

Nous pouvons aussi utiliser :

```bash
capsh --decode=<valeur_hexadécimale>
```

Exemple :

```bash
capsh --decode=0000000000000400
```

Cela permet de transformer une valeur hexadécimale de `/proc/<PID>/status` en noms de capabilities.

---

## 14.5. Capabilities et conteneurs

## 14.5.1. Pourquoi les conteneurs ont des capabilities réduites

Un conteneur lancé en root ne reçoit généralement pas toutes les capabilities de root.

Docker, Podman ou Kubernetes réduisent souvent la liste par défaut.

Cela signifie que dans un conteneur :

```bash
id
```

peut afficher :

```text
uid=0(root)
```

mais cela ne signifie pas que le processus possède toutes les capabilities possibles.

Nous devons distinguer :

```text
UID 0
capabilities effectives
namespaces
montages
profils de sécurité
```

---

## 14.5.2. Observer dans Docker

Dans un conteneur Docker :

```bash
docker run --rm -it alpine sh
```

Nous pouvons exécuter :

```sh
grep Cap /proc/$$/status
```

Ou, si `capsh` est disponible dans l’image :

```sh
capsh --print
```

Nous pouvons comparer avec l’hôte.

---

## 14.5.3. Ajouter une capability

Docker permet d’ajouter une capability :

```bash
docker run --cap-add=NET_ADMIN image
```

Cela donne au conteneur davantage de droits réseau.

Exemple : il peut être capable de modifier certaines interfaces ou règles réseau dans son namespace réseau.

Mais si le conteneur partage le namespace réseau de l’hôte, cette capability devient beaucoup plus dangereuse.

---

## 14.5.4. Retirer une capability

Docker permet aussi de retirer des capabilities :

```bash
docker run --cap-drop=ALL image
```

Puis de n’ajouter que ce qui est strictement nécessaire :

```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE image
```

Cette approche est plus sûre.

Nous appliquons le principe du moindre privilège.

---

## 14.6. `CAP_SYS_ADMIN`, la capability dangereuse

## 14.6.1. Pourquoi elle est particulière

`CAP_SYS_ADMIN` est souvent considérée comme une capability très large.

Elle donne accès à de nombreuses opérations système sensibles.

Elle intervient notamment dans plusieurs opérations liées à :

- montages ;
    
- namespaces ;
    
- certains paramètres noyau ;
    
- certains systèmes de fichiers ;
    
- opérations avancées d’administration ;
    
- manipulation de ressources globales selon contexte.
    

Elle est parfois surnommée de manière informelle “le nouveau root”, parce qu’elle couvre énormément d’actions.

---

## 14.6.2. Pourquoi éviter `CAP_SYS_ADMIN`

Si nous donnons `CAP_SYS_ADMIN` à un conteneur, nous augmentons fortement les risques.

Exemple :

```bash
docker run --cap-add=SYS_ADMIN image
```

Cette option peut être nécessaire dans certains cas très spécifiques, mais elle doit toujours être justifiée.

Avant de l’utiliser, nous demandons :

```text
Quelle opération exacte nécessite CAP_SYS_ADMIN ?
Existe-t-il une capability plus précise ?
Pouvons-nous éviter ce besoin ?
Le conteneur peut-il tourner autrement ?
Le montage ou le périphérique exposé est-il vraiment nécessaire ?
```

---

## 14.6.3. Avec un user namespace

Dans un user namespace, `CAP_SYS_ADMIN` peut être limitée au contexte du namespace.

Mais cela ne la rend pas automatiquement inoffensive.

Nous devons toujours nous demander :

```text
Dans quel user namespace cette capability est-elle effective ?
Quelles ressources appartiennent à ce user namespace ?
Quels montages sont exposés ?
Quels autres namespaces sont partagés ?
```

---

## 14.7. Seccomp

## 14.7.1. Qu’est-ce que seccomp ?

Seccomp signifie _secure computing mode_.

C’est un mécanisme du noyau Linux permettant de filtrer les appels système qu’un processus peut effectuer.

Un appel système est une demande faite au noyau.

Exemples :

```text
open
read
write
clone
mount
ptrace
ioctl
execve
chmod
setns
```

Si un processus n’a pas besoin de certains appels système dangereux, nous pouvons les bloquer.

---

## 14.7.2. Pourquoi seccomp est utile

Même si un processus est dans des namespaces, il parle toujours au même noyau que l’hôte.

Limiter les appels système réduit donc la surface d’attaque.

Exemple :

```text
Si l’application n’a pas besoin de ptrace, nous pouvons bloquer ptrace.
Si elle n’a pas besoin de mount, nous pouvons bloquer mount.
Si elle n’a pas besoin de certains appels très spécialisés, nous les bloquons.
```

Seccomp ne remplace pas les namespaces, mais il les complète.

---

## 14.7.3. Seccomp avec Docker

Docker applique généralement un profil seccomp par défaut.

Nous pouvons désactiver seccomp avec :

```bash
docker run --security-opt seccomp=unconfined image
```

Mais c’est une option sensible.

Elle réduit la protection.

Nous l’évitons sauf diagnostic ou besoin très spécifique.

---

## 14.7.4. Profils seccomp personnalisés

Il est possible de fournir un profil seccomp personnalisé.

Conceptuellement :

```bash
docker run --security-opt seccomp=profil.json image
```

Le profil décrit quels appels système sont autorisés ou interdits.

En production, nous pouvons durcir une application en construisant un profil adapté à son comportement réel.

---

## 14.8. AppArmor

## 14.8.1. Rôle d’AppArmor

AppArmor est un mécanisme de contrôle d’accès obligatoire.

Il permet de définir ce qu’un programme a le droit de faire, notamment :

- quels fichiers il peut lire ;
    
- quels fichiers il peut écrire ;
    
- quelles capacités il peut utiliser ;
    
- quels profils il suit.
    

AppArmor fonctionne avec des profils attachés aux programmes ou conteneurs.

---

## 14.8.2. Observer AppArmor

Sur certaines distributions, nous pouvons voir l’état d’AppArmor avec :

```bash
sudo aa-status
```

Ou vérifier le profil d’un processus :

```bash
cat /proc/<PID>/attr/current
```

Exemple possible :

```text
docker-default (enforce)
```

Cela signifie que le processus est confiné par le profil AppArmor `docker-default`.

---

## 14.8.3. Docker et AppArmor

Docker peut utiliser un profil AppArmor par défaut.

Nous pouvons désactiver ce profil avec :

```bash
docker run --security-opt apparmor=unconfined image
```

C’est une option sensible, car elle retire une couche de sécurité.

Nous l’évitons en production sauf besoin très justifié.

---

## 14.9. SELinux

## 14.9.1. Rôle de SELinux

SELinux est un autre mécanisme de contrôle d’accès obligatoire.

Il est très utilisé dans certaines distributions, notamment les familles Red Hat, Fedora, CentOS Stream, Rocky Linux ou AlmaLinux.

SELinux applique des politiques de sécurité basées sur des contextes.

---

## 14.9.2. Observer SELinux

Nous pouvons vérifier l’état :

```bash
getenforce
```

Sorties possibles :

```text
Enforcing
Permissive
Disabled
```

Nous pouvons voir les contextes de fichiers avec :

```bash
ls -Z
```

Et les contextes de processus avec :

```bash
ps -eZ
```

---

## 14.9.3. SELinux et conteneurs

Avec SELinux, les conteneurs peuvent être confinés par des labels spécifiques.

Un conteneur peut se voir interdire l’accès à certains fichiers même si les permissions Unix semblent l’autoriser.

C’est une source fréquente de confusion, mais aussi une couche de sécurité puissante.

Nous devons donc penser à SELinux lorsqu’un accès fichier échoue sans raison apparente sur une distribution qui l’utilise.

---

## 14.10. Conteneurs privilégiés

## 14.10.1. Qu’est-ce qu’un conteneur privilégié ?

Avec Docker :

```bash
docker run --privileged image
```

Avec Kubernetes :

```yaml
securityContext:
  privileged: true
```

Un conteneur privilégié reçoit beaucoup plus de droits que normalement.

Il peut obtenir :

- davantage de capabilities ;
    
- accès plus large à `/dev` ;
    
- restrictions AppArmor/seccomp amoindries selon configuration ;
    
- capacité à effectuer des opérations proches de l’administration hôte ;
    
- accès à des périphériques sensibles.
    

---

## 14.10.2. Pourquoi c’est dangereux

Un conteneur privilégié réduit fortement l’isolation.

Dans certains cas, il peut devenir très proche d’un processus root sur l’hôte.

Nous ne devons donc pas utiliser `--privileged` comme solution rapide à un problème de permission.

Mauvaise démarche :

```text
Ça ne marche pas, donc nous lançons en privileged.
```

Bonne démarche :

```text
Quel droit précis manque ?
Quelle capability manque ?
Quel fichier ou périphérique doit être exposé ?
Pouvons-nous limiter l’accès ?
Pouvons-nous changer l’architecture ?
```

---

## 14.10.3. Cas où cela peut être justifié

Il existe des cas spécifiques :

- agents système ;
    
- outils d’administration de nœud ;
    
- conteneurs de build très particuliers ;
    
- environnements de test isolés ;
    
- outils qui manipulent des périphériques ;
    
- certains composants bas niveau.
    

Mais même dans ces cas, nous devons préférer une permission ciblée à un privilège global.

---

## 14.11. Montages dangereux

## 14.11.1. Pourquoi les montages sont critiques

Le namespace mount donne au conteneur une vue de fichiers.

Mais si nous montons des chemins sensibles de l’hôte, nous créons une passerelle entre l’environnement isolé et l’hôte.

Les montages peuvent donc annuler une partie importante de l’isolation.

---

## 14.11.2. Monter `/`

Montage dangereux :

```bash
docker run -v /:/host image
```

Le conteneur voit toute la racine de l’hôte sous `/host`.

Même si le rootfs du conteneur est isolé, l’hôte est exposé.

Si le conteneur peut écrire, il peut modifier des fichiers de l’hôte.

---

## 14.11.3. Monter `/proc`

Montage dangereux :

```bash
docker run -v /proc:/host/proc image
```

Cela peut exposer :

- processus de l’hôte ;
    
- informations sensibles ;
    
- arguments de ligne de commande ;
    
- variables d’environnement ;
    
- fichiers ouverts ;
    
- informations réseau ;
    
- paramètres noyau.
    

Nous évitons ce montage sauf cas de supervision ou diagnostic très contrôlé.

---

## 14.11.4. Monter `/sys`

Montage dangereux :

```bash
docker run -v /sys:/host/sys image
```

`/sys` expose des objets noyau, des périphériques, des paramètres matériels et des informations sensibles.

Un accès en écriture à `/sys` peut être particulièrement dangereux.

---

## 14.11.5. Monter `/dev`

Montage dangereux :

```bash
docker run -v /dev:/host/dev image
```

Ou donner trop de périphériques.

Certains fichiers de `/dev` permettent d’interagir avec :

- disques ;
    
- GPU ;
    
- terminaux ;
    
- interfaces bas niveau ;
    
- KVM ;
    
- périphériques USB ;
    
- périphériques système.
    

Nous préférons exposer seulement les périphériques nécessaires.

---

## 14.11.6. Monter le socket Docker

Montage très sensible :

```bash
docker run -v /var/run/docker.sock:/var/run/docker.sock image
```

Un processus dans le conteneur peut alors parler au démon Docker de l’hôte.

Dans beaucoup de configurations, cela revient à donner un contrôle très important sur la machine.

Pourquoi ?

Parce qu’un processus qui contrôle Docker peut souvent lancer un autre conteneur avec des montages sensibles, voire avec `--privileged`.

Nous considérons donc ce montage comme quasi équivalent à un accès administratif sur l’hôte.

---

## 14.12. Namespaces partagés avec l’hôte

## 14.12.1. `--pid=host`

Docker :

```bash
docker run --pid=host image
```

Kubernetes :

```yaml
hostPID: true
```

Le conteneur partage le namespace PID de l’hôte.

Conséquences :

- il voit les processus de l’hôte ;
    
- il peut potentiellement interagir avec eux selon ses droits ;
    
- il réduit l’isolation ;
    
- il expose plus d’informations.
    

---

## 14.12.2. `--network=host`

Docker :

```bash
docker run --network=host image
```

Kubernetes :

```yaml
hostNetwork: true
```

Le conteneur partage la pile réseau de l’hôte.

Conséquences :

- il voit les interfaces de l’hôte ;
    
- il partage les ports de l’hôte ;
    
- il peut écouter directement sur les adresses de l’hôte ;
    
- les conflits de ports sont possibles ;
    
- l’isolation réseau est réduite.
    

---

## 14.12.3. `--ipc=host`

Docker :

```bash
docker run --ipc=host image
```

Kubernetes :

```yaml
hostIPC: true
```

Le conteneur partage les ressources IPC de l’hôte.

Conséquences :

- il peut voir certaines IPC de l’hôte ;
    
- il peut interagir avec des ressources de mémoire partagée ou sémaphores selon permissions ;
    
- il réduit l’isolation.
    

---

## 14.12.4. Bilan

Ces options peuvent être nécessaires pour certains agents système, mais elles doivent être considérées comme sensibles.

Nous posons toujours la question :

```text
Avons-nous vraiment besoin de partager ce namespace avec l’hôte ?
Existe-t-il une alternative plus ciblée ?
```

---

## 14.13. User namespace et sécurité

## 14.13.1. Root dans le conteneur, non-root sur l’hôte

Le user namespace permet de mapper l’UID `0` interne vers un UID non privilégié sur l’hôte.

Exemple :

```text
Dans le conteneur :
UID 0

Sur l’hôte :
UID 100000
```

Cela réduit l’impact potentiel d’un conteneur compromis.

---

## 14.13.2. Rootless containers

Les conteneurs rootless utilisent cette logique.

Ils permettent à un utilisateur non-root de lancer des conteneurs.

Avantages :

- pas de démon root obligatoire ;
    
- réduction de l’impact d’une compromission ;
    
- meilleure isolation des droits hôte ;
    
- utile sur postes développeurs et certains serveurs.
    

Limites :

- réseau parfois plus complexe ;
    
- performances ou fonctionnalités différentes ;
    
- problèmes de permissions avec bind mounts ;
    
- dépendance à `/etc/subuid` et `/etc/subgid`;
    
- support variable selon environnements.
    

---

## 14.13.3. Limite de sécurité

Le user namespace réduit certains risques, mais ne les supprime pas.

Il reste des risques liés :

- au noyau partagé ;
    
- aux montages ;
    
- aux capabilities dans le namespace ;
    
- aux erreurs de configuration ;
    
- aux vulnérabilités du runtime ;
    
- aux fichiers exposés depuis l’hôte.
    

Nous devons l’utiliser comme une couche, pas comme une solution unique.

---

## 14.14. Seccomp, capabilities et namespaces : exemple combiné

## 14.14.1. Cas d’un serveur web

Nous avons un serveur web conteneurisé qui écoute sur le port 8080.

Il n’a pas besoin de :

- modifier les interfaces réseau ;
    
- monter des systèmes de fichiers ;
    
- tracer d’autres processus ;
    
- créer des périphériques ;
    
- changer l’heure système ;
    
- charger des modules noyau.
    

Nous pouvons donc réduire fortement ses privilèges.

---

## 14.14.2. Exemple Docker durci

Exemple indicatif :

```bash
docker run --rm \
  --read-only \
  --cap-drop=ALL \
  --security-opt no-new-privileges:true \
  --pids-limit=256 \
  --memory=512m \
  --cpus=1 \
  -p 8080:8080 \
  image-web
```

Ici, nous combinons :

- root filesystem en lecture seule ;
    
- suppression des capabilities ;
    
- interdiction d’acquérir de nouveaux privilèges ;
    
- limite de processus ;
    
- limite mémoire ;
    
- limite CPU ;
    
- exposition explicite du port.
    

Selon l’application, il faut ajouter des volumes temporaires pour `/tmp` ou `/run`.

---

## 14.14.3. Exemple Kubernetes durci

Exemple indicatif :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-secure
spec:
  containers:
    - name: web
      image: image-web
      ports:
        - containerPort: 8080
      securityContext:
        runAsNonRoot: true
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
        seccompProfile:
          type: RuntimeDefault
      resources:
        requests:
          memory: "256Mi"
          cpu: "250m"
        limits:
          memory: "512Mi"
          cpu: "1"
```

Ce n’est pas une recette universelle, mais une bonne base de réflexion.

---

## 14.15. `no-new-privileges`

## 14.15.1. Principe

`no-new-privileges` empêche un processus et ses enfants d’obtenir de nouveaux privilèges via certains mécanismes, par exemple des exécutables setuid.

Avec Docker :

```bash
docker run --security-opt no-new-privileges:true image
```

Dans Kubernetes :

```yaml
securityContext:
  allowPrivilegeEscalation: false
```

Ce mécanisme réduit les possibilités d’élévation de privilèges.

---

## 14.15.2. Pourquoi c’est utile

Même si un fichier setuid est présent dans l’image, le processus ne doit pas pouvoir s’en servir pour acquérir davantage de droits.

C’est une protection simple et efficace pour beaucoup de workloads applicatifs.

---

## 14.16. Périphériques et `/dev`

## 14.16.1. Pourquoi les périphériques sont sensibles

Les fichiers dans `/dev` ne sont pas des fichiers ordinaires.

Ils donnent accès à des périphériques ou interfaces noyau.

Exemples :

```text
/dev/null
/dev/random
/dev/tty
/dev/sda
/dev/kvm
/dev/dri
/dev/fuse
```

Certains sont inoffensifs ou nécessaires. D’autres sont très sensibles.

---

## 14.16.2. Exposer un périphérique précis

Docker permet :

```bash
docker run --device=/dev/dri image
```

Kubernetes permet des mécanismes autour des device plugins, notamment pour les GPU.

Nous préférons exposer un périphérique précis plutôt que tout `/dev`.

---

## 14.16.3. Risque de `/dev/kvm`

Donner accès à `/dev/kvm` permet d’utiliser l’accélération de virtualisation.

Cela peut être nécessaire pour certains cas, mais c’est sensible.

Nous devons comprendre l’impact avant de l’exposer.

---

## 14.17. Sécurité de `/proc`

## 14.17.1. Informations visibles

Nous avons déjà vu que `/proc` peut exposer :

```text
/proc/<PID>/cmdline
/proc/<PID>/environ
/proc/<PID>/fd
/proc/<PID>/maps
/proc/<PID>/mountinfo
/proc/net/*
```

Ces fichiers peuvent révéler des secrets ou des détails d’architecture.

---

## 14.17.2. Secrets en ligne de commande

Mauvaise pratique :

```bash
app --password supersecret
```

Le secret peut apparaître dans :

```bash
ps aux
tr '\0' ' ' < /proc/<PID>/cmdline
```

Nous évitons les secrets en arguments.

---

## 14.17.3. Secrets en variables d’environnement

Les variables d’environnement peuvent être visibles dans :

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

selon les permissions et le contexte.

Les secrets en variables d’environnement sont pratiques, mais pas parfaitement invisibles.

Nous devons limiter les accès à `/proc`, utiliser des mécanismes de secrets adaptés et éviter de trop exposer les processus.

---

## 14.17.4. Option `hidepid`

Sur un serveur multi-utilisateurs, nous pouvons monter `/proc` avec :

```text
hidepid=2
```

Exemple dans `/etc/fstab` :

```fstab
proc /proc proc defaults,nosuid,nodev,noexec,hidepid=2 0 0
```

Cela réduit la visibilité des processus appartenant à d’autres utilisateurs.

---

## 14.18. Bonnes pratiques de durcissement

## 14.18.1. Principe du moindre privilège

Nous donnons au processus uniquement ce dont il a besoin.

Nous évitons :

```text
--privileged
--network=host
--pid=host
--ipc=host
--cap-add=SYS_ADMIN
montage de /
montage de /proc
montage de /sys
montage du socket Docker
```

sauf justification forte.

---

## 14.18.2. Réduire les capabilities

Bonne approche :

```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE image
```

si l’application a seulement besoin d’écouter sur un port bas.

En Kubernetes :

```yaml
securityContext:
  capabilities:
    drop:
      - ALL
    add:
      - NET_BIND_SERVICE
```

Nous ajoutons seulement les capabilities nécessaires.

---

## 14.18.3. Utiliser un utilisateur non-root

Dockerfile :

```dockerfile
USER 10001
```

Kubernetes :

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 10001
```

Même si les namespaces isolent l’environnement, exécuter l’application en non-root réduit les risques.

---

## 14.18.4. Root filesystem en lecture seule

Docker :

```bash
docker run --read-only image
```

Kubernetes :

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

Nous ajoutons ensuite uniquement les chemins nécessaires en écriture, par exemple :

```text
/tmp
/run
/var/cache/app
```

avec des volumes adaptés.

---

## 14.18.5. Activer seccomp par défaut

En Kubernetes :

```yaml
securityContext:
  seccompProfile:
    type: RuntimeDefault
```

Nous évitons :

```yaml
seccompProfile:
  type: Unconfined
```

sauf besoin très spécifique.

---

## 14.18.6. Limiter les ressources

Nous définissons des limites raisonnables :

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "1"
```

Nous utilisons aussi une limite de processus lorsque c’est possible.

---

## 14.18.7. Éviter les montages sensibles

Nous évitons de monter :

```text
/
/proc
/sys
/dev
/var/run/docker.sock
/var/lib/docker
/var/lib/kubelet
/etc
/root
```

Si un montage est nécessaire, nous préférons :

- chemin précis ;
    
- lecture seule ;
    
- permissions minimales ;
    
- utilisateur non-root ;
    
- politique AppArmor/SELinux adaptée.
    

---

## 14.19. Étude de cas : conteneur trop permissif

## 14.19.1. Configuration dangereuse

Exemple :

```bash
docker run --rm -it \
  --privileged \
  --network=host \
  --pid=host \
  -v /:/host \
  image
```

Cette configuration cumule plusieurs risques.

Le conteneur :

- est privilégié ;
    
- partage le réseau de l’hôte ;
    
- partage les PID de l’hôte ;
    
- voit toute la racine de l’hôte sous `/host`.
    

Ce n’est presque plus une isolation applicative.

---

## 14.19.2. Analyse

Nous identifions les problèmes :

|Élément|Risque|
|---|---|
|`--privileged`|donne trop de droits|
|`--network=host`|supprime l’isolation réseau|
|`--pid=host`|expose les processus de l’hôte|
|`-v /:/host`|expose tout le système de fichiers|
|absence de limites|risque de consommation excessive|
|root par défaut|impact plus important en cas de compromission|

---

## 14.19.3. Version plus raisonnable

Selon le besoin réel, nous cherchons une version réduite :

```bash
docker run --rm \
  --cap-drop=ALL \
  --security-opt no-new-privileges:true \
  --read-only \
  --memory=512m \
  --cpus=1 \
  -v /chemin/necessaire:/data:ro \
  image
```

Nous ne montons que le chemin nécessaire, en lecture seule.

Nous n’utilisons pas `--privileged`.

Nous ne partageons pas les namespaces de l’hôte sauf besoin démontré.

---

## 14.20. Exercices

## Exercice 1 — Observer les capabilities

Nous exécutons :

```bash
grep Cap /proc/$$/status
```

Puis, si disponible :

```bash
capsh --print
```

Nous répondons :

1. Quelles capabilities effectives voyons-nous ?
    
2. Quelle différence faisons-nous entre `CapPrm` et `CapEff` ?
    
3. Pourquoi les capabilities sont-elles plus fines que le simple couple root/non-root ?
    

---

## Exercice 2 — Comparer root hôte et root conteneur

Si Docker est disponible :

```bash
docker run --rm alpine sh -c 'id; grep Cap /proc/$$/status'
```

Nous répondons :

1. Le processus est-il UID 0 dans le conteneur ?
    
2. A-t-il forcément toutes les capabilities de root ?
    
3. Pourquoi devons-nous distinguer UID et capabilities ?
    

---

## Exercice 3 — Ajouter et retirer des capabilities

Nous comparons :

```bash
docker run --rm alpine sh -c 'grep Cap /proc/$$/status'
docker run --rm --cap-drop=ALL alpine sh -c 'grep Cap /proc/$$/status'
docker run --rm --cap-drop=ALL --cap-add=NET_BIND_SERVICE alpine sh -c 'grep Cap /proc/$$/status'
```

Nous répondons :

1. Que change `--cap-drop=ALL` ?
    
2. Que change `--cap-add=NET_BIND_SERVICE` ?
    
3. Pourquoi cette approche est-elle préférable à `--privileged` ?
    

---

## Exercice 4 — Observer AppArmor ou SELinux

Sur une distribution avec AppArmor :

```bash
cat /proc/$$/attr/current
sudo aa-status
```

Sur une distribution avec SELinux :

```bash
getenforce
ps -eZ | head
ls -Z | head
```

Nous répondons :

1. Quel mécanisme est actif ?
    
2. Le processus courant est-il confiné ?
    
3. Pourquoi ces mécanismes complètent-ils les namespaces ?
    

---

## Exercice 5 — Analyser une configuration Docker dangereuse

Nous analysons :

```bash
docker run --privileged --network=host --pid=host -v /:/host image
```

Nous répondons :

1. Quels namespaces sont partagés avec l’hôte ?
    
2. Quels montages sont dangereux ?
    
3. Pourquoi `--privileged` augmente-t-il le risque ?
    
4. Comment proposer une configuration plus sûre ?
    

---

## Exercice 6 — Sécurité de `/proc`

Nous lançons un processus avec un argument fictif :

```bash
sleep 300 --fake-secret 2>/dev/null
```

Cette commande n’est pas forcément acceptée par `sleep` selon les systèmes. Nous pouvons plutôt lancer :

```bash
bash -c 'sleep 300' secret-fictif &
```

Puis :

```bash
ps aux | grep secret
tr '\0' ' ' < /proc/<PID>/cmdline
```

Nous répondons :

1. Les arguments sont-ils visibles ?
    
2. Pourquoi ne devons-nous pas passer de secrets en ligne de commande ?
    
3. Quels autres fichiers de `/proc` peuvent révéler des informations ?
    

---

## Exercice 7 — Kubernetes securityContext

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

1. Quel est le rôle de `runAsNonRoot` ?
    
2. Quel est le rôle de `allowPrivilegeEscalation: false` ?
    
3. Quel est le rôle de `readOnlyRootFilesystem` ?
    
4. Pourquoi supprimons-nous toutes les capabilities ?
    
5. Que fait `RuntimeDefault` pour seccomp ?
    

---

## 14.21. Ce que nous devons retenir

Nous retenons les points suivants :

1. Les namespaces isolent des vues, mais ne suffisent pas à sécuriser un système.
    
2. Les conteneurs partagent le noyau de l’hôte.
    
3. Les capabilities découpent les privilèges root en droits plus fins.
    
4. Être UID 0 ne signifie pas posséder toutes les capabilities.
    
5. `CAP_SYS_ADMIN` est particulièrement sensible.
    
6. `--privileged` réduit fortement l’isolation.
    
7. Seccomp limite les appels système disponibles.
    
8. AppArmor et SELinux ajoutent des politiques de contrôle d’accès obligatoire.
    
9. Les montages de `/`, `/proc`, `/sys`, `/dev` et du socket Docker sont très sensibles.
    
10. Partager `pid`, `network` ou `ipc` avec l’hôte réduit fortement l’isolation.
    
11. Le user namespace permet de réduire l’impact de root dans le conteneur.
    
12. Les conteneurs rootless améliorent la sécurité, mais ne suppriment pas tous les risques.
    
13. Le principe du moindre privilège doit guider la configuration.
    
14. Nous préférons supprimer toutes les capabilities puis ajouter seulement celles nécessaires.
    
15. Nous évitons `--privileged` sauf cas exceptionnel.
    
16. Nous utilisons des limites cgroups pour réduire l’impact opérationnel.
    
17. Nous devons penser ensemble : namespaces, cgroups, capabilities, seccomp, AppArmor/SELinux, montages et utilisateur d’exécution.
    

---

## Conclusion du chapitre 14

Nous avons étudié la sécurité autour des namespaces.

Nous savons maintenant que les namespaces et les cgroups sont nécessaires, mais pas suffisants. Les namespaces isolent les vues, les cgroups limitent les ressources, mais les droits réels d’un processus dépendent aussi des capabilities, de seccomp, d’AppArmor, de SELinux, des montages, des périphériques exposés et du mode d’exécution.

Nous avons compris pourquoi un conteneur privilégié, un montage de `/`, un accès au socket Docker, `--network=host`, `--pid=host` ou `CAP_SYS_ADMIN` peuvent réduire fortement l’isolation.

Nous retenons surtout une méthode professionnelle : nous ne donnons jamais des privilèges “pour que ça marche”. Nous identifions le besoin exact, puis nous accordons le minimum nécessaire.

Dans le chapitre suivant, nous étudions comment Docker et les runtimes de conteneurs assemblent concrètement namespaces, cgroups, capabilities, montages et processus pour construire un conteneur.

---
> [!info] Livre « Les namespaces Linux » — chapitre 14/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 13 — Namespaces et cgroups|← 13 — Namespaces et cgroups]] · [[Les namespaces Linux — 15 — Des namespaces aux conteneurs|15 — Des namespaces aux conteneurs →]]
