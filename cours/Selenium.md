Selenium est à la fois un outil de tests des interfaces web et un outil de scraping permettant facilement l'exécution du code javascript dans l'environnement du navigateur web choisi.
Plan du cours :

1. **Introduction à Selenium et à l'automatisation du Web**
   - Importance de l'automatisation du Web
   - Présentation de Selenium : définition, avantages et cas d'utilisation

2. **Installation et Configuration**
   - Installation de Python et Selenium
   - Configuration des WebDriver : Chrome, Firefox, Safari
   - Importance du PATH system et comment le configurer

3. **Principes de base de Selenium**
   - Les quatre éléments de base de Selenium: WebDriver, WebElement, DesiredCapabilities, Exceptions
   - Introduction aux méthodes et aux attributs de WebDriver et WebElement

4. **Interagir avec le navigateur**
   - Manipulation des fenêtres et des onglets du navigateur
   - Navigation et actualisation de la page
   - Gestion des cookies

5. **Localiser les éléments**
   - Différentes méthodes pour localiser les éléments : ID, Name, Class Name, Tag Name, Link Text, Partial Link Text, CSS Selector, XPath
   - Stratégies pour trouver les meilleurs sélecteurs

6. **Interagir avec les éléments Web**
   - Interactions avec les formulaires : entrée de texte, click, sélection d'options dans les menus déroulants
   - Gestion des alertes JavaScript

7. **Attentes implicites et explicites**
   - Différence entre attentes implicites et explicites
   - Utilisation des attentes pour résoudre les problèmes de synchronisation

8. **Travailler avec des cadres et des fenêtres multiples**
   - Manipulation de fenêtres multiples et de cadres
   - Stratégies pour gérer les fenêtres contextuelles et les annonces

9. **Actions avancées avec Selenium**
   - Glisser-déposer, clic droit, double clic et autres actions avancées avec ActionChains
   - Automatisation du scroll
   - Capture d'écran

10. **Introduction au Selenium Grid**
    - Utilisation de Selenium Grid pour exécuter des tests en parallèle
    - Configuration de Selenium Grid

11. **Bonnes pratiques et débogage**
    - Présentation des bonnes pratiques de codage et des techniques de débogage avec Selenium
    - Gestion des erreurs et exceptions 

12. **Projet final interactif**
    - Application des compétences acquises à un projet d'automatisation Web
    - Présentation des travaux, discussion et feedback


# 1. Introduction à Selenium et à l'Automatisation du Web

## 1.1 Importance de l'automatisation du Web

L'automatisation Web est utilisée dans une variété de contextes, notamment le test de logiciels, l'analyse de données et le web scraping. Il peut également être utilisé pour effectuer des tâches répétitives, comme remplir des formulaires Web ou cliquer sur des boutons.
Elle aide à identifier les régressions et les erreurs plus rapidement que les tests manuels. 

## 1.2 Présentation de Selenium

Selenium est un logiciel open source qui permet aux développeurs de contrôler un navigateur Web de manière programmatique, en utilisant une variété de langages de programmation, dont Python.

Selenium est particulièrement apprécié pour sa capacité à intégrer des tests dans le navigateur, ce qui permet de simuler le comportement de l'utilisateur final. Il est compatible avec de nombreux navigateurs, y compris Chrome, Firefox et Safari.

## 1.3 Avantages de Selenium

Voici les avantages spécifiques de l'utilisation de Selenium pour l'automatisation du Web :

- **Compatibilité cross-browser** : Selenium supporte une large gamme de navigateurs, ce qui permet aux développeurs de tester leurs applications sur différents navigateurs à partir d'une seule plateforme.
- **Support pour plusieurs langages de programmation** : Selenium peut être utilisé avec une variété de langages de programmation, dont Python, Java, C#, et Ruby.
- **Flexibilité** : Selenium permet d'automatiser de nombreuses actions, comme cliquer sur un bouton, remplir un formulaire, ou naviguer à travers une série de pages. Il permet également d'interagir avec des éléments complexes comme des menus déroulants, des sliders et des fenêtres contextuelles.

