---
schema_version: 1
uid: 01M02EX5AZ939RXR9542RWVJ2H
titre: Excalidraw dans Obsidian
aliases:
  - Excalidraw
  - Obsidian Excalidraw
  - Dessin visuel dans Obsidian
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - obsidian
  - excalidraw
  - visualisation
  - diagramme
  - prise-de-notes
  - automatisation
resume: "Cours complet sur Excalidraw et son intégration dans Obsidian : dessin, fichiers Markdown Excalidraw, liens et transclusions, frames, bibliothèques, templates, export, OCR, automatisation avec ExcalidrawAutomate, IA, sécurité, performances et workflows de visual thinking."
niveau: intermediaire
prerequis:
  - "[[Obsidian]]"
  - "[[Outils de modélisation textuels]]"
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2024-03-23
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Excalidraw dans Obsidian

> [!abstract] Objectif
> Utiliser Excalidraw comme un véritable outil de **pensée visuelle** dans Obsidian : créer des croquis, relier un dessin au graphe de connaissances, réutiliser des zones précises d'un schéma, annoter des documents, préparer une présentation, automatiser la génération de dessins et choisir quand préférer Mermaid, PlantUML, Canvas ou un autre outil.

Voir aussi : [[Obsidian]], [[MindMap sous Obsidian]], [[PlantUML pour Obsidian]], [[Outils de modélisation textuels]], [[Getting things done]].

> [!important] Idée centrale
> Excalidraw dans Obsidian n'est pas seulement « Excalidraw dans une fenêtre ». Le greffon ajoute une couche d'intégration au vault : fichiers Markdown Excalidraw, liens Obsidian, transclusions, références vers des éléments ou des frames, scripts, templates, export automatique et intégrations avec d'autres greffons.

# 1. Excalidraw, le composant et le greffon Obsidian

Il faut distinguer trois objets souvent confondus :

| Élément | Rôle |
|---|---|
| `excalidraw.com` | application web de tableau blanc et de dessin |
| composant Excalidraw | bibliothèque open source utilisée dans diverses applications |
| Obsidian Excalidraw | greffon communautaire qui adapte Excalidraw au modèle de fichiers et de liens d'Obsidian |

Le greffon Obsidian est développé indépendamment de l'application web. Il embarque une version adaptée du composant et ajoute de nombreuses fonctions propres à Obsidian.

Au 31 août 2026, la dernière version stable publiée du greffon est la **2.26.4**, publiée le 5 août 2026. Des versions 2.27 sont déjà proposées en préversion ; il est préférable de documenter les workflows sur les fonctions stables et de vérifier les notes de version avant d'utiliser une nouveauté expérimentale en production.

# 2. Quand utiliser Excalidraw ?

Excalidraw est particulièrement adapté aux contenus dont la **géométrie porte du sens** :

- carte mentale libre ;
- schéma d'architecture exploratoire ;
- storyboard ;
- annotation d'une capture d'écran ;
- parcours utilisateur ;
- carte conceptuelle ;
- tableau de synthèse dessiné ;
- support de cours ;
- wireframe ;
- schéma dont les éléments doivent être déplacés rapidement ;
- visualisation d'un raisonnement en cours de construction.

Il est moins adapté lorsque la source doit être :

- déterministe ;
- générée automatiquement à partir de texte ;
- facilement relue dans un diff Git ;
- validée par une grammaire ;
- modifiée massivement par programme.

Dans ces situations, Mermaid, PlantUML, D2, Graphviz ou Structurizr peuvent être plus adaptés. Voir [[Outils de modélisation textuels]].

# 3. Excalidraw et Obsidian Canvas ne répondent pas au même besoin

Obsidian Canvas organise principalement des **cartes et des fichiers** sur un espace visuel. Excalidraw manipule un dessin plus libre composé de formes, traits, textes, images, frames et liens.

Une règle pratique :

```text
organiser des notes existantes      -> Canvas
penser / dessiner librement         -> Excalidraw
produire un diagramme reproductible -> Mermaid / PlantUML / D2
```

Les outils peuvent être combinés. Un Canvas peut contenir un lien vers un dessin Excalidraw, et une note Markdown peut transclure une zone précise d'un dessin.

# 4. Installation dans Obsidian

Le greffon s'installe depuis les **Community plugins** d'Obsidian.

Procédure générale :

1. ouvrir les paramètres ;
2. ouvrir la section des greffons communautaires ;
3. rechercher `Excalidraw` ;
4. installer le greffon de Zsolt Viczian ;
5. l'activer ;
6. redémarrer ou recharger Obsidian si une fonctionnalité ne devient pas immédiatement disponible.

> [!warning]
> Un greffon communautaire exécute du code dans Obsidian. Il faut appliquer les mêmes règles de confiance qu'à une extension d'éditeur : vérifier le projet, limiter les greffons inutiles et maintenir les versions à jour.

# 5. Premier dessin

Après installation, plusieurs chemins sont possibles :

- bouton du ruban ;
- palette de commandes ;
- menu contextuel de l'explorateur de fichiers ;
- commande de création et d'insertion dans la note active.

Une bonne pratique consiste à définir un dossier dédié :

```text
Assets/
└── Excalidraw/
```

ou :

```text
Excalidraw/
├── Drawings/
├── Templates/
├── Scripts/
└── Library/
```

Cela évite de mélanger les dessins, scripts et bibliothèques avec les notes de cours.

# 6. Le format moderne : du Markdown contenant un dessin

Les anciennes versions utilisaient principalement des fichiers `.excalidraw` contenant du JSON.

Le format moderne du greffon est un **fichier Markdown**. Selon la configuration, son nom peut se terminer par :

```text
architecture.excalidraw.md
```

ou simplement :

```text
architecture.md
```

Le fichier contient :

- un frontmatter ;
- éventuellement du texte Markdown lisible ;
- une section de textes ;
- les données du dessin, souvent compressées ;
- des informations sur les fichiers intégrés.

Structure conceptuelle :

````markdown
---
excalidraw-plugin: parsed
---

# Excalidraw Data

## Text Elements
%%
...
%%

## Drawing
```compressed-json
...
```
````

