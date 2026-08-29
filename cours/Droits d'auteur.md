---
schema_version: 1
uid: 01M02EX5AY11W0XJQ58DDC2AE2
titre: Droits d'auteur
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - droit
  - propriete-intellectuelle
  - droit-auteur
  - logiciel-libre
  - intelligence-artificielle
resume: "Cours actualisé sur le droit d'auteur français et européen : originalité, droits moraux et patrimoniaux, logiciels, bases de données, licences libres, domaine public, exceptions, contrats, IA générative et fouille de textes et de données."
niveau: intermediaire
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-04-22
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Droit d'auteur

> [!abstract] Objectif
> Comprendre le droit d'auteur français et européen tel qu'il s'applique en 2026 à un développeur, un enseignant ou un auteur : originalité, droit moral et droits patrimoniaux, régime particulier du logiciel et des bases de données, licences libres, domaine public, exceptions (fouille de textes et de données, enseignement), contrats, et les questions ouvertes par l'IA générative et l'AI Act.

Voir aussi : [[Règlement Général sur la Protection des Données (RGPD)]], [[LibGen]], [[National Emergency Library]], [[Outils IA]], [[LLM]].

> [!warning] Portée du cours
> Ce cours expose les principes généraux du droit français et européen au **29 août 2026**. Il ne remplace pas une consultation juridique : un contrat, une licence, une chaîne de droits ou une durée de protection peuvent dépendre de faits précis, de la loi applicable et de la jurisprudence.

Le droit d'auteur est une branche de la propriété intellectuelle. Il protège certaines **formes d'expression originales** sans exiger, en France, de dépôt constitutif du droit.

Pour un développeur, un chercheur ou un créateur numérique, il faut distinguer plusieurs questions qui sont souvent mélangées :

1. **Est-ce que l'objet est protégeable ?**
2. **Qui est auteur ou titulaire des droits ?**
3. **Quels droits existent ?**
4. **Pendant combien de temps ?**
5. **Existe-t-il une exception permettant l'usage sans autorisation ?**
6. **Une licence ou un contrat donne-t-il une autorisation supplémentaire ?**
7. **D'autres droits s'ajoutent-ils ?** Brevet, marque, base de données, secret des affaires, droit à l'image, données personnelles, etc.

Cette méthode évite une erreur fréquente : croire que « accessible sur Internet », « gratuit », « open source », « dans un dépôt Git » ou « généré par une IA » signifie automatiquement « libre de droits ».

## Sommaire

1. Cartographie de la propriété intellectuelle
2. Naissance du droit d'auteur
3. Idée, information, fonctionnalité et forme d'expression
4. Originalité
5. Auteur, coauteurs et catégories d'œuvres
6. Droit moral et droits patrimoniaux
7. Durée de protection et domaine public
8. Exceptions au droit d'auteur
9. Cession et contrats
10. Régime particulier du logiciel
11. Brevet et invention mise en œuvre par ordinateur
12. Bases de données
13. Licences de logiciels : principes
14. Licences permissives
15. Copyleft : GPL, LGPL, AGPL, MPL et CeCILL
16. Compatibilité des licences et dépendances
17. Creative Commons, documentation et contenus
18. Conformité open source dans un projet logiciel
19. Git, contributions et chaîne de droits
20. API, SaaS et distribution
21. Intelligence artificielle et statut des sorties
22. Entraînement des IA et fouille de textes et de données
23. AI Act et droit d'auteur
24. Preuve, dépôt et traçabilité
25. Contrefaçon et réactions possibles
26. Chronologie et calcul du domaine public
27. Études de cas
28. Checklist de décision
29. Travaux pratiques
30. Projet final
31. Références

---

# 1. Cartographie de la propriété intellectuelle

La **propriété intellectuelle** n'est pas un droit unique. Elle regroupe plusieurs régimes qui peuvent protéger simultanément un même produit.

| Objet | Protection principale possible | Exemple |
|---|---|---|
| Code source original | droit d'auteur | bibliothèque Python |
| Interface graphique originale | droit d'auteur | illustrations et composition graphique |
| Nom d'un produit | marque | nom d'un service SaaS |
| Invention technique | brevet | procédé industriel piloté par logiciel |
| Design d'un objet | dessin et modèle | boîtier d'un appareil |
| Base structurée originale | droit d'auteur | sélection et organisation originale |
| Investissement dans une base | droit sui generis du producteur | base de données coûteuse à constituer |
| Algorithme secret | secret des affaires | méthode interne non divulguée |
| Documentation | droit d'auteur | manuel technique original |
| Modèle 3D | droit d'auteur et parfois dessin/modèle | pièce esthétique |

## 1.1 Propriété littéraire et artistique

Elle comprend notamment :

- le **droit d'auteur** ;
- certains **droits voisins**, par exemple ceux des artistes-interprètes et producteurs ;
- des régimes spéciaux concernant certaines créations.

## 1.2 Propriété industrielle

Elle comprend notamment :

- brevets ;
- marques ;
- dessins et modèles ;
- indications géographiques ;
- certificats d'utilité.

Un brevet d'invention français est délivré pour une durée maximale de **20 ans à compter du dépôt**, sous réserve notamment du paiement des annuités.

## 1.3 Plusieurs protections peuvent se cumuler

Une application mobile peut comporter :

- du code protégé par droit d'auteur ;
- une marque déposée ;
- une base de données protégée ;
- des secrets d'affaires ;
- des éléments graphiques protégés ;
- éventuellement une invention technique brevetable.

Il faut donc éviter la phrase :

> « Le logiciel est protégé par X. »

Une formulation plus exacte est :

> « Plusieurs éléments du produit peuvent relever de régimes juridiques différents. »

---

# 2. Naissance du droit d'auteur

## 2.1 Protection sans formalité constitutive

En France, le droit d'auteur naît **du seul fait de la création d'une œuvre protégeable**.

Il n'est pas nécessaire de :

- déposer l'œuvre ;
- apposer le symbole © ;
- l'enregistrer auprès d'un organisme ;
- publier l'œuvre ;
- la vendre.

Un dépôt peut cependant être très utile pour **prouver une date, un contenu ou une paternité**.

## 2.2 Le symbole ©

Exemple :

```text
© 2026 Michaël Launay — Tous droits réservés
```

Ce symbole n'est pas ce qui crée le droit d'auteur en France. Il a surtout une fonction d'information.

## 2.3 Pas de droit d'auteur sur une simple idée

Le droit d'auteur protège une **forme d'expression**, pas l'idée abstraite qui la sous-tend.

Exemple :

- idée : « créer une application qui classe automatiquement des photos » ;
- expression possible : code, textes, icônes, documentation, structure originale d'une interface.

Une personne peut donc réimplémenter une même idée en produisant sa propre expression, sous réserve d'autres droits éventuels.

---

# 3. Idée, information, fonctionnalité et forme d'expression

Cette distinction est centrale en informatique.

## 3.1 Ce qui n'est pas protégé comme tel par le droit d'auteur

Ne sont pas appropriables par le droit d'auteur en tant que tels :

- une idée ;
- un concept ;
- une information brute ;
- une méthode mathématique ;
- une fonctionnalité abstraite ;
- un langage de programmation en tant que système ;
- un algorithme en tant que principe abstrait ;
- une syntaxe imposée uniquement par une contrainte technique.

Cela ne signifie pas qu'aucun autre régime ne puisse intervenir.

## 3.2 Ce qui peut être protégé

Peuvent notamment être protégés s'ils sont originaux :

- code source ;
- code objet ;
- documentation ;
- textes d'interface ;
- graphismes ;
- illustrations ;
- composition d'une interface ;
- certains éléments préparatoires d'un logiciel ;
- sélection ou organisation originale de données.

## 3.3 Fonctionnalité identique, code différent

Deux programmes peuvent fournir la même fonction sans être des copies l'un de l'autre.

```text
Fonctionnalité
    ↓
« Trier des fichiers par date »
    ↓
Plusieurs implémentations indépendantes possibles
```

La frontière devient plus délicate lorsque :

- la structure interne est copiée ;
- les mêmes choix arbitraires sont reproduits ;
- des portions de code ou de documentation sont reprises ;
- des éléments non nécessaires au fonctionnement sont reproduits de façon très proche.

---

# 4. Originalité

## 4.1 Critère essentiel

