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

# Partie III – L’ADM : méthode de développement de l’architecture

## Chapitre 5 – Vue d’ensemble de l’ADM

Ce chapitre marque une étape clé du cours : nous entrons dans le **cœur opérationnel de TOGAF**, à savoir l’**ADM (Architecture Development Method)**.  
L’ADM n’est pas un simple enchaînement de phases, mais une **méthode de raisonnement et de pilotage** de l’architecture d’entreprise.

![Image](https://www.researchgate.net/publication/224258195/figure/fig3/AS%3A654358022717441%401533022518145/The-ADM-cycle-in-TOGAF-18.png)

![Image](https://miro.medium.com/0%2AC3OYil_iiDLIXBg6.gif)

## 1. Principe itératif et cyclique

Contrairement à une démarche linéaire classique (analyse → conception → réalisation), l’ADM repose sur un **principe fondamental : l’itération**.

### Une démarche cyclique

L’ADM est représenté sous la forme d’un **cycle**, et non d’une chaîne. Cela signifie que :

- l’architecture n’est jamais considérée comme définitivement achevée,    
- chaque itération permet d’ajuster, d’affiner ou de corriger les choix précédents,    
- les retours du terrain (projets, contraintes, évolutions stratégiques) alimentent les cycles suivants.    

Nous ne cherchons donc pas à produire une architecture parfaite, mais une architecture **progressivement maîtrisée**.

### Adaptation au contexte

Toutes les phases de l’ADM ne sont pas nécessairement utilisées avec la même intensité :

- certaines organisations vont fortement investir les phases de cadrage,    
- d’autres se concentreront sur la transformation et la gouvernance,    
- certaines phases peuvent être regroupées ou allégées.    

Cette souplesse est essentielle : **l’ADM est un cadre adaptable**, pas un processus rigide.

## 2. Notion de baseline et de target architecture

Un concept central de l’ADM est la distinction entre **l’existant** et **le futur souhaité**.

### Baseline Architecture

La _baseline architecture_ correspond à :

- l’état actuel du système d’information,
- tel qu’il est réellement, et non tel qu’il est supposé être,
- incluant ses forces, ses incohérences et ses contraintes.

Cette étape est souvent délicate, car :

- la documentation est parfois incomplète ou obsolète,
- le SI peut être historiquement fragmenté,
- certaines décisions passées ne sont plus justifiées.

Pourtant, une baseline mal comprise conduit presque systématiquement à des **architectures cibles irréalistes**.

### Target Architecture

La _target architecture_ décrit l’état futur visé :

- aligné avec la stratégie de l’organisation,
- cohérent entre les domaines métier, données, applicatif et technique,
- soutenable dans le temps.

Il ne s’agit pas d’une projection technologique idéalisée, mais d’un **objectif atteignable**, tenant compte :

- des contraintes organisationnelles,
- des capacités financières,
- du rythme de transformation acceptable.

### L’écart comme objet central

L’ADM se concentre fortement sur l’**écart entre la baseline et la target** :

- ce sont ces écarts qui justifient les projets,
- ils structurent les roadmaps,
- ils permettent de prioriser les actions.

## 3. Gouvernance et pilotage de l’architecture

Un point essentiel de l’ADM est l’intégration explicite de la **gouvernance**.  
TOGAF ne se contente pas de définir une architecture : il cherche à **s’assurer qu’elle est respectée et maîtrisée dans le temps**.

### Architecture et décision

L’architecture devient un outil de pilotage :

- elle éclaire les choix stratégiques,
- elle permet d’arbitrer entre plusieurs solutions,
- elle rend explicites les impacts des décisions.

L’architecte n’est donc pas un simple concepteur, mais un **acteur de la gouvernance**.

### Lien avec les projets

Dans l’ADM :

- les projets sont des **moyens** de mise en œuvre de l’architecture,    
- ils ne doivent pas la redéfinir seuls,    
- leur conformité architecturale est évaluée.    

Cette logique permet d’éviter que le SI n’évolue uniquement sous la pression de projets isolés.

### Amélioration continue

Enfin, la gouvernance s’inscrit dans une logique d’**amélioration continue** :

- retour d’expérience,    
- mise à jour des principes,    
- évolution des standards,    
- adaptation aux nouveaux contextes.    

L’ADM ne fige pas l’architecture ; il **organise son évolution**.

### Conclusion du chapitre

Nous retenons que l’ADM est avant tout :

- une **méthode itérative**, orientée long terme,    
- un cadre structurant pour passer de l’existant au futur,    
- un outil de gouvernance autant que de conception.    

Dans les chapitres suivants, nous entrerons dans le détail des **phases de l’ADM**, en analysant successivement le cadrage, la conception des différents domaines d’architecture, puis la transformation et la gouvernance.

## **Chapitre 6 – Phases préliminaire et Vision (Préliminaire & Phase A)**

Dans ce chapitre, nous abordons les **premières étapes de l’ADM**, qui constituent les fondations de toute démarche d’architecture d’entreprise.  
Avant de produire des modèles ou des diagrammes, TOGAF insiste sur un point essentiel : **l’architecture doit être cadrée, légitimée et alignée sur les enjeux métiers**.

Les phases préliminaire et A visent donc à répondre à deux questions fondamentales :

- _Dans quel cadre allons-nous faire de l’architecture ?_    
- _Pourquoi cette architecture est-elle utile à l’organisation ?_    

### **1. Mise en place du cadre d’architecture (Phase Préliminaire)**

La phase préliminaire consiste à **installer la fonction architecture** au sein de l’organisation.  
Il ne s’agit pas encore de concevoir une architecture cible, mais de définir :

- le périmètre de la démarche,
- les règles de gouvernance,
- les rôles et responsabilités,
- les processus de décision.

Nous cherchons à créer un **cadre de travail stable**, au sein duquel les décisions d’architecture pourront être prises de manière cohérente et traçable.

Cette phase comprend généralement :

- l’évaluation de la **maturité architecturale** de l’organisation,
- l’identification des pratiques existantes,
- l’adaptation de TOGAF au contexte réel (taille, culture, contraintes),
- la mise en place d’un **organe de gouvernance**, souvent appelé _Architecture Board_.

Sans ce travail préparatoire, l’architecture risque d’être :

- ignorée par les projets,
- perçue comme une contrainte administrative,
- ou contournée par les équipes opérationnelles.

La phase préliminaire vise donc à **légitimer la fonction d’architecture** et à lui donner un cadre clair.

### **2. Définition des principes d’architecture**

Un des livrables majeurs de cette phase est la définition des **principes d’architecture**.

Les principes sont des **règles directrices**, stables dans le temps, qui orientent les décisions. Ils permettent de garantir la cohérence du système d’information, même lorsque de nombreux projets sont menés en parallèle.

Un principe d’architecture comporte généralement :

- un **intitulé clair**,
- une **description**,
- une **justification métier**,
- des **implications concrètes** sur les choix futurs.

Exemples de principes courants :

- priorité à la réutilisation des composants existants,
- interopérabilité entre les systèmes,
- centralité et qualité des données,
- sécurité intégrée dès la conception,
- préférence pour des standards ouverts.

Nous insistons sur un point important :  
les principes ne sont pas de simples déclarations d’intention. Ils servent d’**outil d’arbitrage**.  
Lorsqu’un projet doit choisir entre plusieurs solutions, ce sont les principes qui permettent de trancher de manière cohérente.

### **3. Identification et gestion des parties prenantes (Phase A)**

La phase A introduit explicitement la notion de **parties prenantes** (_stakeholders_).  
L’architecture d’entreprise ne se limite pas à une activité technique : elle implique des décisions qui affectent les processus, les organisations et les responsabilités.

Nous devons donc identifier les acteurs concernés, par exemple :

- la direction générale,
- les responsables métiers,
- les équipes informatiques,
- les responsables de la sécurité et de la conformité,
- les partenaires externes.

Pour chaque partie prenante, nous analysons :

- ses objectifs,
- ses contraintes,
- son niveau d’influence,
- ses éventuelles résistances.

L’objectif n’est pas seulement de dresser une liste d’acteurs, mais de construire une **stratégie d’adhésion**.  
Une architecture, même techniquement parfaite, échoue si elle n’est pas acceptée par ceux qui doivent l’appliquer.

### **4. Vision cible et valeur métier (Phase A)**

La phase A se conclut par la formulation d’une **vision d’architecture**.

Cette vision n’est pas encore une architecture détaillée. Elle constitue :

- une **direction stratégique**,
- un message clair sur les bénéfices attendus,
- un point de référence pour les phases suivantes.

Nous cherchons ici à répondre à une question centrale :

> _Quelle valeur l’architecture apportera-t-elle à l’organisation ?_

La vision doit être compréhensible par les décideurs et les métiers. Elle met en avant :

- les gains d’efficacité,
- la réduction des coûts ou des risques,
- l’amélioration de la qualité des données,
- la capacité d’adaptation aux évolutions futures.

Elle peut s’exprimer sous la forme :

- d’un document de synthèse,
- d’une carte des capacités cibles,
- d’un scénario d’évolution du SI.

Cette vision sert de **fil conducteur** pour l’ensemble du cycle ADM. Elle permet d’éviter que la démarche d’architecture ne dérive vers un exercice purement technique ou documentaire.

### **Conclusion du chapitre**

Les phases préliminaire et Vision jouent un rôle fondamental dans l’ADM. Elles posent les bases :

- organisationnelles,
- stratégiques,
- et politiques de la démarche d’architecture.

Nous retenons que l’architecture d’entreprise commence non pas par des diagrammes, mais par :

- un cadre de gouvernance,
- des principes directeurs,
- une compréhension des parties prenantes,
- et une vision claire de la valeur métier.

Dans le chapitre suivant, nous entrerons dans la **conception proprement dite de l’architecture**, en commençant par la **phase B : l’architecture métier**, qui constitue le socle des autres domaines.

## **Chapitre 7 – Phases B, C et D : conception de l’architecture**

Dans ce chapitre, nous entrons dans le **cœur de la conception architecturale** dans l’ADM.  
Après avoir posé le cadre et défini une vision d’ensemble, les phases B, C et D visent à construire les **architectures cibles détaillées** pour chacun des grands domaines du système d’information.

Ces phases correspondent à un mouvement logique :

1. comprendre et structurer les **activités métier**,    
2. organiser les **données et les applications** qui les supportent,    
3. définir l’**infrastructure technique** permettant leur exécution.    

L’objectif n’est pas de produire des documents isolés, mais de construire une **architecture cohérente entre ses différentes couches**.

### **1. Phase B : architecture métier**

La phase B consiste à décrire l’**architecture métier cible**, en cohérence avec la vision définie lors de la phase A.

Nous cherchons ici à répondre à la question :

> _Comment l’organisation doit-elle fonctionner pour atteindre ses objectifs stratégiques ?_

Cette phase comprend généralement :

- l’identification des **capacités métiers**,    
- la modélisation des **processus clés**,    
- la description des **acteurs et rôles**,    
- l’analyse des écarts entre l’existant et la cible.    

L’architecture métier sert de **fondation** pour les autres domaines.  
Si elle est mal définie, les choix applicatifs et techniques risquent d’être incohérents ou inutiles.

Les principaux livrables de cette phase peuvent inclure :

- des cartes de processus,    
- des modèles d’organisation,    
- des catalogues d’acteurs et de capacités,    
- une analyse des écarts entre la situation actuelle et la cible.    

### **2. Phase C : architecture des données et des applications**

La phase C est souvent divisée en deux sous-parties :

- l’architecture des **données**,    
- l’architecture **applicative**.    

Ces deux dimensions sont étroitement liées, car les applications manipulent et produisent les données.

#### **Architecture des données**

L’architecture des données vise à structurer l’information nécessaire aux processus métiers.  
Nous cherchons notamment à :

- identifier les **objets de données clés**,
- modéliser leurs relations,
- décrire les flux d’information,
- définir les règles de gouvernance des données.

L’objectif est de garantir :

- la cohérence des données,
- leur qualité,
- leur disponibilité pour les métiers.

Cette étape est cruciale dans les organisations où la donnée constitue un **actif stratégique**.

#### **Architecture applicative**

L’architecture applicative décrit l’ensemble des applications et services nécessaires pour soutenir les processus métiers.

Elle comprend :

- l’inventaire des applications existantes,
- la définition des applications cibles,    
- la répartition des responsabilités fonctionnelles,
- les interactions entre applications.

Nous cherchons à répondre à des questions telles que :

- quelle application supporte quel processus ?
- quelles redondances existent dans le SI ?
- quels systèmes doivent être modernisés ou remplacés ?

Les livrables de cette phase peuvent inclure :

- des cartographies applicatives,
- des matrices processus/applications,    
- des diagrammes d’interactions entre systèmes.

### **3. Phase D : architecture technologique**

La phase D consiste à définir l’**architecture technique cible**.  
Elle décrit l’environnement dans lequel les applications seront déployées et exécutées.

Cette phase couvre :

- les infrastructures matérielles,
- les environnements cloud ou on-premise,
- les réseaux et la sécurité,
- les plateformes techniques et middleware,
- les standards technologiques.

Nous cherchons ici à répondre à la question :

> _Quelle infrastructure technique est nécessaire pour supporter l’architecture applicative et les besoins métiers ?_

L’objectif n’est pas de choisir une technologie pour elle-même, mais de garantir :

- la cohérence technique,
- la maintenabilité,
- la sécurité,
- la performance,
- la capacité d’évolution.

### **4. Modélisation, cohérence et arbitrages**

Les phases B, C et D ne doivent pas être menées de manière indépendante.  
Leur valeur réside dans la **cohérence globale** de l’architecture.

#### **La modélisation comme outil de communication**

La modélisation permet :

- de représenter les processus,
- de visualiser les flux de données,
- de comprendre les interactions entre systèmes.

Les modèles servent avant tout à :

- faciliter les échanges entre parties prenantes,
- rendre visibles les dépendances,
- soutenir la prise de décision.

#### **La gestion des cohérences**

Chaque domaine influence les autres :

- un changement métier peut nécessiter de nouvelles applications,
- une contrainte technique peut imposer une modification applicative,
- un problème de données peut affecter plusieurs processus.

L’architecte doit donc maintenir une **vision transversale**.

#### **Les arbitrages architecturaux**

Ces phases sont aussi des moments d’arbitrage :

- standardisation ou spécialisation,
- centralisation ou distribution,
- achat ou développement interne,
- modernisation ou maintien en l’état.

Ces arbitrages doivent être :

- justifiés par les principes d’architecture,
- alignés sur la vision métier,
- documentés et gouvernés.

### **Conclusion du chapitre**

Les phases B, C et D constituent le **noyau de la conception architecturale** dans TOGAF.  
Elles permettent de construire une architecture cible cohérente, en partant des besoins métiers pour aboutir aux choix techniques.

Nous retenons que :

- l’architecture métier structure la réflexion,
- l’architecture des données et des applications implémente les capacités,
- l’architecture technique fournit le socle d’exécution,
- la cohérence entre ces domaines est la responsabilité centrale de l’architecte.

Dans le chapitre suivant, nous aborderons les phases E et F, qui concernent la **planification de la transformation** et la construction des trajectoires de migration vers l’architecture cible.

## **Chapitre 8 – Phases E et F : transformation et migration**

Dans ce chapitre, nous passons d’une logique de **conception architecturale** à une logique de **mise en œuvre concrète**.  
Les phases B, C et D ont permis de définir une architecture cible cohérente. Les phases E et F répondent maintenant à une question essentielle :

> _Comment passer de l’architecture actuelle à l’architecture cible, de manière réaliste, maîtrisée et gouvernable ?_

Ces phases constituent le lien entre :

- la **vision architecturale**,    
- et les **projets réels** de transformation.
## **1. Opportunités et solutions (Phase E)**

La phase E consiste à identifier les **solutions concrètes** permettant de réduire l’écart entre l’architecture actuelle (_baseline_) et l’architecture cible (_target_).

Nous partons de l’analyse des écarts réalisée dans les phases précédentes :

- processus à transformer,
- applications à remplacer ou à créer,
- infrastructures à moderniser,
- données à restructurer.

Ces écarts deviennent des **opportunités de transformation**.

### **Identification des solutions**

Chaque écart peut donner lieu à plusieurs solutions possibles :

- développement d’une nouvelle application,
- acquisition d’une solution du marché,
- refonte d’un processus métier,
- migration vers une infrastructure cloud,
- mise en place d’un référentiel de données.

L’objectif n’est pas encore de planifier précisément les projets, mais de :

- regrouper les actions en **solutions cohérentes**,
- identifier les **grands chantiers**,
- estimer les impacts organisationnels et techniques.

### **Regroupement en blocs de solutions**

TOGAF propose souvent de regrouper les actions en **work packages** ou **solution building blocks** :

- modernisation du système de gestion des clients,
- mise en place d’une plateforme de données,
- refonte de l’infrastructure réseau,
- rationalisation du parc applicatif.

Ces blocs facilitent la compréhension globale et préparent la phase de planification.

## **2. Scénarios de migration (Phase F)**

Une fois les solutions identifiées, la phase F consiste à définir **comment et dans quel ordre** elles seront mises en œuvre.

Nous ne cherchons pas une trajectoire unique et idéale, mais plusieurs **scénarios de migration** possibles.

### **Construction de scénarios**

Un scénario de migration décrit :

- une séquence de projets,
- un ordre de transformation,
- un rythme de déploiement.

Plusieurs scénarios peuvent être envisagés :

- transformation rapide et centralisée,
- migration progressive par domaine,
- modernisation par priorités métiers,
- transformation opportuniste liée aux projets existants.

Chaque scénario est évalué selon :

- les coûts,
- les risques,
- les délais,
- les impacts organisationnels.

## **3. Élaboration des roadmaps**

À partir des scénarios retenus, nous construisons une **roadmap de transformation**.

La roadmap est une représentation temporelle de :

- l’enchaînement des projets,
- les jalons principaux,
- les dépendances entre initiatives.

Elle permet de répondre à des questions concrètes :

- quelles transformations doivent être réalisées en priorité ?
- quels projets peuvent être menés en parallèle ?
- quelles dépendances techniques ou organisationnelles existent ?

### **Caractéristiques d’une bonne roadmap**

Une roadmap efficace doit être :

- **réaliste**, en tenant compte des capacités de l’organisation,
- **lisible**, pour les décideurs et les métiers,
- **priorisée**, selon la valeur métier et les risques,
- **évolutive**, pour s’adapter aux changements.

La roadmap constitue un **outil de pilotage stratégique**, et non un simple planning technique.

## **4. Gestion des risques et des contraintes**

Les phases E et F intègrent explicitement la **gestion des risques** et des contraintes.  
Toute transformation architecturale comporte des incertitudes, qu’il est nécessaire d’anticiper.

### **Types de risques**

Les principaux types de risques rencontrés sont :

- risques techniques (complexité, obsolescence, interopérabilité),
- risques organisationnels (résistance au changement, manque de compétences),
- risques financiers (dépassement de budget),
- risques opérationnels (interruption de service, perte de données).

### **Contraintes à prendre en compte**

Les contraintes peuvent être :

- budgétaires,
- réglementaires,
- contractuelles,
- techniques,
- organisationnelles.

Une transformation idéale sur le papier peut être **irréalisable dans les faits** si ces contraintes ne sont pas prises en compte.

### **Stratégies d’atténuation**

Pour chaque risque identifié, nous définissons :

- des mesures de réduction,
- des plans de contingence,
- des priorités de sécurisation.

La gestion des risques est un élément central pour construire une trajectoire **crédible et acceptable**.

## **Conclusion du chapitre**

Les phases E et F marquent la transition entre :

- l’architecture comme **vision et conception**,
- et l’architecture comme **programme de transformation**.

Nous retenons que ces phases visent à :

- identifier les solutions concrètes,
- construire des scénarios de migration,
- établir une roadmap réaliste,
- maîtriser les risques et contraintes.

Elles traduisent l’architecture cible en **trajectoire opérationnelle**, reliant directement le travail des architectes aux projets de transformation.

Dans le chapitre suivant, nous aborderons les phases G et H, consacrées à la **gouvernance de l’architecture** et à son **amélioration continue**.