> [!warning] Ne pas éditer aveuglément les données
> La section `# Excalidraw Data` appartient au format interne du greffon. Un linter Markdown, un script de normalisation ou un remplacement global peut corrompre un dessin s'il modifie cette zone sans connaître son format.

# 7. `.excalidraw` : format historique / mode de compatibilité

Un fichier `.excalidraw` reste utile pour échanger avec certaines applications, mais dans Obsidian il est maintenant traité comme un **format de compatibilité**.

Le greffon propose des commandes de conversion vers le format Markdown moderne :

```text
*.excalidraw -> *.excalidraw.md
```

ou :

```text
*.excalidraw -> *.md
```

Le format Markdown permet l'intégration plus profonde avec Obsidian :

- métadonnées ;
- textes indexables ;
- liens internes ;
- transclusions ;
- options spécifiques au dessin ;
- meilleure intégration à certains workflows de vault.

# 8. Comprendre `raw` et `parsed`

Les fichiers Excalidraw Markdown peuvent exposer des textes dans un mode brut ou interprété selon le frontmatter et les paramètres.

Le mode interprété permet notamment d'utiliser des liens et des transclusions dans certains éléments texte.

Le greffon permet de basculer entre :

```text
Excalidraw view
Markdown view
```

Cette capacité est précieuse pour diagnostiquer un fichier, mais la vue Markdown n'est pas destinée à modifier manuellement la scène JSON à la volée.

# 9. Anatomie d'une scène

Une scène Excalidraw est construite à partir d'éléments :

- rectangle ;
- ellipse ;
- losange ;
- ligne ;
- flèche ;
- dessin libre ;
- texte ;
- image ;
- frame ;
- éléments provenant de bibliothèques.

Chaque élément possède notamment :

- une position ;
- une taille ;
- un style ;
- un identifiant ;
- éventuellement un lien ;
- éventuellement des appartenances à des groupes ou frames.

L'identifiant d'un élément devient très utile dans Obsidian pour créer des références précises.

# 10. Sélection et manipulation

Les opérations fondamentales à maîtriser sont :

- sélection simple ;
- sélection multiple ;
- groupement ;
- verrouillage ;
- duplication ;
- alignement ;
- distribution ;
- changement d'ordre ;
- déplacement dans une frame ;
- redimensionnement ;
- rotation.

Une scène complexe devient rapidement difficile à maintenir si tout est groupé arbitrairement. Il faut utiliser les groupes pour des composants logiques et les frames pour des unités visuelles de plus haut niveau.

# 11. Les frames

Les frames servent à délimiter des zones structurées dans un dessin.

Elles sont utiles pour :

- une diapositive ;
- une étape d'un processus ;
- un écran d'interface ;
- un sous-système d'architecture ;
- une vue réutilisable dans une autre note.

Exemple de structure :

```text
Frame: Contexte
Frame: Architecture
Frame: Déploiement
Frame: Risques
```

Un dessin unique peut alors devenir une série de vues cohérentes.

# 12. Mode présentation

Excalidraw peut être utilisé comme support de présentation. Le principe est de construire des frames ou des zones successives, puis de naviguer dans le dessin.

Avantages :

- même fichier pour travailler et présenter ;
- possibilité de zoomer sur un détail ;
- narration spatiale ;
- pas de séparation stricte entre diapositives et tableau blanc.

Limites :

- un deck complexe peut devenir lourd ;
- l'export PDF doit être testé avant une présentation importante ;
- les polices ou contenus externes doivent être disponibles sur la machine de présentation.

# 13. Liens Obsidian dans le dessin

Un élément Excalidraw peut porter un lien vers :

- une note du vault ;
- un autre dessin ;
- une URL ;
- parfois une ressource intégrée.

Exemple conceptuel :

```text
[Rectangle « API »]
       |
       +--> [[API REST]]
```

Le dessin n'est donc pas une image morte : il peut participer au graphe de connaissances.

# 14. Liens vers le dessin depuis une note

Un dessin peut être transclus comme n'importe quelle ressource Obsidian :

```markdown
![[architecture.excalidraw.md]]
```

Selon la configuration de l'extension, l'extension `.md` peut être omise dans le lien.

La valeur ajoutée du greffon apparaît lorsqu'on ne veut pas afficher **tout** le dessin.

# 15. Référencer un élément précis

Le greffon sait copier un lien vers un élément sélectionné.

Forme conceptuelle :

```markdown
[[architecture#^elementID]]
```

Pour un élément texte, un tel lien peut servir de block reference.

Il est préférable d'utiliser les commandes de copie du greffon plutôt que d'écrire les identifiants à la main.

# 16. Référencer un groupe

Le préfixe `group=` permet d'embarquer l'ensemble d'un groupe lié à l'élément référencé.

Exemple :

```markdown
![[architecture#^group=elementID]]
```

Cela permet de construire un grand dessin source puis de réutiliser des composants dans différentes notes.

# 17. Référencer une zone

Le préfixe `area=` produit une découpe autour d'un élément ou d'une section.

Exemple :

```markdown
![[architecture#^area=elementID]]
```

Le paramètre `padding=` ajuste la marge de la découpe :

```markdown
![[architecture#^area=elementID,padding=40]]
```

Cas d'usage :

- afficher uniquement la partie « authentification » d'une architecture ;
- extraire un schéma pour un cours ;
- garder une seule source visuelle sans copier le dessin.

# 18. Référencer une frame

Les références de frame permettent de transclure une frame spécifique.

Le greffon sait produire des liens utilisant notamment :

```text
frame=
clippedframe=
```

Là encore, mieux vaut utiliser la commande « Copy frame link » afin d'éviter une syntaxe incorrecte.

# 19. Noms de frames et stabilité des liens

Pour un document de long terme, nommer explicitement les frames facilite la maintenance.

Exemple :

```text
01-contexte
02-flux-authentification
03-stockage
04-observabilite
```

Éviter les noms trop génériques comme :

```text
Frame 1
Frame 2
Frame 3
```

# 20. Insertion de notes Markdown dans Excalidraw

Le greffon peut intégrer des fichiers Markdown du vault dans une scène.

Selon les paramètres, l'embed peut être rendu comme :

- contenu interactif ;
- image ;
- représentation spécifique au thème.

