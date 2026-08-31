---
schema_version: 1
uid: 01M1BQ62DX8K2KE03222BNND8T
titre: "Docker — 14 — Utilisation de Docker Compose"
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
resume: "Chapitre 14 sur 17 du livre « Docker » : Utilisation de Docker Compose. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Docker » — chapitre 14/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 13 — Fonctionnement de Docker côté Hôte Mécanismes des Namespaces et Partage de la Couche Réseau|← 13 — Fonctionnement de Docker côté Hôte Mécanismes des Namespaces et Partage de la Couche Réseau]] · [[Docker — 15 — Les réseaux|15 — Les réseaux →]]

# Utilisation de Docker Compose
Utilisation de Docker Compose pour définir et exécuter des applications multi-conteneurs :

## Introduction
Docker Compose est un outil qui permet de définir et de gérer des applications multi-conteneurs en utilisant un fichier de configuration. Il facilite la création, la mise à l'échelle et la gestion des conteneurs d'une application complexe.

## Les concepts clés de Docker Compose
Le fichier `compose.yaml` est le cœur de la configuration d'une application multi-conteneurs.
Il décrit tous les services, réseaux et volumes nécessaires pour faire fonctionner l'application. Quelques sections courantes d'un fichier Compose :
- **services** : Définit les conteneurs qui constituent l'application. Chaque service peut être basé sur une image Docker existante ou construite à partir d'un `Dockerfile`.
- **build** : Spécifie les instructions pour construire une image Docker à partir d'un `Dockerfile`. - **image** : Définit l'image Docker à utiliser pour un service.
- **ports** : Expose des ports spécifiques du conteneur à l'hôte, permettant à des services externes d'accéder à l'application.
- **volumes** : Monte des volumes pour partager des fichiers et des répertoires entre l'hôte et les conteneurs.
- **environment** : Définit des variables d'environnement pour les services, souvent utilisées pour configurer les conteneurs au démarrage.
- **depends_on** : Indique les dépendances entre les services, ce qui permet de spécifier l'ordre de démarrage des conteneurs.
- **networks** : Définit les réseaux personnalisés sur lesquels les services communiqueront.
- **watch** : Permet de synchroniser l'image avec les changements du code source, ce qui est particulièrement utile pour le développement en continu.
## Exemples
Un exemple de Compose file qui définit une application web qui utilise un service Nginx :
```yaml
    reverseproxy:
        image: reverseproxy
        ports:
            - 8080:8080
            - 8081:8081
        restart: always
 
    nginx:
        depends_on:
            - reverseproxy
        image: nginx:alpine
        restart: always
 
    apache:
        depends_on:
            - reverseproxy
        image: httpd:alpine
        restart: always
```

## Principales commandes de `docker-compose`

Voici un aperçu des principales commandes utilisées avec **Docker Compose**, ainsi qu’une brève description de chacune :

1. **docker-compose up**
    
    - Démarre les conteneurs définis dans votre fichier `docker-compose.yml`.
    - En utilisant l’option `-d` (_detach mode_), les conteneurs s’exécutent en arrière-plan :
        
        ```bash
        docker-compose up -d
        ```
        
    - Sans l’option `-d`, vous verrez les logs en direct dans votre terminal.
    
2. **docker-compose down**
    
    - Arrête et supprime les conteneurs, les réseaux et autres artefacts (volumes anonymes, etc.) créés par `docker-compose up`.
    - Permet de nettoyer l’environnement, notamment si vous voulez redémarrer de zéro.
        
        ```bash
        docker-compose down
        ```
        
3. **docker-compose build**
    
    - Construit ou re-construit les images définies dans le `docker-compose.yml`.
    - Peut être utile si vous avez modifié votre Dockerfile ou vos configurations et que vous souhaitez que l’image soit à jour.
        
        ```bash
        docker-compose build
        ```
        
4. **docker-compose pull**
    
    - Récupère (depuis un registry) les dernières images spécifiées dans votre `docker-compose.yml`.
    - Souvent utilisé avant un `docker-compose up` pour s’assurer d’avoir la version la plus récente des images.
        
        ```bash
        docker-compose pull
        ```
        
