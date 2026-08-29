---
schema_version: 1
uid: "01M02EX5C8ABG29Z18H8HACTZR"
titre: "Selenium"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - tests-logiciels
  - automatisation
  - python
  - selenium
resume: "Cours complet sur Selenium 4 avec Python : WebDriver, Selenium Manager, sélecteurs, interactions, attentes, fenêtres et iframes, actions avancées, Page Object Model, pytest, Grid 4, WebDriver BiDi et bonnes pratiques."
niveau: intermediaire
prerequis:
  - "[[Python]]"
  - "[[HTML]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-07-24
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Cours — Automatiser un navigateur avec Selenium

> [!abstract] Objectif
> Piloter un vrai navigateur avec Selenium 4 et Python : installation sans gestion manuelle des drivers (Selenium Manager), localisation robuste des éléments, synchronisation par attentes explicites, tests maintenables avec pytest et le Page Object Model, Grid, BiDi et collecte de données respectueuse.

Voir aussi : [[Python]], [[HTML]], [[Javascript]], [[Sécurité avec Python]].

**Selenium** est un projet open source permettant de piloter de vrais navigateurs Web de manière automatisée. Son usage principal est le **test fonctionnel de bout en bout** (*end-to-end*, E2E), mais il peut également servir à automatiser des tâches répétitives ou, lorsque cela est pertinent, à collecter des données sur des pages nécessitant l'exécution de JavaScript.

Selenium ne remplace ni les tests unitaires, ni les tests d'intégration, ni un client HTTP spécialisé. Il intervient lorsque nous avons réellement besoin de reproduire le comportement d'un utilisateur dans un navigateur.

> [!important]
> Selenium pilote un **navigateur réel** à travers le standard W3C WebDriver. Un test Selenium est donc plus proche de l'expérience utilisateur qu'un simple appel HTTP, mais il est aussi plus lent, plus coûteux et plus sensible à l'environnement.

> [!note] État du cours
> Ce cours a été révisé le **29 août 2026** pour Selenium 4. Au moment de cette révision, le paquet Python stable est **Selenium 4.48.0** et nécessite Python 3.10 ou supérieur. Il faut toujours vérifier la version courante avant de figer une dépendance dans un projet.

# Sommaire

1. Introduction à Selenium et à l'automatisation Web
2. Installation et premier script
3. Architecture de Selenium et configuration du navigateur
4. Navigation et cycle de vie d'une session
5. Localiser les éléments
6. Interagir avec les éléments et le navigateur
7. Synchronisation et attentes
8. Fenêtres, onglets, frames et Shadow DOM
9. Actions avancées et JavaScript
10. Structurer des tests maintenables avec pytest
11. Selenium Grid et exécution distante
12. WebDriver BiDi, réseau et événements du navigateur
13. Débogage, erreurs fréquentes et stabilité
14. Selenium pour l'automatisation et la collecte de données
15. Projet final et synthèse

---

# 1. Introduction à Selenium et à l'automatisation Web

## 1.1. Pourquoi automatiser un navigateur ?

Une application Web moderne est souvent composée de plusieurs couches :

- HTML et CSS ;
- JavaScript exécuté dans le navigateur ;
- appels HTTP vers des API ;
- stockage local, cookies et sessions ;
- composants asynchrones ;
- authentification ;
- parfois plusieurs fenêtres, frames ou domaines.

Un simple test HTTP ne permet donc pas toujours de vérifier le comportement réellement observé par l'utilisateur.

L'automatisation du navigateur permet notamment de tester un scénario complet :

```text
ouvrir le site
   ↓
saisir un identifiant
   ↓
se connecter
   ↓
naviguer vers une page
   ↓
modifier une donnée
   ↓
vérifier le résultat affiché
```

## 1.2. Les composants du projet Selenium

Le projet Selenium comprend plusieurs briques.

### Selenium WebDriver

C'est l'API principale utilisée dans ce cours. Elle permet de contrôler un navigateur depuis Python, Java, JavaScript, C#, Ruby, etc.

### Selenium Grid

Grid permet d'exécuter des sessions WebDriver sur d'autres machines et de paralléliser les tests sur plusieurs navigateurs, versions ou systèmes d'exploitation.

### Selenium IDE

Selenium IDE est une extension de navigateur permettant d'enregistrer et rejouer des interactions. Elle peut être utile pour découvrir Selenium ou prototyper un scénario, mais les suites de tests maintenables sont généralement écrites dans un langage de programmation.

## 1.3. Selenium n'est pas seulement une bibliothèque Python

Avec Python, nous utilisons le **binding Python** de Selenium :

```text
notre programme Python
        ↓
API Selenium Python
        ↓
WebDriver / WebDriver BiDi
        ↓
navigateur
```

Le navigateur reste un programme distinct. Selenium ne « rend » pas lui-même les pages.

## 1.4. Principaux cas d'utilisation

### Tests E2E

Exemple : vérifier qu'un utilisateur peut créer un compte, recevoir l'état attendu dans l'interface puis se déconnecter.

### Tests multi-navigateurs

Un même scénario peut être exécuté avec Chrome/Chromium, Firefox, Edge ou Safari selon l'environnement.

### Automatisation de tâches répétitives

Par exemple :

- remplir un formulaire interne ;
- télécharger un rapport ;
- vérifier périodiquement un écran métier ;
- reproduire automatiquement une suite d'actions pour un diagnostic.

### Collecte de données sur des applications JavaScript

Selenium peut être utile si les données n'apparaissent qu'après exécution du JavaScript ou après une interaction. Pour une page statique ou une API, `requests`/`httpx` et un parseur HTML sont souvent plus simples et plus rapides.

## 1.5. Quand ne pas utiliser Selenium ?

Selenium est rarement le premier choix lorsque :

- une API publique ou interne fournit directement les données ;
- nous voulons tester une fonction Python isolée ;
- nous voulons seulement vérifier une réponse HTTP ;
- le besoin peut être couvert par un test unitaire ou d'intégration plus rapide ;
- nous voulons analyser des milliers de pages statiques sans interaction navigateur.

Une stratégie de tests saine contient généralement **peu de tests E2E**, mais ceux-ci couvrent les parcours les plus importants.

---

# 2. Installation et premier script

## 2.1. Créer un environnement Python isolé

