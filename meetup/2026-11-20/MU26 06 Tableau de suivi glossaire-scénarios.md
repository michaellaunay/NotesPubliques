---
titre: MU26 06 Tableau de suivi glossaire-scénarios
aliases:
- Matrice de traçabilité glossaire-scénarios
type: cours
statut: brouillon
para: ressource
domaines:
- enseignement
themes:
- genie-logiciel
- analyse
- glossaire
- scenarios
- suivi
- intelligence-artificielle
- claude-code
- meetup
resume: 'Tableau de suivi qui indique dans quels scénarios sert chaque terme du glossaire de Kunereti : prompt de génération par script, extrait calculé sur les 69 termes et les 51 scénarios réels, lecture des anomalies (termes jamais employés, employés partout, absents du glossaire), pièges du comptage et bilan de la passe de correction.'
niveau: intermediaire
auteurs:
- Michaël Launay
langue: fr
confidentialite: publique
rag: true
schema_version: 1
uid: 01M7WTQC80MPZV03BTH6GYBA8P
date_creation: 2026-11-20
publication: []
date_modification: 2026-11-20
metadata_verifiees: false
---
# À quoi sert le tableau

Le tableau de suivi croise les termes du glossaire et les scénarios : une ligne par terme, une colonne par scénario, une marque quand le scénario emploie le terme. C'est une matrice de traçabilité réduite à sa forme la plus simple.

Il répond à quatre questions que la relecture ne sait pas traiter à l'échelle de 69 termes et 51 scénarios :

| Ce qu'on observe | Ce que cela veut dire |
| --- | --- |
| Un terme n'apparaît dans aucun scénario | Il est inutile, ou un scénario manque, ou les scénarios le disent autrement |
| Un terme n'apparaît que dans un scénario | Notion locale : à vérifier, souvent légitime |
| Un terme apparaît dans presque tous | Trop général pour distinguer quoi que ce soit, ou notion centrale à définir avec soin |
| Un mot revient dans plusieurs scénarios et n'est pas au glossaire | Terme métier oublié |

Plus tard, le même tableau dit quels scénarios relire quand on change la définition d'un terme. C'est son usage en régime de croisière.

# Le prompt

**Règle : on ne demande pas à un modèle de compter.** Un agent à qui l'on demande « dans quels scénarios apparaît le mot scrutin » répond de mémoire, avec assurance, et se trompe sur quelques cases. On lui demande d'écrire le programme qui compte.

**Moi**

```text
Écris scripts/suivi_glossaire.py et la cible `make suivi`.
Le script lit docs/analyse/glossaire.md (un terme par entrée en gras)
et docs/analyse/scenarios/*.md, et génère docs/analyse/suivi.md :

1. un tableau : un terme par ligne, le nombre de scénarios qui
   l'emploient, puis leurs numéros ;
2. les termes employés nulle part ;
3. les termes employés dans un seul scénario ;
4. les termes employés dans plus des trois quarts des scénarios ;
5. les mots ou groupes de mots présents dans au moins quatre
   scénarios, absents du glossaire, et qui ne sont pas des mots
   courants : candidats à une entrée.

Le fichier généré commence par un avertissement : ne pas modifier à
la main. Montre-moi d'abord le résultat sur trois termes, que je
vérifie à la main, avant de le lancer sur tout.
```

La dernière phrase est la plus importante. Un script écrit par un agent est un programme comme un autre : on le teste sur un cas dont on connaît la réponse.

# Extrait du résultat

Les chiffres qui suivent sont calculés sur les fichiers réels de ce coffre ([[KR Glossaire métier]] et les 51 fiches `KR Scénario`), par recherche du terme en début de mot, sans tenir compte de la casse. Le tableau est réduit au parcours nominal d'une proposition : scénarios 8 (contribution), 11 (idées), 14 (groupe), 18 (amendement), 19 (vote), 20 (dépouillement), 21 (clôture), 30 (soutien) et 31 (avis).

