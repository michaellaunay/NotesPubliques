---
schema_version: 1
uid: 01M02EX5AV43KMZP49615V34WC
titre: C++
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - programmation
  - cpp
  - conception-orientee-objet
resume: "Cours complet de C++ moderne : fondamentaux, RAII, STL, templates, concurrence, C++20/C++23, aperçu C++26, CMake, tests, sécurité mémoire et projets."
niveau: intermediaire
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2024-03-24
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# C++

> [!abstract] Objectif
> Apprendre à écrire du **C++ moderne (C++20/C++23) sûr, testable et maintenable** : maîtriser le noyau du langage et la programmation orientée objet, puis laisser RAII, la bibliothèque standard et l'outillage (CMake, sanitizers, tests) prendre en charge les ressources et la qualité.

Ce cours suit une progression du langage vers l'ingénierie logicielle moderne :

1. histoire, outils, compilation et structure d'un programme ;
2. types, variables, opérateurs, structures de contrôle, fonctions et fondamentaux du C++ moderne ;
3. programmation orientée objet ;
4. durée de vie, ownership, RAII, pointeurs et mémoire ;
5. bibliothèque standard, conteneurs, chaînes et fichiers ;
6. templates, exceptions, concurrence et fonctionnalités de C++20/C++23 ;
7. projets et travaux pratiques ;
8. bonnes pratiques, chaîne d'outils, tests et ressources.

À l'issue du cours, l'objectif n'est pas seulement de savoir écrire du C++ syntaxiquement valide, mais de savoir écrire du **C++ moderne, sûr, testable et maintenable**, en limitant l'utilisation de `new`/`delete` explicites et en laissant les abstractions de la bibliothèque standard gérer les ressources autant que possible.

> [!important]
> Les exemples historiques utilisant des pointeurs bruts, `new`, `delete` ou `using namespace std;` restent utiles pour comprendre le langage et lire du code existant. Ils ne constituent pas la forme recommandée pour un nouveau projet ; les sections 2.5 et 3.5 présentent les idiomes attendus aujourd'hui.

Voir aussi : [[Histoire des langages de programmation]], [[Principes SOLID en COO]], [[Design patterns]], [[Visual studio code]], [[git]].

# Sommaire

1. Introduction au C++ : histoire, environnement, compilation et structure d'un programme
2. Bases du langage : types, opérateurs, structures de contrôle, fonctions, fondamentaux du C++ moderne
3. Programmation orientée objet : classes, encapsulation, héritage, polymorphisme, classes modernes
4. Gestion de la mémoire : allocation dynamique, pointeurs et références, RAII, fuites
5. Bibliothèque standard : STL, conteneurs, itérateurs et algorithmes, chaînes et fichiers, choix d'un conteneur
6. Développement avancé : templates, exceptions, espaces de noms, concurrence, C++20/C++23, aperçu de C++26, qualité et performance
7. Projets et travaux pratiques
8. Bonnes pratiques, chaîne d'outils, ressources et checklist

# 1. Introduction au C++

## 1.1. Présentation du C++ : histoire et applications

### 1.1.1. Histoire du C++

Le C++ est un langage de programmation de haut niveau qui a été développé par Bjarne Stroustrup au début des années 1980, au sein des laboratoires Bell de AT&T. Il est conçu comme une extension du langage C, dans le but d'ajouter des fonctionnalités orientées objet tout en conservant l'efficacité et la flexibilité du C. La première version officielle du C++ est apparue en 1983, et le langage a depuis subi de nombreuses évolutions et standardisations.

En 1998, le premier standard ISO/IEC pour le C++ a été publié sous le nom de C++98. Il a été suivi par C++03, C++11, C++14, C++17, C++20 puis C++23. C++23 a été finalisé en 2023 et publié par l'ISO en 2024 (ISO/IEC 14882:2024). Le comité WG21 a achevé le travail technique sur **C++26** le 28 mars 2026 ; le texte a été enregistré comme *Draft International Standard* en juin 2026 et sa publication officielle par l'ISO est attendue d'ici la fin de l'année. À la date de mise à jour de ce cours (août 2026), les compilateurs implémentent déjà une part importante de C++26 — GCC 16.1 propose par exemple la réflexion (`-freflection`) et les contrats (`-fcontracts`) — mais cette prise en charge reste inégale et souvent qualifiée d'expérimentale.

Pour un nouveau projet, ce cours recommande :

- **C++23** lorsque la chaîne de compilation ciblée implémente les fonctionnalités nécessaires ;
- **C++20** lorsqu'une compatibilité plus large est requise ;
- **C++26** uniquement après vérification précise du support du compilateur et de la bibliothèque standard.

Le C++ moderne ne se résume pas à « ajouter de la programmation orientée objet au C ». Depuis C++11, son style idiomatique repose fortement sur **RAII**, la bibliothèque standard, les valeurs et références, les algorithmes, les templates, les lambdas, les concepts, les ranges et une gestion explicite de l'ownership.

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

Les exemples de ce cours ciblent principalement GNU/Linux. Ubuntu 24.04 LTS et Ubuntu 26.04 LTS constituent des bases adaptées ; les commandes sont similaires sur Debian et ses dérivées.

### 1.2.1. Installer une chaîne de compilation minimale

```bash
sudo apt update
sudo apt install build-essential gdb cmake ninja-build pkg-config
```

Le paquet `build-essential` installe notamment `g++`, `gcc` et `make`.

Vérifions les outils :

```bash
g++ --version
gdb --version
cmake --version
ninja --version
```

Il est utile d'avoir **plus d'un compilateur** lors du développement : des diagnostics différents permettent de détecter davantage de problèmes.

```bash
sudo apt install clang clangd clang-format clang-tidy
```

### 1.2.2. Choisir explicitement le dialecte C++

Un compilateur n'active pas nécessairement le standard désiré par défaut : GCC 11 à 15 compilent en `gnu++17`, GCC 16 en `gnu++20`, et le dialecte `gnu++` active des extensions non standard. Pour ce cours, nous indiquons donc explicitement le dialecte :

```bash
g++ -std=c++23 main.cpp -o programme
```

Pour viser C++20 :

```bash
g++ -std=c++20 main.cpp -o programme
```

Pour expérimenter C++26 avec GCC récent :

```bash
g++ -std=c++26 main.cpp -o programme
```

Le support de C++26 est encore à considérer comme **expérimental** : une fonctionnalité du langage peut être implémentée alors que la partie correspondante de la bibliothèque standard ne l'est pas encore.

### 1.2.3. Activer les avertissements

Une compilation sans avertissement est un premier niveau de contrôle qualité. Pendant le développement :

```bash
g++ -std=c++23 \
    -Wall -Wextra -Wpedantic \
    -Wconversion -Wshadow \
    -g \
    main.cpp -o programme
```

Les avertissements ne sont pas tous des erreurs ; il faut les comprendre avant de décider de les ignorer.

### 1.2.4. Utiliser les sanitizers en développement

Les sanitizers permettent de détecter de nombreuses erreurs qui peuvent autrement produire un comportement indéfini silencieux.

```bash
g++ -std=c++23 \
    -Wall -Wextra -Wpedantic \
    -g -O1 \
    -fsanitize=address,undefined \
    -fno-omit-frame-pointer \
    main.cpp -o programme

./programme
```

Les deux sanitizers les plus utiles pour commencer sont :

- **AddressSanitizer (ASan)** : dépassements de tampon, use-after-free, etc. ;
- **UndefinedBehaviorSanitizer (UBSan)** : de nombreux comportements indéfinis.

ThreadSanitizer (`-fsanitize=thread`) est utile pour rechercher les data races, mais il ne se combine généralement pas avec AddressSanitizer dans la même exécution.

### 1.2.5. Visual Studio Code

La configuration générale de VS Code est détaillée dans [[Visual studio code]]. Pour C++, les extensions les plus utiles sont :

- **C/C++** ou **clangd** pour l'analyse du code ;
- **CMake Tools** pour les projets CMake ;
- éventuellement **clang-format** / **clang-tidy** selon la chaîne choisie.

Pour un vrai projet, il est préférable de laisser **CMake** décrire la compilation plutôt que de maintenir manuellement une commande différente dans `tasks.json`, l'IDE, la CI et la documentation.

### 1.2.6. Premier projet avec CMake

Arborescence minimale :

```text
bonjour/
├── CMakeLists.txt
└── src/
    └── main.cpp
```

`src/main.cpp` :

```cpp
#include <iostream>

int main() {
    std::cout << "Bonjour C++ !\n";
}
```

`CMakeLists.txt` :

```cmake
cmake_minimum_required(VERSION 3.25)
project(BonjourCpp LANGUAGES CXX)

add_executable(bonjour src/main.cpp)
target_compile_features(bonjour PRIVATE cxx_std_23)

if (CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(bonjour PRIVATE -Wall -Wextra -Wpedantic)
endif()
```

Configuration et compilation :

```bash
cmake -S . -B build -G Ninja
cmake --build build
./build/bonjour
```

L'approche **target-based** (`target_compile_features`, `target_link_libraries`, etc.) est préférable à la modification globale de variables comme `CMAKE_CXX_FLAGS`.

### 1.2.7. Organisation conseillée d'un projet

```text
mon-projet/
├── CMakeLists.txt
├── cmake/
├── include/
│   └── monprojet/
├── src/
├── tests/
├── apps/
├── README.md
└── .clang-format
```

Les fichiers générés par CMake restent dans `build/`, qui ne doit généralement pas être versionné.

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

> [!tip]
> En C++ moderne, une constante se déclare avec `constexpr` plutôt qu'avec une macro : `constexpr double pi = 3.14159;`, ou `std::numbers::pi` depuis C++20 (`<numbers>`). Une constante typée respecte les portées et les espaces de noms, apparaît dans le débogueur et ne subit pas les pièges de la substitution textuelle des macros (voir section 2.5.7). `#define` reste utile pour la compilation conditionnelle (`#if`, `#ifdef`) et les *feature-test macros*.

### 1.3.2. Espace de noms

En C++, les espaces de noms (namespaces) sont utilisés pour organiser le code et éviter les conflits de noms. Le plus courant est le namespace `std`, qui contient toutes les fonctionnalités standard de la bibliothèque C++. Par exemple :