Voir également [[Python]].

Sous Linux/macOS :

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install selenium
```

Sous Windows PowerShell :

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install selenium
```

Pour vérifier la version installée :

```bash
python -c "import selenium; print(selenium.__version__)"
```

Dans un projet réel, la version doit être suivie dans le gestionnaire de dépendances du projet.

## 2.2. Selenium Manager : ne plus installer systématiquement les drivers à la main

Historiquement, il fallait :

1. connaître la version de Chrome ou Firefox ;
2. télécharger ChromeDriver ou GeckoDriver ;
3. rendre l'exécutable accessible ;
4. l'ajouter au `PATH` ou renseigner son chemin dans le code ;
5. recommencer lors d'un changement de version du navigateur.

Cette procédure apparaît encore dans de nombreux anciens tutoriels.

Avec les versions modernes de Selenium 4, **Selenium Manager** gère normalement automatiquement la découverte et la mise à disposition du driver nécessaire lorsque nous créons le WebDriver.

Le cas simple devient donc :

```python
from selenium import webdriver

with webdriver.Chrome() as driver:
    driver.get("https://www.selenium.dev/")
    print(driver.title)
```

Pour Firefox :

```python
from selenium import webdriver

with webdriver.Firefox() as driver:
    driver.get("https://www.selenium.dev/")
    print(driver.title)
```

> [!important]
> Installer manuellement ChromeDriver/GeckoDriver et modifier le `PATH` reste possible et parfois nécessaire dans des environnements contrôlés, hors ligne ou particuliers. Ce n'est cependant plus la procédure normale à enseigner pour une installation locale récente.

## 2.3. Le navigateur reste nécessaire

Selenium Manager simplifie la gestion des drivers et peut également aider à gérer des navigateurs dans certains scénarios, mais il ne faut pas confondre :

- **le navigateur** : Chrome, Chromium, Firefox, Edge, Safari… ;
- **le driver WebDriver** : couche permettant à Selenium de contrôler ce navigateur ;
- **la bibliothèque Selenium** : API utilisée par notre code Python.

## 2.4. Premier test complet

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

with webdriver.Chrome() as driver:
    driver.get("https://www.selenium.dev/selenium/web/web-form.html")

    title = driver.title
    assert "Web form" in title

    text_box = driver.find_element(By.NAME, "my-text")
    submit_button = driver.find_element(By.CSS_SELECTOR, "button")

    text_box.send_keys("Selenium")
    submit_button.click()

    message = driver.find_element(By.ID, "message")
    assert message.text == "Received!"
```

Nous retrouvons déjà les quatre étapes fondamentales d'un test WebDriver :

1. créer une session ;
2. naviguer ;
3. localiser et manipuler des éléments ;
4. vérifier un résultat puis fermer la session.

## 2.5. Toujours fermer le navigateur

La méthode :

```python
driver.quit()
```

ferme toute la session WebDriver et les fenêtres associées.

Une construction `with` est particulièrement pratique :

```python
with webdriver.Chrome() as driver:
    driver.get("https://example.org")
```

La session est fermée à la sortie du bloc.

Dans pytest, nous préférerons généralement une **fixture** avec `yield`, vue plus loin.

---

# 3. Architecture de Selenium et configuration du navigateur

## 3.1. WebDriver

`WebDriver` représente une session contrôlant un navigateur.

Exemples de classes concrètes :

```python
webdriver.Chrome()
webdriver.Firefox()
webdriver.Edge()
webdriver.Safari()
```

Pour une session distante :

```python
webdriver.Remote(...)
```

## 3.2. WebElement

Un `WebElement` représente un élément du DOM retourné par Selenium.

```python
from selenium.webdriver.common.by import By

element = driver.find_element(By.ID, "login")
```

Nous pouvons ensuite :

```python
element.click()
element.send_keys("texte")
print(element.text)
print(element.get_attribute("href"))
```

Un `WebElement` n'est pas une copie permanente de l'élément HTML. Il s'agit d'une **référence distante** vers un élément du DOM de la page courante. Si le DOM est reconstruit, cette référence peut devenir périmée et provoquer `StaleElementReferenceException`.

## 3.3. Les classes `Options` remplacent l'ancien usage de `DesiredCapabilities`

Avec Selenium 3, les tutoriels utilisaient souvent `DesiredCapabilities`.

Avec Selenium 4, il faut préférer les classes d'options propres aux navigateurs :

```python
from selenium import webdriver

options = webdriver.ChromeOptions()
options.add_argument("--headless")
options.add_argument("--window-size=1920,1080")

driver = webdriver.Chrome(options=options)
```

Pour Firefox :

```python
options = webdriver.FirefoxOptions()
options.add_argument("-headless")

driver = webdriver.Firefox(options=options)
```

## 3.4. Quelques options communes

### Stratégie de chargement de page

```python
options = webdriver.ChromeOptions()
options.page_load_strategy = "eager"
```

Valeurs principales :

- `normal` : attend l'événement `load` ;
- `eager` : poursuit après `DOMContentLoaded` ;
- `none` : attend le moins possible et laisse le code gérer la synchronisation.

Le choix `eager` ou `none` peut accélérer certains scénarios, mais il exige une bonne stratégie d'attente.

### Certificats non fiables

Pour un environnement de test uniquement :

```python
options.accept_insecure_certs = True
```

Il ne faut pas masquer un problème de certificat en production simplement pour faire passer un test.

### Mode headless

Le mode **headless** exécute le navigateur sans fenêtre graphique visible :

```python
options.add_argument("--headless")
```

Il est utile en CI. Il reste néanmoins prudent d'exécuter aussi les scénarios critiques dans un navigateur visible, car un environnement graphique et un environnement headless peuvent révéler des comportements différents.

## 3.5. Capabilities

Une session WebDriver est négociée à l'aide de **capabilities** conformes au protocole WebDriver.

Nous utilisons généralement les classes `Options` au lieu de construire manuellement un dictionnaire de capabilities :

```python
options = webdriver.ChromeOptions()
options.browser_version = "stable"
options.platform_name = "any"
```

Pour une infrastructure distante, ces informations servent au Grid ou au fournisseur distant à choisir un navigateur compatible.

## 3.6. Service et chemin de driver manuel

Si Selenium Manager ne doit pas être utilisé, nous pouvons spécifier explicitement le driver :

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service

service = Service("/opt/webdrivers/chromedriver")
driver = webdriver.Chrome(service=service)
```