## 1.4 Cas d'utilisation de Selenium

Cas d'utilisation les plus courants de Selenium :

- **Tests d'interface utilisateur** : Selenium est couramment utilisé pour automatiser les tests d'interface utilisateur, en particulier pour les applications Web. Il permet de vérifier que les éléments de l'interface utilisateur fonctionnent comme prévu.
- **Web scraping** : Bien que Selenium ne soit pas un outil de web scraping en soi, il peut être utilisé pour automatiser la navigation et l'interaction avec les sites Web lors de l'exécution de tâches de web scraping.
- **Automatisation de tâches répétitives** : Selenium peut être utilisé pour automatiser les tâches qui nécessitent une interaction répétitive avec un navigateur Web, comme remplir et soumettre des formulaires ou cliquer sur des liens.

# 2. Installation et Configuration

## 2.1. Installation de Selenium

Avant de pouvoir commencer à utiliser Selenium, nous devons installer deux éléments essentiels: Python et Selenium.

Pour l'installation de Python voir le cours [[Python]]

Une fois Python installé, nous pouvons installer Selenium. Pour ce faire, nous utiliserons pip, le gestionnaire de paquets de Python.

   - Ouvrons une invite de commandes (ou un terminal pour les utilisateurs de Mac/Linux).
   - Tapons la commande suivante :
```bash
pip install selenium
```  
   - Appuyons sur Entrée. pip téléchargera et installera le package Selenium.

## 2.2. Configuration des WebDriver : Chrome, Firefox, Safari

Après avoir installé Python et Selenium, le prochain élément clé est le WebDriver. Chaque navigateur a son propre WebDriver. Pour automatiser un navigateur spécifique, nous devons télécharger et configurer le WebDriver correspondant.

### 2.2.1 WebDriver de Chrome (ChromeDriver)
   
   - Allons sur le site officiel de ChromeDriver à l'adresse suivante : https://sites.google.com/a/chromium.org/chromedriver/downloads.
   - Téléchargeons la version de ChromeDriver correspondant à votre version de Chrome. Décompressons le fichier téléchargé.

### 2.2.2 WebDriver de Firefox (GeckoDriver)

   - Rendons-nous sur le site officiel de GeckoDriver à l'adresse suivante : https://github.com/mozilla/geckodriver/releases.
   - Téléchargons la dernière version de GeckoDriver. Décompressons le fichier téléchargé.

### 2.2.3 WebDriver de Safari (SafariDriver)

   - Pour Safari, le WebDriver est déjà intégré dans le navigateur. Nous pouvons l'activer en allant dans les préférences de développement de Safari.

## 2.3. Importance du PATH system et comment le configurer

Le PATH système est une variable d'environnement qui indique au système d'exploitation où chercher les exécutables. Pour que votre système puisse trouver les WebDriver que nous venons d'installer, nous devons ajouter leurs emplacements à votre PATH système.

### 2.3.1 Ajout du WebDriver au PATH (Windows)

   - Localisons le fichier exécutable du WebDriver que nous avons téléchargé et décompressé (ChromeDriver ou GeckoDriver).
   - Copions l'emplacement du dossier contenant le WebDriver.
   - Allons dans les variables d'environnement système (recherchons "variables d'environnement" dans le menu Démarrer).
   - Trouvons la variable "Path" dans la liste des variables d'environnement système et cliquons sur "Modifier".
   - Cliquons sur "Nouveau" et collons l'emplacement du dossier contenant le WebDriver. Cliquons sur "OK".

### 2.3.2 Ajout du WebDriver au PATH (Mac/Linux)

   - Ouvrons un terminal.
   - Utilisons la commande `export PATH=$PATH:/path/to/directory` où "/path/to/directory" est l'emplacement du dossier contenant le WebDriver.
   - Pour rendre cette modification permanente, ajoutons la ligne de commande précédente à votre fichier .bashrc ou .bash_profile.

Après avoir suivi toutes ces étapes, nous devrions être prêt à commencer à utiliser Selenium pour automatiser les navigateurs Web avec Python. 

# 3. Principes de base de Selenium

