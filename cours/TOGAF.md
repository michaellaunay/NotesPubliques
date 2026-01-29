# Plan du cours : **TOGAF – Architecture d’Entreprise**

## **Introduction générale**

- Pourquoi l’architecture d’entreprise est devenue centrale
- Place de TOGAF parmi les cadres existants
- Présentation de **The Open Group** et de la philosophie TOGAF
- Objectifs pédagogiques du cours

## **Partie I – Fondements de l’architecture d’entreprise**

### Chapitre 1 – Qu’est-ce que l’architecture d’entreprise ?

- Définition et périmètre
- Différence entre architecture logicielle, technique et d’entreprise
- Enjeux stratégiques (alignement métier / SI, transformation numérique)
- Cas typiques : administrations, grandes entreprises, SI complexes

### Chapitre 2 – Historique et positionnement de TOGAF

- Origines de TOGAF    
- Évolutions majeures (TOGAF 10.x, TOGAF Standard)    
- Comparaison rapide avec Zachman, ITIL, COBIT    
- Forces et limites de TOGAF
    
## **Partie II – Structure générale de TOGAF**

### Chapitre 3 – Les composants de TOGAF

- Le cadre global TOGAF    
- Le rôle de l’ADM (Architecture Development Method)    
- Le continuum d’entreprise    
- Les référentiels et artefacts    

### Chapitre 4 – Les domaines d’architecture

- Architecture métier    
- Architecture des données    
- Architecture applicative    
- Architecture technologique    
- Relations et dépendances entre domaines

# **Partie III – L’ADM : méthode de développement de l’architecture**

### Chapitre 5 – Vue d’ensemble de l’ADM

- Principe itératif et cyclique    
- Notion de baseline et target architecture    
- Gouvernance et pilotage    

### Chapitre 6 – Phases préliminaire et Vision (Préliminaire & Phase A)

- Mise en place du cadre d’architecture
- Principes d’architecture
- Parties prenantes
- Vision cible et valeur métier

### Chapitre 7 – Phases B, C et D : conception de l’architecture

- Phase B : architecture métier
    
- Phase C : données et applications
    
- Phase D : architecture technique
    
- Modélisation, cohérence et arbitrages
    

### Chapitre 8 – Phases E et F : transformation et migration

- Opportunités et solutions
    
- Scénarios de migration
    
- Roadmaps
    
- Gestion des risques et contraintes
    

### Chapitre 9 – Phases G et H : gouvernance et amélioration continue

- Gouvernance de l’architecture
    
- Conformité des projets
    
- Gestion du changement
    
- Capitalisation et amélioration continue
    

## **Partie IV – Artefacts, livrables et gouvernance**

### Chapitre 10 – Artefacts TOGAF

- Catalogues, matrices, diagrammes
    
- Exemples de livrables concrets
    
- Bonnes pratiques de documentation
    

### Chapitre 11 – Gouvernance de l’architecture

- Rôles et responsabilités
    
- Architecture Board
    
- Indicateurs de conformité
    
- Gestion des dérogations
    

## **Partie V – TOGAF en pratique**

### Chapitre 12 – TOGAF et les approches modernes

- TOGAF et Agile
    
- TOGAF et DevOps
    
- TOGAF et microservices
    
- TOGAF et Cloud / Kubernetes
    

### Chapitre 13 – Études de cas

- Cas d’une université
    
- Cas d’une administration publique
    
- Cas d’une entreprise SaaS
    
- Analyse critique des choix d’architecture
    

## **Partie VI – Perspectives professionnelles et certifications**

### Chapitre 14 – Métiers et usages professionnels

- Architecte d’entreprise
    
- Urbaniste SI
    
- Consultant en transformation numérique
    
- Interaction avec les équipes de développement
    

### Chapitre 15 – Certifications TOGAF

- TOGAF Foundation
    
- TOGAF Certified
    
- Intérêt et limites des certifications
    

## **Conclusion générale**

- Apports réels de TOGAF
    
- Conditions de succès
    
- Limites et dérives possibles
    
- Place de TOGAF dans le paysage actuel de l’ingénierie logicielle
    

## **Évaluation**

- Étude de cas écrite
    
- Présentation orale d’une architecture cible
    
- Analyse critique d’un SI existant à l’aide de TOGAF
    

# Introduction générale

