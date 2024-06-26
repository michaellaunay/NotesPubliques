# Introduction au C++

## 1 Présentation du C++ : Histoire et applications

### 1.1. Histoire du C++

Le C++ est un langage de programmation de haut niveau qui a été développé par Bjarne Stroustrup au début des années 1980, au sein des laboratoires Bell de AT&T. Il est conçu comme une extension du langage C, dans le but d'ajouter des fonctionnalités orientées objet tout en conservant l'efficacité et la flexibilité du C. La première version officielle du C++ est apparue en 1983, et le langage a depuis subi de nombreuses évolutions et standardisations.

En 1998, le premier standard ISO/IEC pour le C++ a été publié, connu sous le nom de C++98. Ce standard a été suivi par plusieurs révisions majeures, notamment C++03, C++11, C++14, C++17, et plus récemment C++20. Chaque nouvelle version a apporté des améliorations significatives en termes de fonctionnalités, de performance et de sécurité.

### 1.1.2. Applications du C++

Le C++ est largement utilisé dans diverses industries en raison de sa performance, de sa flexibilité et de sa capacité à gérer des systèmes complexes. Voici quelques domaines d'application majeurs :

#### 1.1.2.1. Systèmes d'exploitation

Le C++ est souvent utilisé pour le développement de systèmes d'exploitation et de logiciels systèmes en raison de son efficacité et de son contrôle fin des ressources matérielles. Par exemple, une grande partie du système d'exploitation Windows est écrite en C++.

#### 1.1.2.2. Jeux vidéo

Le C++ est le langage de prédilection dans l'industrie des jeux vidéo. Il permet de créer des moteurs de jeu performants et de gérer des graphismes complexes en temps réel. Des moteurs de jeux populaires comme Unreal Engine et Unity sont principalement développés en C++.

#### 1.1.2.3. Logiciels embarqués

Dans les systèmes embarqués, où les ressources sont limitées et les performances critiques, le C++ est largement utilisé. Il permet de développer des logiciels pour des appareils tels que des microcontrôleurs, des systèmes de contrôle industriels et des dispositifs médicaux.

#### 1.1.2.4. Applications bancaires et financières

Les systèmes financiers requièrent une grande précision et des performances élevées pour traiter des transactions complexes. Le C++ est couramment utilisé pour développer des logiciels de trading, des systèmes de gestion des risques et des plateformes de traitement des transactions.

#### 1.1.2.5. Applications scientifiques et d'ingénierie

Le C++ est utilisé dans les domaines scientifiques et d'ingénierie pour des simulations, des modélisations et des calculs intensifs. Par exemple, il est employé dans les logiciels de calculs numériques, de simulation de fluides, et de modélisation 3D.

## 1.2. Installation de l'environnement de développement

Pour débuter avec le développement en C++ sur Ubuntu, nous allons installer et configurer l'environnement de développement. Nous utiliserons la version d'Ubuntu 24.04 (ou 22.04 à défaut) et l'éditeur de code Visual Studio Code (VS Code). Ce processus comprend plusieurs étapes : l'installation du compilateur C++, l'installation de VS Code et la configuration de l'éditeur pour le développement en C++.

### 1.2.1. Installation du compilateur C++

#### 1.2.1.1. Mise à jour du système

Avant d'installer le compilateur, il est conseillé de mettre à jour la liste des paquets et de mettre à jour les paquets existants. Ouvrons un terminal et exécutons les commandes suivantes :

```bash
sudo apt update
sudo apt upgrade
```

#### 1.2.1.2. Installation de g++

Le compilateur GNU C++ (g++) est inclus dans le paquet `build-essential` qui contient les outils nécessaires pour le développement en C et C++. Installons ce paquet avec la commande suivante :

```bash
sudo apt install build-essential
```

Nous pouvons vérifier l'installation de g++ en exécutant la commande :

```bash
g++ --version
```

Cette commande doit afficher la version de g++ installée.

### 1.2.2. Installation de Visual Studio Code

#### 1.2.2.1. Téléchargement de VS Code

Pour installer Visual Studio Code, nous devons d'abord ajouter le dépôt de Microsoft à notre système. Téléchargeons et installons la clé GPG de Microsoft avec les commandes suivantes :

```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
```

Ensuite, ajoutons le dépôt de VS Code à notre liste de sources :

```bash
sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"
```

#### 1.2.2.2. Installation de VS Code

Maintenant, nous pouvons installer Visual Studio Code en exécutant :

```bash
sudo apt update
sudo apt install code
```

#### 1.2.2.3. Vérification de l'installation

Pour vérifier que Visual Studio Code a été correctement installé, lançons l'application en tapant :

```bash
code
```

Plus de détails sont donnés dans la note [[Visual studio code]].

### 1.2.3. Configuration de Visual Studio Code pour le C++

#### 1.2.3.1. Installation des extensions nécessaires

Visual Studio Code nécessite certaines extensions pour faciliter le développement en C++. Les extensions recommandées sont "C/C++" de Microsoft et "Code Runner". Pour les installer, ouvrons VS Code, accédons à l'onglet des extensions (ou appuyons sur `Ctrl+Shift+X`), puis recherchons et installons les extensions suivantes :

- `C/C++` par Microsoft
- `Code Runner`

#### 1.2.3.2. Configuration des tâches de build

Pour compiler et exécuter des programmes C++ directement depuis VS Code, nous devons configurer les tâches de build. Créons un nouveau fichier de configuration de tâches en naviguant dans le menu `Terminal` > `Configure Tasks...` > `Create tasks.json file from template` > `Others`.

Ajoutons la configuration suivante dans le fichier `tasks.json` :

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build",
            "type": "shell",
            "command": "/usr/bin/g++",
            "args": [
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "problemMatcher": ["$gcc"],
            "detail": "Generated task by Debugger."
        }
    ]
}
```

#### 1.2.3.3. Configuration du débogueur

Pour configurer le débogueur, nous devons créer un fichier `launch.json`. Accédons au menu `Run` > `Add Configuration...`, puis sélectionnons `C++ (GDB/LLDB)` et ensuite `g++ - Build and debug active file`. Cette action générera une configuration de lancement automatique.

Le fichier `launch.json` doit contenir une configuration similaire à ceci :

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "g++ - Build and debug active file",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "build",
            "miDebuggerPath": "/usr/bin/gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "internalConsoleOptions": "openOnSessionStart"
        }
    ]
}
```

## 1.3. Structure d'un programme C++

La structure d'un programme en C++ suit une organisation spécifique qui permet au compilateur de comprendre et d'exécuter les instructions fournies par le développeur. Un programme C++ typique se compose de plusieurs éléments clés, notamment les directives de préprocesseur, les déclarations de fonctions, la fonction principale (`main`), et éventuellement des définitions de classes et des bibliothèques externes.

### 1.3.1. Directives de préprocesseur

Les directives de préprocesseur sont des instructions qui commencent par le symbole `#` et qui sont traitées par le préprocesseur avant la compilation du programme. Les directives les plus couramment utilisées sont `#include` et `#define`.

#### 1.3.1.1. `#include`

La directive `#include` est utilisée pour inclure des fichiers d'en-tête (header files) qui contiennent des déclarations de fonctions, de classes, et de constantes. Par exemple :

```cpp
#include <iostream>
```

Cette ligne inclut le fichier d'en-tête `iostream` qui permet d'utiliser les fonctions d'entrée et de sortie standard comme `std::cout` et `std::cin`.

#### 1.3.1.2. `#define`

La directive `#define` permet de définir des macros, qui sont des fragments de code pouvant être réutilisés dans le programme. Par exemple :

```cpp
#define PI 3.14159
```

Cette ligne définit une constante nommée `PI` avec la valeur `3.14159`.

### 1.3.2. Espace de noms

En C++, les espaces de noms (namespaces) sont utilisés pour organiser le code et éviter les conflits de noms. Le plus courant est le namespace `std`, qui contient toutes les fonctionnalités standard de la bibliothèque C++. Par exemple :

```cpp
using namespace std;
```

Cette ligne permet d'utiliser les éléments du namespace `std` sans avoir à le préfixer chaque fois.

### 1.3.3. Fonction principale `main`

La fonction `main` est le point d'entrée de tout programme C++. Elle a généralement la forme suivante :

```cpp
int main() {
    // code du programme
    return 0;
}
```

#### 1.3.3.1. Structure de la fonction `main`

- **Type de retour** : La fonction `main` renvoie un entier (`int`), qui est utilisé pour indiquer le succès ou l'échec de l'exécution du programme.
- **Corps de la fonction** : Le code du programme est écrit à l'intérieur des accolades `{}`.
- **Instruction `return`** : L'instruction `return 0;` indique que le programme s'est terminé avec succès.

### 1.3.4. Instructions et déclarations

À l'intérieur de la fonction `main`, nous pouvons écrire des instructions et des déclarations qui définissent le comportement du programme. Par exemple :

```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "Bonjour, le monde!" << endl;
    return 0;
}
```

#### 1.3.4.1. Déclaration de variables

Les variables doivent être déclarées avant d'être utilisées. Par exemple :

```cpp
int age = 25;
double taille = 1.75;
```

### 1.3.5. Fonctions et classes

En plus de la fonction `main`, un programme C++ peut contenir d'autres fonctions et définitions de classes pour organiser le code de manière modulaire et réutilisable.

#### 1.3.5.1. Déclaration et définition de fonctions

Une fonction est déclarée en spécifiant son type de retour, son nom, et ses paramètres. Par exemple :

```cpp
int addition(int a, int b) {
    return a + b;
}
```

#### 1.3.5.2. Déclaration et définition de classes

Les classes permettent de créer des objets qui encapsulent des données et des comportements. Par exemple :

```cpp
class Personne {
public:
    string nom;
    int age;

    void sePresenter() {
        cout << "Bonjour, je m'appelle " << nom << " et j'ai " << age << " ans." << endl;
    }
};
```

### 1.3.6. Exemple complet

Voici un exemple complet d'un programme C++ qui illustre les éléments mentionnés :

```cpp
#include <iostream>
using namespace std;

#define PI 3.14159

int addition(int a, int b) {
    return a + b;
}

class Personne {
public:
    string nom;
    int age;

    void sePresenter() {
        cout << "Bonjour, je m'appelle " << nom << " et j'ai " << age << " ans." << endl;
    }
};

int main() {
    cout << "Valeur de PI : " << PI << endl;

    int resultat = addition(5, 3);
    cout << "5 + 3 = " << resultat << endl;

    Personne p;
    p.nom = "Alice";
    p.age = 30;
    p.sePresenter();

    return 0;
}
```

## 1.4. Compilation et exécution d'un programme simple

Pour comprendre le processus de développement en C++, il est essentiel de maîtriser la compilation et l'exécution d'un programme simple. Cette section décrit les étapes nécessaires pour écrire, compiler et exécuter un programme C++ en utilisant l'environnement Ubuntu et l'éditeur Visual Studio Code (VS Code).

### 1.4.1. Écriture d'un programme simple

Commençons par écrire un programme C++ simple qui affiche un message à l'écran. Nous utiliserons VS Code pour cette tâche. 

#### 1.4.1.1. Création du fichier source

Ouvrons VS Code et créons un nouveau fichier nommé `bonjour.cpp`. Ce fichier contiendra notre code source.

#### 1.4.1.2. Écriture du code

Dans le fichier `bonjour.cpp`, écrivons le code suivant :

```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "Bonjour, le monde!" << endl;
    return 0;
}
```

Ce programme utilise la bibliothèque `iostream` pour afficher le message "Bonjour, le monde!" à l'écran. La fonction `main` est le point d'entrée du programme, et `cout` est l'objet de sortie standard.

### 1.4.2. Compilation du programme

Une fois le code écrit, nous devons le compiler pour obtenir un fichier exécutable. Sous Ubuntu, nous utilisons le compilateur g++ pour cette tâche.

#### 1.4.2.1. Compilation avec g++

Ouvrons un terminal et naviguons jusqu'au répertoire contenant notre fichier `bonjour.cpp`. Utilisons la commande suivante pour compiler le programme :

```bash
g++ bonjour.cpp -o bonjour
```

Cette commande appelle g++ pour compiler `bonjour.cpp` et génère un exécutable nommé `bonjour`. Voici une explication des options utilisées :

- `g++` : le compilateur C++ de GNU.
- `bonjour.cpp` : le fichier source à compiler.
- `-o bonjour` : spécifie le nom du fichier exécutable généré (dans ce cas, `bonjour`).

#### 1.4.2.2. Vérification de la compilation

Si la compilation réussit, g++ ne produit aucune sortie et revient à l'invite de commande. Si des erreurs de syntaxe ou d'autres erreurs sont présentes dans le code, elles seront affichées dans le terminal.

### 1.4.3. Exécution du programme

Après la compilation, nous pouvons exécuter le fichier exécutable généré.

#### 1.4.3.1. Exécution depuis le terminal

Dans le terminal, exécutons le programme avec la commande suivante :

```bash
./bonjour
```

Cette commande lance le programme `bonjour`, et le message "Bonjour, le monde!" doit s'afficher à l'écran.

### 1.4.4. Compilation et exécution avec Visual Studio Code

VS Code permet également de compiler et d'exécuter des programmes C++ directement à partir de l'éditeur, en utilisant les extensions appropriées.

#### 1.4.4.1. Configuration des tâches de build

Nous devons configurer les tâches de build dans VS Code pour compiler notre programme. Pour ce faire, créons ou modifions le fichier `tasks.json` dans le dossier `.vscode` de notre projet :

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build",
            "type": "shell",
            "command": "/usr/bin/g++",
            "args": [
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "problemMatcher": ["$gcc"],
            "detail": "Generated task by Debugger."
        }
    ]
}
```

#### 1.4.4.2. Configuration du débogueur

Pour exécuter et déboguer notre programme dans VS Code, configurons le fichier `launch.json` :

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "g++ - Build and debug active file",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "build",
            "miDebuggerPath": "/usr/bin/gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "internalConsoleOptions": "openOnSessionStart"
        }
    ]
}
```

#### 1.4.4.3. Compilation et exécution dans VS Code

Pour compiler et exécuter le programme, suivons les étapes suivantes :

1. Ouvrons le fichier `bonjour.cpp` dans VS Code.
2. Appuyons sur `F5` pour lancer le débogueur. VS Code va automatiquement compiler le programme en utilisant la tâche de build configurée, puis exécuter l'exécutable généré.
# 2. Bases du Langage

## 2.1. Types de données et variables

En C++, les types de données et les variables sont des concepts fondamentaux qui permettent de stocker et de manipuler des données. Cette section explique les différents types de données disponibles en C++, la déclaration et l'initialisation des variables, ainsi que les règles de base concernant leur utilisation.

### 2.1.1. Types de données primitifs

Les types de données primitifs en C++ sont les types de base intégrés au langage. Ils incluent les types numériques, les caractères et les booléens.

#### 2.1.1.1. Types numériques

Les types numériques sont utilisés pour représenter des nombres. Ils se divisent en deux catégories : les entiers et les nombres à virgule flottante.

- **Entiers** : Utilisés pour représenter des nombres entiers (sans partie décimale). Ils peuvent être signés (positifs et négatifs) ou non signés (uniquement positifs).

  | Type      | Taille    | Valeurs possibles                      |
  |-----------|-----------|----------------------------------------|
  | `short`   | 2 octets  | -32,768 à 32,767                       |
  | `int`     | 4 octets  | -2,147,483,648 à 2,147,483,647         |
  | `long`    | 8 octets  | -9,223,372,036,854,775,808 à 9,223,372,036,854,775,807 |
  | `unsigned short` | 2 octets | 0 à 65,535                       |
  | `unsigned int`   | 4 octets | 0 à 4,294,967,295                 |
  | `unsigned long`  | 8 octets | 0 à 18,446,744,073,709,551,615    |