```cpp
using namespace std;
```

Cette ligne permet d'utiliser les éléments du namespace `std` sans avoir à le préfixer chaque fois.

> [!warning]
> `using namespace std;` est pratique dans un petit exemple pédagogique, mais il est déconseillé au niveau global d'un programme réel et particulièrement dans un fichier d'en-tête. Préférer `std::cout`, `std::string`, etc., ou des déclarations `using std::cout;` ciblées dans une portée locale.

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
g++ -std=c++23 -Wall -Wextra -Wpedantic bonjour.cpp -o bonjour
```

Cette commande appelle g++ pour compiler `bonjour.cpp` et génère un exécutable nommé `bonjour`. Voici une explication des options utilisées :

- `g++` : le compilateur C++ de GNU ;
- `-std=c++23` : sélectionne le dialecte du langage ;
- `-Wall -Wextra -Wpedantic` : active un ensemble utile d'avertissements ;
- `bonjour.cpp` : le fichier source à compiler ;
- `-o bonjour` : spécifie le nom du fichier exécutable généré.

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

Pour un fichier isolé, VS Code peut lancer directement une commande de compilation. Pour un projet réel, il est préférable d'utiliser la même configuration **CMake** dans le terminal, l'éditeur et la CI.

Avec l'extension **CMake Tools** :

1. ouvrir le dossier contenant `CMakeLists.txt` ;
2. sélectionner un kit/compiler si nécessaire ;
3. lancer `CMake: Configure` ;
4. lancer `CMake: Build` ;
5. utiliser `Run and Debug` pour déboguer la cible sélectionnée.

Cette approche évite de recopier les options du compilateur dans plusieurs fichiers `tasks.json` et réduit les écarts entre le build local et le build automatisé.

Pour les détails sur les profils, les tâches, le débogage et les environnements distants, voir [[Visual studio code]].

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

### 2.1.2. Types de données dérivés

En plus des types primitifs, C++ permet de créer des types de données dérivés, tels que les tableaux, les pointeurs, et les références.

#### 2.1.2.1. Tableaux

Un tableau est une collection de variables du même type, stockées en séquence. Par exemple :

```cpp
int nombres[5] = {1, 2, 3, 4, 5};
```

En C++ moderne, on préfère `std::array<int, 5>` (taille fixe) ou `std::vector<int>` (taille dynamique) aux tableaux C, qui ne connaissent pas leur taille et se dégradent en simple pointeur dès qu'on les passe à une fonction (voir section 2.5.5).

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

## 2.4. Fonctions : Déclaration, définition et appel

Les fonctions en C++ sont des blocs de code réutilisables qui permettent de réaliser des tâches spécifiques. Elles facilitent la structuration, la lisibilité et la maintenance du code. Cette section explique comment déclarer, définir et appeler des fonctions en C++.

### 2.4.1. Déclaration de fonctions

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

### 2.4.2. Définition de fonctions

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

### 2.4.3. Appel de fonctions

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

### 2.4.4. Fonctions avec paramètres

Les paramètres d'une fonction permettent de passer des valeurs à cette fonction pour qu'elle les utilise dans son traitement. Les paramètres sont spécifiés dans la liste de paramètres lors de la déclaration et de la définition de la fonction.

**Exemple :**

```cpp
int multiplier(int a, int b) {
    return a * b;
}

int resultat = multiplier(4, 5); // résultat vaut 20
```

### 2.4.5. Fonctions sans paramètres

Une fonction peut ne pas nécessiter de paramètres. Dans ce cas, les parenthèses sont laissées vides.

**Exemple :**

```cpp
void direBonjour() {
    cout << "Bonjour!" << endl;
}

direBonjour();
```

### 2.4.6. Fonctions avec valeurs de retour

Une fonction peut renvoyer une valeur à l'aide de l'instruction `return`. Le type de la valeur de retour doit correspondre au type de retour spécifié dans la déclaration de la fonction.

**Exemple :**

```cpp
double calculerMoyenne(double note1, double note2) {
    return (note1 + note2) / 2;
}

double moyenne = calculerMoyenne(15.5, 18.0); // moyenne vaut 16.75
```

### 2.4.7. Fonctions void

Les fonctions qui ne renvoient aucune valeur utilisent le type de retour `void`.

**Exemple :**

```cpp
void afficherBienvenue() {
    cout << "Bienvenue dans notre programme!" << endl;
}

afficherBienvenue();
```

### 2.4.8. Surcharge de fonctions

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

### 2.4.9. Fonctions récursives

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

## 2.5. Fondamentaux du C++ moderne

Les sections précédentes présentent le noyau du langage tel qu'il existe depuis C++98. Depuis C++11, un ensemble de mots-clés et d'idiomes est devenu la base de tout code C++ actuel. La suite du cours les utilise sans les réexpliquer ; cette section les regroupe.

### 2.5.1. Déduction de type avec `auto`

`auto` demande au compilateur de déduire le type d'une variable à partir de son initialiseur. Il évite de recopier des noms de types longs (itérateurs, lambdas, types de retour de la bibliothèque standard) et rend le code plus robuste aux changements de type.

```cpp
#include <iostream>
#include <map>
#include <string>
#include <vector>

int main() {
    auto entier = 42;                 // int
    auto reel = 3.14;                 // double
    auto texte = std::string{"Ada"};  // std::string

    std::map<std::string, std::vector<int>> notes{{"Ada", {18, 20}}};
    auto it = notes.find("Ada");      // std::map<std::string, std::vector<int>>::iterator

    const auto& premiere = it->second.front();   // référence constante, sans copie
    std::cout << entier << ' ' << reel << ' ' << texte << ' ' << premiere << '\n';
}
```

Règles pratiques :

- `auto` seul déduit une **valeur**, donc une copie : écrire `auto&` ou `const auto&` pour obtenir une référence ;
- `auto` abandonne le `const` et la référence de l'initialiseur ; `decltype(auto)` les conserve ;
- garder un type explicite lorsque celui-ci porte une information importante (`double moyenne = total / n;` évite une division entière surprise).

### 2.5.2. Initialisation par accolades

C++11 généralise l'initialisation par accolades (*uniform initialization*). Elle refuse les conversions rétrécissantes (*narrowing*) et `{}` fournit une valeur par défaut bien définie :

```cpp
#include <vector>

int main() {
    int a{42};                    // OK
    int b{};                      // 0 : initialisation de valeur, jamais indéterminée
    double d{3.5};
    // int c{3.5};                // erreur de compilation : conversion rétrécissante
    std::vector<int> v{1, 2, 3};  // liste d'initialisation : trois éléments 1, 2, 3
    std::vector<int> w(3, 7);     // parenthèses : trois éléments valant 7
    return a + b + static_cast<int>(d) + static_cast<int>(v.size() + w.size());
}
```

> [!warning]
> Une variable locale de type fondamental déclarée sans initialiseur (`int n;`) contient une valeur indéterminée ; la lire est un comportement indéfini (C++26 requalifie ce cas en « comportement erroné », diagnosticable mais toujours à proscrire). Toujours initialiser : `int n{};`.

### 2.5.3. `nullptr`

`nullptr` remplace `NULL` et `0` pour désigner un pointeur nul. Son type, `std::nullptr_t`, ne se convertit pas en entier, ce qui supprime des ambiguïtés de surcharge.

```cpp
void f(int);
void f(const char*);

void exemple() {
    int* p = nullptr;
    f(nullptr);   // appelle f(const char*) ; f(0) appellerait f(int)
    f(p != nullptr);
}
```

### 2.5.4. Énumérations fortement typées : `enum class`

```cpp
#include <cstdint>

enum class Couleur { rouge, vert, bleu };
enum class Feu : std::uint8_t { rouge, orange, vert };   // type sous-jacent explicite

int code(Couleur c) {
    switch (c) {                       // -Wall signale les cas oubliés
    case Couleur::rouge: return 1;
    case Couleur::vert:  return 2;
    case Couleur::bleu:  return 3;
    }
    return 0;
}

int main() {
    Couleur c = Couleur::vert;
    // int n = c;                      // erreur : pas de conversion implicite vers int
    int n = static_cast<int>(c);
    return code(c) + n + static_cast<int>(Feu::orange);
}
```

Contrairement aux `enum` classiques, les énumérateurs sont **qualifiés** (`Couleur::vert`), ne polluent pas la portée englobante, ne se convertissent pas implicitement en entier et peuvent préciser leur type sous-jacent.

### 2.5.5. `std::array`, `std::vector` et boucle `for` sur une plage

Un tableau C (`int t[5]`) se dégrade en pointeur dès qu'on le passe à une fonction, ne connaît pas sa taille et ne se copie pas. Le C++ moderne lui préfère `std::array` (taille fixe connue à la compilation, aucune allocation) et `std::vector` (taille dynamique).

```cpp
#include <array>
#include <iostream>
#include <vector>

int main() {
    std::array<int, 3> fixe{1, 2, 3};
    std::vector<int> dynamique{4, 5, 6};
    dynamique.push_back(7);

    for (int x : fixe) {               // copie chaque élément (fine pour un int)
        std::cout << x << ' ';
    }
    for (const auto& x : dynamique) {  // sans copie, sans modification
        std::cout << x << ' ';
    }
    for (auto& x : dynamique) {        // modification en place
        x *= 2;
    }
    std::cout << '\n' << fixe.size() << ' ' << dynamique.size() << '\n';
}
```

La boucle `for` sur une plage (*range-based for*, C++11) parcourt tout objet exposant `begin()` et `end()` : conteneurs standard, tableaux C, chaînes, vues `ranges`. Elle élimine les erreurs d'indice et exprime l'intention.

> [!warning]
> Ne jamais ajouter ni supprimer d'éléments d'un conteneur pendant qu'on le parcourt avec une boucle sur une plage : les itérateurs peuvent être invalidés (voir section 5.5.3).

### 2.5.6. Types entiers de taille fixe et `std::size_t`

`int`, `long`... n'ont pas de taille garantie par le standard. Lorsque la taille compte (protocoles réseau, fichiers binaires, registres matériels), utiliser les alias de `<cstdint>` :

```cpp
#include <cstddef>
#include <cstdint>
#include <vector>