Dans cette introduction, nous posons le cadre intellectuel et professionnel du cours. L’objectif est de comprendre **pourquoi l’architecture d’entreprise s’est imposée comme une discipline centrale**, et en quoi **TOGAF** constitue aujourd’hui l’un des référentiels majeurs pour y répondre.

## Pourquoi l’architecture d’entreprise est devenue centrale

Au cours des deux dernières décennies, les systèmes d’information ont connu une **complexification rapide et continue**. Les organisations doivent désormais composer avec :

- la multiplication des applications et des services,
- l’hétérogénéité technologique (on-premise, cloud, SaaS, microservices),
- l’accélération des cycles de transformation numérique,
- des contraintes réglementaires croissantes (sécurité, données personnelles, traçabilité),
- une attente forte d’alignement entre **stratégie métier** et **systèmes informatiques**.

Dans ce contexte, le système d’information ne peut plus être pensé uniquement comme un ensemble de solutions techniques. Il devient un **actif stratégique**, structurant l’organisation elle-même.  
L’architecture d’entreprise émerge alors comme une réponse à une problématique clé :

> _Comment concevoir, faire évoluer et gouverner un système d’information complexe tout en garantissant sa cohérence globale et son alignement avec les objectifs de l’organisation ?_

L’architecture d’entreprise vise précisément à fournir une **vision transversale**, à la fois métier, fonctionnelle, applicative et technique. Elle permet de dépasser une logique de projets isolés pour inscrire les décisions informatiques dans une **trajectoire de long terme**.

## Place de TOGAF parmi les cadres existants

Face à ces enjeux, plusieurs cadres méthodologiques ont été proposés au fil du temps : Zachman, ITIL, COBIT, ArchiMate, entre autres. Chacun répond à des besoins spécifiques : classification, gouvernance, gestion des services, contrôle ou modélisation.

TOGAF se distingue par plusieurs caractéristiques majeures :

- il propose une **méthode structurée** pour concevoir l’architecture (l’ADM),
- il couvre l’ensemble des **dimensions de l’architecture d’entreprise**,
- il met l’accent sur la **gouvernance** et l’alignement stratégique,
- il est conçu comme un **cadre adaptable**, et non comme une méthode rigide.

TOGAF n’est donc ni un simple modèle conceptuel, ni une norme prescriptive. Il s’agit d’un **cadre de référence pragmatique**, largement adopté dans les grandes organisations, aussi bien dans le secteur public que privé.

## Présentation de The Open Group et de la philosophie TOGAF

TOGAF est développé et maintenu par **The Open Group**, un consortium international réunissant entreprises, institutions publiques et acteurs technologiques. Sa vocation est de produire des **standards ouverts**, interopérables et indépendants des éditeurs.

La philosophie de TOGAF repose sur plusieurs principes fondamentaux :

- **Neutralité technologique** : TOGAF ne prescrit aucun outil, langage ou fournisseur.
- **Orientation métier** : l’architecture n’est pas une fin en soi, mais un moyen au service des objectifs de l’organisation.
- **Itération et amélioration continue** : l’architecture évolue dans le temps, par cycles successifs.
- **Gouvernance** : les décisions d’architecture doivent être encadrées, partagées et contrôlées.

TOGAF ne cherche pas à remplacer les méthodes de développement logiciel ou les pratiques agiles, mais à fournir un **cadre de cohérence globale** dans lequel ces approches peuvent s’inscrire.

## Objectifs pédagogiques du cours

À l’issue de ce cours, nous visons plusieurs objectifs complémentaires.

Sur le plan conceptuel, nous cherchons à :

- comprendre les fondements de l’architecture d’entreprise,    
- maîtriser la structure et les concepts clés de TOGAF,
- analyser les enjeux d’alignement entre stratégie, métiers et systèmes d’information.

Sur le plan méthodologique, nous souhaitons que les étudiants soient capables de :

- lire et produire une vision d’architecture cohérente,
- utiliser l’ADM pour structurer une démarche d’architecture,
- identifier les impacts architecturaux d’un projet ou d’une transformation.

Enfin, sur le plan professionnel, ce cours vise à :

- développer une posture d’architecte ou de concepteur transverse,
- dialoguer efficacement avec des profils métiers, techniques et décisionnels,
- porter un regard critique sur les choix d’architecture dans des contextes réels.

Ainsi, TOGAF n’est pas abordé comme un simple référentiel à apprendre, mais comme un **outil intellectuel et méthodologique**, destiné à structurer la réflexion et la prise de décision dans des environnements complexes.