## 3.1 Les quatre éléments de base de Selenium

Selenium est construit autour de quatre concepts clés qui forment la base de son fonctionnement :

### 3.1.1 WebDriver

   Le WebDriver est l'interface principale pour interagir avec le navigateur. Il contient des méthodes pour effectuer des tâches telles que la navigation vers une URL, l'obtention du titre de la page, la manipulation des cookies et bien d'autres. Chaque navigateur a son propre WebDriver distinct (ChromeDriver, GeckoDriver, etc.).

### 3.1.2 WebElement

   Un WebElement représente un élément HTML dans la page web. Les WebElements sont retournés par les méthodes de recherche du WebDriver. Ils contiennent des méthodes pour interagir avec l'élément, comme cliquer sur l'élément, obtenir ou définir la valeur d'un champ de saisie, obtenir le texte d'un élément, etc.

### 3.1.3 DesiredCapabilities

   Les DesiredCapabilities sont un ensemble de préférences utilisées pour configurer le WebDriver. Elles peuvent être utilisées pour définir des options comme le chemin d'exécution du navigateur, la taille de la fenêtre du navigateur, les paramètres de proxy, etc. 

### 3.1.4 Exceptions

   Selenium définit une série d'exceptions spécifiques qui sont levées lorsque des erreurs se produisent pendant l'exécution d'un script Selenium. Ces exceptions nous aident à comprendre ce qui ne va pas dans notre script et comment le corriger.

## 3.2 Introduction aux méthodes et aux attributs de WebDriver et WebElement

Explorons quelques-unes des méthodes et des attributs les plus couramment utilisés de WebDriver et WebElement.

### 3.2.1 Méthodes et attributs communs de WebDriver

   - `get(url)` : Cette méthode est utilisée pour naviguer vers une URL spécifique.
   - `title` : Cet attribut renvoie le titre de la page courante.
   - `current_url` : Cet attribut renvoie l'URL de la page courante.
   - `find_element_by_*` : Ces méthodes sont utilisées pour localiser un élément sur la page. Le * peut être remplacé par différentes stratégies de localisation comme id, name, class_name, tag_name, etc.

### 3.2.2 Méthodes et attributs communs de WebElement

   - `click()` : Cette méthode est utilisée pour cliquer sur un élément.
   - `send_keys(value)` : Cette méthode est utilisée pour entrer une valeur dans un champ de saisie.
   - `text` : Cet attribut renvoie le texte de l'élément.
   - `get_attribute(name)` : Cette méthode est utilisée pour obtenir la valeur d'un attribut spécifique de l'élément.

Ces concepts de base vont nous permettre d'écrire des scripts Selenium pour automatiser diverses tâches dans le navigateur.

# 4. Localiser les éléments

L'une des tâches les plus importantes lors de l'utilisation de Selenium est de localiser les éléments sur la page web avec lesquels nous souhaitons interagir. Selenium fournit plusieurs méthodes pour localiser les éléments.

## 4.1 Différentes méthodes pour localiser les éléments

### 4.1.1 ID

   L'ID est l'identifiant unique d'un élément dans une page HTML. Lorsqu'un élément a un attribut ID, nous pouvons utiliser la méthode `find_element_by_id` pour le localiser.

### 4.1.2 Name

   Le nom est un autre attribut souvent utilisé pour identifier les éléments dans une page HTML, surtout pour les champs de formulaire. Nous pouvons utiliser la méthode `find_element_by_name` pour localiser un élément par son nom.

### 4.1.3 Class Name

   Les classes sont utilisées en HTML pour appliquer un style CSS à un groupe d'éléments. Nous pouvons utiliser la méthode `find_element_by_class_name` pour localiser un élément par sa classe.

### 4.1.4 Tag Name

   Le nom de la balise est le nom de l'élément HTML, comme `a` pour les liens, `input` pour les champs de saisie, etc. Nous pouvons utiliser la méthode `find_element_by_tag_name` pour localiser un élément par son nom de balise.

### 4.1.5 Link Text

   Pour les liens, nous pouvons également les localiser par le texte du lien visible. Nous pouvons utiliser la méthode `find_element_by_link_text` pour cela.