int main() {
    std::uint8_t  octet{0xFF};
    std::int32_t  entier32{-1};
    std::uint64_t compteur{};

    std::vector<int> v(10);
    std::size_t taille = v.size();     // type non signé retourné par sizeof et size()
    for (std::size_t i = 0; i < taille; ++i) {
        v[i] = static_cast<int>(i);
    }
    return octet + entier32 + static_cast<int>(compteur) + v.back();
}
```

`std::size_t` est le type des tailles et des indices dans la bibliothèque standard. Comparer un entier signé et un entier non signé produit un avertissement (`-Wsign-compare`) et des bugs classiques (`-1 < v.size()` est faux) ; C++20 fournit `std::ssize()` et les fonctions `std::cmp_less`, `std::cmp_equal`... de `<utility>` pour comparer correctement.

### 2.5.7. Calcul à la compilation : `constexpr`, `consteval` et `static_assert`

```cpp
#include <numbers>

constexpr double pi = std::numbers::pi;          // constante évaluée à la compilation

constexpr int carre(int x) { return x * x; }      // utilisable à la compilation comme à l'exécution

consteval int cube(int x) { return x * x * x; }   // C++20 : uniquement à la compilation

static_assert(carre(12) == 144, "carre est incorrecte");
static_assert(cube(3) == 27);
static_assert(sizeof(int) >= 4, "int de moins de 32 bits non pris en charge");

int main() {
    int n = 7;
    return carre(n) > pi ? 0 : 1;                  // appel à l'exécution
}
```

`constexpr` remplace les macros `#define` pour les constantes : la constante est typée, respecte les portées et les espaces de noms, et se voit dans le débogueur. `static_assert` vérifie un invariant dès la compilation.

### 2.5.8. Liaisons structurées et `if` avec initialisation (C++17)

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> ages{{"Ada", 36}, {"Linus", 56}};

    for (const auto& [nom, age] : ages) {                // liaison structurée sur la paire clé/valeur
        std::cout << nom << " : " << age << '\n';
    }

    if (auto it = ages.find("Ada"); it != ages.end()) {  // if avec initialisation
        std::cout << it->second << '\n';
    }

    auto [position, insere] = ages.insert({"Grace", 85});
    std::cout << std::boolalpha << insere << ' ' << position->first << '\n';
}
```

Les liaisons structurées décomposent une paire, un tuple, un tableau ou une structure simple en variables nommées. L'instruction `if` (et `switch`) avec initialisation limite la portée d'une variable au bloc conditionnel.

### 2.5.9. Attributs utiles

```cpp
[[nodiscard]] int lire_configuration();       // avertit si le résultat est ignoré

void traiter(int code, [[maybe_unused]] int niveau_debug) {
    switch (code) {
    case 1:
        // préparation
        [[fallthrough]];                      // chute volontaire vers le cas suivant
    case 2:
        // exécution
        break;
    default:
        break;
    }
}
```

`[[nodiscard]]` (C++17) est recommandé sur toute fonction dont ignorer le résultat est presque toujours une erreur : codes d'erreur, `std::expected`, fonctions « pures » comme `empty()`. `[[fallthrough]]` documente une chute volontaire dans un `switch` et fait taire `-Wimplicit-fallthrough`.

### 2.5.10. Passer des paramètres : valeur, référence, référence constante

| Situation | Signature conseillée |
|---|---|
| Petit type bon marché à copier (`int`, `double`, pointeur, `std::string_view`) | par valeur : `int n` |
| Objet coûteux à copier, lecture seule | `const T&` : `const std::string& s` |
| Objet que la fonction doit modifier | `T&` : `std::vector<int>& v` |
| Objet dont la fonction prend possession (stockage dans un membre) | par valeur puis `std::move(s)` |
| Argument optionnel sans transfert de possession | `std::optional<T>` ou pointeur non propriétaire |

Ces conventions suivent les *C++ Core Guidelines* (règles F.15 à F.20). Retourner par valeur est la norme : le compilateur élide les copies (RVO) et les types standard se déplacent efficacement (section 6.5).

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
    virtual ~Personne() = default;

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
#include <memory>

int main() {
    std::unique_ptr<Personne> p = std::make_unique<Etudiant>();
    p->sePresenter(); // Affiche "Bonjour, je suis un étudiant."
}
```

Dans cet exemple, `p` possède un `Etudiant` via l'interface `Personne`. Le dispatch virtuel appelle `Etudiant::sePresenter()` et `std::unique_ptr` libère automatiquement l'objet. Une classe destinée à être détruite polymorphiquement doit avoir un **destructeur virtuel**.

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
    virtual ~Animal() = default;
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
#include <memory>
#include <vector>

int main() {
    std::vector<std::unique_ptr<Animal>> animaux;
    animaux.push_back(std::make_unique<Chien>());
    animaux.push_back(std::make_unique<Chat>());

    for (const auto& animal : animaux) {
        animal->parler();
    }
}
```

### 3.4.3. Polymorphisme et tables des virtualités (vtable)

Le polymorphisme en C++ est généralement implémenté à l'aide de tables de virtualités (vtable). Une vtable est une structure de données utilisée par le compilateur pour prendre en charge les fonctions virtuelles et le polymorphisme. Elle contient des pointeurs vers les fonctions virtuelles de la classe.

Lorsqu'un objet est créé, un pointeur vers la vtable de sa classe est placé dans l'objet. Lorsque la fonction virtuelle est appelée, la vtable est consultée pour déterminer quelle version de la fonction doit être exécutée.

## 3.5. Classes en C++ moderne

Les mécanismes présentés jusqu'ici existent depuis le C++ d'origine. C++11 et ses successeurs ont ajouté des outils qui rendent les classes plus sûres et plus concises ; ils sont attendus dans tout nouveau code.

### 3.5.1. Initialisation des membres

Un membre peut recevoir un initialiseur par défaut directement dans la classe ; les constructeurs n'ont alors à mentionner que ce qui diffère.

```cpp
#include <string>
#include <utility>
#include <vector>

class Compte {
public:
    Compte() = default;
    explicit Compte(std::string titulaire, double solde = 0.0)
        : titulaire_{std::move(titulaire)}, solde_{solde} {}

    const std::string& titulaire() const { return titulaire_; }
    double solde() const { return solde_; }

private:
    std::string titulaire_{"inconnu"};   // initialiseur de membre par défaut
    double solde_{};                     // 0.0
    std::vector<double> mouvements_;     // vide
};

int main() {
    Compte anonyme;
    Compte ada{"Ada", 100.0};
    return anonyme.titulaire() == "inconnu" && ada.solde() == 100.0 ? 0 : 1;
}
```

La **liste d'initialisation** (`: membre{valeur}`) initialise les membres directement, au lieu de les construire par défaut puis de les affecter dans le corps du constructeur. Les membres sont toujours initialisés dans l'ordre de leur **déclaration**, pas dans l'ordre de la liste (`-Wreorder` signale les incohérences).

### 3.5.2. `explicit`

Un constructeur appelable avec un seul argument et non marqué `explicit` définit une **conversion implicite** vers la classe :

```cpp
class Duree {
public:
    explicit Duree(int secondes) : secondes_{secondes} {}
    int secondes() const { return secondes_; }
private:
    int secondes_;
};

int attendre(const Duree& d) { return d.secondes(); }

int main() {
    // attendre(30);              // erreur grâce à explicit : que signifie 30 ?
    return attendre(Duree{30});   // intention claire
}
```

Par défaut, marquer `explicit` tout constructeur pouvant être appelé avec un seul argument ; ne conserver une conversion implicite que lorsqu'elle est réellement naturelle (`std::string` construit depuis `const char*`).

### 3.5.3. `override` et `final`

```cpp
#include <numbers>
#include <string>

class Forme {
public:
    virtual ~Forme() = default;
    virtual double aire() const = 0;
    virtual std::string nom() const { return "forme"; }
};

class Cercle final : public Forme {          // final : Cercle ne peut plus être dérivée
public:
    explicit Cercle(double rayon) : rayon_{rayon} {}
    double aire() const override { return std::numbers::pi * rayon_ * rayon_; }
    std::string nom() const override { return "cercle"; }
private:
    double rayon_;
};
```

`override` demande au compilateur de vérifier qu'une méthode redéfinit bien une méthode virtuelle de la classe de base : une faute de frappe ou une signature différente (oubli d'un `const`) devient une **erreur de compilation** au lieu de créer silencieusement une nouvelle méthode sans rapport. `final` interdit toute redéfinition ultérieure d'une méthode, ou toute dérivation lorsqu'il s'applique à la classe.

### 3.5.4. `= default` et `= delete`

```cpp
#include <string>

class Fichier {
public:
    explicit Fichier(const std::string& chemin);
    ~Fichier();

    Fichier(const Fichier&) = delete;              // non copiable
    Fichier& operator=(const Fichier&) = delete;

    Fichier(Fichier&&) noexcept = default;         // déplaçable
    Fichier& operator=(Fichier&&) noexcept = default;

private:
    int descripteur_{-1};
};
```

`= default` demande explicitement l'implémentation générée par le compilateur : cela documente l'intention et permet de rétablir une opération que le compilateur n'aurait plus générée implicitement. `= delete` interdit une opération : c'est ainsi que l'on rend une classe non copiable, ou que l'on refuse une surcharge indésirable (`void f(double) = delete;` empêche l'appel de `f(int)` avec un `double`).

Ces déclarations s'articulent avec les règles de zéro, de trois et de cinq (section 6.5).

### 3.5.5. Constance et méthodes `const`

Une méthode qui ne modifie pas l'état observable de l'objet doit être déclarée `const` : elle peut alors être appelée sur un objet ou une référence constante, et le compilateur vérifie qu'elle ne modifie aucun membre.

```cpp
#include <vector>

class Pile {
public:
    void empiler(int v) { donnees_.push_back(v); }
    [[nodiscard]] int sommet() const { return donnees_.back(); }
    [[nodiscard]] bool vide() const noexcept { return donnees_.empty(); }
private:
    std::vector<int> donnees_;
};