# Partie I – Fondements de l’architecture d’entreprise

## Chapitre 1 – Qu’est-ce que l’architecture d’entreprise ?

Dans ce premier chapitre, nous posons les bases conceptuelles indispensables à la compréhension de TOGAF. Avant d’aborder une méthode ou un cadre, il est essentiel de clarifier **ce que recouvre exactement la notion d’architecture d’entreprise**, ce qu’elle inclut — et ce qu’elle n’est pas.

### 1. Définition et périmètre de l’architecture d’entreprise

L’architecture d’entreprise peut être définie comme :

> _la discipline qui vise à décrire, concevoir et gouverner de manière cohérente l’ensemble des composantes d’une organisation — métiers, processus, données, applications et infrastructures — afin de soutenir sa stratégie et ses objectifs._

Nous insistons sur plusieurs éléments clés de cette définition :

- **Vision globale** : l’architecture d’entreprise s’intéresse au système d’information dans son ensemble, et non à une application ou un projet isolé.
- **Lien avec l’organisation** : elle ne se limite pas au technique, mais inclut les métiers, les acteurs, les règles et les objectifs.
- **Temporalité** : elle décrit à la fois l’existant (_architecture de référence_) et le futur souhaité (_architecture cible_).
- **Finalité stratégique** : l’architecture n’est pas décorative : elle sert la prise de décision et l’orientation des transformations.

Le périmètre de l’architecture d’entreprise recouvre donc :

- les **processus métiers**,
- les **flux d’information et de données**,
- les **applications et services**,
- les **infrastructures techniques**,
- ainsi que les **règles de gouvernance** qui encadrent l’évolution du SI.

### 2. Différence entre architecture logicielle, technique et d’entreprise

Une confusion fréquente, en particulier chez les profils techniques, consiste à assimiler l’architecture d’entreprise à une forme d’architecture logicielle élargie. Il est important de distinguer clairement ces niveaux.

#### Architecture logicielle

L’architecture logicielle concerne :

- une application ou un système logiciel donné,
- la structuration du code,
- les patterns (MVC, microservices, hexagonal, etc.),
- les interactions entre composants logiciels.

Elle répond à la question :

> _Comment concevoir correctement une application ?_

#### Architecture technique

L’architecture technique s’intéresse aux :

- infrastructures (serveurs, réseaux, cloud),
- environnements d’exécution,
- choix technologiques (bases de données, middleware, sécurité).

Elle répond à la question :

> _Sur quoi et comment les applications s’exécutent-elles ?_

#### Architecture d’entreprise

L’architecture d’entreprise, quant à elle :

- englobe les deux précédentes,
- intègre les **métiers et la stratégie**,
- arbitre les choix sur le long terme,
- assure la cohérence entre projets hétérogènes.

Elle répond à une question plus large :

> _Pourquoi ces systèmes existent-ils, comment s’articulent-ils, et vers où doivent-ils évoluer ?_

Nous pouvons donc considérer l’architecture d’entreprise comme un **niveau de pilotage supérieur**, au-dessus des préoccupations purement logicielles ou techniques.

### 3. Enjeux stratégiques de l’architecture d’entreprise

#### Alignement métier / système d’information

L’un des enjeux majeurs est l’alignement entre :

- la stratégie de l’organisation,
- les besoins métiers,
- et les capacités réelles du système d’information.

Sans architecture d’entreprise, le SI tend à évoluer par accumulation : projets successifs, solutions ponctuelles, dépendances mal maîtrisées. À terme, cela conduit à :

- une perte de lisibilité,
- des coûts de maintenance élevés,
- une faible capacité d’adaptation.

L’architecture d’entreprise vise à rendre le SI **lisible, gouvernable et évolutif**.

#### Transformation numérique

La transformation numérique n’est pas uniquement technologique. Elle implique :

- une évolution des processus,
- une redéfinition des rôles,
- une exploitation stratégique des données.

Sans cadre architectural, ces transformations risquent d’être incohérentes, redondantes ou contre-productives.  
L’architecture d’entreprise permet d’inscrire ces changements dans une **trajectoire maîtrisée**, en évaluant leurs impacts globaux.

### 4. Cas typiques d’application

L’architecture d’entreprise prend tout son sens dans des contextes caractérisés par la **complexité**.

#### Administrations publiques