| Terme | Total sur 51 | 8 | 11 | 14 | 18 | 19 | 20 | 21 | 30 | 31 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Contribution | 14 | ● | ● | | | | | | | |
| Idée | 16 | ● | ● | ● | ● | | | | | |
| Idée mûre | 2 | | ● | ● | | | | | | |
| Groupe de travail | 19 | | ● | ● | ● | ● | ● | ● | | |
| Proposition | 27 | | ● | ● | ● | ● | ● | ● | ● | ● |
| Section | 13 | | | ● | ● | | ● | | | |
| Amendement | 18 | | ● | ● | ● | ● | ● | ● | | |
| Empreinte | 12 | | ● | | ● | | ● | ● | | |
| Scrutin | 21 | | | | | ● | ● | ● | | |
| Bulletin | 14 | | | | | ● | ● | ● | | |
| Jugement majoritaire | 5 | | | | | ● | ● | | | |
| Dépouillement | 8 | | | | | ● | ● | ● | | |
| Co-signature | 1 | | | | | | ● | | | |
| Quorum | 6 | | | ● | | | ● | | | |
| Jeton | 13 | | | | | | | | ● | ● |
| Comité de pilotage | 5 | | | | | | | | ● | ● |
| Avis | 9 | ● | | | | | | | | ● |

Lu en diagonale, le tableau raconte le parcours : la contribution devient idée, l'idée devient groupe et proposition, la proposition reçoit des amendements, les amendements passent au scrutin, et les jetons n'arrivent qu'à la fin. Quand la diagonale se casse, il y a quelque chose à regarder.

# Ce que le tableau a trouvé

**Trois termes définis et comptés à zéro.** « Tirage au sort vérifiable », « Statut de membre » et « Durée de validité d'une décision » n'apparaissent dans aucun scénario sous cette forme. Pour le premier, l'écart est réel : la notion est partout (« tiré au sort » ou « tirage au sort » figure dans neuf scénarios), mais le glossaire la nomme d'une façon et les scénarios d'une autre. Il faut choisir, ou déclarer la variante. Pour les deux autres, c'est le comptage qui se trompe : voir les pièges plus bas.

**Neuf termes employés dans un seul scénario**, parmi lesquels « Citoyenneté » (scénario 3), « Scrutin scellé » (5), « Lien de catalogue » (7), « Co-signature » (20), « Quarantaine » (38). La plupart sont légitimes : une notion, un scénario. « Co-signature » mérite un regard : le glossaire dit que sans elle un résultat ne vaut rien, et un seul scénario en parle.

