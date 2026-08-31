# NotesPubliques

Ce dépôt contient la partie publique de mes notes : ressources pédagogiques, documents, articles et réflexions, tous au format Markdown et rédigés dans [Obsidian](https://obsidian.md).

Les cours sont une réécriture complète et complémentaire des ressources disponibles sur [CoursGNULinux](https://github.com/michaellaunay/CoursGNULinux) et [CoursInformatique](https://github.com/michaellaunay/CoursInformatique). Tous les documents publics que j'ai produits depuis 2005 seront progressivement convertis en notes.

## Trois façons de lire ces notes

**En ligne, sans rien installer** — <https://www.logikascium.com/notes.html>
Le lecteur affiche les notes telles quelles : liens `[[wiki]]` navigables, diagrammes Mermaid, schémas Excalidraw rendus dans le navigateur, et la fiche de métadonnées de chaque note.

**Dans Obsidian** — clonez le dépôt et ouvrez-le comme coffre. C'est l'expérience complète : graphe, rétroliens, recherche.

**Dans un éditeur** — l'extension *Markdown Memo* pour Visual Studio Code comprend les liens `[[wiki]]`.

Comme ce dépôt est une copie partielle du dépôt principal, vous rencontrerez des liens vers des notes non publiées. Le lecteur en ligne les affiche en grisé plutôt qu'en lien mort.

## Ce qui est publié, et ce qui ne l'est pas

Le filtre est appliqué au niveau du dossier :

| Dossier | Contenu |
| --- | --- |
| `cours/` | supports de cours, licence et master |
| `informatique/` | fiches techniques, procédures, veille |
| `sciences/` | mathématiques, biologie, sciences sociales |
| `réflexions/` | textes personnels sur l'informatique et la connaissance |
| `Idées et concepts/` | fiches de concepts, frameworks, modèles |
| `meetup/` | supports de meetups |
| `templates/` | modèles de notes |
| `.nojekyll`, `index.html` | configuration de GitHub Pages |

Restent privés : les journaux quotidiens, les notes de prestation (couvertes par des accords de confidentialité), les brouillons, la thèse en cours, les projets de livres et les notes personnelles.

`_INDEX PRINCIPAL.md` n'est **pas** publié : il énumère tous les dossiers du coffre, y compris ceux qui restent privés. Le point d'entrée public est le lecteur, qui construit sa liste à partir du contenu réel de ce dépôt et ne peut donc pas citer ce qui n'y est pas.

Depuis 2026, chaque note porte un frontmatter YAML dont deux propriétés commandent la publication :

```yaml
confidentialite: publique   # publique | interne | privee | client
publication:
  - notes-publiques
```

Le filtrage par dossier délimite le périmètre ; ce sont ces deux propriétés qui décident, note par note. Une note peut donc cesser d'être publiée sans quitter sa place ni perdre ses liens.

Chaque note porte aussi un identifiant immuable :

```yaml
uid: "01M02JG1NBFPQ0DSVTJ6G19B4X"
```

C'est lui qui permet de reconnaître qu'une note d'ici et son originale dans le dépôt privé sont le même objet. Aucun identifiant Git ne pourrait tenir ce rôle : la construction de ce dépôt réécrit tous les hachages.

Les index de dossier `_INDEX_<dossier>.md` et leurs cartes `_INDEX_<dossier>.excalidraw.md` sont **générés** ; les éditer à la main n'a pas de sens, ils seront écrasés à la prochaine génération.

## Licence

Sauf mention contraire dans la note elle-même, les contenus sont publiés sous licence **Creative Commons Attribution – Partage dans les mêmes conditions 4.0 International (CC BY-SA 4.0)** : réutilisation et modification libres, y compris commerciales, à condition de citer la source et de partager les adaptations sous la même licence. Le texte complet est dans [LICENSE](LICENSE). Les notes issues de sources externes citent leur source dans leur frontmatter.

---

# Un peu de technique pour les curieux

## Comment ce dépôt est produit

Ce dépôt est **entièrement dérivé** du dépôt privé. Rien n'y est écrit à la main : ni les notes, ni les index, ni ce README, ni les fichiers de configuration de GitHub Pages. Tout provient d'une commande unique lancée depuis le coffre privé :

```bash
make publier POUSSER=1
```

Cette commande enchaîne six étapes, dont les trois premières sont des conditions d'arrêt et non des avertissements :

```text
1. le coffre privé doit être propre        publier un état non commité produit
                                           un dépôt que rien ne retrace
2. le schéma doit être valide              publier un coffre invalide propage l'erreur
3. les index doivent être à jour           un index périmé cite des notes retirées
4. calcul des exclusions                   depuis confidentialite et publication
5. construction                            filtrage des dossiers, remontée des
                                           fichiers de publication, neutralisation
                                           des messages de commit, retrait des exclusions
6. contrôles, puis poussée en force
```

Deux contrôles précèdent la poussée. Le premier vérifie qu'aucune note exclue ne subsiste dans l'arbre reconstruit. Le second, plus subtil, vérifie qu'**aucun lien ne divulgue le titre d'une note restée privée** : un lien mort n'est pas gênant — ce dépôt est partiel par construction, et le lecteur en ligne les affiche en grisé — mais `[[Une note privée]]` publierait son titre, donc son sujet, alors que son contenu a été retiré.

Les messages de commit sont remplacés par un texte neutre. Ceux du dépôt privé mentionnent des noms de clients et des titres de notes qui, eux, ne sont pas publiés.

## Comment ce dépôt était produit avant OSIA

Cette section n'a plus d'usage pratique. Elle est conservée parce que la procédure a servi de 2023 à 2026, qu'elle explique la forme de l'historique ancien, et que ses défauts éclairent le choix actuel.

Le dépôt était alors alimenté par report de rustines. Une création initiale :

```bash
git clone git@github.com:michaellaunay/Notes.git NotesPubliquesUp
cd NotesPubliquesUp
git filter-repo --path cours --path informatique --path sciences \
                --path réflexions --path templates \
                --path 'Idées et concepts' --path meetup
git remote add NotesPubliques git@github.com:michaellaunay/NotesPubliques.git
git push NotesPubliques
```

puis, à chaque mise à jour, un report incrémental des commits depuis le dernier hash noté dans `Versions.md` :

```bash
git filter-repo --path cours --path informatique …
git log                                    # repérer le dernier commit reporté
git format-patch -o /tmp/patches --root $HASH_BASE..HEAD
cd ../NotesPubliques
git am /tmp/patches/*
```

Une branche `Notes` portait le miroir filtré du dépôt privé, une branche `master` les modifications propres au dépôt public — ce README notamment.

Cette procédure présentait quatre défauts, dont le dernier est rédhibitoire.

**`git am` applique des rustines, pas des états.** Il échoue donc dès qu'un commit touche un fichier absent du dépôt cible, ce qui arrive constamment puisque le filtrage retire des chemins. Le contournement consistait à découper la livraison en plusieurs commits portant le même message, ce qui déplaçait le problème sans le résoudre.

**Les messages de commit privés étaient publiés.** Le contenu était filtré, pas les messages : « Correction du devis client X » restait lisible. C'était une fuite silencieuse, invisible à la relecture du contenu.

**Le hash de référence devenait caduc** à chaque réécriture d'historique, `filter-repo` renumérotant tous les commits. La reprise était donc manuelle et faillible.

**Rien ne garantissait la cohérence de ce qui partait.** Aucune vérification du schéma, aucun contrôle des index, aucune détection des titres privés cités par une note publiée. Les incohérences se découvraient après coup, dans le dépôt public.

La méthode actuelle règle les quatre points d'un coup, en renonçant à réconcilier deux historiques : le dépôt public est reconstruit et poussé en force, donc il n'y a plus rien à réconcilier. `Versions.md` n'a plus d'objet non plus, puisqu'il n'y a plus de reprise incrémentale à caler.

## Retirer une note du domaine public

Côté dépôt privé, la note reste à sa place ; seules deux propriétés changent :

```bash
python3 -m osia.cli depublier . "réflexions/Ma note.md"
make index                  # les index publiés cessent de la citer
git commit -am "Dépublication"
make publier POUSSER=1
```

`depublier` positionne **ensemble** `confidentialite: privee` et `publication: []` : ne changer que l'une des deux laisse une incohérence que le validateur signale comme erreur bloquante.

L'ordre compte. Régénérer les index avant la dépublication laisserait le titre de la note retirée dans un fichier qui, lui, part ici — `make publier` refuse d'ailleurs de s'exécuter si les index ne sont pas à jour.

Comme la construction réécrit l'historique complet, la note disparaît aussi des versions antérieures. Cela reste une réécriture irréversible côté public, et une note qui a été publiée doit être considérée comme ayant été publique : GitHub conserve un temps les objets accessibles par leur empreinte, les forks éventuels ne sont pas affectés, et les moteurs de recherche ont pu indexer la page.

## GitHub Pages et Jekyll

Le dépôt contient un fichier `.nojekyll` : **la construction Jekyll est désactivée**.

Ce n'est pas un choix esthétique. GitHub Pages exécute Jekyll 3.10, qui applique le moteur de gabarits Liquid à tout fichier possédant un frontmatter — c'est-à-dire, désormais, à toutes les notes. Or une note technique contient légitimement des accolades doubles :

```text
{{date:YYYY-MM-DD}}      dans les modèles Obsidian et le cours sur Obsidian
{{ .NetworkSettings }}   dans le cours Docker
{{.State.Pid}}           dans le cours sur les namespaces
```

La première forme est une **erreur de syntaxe Liquid** — les deux-points ne sont pas admis dans une expression — et fait échouer la construction entière. Il n'existe pas, sur Jekyll 3, de moyen de désactiver Liquid globalement : `render_with_liquid: false` n'existe qu'à partir de Jekyll 4.

Autrement dit, un dépôt de notes techniques et Jekyll sur GitHub Pages sont structurellement incompatibles. `.nojekyll` sert donc les fichiers tels quels, et `index.html` redirige vers le lecteur.

## Le lecteur web

Le rendu en ligne est un fichier statique servi par GitHub Pages, sans étape de compilation : il va chercher les fichiers Markdown de ce dépôt à la demande via `raw.githubusercontent.com`. Le code se trouve dans le dépôt [logikascium](https://github.com/michaellaunay/logikascium) :

```text
notes.html            page du lecteur
js/notes-viewer.js    Markdown, frontmatter, liens [[wiki]], Mermaid, KaTeX, recherche
js/excalidraw-svg.js  rendu SVG des dessins Excalidraw d'Obsidian
```

Les schémas Excalidraw doivent être enregistrés en JSON non compressé pour être lisibles hors d'Obsidian : réglages du greffon → *Compatibility* → décocher la compression.

## La méthode complète

Ce dépôt est un sous-produit d'un système décrit en détail dans le cours **Obsidian OSIA : construire son système d'exploitation personnel augmenté par l'IA**, publié ici même dans `cours/`. Le schéma de métadonnées y est traité au chapitre 5, le modèle métier au chapitre 6, la publication pilotée par les métadonnées à l'annexe B, et l'outillage à l'annexe C.