Ce mécanisme permet de créer des dashboards ou cartes conceptuelles mêlant :

```text
formes + images + notes Markdown + liens
```

> [!tip]
> Une transclusion de note est plus maintenable qu'une copie manuelle de son texte dans le dessin lorsque le contenu doit rester synchronisé.

# 21. Images locales

Une image du vault peut être placée sur le canvas puis annotée avec :

- flèches ;
- encadrés ;
- surlignages ;
- texte ;
- masques ;
- liens.

Excalidraw est donc très pratique pour documenter :

- une interface ;
- un schéma matériel ;
- une photo de tableau ;
- une capture de logs ;
- une topologie réseau.

# 22. Images distantes

Le greffon peut aussi utiliser des images accessibles par URL.

C'est pratique, mais fragile :

```text
URL supprimée -> image disparue
réseau indisponible -> image indisponible
URL privée expirée -> image indisponible
```

Pour un cours ou une documentation durable, il est souvent préférable d'enregistrer les ressources importantes dans le vault, sous réserve des droits d'utilisation.

# 23. Vidéos et pages web

Un lien vidéo peut être représenté visuellement, par exemple via une miniature pointant vers la vidéo.

Il faut distinguer :

- une vraie incorporation locale ;
- une prévisualisation ;
- une miniature avec lien externe.

Ne pas supposer qu'une vidéo est sauvegardée dans le vault simplement parce qu'elle apparaît dans le dessin.

# 24. PDF

Excalidraw peut participer à des workflows PDF :

- importer ou convertir des pages ;
- annoter ;
- combiner avec des formes ;
- exporter un résultat.

Une page PDF convertie en image perd naturellement une partie de la sémantique originale du PDF. Pour une documentation accessible, conserver également le document source.

# 25. Bibliothèque de formes

Excalidraw possède un mécanisme de bibliothèque pour réutiliser des composants.

Exemples :

- symboles réseau ;
- icônes de services ;
- composants UI ;
- blocs pédagogiques ;
- éléments d'architecture ;
- formes propres à une organisation.

Le greffon permet aujourd'hui de stocker la bibliothèque dans un fichier du vault, ce qui est préférable au stockage historique dans `data.json` pour des raisons de stabilité et de synchronisation.

# 26. `.excalidrawlib`

Une bibliothèque peut être exportée sous forme de fichier :

```text
ma-bibliotheque.excalidrawlib
```

Cela permet :

- partage entre vaults ;
- sauvegarde ;
- versionnement de composants ;
- distribution à une équipe.

Une bibliothèque n'est pas un substitut aux symboles formels d'un outil UML : elle fournit surtout de la réutilisation graphique.

# 27. Templates

Un template Excalidraw initialise un nouveau dessin avec :

- formes ;
- frames ;
- styles ;
- métadonnées ;
- texte ;
- scripts Templater éventuels.

Exemples de templates utiles :

```text
Architecture technique
Cours
Réunion
Storyboard
User journey
ADR visuel
Présentation
```

# 28. Structure d'un template de cours

Une structure simple :

```text
+-------------------------------+
| Titre                         |
+-------------------------------+
| Objectif                      |
+---------------+---------------+
| Concepts      | Exemple       |
+---------------+---------------+
| Questions / synthèse          |
+-------------------------------+
```

Le but d'un template est d'accélérer la création sans empêcher l'évolution du dessin.

# 29. Frontmatter spécifique à Excalidraw

Le greffon accepte plusieurs propriétés de frontmatter pour contrôler le comportement d'un dessin.

Exemples disponibles dans l'API actuelle :

```yaml
excalidraw-plugin: parsed
excalidraw-default-mode: view
excalidraw-export-transparent: true
excalidraw-export-dark: false
excalidraw-export-padding: 10
excalidraw-export-pngscale: 2
excalidraw-export-embed-scene: false
excalidraw-autoexport: true
```

Toutes ces clés ne doivent pas être ajoutées systématiquement. N'utiliser qu'une option lorsqu'elle apporte une différence volontaire par rapport aux paramètres globaux.

# 30. Modes normal, view et zen

Les modes répondent à des contextes différents :

| Mode | Usage principal |
|---|---|
| normal | édition complète |
| view | consultation |
| zen | réduction de l'interface pendant la création ou la présentation |

Un fichier peut définir un mode par défaut avec le frontmatter correspondant.

# 31. Thème clair et sombre

Un dessin peut être affiché dans différents thèmes.

Tester :

- contraste du texte ;
- contraste des flèches ;
- couleurs de fond ;
- images transparentes ;
- export destiné à une présentation claire ou sombre.

Une couleur correcte sur fond sombre peut devenir illisible dans un export blanc.

# 32. Export SVG

Le SVG est généralement le meilleur format pour :

- documentation web ;
- texte et formes vectorielles ;
- zoom sans pixellisation ;
- impression de schémas simples.

Avantages :

```text
vectoriel
léger pour les dessins simples
qualité indépendante de la résolution
```

Limites :

```text
certaines polices doivent être disponibles
les images raster restent raster
certaines destinations filtrent le SVG
```

# 33. Export PNG

Le PNG est utile lorsque la cible ne gère pas correctement le SVG.

Le facteur d'échelle peut être configuré.

Exemple :

```yaml
excalidraw-export-pngscale: 2
```

Un PNG ×2 est souvent suffisant pour un écran haute densité, mais il augmente la taille du fichier.

# 34. Export PDF

Le PDF convient aux :

- supports de cours ;
- impressions ;
- présentations distribuées ;
- archivages figés.

Toujours contrôler :

- pagination ;
- dimensions ;
- polices ;
- images ;
- marges ;
- zones coupées.

# 35. Scène embarquée dans un export

Un export peut, selon le mode, embarquer des informations permettant de retrouver une scène éditable.

Cette option augmente la portabilité, mais peut aussi :

- augmenter la taille ;
- exposer davantage d'informations que l'image visible ;
- poser un problème de confidentialité lors d'un partage externe.

> [!warning]
> Avant de publier un export, vérifier si des données éditables ou métadonnées internes sont incluses.

# 36. Auto-export

Le greffon peut produire automatiquement une copie SVG ou PNG à chaque sauvegarde.