- **Nombres à virgule flottante** : Utilisés pour représenter des nombres avec une partie décimale.

  | Type      | Taille    | Précision                             |
  |-----------|-----------|---------------------------------------|
  | `float`   | 4 octets  | Environ 7 chiffres significatifs      |
  | `double`  | 8 octets  | Environ 15 chiffres significatifs     |
  | `long double` | 16 octets | Environ 18 chiffres significatifs  |

#### 2.1.1.2. Caractères

Le type `char` est utilisé pour représenter des caractères individuels. En général, un `char` occupe 1 octet et peut représenter des valeurs de -128 à 127 (signé) ou de 0 à 255 (non signé).

```cpp
char lettre = 'A';
```

#### 2.1.1.3. Booléens

Le type `bool` représente des valeurs booléennes, c'est-à-dire `true` ou `false`.

```cpp
bool estVrai = true;
bool estFaux = false;
```

### 2.1.2.Types de données dérivés

En plus des types primitifs, C++ permet de créer des types de données dérivés, tels que les tableaux, les pointeurs, et les références.

#### 2.1.2.1. Tableaux

Un tableau est une collection de variables du même type, stockées en séquence. Par exemple :

```cpp
int nombres[5] = {1, 2, 3, 4, 5};
```

#### 2.1.2.2. Pointeurs

Un pointeur est une variable qui stocke l'adresse mémoire d'une autre variable. Par exemple :

```cpp
int a = 10;
int* pointeurA = &a;
```

#### 2.1.2.3. Références

Une référence est un alias pour une variable existante. Une fois initialisée, elle ne peut pas être changée pour référer une autre variable. Par exemple :

```cpp
int b = 20;
int& refB = b;
```

### 2.1.3. Variables : Déclaration et initialisation

Une variable doit être déclarée avant d'être utilisée. La déclaration d'une variable indique au compilateur le type de données et le nom de la variable.

#### 2.1.3.1. Déclaration de variables

La syntaxe de déclaration d'une variable est la suivante :

```cpp
type nomDeVariable;
```

Par exemple :

```cpp
int age;
float taille;
```

#### 2.1.3.2. Initialisation de variables

L'initialisation d'une variable consiste à lui attribuer une valeur lors de sa déclaration. Par exemple :

```cpp
int age = 25;
float taille = 1.75;
```

Il est également possible d'initialiser les variables à l'aide de la liste d'initialisation (C++11 et versions ultérieures) :

```cpp
int age {25};
float taille {1.75};
```

### 2.1.4. Portée et durée de vie des variables

La portée d'une variable est la région du code dans laquelle elle est visible et peut être utilisée. La durée de vie d'une variable est la période pendant laquelle elle existe en mémoire.

#### 2.1.4.1. Portée des variables

- **Variables locales** : Déclarées à l'intérieur d'une fonction ou d'un bloc, elles ne sont accessibles que dans cette fonction ou ce bloc.
  
  ```cpp
  void fonction() {
      int x = 10; // x est une variable locale
  }
  ```

- **Variables globales** : Déclarées en dehors de toutes les fonctions, elles sont accessibles dans tout le fichier source.

  ```cpp
  int x = 10; // x est une variable globale

  void fonction() {
      // x est accessible ici
  }
  ```

#### 2.1.4.2. Durée de vie des variables

- **Automatiques (par défaut)** : Les variables locales ont une durée de vie automatique, ce qui signifie qu'elles sont créées à l'entrée du bloc dans lequel elles sont déclarées et détruites à la sortie de ce bloc.
- **Statiques** : Les variables déclarées avec le mot-clé `static` ont une durée de vie étendue, elles existent tout au long de l'exécution du programme.

  ```cpp
  void fonction() {
      static int compteur = 0; // compteur est statique
      compteur++;
      cout << compteur << endl;
  }
  ```

## 2.2. Opérateurs de base

Les opérateurs en C++ sont des symboles qui permettent de réaliser des opérations sur des variables et des valeurs. Ils sont essentiels pour écrire des expressions arithmétiques, logiques et de manipulation de données. Cette section présente les principaux opérateurs de base en C++ et leur utilisation.

### 2.2.1. Opérateurs arithmétiques

Les opérateurs arithmétiques sont utilisés pour effectuer des opérations mathématiques de base telles que l'addition, la soustraction, la multiplication, la division et le modulo.

| Opérateur | Description          | Exemple          |
|-----------|----------------------|------------------|
| `+`       | Addition             | `a + b`          |
| `-`       | Soustraction         | `a - b`          |
| `*`       | Multiplication       | `a * b`          |
| `/`       | Division             | `a / b`          |
| `%`       | Modulo (reste)       | `a % b`          |

#### Exemples d'utilisation

```cpp
int a = 10;
int b = 3;
int somme = a + b;       // somme vaut 13
int difference = a - b;  // difference vaut 7
int produit = a * b;     // produit vaut 30
int quotient = a / b;    // quotient vaut 3
int reste = a % b;       // reste vaut 1
```

### 2.2.2. Opérateurs de comparaison

Les opérateurs de comparaison sont utilisés pour comparer deux valeurs. Ils renvoient une valeur booléenne (`true` ou `false`).

| Opérateur | Description                    | Exemple       |
|-----------|--------------------------------|---------------|
| `==`      | Égal à                         | `a == b`      |
| `!=`      | Différent de                   | `a != b`      |
| `<`       | Inférieur à                    | `a < b`       |
| `>`       | Supérieur à                    | `a > b`       |
| `<=`      | Inférieur ou égal à            | `a <= b`      |
| `>=`      | Supérieur ou égal à            | `a >= b`      |

#### Exemples d'utilisation

```cpp
int a = 10;
int b = 3;
bool estEgal = (a == b);     // estEgal vaut false
bool estDifferent = (a != b); // estDifferent vaut true
bool estInferieur = (a < b); // estInferieur vaut false
bool estSuperieur = (a > b); // estSuperieur vaut true
bool estInferieurOuEgal = (a <= b); // estInferieurOuEgal vaut false
bool estSuperieurOuEgal = (a >= b); // estSuperieurOuEgal vaut true
```

### 2.2.3. Opérateurs logiques

Les opérateurs logiques sont utilisés pour combiner des expressions booléennes. Ils permettent de créer des conditions plus complexes.

| Opérateur | Description                    | Exemple         |
|-----------|--------------------------------|-----------------|
| `&&`      | ET logique                     | `a && b`        |
| `||`      | OU logique                     | `a || b`        |
| `!`       | NON logique                    | `!a`            |

#### Exemples d'utilisation

```cpp
bool a = true;
bool b = false;
bool resultatEt = a && b;    // resultatEt vaut false
bool resultatOu = a || b;    // resultatOu vaut true
bool resultatNon = !a;       // resultatNon vaut false
```

### 2.2.4. Opérateurs d'affectation

Les opérateurs d'affectation sont utilisés pour assigner des valeurs aux variables. L'opérateur d'affectation de base est `=`, mais il existe également des opérateurs d'affectation combinée qui permettent de réaliser une opération et d'affecter le résultat en une seule étape.

| Opérateur | Description                    | Exemple         |
|-----------|--------------------------------|-----------------|
| `=`       | Affectation                    | `a = b`         |
| `+=`      | Addition puis affectation      | `a += b`        |
| `-=`      | Soustraction puis affectation  | `a -= b`        |
| `*=`      | Multiplication puis affectation| `a *= b`        |
| `/=`      | Division puis affectation      | `a /= b`        |
| `%=`      | Modulo puis affectation        | `a %= b`        |

#### Exemples d'utilisation

```cpp
int a = 10;
int b = 3;
a += b;  // a vaut 13
a -= b;  // a vaut 10
a *= b;  // a vaut 30
a /= b;  // a vaut 10
a %= b;  // a vaut 1
```

### 2.2.5. Opérateurs d'incrémentation et de décrémentation

Les opérateurs d'incrémentation et de décrémentation sont utilisés pour augmenter ou diminuer la valeur d'une variable de 1.

| Opérateur | Description          | Exemple | Équivalent     |
|-----------|----------------------|---------|----------------|
| `++`      | Incrémentation       | `a++`   | `a = a + 1`    |
| `--`      | Décrémentation       | `a--`   | `a = a - 1`    |

Il existe une différence entre les formes préfixe et suffixe de ces opérateurs :

- **Préfixe** (`++a` ou `--a`) : La valeur est modifiée avant d'être utilisée dans l'expression.
- **Suffixe** (`a++` ou `a--`) : La valeur est utilisée dans l'expression avant d'être modifiée.

#### Exemples d'utilisation

```cpp
int a = 10;
int b = ++a; // a vaut 11, b vaut 11
int c = a--; // a vaut 10, c vaut 11
```


## 2.3. Structures de contrôle : conditions et boucles

Les structures de contrôle sont des éléments fondamentaux en programmation qui permettent de diriger le flux d'exécution d'un programme en fonction de conditions spécifiques. En C++, les principales structures de contrôle incluent les conditions et les boucles. Cette section détaille leur syntaxe et leur utilisation.

### 2.3.1. Conditions

Les structures conditionnelles permettent d'exécuter des blocs de code uniquement si certaines conditions sont remplies. Les principales structures conditionnelles en C++ sont `if`, `else if`, `else`, et `switch`.

#### 2.3.1.1. if, else if, et else

La structure `if` permet d'exécuter un bloc de code si une condition est vraie. Les structures `else if` et `else` permettent de gérer plusieurs conditions alternatives.

**Syntaxe :**

```cpp
if (condition) {
    // code à exécuter si la condition est vraie
} else if (autre_condition) {
    // code à exécuter si autre_condition est vraie
} else {
    // code à exécuter si aucune des conditions précédentes n'est vraie
}
```

**Exemple :**

```cpp
int age = 20;

if (age < 18) {
    cout << "Vous êtes mineur." << endl;
} else if (age == 18) {
    cout << "Vous venez d'avoir 18 ans." << endl;
} else {
    cout << "Vous êtes majeur." << endl;
}
```

#### 2.3.1.2. switch

La structure `switch` permet de sélectionner parmi plusieurs blocs de code à exécuter en fonction de la valeur d'une expression.

**Syntaxe :**

```cpp
switch (expression) {
    case valeur1:
        // code à exécuter si expression == valeur1
        break;
    case valeur2:
        // code à exécuter si expression == valeur2
        break;
    // ...
    default:
        // code à exécuter si aucune des valeurs précédentes ne correspond
}
```

**Exemple :**

```cpp
int jour = 3;

switch (jour) {
    case 1:
        cout << "Lundi" << endl;
        break;
    case 2:
        cout << "Mardi" << endl;
        break;
    case 3:
        cout << "Mercredi" << endl;
        break;
    default:
        cout << "Jour inconnu" << endl;
}
```

### 2.3.2. Boucles

Les boucles permettent d'exécuter un bloc de code plusieurs fois, en fonction d'une condition. Les principales boucles en C++ sont `for`, `while`, et `do-while`.

#### 2.3.2.1. for

La boucle `for` est utilisée pour itérer sur une séquence d'instructions un nombre spécifique de fois.

**Syntaxe :**

```cpp
for (initialisation; condition; incrément) {
    // code à exécuter à chaque itération
}
```

**Exemple :**

```cpp
for (int i = 0; i < 5; i++) {
    cout << "i vaut " << i << endl;
}
```

#### 2.3.2.2. while

La boucle `while` exécute un bloc de code tant qu'une condition est vraie.

**Syntaxe :**

```cpp
while (condition) {
    // code à exécuter tant que la condition est vraie
}
```

**Exemple :**

```cpp
int i = 0;

while (i < 5) {
    cout << "i vaut " << i << endl;
    i++;
}
```

#### 2.3.2.3. do-while

La boucle `do-while` est similaire à la boucle `while`, mais elle garantit que le bloc de code est exécuté au moins une fois avant que la condition ne soit testée.

**Syntaxe :**

```cpp
do {
    // code à exécuter
} while (condition);
```

**Exemple :**

```cpp
int i = 0;

do {
    cout << "i vaut " << i << endl;
    i++;
} while (i < 5);
```

### 2.3.3. Utilisation de break et continue

Les instructions `break` et `continue` sont utilisées pour contrôler le flux des boucles.

#### 2.3.3.1. break

L'instruction `break` permet de sortir prématurément d'une boucle ou d'un `switch`.

**Exemple :**

```cpp
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break;
    }
    cout << "i vaut " << i << endl;
}
```

#### 2.3.3.2. continue

L'instruction `continue` permet de passer immédiatement à l'itération suivante de la boucle, en sautant le code restant pour l'itération en cours.

**Exemple :**

```cpp
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        continue;
    }
    cout << "i vaut " << i << endl;
}
```

# Module 2 : Bases du Langage

## 2.4 Fonctions : Déclaration, définition et appel

Les fonctions en C++ sont des blocs de code réutilisables qui permettent de réaliser des tâches spécifiques. Elles facilitent la structuration, la lisibilité et la maintenance du code. Cette section explique comment déclarer, définir et appeler des fonctions en C++.

### 2.4.1 Déclaration de fonctions

La déclaration d'une fonction indique au compilateur l'existence de cette fonction et son prototype, c'est-à-dire son type de retour et ses paramètres. La déclaration d'une fonction se fait généralement dans un fichier d'en-tête (.h) ou au début d'un fichier source (.cpp).

**Syntaxe :**

```cpp
type_de_retour nom_de_fonction(liste_de_paramètres);
```

**Exemple :**

```cpp
int addition(int a, int b);
void afficherMessage();
```

### 2.4.2 Définition de fonctions

La définition d'une fonction fournit le corps de la fonction, c'est-à-dire le code qui sera exécuté lorsque la fonction sera appelée. La définition d'une fonction se fait généralement dans un fichier source (.cpp).

**Syntaxe :**

```cpp
type_de_retour nom_de_fonction(liste_de_paramètres) {
    // corps de la fonction
}
```

**Exemple :**

```cpp
int addition(int a, int b) {
    return a + b;
}

void afficherMessage() {
    cout << "Bonjour, le monde!" << endl;
}
```

### 2.4.3 Appel de fonctions

L'appel d'une fonction exécute le code défini dans cette fonction. Pour appeler une fonction, il suffit d'utiliser son nom suivi de parenthèses contenant les arguments nécessaires.

**Syntaxe :**

```cpp
nom_de_fonction(arguments);
```

**Exemple :**

```cpp
int resultat = addition(5, 3);
afficherMessage();
```

### 2.4.4 Fonctions avec paramètres

Les paramètres d'une fonction permettent de passer des valeurs à cette fonction pour qu'elle les utilise dans son traitement. Les paramètres sont spécifiés dans la liste de paramètres lors de la déclaration et de la définition de la fonction.

**Exemple :**

```cpp
int multiplier(int a, int b) {
    return a * b;
}

int resultat = multiplier(4, 5); // résultat vaut 20
```

### 2.4.5 Fonctions sans paramètres

Une fonction peut ne pas nécessiter de paramètres. Dans ce cas, les parenthèses sont laissées vides.

**Exemple :**

```cpp
void direBonjour() {
    cout << "Bonjour!" << endl;
}

direBonjour();
```

### 2.4.6 Fonctions avec valeurs de retour

Une fonction peut renvoyer une valeur à l'aide de l'instruction `return`. Le type de la valeur de retour doit correspondre au type de retour spécifié dans la déclaration de la fonction.

**Exemple :**

```cpp
double calculerMoyenne(double note1, double note2) {
    return (note1 + note2) / 2;
}

double moyenne = calculerMoyenne(15.5, 18.0); // moyenne vaut 16.75
```

### 2.4.7 Fonctions void

Les fonctions qui ne renvoient aucune valeur utilisent le type de retour `void`.

**Exemple :**

```cpp
void afficherBienvenue() {
    cout << "Bienvenue dans notre programme!" << endl;
}

afficherBienvenue();
```

### 2.4.8 Surcharge de fonctions

La surcharge de fonctions permet de définir plusieurs fonctions avec le même nom mais des listes de paramètres différentes. Le compilateur détermine quelle fonction appeler en fonction des arguments fournis.

**Exemple :**