Le droit d'auteur ne protège pas toute production intellectuelle. Une œuvre doit présenter une **originalité**.

Dans la conception européenne, l'originalité est généralement rattachée à l'existence de **choix libres et créatifs** traduisant une contribution intellectuelle propre de l'auteur.

Ce n'est pas :

- la nouveauté au sens du brevet ;
- la qualité esthétique ;
- le temps de travail ;
- la difficulté technique ;
- l'utilité ;
- le prix de vente.

## 4.2 Originalité ≠ nouveauté

Une photographie d'un bâtiment très connu peut être originale par :

- cadrage ;
- éclairage ;
- angle ;
- mise en scène ;
- traitement créatif.

Même si des millions de photographies du même bâtiment existent déjà.

## 4.3 Les contraintes peuvent limiter l'originalité

Plus une forme est imposée par :

- une norme ;
- une API ;
- la compatibilité ;
- une exigence technique ;
- une convention obligatoire ;

moins il reste d'espace pour des choix créatifs.

Exemple :

```python
if __name__ == "__main__":
    main()
```

Une telle formule courante ne doit pas être traitée comme si son auteur pouvait monopoliser toute utilisation de cette construction.

## 4.4 L'originalité s'apprécie concrètement

Une œuvre volumineuse peut contenir :

- des éléments originaux ;
- des éléments banals ;
- des éléments imposés techniquement ;
- des éléments empruntés sous licence.

Le raisonnement doit donc porter sur **ce qui est réellement repris**.

---

# 5. Auteur, coauteurs et catégories d'œuvres

## 5.1 L'auteur

En droit d'auteur, l'auteur est en principe la **personne physique** qui crée l'œuvre.

Une entreprise peut devenir titulaire de droits patrimoniaux :

- par cession ;
- par un régime légal spécial ;
- par le régime de l'œuvre collective ;
- dans le cas particulier du logiciel salarié.

Mais il faut distinguer :

```text
Auteur
≠
Titulaire actuel des droits patrimoniaux
```

## 5.2 Œuvre de collaboration

Une œuvre de collaboration est créée par plusieurs personnes physiques qui ont participé à sa création.

Exemples possibles :

- article écrit à quatre mains ;
- logiciel dont plusieurs développeurs ont créé conjointement des parties originales ;
- musique composée conjointement.

Les coauteurs exercent alors des droits dans le cadre du régime applicable à l'œuvre de collaboration.

## 5.3 Œuvre collective

L'œuvre collective répond à des critères juridiques particuliers. Elle est créée à l'initiative d'une personne qui :

- l'édite ;
- la publie et la divulgue sous son nom ;
- dirige l'ensemble ;
- fait fusionner les contributions dans l'ensemble sans qu'il soit possible d'attribuer à chaque contributeur un droit distinct sur l'ensemble réalisé.

Une encyclopédie ou certains journaux peuvent en être des exemples classiques.

> [!important]
> Le simple fait qu'un dépôt Git contienne des centaines de commits identifiables **ne suffit ni à créer ni à exclure automatiquement** la qualification d'œuvre collective. La qualification dépend des conditions juridiques de création, pas de la seule traçabilité technique.

## 5.4 Œuvre composite

Une œuvre composite incorpore une œuvre préexistante sans collaboration de son auteur.

Exemples :

- traduction ;
- adaptation ;
- remix ;
- documentation incorporant des illustrations tierces.

L'auteur de l'œuvre nouvelle doit respecter les droits attachés aux éléments préexistants.

## 5.5 Œuvres anonymes et pseudonymes

La durée de protection peut être calculée à partir de la publication lorsque l'identité de l'auteur n'est pas révélée dans les conditions prévues par la loi.

Il ne faut pas utiliser cette règle lorsque l'identité réelle de l'auteur est en fait connue juridiquement et permet d'appliquer le régime normal.

---

# 6. Droit moral et droits patrimoniaux

Le droit d'auteur français comprend deux grandes familles de prérogatives.

## 6.1 Le droit moral

Le droit moral protège le lien personnel entre l'auteur et son œuvre.

Il comprend notamment :

- droit au nom et à la qualité ;
- droit au respect de l'œuvre ;
- droit de divulgation ;
- droit de retrait et de repentir dans les conditions prévues par la loi.

L'article L.121-1 du Code de la propriété intellectuelle qualifie le droit moral de :

- perpétuel ;
- inaliénable ;
- imprescriptible.

Il subsiste donc même après l'expiration des droits patrimoniaux.

## 6.2 Les droits patrimoniaux

Ils permettent de contrôler l'exploitation économique de l'œuvre.

Les deux notions centrales sont :

- **reproduction** : fixation/copie de l'œuvre ;
- **représentation** : communication de l'œuvre au public.

S'y rattachent selon le contexte :

- adaptation ;
- traduction ;
- distribution ;
- mise à disposition en ligne.

## 6.3 Droit moral du logiciel : régime particulier

Le logiciel n'est pas soumis exactement au même régime moral qu'un roman ou un tableau.

L'article L.121-7 limite notamment les possibilités pour l'auteur d'un logiciel de :

- s'opposer à certaines modifications réalisées par le cessionnaire ;
- exercer le droit de repentir ou de retrait.

Il est donc incorrect d'affirmer simplement :

> « Le salarié conserve tous les droits moraux ordinaires sur le logiciel exactement comme sur toute autre œuvre. »

Le régime du logiciel est **spécial**.

---

# 7. Durée de protection et domaine public

## 7.1 Règle générale

Pour une œuvre individuelle, les droits patrimoniaux durent en principe :

```text
vie de l'auteur
+
année civile de son décès
+
70 années civiles
```

Un auteur décédé en **1955** voit donc, dans le cas général, ses œuvres entrer dans le domaine public en France le **1er janvier 2026**.

Au 29 août 2026, la règle générale conduit donc à considérer comme dans le domaine public les œuvres d'auteurs morts **avant 1956**, sous réserve des régimes particuliers.

## 7.2 Domaine public ne signifie pas absence de toute règle

Lorsqu'une œuvre entre dans le domaine public :

- les droits patrimoniaux ont expiré ;
- le droit moral subsiste en France ;
- une édition critique récente peut comporter des éléments nouveaux protégés ;
- une traduction récente peut être protégée ;
- une photographie récente de l'œuvre peut parfois soulever des droits distincts ;
- une marque peut encore protéger un signe ;
- le droit à l'image ou d'autres droits peuvent intervenir.

Exemple :

```text
Roman original : domaine public
Traduction de 2024 : potentiellement protégée
Préface de 2026 : potentiellement protégée
Illustrations de 2026 : potentiellement protégées
```

## 7.3 Œuvres collectives, anonymes et pseudonymes

La loi prévoit des modalités particulières, fréquemment fondées sur la date de publication.

Pour une œuvre collective, la durée patrimoniale est en principe calculée sur **70 ans à compter du 1er janvier de l'année civile suivant celle de la publication**.

## 7.4 Œuvre posthume

Lorsqu'une œuvre est divulguée après l'expiration du monopole patrimonial normal, la loi peut accorder au propriétaire de l'œuvre inédite un droit d'exploitation d'une durée de **25 ans à compter du 1er janvier suivant la publication**.

## 7.5 Les prorogations de guerre : ne pas additionner mécaniquement

Les articles L.123-8 et L.123-9 existent toujours, mais leur articulation avec la durée harmonisée de 70 ans est complexe.

La Cour de cassation a jugé en 2007 que la période harmonisée de 70 ans **couvre en principe les anciennes prorogations de guerre**, sauf situation de droits acquis où une durée plus longue avait commencé à courir au 1er juillet 1995.

Il est donc juridiquement dangereux d'utiliser une formule automatique :

```text
70 ans + 6 ans + 8 ans
```

pour tous les auteurs.

## 7.6 « Mort pour la France »

L'article L.123-10 prévoit une prorogation spécifique de **30 ans** dans les conditions qu'il fixe.

Les cas historiques doivent être calculés précisément en tenant compte :

- de la date de décès ;
- du statut « Mort pour la France » ;
- de la date de publication de l'œuvre ;
- de l'état de la protection au 1er juillet 1995 ;
- de la jurisprudence sur les droits acquis.

## 7.7 Saint-Exupéry : bon exemple d'un calcul non trivial

