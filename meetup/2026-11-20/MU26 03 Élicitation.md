---
titre: MU26 03 Élicitation
type: cours
statut: brouillon
para: ressource
domaines:
- enseignement
themes:
- genie-logiciel
- capture-des-besoins
- analyse
- 2tup
- intelligence-artificielle
- claude-code
- meetup
resume: 'Élicitation du besoin de Kunereti avec Claude Code : l''agent interroge le porteur du projet, tient le journal des questions et des hypothèses, reformule le besoin, puis Codex contre-lit. Prompts reconstitués et extraits des réponses.'
niveau: intermediaire
auteurs:
- Michaël Launay
langue: fr
confidentialite: publique
rag: true
schema_version: 1
uid: 01M7WTJM2WWK44P97Q2NQNDCGG
date_creation: 2026-11-20
publication: []
date_modification: 2026-11-20
metadata_verifiees: false
---
# Remarque générale

L'élicitation précise le besoin : elle fait passer du désir du brief ([[MU26 01 Sujet et cahier des charges]]) à un texte sur lequel on peut bâtir un glossaire et des scénarios. En 2024, on sautait presque cette étape : on donnait le cahier des charges à ChatGPT et on lui demandait un glossaire. Le résultat définissait les mots du texte, pas ceux du métier.

En 2026 on inverse les rôles : **c'est l'agent qui pose les questions**. Il le fait bien, pour trois raisons. Il ne se lasse pas, il ne fait pas semblant d'avoir compris, et il n'a aucune solution à défendre si on le lui interdit.

> [!note] Prompts reconstitués
> Cette conversation n'a pas eu lieu sous cette forme : elle a été imaginée pour le meetup. Les réponses, elles, sont les décisions réelles du projet, que l'on retrouve dans [[kunereti]] et [[KR Analyse]].

# Le prompt d'ouverture

**Moi**

```text
Nous commençons l'élicitation. Lis docs/brief.md.

Ton rôle est celui d'un analyste, pas d'un architecte.
- Ne propose aucune solution et ne commente pas celles du brief.
- Pose-moi des questions, quatre au plus à la fois, de la plus
  structurante à la moins structurante.
- Pour chaque question, dis en une ligne pourquoi ma réponse change
  le logiciel.
- Après chaque série, consigne questions et réponses dans
  docs/elicitation/journal.md, et ce que tu as dû supposer dans
  hypotheses.md.
- Arrête-toi quand tu peux décrire, sans rien inventer, qui fait quoi
  du premier contact jusqu'à une décision prise. Reformule alors le
  besoin dans besoin.md.

Commence par séparer, dans le brief, ce qui est un désir, un besoin,
une contrainte et une solution.
```

Chaque ligne a une fonction. « Pas d'architecte » évite la dérive vers Git et les signatures dès la deuxième question. « Pourquoi ma réponse change le logiciel » élimine les questions de politesse. Le critère d'arrêt évite l'interrogatoire sans fin.

# Extraits du journal

Le journal complet compte une quarantaine de questions en sept séries. Voici celles qui ont le plus déplacé le projet.

| # | Question de l'agent | Pourquoi elle compte | Réponse |
| --- | --- | --- | --- |
| 1 | S'il n'y a pas de serveur, y a-t-il quand même un administrateur ? | Sans lui, tout ce qu'un administrateur fait d'habitude (admettre, modérer, réparer) doit être fait par quelqu'un d'autre | Non. Il n'existe aucun compte privilégié. Un rôle est attribué par un vote et se retire par un vote |
| 2 | Qui décide qu'une personne entre dans la communauté ? | C'est le premier acte collectif, et il n'y a personne pour le faire | Ses pairs : des membres tirés au sort vérifient que c'est une personne réelle et qu'elle n'est pas déjà membre |
| 3 | La communauté connaît-elle l'identité de ses membres ? | Cela décide de ce qui peut être écrit, donc copié, donc jamais effacé | Non. Un pseudonyme et rien d'autre |
| 5 | Quand vous dites « une idée », est-ce un texte qui appartient à son auteur ? | Si oui, il faut des droits d'auteur sur l'idée, des doublons, des fusions négociées | Non. Chacun écrit ce qu'il a à dire, c'est une contribution. L'idée est ce que plusieurs contributions ont en commun ; elle n'appartient à personne |
| 6 | Qui range les contributions en idées ? | Si ce sont des humains, il faut un rôle et un processus de plus | L'IA. C'est exactement « ranger sans décider » |
| 9 | Combien de personnes écrivent une proposition ? | Écrire à trois cents ne marche pas ; il faut une règle d'entrée | Un petit groupe : douze au plus, et une liste d'attente |
| 10 | Que se passe-t-il quand deux participants veulent deux textes différents ? | C'est le cœur de l'outil | Chacun propose sa modification avec sa raison, et le groupe vote entre les versions |
| 11 | Avec quelle méthode de vote ? | Elle détermine ce qu'est un bulletin | Le jugement majoritaire : une mention par version, la meilleure médiane l'emporte |
| 14 | Et si quelqu'un refuse le résultat ? | Sans serveur, on ne peut pas l'empêcher de partir avec le texte | Il part avec. Il copie tout et ouvre un groupe concurrent. C'est voulu |
| 17 | Le vote est-il secret ? | Un vote vérifiable par tous et secret est un problème difficile | Pas dans la première version : nominatif sous pseudonyme. À revoir |
| 22 | Que se passe-t-il si un membre perd son téléphone ? | Pas de « mot de passe oublié » sans serveur | Il est réadmis par ses pairs, comme la première fois |
| 31 | Qui fait tourner l'IA, s'il n'y a pas de serveur ? | L'agent d'IA est un acteur ; il lui faut un responsable | Un membre volontaire ou une tâche planifiée. N'importe qui doit pouvoir refaire le travail et comparer |