Cela doit être réservé aux environnements où le cycle de vie du driver est volontairement administré par l'équipe.

---

# 4. Navigation et cycle de vie d'une session

## 4.1. Naviguer vers une URL

```python
driver.get("https://example.org")
```

Informations utiles :

```python
print(driver.title)
print(driver.current_url)
print(driver.page_source)
```

`page_source` fournit la source telle que WebDriver l'expose à ce moment. Pour analyser précisément un DOM dynamique complexe, il est souvent préférable de travailler directement avec les éléments Selenium.

## 4.2. Historique du navigateur

```python
driver.back()
driver.forward()
driver.refresh()
```

## 4.3. Taille et position de la fenêtre

```python
driver.set_window_size(1280, 900)
driver.maximize_window()
```

Fixer une taille déterministe est utile pour les tests sensibles au responsive design.

## 4.4. Cookies

Créer un cookie :

```python
driver.add_cookie({"name": "theme", "value": "dark"})
```

Lire les cookies :

```python
print(driver.get_cookies())
print(driver.get_cookie("theme"))
```

Supprimer :

```python
driver.delete_cookie("theme")
driver.delete_all_cookies()
```

Le domaine du cookie doit être compatible avec la page actuellement ouverte.

## 4.5. Timeouts de session

Exemples :

```python
driver.set_page_load_timeout(30)
driver.set_script_timeout(10)
```

L'attente de localisation d'élément est abordée au chapitre 7.

## 4.6. `close()` et `quit()`

```python
driver.close()
```

ferme la fenêtre ou l'onglet actuellement contrôlé.

```python
driver.quit()
```

termine la session entière.

Dans un teardown de test, c'est presque toujours `quit()` qui est attendu.

---

# 5. Localiser les éléments

Localiser correctement les éléments est l'un des points les plus importants pour obtenir des tests stables.

## 5.1. API moderne : `By`

Les anciennes méthodes Python telles que :

```python
# Ancien code — à ne plus utiliser
# driver.find_element_by_id("login")
# driver.find_element_by_xpath("//button")
```

ont disparu de l'API moderne.

Nous utilisons :

```python
from selenium.webdriver.common.by import By

driver.find_element(By.ID, "login")
driver.find_element(By.XPATH, "//button")
```

## 5.2. Les huit stratégies classiques

### ID

```python
driver.find_element(By.ID, "username")
```

### Name

```python
driver.find_element(By.NAME, "email")
```

### Class name

```python
driver.find_element(By.CLASS_NAME, "notification")
```

`CLASS_NAME` attend **un seul nom de classe**, pas une expression comme `"btn primary"`.

Pour plusieurs classes, utiliser un sélecteur CSS :

```python
driver.find_element(By.CSS_SELECTOR, ".btn.primary")
```

### Tag name

```python
driver.find_element(By.TAG_NAME, "h1")
```

### Link text

```python
driver.find_element(By.LINK_TEXT, "Documentation")
```

### Partial link text

```python
driver.find_element(By.PARTIAL_LINK_TEXT, "Doc")
```

### CSS selector

```python
driver.find_element(By.CSS_SELECTOR, "form#login button[type='submit']")
```

### XPath

```python
driver.find_element(By.XPATH, "//button[@type='submit']")
```

## 5.3. `find_element` et `find_elements`

```python
element = driver.find_element(By.CSS_SELECTOR, ".item")
```

renvoie le premier élément correspondant ou lève `NoSuchElementException`.

```python
elements = driver.find_elements(By.CSS_SELECTOR, ".item")
```

renvoie une liste, éventuellement vide.

Exemple :

```python
for item in driver.find_elements(By.CSS_SELECTOR, "ul.results > li"):
    print(item.text)
```

## 5.4. Rechercher à partir d'un élément

Nous pouvons limiter la recherche à une sous-partie du DOM :

```python
form = driver.find_element(By.ID, "login-form")
username = form.find_element(By.NAME, "username")
password = form.find_element(By.NAME, "password")
```

Cette approche rend souvent le code plus lisible.

## 5.5. Comment choisir un sélecteur robuste ?

Un bon sélecteur doit exprimer l'identité fonctionnelle de l'élément sans dépendre inutilement de la mise en page.

Ordre de préférence indicatif :

1. identifiant stable prévu pour le test ;
2. attribut métier stable (`data-testid`, `data-test`, etc.) ;
3. `name`, rôle ou attribut sémantique stable ;
4. CSS simple ;
5. XPath lisible lorsque CSS ne suffit pas.

Exemple d'attribut volontairement prévu pour les tests :

```html
<button data-testid="save-profile">Enregistrer</button>
```

```python
save_button = driver.find_element(
    By.CSS_SELECTOR,
    "[data-testid='save-profile']",
)
```

> [!tip]
> Dans une application que nous développons nous-mêmes, ajouter des identifiants de test stables est souvent bien plus fiable que d'essayer de deviner des sélecteurs à partir de classes CSS générées par le framework.

## 5.6. Éviter les sélecteurs fragiles

À éviter :

```python
driver.find_element(
    By.CSS_SELECTOR,
    "body > div:nth-child(3) > div > div:nth-child(2) > button",
)
```

ou :

```python
driver.find_element(By.XPATH, "/html/body/div[2]/div[1]/button[3]")
```

Ces sélecteurs décrivent la **position** actuelle de l'élément et cassent au moindre changement de structure.

## 5.7. CSS ou XPath ?

Il n'existe pas de règle absolue imposant CSS dans tous les cas.

CSS est excellent pour :

- les classes ;
- les attributs ;
- les relations parent/enfant simples ;
- les sélecteurs courts et lisibles.

XPath est particulièrement utile pour :

- exprimer des relations complexes ;
- rechercher à partir du texte ;
- remonter ou parcourir certains axes du DOM.

Exemple :

```python
driver.find_element(
    By.XPATH,
    "//label[normalize-space()='Adresse']/following::input[1]",
)
```

Le meilleur sélecteur est surtout celui qui reste **stable, explicite et maintenable**.

## 5.8. Centraliser les locators

Au lieu d'éparpiller :

