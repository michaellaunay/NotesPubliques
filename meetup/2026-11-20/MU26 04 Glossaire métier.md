---
titre: MU26 04 Glossaire métier
type: cours
statut: brouillon
para: ressource
domaines:
- enseignement
themes:
- genie-logiciel
- analyse
- glossaire
- intelligence-artificielle
- claude-code
- meetup
resume: 'Création du glossaire métier de Kunereti avec Claude Code à partir du journal d''élicitation : règles d''écriture d''une entrée, chasse aux synonymes et aux doubles sens, contre-lecture par Codex et règle de mise à jour.'
niveau: intermediaire
auteurs:
- Michaël Launay
langue: fr
confidentialite: publique
rag: true
schema_version: 1
uid: 01M7WTJM3BHT19D6K1BCW46SY0
date_creation: 2026-11-20
publication: []
date_modification: 2026-11-20
metadata_verifiees: false
---
# Remarque générale

Le glossaire métier fixe les mots avant qu'on écrive les scénarios. Il sert trois fois : aux humains, qui cessent de se disputer sur des mots ; aux scénarios, qui n'emploient que lui ; et plus tard au code, dont les noms de classes et de champs en découleront.

Avec un agent, il sert une quatrième fois. C'est **le contexte le plus rentable** que l'on puisse lui donner : quelques pages qu'il relit à chaque session et qui l'empêchent d'appeler la même chose de trois façons dans trois fichiers.

En 2024, le glossaire produit par ChatGPT définissait « Markdown », « Git » et « HTML » ([[MU Glossaire métier]]) : les mots du cahier des charges, que tout le monde connaissait. Le glossaire de 2026 est tiré du journal d'élicitation et définit les mots qui portent des **règles**.

> [!note] Prompts reconstitués
> Les prompts sont imaginés pour le meetup. Les entrées citées sont celles du glossaire réel : [[KR Glossaire métier]], 69 termes au moment du premier tableau de suivi, 74 après sa correction.

# Le prompt

**Moi**

```text
Écris docs/analyse/glossaire.md à partir de docs/elicitation/journal.md
et de besoin.md. Rien d'autre ne fait source.

Une entrée, c'est :
- un terme au singulier, tel qu'un membre du collectif le dirait ;
- une définition en une à trois phrases, qui dit ce que c'est ET la
  règle qui s'y attache (qui le crée, combien, ce qui le distingue) ;
- aucun mot technique, sauf s'il sera visible de l'utilisateur.

Avant d'écrire, donne-moi trois listes :
1. les mots que j'ai employés l'un pour l'autre dans le journal ;
2. les mots que j'ai employés dans deux sens différents ;
3. les mots dont tu as besoin et que je n'ai jamais définis.
Je tranche, puis tu écris.
```

Les trois listes sont le vrai travail. La rédaction des définitions vient après, et elle est facile.

# Ce que les trois listes ont donné

**Synonymes.** Le journal disait tantôt « tour », tantôt « cycle », tantôt « itération » ; tantôt « copie », tantôt « fork » ; tantôt « vote », tantôt « scrutin », tantôt « bulletin ». Décisions : *itération* est le terme, *cycle* est noté comme synonyme ; *copie personnelle* est le terme, *fork* son nom technique entre parenthèses. Pour le vote, il n'y avait pas un mot de trop mais deux objets confondus : le *scrutin* est la définition d'un vote (la question, les candidats, les votants, les dates), le *bulletin* est le vote d'un membre.

**Doubles sens.** L'agent en a trouvé deux.

- *Vérificateur* désignait à la fois le membre tiré au sort qui vérifie une identité et le programme qui contrôle les écritures. On a gardé le mot et écrit les deux sens dans la même entrée, pour que personne ne les confonde en silence.
- *Idée* désignait ce qu'un membre écrit et ce que l'IA en tire. C'est la question 5 de l'élicitation qui revient : *contribution* pour le premier, *idée* pour le second.

