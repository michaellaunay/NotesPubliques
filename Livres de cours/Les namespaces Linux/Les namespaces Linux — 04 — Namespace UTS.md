---
schema_version: 1
uid: 01M1BQ624A89YFWA4EW3Q01EAA
titre: "Les namespaces Linux — 04 — Namespace UTS"
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
resume: "Chapitre 4 sur 16 du livre « Les namespaces Linux » : Namespace UTS. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 4/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 03 — Les outils de manipulation des namespaces|← 03 — Les outils de manipulation des namespaces]] · [[Les namespaces Linux — 05 — Namespace PID|05 — Namespace PID →]]

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

## 4.1. Qu’est-ce que le namespace UTS ?

## 4.1.1. Définition

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

## 4.1.2. Origine du nom UTS

UTS vient historiquement de “UNIX Time-Sharing System”.

Aujourd’hui, dans le contexte des namespaces Linux, nous retenons surtout que le namespace UTS isole le nom de la machine.

Il ne faut pas le confondre avec le namespace time, qui concerne certaines horloges.

Le namespace UTS ne sert pas à gérer le temps. Il sert à isoler l’identité système visible.

---

## 4.2. Le hostname

## 4.2.1. Lire le hostname

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

## 4.2.2. Modifier le hostname

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

## 4.2.3. Hostname actif et configuration persistante

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

## 4.3. Le domainname

## 4.3.1. Lire le domainname

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

## 4.3.2. À ne pas confondre avec DNS

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

## 4.4. Observer le namespace UTS d’un processus

## 4.4.1. Avec `/proc/<PID>/ns/uts`

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
uts:
```

Pour le PID 1 :

```bash
readlink /proc/1/ns/uts
```

Si les deux valeurs sont identiques, notre shell et le PID 1 partagent le même namespace UTS.

---

## 4.4.2. Comparer deux processus

Nous pouvons comparer :

```bash
echo "Shell courant : $(readlink /proc/$$/ns/uts)"
echo "PID 1         : $(readlink /proc/1/ns/uts)"
```

Exemple :

```text
Shell courant : uts:
PID 1         : uts:
```

Cela signifie que les deux processus partagent le même namespace UTS.

Si nous voyons :

```text
Shell courant : uts:
PID 1         : uts:
```

cela signifie que le shell courant est dans un namespace UTS différent de celui du PID 1.

---

## 4.5. Créer un namespace UTS avec `unshare`

## 4.5.1. Expérience simple

Nous commençons par afficher le hostname de l’hôte :

```bash
hostname
readlink /proc/$$/ns/uts
```

Exemple :

```text
machine-hote
uts:
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

## 4.5.2. Modifier le hostname dans le namespace

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

## 4.5.3. Pourquoi cela fonctionne-t-il ?

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

## 4.6. Permissions nécessaires

## 4.6.1. Création d’un namespace UTS

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

## 4.6.2. Modifier le hostname

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

## 4.7. Lien avec Docker et Podman

## 4.7.1. Hostname dans un conteneur Docker

Quand nous lançons un conteneur :

```bash
docker run --rm alpine hostname
```

Docker donne souvent au conteneur un hostname dérivé de son identifiant.

Exemple :

```text
b7f3c2a91d4e
```

Ce hostname n’est pas celui de l’hôte.

Cela s’explique par l’utilisation d’un namespace UTS isolé.

---

## 4.7.2. Choisir le hostname d’un conteneur

Avec Docker :

```bash
docker run --rm --hostname mon-conteneur alpine hostname
```

Sortie :

```text
mon-conteneur
```

Nous pouvons aussi entrer dans un shell :

```bash
docker run --rm -it --hostname mon-conteneur alpine sh
```

Puis :

```sh
hostname
cat /proc/sys/kernel/hostname
```

Nous voyons le hostname du conteneur.

---

## 4.7.3. Partager le namespace UTS de l’hôte

Docker permet de ne pas isoler le namespace UTS avec certaines options avancées, selon le runtime et la configuration.

Le principe à retenir est le suivant :

```text
Si le conteneur partage le namespace UTS de l’hôte,
changer le hostname dans le conteneur peut affecter l’hôte.
```

C’est une configuration à éviter sauf besoin très particulier.

En pratique, les conteneurs standards ont leur propre namespace UTS.

---

## 4.7.4. Podman rootless

Podman peut créer des conteneurs rootless.

Dans ce cas, l’isolation UTS fonctionne en combinaison avec un user namespace.

Nous retrouvons les mêmes principes :

- le conteneur a un hostname isolé ;
    
- l’utilisateur peut être root dans le conteneur ;
    
- les privilèges réels sur l’hôte restent limités.
    

---

## 4.8. Lien avec Kubernetes

## 4.8.1. Hostname dans un pod

Dans Kubernetes, chaque pod dispose généralement de son propre hostname.

