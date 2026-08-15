---
schema_version: 1
uid: "01M02EX5B8FCN41RAHSC297FN9"
titre: "LLM"
aliases:
  - "Large Language Model"
  - "Grands modèles de langage"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - intelligence-artificielle
  - apprentissage-profond
  - llm
  - traitement-du-langage
resume: "Cours sur les grands modèles de langage : place dans l'apprentissage profond, arbre généalogique des LLM, origines et évolutions, capacités, applications, fondations théoriques et défis."
niveau: intermediaire
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2024-03-08
date_modification: 2024-05-06
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# Large Language Models

# Le deeplearning un sous ensemble de l'IA
# Les modèles de langage
- 1964-1966 Eliza (Joseph Weizenbaum MIT)
- SHRDLU 1968-1969 (Terry Winograd MIT)
- 1990-2000 Les n-grams
- 2013 word2vec (Thomas Mikolov Google)
- 2015 Les RNN
- 2017 Les transformers (Vaswani et al) ([Attention Is All You Need](https://arxiv.org/abs/1706.03762))
Depuis 2010 la famille des GPT (Generated Pretrained Tranformer)

| Modèle                   | Vendeur/Créateur |
| ------------------------ | ---------------- |
| GPT(GPT-3, GPT-4, GPT-5) | OpenAI           |
| BERT                     | Google           |
| LaMDA                    | Google           |
| PaLM                     | Google           |
| BLOOM                    | Hugging Face     |
| LLaMA                    | Meta             |
| Claude                   | Anthropic        |
| NeMO LLM                 | NVidia           |
| Generate                 | Cohere           |
| Mixtral                  | Mistral          |
# L'arbre des LLMs
La grande majorité des modèles actuels dérivent de Word2Vec
https://raw.githubusercontent.com/Mooler0410/LLMsPracticalGuide/main/imgs/tree.jpg

Des LLM opensources dédiés au code CodeLama, CodeGen.
# 1. Introduction aux Modèles de Langage à Grande Échelle (LLM)

Dans le domaine en rapide évolution de l'intelligence artificielle (IA), les modèles de langage à grande échelle (LLM) représentent une avancée significative, marquant une ère nouvelle dans la façon dont les machines comprennent et génèrent le langage humain. Issus d'un sous-ensemble de l'IA connu sous le nom de deep learning, les LLMs sont au cœur des innovations technologiques actuelles, transformant les interactions humain-machine, la programmation, l'éducation, et bien plus encore.

## 1.1. Origines et Évolution

Les fondements des LLMs remontent aux années 1990 et 2000 avec l'avènement des n-grams, suivis par l'introduction de Word2Vec par Thomas Mikolov chez Google en 2013. Cette évolution a continué avec les RNN en 2015 et a été révolutionnée par les transformers en 2017, un modèle introduit par Vaswani et al. dans leur papier "Attention Is All You Need". Ces transformers sont la base de la quasi-totalité des LLM actuels, permettant de prédire le mot suivant dans une séquence avec une précision sans précédent.

## 1.2. Capacités et Applications

Les LLMs, avec leur architecture transformer, ont démontré une gamme étonnante de capacités allant de la simple génération de texte à la résolution de problèmes complexes de programmation. Des plateformes comme Hugging Face hébergent aujourd'hui plus de 70 000 modèles open source, témoignant de l'énorme intérêt et de la variabilité des applications possibles. Ces modèles ont trouvé leur place non seulement dans la rédaction de code mais également dans l'éducation, où élèves et enseignants utilisent des outils comme ChatGPT pour les devoirs et la correction, illustrant leur potentiel disruptif et polyvalent.

## 1.3. Défis et Perspectives

Toutefois, les LLMs ne sont pas sans défis. La question de la sécurité, avec la possibilité que les modèles contiennent des backdoors, ainsi que les préoccupations éthiques autour de leur utilisation, nécessitent une attention soutenue. Par ailleurs, les efforts pour comprendre et expliquer les mécanismes sous-jacents des LLMs ne font que commencer, avec des embryons de théories comme les lois de Chinchilla et le phénomène de Grokking qui tentent de cartographier le comportement de ces modèles à mesure qu'ils évoluent.

## Perspectives

L'avenir des LLMs s'annonce riche en innovations et en défis. Alors que nous commençons tout juste à gratter la surface de leurs capacités, leur intégration croissante dans notre quotidien et leur impact potentiel sur divers secteurs promettent de redéfinir notre relation avec la technologie.
Cette technologie, comme tous les outils, apporte avec elle son lot de risques, tant pour la sécurité des données et la véracité des informations, que pour la robustesse des infrastructures, sans oublier l'impact de sa consommation énergétique. Elle nécessite des produits complexes qui demandent des ressources rares, dont l'extraction est polluante. Ainsi, elle porte en elle à la fois des promesses et des menaces. Il nous appartient de l'utiliser pour le meilleur.

# 2. Fondations Théoriques

## 2.1 Traitement du langage naturel (TLP) 
### 2.1.1 Introduction au TLP

Le Traitement du Langage Naturel (TLP), ou Natural Language Processing (NLP) en anglais, se définit comme la branche de l'intelligence artificielle qui se concentre sur l'interaction entre les ordinateurs et le langage humain. Son objectif principal est de permettre aux machines de comprendre, interpréter, et produire du langage humain d'une manière qui soit à la fois significative et utile. Ceci inclut une gamme de tâches telles que la traduction automatique, la reconnaissance vocale, et la génération de texte.

Historiquement, le TLP a commencé avec des approches basées sur des règles, où les linguistes formulaient des règles grammaticales et syntaxiques pour analyser et comprendre le langage. Cette méthode nécessitait une connaissance approfondie de la langue concernée et était extrêmement laborieuse à mettre en œuvre. En outre, elle manquait de flexibilité et ne pouvait pas gérer efficacement les nuances et la variabilité inhérentes au langage humain.

Avec l'avènement de l'informatique et la disponibilité croissante de grandes quantités de données textuelles, le champ du TLP a progressivement évolué vers des méthodes d'apprentissage automatique. Ces approches, contrairement aux systèmes basés sur des règles, apprennent à partir de vastes ensembles de données, permettant aux modèles de s'adapter et de reconnaître des schémas linguistiques complexes sans une programmation explicite pour chaque cas. Cette transition a marqué un tournant dans le domaine du TLP, ouvrant la voie à des progrès significatifs et à la création de systèmes beaucoup plus performants et flexibles.

### 2.1.2 Niveaux de traitement en TLP

Le traitement du langage naturel (TLP) opère à plusieurs niveaux pour analyser et comprendre le langage humain, chacun ajoutant une couche de complexité et de nuance à l'interprétation du texte. Voici un aperçu des principaux niveaux de traitement en TLP :

- **Traitement lexical** : Ce niveau concerne l'analyse des mots individuels hors contexte. Il englobe l'identification des types de mots (noms, verbes, adjectifs, etc.), ainsi que la reconnaissance des entités nommées (noms de personnes, lieux, organisations). Le traitement lexical inclut également l'identification de la racine des mots pour faciliter leur analyse.

- **Analyse morphologique et lemmatisation** : L'analyse morphologique s'intéresse à la structure interne des mots, identifiant les racines, les préfixes, les suffixes, et la formation des mots. La lemmatisation, quant à elle, consiste à ramener un mot à sa forme de base ou de dictionnaire, ce qui est crucial pour réduire la complexité et augmenter la généralité de l'analyse.

- **Analyse syntaxique** : Ce niveau se concentre sur la structure grammaticale des phrases, déterminant la manière dont les mots sont organisés pour former des phrases cohérentes. L'analyse syntaxique peut être réalisée à travers les arbres de dépendance, qui montrent les relations entre les mots d'une phrase, ou les arbres de constituance, qui décomposent la phrase en sous-unités (phrases, groupes nominaux, etc.).

- **Traitement sémantique** : Ici, le but est de comprendre la signification des mots dans leur contexte spécifique. La représentation sémantique implique l'attribution de sens aux phrases et aux textes, allant au-delà de la structure grammaticale pour saisir les nuances de signification, les ambiguïtés, et les relations sémantiques entre les entités et les concepts.

- **Traitement pragmatique** : Le niveau pragmatique s'attache à l'usage et au contexte du langage, considérant comment le sens est construit dans des situations de communication réelles. Cela inclut l'analyse du discours, la reconnaissance de l'ironie ou du sarcasme, et la compréhension de l'intention communicative derrière les énoncés.

Chaque niveau de traitement contribue à une compréhension plus profonde et plus nuancée du texte, permettant aux systèmes de TLP de répondre et d'interagir de manière plus naturelle et intelligente.

### 2.1.3 Les outils et ressources en TLP

Pour mener à bien les recherches et les projets en Traitement du Langage Naturel (TLP), divers outils et ressources sont indispensables. Ces ressources permettent d'analyser, de comprendre, et de générer le langage humain de manière efficace. Voici une vue d'ensemble des principaux outils et ressources disponibles dans le domaine :

- **Corpus de texte** : Les corpus de texte sont des ensembles de documents écrits qui servent de matériel de base pour l'entraînement et le test des modèles de TLP. Ils peuvent être généralistes, couvrant un large éventail de sujets, ou spécialisés dans des domaines spécifiques (médical, juridique, littéraire, etc.). Les corpus sont essentiels pour développer des modèles de langage et pour l'évaluation de leur performance.

- **Lexiques et bases de données linguistiques** : Les lexiques sont des collections de mots ou de phrases avec des informations associées (définitions, synonymes, antonymes, etc.). Les bases de données linguistiques, telles que WordNet, fournissent des structures relationnelles entre les mots, facilitant l'analyse sémantique et le traitement du sens. Ces ressources sont cruciales pour la lemmatisation, la reconnaissance d'entités nommées, et d'autres tâches de TLP.

- **Logiciels et bibliothèques de TLP** :
  - **NLTK (Natural Language Toolkit)** : Une des bibliothèques les plus utilisées pour le travail en TLP. Elle offre des outils pour le traitement de texte, l'analyse syntaxique, la reconnaissance d'entités nommées, et plus encore, rendant le TLP accessible aux débutants comme aux chercheurs expérimentés.
  - **spaCy** : Une bibliothèque moderne et rapide pour le TLP en Python, conçue pour la production. Elle est particulièrement reconnue pour son efficacité dans le traitement et l'analyse de grands volumes de texte.
  - **Stanford NLP** : Un ensemble d'outils développé par le Stanford Natural Language Processing Group. Il inclut une variété de modèles de TLP et d'algorithmes d'analyse linguistique pour le traitement syntaxique, la reconnaissance d'entités nommées, et l'analyse sémantique.

Ces outils et ressources jouent un rôle fondamental dans le développement et l'implémentation de solutions de TLP. Ils permettent aux chercheurs et aux développeurs de construire des applications sophistiquées de traitement du langage, allant de la traduction automatique à l'analyse de sentiments, en passant par les assistants vocaux intelligents.

## 2.2 Modèles de langage statistiques vs. basés sur le deep learning

### 2.2.1 Modèles de langage statistiques

Les modèles de langage statistiques ont joué un rôle fondamental dans les premiers développements du traitement du langage naturel (TLP), en permettant aux ordinateurs de traiter et de comprendre le langage humain à partir de l'analyse statistique de grandes quantités de données textuelles.

- **Modèles N-grammes** : Au cœur de ces modèles statistiques se trouvent les modèles N-grammes, qui prédisent le mot suivant dans une séquence en se basant sur les \(N-1\) mots précédents. Par exemple, dans un modèle trigramme (\(N=3\)), la probabilité du mot suivant est estimée en fonction des deux mots qui le précèdent. Ces modèles sont simples mais étonnamment efficaces pour de nombreuses tâches de TLP. Cependant, ils ont des limites significatives, notamment leur incapacité à capturer des dépendances à long terme dans le texte et leur tendance à gonfler l'espace de stockage nécessaire, car la taille du modèle croît exponentiellement avec la valeur de \(N\).

- **Limites des modèles N-grammes** : Outre les défis liés à l'espace de stockage et aux dépendances à long terme, les modèles N-grammes luttent également contre le problème des mots hors vocabulaire (OOV) et ne peuvent pas gérer efficacement la polysémie (mots ayant plusieurs significations) ou la synonymie (mots différents ayant des significations similaires).

- **Techniques de lissage** : Pour atténuer certains de ces problèmes, les techniques de lissage ont été développées. Ces méthodes ajustent les probabilités des séquences de mots pour éviter les probabilités nulles pour les séquences non observées dans le corpus d'entraînement. Les techniques courantes incluent le lissage de Laplace et le lissage de Good-Turing, qui réallouent certaines des probabilités des événements observés vers les événements non observés.

- **Évaluation des modèles statistiques** : L'évaluation de ces modèles se fait typiquement à l'aide de mesures telles que la perplexité, qui mesure à quel point un modèle est "surpris" par de nouvelles données. Une perplexité plus faible indique un modèle qui prédit mieux les séquences de mots. Cependant, malgré leur utilité, les limites intrinsèques des modèles N-grammes et les défis liés à leur évaluation ont mené à la recherche de nouvelles approches, notamment celle des modèles basés sur le deep learning, qui offrent une capacité supérieure à modéliser des dépendances complexes et à long terme dans le langage.

### 2.2.2 Modèles de langage basés sur le deep learning

Avec l'avancement de la technologie et la quête pour surmonter les limites des modèles statistiques, le domaine du traitement du langage naturel (TLP) a assisté à une révolution majeure grâce à l'adoption du deep learning. Cette transition a ouvert la voie à des modèles de langage beaucoup plus puissants et flexibles, capables de comprendre et de générer du langage avec une précision sans précédent.

- **Introduction aux réseaux de neurones en TLP** : Les réseaux de neurones artificiels s'inspirent du fonctionnement du cerveau humain et sont composés de couches de neurones artificiels. En TLP, ils permettent de modéliser des séquences de mots et de capturer les relations sémantiques complexes entre elles. Contrairement aux modèles statistiques, les réseaux de neurones sont capables de gérer des dépendances à long terme et d'apprendre des représentations riches et hiérarchisées du langage.

- **Architectures de modèle** :
  - **RNN (Réseaux de Neurones Récurrents)** : Les RNN sont spécialement conçus pour traiter des séquences de données, comme le texte, en traitant chaque élément de la séquence un par un tout en maintenant une mémoire (état caché) de ce qui a été traité jusqu'à présent. Cela leur permet de capturer des informations contextuelles dans une séquence. Cependant, les RNN standards luttent contre le problème de la disparition ou de l'explosion des gradients, rendant difficile l'apprentissage de dépendances à long terme.
  
  - **LSTM (Long Short-Term Memory)** : Les LSTM sont une variante des RNN qui introduisent des portes (input, forget, et output gates) pour réguler le flux d'informations. Elles permettent au modèle de retenir ou d'oublier des informations de manière sélective, facilitant ainsi l'apprentissage de dépendances à long terme sans succomber aux problèmes des gradients disparus ou explosifs.
  
  - **GRU (Gated Recurrent Unit)** : Les GRU sont une autre variante des RNN qui simplifient l'architecture LSTM tout en conservant sa capacité à apprendre des dépendances à long terme. Avec seulement deux portes (reset et update gates), les GRU offrent souvent une efficacité comparable aux LSTM mais avec une complexité moindre, ce qui peut conduire à une réduction du temps d'entraînement et des besoins en ressources.

Ces architectures de deep learning ont significativement amélioré la performance des modèles de langage, permettant des avancées majeures dans des applications de TLP telles que la traduction automatique, la génération de texte, et la compréhension de la langue naturelle. Leur capacité à apprendre des représentations profondes du langage a marqué une étape importante dans l'évolution des technologies de traitement du langage.

### 2.2.3 Transition vers le deep learning

La transition des modèles de langage statistiques vers ceux basés sur le deep learning marque une évolution cruciale dans le domaine du traitement du langage naturel (TLP). Cette mutation s'est opérée en réponse aux défis inhérents aux approches statistiques et a été facilitée par les avancées technologiques et l'augmentation de la puissance de calcul disponible.

- **Les défis des modèles statistiques** : Malgré leur utilité dans les premiers stades du développement du TLP, les modèles statistiques, notamment les modèles N-grammes, présentent plusieurs limitations. Leur capacité à capturer des relations complexes et à long terme entre les éléments du langage est restreinte, ce qui les rend moins efficaces pour comprendre le contexte ou la nuance des séquences de mots. De plus, ils sont souvent incapables de gérer de manière efficace les mots rares ou inconnus, limitant ainsi leur applicabilité à de nouveaux domaines ou vocabulaires.

- **La montée du deep learning** : Face à ces défis, le deep learning a émergé comme une solution puissante, offrant une approche fondamentalement différente pour modéliser le langage. Les architectures de réseaux de neurones, telles que les RNN, LSTM, et GRU, ont permis de surmonter bon nombre des limitations des modèles statistiques en apprenant des représentations profondes et hiérarchiques du langage à partir de données. Cette capacité à apprendre de manière endogène à partir de grandes quantités de texte a révolutionné le TLP.

- **Avantages des modèles basés sur le deep learning** :
  - **Contextualisation** : Les modèles de deep learning excellent dans la compréhension du contexte, capable de saisir des nuances sémantiques fines et des dépendances à long terme dans le texte. Cette profondeur de compréhension permet des applications plus sophistiquées et naturelles du TLP, telles que la compréhension du langage naturel (NLU) et la réponse aux questions (QA).
  
  - **Performance** : En termes de performance brute, les modèles de deep learning surpassent largement les approches statistiques traditionnelles sur une multitude de tâches de TLP. Que ce soit dans la traduction automatique, la classification de texte, ou la génération de langage, les modèles basés sur le deep learning établissent régulièrement de nouveaux standards de performance.

La transition vers le deep learning a non seulement adressé les limitations des modèles statistiques mais a également ouvert de nouvelles voies d'exploration et d'innovation dans le TLP. Cette avancée a conduit à une amélioration significative des capacités des systèmes de TLP, rendant possible des interactions homme-machine plus naturelles et intelligentes.

### 2.3. Activités en classe
- Comparaison de la performance d'un modèle N-gramme et d'un modèle LSTM sur une tâche de complétion de texte.

### 2.4. Devoir
- Construire un simple modèle N-gramme et un modèle RNN pour la génération de texte et comparer les résultats.
# 3. Les réseaux de neurones et l'apprentissage profond en TLP

## Objectif
- Fournir une compréhension des réseaux de neurones et de l'apprentissage profond.
- Discuter de l'application de ces concepts en TLP.

## 3.1. Fondements des réseaux de neurones

La compréhension des réseaux de neurones et de l'apprentissage profond constitue une pierre angulaire du traitement du langage naturel (TLP) moderne. Ces concepts empruntent à la neurologie et à l'informatique pour créer des modèles capables de traiter le langage humain de manière inédite.

- **Neurones artificiels et topologies de réseau** : Les neurones artificiels sont les unités de base des réseaux de neurones, inspirés par les neurones biologiques du cerveau humain. Chaque neurone reçoit des entrées, les traite à l'aide d'une fonction d'activation (comme la sigmoïde, ReLU), et transmet le résultat aux neurones suivants. Les réseaux de neurones sont organisés en couches, comprenant généralement une couche d'entrée, une ou plusieurs couches cachées, et une couche de sortie. La topologie d'un réseau, c'est-à-dire l'arrangement de ces neurones et couches, influe directement sur sa capacité à apprendre et à généraliser à partir des données.

- **Processus d'apprentissage et rétropropagation** : L'apprentissage dans les réseaux de neurones se fait à travers un processus appelé rétropropagation (backpropagation). Durant cet apprentissage, le réseau ajuste les poids de ses connexions pour minimiser la différence entre la sortie prédite et la sortie attendue (erreur). La rétropropagation utilise le gradient de l'erreur par rapport à chaque poids, calculé via le calcul différentiel, pour mettre à jour les poids dans la direction qui réduit l'erreur. Ce processus est répété à travers de multiples itérations ou époches d'entraînement, permettant au réseau de devenir progressivement plus précis dans ses prédictions.

Ce cadre fondamental des réseaux de neurones offre une flexibilité et une puissance de calcul qui ont révolutionné le TLP, permettant des avancées significatives dans la compréhension et la génération du langage naturel.

## 3.2. Réseaux de neurones récurrents (RNN)

Les réseaux de neurones récurrents (RNN) constituent une classe d'architectures de réseau de neurones spécialement conçue pour traiter des séquences de données, ce qui les rend particulièrement adaptés aux applications de traitement du langage naturel (TLP). 

### Particularités et utilisation des RNN en TLP

- **Particularités des RNN** : La caractéristique distinctive des RNN est leur capacité à maintenir un état (ou mémoire) qui capture des informations sur ce qui a été traité jusqu'à présent dans la séquence. Cela leur permet de traiter des séquences de données de longueur variable, en prenant en compte non seulement l'entrée actuelle mais aussi le contexte fourni par les entrées précédentes. Cette propriété est cruciale pour le TLP, où la compréhension du contexte et la cohérence sur de longs segments de texte sont essentielles.

- **Utilisation des RNN en TLP** : En TLP, les RNN sont utilisés pour une variété de tâches telles que la traduction automatique, la génération de texte, la reconnaissance vocale, et l'analyse des sentiments. Leur capacité à gérer le contexte les rend particulièrement efficaces pour ces applications, où la signification dépend fortement de la séquence des mots.

### Problèmes des RNN et introduction des LSTM et GRU

- **Problèmes des RNN** : Malgré leur efficacité, les RNN standards font face à des défis significatifs, notamment le problème de la disparition ou de l'explosion des gradients lors de l'apprentissage sur de longues séquences. Ces problèmes rendent difficile pour les RNN d'apprendre des dépendances à long terme dans les données, limitant leur capacité à traiter efficacement des séquences de texte longues ou complexes.

- **Introduction des LSTM et GRU** :
  - **LSTM (Long Short-Term Memory)** : Les LSTM sont une amélioration des RNN conçue pour pallier le problème de la disparition des gradients. Ils introduisent une structure de cellule complexe avec des portes spécifiques (portes d'entrée, de sortie et d'oubli) qui régulent le flux d'informations, permettant au réseau de retenir et d'oublier des informations de manière sélective. Cela les rend capables de capturer des dépendances à long terme sans souffrir de la disparition des gradients.
  
  - **GRU (Gated Recurrent Unit)** : Les GRU simplifient l'architecture des LSTM tout en conservant une grande partie de leur capacité à gérer des dépendances à long terme. Avec seulement deux portes (portes de réinitialisation et de mise à jour), les GRU offrent une alternative plus simple et souvent plus efficace aux LSTM pour certaines tâches.

Ces évolutions des RNN, les LSTM et GRU, ont largement contribué à surmonter les limitations initiales des RNN, permettant des avancées significatives dans le traitement des séquences longues et complexes en TLP.

## 3.3. L'essor des architectures Transformer

L'arrivée des architectures Transformer a marqué un tournant dans le domaine de l'intelligence artificielle, en particulier dans le traitement du langage naturel (TLP). Ces modèles se distinguent par leur capacité exceptionnelle à traiter de manière efficace des séquences de données de grande taille, surpassant les architectures précédentes en termes de performance et de flexibilité.

- **Introduction aux Transformers et à l'auto-attention** : Les Transformers reposent sur le mécanisme d'auto-attention, qui permet au modèle d'évaluer différentes parties d'une séquence d'entrée en fonction de leur pertinence pour chaque élément de la séquence. Cette approche diffère radicalement des modèles RNN et LSTM en ce qu'elle ne traite pas les séquences de manière séquentielle, mais évalue l'ensemble de la séquence simultanément. Cela permet une parallélisation beaucoup plus efficace lors de l'entraînement et améliore la capacité du modèle à gérer des dépendances à longue distance dans le texte.

- **BERT, GPT, et autres modèles basés sur Transformer** : 
  - **BERT (Bidirectional Encoder Representations from Transformers)** : Développé par Google, BERT a innové en appliquant le concept de lecture bidirectionnelle du texte, permettant au modèle de capturer le contexte à la fois à gauche et à droite de chaque token dans une séquence. Cela a mené à des améliorations substantielles dans des tâches telles que la compréhension de texte et la réponse aux questions.
  
  - **GPT (Generative Pre-trained Transformer)** : Lancé par OpenAI, GPT se distingue par sa capacité à générer du texte de manière cohérente et contextuellement pertinente. En utilisant une approche de pré-entraînement suivi d'un affinage spécifique à la tâche, GPT a établi de nouveaux standards dans la génération de texte, la traduction automatique, et au-delà.

Les architectures Transformer ont transformé le paysage du TLP, ouvrant la voie à des avancées significatives dans la compréhension et la génération de langage naturel. Leur flexibilité, combinée à leur performance exceptionnelle, continue de stimuler l'innovation, faisant des Transformers l'une des technologies les plus influentes dans le domaine de l'intelligence artificielle aujourd'hui.

## Activités en classe
- Implémentation et entraînement d'un modèle LSTM pour la classification des sentiments.

## Devoir
- Lecture critique d'un article de recherche sur les architectures Transformer et leur impact sur le TLP.

--------------
# Glossaire
Comme toute nouvelle technologie l'IA vient avec sont lot d’acronymes dont voici les principaux.

- **Auto-attention** : Un mécanisme au cœur des architectures Transformer qui permet à chaque position dans une séquence de tenir compte de toutes les positions dans la même séquence pour encoder une représentation de cette séquence. L'auto-attention aide le modèle à se concentrer sur les parties importantes de l'entrée pour effectuer une tâche spécifique.

- **BERT (Bidirectional Encoder Representations from Transformers)** : Un modèle pré-entraîné sur un grand corpus de texte qui utilise l'architecture Transformer pour générer des représentations contextuelles des mots. BERT a été conçu pour pré-entraîner des représentations profondes bidirectionnelles en examinant le contexte des mots à gauche et à droite dans toutes les couches. En conséquence, le modèle pré-entraîné peut être finement ajusté avec juste une couche de sortie supplémentaire pour créer des modèles de pointe pour une large gamme de tâches de TLP.

- **GPT (Generative Pre-trained Transformer)** : Une série de modèles de langage qui utilisent l'architecture Transformer pour produire du texte. GPT est pré-entraîné sur un grand corpus de texte et peut être utilisé pour générer des textes cohérents et pertinents sur divers sujets en complétant une invite donnée.

- **IA** : Intelligence Artificielle - Le domaine de la science informatique qui se concentre sur la création de systèmes capables de réaliser des tâches qui nécessiteraient normalement l'intelligence humaine, telles que la prise de décision, la reconnaissance de la parole, la traduction entre langues, et plus encore.

- **GRU (Gated Recurrent Unit)** : Similaire aux LSTM, mais avec une structure plus simple, ce qui peut réduire la complexité et le temps de calcul. Les GRU utilisent également des portes pour contrôler le flux d'informations, mais elles combinent la porte d'oubli et la porte d'entrée en une seule.

- **LLM** : Modèles de Langage à Grande Échelle - Des systèmes d'intelligence artificielle conçus pour comprendre, générer, et interagir en langage naturel à une échelle massive. Ils sont entraînés sur d'immenses corpus de texte pour apprendre une grande variété de tâches linguistiques.

- **LSTM (Long Short-Term Memory)** : Une amélioration des RNN traditionnels, conçue pour éviter le problème de disparition du gradient, en introduisant des structures appelées cellules qui permettent au réseau de retenir des informations sur une longue période.

- **n-gram** : Un n-gram est une séquence continue de n éléments (mots, lettres) d'un texte donné ou d'une parole. Les modèles basés sur n-grams ont été parmi les premières approches utilisées pour le traitement du langage naturel.

- **RAG** : ou Retrieval-Augmented Generation, est une technique dans le domaine de l'intelligence artificielle, particulièrement dans le traitement automatique du langage naturel (TALN), qui combine les méthodes de récupération d'informations (retrieval) avec des modèles de génération de texte pour améliorer la qualité et la pertinence des réponses fournies par les systèmes de questions-réponses ou les chatbots. L'idée derrière RAG est relativement simple mais puissante. Au lieu de générer une réponse à une question directement à partir d'un modèle de langage pré-entraîné, RAG commence par chercher dans un grand corpus de textes (comme Wikipedia ou un ensemble de documents spécifiques) pour trouver les passages les plus pertinents liés à la question posée. Ces passages sont ensuite utilisés comme contexte supplémentaire par le modèle de génération pour produire une réponse plus précise, détaillée et informée.

- **RNN (Réseaux de Neurones Récurrents)** : Une classe de réseaux de neurones conçue pour traiter des séquences de données, telle que le texte ou les séries temporelles. Les RNN utilisent leur état interne (mémoire) pour traiter les séquences de données, ce qui les rend idéaux pour des tâches de TLP telles que la prédiction de mots suivants ou la compréhension du langage.

- **Transformers** : Une architecture de modèle introduite dans le papier "Attention Is All You Need" pour le traitement du langage naturel et d'autres tâches de séquence. Elle utilise le mécanisme d'auto-attention pour capter les dépendances à longue distance entre les mots dans une phrase, améliorant significativement la performance par rapport aux RNN et LSTM sur de nombreuses tâches de TLP.

- **Word2Vec** : Un groupe de modèles liés qui sont utilisés pour produire des plongements de mots (word embeddings). Ces modèles sont capables de capturer le contexte d'un mot dans un document, sa signification sémantique et syntaxique, simplement à partir du texte brut.

# Outils
Pour compter les tokens d'un prompt [tokenizer](https://platform.openai.com/tokenizer)
# Ressources
La recherche en France : [IA au Loria, laboratoire du CNRS](https://ia.loria.fr/portfolio/)
[LLM(ChatGPT) - Dé-coder les grands modèles de langage - Christophe Cerisara | Codeurs en Seine](https://youtu.be/GiEcNK3XA_o)

[Aperçu complet sur les LLM et l'ingénierie des prompts | Baamtu](https://youtu.be/MWotDXpD6SI?si=txySpWEWK4vQ2A6V)

[The Practical Guides for Large Language Models | LLMsPracticalGuide](https://github.com/Mooler0410/LLMsPracticalGuide)

https://machinelearningmastery.com

[Les élèves utilisent ChatGPT pour leurs devoirs et les enseignants utilisent ChatGPT pour les corriger, d'après des rapports | Developpez.com](https://intelligence-artificielle.developpez.com/actu/355048/Les-eleves-utilisent-ChatGPT-pour-leurs-devoirs-et-les-enseignants-utilisent-ChatGPT-pour-les-corriger-d-apres-des-rapports-qui-suscitent-des-comparaisons-avec-les-examens-ecrits-et-oraux/)  
  
[Comment utiliser un LLM open source ? | Octo.com](https://blog.octo.com/comment-utiliser-un-llm-open-source-1)  
  
[Les modèles de langage peuvent contenir des backdoors | Nextinpact](https://next.ink/123823/les-modeles-de-langage-peuvent-contenir-des-backdoors/)

[Tuner un LLM à partir de la documentation d'un projet](https://youtu.be/Ivp5PGIbGMw)

[n8n - Workflow Automation](https://github.com/n8n-io)

[Unslow AI training & finetuning Get 30x faster with unsloth](https://unsloth.ai/)

[Votre LLM (ChatGPT-like) à la maison et comment coder par dessus. | Korben](https://youtu.be/1aXPuFrPtr0)

Comprendre le prompting
[# Prompt Engineering Tutorial – Master ChatGPT and LLM Responses](https://youtu.be/_ZvnD73m40o)

Vlog sur le développement avec l'IA
[Don’t Build AI Products The Way Everyone Else Is Doing It](https://www.youtube.com/watch?v=bRFLE9qi3t8)

Groq les puces dédiées à l'IA
LeMondeInformatique
[Groq défie Nvidia avec ses accélérateurs LPU - Le Monde Informatique](https://www.lemondeinformatique.fr/actualites/lire-groq-defie-nvidia-avec-ses-accelerateurs-lpu-92822.html)
[NVidia vient de se faire détronné | Underscore](https://youtu.be/fvWZ2kjTo-Q)

Encore inconnu il y a un an, Groq est bien décidé à surfer sur la vague IA (générative ou autre) avec sa plateforme de calcul LPU taillée pour l'inférence (répondre au prompt).

Réponse de NVIDIA est de se positionner très fortement sur l'apprentissage et le tuning [ Nvidia vient juste de révolutionner l'I.A ? | cocadmin](https://youtu.be/JLYUlRxp_Z8)

[Un équivalent de GitHub Copilot gratuit à installer sur Visual Studio Code | Korben](https://youtu.be/6a5GHdoa8OM) [TabbyML](https://github.com/TabbyML/tabby) Écrit en RUST, Permet d'installer un serveur conversationnel via docker. Le client pour visual studio code est disponible via les extensions en cherchant tabby. Tabby permet de gérer des équipes et possède une API

Le futur de l'IDE **Cursor** [Using Cursor - the AI powered VS Code alt for the first time…| Huw prosser](https://youtu.be/n4DRPGWTmpc)

Agrégation de textes du domaine public pour l'entrainement [common corpus des textes du domaine public pour entrainer des IA generatives](https://next.ink/131929/common-corpus-des-textes-du-domaine-public-pour-entrainer-des-ia-generatives/)

(x**2 for x in range(5))# Les LLM en bref

[résumé de la conférence : LLM(ChatGPT) - Dé-coder les grands modèles de langage - Christophe Cerisara | Codeurs en Seine](https://youtu.be/GiEcNK3XA_o)

# Les LLM en bref

Sur Hugging Face, il y a plus de 1 000 000 modèles open source au 1er mai 2024 [Huggingface/models](https://huggingface.co/models).
Aujourd'hui, l'apprentissage d'un LLM se fait sur 1 000 000 000 000 de mots. L'architecture des transformers (2018 Google) est celle de la quasi-totalité des LLM ; elle vise à prédire le mot suivant.

Les LLM sont très bons pour écrire du code simple ; faire du code compliqué nécessite de le guider et donc d'avoir une interaction avec.

Pour gérer la complexité, il y a des tentatives d'agents de modèles de langage (orchestration de différents LLMs) (Langchain, Coala).

Aujourd'hui, on constate les capacités des LLMs mais on commence à peine à pouvoir les expliquer (presque pas, embryons de théories).
Les observations :
- Mémorisation, compression, structuration, et généralisation.
- Capacités émergentes :
	- "In-context learning" : Capacité de généraliser (seuls les LLM font cela), mais n'émerge qu'à partir de 10^10 de paramètres.
	- Capacité de faire des additions de 3 chiffres (10^10).
	- Répondre à des questions.
	- Générer des programmes.
	- Jason Wei a dénombré 137 capacités émergentes :
		- Décomposition d'un problème en étapes.
		- "Prompt of thought" (Accéder au raisonnement du LLM).
		- "Analogical prompting"  (Faire élaborer au LLM la démarche à appliquer avant de l'appliquer).
		- Instruction procédurale.
		- Anagramme.
		- Arithmétique modulaire.
		- Problèmes simples de mathématiques.
		- Déduction logique.
		- Déduction analytique.
		- Théorie de l'esprit (avoir un modèle de son locuteur) ?
		- ...
Embryons de théories **Passage à l'échelle** ("scaling law") :
	- Lois de Chinchilla (On mesure les capacités du modèle lorsqu'il devient de plus en plus gros. Baidu 2017 puis Google 2018 : l'apprentissage suit des lois qui sont fonction du nombre de données, et aboutit en 2022 à Chinchilla ; ces lois sont en `L(N)=A/N**𝛼` où 𝛼 est différent pour chacune des architectures, avec le meilleur score de 0,54 pour les "transformers").
	- Grokking (Apprend bestialement puis, après beaucoup de données, généralise).
	- Double descente.
	- Transition de phases.
