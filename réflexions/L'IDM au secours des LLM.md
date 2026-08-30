---
schema_version: 1
uid: 01M02JG1WEYSBT12VD1KGZP89B
titre: L'IDM au secours des LLM
aliases:
- IDM au secours des LLM
- Compression de l'Espéranto
- Peut-on transformer un problème de compréhension en un problème de génie logiciel
- Compression de l'Epéranto ou Peut on transformer un problème de compréhension en un problème de génie logiciel ou L'IDM au secours des LLM
type: reflexion
statut: a-relire
para: ressource
domaines:
- communication
themes:
- intelligence-artificielle
- linguistique
- esperanto
- ingenierie-des-modeles
- traitement-du-langage
resume: Essai — l'espéranto comme langue modélisable, la tokenisation morphologique bijective et le pragmène pour ramener ce qui peut l'être d'un problème de compréhension à un problème de génie logiciel traçable, l'ingénierie dirigée par les modèles encadrant le LLM.
auteurs:
- Michaël Launay
langue: fr
date_creation: 2025-08-31
date_modification: 2026-08-31
confidentialite: publique
publication:
- notes-publiques
rag: true
metadata_verifiees: false
---
> [!info] Essai
> Rédigé le 31 août 2026 à partir de mes échanges avec Gemini et ChatGPT (août-septembre 2025, puis une suite ultérieure), conservés tels quels dans [[Compression de l'Espéranto — échanges avec Gemini et ChatGPT]]. Les objections viennent de ces mêmes échanges ; les propositions sont les miennes. Statut `a-relire` : la thèse est assumée, la formulation reste à valider.

# L'IDM au secours des LLM

*Peut-on transformer un problème de compréhension en un problème de génie logiciel ?*

## 1. Le point de départ : une langue qui a moins de mots et moins de règles

L'[[Esperanto]] contient moins de mots et moins de règles que le français. Tout part de là. Sa morphologie est agglutinante et régulière : une racine, des affixes dont chacun a une valeur fixe, une terminaison qui dit sans exception la catégorie du mot — `-o` pour le nom, `-a` pour l'adjectif, `-e` pour l'adverbe, `-as`, `-is`, `-os` pour le présent, le passé et le futur. Un mot comme *malŝanĝebla* (« immuable ») se décompose mécaniquement en *mal-* (contraire), *ŝanĝ-* (changer), *-ebl-* (qui peut être), *-a* (adjectif). Le lexique est fini, la grammaire aussi.

De cette régularité j'ai tiré, dans l'ordre, quatre questions. Une langue plus concise est-elle plus précise ? Une grammaire dont la BNF s'écrit en quelques pages se prête-t-elle mieux à l'automatisation ? Peut-on automatiser l'analyse sémantique d'une telle langue ? Et surtout, puisque c'est mon métier : peut-on déduire plus directement des diagrammes UML ou BPMN d'un énoncé en espéranto que d'un énoncé en français ?

## 2. Ce que la critique a corrigé

J'ai demandé à un second assistant une critique scientifique de ces intuitions, et elle tient. Concision n'est pas précision : la précision d'une langue vient de la richesse de son lexique et de ses usages, et là où le français distingue le chagrin, la mélancolie et la déprime, l'espéranto tend vers *malĝojo* ou la périphrase. La régularité morphologique supprime des ambiguïtés formelles, pas les ambiguïtés de sens : *kapo* est une tête, un chef ou une extrémité, *banko* une banque ou un banc, *lumo* une lumière ou une lampe, et *ruĝa kiel tomato* ne s'analyse pas morphème par morphème. Enfin, les grands modèles de langage n'ont pas eu besoin d'une grammaire régulière pour traiter le français : ils l'ont contournée par la statistique. Sur le terrain du traitement automatique tel qu'il se pratique aujourd'hui, l'avantage comparatif d'une langue construite est faible.

Le point le plus précieux de cette critique concerne ma quatrième question. Prenons *La sistemo kontrolas la datumojn* — le système contrôle les données. Le parsing est immédiat : sujet, verbe au présent, objet à l'accusatif. Mais faut-il modéliser cette phrase comme une méthode d'une classe `Sistemo`, comme une transition d'un processus, ou comme une contrainte métier ? Le texte ne le dit pas. La syntaxe est résolue, la décision de modélisation ne l'est pas. Autrement dit, l'espéranto déplace la frontière entre ce qui est formel et ce qui ne l'est pas ; il ne la supprime pas.

C'est exactement ce déplacement qui m'intéresse, parce qu'une frontière qui bouge se laisse outiller.

## 3. Déplacer la frontière : tokeniser par la morphologie

Un LLM ne manipule pas des mots mais des tokens (voir [[LLM#1.2. Un LLM manipule des tokens, pas directement des mots]]). Un tokeniseur BPE apprend son découpage par fusion des paires de caractères les plus fréquentes, sans considération de sens ; c'est ensuite la rétropropagation qui fixe, dans les embeddings, ce que valent ces fragments. Le résultat fonctionne, mais rien dans un token BPE n'est lisible par un humain, et rien dans la chaîne ne se laisse tracer.

Ma proposition : puisque le lexique et la grammaire de l'espéranto sont finis, ne pas laisser un algorithme statistique décider du découpage, mais utiliser directement une fonction de compression bijective fondée sur la morphologie. Chaque token serait une unité linguistique — racine, préfixe, suffixe, terminaison, marque de pluriel ou d'accusatif — et non un fragment arbitraire. *La uzanto enigas la datumojn* devient, sans perte et sans ambiguïté, `DET(la) ROOT(uzant) -o V(ROOT(enig) -as) DET(la) ROOT(datum) -o -j -n`. La fonction inverse restitue la phrase ; surtout, elle restitue chaque décision du modèle sous une forme compréhensible. Ce que j'en attends, ce sont des gains de sobriété (un vocabulaire fermé et lisible — quelques dizaines d'affixes et de terminaisons, une liste de racines — au lieu de dizaines de milliers de fragments opaques, donc un modèle bien plus petit) et, avant tout, de la traçabilité : on saurait quelles briques de sens ont été assemblées, et l'on pourrait suivre une partie du traitement.

Une précision de vocabulaire s'impose : cette « compression » ne réduit pas la quantité d'information — un recodage sans perte n'y change rien — elle réorganise l'information en unités interprétables. Le gain de puissance ne vient pas de là, il vient de la taille du vocabulaire et de celle du modèle qu'il autorise.

La réponse que j'ai obtenue donne à cette idée une forme d'ingénierie : un transducteur fini réversible entre la surface et les morphèmes, avec une analyse canonique unique pour garantir la bijection ; une grammaire probabiliste explicite (règles `S → NP VP` et traits de temps, de mode, de cas) qui guide l'analyse et filtre la génération ; un modèle statistique volontairement petit — n-grammes sur morphèmes et un mini-Transformer de quelques dizaines de millions de paramètres — qui ne sert qu'à ranger les analyses et à gérer le contexte ; un décodage contraint par la grammaire ; et un journal de chaque règle, chaque affixe, chaque racine choisis. La grammaire y est un objet du génie logiciel au sens le plus classique, celui des [[Parsing Expression Grammars PEG]] et des langages dédiés du cours d'[[UML Ecore EMF Plantuml QVT Mermaid PyEcore#5. Grammaires et langages dédiés]].

Les limites, elles aussi, sont sorties de ces échanges et je les garde. Le lexique n'est fini qu'en apparence : la dérivation est productive et l'espace des formes est infini, ce qui impose une liste de racines et une solution de repli caractère par caractère pour les mots inconnus et les noms propres. La polysémie demeure, *banko* en tête. L'ordre des mots est libre grâce à l'accusatif, et la grammaire doit l'accepter. Les corpus en espéranto sont petits. Et une tokenisation purement morphologique perd une partie de ce que les embeddings d'un grand modèle capturent — comment distinguer la maison du foyer si la racine est la même ?

## 4. Ce que le génie logiciel ne prend pas en charge : la compression native de la phrase

Reste le vrai problème, que la régularité ne touche pas. Une phrase humaine est nativement compressée : elle repose sur la pragmatique, qui embarque de l'intention, de la culture et du non-dit, et c'est à l'interlocuteur de reconstruire la pragmatique qu'il croit percevoir. Aucune grammaire ne décompresse cela.

Je propose d'appeler **pragmène** la plus petite unité de sens implicite ou sous-entendu. Les grands modèles manipulent déjà des pragmènes sans les nommer : quand un modèle complète « le plombier a pris sa grosse… » par « clé » plutôt que par « voiture », il a activé un contexte implicite. L'idée est de rendre ce mécanisme explicite — d'entraîner un modèle spécialisé non pas à consommer le contexte pour prédire le mot suivant, mais à énoncer les pragmènes qu'il a déduits. La tâche est proche de la prédiction du mot le plus probable, ce qui la rend accessible aux techniques existantes ; la différence est que sa sortie est une liste d'implicites, révisables et traçables, chacun avec sa source et son score.

## 5. Les vues comme plans d'un espace, et la complétude d'un énoncé

Un énoncé peut ensuite être projeté dans plusieurs diagrammes qui en donnent des vues complémentaires : un diagramme d'objets pour les relations entre les choses nommées, un diagramme de classes pour le sens, un diagramme d'activités pour la dynamique. Chaque vue est un plan d'un espace multidimensionnel ; réfléchir, c'est passer d'un plan à l'autre pour élaborer des modèles aussi proches que possible de l'énoncé initial.

Ces projections ont une vertu que je n'avais pas vue d'abord : elles servent de test de complétude. Chaque vue attend des éléments — un agent, une action, un patient, un instrument, un lieu, un temps, un but, des conditions — et ce que l'énoncé ne fournit pas apparaît comme un trou. La complétude d'un énoncé se définit alors comme son adéquation à un schéma de situation minimal pour sa fonction communicative, et elle se mesure : couverture des rôles obligatoires, proportion de questions élémentaires auxquelles le modèle sait répondre — qui fait quoi, où, quand, pourquoi, avec quoi —, et parcimonie, pour ne pas récompenser un modèle qui invente. Le résultat est un certificat d'adéquation qui liste les rôles couverts, les rôles manquants et les pragmènes responsables de chaque remplissage implicite.

Les trous se comblent de deux manières. Par analogie de forme, d'abord : des fragments empruntés à des modèles similaires, ce qui rejoint l'idée, développée dans [[L'utopie de la Modélisation]], que des modèles de domaines différents sont substituables au sein d'une même classe de structure. Par transformations de modèles, ensuite : des chaînes préétablies qui mobilisent d'autres métamodèles que celui de la langue et simulent une réflexion, pour aboutir à des modèles abstraits de manipulation pure de concepts — une compression sémantique de l'énoncé de départ, cette fois au sens plein. Deux garde-fous encadrent la complétion : au plus un pragmène par rôle manquant, faute de quoi l'on écrit `UNKNOWN` — la complétude baisse mais la trace reste propre —, et un contrôle par ablation, qui ne conserve un pragmène que s'il améliore robustement la complétude.

Pour rester dans le textuel, et donc dans le versionnable et le transformable, les vues sont décrites en Mermaid ou en D2 plutôt qu'en PlantUML, qui ne dispose pas de grammaire et me paraît de ce fait mal fondé (voir [[Outils de modélisation textuels]]).

## 6. Pourquoi c'est de l'ingénierie dirigée par les modèles

Relue ainsi, la chaîne est une chaîne IDM. Les séquences de morphèmes sont des modèles conformes à un métamodèle de la langue ; les schémas de situation en sont un second ; les vues UML et BPMN en sont d'autres. Les projections d'un énoncé vers ses vues sont des transformations de modèle à modèle, la sortie Mermaid ou D2 une transformation de modèle vers texte — celles du chapitre 4 du cours d'[[UML Ecore EMF Plantuml QVT Mermaid PyEcore]]. Et le LLM n'occupe dans cette chaîne que la place qui lui revient : proposer des pragmènes, ranger des analyses concurrentes, gérer le contexte, sous le contrôle de grammaires, de transformations et d'un journal. C'est en ce sens que l'IDM vient au secours des LLM : elle ne les remplace pas, elle les encadre et rend leur contribution auditable.

La suite de la chaîne est décrite dans [[IAG à base de modèles]] : transformer les modèles obtenus en code exécutable, simuler, mesurer l'écart avec le réel, régénérer — appliquer la roue de Deming aux modèles. Les deux notes se répondent : celle-ci va de l'énoncé au modèle, l'autre du modèle à l'exécution. Et [[L'utopie de la Modélisation]] rappelle ce que le pragmène institutionnalise : un modèle n'est jamais complet, et le plus honnête est de rendre visible l'endroit où il ne l'est pas.

## 7. Ce qu'il faudrait tester

Rien de tout cela ne vaut sans expérience. Le protocole minimal se déduit des échanges eux-mêmes. Écrire le transducteur réversible et la grammaire pour un sous-ensemble de la langue, et vérifier la bijection sur un corpus existant — Tekstaro, la Wikipédia en espéranto, Tatoeba. Comparer la taille du vocabulaire symbolique à celle d'un vocabulaire BPE entraîné sur le même corpus, et celle des modèles que chacun autorise. Entraîner le petit modèle sur les séquences de morphèmes et mesurer, outre la perplexité, l'exactitude morphologique et syntaxique des sorties et leur acceptabilité par des locuteurs. Projeter des énoncés simples de type *La administranto aprobas la peton post valida kontrolo* vers des vues Mermaid et D2, calculer leur complétude, et faire juger par des humains si les pragmènes proposés sont ceux qu'ils auraient eux-mêmes reconstruits. Enfin, auditer la trace : relire le journal d'une génération et vérifier que chaque décision s'y explique.

## 8. Réponse à la question du titre

Peut-on transformer un problème de compréhension en un problème de génie logiciel ? Pour la partie formelle — morphologie, syntaxe, projection des rôles vers des vues — oui, et l'espéranto est ce qui rend ce transfert praticable, parce qu'il place la frontière du formel beaucoup plus loin qu'une langue historique. Pour la pragmatique, non : la compréhension reste un acte de reconstruction de l'implicite, et aucune grammaire ne le fait à notre place. Mais cette part irréductible devient elle-même un objet de génie logiciel : nommée (le pragmène), mesurée (la complétude), bornée (la parcimonie) et journalisée (la trace). Le problème de compréhension n'est pas résolu ; il est circonscrit, et chaque endroit où il subsiste est visible. C'est l'inverse exact d'une boîte noire, et c'est ce que l'ingénierie dirigée par les modèles peut apporter aux modèles de langage.