Antoine de Saint-Exupéry est mort pour la France en 1944 et *Le Petit Prince* a été publié en 1943.

En France, la situation particulière conduit à une protection jusqu'en **2033**, alors que l'œuvre est déjà dans le domaine public dans plusieurs autres pays.

Cet exemple illustre une règle essentielle :

> Le domaine public doit être déterminé **pays par pays et œuvre par œuvre** lorsque le cas sort de la règle générale.

---

# 8. Exceptions au droit d'auteur

Une œuvre protégée peut parfois être utilisée sans autorisation grâce à une exception légale.

Une exception n'est pas une licence générale.

## 8.1 Copie privée

La copie privée répond à des conditions précises :

- source licite ;
- usage privé du copiste ;
- absence d'utilisation collective ;
- exclusions particulières, notamment pour les logiciels.

Pour un logiciel, le régime spécifique de la copie de sauvegarde s'applique.

## 8.2 Courte citation

Une citation doit notamment :

- rester courte au regard de l'œuvre citée et de l'œuvre nouvelle ;
- être justifiée par un but critique, polémique, pédagogique, scientifique ou d'information ;
- indiquer clairement l'auteur et la source.

Il n'existe pas de règle universelle du type :

> « moins de 10 %, c'est légal ».

Un pourcentage mécanique ne remplace pas l'analyse juridique.

## 8.3 Parodie, pastiche et caricature

Ils bénéficient d'une exception dans les conditions prévues par la loi et les usages du genre.

## 8.4 Panorama

Le droit français comporte une exception concernant certaines reproductions et représentations d'œuvres architecturales et de sculptures placées en permanence sur la voie publique, mais elle est notamment limitée aux personnes physiques et exclut l'usage commercial.

La « liberté de panorama » française est donc plus étroite que dans certains pays.

## 8.5 Enseignement et recherche

Il existe des exceptions spécifiques pour :

- illustration dans l'enseignement ;
- formation professionnelle ;
- recherche ;
- certains usages au sein d'établissements.

Le simple fait qu'un usage soit « pédagogique » ne rend pas toute reproduction libre.

## 8.6 Fouille de textes et de données

La fouille de textes et de données dispose d'un régime particulier détaillé au chapitre 22.

## 8.7 Test en trois étapes

Les exceptions ne doivent pas :

- porter atteinte à l'exploitation normale de l'œuvre ;
- causer un préjudice injustifié aux intérêts légitimes de l'auteur.

---

# 9. Cession et contrats

## 9.1 Être propriétaire d'un support ne signifie pas détenir les droits d'auteur

Acheter :

- un tableau ;
- un disque dur ;
- un manuscrit ;
- un fichier ;
- un exemplaire d'un logiciel ;

ne transfère pas automatiquement tous les droits de propriété intellectuelle.

```text
propriété matérielle du support
≠
droits de propriété intellectuelle
```

## 9.2 Cession de droits

L'article L.131-3 exige notamment que :

- chaque droit cédé soit identifié distinctement ;
- le domaine d'exploitation soit délimité ;
- la destination soit précisée ;
- le lieu soit défini ;
- la durée soit définie.

Exemple de questions contractuelles :

```text
Quel droit ?       reproduction / représentation / adaptation
Pour quoi ?        application, documentation, publicité...
Où ?               France, UE, monde...
Combien de temps ? 3 ans, 10 ans, durée légale...
Sur quels supports ? web, mobile, papier...
```

## 9.3 Cession globale des œuvres futures

L'article L.131-1 prévoit que la **cession globale des œuvres futures est nulle**.

Un contrat doit donc être rédigé avec davantage de précision qu'une formule du type :

> « Je cède tout ce que je créerai un jour. »

## 9.4 Prestataire indépendant

Pour un développeur freelance ou une société prestataire, les droits patrimoniaux sur le logiciel développé ne passent pas automatiquement au client uniquement parce que celui-ci a payé la facture.

Le contrat doit traiter explicitement :

- titularité ;
- cession ou licence ;
- composants préexistants ;
- dépendances open source ;
- droit de réutilisation du prestataire ;
- maintenance ;
- documentation ;
- code source ;
- composants tiers.

## 9.5 Licence et cession

Une **cession** transfère des droits.

Une **licence** autorise certains usages tout en laissant le titulaire conserver ses droits.

C'est le modèle typique des licences de logiciels libres.

---

# 10. Régime particulier du logiciel

Le logiciel est protégé par le droit d'auteur mais bénéficie d'un régime spécial.

## 10.1 Éléments susceptibles d'être protégés

Peuvent notamment relever du droit d'auteur :

- code source ;
- code objet ;
- certains matériels de conception préparatoire ;
- documentation, indépendamment ;
- éléments graphiques originaux, indépendamment.

En revanche, ne sont pas protégés en tant que tels par le droit d'auteur :

- fonctionnalité abstraite ;
- idée ;
- algorithme abstrait ;
- langage de programmation en tant que tel.

## 10.2 Logiciel créé par un salarié

L'article L.113-9 prévoit, sauf dispositions statutaires ou stipulations contraires, que les **droits patrimoniaux** sur les logiciels et leur documentation créés par un ou plusieurs employés :

- dans l'exercice de leurs fonctions ;
- ou d'après les instructions de l'employeur ;

sont dévolus à l'employeur qui est seul habilité à les exercer.

Il s'agit d'une règle spéciale au logiciel.

## 10.3 Ne pas généraliser à toutes les créations d'un salarié

Cette règle ne signifie pas :

> « Tout ce qu'un salarié crée appartient automatiquement à son employeur. »

Une photographie, un texte, une formation ou une illustration créée par un salarié ne relève pas automatiquement de L.113-9 simplement parce que l'auteur est salarié.

## 10.4 Droit moral du développeur salarié

Comme vu au chapitre 6, le droit moral du logiciel est aménagé par L.121-7.

Il faut donc distinguer :

```text
Titularité patrimoniale spéciale du logiciel salarié
+
Droit moral du logiciel aménagé
```

## 10.5 Copie de sauvegarde

La personne ayant le droit d'utiliser le logiciel peut effectuer une copie de sauvegarde lorsque celle-ci est nécessaire pour préserver l'utilisation du logiciel.

## 10.6 Observation, étude et test

Le régime du logiciel prévoit des possibilités spécifiques permettant à l'utilisateur légitime d'observer, étudier ou tester le fonctionnement du programme dans certaines conditions.

## 10.7 Décompilation pour interopérabilité

Une décompilation peut être autorisée dans des conditions strictes lorsqu'elle est indispensable pour obtenir les informations nécessaires à l'interopérabilité d'un logiciel créé de façon indépendante.

Cette exception n'autorise pas à :

- cloner arbitrairement le produit ;
- diffuser librement le code obtenu ;
- réutiliser les informations à d'autres fins que celles permises.

---

# 11. Brevet et invention mise en œuvre par ordinateur

## 11.1 Durée du brevet

Le brevet d'invention français est délivré pour **20 ans à compter du dépôt**.

L'ancien cours indiquait à tort une durée générale de 25 ans.

## 11.2 Les programmes d'ordinateur « en tant que tels »

L'article L.611-10 exclut notamment les programmes d'ordinateur de la brevetabilité lorsqu'ils sont considérés **en tant que tels**.

Cela ne signifie pas :

> « Dès qu'une invention contient du logiciel, elle est impossible à breveter. »

## 11.3 Invention mise en œuvre par ordinateur

Une invention présentant un caractère technique peut être brevetable même si sa réalisation comporte du logiciel, à condition de satisfaire les critères de brevetabilité :

- invention ;
- nouveauté ;
- activité inventive ;
- application industrielle.

Exemples possibles à analyser :

- commande améliorée d'une machine ;
- traitement technique d'un signal ;
- gestion technique d'un réseau ;
- procédé industriel piloté par ordinateur.

## 11.4 Droit d'auteur et brevet ne protègent pas la même chose

```text
Droit d'auteur
→ forme d'expression du code

Brevet
→ invention technique revendiquée
```

Le même produit peut donc combiner les deux régimes.

---

# 12. Bases de données

Une base de données peut cumuler deux protections différentes.

## 12.1 Droit d'auteur sur la structure

L'article L.112-3 protège les bases qui, **par le choix ou la disposition des matières**, constituent des créations intellectuelles.