Workflow :

```text
édition .md Excalidraw
        |
        +--> sauvegarde
              |
              +--> export.svg
```

C'est utile pour :

- un site statique ;
- un dépôt ne sachant pas interpréter Excalidraw ;
- un outil de documentation externe ;
- un pipeline CI simple.

# 37. OCR

Le greffon inclut un support OCR via des fonctions dédiées.

Cas d'usage :

- capture d'écran ;
- photo d'un tableau ;
- diagramme contenant du texte ;
- extraction avant transformation en note.

Mais un OCR ne doit pas être considéré comme exact :

```text
OCR -> vérification humaine -> correction -> réutilisation
```

# 38. Confidentialité de l'OCR

Selon le service OCR configuré, une image peut être envoyée à un service tiers.

Avant usage sur un contenu sensible :

1. identifier le fournisseur ;
2. lire ses conditions ;
3. vérifier le lieu de traitement ;
4. éviter les secrets, données personnelles ou documents confidentiels ;
5. préférer une solution locale lorsque la politique de sécurité l'impose.

# 39. Polices personnalisées

Le greffon permet d'utiliser une police personnalisée supplémentaire.

Pour un vault partagé :

- documenter la police ;
- vérifier sa licence ;
- prévoir un fallback ;
- tester l'export sur une machine différente.

Une police locale non distribuée peut dégrader le rendu sur mobile ou chez un collaborateur.

# 40. Stylets, crayons et surligneurs

Le greffon propose des réglages adaptés au dessin au stylet et des profils de crayons personnalisés.

Sur tablette, il faut tester :

- pression ;
- mode stylet ;
- effaceur matériel ;
- palm rejection du système ;
- taille des contrôles ;
- thème ;
- latence.

# 41. Usage mobile

Le greffon n'est pas réservé au desktop.

Sur mobile, les limites viennent davantage de :

- taille de l'écran ;
- mémoire disponible ;
- gros dessins ;
- nombreuses images ;
- performances de rendu.

Conseil : préférer plusieurs dessins cohérents à un seul canvas gigantesque.

# 42. Penser en composants

Une bonne scène complexe est hiérarchisée :

```text
scène
├── frames
│   ├── groupes
│   │   ├── formes
│   │   └── textes
│   └── images
└── liens
```

Cette structure facilite :

- déplacement ;
- réutilisation ;
- export ciblé ;
- présentation ;
- maintenance.

# 43. Éviter le « mega-canvas »

Un très grand dessin finit par poser plusieurs problèmes :

- navigation lente ;
- rendu coûteux ;
- miniatures lourdes ;
- conflits de synchronisation ;
- difficulté à trouver les éléments ;
- faible lisibilité sur mobile.

Découper lorsque le dessin contient plusieurs sujets indépendants.

# 44. Une règle de découpage

Créer un nouveau dessin si :

- une zone a son propre cycle de vie ;
- elle doit être réutilisée ailleurs ;
- elle peut être comprise seule ;
- elle contient beaucoup d'images ;
- elle devient une sous-architecture distincte.

Relier ensuite les dessins via des liens Obsidian.

# 45. Excalidraw et Git

Le fichier est textuel, mais les données de scène sont souvent du JSON compressé ou dense.

Conséquence :

```text
Git stocke bien les versions
mais
un diff humain du dessin reste médiocre
```

Git est utile pour :

- sauvegarde ;
- historique ;
- restauration ;
- synchronisation contrôlée.

Il l'est moins pour relire visuellement une modification élément par élément.

# 46. Conflits de synchronisation

Deux appareils modifiant simultanément le même dessin peuvent produire un conflit de fichier.

Règles :

1. laisser la synchronisation finir avant d'éditer ;
2. éviter d'ouvrir le même dessin en écriture sur deux appareils ;
3. conserver les fichiers de conflit jusqu'à vérification ;
4. utiliser les sauvegardes locales du greffon si nécessaire ;
5. tester la restauration sur un dessin non critique.

# 47. Sauvegardes locales

Le greffon conserve des mécanismes de sauvegarde locale destinés à récupérer certaines scènes après une erreur de sauvegarde ou fermeture inattendue.

Ce mécanisme ne remplace pas :

- une sauvegarde du vault ;
- Git ;
- snapshots ;
- sauvegarde distante.

Principe :

```text
backup applicatif != stratégie de sauvegarde
```

# 48. Obsidian Linter et autres outils de réécriture

Un linter peut modifier le contenu Markdown d'un fichier Excalidraw.

Il faut donc tester soigneusement :

- reformatage de YAML ;
- tri des propriétés ;
- normalisation de titres ;
- suppression de lignes ;
- espaces finaux ;
- conversion automatique.

Pour un linter agressif, exclure les fichiers Excalidraw ou utiliser les mécanismes de désactivation prévus par le linter.

# 49. Recherche et RAG

Le format Markdown est intéressant pour la recherche parce que certains textes du dessin sont représentés sous forme textuelle.

Mais une scène contient aussi de l'information **spatiale** qui n'est pas entièrement représentée par les mots.

Exemple :

```text
A -> B -> C
```

et :

```text
A au-dessus de B, C dans une frame à droite
```

ne sont pas équivalents pour un RAG purement textuel.

Pour une connaissance critique, accompagner le dessin d'une note explicative.

# 50. Accessibilité

Un dessin seul est difficile à exploiter avec certains outils d'assistance.

Pour les supports importants :

- ajouter un résumé textuel ;
- décrire les relations principales ;
- éviter l'information portée uniquement par la couleur ;
- utiliser des contrastes suffisants ;
- fournir un équivalent textuel du processus.

# 51. ExcalidrawAutomate

**ExcalidrawAutomate** est l'API d'automatisation du greffon.

Elle permet notamment de :

- créer des éléments ;
- lire une sélection ;
- modifier une scène ;
- créer un dessin ;
- ajouter des images ;
- utiliser des utilitaires Obsidian ;
- automatiser des workflows.

Elle est utilisée par :

- Script Engine ;
- Templater ;
- QuickAdd ;
- DataviewJS dans certains workflows.

# 52. Script Engine

Un script peut être stocké dans le dossier configuré pour Excalidraw Automate.

Formats acceptés :