```cpp
int addition(int a, int b) {
    return a + b;
}

double addition(double a, double b) {
    return a + b;
}

int resultatInt = addition(2, 3);    // Appelle la version int de addition
double resultatDouble = addition(2.5, 3.5); // Appelle la version double de addition
```

### 2.4.9 Fonctions récursives

Une fonction récursive est une fonction qui s'appelle elle-même. La récursion est utile pour résoudre des problèmes qui peuvent être divisés en sous-problèmes plus petits de nature similaire.

**Exemple :**

```cpp
int factorielle(int n) {
    if (n <= 1) {
        return 1;
    } else {
        return n * factorielle(n - 1);
    }
}

int resultat = factorielle(5); // résultat vaut 120
```

# 3. Programmation Orientée Objet (POO)

## 3.1. Concepts de base de la POO : Classes et objets

La Programmation Orientée Objet (POO) est un paradigme de programmation qui organise le code en objets, chacun représentant une instance d'une classe. Une classe est un modèle définissant les propriétés (attributs) et les comportements (méthodes) de ces objets. Ce modèle permet de structurer le code de manière modulaire et réutilisable. Dans cette section, nous détaillons les concepts de base de la POO en C++, notamment les classes et les objets.

### 3.1.1. Classes

Une classe est une structure de données définissant un ensemble de propriétés et de méthodes. Les propriétés représentent les données de l'objet, tandis que les méthodes définissent les comportements que l'objet peut réaliser.

#### 3.1.1.1. Déclaration d'une classe

La déclaration d'une classe en C++ se fait à l'aide du mot-clé `class`, suivi du nom de la classe et des accolades englobant ses membres.

**Syntaxe :**

```cpp
class NomDeClasse {
public:
    // Propriétés (attributs)
    // Méthodes
};
```

**Exemple :**

```cpp
class Personne {
public:
    string nom;
    int age;

    void sePresenter() {
        cout << "Bonjour, je m'appelle " << nom << " et j'ai " << age << " ans." << endl;
    }
};
```

Dans cet exemple, `Personne` est une classe avec deux propriétés (`nom` et `age`) et une méthode (`sePresenter`).

#### 3.1.1.2. Membres publics et privés

Les membres d'une classe peuvent être publics (`public`) ou privés (`private`). Les membres publics sont accessibles depuis l'extérieur de la classe, tandis que les membres privés ne le sont pas.

**Exemple :**

```cpp
class Personne {
private:
    string nom;
    int age;

public:
    void definirNom(string n) {
        nom = n;
    }

    void definirAge(int a) {
        age = a;
    }

    void sePresenter() {
        cout << "Bonjour, je m'appelle " << nom << " et j'ai " << age << " ans." << endl;
    }
};
```

Dans cet exemple, `nom` et `age` sont privés et ne peuvent être modifiés directement de l'extérieur de la classe. Les méthodes `definirNom` et `definirAge` permettent de définir ces attributs de manière contrôlée.

### 3.1.2. Objets

Un objet est une instance d'une classe. Il représente une entité spécifique avec les propriétés et comportements définis par sa classe.

#### 3.1.2.1. Création d'objets

Pour créer un objet, nous déclarons une variable de type de la classe et utilisons le constructeur de la classe.

**Syntaxe :**

```cpp
NomDeClasse nomDeLObjet;
```

**Exemple :**

```cpp
Personne p1;
p1.definirNom("Alice");
p1.definirAge(30);
p1.sePresenter();
```

Dans cet exemple, `p1` est un objet de la classe `Personne`. Nous utilisons les méthodes de la classe pour définir ses propriétés et l'objet peut ensuite appeler sa méthode `sePresenter`.

### 3.1.3. Constructeurs et destructeurs

Les constructeurs et destructeurs sont des fonctions spéciales de la classe qui sont appelées lors de la création et de la destruction d'un objet.

#### 3.1.3.1. Constructeur

Le constructeur initialise les objets de la classe. Il a le même nom que la classe et ne retourne aucune valeur.

**Exemple :**

```cpp
class Personne {
private:
    string nom;
    int age;

public:
    Personne(string n, int a) {
        nom = n;
        age = a;
    }

    void sePresenter() {
        cout << "Bonjour, je m'appelle " << nom << " et j'ai " << age << " ans." << endl;
    }
};

Personne p1("Alice", 30);
p1.sePresenter();
```

Dans cet exemple, le constructeur `Personne` initialise les propriétés `nom` et `age` lors de la création de l'objet `p1`.

#### 3.1.3.2. Destructeur

Le destructeur nettoie les ressources utilisées par l'objet avant sa destruction. Il a le même nom que la classe, précédé d'un tilde (`~`), et ne retourne aucune valeur.

**Exemple :**

```cpp
class Personne {
private:
    string nom;
    int age;

public:
    Personne(string n, int a) : nom(n), age(a) {}

    ~Personne() {
        cout << "Destruction de l'objet Personne." << endl;
    }

    void sePresenter() {
        cout << "Bonjour, je m'appelle " << nom << " et j'ai " << age << " ans." << endl;
    }
};

Personne p1("Alice", 30);
p1.sePresenter();
```

Dans cet exemple, le destructeur `~Personne` affiche un message lors de la destruction de l'objet `p1`.

### 3.1.4. Méthodes

Les méthodes sont des fonctions définies à l'intérieur d'une classe et opérant sur les objets de cette classe.

#### 3.1.4.1. Méthodes de classe

Les méthodes de classe sont définies dans la classe et peuvent accéder à ses membres.

**Exemple :**

```cpp
class CompteBancaire {
private:
    double solde;

public:
    CompteBancaire(double montantInitial) : solde(montantInitial) {}

    void deposer(double montant) {
        solde += montant;
    }

    void retirer(double montant) {
        if (montant <= solde) {
            solde -= montant;
        } else {
            cout << "Fonds insuffisants." << endl;
        }
    }

    void afficherSolde() {
        cout << "Solde: " << solde << " euros." << endl;
    }
};

CompteBancaire compte(100.0);
compte.deposer(50.0);
compte.retirer(30.0);
compte.afficherSolde();
```

Dans cet exemple, la classe `CompteBancaire` a des méthodes pour déposer et retirer de l'argent, ainsi qu'une méthode pour afficher le solde actuel.

## 3.2. Encapsulation : Accesseurs et mutateurs

L'encapsulation est un principe fondamental de la Programmation Orientée Objet (POO) qui consiste à regrouper les données (attributs) et les méthodes qui les manipulent dans une seule unité appelée classe. Ce principe permet de protéger les données en contrôlant leur accès et en garantissant leur intégrité. Pour ce faire, on utilise des accesseurs et des mutateurs, également appelés getters et setters.

### 3.2.1. Accesseurs (Getters)

Les accesseurs sont des méthodes qui permettent d'obtenir la valeur des attributs privés d'une classe. Ils fournissent un accès contrôlé aux données et sont généralement déclarés comme des méthodes publiques.

**Syntaxe :**

```cpp
type_de_retour getNomDeLAttribut() const;
```

Le mot-clé `const` après la déclaration de la méthode indique que cette méthode ne modifie pas l'état de l'objet.

**Exemple :**

```cpp
class Personne {
private:
    string nom;
    int age;

public:
    string getNom() const {
        return nom;
    }

    int getAge() const {
        return age;
    }
};
```

Dans cet exemple, `getNom` et `getAge` sont des accesseurs qui permettent d'obtenir la valeur des attributs privés `nom` et `age`.

### 3.2.2. Mutateurs (Setters)

Les mutateurs sont des méthodes qui permettent de modifier la valeur des attributs privés d'une classe. Ils fournissent un moyen contrôlé de modifier les données et sont également généralement déclarés comme des méthodes publiques.

**Syntaxe :**

```cpp
void setNomDeLAttribut(type_de_l_attribut nouvelle_valeur);
```

**Exemple :**

```cpp
class Personne {
private:
    string nom;
    int age;

public:
    void setNom(const string& nouveauNom) {
        nom = nouveauNom;
    }

    void setAge(int nouvelAge) {
        if (nouvelAge > 0) { // Validation de l'âge
            age = nouvelAge;
        }
    }
};
```

Dans cet exemple, `setNom` et `setAge` sont des mutateurs qui permettent de modifier la valeur des attributs privés `nom` et `age`. La méthode `setAge` inclut une validation pour s'assurer que l'âge est positif.

### 3.2.3. Avantages de l'encapsulation

L'encapsulation offre plusieurs avantages importants :

- **Protection des données** : En rendant les attributs privés et en contrôlant l'accès à ces attributs par des méthodes publiques, l'encapsulation protège les données contre les modifications non autorisées.
- **Validation des données** : Les mutateurs permettent d'ajouter des vérifications et des validations lors de la modification des attributs, garantissant ainsi que les données restent cohérentes et valides.
- **Modularité et maintenabilité** : L'encapsulation permet de modifier la mise en œuvre interne d'une classe sans affecter le code qui utilise cette classe. Cela facilite la maintenance et l'évolution du code.

### 3.2.4. Exemple complet

Voici un exemple complet illustrant l'utilisation des accesseurs et des mutateurs dans une classe `Personne` :

```cpp
#include <iostream>
using namespace std;

class Personne {
private:
    string nom;
    int age;

public:
    // Constructeur
    Personne(const string& nom, int age) {
        this->nom = nom;
        this->age = age;
    }

    // Accesseurs
    string getNom() const {
        return nom;
    }

    int getAge() const {
        return age;
    }

    // Mutateurs
    void setNom(const string& nouveauNom) {
        nom = nouveauNom;
    }

    void setAge(int nouvelAge) {
        if (nouvelAge > 0) {
            age = nouvelAge;
        }
    }

    // Méthode pour afficher les informations
    void sePresenter() const {
        cout << "Bonjour, je m'appelle " << nom << " et j'ai " << age << " ans." << endl;
    }
};

int main() {
    // Création d'un objet Personne
    Personne p("Alice", 30);

    // Utilisation des accesseurs
    cout << "Nom : " << p.getNom() << endl;
    cout << "Age : " << p.getAge() << endl;

    // Utilisation des mutateurs
    p.setNom("Bob");
    p.setAge(25);

    // Affichage des nouvelles informations
    p.sePresenter();

    return 0;
}
```

Dans cet exemple, la classe `Personne` utilise des accesseurs pour lire les valeurs des attributs privés et des mutateurs pour les modifier. Le programme crée un objet `Personne`, utilise les accesseurs pour afficher les valeurs initiales, utilise les mutateurs pour modifier les valeurs, et affiche les nouvelles valeurs en utilisant une méthode `sePresenter`.

## 3.3. Héritage : Base et dérivée classes

L'héritage est un concept fondamental de la Programmation Orientée Objet (POO) qui permet de créer de nouvelles classes basées sur des classes existantes. Il facilite la réutilisation du code et favorise une structure hiérarchique et modulaire. En C++, une classe qui hérite d'une autre est appelée classe dérivée, tandis que la classe dont elle hérite est appelée classe de base.

### 3.3.1. Classe de base

Une classe de base est une classe existante à partir de laquelle d'autres classes peuvent être dérivées. Elle contient des attributs et des méthodes qui peuvent être partagés avec les classes dérivées.

**Exemple :**

```cpp
class Personne {
protected:
    string nom;
    int age;

public:
    Personne(const string& nom, int age) : nom(nom), age(age) {}

    void sePresenter() const {
        cout << "Bonjour, je m'appelle " << nom << " et j'ai " << age << " ans." << endl;
    }
};
```

Dans cet exemple, `Personne` est une classe de base avec deux attributs protégés (`nom` et `age`) et une méthode publique (`sePresenter`).

### 3.3.2. Classe dérivée

Une classe dérivée est une classe qui hérite des attributs et des méthodes d'une classe de base. Elle peut également avoir ses propres attributs et méthodes supplémentaires.

#### 3.3.2.1. Déclaration d'une classe dérivée

Pour déclarer une classe dérivée en C++, nous utilisons la syntaxe suivante :

```cpp
class NomDeClasseDerivee : public NomDeClasseDeBase {
    // Membres supplémentaires de la classe dérivée
};
```

**Exemple :**

```cpp
class Etudiant : public Personne {
private:
    string universite;

public:
    Etudiant(const string& nom, int age, const string& universite) : Personne(nom, age), universite(universite) {}

    void afficherUniversite() const {
        cout << "Je suis étudiant à " << universite << "." << endl;
    }
};
```

Dans cet exemple, `Etudiant` est une classe dérivée de `Personne`. Elle hérite des attributs et méthodes de `Personne` et ajoute un attribut privé (`universite`) ainsi qu'une méthode publique (`afficherUniversite`).

### 3.3.3. Accès aux membres de la classe de base

Les membres de la classe de base peuvent être accessibles dans la classe dérivée en fonction de leur niveau de protection.

- **Public** : Les membres publics de la classe de base sont accessibles dans la classe dérivée.
- **Protected** : Les membres protégés de la classe de base sont accessibles dans la classe dérivée.
- **Private** : Les membres privés de la classe de base ne sont pas accessibles directement dans la classe dérivée.

**Exemple :**

```cpp
int main() {
    Etudiant etudiant("Alice", 20, "Université de Paris");

    etudiant.sePresenter(); // Appel de la méthode héritée de Personne
    etudiant.afficherUniversite(); // Appel de la méthode de Etudiant

    return 0;
}
```

### 3.3.4. Constructeurs et destructeurs dans les classes dérivées

Les constructeurs et destructeurs des classes dérivées doivent appeler les constructeurs et destructeurs de la classe de base pour initialiser et nettoyer correctement les objets.

#### 3.3.4.1. Constructeurs

Le constructeur de la classe dérivée appelle le constructeur de la classe de base pour initialiser les attributs hérités.

**Exemple :**

```cpp
class Etudiant : public Personne {
private:
    string universite;

public:
    Etudiant(const string& nom, int age, const string& universite) : Personne(nom, age), universite(universite) {}
};
```

#### 3.3.4.2. Destructeurs

Le destructeur de la classe dérivée appelle automatiquement le destructeur de la classe de base.

**Exemple :**

```cpp
class Personne {
public:
    Personne(const string& nom, int age) : nom(nom), age(age) {}

    ~Personne() {
        cout << "Destruction de la personne." << endl;
    }
};

class Etudiant : public Personne {
public:
    Etudiant(const string& nom, int age, const string& universite) : Personne(nom, age), universite(universite) {}

    ~Etudiant() {
        cout << "Destruction de l'étudiant." << endl;
    }
};
```

### 3.3.5. Surcharge et redéfinition de méthodes

Les classes dérivées peuvent redéfinir les méthodes de la classe de base pour fournir une implémentation spécifique.

#### 3.3.5.1. Redéfinition de méthodes

Pour redéfinir une méthode, il suffit de déclarer une méthode avec la même signature dans la classe dérivée.

**Exemple :**

```cpp
class Personne {
public:
    virtual void sePresenter() const {
        cout << "Bonjour, je suis une personne." << endl;
    }
};

class Etudiant : public Personne {
public:
    void sePresenter() const override {
        cout << "Bonjour, je suis un étudiant." << endl;
    }
};
```

Dans cet exemple, la méthode `sePresenter` de la classe dérivée `Etudiant` redéfinit la méthode de la classe de base `Personne`.

### 3.3.6. Héritage multiple

C++ supporte l'héritage multiple, c'est-à-dire qu'une classe dérivée peut hériter de plusieurs classes de base.

**Syntaxe :**

```cpp
class NomDeClasseDerivee : public ClasseDeBase1, public ClasseDeBase2 {
    // Membres de la classe dérivée
};
```

**Exemple :**

```cpp
class Sportif {
public:
    void faireSport() const {
        cout << "Je fais du sport." << endl;
    }
};

class EtudiantSportif : public Etudiant, public Sportif {
public:
    EtudiantSportif(const string& nom, int age, const string& universite) : Etudiant(nom, age, universite) {}
};
```

Dans cet exemple, `EtudiantSportif` hérite à la fois de `Etudiant` et de `Sportif`, combinant les fonctionnalités des deux classes de base.

## 3.4. Polymorphisme : Fonctions virtuelles et classes abstraites