```python
driver.find_element(By.ID, "username")
```

partout dans les tests, nous pouvons déclarer :

```python
USERNAME = (By.ID, "username")
LOGIN_BUTTON = (By.CSS_SELECTOR, "button[type='submit']")
```

puis :

```python
driver.find_element(*USERNAME)
```

Cette pratique est particulièrement utile avec le Page Object Model.

---

# 6. Interagir avec les éléments et le navigateur

## 6.1. Cliquer

```python
button.click()
```

Il est préférable de laisser Selenium effectuer un vrai clic WebDriver plutôt que de remplacer systématiquement l'interaction par JavaScript.

## 6.2. Saisir du texte

```python
input_element.send_keys("Bonjour")
```

Effacer puis saisir :

```python
input_element.clear()
input_element.send_keys("nouvelle valeur")
```

Touches spéciales :

```python
from selenium.webdriver.common.keys import Keys

input_element.send_keys("recherche", Keys.ENTER)
```

## 6.3. Lire l'état d'un élément

```python
print(element.text)
print(element.is_displayed())
print(element.is_enabled())
print(element.is_selected())
```

Attribut HTML :

```python
href = element.get_attribute("href")
```

Selenium expose également des API permettant de distinguer attributs DOM et propriétés lorsque cela est nécessaire.

## 6.4. Menus `<select>` natifs

```python
from selenium.webdriver.support.ui import Select

select_element = driver.find_element(By.ID, "country")
select = Select(select_element)

select.select_by_visible_text("France")
# ou
select.select_by_value("fr")
# ou
select.select_by_index(2)
```

`Select` ne fonctionne que sur une vraie balise HTML `<select>`. Les composants graphiques qui imitent une liste déroulante doivent être manipulés comme des éléments ordinaires.

## 6.5. Cases à cocher et boutons radio

```python
checkbox = driver.find_element(By.ID, "terms")
if not checkbox.is_selected():
    checkbox.click()
```

## 6.6. Alertes JavaScript

Une alerte JavaScript native est gérée via `switch_to.alert` :

```python
alert = driver.switch_to.alert
print(alert.text)
alert.accept()
```

Refuser :

```python
alert.dismiss()
```

Pour un `prompt()` :

```python
alert.send_keys("une valeur")
alert.accept()
```

Une modale HTML créée par Bootstrap, React, Vue, etc. **n'est pas une alerte JavaScript native** : elle doit être localisée dans le DOM comme les autres éléments.

## 6.7. Envoyer un fichier

Pour un champ :

```html
<input type="file">
```

nous pouvons fournir directement un chemin :

```python
file_input = driver.find_element(By.CSS_SELECTOR, "input[type='file']")
file_input.send_keys("/home/user/image.png")
```

Selenium ne doit pas tenter de piloter la boîte de dialogue native du système si nous pouvons envoyer directement le chemin au champ.

## 6.8. Captures d'écran

Page :

```python
driver.save_screenshot("page.png")
```

Élément :

```python
element.screenshot("element.png")
```

Les captures sont particulièrement utiles dans un rapport de test en cas d'échec.

---

# 7. Synchronisation et attentes

La synchronisation est probablement la principale source de tests Selenium instables.

Une page moderne peut afficher un élément avant qu'il soit cliquable, reconstruire un composant après un appel API, ou modifier le DOM plusieurs fois après `DOMContentLoaded`.

## 7.1. Ne pas utiliser `sleep()` comme stratégie principale

```python
import time

time.sleep(5)
```

est simple mais problématique :

- 5 secondes peuvent être insuffisantes sur une machine lente ;
- elles sont inutiles si l'élément est prêt en 100 ms ;
- les tests deviennent plus longs et toujours fragiles.

Un `sleep()` ponctuel peut servir au diagnostic, mais il ne doit pas remplacer une condition d'attente.

## 7.2. Attente implicite

```python
driver.implicitly_wait(5)
```

L'attente implicite s'applique globalement aux recherches d'éléments pendant la session.

La valeur par défaut est 0.

## 7.3. Attente explicite

Les attentes explicites sont généralement plus précises :

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, 10)

button = wait.until(
    EC.element_to_be_clickable((By.ID, "save"))
)
button.click()
```

## 7.4. Conditions courantes

Présence dans le DOM :

```python
EC.presence_of_element_located((By.ID, "result"))
```

Visible :

```python
EC.visibility_of_element_located((By.ID, "result"))
```

Cliquable :

```python
EC.element_to_be_clickable((By.ID, "save"))
```

Texte présent :

```python
EC.text_to_be_present_in_element((By.ID, "status"), "Terminé")
```

Disparition :

```python
EC.invisibility_of_element_located((By.CSS_SELECTOR, ".spinner"))
```

Nombre de fenêtres :

```python
EC.number_of_windows_to_be(2)
```

## 7.5. Conditions personnalisées

`WebDriverWait.until()` accepte une fonction :

```python
wait = WebDriverWait(driver, 10)

wait.until(
    lambda d: d.find_element(By.ID, "counter").text == "10"
)
```

Nous pouvons donc attendre un **état métier**, pas seulement la présence d'un élément.

## 7.6. Ne pas mélanger implicitement attente implicite et explicite

La documentation Selenium avertit que mélanger des délais implicites et explicites peut provoquer des temps d'attente imprévisibles.

Une convention robuste consiste à :

- laisser l'attente implicite à 0 ;
- utiliser des attentes explicites aux endroits où l'état asynchrone le nécessite.

## 7.7. Présent, visible et cliquable ne signifient pas la même chose

Un élément peut :

1. exister dans le DOM ;
2. être caché par CSS ;
3. être visible mais recouvert par une autre couche ;
4. être visible et interactif.

Choisir `presence_of_element_located` alors que nous voulons cliquer peut donc être insuffisant.

## 7.8. `StaleElementReferenceException`

Exemple classique :

```python
button = driver.find_element(By.ID, "save")
# le framework JavaScript reconstruit le DOM ici
button.click()  # peut lever StaleElementReferenceException
```

La référence `button` pointe vers un élément qui n'existe plus.

La bonne stratégie est souvent de **relocaliser l'élément après la modification** :

```python
button = wait.until(
    EC.element_to_be_clickable((By.ID, "save"))
)
button.click()
```

Éviter de conserver longtemps des références `WebElement` sur des pages très dynamiques.

---

# 8. Fenêtres, onglets, frames et Shadow DOM

## 8.1. Handles de fenêtres

Chaque contexte de fenêtre possède un identifiant :

```python
current = driver.current_window_handle
all_windows = driver.window_handles
```

## 8.2. Ouvrir un nouvel onglet ou une nouvelle fenêtre

Selenium 4 fournit directement :

```python
driver.switch_to.new_window("tab")
```

ou :

```python
driver.switch_to.new_window("window")
```

Il n'est donc plus nécessaire d'utiliser l'ancienne astuce :

```python
# À éviter lorsque new_window convient
# driver.execute_script("window.open();")
```

## 8.3. Gérer une fenêtre ouverte par l'application

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

original = driver.current_window_handle

driver.find_element(By.LINK_TEXT, "Ouvrir").click()

WebDriverWait(driver, 10).until(EC.number_of_windows_to_be(2))

new_window = (set(driver.window_handles) - {original}).pop()
driver.switch_to.window(new_window)

# actions dans la nouvelle fenêtre

driver.close()
driver.switch_to.window(original)
```

