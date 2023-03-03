## Comment fonctionne gettext ?

Gettext est un outil de traduction de logiciels open source qui permet de créer et de maintenir des fichiers de traduction pour différentes langues. Il est largement utilisé dans les projets de logiciels libres et est disponible sous de nombreuses plateformes (Linux, Unix, Windows, etc.).

Voici comment Gettext fonctionne en gros :

1.  Lorsque vous développez une application, vous pouvez marquer les chaînes de caractères à traduire dans votre code source à l'aide de la fonction `gettext`. Par exemple, en Python :

```python
from gettext import gettext as _  print(_("Hello, world!"))
```

2.  Une fois que vous avez marqué toutes les chaînes de caractères à traduire dans votre code, vous pouvez utiliser l'outil `xgettext` pour extraire ces chaînes et créer un fichier `.pot` (fichier de catalogue de traduction) qui les contient. Le fichier `.pot` est un fichier de texte qui contient les chaînes de caractères à traduire et des informations sur le projet et les personnes responsables de la traduction.
    
3.  Vous pouvez utiliser un outil de traduction, comme Poedit, pour ouvrir le fichier `.pot` et traduire les chaînes de caractères. Lorsque vous enregistrez les modifications, Poedit créera un fichier `.po` (fichier de catalogue de traduction) pour chaque langue traduite. Le fichier `.po` contiendra les chaînes de caractères à traduire et leur traduction dans la langue cible.
    
5.  Une fois que vous avez créé tous les fichiers `.po` de traduction, vous pouvez utiliser l'outil `msgfmt` pour générer les fichiers `.mo` (fichiers de message binaires) à partir de ceux-ci. Les fichiers `.mo` sont des fichiers binaires qui contiennent les mêmes informations que les fichiers `.po`, mais dans un format plus compact et plus rapide à charger par l'application.
    
6.  Vous pouvez maintenant utiliser la fonction `gettext` dans votre code pour charger les fichiers de traduction en fonction de la langue de l'utilisateur. Par exemple, en Python :
    
```python
import gettext  gettext.install("messages", "locale")
```

Cette commande va charger les fichiers de traduction du projet (nommés "messages") à partir du répertoire "locale", en fonction de la langue de l'utilisateur.

## Installation
Pour générer les fichiers .pot (fichiers de catalogue de traduction) à partir des fichiers source Python d'un projet, on peut utiliser la commande `xgettext` de GNU Gettext. Voici comment procéder :

1.  Assurez-vous d'avoir installé Gettext sur votre ordinateur. Si ce n'est pas le cas, vous pouvez l'installer à l'aide de la commande suivante (sous Debian ou Ubuntu) :

```bash
sudo apt-get install gettext
```

2.  Placez-vous dans le répertoire racine du projet (celui où se trouve le fichier `setup.py`).
    
3.  Exécutez la commande suivante :
    

```bash
xgettext --language=Python --output=locale/messages.pot `find . -name "*.py"`
```


Cette commande va parcourir tous les fichiers `.py` du projet et générer un fichier `messages.pot` dans le répertoire `locale`. Ce fichier contiendra les chaînes de caractères à traduire dans tous les fichiers source.

Vous pouvez également utiliser des outils tels que `pygettext` ou `babel` pour générer les fichiers `.pot` de votre projet. Ces outils sont spécialement conçus pour les projets Python et offrent des fonctionnalités supplémentaires, telles que la détection automatique des chaînes de caractères à traduire ou la génération de fichiers `.pot` à partir de templates de templates Django.

## Comment fait-on avec babel ?

Pour utiliser Babel pour générer les fichiers `.pot` de votre projet Python, vous devez d'abord installer l'outil en exécutant la commande suivante :

`pip install babel`

Une fois installé, placez-vous dans le répertoire racine de votre projet et exécutez la commande suivante :

`pybabel extract -F babel.cfg -o locale/messages.pot .`

Cette commande va parcourir tous les fichiers du projet et générer un fichier `messages.pot` dans le répertoire `locale`. Le fichier de configuration `babel.cfg` contient les options de configuration de Babel. Vous pouvez en créer un en exécutant la commande `pybabel init`.