**Mots manquants.** L'agent avait besoin d'un mot pour ce qui désigne la version exacte d'un texte sur laquelle on vote (ce sera *empreinte*), et d'un mot pour ce qu'une IA produit et qu'on peut recalculer (ce sera *objet dérivé*). Le second est devenu une règle du projet : un objet dérivé n'a aucun poids dans les décisions.

# Extrait du glossaire

> - **Contribution** : Ce qu'un membre a à dire : une remarque, un besoin, une critique, une question. Personnelle, signée, figée à la publication.
> - **Idée (fiche d'idée)** : Fiche rédigée par l'agent curateur à partir de toutes les contributions qui disent la même chose. Elle cite ses sources mot pour mot. Elle n'appartient à personne et n'a aucun pouvoir de décision.
> - **Amendement** : Une ou plusieurs modifications corrélées, réunies par une même intention et justifiées par leur auteur.
> - **Scrutin** : La définition d'un vote : la question, les candidats, la liste figée des votants, les dates, le quorum, la méthode.
> - **Bulletin** : Le vote d'un membre : un fichier signé, un par votant.
> - **Jeton** : Marque de soutien ou d'opposition qu'un membre pose sur un contenu publié. Chaque membre en a sept ; quand ils sont tous posés, il faut en reprendre un pour le placer ailleurs.
> - **Vérificateur** : Deux sens à distinguer. *Membre vérificateur* : membre tiré au sort pour vérifier l'identité d'un candidat. *Vérificateur* tout court : les scripts de contrôle, hébergés hors des dépôts, qui refusent toute écriture mal formée, mal signée ou illégale.

Une bonne entrée porte une règle. « Chaque membre en a sept » est déjà une exigence ; « un par votant » est déjà une contrainte d'intégrité. Une définition qui n'apprend rien au développeur est une définition de dictionnaire, et l'agent en produit volontiers si on ne lui demande pas autre chose.

# La contre-lecture

**Moi**, à Codex

```text
Lis docs/analyse/glossaire.md, puis docs/elicitation/journal.md.
Pour chaque entrée, réponds à trois questions :
- la définition emploie-t-elle un terme qui n'est pas lui-même défini ?
- contredit-elle une réponse du journal ?
- deux entrées disent-elles la même chose ?
Rends un tableau : entrée, problème, passage du journal.
```

**Codex** (résumé) signale des définitions circulaires (*participant* défini par *groupe de travail*, lui-même défini par *participants* : acceptable, mais à savoir) et des termes employés dans les définitions sans être définis, dont *mention*, qui sert à définir le jugement majoritaire. Ce dernier point est resté ouvert : le tableau de suivi le retrouvera ([[MU26 06 Tableau de suivi glossaire-scénarios]]).

# Faire vivre le glossaire

Le glossaire n'est pas fini quand il est écrit. La consigne de `AGENTS.md` (« n'emploie dans un scénario que des termes du glossaire ») le fait grandir avec les scénarios : quand quatorze scénarios ont été ajoutés à l'analyse réelle, leurs termes y sont entrés avec eux (observateur, quarantaine, centre d'intérêt, événement de discussion…).

**Moi**, à chaque nouveau lot de scénarios

```text
Avant de rendre la main : liste les termes que tu as employés dans
les scénarios que tu viens d'écrire et qui ne sont pas dans le
glossaire. Pour chacun, propose soit une entrée, soit le terme du
glossaire qu'il aurait fallu employer. N'ajoute rien sans mon accord.
```

# Ce qu'il faut retenir

- Le glossaire se tire de l'élicitation, pas du brief.
- Demander les synonymes, les doubles sens et les manques avant les définitions.
- Une entrée porte une règle.
- Un mot à deux sens que l'on garde se signale dans l'entrée même.
- Le glossaire est le contexte permanent de l'agent : c'est pour cela qu'il doit rester court et exact.

La suite : [[MU26 05 Scénarios]]. Retour au support : [[MU26 00 Développer avec Claude Code et Codex]].