Le droit porte alors sur la structure originale, pas automatiquement sur chaque donnée brute.

## 12.2 Droit sui generis du producteur

Un producteur peut bénéficier d'un droit sui generis lorsqu'il démontre un investissement substantiel dans :

- constitution ;
- vérification ;
- présentation du contenu.

Ce droit vise notamment certaines extractions ou réutilisations substantielles.

## 12.3 Pourquoi cette distinction compte en data/IA

Une base peut contenir :

- des données non protégées ;
- des œuvres protégées ;
- une structure originale ;
- un investissement protégé par le droit sui generis ;
- des données personnelles soumises au [[Règlement Général sur la Protection des Données (RGPD)|RGPD]].

Dire « les données sont publiques » ne répond donc pas à toutes les questions juridiques.

---

# 13. Licences de logiciels : principes

Une licence de logiciel est une autorisation juridique fondée sur les droits exclusifs du titulaire.

## 13.1 Propriétaire, source disponible, open source et logiciel libre

Ces notions ne sont pas synonymes.

| Situation | Code accessible ? | Redistribution ? | Modification ? |
|---|---:|---:|---:|
| propriétaire classique | parfois non | selon contrat | selon contrat |
| source-available | oui | variable | variable |
| open source OSI | oui | oui selon licence | oui selon licence |
| logiciel libre FSF | oui | oui | oui |

Une licence « source available » peut interdire :

- usage commercial ;
- concurrence ;
- hébergement SaaS ;
- certains secteurs d'activité.

Elle n'est alors pas nécessairement open source au sens de l'Open Source Definition.

## 13.2 Les quatre libertés du logiciel libre

La Free Software Foundation présente classiquement :

- liberté d'exécuter le programme ;
- liberté d'étudier son fonctionnement ;
- liberté de redistribuer des copies ;
- liberté de distribuer des versions modifiées.

L'accès au code source est nécessaire pour certaines de ces libertés.

## 13.3 Une licence libre n'abandonne pas le droit d'auteur

Le mécanisme est précisément l'inverse :

```text
Droit d'auteur
    ↓
Le titulaire dispose de droits exclusifs
    ↓
Il accorde une licence
    ↓
La licence autorise copie / modification / redistribution
sous certaines conditions
```

Sans droit d'auteur, le copyleft n'aurait pas le même support juridique.

---

# 14. Licences permissives

Les licences permissives imposent relativement peu d'obligations lors de la réutilisation.

## 14.1 MIT

La licence MIT autorise largement :

- utilisation ;
- copie ;
- modification ;
- fusion ;
- publication ;
- distribution ;
- sous-licence ;
- vente de copies.

L'obligation essentielle est de conserver la notice de copyright et le texte de licence dans les copies ou parties substantielles.

## 14.2 BSD 2-Clause

Obligations typiques :

- conserver la notice de copyright ;
- conserver les conditions ;
- conserver la clause de non-garantie.

## 14.3 BSD 3-Clause

Elle ajoute une clause empêchant l'utilisation du nom des contributeurs pour promouvoir un produit dérivé sans permission spécifique.

Dire que BSD permet une réutilisation « sans aucune restriction » est donc trop large.

## 14.4 Apache License 2.0

Apache-2.0 est permissive mais comporte des mécanismes plus détaillés que MIT/BSD, notamment :

- obligations de notices ;
- traitement du fichier `NOTICE` lorsqu'il existe ;
- licence explicite de brevets ;
- mécanisme de terminaison lié aux actions en brevet.

Cette licence est souvent intéressante pour des projets d'entreprise exposés à des questions de brevets.

## 14.5 Permissif ne veut pas dire domaine public

Un logiciel MIT ou BSD reste protégé par droit d'auteur.

La licence autorise la réutilisation **sous conditions**.

---

# 15. Copyleft : GPL, LGPL, AGPL, MPL et CeCILL

## 15.1 Principe du copyleft

Le copyleft accorde des libertés de réutilisation tout en exigeant, dans certaines situations de redistribution, que les œuvres dérivées ou combinaisons concernées restent sous des conditions compatibles.

## 15.2 GPL

La GNU GPL est un copyleft fort.

Lorsqu'une œuvre dérivée ou un programme combiné relevant de la GPL est **distribué**, les obligations de la GPL peuvent imposer notamment :

- licence GPL du programme concerné ;
- mise à disposition du code source correspondant ;
- conservation des notices ;
- transmission de la licence ;
- absence de restrictions supplémentaires incompatibles.

## 15.3 Erreur historique : GPLv3 et SaaS

L'ancien cours affirmait :

> « La GPLv3 a corrigé la faille SaaS et impose le code source aux utilisateurs réseau. »

C'est **faux**.

La GPLv3 ordinaire ne déclenche pas une obligation générale de fournir le source uniquement parce qu'un programme modifié est utilisé derrière un service web sans distribution de copie.

C'est précisément l'une des raisons d'être de la **GNU AGPLv3**.

## 15.4 AGPLv3

L'AGPLv3 ajoute une obligation pour les versions modifiées utilisées de manière à permettre à des utilisateurs d'interagir à distance avec le logiciel par un réseau : ces utilisateurs doivent pouvoir obtenir le **source correspondant** dans les conditions de la licence.

Schéma :

```text
GPL
redistribution d'une copie
        ↓
obligations copyleft

AGPL
redistribution
OU interaction réseau avec version modifiée
        ↓
obligations supplémentaires d'accès au source
```

## 15.5 Liaison dynamique : pas de « faille automatique »

L'ancien cours affirmait qu'il suffisait de transformer du code GPL en bibliothèque dynamique pour empêcher la propagation du copyleft.

Cette formulation est fausse.

La Free Software Foundation considère que la liaison statique **ou dynamique** peut former un programme combiné relevant de la GPL.

L'analyse dépend toutefois de la relation réelle entre les composants et les notions d'œuvre dérivée/combinaison applicables.

Il n'existe donc pas de règle :

> « `.so` = pas de GPL ».

## 15.6 LGPL

La LGPL vise notamment les bibliothèques et permet plus facilement leur utilisation par des logiciels sous d'autres licences, tout en protégeant la liberté de la bibliothèque elle-même.

Il faut respecter les conditions permettant notamment à l'utilisateur de remplacer ou modifier la bibliothèque selon le mode de liaison.

## 15.7 MPL 2.0

La Mozilla Public License 2.0 est souvent qualifiée de **copyleft au niveau du fichier**.

Elle peut constituer un compromis lorsque l'on souhaite :

- conserver ouverts les fichiers MPL modifiés ;
- permettre leur intégration dans un ensemble plus vaste comportant du code sous une autre licence.

## 15.8 CeCILL

CeCILL est une famille de licences libres élaborée dans le contexte juridique français, notamment par le CEA, le CNRS et Inria.

La licence CeCILL principale a été pensée avec une compatibilité avec la GPL dans les conditions qu'elle prévoit.

Il faut néanmoins vérifier la **version exacte** de la licence et la compatibilité concrète avant toute combinaison.

---

# 16. Compatibilité des licences et dépendances

Une licence ne se choisit pas uniquement en fonction de préférences philosophiques.

Elle doit être compatible avec les composants réellement utilisés.

## 16.1 Dépendance ≠ copie automatique

Utiliser une dépendance ne signifie pas toujours que son code est copié dans votre dépôt.

Mais le mode de combinaison peut avoir un impact juridique :

- liaison ;
- bundling ;
- vendoring ;
- génération de code ;
- inclusion de ressources ;
- modification directe ;
- communication par processus séparés.

## 16.2 Matrice simplifiée

| Licence de dépendance | Redistribution propriétaire souvent possible ? | Copyleft |
|---|---:|---|
| MIT | oui | non |
| BSD-2/3 | oui | non |
| Apache-2.0 | oui | non |
| MPL-2.0 | oui, sous conditions | fichier |
| LGPL | oui, sous conditions | bibliothèque |
| GPL | souvent incompatible avec distribution propriétaire d'un programme combiné | fort |
| AGPL | idem + enjeu interaction réseau | fort réseau |

Cette table est pédagogique, pas une analyse juridique suffisante pour tous les cas.

## 16.3 Compatibilité GPL

Une licence peut être :

- libre ;
- open source ;
- mais **incompatible avec une version précise de la GPL**.

Il faut donc vérifier :