### 4.1.6 Partial Link Text

   Si nous ne voulons pas ou ne pouvons pas correspondre au texte complet d'un lien, nous pouvons utiliser la méthode `find_element_by_partial_link_text` pour localiser un lien par une partie de son texte.

### 4.1.7 CSS Selector

   Les sélecteurs CSS sont une manière puissante de localiser les éléments en fonction de leur relation avec d'autres éléments, de leurs attributs, de leur classe, etc. Nous pouvons utiliser la méthode `find_element_by_css_selector` pour localiser un élément par son sélecteur CSS.

### 4.1.8 XPath

   XPath est une langue pour naviguer dans les documents XML, qui peut aussi être utilisée pour naviguer dans les documents HTML. Nous pouvons utiliser la méthode `find_element_by_xpath` pour localiser un élément par son XPath.

## 4.2 Stratégies pour trouver les meilleurs sélecteurs

Trouver les meilleurs sélecteurs peut être un défi. Voici quelques stratégies que nous pouvons utiliser :

### 4.2.1 Utiliser les ID et les noms lorsque c'est possible

   Les ID et les noms sont les moyens les plus simples et les plus directs de localiser les éléments. Ils sont également les plus rapides car ils sont indexés par le navigateur.

### 4.2.2 Préférer les sélecteurs CSS aux XPath

   Les sélecteurs CSS sont généralement plus lisibles et plus faciles à écrire que les XPath. Ils sont aussi plus rapides dans la plupart des navigateurs.

### 4.2.3 Éviter les sélecteurs trop spécifiques

   Si un sélecteur est trop spécifique, il peut facilement être brisé si la structure de la page change légèrement. Essayons de trouver un équilibre entre spécificité et robustesse.

### 4.2.4 Utiliser des outils de développement

   Les outils de développement de notre navigateur peuvent être très utiles pour trouver les meilleurs sélecteurs. Ils nous permettent d'inspecter les éléments et de tester nos sélecteurs en temps réel.

En maîtrisant ces techniques de localisation, nous serons en mesure d'interagir efficacement avec n'importe quel élément sur une page web.

# 5. Localiser les éléments

L'un des aspects fondamentaux de l'utilisation de Selenium est la capacité de localiser efficacement les éléments au sein d'une page Web. Pour interagir avec ces éléments, nous devons être capables de les localiser précisément. Dans ce chapitre, nous allons examiner les différentes méthodes pour localiser les éléments et les stratégies pour trouver les meilleurs sélecteurs.

## 5.1 Différentes méthodes pour localiser les éléments

Il existe plusieurs façons de localiser un élément dans une page Web. Certaines des méthodes les plus couramment utilisées sont :

### 5.1.1 ID

Chaque élément peut avoir un attribut id unique. Par exemple, pour localiser un élément avec l'id 'my-id', nous pouvons utiliser la méthode `find_element_by_id`:

   ```python
   element = driver.find_element_by_id('my-id')
   ```

### 5.1.2 Name

Nous pouvons aussi localiser un élément par son attribut name. Par exemple:

   ```python
   element = driver.find_element_by_name('my-name')
   ```

### 5.1.3 Class Name

Pour localiser un élément par son attribut class, nous utilisons la méthode `find_element_by_class_name`:

   ```python
   element = driver.find_element_by_class_name('my-class')
   ```

### 5.1.4 Tag Name

Nous pouvons également localiser un élément par son nom de balise:

   ```python
   element = driver.find_element_by_tag_name('div')
   ```

### 5.1.5 Link Text et Partial Link Text

Pour les liens, nous pouvons les localiser soit par leur texte complet (`find_element_by_link_text`) soit par une partie de leur texte (`find_element_by_partial_link_text`).

### 5.1.6 CSS Selector

Nous pouvons utiliser les sélecteurs CSS pour localiser un élément :

   ```python
   element = driver.find_element_by_css_selector('#my-id')
   ```

### 5.1.7 XPath

