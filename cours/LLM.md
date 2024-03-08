--Large Language Models

Ressources : 
[Aperçu complet sur les LLM et l'ingénierie des prompts | Baamtu](https://youtu.be/MWotDXpD6SI?si=txySpWEWK4vQ2A6V)
[LLMsPracticalGuide](https://github.com/Mooler0410/LLMsPracticalGuide)
https://machinelearningmastery.com
# Le deeplearning un sous ensemble de l'IA
# Les modèles de langage
- 1990-2000 Les n-grams
- 2013 word2vec (Thomas Mikolov Google)
- 2015 Les RNN
- 2017 Les transformers (Vaswani et al) ([Attention Is All You Need](https://arxiv.org/abs/1706.03762))
# L'arbre des LLMs
La grande majorité des modèles actuels dérivent de Word2Vec
https://raw.githubusercontent.com/Mooler0410/LLMsPracticalGuide/main/imgs/tree.jpg

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