# Large Language Models

Ressources : 
La recherche en France :
[IA au Loria, laboratoire du CNRS](https://ia.loria.fr/portfolio/)
[LLM(ChatGPT) - Dé-coder les grands modèles de langage - Christophe Cerisara | Codeurs en Seine](https://www.youtube.com/@Codeursenseine)

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

[Un équivalent de GitHub Copilot gratuit à installer sur Visual Studio Code | Korben](https://youtu.be/6a5GHdoa8OM) [TabbyML](https://github.com/TabbyML/tabby) Écrit en RUST, Permet d'installer un serveur conversationnel via docker. Le client pour visual studio code est disponible via les extensions en cherchant tabby. Tabby permet de gérer des équipes et possède une API

Le futur de l'IDE **Cursor** [Using Cursor - the AI powered VS Code alt for the first time…| Huw prosser](https://youtu.be/n4DRPGWTmpc)

Sur Hugging Face il y a plus de 70 000 Modèles open source [Huggingface/models ](https://huggingface.co/models)
Aujourd'hui l'apprentissage d'un LLM se fait sur 1000 000 000 000 de mots. L'architerture des transformers (2018 Google) est celle de la quasi totalité des LLM, elle vise à prédire le mot suivant.

Les LLM sont très bon pour écrire du code simple, faire du code compliqué nécessite de le guider et donc d'avoir une interaction avec.

Pour gérer la complexité il y a des tentatives d'agents de models de langage (orchestration de différents LLMs) (Langchain, Coala).

Aujourd'hui on constate les capacités des LLMs mais on commence à peine à pouvoir les expliqués (presque pas, embryons de théories).
Les observations :
- Mémorisation, compression, structuration et généralisation
- Capacités émergentes :
	- "In context learning" Capacité de généraliser (seuls les LLM font cela), mais n'émerge qu'à partir de 10e10 de paramètres.
	- Capacité de faire des additions de 3 chiffres (10e10).
	- Répondre à des questions.
	- Générer des programmes.
	- Jason Wei a dénombré 137 capacités émergentes :
		- Décomposition d'un problème en étapes.
		- "Prompt of thought" (Accéder au raisonnement du LLM)
		- "Analogical prompting"  (Faire élaborer au LLM la démarche à appliquer avant de l'appliquer)
		- Instruction procédurale
		- Anagramme
		- Arithmétique modulaire
		- Problèmes simples de math
		- Déduction logique
		- Déduction analytique
		- Théorie de l'esprit (avoir un modèle de son locuteur) ?
		- ...
Embryons de théories **Passage à l'échelle**("scaling law") :
	- Lois de Chinchilla (On mesure les capacité du modèle lorsqu'il devient de plus en plus gros Baidu 2017 puis Google 2018 : l'apprentissage suit des lois qui sont fonction du nombre de données, et aboutisse en 2022 à Chinchilla, c'est loi sont en `L(N)=A/N**𝛼` où 𝛼 est différent pour chacune des architectures avec le meilleur score de 0,54 pour les "transformers")
	- Grokking (Apprend bestialement puis après beaucoup de données généralise)
	- Double descente
	- Transition de phases

# Le deeplearning un sous ensemble de l'IA
# Les modèles de langage
- 1990-2000 Les n-grams
- 2013 word2vec (Thomas Mikolov Google)
- 2015 Les RNN
- 2017 Les transformers (Vaswani et al) ([Attention Is All You Need](https://arxiv.org/abs/1706.03762))
# L'arbre des LLMs
La grande majorité des modèles actuels dérivent de Word2Vec
https://raw.githubusercontent.com/Mooler0410/LLMsPracticalGuide/main/imgs/tree.jpg

Des LLM opensources dédiés au code CodeLama, CodeGen.
# 1. Introduction aux LLMs

# 2. Fondations Théoriques

## 2.1 Traitement du langage naturel (TLP) 
### 2.1.1 Introduction au TLP
- Définition et objectifs du TLP.
- Historique du TLP: de la règle basée à l'apprentissage automatique.

### 2.1.2 Niveaux de traitement en TLP
- Traitement lexical, syntaxique, sémantique, et pragmatique.
- Analyse morphologique et lemmatisation.
- Analyse syntaxique: arbres de dépendance et de constituance.
- Représentation sémantique et compréhension de la langue.

### 2.1.3 Les outils et ressources en TLP
- Corpus de texte, lexiques, et bases de données linguistiques.
- Logiciels et bibliothèques de TLP (NLTK, spaCy, Stanford NLP).

## 2.2 Modèles de langage statistiques vs. basés sur le deep learning

### 2.2.1 Modèles de langage statistiques
- Modèles N-grammes et leurs limites.
- Techniques de lissage et d'évaluation des modèles statistiques.

#### 2.5 Modèles de langage basés sur le deep learning
- Introduction aux réseaux de neurones en TLP.
- Architectures de modèle: RNN, LSTM, GRU.

#### 2.6 Transition vers le deep learning
- Les défis des modèles statistiques et la montée du deep learning.
- Avantages des modèles basés sur le deep learning en termes de contextualisation et performance.

### Activités en classe
- Comparaison de la performance d'un modèle N-gramme et d'un modèle LSTM sur une tâche de complétion de texte.

### Devoir
- Construire un simple modèle N-gramme et un modèle RNN pour la génération de texte et comparer les résultats.

---

## Les réseaux de neurones et l'apprentissage profond en TLP

### Objectif de la leçon
- Fournir une compréhension des réseaux de neurones et de l'apprentissage profond.
- Discuter de l'application de ces concepts en TLP.

### Contenu
#### 2.7 Fondements des réseaux de neurones
- Neurones artificiels et topologies de réseau.
- Processus d'apprentissage et rétropropagation.

#### 2.8 Réseaux de neurones récurrents (RNN)
- Particularités et utilisation des RNN en TLP.
- Problèmes des RNN et introduction des LSTM et GRU.

#### 2.9 L'essor des architectures Transformer
- Introduction aux Transformers et à l'auto-attention.
- BERT, GPT, et autres modèles basés sur Transformer.

### Activités en classe
- Implémentation et entraînement d'un modèle LSTM pour la classification des sentiments.

### Devoir
- Lecture critique d'un article de recherche sur les architectures Transformer et leur impact sur le TLP.

---

Ce plan de cours est conçu pour être progressif, en commençant par les fondamentaux du TLP et en avançant vers des concepts plus complexes comme l'apprentissage profond. Les activités pratiques et les devoirs sont destinés à renforcer la théorie et à donner aux étudiants une expérience pratique avec les techniques modernes du TLP.