- GPLv2-only ou GPLv2-or-later ;
- GPLv3 ;
- clauses de brevets ;
- restrictions supplémentaires ;
- version de chaque licence.

## 16.4 `only` et `or-later`

Ces expressions changent la compatibilité future.

Exemples SPDX :

```text
GPL-2.0-only
GPL-2.0-or-later
GPL-3.0-only
GPL-3.0-or-later
```

Ce ne sont pas des formulations interchangeables.

---

# 17. Creative Commons, documentation et contenus

## 17.1 Creative Commons n'est généralement pas recommandé pour le code

Creative Commons recommande explicitement de **ne pas utiliser les licences CC comme licences de logiciels**.

Pourquoi ?

- elles ne traitent pas le code source comme les licences logicielles ;
- elles n'abordent pas les brevets de la même manière ;
- elles peuvent créer des incompatibilités inutiles.

Pour du logiciel, préférer une licence conçue pour le logiciel :

- MIT ;
- BSD ;
- Apache-2.0 ;
- MPL-2.0 ;
- GPL/LGPL/AGPL ;
- CeCILL ;
- etc.

## 17.2 Creative Commons pour la documentation

CC est en revanche souvent adapté à :

- documentation ;
- cours ;
- images ;
- musique ;
- vidéos ;
- articles ;
- ressources pédagogiques.

## 17.3 Les briques des licences CC

- `BY` : attribution ;
- `SA` : partage dans les mêmes conditions ;
- `NC` : pas d'utilisation commerciale ;
- `ND` : pas de partage de versions adaptées.

Exemples :

```text
CC BY 4.0
CC BY-SA 4.0
CC BY-NC 4.0
CC BY-NC-SA 4.0
CC BY-ND 4.0
CC BY-NC-ND 4.0
```

## 17.4 « NC » et « ND » ne sont pas open source

Une restriction d'usage commercial ou de modification est incompatible avec les libertés attendues d'une licence open source logicielle.

## 17.5 CC0

CC0 cherche à permettre une renonciation aussi large que possible aux droits et prévoit un mécanisme de repli lorsque la renonciation complète n'est pas possible.

Il faut distinguer :

- **CC0** ;
- la **Public Domain Mark**, qui sert à signaler une œuvre déjà dans le domaine public.

---

# 18. Conformité open source dans un projet logiciel

La conformité ne consiste pas seulement à mettre un fichier `LICENSE` à la racine.

## 18.1 Inventaire

Il faut connaître :

- dépendances directes ;
- dépendances transitives ;
- code copié ;
- assets ;
- modèles ;
- polices ;
- datasets ;
- code généré ;
- conteneurs de base.

## 18.2 SPDX

SPDX fournit des identifiants standardisés.

Exemples :

```text
MIT
Apache-2.0
BSD-3-Clause
MPL-2.0
GPL-3.0-only
AGPL-3.0-or-later
```

Un en-tête peut utiliser :

```text
SPDX-License-Identifier: MIT
```

## 18.3 `LICENSE`, `NOTICE`, `COPYING`

Le nom exact dépend des pratiques du projet et de la licence.

Une distribution peut nécessiter de conserver :

- texte de licence ;
- notices de copyright ;
- fichier `NOTICE` ;
- offre ou code source correspondant ;
- attribution de ressources tierces.

## 18.4 SBOM

Une SBOM aide à connaître les composants logiciels mais ne résout pas seule la conformité juridique.

Elle peut inclure :

- nom du composant ;
- version ;
- licence déclarée ;
- provenance ;
- hash ;
- relation de dépendance.

Voir également le cours [[Docker]] pour les SBOM et attestations OCI.

## 18.5 Politique d'entreprise

Une organisation peut définir :

```text
Autorisé sans revue : MIT, BSD, Apache-2.0
Revue requise : MPL, LGPL
Approbation juridique : GPL, AGPL, licences atypiques
Interdit : licence inconnue ou sans texte clair
```

Il s'agit d'un exemple de politique interne, pas d'une classification juridique universelle.

---

# 19. Git, contributions et chaîne de droits

Git est un outil de preuve et de traçabilité très utile, mais ce n'est pas un registre officiel des droits.

## 19.1 Ce que Git peut montrer

Git peut aider à établir :

- date approximative d'une contribution ;
- identité déclarée du committer/auteur ;
- contenu d'un changement ;
- historique de modifications ;
- provenance technique.

## 19.2 Ce que Git ne prouve pas automatiquement

Git ne prouve pas à lui seul :

- l'identité civile réelle ;
- que le contributeur détenait les droits ;
- l'absence de copie d'un tiers ;
- l'existence d'une cession ;
- la validité d'une licence ;
- le statut salarié/prestataire ;
- l'originalité juridique.

## 19.3 Developer Certificate of Origin

Le **DCO** est un mécanisme de déclaration par lequel le contributeur atteste notamment disposer du droit de soumettre sa contribution selon le projet.

On le rencontre avec :

```bash
git commit -s
```

qui ajoute un `Signed-off-by`.

Le DCO n'est pas une cession générale de droits d'auteur.

## 19.4 CLA

Un **Contributor License Agreement** peut donner au projet des droits supplémentaires sur les contributions.

Il peut prendre la forme :

- d'une licence accordée au mainteneur ;
- parfois d'une cession, selon le texte.

Il faut lire le document : « CLA » ne décrit pas un contenu juridique unique.

## 19.5 Voir aussi

Le cours [[git]] détaille :

- signatures de commits ;
- Git LFS ;
- Git Xet ;
- historique ;
- traçabilité des modifications.

---

# 20. API, SaaS et distribution

## 20.1 API publique ≠ code open source

Exposer une API ne signifie pas publier le logiciel serveur.

Un client reçoit généralement :

```text
requête
→ service
→ réponse
```

et non une copie du programme serveur.

## 20.2 GPL et SaaS

Faire fonctionner une version modifiée d'un logiciel GPL sur un serveur sans distribuer de copie ne déclenche pas, à lui seul, l'obligation générale de fournir le source aux utilisateurs du service.

## 20.3 AGPL et interaction réseau

L'AGPL a été créée précisément pour traiter une partie de cette situation.

Pour une version modifiée couverte par l'AGPL et utilisée pour fournir une interaction distante, il faut étudier l'obligation d'offrir le source correspondant aux utilisateurs concernés.

## 20.4 Microservices

Un découpage en microservices ne garantit pas automatiquement l'indépendance juridique des composants.

Questions utiles :

- les processus sont-ils réellement séparés ?
- quel protocole les relie ?
- quel degré de couplage ?
- y a-t-il échange de structures internes spécifiques ?
- un composant est-il distribué ?
- un SDK GPL/AGPL est-il incorporé ?

Le choix d'architecture ne doit pas être conçu uniquement pour « contourner une licence ».

---

# 21. Intelligence artificielle et statut des sorties

Les modèles génératifs rendent nécessaire de séparer plusieurs objets juridiques :

```text
données d'entraînement
        ↓
modèle / poids
        ↓
prompt + contexte
        ↓
sortie générée
        ↓
édition humaine éventuelle
```

Chaque étage peut soulever des questions différentes.

## 21.1 Une IA n'est pas un auteur humain

Dans le cadre européen actuel, la protection par droit d'auteur repose sur une contribution intellectuelle humaine suffisante et sur des choix créatifs de l'auteur.

Une sortie générée **entièrement de manière automatique**, sans contribution créative humaine suffisante, peut donc ne pas bénéficier du droit d'auteur.

## 21.2 IA assistée

À l'inverse, une œuvre réalisée avec une IA peut rester protégeable si l'humain exerce une contribution créative réelle.

Exemples de facteurs potentiellement pertinents :

- conception créative préalable ;
- choix du matériau ;
- sélection entre de nombreuses sorties ;
- composition ;
- retouches importantes ;
- réécriture ;
- agencement ;
- décisions artistiques libres.

La réponse reste **cas par cas**.

## 21.3 Le prompt seul ne garantit rien

Un prompt long n'est pas automatiquement la preuve que la sortie est une œuvre humaine originale.

Il faut analyser :

- ce qui relève réellement de choix humains ;
- la relation entre ces choix et la forme finale ;
- le degré d'autonomie du système ;
- les transformations après génération.

## 21.4 Conditions d'utilisation du fournisseur