Le polymorphisme est un concept clé de la Programmation Orientée Objet (POO) qui permet aux objets de différentes classes d'être traités de manière uniforme grâce à une interface commune. En C++, le polymorphisme est principalement implémenté à l'aide des fonctions virtuelles et des classes abstraites.

### 3.4.1. Fonctions virtuelles

Les fonctions virtuelles permettent à une classe dérivée de redéfinir une fonction membre de la classe de base tout en permettant le comportement polymorphique. Lorsqu'une fonction membre est déclarée virtuelle dans une classe de base, la version de la fonction qui est exécutée est déterminée par le type de l'objet pointé, et non par le type du pointeur.

#### 3.4.1.1. Déclaration d'une fonction virtuelle

Une fonction virtuelle est déclarée dans la classe de base avec le mot-clé `virtual`.

**Syntaxe :**

```cpp
class ClasseDeBase {
public:
    virtual void fonctionVirtuelle() const {
        // implémentation de base
    }
};
```

**Exemple :**

```cpp
class Personne {
public:
    virtual void sePresenter() const {
        cout << "Bonjour, je suis une personne." << endl;
    }
};

class Etudiant : public Personne {
public:
    void sePresenter() const override {
        cout << "Bonjour, je suis un étudiant." << endl;
    }
};
```

#### 3.4.1.2. Utilisation de fonctions virtuelles

Lorsqu'une fonction virtuelle est appelée via un pointeur ou une référence à la classe de base, la version de la fonction qui est exécutée est celle de la classe dérivée à laquelle l'objet appartient.

**Exemple :**

```cpp
int main() {
    Personne* p = new Etudiant();
    p->sePresenter(); // Affiche "Bonjour, je suis un étudiant."
    delete p;
    return 0;
}
```

Dans cet exemple, bien que `p` soit un pointeur de type `Personne`, la fonction `sePresenter` de la classe `Etudiant` est appelée grâce au mécanisme des fonctions virtuelles.

### 3.4.2. Classes abstraites

Une classe abstraite est une classe qui ne peut pas être instanciée directement. Elle est destinée à être une classe de base pour d'autres classes. Une classe abstraite contient au moins une fonction virtuelle pure. Une fonction virtuelle pure est une fonction déclarée avec `= 0`.

#### 3.4.2.1. Déclaration d'une classe abstraite

**Syntaxe :**

```cpp
class ClasseAbstraite {
public:
    virtual void fonctionVirtuellePure() const = 0;
};
```

**Exemple :**

```cpp
class Animal {
public:
    virtual void parler() const = 0; // Fonction virtuelle pure
};
```

#### 3.4.2.2. Implémentation d'une classe dérivée

Une classe dérivée doit implémenter toutes les fonctions virtuelles pures de la classe abstraite pour être instanciable.

**Exemple :**

```cpp
class Chien : public Animal {
public:
    void parler() const override {
        cout << "Le chien aboie." << endl;
    }
};

class Chat : public Animal {
public:
    void parler() const override {
        cout << "Le chat miaule." << endl;
    }
};
```

#### 3.4.2.3. Utilisation de classes abstraites

Les classes abstraites permettent de définir une interface commune pour un ensemble de classes dérivées.

**Exemple :**

```cpp
int main() {
    Animal* a1 = new Chien();
    Animal* a2 = new Chat();

    a1->parler(); // Affiche "Le chien aboie."
    a2->parler(); // Affiche "Le chat miaule."

    delete a1;
    delete a2;

    return 0;
}
```

### 3.4.3. Polymorphisme et tables des virtualités (vtable)

Le polymorphisme en C++ est généralement implémenté à l'aide de tables de virtualités (vtable). Une vtable est une structure de données utilisée par le compilateur pour prendre en charge les fonctions virtuelles et le polymorphisme. Elle contient des pointeurs vers les fonctions virtuelles de la classe.

Lorsqu'un objet est créé, un pointeur vers la vtable de sa classe est placé dans l'objet. Lorsque la fonction virtuelle est appelée, la vtable est consultée pour déterminer quelle version de la fonction doit être exécutée.

# 4. Gestion de la Mémoire

## 4.1. Allocation dynamique de la mémoire

L'allocation dynamique de la mémoire est une technique essentielle en programmation qui permet de gérer efficacement la mémoire durant l'exécution d'un programme. Contrairement à l'allocation statique, où la taille de la mémoire est déterminée à la compilation, l'allocation dynamique permet de réserver et de libérer de la mémoire à la demande pendant l'exécution. En C++, cela se fait principalement à l'aide des opérateurs `new` et `delete`.

### 4.1.1. Opérateur `new`

L'opérateur `new` alloue de la mémoire sur le tas (heap) et retourne un pointeur vers le type spécifié. Si l'allocation échoue, une exception `std::bad_alloc` est lancée.

#### Syntaxe

```cpp
type* pointeur = new type;
```

**Exemple :**

```cpp
int* ptr = new int;        // Alloue de la mémoire pour un entier
*ptr = 42;                 // Assigne la valeur 42 à cette mémoire
cout << *ptr << endl;      // Affiche 42
```

Pour allouer un tableau de taille dynamique :

```cpp
int taille = 10;
int* tableau = new int[taille];  // Alloue de la mémoire pour un tableau de 10 entiers
```

### 4.1.2. Opérateur `delete`

L'opérateur `delete` libère la mémoire allouée par `new`. Ne pas libérer la mémoire allouée dynamiquement conduit à des fuites de mémoire, ce qui peut réduire les performances et épuiser les ressources mémoire.

#### Syntaxe

```cpp
delete pointeur;
delete[] pointeurTableau;
```

**Exemple :**

```cpp
delete ptr;                // Libère la mémoire allouée pour l'entier
delete[] tableau;          // Libère la mémoire allouée pour le tableau d'entiers
```

### 4.1.3. Allocation dynamique pour les objets

L'opérateur `new` peut également être utilisé pour allouer de la mémoire pour des objets, en appelant le constructeur de l'objet.

**Exemple :**

```cpp
class Personne {
public:
    string nom;
    Personne(const string& nom) : nom(nom) {}
    void sePresenter() const {
        cout << "Bonjour, je m'appelle " << nom << "." << endl;
    }
};

Personne* p = new Personne("Alice");
p->sePresenter();          // Affiche "Bonjour, je m'appelle Alice."
delete p;                  // Libère la mémoire allouée pour l'objet
```

Pour un tableau d'objets :

```cpp
Personne* personnes = new Personne[3] {{"Alice"}, {"Bob"}, {"Charlie"}};
for (int i = 0; i < 3; i++) {
    personnes[i].sePresenter();
}
delete[] personnes;        // Libère la mémoire allouée pour le tableau d'objets
```

### 4.1.4. Utilisation de `std::unique_ptr` et `std::shared_ptr`

C++11 a introduit des pointeurs intelligents (smart pointers) dans la bibliothèque standard pour gérer automatiquement l'allocation et la libération de la mémoire, réduisant ainsi les risques de fuites de mémoire et de problèmes de gestion de la mémoire.

#### `std::unique_ptr`

Un `std::unique_ptr` possède une ressource et s'assure que la mémoire est libérée lorsqu'il n'est plus nécessaire.

**Exemple :**

```cpp
#include <memory>

std::unique_ptr<int> ptr = std::make_unique<int>(42);
cout << *ptr << endl;      // Affiche 42
```

#### `std::shared_ptr`

Un `std::shared_ptr` permet à plusieurs pointeurs de partager la même ressource. La mémoire est libérée lorsque le dernier `std::shared_ptr` qui possède la ressource est détruit.

**Exemple :**

```cpp
#include <memory>

std::shared_ptr<int> ptr1 = std::make_shared<int>(42);
std::shared_ptr<int> ptr2 = ptr1;   // ptr2 partage la même ressource que ptr1
cout << *ptr1 << " " << *ptr2 << endl;  // Affiche 42 42
```

### 4.1.5. Avantages et inconvénients de l'allocation dynamique

#### Avantages

- **Flexibilité** : Permet de gérer la mémoire de manière dynamique, en allouant et libérant des blocs de mémoire en fonction des besoins de l'application.
- **Efficacité** : Permet d'optimiser l'utilisation de la mémoire, en allouant exactement la quantité nécessaire.

#### Inconvénients

- **Complexité** : La gestion de la mémoire dynamique peut introduire des bugs difficiles à détecter, comme des fuites de mémoire et des accès à des zones de mémoire non valides.
- **Performance** : L'allocation et la désallocation de mémoire dynamique peuvent être coûteuses en termes de performance, surtout si elles sont fréquemment utilisées.

### 4.1.6. Remarques

L'allocation dynamique de la mémoire est un point d’achoppement de la programmation en C++, qui mène à beaucoup de faille et CoreDump. Toutefois, elle permet de gérer efficacement la mémoire durant l'exécution du programme, offrant ainsi une grande flexibilité, mais, elle requiert une gestion attentive pour éviter les fuites de mémoire et les erreurs d'accès. Les pointeurs intelligents introduits dans C++11 simplifient grandement cette gestion en automatisant la libération des ressources

## 4.2. Pointeurs et références

Les pointeurs et les références sont des concepts fondamentaux en C++ qui permettent de manipuler directement la mémoire. Ils offrent une grande flexibilité et efficacité, mais nécessitent une compréhension approfondie pour éviter les erreurs courantes telles que les fuites de mémoire, les accès illégaux et la corruption de mémoire.

### 4.2.1. Pointeurs

Un pointeur est une variable qui stocke l'adresse mémoire d'une autre variable. Les pointeurs permettent de manipuler directement les adresses mémoire, facilitant ainsi l'allocation dynamique, la manipulation de tableaux et la gestion des ressources.

#### 4.2.1.1. Déclaration et initialisation des pointeurs

**Syntaxe :**

```cpp
type* nomDuPointeur;
```

**Exemple :**

```cpp
int a = 10;
int* pointeurA = &a; // pointeurA contient l'adresse de a
```

#### 4.2.1.2. Accès et modification des valeurs pointées

Pour accéder à la valeur pointée par un pointeur, nous utilisons l'opérateur de déréférencement `*`.

**Exemple :**

```cpp
cout << *pointeurA << endl; // Affiche 10
*pointeurA = 20;
cout << a << endl; // Affiche 20
```

#### 4.2.1.3. Pointeurs et tableaux

Les pointeurs sont étroitement liés aux tableaux. Le nom d'un tableau est un pointeur constant vers son premier élément.

**Exemple :**

```cpp
int tableau[5] = {1, 2, 3, 4, 5};
int* pointeurTableau = tableau;
cout << *(pointeurTableau + 1) << endl; // Affiche 2
```

#### 4.2.1.4. Allocation dynamique avec `new` et `delete`

Les pointeurs sont utilisés pour gérer la mémoire allouée dynamiquement.

**Exemple :**

```cpp
int* ptr = new int(10); // Alloue de la mémoire pour un entier et initialise à 10
delete ptr; // Libère la mémoire allouée
```

Pour les tableaux dynamiques :

```cpp
int* tableauDynamique = new int[5];
delete[] tableauDynamique; // Libère la mémoire allouée pour le tableau
```

### 4.2.2. Références

Une référence est un alias pour une variable existante. Une fois qu'une référence est initialisée, elle ne peut pas être modifiée pour référer une autre variable.

#### 4.2.2.1. Déclaration et initialisation des références

**Syntaxe :**

```cpp
type& nomDeLaReference = variable;
```

**Exemple :**

```cpp
int a = 10;
int& refA = a; // refA est une référence à a
```

#### 4.2.2.2. Utilisation des références

Les références permettent de manipuler directement la variable à laquelle elles sont liées.

**Exemple :**

```cpp
refA = 20;
cout << a << endl; // Affiche 20
```

#### 4.2.2.3. Références comme paramètres de fonction

Les références sont souvent utilisées comme paramètres de fonction pour éviter la copie des arguments et pour permettre à la fonction de modifier les arguments originaux.

**Exemple :**

```cpp
void incrementer(int& ref) {
    ref++;
}

int main() {
    int x = 10;
    incrementer(x);
    cout << x << endl; // Affiche 11
    return 0;
}
```

#### 4.2.2.4. Références constantes

Les références constantes permettent de référencer une variable sans pouvoir la modifier, ce qui est utile pour les paramètres de fonction qui ne doivent pas être modifiés.

**Exemple :**

```cpp
void afficher(const int& ref) {
    cout << ref << endl;
}

int main() {
    int y = 30;
    afficher(y); // Affiche 30
    return 0;
}
```

### 4.2.3. Comparaison entre pointeurs et références

#### 4.2.3.1. Flexibilité

- **Pointeurs** : Peuvent être réaffectés pour pointer vers différentes variables ou mémoires dynamiques.
- **Références** : Doivent être initialisées lors de leur déclaration et ne peuvent pas être réaffectées.

#### 4.2.3.2. Sécurité

- **Pointeurs** : Peuvent être null ou non initialisés, ce qui peut provoquer des erreurs d'exécution.
- **Références** : Ne peuvent pas être nulles et doivent être initialisées, ce qui réduit les risques d'erreurs.

#### 4.2.3.3. Syntaxe et utilisation

- **Pointeurs** : Utilisent `*` pour le déréférencement et `&` pour obtenir l'adresse.
- **Références** : Utilisent directement le nom de la référence pour accéder à la variable référencée.

### 4.2.4. Bonnes pratiques

- **Initialisation** : Toujours initialiser les pointeurs pour éviter les pointeurs non initialisés. Utiliser des références lorsque possible pour garantir l'initialisation.
- **Gestion de la mémoire** : Libérer la mémoire allouée dynamiquement à l'aide de `delete` ou `delete[]` pour éviter les fuites de mémoire.
- **Pointeurs intelligents** : Utiliser des pointeurs intelligents (`std::unique_ptr`, `std::shared_ptr`) pour automatiser la gestion de la mémoire et réduire les risques de fuites de mémoire.

## 4.3. Gestion des ressources : Constructeurs et destructeurs

La gestion des ressources en C++ repose sur l'utilisation correcte des constructeurs et des destructeurs pour allouer et libérer des ressources, respectivement. Les ressources peuvent inclure la mémoire dynamique, les fichiers, les connexions réseau, et autres. La RAII (Resource Acquisition Is Initialization) est un idiome de programmation utilisé en C++ pour gérer ces ressources de manière automatique et sûre.

### 4.3.1. Constructeurs

Un constructeur est une méthode spéciale qui est appelée lors de la création d'un objet. Il initialise l'objet et alloue les ressources nécessaires. Les constructeurs peuvent être surchargés pour accepter différents paramètres et peuvent également inclure une liste d'initialisation des membres.

#### 4.3.1.1. Déclaration et définition d'un constructeur

**Syntaxe :**

```cpp
class NomDeClasse {
public:
    NomDeClasse(paramètres); // Déclaration du constructeur
};
```

**Exemple :**

```cpp
class Personne {
private:
    string nom;
    int age;

public:
    // Constructeur avec initialisation des membres
    Personne(const string& nom, int age) : nom(nom), age(age) {}

    void sePresenter() const {
        cout << "Bonjour, je m'appelle " << nom << " et j'ai " << age << " ans." << endl;
    }
};
```

Dans cet exemple, le constructeur `Personne` initialise les membres `nom` et `age` avec les valeurs fournies.

#### 4.3.1.2. Constructeurs par défaut, de copie et de déplacement

**Constructeur par défaut :**

Si aucun constructeur n'est défini, le compilateur génère automatiquement un constructeur par défaut.

**Exemple :**

```cpp
class Exemple {
public:
    Exemple() {
        // Initialisation par défaut
    }
};
```

**Constructeur de copie :**

Le constructeur de copie initialise un objet à partir d'un autre objet du même type.

**Exemple :**

```cpp
class Exemple {
private:
    int* data;

public:
    // Constructeur de copie
    Exemple(const Exemple& autre) {
        data = new int(*autre.data);
    }
};
```

**Constructeur de déplacement :**

Le constructeur de déplacement initialise un objet en prenant possession des ressources d'un autre objet temporaire.

