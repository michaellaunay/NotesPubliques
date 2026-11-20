---
titre: MU26 07 QQOQCP et CRUD
aliases:
  - QQOCP et CRUD
type: cours
statut: brouillon
para: ressource
domaines:
  - enseignement
themes:
  - genie-logiciel
  - analyse
  - scenarios
  - modelisation
  - donnees
  - intelligence-artificielle
  - claude-code
  - meetup
resume: "Passage des scénarios de Kunereti aux objets et aux données à créer : questionnement QQOQCP appliqué au scénario du vote, inventaire des objets, puis matrice CRUD objets × scénarios et ce qu'elle révèle dans un système où rien ne s'efface."
niveau: intermediaire
auteurs:
  - Michaël Launay
langue: fr
confidentialite: publique
rag: true
schema_version: 1
uid: 01M7WTQC8W0HVN3FS88Z3WWADX
date_creation: 2026-11-20
publication:
  - notes-publiques
date_modification: 2026-11-20
metadata_verifiees: true
---
# Remarque générale

Les scénarios disent ce que font les gens. Il reste à en tirer ce que le logiciel doit **conserver** : les objets et leurs données. Deux outils anciens et simples font ce passage.

- Le **QQOQCP** (Qui, Quoi, Où, Quand, Comment, Pourquoi, parfois abrégé QQOCP, et auquel on ajoute souvent Combien) interroge un scénario à la fois. Chaque réponse désigne un objet ou une donnée.
- La matrice **CRUD** (*Create, Read, Update, Delete* : créer, lire, modifier, supprimer) croise ensuite tous les objets et tous les scénarios. Elle montre ce qui manque.

Le premier ouvre, la seconde referme. Les deux se prêtent bien au travail avec un agent : ce sont des grilles, et un agent remplit bien une grille quand on lui interdit d'en sortir.

> [!note] Prompts et tableaux reconstitués
> Les prompts sont imaginés pour le meetup. La fiche QQOQCP et l'extrait de matrice CRUD ont été dressés pour l'occasion à partir des fiches de scénarios réelles ; ils n'ont pas encore été relus et ne font pas partie de l'analyse de Kunereti.

# QQOQCP sur un scénario

## Le prompt

**Moi**

```text
Applique le QQOQCP au scénario 19, « Voter les amendements au jugement
majoritaire », et écris docs/analyse/qqoqcp/19.md.

Pour chaque question (Qui, Quoi, Où, Quand, Comment, Combien,
Pourquoi), trois colonnes :
- la réponse, tirée du scénario, avec le numéro de l'étape ;
- l'objet que cela désigne (un terme du glossaire) ;
- les données que cet objet doit porter pour que l'étape soit possible.

Si le scénario ne permet pas de répondre, écris « non dit » et ajoute
la question à hypotheses.md. Ne complète pas de toi-même.
```

« Ne complète pas de toi-même » est la consigne qui fait la valeur de l'exercice. Les cases « non dit » sont le résultat.

## La fiche obtenue

Scénario de référence : [[KR Scénario 19 Voter les amendements au jugement majoritaire]].

| Question | Réponse (étape) | Objet | Données |
| --- | --- | --- | --- |
| **Qui** vote ? | Bob, participant du groupe (3 à 6) | Participant, membre | Pseudonyme, clé de signature, appartenance au groupe |
| **Qui** a le droit de voter ? | Ceux de la liste figée à l'ouverture (2, 7) | Scrutin | Liste des votants, figée |
| **Qui** contrôle ? | Le vérificateur (7) | Vérificateur | Aucune : il lit le scrutin et le bulletin |
| **Quoi** : sur quoi vote-t-on ? | Chaque amendement d'un groupe d'amendements concurrents, et le texte d'origine (2) | Scrutin, amendement, section | Liste des candidats ; pour chaque amendement, son intention, sa justification, les sections visées et leur empreinte |
| **Quoi** : que produit le vote ? | Un bulletin signé, un fichier par votant (6) | Bulletin | Identifiant du scrutin, une mention par candidat, signature |
| **Où** ? | Dans le dépôt du groupe de travail ; depuis l'application (4, 6) | Dépôt de groupe | Emplacement des bulletins sous leur scrutin |
| **Quand** s'ouvre le vote ? | À l'échéance de l'itération (1) | Itération | Date d'échéance |
| **Quand** se ferme-t-il ? | À la clôture, datée (2, 8) | Scrutin | Date de clôture |
| **Comment** regroupe-t-on les amendements ? | Par un calcul des recouvrements sur le texte, pas par un modèle (1) | Amendement | Passages visés, amendements concurrents |
| **Comment** vote-t-on ? | Une mention par version, parmi sept (5) | Bulletin | Mentions |
| **Comment** change-t-on d'avis ? | On remplace son bulletin ; seul le dernier compte (8) | Bulletin | Date, ou ordre, des bulletins d'un même votant |
| **Combien** ? | Un scrutin par groupe d'amendements ; un bulletin par votant ; douze votants au plus (2, 6, 8) | Scrutin, bulletin | Quorum |
| **Pourquoi** des mentions ? | Pour départager des versions concurrentes sans vote stratégique | Jugement majoritaire | Méthode inscrite dans le scrutin |
| **Pourquoi** figer la liste ? | Pour qu'un arrivant de dernière minute ne change pas le résultat | Scrutin | (déjà vu) |

## Ce que la fiche fait apparaître