## 8.4. Iframes

Un `<iframe>` contient un contexte de document différent. Avant de chercher ses éléments, il faut y entrer.

Par identifiant ou nom :

```python
driver.switch_to.frame("payment-frame")
```

Par élément :

```python
frame = driver.find_element(By.CSS_SELECTOR, "iframe.payment")
driver.switch_to.frame(frame)
```

Retour au document principal :

```python
driver.switch_to.default_content()
```

Retour d'un niveau :

```python
driver.switch_to.parent_frame()
```

Attendre puis entrer dans une frame :

```python
wait.until(
    EC.frame_to_be_available_and_switch_to_it((By.ID, "payment-frame"))
)
```

## 8.5. Shadow DOM

Les Web Components peuvent encapsuler leur DOM dans une **shadow root**.

Exemple :

```python
host = driver.find_element(By.CSS_SELECTOR, "my-component")
shadow = host.shadow_root
button = shadow.find_element(By.CSS_SELECTOR, "button")
button.click()
```

Un sélecteur CSS ordinaire exécuté sur le document principal ne traverse pas automatiquement les limites d'un Shadow DOM.

---

# 9. Actions avancées et JavaScript

## 9.1. ActionChains

`ActionChains` permet de composer des interactions proches des périphériques utilisateur.

```python
from selenium.webdriver import ActionChains

actions = ActionChains(driver)
actions.move_to_element(element).click().perform()
```

## 9.2. Double clic

```python
ActionChains(driver).double_click(element).perform()
```

## 9.3. Clic contextuel

```python
ActionChains(driver).context_click(element).perform()
```

## 9.4. Glisser-déposer

```python
ActionChains(driver).drag_and_drop(source, target).perform()
```

Selon l'application, les bibliothèques JavaScript de glisser-déposer peuvent nécessiter une séquence plus fine de mouvements et d'appuis.

## 9.5. Survol

```python
ActionChains(driver).move_to_element(menu).perform()
```

Utile pour révéler un sous-menu au survol.

## 9.6. Clavier

```python
from selenium.webdriver.common.keys import Keys

ActionChains(driver) \
    .key_down(Keys.CONTROL) \
    .send_keys("a") \
    .key_up(Keys.CONTROL) \
    .perform()
```

Le raccourci dépend du système et du scénario ; les tests doivent éviter les hypothèses inutiles sur la plateforme.

## 9.7. Défilement

Selenium fournit des actions de scroll, mais nous pouvons aussi faire défiler jusqu'à un élément avec JavaScript lorsqu'il y a une raison précise :

```python
driver.execute_script(
    "arguments[0].scrollIntoView({block: 'center'});",
    element,
)
```

## 9.8. Exécuter du JavaScript

```python
result = driver.execute_script("return document.title")
```

ou :

```python
visible = driver.execute_script(
    "return arguments[0].getBoundingClientRect().height > 0;",
    element,
)
```

> [!warning]
> `execute_script()` est puissant, mais il ne faut pas l'utiliser pour contourner systématiquement WebDriver. Par exemple, un `arguments[0].click()` JavaScript peut réussir alors qu'un vrai utilisateur ne pourrait pas cliquer sur l'élément. Nous risquerions alors de masquer un défaut réel de l'interface.

## 9.9. Capture d'informations pour le diagnostic

Lors d'un échec, il est souvent utile de conserver :

- l'URL ;
- le titre ;
- une capture d'écran ;
- éventuellement le HTML ;
- les logs du navigateur ;
- le message et la pile d'exception ;
- les informations de version du navigateur et de Selenium.

---

# 10. Structurer des tests maintenables avec pytest

Selenium fournit l'automatisation du navigateur. Un framework comme **pytest** fournit l'organisation des tests, fixtures, assertions, rapports et paramétrage.

## 10.1. Installer pytest

```bash
python -m pip install pytest selenium
```

## 10.2. Une fixture WebDriver

```python
import pytest
from selenium import webdriver


@pytest.fixture
def driver():
    options = webdriver.ChromeOptions()
    options.add_argument("--window-size=1280,900")

    driver = webdriver.Chrome(options=options)
    yield driver
    driver.quit()
```

Test :

```python
def test_homepage_title(driver):
    driver.get("https://www.selenium.dev/")
    assert "Selenium" in driver.title
```

L'avantage de la fixture est que le navigateur sera fermé même si l'assertion échoue, grâce à la reprise après `yield`.

## 10.3. AAA : Arrange, Act, Assert

Un test gagne en lisibilité lorsqu'il distingue :

```python
# Arrange : préparer l'état
# Act     : effectuer l'action testée
# Assert  : vérifier le résultat
```

Exemple :

```python
def test_login(driver):
    # Arrange
    driver.get("https://example.test/login")

    # Act
    driver.find_element(By.NAME, "username").send_keys("alice")
    driver.find_element(By.NAME, "password").send_keys("secret")
    driver.find_element(By.CSS_SELECTOR, "button[type='submit']").click()

    # Assert
    WebDriverWait(driver, 10).until(
        EC.url_contains("/dashboard")
    )
    assert driver.find_element(By.TAG_NAME, "h1").text == "Tableau de bord"
```

