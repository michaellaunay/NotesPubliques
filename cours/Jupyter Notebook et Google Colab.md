Jupyter est aujourd’hui l’un des outils les plus utilisés en data science, mais il n’est pas né directement sous ce nom. Son histoire est liée à l’évolution du langage Python et des besoins des chercheurs en calcul scientifique.

# Origine : le projet IPython (2001)

Au départ, tout commence avec **IPython**, créé en 2001 par **Fernando Pérez**, alors étudiant en physique.

L’idée était simple :  
créer un **interpréteur Python interactif amélioré** pour faciliter le travail scientifique.

À cette époque, Python était déjà utilisé dans la recherche, mais son terminal interactif était très limité.

IPython apportait :

- l’auto-complétion,    
- l’historique des commandes,    
- une meilleure lisibilité,    
- des outils pour l’exploration de données.    

C’était donc un **shell scientifique interactif** pour Python.

# Naissance du notebook (vers 2011)

Vers 2010–2011, l’équipe IPython introduit une innovation majeure :  
le **notebook interactif dans le navigateur**.

Ce nouvel outil permettait :

- d’écrire du code,    
- d’exécuter les cellules,    
- d’afficher des graphiques,    
- de mélanger texte, équations et code.

On ne travaillait plus seulement dans un terminal, mais dans un **document interactif**.

Ce concept était révolutionnaire pour :

- l’enseignement,    
- la recherche,    
- l’analyse de données.
# Le projet Jupyter (2014)

En 2014, le projet IPython évolue pour devenir **Project Jupyter**.

Le nom **Jupyter** vient de la combinaison de trois langages :

- **Ju** : Julia
- **Py** : Python
- **R** : R

L’objectif était clair :

> créer une plateforme de notebooks **multi-langages**, et non plus limitée à Python.

IPython devient alors :

- le **kernel Python** de Jupyter,
- tandis que Jupyter devient une architecture générale.
# Architecture de Jupyter

Jupyter repose sur une architecture en trois parties :

1. **Le Notebook (interface)**  
    Interface web dans le navigateur.
2. **Le Kernel (moteur d’exécution)**  
    Processus qui exécute le code (Python, R, Julia, etc.).
3. **Le protocole Jupyter**  
    Système de communication entre l’interface et le kernel.

Cela permet d’avoir :

- un notebook Python,
- un notebook R,
- un notebook Julia,
- ou même des notebooks pour SQL, Java, etc.
# Pourquoi Jupyter est devenu central en data science

## 1) Un outil interactif et exploratoire

La data science repose sur :

- l’exploration de données,
- les tests rapides,
- les visualisations.

Jupyter permet :

- d’exécuter du code cellule par cellule,
- de modifier et relancer rapidement,
- d’afficher des graphiques immédiatement.

C’est idéal pour le travail **itératif**.

## 2) Un document scientifique complet

Un notebook peut contenir :

- du code Python,
- du texte explicatif (Markdown),
- des formules mathématiques (LaTeX),
- des graphiques,
- des tableaux.

On obtient un **document reproductible** :

> code + résultats + explications au même endroit.

C’est parfait pour :

- la recherche,
- les rapports,
- l’enseignement.

## 3) Facile à partager

Les notebooks sont des fichiers `.ipynb` (format JSON) qui peuvent être :

- partagés par email,
- versionnés sur Git,
- visualisés directement sur GitHub,
- exécutés sur des plateformes comme :
    - Google Colab,
    - Kaggle,
    - Binder.

Cela a fortement contribué à leur adoption.

## 4) Intégration avec l’écosystème Python

Jupyter s’est développé en même temps que l’écosystème scientifique Python :

- NumPy
- pandas
- matplotlib
- scikit-learn
- TensorFlow
- PyTorch

Tous ces outils sont parfaitement utilisables dans un notebook.
# 5) Standard de facto dans l’enseignement et la recherche

Aujourd’hui, Jupyter est utilisé :

- dans les universités,
- dans les MOOCs,
- dans les entreprises,
- dans les laboratoires de recherche.

Il est devenu le **format standard** pour :

- les cours de data science,
- les tutoriels,
- les compétitions Kaggle.

# Limites de Jupyter

Malgré son succès, Jupyter a aussi des défauts :

1. **Ordre d’exécution non linéaire**  
    On peut exécuter les cellules dans n’importe quel ordre, ce qui peut produire des résultats incohérents.
2. **Difficile à maintenir en production**  
    Les notebooks ne sont pas faits pour :
    - le déploiement,
    - les pipelines robustes,
    - les tests automatisés.
3. **Conflits Git fréquents**  
    Les fichiers `.ipynb` sont du JSON, ce qui rend les merges difficiles.
# Résumé chronologique

| Année       | Événement                                          |
| ----------- | -------------------------------------------------- |
| 2001        | Création d’IPython par Fernando Pérez              |
| ~2011       | Apparition du notebook interactif                  |
| 2014        | Naissance du projet Jupyter (multi-langages)       |
| 2015–2020   | Adoption massive en data science                   |
| Aujourd’hui | Standard de facto pour l’analyse et l’enseignement |

# Idée clé à retenir

Jupyter n’est pas seulement un outil technique.  
C’est un **nouveau paradigme de travail scientifique** :

> Un document exécutable, reproductible et interactif.

C’est cette combinaison entre :

