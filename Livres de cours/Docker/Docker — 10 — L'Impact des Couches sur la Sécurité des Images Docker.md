---
schema_version: 1
uid: 01M1BQ62DWMTPK846GSWZ5F0P2
titre: "Docker — 10 — L'Impact des Couches sur la Sécurité des Images Docker"
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
resume: "Chapitre 10 sur 17 du livre « Docker » : L'Impact des Couches sur la Sécurité des Images Docker. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Docker » — chapitre 10/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 09 — Les Dockerfiles|← 09 — Les Dockerfiles]] · [[Docker — 11 — Les Différences entre les Systèmes de Fichiers en Couches Utilisés par Docker OverlayFS, AUFS, etc|11 — Les Différences entre les Systèmes de Fichiers en Couches Utilisés par Docker OverlayFS, AUFS, etc →]]

# L'Impact des Couches sur la Sécurité des Images Docker
Le mécanisme des couches (*layers*) dans Docker joue un rôle significatif dans la **sécurité** des applications conteneurisées. Comprendre comment les couches affectent la sécurité des images Docker est essentiel pour éviter les vulnérabilités et assurer la protection de nos systèmes.

## Introduction

Dans Docker, une image est construite à partir d'une série d'instructions spécifiées dans un **Dockerfile**. Chaque instruction crée une nouvelle couche, qui est empilée sur les couches précédentes. Ces couches sont immuables et forment ensemble le système de fichiers de l'image. Bien que ce mécanisme offre des avantages en termes de réutilisation et d'efficacité, il peut introduire des risques de sécurité s'il n'est pas géré correctement.

## Risques de Sécurité Associés aux Couches

### 1. **Propagation des Vulnérabilités**

- **Images de Base Vulnérables** : Si nous utilisons une image de base contenant des vulnérabilités connues, toutes les images dérivées héritent de ces failles. Par exemple, une image basée sur une version obsolète d'Ubuntu peut être exposée à des vulnérabilités critiques.

- **Impact Multiplié** : Les vulnérabilités dans les couches inférieures peuvent affecter un grand nombre d'images et de conteneurs, amplifiant l'impact potentiel d'une faille de sécurité.

### 2. **Inclusion Accidentelle de Données Sensibles**

- **Persistences des Données** : Si des secrets (clés, mots de passe, certificats) sont ajoutés dans une couche et supprimés dans une couche ultérieure, ils restent présents dans l'historique des couches. Un attaquant pourrait extraire ces informations en inspectant les couches.

- **Difficulté de Suppression** : Une fois qu'une donnée sensible est incluse dans une couche, il est difficile de l'éliminer complètement sans reconstruire l'image depuis le début.

### 3. **Surface d'Attaque Élargie**

- **Taille des Images** : Des images volumineuses avec de nombreuses couches peuvent inclure des composants inutiles, augmentant ainsi la surface d'attaque.

- **Composants Obsolètes** : Les couches contenant des paquets ou des dépendances non mis à jour peuvent introduire des vulnérabilités.

### 4. **Cache de Construction Non Mis à Jour**

- **Réutilisation du Cache** : Docker utilise le cache pour accélérer la construction des images. Si le cache contient des versions obsolètes de paquets, les mises à jour de sécurité peuvent être omises.

- **Instructions RUN** : Les commandes `RUN` qui installent des paquets sans forcer la mise à jour peuvent entraîner l'inclusion de versions vulnérables.

### 5. **Isolation Incomplète**

- **Permissions Excessives** : Si les couches ne sont pas configurées pour exécuter des processus avec des utilisateurs non privilégiés, les conteneurs peuvent être exécutés avec des privilèges root, augmentant les risques en cas de compromission.

## Bonnes Pratiques pour Renforcer la Sécurité des Couches

### 1. **Utiliser des Images de Base Sécurisées**

- **Sources Fiables** : Utilisez des images officielles ou vérifiées provenant de dépôts de confiance.

- **Images Minimisées** : Préférez des images légères comme `alpine` ou `scratch` pour réduire le nombre de composants potentiellement vulnérables.

### 2. **Maintenir les Images à Jour**

- **Mises à Jour Régulières** : Reconstruisez régulièrement nos images pour inclure les dernières mises à jour de sécurité.

- **Tags Spécifiques** : Évitez d'utiliser le tag `latest` et spécifiez des versions précises pour mieux contrôler les mises à jour.

### 3. **Minimiser le Nombre de Couches**

- **Combiner les Commandes** : Combinez les instructions `RUN` pour réduire le nombre de couches et éviter la duplication des données.

- **Nettoyage** : Supprimez les fichiers temporaires et les caches dans la même instruction `RUN` pour empêcher leur inclusion dans des couches antérieures.

### 4. **Gérer les Secrets de Manière Sécurisée**

- **Éviter d'Inclure des Secrets** : Ne jamais inclure de secrets ou de données sensibles dans le Dockerfile ou les arguments de construction.

- **Utiliser des Outils Sécurisés** : Employez des solutions comme Docker Secrets ou des variables d'environnement injectées au moment de l'exécution.