**Exemple :**

```cpp
class Exemple {
private:
    int* data;

public:
    // Constructeur de déplacement
    Exemple(Exemple&& autre) noexcept : data(autre.data) {
        autre.data = nullptr;
    }
};
```

### 4.3.2. Destructeurs

Un destructeur est une méthode spéciale appelée automatiquement lorsque l'objet est détruit. Il libère les ressources allouées par l'objet. Le destructeur a le même nom que la classe, précédé d'un tilde (`~`).

#### 4.3.2.1. Déclaration et définition d'un destructeur

**Syntaxe :**

```cpp
class NomDeClasse {
public:
    ~NomDeClasse(); // Déclaration du destructeur
};
```

**Exemple :**

```cpp
class Personne {
private:
    string nom;
    int age;

public:
    // Constructeur
    Personne(const string& nom, int age) : nom(nom), age(age) {}

    // Destructeur
    ~Personne() {
        // Libération des ressources
        cout << "Destruction de l'objet Personne." << endl;
    }

    void sePresenter() const {
        cout << "Bonjour, je m'appelle " << nom << " et j'ai " << age << " ans." << endl;
    }
};
```

Dans cet exemple, le destructeur `Personne` affiche un message lors de la destruction de l'objet.

### 4.3.3. RAII (Resource Acquisition Is Initialization)

La RAII est une technique de gestion des ressources où l'acquisition de ressources (par exemple, la mémoire, les fichiers) est liée à la durée de vie des objets. Lorsqu'un objet est créé, il acquiert des ressources dans son constructeur, et lorsque l'objet est détruit, il libère ces ressources dans son destructeur.

#### 4.3.3.1. Exemple de RAII

**Exemple :**

```cpp
class Fichier {
private:
    FILE* fichier;

public:
    // Constructeur ouvre le fichier
    Fichier(const char* nomFichier) {
        fichier = fopen(nomFichier, "r");
        if (!fichier) {
            throw runtime_error("Impossible d'ouvrir le fichier");
        }
    }

    // Destructeur ferme le fichier
    ~Fichier() {
        if (fichier) {
            fclose(fichier);
        }
    }

    // Méthode pour lire le fichier
    void lire() {
        // Lecture du fichier
    }
};
```

Dans cet exemple, la classe `Fichier` utilise la RAII pour s'assurer que le fichier est fermé correctement lorsque l'objet `Fichier` est détruit.

### 4.3.4. Gestion des ressources avec les smart pointers

Les pointeurs intelligents (`smart pointers`) de la bibliothèque standard C++ automatisent la gestion de la mémoire et des ressources.

#### 4.3.4.1. `std::unique_ptr`

Un `std::unique_ptr` possède une ressource de manière exclusive. Lorsque le `std::unique_ptr` est détruit, la ressource est automatiquement libérée.

**Exemple :**

```cpp
#include <memory>

class Exemple {
public:
    Exemple() {
        cout << "Constructeur" << endl;
    }
    ~Exemple() {
        cout << "Destructeur" << endl;
    }
};

int main() {
    {
        std::unique_ptr<Exemple> ptr = std::make_unique<Exemple>();
    } // Destructeur appelé ici
    return 0;
}
```

#### 4.3.4.2. `std::shared_ptr`

Un `std::shared_ptr` permet à plusieurs pointeurs de partager la même ressource. La ressource est libérée lorsque le dernier `std::shared_ptr` qui la possède est détruit.

**Exemple :**

```cpp
#include <memory>

class Exemple {
public:
    Exemple() {
        cout << "Constructeur" << endl;
    }
    ~Exemple() {
        cout << "Destructeur" << endl;
    }
};

int main() {
    {
        std::shared_ptr<Exemple> ptr1 = std::make_shared<Exemple>();
        {
            std::shared_ptr<Exemple> ptr2 = ptr1;
        } // Destructeur non appelé ici, ptr1 possède toujours la ressource
    } // Destructeur appelé ici, ptr1 est détruit
    return 0;
}
```

## 4.4. Les fuites de mémoire et leur prévention

Les fuites de mémoire se produisent lorsque la mémoire allouée dynamiquement n'est pas libérée correctement, entraînant une perte de mémoire disponible pour le programme. Les fuites de mémoire peuvent réduire les performances du système, provoquer des plantages et rendre les programmes non fiables. Cette section explore les causes des fuites de mémoire et les techniques pour les prévenir.

### 4.4.1. Causes des fuites de mémoire

Les fuites de mémoire peuvent survenir pour plusieurs raisons, notamment :

1. **Oubli de libération de mémoire** : Lorsque la mémoire allouée dynamiquement n'est pas libérée à la fin de son utilisation.
2. **Chemins de code non atteints** : Lorsque certains chemins de code ne libèrent pas correctement la mémoire allouée en raison d'erreurs logiques ou de gestion d'exceptions.
3. **Pertes de pointeurs** : Lorsque tous les pointeurs vers un bloc de mémoire alloué sont perdus, rendant la mémoire inaccessible et non libérable.

### 4.4.2. Techniques de prévention des fuites de mémoire

Il existe plusieurs techniques et bonnes pratiques pour prévenir les fuites de mémoire en C++.

#### 4.4.2.1. Utilisation appropriée de `delete` et `delete[]`

Chaque appel à `new` doit être apparié avec un appel à `delete`, et chaque appel à `new[]` doit être apparié avec un appel à `delete[]`.

**Exemple :**

```cpp
int* ptr = new int(10);
// ... utilisation de ptr
delete ptr; // Libération de la mémoire allouée

int* tableau = new int[10];
// ... utilisation du tableau
delete[] tableau; // Libération de la mémoire allouée pour le tableau
```

#### 4.4.2.2. Utilisation des pointeurs intelligents

Les pointeurs intelligents (`smart pointers`) de la bibliothèque standard C++ automatisent la gestion de la mémoire, garantissant que la mémoire est libérée lorsque les objets ne sont plus utilisés.

**`std::unique_ptr`** :

Un `std::unique_ptr` possède une ressource de manière exclusive et la libère lorsque le pointeur est détruit.

**Exemple :**

```cpp
#include <memory>

std::unique_ptr<int> ptr = std::make_unique<int>(10); // Mémoire allouée
// Mémoire automatiquement libérée lorsque ptr sort de la portée
```

**`std::shared_ptr`** :

Un `std::shared_ptr` permet à plusieurs pointeurs de partager la même ressource. La mémoire est libérée lorsque le dernier `std::shared_ptr` qui possède la ressource est détruit.

**Exemple :**

```cpp
#include <memory>

std::shared_ptr<int> ptr1 = std::make_shared<int>(10);
std::shared_ptr<int> ptr2 = ptr1; // ptr2 partage la ressource avec ptr1
// Mémoire automatiquement libérée lorsque le dernier pointeur (ptr1 ou ptr2) sort de la portée
```

#### 4.4.2.3. Utilisation de RAII (Resource Acquisition Is Initialization)

La RAII est une technique qui garantit que les ressources sont libérées lorsque les objets qui les possèdent sont détruits. Cela est particulièrement utile pour la gestion des ressources comme les fichiers et les connexions réseau.

**Exemple :**

```cpp
class Fichier {
private:
    FILE* fichier;

public:
    Fichier(const char* nomFichier) {
        fichier = fopen(nomFichier, "r");
        if (!fichier) {
            throw std::runtime_error("Impossible d'ouvrir le fichier");
        }
    }

    ~Fichier() {
        if (fichier) {
            fclose(fichier);
        }
    }

    void lire() {
        // Lecture du fichier
    }
};

// Utilisation de la classe Fichier
void traiterFichier() {
    Fichier fichier("exemple.txt");
    fichier.lire();
} // Le destructeur de Fichier est appelé ici, garantissant que le fichier est fermé
```

#### 4.4.2.4. Utilisation de bibliothèques et outils de vérification de mémoire

Plusieurs bibliothèques et outils peuvent aider à détecter et à prévenir les fuites de mémoire :

- **Valgrind** : Un outil de détection des fuites de mémoire qui analyse les programmes et rapporte les blocs de mémoire qui n'ont pas été libérés.
- **AddressSanitizer** : Un outil de détection des erreurs de mémoire intégré à certains compilateurs comme GCC et Clang.
- **Smart Pointers** : Utilisation des smart pointers fournis par la bibliothèque standard C++ (`std::unique_ptr`, `std::shared_ptr`) pour automatiser la gestion de la mémoire.

### 4.4.3. Exemple pratique de prévention des fuites de mémoire

Voici un exemple complet qui montre comment utiliser les techniques de prévention des fuites de mémoire en C++ :

**Exemple :**

```cpp
#include <iostream>
#include <memory>

class Ressource {
public:
    Ressource() {
        std::cout << "Ressource acquise" << std::endl;
    }
    ~Ressource() {
        std::cout << "Ressource libérée" << std::endl;
    }
    void utiliser() {
        std::cout << "Utilisation de la ressource" << std::endl;
    }
};

void utiliserRessource() {
    std::unique_ptr<Ressource> ressource = std::make_unique<Ressource>();
    ressource->utiliser();
    // Ressource automatiquement libérée ici
}

int main() {
    utiliserRessource();
    return 0;
}
```

Dans cet exemple, la classe `Ressource` acquiert et libère une ressource. La fonction `utiliserRessource` utilise un `std::unique_ptr` pour gérer la mémoire automatiquement, garantissant ainsi qu'il n'y a pas de fuite de mémoire.

# 5. Bibliothèques Standards et Utilitaires

## 5.1. La bibliothèque standard de template (STL)

La bibliothèque standard de template (STL) est une composante essentielle de la bibliothèque standard C++. Elle fournit une collection de classes et de fonctions génériques, permettant une manipulation efficace et flexible des données. La STL est basée sur trois concepts principaux : les conteneurs, les itérateurs et les algorithmes.

### 5.1.1. Introduction à la STL

La STL (Standard Template Library) est conçue pour fournir des structures de données et des algorithmes génériques réutilisables. Elle inclut :

- **Conteneurs** : Classes qui stockent des collections d'objets.
- **Itérateurs** : Objets qui permettent de parcourir les éléments des conteneurs.
- **Algorithmes** : Fonctions génériques qui opèrent sur les éléments des conteneurs via des itérateurs.

### 5.1.2. Conteneurs

Les conteneurs de la STL sont des classes génériques utilisées pour stocker et organiser des collections d'objets. Ils se divisent en trois catégories principales : séquentiels, associatifs et adaptatifs.

#### 5.1.2.1. Conteneurs séquentiels

Les conteneurs séquentiels stockent les éléments dans un ordre linéaire.

- **vector** : Un tableau dynamique qui peut changer de taille.
  
  **Exemple :**
  ```cpp
  #include <vector>
  #include <iostream>
  
  std::vector<int> v = {1, 2, 3, 4, 5};
  v.push_back(6); // Ajoute un élément à la fin
  ```

- **deque** : Un tableau doublement terminé permettant l'insertion et la suppression d'éléments aux deux extrémités.
  
  **Exemple :**
  ```cpp
  #include <deque>
  
  std::deque<int> d = {1, 2, 3, 4, 5};
  d.push_front(0); // Ajoute un élément au début
  ```

- **list** : Une liste doublement chaînée permettant une insertion et une suppression rapides de n'importe quel élément.
  
  **Exemple :**
  ```cpp
  #include <list>
  
  std::list<int> l = {1, 2, 3, 4, 5};
  l.push_back(6); // Ajoute un élément à la fin
  ```

#### 5.1.2.2. Conteneurs associatifs

Les conteneurs associatifs stockent les éléments sous forme de paires clé-valeur et permettent une recherche rapide.

- **set** : Un ensemble d'éléments uniques, ordonnés.
  
  **Exemple :**
  ```cpp
  #include <set>
  
  std::set<int> s = {1, 2, 3, 4, 5};
  s.insert(6); // Ajoute un élément unique
  ```

- **map** : Une collection de paires clé-valeur, avec des clés uniques.
  
  **Exemple :**
  ```cpp
  #include <map>
  
  std::map<int, std::string> m;
  m[1] = "one";
  m[2] = "two";
  ```

- **unordered_set** et **unordered_map** : Versions non ordonnées des conteneurs `set` et `map`, offrant des performances de recherche plus rapides.

  **Exemple :**
  ```cpp
  #include <unordered_set>
  #include <unordered_map>
  
  std::unordered_set<int> us = {1, 2, 3, 4, 5};
  us.insert(6); // Ajoute un élément unique
  
  std::unordered_map<int, std::string> um;
  um[1] = "one";
  um[2] = "two";
  ```

#### 5.1.2.3. Conteneurs adaptatifs

Les conteneurs adaptatifs fournissent une interface spécifique pour les structures de données sous-jacentes.

- **stack** : Une pile (LIFO - Last In First Out).
  
  **Exemple :**
  ```cpp
  #include <stack>
  
  std::stack<int> s;
  s.push(1);
  s.push(2);
  ```

- **queue** : Une file d'attente (FIFO - First In First Out).
  
  **Exemple :**
  ```cpp
  #include <queue>
  
  std::queue<int> q;
  q.push(1);
  q.push(2);
  ```

- **priority_queue** : Une file de priorité, où les éléments de plus haute priorité sont accessibles en premier.
  
  **Exemple :**
  ```cpp
  #include <queue>
  
  std::priority_queue<int> pq;
  pq.push(10);
  pq.push(5);
  ```

### 5.1.3. Itérateurs

Les itérateurs sont des objets qui pointent vers des éléments dans les conteneurs et permettent de les parcourir. Ils offrent une interface standard pour accéder aux éléments des conteneurs de manière séquentielle.

#### 5.1.3.1. Types d'itérateurs

- **Input Iterator** : Utilisé pour lire des données séquentiellement.
- **Output Iterator** : Utilisé pour écrire des données séquentiellement.
- **Forward Iterator** : Permet de lire et d'écrire des données dans une seule direction.
- **Bidirectional Iterator** : Permet de lire et d'écrire des données dans les deux directions.
- **Random Access Iterator** : Permet un accès direct à n'importe quel élément, comme un pointeur.

**Exemple :**

```cpp
#include <vector>
#include <iostream>

std::vector<int> v = {1, 2, 3, 4, 5};
for (std::vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
    std::cout << *it << " "; // Affiche les éléments de v
}
```

### 5.1.4. Algorithmes

La STL fournit de nombreux algorithmes génériques qui peuvent être utilisés avec les conteneurs via des itérateurs. Ces algorithmes incluent des fonctions pour le tri, la recherche, la modification, et bien d'autres opérations sur les données.

#### 5.1.4.1. Algorithmes de tri et de recherche

- **sort** : Trie les éléments dans un conteneur.
  
  **Exemple :**
  ```cpp
  #include <algorithm>
  
  std::vector<int> v = {4, 2, 5, 1, 3};
  std::sort(v.begin(), v.end()); // Trie les éléments de v
  ```

- **find** : Trouve le premier élément correspondant à une valeur donnée.
  
  **Exemple :**
  ```cpp
  #include <algorithm>
  
  std::vector<int> v = {1, 2, 3, 4, 5};
  auto it = std::find(v.begin(), v.end(), 3); // Trouve l'élément 3
  ```

#### 5.1.4.2. Algorithmes de modification

- **copy** : Copie les éléments d'une source vers une destination.
  
  **Exemple :**
  ```cpp
  #include <algorithm>
  #include <vector>
  
  std::vector<int> source = {1, 2, 3};
  std::vector<int> destination(3);
  std::copy(source.begin(), source.end(), destination.begin());
  ```

- **transform** : Applique une fonction à chaque élément d'une source et stocke le résultat dans une destination.
  
  **Exemple :**
  ```cpp
  #include <algorithm>
  #include <vector>
  #include <functional>
  
  std::vector<int> v = {1, 2, 3};
  std::vector<int> result(3);
  std::transform(v.begin(), v.end(), result.begin(), [](int x) { return x * x; }); // Calcule le carré de chaque élément
  ```

## 5.2. Utilisation des conteneurs : Vector, Map, Set