- code,
- visualisation,
- explication,

qui a fait de Jupyter l’outil central de la data science moderne.

# Jupyter Notebook et Google Colab

Dans un environnement Google Colab ou Jupyter Notebook, la plupart des commandes Bash standards sont accessibles via le préfixe `!` dans les cellules de code Python. Voici un aperçu des commandes Bash courantes et disponibles dans ces environnements :

# Commandes shell
## 1. **Commandes de gestion des fichiers et répertoires**
- `ls` : Liste les fichiers et répertoires dans le répertoire courant.
- `pwd` : Affiche le chemin du répertoire courant.
- `cd` : Change de répertoire.
- `mkdir` : Crée un nouveau répertoire.
- `rm` : Supprime des fichiers ou des répertoires (avec l'option `-r` pour supprimer récursivement).
- `cp` : Copie des fichiers ou des répertoires.
- `mv` : Déplace ou renomme des fichiers ou des répertoires.
- `cat` : Affiche le contenu d’un fichier.
- `touch` : Crée un fichier vide ou met à jour le timestamp d'un fichier.

## 2. **Commandes réseau**
- `wget` : Télécharge des fichiers depuis une URL.
- `curl` : Transfère des données vers/depuis un serveur avec une URL.
- `ping` : Envoie des paquets ICMP à une adresse IP pour tester la connectivité.
- `ifconfig` / `ip` : Affiche la configuration réseau de la machine.
  
## 3. **Commandes de gestion des processus**
- `ps` : Liste les processus en cours d'exécution.
- `top` : Affiche une vue dynamique des processus.
- `kill` : Termine un processus par son PID.
- `htop` : Version améliorée de `top` (nécessite parfois installation préalable).

## 4. **Commandes de gestion des paquets**
- `pip` : Installe ou gère des paquets Python (souvent utilisé dans Colab/Jupyter).
  - Ex. : `!pip install numpy`
- `apt-get` : Installe des paquets sur les distributions Linux basées sur Debian.
  - Ex. : `!apt-get install git`
  
## 5. **Commandes d'édition et de manipulation de fichiers**
- `echo` : Affiche une chaîne de caractères ou écrit dans un fichier.
- `nano` / `vi` : Éditeurs de texte en ligne de commande (souvent nano est préféré dans Colab).
- `head` / `tail` : Affiche les premières ou dernières lignes d'un fichier.
- `grep` : Recherche une chaîne de texte dans un fichier.
- `find` : Recherche des fichiers et répertoires.
- `sed` : Modifie des fichiers en utilisant des expressions régulières.
- `awk` : Traite et analyse des fichiers texte.

## 6. **Commandes de compression et archivage**
- `tar` : Compresse ou extrait des fichiers dans une archive `.tar`.
- `gzip` / `gunzip` : Compresse ou décompresse des fichiers `.gz`.
- `unzip` : Décompresse des fichiers `.zip`.

## 7. **Commandes système**
- `uname -a` : Affiche les informations sur le système (noyau, architecture, etc.).
- `df` : Affiche l'utilisation du disque.
- `du` : Affiche la taille des fichiers ou répertoires.
- `free` : Montre l'utilisation de la mémoire.

## 8. **Commandes Git**
- `git clone` : Clone un dépôt Git.
- `git pull` : Récupère les dernières modifications d'un dépôt distant.
- `git commit` : Enregistre des changements dans l'historique local du dépôt.
- `git push` : Envoie les commits locaux vers un dépôt distant.

## 9. **Redirections et Pipes**
- `>` : Redirige la sortie d’une commande vers un fichier.
  - Ex. : `echo "Hello, World!" > file.txt`
- `>>` : Ajoute la sortie d’une commande à la fin d’un fichier existant.
- `|` : Pipe, redirige la sortie d’une commande vers une autre.
  - Ex. : `ps aux | grep python`

## 10. **Variables et contrôle du shell**
- `export` : Définit une variable d'environnement.
  - Ex. : `export MY_VAR="value"`
- `env` : Liste toutes les variables d'environnement.
- `which` : Trouve l'emplacement d'une commande.
- `source` : Exécute un fichier script dans l'environnement courant.

## 11. **Commandes spécifiques à Google Colab**
Google Colab ajoute quelques utilitaires spécifiques :
- `!nvidia-smi` : Affiche des informations sur le GPU (si l'accès GPU est activé).
- `!pip install <package>` : Gère les installations de paquets Python directement dans l'environnement.

## 12. **Commandes système supplémentaires**
- `chmod` : Change les permissions d'un fichier.
- `chown` : Change le propriétaire d'un fichier.
- `df -h` : Affiche l’espace disque libre de manière lisible.

# Durée de l'import des fichiers
Les fichiers importés dans Google Notebook ne sont conservés que 12h ou jusqu'à la prochaine réinitialisation. Pour importer un fichier il suffit de cliquer sur l'icone `dossier` dans la barre d'outils à droite du Notebook, ce qui importe localement le fichier.
Pour ne pas avoir à importer les fichiers à chaque redémarrage du NoteBook il suffit de connecter son Google Drive au Notebook, et de déposer dessus les fichiers à importer.
# Connexion de Google Notebook à Google Drive
Pour cela il suffit de cliquer sur le symbole dossier dans la barre d'outils à droite, puis sur l'icône drive.