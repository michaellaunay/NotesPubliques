---
titre: MU26 08 Sketchs
aliases:
  - MU26 Croquis d'écrans
type: cours
statut: brouillon
para: ressource
domaines:
  - enseignement
themes:
  - genie-logiciel
  - analyse
  - scenarios
  - design
  - excalidraw
  - intelligence-artificielle
  - claude-code
  - meetup
resume: "Création des croquis d'écrans de Kunereti à partir des scénarios : description textuelle de l'écran validée avant le dessin, génération du fichier Excalidraw par l'agent, correspondance croquis-scénarios et ce que le croquis révèle."
niveau: intermediaire
auteurs:
  - Michaël Launay
langue: fr
confidentialite: publique
rag: true
schema_version: 1
uid: 01M7WTQC9A9AGH078JQTV2VK9M
date_creation: 2026-11-20
publication:
  - notes-publiques
date_modification: 2026-11-20
metadata_verifiees: true
---
# Remarque générale

Un sketch est un croquis d'écran : des rectangles, des textes, des boutons, sans couleurs ni charte graphique. Il sert à vérifier un scénario par un autre chemin. Le scénario dit « l'application vérifie l'empreinte de la clé racine » ; le croquis oblige à dire ce que Chloé **voit** à ce moment-là, et avec quels mots.

En 2024, on demandait à ChatGPT de décrire un écran et on le dessinait soi-même dans Excalidraw. En 2026, l'agent écrit le fichier Excalidraw. Le dessin n'est plus le travail ; le travail est de décider ce qu'il y a sur l'écran.

Kunereti a quatorze croquis, mobile et PC, visibles dans [[présentation_kunereti]].

> [!note] Prompts reconstitués
> Les prompts sont imaginés pour le meetup. Dans le projet réel, les croquis ont été faits pour un article de présentation, indépendamment des scénarios ; le meetup les présente dans l'ordre de la méthode. La correspondance croquis-scénarios donnée plus bas est une proposition, à relire.

# Étape 1 : décrire l'écran avant de le dessiner

Demander directement « dessine l'écran du scénario 2 » donne un dessin plausible et des textes inventés. On demande d'abord une description, que l'on corrige en trente secondes, et seulement ensuite le dessin.

**Moi**

```text
Pour le scénario 2, « Rejoindre une communauté », décris les écrans
du téléphone, sans rien dessiner. Pour chaque écran :
- l'étape ou les étapes du scénario qu'il couvre ;
- les zones, de haut en bas ;
- le texte exact de chaque libellé et de chaque bouton ;
- ce qui se passe quand on touche chaque bouton ;
- ce que l'écran montre quand ça échoue.

Règles : aucun mot technique que le glossaire n'autorise pas à
l'écran. L'utilisatrice ne doit jamais lire « Git », « clone »,
« commit » ou « clé publique ».
```

**Claude Code** (résumé) propose deux écrans.

| Écran | Étapes | Contenu, de haut en bas |
| --- | --- | --- |
| Rejoindre | 1, 2 | Titre « Rejoindre ». Cadre de visée, avec « Visez le QR code de la communauté ». Lien « Saisir une adresse à la place » |
| Communauté trouvée | 3 à 6 | Nom de la communauté, nombre de membres et de groupes. « Clé racine », son empreinte, et la mention « identique au QR code ». « Charte signée » avec sa date, bouton « Lire la charte ». Champ « Votre pseudonyme ». Phrase « Vos clés sont créées sur ce téléphone et n'en sortent pas. » Bouton « Demander mon admission » |

La description fait déjà son office : elle montre que l'étape 5 du scénario (« elle lui fait noter une phrase de secours ») n'a pas d'écran. Soit il manque un écran, soit l'étape est mal placée. On le voit avant d'avoir tracé un trait.

# Étape 2 : le dessin

**Moi**

```text
Dessine ces deux écrans dans un seul fichier Excalidraw,
docs/analyse/croquis/01_rejoindre.excalidraw.md, côte à côte, au
format d'un téléphone. Traits noirs, fond blanc, aucune couleur sauf
le bouton principal. Reprends les textes mot pour mot. Numérote
chaque écran avec les étapes du scénario qu'il couvre.
Ouvre ensuite le fichier et vérifie qu'aucun texte ne déborde de son
cadre.
```