**Un terme partout.** « Dépôt » apparaît dans les 51 scénarios. Il ne distingue rien ; ce sont ses trois espèces (dépôt de communauté, d'idées, de groupe) qu'il faudrait suivre.

**Des mots absents du glossaire.** « Mention » revient dans huit scénarios et sert à définir le jugement majoritaire, sans avoir d'entrée : Codex l'avait signalé à la contre-lecture du glossaire ([[MU26 04 Glossaire métier]]), le tableau le confirme par les chiffres. De même « notice » (six scénarios), « fiche de membre » (cinq), « attestation » (quatre).

# Les pièges du comptage

Le tableau est calculé, donc exact sur ce qu'il compte. Reste à savoir ce qu'il compte. La première version du script en a donné trois exemples, tous instructifs.

- **« Participant » dans 51 scénarios sur 51.** Le chiffre était faux : chaque scénario contient un diagramme de séquence Mermaid, dont le mot-clé est `participant`. En ignorant les blocs de code, le terme tombe à 22 scénarios. C'est pour ce genre de cas qu'on vérifie trois termes à la main avant de croire le reste.
- **« Avis » au scénario 8.** Dans le glossaire, l'avis est la décision du comité de pilotage. Au scénario 8, le mot désigne l'avis de trois membres tirés au sort sur une contribution. La case est cochée à tort, ou plutôt elle révèle que le scénario emploie un mot du glossaire dans son sens courant, ce que la consigne interdisait.
- **Les pluriels et les flexions.** « Statut de membre » et « Durée de validité d'une décision » étaient comptés à zéro. Le scénario 40 écrit pourtant « les statuts de membre », et le scénario 44 « la durée de validité de la décision ». Les deux termes sont employés, et correctement. La première lecture du tableau les avait rangés parmi les termes inutiles : c'est en ouvrant les scénarios pour les corriger qu'on a vu qu'il n'y avait rien à corriger. Un zéro se vérifie comme un 51.
- **Les variantes.** « Tiré au sort » ne contient pas « tirage au sort vérifiable ». Une recherche de texte ne connaît ni les synonymes ni les flexions. On y remédie en ajoutant des variantes à chaque entrée du glossaire, pas en demandant au modèle de « comprendre » : le résultat doit rester reproductible.

# Le prompt de correction

**Moi**

```text
Lis docs/analyse/suivi.md. Pour chaque anomalie des sections 2 à 5,
propose une et une seule action parmi :
- supprimer l'entrée du glossaire ;
- ajouter une variante à l'entrée ;
- corriger le ou les scénarios (dis lesquels) ;
- ajouter une entrée ;
- ne rien faire, et dis pourquoi.
Rends un tableau. Ne modifie rien : je coche, puis tu appliques et
tu relances `make suivi`.
```

# Ce qu'il faut retenir

- Le tableau se génère ; il ne se rédige pas et ne se retouche pas.
- L'agent écrit le script, l'humain vérifie trois lignes à la main.
- Les anomalies sont le produit : zéro, un, partout, et hors glossaire.
- Un chiffre surprenant est d'abord une question sur le comptage, qu'il soit trop grand ou nul.
- La boucle est courte : corriger le glossaire ou les scénarios, relancer, relire.

# Après correction

Les anomalies ci-dessus sont celles des fichiers au matin du 20 novembre 2026. Elles ont été traitées le jour même, ce qui donne un exemple complet de la boucle.

| Anomalie | Action retenue | Résultat |
| --- | --- | --- |
| « Mention », « notice », « fiche de membre », « attestation », « phrase de secours » hors glossaire | Ajouter cinq entrées | Le glossaire passe de 69 à 74 termes ; les cinq sont suivis (6, 6, 5, 3 et 2 scénarios) |
| « Mention » employé aussi au sens courant (un rôle « est une mention dans la fiche », une synthèse affichée « sous la mention ») | Corriger les scénarios 6, 26 et 29 ; réserver le mot au vote dans le glossaire | Les six scénarios qui emploient encore le mot parlent tous du jugement majoritaire |
| « Avis » employé pour le comité, mais aussi pour les vérificateurs, les modérateurs et les jurés | Déclarer deux sens dans l'entrée, comme pour « Vérificateur » | Aucun scénario à réécrire : c'était le glossaire qui était trop étroit |
| « Tirage au sort vérifiable » jamais employé tel quel | Ajouter la variante « tiré au sort » à l'entrée | Le terme n'est plus à zéro |
| « Statut de membre », « Durée de validité d'une décision » à zéro | Ne rien faire | Fausses alertes du comptage |
| « Dépôt » partout, « Co-signature » dans un seul scénario | Ne rien faire | « Dépôt » est légitime ; la co-signature figure en réalité dans six scénarios sous la forme « co-signé » |

Trois leçons de cette passe.

- Sur les six lignes du tableau, une seule a demandé de toucher aux scénarios. Le plus souvent, c'est le glossaire qui était en retard sur eux.
- Deux lignes sur six relevaient, en tout ou en partie, d'un artefact du comptage. Le tableau désigne où regarder ; il ne dit pas quoi conclure.
- La décision « ne rien faire » s'écrit aussi. Sans elle, la même anomalie sera réexaminée à chaque génération du tableau.

> [!warning] État réel
> Les corrections ont été faites par un agent et attendent leur relecture : les fiches concernées de Kunereti portent `metadata_verifiees: false`. Le script utilisé ne connaît toujours ni les pluriels ni les flexions.

La suite : [[MU26 07 QQOQCP et CRUD]]. Retour au support : [[MU26 00 Développer avec Claude Code et Codex]].
