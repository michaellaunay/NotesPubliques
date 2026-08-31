---
schema_version: 1
uid: 01M1BQ629Z7A0TDK94V0Q96A3N
titre: "proc — Compléments 2026"
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
  - procfs
resume: "Compléments apportés au livre « proc » : sections de la version condensée du cours [[proc]] (31 août 2026) dont le sujet est absent de la version longue."
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

> [!info] Livre « proc »
> [[proc — Sommaire|Sommaire]] · [[proc — 11 — Limites de proc|← 11 — Limites de proc]] · [[proc — Conclusion, travaux pratiques et annexes|Conclusion →]]

# Compléments 2026

> [!info] Origine
> Les sections ci-dessous proviennent de la version condensée et actualisée du cours [[proc]] (31 août 2026). Elles traitent de sujets absents de la version longue et n'ont pas été fondues dans les chapitres ; pour les versions logicielles et l'état de l'art du moment, la version condensée fait foi.

## Glossaire

**procfs**
Pseudo-système de fichiers Linux monté généralement sur `/proc`.

**VFS**
Couche d'abstraction du noyau fournissant une interface commune aux systèmes de fichiers.

**PID**
Identifiant numérique d'un processus dans un PID namespace donné.

**TGID**
Identifiant du groupe de threads ; correspond généralement au PID présenté comme processus.

**TID**
Identifiant d'une tâche/thread.

**pidfd**
Descripteur représentant un processus et évitant certaines races liées à la réutilisation des PID.

**RSS**
Resident Set Size : pages actuellement résidentes attribuées à un processus, avec partage potentiellement compté plusieurs fois.

**PSS**
Proportional Set Size : mémoire partagée répartie proportionnellement entre processus.

**PSI**
Pressure Stall Information : métriques de temps perdu à attendre CPU, mémoire ou I/O.

**sysctl**
Interface de lecture/modification de paramètres du noyau, dont beaucoup sont exposés via `/proc/sys`.

**namespace**
Mécanisme d'isolation d'une vue de certaines ressources noyau.

**cgroup**
Mécanisme de contrôle et de comptabilité des ressources.

**capability**
Découpage fin des privilèges traditionnellement associés à root.

**seccomp**
Mécanisme de filtrage des appels système.

**ASLR**
Randomisation de l'espace d'adressage.

**mountinfo**
Interface procfs riche décrivant les montages vus par un processus.

---

## Aide-mémoire opérationnel

```bash
### Montage procfs
findmnt -T /proc

### Processus courant
cat /proc/self/status
tr '\0' '\n' < /proc/self/cmdline

### PID namespaces
cat /proc/self/status | grep '^NSpid:'

### Namespaces
ls -l /proc/self/ns

### Cgroup
cat /proc/self/cgroup

### FDs
ls -l /proc/self/fd

### Mémoire processus
cat /proc/self/smaps_rollup

### Limites
cat /proc/self/limits

### CPU global
head /proc/stat

### Mémoire globale
grep -E '^(MemTotal|MemAvailable|SwapTotal|SwapFree):' /proc/meminfo

### Charge
cat /proc/loadavg

### PSI
cat /proc/pressure/cpu
cat /proc/pressure/memory
cat /proc/pressure/io

### Montages
cat /proc/self/mountinfo

### Sysctl
sysctl kernel.hostname
sysctl vm.swappiness

### Sécurité processus
grep -E '^(Cap|NoNewPrivs|Seccomp)' /proc/self/status
```

---