Dans un conteneur d’un pod, nous pouvons faire :

```sh
hostname
```

Le résultat est souvent lié au nom du pod.

Kubernetes configure cette identité au lancement du pod.

---

## 4.8.2. Containers d’un même pod

Dans Kubernetes, plusieurs containers d’un même pod partagent certains namespaces, notamment souvent le namespace réseau.

Pour le namespace UTS, la gestion dépend du runtime et de la configuration du pod, mais du point de vue applicatif, les containers d’un même pod partagent généralement une identité cohérente liée au pod.

Nous retenons surtout que Kubernetes ne crée pas une machine virtuelle par pod. Il organise des processus avec des namespaces, des cgroups et d’autres mécanismes du noyau.

---

## 4.8.3. Hostname, DNS et service discovery

Dans Kubernetes, le hostname seul ne suffit pas à comprendre la découverte de services.

Nous devons distinguer :

- hostname du pod ;
    
- nom du pod ;
    
- nom du service Kubernetes ;
    
- DNS interne du cluster ;
    
- namespace Kubernetes ;
    
- domaine cluster, souvent `cluster.local`.
    

Le namespace UTS isole le hostname, mais Kubernetes ajoute une couche complète de DNS et d’orchestration.

---

## 4.9. Lien avec `/proc`

## 4.9.1. Lire le hostname via `/proc`

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

## 4.9.2. Lire le namespace UTS via `/proc/<PID>/ns`

Nous pouvons voir le namespace UTS courant :

```bash
readlink /proc/$$/ns/uts
```

Dans deux terminaux différents, nous pouvons comparer les valeurs.

Dans un namespace UTS créé avec `unshare`, l’identifiant change.

---

## 4.9.3. `/proc/sys/kernel/hostname` dépend du namespace

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

## 4.10. Expérience complète guidée

## 4.10.1. Terminal 1 : créer un namespace UTS

Nous lançons :

```bash
echo "Avant :"
hostname
readlink /proc/$$/ns/uts

unshare --uts bash
```

Dans le shell isolé :

```bash
echo "Dans le namespace :"
hostname
readlink /proc/$$/ns/uts

hostname ns-formation
hostname
cat /proc/sys/kernel/hostname

echo $$
sleep 1000
```

Nous gardons ce shell ouvert.

---

## 4.10.2. Terminal 2 : observer depuis l’hôte

Dans un second terminal, nous cherchons le processus :

```bash
ps aux | grep sleep
```

Ou bien nous utilisons le PID affiché dans le terminal 1.

Nous comparons :

```bash
readlink /proc/<PID>/ns/uts
readlink /proc/1/ns/uts
```

Nous pouvons aussi entrer dans le namespace UTS :

```bash
sudo nsenter --target <PID> --uts bash
hostname
```

Nous devons voir :

```text
ns-formation
```

Nous quittons le shell `nsenter` :

```bash
exit
```

---

## 4.10.3. Nettoyage

Dans le terminal 1, nous arrêtons :

```bash
exit
```

ou nous tuons le processus `sleep` si nécessaire :

```bash
kill <PID>
```

Quand le dernier processus du namespace disparaît, le namespace UTS disparaît également, sauf s’il est maintenu par une autre référence.

---

## 4.11. Limites du namespace UTS

## 4.11.1. Il n’isole pas le réseau

Changer le hostname ne crée pas une nouvelle pile réseau.

Pour isoler les interfaces, les routes, les ports et les sockets, nous avons besoin du namespace réseau.

Le namespace UTS ne fait que modifier l’identité système visible.

---

## 4.11.2. Il n’isole pas le système de fichiers

Le namespace UTS ne change pas l’arborescence de fichiers.

Même si le hostname est différent, les processus peuvent encore voir les mêmes fichiers si le namespace mount n’est pas isolé.

Pour isoler les montages, nous avons besoin du namespace mount.

---

## 4.11.3. Il n’isole pas les processus

Le namespace UTS ne modifie pas la table des processus visible.

Pour isoler les PID, nous avons besoin du namespace PID.

Exemple :

```bash
unshare --uts bash
ps aux
```

Nous voyons encore les processus du même namespace PID que le parent.

---

## 4.11.4. Il ne limite aucune ressource

Le namespace UTS ne limite pas :

- CPU ;
    
- mémoire ;
    
- nombre de processus ;
    
- I/O disque ;
    
- réseau.
    

Pour limiter les ressources, nous avons besoin des cgroups.

---

## 4.12. Cas d’usage du namespace UTS

## 4.12.1. Donner une identité à un conteneur

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

## 4.12.2. Tests et environnements temporaires

Nous pouvons tester des applications qui lisent le hostname sans modifier l’hôte.

Exemple :

```bash
unshare --uts bash
hostname test-env
./application
```

