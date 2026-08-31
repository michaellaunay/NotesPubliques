---
schema_version: 1
uid: 01M1BQ624S2ZJZHEQP52JPX1Z7
titre: "Les namespaces Linux — Compléments 2026"
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
resume: "Compléments apportés au livre « Les namespaces Linux » : sections de la version condensée du cours [[Les namespaces Linux]] (31 août 2026) dont le sujet est absent de la version longue."
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

> [!info] Livre « Les namespaces Linux »
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 16 — Namespaces dans Kubernetes|← 16 — Namespaces dans Kubernetes]] · [[Les namespaces Linux — Sommaire|Sommaire →]]

# Compléments 2026

> [!info] Origine
> Les sections ci-dessous proviennent de la version condensée et actualisée du cours [[Les namespaces Linux]] (31 août 2026). Elles traitent de sujets absents de la version longue et n'ont pas été fondues dans les chapitres ; pour les versions logicielles et l'état de l'art du moment, la version condensée fait foi.

## 13.3. pidfds

Un numéro de PID peut être réutilisé après la mort d'un processus. Util-linux 2.42 ajoute aussi à `nsenter` une convention `PID:INO`, où l'inode permet de préciser l'instance de processus visée.

Un **pidfd** fournit une référence FD à un processus précis.

Des APIs modernes peuvent l'utiliser pour :

- signaler un processus de manière moins sujette aux races ;
- attendre sa terminaison ;
- rejoindre plusieurs de ses namespaces via `setns()` selon les flags et permissions.

Ce modèle FD-based est une tendance importante des API Linux modernes.

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