Les conteneurs de la bibliothèque standard de template (STL) offrent une variété de structures de données pour stocker et organiser les collections d'éléments. Parmi les plus couramment utilisés figurent `vector`, `map`, et `set`. Chacun de ces conteneurs a des caractéristiques et des utilisations spécifiques qui le rendent adapté à différentes situations.

### 5.2.1. Vector

`vector` est un conteneur séquentiel qui représente un tableau dynamique capable de changer de taille. Il offre un accès rapide aux éléments et une gestion efficace de la mémoire.

#### 5.2.1.1. Déclaration et initialisation

**Syntaxe :**

```cpp
std::vector<type> nom_du_vecteur;
```

**Exemple :**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    return 0;
}
```

#### 5.2.1.2. Accès aux éléments

Les éléments d'un `vector` peuvent être accédés à l'aide de l'opérateur de subscript `[]` ou de la méthode `at()`.

**Exemple :**

```cpp
std::cout << v[0] << std::endl; // Accès au premier élément
std::cout << v.at(1) << std::endl; // Accès au deuxième élément
```

#### 5.2.1.3. Ajout et suppression d'éléments

Les méthodes `push_back()`, `pop_back()`, `insert()`, et `erase()` permettent d'ajouter et de supprimer des éléments.

**Exemple :**

```cpp
v.push_back(6); // Ajoute un élément à la fin
v.pop_back();   // Supprime le dernier élément

v.insert(v.begin() + 2, 10); // Insère 10 à la troisième position
v.erase(v.begin() + 1); // Supprime le deuxième élément
```

#### 5.2.1.4. Itération

Les itérateurs permettent de parcourir les éléments du `vector`.

**Exemple :**

```cpp
for (std::vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
    std::cout << *it << " ";
}
```

### 5.2.2. Map

`map` est un conteneur associatif qui stocke des paires clé-valeur, où chaque clé est unique. Il est implémenté sous forme d'arbre binaire de recherche, offrant des opérations de recherche, d'insertion et de suppression en temps logarithmique.

#### 5.2.2.1. Déclaration et initialisation

**Syntaxe :**

```cpp
std::map<type_cle, type_valeur> nom_de_la_map;
```

**Exemple :**

```cpp
#include <map>
#include <iostream>

int main() {
    std::map<int, std::string> m;
    m[1] = "un";
    m[2] = "deux";
    return 0;
}
```

#### 5.2.2.2. Accès aux éléments

Les éléments d'un `map` peuvent être accédés à l'aide de l'opérateur de subscript `[]` ou de la méthode `at()`.

**Exemple :**

```cpp
std::cout << m[1] << std::endl; // Accès à la valeur associée à la clé 1
std::cout << m.at(2) << std::endl; // Accès à la valeur associée à la clé 2
```

#### 5.2.2.3. Ajout et suppression d'éléments

Les méthodes `insert()` et `erase()` permettent d'ajouter et de supprimer des éléments.

**Exemple :**

```cpp
m.insert(std::make_pair(3, "trois")); // Insère une nouvelle paire clé-valeur
m.erase(1); // Supprime la paire avec la clé 1
```

#### 5.2.2.4. Itération

Les itérateurs permettent de parcourir les éléments du `map`.

**Exemple :**

```cpp
for (std::map<int, std::string>::iterator it = m.begin(); it != m.end(); ++it) {
    std::cout << it->first << ": " << it->second << std::endl;
}
```

### 5.2.3. Set

`set` est un conteneur associatif qui stocke des éléments uniques, ordonnés par une fonction de comparaison. Il est implémenté sous forme d'arbre binaire de recherche, offrant des opérations de recherche, d'insertion et de suppression en temps logarithmique.

#### 5.2.3.1. Déclaration et initialisation

**Syntaxe :**

```cpp
std::set<type> nom_du_set;
```

**Exemple :**

```cpp
#include <set>
#include <iostream>

int main() {
    std::set<int> s = {1, 2, 3, 4, 5};
    return 0;
}
```

#### 5.2.3.2. Accès aux éléments

Les éléments d'un `set` ne peuvent pas être accédés directement par indice comme dans un `vector`. Les méthodes `find()` et `count()` permettent de rechercher des éléments.

**Exemple :**

```cpp
if (s.find(3) != s.end()) {
    std::cout << "3 trouvé dans le set" << std::endl;
}
if (s.count(4) > 0) {
    std::cout << "4 trouvé dans le set" << std::endl;
}
```

#### 5.2.3.3. Ajout et suppression d'éléments

Les méthodes `insert()` et `erase()` permettent d'ajouter et de supprimer des éléments.

**Exemple :**

```cpp
s.insert(6); // Ajoute un nouvel élément
s.erase(2); // Supprime l'élément 2
```

#### 5.2.3.4. Itération

Les itérateurs permettent de parcourir les éléments du `set`.

**Exemple :**

```cpp
for (std::set<int>::iterator it = s.begin(); it != s.end(); ++it) {
    std::cout << *it << " ";
}
```

## 5.3. Itérateurs et algorithmes sur les conteneurs

Les itérateurs et les algorithmes de la bibliothèque standard de templates (STL) sont des outils puissants qui permettent de manipuler les éléments des conteneurs de manière générique et efficace. Les itérateurs fournissent une interface standard pour parcourir les éléments d'un conteneur, tandis que les algorithmes offrent une variété de fonctions pour effectuer des opérations courantes sur ces éléments.

### 5.3.1. Itérateurs

Les itérateurs sont des objets qui pointent vers des éléments dans un conteneur et permettent de parcourir ces éléments. Ils offrent une interface standard et peuvent être utilisés de manière similaire aux pointeurs.

#### 5.3.1.1. Types d'itérateurs

Les itérateurs peuvent être classés en plusieurs catégories en fonction des opérations qu'ils supportent :

- **Input Iterator** : Utilisé pour lire des données séquentiellement.
- **Output Iterator** : Utilisé pour écrire des données séquentiellement.
- **Forward Iterator** : Permet de lire et d'écrire des données dans une seule direction.
- **Bidirectional Iterator** : Permet de lire et d'écrire des données dans les deux directions.
- **Random Access Iterator** : Permet un accès direct à n'importe quel élément, comme un pointeur.

#### 5.3.1.2. Utilisation des itérateurs

Les itérateurs sont généralement obtenus via les méthodes `begin()` et `end()` des conteneurs.

**Exemple :**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};

    // Utilisation des itérateurs pour parcourir les éléments
    for (std::vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### 5.3.2. Algorithmes

La STL fournit un grand nombre d'algorithmes génériques qui peuvent être utilisés avec les conteneurs via des itérateurs. Ces algorithmes incluent des fonctions pour le tri, la recherche, la modification, et bien d'autres opérations sur les données.

#### 5.3.2.1. Algorithmes de tri et de recherche

- **sort** : Trie les éléments dans un conteneur.

  **Exemple :**

  ```cpp
  #include <algorithm>
  #include <vector>
  #include <iostream>

  int main() {
      std::vector<int> v = {4, 2, 5, 1, 3};
      std::sort(v.begin(), v.end()); // Trie les éléments de v

      for (const auto& elem : v) {
          std::cout << elem << " ";
      }
      std::cout << std::endl;

      return 0;
  }
  ```

- **find** : Trouve le premier élément correspondant à une valeur donnée.

  **Exemple :**

  ```cpp
  #include <algorithm>
  #include <vector>
  #include <iostream>

  int main() {
      std::vector<int> v = {1, 2, 3, 4, 5};
      auto it = std::find(v.begin(), v.end(), 3); // Trouve l'élément 3

      if (it != v.end()) {
          std::cout << "Élément trouvé : " << *it << std::endl;
      } else {
          std::cout << "Élément non trouvé" << std::endl;
      }

      return 0;
  }
  ```

#### 5.3.2.2. Algorithmes de modification

- **copy** : Copie les éléments d'une source vers une destination.

  **Exemple :**

  ```cpp
  #include <algorithm>
  #include <vector>
  #include <iostream>

  int main() {
      std::vector<int> source = {1, 2, 3};
      std::vector<int> destination(3);
      std::copy(source.begin(), source.end(), destination.begin());

      for (const auto& elem : destination) {
          std::cout << elem << " ";
      }
      std::cout << std::endl;

      return 0;
  }
  ```

- **transform** : Applique une fonction à chaque élément d'une source et stocke le résultat dans une destination.

  **Exemple :**

  ```cpp
  #include <algorithm>
  #include <vector>
  #include <iostream>

  int main() {
      std::vector<int> v = {1, 2, 3};
      std::vector<int> result(3);
      std::transform(v.begin(), v.end(), result.begin(), [](int x) { return x * x; }); // Calcule le carré de chaque élément

      for (const auto& elem : result) {
          std::cout << elem << " ";
      }
      std::cout << std::endl;

      return 0;
  }
  ```

#### 5.3.2.3. Algorithmes de permutation

- **next_permutation** : Génère la permutation suivante lexicographiquement.

  **Exemple :**

  ```cpp
  #include <algorithm>
  #include <vector>
  #include <iostream>

  int main() {
      std::vector<int> v = {1, 2, 3};

      do {
          for (const auto& elem : v) {
              std::cout << elem << " ";
          }
          std::cout << std::endl;
      } while (std::next_permutation(v.begin(), v.end()));

      return 0;
  }
  ```

- **prev_permutation** : Génère la permutation précédente lexicographiquement.

  **Exemple :**

  ```cpp
  #include <algorithm>
  #include <vector>
  #include <iostream>

  int main() {
      std::vector<int> v = {3, 2, 1};

      do {
          for (const auto& elem : v) {
              std::cout << elem << " ";
          }
          std::cout << std::endl;
      } while (std::prev_permutation(v.begin(), v.end()));

      return 0;
  }
  ```

### 5.3.3. Utilisation des algorithmes avec différents conteneurs

Les algorithmes de la STL peuvent être utilisés avec n'importe quel conteneur qui supporte les itérateurs appropriés. Voici quelques exemples d'utilisation avec différents conteneurs :

#### 5.3.3.1 Utilisation avec `vector`

**Exemple :**

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {4, 2, 5, 1, 3};
    std::sort(v.begin(), v.end()); // Trie les éléments de v

    auto it = std::find(v.begin(), v.end(), 3); // Trouve l'élément 3
    if (it != v.end()) {
        std::cout << "Élément trouvé : " << *it << std::endl;
    } else {
        std::cout << "Élément non trouvé" << std::endl;
    }

    return 0;
}
```

#### 5.3.3.2. Utilisation avec `map`

**Exemple :**

```cpp
#include <algorithm>
#include <map>
#include <iostream>

int main() {
    std::map<int, std::string> m = {{1, "un"}, {2, "deux"}, {3, "trois"}};
    
    // Utilisation de std::for_each pour afficher les paires clé-valeur
    std::for_each(m.begin(), m.end(), [](const std::pair<int, std::string>& p) {
        std::cout << p.first << ": " << p.second << std::endl;
    });

    return 0;
}
```

#### 5.3.3.3. Utilisation avec `set`

**Exemple :**

```cpp
#include <algorithm>
#include <set>
#include <iostream>

int main() {
    std::set<int> s = {3, 1, 4, 1, 5, 9};
    
    // Utilisation de std::for_each pour afficher les éléments du set
    std::for_each(s.begin(), s.end(), [](int n) {
        std::cout << n << " ";
    });
    std::cout << std::endl;

    return 0;
}
```

## 5.4. Gestion des chaînes de caractères et des fichiers

La gestion des chaînes de caractères et des fichiers est une tâche courante en programmation. En C++, la bibliothèque standard fournit des classes et des fonctions puissantes pour manipuler les chaînes de caractères et les fichiers de manière efficace et intuitive.

### 5.4.1. Gestion des chaînes de caractères

En C++, les chaînes de caractères peuvent être manipulées en utilisant principalement deux approches : les chaînes de caractères C (tableaux de `char`) et la classe `std::string` de la bibliothèque standard.

#### 5.4.1.1. Les chaînes de caractères C

Les chaînes de caractères C sont des tableaux de caractères terminés par un caractère nul (`'\0'`).

**Exemple :**

```cpp
#include <iostream>
#include <cstring>

int main() {
    char chaineC[] = "Bonjour, le monde!";
    std::cout << "Chaîne de caractères C : " << chaineC << std::endl;

    // Longueur de la chaîne
    std::cout << "Longueur : " << std::strlen(chaineC) << std::endl;

    // Copier une chaîne
    char copie[50];
    std::strcpy(copie, chaineC);
    std::cout << "Copie : " << copie << std::endl;

    // Concaténer des chaînes
    char ajout[] = " Comment ça va ?";
    std::strcat(copie, ajout);
    std::cout << "Concaténation : " << copie << std::endl;

    return 0;
}
```

#### 5.4.1.2. La classe `std::string`

La classe `std::string` offre une manière plus pratique et sûre de manipuler les chaînes de caractères.

**Exemple :**

```cpp
#include <iostream>
#include <string>

int main() {
    std::string str = "Bonjour, le monde!";
    std::cout << "Chaîne de caractères std::string : " << str << std::endl;

    // Longueur de la chaîne
    std::cout << "Longueur : " << str.length() << std::endl;

    // Ajouter à la chaîne
    str += " Comment ça va ?";
    std::cout << "Ajout : " << str << std::endl;

    // Sous-chaîne
    std::string sousChaine = str.substr(8, 11);
    std::cout << "Sous-chaîne : " << sousChaine << std::endl;

    // Trouver une sous-chaîne
    size_t position = str.find("monde");
    if (position != std::string::npos) {
        std::cout << "\"monde\" trouvé à la position " << position << std::endl;
    }

    return 0;
}
```

### 5.4.2. Gestion des fichiers

La bibliothèque standard C++ offre des classes pour manipuler les fichiers, principalement `std::ifstream` pour la lecture, `std::ofstream` pour l'écriture, et `std::fstream` pour les deux.

#### 5.4.2.1. Lecture de fichiers

Pour lire des fichiers, nous utilisons `std::ifstream`.

**Exemple :**

```cpp
#include <iostream>
#include <fstream>
#include <string>

int main() {
    std::ifstream fichier("exemple.txt");
    if (fichier.is_open()) {
        std::string ligne;
        while (std::getline(fichier, ligne)) {
            std::cout << ligne << std::endl;
        }
        fichier.close();
    } else {
        std::cerr << "Impossible d'ouvrir le fichier." << std::endl;
    }

    return 0;
}
```

#### 5.4.2.2. Écriture de fichiers

Pour écrire dans des fichiers, nous utilisons `std::ofstream`.

**Exemple :**

```cpp
#include <iostream>
#include <fstream>

int main() {
    std::ofstream fichier("sortie.txt");
    if (fichier.is_open()) {
        fichier << "Bonjour, le monde!" << std::endl;
        fichier << "Écriture dans un fichier." << std::endl;
        fichier.close();
    } else {
        std::cerr << "Impossible d'ouvrir le fichier." << std::endl;
    }

    return 0;
}
```

#### 5.4.2.3. Lecture et écriture de fichiers

Pour lire et écrire dans des fichiers, nous utilisons `std::fstream`.

**Exemple :**

```cpp
#include <iostream>
#include <fstream>
#include <string>

int main() {
    std::fstream fichier("data.txt", std::ios::in | std::ios::out | std::ios::app);
    if (fichier.is_open()) {
        // Écriture dans le fichier
        fichier << "Nouvelle ligne de texte." << std::endl;

        // Repositionner le curseur au début pour la lecture
        fichier.seekg(0, std::ios::beg);

        // Lecture du fichier
        std::string ligne;
        while (std::getline(fichier, ligne)) {
            std::cout << ligne << std::endl;
        }

        fichier.close();
    } else {
        std::cerr << "Impossible d'ouvrir le fichier." << std::endl;
    }

    return 0;
}
```

### 5.4.3. Gestion des erreurs