int lire(const Pile& p) {
    // p.empiler(1);                // erreur : p est const, empiler() ne l'est pas
    return p.vide() ? 0 : p.sommet();   // OK : vide() et sommet() sont const
}
```

La **const-correctness** se propage : une fonction qui reçoit un `const T&` ne peut appeler que les méthodes `const` de `T`. Le mot-clé `mutable` autorise exceptionnellement un membre (cache, mutex) à être modifié dans une méthode `const`.

### 3.5.6. `struct`, agrégats et initialisation désignée

`struct` et `class` sont identiques à une différence près : l'accès par défaut est `public` pour `struct` et `private` pour `class`. Par convention, `struct` sert aux **agrégats** — regroupements de données sans invariant à protéger — et `class` aux types qui encapsulent un invariant.

```cpp
struct Point {
    double x{};
    double y{};
};

int main() {
    Point p1{1.0, 2.0};
    Point p2{.x = 1.0, .y = 2.0};   // C++20 : initialisation désignée, plus lisible
    Point p3{.y = 5.0};             // x vaut 0.0
    return p1.x + p2.y + p3.x > 0 ? 0 : 1;
}
```

### 3.5.7. Comparaisons : `==` et `<=>` (C++20)

C++20 permet de faire générer les opérateurs de comparaison par le compilateur :

```cpp
#include <compare>

struct Version {
    int majeur{};
    int mineur{};
    int correctif{};

    auto operator<=>(const Version&) const = default;  // génère <, <=, >, >=, == et !=
};

static_assert(Version{1, 2, 0} < Version{1, 10, 0});
static_assert(Version{2, 0, 0} == Version{2, 0, 0});
```

L'opérateur `<=>` (*three-way comparison*, dit « vaisseau spatial ») compare les membres un à un dans l'ordre de déclaration. Lorsque seule l'égalité a un sens, déclarer uniquement `bool operator==(const Version&) const = default;`.

### 3.5.8. Composition, interfaces et conception

Toute relation entre types n'a pas vocation à devenir un héritage. Les *C++ Core Guidelines* recommandent :

- l'**héritage public** uniquement pour exprimer une relation d'interface (« est un », substituable) ;
- la **composition** (un membre) pour réutiliser une implémentation ;
- une **classe abstraite** avec destructeur virtuel pour définir une interface ;
- l'héritage multiple d'**interfaces** plutôt que de classes concrètes.

Ces principes rejoignent [[Principes SOLID en COO]] et les [[Design patterns]]. Le polymorphisme d'exécution par fonctions virtuelles n'est d'ailleurs pas le seul possible en C++ : les templates (section 6.1) et `std::variant` (section 6.9) offrent un polymorphisme **statique**, résolu à la compilation et souvent plus performant.

# 4. Gestion de la Mémoire

## 4.1. Allocation dynamique de la mémoire

L'allocation dynamique permet de créer des objets dont la durée de vie ou la taille ne sont pas connues à la compilation. Les opérateurs `new` et `delete` font partie du langage et doivent être compris, notamment pour lire du code ancien ou écrire certaines abstractions bas niveau.

En **C++ moderne**, un code applicatif ne devrait cependant presque jamais posséder directement une ressource avec un `new`/`delete` nu. On privilégie :

1. les objets à durée de vie automatique ;
2. `std::vector`, `std::string` et les autres conteneurs ;
3. RAII ;
4. `std::unique_ptr` pour un ownership unique ;
5. `std::shared_ptr` uniquement lorsque l'ownership est réellement partagé.

Autrement dit : apprendre `new` et `delete` est nécessaire, mais leur absence dans le code métier est souvent un bon signe.

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
#include <iostream>

void incrementer(int& ref) {
    ++ref;
}

int main() {
    int x = 10;
    incrementer(x);
    std::cout << x << '\n'; // Affiche 11
}
```

#### 4.2.2.4. Références constantes

Les références constantes permettent de référencer une variable sans pouvoir la modifier, ce qui est utile pour les paramètres de fonction qui ne doivent pas être modifiés.

**Exemple :**

```cpp
#include <iostream>

void afficher(const int& ref) {
    std::cout << ref << '\n';
}

int main() {
    int y = 30;
    afficher(y); // Affiche 30
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
#include <iostream>
#include <memory>

class Exemple {
public:
    Exemple() {
        std::cout << "Constructeur\n";
    }
    ~Exemple() {
        std::cout << "Destructeur\n";
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
#include <iostream>
#include <memory>

class Exemple {
public:
    Exemple() {
        std::cout << "Constructeur\n";
    }
    ~Exemple() {
        std::cout << "Destructeur\n";
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

Dans du code bas niveau ou ancien, chaque allocation effectuée par `new` doit avoir une désallocation cohérente par `delete` (`new[]` avec `delete[]`). Dans du nouveau code applicatif, il vaut mieux éviter d'avoir à assurer manuellement cette correspondance en utilisant RAII, les conteneurs et les pointeurs intelligents.

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
  m = "one";
  m = "two";
  ```

- **unordered_set** et **unordered_map** : Versions non ordonnées des conteneurs `set` et `map`, offrant des performances de recherche plus rapides.

  **Exemple :**
  ```cpp
  #include <unordered_set>
  #include <unordered_map>
  
  std::unordered_set<int> us = {1, 2, 3, 4, 5};
  us.insert(6); // Ajoute un élément unique
  
  std::unordered_map<int, std::string> um;
  um = "one";
  um = "two";
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

## 5.5. Choisir un conteneur et gérer les itérateurs

### 5.5.1. Quel conteneur ?

| Besoin | Conteneur | Remarques |
|---|---|---|
| Séquence, cas général | `std::vector` | Le choix par défaut : stockage contigu, parcours très rapide, `push_back` en O(1) amorti |
| Taille fixe connue à la compilation | `std::array` | Aucune allocation dynamique |
| Insertions et suppressions fréquentes aux deux extrémités | `std::deque` | Accès indexé conservé |
| Dictionnaire, ordre sans importance | `std::unordered_map` | Table de hachage, recherche en O(1) en moyenne |
| Dictionnaire trié, parcours ordonné, recherche par bornes | `std::map` | Arbre équilibré, O(log n) |
| Ensemble de valeurs uniques | `std::unordered_set` / `std::set` | Mêmes critères que les dictionnaires |
| Petit dictionnaire lu bien plus souvent que modifié | `std::flat_map` (C++23) | Stockage contigu, très bonne localité |
| Pile, file | `std::stack`, `std::queue` | Adaptateurs au-dessus d'un autre conteneur |
| Liste chaînée | `std::list` | Rarement le meilleur choix : la localité mémoire de `std::vector` l'emporte presque toujours |

En cas de doute, commencer par `std::vector` et ne changer qu'après mesure (section 6.17).

### 5.5.2. Réserver et construire sur place

```cpp
#include <string>
#include <vector>

int main() {
    std::vector<std::string> noms;
    noms.reserve(1000);                   // évite les réallocations successives
    noms.emplace_back("Ada");             // construit l'élément directement dans le conteneur
    noms.push_back(std::string{"Linus"}); // construit puis déplace
    return static_cast<int>(noms.size());
}
```

### 5.5.3. Invalidation des itérateurs

Un itérateur, un pointeur ou une référence vers un élément peut devenir invalide après une modification du conteneur :

- `std::vector` : toute insertion susceptible de réallouer invalide tout ; `erase` invalide les éléments situés après ;
- `std::deque` : une insertion aux extrémités invalide les itérateurs mais pas les références ;
- `std::map`, `std::set`, `std::list` : seul l'élément supprimé est invalidé ;
- `std::unordered_map` : une réorganisation (*rehash*) invalide les itérateurs mais pas les références.

Code incorrect classique :

```cpp
for (auto it = v.begin(); it != v.end(); ++it) {
    if (*it % 2 == 0) {
        v.erase(it);   // it est invalidé : comportement indéfini à l'itération suivante
    }
}
```

Forme correcte :

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> v{1, 2, 3, 4, 5, 6};
    std::erase_if(v, [](int x) { return x % 2 == 0; });          // C++20
    // avant C++20, idiome erase-remove :
    // v.erase(std::remove_if(v.begin(), v.end(), pred), v.end());
    return static_cast<int>(v.size());
}
```

AddressSanitizer (section 1.2.4) détecte la plupart de ces erreurs à l'exécution ; les itérateurs de débogage de libstdc++ (`-D_GLIBCXX_DEBUG`) et le durcissement de la bibliothèque (`-D_GLIBCXX_ASSERTIONS`) en signalent d'autres dès l'accès fautif.

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
#include <iostream>

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

## 6.4. Concurrence et programmation multithread

La concurrence permet à plusieurs tâches de progresser pendant une même période. Elle est utile pour exploiter plusieurs cœurs, masquer des attentes d'E/S ou structurer certains systèmes réactifs. Elle introduit aussi une catégorie de bugs difficiles : **data races**, deadlocks, starvation et problèmes d'ordre mémoire.

### 6.4.1. `std::thread` et `std::jthread`

Depuis C++11, `std::thread` représente un thread natif. Un `std::thread` encore *joinable* lors de sa destruction provoque `std::terminate()`, ce qui impose de toujours appeler `join()` ou `detach()`.

C++20 apporte `std::jthread`, généralement plus sûr : il effectue automatiquement le `join` et intègre un mécanisme de demande d'arrêt.

```cpp
#include <chrono>
#include <iostream>
#include <thread>

using namespace std::chrono_literals;

int main() {
    std::jthread worker([] {
        std::this_thread::sleep_for(100ms);
        std::cout << "Travail terminé\n";
    });
} // join automatique
```

Pour du nouveau code, `std::jthread` est souvent préférable à `std::thread` lorsqu'un thread dédié est réellement nécessaire.

### 6.4.2. Arrêt coopératif avec `std::stop_token`

```cpp
#include <chrono>
#include <iostream>
#include <stop_token>
#include <thread>

using namespace std::chrono_literals;

int main() {
    std::jthread worker([](std::stop_token stop) {
        while (!stop.stop_requested()) {
            std::cout << "." << std::flush;
            std::this_thread::sleep_for(100ms);
        }
    });

    std::this_thread::sleep_for(350ms);
    worker.request_stop();
}
```

La demande d'arrêt est **coopérative** : le code du worker doit observer le token et sortir proprement.