Même lorsqu'une sortie est susceptible d'être protégée, il faut vérifier les conditions contractuelles de l'outil :

- droits accordés sur les sorties ;
- garanties ou absence de garantie ;
- politique de réutilisation des prompts ;
- confidentialité ;
- restrictions d'usage ;
- clauses d'indemnisation.

Le contrat ne peut cependant pas inventer un droit d'auteur là où les conditions légales de protection ne sont pas réunies.

## 21.5 Risque de reproduction

Une sortie générative peut parfois reproduire ou approcher fortement :

- un passage de texte ;
- du code ;
- une image ;
- un personnage ;
- une musique ;
- une marque.

L'utilisateur doit donc éviter la règle naïve :

> « L'IA l'a généré, donc personne n'a de droits dessus. »

---

# 22. Entraînement des IA et fouille de textes et de données

Le droit européen comporte des exceptions de **text and data mining (TDM)** transposées en droit français.

## 22.1 Définition

L'article L.122-5-3 définit la fouille de textes et de données comme une technique d'analyse automatisée de textes et données numériques permettant d'en dégager des informations telles que :

- constantes ;
- tendances ;
- corrélations.

## 22.2 Recherche scientifique

Le régime français autorise, dans certaines conditions, des copies d'œuvres accessibles licitement pour des fouilles réalisées à des fins de recherche scientifique par des organismes et institutions déterminés.

Ce régime comporte notamment :

- accès licite ;
- catégories précises de bénéficiaires ;
- sécurité du stockage ;
- conservation possible pour vérification scientifique dans les conditions légales.

## 22.3 Exception générale de TDM

L'article L.122-5-3 III prévoit aussi une exception pouvant bénéficier à **toute personne et quelle que soit la finalité** lorsque :

- l'accès aux œuvres est licite ;
- le titulaire ne s'est pas opposé de manière appropriée ;
- les autres conditions sont respectées.

## 22.4 Réservation des droits / opt-out

Pour les contenus mis en ligne, l'opposition peut notamment être exprimée par des moyens **lisibles par machine**, par des métadonnées ou par les conditions générales d'un site ou service selon les règles françaises applicables.

Il faut donc concevoir les pipelines de collecte avec :

- conservation de la provenance ;
- détection des réservations de droits ;
- journalisation des décisions ;
- politique de suppression ;
- contrôle de l'accès licite.

## 22.5 Copies temporaires et destruction

Pour le régime général, les copies de TDM sont soumises à des exigences de sécurité et doivent être détruites à l'issue de la fouille dans les conditions prévues par la loi.

## 22.6 TDM ≠ autorisation universelle de tout entraînement

L'existence de l'exception ne répond pas automatiquement à toutes les questions :

- l'accès était-il licite ?
- une réservation de droits existe-t-elle ?
- quel droit national s'applique ?
- la base de données bénéficie-t-elle d'un droit sui generis ?
- le dataset contient-il des données personnelles ?
- des secrets ou obligations contractuelles existent-ils ?
- le modèle peut-il mémoriser/restituer des éléments protégés ?

Voir aussi [[LLM]] et [[Règlement Général sur la Protection des Données (RGPD)]].

---

# 23. AI Act et droit d'auteur

L'AI Act ne remplace pas le droit d'auteur. Il ajoute certaines obligations aux fournisseurs de modèles d'IA à usage général.

## 23.1 Politique de conformité copyright

L'article 53 de l'AI Act exige notamment des fournisseurs concernés qu'ils mettent en place une politique de conformité avec le droit de l'Union relatif au droit d'auteur et aux droits voisins.

Cette politique doit notamment permettre d'identifier et respecter les réservations de droits formulées dans le cadre du régime TDM.

## 23.2 Résumé public du contenu d'entraînement

Les fournisseurs de modèles d'IA à usage général doivent publier un **résumé suffisamment détaillé du contenu utilisé pour l'entraînement**, selon le modèle fourni par la Commission européenne.

Le modèle de résumé couvre notamment :

- informations générales sur le modèle ;
- types de contenus ;
- grandes catégories et volumes ;
- sources de données ;
- datasets publics ou privés ;
- contenus collectés sur le Web ;
- données utilisateur ;
- données synthétiques ;
- certaines informations relatives au traitement des données.

## 23.3 Calendrier en 2026

L'obligation de résumé pour les nouveaux modèles concernés s'applique depuis le **2 août 2025**.

Au 29 août 2026, l'AI Office peut engager l'application des obligations concernant les nouveaux modèles selon le calendrier de l'AI Act.

Pour certains modèles déjà mis sur le marché avant le 2 août 2025, un calendrier transitoire allant jusqu'au **2 août 2027** existe.

## 23.4 Code de bonnes pratiques GPAI

Le Code de bonnes pratiques GPAI publié en 2025 contient notamment des chapitres :

- Transparence ;
- Droit d'auteur ;
- Sûreté et sécurité pour les modèles à risque systémique.

Il s'agit d'un outil volontaire permettant d'aider les fournisseurs à démontrer leur conformité.

---

# 24. Preuve, dépôt et traçabilité

Le droit naît sans dépôt, mais la preuve est essentielle en cas de litige.

## 24.1 Ce qu'il faut pouvoir démontrer

Selon la situation :

- date de création ;
- versions successives ;
- auteur ;
- contenu exact ;
- contrats ;
- licences entrantes ;
- provenance des composants ;
- date de publication ;
- cessions de droits.

## 24.2 Moyens techniques

Exemples :

- historique Git signé ;
- archives horodatées ;
- signatures cryptographiques ;
- journaux de CI ;
- publication datée ;
- dépôt auprès d'un tiers ;
- constat ;
- enveloppe ou service de preuve adapté.

## 24.3 e-Soleau

L'INPI propose notamment le service **e-Soleau** permettant de constituer une preuve de date et de contenu.

Un tel dépôt :

- ne crée pas le droit d'auteur ;
- n'établit pas automatiquement l'originalité ;
- n'empêche pas une contestation ;
- mais peut constituer un élément probatoire très utile.

## 24.4 Preuve d'une chaîne open source

Pour un logiciel complexe, conserver :

```text
source du composant
version
hash
licence
URL d'origine
date d'intégration
modifications internes
notice requise
```

Cette discipline facilite :

- audit ;
- due diligence ;
- fusion/acquisition ;
- publication open source ;
- réponse à une réclamation.

---

# 25. Contrefaçon et réactions possibles

## 25.1 Contrefaçon

La contrefaçon peut résulter d'une exploitation réalisée sans autorisation alors qu'aucune exception ne s'applique.

Dans le logiciel :

- copie de code ;
- redistribution non conforme à une licence ;
- modification et distribution en violation des conditions ;
- réutilisation d'assets protégés ;

peuvent soulever des questions de contrefaçon.

## 25.2 Violation d'une licence libre

Une licence libre n'est pas une absence de règles.

Exemple :

```text
Code GPL redistribué
+
aucun accès au source correspondant
+
notices supprimées
=
risque de non-conformité à la licence
```

## 25.3 Avant d'accuser

Vérifier :

1. titularité ;
2. originalité de ce qui est invoqué ;
3. correspondance entre les œuvres ;
4. licence existante ;
5. exception éventuelle ;
6. loi applicable ;
7. preuve de l'usage litigieux.

## 25.4 Réactions graduées

Selon le cas :

- conserver les preuves ;
- contacter la partie ;
- demander attribution ou mise en conformité ;
- utiliser une procédure de notification de plateforme ;
- négocier une licence ;
- mise en demeure ;
- action en justice.

Les mécanismes de plateforme ne remplacent pas l'analyse juridique.

---

# 26. Chronologie et calcul du domaine public

## 26.1 Repères historiques utiles

Cette chronologie est volontairement simplifiée :

| Date | Repère |
|---|---|
| 1791 | premières grandes lois révolutionnaires sur les auteurs dramatiques |
| 1793 | reconnaissance élargie des droits des auteurs |
| 1866 | durée post mortem portée à 50 ans en France |
| 1957 | grande loi française sur la propriété littéraire et artistique |
| 1985 | réforme importante, logiciels et droits voisins, durée spécifique accrue pour certaines œuvres musicales |
| 1992 | création du Code de la propriété intellectuelle |
| 1993 | directive européenne d'harmonisation des durées |
| 1995/1997 | mise en œuvre française du principe général de 70 ans post mortem |
| 1998 | protection juridique des bases de données transposée en droit français |
| 2001 | directive européenne « société de l'information » |
| 2019 | directive DSM, dont exceptions de fouille de textes et de données |
| 2021 | transposition française des nouvelles exceptions TDM |
| 2024 | adoption de l'AI Act |
| 2025 | application de certaines obligations GPAI, dont transparence et copyright |
| 2026 | contrôle renforcé du respect de ces obligations selon le calendrier de l'AI Act |

