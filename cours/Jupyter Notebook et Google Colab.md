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