Les administrations présentent souvent :

- des systèmes anciens (legacy),
- une forte contrainte réglementaire,
- une multiplicité d’acteurs et de tutelles.

L’architecture d’entreprise y est utilisée pour :

- rationaliser les SI,
- favoriser l’interopérabilité,
- accompagner les réformes structurelles.

#### Grandes entreprises

Dans les grands groupes, on observe :

- une forte hétérogénéité applicative,
- des SI construits par strates successives,
- des enjeux de gouvernance internationale.

L’architecture d’entreprise sert ici à :

- harmoniser les pratiques,
- maîtriser les coûts,
- soutenir les fusions, acquisitions et réorganisations.

#### Systèmes d’information complexes

Enfin, l’architecture d’entreprise est particulièrement pertinente pour :

- les écosystèmes numériques,    
- les plateformes multi-services,    
- les organisations en forte croissance.    

Elle permet d’éviter une dérive vers une complexité incontrôlée et de maintenir une **cohérence d’ensemble** malgré l’évolution rapide des technologies.

#### Conclusion du chapitre

Nous retenons que l’architecture d’entreprise n’est ni une surcouche bureaucratique, ni une simple formalisation technique. Elle constitue un **outil structurant de compréhension, de décision et de gouvernance**, indispensable dès lors que le système d’information devient critique pour l’organisation.

Dans le chapitre suivant, nous analyserons plus en détail **l’émergence historique de TOGAF** et son positionnement parmi les autres cadres d’architecture existants.
## Chapitre 2 – Historique et positionnement de TOGAF

Dans ce chapitre, nous revenons sur la **genèse de TOGAF**, son évolution au fil du temps, et sa place parmi les grands cadres de référence utilisés aujourd’hui pour structurer et gouverner les systèmes d’information. L’objectif est de comprendre **d’où vient TOGAF**, **ce qu’il est devenu**, et **pourquoi il continue d’être largement utilisé**.
### 1. Origines de TOGAF

TOGAF apparaît dans les années 1990, à une période où les grandes organisations font face à une **explosion de la complexité de leurs systèmes d’information**. Les infrastructures se diversifient, les applications se multiplient, et les coûts de maintenance augmentent rapidement.

À l’origine, TOGAF s’appuie sur le **Technical Architecture Framework for Information Management (TAFIM)**, développé par le département de la Défense des États-Unis. Ce cadre visait déjà à structurer les architectures techniques à grande échelle, mais restait fortement orienté infrastructure.

C’est **The Open Group** qui reprend ces travaux en 1994 pour les faire évoluer vers un cadre plus général, ouvert et applicable au monde civil et industriel. Dès ses premières versions en 1995, TOGAF se distingue par deux choix structurants :

- une volonté de **standard ouvert**, indépendant des éditeurs,    
- une approche **méthodologique**, et non uniquement descriptive.    

Progressivement, TOGAF élargit son périmètre : il ne s’agit plus seulement d’architecture technique, mais bien d’**architecture d’entreprise**, intégrant les dimensions métier, applicative et organisationnelle.

### 2. Évolutions majeures : de TOGAF 9.x à TOGAF Standard (10.x)

#### TOGAF 9.x : la maturité

Les versions TOGAF 9 (9.0, 9.1, 9.2) marquent une phase de stabilisation et de diffusion massive du cadre. TOGAF devient alors :

- un **standard de facto** dans de nombreuses grandes organisations,    
- un socle pour des **certifications professionnelles**,    
- un langage commun entre architectes, DSI et directions métiers.    

L’ADM (Architecture Development Method) s’impose comme le cœur du framework, avec une structuration claire des phases, des livrables et de la gouvernance.

#### TOGAF Standard (10.x) : clarification et modularité

Avec TOGAF 10, publié sous l’appellation **TOGAF Standard**, une évolution importante est introduite. L’objectif n’est pas de révolutionner le cadre, mais de le rendre :

- plus **lisible**,    
- plus **modulaire**,    
- plus **facilement adaptable** aux contextes modernes (agile, digital, cloud).    

Le standard est désormais organisé autour de deux grandes parties :

- **TOGAF Fundamental Content** : le socle commun (ADM, concepts clés),    
- **TOGAF Series Guides** : des guides spécialisés selon les contextes d’usage.    