Trois remarques pour la salle.

- La question 5 est la plus rentable de toute l'élicitation. Elle remplace un objet (« l'idée ») par deux (« la contribution », « l'idée calculée »), et supprime un formulaire entier de l'interface.
- La question 14 fait d'un risque une fonction. L'agent l'avait posée comme un problème ; la réponse en a fait un principe.
- La question 17 ne se résout pas. Elle va dans les questions ouvertes, et c'est un résultat légitime de l'élicitation.

# Le prompt de reformulation

**Moi**

```text
Tu as atteint le critère d'arrêt. Écris docs/elicitation/besoin.md :
1. le besoin en dix lignes, sans aucun terme technique ;
2. la liste des acteurs, humains et non humains ;
3. ce qui est hors périmètre, et pourquoi ;
4. les questions restées ouvertes.
N'y mets rien qui ne soit pas dans le journal. Si tu veux ajouter
quelque chose, pose-moi d'abord la question.
```

**Claude Code** (extrait de `besoin.md`)

> Une communauté est un ensemble de personnes qui délibèrent sous une même charte. On y entre parce que d'autres membres, tirés au sort, attestent que l'on est une personne réelle. Chacun y est connu sous un pseudonyme.
>
> Un membre écrit ce qu'il a à dire. Une IA regroupe ce que plusieurs membres ont dit de semblable et en fait une idée, qui cite ses sources et n'appartient à personne. Quand une idée est portée par assez de monde, un membre ouvre un groupe de travail. Douze personnes au plus y co-écrivent une proposition, par tours : chacun propose ses modifications avec ses raisons, le groupe vote entre les versions, puis décide de publier ou de refaire un tour.
>
> Une proposition publiée est soumise à tous les membres, qui la soutiennent ou la rejettent. Des membres élus disent si elle sera mise en œuvre.
>
> Personne ne détient les données ni la décision : chacun a une copie de tout, peut travailler sans réseau, et peut refaire lui-même le calcul de n'importe quel vote.

Hors périmètre : héberger des réunions, tenir des agendas, gérer des cotisations (on en constate seulement l'effet sur les droits). Questions ouvertes : le secret du vote, l'exclusion d'un membre, le droit à l'effacement.

# La contre-lecture

**Moi**, à Codex

```text
Lis docs/brief.md, docs/elicitation/journal.md et besoin.md.
Tu n'as pas assisté à la conversation et c'est voulu.
1. Y a-t-il dans besoin.md une affirmation que le journal ne justifie pas ?
2. Y a-t-il deux réponses du journal qui se contredisent ?
3. Quelles questions aurait posées un juriste, un trésorier
   d'association, un membre de mauvaise foi ?
Rends une liste, sans rien corriger.
```

**Codex** (résumé) trouve une affirmation non justifiée (« douze au plus » : le journal dit « un petit groupe », le chiffre vient d'une réponse ultérieure qu'il faut rattacher) et trois questions que personne n'avait posées :

1. Sans registre d'identités, qu'est-ce qui empêche une personne exclue ou partie de revenir sous un autre pseudonyme ?
2. Si chacun a une copie de tout et que rien ne s'efface, que répond-on à un membre qui demande la suppression de ses écrits ?
3. Ceux qui vérifient une identité ou jugent un signalement signent leur avis : sont-ils encore anonymes ?

Ces trois points figurent aujourd'hui dans [[KR Analyse]] comme les endroits « où le passage au réparti ne va pas de soi ». Ils n'ont pas de bonne réponse ; ils ont désormais une place.

# Ce qu'il faut retenir

- L'agent interroge mieux qu'il ne devine. Lui interdire de proposer est la consigne la plus utile du prompt.
- Le journal est un livrable. Six mois plus tard, c'est lui qui dit pourquoi une règle existe.
- Une hypothèse écrite est une question posée ; une hypothèse tue est un bogue.
- Le contre-lecteur est précieux parce qu'il n'était pas là.

La suite : [[MU26 04 Glossaire métier]]. Retour au support : [[MU26 00 Développer avec Claude Code et Codex]].