Lors de la manipulation des fichiers, il est crucial de vérifier les erreurs telles que l'impossibilité d'ouvrir un fichier, des erreurs de lecture ou d'écriture. Les méthodes `is_open()`, `eof()`, `fail()`, `bad()`, et `good()` de `std::ifstream`, `std::ofstream`, et `std::fstream` sont utilisées pour gérer ces erreurs.

**Exemple :**

```cpp
#include <iostream>
#include <fstream>

int main() {
    std::ifstream fichier("inexistant.txt");
    if (!fichier) {
        std::cerr << "Erreur d'ouverture du fichier." << std::endl;
        return 1;
    }

    std::string ligne;
    while (std::getline(fichier, ligne)) {
        if (fichier.bad()) {
            std::cerr << "Erreur de lecture." << std::endl;
            break;
        }
        std::cout << ligne << std::endl;
    }

    fichier.close();
    return 0;
}
```

# 6. Développement Avancé

## 6.1. Programmation générique avec les templates

La programmation générique permet d'écrire des fonctions et des classes qui peuvent fonctionner avec n'importe quel type de données. En C++, cela est réalisé à l'aide des templates (modèles). Les templates permettent de créer du code réutilisable et flexible qui peut être utilisé avec différents types de données sans redondance.

### 6.1.1. Templates de fonctions

Les templates de fonctions permettent de définir des fonctions génériques qui peuvent accepter des arguments de différents types.

#### 6.1.1.1. Déclaration et définition

**Syntaxe :**

```cpp
template <typename T>
T nomDeLaFonction(T param1, T param2);
```

**Exemple :**

```cpp
#include <iostream>

// Template de fonction pour retourner le maximum de deux valeurs
template <typename T>
T maximum(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    std::cout << "Maximum de 3 et 7 : " << maximum(3, 7) << std::endl;          // Utilisation avec des entiers
    std::cout << "Maximum de 3.5 et 2.1 : " << maximum(3.5, 2.1) << std::endl;  // Utilisation avec des flottants
    std::cout << "Maximum de 'a' et 'b' : " << maximum('a', 'b') << std::endl;  // Utilisation avec des caractères

    return 0;
}
```

Dans cet exemple, la fonction `maximum` est définie comme un template et peut être utilisée avec différents types de données (int, double, char).

### 6.1.2. Templates de classes

Les templates de classes permettent de définir des classes génériques qui peuvent fonctionner avec différents types de données.

#### 6.1.2.1. Déclaration et définition

**Syntaxe :**

```cpp
template <typename T>
class NomDeLaClasse {
    // Membres de la classe
};
```

**Exemple :**

```cpp
#include <iostream>

// Template de classe pour une paire de valeurs
template <typename T>
class Paire {
private:
    T premier, second;

public:
    Paire(T a, T b) : premier(a), second(b) {}

    T getPremier() const {
        return premier;
    }

    T getSecond() const {
        return second;
    }
};

int main() {
    Paire<int> paireInt(1, 2);
    std::cout << "Paire d'entiers : (" << paireInt.getPremier() << ", " << paireInt.getSecond() << ")" << std::endl;

    Paire<double> paireDouble(1.1, 2.2);
    std::cout << "Paire de doubles : (" << paireDouble.getPremier() << ", " << paireDouble.getSecond() << ")" << std::endl;

    Paire<char> paireChar('a', 'b');
    std::cout << "Paire de caractères : (" << paireChar.getPremier() << ", " << paireChar.getSecond() << ")" << std::endl;

    return 0;
}
```

Dans cet exemple, la classe `Paire` est définie comme un template et peut être utilisée avec différents types de données.

### 6.1.3. Templates non-type

Les templates non-type utilisent des paramètres qui ne sont pas des types mais des valeurs constantes.

#### 6.1.3.1. Déclaration et définition

**Syntaxe :**

```cpp
template <typename T, int N>
class NomDeLaClasse {
    // Membres de la classe
};
```

**Exemple :**

```cpp
#include <iostream>

// Template de classe pour un tableau fixe
template <typename T, int N>
class Tableau {
private:
    T elements[N];

public:
    void setElement(int index, T value) {
        if (index >= 0 && index < N) {
            elements[index] = value;
        }
    }

    T getElement(int index) const {
        if (index >= 0 && index < N) {
            return elements[index];
        }
        return T(); // Retourne une valeur par défaut si l'index est invalide
    }

    int getSize() const {
        return N;
    }
};

int main() {
    Tableau<int, 5> tableauInt;
    tableauInt.setElement(0, 10);
    tableauInt.setElement(1, 20);

    for (int i = 0; i < tableauInt.getSize(); i++) {
        std::cout << "Element " << i << ": " << tableauInt.getElement(i) << std::endl;
    }

    return 0;
}
```

Dans cet exemple, la classe `Tableau` est définie comme un template avec un paramètre non-type pour la taille du tableau.

### 6.1.4. Templates et spécialisation

La spécialisation des templates permet de définir des implémentations spécifiques pour certains types de données.

#### 6.1.4.1. Spécialisation complète

**Syntaxe :**

```cpp
template <>
class NomDeLaClasse<type_specifique> {
    // Membres de la classe pour le type spécifique
};
```

**Exemple :**

```cpp
#include <iostream>

// Template de classe général
template <typename T>
class Affichage {
public:
    void afficher(const T& valeur) {
        std::cout << "Valeur : " << valeur << std::endl;
    }
};

// Spécialisation de template pour les chaînes de caractères
template <>
class Affichage<std::string> {
public:
    void afficher(const std::string& valeur) {
        std::cout << "Chaîne de caractères : " << valeur << std::endl;
    }
};

int main() {
    Affichage<int> affichageInt;
    affichageInt.afficher(42);

    Affichage<std::string> affichageString;
    affichageString.afficher("Bonjour");

    return 0;
}
```

Dans cet exemple, la classe `Affichage` est spécialisée pour les `std::string` afin de fournir une implémentation spécifique pour l'affichage des chaînes de caractères.

### 6.1.5. Templates et héritage

Les templates peuvent également être utilisés avec l'héritage pour créer des classes génériques et réutilisables.

**Exemple :**

```cpp
#include <iostream>

// Template de classe de base
template <typename T>
class Base {
protected:
    T valeur;

public:
    Base(T v) : valeur(v) {}

    void afficher() const {
        std::cout << "Valeur : " << valeur << std::endl;
    }
};

// Template de classe dérivée
template <typename T>
class Derivee : public Base<T> {
public:
    Derivee(T v) : Base<T>(v) {}

    void afficherDouble() const {
        std::cout << "Double valeur : " << this->valeur * 2 << std::endl;
    }
};

int main() {
    Derivee<int> d(5);
    d.afficher();
    d.afficherDouble();

    return 0;
}
```

Dans cet exemple, la classe `Derivee` hérite de la classe `Base` et ajoute une méthode pour afficher le double de la valeur.

## 6.2. Gestion des exceptions

La gestion des exceptions en C++ permet de gérer les erreurs et les conditions exceptionnelles de manière structurée et robuste. Les exceptions offrent un mécanisme pour signaler et traiter les erreurs sans encombrer le code principal avec des vérifications d'erreurs. Cette section détaille les concepts de base de la gestion des exceptions, y compris la syntaxe, les types d'exceptions et les bonnes pratiques.

### 6.2.1. Concepts de base

Les exceptions en C++ sont utilisées pour gérer les erreurs ou les situations inattendues qui surviennent pendant l'exécution d'un programme. Lorsqu'une erreur se produit, une exception est lancée (`throw`) et peut être capturée (`catch`) pour être traitée.

#### 6.2.1.1. Lancer une exception

Pour lancer une exception, on utilise le mot-clé `throw` suivi de l'objet de l'exception.

**Syntaxe :**

```cpp
throw objet_exception;
```

**Exemple :**

```cpp
#include <iostream>

void verifierValeur(int valeur) {
    if (valeur < 0) {
        throw std::invalid_argument("Valeur négative non permise");
    }
}

int main() {
    try {
        verifierValeur(-1);
    } catch (const std::invalid_argument& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    return 0;
}
```

#### 6.2.1.2. Capturer une exception

Pour capturer et traiter une exception, on utilise les blocs `try` et `catch`.

**Syntaxe :**

```cpp
try {
    // Code susceptible de lancer une exception
} catch (const type_exception& e) {
    // Traitement de l'exception
}
```

**Exemple :**

```cpp
try {
    verifierValeur(-1);
} catch (const std::invalid_argument& e) {
    std::cerr << "Exception: " << e.what() << std::endl;
}
```

### 6.2.2. Types d'exceptions

En C++, les exceptions peuvent être de n'importe quel type, mais il est courant d'utiliser des classes dérivées de `std::exception`, qui est la classe de base pour les exceptions standard.

#### 6.2.2.1. std::exception

La classe `std::exception` est la classe de base pour toutes les exceptions standard en C++. Elle fournit la méthode `what()` qui retourne une description de l'erreur.

**Exemple :**

```cpp
#include <iostream>
#include <exception>

int main() {
    try {
        throw std::exception();
    } catch (const std::exception& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    return 0;
}
```

#### 6.2.2.2. std::runtime_error

`std::runtime_error` est une classe d'exception utilisée pour signaler des erreurs d'exécution.

**Exemple :**

```cpp
#include <iostream>
#include <stdexcept>

int main() {
    try {
        throw std::runtime_error("Erreur d'exécution");
    } catch (const std::runtime_error& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    return 0;
}
```

#### 6.2.2.3. std::invalid_argument

`std::invalid_argument` est une classe d'exception utilisée pour signaler des arguments invalides passés à une fonction.

**Exemple :**

```cpp
#include <iostream>
#include <stdexcept>

void verifierValeur(int valeur) {
    if (valeur < 0) {
        throw std::invalid_argument("Valeur négative non permise");
    }
}

int main() {
    try {
        verifierValeur(-1);
    } catch (const std::invalid_argument& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    return 0;
}
```

### 6.2.3. Gestion des exceptions personnalisées

Il est possible de définir ses propres classes d'exceptions en dérivant de `std::exception` ou de l'une de ses sous-classes.

**Exemple :**

```cpp
#include <iostream>
#include <exception>

class MaException : public std::exception {
public:
    const char* what() const noexcept override {
        return "Une erreur personnalisée est survenue";
    }
};

int main() {
    try {
        throw MaException();
    } catch (const MaException& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    return 0;
}
```

### 6.2.4. Propagation des exceptions

Les exceptions peuvent être propagées à travers les fonctions jusqu'à ce qu'elles soient capturées par un bloc `catch`. Cela permet de déléguer la gestion des erreurs à des niveaux supérieurs du code.

**Exemple :**

```cpp
#include <iostream>
#include <stdexcept>

void niveauBas() {
    throw std::runtime_error("Erreur au niveau bas");
}

void niveauIntermediaire() {
    niveauBas();
}

int main() {
    try {
        niveauIntermediaire();
    } catch (const std::exception& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    return 0;
}
```

### 6.2.5. Utilisation de `noexcept`

Le mot-clé `noexcept` est utilisé pour indiquer que la fonction ne lance pas d'exception. Cela peut aider le compilateur à optimiser le code.

**Exemple :**

```cpp
void fonctionNoexcept() noexcept {
    // Code qui ne lance pas d'exception
}
```

### 6.2.6. Bonnes pratiques de gestion des exceptions

- **Utiliser les exceptions pour les erreurs exceptionnelles** : Ne pas utiliser les exceptions pour le contrôle de flux normal.
- **Attraper les exceptions par référence constante** : Cela évite les copies inutiles et permet de capturer correctement les sous-classes.
- **Fournir des messages d'erreur clairs** : Utiliser la méthode `what()` pour donner des informations détaillées sur l'erreur.
- **Nettoyer les ressources** : Utiliser RAII (Resource Acquisition Is Initialization) pour garantir que les ressources sont libérées même en cas d'exception.
- **Propager les exceptions si nécessaire** : Permettre aux exceptions d'être capturées par un niveau supérieur si elles ne peuvent pas être correctement traitées à un niveau inférieur.

## 6.3. Espaces de noms (namespaces)

Les espaces de noms (namespaces) en C++ sont utilisés pour organiser le code et éviter les conflits de noms. Ils permettent de regrouper les déclarations de classes, de fonctions, de variables, et d'autres entités sous un même nom logique. Cela est particulièrement utile dans les grands projets où les mêmes noms peuvent être utilisés dans différentes parties du code.

### 6.3.1. Déclaration et utilisation des espaces de noms

Un espace de noms est déclaré à l'aide du mot-clé `namespace`, suivi du nom de l'espace de noms et d'un bloc englobant les déclarations.

#### 6.3.1.1. Déclaration d'un espace de noms

**Syntaxe :**

```cpp
namespace NomEspaceDeNoms {
    // Déclarations de classes, fonctions, variables, etc.
}
```

**Exemple :**

```cpp
namespace Math {
    const double PI = 3.14159;

    double carre(double x) {
        return x * x;
    }
}
```

#### 6.3.1.2. Utilisation des éléments d'un espace de noms

Pour accéder aux éléments d'un espace de noms, nous utilisons l'opérateur de résolution de portée `::`.

**Exemple :**

```cpp
#include <iostream>

int main() {
    std::cout << "PI : " << Math::PI << std::endl;
    std::cout << "Carré de 5 : " << Math::carre(5) << std::endl;
    return 0;
}
```

### 6.3.2. Espace de noms `std`

L'espace de noms `std` (standard) est utilisé par la bibliothèque standard C++. Toutes les entités de la bibliothèque standard, telles que les classes, les fonctions et les objets, sont définies dans cet espace de noms.

**Exemple :**

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    for (const auto& elem : v) {
        std::cout << elem << " ";
    }
    std::cout << std::endl;
    return 0;
}
```

### 6.3.3. Directives `using`

Les directives `using` permettent d'éviter d'utiliser l'opérateur de résolution de portée `::` à chaque fois que nous accédons aux éléments d'un espace de noms.

#### 6.3.3.1. Directive `using` pour une entité spécifique

**Syntaxe :**

```cpp
using NomEspaceDeNoms::NomEntite;
```

**Exemple :**

```cpp
#include <iostream>
using std::cout;

int main() {
    cout << "Bonjour, le monde!" << std::endl;
    return 0;
}
```

#### 6.3.3.2. Directive `using` pour un espace de noms entier

**Syntaxe :**

```cpp
using namespace NomEspaceDeNoms;
```

**Exemple :**

```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "Bonjour, le monde!" << endl;
    return 0;
}
```

### 6.3.4. Espaces de noms imbriqués

Les espaces de noms peuvent être imbriqués pour créer une hiérarchie logique de noms.

**Exemple :**

```cpp
namespace Entreprise {
    namespace Projet {
        void afficherMessage() {
            std::cout << "Message du projet de l'entreprise" << std::endl;
        }
    }
}

int main() {
    Entreprise::Projet::afficherMessage();
    return 0;
}
```

### 6.3.5. Alias d'espaces de noms

Les alias d'espaces de noms permettent de créer des noms plus courts ou plus significatifs pour les espaces de noms.

**Syntaxe :**

```cpp
namespace Alias = NomEspaceDeNoms;
```

**Exemple :**

```cpp
namespace CompteRendu = Entreprise::Projet;

