---
schema_version: 1
uid: 01M1BQ62DXWK58P47PNWGYQC4E
titre: "Docker — Compléments 2026"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - administration-systeme
  - conteneurisation
  - docker
  - devops
resume: "Compléments apportés au livre « Docker » : sections de la version condensée du cours [[Docker]] (31 août 2026) dont le sujet est absent de la version longue."
niveau: intermediaire
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-01-27
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre « Docker »
> [[Docker — Sommaire|Sommaire]] · [[Docker — 17 — Exemple de multi-réseaux pour un conteneur|← 17 — Exemple de multi-réseaux pour un conteneur]] · [[Docker — Sommaire|Sommaire →]]

# Compléments 2026

> [!info] Origine
> Les sections ci-dessous proviennent de la version condensée et actualisée du cours [[Docker]] (31 août 2026). Elles traitent de sujets absents de la version longue et n'ont pas été fondues dans les chapitres ; pour les versions logicielles et l'état de l'art du moment, la version condensée fait foi.

## 6.2 Signaux

Lors de :

```bash
docker stop app
```

Docker envoie d'abord le signal d'arrêt configuré, généralement `SIGTERM`, puis attend un délai avant de forcer avec `SIGKILL` si nécessaire.

L'application doit donc :

- recevoir correctement les signaux ;
- terminer proprement les requêtes en cours ;
- fermer les connexions ;
- sortir avant le timeout.

## 18. Healthchecks

Un processus vivant n'est pas nécessairement une application saine.

Dockerfile :

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD ["/app/healthcheck"]
```

Inspecter :

```bash
docker inspect --format '{{json .State.Health}}' app
```

Un healthcheck doit être :

- rapide ;
- local ;
- stable ;
- représentatif de la capacité du service à fonctionner.

Éviter un healthcheck qui déclenche une opération métier coûteuse.

## 27. Multi-architecture

Lister les plateformes du builder :

```bash
docker buildx inspect --bootstrap
```

Construire :

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --tag registry.example.org/app:1.0 \
  --push .
```

Vérifier :

```bash
docker buildx imagetools inspect registry.example.org/app:1.0
```

Attention à ne pas supposer que la simple émulation QEMU équivaut toujours à une compilation native en performances et comportement.

## 29. Supply chain : SBOM et provenance

La sécurité d'une image ne concerne pas uniquement sa configuration runtime.

Nous devons pouvoir répondre à :

- quels paquets contient-elle ?
- quelle image de base a été utilisée ?
- quel commit a produit l'artefact ?
- avec quel builder ?
- quelles vulnérabilités connues existent ?

## 44. TP 1 — Cycle de vie

Objectif : comprendre l'immutabilité de l'image et l'éphémérité du conteneur.

1. Lancer :

```bash
docker run --name tp1 -it alpine sh
```

2. Dans le conteneur :

```sh
echo test > /tmp/fichier
```

3. Quitter.
4. Relancer le même conteneur :

```bash
docker start -ai tp1
```

5. Vérifier que `/tmp/fichier` existe.
6. Supprimer le conteneur puis créer un nouveau conteneur depuis la même image.
7. Expliquer pourquoi le fichier a disparu.

## 56. TP 13 — Diagnostic

Un enseignant prépare volontairement une application avec plusieurs erreurs :

- mauvais port ;
- application liée sur loopback ;
- volume non inscriptible ;
- healthcheck erroné ;
- DNS de service incorrect ;
- OOM ;
- processus principal qui termine immédiatement.

Utiliser :

```bash
docker ps -a
docker logs
docker inspect
docker stats
docker network inspect
docker volume inspect
docker system df
```

pour établir un diagnostic reproductible.

## 60. Ce qu'il faut retenir

Docker ne doit pas être appris comme une liste de commandes.

Le modèle mental essentiel est :

```text
source
  |
Dockerfile
  |
Buildx
  |
BuildKit
  |
image OCI immuable
  |
registry
  |
conteneur
  |
namespaces + cgroups + LSM + seccomp
  |
volumes / réseaux / configuration / secrets
```

Les points les plus importants sont :

1. **une image est un artefact ; un conteneur est une exécution de cet artefact** ;
2. **un conteneur partage le noyau hôte** ;
3. **la couche écrivable du conteneur n'est pas une stratégie de persistance** ;
4. **les secrets ne doivent jamais être intégrés à l'image** ;
5. **BuildKit est le builder moderne par défaut** ;
6. **`docker compose` remplace l'ancien `docker-compose`** ;
7. **Compose est excellent en mono-hôte, mais ce n'est pas un orchestrateur de cluster complet** ;
8. **le socket Docker est privilégié** ;
9. **rootless et le moindre privilège réduisent le risque** ;
10. **la supply chain doit être vérifiable avec digests, SBOM et provenance** ;
11. **les données doivent être sauvegardées indépendamment des conteneurs** ;
12. **un déploiement doit être reproductible depuis le code et la configuration**.

## Références

Documentation officielle Docker :

```text
https://docs.docker.com/
https://docs.docker.com/engine/
https://docs.docker.com/build/
https://docs.docker.com/compose/
https://docs.docker.com/engine/security/
https://docs.docker.com/engine/security/rootless/
https://docs.docker.com/storage/
https://docs.docker.com/network/
https://docs.docker.com/scout/
```

Standards OCI :

```text
https://opencontainers.org/
https://github.com/opencontainers/image-spec
https://github.com/opencontainers/runtime-spec
https://github.com/opencontainers/distribution-spec
```

Voir aussi :

- [[Les namespaces Linux]]
- [[GNULinux]]
- [[Sécurité avancée sous Linux]]
- [[git]]
