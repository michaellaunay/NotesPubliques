Ceci est un **cours de niveau Master** sur les **Transformers**, pensé comme une progression pédagogique complète.

# Cours Master — Comprendre les Transformers

## Objectif général du cours

Nous allons comprendre pourquoi les Transformers ont profondément changé l’apprentissage profond moderne, en particulier en traitement automatique du langage, en vision, en audio, en génération de code et dans les grands modèles de langage.

Nous partirons des limites des réseaux récurrents, puis nous construirons progressivement l’architecture Transformer, jusqu’à comprendre en détail le papier fondateur **“Attention Is All You Need”** publié par Vaswani et al. en 2017.

Nous utiliserons régulièrement des **diagrammes Mermaid** pour représenter les architectures, les flux de données, les mécanismes d’attention et les variantes modernes.

---

# Chapitre 1 — Pourquoi les Transformers ?

## 1.1. Objectif du chapitre

Dans ce premier chapitre, nous allons comprendre **pourquoi les Transformers sont apparus** et **quel problème ils ont résolu**.

Nous ne commencerons pas directement par les équations. Nous allons d’abord replacer les Transformers dans l’histoire des modèles de traitement de séquences.

Une **séquence**, en informatique et en apprentissage automatique, est une donnée composée d’éléments ordonnés :

- une phrase ;
    
- un paragraphe ;
    
- une suite de mots ;
    
- une suite de caractères ;
    
- une série temporelle ;
    
- une séquence audio ;
    
- une vidéo découpée en images ;
    
- une suite d’instructions de code.
    

Le problème général est donc le suivant :

> Comment construire un modèle capable de comprendre une suite d’éléments dont le sens dépend à la fois du contenu et de l’ordre ?

Par exemple, les phrases suivantes contiennent presque les mêmes mots, mais n’ont pas le même sens :

```txt
Le chien mord l’homme.
L’homme mord le chien.
```

Le modèle doit donc comprendre :

- les mots ;
    
- leur ordre ;
    
- leurs relations ;
    
- leur contexte ;
    
- les dépendances longues ;
    
- les ambiguïtés.
    

C’est précisément ce type de problème qui a conduit à l’apparition des architectures de type Transformer.

---

## 1.2. Avant les Transformers : le traitement séquentiel