## 26.2 Seuil général en 2026

En appliquant la règle ordinaire de 70 ans post mortem :

| Décès | Situation générale au 29 août 2026 |
|---|---|
| 1955 ou avant | domaine public depuis le 1er janvier 2026 au plus tard |
| 1956 | protégé jusqu'au 31 décembre 2026 ; entrée générale le 1er janvier 2027 |
| 1960 | entrée générale le 1er janvier 2031 |
| 1973 | entrée générale le 1er janvier 2044 |

La BnF utilise en 2026 le seuil pratique **date de mort antérieure à 1956** pour ses requêtes générales de domaine public.

## 26.3 Attention aux exceptions

Avant de conclure, vérifier :

- « Mort pour la France » ;
- droits acquis historiques ;
- œuvre collective ;
- anonymat/pseudonyme ;
- œuvre posthume ;
- œuvre de collaboration ;
- traduction ou adaptation récente ;
- droits voisins ;
- pays d'exploitation.

## 26.4 Apollinaire

Guillaume Apollinaire est mort pour la France en 1918.

Ses œuvres sont entrées dans le domaine public en France en **2013** après application du régime particulier qui le concernait.

## 26.5 Saint-Exupéry

*Le Petit Prince* reste protégé en France jusqu'en **2033** en raison de l'articulation particulière des droits acquis, de la publication pendant la Seconde Guerre mondiale et du statut « Mort pour la France ».

Ce cas ne doit pas être transformé en formule générale applicable à tous les auteurs morts pour la France.

---

# 27. Études de cas

## 27.1 Je copie 20 lignes d'une bibliothèque MIT

À faire :

1. vérifier la licence de la version exacte ;
2. déterminer si la portion est substantielle ;
3. conserver la notice requise ;
4. documenter la provenance ;
5. vérifier les autres dépendances.

## 27.2 Je modifie un serveur AGPL sans distribuer de binaire

Le simple raisonnement « je ne distribue rien donc aucune obligation » est insuffisant.

Il faut vérifier l'article relatif à l'interaction à distance et offrir le source correspondant dans les conditions prévues si le cas entre dans son champ.

## 27.3 Je modifie un serveur GPLv3 sans distribuer de copie

La GPLv3 ordinaire ne contient pas l'obligation réseau générale de l'AGPL.

Cela ne veut pas dire que toute utilisation est libre de toute contrainte : d'autres actes peuvent constituer une distribution, et des contrats ou composants distincts peuvent s'appliquer.

## 27.4 Je développe pour un client en freelance

Ne pas supposer que « facture payée = droits transférés ».

Le contrat devrait définir :

- composants antérieurs ;
- cession/licence du nouveau code ;
- périmètre ;
- territoire ;
- durée ;
- droits de modification ;
- dépendances open source ;
- documentation ;
- réutilisation générique du savoir-faire.

## 27.5 Je développe un logiciel dans le cadre de mon emploi

Si le logiciel est créé dans l'exercice des fonctions ou sur instruction de l'employeur, L.113-9 dévolue en principe les droits patrimoniaux à l'employeur.

Le contrat, les fonctions réelles et les circonstances restent importants.

## 27.6 Je récupère une photo Google Images

Google Images est un moteur de recherche, pas une banque de contenus libres.

Il faut identifier :

- auteur ;
- source originale ;
- licence ;
- éventuels droits de personnes représentées ;
- conditions de réutilisation.

## 27.7 Je génère une image avec une IA

Questions :

1. quelle contribution humaine existe ?
2. les CGU permettent-elles l'usage prévu ?
3. la sortie reproduit-elle une œuvre existante ?
4. contient-elle une marque ou une personne identifiable ?
5. ai-je conservé une trace du processus créatif ?

## 27.8 J'entraîne un modèle sur le Web

Questions supplémentaires :

- accès licite ;
- robots/réservations de droits lisibles par machine ;
- CGU ;
- provenance ;
- TDM ;
- droit sui generis des bases ;
- données personnelles ;
- secrets ;
- politique de retrait ;
- obligation de résumé GPAI si applicable.

## 27.9 Je mets une œuvre du domaine public sur un site

Vérifier :

- que l'œuvre elle-même est bien dans le domaine public ;
- que l'édition numérisée n'ajoute pas d'éléments distincts protégés ;
- respect du droit moral ;
- métadonnées et conditions du fournisseur du fichier ;
- marque éventuelle.

---

# 28. Checklist de décision

## 28.1 Avant de réutiliser un contenu

- [ ] Quelle est la nature du contenu ?
- [ ] Qui l'a créé ?
- [ ] Est-il original ?
- [ ] Qui détient les droits patrimoniaux ?
- [ ] Est-il encore protégé ?
- [ ] Une licence est-elle clairement indiquée ?
- [ ] La licence couvre-t-elle mon usage ?
- [ ] Dois-je attribuer ?
- [ ] Dois-je republier le source ?
- [ ] Dois-je conserver un `NOTICE` ?
- [ ] Une exception légale s'applique-t-elle ?
- [ ] Existe-t-il des droits voisins ?
- [ ] Existe-t-il un droit de base de données ?
- [ ] Une marque intervient-elle ?
- [ ] Des données personnelles sont-elles présentes ?
- [ ] Ai-je conservé une preuve de la provenance ?

## 28.2 Avant de publier un logiciel libre

- [ ] J'ai l'autorité pour licencier le code.
- [ ] Les contributeurs ont une chaîne de droits claire.
- [ ] Les dépendances sont compatibles.
- [ ] Le fichier `LICENSE` correspond à la licence annoncée.
- [ ] Les notices tierces sont conservées.
- [ ] Les ressources non logicielles ont leur propre licence si nécessaire.
- [ ] Les fichiers générés sont identifiés.
- [ ] Les secrets ont été supprimés de l'historique Git.
- [ ] Les brevets éventuels ont été analysés.
- [ ] Le README indique clairement la licence.
- [ ] Les identifiants SPDX sont cohérents.

## 28.3 Avant de publier un dataset

- [ ] Origine des données documentée.
- [ ] Licence des sources vérifiée.
- [ ] Droit sui generis vérifié.
- [ ] Données personnelles vérifiées.
- [ ] Opt-out TDM pris en compte si pertinent.
- [ ] Restrictions contractuelles analysées.
- [ ] Documentation de dataset produite.

## 28.4 Avant d'entraîner un modèle d'IA

- [ ] Sources inventoriées.
- [ ] Accès licite documenté.
- [ ] Réservations de droits détectées.
- [ ] Politique TDM documentée.
- [ ] Données personnelles traitées conformément au RGPD.
- [ ] Suppression/retrait possible autant que nécessaire.
- [ ] Datasets et versions tracés.
- [ ] Licences des modèles de départ vérifiées.
- [ ] Obligations AI Act identifiées.
- [ ] Résumé d'entraînement préparé si applicable.

---

# 29. Travaux pratiques

## TP 1 — Identifier les régimes applicables

Pour chacun des éléments d'une application, indiquer les protections possibles :

- logo ;
- code ;
- base d'utilisateurs ;
- nom du service ;
- documentation ;
- modèle ML ;
- dataset ;
- algorithme secret.

**Objectif :** ne plus réduire la propriété intellectuelle au seul droit d'auteur.

## TP 2 — Originalité ou fonctionnalité ?

Classer :

- algorithme de tri ;
- code concret du tri ;
- protocole HTTP ;
- article expliquant HTTP ;
- nom de variable `user_id` ;
- illustration originale d'une architecture réseau.

Justifier chaque réponse.

## TP 3 — Calcul simple du domaine public

Calculer la date d'entrée théorique dans le domaine public pour un auteur mort :

- en 1940 ;
- en 1955 ;
- en 1956 ;
- en 1980.

Puis expliquer pourquoi cette méthode est insuffisante pour Saint-Exupéry.