XPath est une autre façon puissante de localiser un élément :

   ```python
   element = driver.find_element_by_xpath('//div[@id="my-id"]')
   ```

## 5.2 Stratégies pour écrire de meilleurs sélecteurs

Pour avoir un code qui ne doit pas être adapté à chaque changement fait par l'éditeur du site, il faut:

1. **Préférer les sélecteurs uniques** : Lorsque c'est possible, nous devrions utiliser des id, des noms ou d'autres attributs qui sont uniques pour l'élément que nous voulons localiser.

2. **Utiliser des sélecteurs courts et robustes** : Les sélecteurs courts sont généralement plus efficaces et moins susceptibles d'être affectés par les changements dans la structure de la page.

3. **Éviter de compter sur la position** : La position d'un élément dans la page peut changer, donc nous devrions éviter de localiser les éléments en se basant uniquement sur leur position.

# 6. Interagir avec les éléments Web

L'une des principales utilisations de Selenium est d'automatiser les interactions avec les éléments sur une page web. Maintenant que nous savons comment localiser ces éléments, nous allons apprendre à interagir avec eux.

## 6.1 Interactions avec les formulaires

Les formulaires web sont une composante essentielle de nombreux sites web, que ce soit pour se connecter, pour s'inscrire, pour remplir un questionnaire, etc. Selenium nous permet d'automatiser toutes ces tâches.

## 6.1.1 Entrée de texte

   Pour entrer du texte dans un champ de saisie, nous utilisons la méthode `send_keys`. Par exemple, si nous avons localisé un champ de saisie dans la variable `input_field`, nous pouvons entrer le texte "Hello, world!" de cette façon : 

```python
input_field.send_keys("Hello, world!")
```

### 6.1.2 Click

   Pour cliquer sur un bouton ou un lien, nous utilisons la méthode `click`. Par exemple, si nous avons localisé un bouton dans la variable `button`, nous pouvons cliquer sur ce bouton de cette façon :
 
```python
button.click()
```

### 6.1.3 Sélection d'options dans les menus déroulants

   Pour sélectionner une option dans un menu déroulant, nous devons d'abord localiser l'élément `select`, puis utiliser la classe `Select` de Selenium. Par exemple, si nous avons localisé un menu déroulant dans la variable `dropdown`, nous pouvons sélectionner l'option avec la valeur "option1" de cette façon :

```python
from selenium.webdriver.support.ui import Select

select = Select(dropdown)
select.select_by_value("option1")
```

## 6.2 Gestion des alertes JavaScript

Les alertes JavaScript sont des fenêtres contextuelles qui apparaissent pour demander une confirmation ou fournir une information. Selenium peut automatiser la gestion de ces alertes.

Pour interagir avec une alerte JavaScript, nous utilisons l'objet `Alert` de Selenium. Par exemple, pour accepter une alerte, nous pouvons faire comme suit :

```python
from selenium.webdriver.common.alert import Alert

alert = Alert(driver)
alert.accept()
```

Et pour refuser une alerte, nous pouvons faire comme ceci :

```python
alert.dismiss()
```

Nous savons maintenant comment interagir avec les éléments web en utilisant Selenium.

# 7. Attentes implicites et explicites

En travaillant avec des pages web, nous devons souvent attendre qu'un certain état soit atteint avant de pouvoir interagir avec la page ou ses éléments. Par exemple, nous devons parfois attendre qu'une page soit complètement chargée, qu'un élément soit rendu, qu'un élément disparaisse, etc. Selenium nous offre deux mécanismes pour gérer ces situations : les attentes implicites et explicites.

## 7.1 Différence entre attentes implicites et explicites

### 7.1.1 Attentes implicites

Une attente implicite indique à WebDriver d'attendre un certain temps par défaut avant de lancer une exception si l'élément n'est pas trouvé. Il s'agit d'une attente globale qui s'applique à toutes les tentatives de localisation d'éléments. Voici comment nous définissons une attente implicite :

```python
driver.implicitly_wait(10)  # attend jusqu'à 10 secondes
```

Dans cet exemple, si WebDriver ne trouve pas immédiatement un élément, il attendra jusqu'à 10 secondes avant de lancer une `NoSuchElementException`.