## 10.4. Un test doit être indépendant

Éviter :

```text
test_1 crée le compte
      ↓
test_2 suppose que test_1 est passé
      ↓
test_3 suppose que test_2 est passé
```

Sinon :

- l'ordre des tests devient obligatoire ;
- un seul échec provoque une cascade ;
- la parallélisation devient difficile.

Un test doit préparer son propre état ou utiliser des fixtures contrôlées.

## 10.5. Ne pas utiliser l'interface pour préparer tout l'état

Si le but est de tester la modification d'un profil, nous n'avons pas forcément intérêt à créer l'utilisateur via 15 écrans Selenium avant chaque test.

Si l'application fournit une API ou un accès de test permettant de créer l'état initial de manière fiable, nous pouvons l'utiliser pour la phase **Arrange**, puis réserver Selenium à l'interaction réellement testée.

Cela rend les tests :

- plus rapides ;
- plus ciblés ;
- moins fragiles.

## 10.6. Page Object Model

Le **Page Object Model** (POM) consiste à représenter une page ou un composant par une classe.

Exemple :

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC


class LoginPage:
    USERNAME = (By.NAME, "username")
    PASSWORD = (By.NAME, "password")
    SUBMIT = (By.CSS_SELECTOR, "button[type='submit']")

    def __init__(self, driver):
        self.driver = driver
        self.wait = WebDriverWait(driver, 10)

    def open(self):
        self.driver.get("https://example.test/login")
        return self

    def login(self, username, password):
        self.driver.find_element(*self.USERNAME).send_keys(username)
        self.driver.find_element(*self.PASSWORD).send_keys(password)
        self.driver.find_element(*self.SUBMIT).click()
        self.wait.until(EC.url_contains("/dashboard"))
```

Test :

```python
def test_login(driver):
    page = LoginPage(driver).open()
    page.login("alice", "secret")

    assert "/dashboard" in driver.current_url
```

Les avantages sont :

- locators centralisés ;
- intention métier plus lisible ;
- modifications de l'interface moins dispersées dans les tests.

## 10.7. Ce qu'un Page Object ne doit pas devenir

Un mauvais Page Object peut devenir une gigantesque classe contenant chaque détail du DOM.

Nous pouvons également créer des **Component Objects** pour des composants réutilisables :

- barre de navigation ;
- tableau ;
- calendrier ;
- modale ;
- sélecteur personnalisé.

## 10.8. Assertions

Selenium effectue des actions et récupère des valeurs ; c'est le framework de test qui porte l'assertion :

```python
assert status.text == "Enregistré"
```

Il faut vérifier **le résultat métier important**, pas simplement que le clic n'a pas levé d'exception.

## 10.9. Paramétrer les navigateurs

Un projet peut sélectionner le navigateur via une option pytest, une variable d'environnement ou une configuration CI.

Exemple simplifié :

```python
import os
import pytest
from selenium import webdriver


@pytest.fixture
def driver():
    browser = os.getenv("BROWSER", "chrome")

    if browser == "firefox":
        driver = webdriver.Firefox()
    elif browser == "chrome":
        driver = webdriver.Chrome()
    else:
        raise ValueError(f"Navigateur non pris en charge: {browser}")

    yield driver
    driver.quit()
```

Puis :

```bash
BROWSER=firefox pytest
```

---

# 11. Selenium Grid et exécution distante

## 11.1. Pourquoi Grid ?

Selenium Grid permet de lancer WebDriver sur une infrastructure distante afin de :

- paralléliser les tests ;
- utiliser plusieurs navigateurs ;
- utiliser plusieurs versions ;
- tester sur plusieurs systèmes ;
- centraliser les navigateurs d'une CI.

## 11.2. Grid 4 n'est plus le Grid Selenium 3 des anciens tutoriels

Un ancien tutoriel peut montrer :

```bash
# Ancien Selenium 3 — ne pas utiliser comme exemple moderne
# java -jar selenium-server-standalone-3.x.y.jar -role hub
# java -jar selenium-server-standalone-3.x.y.jar -role node ...
```

Selenium Grid 4 possède une architecture différente et plusieurs modes de déploiement.

## 11.3. Mode Standalone

Pour apprendre ou pour une petite CI :

```bash
java -jar selenium-server-<version>.jar standalone
```

Le serveur écoute par défaut sur :

```text
http://localhost:4444
```

Prérequis courants :

- Java 11 ou supérieur ;
- un navigateur ;
- le serveur Selenium ;
- un driver détectable, ou l'utilisation de Selenium Manager selon la configuration.

## 11.4. Se connecter avec `Remote`

```python
from selenium import webdriver

options = webdriver.ChromeOptions()

driver = webdriver.Remote(
    command_executor="http://localhost:4444",
    options=options,
)

try:
    driver.get("https://www.selenium.dev/")
    print(driver.title)
finally:
    driver.quit()