## TP 4 — Audit d'une licence MIT

Choisir un projet MIT et relever :

- titulaire ;
- année ;
- texte de licence ;
- obligations de redistribution ;
- dépendances sous une autre licence.

## TP 5 — MIT vs Apache-2.0

Comparer :

- longueur ;
- attribution ;
- brevets ;
- modifications ;
- notices ;
- intégration propriétaire.

## TP 6 — GPL vs AGPL

Étudier deux scénarios :

1. logiciel distribué aux clients ;
2. logiciel modifié uniquement exécuté comme service web.

Expliquer les différences de déclenchement des obligations.

## TP 7 — Inventaire SPDX

Créer un fichier :

```text
THIRD_PARTY_LICENSES.md
```

avec :

```text
Nom
Version
SPDX
URL
Copyright
Mode d'utilisation
Obligations
```

pour cinq dépendances d'un projet réel.

## TP 8 — Cession d'un logiciel freelance

Écrire une checklist contractuelle couvrant :

- droits cédés ;
- destination ;
- territoire ;
- durée ;
- composants antérieurs ;
- open source ;
- maintenance ;
- garantie de provenance.

Ne pas rédiger un contrat juridique définitif : identifier les clauses nécessaires.

## TP 9 — TDM et dataset Web

Construire une fiche de provenance pour un corpus :

```yaml
source: https://example.org
access_licite: true
licence: null
reservation_tdm: true
mecanisme_reservation: metadata
personal_data: possible
collected_at: 2026-08-29
```

Puis déterminer si la source doit être intégrée ou exclue.

## TP 10 — IA et processus créatif

Produire une illustration assistée par IA et documenter :

- prompts ;
- références ;
- variantes ;
- choix humains ;
- retouches ;
- éléments repris ;
- licence/CGU de l'outil.

L'objectif est de distinguer génération automatique et contribution créative humaine.

## TP 11 — Audit d'un dépôt Git

À partir de `git log`, identifier :

- principaux contributeurs ;
- fichiers tiers ;
- licences ;
- commits importés ;
- vendor directories ;
- fichiers sans provenance claire.

Puis expliquer pourquoi Git ne suffit pas à prouver juridiquement tous les droits.

## TP 12 — Mise en conformité d'une publication open source

Préparer une release avec :

- `LICENSE` ;
- `README.md` ;
- `NOTICE` si nécessaire ;
- `THIRD_PARTY_LICENSES.md` ;
- headers SPDX ;
- SBOM ;
- archive du source correspondant si nécessaire ;
- notes de provenance.

---

# 30. Projet final — Audit juridique d'un produit logiciel et IA

## 30.1 Contexte

Une entreprise souhaite publier un service comprenant :

- backend Python ;
- frontend JavaScript ;
- base PostgreSQL ;
- modèle de langage fine-tuné ;
- corpus Web ;
- bibliothèque GPL ;
- bibliothèque Apache-2.0 ;
- images Creative Commons ;
- documentation interne ;
- service SaaS public.

## 30.2 Livrables

Produire :

1. cartographie des actifs ;
2. chaîne de titulaires ;
3. matrice des licences ;
4. inventaire SPDX ;
5. analyse GPL/AGPL ;
6. analyse du dataset ;
7. analyse TDM ;
8. analyse du modèle et de ses poids ;
9. analyse des sorties générées ;
10. checklist AI Act GPAI si applicable ;
11. recommandations de mise en conformité ;
12. registre des preuves.

## 30.3 Exemple de matrice

| Composant | Origine | Licence/droit | Distribution ? | Action |
|---|---|---|---:|---|
| backend | interne | à choisir | oui | définir licence |
| lib A | PyPI | Apache-2.0 | oui | conserver licence/NOTICE |
| lib B | GitHub | GPL-3.0 | oui | analyser combinaison |
| corpus | Web | mixte | non | analyser TDM/licences |
| modèle | Hub | licence modèle | API | lire licence exacte |
| image | CC BY | CC BY 4.0 | oui | attribution |

## 30.4 Critères de réussite

Le projet est réussi si l'étudiant sait expliquer pourquoi :

- « disponible en ligne » n'est pas une licence ;
- « open source » n'est pas une absence de droit d'auteur ;
- GPL et AGPL ne sont pas équivalentes en SaaS ;
- Git ne transfère pas des droits ;
- une sortie IA n'est pas automatiquement protégée ni automatiquement libre ;
- TDM, RGPD, AI Act et droit d'auteur sont des couches différentes ;
- domaine public et droit moral doivent être distingués.

---

# 31. Références

## Sources juridiques françaises

- Code de la propriété intellectuelle : <https://www.legifrance.gouv.fr/codes/texte_lc/LEGITEXT000006069414/>
- Article L.121-1 — droit moral : <https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000006278891/>
- Article L.121-7 — droit moral du logiciel : <https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000006278899/>
- Article L.122-5 — exceptions : <https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000048603495/>
- Article L.122-5-3 — fouille de textes et de données : <https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000044363192/>
- Article L.113-9 — logiciel salarié : <https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000039279818/>
- Article L.123-1 — durée générale : <https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000006278917/>
- Articles L.123-8 à L.123-10 — prorogations historiques : <https://www.legifrance.gouv.fr/codes/section_lc/LEGITEXT000006069414/LEGISCTA000006161638/>
- Article L.131-1 — œuvres futures : <https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000006278955/>
- Article L.131-3 — cession : <https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000006278958/>
- Article L.611-2 — durée du brevet : <https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000041573176/>
- Article L.611-10 — brevetabilité : <https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000019298703/>

## Organismes publics

- INPI — droit d'auteur : <https://www.inpi.fr/ressources/propriete-intellectuelle/droit-dauteur>
- INPI — protection des logiciels : <https://www.inpi.fr/realiser-demarches/propriete-intellectuelle/cas-particulier-logiciels>
- Ministère de la Culture — droit d'auteur et domaine public : <https://www.culture.gouv.fr/>
- BnF — données du domaine public : <https://api.bnf.fr/>

## Union européenne

- Directive (UE) 2019/790 sur le droit d'auteur dans le marché unique numérique : <https://eur-lex.europa.eu/eli/dir/2019/790/oj>
- AI Act — règlement (UE) 2024/1689 : <https://eur-lex.europa.eu/eli/reg/2024/1689/oj>
- Commission européenne — modèle de résumé du contenu d'entraînement GPAI : <https://digital-strategy.ec.europa.eu/en/faqs/template-general-purpose-ai-model-providers-summarise-their-training-content>
- Commission européenne — Code de bonnes pratiques GPAI : <https://digital-strategy.ec.europa.eu/en/policies/contents-code-gpai>

## Logiciel libre et licences

- SPDX License List : <https://spdx.org/licenses/>
- Open Source Initiative : <https://opensource.org/licenses>
- GNU licenses : <https://www.gnu.org/licenses/>
- FAQ GPL : <https://www.gnu.org/licenses/gpl-faq.html>
- GNU AGPL : <https://www.gnu.org/licenses/agpl-3.0.html>
- Pourquoi l'AGPL : <https://www.gnu.org/licenses/why-affero-gpl.html>
- CeCILL : <https://cecill.info/>
- Creative Commons FAQ : <https://creativecommons.org/faq/>

---

# À retenir

```text
1. Le droit d'auteur protège une forme originale, pas une idée.
2. En France, la protection naît sans dépôt.
3. Auteur et titulaire des droits patrimoniaux ne sont pas toujours la même personne.
4. Le logiciel salarié dispose d'un régime spécial.
5. Le brevet logiciel « en tant que tel » est exclu, mais une invention technique mise en œuvre par ordinateur peut être brevetable.
6. Une licence libre utilise le droit d'auteur ; elle ne le supprime pas.
7. GPLv3 ne contient pas l'obligation réseau générale de l'AGPL.
8. La liaison dynamique ne constitue pas une échappatoire automatique à la GPL.
9. Creative Commons déconseille ses licences pour le logiciel lui-même.
10. Une œuvre du domaine public peut être entourée d'éléments nouveaux encore protégés.
11. En 2026, la règle générale du domaine public français couvre les auteurs morts avant 1956, sous réserve des cas spéciaux.
12. Pour l'IA, il faut séparer sorties, données d'entraînement, TDM, RGPD et obligations de l'AI Act.
```