### 7.1.2 Attentes explicites

Une attente explicite est une attente spécifique pour une certaine condition. Contrairement à l'attente implicite, elle ne s'applique qu'à une seule tentative de localisation d'élément. Les attentes explicites sont généralement préférées aux attentes implicites, car elles permettent une plus grande flexibilité et peuvent résoudre des problèmes de synchronisation plus complexes. Voici comment nous définissons une attente explicite :

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By

wait = WebDriverWait(driver, 10)
element = wait.until(EC.presence_of_element_located((By.ID, 'someid')))
```

Dans cet exemple, WebDriver attendra jusqu'à 10 secondes que l'élément avec l'ID 'someid' soit présent dans la page. Si l'élément est trouvé avant l'expiration du délai, il est immédiatement retourné. Si le délai expire avant que l'élément soit trouvé, une `TimeoutException` est lancée.

## 7.2 Utilisation des attentes pour résoudre les problèmes de synchronisation

Les problèmes de synchronisation se produisent lorsque WebDriver tente d'interagir avec un élément qui n'est pas encore prêt. Cela peut se produire pour diverses raisons, comme lorsque l'élément n'est pas encore rendu, lorsque l'élément est rendu mais pas encore visible, lorsque l'élément est en cours de chargement, etc.

Les attentes nous permettent de résoudre ces problèmes en retardant l'interaction avec l'élément jusqu'à ce qu'il soit prêt. Par exemple, si nous voulons cliquer sur un bouton qui est initialement désactivé et qui est activé après quelques secondes, nous pouvons utiliser une attente explicite pour attendre que le bouton soit activé :

```python
wait = WebDriverWait(driver, 10)
button = wait.until(EC.element_to_be_clickable((By.ID, 'someid')))
button.click()
```

Dans cet exemple, WebDriver attendra jusqu'à 10 secondes que le bouton soit cliquable avant de tenter de cliquer dessus.

En maîtrisant les attentes implicites et explicites, nous pouvons résoudre de nombreux problèmes de synchronisation et rendre nos scripts Selenium plus robustes et fiables.

# 8. Travailler avec des cadres et des fenêtres multiples

Lorsque nous travaillons avec des applications web modernes, il n'est pas rare d'avoir à interagir avec plusieurs fenêtres, onglets ou cadres (également connus sous le nom d'iframes) dans la même session. Selenium nous offre plusieurs outils pour gérer ces situations.

## 8.1 Manipulation de fenêtres multiples et de cadres

### 8.1.1 Fenêtres multiples

Chaque fenêtre ou onglet ouvert dans une session de navigateur a un identifiant unique appelé "handle". Nous pouvons utiliser ces handles pour passer d'une fenêtre ou d'un onglet à un autre. Voici comment nous pouvons faire cela :

```python
# Stocker le handle de la fenêtre principale
main_window = driver.current_window_handle

# Ouvrir une nouvelle fenêtre ou un nouvel onglet
... # Faire des actions qui déclenche l'ouverture
# Ou si nous souhaitons forcer l'ouverture
driver.execute_script("window.open();")

# Passer à la nouvelle fenêtre ou onglet
new_window = [window for window in driver.window_handles if window != main_window][0]
driver.switch_to.window(new_window)

# Interagir avec la nouvelle fenêtre ou onglet
...

# Retourner à la fenêtre principale
driver.switch_to.window(main_window)
```

Normalement l'ouverture d'une nouvelle fenêtre est la conséquence d'une action. Ici nous avons "forcer" l'ouverture avec `driver.execute_script("window.open();")` qui est une astuce. Lorsque nous travaillons avec des navigateurs Web dans Selenium, nous n'avons pas directement de méthode pour ouvrir une nouvelle fenêtre ou un nouvel onglet. Cependant, nous pouvons contourner cette limitation en utilisant la méthode `execute_script` pour exécuter un peu de JavaScript dans le contexte de la page Web.

L'instruction `window.open();` est une commande JavaScript qui ouvre une nouvelle fenêtre ou un nouvel onglet dans le navigateur. En l'exécutant avec la méthode `execute_script`, nous pouvons utiliser Selenium pour simuler le comportement d'un utilisateur qui ouvre une nouvelle fenêtre ou un nouvel onglet. Cela peut être utile dans divers scénarios de test, tels que tester comment notre application se comporte lorsque l'utilisateur travaille avec plusieurs fenêtres ou onglets.

### 8.1.2 Cadres

Les cadres sont des pages web à l'intérieur d'autres pages web. Pour interagir avec les éléments à l'intérieur d'un cadre, nous devons d'abord passer au cadre. Voici comment nous pouvons faire cela :

```python
# Passer au cadre
driver.switch_to.frame("frame_name_or_id")