```

Les anciennes constructions utilisant `desired_capabilities=` doivent être remplacées par des objets `Options` adaptés.

## 11.5. Hub et Node

Démarrer le Hub :

```bash
java -jar selenium-server-<version>.jar hub
```

Démarrer un Node sur la même infrastructure :

```bash
java -jar selenium-server-<version>.jar node --hub http://<hub-ip>:4444
```

Grid 4 contient plusieurs composants internes : Router, Distributor, Session Map, New Session Queue, Event Bus, Node, etc. En mode Hub/Node, plusieurs de ces composants sont regroupés pour simplifier le déploiement.

## 11.6. Mode distribué

Dans une grande infrastructure, les composants de Grid peuvent être lancés séparément. Ce mode est réservé aux besoins où l'exploitation d'une grille importante justifie cette complexité.

## 11.7. Grid avec Docker

Docker est fréquemment utilisé pour isoler les navigateurs et faciliter le provisionnement d'une infrastructure Selenium.

Il faut cependant distinguer :

- la documentation officielle Selenium Grid ;
- les images Docker du projet Selenium ;
- la configuration réseau, ressources et sécurité propre à notre infrastructure.

Voir [[Docker]].

## 11.8. Dimensionnement

Un navigateur consomme de la mémoire et du CPU. Lancer 50 sessions en parallèle n'accélère pas automatiquement une suite si la machine ne peut réellement en exécuter que 10 correctement.

Le nombre de sessions doit être déterminé par mesure :

- CPU ;
- mémoire ;
- temps moyen de session ;
- stabilité ;
- débit réseau ;
- capacité de l'application testée.

## 11.9. Sécurité du Grid

> [!danger]
> Un Selenium Grid ne doit pas être exposé directement à Internet.

Une personne capable de créer librement des sessions sur le Grid peut potentiellement :

- accéder à des applications internes depuis le navigateur ;
- exploiter les accès réseau du Grid ;
- interagir avec des données auxquelles le navigateur a accès ;
- abuser de la capacité d'exécution de l'infrastructure.

Il faut protéger le Grid par le réseau, les pare-feux et les mécanismes d'accès adaptés.

---

# 12. WebDriver BiDi, réseau et événements du navigateur

## 12.1. Limitation du WebDriver classique

Le modèle classique est principalement :

```text
client → commande → navigateur
client ← réponse  ← navigateur
```

Mais certaines informations sont naturellement **événementielles** :

- message `console.log` ;
- erreur JavaScript ;
- requête réseau ;
- événement de navigation ;
- création d'un contexte de navigation.

## 12.2. WebDriver BiDi

**WebDriver BiDi** est le protocole W3C bidirectionnel destiné à apporter une API standard, événementielle et multi-navigateurs.

Il utilise notamment une connexion WebSocket permettant au navigateur d'envoyer des événements vers le client sans attendre une commande classique.

```text
              commandes
client ─────────────────────► navigateur
       ◄─────────────────────
              événements
```

## 12.3. Activer BiDi en Python

Selon les API utilisées :

```python
from selenium import webdriver

options = webdriver.ChromeOptions()
options.enable_bidi = True

driver = webdriver.Chrome(options=options)
```

La surface de l'API BiDi évolue encore. Il faut donc consulter la documentation correspondant à la version de Selenium installée.

## 12.4. Domaines d'utilisation

Les API BiDi couvrent progressivement des domaines tels que :

- **browsing context** ;
- **log** ;
- **network** ;
- **script** ;
- événements d'entrée et de navigation.

Elles permettent par exemple de réagir à des messages de console ou à des événements réseau.

## 12.5. BiDi et Chrome DevTools Protocol

Selenium propose historiquement des accès au **Chrome DevTools Protocol (CDP)** pour certaines fonctions avancées de Chromium.

Cependant :

- CDP est lié à Chromium/Chrome ;
- ses versions suivent le navigateur ;
- Selenium considère son support CDP comme une solution transitoire pour les fonctions progressivement couvertes par WebDriver BiDi.

Pour du nouveau code multi-navigateurs, **préférer les API WebDriver/BiDi de haut niveau lorsqu'elles couvrent le besoin**.

## 12.6. Éviter de dépendre des API BiDi internes

Les classes de bas niveau du binding Python peuvent être marquées comme internes. Elles reflètent directement la structure du protocole et sont plus susceptibles de changer.

Lorsque Selenium fournit une API de haut niveau pour le réseau ou les scripts, celle-ci doit être préférée.

---

# 13. Débogage, erreurs fréquentes et stabilité

## 13.1. `NoSuchElementException`

Le locator ne trouve aucun élément.

Causes possibles :

- locator faux ;
- élément pas encore chargé ;
- mauvais iframe ;
- mauvaise fenêtre ;
- élément dans un Shadow DOM ;
- DOM différent de celui attendu.

Diagnostic :

1. vérifier l'URL ;
2. inspecter le DOM ;
3. vérifier le contexte frame/fenêtre ;
4. essayer le locator dans les DevTools ;
5. ajouter une attente explicite si l'élément est asynchrone.

## 13.2. `TimeoutException`

Une condition de `WebDriverWait` n'a pas été satisfaite dans le délai imparti.

Ne pas répondre immédiatement en passant le timeout de 10 à 60 secondes. Il faut d'abord savoir **quelle condition** ne devient jamais vraie et pourquoi.

## 13.3. `ElementClickInterceptedException`

Un autre élément reçoit le clic :

- overlay ;
- bannière de cookies ;
- animation ;
- menu ;
- loader ;
- élément hors zone visible.

La solution est de corriger l'état de la page ou l'attente, pas de transformer automatiquement tous les clics en JavaScript.

## 13.4. `ElementNotInteractableException`

L'élément existe mais ne peut pas être utilisé dans son état actuel.

Vérifier :

- visibilité ;
- `disabled` ;
- bon élément parmi plusieurs correspondances ;
- animation ;
- contexte.

## 13.5. `StaleElementReferenceException`

Le DOM a été remplacé depuis l'obtention de la référence. Voir chapitre 7.

## 13.6. Session ou navigateur impossible à démarrer

Vérifier :

```bash
python -c "import selenium; print(selenium.__version__)"
```

puis :

- navigateur disponible ;
- architecture système ;
- droits d'exécution ;
- proxy ;
- accès réseau nécessaire à Selenium Manager ;
- logs du driver ;
- compatibilité des versions dans un environnement géré manuellement.

## 13.7. Tests « flaky »

Un test **flaky** réussit ou échoue sans changement fonctionnel correspondant.

Causes classiques :

- `sleep()` arbitraires ;
- locators dépendant de la position ;
- état partagé entre tests ;
- animations ;
- ordre de tests implicite ;
- données de test non déterministes ;
- environnement surchargé ;
- appels externes non maîtrisés ;
- éléments localisés avant reconstruction du DOM.

## 13.8. Principes pour réduire la flakiness

1. Utiliser des locators stables.
2. Attendre l'état réellement nécessaire.
3. Rendre les tests indépendants.
4. Contrôler les données de test.
5. Éviter les dépendances externes inutiles.
6. Ne pas mélanger attente implicite et explicite.
7. Capturer assez d'informations lors d'un échec.
8. Garder les scénarios E2E ciblés.
9. Ne pas utiliser JavaScript pour masquer un comportement utilisateur impossible.
10. Mesurer les causes d'échec au lieu d'ajouter aveuglément des retries.

## 13.9. Les retries ne sont pas une réparation

Relancer automatiquement un test défaillant peut être utile pour distinguer certains incidents d'infrastructure, mais un test qui ne passe que « au deuxième essai » reste un signal à analyser.

Les retries ne doivent pas devenir une manière d'accepter une suite instable.

---

# 14. Selenium pour l'automatisation et la collecte de données

## 14.1. Selenium n'est pas un parseur de pages

Pour une page statique :

```text
HTTP → HTML → parseur
```

est généralement plus efficace que :

```text
HTTP → navigateur complet → JavaScript → DOM → Selenium
```

Selenium est pertinent lorsque le navigateur apporte réellement quelque chose :

- rendu JavaScript ;
- interaction ;
- session ;
- navigation complexe ;
- authentification interactive ;
- comportement dépendant du navigateur.

## 14.2. Préférer une API lorsqu'elle existe

Si l'application appelle une API JSON documentée et autorisée, utiliser cette API peut être :

- plus rapide ;
- plus stable ;
- moins consommateur ;
- plus simple à tester.

Il ne faut cependant pas contourner des restrictions d'accès ou utiliser une API privée sans en respecter les conditions.

## 14.3. Respecter le site automatisé

Avant d'automatiser un service tiers, vérifier notamment :

- ses conditions d'utilisation ;
- les limitations d'accès ;
- les contraintes légales applicables ;
- la charge provoquée par l'automatisation ;
- le traitement des données personnelles ;
- les mécanismes d'authentification et d'autorisation.

Automatiser techniquement une action ne signifie pas automatiquement que nous sommes autorisés à la réaliser à grande échelle.

## 14.4. Ne pas utiliser Selenium pour contourner une protection

Les mécanismes anti-bot, CAPTCHA, restrictions d'accès ou contrôles de sécurité sont des limites explicites du service. Le but d'un cours Selenium est d'apprendre l'automatisation et les tests légitimes, pas de contourner ces protections.

---

# 15. Projet final et synthèse

## 15.1. Projet proposé

Construire une petite suite de tests pour une application Web de démonstration comprenant :

1. ouverture de la page ;
2. connexion ;
3. vérification de l'écran principal ;
4. création ou modification d'une donnée ;
5. interaction avec une modale ou une seconde fenêtre ;
6. vérification d'un résultat asynchrone ;
7. capture d'écran en cas d'échec ;
8. exécution sur Chrome puis Firefox ;
9. exécution locale puis sur Selenium Grid.

## 15.2. Contraintes pédagogiques

Le projet doit :

- utiliser `By` et non les anciennes méthodes `find_element_by_*` ;
- utiliser Selenium Manager dans le cas local normal ;
- utiliser au moins une attente explicite ;
- ne pas utiliser `time.sleep()` comme synchronisation principale ;
- centraliser les locators importants ;
- utiliser une fixture pytest ;
- être composé de tests indépendants ;
- comporter au moins un Page Object ou Component Object ;
- fermer proprement chaque session ;
- documenter les commandes pour lancer les tests.

## 15.3. Exemple d'arborescence

```text
project/
├── pages/
│   ├── __init__.py
│   ├── login_page.py
│   └── dashboard_page.py
├── tests/
│   ├── conftest.py
│   ├── test_login.py
│   └── test_profile.py
├── pyproject.toml
└── README.md
```

## 15.4. Ce qu'il faut retenir

### Selenium sert à piloter un navigateur réel

Il est particulièrement utile pour les tests E2E et multi-navigateurs.

### Selenium 4 a modernisé plusieurs pratiques historiques

En particulier :

- **Selenium Manager** prend normalement en charge la gestion des drivers ;
- les locators Python utilisent `find_element(By.…)` ;
- les paramètres de navigateur passent par les classes `Options` ;
- `switch_to.new_window()` gère directement onglets et fenêtres ;
- Grid 4 possède une nouvelle architecture ;
- WebDriver BiDi apporte progressivement les événements bidirectionnels standardisés.

### Les attentes sont fondamentales

La qualité d'une suite Selenium dépend davantage d'une bonne synchronisation que d'une multiplication de `sleep()`.

### Un bon test vérifie un comportement métier

Il ne suffit pas de réussir à cliquer. Nous devons vérifier l'effet observable attendu.

### Les tests E2E doivent rester ciblés

Ils sont indispensables pour certains parcours, mais coûtent plus cher que les tests de niveaux inférieurs.

---

# Aide-mémoire Selenium Python

## Créer un navigateur

```python
from selenium import webdriver