- **« Seul le dernier compte »** suppose un ordre entre deux bulletins du même votant. Or l'élicitation a établi qu'il n'y a pas d'horloge commune. Qu'est-ce qui dit lequel est le dernier ? Non dit : hypothèse à trancher.
- **Le quorum** n'apparaît pas dans ce scénario ; il n'apparaît qu'au dépouillement. Faut-il l'afficher au votant ?
- **La mention** est une donnée du bulletin et n'avait pas d'entrée au glossaire. Troisième fois que ce mot se signalait, après la contre-lecture du glossaire et le tableau de suivi ; l'entrée a été ajoutée depuis.
- **Qui déclenche l'ouverture ?** L'étape 1 dit « la transition est déclenchée », à la voix passive. Le QQOQCP n'aime pas la voix passive : le « Qui » reste vide.

Une fiche, quatre questions. Sur 51 scénarios, on délègue (un sous-agent par cas d'utilisation, comme pour l'écriture) et on demande une consolidation.

## Des fiches à l'inventaire

**Moi**

```text
À partir de toutes les fiches de docs/analyse/qqoqcp/, écris
docs/analyse/objets.md : un objet par ligne, avec
- les données qu'il porte, dédoublonnées, et les scénarios qui les exigent ;
- qui le crée ;
- les objets auxquels il renvoie.
Signale les données demandées par un seul scénario, et les objets
qui ne portent aucune donnée.
```

L'inventaire converge vers une quinzaine d'objets : communauté et charte, membre, agent, contribution, question, idée, groupe de travail, proposition, section, amendement, scrutin, bulletin, jeton, avis, message, synthèse, processus. C'est la liste que la note de conception [[kunereti]] donne dans son tableau des types. Deux chemins indépendants qui arrivent au même endroit : c'est le meilleur signe qu'on puisse avoir à ce stade.

# La matrice CRUD

## Le prompt

Comme pour le tableau de suivi, la matrice est générée. Mais ici le script ne peut pas tout faire : savoir si un scénario *crée* ou *lit* un bulletin demande de comprendre la phrase. On sépare donc les deux travaux. L'agent annote chaque scénario, dans son en-tête ; le script assemble.

**Moi**

```text
1. Dans l'en-tête de chaque scénario, ajoute un champ `crud` : pour
   chaque objet de objets.md que le scénario touche, les lettres C, R,
   U, D qui s'appliquent et le numéro de l'étape qui le justifie.
   Pas d'étape, pas de lettre.
2. Écris scripts/crud.py et `make crud`, qui assemble ces champs en
   docs/analyse/crud.md : objets en lignes, scénarios en colonnes.
3. Sous le tableau, liste : les objets jamais créés, jamais lus,
   jamais modifiés, jamais supprimés ; et les scénarios qui ne
   touchent aucun objet.
```

« Pas d'étape, pas de lettre » rend chaque case vérifiable : on peut ouvrir le scénario et lire l'étape.

## Extrait

Même parcours nominal que pour le tableau de suivi.

| Objet | 8 Contribuer | 11 Tenir les idées | 14 Ouvrir un groupe | 18 Amender | 19 Voter | 20 Dépouiller | 21 Clore | 30 Soutenir | 31 Avis |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Contribution | C | R | | | | | | | |
| Idée | | C U | R | R | | | | | |
| Groupe de travail | | | C | | | | | | |
| Proposition | | | C | R | | U | U | R | U |
| Section | | | C | R | R | U | | | |
| Amendement | | R | | C | R U | U | | | |
| Scrutin | | | | | C | U | C | | |
| Bulletin | | | | | C | R | C | | |
| Jeton | | | | | | | | C | U ? |
| Avis | | | | | | | | | C |

## Ce que la matrice fait apparaître

**La colonne D est vide, partout.** Dans une application ordinaire, un objet qu'on ne supprime jamais est un oubli. Ici c'est un principe : un historique signé et copié par tous ne s'efface pas. Masquer un message le retire de l'affichage, pas de l'historique ; quitter la communauté révoque une clé, n'efface rien. La matrice ne révèle pas un scénario manquant, elle révèle une **question de droit** : que répond-on à qui demande l'effacement de ses écrits ? Elle figure dans les questions ouvertes du projet.

**Le U cache souvent un C.** Au scénario 30, Chloé « déplace » un jeton. À l'étape 4, l'application crée en réalité deux fichiers signés : le retrait de l'ancien jeton et la pose du nouveau. Rien n'est modifié. De même, « remplacer son bulletin » crée un second bulletin. Dans un système de fichiers signés, modifier et supprimer deviennent créer. La matrice CRUD classique doit donc être lue à deux niveaux : ce que l'utilisateur croit faire (U), ce que le système écrit (C). C'est une décision de conception qui est sortie d'un tableau d'analyse.

**Une case douteuse.** L'avis du comité « rend les jetons à leurs propriétaires » (scénario 31). Est-ce une modification des jetons, ou un effet que chaque application déduit de l'avis ? Le scénario ne le dit pas. D'où le point d'interrogation, et une ligne de plus dans `hypotheses.md`.

**Une ligne incomplète hors de l'extrait.** Le membre est créé (scénario 2), modifié (admission, rôles, profil, suspension des droits), et sa clé est révoquée quand il part (scénario 38). Rien ne dit comment une communauté exclut un membre contre son gré. L'analyse réelle le note parmi ce qui n'est « toujours pas couvert ».

# Ce qu'il faut retenir

- Le QQOQCP interroge un scénario ; le CRUD croise tous les scénarios. On fait les deux.
- Interdire à l'agent de compléter : la case vide est l'information.
- La voix passive dans un scénario est un « Qui » manquant.
- Chaque lettre de la matrice renvoie à une étape.
- Une anomalie de la matrice n'est pas toujours un oubli : c'est parfois une propriété du système, qu'il vaut mieux découvrir maintenant.

La suite : [[MU26 08 Sketchs]]. Retour au support : [[MU26 00 Développer avec Claude Code et Codex]].