### 6.4.3. Data race

Une data race se produit lorsque plusieurs threads accèdent concurremment à la même zone mémoire, qu'au moins un accès est une écriture et qu'aucune synchronisation adaptée n'ordonne ces accès. Une data race entraîne un **comportement indéfini**.

Code incorrect :

```cpp
int compteur = 0;

// Deux threads exécutent ceci en même temps : data race.
for (int i = 0; i < 1000; ++i) {
    ++compteur;
}
```

### 6.4.4. Mutex et verrouillage RAII

```cpp
#include <mutex>
#include <vector>

class Journal {
public:
    void ajouter(int valeur) {
        std::lock_guard verrou{mutex_};
        valeurs_.push_back(valeur);
    }

private:
    std::mutex mutex_;
    std::vector<int> valeurs_;
};
```

`std::lock_guard` libère le mutex automatiquement, y compris lorsqu'une exception traverse la portée.

Pour verrouiller plusieurs mutex sans créer un ordre incohérent :

```cpp
std::scoped_lock verrou{mutex_a, mutex_b};
```

### 6.4.5. `std::condition_variable`

Une variable de condition permet à un thread d'attendre qu'un état partagé devienne vrai sans effectuer une boucle active.

```cpp
#include <condition_variable>
#include <mutex>
#include <queue>

std::mutex mutex;
std::condition_variable cv;
std::queue<int> file;
bool termine = false;

void consommateur() {
    std::unique_lock verrou{mutex};
    cv.wait(verrou, [] { return !file.empty() || termine; });

    if (!file.empty()) {
        const int valeur = file.front();
        file.pop();
        // traiter valeur
    }
}
```

Toujours utiliser un **prédicat** avec `wait` afin de gérer correctement les réveils intempestifs.

### 6.4.6. Atomiques

Pour une donnée simple, un type atomique peut éviter un mutex :

```cpp
#include <atomic>

std::atomic<int> compteur{0};

void incrementer() {
    compteur.fetch_add(1, std::memory_order_relaxed);
}
```

`memory_order_relaxed` garantit l'atomicité mais n'ajoute pas de synchronisation avec d'autres données. Les ordres mémoire constituent un sujet avancé : utiliser l'ordre par défaut (`seq_cst`) ou une abstraction plus haut niveau tant que l'on n'a pas démontré la nécessité d'une optimisation plus faible.

### 6.4.7. Futures et tâches asynchrones

```cpp
#include <future>
#include <iostream>

int calcul_lourd() {
    return 42;
}

int main() {
    auto resultat = std::async(std::launch::async, calcul_lourd);
    std::cout << resultat.get() << '\n';
}
```

`std::future` transporte soit une valeur, soit l'exception produite par la tâche.

### 6.4.8. Bonnes pratiques de concurrence

- minimiser l'état mutable partagé ;
- préférer le passage de messages et les objets immuables lorsque c'est possible ;
- ne pas utiliser `detach()` par défaut ;
- documenter l'ownership et le protocole d'arrêt ;
- verrouiller avec RAII ;
- fixer un ordre de verrouillage ou utiliser `std::scoped_lock` ;
- tester avec ThreadSanitizer lorsque la plateforme le permet ;
- mesurer avant de paralléliser : davantage de threads ne signifie pas automatiquement davantage de performances.

## 6.5. Sémantique de déplacement et catégories de valeurs

La sémantique de déplacement, introduite en C++11, est fondamentale pour comprendre le C++ moderne. Elle permet de **transférer** des ressources au lieu de les copier.

### 6.5.1. Lvalue et rvalue

De manière simplifiée :

- une **lvalue** désigne généralement un objet identifiable et réutilisable ;
- une **rvalue** représente souvent une valeur temporaire ou une valeur dont les ressources peuvent être transférées.

```cpp
#include <string>
#include <utility>

std::string nom = "Ada";              // nom est une lvalue
std::string copie = nom;              // copie
std::string deplace = std::move(nom); // autorise le déplacement
```

`std::move` **ne déplace rien par lui-même** : il convertit son argument en une expression pouvant sélectionner une opération de déplacement.

Après un déplacement, l'objet source reste **valide mais dans un état non spécifié**, sauf garantie plus précise du type.

### 6.5.2. Règle de zéro

Lorsqu'une classe est composée de membres qui gèrent eux-mêmes correctement leurs ressources (`std::string`, `std::vector`, `std::unique_ptr`...), la meilleure stratégie consiste souvent à ne déclarer **aucun** destructeur, constructeur de copie ou de déplacement personnalisé.

```cpp
#include <string>
#include <vector>

class Utilisateur {
public:
    std::string nom;
    std::vector<int> scores;
};
```

C'est la **Rule of Zero**.

### 6.5.3. Règles de trois et de cinq

Une classe qui gère directement une ressource bas niveau peut devoir définir :

- destructeur ;
- constructeur de copie ;
- opérateur d'affectation par copie ;
- constructeur de déplacement ;
- opérateur d'affectation par déplacement.

Mais il faut d'abord se demander si cette gestion ne devrait pas être déléguée à un type RAII existant.

### 6.5.4. Perfect forwarding

Dans du code générique, `std::forward` permet de préserver la catégorie de valeur d'un argument :

```cpp
#include <utility>

template <class F, class T>
decltype(auto) appliquer(F&& f, T&& valeur) {
    return std::forward<F>(f)(std::forward<T>(valeur));
}
```

Ce mécanisme est puissant, mais inutile dans la majorité du code applicatif. Il apparaît surtout dans les bibliothèques génériques et les factories.

## 6.6. Lambdas et objets appelables

Les lambdas sont devenues un outil central du C++ moderne, notamment avec les algorithmes et les ranges.

```cpp
#include <vector>

int main() {
    std::vector<int> valeurs{1, 2, 3, 4, 5};

    std::erase_if(valeurs, [](int valeur) {   // supprime les valeurs paires
        return valeur % 2 == 0;
    });
}
```

### 6.6.1. Captures

```cpp
int seuil = 10;

auto superieur = [seuil](int valeur) {
    return valeur > seuil;
};
```

- `[seuil]` : capture par valeur ;
- `[&seuil]` : capture par référence ;
- `[=]` : capture implicite par valeur ;
- `[&]` : capture implicite par référence.

Préférer des captures explicites lorsque la durée de vie devient non triviale, notamment en asynchrone.

### 6.6.2. Lambda générique

```cpp
auto additionner = [](const auto& a, const auto& b) {
    return a + b;
};
```

### 6.6.3. `std::function` et alternatives

`std::function` réalise un effacement de type pratique pour stocker plusieurs types d'objets appelables sous une même interface :

```cpp
#include <functional>

std::function<int(int, int)> operation =
    [](int a, int b) { return a + b; };
```

Il peut entraîner une allocation ou un coût d'indirection. Pour du code générique interne, un template ou `auto` est souvent plus léger. C++23 ajoute aussi `std::move_only_function` pour les callables non copiables.

## 6.7. Concepts et contraintes — C++20

Les **concepts** permettent d'exprimer les contraintes d'un template dans son interface. Ils remplacent de nombreux usages difficiles à lire de SFINAE.

```cpp
#include <concepts>

template <std::integral T>
T doubler(T valeur) {
    return valeur * 2;
}
```

Forme avec `requires` :

```cpp
#include <concepts>

template <class T>
requires std::floating_point<T>
T moyenne(T a, T b) {
    return (a + b) / T{2};
}
```

### 6.7.1. Concept personnalisé

```cpp
#include <concepts>

template <class T>
concept Additionnable = requires(T a, T b) {
    { a + b } -> std::convertible_to<T>;
};

template <Additionnable T>
T somme(T a, T b) {
    return a + b;
}
```

Les concepts améliorent :

- la lisibilité des interfaces ;
- les diagnostics du compilateur ;
- la documentation implicite des attentes d'un template.

## 6.8. Ranges et vues — C++20/C++23

La bibliothèque ranges permet de composer des transformations sans écrire explicitement toute la mécanique des itérateurs.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> valeurs{1, 2, 3, 4, 5, 6};

    auto resultat = valeurs
        | std::views::filter([](int x) { return x % 2 == 0; })
        | std::views::transform([](int x) { return x * x; });

    for (int x : resultat) {
        std::cout << x << ' ';
    }
}
```

Une **view** est généralement paresseuse : elle décrit une transformation et calcule les éléments au moment où ils sont parcourus.

### 6.8.1. Algorithmes ranges

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> valeurs{4, 1, 3, 2};
    std::ranges::sort(valeurs);                       // pas de couple begin()/end()
    auto pos = std::ranges::find(valeurs, 3);
    return pos != valeurs.end() ? 0 : 1;
}
```

Le modèle ranges réduit certains couples `begin()` / `end()` et permet de mieux exprimer l'intention.

### 6.8.2. Durée de vie des vues

Une vue peut référencer un objet existant. Il faut donc s'assurer que cet objet vit assez longtemps. Éviter de conserver une vue vers un temporaire lorsque l'API ne garantit pas la sécurité de cette utilisation.

## 6.9. Types vocabulaire : `optional`, `variant` et `expected`

### 6.9.1. `std::optional`

Utiliser `std::optional<T>` lorsqu'une valeur peut légitimement être absente.

```cpp
#include <optional>
#include <string>

std::optional<int> trouver_age(const std::string& nom) {
    if (nom == "Ada") {
        return 36;
    }
    return std::nullopt;
}
```

### 6.9.2. `std::variant`

`std::variant` représente une valeur qui appartient à un ensemble fermé de types :

```cpp
#include <string>
#include <variant>

using Valeur = std::variant<int, double, std::string>;
```

Contrairement à une union C traditionnelle, `std::variant` connaît le type actif.

### 6.9.3. `std::expected` — C++23

`std::expected<T, E>` représente soit un résultat, soit une erreur attendue :

```cpp
#include <charconv>
#include <expected>
#include <string_view>

std::expected<int, std::string_view> convertir(std::string_view texte) {
    int valeur{};
    const auto [ptr, ec] = std::from_chars(
        texte.data(), texte.data() + texte.size(), valeur
    );

    if (ec != std::errc{} || ptr != texte.data() + texte.size()) {
        return std::unexpected("entier invalide");
    }
    return valeur;
}
```