### 5. **Analyser les Images pour les Vulnérabilités**

- **Outils de Scan** : Utilisez des outils d'analyse tels que **Trivy**, **Clair**, ou **Anchore** pour détecter les vulnérabilités dans nos images.

- **Intégration Continue** : Intégrez ces analyses dans notre pipeline CI/CD pour détecter les problèmes le plus tôt possible.

### 6. **Exécuter en Tant qu'Utilisateur Non Privilégié**

- **Instruction USER** : Utilisez `USER` pour spécifier un utilisateur non-root pour l'exécution des processus dans le conteneur.

- **Réduction des Privilèges** : Limitez les permissions au strict nécessaire pour l'application.

### 7. **Utiliser le Fichier .dockerignore**

- **Exclusion de Fichiers** : Créez un fichier `.dockerignore` pour exclure les fichiers et dossiers non nécessaires, tels que les clés privées, les fichiers de configuration sensibles, ou les répertoires de build locaux.

### 8. **Gérer le Cache de Construction**

- **Forcer les Mises à Jour** : Incluez `apt-get update` et `apt-get install` dans la même instruction `RUN` pour éviter l'utilisation du cache.

- **Invalidation du Cache** : Ajoutez des arguments ou des variables qui changent régulièrement pour invalider le cache lorsque c'est nécessaire.

### 9. **Vérifier les Sommes de Contrôle**

- **Intégrité des Téléchargements** : Lorsque nous téléchargeons des fichiers externes, vérifions les sommes de contrôle pour nous assurer qu'ils n'ont pas été compromis.

- **Utiliser HTTPS** : Téléchargez toujours les dépendances via des connexions sécurisées.

## Exemples Concrets

### Inclusion de Données Sensibles

**Dockerfile Non Sécurisé :**

```dockerfile
FROM python:3.9

# Mauvaise pratique : inclusion d'un fichier de clés
COPY id_rsa /root/.ssh/id_rsa

# Suppression du fichier de clés
RUN rm /root/.ssh/id_rsa

CMD ["python", "app.py"]
```

**Problème :** La clé privée `id_rsa` est toujours présente dans une couche précédente et peut être récupérée.

**Solution :**

- **Ne pas inclure de secrets dans le Dockerfile.**
- **Utiliser des volumes ou des variables d'environnement pour injecter les secrets au moment de l'exécution.**

### Gestion Correcte des Mises à Jour

**Dockerfile Vulnérable :**

```dockerfile
FROM ubuntu:20.04

RUN apt-get update
RUN apt-get install -y openssl
```

**Problème :** Si le cache est utilisé, `apt-get update` peut ne pas récupérer les dernières informations, laissant des paquets obsolètes.

**Dockerfile Sécurisé :**

```dockerfile
FROM ubuntu:20.04

RUN apt-get update && \
    apt-get install -y openssl && \
    rm -rf /var/lib/apt/lists/*
```

**Avantages :**

- **Assure que les dernières mises à jour sont installées.**
- **Réduit la taille de l'image en nettoyant les listes de paquets.**

## Impact sur la Chaîne d'Approvisionnement

- **Effet Domino** : Une vulnérabilité dans une couche de base peut affecter toutes les images dérivées, compromettant potentiellement l'ensemble de la chaîne d'approvisionnement logicielle.

- **Importance de la Vérification** : Il est essentiel de vérifier régulièrement les images de base pour détecter les vulnérabilités et les mettre à jour.

## Conclusion

Les couches Docker sont un élément fondamental de la construction des images, mais elles nécessitent une attention particulière en matière de sécurité. En adoptant des pratiques sécurisées lors de la création et de la gestion des couches, nous pouvons :

- **Réduire les Risques de Vulnérabilités** : En utilisant des images de base sécurisées et en maintenant nos images à jour.

- **Protéger les Données Sensibles** : En évitant d'inclure des secrets dans les couches et en gérant les configurations de manière sécurisée.

- **Optimiser la Sécurité** : En minimisant la surface d'attaque grâce à des images légères et à un nombre réduit de couches.

- **Assurer la Conformité** : En intégrant des scans de sécurité et des mises à jour régulières dans nos processus de développement.

## Actions Recommandées

- **Audit Régulier** : Mettez en place des audits réguliers de nos images pour identifier les vulnérabilités potentielles.

- **Formation** : Sensibilisez les développeurs et les équipes DevOps aux meilleures pratiques de sécurité liées aux couches Docker.

- **Automatisation** : Intégrez des outils d'analyse de sécurité dans nos pipelines CI/CD pour détecter et corriger rapidement les problèmes.

- **Documentation** : Maintenez une documentation à jour des images utilisées, des versions, et des modifications apportées aux Dockerfiles.

---
> [!info] Livre « Docker » — chapitre 10/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 09 — Les Dockerfiles|← 09 — Les Dockerfiles]] · [[Docker — 11 — Les Différences entre les Systèmes de Fichiers en Couches Utilisés par Docker OverlayFS, AUFS, etc|11 — Les Différences entre les Systèmes de Fichiers en Couches Utilisés par Docker OverlayFS, AUFS, etc →]]
