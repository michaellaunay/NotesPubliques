## Comment fonctionne gettext ?

Gettext est un outil de traduction de logiciels open source qui permet de créer et de maintenir des fichiers de traduction pour différentes langues. Il est largement utilisé dans les projets de logiciels libres et est disponible sous de nombreuses plateformes (Linux, Unix, Windows, etc.).

Voici comment Gettext fonctionne en gros :

1.  Lorsque nous développons une application, nous pouvons marquer les chaînes de caractères à traduire dans votre code source à l'aide de la fonction `gettext`. Par exemple, en Python :

```python
from gettext import gettext as _  print(_("Hello, world!"))
```

2.  Une fois que nous avons marqué toutes les chaînes de caractères à traduire dans votre code, nous pouvons utiliser l'outil `xgettext` pour extraire ces chaînes et créer un fichier `.pot` (fichier de catalogue de traduction) qui les contient. Le fichier `.pot` est un fichier de texte qui contient les chaînes de caractères à traduire et des informations sur le projet et les personnes responsables de la traduction.
    
3.  nous pouvons utiliser un outil de traduction, comme Poedit, pour ouvrir le fichier `.pot` et traduire les chaînes de caractères. Lorsque nous enregistrons les modifications, Poedit créera un fichier `.po` (fichier de catalogue de traduction) pour chaque langue traduite. Le fichier `.po` contiendra les chaînes de caractères à traduire et leur traduction dans la langue cible.
    
5.  Une fois que nous avons créé tous les fichiers `.po` de traduction, nous pouvons utiliser l'outil `msgfmt` pour générer les fichiers `.mo` (fichiers de message binaires) à partir de ceux-ci. Les fichiers `.mo` sont des fichiers binaires qui contiennent les mêmes informations que les fichiers `.po`, mais dans un format plus compact et plus rapide à charger par l'application.
    
6.  nous pouvons maintenant utiliser la fonction `gettext` dans votre code pour charger les fichiers de traduction en fonction de la langue de l'utilisateur. Par exemple, en Python :
    
```python
import gettext  gettext.install("messages", "locale")
```

Cette commande va charger les fichiers de traduction du projet (nommés "messages") à partir du répertoire "locale", en fonction de la langue de l'utilisateur.

## Installation
Pour générer les fichiers .pot (fichiers de catalogue de traduction) à partir des fichiers source Python d'un projet, on peut utiliser la commande `xgettext` de GNU Gettext. Voici comment procéder :

1.  Assurons-nous d'avoir installé Gettext sur votre ordinateur. Si ce n'est pas le cas, nous pouvons l'installer à l'aide de la commande suivante (sous Debian ou Ubuntu) :

```bash
sudo apt-get install gettext
```

2.  Plaçons-nous dans le répertoire racine du projet (celui où se trouve le fichier `setup.py`).
    
3.  Exécutons la commande suivante :
    

```bash
xgettext --language=Python --output=locale/messages.pot `find . -name "*.py"`
```


Cette commande va parcourir tous les fichiers `.py` du projet et générer un fichier `messages.pot` dans le répertoire `locale`. Ce fichier contiendra les chaînes de caractères à traduire dans tous les fichiers source.

Nous pouvons également utiliser des outils tels que `pygettext` ou `babel` pour générer les fichiers `.pot` de votre projet. Ces outils sont spécialement conçus pour les projets Python et offrent des fonctionnalités supplémentaires, telles que la détection automatique des chaînes de caractères à traduire ou la génération de fichiers `.pot` à partir de templates de templates Django.

## Comment fait-on avec babel ?

Pour utiliser Babel pour générer les fichiers `.pot` de votre projet Python, nous devons d'abord installer l'outil en exécutant la commande suivante :

`pip install babel`

Une fois installé, plaçons-nous dans le répertoire racine de votre projet et exécutons la commande suivante :

`pybabel extract -F babel.cfg -o locale/messages.pot .`

Cette commande va parcourir tous les fichiers du projet et générer un fichier `messages.pot` dans le répertoire `locale`. Le fichier de configuration `babel.cfg` contient les options de configuration de Babel. nous pouvons en créer un en exécutant la commande `pybabel init`.