Cette évolution traduit une reconnaissance explicite d’un point essentiel : **TOGAF n’est pas une méthode universelle clé en main**, mais un cadre à adapter.
### 3. Comparaison rapide avec d’autres cadres

Pour bien positionner TOGAF, il est utile de le comparer à d’autres référentiels fréquemment rencontrés dans les organisations.

#### Zachman

Le cadre de Zachman est avant tout une **grille de classification**. Il permet de structurer les points de vue sur un système (quoi, comment, où, qui, quand, pourquoi), mais :

- il ne fournit pas de méthode,    
- il ne guide pas la transformation.    

Zachman répond à la question : _comment décrire un système ?_  
TOGAF répond à la question : _comment concevoir et faire évoluer une architecture ?_

#### ITIL

ITIL est centré sur la **gestion des services informatiques** :

- exploitation,    
- support,    
- qualité de service.    

Il s’intéresse peu à la conception globale de l’architecture.  
TOGAF, au contraire, intervient **en amont**, sur la structuration du SI et les choix d’architecture.

#### COBIT

COBIT est un cadre de **gouvernance et de contrôle** :

- conformité,    
- audit,    
- gestion des risques.    

Il fournit des mécanismes de pilotage, mais peu de contenus sur la conception de l’architecture elle-même.  
TOGAF est donc souvent **complémentaire** de COBIT.

### 4. Forces et limites de TOGAF

#### Forces

Parmi les principaux atouts de TOGAF, nous pouvons souligner :

- une **vision globale et transverse** du SI,    
- une méthode structurée et éprouvée (ADM),    
- une forte **reconnaissance internationale**,    
- une indépendance vis-à-vis des technologies et des éditeurs,    
- une capacité à structurer le dialogue entre métiers et IT.    

TOGAF est particulièrement efficace dans des environnements complexes, où les décisions doivent être **cohérentes, traçables et gouvernées**.

#### Limites

Cependant, TOGAF présente également des limites qu’il est essentiel d’identifier :

- il peut être perçu comme **lourd** ou bureaucratique s’il est appliqué de manière rigide,    
- il ne fournit pas de modèles opérationnels prêts à l’emploi,    
- il nécessite une **maturité organisationnelle** pour être réellement efficace,    
- il doit être adapté pour fonctionner avec des approches agiles ou DevOps.    

Un point clé à retenir est donc le suivant :

> _TOGAF échoue rarement par ses concepts, mais souvent par une mauvaise appropriation ou une application dogmatique._

### Conclusion du chapitre

TOGAF s’inscrit dans une histoire longue de la structuration des systèmes d’information. Il a évolué d’un cadre technique vers un **référentiel complet d’architecture d’entreprise**, tout en conservant une philosophie d’ouverture et d’adaptabilité.

Dans la suite du cours, nous entrerons dans le cœur opérationnel de TOGAF, en étudiant **sa structure interne et les principes de l’ADM**, afin de comprendre comment ce cadre se traduit concrètement dans une démarche d’architecture.

# Partie II – Structure générale de TOGAF

## Chapitre 3 – Les composants de TOGAF

Dans ce chapitre, nous analysons la **structure interne de TOGAF**. Il s’agit de comprendre comment le cadre est organisé, quels sont ses composants majeurs, et comment ils s’articulent pour former un ensemble cohérent au service de l’architecture d’entreprise.

### 1. Le cadre global TOGAF

TOGAF est conçu comme un **framework modulaire** plutôt qu’un document monolithique. Il propose un ensemble de concepts, de méthodes et de ressources destinés à être **sélectionnés, adaptés et combinés** selon le contexte de l’organisation.

Le cadre global TOGAF repose sur quatre piliers principaux :

- une **méthode centrale** (l’ADM),
- un **modèle de structuration des architectures** (domaines, vues, niveaux),
- un **continuum** permettant de gérer la réutilisation et l’évolution,
- un ensemble de **référentiels et d’artefacts** pour documenter et gouverner.

Ce cadre est développé et maintenu par **The Open Group**, avec une logique de standard ouvert, indépendant des éditeurs et des technologies.

L’idée directrice est la suivante : TOGAF ne dicte pas _quoi construire_, mais fournit un cadre pour **raisonner, décider et structurer** l’architecture.

### 2. Le rôle de l’ADM (Architecture Development Method)

L’**ADM** constitue le **cœur méthodologique** de TOGAF. Il s’agit d’une méthode itérative destinée à guider la conception, l’évolution et la gouvernance de l’architecture d’entreprise.