```text
.md
.txt
.js
```

La condition essentielle est que le fichier contienne du JavaScript valide.

Une fois chargé, le script peut apparaître dans la palette de commandes et recevoir un raccourci clavier.

# 53. Objet `ea`

Dans un script lancé par le Script Engine, le greffon fournit un objet :

```javascript
ea
```

Il représente l'interface ExcalidrawAutomate associée au dessin cible.

Exemples d'intention :

```javascript
// obtenir de l'aide sur une fonction
ea.help("addText");

// accéder au module Obsidian exposé par ExcalidrawAutomate
const obsidian = ea.obsidian;
```

> [!warning]
> L'API évolue. Vérifier la documentation et utiliser `ea.help()` lorsque la signature exacte d'une fonction est importante.

# 54. Exemple de macro : encadrer une sélection

Pseudo-code pédagogique :

```javascript
const selected = ea.getViewSelectedElements();
if (!selected || selected.length === 0) {
  new Notice("Sélectionnez au moins un élément");
  return;
}

// Calculer la boîte englobante,
// ajouter un rectangle,
// puis synchroniser la scène.
```

Pour un vrai script, utiliser les helpers d'ExcalidrawAutomate plutôt que de réimplémenter la géométrie si une fonction existe déjà.

# 55. Principe de sécurité des scripts

Un script Excalidraw Automate est du JavaScript exécuté dans l'environnement Obsidian.

Il peut potentiellement :

- lire le vault ;
- modifier des fichiers ;
- contacter le réseau ;
- accéder à des APIs exposées par Obsidian.

Donc :

```text
script téléchargé = code à auditer
```

Ne jamais installer automatiquement une bibliothèque de scripts inconnue dans un vault contenant des secrets.

# 56. Templater

Excalidraw peut être combiné à Templater pour :

- nommer automatiquement un dessin ;
- insérer des métadonnées ;
- générer un titre ;
- préremplir des frames ;
- adapter un template au contexte de la note source.

Exemple conceptuel :

```text
Nouvelle note projet
      |
      +--> Templater
              |
              +--> nouveau dessin Excalidraw
                    prérempli avec le nom du projet
```

# 57. QuickAdd

QuickAdd peut servir de point d'entrée pour lancer un workflow :

```text
commande « nouvelle architecture »
  -> créer note
  -> créer dessin
  -> créer liens croisés
  -> ouvrir dessin
```

Cette approche est utile lorsqu'un vault suit des conventions strictes.

# 58. Dataview et dessins générés

ExcalidrawAutomate peut être utilisé depuis des workflows DataviewJS pour générer des représentations visuelles à partir de métadonnées.

Exemples :

- arbre familial ;
- mindmap de notes ;
- carte de dépendances ;
- tableau synthétique.

Mais dès que la visualisation est entièrement déterministe et générée à chaque build, D2, Mermaid ou Graphviz peuvent être plus simples à maintenir.

# 59. API Obsidian depuis ExcalidrawAutomate

Le script peut accéder à des fonctions de l'environnement Obsidian via les objets exposés.

Cela permet :

- créer un fichier ;
- ouvrir une note ;
- chercher des métadonnées ;
- construire un lien ;
- afficher une modal.

Il faut traiter cette capacité comme une API d'application, pas comme un simple langage de dessin.

# 60. IA dans Excalidraw

Les versions récentes du greffon comportent des fonctionnalités IA expérimentales.

La couche IA actuelle peut utiliser des profils de fournisseurs tels que :

- OpenAI ;
- Anthropic / Claude ;
- Google / Gemini ;
- xAI / Grok ;
- endpoints compatibles OpenAI, y compris certains endpoints locaux.

Elle est utilisée par différentes fonctions comme :

- dialogue Mermaid ;
- génération ou transformation de diagrammes ;
- diagram-to-code ;
- scripts ExcaliAI ;
- fonctions IA d'ExcalidrawAutomate.

# 61. Les clés API ne sont pas des données de dessin

Ne jamais écrire une clé API dans :

- un élément texte ;
- une note publiée ;
- un script commité publiquement ;
- un template partagé.

Les paramètres du greffon proposent une gestion dédiée des fournisseurs. Même lorsqu'une clé est obfusquée dans des paramètres, l'obfuscation n'est pas l'équivalent d'un coffre-fort cryptographique.

# 62. IA : confidentialité

Avant d'envoyer une image ou un dessin à un modèle :

- identifier les données visibles ;
- vérifier les notes transcluses ;
- retirer les secrets ;
- contrôler les URLs ;
- vérifier la politique du fournisseur ;
- préférer un endpoint local lorsqu'il répond aux exigences.

> [!important]
> Un diagramme d'architecture peut révéler autant d'informations sensibles qu'un fichier de configuration.

# 63. IA : ne pas remplacer le modèle source

Une génération IA doit produire une **proposition**, pas une vérité architecturale.

Workflow recommandé :

```text
besoin
 -> proposition IA
 -> revue humaine
 -> correction
 -> validation technique
 -> dessin final
```

# 64. Workflow de carte mentale

Pour une mindmap exploratoire :

1. écrire le thème au centre ;
2. créer quatre à huit branches majeures ;
3. ne pas commencer par le style ;
4. déplacer les branches après la première passe ;
5. transformer certains nœuds en liens vers des notes ;
6. créer des dessins secondaires pour les branches devenues complexes.

# 65. Workflow d'architecture logicielle

Utiliser plusieurs niveaux :

```text
Frame 1 : contexte
Frame 2 : conteneurs / services
Frame 3 : flux critique
Frame 4 : déploiement
Frame 5 : observabilité
```

Pour une architecture normative, compléter le dessin avec une description textuelle et éventuellement un modèle C4/Structurizr.

# 66. Architecture : conventions visuelles

Définir une légende :

```text
rectangle bleu   = service
rectangle vert   = stockage
losange          = décision
double bordure   = système externe
flèche pleine    = appel synchrone
flèche pointillée= événement
```

Ne pas introduire une nouvelle signification de couleur à chaque frame.

# 67. Diagrammes réseau

Excalidraw est utile pour un schéma réseau pédagogique ou de dépannage.

Pour une documentation d'exploitation, ajouter :