Pour plus d'informations sur l'utilisation de Babel pour la traduction de projets Python, je nous recommande de consulter la documentation officielle de Babel : [https://babel.readthedocs.io/en/latest/](https://babel.readthedocs.io/en/latest/).

Les options `-i` et `-d` de la commande `pybabel init` peuvent contenir des chemins absolus ou relatifs vers les fichiers et répertoires correspondants.

Par exemple, si nous souhaitons créer le fichier de configuration `babel.cfg` à partir d'un fichier `messages.pot` situé dans le répertoire `locale`, nous pouvons utiliser la commande suivante :

```bash
pybabel init -i locale/messages.pot -d locale -l fr
```

N'oublions pas de remplacer `fr` par la langue de base de votre projet. nous pouvons également utiliser l'option `-D` pour définir des variables de configuration dans le fichier de configuration généré, comme je nous l'ai indiqué précédemment.

## Le fichier \*.pot

Un fichier `.pot` (fichier de catalogue de traduction) contient principalement deux parties : l'entête et les entrées de traduction. L'entête du fichier est un bloc de commentaires qui se trouve en haut du fichier et qui fournit des informations sur le projet, la version du fichier `.pot` et les personnes responsables de la traduction.

Voici ce que doit contenir l'entête d'un fichier `.pot` selon la spécification de Gettext :

```pot
# SOME DESCRIPTIVE TITLE.
# Copyright (C) YEAR THE PACKAGE'S COPYRIGHT HOLDER
# This file is distributed under the same license as the PACKAGE package.
# FIRST AUTHOR <EMAIL@ADDRESS>, YEAR.
#
#, fuzzy
msgid ""
msgstr ""
"Project-Id-Version: PACKAGE VERSION\n"
"Report-Msgid-Bugs-To: EMAIL@ADDRESS\n"
"POT-Creation-Date: YEAR-MO-DA HO:MI+ZONE\n"
"PO-Revision-Date: YEAR-MO-DA HO:MI+ZONE\n"
"Last-Translator: FULL NAME <EMAIL@ADDRESS>\n"
"Language-Team: LANGUAGE <LL@li.org>\n"
"Language: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=CHARSET\n"
"Content-Transfer-Encoding: 8bit\n"
```

Explication des différents champs de l'entête :

-   `SOME DESCRIPTIVE TITLE` : titre descriptif du projet.
-   `YEAR THE PACKAGE'S COPYRIGHT HOLDER` : année de copyright du paquet.
-   `FIRST AUTHOR <EMAIL@ADDRESS>` : nom et adresse e-mail de la première personne à avoir traduit le fichier.
-   `PACKAGE VERSION` : version du paquet.
-   `EMAIL@ADDRESS` : adresse e-mail à laquelle envoyer les rapports d'erreurs.
-   `YEAR-MO-DA HO:MI+ZONE` : date de création et de modification du fichier `.pot`.
-   `FULL NAME <EMAIL@ADDRESS>` : nom et adresse e-mail de la dernière personne à avoir traduit le fichier.
-   `LANGUAGE` : langue de traduction.
-   `CHARSET` : jeu de caractères utilisé dans le fichier.

Il est important de remplir correctement ces champs afin de fournir des informations précises sur le projet et sur les personnes qui ont contribué à la traduction. Nous pouvons également ajouter d'autres champs à l'entête si nous le souhaitons, mais assurons-nous de respecter la syntaxe de Gettext.

## Signification du sous répertoire LC_MESSAGES qui est dans "locale"

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

Le répertoire `LC_MESSAGES` doit être nommé ainsi pour être reconnu par les outils de traduction. Si nous utilisons un autre nom, il se peut que les fichiers de traduction ne soient pas chargés correctement par l'application.

## Différences ente *.mo et *.po

Les fichiers `.mo` et `.po` sont utilisés par les outils de traduction, comme `gettext`, pour stocker les traductions d'un projet.

-   Les fichiers `.po` (fichiers de catalogue de traduction) sont des fichiers de texte qui contiennent les chaînes de caractères à traduire du projet, ainsi que leur traduction dans la langue cible. Ils sont générés à partir des fichiers source du projet (par exemple, des fichiers `.py` pour un projet Python) à l'aide d'outils tels que `xgettext` ou `pybabel extract`.
    
-   Les fichiers `.mo` (fichiers de message binaires) sont des fichiers binaires qui contiennent les mêmes informations que les fichiers `.po`, mais dans un format plus compact et plus rapide à charger par l'application. Ils sont générés à partir des fichiers `.po` à l'aide de l'outil `msgfmt`.

Lorsque nous traduisons un projet, nous travaillons généralement avec des fichiers `.po`, qui sont plus faciles à lire et à modifier. Une fois que nous avons terminé la traduction, nous pouvons générer les fichiers `.mo` à partir des fichiers `.po` pour que l'application puisse les charger plus rapidement.

Il est important de ne pas modifier directement les fichiers `.mo`, car ils sont au format binaire et ne sont pas lisibles par les humains. Si nous souhaitons apporter des modifications à la traduction, nous devons le faire dans les fichiers `.po` correspondants et regénérer les fichiers `.mo` à partir de ceux-ci.

# Création des fichiers po

Pour créer de nouveaux fichiers `.po` à partir d'un fichiers `.pot` (Portable Object Template), nous pouvons utiliser la commande `msginit` du package gettext. Voici comment :

1. Ouvrons un terminal et naviguons jusqu'au répertoire où se trouve votre fichier `.pot`.

2. Utilisons la commande suivante pour générer un fichier `.po` :

    ```bash
    msginit -i mydomain.pot -o mylanguage.po -l mylanguage
    ```

   Ici, `mydomain.pot` est notre fichier `.pot` source, `mylanguage.po` est le fichier `.po` que nous voulons créer et `mylanguage` est le code de langue pour lequel nous voulons créer le fichier (par exemple `fr` pour le français, `de` pour l'allemand, etc.).

Par exemple, pour générer un fichier `.po` pour le français à partir d'un fichier `alirpunkto.pot`, nous utiliserons :

```bash
cd alirpunkto/locale
msginit -i alirpunkto.pot -o en/LC_MESSAGES/alirpunkto.po -l en
msginit -i alirpunkto.pot -o fr/LC_MESSAGES/alirpunkto.po -l fr
```

Cette commande créera ou **écrasera** le fichier `fr.po` à partir du fichier `mydomain.pot`. Nous pouvons alors éditer ce fichier `fr.po` pour traduire les chaînes de caractères en français. Une fois les traductions terminées, nous pouvons générer un fichier `.mo`.

# Mettre à jour des fichiers pot et po
Si nous avons des chaînes nouvellement ajoutées dans notre code source et que nous souhaitons mettre à jour notre fichier `.pot` existant sans l'écraser, nous pouvons utiliser l'outil `xgettext` pour extraire les nouvelles chaînes, puis `msgmerge` pour fusionner ces nouvelles chaînes avec notre fichier `.pot` existant.

Voici comment le faire :

1. Utilisons `xgettext` pour extraire les nouvelles chaînes de notre code source dans un nouveau fichier `.pot`. Par exemple :

    ```bash
    xgettext -o new.pot source.py
    ```

    Ici, `new.pot` est le fichier de sortie que `xgettext` générera et `source.py` est notre code source Python. Nous devrions remplacer ces noms par ceux de notre projet. Nous pouvons également spécifier plusieurs fichiers source à la fois.

2. Ensuite, utilisons `msgmerge` pour fusionner le nouveau fichier `.pot` avec l'ancien. Par exemple :

    ```bash
    msgmerge -U old.pot new.pot
    ```

    Ici, `old.pot` est notre ancien fichier `.pot` et `new.pot` est le fichier que nous venons de générer avec `xgettext`. La commande `msgmerge` mettra à jour `old.pot` pour y inclure les nouvelles chaînes de `new.pot`.

Les anciennes traductions resteront dans `old.pot` après cette opération. Les nouvelles chaînes extraites seront ajoutées sans traductions, prêtes à être traduites.

Remarque : pour que `xgettext` fonctionne correctement, nous devons utiliser les méthodes standard de marquage des chaînes localisables dans votre code, comme `_()` ou `gettext()`.

# Génération des fichiers mo

Pour générer les fichiers `.mo` (Machine Object) à partir des fichiers `.po` (Portable Object) dans notre répertoire `locale`, nous pouvons utiliser la commande `msgfmt` du package gettext. Voici comment nous pouvons faire :

1. Ouvrons un terminal et naviguons jusqu'au répertoire `locale` de notre projet.

2. Utilisons la commande suivante pour générer le fichier `.mo` :

    ```bash
    msgfmt -o LC_MESSAGES/mydomain.mo LC_MESSAGES/mydomain.po
    ```

    Dans cette commande, remplaçons `mydomain` par le nom de domaine de notre application.

Notez que vous devez répéter ce processus pour chaque fichier `.po` dans votre projet. 

Si vous avez de nombreux fichiers `.po`, vous pouvez utiliser une boucle pour les parcourir tous. Par exemple, si vous êtes sous un système Unix-like, vous pouvez utiliser une commande `find` couplée à une boucle `for` :

```bash
find . -name "*.po" -exec msgfmt {} -o {}.mo \;
```

Cette commande trouve tous les fichiers `.po` dans le répertoire courant et ses sous-répertoires, et exécute `msgfmt` sur chaque fichier pour générer le fichier `.mo` correspondant.