Les exceptions et `std::expected` ne sont pas des solutions concurrentes absolues :

- une exception convient bien à une erreur qui rompt le flux normal et doit remonter plusieurs couches ;
- `expected` convient bien à une opération dont l'échec fait partie du contrat normal et doit être traité explicitement.

## 6.10. `std::span`, `std::string_view` et vues non propriétaires

Une vue non propriétaire permet de lire des données sans les copier, mais ne prolonge pas leur durée de vie.

### 6.10.1. `std::span`

```cpp
#include <span>

int somme(std::span<const int> valeurs) {
    int total = 0;
    for (int v : valeurs) {
        total += v;
    }
    return total;
}
```

Cette fonction accepte naturellement un `std::array`, un `std::vector` ou une zone contiguë compatible.

### 6.10.2. `std::string_view`

```cpp
#include <string_view>

bool commence_par_http(std::string_view texte) {
    return texte.starts_with("http://") || texte.starts_with("https://");
}
```

Ne jamais retourner une `string_view` qui référence une chaîne locale détruite à la sortie de la fonction.

## 6.11. Bibliothèque standard moderne : formatage, temps et système de fichiers

### 6.11.1. `std::format` et `std::print`

C++20 fournit `std::format` et C++23 ajoute `<print>` :

```cpp
#include <print>
#include <string>

int main() {
    std::string nom = "Ada";
    std::println("Bonjour {}, réponse = {}", nom, 42);
}
```

Le support exact de `<print>` dépend de la version de la bibliothèque standard, pas seulement du compilateur.

### 6.11.2. `std::chrono`

```cpp
#include <chrono>
#include <thread>

using namespace std::chrono_literals;

std::this_thread::sleep_for(250ms);
```

Les types de `std::chrono` rendent les unités explicites et évitent les interfaces basées sur des entiers ambigus.

### 6.11.3. `std::filesystem`

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    for (const auto& entree : fs::directory_iterator{"."}) {
        std::cout << entree.path() << '\n';
    }
}
```

## 6.12. Coroutines — C++20

Une coroutine est une fonction qui peut suspendre son exécution et la reprendre ultérieurement. Le langage fournit trois mots-clés :

- `co_await` ;
- `co_yield` ;
- `co_return`.

C++20 standardise le **mécanisme**, mais ne fournit pas à lui seul un runtime asynchrone universel. Une coroutine dépend d'un type de retour et d'une `promise_type` qui définissent son comportement.

C++23 introduit notamment `std::generator`, qui exploite les coroutines pour produire une séquence paresseuse, mais son support dans les bibliothèques standard peut rester incomplet selon la chaîne de compilation.

Utilisations typiques :

- générateurs ;
- E/S asynchrones ;
- pipelines ;
- moteurs de jeux ;
- systèmes événementiels.

Une coroutine n'est **pas nécessairement un thread** et n'implique pas automatiquement du parallélisme.

## 6.13. Modules — C++20

Les modules cherchent à réduire certains problèmes du modèle historique `#include` : temps de compilation, dépendances textuelles et macros traversant les frontières.

Interface simplifiée :

```cpp
export module calcul;

export int addition(int a, int b) {
    return a + b;
}
```

Utilisation :

```cpp
import calcul;

int main() {
    return addition(20, 22) == 42 ? 0 : 1;
}
```

> [!warning]
> En 2026, le support des modules dépend encore fortement du compilateur, de sa version, de la bibliothèque standard et du système de build. Les commandes exactes diffèrent entre GCC, Clang et MSVC. Il faut tester la chaîne de compilation ciblée avant d'imposer les modules à un projet multiplateforme.

Avec GCC 16, les modules restent activés par l'option `-fmodules` et le module standard `std` (C++23, `import std;`) se prépare avec `--compile-std-module`. CMake sait décrire les unités de module depuis la version 3.28 (`target_sources(cible PUBLIC FILE_SET CXX_MODULES FILES ...)`) avec les générateurs Ninja et Visual Studio.

Les headers classiques et les include guards restent donc indispensables à connaître.

## 6.14. Nouveautés importantes de C++23

C++23 est une évolution incrémentale importante de C++20. Parmi les ajouts utiles :

- `std::expected` ;
- `std::print` / `std::println` ;
- `std::mdspan` pour représenter des vues multidimensionnelles ;
- `std::flat_map` et `std::flat_set` ;
- `std::generator` ;
- `std::move_only_function` ;
- nouvelles opérations ranges et nouveaux adaptateurs de vues ;
- `if consteval` ;
- déduction de `this` (*explicit object parameter*) ;
- amélioration de `constexpr` ;
- nouveaux outils de la bibliothèque autour des tuples, chaînes et conteneurs.

### 6.14.1. Exemple de `std::mdspan`

`std::mdspan` est une **vue non propriétaire** multidimensionnelle sur une mémoire existante :

```cpp
#include <mdspan>
#include <vector>

int main() {
    std::vector<int> donnees(12);
    std::mdspan matrice(donnees.data(), 3, 4);   // vue 3 × 4 sur les 12 entiers
    matrice[1, 2] = 42;                          // opérateur [] multidimensionnel (C++23)
    return donnees[1 * 4 + 2] == 42 ? 0 : 1;
}
```

La disponibilité de chaque composant C++23 doit être vérifiée dans la **bibliothèque standard** utilisée (`libstdc++`, `libc++` ou MSVC STL).

## 6.15. Aperçu de C++26

Le travail technique sur C++26 s'est achevé le 28 mars 2026 (réunion WG21 de Londres) ; le document est soumis au vote d'approbation des organismes nationaux (*Draft International Standard*) et sa publication par l'ISO est attendue fin 2026. Le contenu du standard ne changera donc plus, mais il faut distinguer :

- ce que le texte de C++26 spécifie ;
- ce que le compilateur **et** sa bibliothèque standard implémentent réellement ;
- ce qui n'est activé que par une option explicite (`-freflection`, `-fcontracts`, `-fmodules`...).

Parmi les évolutions majeures de C++26 :

- **contrats** : préconditions `pre`, postconditions `post` et assertions `contract_assert` vérifiées à l'exécution ;
- **réflexion statique** (`^^T`, `<meta>`) : introspection et génération de code à la compilation ;
- **exécution asynchrone** `std::execution` (modèle *senders/receivers*) ;
- nouveaux conteneurs `std::inplace_vector` et `std::hive` ;
- `std::optional<T&>`, `std::function_ref`, `std::copyable_function` ;
- bibliothèque SIMD (`<simd>`) et algèbre linéaire (`<linalg>`) ;
- outils de concurrence tels que hazard pointers et RCU ;
- durcissement (*hardening*) de la bibliothèque standard et requalification de certains comportements indéfinis en « comportements erronés », par exemple la lecture d'une variable non initialisée ;
- nouvelles possibilités `constexpr`.

Exemple de contrats, compilable avec GCC 16.1 (`-std=c++26 -fcontracts`) :

```cpp
#include <cmath>

double racine(double x)
    pre (x >= 0.0)          // précondition : vérifiée avant l'appel
    post (r : r >= 0.0)     // postcondition : r désigne la valeur retournée
{
    return std::sqrt(x);
}
```

Un contrat exprime une **erreur de programmation** (appel incorrect) ; il ne remplace ni les exceptions ni `std::expected`, qui traitent des échecs attendus à l'exécution.

### 6.15.1. Ne pas écrire du code « C++26 » à l'aveugle

Avant d'utiliser une fonctionnalité récente :

1. vérifier le statut de la proposition ;
2. vérifier le support du compilateur ;
3. vérifier le support de sa bibliothèque standard ;
4. vérifier les plateformes de CI et de production ;
5. éventuellement tester une macro de fonctionnalité.

Exemple :

```cpp
#if defined(__cpp_lib_expected) && __cpp_lib_expected >= 202202L
// fonctionnalité disponible
#endif
```

Les **feature-test macros** sont préférables à un simple test de version du compilateur.

## 6.16. Comportement indéfini et sécurité mémoire

Le C++ autorise des opérations très proches du matériel. Certaines erreurs ne produisent pas une exception : elles provoquent un **comportement indéfini (UB)**, c'est-à-dire que le standard n'impose plus le comportement du programme.

Exemples classiques :

- accès hors limites ;
- dereferencement d'un pointeur invalide ;
- use-after-free ;
- signed integer overflow ;
- data race ;
- utilisation de certaines valeurs indéterminées ;
- violation des règles de durée de vie ou d'aliasing.

### 6.16.1. Prévention

- préférer les types qui expriment la durée de vie et l'ownership ;
- utiliser `std::span` plutôt qu'un couple pointeur + taille lorsque possible ;
- préférer `std::vector` et `std::array` aux tableaux manuels ;
- compiler avec des avertissements élevés ;
- tester avec ASan/UBSan/TSan ;
- utiliser `clang-tidy` et l'analyse statique ;
- effectuer du fuzzing sur les parseurs et entrées non fiables ;
- relire avec les C++ Core Guidelines.

## 6.17. Performance : mesurer avant d'optimiser

Le C++ permet des optimisations fines, mais l'optimisation prématurée rend souvent le code plus complexe sans bénéfice mesurable.

### 6.17.1. Build Debug et Release

Avec CMake :

```bash
cmake -S . -B build-debug -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build build-debug

cmake -S . -B build-release -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build-release
```

Pour les générateurs multi-configuration, la sélection de configuration s'effectue autrement ; ne pas supposer que `CMAKE_BUILD_TYPE` s'applique partout.

### 6.17.2. Benchmark

Un benchmark doit :

- isoler ce que l'on mesure ;
- empêcher le compilateur d'éliminer le travail ;
- être répété ;
- prendre en compte les warmups et la variance ;
- comparer des builds équivalents.

Google Benchmark est une bibliothèque courante pour automatiser cette démarche.

### 6.17.3. Profilage

Outils courants sous Linux :

```bash
perf record ./programme
perf report
```

Valgrind reste utile pour certains diagnostics, mais ASan est souvent plus rapide pendant le développement. Les outils ne se remplacent pas tous : chacun observe une classe de problèmes différente.

## 6.18. Qualité : formatage, analyse statique et tests