- adresses ou plages si elles peuvent être publiées ;
- protocoles ;
- sens des flux ;
- ports pertinents ;
- frontière de confiance ;
- rôle des pare-feux.

Éviter d'exporter publiquement une topologie interne détaillée.

# 68. Wireframes

Excalidraw est particulièrement efficace pour des wireframes rapides car son style « dessin » décourage la fausse précision.

Objectif :

```text
valider structure et parcours
avant
pixel-perfect design
```

# 69. User journey

Une frame par étape :

```text
Découverte
 -> Inscription
 -> Activation
 -> Première utilisation
 -> Retour
```

Ajouter :

- objectif utilisateur ;
- émotion ;
- friction ;
- donnée produite ;
- équipe responsable.

# 70. Réunion et facilitation

Excalidraw peut servir de tableau de réunion :

- brainstorming ;
- regroupement d'idées ;
- vote visuel ;
- cartographie de risques ;
- rétrospective.

À la fin, convertir les décisions en notes et tâches structurées. Le dessin ne doit pas devenir le seul endroit où une décision opérationnelle est stockée.

# 71. Cours et pédagogie

Pour enseigner :

- utiliser une frame par concept ;
- révéler progressivement ;
- conserver des espaces blancs ;
- utiliser les couleurs avec parcimonie ;
- lier les notions à des notes détaillées ;
- fournir le schéma final et un résumé textuel.

# 72. Annotation de capture d'écran

Workflow :

1. insérer la capture ;
2. verrouiller l'image ;
3. créer une couche de flèches ;
4. numéroter les étapes ;
5. ajouter une légende ;
6. exporter une zone si nécessaire.

Cette méthode est efficace pour une procédure technique.

# 73. Documentation d'incident

Excalidraw peut expliquer un incident :

```text
T0 requête entrante
T1 saturation file
T2 timeout
T3 retry storm
T4 indisponibilité
```

Le dessin complète la timeline textuelle ; il ne remplace pas les preuves issues des logs et métriques.

# 74. ADR visuel

Un Architecture Decision Record peut intégrer un schéma :

```text
Contexte
Options
Décision
Conséquences
```

Créer une frame pour chaque section puis transclure la frame correspondante dans l'ADR Markdown.

# 75. Excalidraw vs Mermaid

| Critère | Excalidraw | Mermaid |
|---|---|---|
| dessin libre | excellent | faible |
| génération textuelle | faible | excellent |
| diff Git | médiocre | excellent |
| annotation d'image | excellent | non |
| mise en page automatique | limitée | oui |
| présentation spatiale | excellente | moyenne |
| reproductibilité | moyenne | élevée |

# 76. Excalidraw vs PlantUML

PlantUML est préférable si :

- UML doit être reproductible ;
- le schéma doit être généré en CI ;
- la source textuelle doit être relue ;
- le diagramme suit une syntaxe précise.

Excalidraw est préférable pour :

- exploration ;
- annotation ;
- workshop ;
- conception visuelle rapide.

# 77. Excalidraw vs D2

D2 combine source textuelle et mise en page automatique moderne.

Utiliser D2 lorsque la géométrie exacte n'est pas importante et que la source textuelle doit rester la référence.

Utiliser Excalidraw lorsqu'on veut contrôler directement la position et la narration spatiale.

# 78. Excalidraw vs Canvas

Canvas manipule naturellement des notes et cartes comme objets de premier niveau.

Excalidraw manipule naturellement des **formes dessinées**.

Si le problème est « comment relier dix notes ? » : Canvas.

Si le problème est « comment représenter visuellement ce mécanisme ? » : Excalidraw.

# 79. Utiliser plusieurs outils dans un même cours

Exemple :

```text
concept exploré       -> Excalidraw
séquence normative    -> Mermaid
classes / composants  -> PlantUML
notes détaillées      -> Markdown
vue d'ensemble du PKM -> Canvas
```

Choisir un outil par fonction plutôt que chercher un outil unique.

# 80. Performance : principales causes de ralentissement

Une scène devient coûteuse avec :

- beaucoup d'éléments ;
- images très grandes ;
- nombreux embeds Markdown ;
- PDF rasterisés ;
- caches à reconstruire ;
- rendu simultané de plusieurs dessins dans une même note.

# 81. Réduire la taille des images

Avant d'insérer une capture 8K destinée à être affichée sur 500 px :

- redimensionner ;
- compresser ;
- supprimer les métadonnées inutiles si la politique le demande ;
- choisir PNG ou JPEG selon le contenu.

Une réduction avant import évite de transporter inutilement des dizaines de mégaoctets dans le vault.

# 82. Cache d'images

Les versions récentes du greffon utilisent des mécanismes de cache pour accélérer le rendu des scènes et des embeds.

Le cache peut être local à chaque appareil. Ne pas l'utiliser comme source de vérité : le dessin et ses ressources originales doivent rester récupérables sans ce cache.

# 83. Diagnostic d'un dessin lent

Procédure :

1. vérifier la taille du fichier ;
2. compter les images ;
3. identifier les plus grandes ;
4. tester sans embeds Markdown interactifs ;
5. vérifier si le problème existe dans un nouveau vault ;
6. vérifier la version du greffon ;
7. consulter la console développeur si nécessaire ;
8. tester une copie du dessin avant toute conversion destructive.

# 84. Dessin qui ne s'ouvre plus

Ne pas écraser immédiatement le fichier.

Procédure prudente :

1. faire une copie du vault ;
2. chercher une sauvegarde locale proposée par le greffon ;
3. vérifier les fichiers de conflit ;
4. ouvrir la vue Markdown uniquement pour inspection ;
5. vérifier la présence de `# Excalidraw Data` ;
6. vérifier l'historique Git / Sync ;
7. tester avec la version actuelle du greffon.

# 85. Conversion d'un ancien `.excalidraw`

Toujours convertir une **copie** si le dessin est important.

Après conversion :

- ouvrir le dessin ;
- tester les liens ;
- tester les images ;
- tester les fonts ;
- tester les exports ;
- seulement ensuite supprimer l'ancienne copie.

# 86. Images absentes

Questions à poser :