Avant les Transformers, les modèles dominants pour traiter les séquences étaient les **réseaux de neurones récurrents**, appelés **[RNN][https://fr.wikipedia.org/wiki/R%C3%A9seau_de_neurones_r%C3%A9currents)**.

Un RNN lit une séquence élément par élément. À chaque étape, il reçoit un nouvel élément et met à jour un état interne.

```mermaid
flowchart LR
    X1["Token 1"] --> R1["RNN"]
    R1 --> R2["RNN"]
    X2["Token 2"] --> R2
    R2 --> R3["RNN"]
    X3["Token 3"] --> R3
    R3 --> R4["RNN"]
    X4["Token 4"] --> R4
    R4 --> H["Représentation finale"]
```

Nous pouvons résumer l’idée ainsi :

> Le RNN lit la phrase de gauche à droite et transporte une mémoire interne au fur et à mesure.

Par exemple, pour la phrase :

```txt
Le chat dort sur le canapé.
```

Le modèle lit :

```txt
Le → chat → dort → sur → le → canapé
```

À chaque mot, il met à jour son état.

---

## 1.3. Le principe des RNN

Un RNN reçoit deux informations à chaque étape :

1. l’entrée actuelle ;
    
2. l’état précédent.
    

Il produit ensuite :

1. un nouvel état ;
    
2. éventuellement une sortie.
    

```mermaid
flowchart TD
    A["Entrée actuelle x_t"] --> C["Cellule RNN"]
    B["État précédent h_(t-1)"] --> C
    C --> D["Nouvel état h_t"]
    C --> E["Sortie éventuelle y_t"]
```

Mathématiquement, on peut écrire :

[  
h_t = f(x_t, h_{t-1})  
]

où :

- $x_t$ est l’entrée au temps $t$ ;
    
- $h_{t-1}$ est l’état précédent ;
    
- $h_t$ est le nouvel état ;
    
-$f$est une fonction apprise par le réseau.
    

L’idée est élégante : le modèle possède une forme de mémoire.

Mais cette mémoire est limitée.

---

## 1.4. Le problème des dépendances longues

Prenons une phrase comme :

```txt
Le livre que Paul a acheté hier dans une petite librairie du centre-ville est passionnant.
```

Le sujet principal est :

```txt
Le livre
```

Le verbe associé est :

```txt
est
```

Mais entre les deux, nous avons beaucoup d’informations intermédiaires :

```txt
que Paul a acheté hier dans une petite librairie du centre-ville
```

Un modèle doit comprendre que :

```txt
Le livre ... est passionnant.
```

et non :

```txt
Paul ... est passionnant.
la librairie ... est passionnant.
le centre-ville ... est passionnant.
```

Le problème est que les RNN doivent transporter l’information importante à travers plusieurs étapes successives.

```mermaid
flowchart LR
    A["Le livre"] --> B["que"]
    B --> C["Paul"]
    C --> D["a acheté"]
    D --> E["hier"]
    E --> F["dans une librairie"]
    F --> G["du centre-ville"]
    G --> H["est passionnant"]

    A -. "dépendance longue" .-> H
```

Plus la séquence est longue, plus il devient difficile de conserver les informations importantes.

C’est ce que nous appelons le problème des **dépendances longues**.

---

## 1.5. Le problème du gradient qui disparaît

Pendant l’entraînement, le modèle apprend en corrigeant ses erreurs. Cette correction se fait par un mécanisme appelé **[rétropropagation du gradient](https://fr.wikipedia.org/wiki/R%C3%A9tropropagation_du_gradient)**.

Dans un RNN, la rétropropagation doit traverser toutes les étapes temporelles.

```mermaid
flowchart RL
    L["Erreur finale"] --> H4["h4"]
    H4 --> H3["h3"]
    H3 --> H2["h2"]
    H2 --> H1["h1"]
    H1 --> X1["Premier token"]
```

Quand la séquence est longue, le signal d’apprentissage peut devenir très faible en remontant vers les premiers tokens.

C’est le problème du **vanishing gradient**, ou **gradient qui disparaît**.

Conséquence :

> Le modèle apprend mal les relations éloignées dans la séquence.

Il peut très bien apprendre des dépendances courtes :

```txt
un chat noir
```

Mais il peut avoir plus de difficulté avec :

```txt
Le chat que j’ai vu hier dans la rue près de la gare était noir.
```

---

## 1.6. Les LSTM et GRU : une amélioration des RNN

Pour limiter ces problèmes, des architectures plus avancées ont été proposées, notamment :

- les **LSTM** ([Long short-term memory](https://fr.wikipedia.org/wiki/R%C3%A9seau_de_neurones_r%C3%A9currents#Long_short-term_memory));
    
- les **GRU** ([Gate Recurrent Unit](https://fr.wikipedia.org/wiki/Unit%C3%A9_r%C3%A9currente_ferm%C3%A9e)).
    

Ces modèles ajoutent des mécanismes de contrôle de la mémoire.

L’idée est de permettre au réseau de décider :

- quoi oublier ;
    
- quoi conserver ;
    
- quoi mettre à jour ;
    
- quoi transmettre.
    

```mermaid
flowchart TD
    A["Entrée x_t"] --> L["Cellule LSTM / GRU"]
    B["Mémoire précédente"] --> L
    L --> C["Nouvelle mémoire"]
    L --> D["Sortie"]
    
    L --> E["Porte d'oubli"]
    L --> F["Porte d'entrée"]
    L --> G["Porte de sortie"]
```

Les LSTM et GRU ont beaucoup amélioré la capacité des réseaux à traiter des séquences.

Mais ils conservent une limite importante :

> Ils traitent toujours les éléments principalement les uns après les autres.

Cette nature séquentielle devient un problème lorsque nous voulons entraîner de grands modèles sur beaucoup de données.

---

## 1.7. Le problème de la parallélisation

Un RNN doit calculer l’état $h_t$ à partir de l’état $h_{t-1}$.

Cela signifie que nous ne pouvons pas facilement calculer tous les états en même temps.

Pour calculer le mot 4, nous devons avoir calculé le mot 3.

Pour calculer le mot 3, nous devons avoir calculé le mot 2.

Pour calculer le mot 2, nous devons avoir calculé le mot 1.

```mermaid
flowchart LR
    X1["x1"] --> H1["h1"]
    H1 --> H2["h2"]
    X2["x2"] --> H2
    H2 --> H3["h3"]
    X3["x3"] --> H3
    H3 --> H4["h4"]
    X4["x4"] --> H4
```

Le calcul est donc fortement séquentiel.

Or, les GPU et TPU sont très efficaces quand nous pouvons faire beaucoup de calculs en parallèle.

Les RNN utilisent mal cette capacité de parallélisation.

C’est une limite majeure pour entraîner des modèles très grands.

---

## 1.8. Les modèles Seq2Seq

Avant les Transformers, une architecture très utilisée pour la traduction automatique était le modèle **sequence-to-sequence**, ou **Seq2Seq**.

L’idée est simple :

- un encodeur lit la phrase source ;
    
- il produit une représentation ;
    
- un décodeur génère la phrase cible.
    

Par exemple :

```txt
Source : I love machine learning.
Cible  : J'aime l'apprentissage automatique.
```

```mermaid
flowchart LR
    A["Phrase source"] --> B["Encodeur RNN"]
    B --> C["Vecteur de contexte"]
    C --> D["Décodeur RNN"]
    D --> E["Phrase cible"]
```

Le problème est que toute la phrase source doit être compressée dans un seul vecteur de contexte.

Pour une phrase courte, cela peut fonctionner.

Pour une phrase longue, c’est beaucoup plus difficile.

---

## 1.9. Le goulot d’étranglement du vecteur de contexte

Dans les premiers modèles Seq2Seq, l’encodeur devait résumer toute la phrase dans un vecteur unique.

Imaginons que nous devions traduire :

```txt
Although the committee had initially rejected the proposal, it later accepted a revised version after several months of discussion.
```

Il est difficile de condenser toutes les informations importantes dans une seule représentation fixe.

```mermaid
flowchart LR
    A["Phrase longue source"] --> B["Encodeur"]
    B --> C["Petit vecteur de contexte"]
    C --> D["Décodeur"]
    D --> E["Traduction"]
```

Ce vecteur devient un **goulot d’étranglement informationnel**.

Le décodeur doit générer une phrase complète à partir d’un résumé compact, alors qu’il aurait parfois besoin de regarder directement certaines parties précises de la phrase source.

C’est ici que l’attention va devenir importante.

---

## 1.10. L’arrivée de l’attention

L’idée de l’attention est de permettre au décodeur de ne pas dépendre uniquement d’un seul vecteur global.

Au lieu de cela, à chaque étape de génération, le décodeur peut regarder différentes parties de la phrase source.

```mermaid
flowchart TD
    A["Phrase source"] --> E1["Représentation mot 1"]
    A --> E2["Représentation mot 2"]
    A --> E3["Représentation mot 3"]
    A --> E4["Représentation mot 4"]

    D["Décodeur"] --> W1["Poids attention mot 1"]
    D --> W2["Poids attention mot 2"]
    D --> W3["Poids attention mot 3"]
    D --> W4["Poids attention mot 4"]

    E1 --> C["Contexte dynamique"]
    E2 --> C
    E3 --> C
    E4 --> C

    W1 --> C
    W2 --> C
    W3 --> C
    W4 --> C
```

Nous pouvons décrire l’attention ainsi :

> L’attention permet au modèle de sélectionner dynamiquement les informations utiles dans une séquence.

Dans une tâche de traduction, lorsqu’il génère un mot français, le modèle peut regarder les mots anglais les plus pertinents.

Par exemple :

```txt
The black cat sleeps.
Le chat noir dort.
```

Quand le modèle génère :

```txt
chat
```

il doit surtout regarder :

```txt
cat
```

Quand il génère :

```txt
noir
```

il doit surtout regarder :

```txt
black
```

---

## 1.11. L’attention comme alignement

Dans la traduction automatique, l’attention peut être vue comme une forme d’alignement entre les mots source et les mots cible.

```mermaid
flowchart LR
    A1["The"] -.-> B1["Le"]
    A2["black"] -.-> B3["noir"]
    A3["cat"] -.-> B2["chat"]
    A4["sleeps"] -.-> B4["dort"]
```

Ce point est important historiquement.

Avant les modèles neuronaux modernes, la traduction automatique utilisait souvent des mécanismes explicites d’alignement statistique.

L’attention a permis de retrouver une forme d’alignement, mais apprise automatiquement par le réseau.

---

## 1.12. Première rupture : l’attention améliore les RNN

Dans un premier temps, l’attention n’a pas remplacé les RNN.

Elle les a complétés.

Nous avions donc des architectures de ce type :

```mermaid
flowchart LR
    A["Phrase source"] --> B["Encodeur RNN"]
    B --> C["États cachés de chaque token"]
    C --> D["Mécanisme d'attention"]
    D --> E["Décodeur RNN"]
    E --> F["Phrase cible"]
```

Cela a permis de grandes améliorations, car le décodeur pouvait accéder à tous les états de l’encodeur, et pas seulement au dernier.

Mais le modèle restait encore partiellement séquentiel.

L’encodeur était souvent récurrent.

Le décodeur était récurrent.

La parallélisation restait limitée.

---

## 1.13. La question centrale

À ce stade, une question devient naturelle :

> Si l’attention est si utile, avons-nous encore besoin des RNN ?

C’est exactement la rupture proposée par le papier **Attention Is All You Need**.

L’idée fondamentale est :

> Nous pouvons construire un modèle de séquence uniquement à partir de mécanismes d’attention, sans récurrence et sans convolution.

Autrement dit, au lieu de lire la phrase mot par mot, nous la traitons globalement.

---

## 1.14. La rupture Transformer

Le Transformer remplace le traitement séquentiel par un traitement fondé sur l’attention entre tous les tokens.

Dans un RNN, chaque token dépend surtout de l’état précédent.

Dans un Transformer, chaque token peut directement interagir avec tous les autres tokens.

```mermaid
flowchart TD
    T1["Token 1"] <--> T2["Token 2"]
    T1 <--> T3["Token 3"]
    T1 <--> T4["Token 4"]
    T2 <--> T3
    T2 <--> T4
    T3 <--> T4
```

Cela change profondément la manière de traiter les séquences.

Nous ne sommes plus dans une lecture strictement linéaire.

Nous sommes dans une mise en relation globale.

---

## 1.15. Exemple intuitif

Prenons la phrase :

```txt
La souris que le chat poursuit court très vite.
```

Le mot :

```txt
court
```

doit être relié à :

```txt
La souris
```

et non à :

```txt
le chat
```

Un Transformer peut apprendre à faire regarder le token `court` vers les tokens utiles :

```mermaid
flowchart LR
    A["La souris"] -. "forte attention" .-> D["court"]
    B["le chat"] -. "attention plus faible" .-> D
    C["poursuit"] -. "contexte" .-> D
```

Le modèle apprend donc des relations grammaticales, sémantiques et contextuelles à partir des données.

---

## 1.16. Comparaison RNN vs Transformer

Nous pouvons comparer les deux approches simplement.

|Critère|RNN / LSTM / GRU|Transformer|
|---|---|---|
|Traitement|Séquentiel|Global et parallélisable|
|Dépendances longues|Difficiles|Plus directes|
|Parallélisation|Limitée|Très forte|
|Mémoire du contexte|Compressée dans des états successifs|Relations directes entre tokens|
|Architecture dominante aujourd’hui|Moins utilisée pour NLP massif|Dominante dans les LLM|

Le point clé est le suivant :

> Le Transformer rend beaucoup plus efficace l’apprentissage sur de grands corpus grâce à sa parallélisation et à son accès direct aux relations entre tokens.

---

## 1.17. Ce que le Transformer gagne

Le Transformer apporte plusieurs avantages majeurs.

### 1.17.1 Meilleure parallélisation

Comme tous les tokens d’une séquence peuvent être traités en même temps dans certaines parties du modèle, l’entraînement devient beaucoup plus efficace sur GPU ou TPU.

```mermaid
flowchart LR
    A["Token 1"] --> P["Traitement parallèle"]
    B["Token 2"] --> P
    C["Token 3"] --> P
    D["Token 4"] --> P
    P --> O["Représentations contextualisées"]
```

Cela permet d’entraîner des modèles plus grands sur davantage de données.

---

### 1.17.2 Meilleure gestion des dépendances longues

Dans un Transformer, deux tokens éloignés peuvent interagir directement via l’attention.

```mermaid
flowchart LR
    A["Début de phrase"] -. "attention directe" .-> Z["Fin de phrase"]
```

Dans un RNN, l’information doit passer par tous les états intermédiaires.

```mermaid
flowchart LR
    A["Début"] --> B["..."]
    B --> C["..."]
    C --> D["..."]
    D --> Z["Fin"]
```

La différence est essentielle.

---

### 1.17.3 Représentations contextualisées

Dans un Transformer, le vecteur associé à un mot dépend des autres mots de la phrase.

Le mot `banque` n’a pas la même représentation dans :

```txt
Je vais à la banque déposer un chèque.
```

et :

```txt
Nous nous asseyons sur la banque au bord de la rivière.
```

Même mot, mais contexte différent.

Le Transformer produit donc une représentation contextualisée.

```mermaid
flowchart TD
    A["banque + argent + chèque"] --> B["Sens financier"]
    C["banque + rivière + bord"] --> D["Sens géographique"]
```

---

## 1.18. Ce que le Transformer perd ou complique

Il ne faut pas présenter les Transformers comme une solution magique.

Ils ont aussi des limites.

### 1.18.1 Coût quadratique de l’attention

Si chaque token regarde tous les autres tokens, alors le nombre de relations à calculer augmente très vite.

Pour une séquence de longueur $n$, la matrice d’attention contient :

$$n \times n$$


relations.

```mermaid
flowchart TD
    A["n tokens"] --> B["Matrice d'attention n x n"]
    B --> C["Coût mémoire important"]
    B --> D["Coût calculatoire important"]
```

C’est pourquoi les longues séquences sont coûteuses.

Nous reviendrons sur ce point dans le chapitre consacré à la complexité.

---

### 1.18.2 Absence d’ordre naturel

Un RNN lit les mots dans l’ordre. L’ordre est donc intégré naturellement dans le processus.

Un Transformer, lui, regarde tous les tokens en parallèle.

Il faut donc lui ajouter explicitement une information de position.

```mermaid
flowchart LR
    A["Token embedding"] --> C["Entrée Transformer"]
    B["Position encoding"] --> C
```

Sans information de position, le Transformer ne saurait pas distinguer :

```txt
Le chien mord l’homme.
```

de :

```txt
L’homme mord le chien.
```

Nous étudierons ce problème en détail dans le chapitre 3.

---

### 1.18.3 Besoin massif de données

Les Transformers modernes sont très puissants, mais ils nécessitent souvent :

- beaucoup de données ;
    
- beaucoup de calcul ;
    
- beaucoup de mémoire ;
    
- une infrastructure matérielle importante.
    

C’est particulièrement vrai pour les grands modèles de langage.

---

## 1.19. Transformer et changement d’échelle

Le succès des Transformers ne vient pas seulement de leur élégance théorique.

Il vient aussi de leur compatibilité avec le passage à l’échelle.

Autrement dit, les Transformers se sont révélés très efficaces lorsque nous augmentons :

- la taille des données ;
    
- la taille du modèle ;
    
- la durée d’entraînement ;
    
- la puissance de calcul.
    

```mermaid
flowchart TD
    A["Architecture parallélisable"] --> B["Entraînement sur GPU/TPU"]
    B --> C["Plus de données"]
    C --> D["Modèles plus grands"]
    D --> E["Meilleures performances"]
    E --> F["LLM modernes"]
```

C’est ce qui a permis l’émergence des grands modèles de langage modernes.

---

## 1.20. Les grandes étapes historiques

Nous pouvons résumer l’évolution des architectures de séquences ainsi :

```mermaid
timeline
    title Évolution des modèles de séquences
    RNN : Lecture séquentielle simple
    LSTM / GRU : Meilleure mémoire
    Seq2Seq : Traduction neuronale
    Attention : Accès dynamique au contexte
    Transformer : Attention sans récurrence
    BERT / GPT : Pré-entraînement massif
    LLM modernes : Passage à l'échelle
```

Cette progression montre que les Transformers ne sont pas apparus de nulle part.

Ils sont la réponse à plusieurs limites accumulées :

- les RNN traitaient mal les dépendances longues ;
    
- les modèles Seq2Seq compressaient trop l’information ;
    
- les modèles récurrents étaient difficiles à paralléliser ;
    
- l’attention avait déjà montré son efficacité ;
    
- les GPU/TPU favorisaient les architectures parallélisables.
    

---

## 1.21. L’idée centrale du chapitre

Nous pouvons maintenant formuler l’idée centrale :

> Le Transformer est une architecture conçue pour traiter les séquences en reliant directement les éléments entre eux grâce à l’attention, au lieu de les traiter uniquement dans un ordre séquentiel.

Cela permet :

- de mieux capturer les dépendances longues ;
    
- de paralléliser fortement l’entraînement ;
    
- de produire des représentations contextualisées ;
    
- de construire des modèles très grands ;
    
- de généraliser l’architecture à de nombreux domaines.
    

---

## 1.22. Exemple global : d’une phrase à une représentation contextualisée

Prenons la phrase :

```txt
Le chat noir dort sur le canapé.
```

Un Transformer ne se contente pas d’associer un vecteur fixe à chaque mot.

Il construit une représentation de chaque mot en fonction de tous les autres.

```mermaid
flowchart TD
    A["Le"] --> T["Transformer"]
    B["chat"] --> T
    C["noir"] --> T
    D["dort"] --> T
    E["sur"] --> T
    F["le"] --> T
    G["canapé"] --> T

    T --> A2["Représentation contextualisée de Le"]
    T --> B2["Représentation contextualisée de chat"]
    T --> C2["Représentation contextualisée de noir"]
    T --> D2["Représentation contextualisée de dort"]
    T --> G2["Représentation contextualisée de canapé"]
```

Le mot `chat` sera influencé par :

- `Le`, qui indique le déterminant ;
    
- `noir`, qui apporte une propriété ;
    
- `dort`, qui indique l’action ;
    
- `canapé`, qui donne le contexte de la scène.
    

---

## 1.23. Attention : ce que nous ne savons pas encore

À ce stade du cours, nous comprenons pourquoi les Transformers sont nécessaires, mais nous n’avons pas encore détaillé comment ils fonctionnent.

Nous ne savons pas encore précisément :

- comment un mot devient un vecteur ;
    
- comment le modèle encode l’ordre ;
    
- ce que sont Query, Key et Value ;
    
- comment l’attention est calculée ;
    
- pourquoi on parle de multi-head attention ;
    
- comment fonctionne un bloc encoder ;
    
- comment fonctionne un bloc decoder ;
    
- comment le modèle est entraîné.
    

Ces éléments seront construits progressivement dans les chapitres suivants.

---

## 1.24. Résumé du chapitre

Nous avons vu que les Transformers répondent à plusieurs limites des architectures précédentes.

Les **RNN** lisent les séquences pas à pas, ce qui rend les dépendances longues difficiles et limite la parallélisation.

Les **LSTM** et **GRU** améliorent la mémoire, mais restent fondamentalement séquentiels.

Les modèles **Seq2Seq** ont permis de grandes avancées, notamment en traduction automatique, mais ils souffraient du goulot d’étranglement du vecteur de contexte.

L’**attention** a permis au modèle de sélectionner dynamiquement les parties pertinentes d’une séquence.

Le **Transformer** pousse cette idée plus loin :

> Nous supprimons la récurrence et nous construisons une architecture entièrement fondée sur l’attention.

Cela permet de traiter les séquences de manière plus globale, plus parallélisable et plus adaptée au passage à l’échelle.

---

## 1.25. Schéma de synthèse

```mermaid
flowchart TD
    A["Problème : traiter des séquences"] --> B["RNN"]
    B --> C["Limite : dépendances longues"]
    B --> D["Limite : faible parallélisation"]

    C --> E["LSTM / GRU"]
    D --> E

    E --> F["Seq2Seq"]
    F --> G["Limite : vecteur de contexte unique"]

    G --> H["Attention"]
    H --> I["Meilleur accès au contexte"]

    I --> J["Transformer"]
    J --> K["Attention sans récurrence"]
    J --> L["Parallélisation"]
    J --> M["Dépendances longues"]
    J --> N["LLM modernes"]
```

---

## 1.26. Questions de compréhension

Pour vérifier que nous avons compris ce chapitre, nous pouvons répondre aux questions suivantes.

### Question 1

Pourquoi les RNN sont-ils naturellement adaptés aux séquences ?

Réponse attendue : parce qu’ils lisent les éléments les uns après les autres et maintiennent un état interne qui transporte l’information au fil du temps.

### Question 2

Pourquoi les RNN ont-ils du mal avec les dépendances longues ?

Réponse attendue : parce que l’information doit traverser de nombreuses étapes successives, ce qui peut entraîner une perte d’information et un affaiblissement du gradient.

### Question 3

Quel problème les LSTM et GRU cherchent-ils à résoudre ?

Réponse attendue : ils cherchent à améliorer la mémoire des RNN grâce à des mécanismes de portes permettant de contrôler ce qui est conservé, oublié ou transmis.

### Question 4

Quel est le problème du vecteur de contexte unique dans les premiers modèles Seq2Seq ?

Réponse attendue : toute la phrase source doit être compressée dans une seule représentation, ce qui devient insuffisant pour les phrases longues ou complexes.

### Question 5

Quelle est l’idée principale de l’attention ?

Réponse attendue : permettre au modèle de regarder dynamiquement les parties pertinentes de la séquence au lieu de dépendre d’une seule représentation globale.

### Question 6

Quelle est la rupture introduite par le Transformer ?

Réponse attendue : le Transformer supprime la récurrence et repose principalement sur l’attention pour relier directement les tokens entre eux.

### Question 7

Pourquoi les Transformers sont-ils plus faciles à paralléliser que les RNN ?

Réponse attendue : parce que les représentations des tokens peuvent être calculées simultanément dans les couches d’attention, alors que les RNN nécessitent de calculer les états dans l’ordre.

### Question 8

Quelle est une limite importante des Transformers ?

Réponse attendue : le coût de l’attention augmente quadratiquement avec la longueur de la séquence, car chaque token peut regarder tous les autres tokens.

---

## 1.27. Transition vers le chapitre 2

Dans ce chapitre, nous avons compris pourquoi les Transformers ont été proposés.

Dans le chapitre suivant, nous allons préparer les bases nécessaires pour entrer dans leur fonctionnement interne.

Nous verrons comment passer de ceci :

```txt
Le chat dort.
```

à ceci :

```txt
[154, 932, 421, 13]
```

puis à des vecteurs numériques manipulables par un réseau de neurones.

Autrement dit, nous allons étudier :

- la tokenisation ;
    
- les tokens ;
    
- les embeddings ;
    
- les tenseurs ;
    
- les dimensions utilisées dans un Transformer.
    

Le chapitre 2 construira donc le pont entre le texte brut et l’entrée numérique du modèle.

---

# Chapitre 2 Rappels nécessaires : tenseurs, embeddings et séquences

## 2.1. Objectif du chapitre

Dans le chapitre précédent, nous avons compris **pourquoi les Transformers ont été introduits** : ils répondent aux limites des architectures séquentielles comme les RNN, LSTM et GRU.

Dans ce deuxième chapitre, nous allons préparer les bases techniques nécessaires pour comprendre l’entrée d’un Transformer.

Un Transformer ne manipule pas directement du texte brut comme :

```txt
Le chat dort sur le canapé.
```

Il manipule des **nombres**, organisés dans des **vecteurs**, eux-mêmes regroupés dans des **matrices** ou des **tenseurs**.

Nous allons donc comprendre comment nous passons de ceci :

```txt
Le chat dort sur le canapé.
```

à ceci :

```txt
[154, 932, 421, 78, 154, 2356]
```

puis à ceci :

```txt
[
  [0.12, -0.45, 0.88, ...],
  [0.31,  0.07, -0.22, ...],
  ...
]
```

Autrement dit, nous allons étudier :

- la tokenisation ;
    
- les tokens ;
    
- les identifiants de tokens ;
    
- les embeddings ;
    
- les tenseurs ;
    
- les dimensions utilisées dans les Transformers ;
    
- la différence entre représentation symbolique et représentation vectorielle.
    

---

## 2.2. Du texte brut aux données numériques

Un réseau de neurones ne comprend pas directement les mots.

Pour un humain, cette phrase est lisible :

```txt
Le chat dort.
```

Mais pour une machine learning model, ce texte doit être converti en nombres.

Nous avons donc une chaîne de transformation :

```mermaid
flowchart LR
    A["Texte brut"] --> B["Tokenisation"]
    B --> C["Tokens"]
    C --> D["IDs de tokens"]
    D --> E["Embeddings"]
    E --> F["Tenseur d'entrée du Transformer"]
```

L’idée générale est la suivante :

1. nous découpons le texte en morceaux ;
    
2. nous associons chaque morceau à un identifiant numérique ;
    
3. nous transformons chaque identifiant en vecteur dense ;
    
4. nous envoyons ces vecteurs dans le Transformer.
    

---

## 2.3. Qu’est-ce qu’un token ?

Un **token** est une unité de texte manipulée par le modèle.

Un token peut être :

- un mot ;
    
- une partie de mot ;
    
- un caractère ;
    
- un symbole ;
    
- un signe de ponctuation ;
    
- un espace ou une marque spéciale selon le tokenizer.
    

Par exemple, la phrase :

```txt
Le chat dort.
```

peut être découpée ainsi :

```txt
["Le", "chat", "dort", "."]
```

Mais selon le tokenizer, elle pourrait aussi être découpée différemment.

Par exemple :

```txt
["Le", "Ġchat", "Ġdort", "."]
```

ou encore :

```txt
["L", "e", "chat", "dort", "."]
```

Le token n’est donc pas forcément un mot.

C’est un point fondamental.

---

## 2.4. Pourquoi ne pas utiliser directement les mots ?

Nous pourrions imaginer un modèle qui manipule directement les mots du dictionnaire.

Mais cela pose plusieurs problèmes.

### 2.4.1 Vocabulaire énorme

Une langue contient énormément de mots :

```txt
chat, chats, chatte, chaton, chatière, etc.
```

Si nous ajoutons :

- les conjugaisons ;
    
- les accords ;
    
- les noms propres ;
    
- les fautes de frappe ;
    
- les mots rares ;
    
- les termes techniques ;
    
- les mots étrangers ;
    

le vocabulaire devient gigantesque.

### 2.4.2 Mots inconnus

Si le modèle rencontre un mot absent de son vocabulaire, il ne sait pas quoi faire.

Par exemple :

```txt
anticonstitutionnellement
```

ou :

```txt
GmodIntegration
```

ou encore :

```txt
docker-compose.override.yml
```

Un tokenizer moderne doit pouvoir traiter ces formes rares ou nouvelles.

### 2.4.3 Sous-mots

Pour résoudre ce problème, beaucoup de modèles utilisent des tokens de type **sous-mots**.

Par exemple :

```txt
anticonstitutionnellement
```

pourrait être découpé en :

```txt
["anti", "constitution", "nelle", "ment"]
```

L’avantage est que le modèle peut comprendre un mot rare à partir de morceaux plus fréquents.

---

## 2.5. Tokenisation par mots, caractères et sous-mots

Nous pouvons distinguer trois grandes approches.

### 2.5.1 Tokenisation par mots

La phrase :

```txt
Le chat dort.
```

devient :

```txt
["Le", "chat", "dort", "."]
```

Avantage : c’est intuitif.

Inconvénient : le vocabulaire devient très grand et les mots rares posent problème.

---

### 2.5.2 Tokenisation par caractères

La phrase :

```txt
chat
```

devient :

```txt
["c", "h", "a", "t"]
```

Avantage : presque aucun mot inconnu.

Inconvénient : les séquences deviennent beaucoup plus longues.

Le modèle doit reconstruire lui-même les mots à partir des caractères.

---

### 2.5.3 Tokenisation par sous-mots

La phrase :

```txt
reconstruction
```

peut devenir :

```txt
["re", "construction"]
```

ou :

```txt
["recon", "struction"]
```

ou encore :

```txt
["re", "construct", "ion"]
```

C’est l’approche la plus utilisée dans les grands modèles de langage modernes.

Elle offre un bon compromis entre :

- vocabulaire raisonnable ;
    
- capacité à traiter les mots rares ;
    
- longueur de séquence acceptable.
    

```mermaid
flowchart TD
    A["Texte brut"] --> B1["Tokenisation par mots"]
    A --> B2["Tokenisation par caractères"]
    A --> B3["Tokenisation par sous-mots"]

    B1 --> C1["Simple mais vocabulaire énorme"]
    B2 --> C2["Robuste mais séquences longues"]
    B3 --> C3["Compromis utilisé par beaucoup de LLM"]
```

---

## 2.6. Le vocabulaire du modèle

Un modèle de langage possède un **vocabulaire**, c’est-à-dire une liste finie de tokens qu’il connaît.

Par exemple, un vocabulaire simplifié pourrait être :

|ID|Token|
|--:|---|
|0|`<pad>`|
|1|`<unk>`|
|2|`Le`|
|3|`chat`|
|4|`dort`|
|5|`sur`|
|6|`canapé`|
|7|`.`|

La phrase :

```txt
Le chat dort sur le canapé.
```

pourrait alors devenir :

```txt
[2, 3, 4, 5, 2, 6, 7]
```

Nous avons remplacé les tokens par des identifiants numériques.

Mais attention : ces identifiants ne portent pas encore de sens mathématique.

Le fait que `chat = 3` et `dort = 4` ne signifie pas que `dort` est “plus grand” que `chat`.

Ces IDs sont seulement des indices dans une table.

---

## 2.7. Tokens spéciaux

Les modèles utilisent souvent des tokens spéciaux.

Par exemple :

|Token spécial|Rôle|
|---|---|
|`<pad>`|Remplissage pour obtenir des séquences de même longueur|
|`<unk>`|Token inconnu|
|`<bos>`|Début de séquence|
|`<eos>`|Fin de séquence|
|`<mask>`|Token masqué, utilisé notamment dans BERT|
|`<sep>`|Séparateur entre deux segments|
|`<cls>`|Token de classification, utilisé notamment dans BERT|

Par exemple, pour une tâche de classification, nous pourrions avoir :

```txt
[CLS] Le chat dort. [SEP]
```

Pour une tâche de génération, nous pourrions avoir :

```txt
[BOS] Le chat dort. [EOS]
```

Ces tokens spéciaux permettent au modèle de repérer la structure de l’entrée.

---

# 2.8. Padding : pourquoi compléter les séquences ?

Dans un batch, nous entraînons souvent le modèle sur plusieurs phrases en même temps.

Mais les phrases n’ont pas toutes la même longueur.

Par exemple :

```txt
Phrase 1 : Le chat dort.
Phrase 2 : Le petit chat noir dort sur le canapé.
```

Après tokenisation :

```txt
Phrase 1 : [2, 3, 4, 7]
Phrase 2 : [2, 8, 3, 9, 4, 5, 2, 6, 7]
```

Pour les mettre dans un même tenseur, nous devons souvent les compléter avec un token de padding :

```txt
Phrase 1 : [2, 3, 4, 7, 0, 0, 0, 0, 0]
Phrase 2 : [2, 8, 3, 9, 4, 5, 2, 6, 7]
```

Ici, `0` correspond à `<pad>`.

```mermaid
flowchart TD
    A["Séquences de longueurs différentes"] --> B["Ajout de tokens <pad>"]
    B --> C["Séquences de même longueur"]
    C --> D["Batch tensoriel"]
```

Cependant, le modèle ne doit pas considérer les tokens `<pad>` comme du vrai texte.

Nous utiliserons donc plus tard un **padding mask** pour les ignorer dans l’attention.

---

# 2.9. Représentation symbolique contre représentation distribuée

Une fois que nous avons des IDs de tokens, nous avons une représentation symbolique :

```txt
Le    → 2
chat  → 3
dort  → 4
```

Mais cette représentation est pauvre.

Elle ne dit pas que :

- `chat` est proche de `chien` ;
    
- `dort` est proche de `sommeille` ;
    
- `canapé` est proche de `fauteuil` ;
    
- `manger` est différent de `dormir`.
    

Pour que le modèle apprenne des relations sémantiques, chaque token est transformé en vecteur.

C’est le rôle des **embeddings**.

---

## 2.10. Qu’est-ce qu’un embedding ?

Un **embedding** est un vecteur dense associé à un token.

Par exemple, le token `chat` peut être représenté par un vecteur :

```txt
chat → [0.21, -0.38, 0.74, 0.12, ...]
```

Dans un vrai Transformer, ce vecteur peut avoir une dimension comme :

```txt
768, 1024, 4096, 8192, ...
```

selon la taille du modèle.

Nous appelons souvent cette dimension :

$$d_{model}$$
C’est la dimension principale des représentations internes du Transformer.

---

## 2.11. Table d’embeddings

Concrètement, le modèle contient une grande matrice appelée **table d’embeddings**.

Si le vocabulaire contient (V) tokens et que chaque embedding a une dimension $d_{model}$, alors la table d’embeddings a la forme :

$$V \times d_{model}$$

Par exemple, avec :

```txt
V = 50 000 tokens
d_model = 768
```

la matrice d’embeddings contient :

```txt
50 000 × 768
```

valeurs apprises.

```mermaid
flowchart LR
    A["ID du token"] --> B["Table d'embeddings"]
    B --> C["Vecteur dense de dimension d_model"]
```

Exemple simplifié :

|Token|ID|Embedding simplifié|
|---|--:|---|
|Le|2|`[0.1, 0.4, -0.2]`|
|chat|3|`[0.7, -0.1, 0.5]`|
|dort|4|`[-0.3, 0.8, 0.2]`|

Donc :

```txt
[2, 3, 4]
```

devient :

```txt
[
  [0.1, 0.4, -0.2],
  [0.7, -0.1, 0.5],
  [-0.3, 0.8, 0.2]
]
```

---

## 2.12. Les embeddings sont appris

Les embeddings ne sont pas écrits à la main.

Ils sont appris pendant l’entraînement.

Au début, ils peuvent être initialisés aléatoirement.

Puis, au fil de l’apprentissage, le modèle ajuste les vecteurs pour mieux résoudre sa tâche.

Par exemple, dans un modèle de langage, le modèle apprend progressivement que certains tokens apparaissent dans des contextes similaires.

Ainsi, les embeddings de mots proches sémantiquement peuvent finir par se rapprocher dans l’espace vectoriel.

```mermaid
flowchart TD
    A["Initialisation aléatoire"] --> B["Entraînement"]
    B --> C["Correction par gradient"]
    C --> D["Embeddings plus utiles"]
    D --> E["Représentations sémantiques apprises"]
```

---

## 2.13. Intuition géométrique des embeddings

Nous pouvons voir les embeddings comme des points dans un espace multidimensionnel.

Dans un espace très simplifié à deux dimensions, nous pourrions imaginer :

```mermaid
quadrantChart
    title Espace d'embeddings simplifié
    x-axis "animal" --> "objet"
    y-axis "repos" --> "action"
    quadrant-1 "Objets actifs"
    quadrant-2 "Animaux actifs"
    quadrant-3 "Animaux au repos"
    quadrant-4 "Objets au repos"
    "chat": [0.2, 0.3]
    "chien": [0.25, 0.35]
    "canapé": [0.8, 0.2]
    "dormir": [0.3, 0.15]
```

Bien sûr, dans un vrai modèle, l’espace n’a pas deux dimensions mais souvent plusieurs centaines ou milliers.

L’important est l’idée suivante :

> Les embeddings permettent de représenter les tokens dans un espace où les relations entre vecteurs peuvent porter du sens.

---

## 2.14. Embedding statique et embedding contextualisé

Il faut distinguer deux notions.

### 2.14.1 Embedding statique

La table d’embeddings donne une représentation initiale du token.

Par exemple :

```txt
banque → [0.18, -0.22, 0.91, ...]
```

Cette représentation est la même au départ, quel que soit le contexte.

### 2.14.2 Représentation contextualisée

Après passage dans le Transformer, le vecteur du token dépend du contexte.

Le mot `banque` dans :

```txt
Je vais à la banque déposer un chèque.
```

n’aura pas la même représentation finale que dans :

```txt
Nous marchons sur la banque de sable.
```

```mermaid
flowchart TD
    A["Embedding initial de banque"] --> T1["Transformer avec contexte financier"]
    A --> T2["Transformer avec contexte géographique"]

    T1 --> B["Représentation contextualisée : établissement financier"]
    T2 --> C["Représentation contextualisée : bord / dépôt naturel"]
```

Donc :

> L’embedding initial donne un point de départ, mais le Transformer construit ensuite une représentation dépendante du contexte.

---

## 2.15. Qu’est-ce qu’un tenseur ?

Un **tenseur** est une généralisation des scalaires, vecteurs et matrices.

Nous pouvons retenir simplement :

|Objet|Exemple|Nombre de dimensions|
|---|---|--:|
|Scalaire|`3.14`|0D|
|Vecteur|`[0.1, 0.2, 0.3]`|1D|
|Matrice|`[[1, 2], [3, 4]]`|2D|
|Tenseur|batch de matrices|3D ou plus|

Dans les Transformers, nous manipulons très souvent des tenseurs à trois dimensions :

```txt
(batch_size, sequence_length, d_model)
```

---

## 2.16. Dimensions classiques dans un Transformer

Prenons un exemple.

Nous avons un batch de 32 phrases.

Chaque phrase est tronquée ou complétée à 128 tokens.

Chaque token est représenté par un embedding de dimension 768.

Le tenseur d’entrée a donc la forme :

```txt
(32, 128, 768)
```

Ce qui signifie :

```txt
32 phrases
128 tokens par phrase
768 valeurs par token
```

```mermaid
flowchart TD
    A["Batch size = 32"] --> D["Tenseur d'entrée"]
    B["Sequence length = 128"] --> D
    C["d_model = 768"] --> D
    D --> E["Shape : 32 x 128 x 768"]
```

Nous retrouverons ces dimensions tout au long du cours.

---

## 2.17. Dimension batch

La dimension **batch** correspond au nombre d’exemples traités en même temps.

Par exemple :

```txt
batch_size = 4
```

signifie que nous envoyons 4 séquences en parallèle au modèle.

```txt
Phrase 1 : Le chat dort.
Phrase 2 : Le chien aboie.
Phrase 3 : Il pleut aujourd’hui.
Phrase 4 : J’aime les Transformers.
```

Le batch permet :

- d’accélérer l’entraînement ;
    
- de mieux utiliser le GPU ;
    
- de stabiliser l’estimation du gradient.
    

---

## 2.18. Dimension sequence length

La dimension **sequence length** correspond au nombre de tokens dans chaque séquence.

Par exemple :

```txt
Le chat dort.
```

peut devenir :

```txt
["Le", "chat", "dort", "."]
```

Donc :

```txt
sequence_length = 4
```

Mais dans un batch, nous fixons souvent une longueur maximale :

```txt
max_sequence_length = 128
```

Les phrases plus courtes sont complétées avec du padding.

Les phrases plus longues sont tronquées ou découpées.

---

## 2.19. Dimension d_model

La dimension $d_{model}$ est la taille du vecteur associé à chaque token.

Par exemple :

```txt
d_model = 768
```

signifie que chaque token est représenté par 768 nombres.

Le choix de $d_{model}$ influence :

- la capacité du modèle ;
    
- le nombre de paramètres ;
    
- le coût mémoire ;
    
- le coût calculatoire.
    

Un modèle avec un $d_{model}$ plus grand peut représenter plus d’informations, mais coûte plus cher à entraîner et à utiliser.

---

## 2.20. Exemple complet de transformation

Prenons la phrase :

```txt
Le chat dort.
```

Étape 1 : tokenisation

```txt
["Le", "chat", "dort", "."]
```

Étape 2 : conversion en IDs

```txt
[2, 3, 4, 7]
```

Étape 3 : embeddings simplifiés

```txt
[
  [0.1, 0.4, -0.2],
  [0.7, -0.1, 0.5],
  [-0.3, 0.8, 0.2],
  [0.0, -0.5, 0.9]
]
```

Ici, nous avons :

```txt
sequence_length = 4
d_model = 3
```

Donc la forme est :

```txt
(4, 3)
```

Si nous ajoutons un batch de taille 1, la forme devient :

```txt
(1, 4, 3)
```

```mermaid
flowchart LR
    A["Le chat dort."] --> B["Tokens"]
    B --> C["IDs : [2, 3, 4, 7]"]
    C --> D["Embeddings"]
    D --> E["Shape : 1 x 4 x 3"]
```

---

## 2.21. Pourquoi les embeddings ne suffisent pas ?

Les embeddings initiaux ne connaissent pas encore le contexte précis.

Par exemple, dans :

```txt
Le chat dort.
```

le token `chat` reçoit un vecteur initial.

Dans :

```txt
Le chat de discussion est ouvert.
```

le token `chat` peut avoir une signification différente selon le contexte.

L’embedding initial ne suffit donc pas.

Le rôle du Transformer sera de transformer ces embeddings initiaux en représentations contextualisées.

```mermaid
flowchart LR
    A["Embeddings initiaux"] --> B["Transformer"]
    B --> C["Représentations contextualisées"]
```

---

## 2.22. Séquence d’entrée dans un Transformer

L’entrée d’un Transformer est donc une matrice de vecteurs.

Pour une phrase de $n$ tokens, nous avons :
 
$$X \in \mathbb{R}^{n \times d_{model}}$$

où :

- $n$ est la longueur de la séquence ;
    
- $d_{model}$ est la dimension des embeddings.
    

Pour un batch, nous avons :

$$X \in \mathbb{R}^{B \times n \times d_{model}}$$
où :

- $B$ est la taille du batch ;
    
- $n$ est la longueur de séquence ;
    
- $d_{model}$ est la dimension du modèle.
    

C’est ce tenseur qui sera envoyé dans les couches du Transformer.

---

## 2.23. Le problème restant : l’ordre

À ce stade, nous avons transformé les tokens en vecteurs.

Mais nous avons un problème majeur.

Si nous envoyons simplement une liste de vecteurs au Transformer, l’architecture d’attention ne connaît pas naturellement l’ordre des tokens.

La phrase :

```txt
Le chien mord l’homme.
```

et la phrase :

```txt
L’homme mord le chien.
```

contiennent presque les mêmes tokens.

Mais le sens est différent.

```mermaid
flowchart TD
    A["Même ensemble de mots"] --> B["Ordre différent"]
    B --> C["Sens différent"]
    C --> D["Il faut encoder la position"]
```

Un RNN encode implicitement l’ordre car il lit la phrase de gauche à droite.

Un Transformer, lui, traite les tokens en parallèle.

Nous devrons donc ajouter explicitement une information de position.

C’est le sujet du chapitre suivant.

---

## 2.24. Résumé du chapitre

Dans ce chapitre, nous avons construit le pipeline d’entrée d’un Transformer.

Nous avons vu que le texte brut doit être transformé en nombres.

La première étape est la **tokenisation**, qui découpe le texte en tokens.

Ces tokens sont ensuite convertis en **IDs**, c’est-à-dire en indices dans le vocabulaire du modèle.

Ces IDs sont transformés en **embeddings**, c’est-à-dire en vecteurs denses appris pendant l’entraînement.

Ces vecteurs sont regroupés dans un **tenseur** de forme typique :

```txt
(batch_size, sequence_length, d_model)
```

Nous avons aussi distingué :

- l’embedding initial, associé au token ;
    
- la représentation contextualisée, produite ensuite par le Transformer.
    

Enfin, nous avons identifié le problème suivant :

> Les Transformers ne connaissent pas naturellement l’ordre des tokens.

Nous devrons donc ajouter une information de position.

---

# 25. Schéma de synthèse

```mermaid
flowchart TD
    A["Texte brut"] --> B["Tokenisation"]
    B --> C["Tokens"]
    C --> D["IDs de tokens"]
    D --> E["Table d'embeddings"]
    E --> F["Vecteurs denses"]
    F --> G["Tenseur : batch x séquence x d_model"]
    G --> H["Entrée du Transformer"]

    H --> I["Problème restant : ordre des tokens"]
    I --> J["Chapitre 3 : positional encoding"]
```

---

## 2.26. Questions de compréhension

### Question 1

Pourquoi un Transformer ne peut-il pas manipuler directement du texte brut ?

Réponse attendue : parce qu’un réseau de neurones manipule des nombres, pas des chaînes de caractères. Le texte doit donc être converti en tokens, puis en IDs, puis en vecteurs.

### Question 2

Qu’est-ce qu’un token ?

Réponse attendue : un token est une unité de texte manipulée par le modèle. Il peut correspondre à un mot, un sous-mot, un caractère, une ponctuation ou un symbole spécial.

### Question 3

Pourquoi utilise-t-on souvent des sous-mots plutôt que des mots entiers ?

Réponse attendue : parce que les sous-mots permettent de limiter la taille du vocabulaire tout en traitant correctement les mots rares, composés ou inconnus.

### Question 4

Quelle est la différence entre un ID de token et un embedding ?

Réponse attendue : un ID est simplement un indice numérique dans le vocabulaire. Un embedding est un vecteur dense appris qui représente le token dans un espace numérique.

### Question 5

Quelle est la forme typique d’un tenseur d’entrée dans un Transformer ?

Réponse attendue :

```txt
(batch_size, sequence_length, d_model)
```

### Question 6

Que signifie $d_{model}$ ?

Réponse attendue : $d_{model}$ est la dimension des vecteurs internes du Transformer, donc la taille de la représentation associée à chaque token.

### Question 7

Pourquoi les tokens `<pad>` sont-ils nécessaires ?

Réponse attendue : ils permettent de compléter les séquences plus courtes afin d’obtenir des séquences de même longueur dans un batch.

### Question 8

Pourquoi les embeddings initiaux ne suffisent-ils pas ?

Réponse attendue : parce qu’ils ne dépendent pas encore du contexte précis de la phrase. Le Transformer doit ensuite produire des représentations contextualisées.

### Question 9

Quel problème reste à résoudre à la fin du chapitre ?

Réponse attendue : il reste à représenter l’ordre des tokens, car le Transformer ne lit pas naturellement la séquence de gauche à droite comme un RNN.

---

## 2.27. Transition vers le chapitre 3

Nous savons maintenant comment transformer du texte brut en tenseur d’entrée.

Mais nous avons identifié un problème fondamental : une simple collection de vecteurs ne suffit pas à représenter une phrase.

L’ordre des mots est essentiel.

Dans le chapitre suivant, nous allons donc étudier :

- pourquoi le Transformer ne connaît pas naturellement les positions ;
    
- comment ajouter une information de position ;
    
- les positional encodings sinusoïdaux du papier original ;
    
- les positional embeddings appris ;
    
- les méthodes modernes comme RoPE et ALiBi.
    

Le chapitre 3 répondra donc à cette question :

> Comment un Transformer sait-il où se trouve chaque token dans la séquence ?

---
# Chapitre 3 — Le principe des RNN

## 3.1. Objectif du chapitre

Dans le chapitre précédent, nous avons commencé à comprendre comment les modèles de traitement séquentiel fonctionnent avant l’arrivée des Transformers.

Nous avons vu que les **RNN**, ou **réseaux de neurones récurrents**, lisent une séquence élément par élément.

Dans ce chapitre, nous allons entrer plus précisément dans leur fonctionnement interne.

Nous allons comprendre :

- ce qu’est une cellule RNN ;
    
- ce qu’est un état caché ;
    
- comment le modèle transporte une mémoire ;
    
- comment une séquence est traitée pas à pas ;
    
- pourquoi cette mémoire est utile ;
    
- pourquoi elle reste limitée ;
    
- en quoi cette limite préparera l’arrivée des Transformers.
    

L’objectif est simple : nous devons comprendre le fonctionnement des RNN pour mieux comprendre ensuite pourquoi les Transformers ont proposé une rupture.

---

## 3.2. Rappel : une séquence se lit étape par étape

Un RNN traite une séquence dans l’ordre.

Prenons la phrase :

```txt
Le chat dort sur le canapé.
```

Après tokenisation, nous pouvons obtenir :

```txt
["Le", "chat", "dort", "sur", "le", "canapé", "."]
```

Le RNN lit alors les tokens un par un :

```txt
Le → chat → dort → sur → le → canapé → .
```

À chaque étape, il met à jour une représentation interne que nous appelons **état caché**.

```mermaid
flowchart LR
    X1["Le"] --> R1["RNN"]
    R1 --> R2["RNN"]
    X2["chat"] --> R2
    R2 --> R3["RNN"]
    X3["dort"] --> R3
    R3 --> R4["RNN"]
    X4["sur"] --> R4
    R4 --> R5["RNN"]
    X5["le canapé"] --> R5
    R5 --> H["Représentation finale"]
```

Nous pouvons donc voir le RNN comme un lecteur qui avance dans la phrase en gardant une mémoire de ce qu’il a déjà lu.

---

## 3.3. La cellule RNN

Le cœur d’un RNN est la **cellule récurrente**.

À chaque position $t$, cette cellule reçoit deux informations :

1. l’entrée actuelle $x_t$ ;
    
2. l’état caché précédent $h_{t-1}$.
    

Elle produit ensuite :

1. un nouvel état caché $h_t$ ;
    
2. éventuellement une sortie $y_t$.
    

```mermaid
flowchart TD
    A["Entrée actuelle x_t"] --> C["Cellule RNN"]
    B["État précédent h_(t-1)"] --> C
    C --> D["Nouvel état h_t"]
    C --> E["Sortie éventuelle y_t"]
```

L’idée est très importante :

> Le RNN ne regarde pas seulement le token actuel. Il regarde aussi une mémoire de ce qui a été lu avant.

C’est ce qui distingue un RNN d’un simple réseau dense appliqué indépendamment à chaque mot.

---

## 3.4. La formule fondamentale

Mathématiquement, nous pouvons écrire :


$$h_t = f(x_t, h_{t-1})$$

où :

- $x_t$ est l’entrée au temps $t$ ;
    
- $h_{t-1}$ est l’état caché précédent ;
    
- $h_t$ est le nouvel état caché ;
    
-$f$est une fonction apprise par le réseau.
    

Cette formule dit simplement :

> Le nouvel état dépend à la fois de l’entrée actuelle et de la mémoire précédente.

Dans un RNN classique, on utilise souvent une formule de ce type :

$$h_t = \tanh(W_x x_t + W_h h_{t-1} + b)$$

où :

- $W_x$ est la matrice de poids appliquée à l’entrée actuelle ;
    
- $W_h$ est la matrice de poids appliquée à l’état précédent ;
    
- $b$ est un biais ;
    
- $\tanh$ est une fonction d’activation non linéaire.
    

Nous pouvons la lire ainsi :

```txt
nouvel état = activation(entrée transformée + mémoire transformée + biais)
```

---

## 3.5. Exemple intuitif avec une phrase

@TODO corriger les expressions tex

Prenons la phrase :

```txt
Le chat dort.
```

Nous pouvons noter :

```txt
x1 = "Le"
x2 = "chat"
x3 = "dort"
x4 = "."
```

Le RNN calcule successivement :

[  
h_1 = f(x_1, h_0)  
]

[  
h_2 = f(x_2, h_1)  
]

[  
h_3 = f(x_3, h_2)  
]

[  
h_4 = f(x_4, h_3)  
]

L’état initial (h_0) est souvent un vecteur de zéros ou un vecteur appris.

```mermaid
flowchart LR
    H0["h0 mémoire initiale"] --> R1["RNN"]
    X1["x1 = Le"] --> R1
    R1 --> H1["h1"]

    H1 --> R2["RNN"]
    X2["x2 = chat"] --> R2
    R2 --> H2["h2"]

    H2 --> R3["RNN"]
    X3["x3 = dort"] --> R3
    R3 --> H3["h3"]

    H3 --> R4["RNN"]
    X4["x4 = ."] --> R4
    R4 --> H4["h4"]
```

Chaque état contient donc une représentation partielle de la séquence lue jusqu’ici.

---

## 3.6. Ce que contient l’état caché

L’état caché $h_t$ est un vecteur.

Il ne contient pas les mots sous forme lisible.

Il contient une représentation numérique apprise.

Par exemple, après avoir lu :

```txt
Le chat
```

l’état caché peut contenir implicitement des informations comme :

- nous parlons probablement d’un animal ;
    
- le sujet est au singulier ;
    
- une action peut suivre ;
    
- le contexte grammatical est en cours de construction.
    

Évidemment, le modèle ne stocke pas ces informations sous forme de phrases humaines. Il les encode dans des nombres.

Nous pouvons représenter cela ainsi :

```mermaid
flowchart TD
    A["Tokens déjà lus : Le chat"] --> B["État caché h_t"]
    B --> C["Informations grammaticales"]
    B --> D["Informations sémantiques"]
    B --> E["Contexte partiel"]
```

L’état caché est donc une forme de mémoire compressée.

---

## 3.7. RNN déroulé dans le temps

Quand nous dessinons un RNN, nous pouvons le représenter de deux façons.

La première est compacte :

```mermaid
flowchart TD
    X["Entrée x_t"] --> R["Cellule RNN"]
    Hprev["h_(t-1)"] --> R
    R --> H["h_t"]
```

Mais pour comprendre son traitement d’une séquence, nous le **déroulons dans le temps**.

```mermaid
flowchart LR
    X1["x1"] --> R1["RNN"]
    H0["h0"] --> R1
    R1 --> H1["h1"]

    X2["x2"] --> R2["RNN"]
    H1 --> R2
    R2 --> H2["h2"]

    X3["x3"] --> R3["RNN"]
    H2 --> R3
    R3 --> H3["h3"]

    X4["x4"] --> R4["RNN"]
    H3 --> R4
    R4 --> H4["h4"]
```

Il faut bien comprendre que les cellules dessinées (R1), (R2), (R3), (R4) représentent généralement **la même cellule RNN réutilisée à chaque étape**.

Autrement dit, les poids sont partagés dans le temps.

---

## 3.8. Partage des poids dans le temps

Un RNN n’apprend pas une matrice différente pour chaque position.

Il utilise les mêmes poids à chaque étape.

Cela signifie que la même transformation est appliquée à :

```txt
x1 avec h0
x2 avec h1
x3 avec h2
x4 avec h3
```

```mermaid
flowchart LR
    A["Étape 1 : mêmes poids W"] --> B["Étape 2 : mêmes poids W"]
    B --> C["Étape 3 : mêmes poids W"]
    C --> D["Étape 4 : mêmes poids W"]
```

Ce partage des poids a deux avantages :

1. le modèle peut traiter des séquences de longueurs variables ;
    
2. le nombre de paramètres ne dépend pas directement de la longueur de la séquence.
    

C’est une idée élégante : nous apprenons une règle générale de mise à jour de la mémoire, puis nous l’appliquons autant de fois que nécessaire.

---

## 3.9. Dimensions dans un RNN

Supposons que chaque token soit représenté par un embedding de dimension :

[  
d_{input}  
]

et que l’état caché ait une dimension :

[  
d_{hidden}  
]

Alors :

[  
x_t \in \mathbb{R}^{d_{input}}  
]

[  
h_t \in \mathbb{R}^{d_{hidden}}  
]

La matrice $W_x$ transforme l’entrée vers l’espace caché :

[  
W_x \in \mathbb{R}^{d_{hidden} \times d_{input}}  
]

La matrice $W_h$ transforme l’état précédent vers le nouvel état caché :

[  
W_h \in \mathbb{R}^{d_{hidden} \times d_{hidden}}  
]

La formule complète est donc :

[  
h_t = \tanh(W_x x_t + W_h h_{t-1} + b)  
]

avec :

[  
b \in \mathbb{R}^{d_{hidden}}  
]

Nous devons retenir que l’état caché est un vecteur de taille fixe, même si la séquence est longue.

C’est un point qui deviendra important pour comprendre les limites des RNN.

---

## 3.10. Sortie à chaque étape ou sortie finale

Un RNN peut être utilisé de plusieurs manières.

### 3.10.1 Sortie finale uniquement

Pour une tâche de classification de phrase, nous pouvons utiliser uniquement le dernier état caché.

Exemple :

```txt
Phrase : Ce film est excellent.
Tâche : sentiment positif ou négatif
```

```mermaid
flowchart LR
    X1["Ce"] --> R1["RNN"]
    R1 --> R2["RNN"]
    X2["film"] --> R2
    R2 --> R3["RNN"]
    X3["est"] --> R3
    R3 --> R4["RNN"]
    X4["excellent"] --> R4
    R4 --> C["Classification positive/négative"]
```

Ici, le dernier état doit résumer toute la phrase.

---

### 3.10.2 Sortie à chaque étape

Pour une tâche d’étiquetage de séquence, nous pouvons produire une sortie à chaque token.

Exemple :

```txt
Phrase : Marie habite Paris.
Tâche : reconnaissance d'entités nommées
```

Nous voulons prédire :

```txt
Marie  → PERSONNE
habite → O
Paris  → LIEU
```

```mermaid
flowchart LR
    X1["Marie"] --> R1["RNN"]
    R1 --> Y1["PERSONNE"]
    R1 --> R2["RNN"]

    X2["habite"] --> R2
    R2 --> Y2["O"]
    R2 --> R3["RNN"]

    X3["Paris"] --> R3
    R3 --> Y3["LIEU"]
```

Dans ce cas, chaque état caché sert à produire une prédiction locale.

---

## 3.11. Les grandes configurations des RNN

Nous pouvons classer les usages des RNN selon la forme de l’entrée et de la sortie.

### 3.11.1 One-to-one

Un seul input donne un seul output.

Ce n’est pas vraiment le cas typique des RNN, mais cela correspond à un réseau classique.

```mermaid
flowchart LR
    A["Entrée"] --> B["Modèle"] --> C["Sortie"]
```

---

### 3.11.2 One-to-many

Une seule entrée produit une séquence.

Exemple : génération de légende d’image.

```mermaid
flowchart LR
    A["Image"] --> B["RNN Decoder"]
    B --> C["Un"]
    C --> D["chat"]
    D --> E["dort"]
```

---

### 3.11.3 Many-to-one

Une séquence produit une seule sortie.

Exemple : classification de sentiment.

```mermaid
flowchart LR
    A["Ce"] --> B["film"] --> C["est"] --> D["excellent"]
    D --> E["Sentiment positif"]
```

---

### 3.11.4 Many-to-many aligné

Une séquence produit une sortie à chaque position.

Exemple : étiquetage grammatical.

```mermaid
flowchart LR
    A["Le"] --> A2["DET"]
    B["chat"] --> B2["NOM"]
    C["dort"] --> C2["VERBE"]
```

---

### 3.11.5 Many-to-many non aligné

Une séquence produit une autre séquence de longueur différente.

Exemple : traduction automatique.

```mermaid
flowchart LR
    A["I love cats"] --> B["Encoder RNN"]
    B --> C["Decoder RNN"]
    C --> D["J'aime les chats"]
```

Cette dernière configuration a joué un rôle central avant les Transformers.

---

## 3.12. Le RNN comme mémoire compressée

Nous pouvons maintenant formuler une idée essentielle :

> Dans un RNN, l’état caché est une mémoire compressée de tous les tokens précédents.

Après avoir lu une longue phrase, le dernier état doit contenir les informations nécessaires à la tâche.

Prenons :

```txt
Le livre que Paul a acheté hier dans une petite librairie du centre-ville est passionnant.
```

Si nous voulons classifier cette phrase ou la traduire, le dernier état doit contenir beaucoup d’informations :

- le sujet principal : `Le livre` ;
    
- l’action secondaire : `Paul a acheté` ;
    
- le lieu : `librairie du centre-ville` ;
    
- le prédicat principal : `est passionnant`.
    

```mermaid
flowchart TD
    A["Phrase longue"] --> B["RNN"]
    B --> C["Dernier état caché"]
    C --> D["Résumé compressé de toute la phrase"]
```

Le problème est que cette mémoire a une taille fixe.

Même si la phrase contient 10 tokens, 100 tokens ou 1000 tokens, l’état caché garde la même dimension.

---

## 3.13. Pourquoi cette mémoire est limitée

La limite principale vient du fait que l’information est transmise étape après étape.

Si une information importante apparaît au début de la séquence, elle doit survivre à toutes les mises à jour successives.

```mermaid
flowchart LR
    A["Information importante au début"] --> B["Étape 1"]
    B --> C["Étape 2"]
    C --> D["Étape 3"]
    D --> E["..."]
    E --> F["Étape 50"]
    F --> G["Utilisation finale"]
```

À chaque étape, le modèle peut modifier, écraser ou diluer cette information.

C’est un peu comme si nous devions retenir une phrase longue tout en recevant constamment de nouveaux mots.

Certaines informations anciennes peuvent se perdre.

---

## 3.14. Exemple de dépendance longue

Prenons la phrase :

```txt
Les clés que mon frère a laissées hier dans la voiture de notre voisin sont sur la table.
```

Le verbe `sont` dépend du sujet `Les clés`.

Mais entre les deux, nous avons une longue proposition relative :

```txt
que mon frère a laissées hier dans la voiture de notre voisin
```

Pour comprendre correctement la phrase, le modèle doit conserver l’information :

```txt
sujet = Les clés
nombre = pluriel
```

jusqu’au mot :

```txt
sont
```

```mermaid
flowchart LR
    A["Les clés"] -. "information à conserver : pluriel" .-> G["sont"]
    B["mon frère"] --> C["a laissées"]
    C --> D["hier"]
    D --> E["dans la voiture"]
    E --> F["de notre voisin"]
    F --> G
```

Un RNN peut théoriquement apprendre cette dépendance.

Mais en pratique, plus la distance augmente, plus cela devient difficile.

---

## 3.15. Le problème du gradient

Pour apprendre, un réseau de neurones utilise la rétropropagation.

Dans un RNN, comme la même cellule est répétée dans le temps, nous devons rétropropager l’erreur à travers toutes les étapes.

C’est ce que nous appelons la **Backpropagation Through Time**, ou **BPTT**.

```mermaid
flowchart RL
    L["Erreur finale"] --> H5["h5"]
    H5 --> H4["h4"]
    H4 --> H3["h3"]
    H3 --> H2["h2"]
    H2 --> H1["h1"]
```

Le problème est que le gradient peut :

- devenir très petit ;
    
- devenir très grand.
    

Nous parlons alors de :

- **vanishing gradient** : gradient qui disparaît ;
    
- **exploding gradient** : gradient qui explose.
    

---

## 3.16. Vanishing gradient

Le **vanishing gradient** se produit quand le signal d’apprentissage devient de plus en plus faible en remontant dans le temps.

Conséquence : les premiers tokens reçoivent un signal de correction très faible.

```mermaid
flowchart RL
    A["Erreur finale"] --> B["Gradient fort"]
    B --> C["Gradient moyen"]
    C --> D["Gradient faible"]
    D --> E["Gradient presque nul"]
    E --> F["Début de séquence"]
```

Le modèle apprend donc difficilement que des éléments très anciens influencent la sortie finale.

Cela explique pourquoi les RNN classiques ont du mal avec les dépendances longues.

---

## 3.17. Exploding gradient

À l’inverse, le gradient peut aussi devenir très grand.

Dans ce cas, les poids du modèle peuvent être mis à jour de manière trop brutale.

Cela rend l’entraînement instable.

```mermaid
flowchart RL
    A["Erreur finale"] --> B["Gradient normal"]
    B --> C["Gradient grand"]
    C --> D["Gradient très grand"]
    D --> E["Explosion numérique"]
```

Une technique classique pour limiter ce problème est le **gradient clipping**.

L’idée est de limiter la norme du gradient pour éviter des mises à jour trop importantes.

---

## 3.18. Pourquoi utiliser une fonction tanh ?

Dans un RNN classique, nous utilisons souvent :

$$h_t = \tanh(W_x x_t + W_h h_{t-1} + b)$$

La fonction $\tanh$ renvoie des valeurs entre (-1) et (1).

Cela permet de garder l’état caché dans une plage contrôlée.

```mermaid
flowchart LR
    A["Combinaison linéaire"] --> B["tanh"]
    B --> C["Valeurs entre -1 et 1"]
```

Mais cette saturation peut aussi contribuer au vanishing gradient.

Quand $\tanh$ est saturée, sa dérivée devient très faible.

Donc le signal d’apprentissage peut diminuer rapidement.

---

## 3.19. RNN bidirectionnels

Pour certaines tâches, il est utile de connaître à la fois le contexte gauche et le contexte droit.

Par exemple, dans la phrase :

```txt
Il mange une pomme verte.
```

Pour comprendre `pomme`, il est utile de savoir ce qui vient avant :

```txt
Il mange une
```

mais aussi ce qui vient après :

```txt
verte
```

Les **RNN bidirectionnels** lisent donc la séquence dans les deux sens :

- un RNN de gauche à droite ;
    
- un RNN de droite à gauche.
    

```mermaid
flowchart LR
    A["Token 1"] --> B["Token 2"] --> C["Token 3"] --> D["Token 4"]
    D --> E["RNN backward"]
    C --> E
    B --> E
    A --> E

    A --> F["RNN forward"]
    B --> F
    C --> F
    D --> F
```

Une représentation finale combine alors les deux directions.

```mermaid
flowchart TD
    A["Contexte gauche → droite"] --> C["Représentation du token"]
    B["Contexte droite → gauche"] --> C
```

Les RNN bidirectionnels sont utiles pour des tâches de compréhension, mais ils ne conviennent pas directement à la génération autoregressive, car dans la génération nous ne devons pas regarder les tokens futurs.

---

## 3.20. RNN profonds

Nous pouvons aussi empiler plusieurs couches RNN.

La sortie d’une couche devient l’entrée de la couche suivante.

```mermaid
flowchart TD
    X["Séquence d'entrée"] --> R1["Couche RNN 1"]
    R1 --> R2["Couche RNN 2"]
    R2 --> R3["Couche RNN 3"]
    R3 --> Y["Sortie"]
```

Cela augmente la capacité du modèle.

La première couche peut apprendre des motifs simples.

Les couches supérieures peuvent apprendre des représentations plus abstraites.

Mais cela rend aussi l’entraînement plus difficile, notamment à cause des gradients et du coût séquentiel.

---

## 3.21. RNN pour la génération de texte

Un RNN peut être utilisé pour générer du texte.

L’idée est de prédire le prochain token à partir des tokens précédents.

Par exemple :

```txt
Le chat
```

Le modèle peut prédire :

```txt
dort
```

Puis il réinjecte ce token dans le modèle pour prédire le suivant :

```txt
Le chat dort
```

```mermaid
flowchart LR
    A["Le"] --> R1["RNN"]
    R1 --> B["prédit : chat"]
    B --> R2["RNN"]
    R2 --> C["prédit : dort"]
    C --> R3["RNN"]
    R3 --> D["prédit : ."]
```

Cette logique autoregressive sera aussi présente dans les modèles GPT, mais avec une architecture Transformer decoder-only.

---

## 3.22. Entraînement d’un RNN génératif

Pendant l’entraînement, nous donnons au modèle une séquence réelle et nous lui demandons de prédire le token suivant.

Pour la phrase :

```txt
Le chat dort.
```

Nous créons les couples :

```txt
Entrée : Le       → cible : chat
Entrée : Le chat  → cible : dort
Entrée : Le chat dort → cible : .
```

En pratique, le modèle prédit à chaque position le token suivant.

```mermaid
flowchart TD
    A["Entrée : Le chat dort"] --> B["RNN"]
    B --> C["Prédictions à chaque position"]
    C --> D["Cibles : chat dort ."]
    D --> E["Calcul de la loss"]
```

Cette idée de prédiction du prochain token reste centrale dans les grands modèles de langage modernes.

---

## 3.23. RNN Seq2Seq

Les RNN ont aussi été très utilisés dans les architectures **Seq2Seq**.

Une architecture Seq2Seq comporte deux parties :

1. un encodeur ;
    
2. un décodeur.
    

L’encodeur lit la séquence source.

Le décodeur produit la séquence cible.

Exemple :

```txt
Source : I love cats.
Cible  : J'aime les chats.
```

```mermaid
flowchart LR
    A["I"] --> E1["Encoder RNN"]
    E1 --> E2["Encoder RNN"]
    B["love"] --> E2
    E2 --> E3["Encoder RNN"]
    C["cats"] --> E3
    E3 --> H["Vecteur de contexte"]

    H --> D1["Decoder RNN"]
    D1 --> Y1["J'"]
    D1 --> D2["Decoder RNN"]
    D2 --> Y2["aime"]
    D2 --> D3["Decoder RNN"]
    D3 --> Y3["les chats"]
```

Le vecteur de contexte sert de résumé de la phrase source.

C’est précisément ce résumé unique qui deviendra une limite importante.

---

## 3.24. Limite du vecteur de contexte

Dans un Seq2Seq RNN classique, l’encodeur compresse toute la phrase source dans un seul vecteur.

Pour une phrase courte, cela peut fonctionner :

```txt
I love cats.
```

Mais pour une phrase longue, cela devient difficile :

```txt
Although the committee initially rejected the proposal, it later accepted a revised version after several months of discussion.
```

```mermaid
flowchart LR
    A["Phrase source longue"] --> B["Encoder RNN"]
    B --> C["Vecteur unique"]
    C --> D["Decoder RNN"]
    D --> E["Traduction"]
```

Le décodeur doit produire toute la traduction à partir d’un seul résumé.

Nous avons donc un goulot d’étranglement informationnel.

L’attention est apparue en partie pour résoudre ce problème.

---

## 3.25. Comparaison avec un modèle non récurrent

Pour mieux comprendre le rôle du RNN, comparons deux approches.

### 3.25.1 Modèle sans mémoire

Si nous appliquons un réseau indépendamment à chaque token, le modèle voit seulement le token actuel.

```mermaid
flowchart TD
    A["Token actuel"] --> B["Réseau dense"]
    B --> C["Sortie"]
```

Il ne sait pas ce qui a été lu avant.

### 3.25.2 Modèle récurrent

Un RNN reçoit le token actuel et l’état précédent.

```mermaid
flowchart TD
    A["Token actuel"] --> C["RNN"]
    B["Mémoire précédente"] --> C
    C --> D["Nouvelle mémoire"]
```

Il peut donc accumuler de l’information.

La récurrence est précisément ce mécanisme de réutilisation de l’état précédent.

---

## 3.26. Ce que les RNN ont apporté

Les RNN ont été fondamentaux dans l’histoire du deep learning.

Ils ont permis de traiter des données séquentielles comme :

- le texte ;
    
- la parole ;
    
- les séries temporelles ;
    
- les signaux ;
    
- la musique ;
    
- les logs ;
    
- les séquences biologiques.
    

Ils ont introduit une idée essentielle :

> Une donnée n’est pas toujours indépendante : son interprétation dépend de ce qui précède.

Cette idée reste fondamentale dans les Transformers.

La différence est que les Transformers ne transportent pas l’information de proche en proche de la même façon.

---

## 3.27. Ce que les RNN ne résolvent pas bien

Les RNN classiques ont trois limites majeures.

### 3.27.1 Dépendances longues

Ils ont du mal à conserver des informations importantes pendant de nombreuses étapes.

### 3.27.2 Faible parallélisation

Le calcul de $h_t$ dépend de $h_{t-1}$.

Donc nous ne pouvons pas facilement calculer toutes les positions en parallèle.

```mermaid
flowchart LR
    H1["h1"] --> H2["h2"] --> H3["h3"] --> H4["h4"]
```

### 3.27.3 Compression excessive

Dans les modèles Seq2Seq classiques, toute la séquence source peut devoir être compressée dans un seul vecteur.

Ces trois limites ouvriront la voie :

- aux LSTM et GRU ;
    
- aux mécanismes d’attention ;
    
- puis aux Transformers.
    

---

## 3.28. L’idée clé à retenir

Nous pouvons résumer le principe des RNN ainsi :

> Un RNN lit une séquence pas à pas et met à jour une mémoire interne appelée état caché.

Cette mémoire permet au modèle de tenir compte du contexte précédent.

Mais cette mémoire est :

- compressée ;
    
- mise à jour à chaque étape ;
    
- difficile à préserver sur de longues distances ;
    
- coûteuse à calculer séquentiellement.
    

C’est pourquoi les RNN sont élégants, mais limités pour les très grands modèles de langage.

---

## 3.29. Schéma de synthèse

```mermaid
flowchart TD
    A["Entrée actuelle x_t"] --> C["Cellule RNN"]
    B["État précédent h_(t-1)"] --> C

    C --> D["Nouvel état h_t"]
    D --> E["Mémoire mise à jour"]

    E --> F["Étape suivante"]
    F --> G["Traitement séquentiel"]

    G --> H["Avantage : contexte précédent"]
    G --> I["Limite : dépendances longues"]
    G --> J["Limite : faible parallélisation"]
    G --> K["Limite : mémoire compressée"]
```

---

## 3.30. Questions de compréhension

### Question 1

Quelles sont les deux entrées principales d’une cellule RNN à l’étape $t$ ?

Réponse attendue : l’entrée actuelle $x_t$ et l’état caché précédent $h_{t-1}$.

### Question 2

Que représente l’état caché $h_t$ ?

Réponse attendue : il représente une mémoire numérique de la séquence lue jusqu’à l’étape $t$.

### Question 3

Pourquoi dit-on que les poids d’un RNN sont partagés dans le temps ?

Réponse attendue : parce que la même cellule, avec les mêmes matrices de poids, est appliquée à chaque étape de la séquence.

### Question 4

Quelle est la formule simple d’un RNN ?

Réponse attendue :

$$h_t = f(x_t, h_{t-1})$$

ou, dans une version classique :

$$h_t = \tanh(W_x x_t + W_h h_{t-1} + b)$$
### Question 5

Pourquoi les RNN peuvent-ils traiter des séquences de longueurs variables ?

Réponse attendue : parce que la même cellule peut être appliquée autant de fois qu’il y a de tokens dans la séquence.

### Question 6

Pourquoi les RNN ont-ils du mal avec les dépendances longues ?

Réponse attendue : parce que l’information doit traverser de nombreuses étapes successives, ce qui peut entraîner une perte d’information et un affaiblissement du gradient.

### Question 7

Qu’est-ce que la Backpropagation Through Time ?

Réponse attendue : c’est la rétropropagation du gradient à travers les différentes étapes temporelles d’un RNN déroulé.

### Question 8

Quelle différence faisons-nous entre une sortie finale et une sortie à chaque étape ?

Réponse attendue : une sortie finale est utilisée quand toute la séquence produit une seule prédiction, tandis qu’une sortie à chaque étape est utilisée quand chaque token doit recevoir une prédiction.

---

## 3.31. Transition vers le chapitre suivant

Nous savons maintenant comment fonctionne un RNN classique.

Nous avons compris que son état caché joue le rôle d’une mémoire.

Mais nous avons aussi vu que cette mémoire est limitée, surtout lorsque les dépendances sont longues.

Dans le chapitre suivant, nous allons donc étudier plus précisément le problème des dépendances longues.

Nous verrons pourquoi une information située au début d’une séquence peut être difficile à utiliser beaucoup plus tard, et pourquoi cette difficulté a motivé l’apparition des LSTM, des GRU, puis de l’attention.


---

# Chapitre 4 — Le problème des dépendances longues

## 4.1. Objectif du chapitre

Dans le chapitre précédent, nous avons étudié le principe des **RNN**.

Nous avons vu qu’un RNN lit une séquence étape par étape, en mettant à jour un **état caché** :

[  
h_t = f(x_t, h_{t-1})  
]

Cette idée est puissante, car le modèle possède une forme de mémoire.

Mais cette mémoire pose un problème important :

> Plus une information doit être conservée longtemps, plus elle risque d’être perdue, déformée ou écrasée.

Dans ce chapitre, nous allons donc comprendre le problème des **dépendances longues**.

Nous verrons pourquoi ce problème est central dans le traitement du langage naturel, pourquoi les RNN classiques y sont particulièrement sensibles, et pourquoi cette difficulté a motivé l’apparition des LSTM, des GRU, puis des mécanismes d’attention.

---

## 4.2. Une phrase simple : dépendance courte

Commençons par une phrase très simple :

```txt
Le chat dort.
```

Ici, la relation entre le sujet et le verbe est courte :

```txt
Le chat → dort
```

Le modèle doit comprendre que :

- `Le chat` est le sujet ;
    
- `dort` est le verbe ;
    
- l’action de dormir concerne le chat.
    

```mermaid
flowchart LR
    A["Le chat"] --> B["dort"]
```

Dans ce cas, un RNN classique peut souvent gérer correctement la relation.

L’information importante n’a pas besoin de traverser beaucoup d’étapes.

---

## 4.3. Une phrase plus longue : dépendance longue

Prenons maintenant une phrase plus complexe :

```txt
Le livre que Paul a acheté hier dans une petite librairie du centre-ville est passionnant.
```

Le sujet principal est :

```txt
Le livre
```

Le verbe associé est :

```txt
est
```

Mais entre les deux, nous avons beaucoup d’informations intermédiaires :

```txt
que Paul a acheté hier dans une petite librairie du centre-ville
```

Le modèle doit comprendre que :

```txt
Le livre ... est passionnant.
```

et non :

```txt
Paul ... est passionnant.
la librairie ... est passionnant.
le centre-ville ... est passionnant.
```

Le problème est que les RNN doivent transporter l’information importante à travers plusieurs étapes successives.

```mermaid
flowchart LR
    A["Le livre"] --> B["que"]
    B --> C["Paul"]
    C --> D["a acheté"]
    D --> E["hier"]
    E --> F["dans une librairie"]
    F --> G["du centre-ville"]
    G --> H["est passionnant"]

    A -. "dépendance longue" .-> H
```

Plus la séquence est longue, plus il devient difficile de conserver les informations importantes.

C’est ce que nous appelons le problème des **dépendances longues**.

---

## 4.4. Qu’est-ce qu’une dépendance longue ?

Une **dépendance longue** apparaît lorsqu’un élément d’une séquence doit être relié à un autre élément éloigné.

Dans le langage naturel, cela arrive très souvent.

Par exemple :

```txt
Les clés que mon frère a laissées dans la voiture sont sur la table.
```

Le sujet principal est :

```txt
Les clés
```

Le verbe principal est :

```txt
sont
```

Le modèle doit donc garder en mémoire que le sujet est pluriel.

```mermaid
flowchart LR
    A["Les clés"] -. "pluriel" .-> F["sont"]
    B["mon frère"] --> C["a laissées"]
    C --> D["dans la voiture"]
    D --> F
```

Si le modèle se laisse influencer par `mon frère`, il pourrait produire une mauvaise analyse grammaticale.

Il doit donc être capable de distinguer :

- l’information principale ;
    
- les informations secondaires ;
    
- les groupes syntaxiques imbriqués ;
    
- les relations éloignées mais importantes.
    

---

## 4.5. Pourquoi les dépendances longues sont importantes ?

Les dépendances longues ne sont pas un détail.

Elles sont essentielles pour comprendre correctement :

- la grammaire ;
    
- le sens d’une phrase ;
    
- les références pronominales ;
    
- les accords ;
    
- la structure logique ;
    
- les relations de cause à effet ;
    
- les textes longs ;
    
- les dialogues ;
    
- les programmes informatiques.
    

Prenons un exemple avec un pronom :

```txt
Marie a donné son livre à Julie parce qu’elle partait en voyage.
```

Le pronom :

```txt
elle
```

peut théoriquement désigner :

```txt
Marie
```

ou :

```txt
Julie
```

Pour interpréter correctement la phrase, le modèle doit utiliser le contexte.

```mermaid
flowchart TD
    A["Marie"] --> C["elle ?"]
    B["Julie"] --> C
    C --> D["Résolution de référence selon le contexte"]
```

La compréhension du langage dépend donc fortement de la capacité à relier des éléments parfois très éloignés.

---

## 4.6. Le RNN face à une dépendance longue

Dans un RNN, l’information est transmise de proche en proche.

Si une information apparaît au temps (t = 1), mais qu’elle est utile au temps (t = 20), elle doit passer par tous les états intermédiaires :

[  
h_1 \rightarrow h_2 \rightarrow h_3 \rightarrow \dots \rightarrow h_{20}  
]

```mermaid
flowchart LR
    H1["h1 : information initiale"] --> H2["h2"]
    H2 --> H3["h3"]
    H3 --> H4["h4"]
    H4 --> H5["..."]
    H5 --> H20["h20 : information utilisée"]
```

À chaque étape, l’état caché est recalculé :

[  
h_t = f(x_t, h_{t-1})  
]

Cela signifie que l’information ancienne est constamment mélangée avec une nouvelle entrée.

Le modèle doit apprendre à préserver ce qui est important et à oublier ce qui ne l’est pas.

Mais dans un RNN classique, ce contrôle est assez faible.

---

## 4.7. La mémoire cachée comme résumé compressé

L’état caché d’un RNN a une taille fixe.

Par exemple, il peut avoir une dimension de 256, 512 ou 1024.

Mais la phrase peut être courte ou très longue.

Cela signifie qu’un même vecteur doit parfois résumer beaucoup d’informations.

```mermaid
flowchart TD
    A["Séquence courte"] --> C["État caché de taille fixe"]
    B["Séquence longue"] --> C
    C --> D["Résumé compressé"]
```

Pour une phrase courte, ce résumé peut suffire.

Pour une phrase longue, il devient difficile de tout conserver.

C’est comme essayer de résumer un roman entier dans une seule phrase : certaines informations seront forcément perdues.

---

## 4.8. Exemple détaillé : sujet et verbe éloignés

Reprenons la phrase :

```txt
Le livre que Paul a acheté hier dans une petite librairie du centre-ville est passionnant.
```

Nous pouvons découper la phrase ainsi :

```txt
[Le livre] [que Paul a acheté hier dans une petite librairie du centre-ville] [est passionnant]
```

La structure principale est :

```txt
Le livre est passionnant.
```

Mais le RNN lit la phrase linéairement :

```txt
Le → livre → que → Paul → a → acheté → hier → dans → une → petite → librairie → du → centre-ville → est → passionnant
```

Quand il arrive à `est`, il doit encore avoir conservé l’information :

```txt
Le livre = sujet principal
```

```mermaid
flowchart LR
    A["Le"] --> B["livre"]
    B --> C["que"]
    C --> D["Paul"]
    D --> E["a acheté"]
    E --> F["hier"]
    F --> G["dans une petite librairie"]
    G --> H["du centre-ville"]
    H --> I["est"]
    I --> J["passionnant"]

    B -. "information sujet à conserver" .-> I
```

Mais plusieurs informations concurrentes sont apparues entre-temps :

- `Paul` ;
    
- `hier` ;
    
- `librairie` ;
    
- `centre-ville`.
    

Le modèle doit donc ne pas confondre le sujet principal avec les éléments intermédiaires.

---

## 4.9. Dépendance longue et accord grammatical

Les dépendances longues sont particulièrement visibles dans les accords.

Exemple :

```txt
Les propositions que le directeur a présentées pendant la réunion sont intéressantes.
```

Le sujet est :

```txt
Les propositions
```

Le verbe est :

```txt
sont
```

Le modèle doit comprendre que le verbe doit être au pluriel.

Pourtant, plusieurs mots singuliers apparaissent entre les deux :

```txt
le directeur
la réunion
```

```mermaid
flowchart LR
    A["Les propositions"] -. "pluriel" .-> F["sont"]
    B["le directeur"] --> C["a présentées"]
    C --> D["pendant"]
    D --> E["la réunion"]
    E --> F
```

Un modèle faible pourrait être attiré par le nom le plus proche, par exemple `réunion`, et perdre la structure grammaticale globale.

---

## 4.10. Dépendance longue et référence pronominale

Les dépendances longues apparaissent aussi avec les pronoms.

Exemple :

```txt
Thomas a rangé le dossier dans l’armoire avant de partir, mais il ne l’a pas fermé correctement.
```

Le pronom :

```txt
l’
```

renvoie probablement à :

```txt
le dossier
```

Mais plusieurs mots sont apparus entre les deux.

```mermaid
flowchart LR
    A["le dossier"] -. "référence" .-> F["l’a"]
    B["l’armoire"] --> C["avant de partir"]
    C --> D["il"]
    D --> F
```

Pour comprendre le texte, le modèle doit résoudre cette référence.

Cela demande une mémoire du contexte, mais aussi une capacité à distinguer les entités mentionnées.

---

## 4.11. Dépendance longue et logique du discours

Dans un texte plus long, une information introduite au début peut être nécessaire beaucoup plus tard.

Exemple :

```txt
Au début du chapitre, nous avons défini la notion d’état caché. Nous allons maintenant expliquer pourquoi cette notion devient problématique lorsque les séquences deviennent longues.
```

Pour comprendre `cette notion`, le modèle doit relier l’expression à :

```txt
la notion d’état caché
```

```mermaid
flowchart LR
    A["notion d’état caché"] -. "référence longue" .-> B["cette notion"]
```

Dans un dialogue, un rapport, un article scientifique ou un document juridique, les dépendances peuvent s’étendre sur plusieurs paragraphes.

C’est une difficulté majeure pour les modèles séquentiels classiques.

---

## 4.12. Pourquoi les RNN oublient-ils ?

Un RNN n’oublie pas volontairement comme un humain.

Mais à chaque étape, il recalcule son état :

[  
h_t = \tanh(W_x x_t + W_h h_{t-1} + b)  
]

Cela signifie que l’ancien état (h_{t-1}) est transformé puis mélangé avec la nouvelle entrée (x_t).

Si une information ancienne n’est pas renforcée ou protégée, elle peut être progressivement diluée.

```mermaid
flowchart TD
    A["Information ancienne"] --> B["Mélange avec nouveau token"]
    B --> C["Nouvel état"]
    C --> D["Mélange avec token suivant"]
    D --> E["Information diluée"]
```

Cette dilution est une des causes pratiques de la difficulté à conserver des dépendances longues.

---

## 4.13. Le lien avec le gradient qui disparaît

Le problème des dépendances longues est aussi lié au problème du **gradient qui disparaît**.

Pendant l’entraînement, si une erreur à la fin de la séquence dépend d’un mot au début, le gradient doit remonter sur beaucoup d’étapes.

```mermaid
flowchart RL
    A["Erreur à la fin"] --> B["h_t"]
    B --> C["h_(t-1)"]
    C --> D["h_(t-2)"]
    D --> E["..."]
    E --> F["h_1"]
```

À chaque étape, le gradient peut être multiplié par des valeurs qui le réduisent.

Résultat :

> Le début de la séquence reçoit un signal d’apprentissage très faible.

Donc le modèle apprend difficilement que les premiers mots peuvent être importants pour une décision prise beaucoup plus tard.

---

## 4.14. Exemple pédagogique du gradient

Imaginons que le modèle fasse une erreur sur le verbe :

```txt
Les clés ... est sur la table.
```

au lieu de :

```txt
Les clés ... sont sur la table.
```

L’erreur apparaît au moment de prédire :

```txt
sont
```

Mais pour corriger cette erreur, le modèle doit comprendre que l’information importante était au début :

```txt
Les clés
```

```mermaid
flowchart RL
    A["Erreur : mauvais accord sur sont"] --> B["Position du verbe"]
    B --> C["Mots intermédiaires"]
    C --> D["Sujet : Les clés"]
```

Si le gradient ne remonte pas correctement jusqu’au début, le modèle ne corrige pas bien son comportement.

---

## 4.15. Une difficulté mathématique simple

Dans une chaîne de calculs répétitifs, les gradients sont multipliés plusieurs fois.

Si nous multiplions plusieurs fois par un nombre inférieur à 1, le résultat devient très petit.

Par exemple :

[  
0.5^{10} = 0.0009765625  
]

[  
0.5^{20} = 0.0000009536743164  
]

Cela illustre intuitivement pourquoi un signal peut disparaître quand il traverse beaucoup d’étapes.

Inversement, si nous multiplions plusieurs fois par un nombre supérieur à 1, le signal peut exploser.

Par exemple :

[  
2^{10} = 1024  
]

[  
2^{20} = 1,048,576  
]

Dans les RNN, les dépendances temporelles longues entraînent ce type de phénomènes lors de la rétropropagation.

---

## 4.16. Dépendance longue et coût séquentiel

Le problème n’est pas seulement la mémoire.

Il est aussi computationnel.

Dans un RNN, pour atteindre le token numéro 100, nous devons avoir calculé les 99 états précédents.

```mermaid
flowchart LR
    H1["h1"] --> H2["h2"]
    H2 --> H3["h3"]
    H3 --> H4["..."]
    H4 --> H100["h100"]
```

Nous ne pouvons pas facilement calculer (h_{100}) directement.

Cela limite la parallélisation.

Donc les RNN souffrent de deux difficultés liées :

1. ils transportent difficilement l’information sur de longues distances ;
    
2. ils calculent les états dans un ordre strictement séquentiel.
    

Ces deux points seront précisément remis en question par les Transformers.

---

## 4.17. Les LSTM et GRU : une réponse partielle

Les **LSTM** et les **GRU** ont été conçus pour mieux gérer les dépendances longues.

Ils introduisent des mécanismes de portes.

Ces portes permettent au modèle de décider :

- ce qu’il faut oublier ;
    
- ce qu’il faut conserver ;
    
- ce qu’il faut ajouter ;
    
- ce qu’il faut transmettre.
    

```mermaid
flowchart TD
    A["Entrée actuelle"] --> B["Cellule LSTM / GRU"]
    C["Mémoire précédente"] --> B

    B --> D["Porte d'oubli"]
    B --> E["Porte de mise à jour"]
    B --> F["Nouvelle mémoire"]
```

L’idée est de protéger certaines informations importantes contre l’effacement progressif.

Par exemple, si le modèle lit :

```txt
Les clés ...
```

il peut apprendre à conserver l’information :

```txt
sujet pluriel
```

jusqu’au verbe.

Mais même les LSTM et GRU restent fondamentalement séquentiels.

Ils améliorent la mémoire, mais ne suppriment pas complètement le problème.

---

## 4.18. L’attention : une autre manière de résoudre le problème

L’attention propose une idée différente.

Au lieu de forcer le modèle à transporter toute l’information de proche en proche, nous lui permettons de regarder directement les éléments utiles.

Dans une phrase comme :

```txt
Le livre que Paul a acheté hier dans une petite librairie du centre-ville est passionnant.
```

le mot `est` pourrait directement regarder `Le livre`.

```mermaid
flowchart LR
    A["Le livre"] -. "attention directe" .-> H["est passionnant"]
    B["Paul"] --> C["a acheté"]
    C --> D["hier"]
    D --> E["dans une librairie"]
    E --> F["du centre-ville"]
```

C’est une rupture importante.

Nous ne dépendons plus uniquement d’une mémoire transmise étape par étape.

Nous créons des connexions directes entre les positions pertinentes.

---

## 4.19. Comparaison : RNN contre attention

Dans un RNN, l’information doit suivre un chemin long :

```mermaid
flowchart LR
    A["Token important"] --> B["h1"]
    B --> C["h2"]
    C --> D["h3"]
    D --> E["..."]
    E --> F["Token qui en a besoin"]
```

Avec l’attention, le chemin peut être direct :

```mermaid
flowchart LR
    A["Token important"] -. "connexion directe" .-> F["Token qui en a besoin"]
```

Cette différence est fondamentale.

Le Transformer utilisera précisément cette idée :

> Chaque token peut regarder directement les autres tokens de la séquence.

Cela ne signifie pas que tout est résolu, mais cela rend les dépendances longues plus accessibles.

---

## 4.20. Exemple avec une matrice d’attention

Imaginons une phrase simplifiée :

```txt
Le livre est passionnant.
```

Nous pouvons représenter les relations entre tokens sous forme de matrice.

Chaque ligne correspond au token qui regarde.

Chaque colonne correspond au token regardé.

```txt
                 Le   livre   est   passionnant
Le              0.2    0.5   0.1       0.2
livre           0.3    0.4   0.1       0.2
est             0.1    0.7   0.1       0.1
passionnant     0.1    0.6   0.2       0.1
```

Ici, nous pouvons imaginer que `est` accorde beaucoup d’attention à `livre`.

```mermaid
flowchart LR
    A["est"] -. "poids fort" .-> B["livre"]
    C["passionnant"] -. "poids fort" .-> B
```

L’attention permet donc au modèle de construire des relations explicites entre positions, même éloignées.

---

## 4.21. Les dépendances longues dans le code informatique

Les dépendances longues ne concernent pas seulement le langage naturel.

Elles sont aussi très importantes dans le code.

Exemple :

```js
function calculerTotal(prix, quantite) {
    const total = prix * quantite;

    // beaucoup de lignes intermédiaires

    return total;
}
```

Le `return total` dépend de la déclaration :

```js
const total = prix * quantite;
```

qui peut être située plusieurs lignes avant.

```mermaid
flowchart LR
    A["const total = ..."] -. "référence variable" .-> B["return total"]
```

Pour comprendre ou générer du code, un modèle doit pouvoir relier :

- une variable à sa déclaration ;
    
- une fonction à ses appels ;
    
- une classe à ses méthodes ;
    
- une parenthèse ouvrante à sa parenthèse fermante ;
    
- une condition à son bloc associé.
    

Cela explique pourquoi les dépendances longues sont aussi centrales dans les modèles de génération de code.

---

## 4.22. Les dépendances longues dans les séries temporelles

Dans les séries temporelles, un événement ancien peut influencer un événement futur.

Exemple :

```txt
Une anomalie faible apparaît au début d’un signal, puis une panne se produit beaucoup plus tard.
```

Le modèle doit pouvoir relier :

```txt
anomalie faible
```

à :

```txt
panne future
```

```mermaid
flowchart LR
    A["t = 1 : anomalie faible"] -. "influence retardée" .-> B["t = 100 : panne"]
```

Les RNN ont longtemps été utilisés pour ce type de données, mais les architectures attentionnelles sont devenues très intéressantes lorsque les dépendances temporelles sont longues.

---

## 4.23. Les dépendances longues dans un document

Dans un document complet, une information peut être introduite très tôt, puis réutilisée beaucoup plus tard.

Exemple :

```txt
Au début du rapport, nous définissons une politique de confidentialité stricte.
...
Plus loin, nous analysons les conséquences de cette politique sur le traitement des données utilisateurs.
```

L’expression :

```txt
cette politique
```

renvoie à une information précédente.

```mermaid
flowchart TD
    A["Définition initiale"] --> B["Plusieurs paragraphes intermédiaires"]
    B --> C["Référence ultérieure : cette politique"]
    A -. "dépendance documentaire longue" .-> C
```

C’est un problème majeur pour les systèmes de question-réponse, de résumé automatique et de RAG.

---

## 4.24. Pourquoi ce problème prépare les Transformers

Nous pouvons maintenant comprendre pourquoi les Transformers sont apparus.

Les RNN imposent un chemin séquentiel :

```txt
token 1 → token 2 → token 3 → ... → token n
```

Les Transformers proposent une approche différente :

```txt
chaque token peut interagir avec chaque autre token
```

```mermaid
flowchart TD
    A["RNN"] --> B["Information transmise étape par étape"]
    B --> C["Difficulté avec les dépendances longues"]

    D["Transformer"] --> E["Attention entre tous les tokens"]
    E --> F["Accès plus direct aux relations éloignées"]
```

Le problème des dépendances longues est donc l’une des motivations majeures de l’attention et des Transformers.

---

## 4.25. Attention : les Transformers ne sont pas magiques

Il faut toutefois rester prudent.

Les Transformers facilitent les dépendances longues, mais ils ne les résolvent pas parfaitement.

Ils ont leurs propres limites :

- coût quadratique de l’attention ;
    
- longueur de contexte limitée ;
    
- difficulté à hiérarchiser les informations très longues ;
    
- risque de se concentrer sur des corrélations superficielles ;
    
- besoin de beaucoup de données.
    

Mais ils suppriment une contrainte majeure des RNN :

> l’obligation de transmettre toute l’information uniquement de proche en proche.

C’est ce qui explique leur succès dans de nombreuses tâches.

---

## 4.26. Résumé du chapitre

Dans ce chapitre, nous avons étudié le problème des **dépendances longues**.

Nous avons vu qu’une dépendance longue apparaît lorsqu’un élément d’une séquence doit être relié à un autre élément éloigné.

Dans le langage naturel, cela se produit souvent avec :

- les accords sujet-verbe ;
    
- les pronoms ;
    
- les propositions relatives ;
    
- les références dans un texte ;
    
- les relations logiques entre phrases.
    

Les RNN ont du mal avec ces situations, car ils transmettent l’information étape par étape à travers un état caché de taille fixe.

Plus la distance augmente, plus l’information risque d’être diluée, écrasée ou mal transmise.

Ce problème est lié à deux limites majeures :

- la mémoire compressée ;
    
- le gradient qui disparaît.
    

Les LSTM et GRU améliorent la situation grâce à des mécanismes de portes, mais ils restent séquentiels.

L’attention propose une autre solution : permettre à un token de regarder directement les tokens pertinents, même s’ils sont éloignés.

C’est cette idée qui préparera l’architecture Transformer.

---

## 4.27. Schéma de synthèse

```mermaid
flowchart TD
    A["Séquence longue"] --> B["Information importante au début"]
    B --> C["RNN : transmission étape par étape"]
    C --> D["Risque de dilution"]
    C --> E["Gradient qui disparaît"]
    C --> F["Difficulté à apprendre"]

    D --> G["Dépendance longue mal capturée"]
    E --> G
    F --> G

    G --> H["LSTM / GRU"]
    H --> I["Meilleure mémoire mais toujours séquentielle"]

    G --> J["Attention"]
    J --> K["Connexion directe entre tokens éloignés"]

    K --> L["Transformer"]
```

---

## 4.28. Questions de compréhension

### Question 1

Qu’est-ce qu’une dépendance longue ?

Réponse attendue : c’est une relation entre deux éléments éloignés dans une séquence, par exemple un sujet au début d’une phrase et son verbe beaucoup plus loin.

### Question 2

Pourquoi les RNN ont-ils du mal avec les dépendances longues ?

Réponse attendue : parce que l’information doit être transportée étape par étape dans l’état caché, ce qui peut la diluer ou l’effacer progressivement.

### Question 3

Pourquoi l’état caché d’un RNN est-il une mémoire compressée ?

Réponse attendue : parce qu’il doit résumer la séquence déjà lue dans un vecteur de taille fixe.

### Question 4

Quel est le lien entre dépendances longues et vanishing gradient ?

Réponse attendue : pour apprendre une dépendance longue, le gradient doit remonter sur beaucoup d’étapes ; il peut devenir très faible avant d’atteindre les premiers tokens.

### Question 5

Comment les LSTM et GRU améliorent-ils les RNN classiques ?

Réponse attendue : ils ajoutent des mécanismes de portes qui permettent de mieux contrôler ce qui est conservé, oublié ou mis à jour dans la mémoire.

### Question 6

Pourquoi l’attention est-elle intéressante pour les dépendances longues ?

Réponse attendue : parce qu’elle permet à un token de regarder directement un autre token pertinent, même s’il est éloigné dans la séquence.

### Question 7

Pourquoi les Transformers ne résolvent-ils pas tout ?

Réponse attendue : parce que l’attention complète coûte cher en mémoire et en calcul, notamment pour les longues séquences, et parce que la compréhension de très longs contextes reste difficile.

---

## 4.29. Transition vers le chapitre suivant

Nous avons maintenant compris pourquoi les dépendances longues posent problème aux RNN classiques.

Dans le chapitre suivant, nous allons étudier plus précisément le **problème du gradient qui disparaît**.

Nous verrons pourquoi l’apprentissage devient difficile lorsque le signal d’erreur doit traverser de nombreuses étapes, et pourquoi cette difficulté a poussé les chercheurs à concevoir des architectures plus stables comme les LSTM et GRU.

---

# Chapitre 5 — Le problème du gradient qui disparaît

## 5.1. Objectif du chapitre

Dans le chapitre précédent, nous avons étudié le problème des **dépendances longues**.

Nous avons vu qu’un RNN peut avoir du mal à relier deux informations éloignées dans une séquence, par exemple :

```txt
Le chat que j’ai vu hier dans la rue près de la gare était noir.
```

Dans cette phrase, le modèle doit comprendre que :

```txt
Le chat ... était noir.
```

Même si beaucoup de mots apparaissent entre `chat` et `était noir`.

Dans ce chapitre, nous allons expliquer une cause majeure de cette difficulté : le **gradient qui disparaît**, appelé en anglais **vanishing gradient**.

Nous allons comprendre :

- ce qu’est un gradient ;
    
- pourquoi il est nécessaire pour apprendre ;
    
- comment fonctionne la rétropropagation dans un RNN ;
    
- pourquoi le signal d’apprentissage peut devenir trop faible ;
    
- pourquoi cela empêche le modèle d’apprendre les dépendances longues ;
    
- comment les LSTM, GRU et Transformers répondent partiellement à ce problème.
    

---

## 5.2. Rappel : comment un modèle apprend-il ?

Un réseau de neurones apprend en corrigeant progressivement ses erreurs.

Prenons une tâche simple : prédire le prochain mot.

Entrée :

```txt
Le chat
```

Cible attendue :

```txt
dort
```

Le modèle produit une prédiction :

```txt
mange
```

Il y a donc une erreur.

Nous calculons alors une fonction de perte, appelée **loss**, qui mesure l’écart entre la prédiction et la bonne réponse.

```mermaid
flowchart LR
    A["Entrée : Le chat"] --> B["Modèle"]
    B --> C["Prédiction : mange"]
    D["Cible : dort"] --> E["Loss"]
    C --> E
    E --> F["Correction des poids"]
```

L’objectif de l’entraînement est de modifier les poids du réseau pour réduire cette loss.

---

## 5.3. Qu’est-ce qu’un gradient ?

Le **gradient** indique dans quelle direction modifier les paramètres du modèle pour réduire l’erreur.

Nous pouvons l’imaginer comme une pente.

Si nous sommes sur une montagne et que nous voulons descendre vers une vallée, nous regardons la pente locale pour savoir dans quelle direction aller.

```mermaid
flowchart TD
    A["Position actuelle du modèle"] --> B["Calcul de la loss"]
    B --> C["Calcul du gradient"]
    C --> D["Direction de correction"]
    D --> E["Mise à jour des poids"]
```

Dans un réseau de neurones, le gradient indique comment chaque poids contribue à l’erreur finale.

Si un poids contribue beaucoup à l’erreur, il doit être corrigé davantage.

Si un poids contribue peu, il est peu modifié.

---

## 5.4. La rétropropagation du gradient

Pour entraîner un réseau de neurones, nous utilisons la **rétropropagation**.

L’idée est la suivante :

1. nous faisons passer les données dans le réseau ;
    
2. le modèle produit une sortie ;
    
3. nous calculons l’erreur ;
    
4. nous faisons remonter cette erreur dans le réseau ;
    
5. nous ajustons les poids.
    

```mermaid
flowchart LR
    A["Entrée"] --> B["Couche 1"]
    B --> C["Couche 2"]
    C --> D["Couche 3"]
    D --> E["Sortie"]
    E --> F["Loss"]

    F -. "rétropropagation" .-> D
    D -. "gradient" .-> C
    C -. "gradient" .-> B
    B -. "gradient" .-> A
```

Dans un réseau classique, la rétropropagation traverse les couches.

Dans un RNN, elle traverse aussi les étapes temporelles.

---

## 5.5. Rétropropagation dans un RNN

Dans un RNN, une séquence est traitée étape par étape.

Par exemple :

```txt
Le → chat → dort → .
```

Le RNN calcule :

[  
h_1, h_2, h_3, h_4  
]

Si l’erreur apparaît à la fin, le gradient doit revenir en arrière à travers les états cachés.

```mermaid
flowchart RL
    L["Erreur finale"] --> H4["h4"]
    H4 --> H3["h3"]
    H3 --> H2["h2"]
    H2 --> H1["h1"]
    H1 --> X1["Premier token"]
```

Cette rétropropagation à travers les étapes temporelles s’appelle :

```txt
Backpropagation Through Time
```

ou **BPTT**.

Nous pouvons la traduire par :

> rétropropagation à travers le temps.

---

## 5.6. Pourquoi parle-t-on de “temps” ?

Dans les RNN, le mot **temps** ne désigne pas forcément le temps réel.

Il désigne surtout la position dans la séquence.

Pour une phrase :

```txt
Le chat dort.
```

nous avons :

```txt
t = 1 → Le
t = 2 → chat
t = 3 → dort
t = 4 → .
```

Chaque token correspond à une étape temporelle.

```mermaid
flowchart LR
    T1["t=1 : Le"] --> T2["t=2 : chat"]
    T2 --> T3["t=3 : dort"]
    T3 --> T4["t=4 : ."]
```

Dans une série temporelle, cela peut correspondre à du vrai temps.

Dans une phrase, cela correspond simplement à l’ordre des tokens.

---

## 5.7. Le problème central

Le problème apparaît lorsque la séquence est longue.

Si une erreur à la fin dépend d’une information au début, le gradient doit traverser beaucoup d’étapes.

Exemple :

```txt
Le chat que j’ai vu hier dans la rue près de la gare était noir.
```

La relation importante est :

```txt
Le chat → était noir
```

Mais entre les deux, le gradient doit traverser de nombreuses positions.

```mermaid
flowchart RL
    A["Erreur sur : était noir"] --> B["près de la gare"]
    B --> C["dans la rue"]
    C --> D["hier"]
    D --> E["que j’ai vu"]
    E --> F["Le chat"]
```

Plus le chemin est long, plus le signal peut s’affaiblir.

---

## 5.8. Définition du vanishing gradient

Le **vanishing gradient** désigne le cas où le gradient devient extrêmement faible pendant la rétropropagation.

Autrement dit, le signal de correction arrive presque nul aux premières étapes.

```mermaid
flowchart RL
    A["Erreur finale"] --> B["Gradient fort"]
    B --> C["Gradient moyen"]
    C --> D["Gradient faible"]
    D --> E["Gradient très faible"]
    E --> F["Gradient presque nul"]
```

Conséquence :

> Les premiers tokens reçoivent très peu de correction, même s’ils sont importants pour la prédiction finale.

Le modèle apprend donc mal les dépendances longues.

---

## 5.9. Exemple intuitif

Imaginons que le modèle doive apprendre cette relation :

```txt
Les clés ... sont
```

Phrase complète :

```txt
Les clés que mon frère a laissées hier dans la voiture sont sur la table.
```

Si le modèle prédit incorrectement :

```txt
Les clés ... est sur la table.
```

la correction doit remonter jusqu’à :

```txt
Les clés
```

car c’est cette information qui indique le pluriel.

```mermaid
flowchart RL
    A["Erreur : est au lieu de sont"] --> B["voiture"]
    B --> C["hier"]
    C --> D["a laissées"]
    D --> E["mon frère"]
    E --> F["Les clés"]
```

Mais si le gradient disparaît avant d’atteindre `Les clés`, le modèle ne comprend pas bien pourquoi il s’est trompé.

Il risque alors de continuer à faire ce type d’erreur.

---

## 5.10. Pourquoi le gradient diminue-t-il ?

Pour comprendre simplement, nous devons regarder ce qui se passe dans une chaîne de calculs.

Dans un RNN, chaque état dépend de l’état précédent :

[  
h_t = f(h_{t-1}, x_t)  
]

Donc, pour savoir comment une erreur finale dépend d’un état ancien, nous devons appliquer la règle de dérivation en chaîne.

Cela produit une multiplication de plusieurs termes.

Schématiquement :

# [  
\frac{\partial L}{\partial h_1}

\frac{\partial L}{\partial h_T}  
\times  
\frac{\partial h_T}{\partial h_{T-1}}  
\times  
\frac{\partial h_{T-1}}{\partial h_{T-2}}  
\times  
\dots  
\times  
\frac{\partial h_2}{\partial h_1}  
]

Cette formule signifie :

> Pour corriger le début de la séquence, le gradient doit traverser toutes les transformations intermédiaires.

Si beaucoup de facteurs sont inférieurs à 1, le produit devient très petit.

---

## 5.11. Exemple numérique simple

Prenons un exemple simplifié.

Supposons qu’à chaque étape, le gradient soit multiplié par :

[  
0.5  
]

Après 1 étape :

[  
0.5  
]

Après 2 étapes :

[  
0.5 \times 0.5 = 0.25  
]

Après 5 étapes :

[  
0.5^5 = 0.03125  
]

Après 10 étapes :

[  
0.5^{10} = 0.0009765625  
]

Après 20 étapes :

[  
0.5^{20} \approx 0.00000095  
]

Le signal devient quasiment nul.

```mermaid
flowchart LR
    A["Gradient initial : 1"] --> B["×0.5 = 0.5"]
    B --> C["×0.5 = 0.25"]
    C --> D["×0.5 = 0.125"]
    D --> E["..."]
    E --> F["Presque 0"]
```

C’est l’intuition fondamentale du vanishing gradient.

---

## 5.12. Le rôle de la fonction d’activation

Dans un RNN classique, nous avons souvent :

[  
h_t = \tanh(W_x x_t + W_h h_{t-1} + b)  
]

La fonction (\tanh) transforme les valeurs pour les ramener entre (-1) et (1).

```mermaid
flowchart LR
    A["Valeur quelconque"] --> B["tanh"]
    B --> C["Valeur entre -1 et 1"]
```

C’est utile pour stabiliser les activations.

Mais cela peut aussi poser problème.

Quand (\tanh) est saturée, sa dérivée devient très faible.

Autrement dit, si les valeurs sont trop grandes ou trop petites, la fonction devient presque plate.

Une fonction presque plate donne un gradient presque nul.

```mermaid
flowchart TD
    A["Valeurs très positives ou très négatives"] --> B["tanh saturée"]
    B --> C["Dérivée faible"]
    C --> D["Gradient faible"]
    D --> E["Apprentissage difficile"]
```

---

## 5.13. Conséquence sur l’apprentissage

Le vanishing gradient ne signifie pas que le modèle ne peut plus rien apprendre.

Il peut encore apprendre des relations courtes.

Par exemple :

```txt
un chat noir
```

Ici, `chat` et `noir` sont proches.

Le gradient n’a pas besoin de traverser beaucoup d’étapes.

```mermaid
flowchart LR
    A["chat"] --> B["noir"]
```

Mais pour une phrase plus longue :

```txt
Le chat que j’ai vu hier dans la rue près de la gare était noir.
```

la relation est plus éloignée :

```txt
chat → noir
```

```mermaid
flowchart LR
    A["Le chat"] --> B["que j’ai vu"]
    B --> C["hier"]
    C --> D["dans la rue"]
    D --> E["près de la gare"]
    E --> F["était noir"]

    A -. "dépendance longue" .-> F
```

Le modèle peut donc apprendre correctement certaines régularités locales, mais échouer à capturer les dépendances globales.

---

## 5.14. Vanishing gradient et mémoire courte

Le vanishing gradient donne aux RNN classiques une forme de **mémoire courte effective**.

Théoriquement, un RNN peut transporter de l’information sur une séquence très longue.

Mais en pratique, il apprend surtout à exploiter les informations proches.

Nous pouvons résumer ainsi :

```mermaid
flowchart TD
    A["RNN théorique"] --> B["Peut utiliser tout le passé"]
    C["RNN entraîné en pratique"] --> D["Utilise surtout le passé proche"]
    D --> E["Dépendances longues difficiles"]
```

C’est une distinction importante :

> Le problème n’est pas seulement ce que l’architecture peut représenter théoriquement, mais ce qu’elle peut apprendre efficacement.

---

## 5.15. Le problème inverse : exploding gradient

Le gradient peut aussi faire l’inverse : devenir trop grand.

C’est le problème de l’**exploding gradient**, ou **gradient qui explose**.

Si, à chaque étape, le gradient est multiplié par un facteur supérieur à 1, il peut croître très vite.

Par exemple :

[  
2^{10} = 1024  
]

[  
2^{20} = 1,048,576  
]

```mermaid
flowchart RL
    A["Erreur finale"] --> B["Gradient normal"]
    B --> C["Gradient grand"]
    C --> D["Gradient très grand"]
    D --> E["Instabilité numérique"]
```

Dans ce cas, les mises à jour des poids deviennent trop importantes.

Le modèle peut devenir instable, voire produire des valeurs numériques invalides.

---

## 5.16. Gradient clipping

Pour limiter l’exploding gradient, on utilise souvent le **gradient clipping**.

L’idée est simple :

> Si le gradient devient trop grand, nous le réduisons avant de mettre à jour les poids.

```mermaid
flowchart TD
    A["Gradient calculé"] --> B{"Gradient trop grand ?"}
    B -->|Oui| C["On limite sa norme"]
    B -->|Non| D["On le garde"]
    C --> E["Mise à jour des poids"]
    D --> E
```

Le gradient clipping ne résout pas vraiment le vanishing gradient.

Il aide surtout contre l’explosion du gradient.

Pour le gradient qui disparaît, il faut modifier l’architecture ou les chemins de gradient.

---

## 5.17. LSTM : une réponse au vanishing gradient

Les **LSTM** ont été conçus pour mieux conserver l’information sur de longues distances.

LSTM signifie :

```txt
Long Short-Term Memory
```

L’idée centrale est d’ajouter une mémoire plus stable, appelée souvent **cell state**.

Cette mémoire peut transporter de l’information avec moins de transformations destructrices.

```mermaid
flowchart LR
    A["Mémoire précédente c_(t-1)"] --> B["Cellule LSTM"]
    B --> C["Nouvelle mémoire c_t"]

    D["Entrée x_t"] --> B
    E["État caché h_(t-1)"] --> B
    B --> F["Sortie h_t"]
```

Les LSTM utilisent des portes pour contrôler :

- ce qui est oublié ;
    
- ce qui est ajouté ;
    
- ce qui est transmis.
    

---

## 5.18. Les portes dans un LSTM

Un LSTM contient principalement trois portes :

|Porte|Rôle|
|---|---|
|Porte d’oubli|décide quoi supprimer de la mémoire|
|Porte d’entrée|décide quoi ajouter à la mémoire|
|Porte de sortie|décide quoi exposer comme état caché|

```mermaid
flowchart TD
    A["Entrée x_t"] --> L["Cellule LSTM"]
    B["État précédent h_(t-1)"] --> L
    C["Mémoire précédente c_(t-1)"] --> L

    L --> F["Porte d'oubli"]
    L --> I["Porte d'entrée"]
    L --> O["Porte de sortie"]

    F --> M["Mémoire mise à jour c_t"]
    I --> M
    M --> O
    O --> H["État caché h_t"]
```

Ces portes permettent au modèle de mieux protéger certaines informations importantes.

Par exemple, dans :

```txt
Les clés que mon frère a laissées dans la voiture sont sur la table.
```

un LSTM peut apprendre à conserver l’information :

```txt
sujet pluriel
```

jusqu’au verbe `sont`.

---

## 5.19. GRU : une version plus compacte

Les **GRU**, ou **Gated Recurrent Units**, sont une autre réponse aux limites des RNN classiques.

Ils sont souvent plus simples que les LSTM.

Ils utilisent notamment :

- une porte de mise à jour ;
    
- une porte de réinitialisation.
    

```mermaid
flowchart TD
    A["Entrée x_t"] --> G["Cellule GRU"]
    B["État précédent h_(t-1)"] --> G

    G --> U["Porte de mise à jour"]
    G --> R["Porte de réinitialisation"]
    G --> H["Nouvel état h_t"]
```

Les GRU cherchent le même objectif général :

> mieux contrôler la circulation de l’information dans le temps.

Ils peuvent être plus rapides à entraîner que les LSTM, tout en donnant souvent de bonnes performances.

---

## 5.20. Limite des LSTM et GRU

Les LSTM et GRU améliorent fortement les RNN classiques.

Mais ils ne suppriment pas toutes les difficultés.

Ils restent :

- séquentiels ;
    
- difficiles à paralléliser ;
    
- coûteux pour les très longues séquences ;
    
- dépendants d’un état transmis étape par étape.
    

```mermaid
flowchart TD
    A["RNN classique"] --> B["Vanishing gradient important"]
    B --> C["LSTM / GRU"]
    C --> D["Meilleure mémoire"]
    D --> E["Mais traitement toujours séquentiel"]
```

Même avec des portes, l’information circule encore principalement dans l’ordre de la séquence.

C’est précisément ce que les Transformers vont changer.

---

## 5.21. L’attention comme chemin plus court pour le gradient

Avec l’attention, un token peut regarder directement un autre token.

Cela crée des chemins plus courts entre des positions éloignées.

Dans une phrase comme :

```txt
Le chat que j’ai vu hier dans la rue près de la gare était noir.
```

le token `noir` peut accorder de l’attention directement à `chat`.

```mermaid
flowchart LR
    A["Le chat"] -. "attention directe" .-> F["était noir"]
    B["que j’ai vu"] --> C["hier"]
    C --> D["dans la rue"]
    D --> E["près de la gare"]
```

Cela ne signifie pas que le gradient ne pose plus jamais problème, mais cela réduit fortement la dépendance à une chaîne temporelle longue.

Le chemin entre deux tokens importants peut devenir beaucoup plus direct.

---

## 5.22. RNN contre Transformer : chemin de gradient

Dans un RNN, pour relier le token 1 au token 10, le signal passe par toutes les étapes :

```mermaid
flowchart LR
    A["Token 1"] --> B["Token 2"]
    B --> C["Token 3"]
    C --> D["..."]
    D --> E["Token 10"]
```

Dans un Transformer avec self-attention, le token 10 peut accéder directement au token 1 :

```mermaid
flowchart LR
    A["Token 1"] -. "attention" .-> E["Token 10"]
```

Cette différence est une motivation majeure du Transformer.

---

## 5.23. Le lien avec “Attention Is All You Need”

Le papier **Attention Is All You Need** propose de supprimer complètement la récurrence.

Cela signifie que nous n’avons plus besoin de propager l’information uniquement ainsi :

```txt
h1 → h2 → h3 → h4 → ... → hn
```

À la place, nous construisons des représentations où chaque token peut interagir avec les autres via l’attention.

```mermaid
flowchart TD
    A["RNN"] --> B["Chaîne temporelle"]
    B --> C["Gradient traverse de nombreuses étapes"]

    D["Transformer"] --> E["Self-attention"]
    E --> F["Relations directes entre positions"]
```

C’est l’une des raisons pour lesquelles les Transformers ont permis de meilleurs résultats sur les longues séquences et un entraînement plus efficace sur matériel parallèle.

---

## 5.24. Attention : le problème n’est pas entièrement supprimé

Il serait faux de dire que les Transformers n’ont plus aucun problème de gradient.

Les grands modèles profonds peuvent encore rencontrer :

- des problèmes de stabilité ;
    
- des gradients mal conditionnés ;
    
- des difficultés d’optimisation ;
    
- des problèmes liés à la profondeur ;
    
- des problèmes liés à la longueur de contexte.
    

C’est pourquoi les Transformers utilisent aussi :

- des connexions résiduelles ;
    
- de la normalisation ;
    
- des initialisations adaptées ;
    
- des optimiseurs comme Adam ;
    
- des schedules de learning rate ;
    
- parfois du gradient clipping.
    

Mais ils évitent une limite majeure des RNN :

> l’obligation de transmettre toute l’information dans une chaîne temporelle strictement séquentielle.

---

## 5.25. Exemple comparatif complet

Prenons la phrase :

```txt
La décision que le comité a longuement discutée pendant plusieurs réunions a été validée.
```

La relation principale est :

```txt
La décision → a été validée
```

### Avec un RNN

L’information `La décision` doit traverser :

```txt
que → le comité → a longuement discutée → pendant → plusieurs réunions
```

avant d’être utilisée.

```mermaid
flowchart LR
    A["La décision"] --> B["que"]
    B --> C["le comité"]
    C --> D["a discutée"]
    D --> E["pendant plusieurs réunions"]
    E --> F["a été validée"]

    A -. "information à conserver longtemps" .-> F
```

### Avec l’attention

Le token correspondant à `validée` peut apprendre à regarder directement `La décision`.

```mermaid
flowchart LR
    A["La décision"] -. "attention directe" .-> F["validée"]
    B["le comité"] --> C["a discutée"]
    C --> D["pendant plusieurs réunions"]
```

Cette connexion directe facilite l’apprentissage de la relation.

---

## 5.26. Ce qu’il faut retenir mathématiquement

Le point mathématique essentiel est que le gradient dans un RNN traverse une chaîne de dépendances.

Si la sortie finale dépend d’un état ancien, la correction doit passer par beaucoup de dérivées intermédiaires.

Schématiquement :

# [  
\frac{\partial L}{\partial h_1}

\frac{\partial L}{\partial h_T}  
\prod_{t=2}^{T}  
\frac{\partial h_t}{\partial h_{t-1}}  
]

Cette multiplication répétée peut rendre le gradient :

- très petit : vanishing gradient ;
    
- très grand : exploding gradient.
    

Dans le cas du vanishing gradient, les premières étapes n’apprennent presque plus.

---

## 5.27. Ce qu’il faut retenir intuitivement

Nous pouvons retenir l’image suivante.

Un RNN doit transmettre un message de personne en personne.

```mermaid
flowchart LR
    A["Message initial"] --> B["Personne 1"]
    B --> C["Personne 2"]
    C --> D["Personne 3"]
    D --> E["..."]
    E --> F["Personne 20"]
```

À chaque transmission, le message peut être légèrement modifié ou affaibli.

Au bout d’un grand nombre de transmissions, le message original peut être perdu.

L’attention, elle, permettrait à la personne 20 de demander directement l’information à la personne 1.

```mermaid
flowchart LR
    A["Personne 1"] -. "connexion directe" .-> F["Personne 20"]
```

C’est cette intuition qui nous aidera à comprendre le cœur des Transformers.

---

## 5.28. Résumé du chapitre

Dans ce chapitre, nous avons étudié le **gradient qui disparaît**.

Nous avons vu que les RNN apprennent grâce à la rétropropagation à travers le temps.

Quand une erreur finale dépend d’un élément ancien de la séquence, le gradient doit remonter à travers de nombreuses étapes.

À chaque étape, le signal peut être réduit.

Si cette réduction se répète plusieurs fois, le gradient devient presque nul.

Conséquence :

> Le modèle apprend mal les relations éloignées.

Il peut apprendre correctement des dépendances courtes comme :

```txt
un chat noir
```

mais il aura plus de difficulté avec :

```txt
Le chat que j’ai vu hier dans la rue près de la gare était noir.
```

Les LSTM et GRU améliorent cette situation en ajoutant des mécanismes de portes.

Mais ils restent séquentiels.

Les Transformers proposent une autre solution : utiliser l’attention pour créer des relations directes entre tokens éloignés.

---

## 5.29. Schéma de synthèse

```mermaid
flowchart TD
    A["Erreur finale"] --> B["Rétropropagation dans le temps"]
    B --> C["Gradient traverse de nombreuses étapes"]
    C --> D{"Multiplications répétées"}

    D --> E["Facteurs < 1"]
    E --> F["Gradient qui disparaît"]
    F --> G["Début de séquence peu corrigé"]
    G --> H["Dépendances longues mal apprises"]

    D --> I["Facteurs > 1"]
    I --> J["Gradient qui explose"]
    J --> K["Entraînement instable"]

    H --> L["LSTM / GRU"]
    L --> M["Meilleure mémoire"]

    H --> N["Attention"]
    N --> O["Chemins plus directs entre tokens"]
```

---

## 5.30. Questions de compréhension

### Question 1

Qu’est-ce que la rétropropagation du gradient ?

Réponse attendue : c’est le mécanisme qui permet de calculer comment modifier les poids du modèle pour réduire l’erreur.

### Question 2

Pourquoi parle-t-on de Backpropagation Through Time dans les RNN ?

Réponse attendue : parce que le RNN est déroulé sur les différentes étapes de la séquence, et que le gradient doit remonter à travers ces étapes temporelles.

### Question 3

Qu’est-ce que le vanishing gradient ?

Réponse attendue : c’est le phénomène où le gradient devient très faible pendant la rétropropagation, au point que les premières couches ou premières étapes apprennent très peu.

### Question 4

Pourquoi le vanishing gradient gêne-t-il l’apprentissage des dépendances longues ?

Réponse attendue : parce que les informations anciennes reçoivent un signal de correction trop faible, même si elles sont importantes pour une erreur observée plus tard.

### Question 5

Quelle est la différence entre vanishing gradient et exploding gradient ?

Réponse attendue : dans le vanishing gradient, le gradient devient trop petit ; dans l’exploding gradient, il devient trop grand et rend l’entraînement instable.

### Question 6

Quel est le rôle du gradient clipping ?

Réponse attendue : il limite la taille du gradient lorsqu’il devient trop grand, afin d’éviter les mises à jour instables.

### Question 7

Pourquoi les LSTM et GRU aident-ils ?

Réponse attendue : ils ajoutent des portes qui contrôlent mieux la mémoire et permettent de conserver certaines informations plus longtemps.

### Question 8

Pourquoi l’attention aide-t-elle aussi ?

Réponse attendue : elle permet de créer des connexions directes entre tokens éloignés, ce qui réduit la dépendance à une transmission étape par étape.

---

## 5.31. Transition vers le chapitre suivant

Nous avons maintenant compris pourquoi les RNN classiques ont du mal à apprendre les relations éloignées : le signal d’apprentissage peut disparaître en remontant dans le temps.

Dans le chapitre suivant, nous allons étudier les **LSTM et GRU**, qui ont été conçus précisément pour améliorer la mémoire des RNN.

Nous verrons comment ces architectures ajoutent des portes de contrôle pour décider :

- quoi oublier ;
    
- quoi conserver ;
    
- quoi ajouter ;
    
- quoi transmettre.
    

Cela nous permettra de comprendre pourquoi les LSTM ont représenté une étape importante avant l’arrivée des Transformers.

---

# Chapitre 6 — Les LSTM et GRU : une amélioration des RNN

## 6.1. Objectif du chapitre

Dans les chapitres précédents, nous avons étudié deux limites majeures des RNN classiques :

1. le problème des **dépendances longues** ;
    
2. le problème du **gradient qui disparaît**.
    

Nous avons vu qu’un RNN classique possède une mémoire sous forme d’état caché :

[  
h_t = f(x_t, h_{t-1})  
]

Mais cette mémoire est fragile.

À chaque étape, elle est modifiée, mélangée avec une nouvelle entrée, puis transmise à l’étape suivante.

Cela rend difficile la conservation d’une information importante pendant longtemps.

Pour limiter ces problèmes, des architectures plus avancées ont été proposées, notamment :

- les **LSTM** ;
    
- les **GRU**.
    

Ces modèles ajoutent des mécanismes de contrôle de la mémoire.

L’idée est de permettre au réseau de décider :

- quoi oublier ;
    
- quoi conserver ;
    
- quoi mettre à jour ;
    
- quoi transmettre.
    

```mermaid
flowchart TD
    A["Entrée x_t"] --> L["Cellule LSTM / GRU"]
    B["Mémoire précédente"] --> L
    L --> C["Nouvelle mémoire"]
    L --> D["Sortie"]
    
    L --> E["Porte d'oubli"]
    L --> F["Porte d'entrée"]
    L --> G["Porte de sortie"]
```

Les LSTM et GRU ont beaucoup amélioré la capacité des réseaux à traiter des séquences.

Mais ils conservent une limite importante :

> Ils traitent toujours les éléments principalement les uns après les autres.

Cette nature séquentielle devient un problème lorsque nous voulons entraîner de grands modèles sur beaucoup de données.

---

## 6.2. Pourquoi les RNN classiques sont insuffisants ?

Reprenons la formule simplifiée d’un RNN classique :

[  
h_t = \tanh(W_x x_t + W_h h_{t-1} + b)  
]

À chaque étape, le nouvel état caché (h_t) dépend :

- du token actuel (x_t) ;
    
- de l’état caché précédent (h_{t-1}).
    

Le problème est que cette mise à jour est assez brutale.

L’état précédent est transformé, combiné avec la nouvelle entrée, puis passé dans une fonction d’activation.

```mermaid
flowchart LR
    A["Mémoire précédente h_(t-1)"] --> C["Transformation"]
    B["Entrée actuelle x_t"] --> C
    C --> D["tanh"]
    D --> E["Nouvelle mémoire h_t"]
```

Le modèle n’a pas de mécanisme explicite pour dire :

```txt
Cette information est très importante, il faut la garder longtemps.
```

ou :

```txt
Cette information n’est plus utile, nous pouvons l’oublier.
```

Tout passe par la même mise à jour.

Les LSTM et GRU introduisent donc une idée centrale :

> Nous allons ajouter des portes qui contrôlent le flux d’information.

---

## 6.3. L’idée des portes

Une **porte**, dans un réseau récurrent, est un mécanisme qui décide quelle quantité d’information doit passer.

Nous pouvons l’imaginer comme un robinet.

- Si la porte vaut près de 0, l’information est bloquée.
    
- Si la porte vaut près de 1, l’information passe.
    
- Si la porte vaut une valeur intermédiaire, l’information passe partiellement.
    

Mathématiquement, une porte est souvent calculée avec une fonction sigmoïde :

[  
\sigma(z) = \frac{1}{1 + e^{-z}}  
]

Cette fonction renvoie une valeur entre 0 et 1.

```mermaid
flowchart LR
    A["Information"] --> B{"Porte"}
    B -->|Valeur proche de 0| C["Information bloquée"]
    B -->|Valeur proche de 1| D["Information transmise"]
    B -->|Valeur intermédiaire| E["Information partiellement transmise"]
```

Dans les LSTM et GRU, ces portes sont apprises automatiquement pendant l’entraînement.

Le modèle apprend donc à contrôler sa mémoire.

---

## 6.4. Intuition générale des LSTM

LSTM signifie :

```txt
Long Short-Term Memory
```

En français, nous pourrions traduire par :

```txt
mémoire à long et court terme
```

Le LSTM a été conçu pour mieux conserver les informations importantes sur de longues distances.

L’idée centrale est d’ajouter une mémoire plus stable appelée :

[  
c_t  
]

Cette mémoire est souvent appelée **cell state**.

Nous avons donc deux états principaux :

- (h_t) : l’état caché, utilisé comme sortie de la cellule ;
    
- (c_t) : l’état de cellule, qui transporte la mémoire à plus long terme.
    

```mermaid
flowchart TD
    A["Entrée x_t"] --> L["Cellule LSTM"]
    B["État caché précédent h_(t-1)"] --> L
    C["Mémoire précédente c_(t-1)"] --> L

    L --> D["Nouvel état caché h_t"]
    L --> E["Nouvelle mémoire c_t"]
```

La différence avec un RNN classique est importante.

Dans un RNN classique, nous avons principalement :

[  
h_t  
]

Dans un LSTM, nous avons :

[  
h_t \quad \text{et} \quad c_t  
]

Le LSTM sépare donc davantage :

- la mémoire interne ;
    
- la sortie exposée à l’étape suivante.
    

---

## 6.5. Pourquoi ajouter une mémoire de cellule ?

Dans un RNN classique, la mémoire est constamment recalculée.

Dans un LSTM, la mémoire de cellule (c_t) peut être transmise plus directement d’une étape à l’autre.

```mermaid
flowchart LR
    C1["c_(t-1)"] --> C2["c_t"]
    C2 --> C3["c_(t+1)"]
    C3 --> C4["c_(t+2)"]
```

Cette circulation plus directe facilite le passage du gradient.

Autrement dit, l’information importante peut survivre plus longtemps.

Cela ne rend pas le modèle parfait, mais cela améliore fortement la capacité à apprendre des dépendances longues.

---

## 6.6. Les trois portes principales du LSTM

Un LSTM utilise principalement trois portes :

1. la **porte d’oubli** ;
    
2. la **porte d’entrée** ;
    
3. la **porte de sortie**.
    

```mermaid
flowchart TD
    A["Entrée x_t"] --> L["Cellule LSTM"]
    B["h_(t-1)"] --> L
    C["c_(t-1)"] --> L

    L --> F["Porte d'oubli"]
    L --> I["Porte d'entrée"]
    L --> O["Porte de sortie"]

    F --> M["Mise à jour de la mémoire"]
    I --> M
    M --> O
    O --> H["h_t"]
```

Ces trois portes répondent à trois questions :

|Porte|Question posée|
|---|---|
|Porte d’oubli|Que devons-nous supprimer de l’ancienne mémoire ?|
|Porte d’entrée|Quelle nouvelle information devons-nous ajouter ?|
|Porte de sortie|Quelle partie de la mémoire devons-nous exposer en sortie ?|

---

## 6.7. La porte d’oubli

La **porte d’oubli** décide quelle partie de l’ancienne mémoire doit être conservée ou supprimée.

Elle prend en compte :

- l’entrée actuelle (x_t) ;
    
- l’état caché précédent (h_{t-1}).
    

Elle produit un vecteur (f_t), dont les valeurs sont entre 0 et 1.

[  
f_t = \sigma(W_f [h_{t-1}, x_t] + b_f)  
]

où :

- (f_t) est la porte d’oubli ;
    
- (W_f) est une matrice de poids ;
    
- (b_f) est un biais ;
    
- (\sigma) est la fonction sigmoïde ;
    
- ([h_{t-1}, x_t]) représente la concaténation de l’état précédent et de l’entrée actuelle.
    

Si une composante de (f_t) est proche de 0, l’information correspondante est oubliée.

Si elle est proche de 1, elle est conservée.

```mermaid
flowchart LR
    A["Ancienne mémoire c_(t-1)"] --> B{"Porte d'oubli f_t"}
    B -->|0| C["Oubli"]
    B -->|1| D["Conservation"]
    B -->|entre 0 et 1| E["Conservation partielle"]
```

Exemple intuitif :

Si nous lisons :

```txt
Marie habitait à Lille. Elle a ensuite déménagé à Toulouse.
```

Quand le modèle lit `Toulouse`, il peut apprendre à diminuer l’importance de l’ancienne information `Lille` pour la résidence actuelle.

---

## 6.8. La porte d’entrée

La **porte d’entrée** décide quelles nouvelles informations doivent être ajoutées à la mémoire.

Elle est généralement associée à deux calculs :

1. une porte d’entrée (i_t) ;
    
2. une mémoire candidate (\tilde{c}_t).
    

La porte d’entrée :

[  
i_t = \sigma(W_i [h_{t-1}, x_t] + b_i)  
]

La mémoire candidate :

[  
\tilde{c}_t = \tanh(W_c [h_{t-1}, x_t] + b_c)  
]

Puis nous décidons quelle partie de cette mémoire candidate est ajoutée.

```mermaid
flowchart TD
    A["Entrée x_t et h_(t-1)"] --> B["Mémoire candidate c~_t"]
    A --> C["Porte d'entrée i_t"]
    B --> D["Nouvelle information possible"]
    C --> E["Quantité à ajouter"]
    D --> F["Ajout contrôlé à la mémoire"]
    E --> F
```

Exemple intuitif :

Si la phrase introduit une nouvelle entité importante :

```txt
Le contrat signé par l’entreprise prévoit une clause de confidentialité.
```

Le modèle peut apprendre à ajouter à sa mémoire :

```txt
entité importante = le contrat
```

---

## 6.9. Mise à jour de la mémoire dans un LSTM

La nouvelle mémoire (c_t) combine :

1. l’ancienne mémoire, filtrée par la porte d’oubli ;
    
2. la nouvelle mémoire candidate, filtrée par la porte d’entrée.
    

La formule est :

[  
c_t = f_t \odot c_{t-1} + i_t \odot \tilde{c}_t  
]

où (\odot) désigne la multiplication élément par élément.

Nous pouvons lire cette formule ainsi :

```txt
nouvelle mémoire =
ce que nous gardons de l’ancienne mémoire
+
ce que nous ajoutons comme nouvelle information
```

```mermaid
flowchart LR
    A["Ancienne mémoire c_(t-1)"] --> B["× porte d'oubli f_t"]
    C["Mémoire candidate c~_t"] --> D["× porte d'entrée i_t"]

    B --> E["Addition"]
    D --> E
    E --> F["Nouvelle mémoire c_t"]
```

Cette formule est le cœur du LSTM.

Elle permet à la mémoire de se modifier de manière plus contrôlée que dans un RNN classique.

---

## 6.10. La porte de sortie

La **porte de sortie** décide quelle partie de la mémoire interne doit être exposée comme état caché (h_t).

Elle est calculée ainsi :

[  
o_t = \sigma(W_o [h_{t-1}, x_t] + b_o)  
]

Puis l’état caché est :

[  
h_t = o_t \odot \tanh(c_t)  
]

Autrement dit :

```txt
état caché =
partie visible de la mémoire interne
```

```mermaid
flowchart LR
    A["Mémoire c_t"] --> B["tanh"]
    B --> C["× porte de sortie o_t"]
    C --> D["État caché h_t"]
```

Le LSTM peut donc conserver une information dans sa mémoire interne sans forcément l’exposer entièrement en sortie à chaque étape.

---

## 6.11. Résumé du fonctionnement d’un LSTM

Nous pouvons résumer une cellule LSTM ainsi :

```mermaid
flowchart TD
    A["Entrée x_t"] --> L["Cellule LSTM"]
    B["État caché précédent h_(t-1)"] --> L
    C["Mémoire précédente c_(t-1)"] --> L

    L --> F["1. Porte d'oubli : quoi garder de c_(t-1) ?"]
    L --> I["2. Porte d'entrée : quoi ajouter ?"]
    L --> O["3. Porte de sortie : quoi exposer ?"]

    F --> M["Nouvelle mémoire c_t"]
    I --> M
    M --> O
    O --> H["Nouvel état caché h_t"]
```

Le LSTM est donc une cellule récurrente avec une mémoire contrôlée.

Nous pouvons le voir comme une version améliorée du RNN classique, capable de mieux préserver certaines informations.

---

## 6.12. Exemple pédagogique : accord sujet-verbe

Prenons la phrase :

```txt
Les clés que mon frère a laissées dans la voiture sont sur la table.
```

Le modèle doit retenir que le sujet principal est :

```txt
Les clés
```

et que ce sujet est pluriel.

Plus tard, lorsqu’il arrive à :

```txt
sont
```

il doit utiliser cette information.

```mermaid
flowchart LR
    A["Les clés"] -. "information à conserver : pluriel" .-> F["sont"]
    B["mon frère"] --> C["a laissées"]
    C --> D["dans la voiture"]
    D --> F
```

Dans un RNN classique, l’information `pluriel` peut être diluée.

Dans un LSTM, la mémoire de cellule peut apprendre à conserver cette information plus longtemps.

```mermaid
flowchart LR
    A["Les clés : pluriel"] --> B["Mémoire LSTM"]
    B --> C["mots intermédiaires"]
    C --> D["Mémoire encore disponible"]
    D --> E["sont"]
```

---

## 6.13. Exemple pédagogique : changement de sujet

Prenons maintenant :

```txt
Pierre vivait à Lille. Après ses études, il a déménagé à Lyon.
```

Au début, la mémoire peut contenir :

```txt
Pierre → ville actuelle : Lille
```

Puis, quand nous lisons :

```txt
a déménagé à Lyon
```

le modèle doit mettre à jour cette information.

```mermaid
flowchart TD
    A["Pierre vivait à Lille"] --> B["Mémoire : ville = Lille"]
    B --> C["a déménagé à Lyon"]
    C --> D["Porte d'oubli : réduire Lille"]
    C --> E["Porte d'entrée : ajouter Lyon"]
    D --> F["Mémoire mise à jour"]
    E --> F
    F --> G["ville actuelle = Lyon"]
```

Le LSTM permet ce type de mise à jour contrôlée.

---

## 6.14. Les GRU : une autre amélioration

Les **GRU**, ou **Gated Recurrent Units**, sont une autre architecture récurrente avec des portes.

Ils poursuivent le même objectif général que les LSTM :

> mieux contrôler la circulation de l’information dans le temps.

Mais ils sont plus simples.

Un GRU n’a pas de mémoire de cellule séparée (c_t).

Il utilise seulement un état caché (h_t), mais avec des portes de contrôle.

```mermaid
flowchart TD
    A["Entrée x_t"] --> G["Cellule GRU"]
    B["État précédent h_(t-1)"] --> G

    G --> C["Porte de mise à jour"]
    G --> D["Porte de réinitialisation"]
    G --> E["Nouvel état h_t"]
```

Cette architecture est souvent plus légère que le LSTM.

---

## 6.15. Les deux portes principales du GRU

Un GRU utilise principalement deux portes :

|Porte|Rôle|
|---|---|
|Porte de mise à jour|décide combien de l’ancien état conserver|
|Porte de réinitialisation|décide combien de l’ancien état utiliser pour calculer la nouvelle information|

Ces portes sont :

- la porte de mise à jour (z_t) ;
    
- la porte de réinitialisation (r_t).
    

---

## 6.16. La porte de mise à jour du GRU

La porte de mise à jour (z_t) décide dans quelle mesure nous gardons l’ancien état caché.

[  
z_t = \sigma(W_z [h_{t-1}, x_t])  
]

Si (z_t) est proche de 1, nous conservons beaucoup de l’ancien état.

Si (z_t) est proche de 0, nous remplaçons davantage l’ancien état par une nouvelle information.

```mermaid
flowchart LR
    A["Ancien état h_(t-1)"] --> B{"Porte de mise à jour z_t"}
    B -->|proche de 1| C["On conserve l'ancien état"]
    B -->|proche de 0| D["On accepte davantage la nouvelle information"]
```

Cette porte joue un rôle proche de la combinaison entre la porte d’oubli et la porte d’entrée du LSTM.

---

## 6.17. La porte de réinitialisation du GRU

La porte de réinitialisation (r_t) décide combien de l’ancien état doit être utilisé pour calculer la mémoire candidate.

[  
r_t = \sigma(W_r [h_{t-1}, x_t])  
]

Si (r_t) est proche de 0, le modèle ignore largement l’ancien contexte pour calculer la nouvelle information.

Si (r_t) est proche de 1, il utilise fortement l’ancien contexte.

```mermaid
flowchart LR
    A["Ancien état h_(t-1)"] --> B{"Porte de réinitialisation r_t"}
    B -->|proche de 0| C["Ancien contexte ignoré"]
    B -->|proche de 1| D["Ancien contexte utilisé"]
```

Cela permet au GRU de décider quand repartir presque de zéro, par exemple lorsqu’un nouveau segment de phrase ou un nouveau sujet commence.

---

## 6.18. Mise à jour de l’état dans un GRU

Le GRU calcule d’abord une mémoire candidate (\tilde{h}_t) :

[  
\tilde{h}_t = \tanh(W [r_t \odot h_{t-1}, x_t])  
]

Puis il combine l’ancien état et le nouvel état candidat :

[  
h_t = (1 - z_t) \odot h_{t-1} + z_t \odot \tilde{h}_t  
]

Selon les conventions, on peut parfois trouver une formule inversée sur le rôle de (z_t), mais l’idée reste la même :

> le GRU apprend à mélanger l’ancienne mémoire et la nouvelle information.

```mermaid
flowchart LR
    A["Ancien état h_(t-1)"] --> B["Conservation contrôlée"]
    C["État candidat h~_t"] --> D["Ajout contrôlé"]
    B --> E["Nouvel état h_t"]
    D --> E
```

Le GRU est donc plus simple que le LSTM, mais il conserve l’idée fondamentale des portes.

---

## 6.19. LSTM contre GRU

Nous pouvons comparer les deux architectures.

|Critère|LSTM|GRU|
|---|---|---|
|Mémoire séparée|Oui, (c_t) et (h_t)|Non, principalement (h_t)|
|Nombre de portes|3 principales|2 principales|
|Complexité|Plus élevée|Plus compacte|
|Coût calculatoire|Plus important|Souvent plus faible|
|Capacité mémoire|Très bonne|Très bonne aussi dans beaucoup de cas|
|Usage historique|Très utilisé en NLP, parole, séries temporelles|Très utilisé aussi, souvent plus simple|

Il n’y a pas de gagnant absolu.

Le choix dépend :

- de la tâche ;
    
- de la taille du dataset ;
    
- du coût acceptable ;
    
- de la longueur des séquences ;
    
- de l’architecture globale.
    

---

## 6.20. Pourquoi les portes aident contre le gradient qui disparaît ?

Les portes aident parce qu’elles créent des chemins plus contrôlés pour l’information et le gradient.

Dans un RNN classique, l’état est transformé à chaque étape par une fonction non linéaire.

Dans un LSTM, une partie de la mémoire peut être transmise plus directement.

```mermaid
flowchart TD
    A["RNN classique"] --> B["Mémoire transformée à chaque étape"]
    B --> C["Gradient peut disparaître rapidement"]

    D["LSTM / GRU"] --> E["Mémoire contrôlée par portes"]
    E --> F["Information mieux conservée"]
    F --> G["Dépendances longues mieux apprises"]
```

L’idée n’est pas que le gradient ne disparaît plus jamais.

L’idée est que l’architecture rend plus facile la conservation d’informations importantes.

---

## 6.21. Exemple comparatif RNN / LSTM

Prenons la phrase :

```txt
La maison que les ouvriers ont rénovée pendant l’été est magnifique.
```

La relation importante est :

```txt
La maison → est magnifique
```

## Avec un RNN classique

```mermaid
flowchart LR
    A["La maison"] --> B["que"]
    B --> C["les ouvriers"]
    C --> D["ont rénovée"]
    D --> E["pendant l’été"]
    E --> F["est magnifique"]

    A -. "information fragile" .-> F
```

L’information `La maison` peut être progressivement diluée.

## Avec un LSTM

```mermaid
flowchart LR
    A["La maison"] --> B["Mémoire LSTM"]
    B --> C["mots intermédiaires"]
    C --> D["Mémoire conservée"]
    D --> E["est magnifique"]
```

Grâce aux portes, le modèle peut mieux préserver l’information principale.

---

## 6.22. Les LSTM et GRU dans les modèles Seq2Seq

Avant les Transformers, les LSTM et GRU ont été très utilisés dans les modèles **Seq2Seq**.

Un encodeur lit une phrase source.

Un décodeur produit une phrase cible.

```mermaid
flowchart LR
    A["Phrase source"] --> B["Encodeur LSTM / GRU"]
    B --> C["Représentation de contexte"]
    C --> D["Décodeur LSTM / GRU"]
    D --> E["Phrase cible"]
```

Exemple :

```txt
Source : I love machine learning.
Cible  : J’aime l’apprentissage automatique.
```

Les LSTM et GRU ont permis de meilleurs résultats que les RNN classiques, notamment parce qu’ils géraient mieux les séquences longues.

Mais ils restaient confrontés au problème du **vecteur de contexte unique**.

---

## 6.23. LSTM, GRU et attention

L’attention a d’abord été ajoutée aux modèles récurrents.

L’idée était de permettre au décodeur de regarder directement les états de l’encodeur.

```mermaid
flowchart LR
    A["Phrase source"] --> B["Encodeur LSTM"]
    B --> C["États cachés h1, h2, h3, ..."]
    C --> D["Mécanisme d'attention"]
    D --> E["Décodeur LSTM"]
    E --> F["Phrase cible"]
```

Cela a beaucoup amélioré les modèles Seq2Seq.

Le décodeur n’était plus obligé de dépendre uniquement d’un seul vecteur de contexte.

À chaque étape, il pouvait sélectionner les parties utiles de la phrase source.

```mermaid
flowchart TD
    A["États encodeur"] --> B["Attention"]
    C["État courant du décodeur"] --> B
    B --> D["Contexte dynamique"]
    D --> E["Prédiction du prochain token"]
```

Cette combinaison RNN + attention prépare directement l’arrivée du Transformer.

---

## 6.24. La limite fondamentale restante : le traitement séquentiel

Même avec LSTM ou GRU, le modèle reste récurrent.

Cela signifie que pour calculer l’état (h_t), nous devons connaître (h_{t-1}).

Donc :

[  
h_1 \rightarrow h_2 \rightarrow h_3 \rightarrow \dots \rightarrow h_t  
]

```mermaid
flowchart LR
    H1["h1"] --> H2["h2"]
    H2 --> H3["h3"]
    H3 --> H4["h4"]
    H4 --> H5["..."]
    H5 --> HT["h_t"]
```

Nous ne pouvons pas facilement calculer tous les états en parallèle.

C’est une limite très importante lorsque nous voulons entraîner de grands modèles sur des corpus massifs.

---

## 6.25. Pourquoi la séquentialité limite le passage à l’échelle ?

Les GPU et TPU sont très efficaces lorsqu’ils peuvent effectuer beaucoup d’opérations en parallèle.

Mais les LSTM et GRU imposent une dépendance temporelle forte.

Pour calculer (h_{50}), il faut avoir calculé :

```txt
h1, h2, h3, ..., h49
```

Cela crée une forme d’attente.

```mermaid
flowchart TD
    A["GPU / TPU"] --> B["Très efficace pour calculs parallèles"]
    C["LSTM / GRU"] --> D["Calcul séquentiel étape par étape"]
    D --> E["Parallélisation limitée"]
    E --> F["Passage à l'échelle plus difficile"]
```

Même si les calculs internes d’une cellule peuvent être parallélisés, la dépendance entre étapes reste un goulot d’étranglement.

---

## 6.26. Comparaison avec l’attention

L’attention propose une autre approche.

Au lieu de transmettre l’information token par token, nous calculons directement les relations entre les tokens.

```mermaid
flowchart TD
    A["Token 1"] <--> B["Token 2"]
    A <--> C["Token 3"]
    A <--> D["Token 4"]
    B <--> C
    B <--> D
    C <--> D
```

Cela permet une parallélisation beaucoup plus forte pendant l’entraînement.

Tous les tokens d’une séquence peuvent être projetés en même temps, puis comparés via des opérations matricielles.

C’est exactement le type d’opérations que les GPU savent très bien faire.

---

## 6.27. Pourquoi les LSTM et GRU restent importants historiquement ?

Même si les Transformers dominent aujourd’hui de nombreuses tâches de NLP, les LSTM et GRU restent importants à comprendre.

Ils représentent une étape majeure dans l’histoire des architectures séquentielles.

Ils ont montré que la mémoire devait être contrôlée explicitement.

Ils ont aussi préparé plusieurs idées reprises ensuite :

- importance des chemins de gradient ;
    
- contrôle du flux d’information ;
    
- traitement des dépendances longues ;
    
- modélisation de séquences ;
    
- architectures encodeur-décodeur ;
    
- génération autoregressive.
    

```mermaid
flowchart LR
    A["RNN"] --> B["LSTM / GRU"]
    B --> C["Seq2Seq"]
    C --> D["Seq2Seq + Attention"]
    D --> E["Transformer"]
```

Nous ne devons donc pas les voir comme des architectures dépassées sans intérêt.

Nous devons les voir comme une étape essentielle pour comprendre pourquoi les Transformers ont été une rupture.

---

## 6.28. LSTM et GRU sont-ils encore utilisés ?

Oui, même si les Transformers sont devenus dominants dans beaucoup de tâches de langage.

Les LSTM et GRU peuvent encore être utiles lorsque :

- les données sont de taille modérée ;
    
- les séquences ne sont pas trop longues ;
    
- le coût de calcul doit rester faible ;
    
- nous travaillons sur des séries temporelles simples ;
    
- nous avons besoin d’un modèle plus léger ;
    
- nous ne disposons pas d’une infrastructure lourde ;
    
- nous voulons un modèle embarqué ou rapide.
    

Dans certains contextes, un LSTM bien conçu peut être plus approprié qu’un Transformer trop coûteux.

Mais pour le NLP massif, la traduction moderne, les LLM et les modèles multimodaux, les Transformers sont devenus l’architecture dominante.

---

## 6.29. Résumé du chapitre

Dans ce chapitre, nous avons étudié les **LSTM** et les **GRU**.

Nous avons vu que ces architectures ont été conçues pour améliorer les RNN classiques, notamment face au problème des dépendances longues et du gradient qui disparaît.

Les LSTM introduisent une mémoire de cellule (c_t), distincte de l’état caché (h_t).

Ils utilisent trois portes principales :

- une porte d’oubli ;
    
- une porte d’entrée ;
    
- une porte de sortie.
    

Ces portes permettent de contrôler ce qui est conservé, ajouté ou exposé.

Les GRU proposent une version plus compacte, avec principalement :

- une porte de mise à jour ;
    
- une porte de réinitialisation.
    

Ces architectures améliorent fortement les RNN classiques.

Mais elles conservent une limite majeure :

> Elles restent fondamentalement séquentielles.

C’est cette limite qui prépare l’arrivée des architectures fondées sur l’attention, puis des Transformers.

---

## 6.30. Schéma de synthèse

```mermaid
flowchart TD
    A["RNN classique"] --> B["Mémoire fragile h_t"]
    B --> C["Dépendances longues difficiles"]
    B --> D["Gradient qui disparaît"]

    C --> E["LSTM"]
    D --> E
    E --> F["Mémoire de cellule c_t"]
    E --> G["Porte d'oubli"]
    E --> H["Porte d'entrée"]
    E --> I["Porte de sortie"]

    C --> J["GRU"]
    D --> J
    J --> K["Porte de mise à jour"]
    J --> L["Porte de réinitialisation"]

    E --> M["Meilleure mémoire"]
    J --> M

    M --> N["Mais traitement toujours séquentiel"]
    N --> O["Besoin d'attention"]
    O --> P["Transformer"]
```

---

## 6.31. Questions de compréhension

### Question 1

Pourquoi les LSTM et GRU ont-ils été proposés ?

Réponse attendue : ils ont été proposés pour améliorer les RNN classiques, notamment face aux dépendances longues et au problème du gradient qui disparaît.

### Question 2

Quelle est l’idée principale des portes ?

Réponse attendue : une porte contrôle la quantité d’information qui doit passer, être conservée, oubliée ou ajoutée.

### Question 3

Quelle est la différence principale entre un RNN classique et un LSTM ?

Réponse attendue : un LSTM possède une mémoire de cellule (c_t) et plusieurs portes de contrôle, alors qu’un RNN classique met simplement à jour un état caché (h_t).

### Question 4

Quelles sont les trois portes principales d’un LSTM ?

Réponse attendue : la porte d’oubli, la porte d’entrée et la porte de sortie.

### Question 5

Quel est le rôle de la porte d’oubli ?

Réponse attendue : elle décide quelle partie de l’ancienne mémoire doit être conservée ou supprimée.

### Question 6

Quel est le rôle de la porte d’entrée ?

Réponse attendue : elle décide quelle nouvelle information doit être ajoutée à la mémoire.

### Question 7

Quel est le rôle de la porte de sortie ?

Réponse attendue : elle décide quelle partie de la mémoire interne doit être exposée comme état caché.

### Question 8

Quelle est la différence principale entre LSTM et GRU ?

Réponse attendue : le GRU est plus compact, avec moins de portes et sans mémoire de cellule séparée.

### Question 9

Pourquoi les LSTM et GRU restent-ils limités malgré leurs améliorations ?

Réponse attendue : parce qu’ils restent séquentiels : le calcul de l’état courant dépend de l’état précédent.

### Question 10

Pourquoi cette limite est-elle importante pour les grands modèles ?

Réponse attendue : parce qu’elle limite la parallélisation sur GPU/TPU et rend plus difficile l’entraînement sur de très grandes quantités de données.

---

## 6.32. Transition vers le chapitre suivant

Nous avons maintenant compris pourquoi les LSTM et GRU ont constitué une amélioration majeure des RNN classiques.

Ils permettent de mieux contrôler la mémoire et de mieux apprendre certaines dépendances longues.

Mais ils restent prisonniers d’une contrainte fondamentale :

```txt
nous devons traiter la séquence étape par étape
```

Dans le chapitre suivant, nous allons donc étudier le **problème de la parallélisation**.

Nous verrons pourquoi le traitement séquentiel devient un obstacle majeur lorsque nous voulons entraîner des modèles très grands sur de très grands corpus, et pourquoi cette difficulté a fortement motivé l’abandon progressif de la récurrence au profit de l’attention.

---

# Chapitre 7 — Le problème de la parallélisation

## 7.1. Objectif du chapitre

Dans les chapitres précédents, nous avons étudié les limites des RNN classiques, puis les améliorations apportées par les LSTM et les GRU.

Nous avons vu que ces architectures permettent de mieux gérer la mémoire, notamment grâce à des mécanismes de portes.

Mais une limite fondamentale demeure :

> Les RNN, LSTM et GRU traitent les séquences principalement étape par étape.

Dans ce chapitre, nous allons comprendre pourquoi cette nature séquentielle devient un problème majeur lorsque nous voulons entraîner de grands modèles sur de très grandes quantités de données.

Nous allons donc étudier :

- pourquoi un RNN est difficile à paralléliser ;
    
- pourquoi les GPU et TPU préfèrent les calculs massivement parallèles ;
    
- pourquoi la dépendance (h_t \leftarrow h_{t-1}) ralentit l’entraînement ;
    
- comment cela limite le passage à l’échelle ;
    
- pourquoi les Transformers ont été conçus pour mieux exploiter le parallélisme matériel.
    

---

## 7.2. Rappel : le calcul récurrent

Dans un RNN, chaque état caché dépend de l’état précédent.

La formule générale est :

[  
h_t = f(x_t, h_{t-1})  
]

Cela signifie que, pour calculer (h_t), nous avons besoin de deux éléments :

- l’entrée actuelle (x_t) ;
    
- l’état précédent (h_{t-1}).
    

```mermaid
flowchart TD
    A["Entrée actuelle x_t"] --> C["Cellule RNN"]
    B["État précédent h_(t-1)"] --> C
    C --> D["Nouvel état h_t"]
```

Cette relation est au cœur des RNN.

Elle permet au modèle de transporter une mémoire, mais elle impose aussi un ordre strict de calcul.

---

## 7.3. Le calcul étape par étape

Un RNN doit calculer l’état (h_t) à partir de l’état (h_{t-1}).

Cela signifie que nous ne pouvons pas facilement calculer tous les états en même temps.

Pour calculer le mot 4, nous devons avoir calculé le mot 3.

Pour calculer le mot 3, nous devons avoir calculé le mot 2.

Pour calculer le mot 2, nous devons avoir calculé le mot 1.

```mermaid
flowchart LR
    X1["x1"] --> H1["h1"]
    H1 --> H2["h2"]
    X2["x2"] --> H2
    H2 --> H3["h3"]
    X3["x3"] --> H3
    H3 --> H4["h4"]
    X4["x4"] --> H4
```

Le calcul est donc fortement séquentiel.

Or, les GPU et TPU sont très efficaces quand nous pouvons faire beaucoup de calculs en parallèle.

Les RNN utilisent mal cette capacité de parallélisation.

C’est une limite majeure pour entraîner des modèles très grands.

---

## 7.4. Exemple concret avec une phrase

Prenons la phrase :

```txt
Le chat dort sur le canapé.
```

Après tokenisation, nous obtenons :

```txt
["Le", "chat", "dort", "sur", "le", "canapé", "."]
```

Le RNN calcule :

[  
h_1 = f(x_1, h_0)  
]

[  
h_2 = f(x_2, h_1)  
]

[  
h_3 = f(x_3, h_2)  
]

[  
h_4 = f(x_4, h_3)  
]

et ainsi de suite.

```mermaid
flowchart LR
    A["Le"] --> H1["h1"]
    H1 --> H2["h2"]
    B["chat"] --> H2
    H2 --> H3["h3"]
    C["dort"] --> H3
    H3 --> H4["h4"]
    D["sur"] --> H4
    H4 --> H5["h5"]
    E["le"] --> H5
    H5 --> H6["h6"]
    F["canapé"] --> H6
```

Le point important est le suivant :

> Même si nous avons tous les tokens disponibles dès le départ, le RNN ne peut pas calculer (h_6) avant d’avoir calculé (h_5).

Cette contrainte limite fortement le parallélisme.

---

## 7.5. Pourquoi le parallélisme est-il important ?

L’apprentissage profond moderne repose sur de très grands volumes de calcul.

Pour entraîner un modèle, nous devons effectuer des opérations sur :

- des millions ou milliards de paramètres ;
    
- des millions ou milliards de tokens ;
    
- de grands batches ;
    
- plusieurs couches ;
    
- plusieurs époques d’entraînement.
    

Si chaque opération doit attendre la précédente, l’entraînement devient très lent.

Au contraire, si nous pouvons effectuer beaucoup d’opérations en même temps, nous pouvons mieux exploiter le matériel.

```mermaid
flowchart TD
    A["Grand modèle"] --> B["Beaucoup de paramètres"]
    A --> C["Beaucoup de données"]
    A --> D["Beaucoup d'opérations"]

    B --> E["Besoin de parallélisme"]
    C --> E
    D --> E
```

Le parallélisme est donc une condition essentielle du passage à l’échelle.

---

## 7.6. CPU, GPU et TPU : intuition

Un CPU est très polyvalent.

Il est excellent pour exécuter des tâches variées, complexes et parfois séquentielles.

Un GPU, lui, est conçu pour exécuter beaucoup d’opérations similaires en parallèle.

Un TPU est également conçu pour accélérer massivement les calculs matriciels utilisés en apprentissage profond.

Nous pouvons simplifier ainsi :

|Matériel|Point fort|
|---|---|
|CPU|Flexibilité, logique générale, tâches séquentielles|
|GPU|Calculs parallèles massifs, matrices, tenseurs|
|TPU|Calculs tensoriels spécialisés pour le deep learning|

Les réseaux de neurones modernes utilisent énormément de multiplications de matrices.

Les GPU et TPU sont donc particulièrement adaptés.

---

## 7.7. Le calcul matriciel est naturellement parallèle

Une opération matricielle peut souvent être découpée en de très nombreux petits calculs indépendants.

Par exemple, pour multiplier deux matrices, chaque élément du résultat peut être calculé à partir d’une combinaison de lignes et de colonnes.

```mermaid
flowchart TD
    A["Matrice A"] --> C["Multiplication matricielle"]
    B["Matrice B"] --> C
    C --> D["Matrice résultat"]
    C --> E["Beaucoup d'opérations parallèles"]
```

Les GPU excellent dans ce type d’opérations.

C’est pourquoi les architectures qui transforment le maximum de calculs en opérations matricielles parallèles sont très efficaces.

Les Transformers reposent beaucoup sur cette idée.

---

## 7.8. Le problème particulier des RNN

Dans un RNN, même si chaque étape contient des multiplications de matrices, les étapes elles-mêmes dépendent les unes des autres.

Nous pouvons paralléliser une partie du calcul interne à une cellule, mais pas facilement la progression temporelle complète.

```mermaid
flowchart TD
    A["Calcul interne de h_t"] --> B["Partiellement parallélisable"]
    C["Dépendance h_t après h_(t-1)"] --> D["Difficile à paralléliser"]
```

Autrement dit :

- le calcul à l’intérieur d’un pas temporel peut utiliser le GPU ;
    
- mais le passage de (h_1) à (h_2), puis (h_3), puis (h_4), reste séquentiel.
    

C’est ce qui limite fortement les RNN sur de longues séquences.

---

## 7.9. Exemple avec une séquence longue

Supposons que nous ayons une séquence de 1 000 tokens.

Un RNN doit effectuer 1 000 étapes successives :

```txt
h1 → h2 → h3 → ... → h1000
```

```mermaid
flowchart LR
    H1["h1"] --> H2["h2"]
    H2 --> H3["h3"]
    H3 --> H4["..."]
    H4 --> H1000["h1000"]
```

Même si chaque étape est rapide, l’accumulation de 1 000 dépendances successives devient coûteuse.

Le problème est encore plus fort si nous empilons plusieurs couches récurrentes.

```mermaid
flowchart TD
    A["Séquence de 1000 tokens"] --> B["Couche RNN 1"]
    B --> C["Couche RNN 2"]
    C --> D["Couche RNN 3"]
    D --> E["Beaucoup d'étapes séquentielles"]
```

Le temps d’entraînement augmente fortement avec la longueur de séquence.

---

## 7.10. Batch parallèle, temps séquentiel

Il faut être précis : les RNN peuvent quand même exploiter une forme de parallélisme.

Nous pouvons traiter plusieurs séquences en batch.

Par exemple, nous pouvons traiter 64 phrases en même temps.

```mermaid
flowchart TD
    A["Batch de 64 phrases"] --> B["RNN"]
    B --> C["Calcul parallèle entre exemples du batch"]
```

Mais à l’intérieur de chaque phrase, la progression temporelle reste séquentielle.

```mermaid
flowchart LR
    A["Phrase 1 : h1"] --> B["Phrase 1 : h2"] --> C["Phrase 1 : h3"]
    D["Phrase 2 : h1"] --> E["Phrase 2 : h2"] --> F["Phrase 2 : h3"]
```

Donc nous avons :

- parallélisme entre exemples du batch ;
    
- mais dépendance séquentielle dans chaque exemple.
    

C’est mieux que rien, mais insuffisant pour exploiter pleinement les accélérateurs modernes.

---

## 7.11. Le problème de la latence

La nature séquentielle des RNN pose aussi un problème de latence.

Si nous devons attendre chaque étape avant de calculer la suivante, nous accumulons du temps d’attente.

Pour une phrase courte, cela peut être acceptable.

Pour une longue séquence, cela devient problématique.

```mermaid
gantt
    title Calcul séquentiel simplifié d'un RNN
    dateFormat X
    axisFormat %s

    section RNN
    Calcul h1 :0, 1
    Calcul h2 :1, 1
    Calcul h3 :2, 1
    Calcul h4 :3, 1
    Calcul h5 :4, 1
```

Chaque calcul commence après le précédent.

Nous ne pouvons pas les lancer tous simultanément.

---

## 7.12. Parallélisme idéal

Dans un modèle plus parallélisable, nous voudrions calculer plusieurs représentations en même temps.

```mermaid
gantt
    title Calcul parallèle simplifié
    dateFormat X
    axisFormat %s

    section Calcul parallèle
    Calcul position 1 :0, 1
    Calcul position 2 :0, 1
    Calcul position 3 :0, 1
    Calcul position 4 :0, 1
    Calcul position 5 :0, 1
```

Toutes les positions commencent au même moment.

Ce type de structure exploite beaucoup mieux les GPU et TPU.

C’est précisément l’un des grands avantages des Transformers pendant l’entraînement.

---

## 7.13. Pourquoi les Transformers sont plus parallélisables ?

Dans un Transformer, nous ne calculons pas un état caché (h_t) à partir de (h_{t-1}).

Nous partons d’une matrice d’embeddings pour toute la séquence.

Par exemple, une phrase de (n) tokens devient :

[  
X \in \mathbb{R}^{n \times d_{model}}  
]

Nous pouvons projeter tous les tokens en même temps vers des représentations internes.

```mermaid
flowchart TD
    A["Tous les embeddings de la séquence"] --> B["Projections matricielles"]
    B --> C["Q, K, V pour tous les tokens"]
    C --> D["Attention"]
```

Les calculs principaux se font sous forme d’opérations matricielles.

Cela permet une parallélisation massive.

---

## 7.14. Différence clé : dépendance temporelle contre dépendance matricielle

Dans un RNN :

[  
h_t = f(x_t, h_{t-1})  
]

Le calcul de (h_t) dépend du résultat précédent.

Dans un Transformer, pour une couche donnée, les représentations des positions peuvent être calculées ensemble à partir de matrices.

```mermaid
flowchart TD
    A["RNN"] --> B["Dépendance temporelle"]
    B --> C["Calcul étape par étape"]

    D["Transformer"] --> E["Dépendance par attention"]
    E --> F["Calcul matriciel parallèle"]
```

Le Transformer a bien sûr des dépendances entre couches : la couche 2 dépend de la couche 1.

Mais à l’intérieur d’une couche, toutes les positions peuvent être traitées beaucoup plus simultanément.

---

## 7.15. Comparaison visuelle

## RNN

```mermaid
flowchart LR
    X1["x1"] --> H1["h1"]
    H1 --> H2["h2"]
    X2["x2"] --> H2
    H2 --> H3["h3"]
    X3["x3"] --> H3
    H3 --> H4["h4"]
    X4["x4"] --> H4
```

## Transformer

```mermaid
flowchart TD
    X1["x1"] --> A["Self-attention"]
    X2["x2"] --> A
    X3["x3"] --> A
    X4["x4"] --> A

    A --> Y1["y1"]
    A --> Y2["y2"]
    A --> Y3["y3"]
    A --> Y4["y4"]
```

Dans le RNN, la progression est linéaire.

Dans le Transformer, la couche d’attention reçoit toute la séquence et produit toutes les représentations contextualisées.

---

## 7.16. Le coût caché du Transformer

Il ne faut pas croire que le Transformer est simplement moins coûteux.

Il est plus parallélisable, mais son attention complète a un coût important.

Si chaque token regarde tous les autres, nous construisons une matrice d’attention de taille :

[  
n \times n  
]

où (n) est la longueur de la séquence.

```mermaid
flowchart TD
    A["n tokens"] --> B["Chaque token regarde tous les autres"]
    B --> C["Matrice d'attention n x n"]
    C --> D["Coût O(n²)"]
```

Donc les Transformers ont une autre limite :

> Ils sont très parallélisables, mais leur coût mémoire et calculatoire augmente rapidement avec la longueur de contexte.

Nous reviendrons sur ce problème plus loin dans le cours.

---

## 7.17. Parallélisation et entraînement des grands modèles

Pourquoi cette différence a-t-elle été aussi importante historiquement ?

Parce que les grands modèles modernes sont entraînés sur des volumes massifs de données.

Pour entraîner un grand modèle de langage, nous devons traiter énormément de tokens.

Si le modèle est fortement séquentiel, l’entraînement devient trop lent.

Si le modèle est fortement parallélisable, nous pouvons mieux utiliser :

- plusieurs GPU ;
    
- plusieurs TPU ;
    
- des calculs matriciels optimisés ;
    
- des batches plus grands ;
    
- du data parallelism ;
    
- du tensor parallelism ;
    
- du pipeline parallelism.
    

```mermaid
flowchart TD
    A["Architecture parallélisable"] --> B["Meilleure utilisation GPU/TPU"]
    B --> C["Entraînement plus rapide"]
    C --> D["Modèles plus grands"]
    D --> E["Plus de données"]
    E --> F["LLM modernes"]
```

La parallélisation a donc été une condition technique importante de l’explosion des Transformers.

---

## 7.18. La différence entre entraînement et génération

Nous devons cependant distinguer deux situations :

1. l’entraînement ;
    
2. la génération autoregressive.
    

Pendant l’entraînement d’un Transformer decoder-only, nous pouvons calculer les prédictions pour tous les tokens d’une séquence en parallèle grâce au masque causal.

Par exemple, pour :

```txt
Le chat dort sur le canapé
```

le modèle apprend à prédire :

```txt
chat, dort, sur, le, canapé
```

en une seule passe parallèle.

```mermaid
flowchart TD
    A["Séquence complète"] --> B["Transformer avec masque causal"]
    B --> C["Prédictions pour toutes les positions"]
```

Mais pendant la génération, nous produisons souvent les tokens un par un.

```mermaid
flowchart LR
    A["Le"] --> B["prédit : chat"]
    B --> C["prédit : dort"]
    C --> D["prédit : sur"]
    D --> E["..."]
```

Donc les Transformers sont très parallélisables à l’entraînement, mais la génération autoregressive reste en partie séquentielle.

C’est une nuance importante.

---

## 7.19. Pourquoi le masque causal permet l’entraînement parallèle ?

Dans un modèle autoregressif, le token à la position (t) ne doit pas voir les tokens futurs.

Par exemple, pour prédire `dort`, le modèle peut voir :

```txt
Le chat
```

mais pas :

```txt
sur le canapé
```

Le masque causal interdit l’accès au futur.

```mermaid
flowchart TD
    A["Séquence complète disponible"] --> B["Masque causal"]
    B --> C["Chaque position voit seulement le passé"]
    C --> D["Calcul parallèle des prédictions"]
```

Grâce à ce masque, nous pouvons fournir toute la séquence au modèle pendant l’entraînement, tout en respectant la logique autoregressive.

Cela permet de calculer beaucoup de prédictions en parallèle.

---

## 7.0. Exemple comparatif d’entraînement

## RNN génératif

Pour entraîner un RNN génératif, les états doivent être calculés dans l’ordre :

```mermaid
flowchart LR
    A["Le"] --> H1["h1"]
    H1 --> H2["h2"]
    B["chat"] --> H2
    H2 --> H3["h3"]
    C["dort"] --> H3
    H3 --> H4["h4"]
    D["sur"] --> H4
```

## Transformer génératif

Pour entraîner un Transformer génératif, nous pouvons traiter toute la séquence en parallèle avec un masque causal :

```mermaid
flowchart TD
    A["Le"] --> T["Transformer masqué"]
    B["chat"] --> T
    C["dort"] --> T
    D["sur"] --> T

    T --> P1["prédit chat"]
    T --> P2["prédit dort"]
    T --> P3["prédit sur"]
    T --> P4["prédit ..."]
```

Cette différence rend les Transformers beaucoup plus efficaces à entraîner sur de grands corpus.

---

## 7.21. Parallélisme et taille des batches

Un autre avantage des Transformers est qu’ils s’intègrent bien aux gros batches.

Un batch peut être représenté par un tenseur :

[  
B \times n \times d_{model}  
]

où :

- (B) est la taille du batch ;
    
- (n) est la longueur de séquence ;
    
- (d_{model}) est la dimension du modèle.
    

```mermaid
flowchart TD
    A["Batch de séquences"] --> B["Tenseur B x n x d_model"]
    B --> C["Calculs matriciels massifs"]
    C --> D["GPU/TPU bien utilisés"]
```

Les calculs d’attention, de projections linéaires et de feed-forward networks s’expriment naturellement en opérations tensorisées.

Cela rend l’architecture très compatible avec les bibliothèques modernes comme PyTorch, TensorFlow ou JAX.

---

## 7.22. La parallélisation entre couches

Même dans un Transformer, toutes les opérations ne sont pas parallélisables.

La couche 2 dépend de la sortie de la couche 1.

La couche 3 dépend de la sortie de la couche 2.

```mermaid
flowchart TD
    A["Entrée"] --> B["Couche Transformer 1"]
    B --> C["Couche Transformer 2"]
    C --> D["Couche Transformer 3"]
    D --> E["Sortie"]
```

Donc il reste une profondeur séquentielle.

Mais la différence avec les RNN est que cette séquentialité dépend du nombre de couches, pas directement du nombre de tokens.

Dans un RNN, la profondeur temporelle dépend de la longueur de la séquence.

Dans un Transformer, pour une couche donnée, les tokens sont traités ensemble.

---

## 7.23. Comparaison : longueur de séquence et profondeur de calcul

Supposons une séquence de 1 000 tokens.

Un RNN doit traverser 1 000 étapes temporelles.

Un Transformer avec 12 couches doit traverser 12 couches.

Bien sûr, chaque couche Transformer coûte plus cher, notamment à cause de l’attention (O(n^2)).

Mais la profondeur séquentielle liée aux tokens est beaucoup plus faible.

|Architecture|Dépendance séquentielle principale|
|---|---|
|RNN|Longueur de séquence|
|LSTM / GRU|Longueur de séquence|
|Transformer|Nombre de couches|

Cela explique pourquoi les Transformers peuvent mieux exploiter le calcul parallèle.

---

## 7.24. Impact sur la recherche en NLP

Cette capacité de parallélisation a changé l’échelle des modèles entraînables.

Avant les Transformers, les modèles récurrents étaient performants, mais plus difficiles à entraîner massivement.

Avec les Transformers, il est devenu plus réaliste d’entraîner des modèles :

- plus profonds ;
    
- plus larges ;
    
- sur plus de données ;
    
- avec plus de contexte ;
    
- sur des infrastructures distribuées.
    

```mermaid
flowchart LR
    A["RNN / LSTM"] --> B["Bon traitement séquentiel"]
    B --> C["Mais parallélisation limitée"]

    D["Transformer"] --> E["Attention parallélisable"]
    E --> F["Passage à l'échelle"]
    F --> G["BERT, GPT, T5, LLM"]
```

La rupture n’est donc pas seulement conceptuelle.

Elle est aussi matérielle et industrielle.

---

## 7.25. Attention et opérations matricielles

Dans l’attention, nous calculons notamment :

[  
QK^T  
]

où :

- (Q) représente les queries ;
    
- (K) représente les keys ;
    
- (V) représente les values.
    

Même sans encore entrer dans les détails, nous devons retenir que ce calcul est une multiplication matricielle.

```mermaid
flowchart LR
    Q["Matrice Q"] --> M["QK^T"]
    K["Matrice K"] --> M
    M --> S["Scores d'attention"]
    S --> V["Combinaison avec V"]
```

Les multiplications matricielles sont précisément le type d’opérations très bien optimisées sur GPU et TPU.

C’est une des raisons pratiques du succès des Transformers.

---

## 7.26. Mais pourquoi les RNN ne peuvent-ils pas faire pareil ?

Un RNN utilise aussi des matrices.

Par exemple :

[  
W_x x_t + W_h h_{t-1}  
]

Mais le problème est la dépendance sur (h_{t-1}).

Même si la multiplication matricielle est rapide, nous devons attendre le résultat précédent.

```mermaid
flowchart TD
    A["Multiplication rapide à l'étape t"] --> B["Produit h_t"]
    B --> C["Nécessaire pour l'étape t+1"]
    C --> D["Attente séquentielle"]
```

Le goulot d’étranglement n’est donc pas seulement le type d’opération.

C’est la structure de dépendance entre les opérations.

---

## 7.27. Analogie : chaîne de montage contre équipe parallèle

Nous pouvons utiliser une analogie.

Un RNN ressemble à une chaîne où chaque personne doit attendre le travail de la personne précédente.

```mermaid
flowchart LR
    A["Personne 1"] --> B["Personne 2"]
    B --> C["Personne 3"]
    C --> D["Personne 4"]
```

Si la personne 2 n’a pas terminé, la personne 3 ne peut pas commencer.

Un Transformer ressemble davantage à une équipe qui reçoit toutes les informations en même temps et calcule les relations entre elles.

```mermaid
flowchart TD
    A["Information 1"] --> E["Équipe"]
    B["Information 2"] --> E
    C["Information 3"] --> E
    D["Information 4"] --> E
    E --> F["Résultat collectif"]
```

Cette analogie simplifie beaucoup, mais elle capture bien l’idée :

> Les Transformers permettent davantage de travail simultané.

---

## 7.28. Conséquence sur l’entraînement de modèles très grands

Quand nous entraînons un modèle de quelques milliers ou millions de paramètres, la parallélisation est déjà utile.

Mais quand nous passons à des centaines de millions ou milliards de paramètres, elle devient indispensable.

Un modèle très grand nécessite :

- beaucoup d’opérations ;
    
- beaucoup de mémoire ;
    
- beaucoup de données ;
    
- beaucoup de temps d’entraînement.
    

Sans parallélisme, l’entraînement devient rapidement impraticable.

```mermaid
flowchart TD
    A["Modèle très grand"] --> B["Coût énorme"]
    B --> C["Besoin de GPU/TPU"]
    C --> D["Besoin d'opérations parallèles"]
    D --> E["Architecture adaptée : Transformer"]
```

C’est pourquoi le passage des RNN aux Transformers est aussi un passage vers une architecture mieux adaptée aux infrastructures modernes.

---

##7.29. Limite actuelle : les longues séquences

Même si les Transformers sont plus parallélisables, ils ont un problème avec les très longues séquences.

La self-attention complète demande de comparer tous les tokens entre eux.

Pour (n) tokens, cela donne :

[  
n^2  
]

relations.

Si (n = 1,000), cela fait :

[  
1,000,000  
]

relations.

Si (n = 10,000), cela fait :

[  
100,000,000  
]

relations.

```mermaid
flowchart TD
    A["Longueur n"] --> B["Relations d'attention n²"]
    B --> C["n=1 000 → 1 000 000"]
    B --> D["n=10 000 → 100 000 000"]
    C --> E["Coût mémoire"]
    D --> E
```

Cela explique pourquoi de nombreuses recherches cherchent à optimiser l’attention :

- attention sparse ;
    
- attention locale ;
    
- sliding window attention ;
    
- FlashAttention ;
    
- linear attention ;
    
- architectures hybrides ;
    
- state-space models.
    

Nous reviendrons sur ces sujets plus loin dans le cours.

---

## 7.30. Le compromis fondamental

Nous pouvons maintenant formuler le compromis.

Les RNN ont un coût séquentiel important, mais ils traitent chaque étape de manière relativement locale.

Les Transformers sont beaucoup plus parallélisables, mais l’attention complète coûte cher pour les longues séquences.

|Architecture|Avantage|Limite|
|---|---|---|
|RNN / LSTM / GRU|Mémoire séquentielle naturelle|Faible parallélisation|
|Transformer|Forte parallélisation|Attention coûteuse en (O(n^2))|

Le succès des Transformers vient du fait que, pour beaucoup de tâches modernes, la parallélisation massive compense largement le coût de l’attention.

---

## 7.31. Résumé du chapitre

Dans ce chapitre, nous avons étudié le problème de la parallélisation.

Nous avons vu qu’un RNN calcule chaque état caché à partir de l’état précédent :

[  
h_t = f(x_t, h_{t-1})  
]

Cette dépendance impose un traitement étape par étape.

Même si nous pouvons paralléliser certains calculs internes et traiter plusieurs séquences en batch, la progression temporelle reste séquentielle.

Cette contrainte limite l’utilisation efficace des GPU et TPU.

Or, les grands modèles modernes nécessitent énormément de calcul et doivent donc exploiter le parallélisme matériel.

Les Transformers répondent à cette limite en remplaçant la récurrence par des opérations d’attention et des calculs matriciels appliqués à toute la séquence.

Cela rend l’entraînement beaucoup plus parallélisable.

Mais cette parallélisation a un coût : l’attention complète nécessite une matrice de taille (n \times n), donc un coût quadratique en longueur de séquence.

---

## 7.32. Schéma de synthèse

```mermaid
flowchart TD
    A["RNN / LSTM / GRU"] --> B["h_t dépend de h_(t-1)"]
    B --> C["Calcul étape par étape"]
    C --> D["Faible parallélisation temporelle"]
    D --> E["Mauvaise exploitation GPU/TPU"]
    E --> F["Passage à l'échelle difficile"]

    G["Transformer"] --> H["Toute la séquence en entrée"]
    H --> I["Calculs matriciels"]
    I --> J["Self-attention"]
    J --> K["Forte parallélisation"]
    K --> L["Entraînement de grands modèles"]

    J --> M["Coût O(n²)"]
    M --> N["Limite pour longues séquences"]
```

---

## 7.33. Questions de compréhension

### Question 1

Pourquoi un RNN est-il difficile à paralléliser sur la dimension temporelle ?

Réponse attendue : parce que le calcul de (h_t) dépend de (h_{t-1}), donc il faut calculer les états dans l’ordre.

### Question 2

Quelle est la différence entre paralléliser un batch et paralléliser une séquence ?

Réponse attendue : paralléliser un batch signifie traiter plusieurs exemples en même temps ; paralléliser une séquence signifie traiter plusieurs positions d’une même séquence en même temps. Les RNN parallélisent mieux le batch que la séquence.

### Question 3

Pourquoi les GPU et TPU sont-ils importants pour les grands modèles ?

Réponse attendue : parce qu’ils accélèrent massivement les calculs matriciels et tensoriels utilisés dans l’apprentissage profond.

### Question 4

Pourquoi les Transformers exploitent-ils mieux les GPU/TPU ?

Réponse attendue : parce qu’ils utilisent de grandes opérations matricielles sur toute la séquence, notamment dans les projections et l’attention.

### Question 5

Pourquoi un Transformer n’est-il pas totalement sans contrainte séquentielle ?

Réponse attendue : parce que les couches restent empilées : la couche suivante dépend de la couche précédente. En génération autoregressive, les tokens sont aussi produits un par un.

### Question 6

Quelle est la différence importante entre entraînement et génération autoregressive ?

Réponse attendue : pendant l’entraînement, les prédictions de plusieurs positions peuvent être calculées en parallèle grâce au masque causal ; pendant la génération, les tokens sont souvent produits un par un.

### Question 7

Quelle est la principale limite de la self-attention complète ?

Réponse attendue : son coût mémoire et calculatoire augmente en (O(n^2)) avec la longueur de la séquence.

### Question 8

Pourquoi cette question de parallélisation a-t-elle été décisive pour les LLM ?

Réponse attendue : parce que les LLM nécessitent un entraînement massif sur énormément de tokens, ce qui impose une architecture capable d’exploiter fortement les GPU/TPU.

---

## 34. Transition vers le chapitre suivant

Nous avons maintenant compris une limite fondamentale des RNN, LSTM et GRU : leur traitement séquentiel freine la parallélisation.

Dans le chapitre suivant, nous allons étudier les **modèles Seq2Seq**, qui ont joué un rôle central avant les Transformers, notamment en traduction automatique.

Nous verrons comment un encodeur lit une séquence source, comment un décodeur produit une séquence cible, et pourquoi le vecteur de contexte unique est devenu un goulot d’étranglement.

Ce sera l’étape nécessaire avant d’introduire l’attention comme mécanisme de dépassement.

---

# 8. Les modèles Seq2Seq

Avant les Transformers, une architecture très utilisée pour la traduction automatique était le modèle **sequence-to-sequence**, ou **Seq2Seq**.

L’idée est simple :

- un encodeur lit la phrase source ;
    
- il produit une représentation ;
    
- un décodeur génère la phrase cible.
    

Par exemple :

```txt
Source : I love machine learning.
Cible  : J'aime l'apprentissage automatique.
```

```mermaid
flowchart LR
    A["Phrase source"] --> B["Encodeur RNN"]
    B --> C["Vecteur de contexte"]
    C --> D["Décodeur RNN"]
    D --> E["Phrase cible"]
```

Le problème est que toute la phrase source doit être compressée dans un seul vecteur de contexte.

Pour une phrase courte, cela peut fonctionner.

Pour une phrase longue, c’est beaucoup plus difficile.

---

# 9. Le goulot d’étranglement du vecteur de contexte

Dans les premiers modèles Seq2Seq, l’encodeur devait résumer toute la phrase dans un vecteur unique.

Imaginons que nous devions traduire :

```txt
Although the committee had initially rejected the proposal, it later accepted a revised version after several months of discussion.
```

Il est difficile de condenser toutes les informations importantes dans une seule représentation fixe.

```mermaid
flowchart LR
    A["Phrase longue source"] --> B["Encodeur"]
    B --> C["Petit vecteur de contexte"]
    C --> D["Décodeur"]
    D --> E["Traduction"]
```

Ce vecteur devient un **goulot d’étranglement informationnel**.

Le décodeur doit générer une phrase complète à partir d’un résumé compact, alors qu’il aurait parfois besoin de regarder directement certaines parties précises de la phrase source.

C’est ici que l’attention va devenir importante.

---

# 10. L’arrivée de l’attention

L’idée de l’attention est de permettre au décodeur de ne pas dépendre uniquement d’un seul vecteur global.

Au lieu de cela, à chaque étape de génération, le décodeur peut regarder différentes parties de la phrase source.

```mermaid
flowchart TD
    A["Phrase source"] --> E1["Représentation mot 1"]
    A --> E2["Représentation mot 2"]
    A --> E3["Représentation mot 3"]
    A --> E4["Représentation mot 4"]

    D["Décodeur"] --> W1["Poids attention mot 1"]
    D --> W2["Poids attention mot 2"]
    D --> W3["Poids attention mot 3"]
    D --> W4["Poids attention mot 4"]

    E1 --> C["Contexte dynamique"]
    E2 --> C
    E3 --> C
    E4 --> C

    W1 --> C
    W2 --> C
    W3 --> C
    W4 --> C
```

Nous pouvons décrire l’attention ainsi :

> L’attention permet au modèle de sélectionner dynamiquement les informations utiles dans une séquence.

Dans une tâche de traduction, lorsqu’il génère un mot français, le modèle peut regarder les mots anglais les plus pertinents.

Par exemple :

```txt
The black cat sleeps.
Le chat noir dort.
```

Quand le modèle génère :

```txt
chat
```

il doit surtout regarder :

```txt
cat
```

Quand il génère :

```txt
noir
```

il doit surtout regarder :

```txt
black
```

---

# 11. L’attention comme alignement

Dans la traduction automatique, l’attention peut être vue comme une forme d’alignement entre les mots source et les mots cible.

```mermaid
flowchart LR
    A1["The"] -.-> B1["Le"]
    A2["black"] -.-> B3["noir"]
    A3["cat"] -.-> B2["chat"]
    A4["sleeps"] -.-> B4["dort"]
```

Ce point est important historiquement.

Avant les modèles neuronaux modernes, la traduction automatique utilisait souvent des mécanismes explicites d’alignement statistique.

L’attention a permis de retrouver une forme d’alignement, mais apprise automatiquement par le réseau.

---

# 12. Première rupture : l’attention améliore les RNN

Dans un premier temps, l’attention n’a pas remplacé les RNN.

Elle les a complétés.

Nous avions donc des architectures de ce type :

```mermaid
flowchart LR
    A["Phrase source"] --> B["Encodeur RNN"]
    B --> C["États cachés de chaque token"]
    C --> D["Mécanisme d'attention"]
    D --> E["Décodeur RNN"]
    E --> F["Phrase cible"]
```

Cela a permis de grandes améliorations, car le décodeur pouvait accéder à tous les états de l’encodeur, et pas seulement au dernier.

Mais le modèle restait encore partiellement séquentiel.

L’encodeur était souvent récurrent.

Le décodeur était récurrent.

La parallélisation restait limitée.

---

# 13. La question centrale

À ce stade, une question devient naturelle :

> Si l’attention est si utile, avons-nous encore besoin des RNN ?

C’est exactement la rupture proposée par le papier **Attention Is All You Need**.

L’idée fondamentale est :

> Nous pouvons construire un modèle de séquence uniquement à partir de mécanismes d’attention, sans récurrence et sans convolution.

Autrement dit, au lieu de lire la phrase mot par mot, nous la traitons globalement.

---

# 14. La rupture Transformer

Le Transformer remplace le traitement séquentiel par un traitement fondé sur l’attention entre tous les tokens.

Dans un RNN, chaque token dépend surtout de l’état précédent.

Dans un Transformer, chaque token peut directement interagir avec tous les autres tokens.

```mermaid
flowchart TD
    T1["Token 1"] <--> T2["Token 2"]
    T1 <--> T3["Token 3"]
    T1 <--> T4["Token 4"]
    T2 <--> T3
    T2 <--> T4
    T3 <--> T4
```

Cela change profondément la manière de traiter les séquences.

Nous ne sommes plus dans une lecture strictement linéaire.

Nous sommes dans une mise en relation globale.

---

# 15. Exemple intuitif

Prenons la phrase :

```txt
La souris que le chat poursuit court très vite.
```

Le mot :

```txt
court
```

doit être relié à :

```txt
La souris
```

et non à :

```txt
le chat
```

Un Transformer peut apprendre à faire regarder le token `court` vers les tokens utiles :

```mermaid
flowchart LR
    A["La souris"] -. "forte attention" .-> D["court"]
    B["le chat"] -. "attention plus faible" .-> D
    C["poursuit"] -. "contexte" .-> D
```

Le modèle apprend donc des relations grammaticales, sémantiques et contextuelles à partir des données.

---

# 16. Comparaison RNN vs Transformer

Nous pouvons comparer les deux approches simplement.

|Critère|RNN / LSTM / GRU|Transformer|
|---|---|---|
|Traitement|Séquentiel|Global et parallélisable|
|Dépendances longues|Difficiles|Plus directes|
|Parallélisation|Limitée|Très forte|
|Mémoire du contexte|Compressée dans des états successifs|Relations directes entre tokens|
|Architecture dominante aujourd’hui|Moins utilisée pour NLP massif|Dominante dans les LLM|

Le point clé est le suivant :

> Le Transformer rend beaucoup plus efficace l’apprentissage sur de grands corpus grâce à sa parallélisation et à son accès direct aux relations entre tokens.

---

# 17. Ce que le Transformer gagne

Le Transformer apporte plusieurs avantages majeurs.

## 17.1 Meilleure parallélisation

Comme tous les tokens d’une séquence peuvent être traités en même temps dans certaines parties du modèle, l’entraînement devient beaucoup plus efficace sur GPU ou TPU.

```mermaid
flowchart LR
    A["Token 1"] --> P["Traitement parallèle"]
    B["Token 2"] --> P
    C["Token 3"] --> P
    D["Token 4"] --> P
    P --> O["Représentations contextualisées"]
```

Cela permet d’entraîner des modèles plus grands sur davantage de données.

---

## 17.2 Meilleure gestion des dépendances longues

Dans un Transformer, deux tokens éloignés peuvent interagir directement via l’attention.

```mermaid
flowchart LR
    A["Début de phrase"] -. "attention directe" .-> Z["Fin de phrase"]
```

Dans un RNN, l’information doit passer par tous les états intermédiaires.

```mermaid
flowchart LR
    A["Début"] --> B["..."]
    B --> C["..."]
    C --> D["..."]
    D --> Z["Fin"]
```

La différence est essentielle.

---

## 17.3 Représentations contextualisées

Dans un Transformer, le vecteur associé à un mot dépend des autres mots de la phrase.

Le mot `banque` n’a pas la même représentation dans :

```txt
Je vais à la banque déposer un chèque.
```

et :

```txt
Nous nous asseyons sur la banque au bord de la rivière.
```

Même mot, mais contexte différent.

Le Transformer produit donc une représentation contextualisée.

```mermaid
flowchart TD
    A["banque + argent + chèque"] --> B["Sens financier"]
    C["banque + rivière + bord"] --> D["Sens géographique"]
```

---

# 18. Ce que le Transformer perd ou complique

Il ne faut pas présenter les Transformers comme une solution magique.

Ils ont aussi des limites.

## 18.1 Coût quadratique de l’attention

Si chaque token regarde tous les autres tokens, alors le nombre de relations à calculer augmente très vite.

Pour une séquence de longueur $n$, la matrice d’attention contient :

[  
n \times n  
]

relations.

```mermaid
flowchart TD
    A["n tokens"] --> B["Matrice d'attention n x n"]
    B --> C["Coût mémoire important"]
    B --> D["Coût calculatoire important"]
```

C’est pourquoi les longues séquences sont coûteuses.

Nous reviendrons sur ce point dans le chapitre consacré à la complexité.

---

## 18.2 Absence d’ordre naturel

Un RNN lit les mots dans l’ordre. L’ordre est donc intégré naturellement dans le processus.

Un Transformer, lui, regarde tous les tokens en parallèle.

Il faut donc lui ajouter explicitement une information de position.

```mermaid
flowchart LR
    A["Token embedding"] --> C["Entrée Transformer"]
    B["Position encoding"] --> C
```

Sans information de position, le Transformer ne saurait pas distinguer :

```txt
Le chien mord l’homme.
```

de :

```txt
L’homme mord le chien.
```

Nous étudierons ce problème en détail dans le chapitre 3.

---

## 18.3 Besoin massif de données

Les Transformers modernes sont très puissants, mais ils nécessitent souvent :

- beaucoup de données ;
    
- beaucoup de calcul ;
    
- beaucoup de mémoire ;
    
- une infrastructure matérielle importante.
    

C’est particulièrement vrai pour les grands modèles de langage.

---

# 19. Transformer et changement d’échelle

Le succès des Transformers ne vient pas seulement de leur élégance théorique.

Il vient aussi de leur compatibilité avec le passage à l’échelle.

Autrement dit, les Transformers se sont révélés très efficaces lorsque nous augmentons :

- la taille des données ;
    
- la taille du modèle ;
    
- la durée d’entraînement ;
    
- la puissance de calcul.
    

```mermaid
flowchart TD
    A["Architecture parallélisable"] --> B["Entraînement sur GPU/TPU"]
    B --> C["Plus de données"]
    C --> D["Modèles plus grands"]
    D --> E["Meilleures performances"]
    E --> F["LLM modernes"]
```

C’est ce qui a permis l’émergence des grands modèles de langage modernes.

---

# 20. Les grandes étapes historiques

Nous pouvons résumer l’évolution des architectures de séquences ainsi :

```mermaid
timeline
    title Évolution des modèles de séquences
    RNN : Lecture séquentielle simple
    LSTM / GRU : Meilleure mémoire
    Seq2Seq : Traduction neuronale
    Attention : Accès dynamique au contexte
    Transformer : Attention sans récurrence
    BERT / GPT : Pré-entraînement massif
    LLM modernes : Passage à l'échelle
```

Cette progression montre que les Transformers ne sont pas apparus de nulle part.

Ils sont la réponse à plusieurs limites accumulées :

- les RNN traitaient mal les dépendances longues ;
    
- les modèles Seq2Seq compressaient trop l’information ;
    
- les modèles récurrents étaient difficiles à paralléliser ;
    
- l’attention avait déjà montré son efficacité ;
    
- les GPU/TPU favorisaient les architectures parallélisables.
    

---

# 21. L’idée centrale du chapitre

Nous pouvons maintenant formuler l’idée centrale :

> Le Transformer est une architecture conçue pour traiter les séquences en reliant directement les éléments entre eux grâce à l’attention, au lieu de les traiter uniquement dans un ordre séquentiel.

Cela permet :

- de mieux capturer les dépendances longues ;
    
- de paralléliser fortement l’entraînement ;
    
- de produire des représentations contextualisées ;
    
- de construire des modèles très grands ;
    
- de généraliser l’architecture à de nombreux domaines.
    

---

# 22. Exemple global : d’une phrase à une représentation contextualisée

Prenons la phrase :

```txt
Le chat noir dort sur le canapé.
```

Un Transformer ne se contente pas d’associer un vecteur fixe à chaque mot.

Il construit une représentation de chaque mot en fonction de tous les autres.

```mermaid
flowchart TD
    A["Le"] --> T["Transformer"]
    B["chat"] --> T
    C["noir"] --> T
    D["dort"] --> T
    E["sur"] --> T
    F["le"] --> T
    G["canapé"] --> T

    T --> A2["Représentation contextualisée de Le"]
    T --> B2["Représentation contextualisée de chat"]
    T --> C2["Représentation contextualisée de noir"]
    T --> D2["Représentation contextualisée de dort"]
    T --> G2["Représentation contextualisée de canapé"]
```

Le mot `chat` sera influencé par :

- `Le`, qui indique le déterminant ;
    
- `noir`, qui apporte une propriété ;
    
- `dort`, qui indique l’action ;
    
- `canapé`, qui donne le contexte de la scène.
    

---

# 23. Attention : ce que nous ne savons pas encore

À ce stade du cours, nous comprenons pourquoi les Transformers sont nécessaires, mais nous n’avons pas encore détaillé comment ils fonctionnent.

Nous ne savons pas encore précisément :

- comment un mot devient un vecteur ;
    
- comment le modèle encode l’ordre ;
    
- ce que sont Query, Key et Value ;
    
- comment l’attention est calculée ;
    
- pourquoi on parle de multi-head attention ;
    
- comment fonctionne un bloc encoder ;
    
- comment fonctionne un bloc decoder ;
    
- comment le modèle est entraîné.
    

Ces éléments seront construits progressivement dans les chapitres suivants.

---

# 24. Résumé du chapitre

Nous avons vu que les Transformers répondent à plusieurs limites des architectures précédentes.

Les **RNN** lisent les séquences pas à pas, ce qui rend les dépendances longues difficiles et limite la parallélisation.

Les **LSTM** et **GRU** améliorent la mémoire, mais restent fondamentalement séquentiels.

Les modèles **Seq2Seq** ont permis de grandes avancées, notamment en traduction automatique, mais ils souffraient du goulot d’étranglement du vecteur de contexte.

L’**attention** a permis au modèle de sélectionner dynamiquement les parties pertinentes d’une séquence.

Le **Transformer** pousse cette idée plus loin :

> Nous supprimons la récurrence et nous construisons une architecture entièrement fondée sur l’attention.

Cela permet de traiter les séquences de manière plus globale, plus parallélisable et plus adaptée au passage à l’échelle.

---

# 25. Schéma de synthèse

```mermaid
flowchart TD
    A["Problème : traiter des séquences"] --> B["RNN"]
    B --> C["Limite : dépendances longues"]
    B --> D["Limite : faible parallélisation"]

    C --> E["LSTM / GRU"]
    D --> E

    E --> F["Seq2Seq"]
    F --> G["Limite : vecteur de contexte unique"]

    G --> H["Attention"]
    H --> I["Meilleur accès au contexte"]

    I --> J["Transformer"]
    J --> K["Attention sans récurrence"]
    J --> L["Parallélisation"]
    J --> M["Dépendances longues"]
    J --> N["LLM modernes"]
```

---

# 26. Questions de compréhension

Pour vérifier que nous avons compris ce chapitre, nous pouvons répondre aux questions suivantes.

## Question 1

Pourquoi les RNN sont-ils naturellement adaptés aux séquences ?

Réponse attendue : parce qu’ils lisent les éléments les uns après les autres et maintiennent un état interne qui transporte l’information au fil du temps.

## Question 2

Pourquoi les RNN ont-ils du mal avec les dépendances longues ?

Réponse attendue : parce que l’information doit traverser de nombreuses étapes successives, ce qui peut entraîner une perte d’information et un affaiblissement du gradient.

## Question 3

Quel problème les LSTM et GRU cherchent-ils à résoudre ?

Réponse attendue : ils cherchent à améliorer la mémoire des RNN grâce à des mécanismes de portes permettant de contrôler ce qui est conservé, oublié ou transmis.

## Question 4

Quel est le problème du vecteur de contexte unique dans les premiers modèles Seq2Seq ?

Réponse attendue : toute la phrase source doit être compressée dans une seule représentation, ce qui devient insuffisant pour les phrases longues ou complexes.

## Question 5

Quelle est l’idée principale de l’attention ?

Réponse attendue : permettre au modèle de regarder dynamiquement les parties pertinentes de la séquence au lieu de dépendre d’une seule représentation globale.

## Question 6

Quelle est la rupture introduite par le Transformer ?

Réponse attendue : le Transformer supprime la récurrence et repose principalement sur l’attention pour relier directement les tokens entre eux.

## Question 7

Pourquoi les Transformers sont-ils plus faciles à paralléliser que les RNN ?

Réponse attendue : parce que les représentations des tokens peuvent être calculées simultanément dans les couches d’attention, alors que les RNN nécessitent de calculer les états dans l’ordre.

## Question 8

Quelle est une limite importante des Transformers ?

Réponse attendue : le coût de l’attention augmente quadratiquement avec la longueur de la séquence, car chaque token peut regarder tous les autres tokens.

---

# 27. Transition vers le chapitre 2

Dans ce chapitre, nous avons compris pourquoi les Transformers ont été proposés.

Dans le chapitre suivant, nous allons préparer les bases nécessaires pour entrer dans leur fonctionnement interne.

Nous verrons comment passer de ceci :

```txt
Le chat dort.
```

à ceci :

```txt
[154, 932, 421, 13]
```

puis à des vecteurs numériques manipulables par un réseau de neurones.

Autrement dit, nous allons étudier :

- la tokenisation ;
    
- les tokens ;
    
- les embeddings ;
    
- les tenseurs ;
    
- les dimensions utilisées dans un Transformer.
    

Le chapitre 2 construira donc le pont entre le texte brut et l’entrée numérique du modèle.
---

## Chapitre 2 — Rappels nécessaires : tenseurs, embeddings et séquences

Avant d’entrer dans l’architecture Transformer, nous rappellerons les notions fondamentales nécessaires.

Nous verrons comment un texte brut est transformé en données numériques exploitables par un réseau de neurones.

Nous étudierons :

- la tokenisation ;
    
- les tokens ;
    
- les embeddings ;
    
- les tenseurs ;
    
- les dimensions batch, séquence et features ;
    
- la représentation vectorielle des mots ou fragments de mots ;
    
- la différence entre représentation symbolique et représentation distribuée.
    

Diagramme prévu :

```mermaid
flowchart LR
    A["Texte brut"] --> B["Tokenisation"]
    B --> C["IDs de tokens"]
    C --> D["Embedding"]
    D --> E["Vecteurs continus"]
    E --> F["Entrée du Transformer"]
```

---

## Chapitre 3 — Le problème de l’ordre dans les séquences

Les Transformers ne lisent pas naturellement les tokens les uns après les autres comme un RNN. Nous devrons donc comprendre comment l’ordre des mots est injecté dans le modèle.

Nous étudierons :

- pourquoi l’ordre est indispensable ;
    
- pourquoi l’attention seule ne suffit pas à représenter la position ;
    
- les positional encodings sinusoïdaux ;
    
- les positional embeddings appris ;
    
- les encodages relatifs de position ;
    
- les approches modernes comme RoPE et ALiBi.
    

Diagramme prévu :

```mermaid
flowchart TD
    A["Token embedding"] --> C["Représentation finale"]
    B["Position encoding"] --> C
    C --> D["Transformer"]
```

---

## Chapitre 4 — Le mécanisme d’attention : intuition et formulation

Nous entrerons ensuite dans le cœur du Transformer : l’attention.

Nous expliquerons l’idée fondamentale : pour chaque token, le modèle apprend à regarder les autres tokens utiles pour construire une représentation contextualisée.

Nous verrons :

- l’intuition de l’attention ;
    
- les notions de Query, Key et Value ;
    
- le produit scalaire entre queries et keys ;
    
- le rôle du softmax ;
    
- la pondération des values ;
    
- la notion de contexte ;
    
- pourquoi l’attention permet de relier des mots éloignés.
    

Diagramme prévu :

```mermaid
flowchart LR
    A["Token courant"] --> Q["Query"]
    B["Tous les tokens"] --> K["Keys"]
    B --> V["Values"]
    Q --> S["Scores d'attention"]
    K --> S
    S --> SM["Softmax"]
    SM --> W["Poids d'attention"]
    W --> O["Somme pondérée des Values"]
    V --> O
```

---

## Chapitre 5 — Scaled Dot-Product Attention

Nous formaliserons mathématiquement le mécanisme d’attention utilisé dans le papier **Attention Is All You Need**.

Nous détaillerons la formule :

[  
Attention(Q,K,V) = softmax\left(\frac{QK^T}{\sqrt{d_k}}\right)V  
]

Nous expliquerons :

- pourquoi on utilise un produit scalaire ;
    
- pourquoi on divise par (\sqrt{d_k}) ;
    
- le rôle de la matrice (QK^T) ;
    
- le rôle du softmax ;
    
- le calcul final avec (V) ;
    
- les dimensions des matrices ;
    
- un exemple numérique simplifié.
    

Diagramme prévu :

```mermaid
flowchart TD
    Q["Q"] --> M["QK^T"]
    K["K"] --> M
    M --> S["Division par sqrt(d_k)"]
    S --> SM["Softmax"]
    SM --> P["Poids d'attention"]
    P --> O["Multiplication par V"]
    V --> O
```

---

## Chapitre 6 — Multi-Head Attention

Nous verrons ensuite pourquoi une seule attention ne suffit pas toujours.

L’idée de la multi-head attention est de permettre au modèle d’apprendre plusieurs relations différentes en parallèle.

Nous étudierons :

- la limite d’une seule tête d’attention ;
    
- le principe des têtes multiples ;
    
- les projections linéaires de (Q), (K), (V) ;
    
- la concaténation des têtes ;
    
- la projection finale ;
    
- l’interprétation des têtes comme sous-espaces relationnels ;
    
- les limites de l’interprétation trop naïve des têtes d’attention.
    

Diagramme prévu :

```mermaid
flowchart TD
    X["Entrée X"] --> H1["Head 1"]
    X --> H2["Head 2"]
    X --> H3["Head 3"]
    X --> H4["Head 4"]

    H1 --> C["Concaténation"]
    H2 --> C
    H3 --> C
    H4 --> C

    C --> O["Projection linéaire finale"]
```

---

## Chapitre 7 — Architecture Encoder-Decoder du Transformer original

Nous étudierons l’architecture complète proposée dans **Attention Is All You Need**.

Le Transformer original est une architecture **encoder-decoder**, conçue initialement pour la traduction automatique.

Nous verrons :

- le rôle de l’encoder ;
    
- le rôle du decoder ;
    
- l’empilement de plusieurs couches ;
    
- la self-attention dans l’encoder ;
    
- la masked self-attention dans le decoder ;
    
- l’encoder-decoder attention ;
    
- le lien avec les tâches de traduction.
    

Diagramme prévu :

```mermaid
flowchart LR
    A["Phrase source"] --> B["Embedding source"]
    B --> C["Encoder stack"]
    C --> D["Représentations encodées"]

    E["Phrase cible décalée"] --> F["Embedding cible"]
    F --> G["Decoder stack"]
    D --> G
    G --> H["Distribution sur le vocabulaire"]
```

---

## Chapitre 8 — Bloc Encoder en détail

Nous détaillerons chaque composant d’un bloc encoder.

Nous verrons :

- la self-attention ;
    
- les connexions résiduelles ;
    
- la normalisation ;
    
- le feed-forward network ;
    
- l’empilement des blocs ;
    
- le rôle de chaque sous-couche ;
    
- pourquoi le modèle alterne attention et transformation non linéaire.
    

Diagramme prévu :

```mermaid
flowchart TD
    X["Entrée"] --> A["Multi-Head Self-Attention"]
    A --> R1["Add & Norm"]
    X --> R1
    R1 --> F["Feed Forward Network"]
    F --> R2["Add & Norm"]
    R1 --> R2
    R2 --> Y["Sortie du bloc encoder"]
```

---

## Chapitre 9 — Bloc Decoder en détail

Nous détaillerons ensuite le bloc decoder.

Nous expliquerons pourquoi il est plus complexe que le bloc encoder.

Nous étudierons :

- la masked self-attention ;
    
- le masque causal ;
    
- l’attention vers l’encoder ;
    
- la génération token par token ;
    
- le décalage de la séquence cible pendant l’entraînement ;
    
- la différence entre entraînement et inférence.
    

Diagramme prévu :

```mermaid
flowchart TD
    X["Tokens cibles précédents"] --> A["Masked Multi-Head Self-Attention"]
    A --> R1["Add & Norm"]
    R1 --> B["Encoder-Decoder Attention"]
    E["Sortie Encoder"] --> B
    B --> R2["Add & Norm"]
    R2 --> F["Feed Forward Network"]
    F --> R3["Add & Norm"]
    R3 --> Y["Prédiction du prochain token"]
```

---

## Chapitre 10 — Masques d’attention

Nous consacrerons un chapitre aux masques, car ils sont essentiels pour comprendre les Transformers génératifs.

Nous verrons :

- le padding mask ;
    
- le causal mask ;
    
- le look-ahead mask ;
    
- pourquoi le decoder ne doit pas voir le futur ;
    
- comment les masques modifient les scores d’attention ;
    
- le lien avec la génération autoregressive ;
    
- les erreurs fréquentes liées aux masques.
    

Diagramme prévu :

```mermaid
flowchart TD
    A["Scores d'attention"] --> B["Application du masque"]
    B --> C["Valeurs interdites = -inf"]
    C --> D["Softmax"]
    D --> E["Poids d'attention autorisés"]
```

---

## Chapitre 11 — Feed-Forward Network et non-linéarités

Nous verrons que chaque bloc Transformer ne contient pas seulement de l’attention.

Après l’attention, chaque position passe dans un petit réseau dense appliqué indépendamment à chaque token.

Nous étudierons :

- le feed-forward network position-wise ;
    
- les dimensions internes ;
    
- les fonctions d’activation ;
    
- ReLU dans le papier original ;
    
- GELU dans les modèles modernes ;
    
- le rôle de la capacité non linéaire ;
    
- pourquoi l’attention seule n’est pas suffisante.
    

Diagramme prévu :

```mermaid
flowchart LR
    X["Vecteur token"] --> L1["Linear d_model -> d_ff"]
    L1 --> A["Activation"]
    A --> L2["Linear d_ff -> d_model"]
    L2 --> Y["Vecteur transformé"]
```

---

## Chapitre 12 — Résidus, normalisation et stabilité de l’entraînement

Nous expliquerons pourquoi les Transformers profonds ont besoin de mécanismes de stabilisation.

Nous verrons :

- les connexions résiduelles ;
    
- LayerNorm ;
    
- Add & Norm ;
    
- Post-LN dans le Transformer original ;
    
- Pre-LN dans de nombreux modèles modernes ;
    
- le gradient flow ;
    
- les problèmes d’instabilité ;
    
- l’importance de la normalisation dans les grands modèles.
    

Diagramme prévu :

```mermaid
flowchart TD
    X["Entrée"] --> S["Sous-couche"]
    S --> A["Addition résiduelle"]
    X --> A
    A --> N["LayerNorm"]
    N --> Y["Sortie"]
```

---

## Chapitre 13 — Entraînement du Transformer original

Nous étudierons comment le Transformer original est entraîné pour la traduction automatique.

Nous verrons :

- le teacher forcing ;
    
- le décalage de la cible ;
    
- la fonction de perte cross-entropy ;
    
- l’optimiseur Adam ;
    
- le learning rate schedule ;
    
- le warmup ;
    
- le label smoothing ;
    
- le batching ;
    
- la parallélisation.
    

Diagramme prévu :

```mermaid
flowchart LR
    A["Phrase source"] --> T["Transformer"]
    B["Phrase cible décalée"] --> T
    T --> P["Prédictions"]
    C["Phrase cible réelle"] --> L["Loss"]
    P --> L
    L --> O["Optimisation"]
```

---

## Chapitre 14 — Lecture détaillée du papier “Attention Is All You Need”

Ce chapitre sera central.

Nous lirons et expliquerons le papier section par section.

Nous verrons :

- le contexte scientifique de 2017 ;
    
- le problème de la traduction automatique ;
    
- les limites des RNN et CNN ;
    
- l’idée principale : supprimer la récurrence ;
    
- la structure encoder-decoder ;
    
- la scaled dot-product attention ;
    
- la multi-head attention ;
    
- les positional encodings ;
    
- les résultats expérimentaux ;
    
- les performances sur WMT 2014 English-German et English-French ;
    
- les gains en parallélisation ;
    
- les limites du papier ;
    
- ce qui était révolutionnaire ;
    
- ce qui a été dépassé depuis.
    

Diagramme prévu :

```mermaid
flowchart TD
    A["Problème : traduction automatique"] --> B["Limites RNN/CNN"]
    B --> C["Idée : tout faire par attention"]
    C --> D["Architecture Transformer"]
    D --> E["Résultats expérimentaux"]
    E --> F["Impact sur les LLM modernes"]
```

---

## Chapitre 15 — Complexité algorithmique des Transformers

Nous analyserons le coût calculatoire du Transformer.

Nous verrons :

- la complexité de l’attention en (O(n^2)) ;
    
- le coût mémoire ;
    
- le rôle de la longueur de contexte ;
    
- comparaison avec RNN et CNN ;
    
- pourquoi les longues séquences sont difficiles ;
    
- les optimisations modernes ;
    
- FlashAttention ;
    
- sparse attention ;
    
- linear attention ;
    
- sliding window attention.
    

Diagramme prévu :

```mermaid
flowchart LR
    A["Longueur de séquence n"] --> B["Matrice attention n x n"]
    B --> C["Coût mémoire O(n²)"]
    B --> D["Coût calcul O(n²)"]
    C --> E["Limite contexte long"]
    D --> E
```

---

## Chapitre 16 — Grandes familles de Transformers

Nous verrons que le Transformer original a donné naissance à plusieurs familles majeures.

Nous distinguerons :

- les modèles encoder-only ;
    
- les modèles decoder-only ;
    
- les modèles encoder-decoder ;
    
- BERT ;
    
- GPT ;
    
- T5 ;
    
- Vision Transformer ;
    
- multimodal Transformers.
    

Diagramme prévu :

```mermaid
flowchart TD
    A["Transformer"] --> B["Encoder-only"]
    A --> C["Decoder-only"]
    A --> D["Encoder-Decoder"]

    B --> BERT["BERT"]
    C --> GPT["GPT"]
    D --> T5["T5 / Traduction / Résumé"]
```

---

## Chapitre 17 — BERT et les Transformers encoder-only

Nous étudierons les modèles de type BERT.

Nous verrons :

- l’architecture encoder-only ;
    
- le masked language modeling ;
    
- la prédiction de tokens masqués ;
    
- les embeddings de segment ;
    
- la classification avec le token `[CLS]` ;
    
- les usages en compréhension du langage ;
    
- les limites de BERT pour la génération.
    

Diagramme prévu :

```mermaid
flowchart LR
    A["Texte avec token masqué"] --> B["BERT Encoder"]
    B --> C["Représentation contextualisée"]
    C --> D["Prédiction du token masqué"]
    C --> E["Classification"]
```

---

## Chapitre 18 — GPT et les Transformers decoder-only

Nous étudierons les modèles de type GPT.

Nous verrons :

- l’architecture decoder-only ;
    
- la génération autoregressive ;
    
- le causal mask ;
    
- la prédiction du prochain token ;
    
- le pré-entraînement sur grands corpus ;
    
- l’émergence des capacités de génération ;
    
- le lien avec les LLM modernes ;
    
- les différences entre GPT, BERT et T5.
    

Diagramme prévu :

```mermaid
flowchart LR
    A["Tokens précédents"] --> B["Transformer Decoder-only"]
    B --> C["Distribution prochain token"]
    C --> D["Token généré"]
    D --> A
```

---

## Chapitre 19 — T5, traduction, résumé et modèles encoder-decoder modernes

Nous reviendrons aux architectures encoder-decoder modernes.

Nous verrons :

- le paradigme text-to-text ;
    
- T5 ;
    
- BART ;
    
- les tâches de traduction ;
    
- le résumé automatique ;
    
- la reformulation ;
    
- les tâches sequence-to-sequence ;
    
- les avantages et limites face aux modèles decoder-only.
    

Diagramme prévu :

```mermaid
flowchart LR
    A["Entrée textuelle"] --> B["Encoder"]
    B --> C["Decoder"]
    C --> D["Sortie textuelle"]
```

---

## Chapitre 20 — Transformers pour la vision

Nous verrons comment les Transformers ont été adaptés à la vision par ordinateur.

Nous étudierons :

- Vision Transformer ;
    
- découpage de l’image en patches ;
    
- patch embeddings ;
    
- classification d’image ;
    
- comparaison avec CNN ;
    
- Data-efficient Image Transformers ;
    
- Swin Transformer ;
    
- attention locale et hiérarchique.
    

Diagramme prévu :

```mermaid
flowchart LR
    A["Image"] --> B["Découpage en patches"]
    B --> C["Patch embeddings"]
    C --> D["Transformer Encoder"]
    D --> E["Classification"]
```

---

## Chapitre 21 — Transformers multimodaux

Nous aborderons les modèles capables de traiter plusieurs modalités.

Nous verrons :

- texte + image ;
    
- texte + audio ;
    
- texte + vidéo ;
    
- embeddings multimodaux ;
    
- cross-attention ;
    
- modèles vision-language ;
    
- génération d’images ;
    
- modèles multimodaux modernes.
    

Diagramme prévu :

```mermaid
flowchart TD
    A["Texte"] --> E["Espace latent partagé"]
    B["Image"] --> E
    C["Audio"] --> E
    E --> M["Transformer multimodal"]
    M --> O["Sortie : texte, image, action..."]
```

---

## Chapitre 22 — Transformers et modèles de langage modernes

Nous relierons les Transformers aux grands modèles de langage actuels.

Nous verrons :

- scaling laws ;
    
- pré-entraînement ;
    
- instruction tuning ;
    
- RLHF ;
    
- préférences humaines ;
    
- alignement ;
    
- contexte long ;
    
- hallucinations ;
    
- raisonnement ;
    
- limites structurelles des LLM ;
    
- coût énergétique et matériel.
    

Diagramme prévu :

```mermaid
flowchart TD
    A["Pré-entraînement massif"] --> B["Modèle de base"]
    B --> C["Instruction tuning"]
    C --> D["Alignement / RLHF"]
    D --> E["Assistant conversationnel"]
```

---

## Chapitre 23 — Transformers et RAG

Nous étudierons comment les Transformers sont utilisés dans les systèmes de **Retrieval-Augmented Generation**.

Nous verrons :

- pourquoi un LLM seul ne suffit pas toujours ;
    
- embeddings et recherche vectorielle ;
    
- retrieval ;
    
- injection de contexte ;
    
- génération sourcée ;
    
- limites du contexte ;
    
- reranking ;
    
- hybrid search ;
    
- Graph RAG ;
    
- liens entre attention interne et récupération externe.
    

Diagramme prévu :

```mermaid
flowchart LR
    A["Question utilisateur"] --> B["Recherche documentaire"]
    B --> C["Documents pertinents"]
    C --> D["Prompt enrichi"]
    D --> E["LLM Transformer"]
    E --> F["Réponse sourcée"]
```

---

## Chapitre 24 — Implémenter un mini-Transformer

Nous passerons ensuite à une approche pratique.

Nous construirons un mini-Transformer pédagogique.

Nous verrons :

- les embeddings ;
    
- l’attention ;
    
- la multi-head attention ;
    
- les blocs Transformer ;
    
- le masque causal ;
    
- la boucle d’entraînement ;
    
- la génération autoregressive ;
    
- les erreurs classiques d’implémentation.
    

Diagramme prévu :

```mermaid
flowchart TD
    A["Dataset texte"] --> B["Tokenisation"]
    B --> C["Mini-Transformer"]
    C --> D["Loss"]
    D --> E["Backpropagation"]
    E --> C
    C --> F["Génération"]
```

---

## Chapitre 25 — Limites, critiques et perspectives

Nous terminerons le cours par une réflexion critique.

Nous verrons :

- la complexité quadratique ;
    
- la dépendance aux données massives ;
    
- les biais ;
    
- les hallucinations ;
    
- la difficulté du raisonnement fiable ;
    
- le manque d’interprétabilité ;
    
- la consommation énergétique ;
    
- les limites du benchmark ;
    
- les architectures post-Transformer ;
    
- les modèles hybrides ;
    
- les pistes comme Mamba, RWKV, state-space models et architectures mémoire.
    

Diagramme prévu :

```mermaid
flowchart TD
    A["Transformers"] --> B["Succès massif"]
    A --> C["Limites"]
    C --> D["Coût O(n²)"]
    C --> E["Hallucinations"]
    C --> F["Biais"]
    C --> G["Difficulté d'interprétation"]
    C --> H["Nouvelles architectures"]
```

---

# Progression pédagogique proposée

Nous pouvons organiser le cours en quatre grandes parties.

```mermaid
flowchart TD
    A["Partie 1 : Fondations"] --> B["Embeddings, séquences, attention"]
    B --> C["Partie 2 : Transformer original"]
    C --> D["Papier Attention Is All You Need"]
    D --> E["Partie 3 : Familles modernes"]
    E --> F["BERT, GPT, T5, ViT, multimodal"]
    F --> G["Partie 4 : Pratique et critique"]
    G --> H["Implémentation, RAG, limites"]
```

## Partie 1 — Fondations

Elle regroupe les chapitres 1 à 6.

Nous y construisons les bases nécessaires : séquences, embeddings, attention, multi-head attention.

## Partie 2 — Transformer original

Elle regroupe les chapitres 7 à 15.

Nous y détaillons l’architecture complète du papier **Attention Is All You Need**, puis nous analysons son coût, son entraînement et son impact.

## Partie 3 — Transformers modernes

Elle regroupe les chapitres 16 à 23.

Nous voyons comment l’architecture s’est déclinée en BERT, GPT, T5, Vision Transformers, modèles multimodaux et systèmes RAG.

## Partie 4 — Implémentation et regard critique

Elle regroupe les chapitres 24 et 25.

Nous construisons un mini-Transformer, puis nous prenons du recul sur les limites de cette famille de modèles.

---

# Fil conducteur du cours

Le fil conducteur sera le suivant :

```mermaid
flowchart LR
    A["Comment représenter une séquence ?"]
    --> B["Comment chaque token regarde les autres ?"]
    --> C["Comment empiler ces mécanismes ?"]
    --> D["Comment entraîner le modèle ?"]
    --> E["Comment générer du texte ?"]
    --> F["Comment passer aux LLM modernes ?"]
```

À chaque chapitre, nous garderons trois niveaux de lecture :

1. **intuition** : comprendre l’idée simplement ;
    
2. **formalisme** : comprendre les équations et les dimensions ;
    
3. **implémentation** : savoir comment cela se traduit en code.
    

---

# Chapitres particulièrement importants

Pour un cours de Master, les chapitres les plus fondamentaux seront :

- Chapitre 4 — Le mécanisme d’attention ;
    
- Chapitre 5 — Scaled Dot-Product Attention ;
    
- Chapitre 6 — Multi-Head Attention ;
    
- Chapitre 7 — Architecture Encoder-Decoder ;
    
- Chapitre 9 — Bloc Decoder ;
    
- Chapitre 10 — Masques d’attention ;
    
- Chapitre 14 — Lecture détaillée de **Attention Is All You Need** ;
    
- Chapitre 18 — GPT et les Transformers decoder-only ;
    
- Chapitre 24 — Implémenter un mini-Transformer.
    

Ce sont ces chapitres qui permettront vraiment de passer de “je connais le mot Transformer” à “je comprends comment ça fonctionne techniquement”.