driver = webdriver.Chrome()
```

## Naviguer

```python
driver.get("https://example.org")
driver.back()
driver.forward()
driver.refresh()
```

## Localiser

```python
from selenium.webdriver.common.by import By

element = driver.find_element(By.ID, "id")
elements = driver.find_elements(By.CSS_SELECTOR, ".item")
```

## Interagir

```python
element.click()
element.clear()
element.send_keys("texte")
```

## Attendre

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, 10)
element = wait.until(
    EC.element_to_be_clickable((By.ID, "save"))
)
```

## Nouvelle fenêtre

```python
driver.switch_to.new_window("tab")
```

## Iframe

```python
driver.switch_to.frame("frame-id")
driver.switch_to.default_content()
```

## Capture

```python
driver.save_screenshot("screenshot.png")
```

## Fermer

```python
driver.quit()
```

---

# Documentation et sources

## Documentation officielle

- Documentation Selenium : https://www.selenium.dev/documentation/
- Documentation WebDriver : https://www.selenium.dev/documentation/webdriver/
- Stratégies de localisation : https://www.selenium.dev/documentation/webdriver/elements/locators/
- Attentes : https://www.selenium.dev/documentation/webdriver/waits/
- Fenêtres et onglets : https://www.selenium.dev/documentation/webdriver/interactions/windows/
- Selenium Grid : https://www.selenium.dev/documentation/grid/
- Démarrage de Grid : https://www.selenium.dev/documentation/grid/getting_started/
- WebDriver BiDi : https://www.selenium.dev/documentation/webdriver/bidi/
- API Python : https://www.selenium.dev/selenium/docs/api/py/

## Projet et versions

- Dépôt Selenium : https://github.com/SeleniumHQ/selenium
- Paquet Python Selenium : https://pypi.org/project/selenium/

## Pour aller plus loin

- [[Python]]
- [[HTML]]
- [[CSS]]
- [[Javascript]]
- [[Docker]]
- [[git]]