```text
image locale ou URL ?
chemin déplacé ?
fichier synchronisé ?
URL encore accessible ?
cache seulement ?
```

Un lien externe qui s'affichait hier peut disparaître sans que le dessin lui-même soit modifié.

# 87. Liens cassés après déplacement

Obsidian sait mettre à jour de nombreux liens internes lorsqu'un fichier est déplacé via l'application.

Un déplacement fait directement hors Obsidian peut ne pas bénéficier de la même logique.

Préférer :

```text
déplacement via Obsidian
puis vérification des backlinks
```

# 88. Export différent de l'écran

Vérifier :

- thème d'export ;
- fond transparent ;
- polices ;
- padding ;
- échelle PNG ;
- zone/frame sélectionnée ;
- images distantes ;
- embeds non exportables de la même manière qu'à l'écran.

# 89. Bonnes pratiques de nommage

Éviter :

```text
Drawing 1.md
Drawing 2.md
New drawing 7.md
```

Préférer :

```text
architecture-authentification.excalidraw.md
workflow-inscription.excalidraw.md
cours-systemd-processus.excalidraw.md
```

Le nom doit rester intelligible hors de l'interface graphique.

# 90. Dossier d'assets

Une structure possible :

```text
Assets/
├── Excalidraw/
│   ├── Drawings/
│   ├── Library/
│   ├── Scripts/
│   └── Templates/
├── Images/
└── PDFs/
```

Le bon choix dépend du vault ; l'important est la cohérence.

# 91. Liens bidirectionnels

Pour un dessin important, créer une note compagnon :

```markdown
# Architecture d'authentification

![[architecture-authentification.excalidraw.md]]

## Résumé
...

## Décisions
...
```

Puis ajouter depuis le dessin un lien vers cette note.

Cela améliore :

- recherche ;
- RAG ;
- accessibilité ;
- maintenance ;
- compréhension sans rendu graphique.

# 92. Anti-pattern : tout mettre dans le dessin

Ne pas transformer Excalidraw en base de données textuelle.

Mauvais usage :

- longs paragraphes ;
- procédures entières ;
- centaines de lignes de logs ;
- secrets ;
- configurations complètes.

Le dessin doit pointer vers la source détaillée.

# 93. Anti-pattern : utiliser la couleur sans légende

Exemple problématique :

```text
rouge = critique dans une frame
rouge = externe dans une autre
rouge = base de données dans une troisième
```

Définir une sémantique stable ou ajouter une légende.

# 94. Anti-pattern : copier-coller le même sous-schéma

Si cinq notes contiennent cinq copies visuelles du même bloc :

```text
une correction -> cinq mises à jour
```

Préférer une source unique et des embeds `group=`, `area=` ou `frame=`.

# 95. Anti-pattern : prendre Excalidraw pour un outil de modélisation formelle

Une flèche dessinée ne garantit pas :

- cardinalité ;
- type ;
- protocole ;
- contrainte ;
- cohérence syntaxique.

Pour une modélisation vérifiable, compléter ou remplacer par un langage adapté.

# 96. Anti-pattern : dépendre d'une URL privée

Une URL signée ou un lien de partage temporaire peut expirer.

Ne pas l'utiliser comme ressource durable d'un cours.

# 97. Anti-pattern : automatiser sans versionner le script

Un dessin généré dépend :

```text
script + version du greffon + données d'entrée
```

Versionner le script et documenter ses prérequis.

# 98. TP 1 — Premier dessin lié à une note

Objectif : créer une carte conceptuelle simple.

1. créer `Concepts/HTTP.md` ;
2. créer un dessin `HTTP.excalidraw.md` ;
3. dessiner client, serveur et proxy ;
4. ajouter des liens vers `[[HTTP]]`, `[[TLS]]` et `[[Proxy inverse]]` ;
5. transclure le dessin dans la note HTTP ;
6. vérifier les backlinks.

Critères :

- dessin ouvrable ;
- au moins trois liens internes ;
- titre clair ;
- ressource stockée dans le dossier prévu.

# 99. TP 2 — Réutiliser une zone

Créer un grand dessin d'architecture avec trois frames :

```text
Frontend
Backend
Données
```

Puis, dans trois notes différentes, insérer uniquement la zone correspondante avec les commandes de copie de lien de frame/area.

Objectif : vérifier qu'une modification du dessin source se reflète dans les vues réutilisées.

# 100. TP 3 — Template

Créer un template avec :

- titre ;
- frame « contexte » ;
- frame « flux » ;
- frame « risques » ;
- légende ;
- mode d'ouverture souhaité.

Configurer le template comme modèle par défaut puis créer deux dessins différents à partir de ce modèle.

# 101. TP 4 — Script Engine

Créer un script simple qui :

1. vérifie qu'un élément est sélectionné ;
2. demande une valeur à l'utilisateur ;
3. applique une modification via ExcalidrawAutomate ;
4. affiche un message de succès.

Le script doit :

- rester dans le dossier Scripts ;
- être versionné ;
- ne pas contenir de secret ;
- vérifier sa compatibilité si une API récente est utilisée.

# 102. TP 5 — Export documentaire

Créer un dessin avec :

- texte ;
- image ;
- flèches ;
- frame.

Exporter :

1. SVG ;
2. PNG ×2 ;
3. PDF.

Comparer :

- taille ;
- lisibilité ;
- comportement au zoom ;
- fond ;
- polices ;
- capacité d'édition ultérieure.

# 103. TP 6 — Audit de confidentialité

Prendre une copie d'un dessin réel et lister :

- URLs distantes ;
- notes transcluses ;
- noms de machines ;
- adresses ;
- identifiants ;
- images ;
- métadonnées ;
- éventuelle scène embarquée dans l'export.

Produire ensuite une version publiable sans donnée sensible.

# 104. Checklist avant publication

- [ ] Le dessin ne contient pas de secret.
- [ ] Les URLs externes sont nécessaires et stables.
- [ ] Les données personnelles sont autorisées à la publication.
- [ ] Les polices sont compatibles avec la diffusion.
- [ ] Le contraste clair/sombre a été testé.
- [ ] L'export n'embarque pas involontairement la scène complète.
- [ ] Un résumé textuel existe pour le contenu important.
- [ ] Les liens internes ne révèlent pas d'informations confidentielles.