Nous insistons sur plusieurs caractéristiques fondamentales de l’ADM :

- **Itératif** : on ne parcourt pas l’ADM une seule fois ; les cycles se répètent et s’enrichissent.
- **Orienté décision** : chaque phase vise à produire des choix structurants, pas uniquement des documents.
- **Piloté par la valeur métier** : l’architecture est toujours justifiée par des objectifs organisationnels.

![Image](https://www.researchgate.net/publication/224258195/figure/fig3/AS%3A654358022717441%401533022518145/The-ADM-cycle-in-TOGAF-18.png)

![Image](https://www.researchgate.net/publication/337517962/figure/fig1/AS%3A869374781046784%401584286508136/shows-the-eight-main-phases-of-TOGAF-ADM-Stages-stages-of-TOGAF-ADM-include-1.ppm)

Les phases de l’ADM couvrent l’ensemble du cycle de vie de l’architecture :

- cadrage et vision,
- conception des architectures métier, données, applicatives et techniques,
- planification de la transformation,
- gouvernance,
- amélioration continue.

L’ADM agit donc comme une **colonne vertébrale**, autour de laquelle viennent se greffer les autres composants de TOGAF.

### 3. Le continuum d’entreprise

Le **continuum d’entreprise** est un concept central mais souvent sous-estimé. Il permet de penser l’architecture non pas comme un état figé, mais comme un **ensemble de ressources réutilisables**, situées sur un axe allant du générique au spécifique.

TOGAF distingue notamment :

- le **Foundation Architecture** : principes et modèles génériques,
- les **Common Systems Architectures** : architectures partagées (ex. ERP, sécurité),
- les **Industry Architectures** : spécifiques à un secteur,
- les **Organization-Specific Architectures** : propres à une organisation.

Le continuum sert plusieurs objectifs :

- capitaliser sur l’existant,
- éviter de « réinventer la roue » à chaque projet,
- favoriser la cohérence entre initiatives.

Nous ne concevons donc pas une architecture ex nihilo, mais à partir d’un **patrimoine architectural** en constante évolution.

### 4. Les référentiels et artefacts

TOGAF accorde une place importante à la **formalisation**. Pour piloter un SI complexe, il est nécessaire de produire des représentations partagées, compréhensibles et traçables.

#### Les référentiels

Les référentiels TOGAF regroupent l’ensemble des éléments structurants :

- principes d’architecture,
- modèles de référence,
- standards techniques,
- règles de gouvernance.

Ils constituent une **mémoire organisationnelle**, permettant de stabiliser les décisions et d’assurer leur cohérence dans le temps.

#### Les artefacts

Les artefacts sont les livrables produits tout au long de l’ADM. Ils prennent différentes formes :

- catalogues (acteurs, applications, technologies),
- matrices (relations, dépendances, impacts),
- diagrammes (processus, flux, architectures).

Ces artefacts ne sont pas une fin en soi. Leur rôle est de :

- faciliter la communication entre parties prenantes,
- soutenir la prise de décision,
- permettre le contrôle et la gouvernance de l’architecture.

Nous insistons ici sur un point clé : **la valeur d’un artefact dépend de son usage**, non de son exhaustivité.

### Conclusion du chapitre

Nous retenons que TOGAF est un cadre structuré autour de composants complémentaires :

- l’ADM comme méthode centrale,
- le continuum pour gérer la réutilisation et l’évolution,
- les référentiels et artefacts pour documenter et gouverner.

Cette structure permet à TOGAF d’être à la fois **robuste** et **adaptable**, à condition d’être utilisée avec discernement.  
Dans le chapitre suivant, nous entrerons plus en détail dans l’analyse des **domaines d’architecture**, afin de comprendre comment TOGAF découpe et articule les différentes dimensions du système d’information.
## Chapitre 4 – Les domaines d’architecture

Dans ce chapitre, nous étudions la manière dont TOGAF structure l’architecture d’entreprise en **domaines complémentaires**. Cette décomposition n’est pas arbitraire : elle permet de **raisonner par points de vue**, tout en conservant une cohérence globale. Chaque domaine répond à des questions différentes, mais aucun ne peut être conçu de manière isolée.

![Image](https://upload.wikimedia.org/wikipedia/commons/a/a1/TOGAF_ADM.jpg)

![Image](https://www.dragon1.com/images/Layers_of_the_Enterprise_Architecture.jpg)

### 1. L’architecture métier (Business Architecture)

L’architecture métier constitue le **point de départ logique** de toute démarche d’architecture d’entreprise. Elle décrit **ce que fait l’organisation**, pourquoi elle le fait, et comment elle crée de la valeur.

Elle couvre notamment :

- les **objectifs stratégiques**,    
- les **processus métiers**,    
- les **acteurs et rôles**,    
- les règles, politiques et contraintes organisationnelles.    

Nous insistons sur un point fondamental :  
l’architecture métier ne dépend pas de l’informatique, même si elle s’appuie ensuite sur elle. Elle doit pouvoir être comprise par les décideurs, les responsables métiers et les architectes.

Dans TOGAF, l’architecture métier permet :

- de traduire la stratégie en capacités opérationnelles,    
- de justifier les choix applicatifs et techniques,    
- d’éviter un SI déconnecté des besoins réels.    

### 2. L’architecture des données (Data Architecture)

L’architecture des données s’intéresse à **l’information comme actif stratégique**. Elle décrit comment les données sont :

- définies,    
- produites,    
- stockées,    
- échangées,    
- gouvernées.    

Elle inclut :

- les modèles de données conceptuels et logiques,    
- les flux d’information,    
- les responsabilités liées aux données,    
- les règles de qualité, de sécurité et de conformité.    

Dans de nombreuses organisations, la donnée est un point de tension majeur. Sans architecture claire, on observe :

- des redondances,    
- des incohérences,    
- une perte de confiance dans les données.    

TOGAF positionne l’architecture des données comme un **pont** entre le métier (sens de la donnée) et les applications (usage de la donnée).

### 3. L’architecture applicative (Application Architecture)

L’architecture applicative décrit l’ensemble des **applications et services** qui soutiennent les processus métiers. Elle ne se limite pas à un inventaire, mais s’intéresse à :

- la répartition des responsabilités fonctionnelles,    
- les interactions entre applications,    
- les dépendances et redondances,    
- les principes d’urbanisation.    

Elle répond à des questions telles que :

- quelles applications supportent quels processus ?    
- comment les applications communiquent-elles ?    
- où se situent les points de couplage critiques ?    

Dans une perspective TOGAF, l’architecture applicative permet :

- de maîtriser la complexité du parc applicatif,    
- de faciliter l’évolution du SI,    
- de guider les décisions de rationalisation ou de modernisation.    

### 4. L’architecture technologique (Technology Architecture)

L’architecture technologique constitue le **socle d’exécution** du système d’information. Elle décrit :

- les infrastructures (serveurs, réseaux, cloud),
- les environnements techniques,
- les plateformes et middleware,
- les standards technologiques.

Contrairement à une approche purement opérationnelle, TOGAF aborde l’architecture technologique sous l’angle :

- de la cohérence globale,
- de la soutenabilité,
- de l’alignement avec les besoins métiers et applicatifs.

Nous ne cherchons pas à optimiser une technologie isolée, mais à garantir que les choix techniques **soutiennent durablement** l’architecture cible.

### 5. Relations et dépendances entre les domaines

Un point central de TOGAF est le refus d’une approche en silos. Les domaines d’architecture sont **fortement interdépendants** :

- l’architecture métier **oriente** les autres domaines,
- l’architecture des données **structure** l’information manipulée,
- l’architecture applicative **implémente** les capacités métiers,
- l’architecture technologique **supporte** l’ensemble.

Toute modification dans un domaine a des **impacts en chaîne** sur les autres.  
C’est précisément pour gérer ces dépendances que l’architecture d’entreprise est nécessaire.

Nous retenons donc que la valeur de TOGAF ne réside pas uniquement dans la description de chaque domaine, mais dans sa capacité à :

- rendre visibles les liens entre eux,
- anticiper les impacts des décisions,
- garantir une cohérence d’ensemble dans le temps.
### Conclusion du chapitre

Les domaines d’architecture proposés par TOGAF offrent une **grille de lecture structurante** du système d’information. Ils permettent d’aborder la complexité sans la réduire artificiellement, en maintenant un équilibre entre spécialisation et vision globale.

Dans la suite du cours, nous entrerons dans le **cœur opérationnel de TOGAF** en étudiant plus en détail l’**ADM**, et la manière dont ces domaines sont mobilisés concrètement au fil des phases de la méthode.