### 6.18.1. `clang-format`

```bash
clang-format -i src/*.cpp include/monprojet/*.hpp
```

Stocker un `.clang-format` à la racine garantit une mise en forme cohérente entre développeurs et CI.

### 6.18.2. `clang-tidy`

```bash
clang-tidy src/main.cpp -- -std=c++23
```

Sur un projet CMake, exporter la base de compilation améliore fortement la précision :

```bash
cmake -S . -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
clang-tidy -p build src/main.cpp
```

### 6.18.3. Tests

Un test doit vérifier le **contrat observable** plutôt que l'implémentation interne. Les frameworks courants comprennent GoogleTest, Catch2 et doctest.

Avec CMake/CTest :

```cmake
include(CTest)
enable_testing()

add_executable(test_calcul tests/test_calcul.cpp)
target_link_libraries(test_calcul PRIVATE monprojet)
add_test(NAME calcul COMMAND test_calcul)
```

Puis :

```bash
ctest --test-dir build --output-on-failure
```

# 7. Projets et Applications Pratiques

Les projets suivants doivent être réalisés avec une chaîne de compilation moderne, des avertissements activés et un historique Git propre. L'objectif est de pratiquer autant la **conception** que la syntaxe.

## 7.1. Projet 1 — Gestionnaire de tâches en console

### Objectifs

Mettre en pratique :

- classes et encapsulation ;
- `std::vector` ;
- `std::string` ;
- `std::chrono` ;
- algorithmes/ranges ;
- lecture/écriture de fichiers ;
- tests ;
- CMake.

### Fonctionnalités minimales

Le programme doit permettre de :

1. créer une tâche ;
2. lister les tâches ;
3. marquer une tâche comme terminée ;
4. supprimer une tâche ;
5. filtrer les tâches par état ;
6. sauvegarder les données ;
7. recharger les données au démarrage.

Modèle de départ :

```cpp
#pragma once

#include <cstdint>
#include <string>

struct Tache {
    std::uint64_t id{};
    std::string titre;
    bool terminee{false};
};
```

Le stockage peut d'abord utiliser un format texte simple. L'exercice consiste à séparer :

- domaine ;
- persistance ;
- interface utilisateur.

### Critères de qualité

- aucune possession via pointeur brut ;
- pas de variable globale mutable ;
- erreurs d'entrée utilisateur traitées ;
- tests sur les opérations du domaine ;
- build hors source ;
- compilation sans warning avec le profil choisi.

## 7.2. Projet 2 — Indexeur de fichiers

Créer un outil qui parcourt un répertoire et produit des statistiques :

- nombre de fichiers ;
- taille totale ;
- extensions les plus fréquentes ;
- plus gros fichiers ;
- recherche par extension ou motif.

Concepts à utiliser :

- `std::filesystem` ;
- `std::error_code` ou exceptions ;
- ranges/algorithmes ;
- `std::unordered_map` ;
- vues non propriétaires lorsque pertinentes ;
- tests sur un répertoire temporaire.

### Extension

Paralléliser **uniquement après mesure** certains traitements coûteux. Comparer la version séquentielle et la version concurrente.

## 7.3. Projet 3 — File de travaux concurrente

Construire une petite file de travaux avec :

- plusieurs workers ;
- arrêt coopératif ;
- attente sans boucle active ;
- remontée des erreurs ;
- tests de concurrence.

Une première version peut utiliser :

- `std::jthread` ;
- `std::mutex` ;
- `std::condition_variable` ;
- `std::queue` ;
- `std::stop_token`.

Le projet doit répondre explicitement aux questions :

- qui possède les workers ?
- comment l'arrêt est-il déclenché ?
- que se passe-t-il pour les tâches restantes ?
- comment les exceptions sont-elles observées ?
- quelles données sont protégées par quel mutex ?

## 7.4. Projet graphique optionnel

Pour une application graphique multiplateforme, **Qt 6** constitue une solution courante en C++. L'objectif n'est pas d'apprendre l'ensemble de Qt, mais de relier une interface à un domaine C++ déjà testé.

Architecture conseillée :

```text
UI Qt
  ↓
services applicatifs
  ↓
domaine C++ indépendant de Qt
  ↓
persistance
```

Éviter de placer toute la logique métier directement dans les callbacks de l'interface.

## 7.5. Projet final — Application C++ complète

Le projet final doit intégrer au minimum :

- C++20 ou C++23 ;
- CMake target-based ;
- tests automatiques ;
- RAII ;
- STL et algorithmes/ranges ;
- gestion d'erreur cohérente ;
- persistance ou interaction réseau ;
- documentation d'architecture ;
- CI compilant avec au moins un compilateur supplémentaire si possible.

### Exemple de sujet

Un gestionnaire de corpus local :

- indexation de fichiers ;
- métadonnées ;
- recherche ;
- cache ;
- tâches asynchrones ;
- export des résultats.

Le projet peut ensuite servir de support à des extensions : interface graphique, serveur HTTP, base de données ou calcul intensif.

## 7.6. Travaux pratiques progressifs

### TP 1 — Compilation et diagnostics

Compiler volontairement du code imparfait avec plusieurs niveaux d'avertissement et expliquer chaque diagnostic.

### TP 2 — Valeurs, références et durée de vie

Écrire plusieurs fonctions utilisant valeur, `const&`, `&` et déplacement ; justifier le choix de chaque signature.

### TP 3 — RAII

Écrire une classe qui gère une ressource simple, puis remplacer sa gestion manuelle par une abstraction standard et comparer.

### TP 4 — STL et algorithmes

Transformer un programme basé sur des boucles indexées en version utilisant algorithmes et ranges.

### TP 5 — Templates et concepts

Écrire une fonction générique non contrainte, observer ses diagnostics sur un type incompatible, puis introduire un concept.

### TP 6 — Gestion d'erreur

Implémenter la même opération avec exception puis avec `std::expected` et comparer les contrats.

### TP 7 — Concurrence

Créer un producteur/consommateur correctement synchronisé, puis vérifier le programme avec ThreadSanitizer.

### TP 8 — Tests et CMake

Créer une bibliothèque, un exécutable et une cible de test séparée avec CTest.

### TP 9 — Analyse dynamique

Introduire volontairement un accès hors limites et un use-after-free dans un exercice isolé, puis les retrouver avec ASan.

### TP 10 — Projet final

Assembler les notions précédentes et documenter les choix de design, d'ownership, d'erreur et de build.

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
#include <iostream>
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
#include <iostream>