L’application voit `test-env`, mais l’hôte n’est pas modifié.

---

## 4.12.3. Formation et démonstration

Le namespace UTS est idéal pour introduire les namespaces, car :

- il est simple ;
    
- il produit un effet visible immédiatement ;
    
- il ne demande pas de configuration réseau ;
    
- il illustre bien la notion de vue isolée.
    

---

## 4.13. Pièges classiques

## 4.13.1. Croire que le hostname DNS change automatiquement

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

## 4.13.2. Croire que le conteneur a son propre noyau

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

## 4.13.3. Oublier les permissions

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

## 4.14. Exercices

## Exercice 1 — Observer le namespace UTS courant

Nous exécutons :

```bash
hostname
cat /proc/sys/kernel/hostname
readlink /proc/$$/ns/uts
readlink /proc/1/ns/uts
```

Nous répondons :

1. Quel est le hostname courant ?
    
2. Le contenu de `/proc/sys/kernel/hostname` est-il identique ?
    
3. Le shell courant partage-t-il le namespace UTS avec le PID 1 ?
    
4. Que signifie une valeur identique ?
    

---

## Exercice 2 — Créer un namespace UTS

Nous exécutons :

```bash
hostname
readlink /proc/$$/ns/uts

unshare --uts bash
readlink /proc/$$/ns/uts
hostname ns-test
hostname
cat /proc/sys/kernel/hostname
exit

hostname
readlink /proc/$$/ns/uts
```

Nous répondons :

1. L’identifiant du namespace UTS change-t-il dans le shell isolé ?
    
2. Le hostname change-t-il dans le shell isolé ?
    
3. Le hostname de l’hôte change-t-il après `exit` ?
    
4. Pourquoi ?
    

---

## Exercice 3 — Comparer deux processus

Dans un terminal :

```bash
unshare --uts bash
hostname ns-comparaison
echo $$
sleep 1000
```

Dans un second terminal :

```bash
readlink /proc/<PID>/ns/uts
readlink /proc/1/ns/uts
```

Nous répondons :

1. Les identifiants sont-ils identiques ?
    
2. Que signifie la différence ?
    
3. Le processus isolé partage-t-il encore les autres namespaces ?
    
4. Comment pouvons-nous vérifier les autres namespaces ?
    

---

## Exercice 4 — Entrer dans un namespace UTS existant

Avec le PID du processus précédent :

```bash
sudo nsenter --target <PID> --uts bash
hostname
readlink /proc/$$/ns/uts
exit
```

Nous répondons :

1. Quel hostname voyons-nous ?
    
2. Avons-nous créé un nouveau namespace ou rejoint un namespace existant ?
    
3. Quel outil avons-nous utilisé ?
    
4. Quelle option de `nsenter` est nécessaire ?
    

---

## Exercice 5 — Comparaison avec Docker

Si Docker est disponible :

```bash
docker run --rm alpine hostname
docker run --rm --hostname cours-uts alpine hostname
```

Nous répondons :

1. Quel hostname Docker donne-t-il par défaut ?
    
2. Que change l’option `--hostname` ?
    
3. Quel namespace explique ce comportement ?
    
4. Le noyau est-il différent de celui de l’hôte ?
    

---

## Exercice 6 — Vérifier que UTS n’isole pas les processus

Nous exécutons :

```bash
unshare --uts bash
hostname ns-mais-pas-pid
ps aux | head
exit
```

Nous répondons :

1. Le hostname est-il isolé ?
    
2. La liste des processus est-elle isolée ?
    
3. Quel namespace faudrait-il ajouter pour isoler les PID ?
    
4. Quelle commande utiliserions-nous ?
    

---

## 4.15. Ce que nous devons retenir

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

## Conclusion du chapitre 4

Nous avons étudié le namespace UTS, l’un des namespaces les plus simples et les plus pédagogiques.

Nous savons maintenant qu’il permet d’isoler le hostname et le domainname d’un groupe de processus. Nous avons vu comment créer un namespace UTS avec `unshare`, comment observer son identifiant avec `/proc/<PID>/ns/uts`, comment modifier le hostname dans une vue isolée, et comment entrer dans ce namespace avec `nsenter`.

Nous retenons surtout que le namespace UTS ne crée pas une machine virtuelle. Il ne change ni le noyau, ni la pile réseau, ni les processus visibles, ni les ressources disponibles. Il isole seulement une partie de l’identité système.

Dans le chapitre suivant, nous étudions le namespace PID, beaucoup plus central pour les conteneurs, car il permet à un processus d’être PID 1 dans son propre environnement.

---
> [!info] Livre « Les namespaces Linux » — chapitre 4/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 03 — Les outils de manipulation des namespaces|← 03 — Les outils de manipulation des namespaces]] · [[Les namespaces Linux — 05 — Namespace PID|05 — Namespace PID →]]
