---
schema_version: 1
uid: 01M1BQ62DVXNNTYT6YJFTTJ6CY
titre: "Docker — 06 — Le mode détaché"
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
resume: "Chapitre 6 sur 17 du livre « Docker » : Le mode détaché. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Docker » — chapitre 6/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 05 — Les commandes de bases|← 05 — Les commandes de bases]] · [[Docker — 07 — Principales commandes|07 — Principales commandes →]]

# Le mode détaché
L'option `-d` ou `--detach` dans Docker signifie "détaché". Lorsqu'elle est utilisée, cette option permet de lancer un conteneur en arrière-plan et de le détacher du terminal. Cela signifie que la commande Docker retourne immédiatement le contrôle au terminal et le conteneur continue de s'exécuter en arrière-plan.

## Détails de l'Option `-d` :
- **Arrière-plan** : Le conteneur est lancé en mode détaché, ce qui est utile lorsque nous ne souhaitons pas que le conteneur monopolise la ligne de commande. Nous pouvons ainsi continuer à utiliser le terminal pour d'autres tâches pendant que le conteneur fonctionne.
- **Sortie du conteneur** : Étant donné que le conteneur est exécuté en arrière-plan, la sortie standard du conteneur n'apparaît pas dans le terminal. Cependant, nous pouvons voir les logs du conteneur en utilisant la commande `docker logs`.
- **Cas d'utilisation** : Le mode détaché est souvent utilisé pour les conteneurs de services qui doivent tourner de manière autonome, comme un serveur web, une base de données, etc.

## Exemple d'utilisation :
```bash
docker run -d --name mon_conteneur nginx
```
Dans cet exemple, le conteneur utilisant l'image `nginx` sera lancé en mode détaché et nommé `mon_conteneur`. Nous pourrons continuer à utiliser le terminal après l'exécution de cette commande.

Pour vérifier que le conteneur est bien en cours d'exécution, utilisez :
```bash
docker ps
```

## Avantages :
- **Contrôle du terminal** : Nous pouvons continuer à travailler dans notre terminal pendant que le conteneur tourne en arrière-plan.
- **Facilité de gestion** : Idéal pour les applications de fond et les services qui doivent rester actifs même lorsque l'utilisateur n'est pas connecté.

---
> [!info] Livre « Docker » — chapitre 6/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 05 — Les commandes de bases|← 05 — Les commandes de bases]] · [[Docker — 07 — Principales commandes|07 — Principales commandes →]]