int main() {
    if (true) {
        std::cout << "Bonjour, le monde!\n";
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

## 8.2. Chaîne d'outils moderne

Un projet C++ fiable ne dépend pas d'un IDE particulier. L'IDE doit s'appuyer sur une chaîne reproductible utilisable aussi en terminal et en CI.

### 8.2.1. GCC et Clang

Deux compilateurs permettent de détecter des hypothèses de portabilité :

```bash
g++ -std=c++23 -Wall -Wextra -Wpedantic src/main.cpp -o app
clang++ -std=c++23 -Wall -Wextra -Wpedantic src/main.cpp -o app
```

Le support d'une révision du standard comporte deux dimensions :

1. le **langage** implémenté par le compilateur ;
2. la **bibliothèque standard** (`libstdc++`, `libc++`, MSVC STL).

Une option `-std=c++23` acceptée ne garantit donc pas que tous les headers C++23 soient disponibles.

### 8.2.2. Profils de compilation

#### Développement

```bash
g++ -std=c++23 \
    -Wall -Wextra -Wpedantic -Wconversion -Wshadow \
    -g -O0 \
    src/main.cpp -o app
```

#### Diagnostic mémoire

```bash
g++ -std=c++23 \
    -Wall -Wextra -Wpedantic \
    -g -O1 \
    -fsanitize=address,undefined \
    -fno-omit-frame-pointer \
    src/main.cpp -o app
```

#### Release

```bash
g++ -std=c++23 -O2 -DNDEBUG src/main.cpp -o app
```

Ne pas copier aveuglément une collection de flags entre compilateurs : certaines options sont spécifiques à GCC ou Clang.

### 8.2.3. CMake target-based

Exemple de projet avec une bibliothèque et un exécutable :

```cmake
cmake_minimum_required(VERSION 3.25)
project(MonProjet VERSION 1.0 LANGUAGES CXX)

add_library(monprojet
    src/calcul.cpp
)

target_include_directories(monprojet
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
)

target_compile_features(monprojet PUBLIC cxx_std_23)

add_executable(monapp apps/main.cpp)
target_link_libraries(monapp PRIVATE monprojet)
```

Éviter autant que possible :

```cmake
set(CMAKE_CXX_FLAGS "...")
include_directories(...)
link_libraries(...)
```

Les propriétés attachées à des **targets** se propagent de façon plus précise et composable.

### 8.2.4. CMake Presets

`CMakePresets.json` permet de versionner des configurations communes :

```json
{
  "version": 6,
  "configurePresets": [
    {
      "name": "debug",
      "generator": "Ninja",
      "binaryDir": "${sourceDir}/build/debug",
      "cacheVariables": {
        "CMAKE_BUILD_TYPE": "Debug",
        "CMAKE_EXPORT_COMPILE_COMMANDS": "ON"
      }
    }
  ],
  "buildPresets": [
    {
      "name": "debug",
      "configurePreset": "debug"
    }
  ]
}
```

Utilisation :

```bash
cmake --preset debug
cmake --build --preset debug
```

### 8.2.5. Ninja

Ninja est un générateur de build rapide couramment utilisé avec CMake :

```bash
cmake -S . -B build -G Ninja
cmake --build build
```

### 8.2.6. Dépendances : package managers

Pour un projet non trivial, éviter de copier manuellement des bibliothèques dans le dépôt sans stratégie de mise à jour.

Deux solutions courantes sont :

- **Conan** ;
- **vcpkg**.

Le choix dépend de l'écosystème, des plateformes cibles, de la CI et de la politique de reproductibilité. Une dépendance doit être versionnée explicitement et faire l'objet d'une surveillance de sécurité.

### 8.2.7. VS Code

La note [[Visual studio code]] couvre l'éditeur en détail. Pour un projet C++ :

- utiliser **CMake Tools** lorsque CMake gère le build ;
- utiliser C/C++ ou clangd pour l'analyse ;
- laisser l'IDE lire `compile_commands.json` ou la configuration CMake ;
- ne pas dupliquer tous les paramètres de compilation dans `tasks.json`.

Un `launch.json` minimal avec GDB peut ressembler à :

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug app",
      "type": "cppdbg",
      "request": "launch",
      "program": "${workspaceFolder}/build/debug/monapp",
      "cwd": "${workspaceFolder}",
      "MIMode": "gdb",
      "miDebuggerPath": "/usr/bin/gdb"
    }
  ]
}
```

### 8.2.8. Débogueurs

#### GDB

```bash
gdb ./app
```

Commandes utiles :

```text
break main
run
next
step
print variable
bt
continue
```

#### LLDB

LLDB est l'alternative du projet LLVM/Clang et fournit les mêmes grandes fonctions : breakpoints, inspection de pile, variables et stepping.

### 8.2.9. Analyse statique

- diagnostics du compilateur ;
- `clang-tidy` ;
- analyseurs intégrés des IDE ;
- éventuellement des outils spécialisés comme cppcheck.

La compilation doit idéalement générer `compile_commands.json` afin que les outils disposent des vrais include paths et options :

```bash
cmake -S . -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
```

### 8.2.10. Tests et CI

Une CI C++ utile peut exécuter plusieurs jobs :

1. GCC Debug + tests ;
2. Clang Debug + tests ;
3. sanitizers ;
4. `clang-tidy` ;
5. build Release.

L'objectif est de détecter tôt les divergences entre compilateurs et les erreurs dynamiques.

## 8.3. Ressources pour approfondir ses connaissances en C++

Le C++ évolue tous les trois ans. Une ressource peut donc rester excellente sur un concept fondamental tout en étant dépassée sur les pratiques modernes. Toujours distinguer **référence normative**, **référence technique** et **tutoriel**.

### 8.3.1. Références prioritaires

- [WG21 — comité de standardisation C++](https://www.open-std.org/jtc1/sc22/wg21/) : working drafts et propositions ;
- [cppreference](https://en.cppreference.com/) : référence pratique du langage et de la bibliothèque standard ;
- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines) : recommandations de conception et de sécurité ;
- [GCC C++ status](https://gcc.gnu.org/projects/cxx-status.html) : niveau d'implémentation du langage ;
- [Clang C++ status](https://clang.llvm.org/cxx_status.html) : niveau d'implémentation du langage ;
- [Compiler Explorer](https://godbolt.org/) : comparaison des compilateurs, options et code machine ;
- [CMake Documentation](https://cmake.org/cmake/help/latest/) : référence du système de build.

> [!important]
> `cppreference` est une excellente référence technique, mais le texte normatif du standard et les documents WG21 restent la source ultime lorsqu'une question de conformité est litigieuse.


### 8.3.2. Livres

Les livres offrent une source d'information approfondie et structurée. Voici quelques ouvrages de référence pour le développement en C++.

#### 8.3.2.1. "The C++ Programming Language" par Bjarne Stroustrup

Ce livre est écrit par le créateur du langage C++, Bjarne Stroustrup. Il couvre en détail les concepts fondamentaux et avancés du C++.

**Points clés :**
- Historique et philosophie du langage
- Concepts de base et avancés
- Programmation générique
- Programmation orientée objet
- Bibliothèque standard C++

#### 8.3.2.2. "Effective Modern C++" par Scott Meyers

Ce livre reste très utile pour comprendre le virage du C++ moderne, mais il cible principalement **C++11 et C++14**. Il faut donc compléter ses recommandations par des ressources couvrant C++17, C++20 et C++23.

**Points clés :**
- Utilisation des nouvelles fonctionnalités de C++11 et C++14
- Meilleures pratiques et idiomes modernes
- Gestion des ressources et des exceptions
- Programmation concurrente

#### 8.3.2.3. "C++ Primer" par Stanley B. Lippman, Josée Lajoie, et Barbara E. Moo

Ce livre est parfait pour les débutants ainsi que pour les développeurs ayant une certaine expérience en C++.

**Points clés :**
- Introduction au langage C++
- Concepts de base et intermédiaires
- Programmation orientée objet
- Templates et STL
- Programmation avancée

### 8.3.3. Cours en ligne et tutoriels

Les cours en ligne et les tutoriels offrent une approche interactive pour l'apprentissage du C++. Voici quelques plateformes et cours recommandés.

#### 8.3.3.1. Coursera

**Cours recommandés :**
- **"C++ For C Programmers, Part A"** par l'Université de Californie, Santa Cruz
- **"C++ For C Programmers, Part B"** par l'Université de Californie, Santa Cruz

Ces cours couvrent les bases du C++ pour les développeurs ayant déjà une expérience en C.

#### 8.3.3.2. edX

**Cours recommandés :**
- **"Introduction to C++"** par Microsoft
- **"Advanced C++"** par Microsoft

Ces cours offrent une introduction complète au C++ ainsi que des sujets avancés pour les développeurs expérimentés.

#### 8.3.3.3. Udemy

**Cours recommandés :**
- **"Beginning C++ Programming - From Beginner to Beyond"** par Tim Buchalka
- **"Unreal Engine C++ Developer: Learn C++ and Make Video Games"** par Ben Tristem et GameDev.tv

Ces cours sont parfaits pour les débutants ainsi que pour ceux intéressés par le développement de jeux vidéo avec C++.

#### 8.3.3.4. FUN MOOC

**Cours recommandés :**

- **"Programmation en C++"** par l'Institut Mines-Télécom
    - Ce cours couvre les concepts fondamentaux de la programmation en C++ et est idéal pour les débutants et les intermédiaires.

#### 8.3.3.5. Developpez.com

**Ressources recommandées :**

- **Tutoriels et articles** : [Developpez.com](https://cpp.developpez.com) propose une vaste collection de tutoriels, d'articles et de forums dédiés à C++.
    - Tutoriels C++
    - Articles C++

#### 8.3.3.6. OpenClassrooms

**Cours recommandés :**

- **["Apprenez à programmer en C++"](https://openclassrooms.com/fr/courses/19980-apprenez-a-programmer-en-c)** : Ce cours gratuit d'OpenClassrooms couvre les bases du langage C++ et permet de se familiariser avec les concepts fondamentaux.
### 8.3.4. Documentation et sites web

La documentation officielle et les sites web dédiés sont des ressources essentielles pour rester à jour avec les dernières évolutions du langage C++.

#### 8.3.4.1. Documentation officielle

**[cppreference](https://en.cppreference.com/)**
- Une référence complète pour les bibliothèques C++ standard et les fonctionnalités du langage.
- Inclut des exemples de code et des explications détaillées.

**cplusplus.com :**
- Un site web dédié aux tutoriels et à la documentation C++.
- Inclut des guides de référence pour la bibliothèque standard C++.

#### 8.3.4.2. Stack Overflow

Stack Overflow est une communauté en ligne où les développeurs peuvent poser des questions et obtenir des réponses sur divers sujets de programmation, y compris C++.

**Utilisation :**
- Rechercher des solutions aux problèmes courants.
- Poser des questions spécifiques et obtenir des réponses de la communauté.
- Participer aux discussions et aider d'autres développeurs.

#### 8.3.4.3. GitHub

GitHub est une plateforme de développement collaboratif où les développeurs peuvent héberger et partager leur code.

**Utilisation :**
- Explorer les projets open source en C++.
- Contribuer à des projets existants.
- Apprendre des meilleures pratiques en étudiant le code des autres développeurs.

### 8.3.5. Forums et communautés

Les forums et les communautés en ligne offrent des opportunités de réseautage et d'apprentissage continu.

#### 8.3.5.1. Reddit

**Subreddits recommandés :**
- **r/cpp** : Un subreddit dédié aux discussions sur C++.
- **r/learnprogramming** : Un subreddit pour les débutants en programmation.

#### 8.3.5.2. C++ Forum

**[cplusplus.com ](https://cplusplus.com):**
- Un forum dédié aux discussions sur C++.
- Inclut des sections pour les débutants, les questions avancées, et les discussions générales.

## 8.4. Checklist d'un projet C++ moderne

Avant de considérer un projet prêt à être livré :

- [ ] le standard C++ minimal est explicitement documenté ;
- [ ] le projet compile avec des warnings élevés ;
- [ ] les warnings importants sont corrigés plutôt que masqués ;
- [ ] l'ownership des ressources est explicite ;
- [ ] `new`/`delete` nus sont absents du code applicatif sauf justification ;
- [ ] RAII protège fichiers, verrous, sockets et autres ressources ;
- [ ] les interfaces évitent les copies inutiles sans introduire de dangling references ;
- [ ] les tests sont lancés par CTest ou un mécanisme automatisé équivalent ;
- [ ] ASan/UBSan sont exécutés régulièrement ;
- [ ] TSan est utilisé sur les parties concurrentes lorsque possible ;
- [ ] la CI construit au moins les plateformes et compilateurs réellement supportés ;
- [ ] les dépendances sont versionnées et mises à jour ;
- [ ] les performances critiques ont été mesurées ;
- [ ] le comportement sur erreur est documenté ;
- [ ] le build peut être reproduit sans l'IDE du développeur.

## 8.5. À retenir

Le C++ moderne privilégie l'expression de l'intention et de la durée de vie dans les **types** :

- un objet local plutôt qu'une allocation dynamique inutile ;
- `std::vector` plutôt qu'un tableau alloué manuellement ;
- `std::unique_ptr` plutôt qu'un pointeur propriétaire ambigu ;
- `std::span` ou `std::string_view` pour une vue non propriétaire ;
- concepts pour documenter les templates ;
- ranges pour composer des traitements ;
- `std::expected` lorsque l'erreur fait naturellement partie du résultat ;
- `std::jthread` et RAII pour les threads ;
- CMake target-based, tests et analyse statique pour l'ingénierie du projet.

La règle la plus importante reste : **ne pas confondre contrôle bas niveau et obligation d'écrire du code bas niveau**. La force du C++ moderne vient précisément de sa capacité à encapsuler les détails dangereux dans des abstractions à coût maîtrisé.
