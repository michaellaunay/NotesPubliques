---
titre: MU26 01 Sujet et cahier des charges
type: cours
statut: brouillon
para: ressource
domaines:
  - enseignement
themes:
  - genie-logiciel
  - cahier-des-charges
  - capture-des-besoins
  - democratie
  - decentralisation
  - meetup
resume: "Le sujet du meetup 2026 : le brief de départ de Kunereti, volontairement court et flou, et ce qu'il mélange (désir, besoin, contrainte, solution) avant l'élicitation."
niveau: debutant
auteurs:
  - Michaël Launay
langue: fr
confidentialite: publique
rag: true
schema_version: 1
uid: 01M7WTJM16QTG8ZJ0EQDQ2B81D
date_creation: 2026-11-20
publication:
  - notes-publiques
date_modification: 2026-11-20
metadata_verifiees: true
---
# Le parti pris : partir de zéro

Dans la réalité, Kunereti vient d'une réflexion sur la modernisation de KuneAgi, un logiciel serveur existant ([[kunereti]] en raconte l'origine). Pour le meetup, on fait comme si rien n'existait : ni code à reprendre, ni spécifications, ni vocabulaire hérité. Un porteur de projet arrive avec une envie d'une dizaine de lignes et un dépôt vide.

Ce parti pris sert la démonstration. Une modernisation commence par lire l'existant ; un démarrage de zéro commence par **faire parler** celui qui a le besoin. C'est l'élicitation, et c'est là que l'agent change le plus la pratique.

# Le brief

Voici le texte déposé dans `docs/brief.md`. C'est tout ce que les agents savent au départ.

```md
# Kunereti : brief

Je veux un outil qui permette à une association, une coopérative ou un
collectif de faire émerger des idées, de les transformer en propositions
écrites à plusieurs, et de décider ensemble.

Je ne veux pas de serveur central : personne ne doit pouvoir couper
l'accès, lire ce qui ne le regarde pas ou modifier un vote. Chacun doit
pouvoir travailler hors ligne et vérifier lui-même ce qui a été décidé.

L'outil doit être utilisable sur un téléphone par quelqu'un qui n'est
pas informaticien.

Je veux que l'IA aide à ranger et à résumer, pas qu'elle décide.

Je pense m'appuyer sur Git et sur des fichiers Markdown.
```

# Ce que ce brief mélange

Un client exprime d'abord un désir, rarement un besoin. Le brief ci-dessus contient quatre choses de nature différente, et la première tâche de l'analyse est de les séparer.

| Phrase | Nature | Ce qu'il faudra demander |
| --- | --- | --- |
| « faire émerger des idées, les transformer en propositions, décider ensemble » | Désir | Qui a une idée ? Qui écrit ? Qui décide, et comment ? |
| « personne ne doit pouvoir modifier un vote », « vérifier lui-même » | Besoin | Vérifier quoi, par qui, avec quelle preuve ? |
| « utilisable sur un téléphone par un non-informaticien » | Contrainte | Que voit-il de Git, des clés, des signatures ? |
| « l'IA aide à ranger, pas à décider » | Règle | Où passe la frontière ? Qui fait tourner le modèle ? |
| « je pense m'appuyer sur Git et Markdown » | Solution | À garder de côté : ce n'est pas un besoin |

# Ce qu'il ne dit pas

- Qui peut entrer dans la communauté, et qui le décide s'il n'y a pas d'administrateur.
- Ce qu'est une « idée » : un texte que quelqu'un possède, ou autre chose.
- Combien de personnes écrivent une proposition, et ce qui se passe quand elles ne sont pas d'accord.
- La méthode de vote, le quorum, le secret du vote.
- Ce qui se passe quand quelqu'un perd son téléphone, quitte le collectif ou se comporte mal.
- Ce qu'on fait de l'existant quand un collectif utilise déjà un autre outil.

Aucune de ces questions n'est technique. Toutes changent le logiciel. La suite : [[MU26 02 Mise en place du projet]], puis [[MU26 03 Élicitation]].

Retour au support : [[MU26 00 Développer avec Claude Code et Codex]].