# 105. Checklist avant synchronisation multi-appareils

- [ ] Le fichier a fini de se synchroniser.
- [ ] Les images locales sont synchronisées.
- [ ] La bibliothèque est stockée dans un emplacement synchronisé si nécessaire.
- [ ] Les polices personnalisées sont disponibles.
- [ ] Les scripts nécessaires sont présents.
- [ ] Les paramètres ne dépendent pas d'un chemin propre à une machine.

# 106. Checklist de diagnostic

```text
1. version Obsidian ?
2. version Excalidraw ?
3. format .md moderne ou .excalidraw historique ?
4. problème dans un vault minimal ?
5. image locale ou distante ?
6. conflit de sync ?
7. sauvegarde locale disponible ?
8. linter ou autre plugin ayant réécrit le fichier ?
9. scène particulièrement volumineuse ?
10. erreur dans la console développeur ?
```

# 107. Raccourcis conceptuels à retenir

Il existe de nombreux raccourcis et ils peuvent évoluer. Retenir surtout les actions :

- dupliquer ;
- grouper / dégrouper ;
- verrouiller ;
- zoomer ;
- aligner ;
- copier le lien d'un élément ;
- copier le lien d'une frame ;
- ouvrir en Markdown ;
- basculer mode raw/preview.

Configurer ses propres hotkeys Obsidian pour les commandes répétées est souvent plus durable que mémoriser tous les raccourcis du composant.

# 108. Configuration minimale recommandée pour un vault technique

```text
Dossier dessins       : Assets/Excalidraw/Drawings
Dossier templates     : Assets/Excalidraw/Templates
Dossier scripts       : Assets/Excalidraw/Scripts
Bibliothèque          : fichier dans le vault
Format                 : Markdown moderne
Auto-export            : selon besoin du site / CI
OCR                     : désactivé sauf besoin explicite
IA                      : désactivée sauf besoin explicite
```

# 109. Configuration pour un vault pédagogique

```text
format Markdown moderne
frames nommées
export SVG
PNG ×2 pour LMS si nécessaire
templates de cours
légende de couleurs stable
résumé textuel dans la note compagnon
```

# 110. Configuration pour visual PKM

Pour un système très visuel :

- liens fréquents vers des notes ;
- embeds Markdown ;
- scripts légers ;
- bibliothèques personnalisées ;
- frames réutilisables ;
- dessins index de domaine ;
- notes compagnons pour le texte long.

# 111. Ce que le greffon apporte par rapport à excalidraw.com

Dans Obsidian, le greffon ajoute notamment :

- intégration au vault ;
- liens Obsidian ;
- transclusions ;
- Markdown embeds ;
- références à des parties du dessin ;
- ExcalidrawAutomate ;
- intégration Templater/QuickAdd/Dataview ;
- paramètres de sauvegarde/export propres au vault ;
- fonctions OCR et IA optionnelles ;
- workflows de fichiers locaux.

# 112. Collaboration temps réel et synchronisation de fichiers

Ne pas confondre :

```text
collaboration temps réel
```

et :

```text
synchronisation du fichier dans le vault
```

L'application web Excalidraw propose des mécanismes de collaboration. Dans un vault, la synchronisation d'un fichier entre appareils ne fournit pas automatiquement la même sémantique qu'une session de tableau blanc multi-utilisateur.

Pour une session collaborative, choisir explicitement l'outil et le canal adaptés puis réintégrer le résultat dans le vault.

# 113. Gestion des versions

Avant une mise à jour majeure du greffon :

1. sauvegarder le vault ;
2. lire les release notes ;
3. tester un dessin complexe ;
4. tester un ancien `.excalidraw` ;
5. tester scripts et templates ;
6. seulement ensuite généraliser sur plusieurs machines.

Les versions beta sont utiles pour tester, pas pour rendre un cours ou un processus critique dépendant d'une fonction non stabilisée.

# 114. Conclusion

Excalidraw devient particulièrement puissant dans Obsidian lorsqu'on cesse de le considérer comme une simple image.

Le modèle à retenir est :

```text
Dessin
  + liens
  + transclusions
  + frames
  + Markdown
  + scripts
  + exports
  = ressource visuelle intégrée au système de connaissances
```

Les bonnes pratiques sont simples :

1. utiliser le **format Markdown moderne** ;
2. structurer les gros dessins avec frames et groupes ;
3. réutiliser les zones plutôt que les copier ;
4. garder le texte long dans des notes Markdown ;
5. versionner et auditer les scripts ;
6. traiter OCR et IA comme des échanges potentiels avec des services tiers ;
7. découper les mega-canvases ;
8. choisir Mermaid/PlantUML/D2 lorsqu'une source textuelle déterministe est préférable ;
9. sauvegarder le vault indépendamment du mécanisme interne du greffon.

# 115. Références officielles et techniques

- Projet Obsidian Excalidraw : <https://github.com/zsviczian/obsidian-excalidraw-plugin>
- README du greffon : <https://github.com/zsviczian/obsidian-excalidraw-plugin/blob/master/README.md>
- Releases du greffon : <https://github.com/zsviczian/obsidian-excalidraw-plugin/releases>
- Documentation ExcalidrawAutomate : <https://github.com/zsviczian/obsidian-excalidraw-plugin/tree/master/docs/API>
- Documentation Script Engine : <https://github.com/zsviczian/obsidian-excalidraw-plugin/blob/master/docs/ExcalidrawScriptsEngine.md>
- Community Wiki : <https://community.sketch-your-mind.com/Wiki>
- Excalidraw : <https://excalidraw.com/>
- Documentation développeur Excalidraw : <https://docs.excalidraw.com/>

# 116. Pour aller plus loin

Exercices de prolongement :

- créer une bibliothèque de composants d'architecture ;
- générer automatiquement une mindmap avec ExcalidrawAutomate ;
- comparer le même diagramme en Excalidraw, Mermaid et D2 ;
- créer un template de présentation à frames ;
- intégrer un dessin dans un pipeline de publication avec auto-export SVG ;
- auditer un vault pour détecter les images distantes utilisées par les dessins ;
- documenter une convention commune de couleurs, frames et dossiers pour une équipe.