Pour plus d'informations sur l'utilisation de Babel pour la traduction de projets Python, je vous recommande de consulter la documentation officielle de Babel : [https://babel.readthedocs.io/en/latest/](https://babel.readthedocs.io/en/latest/).

Les options `-i` et `-d` de la commande `pybabel init` peuvent contenir des chemins absolus ou relatifs vers les fichiers et répertoires correspondants.

Par exemple, si vous souhaitez créer le fichier de configuration `babel.cfg` à partir d'un fichier `messages.pot` situé dans le répertoire `locale`, vous pouvez utiliser la commande suivante :

```bash
pybabel init -i locale/messages.pot -d locale -l fr
```

N'oubliez pas de remplacer `fr` par la langue de base de votre projet. Vous pouvez également utiliser l'option `-D` pour définir des variables de configuration dans le fichier de configuration généré, comme je vous l'ai indiqué précédemment.

## Quelle est la différence ente *.mo et *.po ?

Un fichier `.pot` (fichier de catalogue de traduction) contient principalement deux parties : l'entête et les entrées de traduction. L'entête du fichier est un bloc de commentaires qui se trouve en haut du fichier et qui fournit des informations sur le projet, la version du fichier `.pot` et les personnes responsables de la traduction.

Voici ce que doit contenir l'entête d'un fichier `.pot` selon la spécification de Gettext :

`# SOME DESCRIPTIVE TITLE. # Copyright (C) YEAR THE PACKAGE'S COPYRIGHT HOLDER # This file is distributed under the same license as the PACKAGE package. # FIRST AUTHOR <EMAIL@ADDRESS>, YEAR. # #, fuzzy msgid "" msgstr "" "Project-Id-Version: PACKAGE VERSION\n" "Report-Msgid-Bugs-To: EMAIL@ADDRESS\n" "POT-Creation-Date: YEAR-MO-DA HO:MI+ZONE\n" "PO-Revision-Date: YEAR-MO-DA HO:MI+ZONE\n" "Last-Translator: FULL NAME <EMAIL@ADDRESS>\n" "Language-Team: LANGUAGE <LL@li.org>\n" "Language: \n" "MIME-Version: 1.0\n" "Content-Type: text/plain; charset=CHARSET\n" "Content-Transfer-Encoding: 8bit\n"`

Voici une explication des différents champs de l'entête :

-   `SOME DESCRIPTIVE TITLE` : titre descriptif du projet.
-   `YEAR THE PACKAGE'S COPYRIGHT HOLDER` : année de copyright du paquet.
-   `FIRST AUTHOR <EMAIL@ADDRESS>` : nom et adresse e-mail de la première personne à avoir traduit le fichier.
-   `PACKAGE VERSION` : version du paquet.
-   `EMAIL@ADDRESS` : adresse e-mail à laquelle envoyer les rapports d'erreurs.
-   `YEAR-MO-DA HO:MI+ZONE` : date de création et de modification du fichier `.pot`.
-   `FULL NAME <EMAIL@ADDRESS>` : nom et adresse e-mail de la dernière personne à avoir traduit le fichier.
-   `LANGUAGE` : langue de traduction.
-   `CHARSET` : jeu de caractères utilisé dans le fichier.

Il est important de remplir correctement ces champs afin de fournir des informations précises sur le projet et sur les personnes qui ont contribué à la traduction. Vous pouvez également ajouter d'autres champs à l'entête si vous le souhaitez, mais assurez-vous de respecter la syntaxe de Gettext.

que signifie le sous répertoire LC_MESSAGES qui est dans "locale" ?

Le répertoire `LC_MESSAGES` se trouve généralement à l'intérieur du répertoire `locale` d'un projet et contient les fichiers de traduction de ce projet. Le répertoire `LC_MESSAGES` est utilisé par les outils de traduction, comme `gettext`, pour trouver les fichiers de traduction correspondants à la langue de l'utilisateur.

La nomenclature du répertoire `LC_MESSAGES` est la suivante :

```bash
locale/
   fr/
        LC_MESSAGES/
           messages.po
           messages.mo
    en/
        LC_MESSAGES/
           messages.po
           messages.mo
```

Dans cet exemple, les fichiers `messages.po` et `messages.mo` contiennent la traduction du projet en français et en anglais, respectivement. Le répertoire `fr` correspond à la langue française et le répertoire `en` à la langue anglaise.

Le répertoire `LC_MESSAGES` doit être nommé ainsi pour être reconnu par les outils de traduction. Si vous utilisez un autre nom, il se peut que les fichiers de traduction ne soient pas chargés correctement par l'application.

quelle est la différence ente *.mo et *.po ?

Les fichiers `.mo` et `.po` sont utilisés par les outils de traduction, comme `gettext`, pour stocker les traductions d'un projet.

-   Les fichiers `.po` (fichiers de catalogue de traduction) sont des fichiers de texte qui contiennent les chaînes de caractères à traduire du projet, ainsi que leur traduction dans la langue cible. Ils sont générés à partir des fichiers source du projet (par exemple, des fichiers `.py` pour un projet Python) à l'aide d'outils tels que `xgettext` ou `pybabel extract`.
    
-   Les fichiers `.mo` (fichiers de message binaires) sont des fichiers binaires qui contiennent les mêmes informations que les fichiers `.po`, mais dans un format plus compact et plus rapide à charger par l'application. Ils sont générés à partir des fichiers `.po` à l'aide de l'outil `msgfmt`.
    

Lorsque vous traduisez un projet, vous travaillez généralement avec des fichiers `.po`, qui sont plus faciles à lire et à modifier. Une fois que vous avez terminé la traduction, vous pouvez générer les fichiers `.mo` à partir des fichiers `.po` pour que l'application puisse les charger plus rapidement.

Il est important de ne pas modifier directement les fichiers `.mo`, car ils sont au format binaire et ne sont pas lisibles par les humains. Si vous souhaitez apporter des modifications à la traduction, vous devez le faire dans les fichiers `.po` correspondants et regénérer les fichiers `.mo` à partir de ceux-ci.