Le croquis réel :

![[kunereti/kunereti_images/kunereti_croquis_01_rejoindre.excalidraw]]

Trois conseils pratiques.

- **Un fichier texte, donc un diff.** Un croquis Excalidraw est du JSON dans une note Markdown : il se versionne, se relit et se corrige par un prompt (« le bouton est trop bas », « ajoute l'écran de la phrase de secours »).
- **L'agent ne voit pas ce qu'il dessine**, sauf si on lui donne le moyen de regarder le rendu. Les textes qui débordent et les éléments qui se chevauchent sont ses fautes typiques. Soit on lui fournit une capture, soit on regarde soi-même.
- **Pas de finition.** Un croquis propre invite à discuter des couleurs. Un croquis brut invite à discuter de ce qui manque.

# Étape 3 : relier les croquis aux scénarios

Comme le glossaire, les croquis se suivent. Un tableau dit quel scénario chaque croquis illustre, et surtout quels scénarios n'en ont pas.

| Croquis | Support | Scénarios illustrés |
| --- | --- | --- |
| 01 Rejoindre | Mobile | 2 |
| 02 Contribuer | Mobile | 8 |
| 03 Accueil | PC | 10, 25, 30 |
| 04 Idée | Mobile | 11, 12 |
| 05 Carte | PC | 13 |
| 06 Appel | Mobile | 10 |
| 07 Proposition | PC | 14, 15 |
| 08 Itération | Mobile | 21 |
| 09 Validation | PC | 17 |
| 10 Amendements | PC | 18 |
| 11 Jugement majoritaire | Mobile | 19 |
| 12 Version | PC | 20 |
| 13 Discussion et notifications | Mobile | 24, 25 |
| 14 Jetons et comité | Mobile | 30, 31 |

Quatorze croquis couvrent dix-sept scénarios sur 51. Les scénarios sans écran se rangent en trois familles, et elles n'appellent pas la même réponse :

- ceux où **il n'y a rien à voir**, parce que l'acteur est un agent ou un programme (11, 26, 33, 36) : pas de croquis, mais peut-être un compte rendu à afficher ;
- ceux qui **réutilisent un écran** existant (22, dupliquer une proposition) : une variante suffit ;
- ceux qui **n'ont pas d'écran alors qu'il en faudrait un** : admettre un candidat (3), voter à bulletin scellé (5), signaler un contenu (28), contester un scrutin (34). Ce sont les parcours rares, ceux qu'on dessine en dernier et qu'on rate en premier.

**Moi**

```text
Dresse le tableau croquis × scénarios. Pour chaque scénario sans
croquis, classe-le : rien à voir, écran réutilisé (lequel), ou écran
manquant. Pour les écrans manquants, décris-les comme à l'étape 1,
sans les dessiner.
```

# Ce que le croquis apporte à l'analyse

- **Le vocabulaire de l'écran.** Le glossaire dit *empreinte de la clé racine* ; l'écran dit « Clé racine » et « identique au QR code ». Si le mot du glossaire ne tient pas sur un bouton, c'est peut-être le glossaire qui a tort.
- **Les étapes sans écran, et les écrans sans étape.** La phrase de secours plus haut ; à l'inverse, un bouton que l'agent ajoute « parce qu'il y en a toujours un » et qu'aucun scénario ne justifie.
- **Les données affichées.** « 314 membres, 27 groupes » : l'écran affiche deux nombres que personne n'avait prévus dans l'inventaire des objets. Il faut pouvoir les calculer. On retourne au QQOQCP ([[MU26 07 QQOQCP et CRUD]]).
- **Les états d'erreur.** « Ce que l'écran montre quand ça échoue » est la ligne du prompt qui produit le plus de questions.

# Ce qu'il faut retenir

- Décrire, valider, puis dessiner : dans cet ordre.
- Les textes de l'écran viennent du glossaire, ou le remettent en cause.
- Le croquis est un fichier texte versionné, que l'on corrige par un prompt.
- Suivre les croquis comme les termes : un tableau, et l'on regarde les trous.
- Chaque livrable d'analyse contrôle les autres. C'est le fil du meetup : [[MU26 00 Développer avec Claude Code et Codex]].