# Interagir avec les éléments à l'intérieur du cadre
...

# Revenir au contenu principal
driver.switch_to.default_content()
```

## 8.2 Stratégies pour gérer les fenêtres contextuelles et les annonces

Les fenêtres contextuelles et les annonces peuvent souvent interférer avec nos scripts Selenium. Voici quelques stratégies que nous pouvons utiliser pour les gérer :

### 8.2.1 Attendre la fenêtre contextuelle ou l'annonce

Nous pouvons utiliser des attentes explicites pour attendre que la fenêtre contextuelle ou l'annonce apparaisse, puis interagir avec elle (par exemple, la fermer).

### 8.2.2 Désactiver les fenêtres contextuelles et les annonces

Dans certains cas, nous pouvons configurer le navigateur pour désactiver les fenêtres contextuelles et les annonces. Cela dépend du navigateur et de la nature de la fenêtre contextuelle ou de l'annonce.

### 8.2.3 Ignorer les fenêtres contextuelles et les annonces

Si la fenêtre contextuelle ou l'annonce n'affecte pas nos scripts, nous pouvons choisir de l'ignorer.

En maîtrisant la manipulation de fenêtres multiples et de cadres, et en élaborant des stratégies pour gérer les fenêtres contextuelles et les annonces, nous savons maintenant interagir efficacement avec des applications web complexes.

# 9 Actions avancées avec Selenium

Au-delà des interactions de base avec les éléments web, Selenium fournit également des outils pour réaliser des actions plus avancées, telles que le glisser-déposer, le clic droit, le double clic et plus encore. De plus, Selenium nous permet d'automatiser le défilement de la page et de prendre des captures d'écran.

## 9.1 Glisser-déposer, clic droit, double clic et autres actions avancées avec ActionChains

### 9.1.1 Glisser-déposer

   Pour réaliser une action de glisser-déposer, nous utilisons la classe `ActionChains` de Selenium. Voici comment nous pouvons faire cela :

   ```python
   from selenium.webdriver import ActionChains

   action_chains = ActionChains(driver)
   action_chains.drag_and_drop(element, target).perform()
   ```

   Dans cet exemple, `element` est l'élément que nous voulons glisser et `target` est l'endroit où nous voulons déposer l'élément.

### 9.1.2  Clic droit

   Pour réaliser un clic droit, nous utilisons également la classe `ActionChains`. Voici comment nous pouvons faire cela :

   ```python
   action_chains.context_click(element).perform()
   ```

   Dans cet exemple, `element` est l'élément sur lequel nous voulons faire un clic droit.

### 9.1.3 Double clic

   Pour réaliser un double clic, nous utilisons également la classe `ActionChains`. Voici comment nous pouvons faire cela :

   ```python
   action_chains.double_click(element).perform()
   ```

   Dans cet exemple, `element` est l'élément sur lequel nous voulons double-cliquer.

## 9.2 Automatisation du scroll

Pour automatiser le défilement de la page, nous pouvons utiliser la méthode `execute_script` de WebDriver pour exécuter un script JavaScript. Par exemple, pour défiler jusqu'au bas de la page, nous pouvons faire comme suit :

```python
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
```

Et pour défiler jusqu'à un certain élément, nous pouvons faire comme ceci :

```python
element = driver.find_element_by_id('someid')
driver.execute_script("arguments[0].scrollIntoView();", element)
```

## 9.3 Capture d'écran

Pour prendre une capture d'écran, nous utilisons la méthode `save_screenshot` de WebDriver. Par exemple, pour prendre une capture d'écran et la sauvegarder dans un fichier appelé "screenshot.png", nous pouvons faire comme suit :

```python
driver.save_screenshot("screenshot.png")
```

Nous savons maintenant interagir avec les applications web d'une manière plus sophistiquée et réaliser des tâches d'automatisation plus complexes.

# 10. Introduction au Selenium Grid

Alors que Selenium WebDriver nous permet d'exécuter des tests sur un seul navigateur à la fois, Selenium Grid nous permet d'exécuter des tests sur plusieurs navigateurs et systèmes d'exploitation en parallèle. Cela peut nous aider à augmenter la vitesse de nos tests et à assurer que nos applications fonctionnent correctement sur différentes configurations.

## 10.1 Utilisation de Selenium Grid pour exécuter des tests en parallèle

Pour utiliser Selenium Grid, nous devons d'abord configurer un "hub" et un ou plusieurs "nodes". Le hub est le serveur central qui gère la distribution des tests aux nodes. Chaque node est une machine (qui peut être le même ordinateur que le hub ou un ordinateur différent) qui exécute les tests.

Lorsque nous exécutons un test avec Selenium Grid, nous spécifions la configuration du navigateur et du système d'exploitation que nous souhaitons tester. Le hub distribue ensuite le test à un node qui correspond à cette configuration.

Voici un exemple de code pour exécuter un test avec Selenium Grid :

```python
from selenium import webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities

hub_url = "http://localhost:4444/wd/hub"  # Remplacer par l'URL de notre hub
desired_cap = DesiredCapabilities.CHROME  # Remplacer par le navigateur souhaité

driver = webdriver.Remote(command_executor=hub_url, desired_capabilities=desired_cap)

# Écrivons ici notre code de test

driver.quit()
```

## 10.2 Configuration de Selenium Grid

Pour configurer Selenium Grid, nous devons d'abord télécharger le fichier JAR de Selenium Server (qui contient Selenium Grid) depuis le site web de Selenium. Ensuite, nous devons démarrer le hub et les nodes avec les commandes appropriées.

Pour démarrer le hub, nous utilisons la commande suivante :

```bash
java -jar selenium-server-standalone-3.x.y.jar -role hub
```

Et pour démarrer un node, nous utilisons la commande suivante :

```bash
java -jar selenium-server-standalone-3.x.y.jar -role node -hub http://localhost:4444/grid/register
```

Notons que dans ces commandes, "3.x.y" doit être remplacé par le numéro de version de Selenium Server que nous avons téléchargé, et "localhost:4444" doit être remplacé par l'adresse et le port de votre hub.

En maîtrisant Selenium Grid, nous pouvons automatiser des fermes de navigateurs de tests et ainsi paramétrer des scénarios très complexes de validation.

# Documentation

Selenium a une documentation assez complète qui est répartie en plusieurs endroits, voici les principaux liens vers ces ressources :

1. **Documentation principale de Selenium** : Nous trouverons des guides, des tutoriels, des références d'API et d'autres ressources pour Selenium sur la page principale de la documentation de Selenium : https://www.selenium.dev/documentation/

2. **Référence de l'API Python pour Selenium** : La référence de l'API Python pour Selenium donne des informations détaillées sur toutes les classes, méthodes et fonctions disponibles dans le binding Python de Selenium. Vous pouvez la trouver ici : https://www.selenium.dev/selenium/docs/api/py/

3. **Documentation de Selenium Grid** : Si nous souhaitons en savoir plus sur Selenium Grid, nous pouvons consulter la documentation spécifique à Selenium Grid : https://www.selenium.dev/documentation/grid/

4. **GitHub de Selenium** : Pour des informations encore plus détaillées, y compris des notes de version et du code source, nous pouveons consulter le dépôt GitHub de Selenium : https://github.com/SeleniumHQ/selenium

# Liens

Une démo [Xavki](https://youtu.be/ecFpzceYmD4)