int main() {
    CompteRendu::afficherMessage();
    return 0;
}
```

### 6.3.6. Bonnes pratiques avec les espaces de noms

- **Utiliser des espaces de noms pour les grandes bibliothèques et les projets** : Cela aide à organiser le code et à éviter les conflits de noms.
- **Éviter les directives `using` dans les en-têtes** : Pour prévenir les conflits de noms dans les fichiers qui incluent ces en-têtes.
- **Privilégier les alias d'espaces de noms pour la clarté et la simplicité** : Les alias permettent de réduire la longueur des noms tout en maintenant la lisibilité du code.
- **Utiliser des noms d'espaces de noms significatifs** : Les noms d'espaces de noms doivent refléter la structure logique et l'organisation du projet.

## 6.4. Introduction à la programmation multithread

# 7. Projets et Applications Pratiques

## 7.1. Développement d'une application console simple
## 7.2. Création d'une application avec interface graphique (optionnel)
## 7.3. Projet de fin de cours : Application C++ intégrant les concepts appris

# 8. Conclusion et Ressources pour Aller Plus Loin

## 8.1. Bonnes pratiques de développement en C++

Pour écrire du code C++ propre, maintenable et efficace, il est essentiel de suivre des bonnes pratiques de développement. Ces pratiques couvrent divers aspects du codage, de la gestion de la mémoire à la structure du code, en passant par l'utilisation de la bibliothèque standard et les techniques de débogage.

### 8.1.1. Utilisation de la bibliothèque standard

La bibliothèque standard C++ (STL) offre une vaste collection de classes et de fonctions pour des tâches courantes. Utiliser cette bibliothèque plutôt que d'écrire du code personnalisé peut améliorer la robustesse et la maintenabilité du code.

**Exemple :**

- Utiliser `std::vector` au lieu de tableaux dynamiques.
- Utiliser `std::string` pour manipuler des chaînes de caractères.
- Utiliser `std::unique_ptr` et `std::shared_ptr` pour gérer la mémoire.

### 8.1.2. Gestion de la mémoire

La gestion correcte de la mémoire est cruciale en C++. Les fuites de mémoire et les accès invalides peuvent entraîner des comportements imprévisibles et des plantages.

#### 8.1.2.1. Utilisation des pointeurs intelligents

Les pointeurs intelligents (`std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`) aident à gérer la durée de vie des objets et à prévenir les fuites de mémoire.

**Exemple :**

```cpp
#include <memory>

void utiliserPointeurIntelligent() {
    std::unique_ptr<int> ptr = std::make_unique<int>(42);
    std::cout << *ptr << std::endl;
} // ptr est automatiquement détruit ici
```

#### 8.1.2.2. RAII (Resource Acquisition Is Initialization)

RAII est une technique qui garantit que les ressources sont libérées lorsque les objets qui les possèdent sont détruits.

**Exemple :**

```cpp
#include <iostream>
#include <fstream>

void lireFichier(const std::string& nomFichier) {
    std::ifstream fichier(nomFichier);
    if (!fichier.is_open()) {
        throw std::runtime_error("Impossible d'ouvrir le fichier");
    }
    // Fichier automatiquement fermé lorsque `fichier` sort de la portée
}
```

### 8.1.3. Utilisation des const et constexpr

L'utilisation de `const` et `constexpr` améliore la sécurité du code en empêchant les modifications non intentionnelles et en permettant des optimisations à la compilation.

**Exemple :**

```cpp
const int tailleMax = 100;
constexpr int carre(int x) {
    return x * x;
}
```

### 8.1.4. Structuration et lisibilité du code

Un code bien structuré et lisible est plus facile à comprendre, à maintenir et à déboguer.

#### 8.1.4.1. Nommer les variables et fonctions de manière descriptive

Utiliser des noms de variables et de fonctions qui décrivent clairement leur rôle et leur contenu.

**Exemple :**

```cpp
int calculerSomme(int a, int b) {
    return a + b;
}
```

#### 8.1.4.2. Indentation et espacement

Respecter les conventions d'indentation et d'espacement pour améliorer la lisibilité.

**Exemple :**

```cpp
int main() {
    if (true) {
        std::cout << "Bonjour, le monde!" << std::endl;
    }
    return 0;
}
```

### 8.1.5. Utilisation des exceptions pour la gestion des erreurs

Les exceptions permettent de gérer les erreurs de manière propre et cohérente.

**Exemple :**

```cpp
#include <iostream>
#include <stdexcept>

void verifierValeur(int valeur) {
    if (valeur < 0) {
        throw std::invalid_argument("Valeur négative non permise");
    }
}

int main() {
    try {
        verifierValeur(-1);
    } catch (const std::invalid_argument& e) {
        std::cerr << "Erreur : " << e.what() << std::endl;
    }
    return 0;
}
```

### 8.1.6. Tests et validation

Écrire des tests unitaires pour vérifier le bon fonctionnement des fonctions et des classes. Utiliser des frameworks de tests comme Google Test pour automatiser et structurer les tests.

**Exemple :**

```cpp
#include <gtest/gtest.h>

int addition(int a, int b) {
    return a + b;
}

TEST(AdditionTest, Positif) {
    EXPECT_EQ(addition(1, 2), 3);
    EXPECT_EQ(addition(10, 20), 30);
}

int main(int argc, char **argv) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

### 8.1.7. Documentation et commentaires

Documenter le code pour expliquer les intentions, les concepts complexes et les choix de conception.

**Exemple :**

```cpp
/**
 * @brief Calcule la somme de deux entiers.
 *
 * @param a Premier entier.
 * @param b Deuxième entier.
 * @return La somme de a et b.
 */
int addition(int a, int b) {
    return a + b;
}
```

### 8.1.8. Optimisation

Optimiser le code uniquement lorsque cela est nécessaire et après avoir identifié les goulots d'étranglement à l'aide d'outils de profilage.

**Exemple :**

```cpp
#include <vector>
#include <algorithm>

// Éviter de réallouer la mémoire en réservant de l'espace à l'avance
std::vector<int> v;
v.reserve(100);
std::generate_n(std::back_inserter(v), 100, []{ return rand() % 100; });
```

## 8.2. Outils de développement et environnements intégrés (IDE)

Le choix des outils de développement et des environnements intégrés (IDE) est crucial pour optimiser le flux de travail et améliorer la productivité des développeurs C++. Sur Ubuntu, une combinaison de gcc (GNU Compiler Collection) et Visual Studio Code (VS Code) constitue un environnement de développement puissant et flexible.

### 8.2.1. GCC (GNU Compiler Collection)

GCC est un ensemble de compilateurs pour différents langages de programmation, dont C et C++. C'est l'un des compilateurs les plus populaires et largement utilisés dans le monde Linux.

#### 8.2.1.1. Installation de GCC

Sur Ubuntu, GCC peut être installé via le gestionnaire de paquets `apt`.

**Commande :**

```bash
sudo apt update
sudo apt install build-essential
```

Le paquet `build-essential` inclut GCC, G++ (le compilateur C++), ainsi que d'autres outils de développement nécessaires.

#### 8.2.1.2. Compilation avec GCC

Pour compiler un programme C++ avec GCC, on utilise la commande `g++`.

**Exemple :**

```cpp
// fichier main.cpp
#include <iostream>

int main() {
    std::cout << "Bonjour, le monde!" << std::endl;
    return 0;
}
```

**Commande de compilation :**

```bash
g++ -o monprogramme main.cpp
```

Cette commande compile `main.cpp` et génère un exécutable nommé `monprogramme`.

#### 8.2.1.3. Options courantes de GCC

- **`-Wall`** : Active les messages d'avertissement.
- **`-O2`** : Active les optimisations de niveau 2.
- **`-g`** : Génère des informations de débogage.
- **`-std=c++17`** : Utilise la norme C++17.

**Exemple :**

```bash
g++ -Wall -O2 -g -std=c++17 -o monprogramme main.cpp
```

### 8.2.2. Visual Studio Code (VS Code)

Visual Studio Code est un éditeur de code source développé par Microsoft. Il est léger, extensible et supporte de nombreux langages de programmation, dont C++. Il est particulièrement populaire pour son interface utilisateur intuitive et ses nombreuses extensions.

#### 8.2.2.1. Installation de VS Code

Sur Ubuntu, VS Code peut être installé via le gestionnaire de paquets `snap` ou `apt`.

**Commande snap :**

```bash
sudo snap install --classic code
```

**Commande apt :**

```bash
sudo apt update
sudo apt install code
```

#### 8.2.2.2. Configuration de VS Code pour C++

Après l'installation, quelques extensions et configurations supplémentaires sont nécessaires pour optimiser VS Code pour le développement C++.

**Extensions recommandées :**

- **C/C++** : Fournit un support de base pour le langage, y compris IntelliSense, le débogage et la gestion des configurations de build.
- **CMake Tools** : Simplifie la gestion des projets CMake.
- **Code Runner** : Permet d'exécuter rapidement des snippets de code.

**Installation des extensions :**

1. Ouvrir VS Code.
2. Aller dans l'onglet Extensions (icône des blocs sur la barre latérale gauche).
3. Rechercher et installer les extensions mentionnées ci-dessus.

#### 8.2.2.3. Configuration de IntelliSense

IntelliSense offre des fonctionnalités d'autocomplétion, de vérification de syntaxe et de documentation contextuelle.

**Configurer `c_cpp_properties.json` :**

1. Ouvrir la palette de commandes (Ctrl+Shift+P).
2. Taper `C/C++: Edit Configurations (UI)` et sélectionner l'option.
3. Configurer les chemins d'inclusion et les options du compilateur.

**Exemple :**

```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "/usr/include",
                "/usr/local/include"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/g++",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "gcc-x64"
        }
    ],
    "version": 4
}
```

#### 8.2.2.4. Débogage avec VS Code

VS Code dispose d'un débogueur intégré puissant. Pour configurer le débogueur pour un projet C++ :

1. Ouvrir la palette de commandes (Ctrl+Shift+P).
2. Taper `Debug: Open launch.json` et sélectionner l'option.
3. Ajouter une configuration de débogage.

**Exemple :**

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Déboguer C++",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/monprogramme",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Activer l'impression jpretty",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "build",
            "miDebuggerPath": "/usr/bin/gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "logging": {
                "engineLogging": true
            }
        }
    ]
}
```

### 8.2.3. Intégration avec CMake

CMake est un outil de génération de build qui gère la compilation de projets multi-plateformes. Il est souvent utilisé en combinaison avec GCC et VS Code pour gérer des projets C++ complexes.

#### 8.2.3.1. Installation de CMake

**Commande :**

```bash
sudo apt install cmake
```

#### 8.2.3.2. Configuration de CMake

Créer un fichier `CMakeLists.txt` à la racine de votre projet.

**Exemple :**

```cmake
cmake_minimum_required(VERSION 3.10)
project(MonProjet)

set(CMAKE_CXX_STANDARD 17)

add_executable(monprogramme main.cpp)
```

#### 8.2.3.3. Intégration de CMake dans VS Code

1. Installer l'extension **CMake Tools**.
2. Ouvrir la palette de commandes (Ctrl+Shift+P).
3. Taper `CMake: Configure` pour configurer le projet.
4. Taper `CMake: Build` pour construire le projet.
5. Taper `CMake: Run Without Debugging` pour exécuter le projet.

# Module 8 : Conclusion et Ressources pour Aller Plus Loin

## 8.3 Ressources pour approfondir ses connaissances en C++

Pour devenir un expert en C++, il est essentiel d'explorer une variété de ressources d'apprentissage. Les livres, les cours en ligne, les tutoriels, et les documentations officielles sont autant de moyens efficaces pour approfondir ses connaissances. Voici une liste de ressources recommandées pour les développeurs C++ souhaitant améliorer leurs compétences.

### 8.3.1 Livres

Les livres offrent une source d'information approfondie et structurée. Voici quelques ouvrages de référence pour le développement en C++.

#### 8.3.1.1 "The C++ Programming Language" par Bjarne Stroustrup

Ce livre est écrit par le créateur du langage C++, Bjarne Stroustrup. Il couvre en détail les concepts fondamentaux et avancés du C++.

**Points clés :**
- Historique et philosophie du langage
- Concepts de base et avancés
- Programmation générique
- Programmation orientée objet
- Bibliothèque standard C++

#### 8.3.1.2 "Effective Modern C++" par Scott Meyers

Ce livre est indispensable pour les développeurs souhaitant maîtriser les nouvelles fonctionnalités introduites dans les versions modernes de C++ (C++11 et C++14).

**Points clés :**
- Utilisation des nouvelles fonctionnalités de C++11 et C++14
- Meilleures pratiques et idiomes modernes
- Gestion des ressources et des exceptions
- Programmation concurrente

#### 8.3.1.3 "C++ Primer" par Stanley B. Lippman, Josée Lajoie, et Barbara E. Moo

Ce livre est parfait pour les débutants ainsi que pour les développeurs ayant une certaine expérience en C++.

**Points clés :**
- Introduction au langage C++
- Concepts de base et intermédiaires
- Programmation orientée objet
- Templates et STL
- Programmation avancée

### 8.3.2 Cours en ligne et tutoriels

Les cours en ligne et les tutoriels offrent une approche interactive pour l'apprentissage du C++. Voici quelques plateformes et cours recommandés.

#### 8.3.2.1 Coursera

**Cours recommandés :**
- **"C++ For C Programmers, Part A"** par l'Université de Californie, Santa Cruz
- **"C++ For C Programmers, Part B"** par l'Université de Californie, Santa Cruz

Ces cours couvrent les bases du C++ pour les développeurs ayant déjà une expérience en C.

#### 8.3.2.2 edX

**Cours recommandés :**
- **"Introduction to C++"** par Microsoft
- **"Advanced C++"** par Microsoft

Ces cours offrent une introduction complète au C++ ainsi que des sujets avancés pour les développeurs expérimentés.

#### 8.3.2.3 Udemy

**Cours recommandés :**
- **"Beginning C++ Programming - From Beginner to Beyond"** par Tim Buchalka
- **"Unreal Engine C++ Developer: Learn C++ and Make Video Games"** par Ben Tristem et GameDev.tv

Ces cours sont parfaits pour les débutants ainsi que pour ceux intéressés par le développement de jeux vidéo avec C++.

#### 8.3.2.4 FUN MOOC

**Cours recommandés :**

- **"Programmation en C++"** par l'Institut Mines-Télécom
    - Ce cours couvre les concepts fondamentaux de la programmation en C++ et est idéal pour les débutants et les intermédiaires.

#### 8.3.2.5 Developpez.com

**Ressources recommandées :**

- **Tutoriels et articles** : [Developpez.com](https://cpp.developpez.com) propose une vaste collection de tutoriels, d'articles et de forums dédiés à C++.
    - Tutoriels C++
    - Articles C++

#### 8.3.2.7 OpenClassrooms

**Cours recommandés :**

- **["Apprenez à programmer en C++"](https://openclassrooms.com/fr/courses/19980-apprenez-a-programmer-en-c)** : Ce cours gratuit d'OpenClassrooms couvre les bases du langage C++ et permet de se familiariser avec les concepts fondamentaux.
### 8.3.3 Documentation et sites web

La documentation officielle et les sites web dédiés sont des ressources essentielles pour rester à jour avec les dernières évolutions du langage C++.

#### 8.3.3.1 Documentation officielle

**[cppreference.com ](cppreference.com):**
- Une référence complète pour les bibliothèques C++ standard et les fonctionnalités du langage.
- Inclut des exemples de code et des explications détaillées.

**cplusplus.com :**
- Un site web dédié aux tutoriels et à la documentation C++.
- Inclut des guides de référence pour la bibliothèque standard C++.

#### 8.3.3.2 Stack Overflow

Stack Overflow est une communauté en ligne où les développeurs peuvent poser des questions et obtenir des réponses sur divers sujets de programmation, y compris C++.

**Utilisation :**
- Rechercher des solutions aux problèmes courants.
- Poser des questions spécifiques et obtenir des réponses de la communauté.
- Participer aux discussions et aider d'autres développeurs.

#### 8.3.3.3 GitHub

GitHub est une plateforme de développement collaboratif où les développeurs peuvent héberger et partager leur code.

**Utilisation :**
- Explorer les projets open source en C++.
- Contribuer à des projets existants.
- Apprendre des meilleures pratiques en étudiant le code des autres développeurs.

### 8.3.4 Forums et communautés

Les forums et les communautés en ligne offrent des opportunités de réseautage et d'apprentissage continu.

#### 8.3.4.1. Reddit

**Subreddits recommandés :**
- **r/cpp** : Un subreddit dédié aux discussions sur C++.
- **r/learnprogramming** : Un subreddit pour les débutants en programmation.

#### 8.3.4.2. C++ Forum

**[cplusplus.com ](https://cplusplus.com):**
- Un forum dédié aux discussions sur C++.
- Inclut des sections pour les débutants, les questions avancées, et les discussions générales.