5. **docker-compose push**
    
    - Envoie (pousse) les images construites localement vers un registry Docker distant.
    - Nécessite d’avoir la bonne configuration de registry et les tags appropriés.
        
        ```bash
        docker-compose push
        ```
        
6. **docker-compose run**
    
    - Lance un _one-off container_, c’est-à-dire un conteneur ponctuel pour exécuter une commande.
    - Par exemple, exécuter un script de migration de base de données :
        
        ```bash
        docker-compose run web python manage.py migrate
        ```
        
    - Ne démarre pas tous les services, uniquement celui que vous spécifiez (ici `web`).
    
7. **docker-compose exec**
    
    - Permet d’exécuter une commande dans un conteneur déjà en cours d’exécution.
    - Par exemple, pour ouvrir un shell dans un conteneur :
        
        ```bash
        docker-compose exec web sh
        ```
        
    - S’emploie souvent pour diagnostiquer ou administrer une application à l’intérieur du conteneur.
    
8. **docker-compose logs**
    
    - Affiche les logs des conteneurs.
    - Vous pouvez préciser un service particulier :
        
        ```bash
        docker-compose logs web
        ```
        
    - L’option `-f` (follow) vous permet de suivre les logs en continu :
        
        ```bash
        docker-compose logs -f web
        ```
        
9. **docker-compose ps**
    
    - Liste les conteneurs gérés par Docker Compose dans le projet courant.
    - Vous donne un aperçu rapide des conteneurs en cours d’exécution ou stoppés, et leurs ports.
        
        ```bash
        docker-compose ps
        ```
        
10. **docker-compose start / stop / restart**
    
    - Respectivement pour démarrer, arrêter ou redémarrer les conteneurs déjà créés.
    - Exemple :
        
        ```bash
        docker-compose stop
        docker-compose start
        docker-compose restart
        ```
        
11. **docker-compose pause / unpause**
    
    - Met en pause (`pause`) ou reprend (`unpause`) l’exécution des conteneurs.
    - Utile pour stopper temporairement un service sans l’arrêter complètement.
        
        ```bash
        docker-compose pause
        docker-compose unpause
        ```
        
12. **docker-compose rm**
    
    - Supprime les conteneurs arrêtés qui appartiennent à votre projet compose.
    - Si vous avez fait un `docker-compose stop` et que vous ne souhaitez plus conserver ces conteneurs, vous pouvez faire :
        
        ```bash
        docker-compose rm
        ```
        
    - Par défaut, il demande confirmation avant de supprimer.
    
13. **docker-compose config**
    
    - Valide et affiche la configuration combinée du fichier `docker-compose.yml`.
    - Pratique pour vérifier si votre syntaxe est correcte et pour visualiser la configuration finale après le merge avec d’éventuels fichiers d’override.
        
        ```bash
        docker-compose config
        ```

### Résumé des commandes `docker-compose`

- **up** : Lance tous les services définis.
- **down** : Arrête et supprime les ressources associées.
- **build** : Construit les images.
- **pull/push** : Récupère ou publie les images depuis/vers un registry.
- **run** : Lance un conteneur ponctuel pour une commande donnée.
- **exec** : Exécute une commande dans un conteneur en cours d’exécution.
- **logs** : Affiche ou suit les journaux.
- **ps** : Liste les conteneurs en cours de gestion.
- **start/stop/restart** : Gère l’état d’exécution des conteneurs.
- **pause/unpause** : Met en pause ou reprend l’exécution.
- **rm** : Supprime les conteneurs arrêtés.
- **config** : Valide et affiche la configuration.

Ces commandes couvrent la majorité des opérations de base que nous réaliserons avec Docker Compose pour développer, tester et déployer nos applications multi-conteneurs.
## Pratique
-   Créez un fichier Compose
-   Utilisez les instructions pour définir une application multi-conteneurs
-   Exécutez l'application en utilisant les commandes de Compose

---
> [!info] Livre « Docker » — chapitre 14/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 13 — Fonctionnement de Docker côté Hôte Mécanismes des Namespaces et Partage de la Couche Réseau|← 13 — Fonctionnement de Docker côté Hôte Mécanismes des Namespaces et Partage de la Couche Réseau]] · [[Docker — 15 — Les réseaux|15 — Les réseaux →]]
