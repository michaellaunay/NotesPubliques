---
schema_version: 1
uid: "01M02EX5C70HV6DMYWV6S30YEB"
titre: "Règlement Général sur la Protection des Données (RGPD)"
aliases:
  - "RGPD"
  - "GDPR"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - droit
  - donnees-personnelles
  - rgpd
  - conformite
  - vie-privee
resume: "Cours complet et actualisé sur le RGPD : champ d'application, données personnelles, bases légales, droits, accountability, sous-traitance, sécurité, violations, AIPD, transferts internationaux, cookies, IA et mise en conformité opérationnelle."
niveau: intermediaire
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-06-21
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---

# Règlement Général sur la Protection des Données — RGPD

> [!important]
> Ce cours présente le RGPD dans une perspective pédagogique et opérationnelle, avec un accent sur la France et l'Union européenne. Il ne constitue pas un avis juridique personnalisé. Pour un traitement à fort risque, un contentieux ou une situation sectorielle réglementée, il faut vérifier les textes applicables et, si nécessaire, consulter un DPO ou un juriste spécialisé.

Le **Règlement général sur la protection des données** est le règlement (UE) 2016/679 du 27 avril 2016. Il est applicable depuis le **25 mai 2018**.

Le RGPD ne se résume ni au consentement, ni aux bandeaux cookies, ni à la cybersécurité. Il organise un système complet de gouvernance des traitements de données personnelles fondé sur trois idées :

1. un traitement doit avoir une **finalité et une base juridique** ;
2. les personnes disposent de **droits effectifs** ;
3. les organisations doivent être capables de **démontrer** leur conformité.

Cette dernière idée est appelée **accountability** ou responsabilisation.

---

# Sommaire

1. Fondements, histoire et champ d'application
2. Données personnelles, anonymisation et pseudonymisation
3. Acteurs et responsabilités
4. Principes fondamentaux du RGPD
5. Bases légales des traitements
6. Données sensibles et catégories particulières
7. Transparence et information des personnes
8. Droits des personnes
9. Profilage et décisions automatisées
10. Accountability, registre et privacy by design
11. Sous-traitants, contrats et fournisseurs
12. Sécurité et violations de données
13. AIPD / DPIA et traitements à risque élevé
14. Durées de conservation et cycle de vie
15. Transferts internationaux
16. Cookies, traceurs et ePrivacy
17. RGPD et développement logiciel
18. Cloud, SaaS et services externes
19. Intelligence artificielle, modèles et AI Act
20. Relations de travail, surveillance et vidéosurveillance
21. Sanctions, contrôles et contentieux
22. Construire un programme de conformité
23. Cas pratiques
24. Travaux pratiques
25. Projet final
26. Checklist de conformité
27. Glossaire
28. Références officielles

---

# 1. Fondements, histoire et champ d'application

## 1.1 De la directive 95/46/CE au RGPD

Avant le RGPD, la protection des données personnelles dans l'Union européenne reposait notamment sur la **directive 95/46/CE**. Une directive doit être transposée dans le droit national, ce qui avait conduit à des divergences entre États membres.

Le RGPD est un **règlement** : il est directement applicable dans l'Union européenne et l'Espace économique européen, tout en laissant certaines marges de manœuvre aux législations nationales.

En France, il s'articule notamment avec la **loi Informatique et Libertés** du 6 janvier 1978 modifiée.

## 1.2 Le RGPD protège des personnes, pas des données abstraites

L'objet du RGPD est la protection des **personnes physiques** à l'égard du traitement de leurs données personnelles.

Il ne protège pas en tant que telles :

- les données relatives à une personne morale ;
- une statistique véritablement anonyme ;
- une information qui ne permet raisonnablement plus d'identifier une personne.

En revanche, une donnée banale peut devenir personnelle lorsqu'elle est combinée à d'autres informations.

## 1.3 Champ matériel

Le RGPD s'applique aux traitements de données personnelles :

- automatisés ;
- ou non automatisés lorsque les données sont contenues ou destinées à être contenues dans un fichier structuré.

Le terme **traitement** est très large. Il comprend notamment :

- collecter ;
- enregistrer ;
- organiser ;
- consulter ;
- modifier ;
- rapprocher ;
- transmettre ;
- mettre à disposition ;
- archiver ;
- effacer ;
- détruire.

Une simple consultation de données personnelles peut donc constituer un traitement.

## 1.4 Champ territorial

Le RGPD s'applique notamment :

1. aux traitements effectués dans le cadre des activités d'un établissement situé dans l'Union ;
2. à certains traitements réalisés par un acteur non établi dans l'Union lorsqu'ils concernent des **personnes qui se trouvent dans l'Union** et sont liés :
   - à l'offre de biens ou de services à ces personnes ;
   - ou au suivi de leur comportement dans l'Union.

> [!warning]
> Dire que le RGPD s'applique à « toutes les données des résidents européens » est une simplification trompeuse. L'article 3 raisonne notamment à partir de l'établissement de l'organisme et, dans certains cas extraterritoriaux, des **personnes concernées se trouvant dans l'Union**.

## 1.5 Exclusions importantes

Le RGPD ne s'applique pas de la même manière à tous les traitements. Parmi les exclusions ou régimes spécifiques figurent notamment :

- les activités strictement personnelles ou domestiques ;
- certains traitements liés à la sécurité nationale ;
- les traitements des autorités compétentes à des fins pénales, régis notamment par la directive « Police-Justice » 2016/680.

---

# 2. Données personnelles, anonymisation et pseudonymisation

## 2.1 Qu'est-ce qu'une donnée personnelle ?

Une **donnée à caractère personnel** est une information se rapportant à une personne physique identifiée ou identifiable.

Exemples évidents :

- nom et prénom ;
- adresse postale ;
- adresse électronique nominative ;
- numéro de téléphone ;
- photographie d'une personne ;
- numéro de sécurité sociale.

Exemples parfois moins évidents :

- identifiant client ;
- identifiant publicitaire ;
- adresse IP ;
- identifiant de cookie ;
- identifiant de terminal ;
- données de localisation ;
- historique de navigation ;
- combinaison de caractéristiques permettant de distinguer une personne.

L'identification peut être **directe** ou **indirecte**.

## 2.2 Identifiable ne signifie pas « nom connu »

Une personne peut être identifiable même si son nom n'apparaît jamais dans la base.

Exemple :

```text
utilisateur = 4f93c7
ville = Lille
âge = 47
profession = chirurgien pédiatrique
```

Si le croisement des informations permet raisonnablement de retrouver la personne, les données peuvent rester personnelles.

## 2.3 Pseudonymisation

La **pseudonymisation** remplace ou sépare les identifiants directs, par exemple :

```text
Jean Dupont -> patient_008471
```

La table permettant de retrouver Jean Dupont est stockée séparément.

La pseudonymisation est une mesure de sécurité et de minimisation très utile, mais :

> [!important]
> Les données pseudonymisées restent des **données personnelles** dès lors que la réidentification demeure raisonnablement possible à l'aide d'informations supplémentaires.

## 2.4 Anonymisation

Une donnée véritablement **anonyme** ne permet plus d'identifier une personne par des moyens raisonnablement susceptibles d'être utilisés.

L'anonymisation doit être évaluée dans son contexte :

- singularisation possible ?
- possibilité de relier plusieurs enregistrements ?
- possibilité d'inférer l'identité ou des attributs ?
- existence de bases externes permettant une réidentification ?

Supprimer simplement le nom et le prénom n'est généralement **pas** une anonymisation.

## 2.5 Chiffrement

Le chiffrement protège les données contre certains accès non autorisés, mais une donnée chiffrée n'est pas pour autant anonyme.

Si l'organisation peut la déchiffrer, elle reste généralement une donnée personnelle.

---

# 3. Acteurs et responsabilités

## 3.1 Personne concernée

La **personne concernée** est la personne physique à laquelle les données se rapportent.

Exemples :

- client ;
- salarié ;
- candidat ;
- utilisateur d'une application ;
- patient ;
- visiteur d'un site.

## 3.2 Responsable du traitement

Le **responsable du traitement** détermine les finalités et les moyens essentiels du traitement.

Questions à poser :

- Pourquoi traite-t-on ces données ?
- Qui décide de cette finalité ?
- Qui fixe les choix structurants du traitement ?

## 3.3 Sous-traitant

Le **sous-traitant** traite des données personnelles **pour le compte** du responsable du traitement.

Exemples possibles selon le contexte :

- hébergeur ;
- fournisseur SaaS ;
- prestataire de paie ;
- société d'envoi d'e-mails ;
- centre de support ;
- prestataire d'infogérance.

Un prestataire n'est pas automatiquement sous-traitant. S'il détermine lui-même ses propres finalités, il peut être responsable de traitement pour tout ou partie des opérations.

## 3.4 Responsable conjoint

Lorsque plusieurs acteurs déterminent **conjointement** les finalités et moyens essentiels d'un traitement, ils peuvent être responsables conjoints.

L'article 26 impose alors de répartir de manière transparente leurs obligations, sans empêcher la personne concernée d'exercer ses droits à leur égard lorsque le RGPD le permet.

## 3.5 Délégué à la protection des données — DPO

Le DPO conseille, contrôle et accompagne la conformité.

Sa désignation est notamment obligatoire lorsque :

- le traitement est effectué par une autorité ou un organisme public, sauf certaines juridictions ;
- les activités de base impliquent un suivi régulier et systématique à grande échelle ;
- les activités de base impliquent à grande échelle des catégories particulières de données ou certaines données pénales.

Le DPO doit bénéficier :

- d'une indépendance suffisante ;
- de ressources ;
- d'un accès aux informations nécessaires ;
- d'une absence de conflit d'intérêts.

Le DPO n'est pas « responsable de la conformité à la place de l'entreprise » : la responsabilité juridique reste portée par les acteurs désignés par le règlement.

---

# 4. Principes fondamentaux du RGPD

L'article 5 constitue la colonne vertébrale du règlement.

## 4.1 Licéité, loyauté et transparence

Un traitement doit :

- reposer sur une base juridique ;
- ne pas tromper ou surprendre injustement les personnes ;
- être expliqué de manière accessible.

## 4.2 Limitation des finalités

Les données doivent être collectées pour des finalités :

- déterminées ;
- explicites ;
- légitimes.

Une donnée collectée pour une finalité ne peut pas être librement réutilisée pour n'importe quel autre objectif.

## 4.3 Minimisation

Il faut limiter les données à ce qui est **adéquat, pertinent et nécessaire**.

Exemple : pour envoyer une newsletter, demander la date de naissance complète, le numéro de téléphone et la profession est généralement difficile à justifier si ces informations ne servent pas la finalité.

## 4.4 Exactitude

Les données doivent être exactes et, si nécessaire, tenues à jour.

Une donnée incorrecte peut produire des décisions incorrectes, parfois avec de fortes conséquences pour la personne.

## 4.5 Limitation de la conservation

Les données ne doivent pas être conservées sous une forme identifiante plus longtemps que nécessaire.

Il faut donc définir :

- une durée ;
- ou des critères permettant de déterminer cette durée ;
- puis automatiser autant que possible l'archivage, la purge ou l'anonymisation.

## 4.6 Intégrité et confidentialité

Le traitement doit être protégé contre :

- l'accès non autorisé ;
- la divulgation ;
- la modification ;
- la perte ;
- la destruction ;
- l'indisponibilité accidentelle.

La sécurité concerne donc la **confidentialité, l'intégrité et la disponibilité**.

## 4.7 Accountability

Le responsable du traitement doit respecter les principes **et pouvoir démontrer qu'il les respecte**.

Cette exigence transforme la conformité en démarche documentaire et continue :

```text
identifier -> décider -> documenter -> mettre en œuvre -> contrôler -> améliorer
```

---

# 5. Bases légales des traitements

## 5.1 Une base légale par finalité

Chaque finalité de traitement doit reposer sur une base légale appropriée.

L'article 6 prévoit six bases principales :

1. consentement ;
2. contrat ou mesures précontractuelles ;
3. obligation légale ;
4. sauvegarde des intérêts vitaux ;
5. mission d'intérêt public ou exercice de l'autorité publique ;
6. intérêt légitime.

> [!warning]
> Le **consentement n'est pas la base légale par défaut** du RGPD. Dans de nombreux traitements, une autre base est plus adaptée.

## 5.2 Consentement

Pour être valide, le consentement doit notamment être :

- libre ;
- spécifique ;
- éclairé ;
- univoque ;
- démontrable ;
- retirable aussi facilement qu'il a été donné.

Une case précochée ne constitue pas un consentement valable.

Un refus ne doit pas entraîner de conséquence injustifiée lorsque le traitement n'est pas nécessaire au service demandé.

## 5.3 Exécution d'un contrat

Cette base couvre les traitements **objectivement nécessaires** :

- à l'exécution d'un contrat auquel la personne est partie ;
- ou à des mesures précontractuelles prises à sa demande.

Le simple fait d'inscrire une opération dans les conditions générales ne la rend pas automatiquement « nécessaire au contrat ».

## 5.4 Obligation légale

Exemple : conservation de certaines pièces pour respecter une obligation comptable ou fiscale.

La base doit pouvoir être reliée à une obligation du droit de l'Union ou du droit national applicable.

## 5.5 Intérêts vitaux

Cette base vise notamment des situations où le traitement est nécessaire pour protéger la vie ou l'intégrité d'une personne lorsque d'autres bases ne sont pas adaptées.

## 5.6 Mission d'intérêt public

Elle concerne notamment les traitements nécessaires à l'exécution d'une mission d'intérêt public ou relevant de l'exercice de l'autorité publique, sur une base juridique adaptée.

## 5.7 Intérêt légitime

L'intérêt légitime nécessite une analyse structurée :

1. **finalité légitime** : quel intérêt réel poursuit l'organisation ou un tiers ?
2. **nécessité** : le traitement est-il réellement nécessaire à cet intérêt ?
3. **mise en balance** : les droits et libertés de la personne prévalent-ils ?

Il faut notamment examiner :

- les attentes raisonnables de la personne ;
- la nature des données ;
- le contexte de collecte ;
- l'impact potentiel ;
- les garanties mises en œuvre ;
- la possibilité de s'opposer.

L'intérêt légitime n'est pas une clause permettant de justifier a posteriori n'importe quel traitement.

## 5.8 Changer de base légale

La base légale doit être choisie **avant** le traitement. La remplacer opportunément après un problème de conformité est généralement une mauvaise pratique et peut rendre l'information fournie aux personnes incohérente.

---

# 6. Données sensibles et catégories particulières

## 6.1 Article 9

Certaines catégories de données bénéficient d'une protection renforcée :

- origine raciale ou ethnique ;
- opinions politiques ;
- convictions religieuses ou philosophiques ;
- appartenance syndicale ;
- données génétiques ;
- données biométriques traitées pour identifier une personne de manière unique ;
- données concernant la santé ;
- vie sexuelle ;
- orientation sexuelle.

Leur traitement est **en principe interdit**, sauf si une exception de l'article 9(2) s'applique.

> [!important]
> Une base de l'article 6 et une condition appropriée de l'article 9 peuvent être nécessaires ensemble.

## 6.2 Données biométriques

Toutes les photos ne sont pas automatiquement des « données biométriques sensibles ».

Elles le deviennent notamment lorsqu'elles résultent d'un traitement technique spécifique relatif aux caractéristiques physiques, physiologiques ou comportementales et sont utilisées pour identifier ou authentifier une personne de manière unique.

## 6.3 Données pénales

Les données relatives aux condamnations pénales et infractions relèvent de l'article 10 et de règles nationales spécifiques.

En France, leur traitement est fortement encadré par la loi Informatique et Libertés.

## 6.4 Mineurs

Pour les services de la société de l'information lorsque le traitement repose sur le consentement, le RGPD fixe en principe le seuil à 16 ans mais permet aux États membres de l'abaisser jusqu'à 13 ans.

En France, l'article 45 de la loi Informatique et Libertés fixe le seuil à **15 ans**. En dessous, le consentement doit, dans ce cadre, être recueilli conjointement auprès du mineur et du ou des titulaires de l'autorité parentale.

Les informations destinées aux mineurs doivent être particulièrement claires et compréhensibles.

---

# 7. Transparence et information des personnes

## 7.1 Articles 12 à 14

La personne doit savoir :

- qui traite ses données ;
- pour quelles finalités ;
- sur quelles bases légales ;
- quelles catégories de données sont concernées ;
- qui peut les recevoir ;
- combien de temps elles sont conservées ou selon quels critères ;
- quels droits elle possède ;
- comment contacter le DPO lorsqu'il existe ;
- s'il existe des transferts hors EEE et sur quel mécanisme ils reposent ;
- le cas échéant, l'existence d'une prise de décision automatisée et les informations requises par le RGPD.

## 7.2 Collecte directe et collecte indirecte

- **Article 13** : données collectées auprès de la personne.
- **Article 14** : données obtenues auprès d'une autre source.

La collecte indirecte exige notamment d'informer sur les catégories de données et leur source, sous réserve des exceptions prévues par le règlement.

## 7.3 Information en couches

Une bonne interface peut présenter :

### Premier niveau

- finalité essentielle ;
- responsable ;
- choix significatifs ;
- lien vers plus d'informations.

### Deuxième niveau

- politique complète avec tous les éléments nécessaires.

Cette approche évite de choisir entre une interface illisible et une information juridiquement insuffisante.

## 7.4 Éviter les dark patterns

Une interface de protection des données ne doit pas pousser abusivement l'utilisateur vers l'option la plus intrusive.

Exemples problématiques :

- bouton « accepter » énorme et refus presque invisible ;
- multiples écrans pour refuser ;
- vocabulaire culpabilisant ;
- cases ambiguës ;
- double négation ;
- impossibilité pratique de retirer le consentement.

---

# 8. Droits des personnes

## 8.1 Délai de réponse

Le responsable doit en principe répondre **dans un délai d'un mois** à compter de la réception de la demande.

Ce délai peut être prolongé de deux mois lorsque cela est nécessaire compte tenu de la complexité et du nombre de demandes, à condition d'en informer la personne dans le premier mois et de motiver la prolongation.

## 8.2 Vérification de l'identité

Il faut trouver un équilibre :

- ne pas divulguer les données à un imposteur ;
- ne pas exiger systématiquement une pièce d'identité lorsque ce n'est pas nécessaire.

Des informations supplémentaires ne doivent être demandées que lorsqu'il existe des doutes raisonnables sur l'identité.

## 8.3 Droit d'accès

La personne peut obtenir notamment :

- confirmation qu'un traitement existe ou non ;
- accès aux données ;
- copie des données ;
- informations complémentaires prévues par l'article 15.

L'accès ne doit pas porter atteinte aux droits et libertés d'autrui.

## 8.4 Rectification

Une personne peut demander la correction des données inexactes ou la complétion de données incomplètes.

## 8.5 Effacement

Le droit à l'effacement s'applique dans plusieurs situations, par exemple lorsque :

- les données ne sont plus nécessaires ;
- le consentement est retiré et aucune autre base ne subsiste ;
- la personne s'oppose avec succès au traitement ;
- le traitement est illicite.

Mais il existe des exceptions :

- liberté d'expression et d'information ;
- obligation légale ;
- mission d'intérêt public dans certains cas ;
- santé publique ;
- archives, recherche ou statistiques sous conditions ;
- constatation, exercice ou défense de droits en justice.

« Droit à l'oubli » ne signifie donc pas « droit à effacer toute information partout ».

## 8.6 Limitation du traitement

La limitation peut permettre de geler temporairement certains usages sans supprimer immédiatement les données.

## 8.7 Portabilité

La portabilité s'applique lorsque :

- le traitement est fondé sur le consentement ou le contrat ;
- le traitement est effectué à l'aide de procédés automatisés.

Elle porte en principe sur les données fournies par la personne, dans un format structuré, couramment utilisé et lisible par machine.

## 8.8 Opposition

La personne peut notamment s'opposer à certains traitements fondés sur :

- l'intérêt légitime ;
- une mission d'intérêt public.

Pour la **prospection commerciale directe**, l'opposition bénéficie d'un régime particulièrement fort : lorsqu'une personne s'y oppose, les données ne doivent plus être traitées à cette fin.

## 8.9 Ne pas confondre les droits

```text
accès       -> voir et obtenir ses données
rectification -> corriger
suppression -> effacer lorsque les conditions sont réunies
limitation  -> geler certains traitements
portabilité -> récupérer/transmettre certaines données
opposition  -> demander l'arrêt d'un traitement dans les cas prévus
```

---

# 9. Profilage et décisions automatisées

## 9.1 Profilage

Le profilage est une forme de traitement automatisé visant à évaluer certains aspects personnels, par exemple :

- performance au travail ;
- situation économique ;
- santé ;
- préférences ;
- fiabilité ;
- comportement ;
- localisation.

Le profilage n'est pas systématiquement interdit, mais il doit respecter toutes les exigences applicables du RGPD.

## 9.2 Article 22

L'article 22 protège la personne contre certaines décisions fondées **exclusivement** sur un traitement automatisé produisant des effets juridiques la concernant ou l'affectant de manière similaire de façon significative.

Des exceptions existent notamment lorsque la décision :

- est nécessaire à la conclusion ou à l'exécution d'un contrat ;
- est autorisée par le droit applicable avec garanties appropriées ;
- est fondée sur le consentement explicite.

Des garanties doivent alors être prévues lorsque le règlement l'exige, notamment la possibilité d'obtenir une intervention humaine et de contester la décision.

## 9.3 Exemple récent : décisions automatisées

En août 2026, l'autorité néerlandaise de protection des données, en coopération avec la CNIL, a annoncé une sanction de près de **825 millions d'euros** à l'encontre d'Uber au sujet de décisions individuelles automatisées concernant des chauffeurs.

L'intérêt pédagogique de cette affaire n'est pas seulement le montant : elle montre que l'article 22 et la gouvernance des décisions algorithmiques sont des sujets opérationnels majeurs.

## 9.4 Explicabilité

Il ne faut pas réduire la transparence à la publication du code source d'un algorithme.

Il faut notamment être capable de documenter selon le contexte :

- les données utilisées ;
- les grandes catégories de facteurs ;
- la logique générale du processus ;
- les conséquences possibles ;
- les mécanismes de recours ;
- les contrôles humains.

---

# 10. Accountability, registre et privacy by design

## 10.1 Le registre des traitements

L'article 30 prévoit un registre des activités de traitement dans les cas applicables.

Exemple de structure :

| Champ | Exemple |
|---|---|
| Traitement | Gestion des comptes clients |
| Finalité | Fournir le service |
| Base légale | Contrat |
| Personnes | Clients |
| Données | identité, contact, identifiant |
| Destinataires | support, hébergeur |
| Transferts | à documenter |
| Durée | durée du contrat + archivage justifié |
| Sécurité | MFA, chiffrement, contrôle d'accès |

## 10.2 Attention à l'exception des moins de 250 salariés

Le RGPD contient une exception limitée pour certains organismes employant moins de 250 personnes.

Elle ne signifie pas « PME = pas de registre ». L'exception ne s'applique notamment pas lorsque le traitement :

- est susceptible de comporter un risque pour les droits et libertés ;
- n'est pas occasionnel ;
- porte sur certaines données sensibles ou pénales.

En pratique, un registre constitue souvent un outil de gouvernance utile même lorsqu'une lecture stricte du texte pourrait permettre une exemption partielle.

## 10.3 Protection des données dès la conception

Le **privacy by design** consiste à intégrer la protection des données dès la conception :

```text
besoin -> architecture -> minimisation -> droits -> sécurité -> test -> exploitation
```

Pas :

```text
produit terminé -> ajouter une politique de confidentialité
```

## 10.4 Protection des données par défaut

Le **privacy by default** signifie que les paramètres par défaut doivent être les plus protecteurs compatibles avec la finalité.

Exemples :

- profil privé par défaut lorsque le service n'exige pas une publication publique ;
- collecte optionnelle désactivée ;
- durée de conservation limitée ;
- partage non nécessaire désactivé.

## 10.5 Preuves de conformité

Exemples de documentation :

- registre ;
- analyses de base légale ;
- tests de mise en balance ;
- AIPD ;
- contrats article 28 ;
- politiques de conservation ;
- procédures de droits ;
- registre des violations ;
- preuves de consentement ;
- audits de sécurité ;
- analyses de transferts.

---

# 11. Sous-traitants, contrats et fournisseurs

## 11.1 Choisir un sous-traitant

Un responsable du traitement ne doit pas sélectionner un fournisseur uniquement sur la base du prix ou de la qualité fonctionnelle.

Il faut examiner notamment :

- localisation des données ;
- sous-traitants ultérieurs ;
- mesures de sécurité ;
- capacité à assister sur les droits ;
- suppression/restitution des données ;
- transferts internationaux ;
- notification des incidents ;
- audits et preuves disponibles.

## 11.2 Contrat article 28

Le contrat doit notamment encadrer :

- objet et durée ;
- nature et finalité ;
- type de données ;
- catégories de personnes ;
- instructions documentées ;
- confidentialité ;
- sécurité ;
- sous-traitance ultérieure ;
- assistance sur les droits ;
- assistance en matière de sécurité/AIPD ;
- sort des données en fin de contrat ;
- informations permettant de démontrer la conformité et audits.

## 11.3 Sous-traitant ultérieur

Un fournisseur SaaS peut lui-même s'appuyer sur :

- un cloud public ;
- un service d'e-mail ;
- une solution de logs ;
- un prestataire de support.

La chaîne de sous-traitance doit être connue et encadrée.

## 11.4 Responsable ou sous-traitant : raisonner par opération

Une même entreprise peut être :

- sous-traitante pour une opération ;
- responsable de traitement pour une autre.

Exemple : un prestataire peut traiter les données pour fournir le service au client, tout en étant responsable de ses propres traitements de facturation, sécurité ou obligations légales selon le contexte.

---

# 12. Sécurité et violations de données

## 12.1 Article 32 : sécurité adaptée au risque

Le RGPD n'impose pas une liste figée de technologies. Il exige des mesures **appropriées au risque**.

Exemples :

- contrôle d'accès ;
- MFA ;
- chiffrement ;
- pseudonymisation ;
- segmentation ;
- journalisation ;
- sauvegardes ;
- tests de restauration ;
- gestion des correctifs ;
- revue des habilitations ;
- protection contre l'exfiltration ;
- tests et audits réguliers.

## 12.2 Violation de données personnelles

Une violation de données est une violation de sécurité entraînant, de manière accidentelle ou illicite :

- destruction ;
- perte ;
- altération ;
- divulgation non autorisée ;
- accès non autorisé.

Elle peut toucher :

- la confidentialité ;
- l'intégrité ;
- la disponibilité.

## 12.3 Les trois niveaux de réaction

### Aucun risque pour les personnes

- documenter dans le registre interne des violations ;
- pas nécessairement de notification à l'autorité.

### Risque pour les droits et libertés

- documenter ;
- notifier l'autorité de contrôle compétente dans les meilleurs délais et, si possible, **dans les 72 heures** après en avoir pris connaissance.

### Risque élevé

- documenter ;
- notifier l'autorité ;
- informer en principe les personnes concernées dans les meilleurs délais, sauf exception prévue par le règlement.

## 12.4 Sous-traitant et incident

Le sous-traitant doit notifier au responsable du traitement la violation **dans les meilleurs délais** après en avoir pris connaissance.

Le contrat doit prévoir un canal d'escalade suffisamment rapide pour permettre au responsable de respecter ses propres délais.

## 12.5 Notification progressive

Il ne faut pas attendre d'avoir terminé toute l'enquête si le délai de 72 heures expire.

Une notification initiale peut être complétée ultérieurement lorsque toutes les informations ne sont pas encore disponibles.

## 12.6 Retour d'expérience récent

La CNIL a sanctionné en janvier 2026 Free Mobile et Free à hauteur de **42 millions d'euros au total**, notamment pour des mesures de sécurité jugées inadaptées ; la décision concernant Free Mobile portait également sur des durées de conservation excessives.

Cet exemple rappelle que la sécurité et la conservation sont contrôlées comme des obligations concrètes, pas comme de simples principes déclaratifs.

---

# 13. AIPD / DPIA et traitements à risque élevé

L'**analyse d'impact relative à la protection des données** est appelée :

- AIPD en français ;
- DPIA en anglais, pour *Data Protection Impact Assessment*.

## 13.1 Quand est-elle requise ?

Une AIPD est obligatoire lorsqu'un type de traitement est susceptible d'engendrer un **risque élevé** pour les droits et libertés des personnes.

L'article 35 cite notamment certains cas :

- évaluation systématique et approfondie de personnes fondée sur un traitement automatisé produisant certains effets ;
- traitement à grande échelle de catégories particulières de données ou de données pénales ;
- surveillance systématique à grande échelle d'une zone accessible au public.

La CNIL publie également des listes de traitements pour lesquels une AIPD est requise ou non requise.

## 13.2 Contenu d'une AIPD

Une AIPD comprend notamment :

1. description du traitement ;
2. finalités ;
3. nécessité et proportionnalité ;
4. risques pour les personnes ;
5. mesures prévues pour réduire ces risques.

## 13.3 Risque résiduel élevé

Si un risque élevé subsiste malgré les mesures envisagées, une **consultation préalable** de l'autorité de contrôle peut être nécessaire avant la mise en œuvre du traitement.

## 13.4 L'AIPD est un processus vivant

Elle doit être réévaluée lorsque le risque change :

- nouvelle finalité ;
- nouveau jeu de données ;
- nouveau fournisseur ;
- nouvelle technologie ;
- changement d'échelle ;
- incident révélant une faiblesse.

---

# 14. Durées de conservation et cycle de vie

## 14.1 Pas de durée universelle

Le RGPD ne dit pas « conserver toutes les données pendant X ans ».

La durée doit être déterminée à partir :

- de la finalité ;
- d'obligations légales ;
- de délais de prescription ;
- de recommandations sectorielles ;
- des risques ;
- de la nécessité réelle.

## 14.2 Trois phases utiles

La CNIL distingue couramment :

1. **base active** : données nécessaires à l'activité courante ;
2. **archivage intermédiaire** : accès restreint pour une obligation ou un contentieux potentiel ;
3. **archivage définitif** : seulement lorsque sa justification juridique existe, notamment dans certains contextes archivistiques.

Toutes les données ne passent pas nécessairement par les trois phases.

## 14.3 Exemple

```text
compte actif
   |
   v
fin de relation commerciale
   |
   +--> suppression des données devenues inutiles
   |
   +--> archivage restreint des éléments nécessaires
            |
            v
       purge à échéance
```

## 14.4 Concevoir la purge

Une politique de conservation doit être traduite en mécanismes techniques :

- TTL ;
- jobs de purge ;
- partitions temporelles ;
- archivage chiffré ;
- règles de rétention des sauvegardes ;
- suppression des répliques ;
- traitement des logs ;
- processus couvrant les sous-traitants.

> [!warning]
> « On supprimera manuellement si un jour quelqu'un y pense » n'est pas une politique de conservation robuste.

---

# 15. Transferts internationaux

## 15.1 Qu'est-ce qu'un transfert ?

Un transfert international doit être analysé lorsqu'un exportateur soumis au RGPD met des données à disposition d'un destinataire situé dans un pays tiers ou une organisation internationale, selon les conditions du chapitre V.

Le simple lieu physique du serveur n'est pas le seul élément à examiner : l'accès distant et la chaîne de fournisseurs peuvent également compter.

## 15.2 Décision d'adéquation

La Commission européenne peut reconnaître qu'un pays, territoire, secteur ou organisation offre un niveau de protection adéquat.

Lorsqu'une décision d'adéquation couvre le transfert, il n'est pas nécessaire d'ajouter un mécanisme de l'article 46 pour ce transfert couvert.

La liste évolue. Elle doit donc être vérifiée au moment du projet.

Parmi les évolutions récentes figurent notamment :

- l'adéquation de l'Organisation européenne des brevets en 2025 ;
- le renouvellement de l'adéquation du Royaume-Uni en décembre 2025 ;
- une décision d'adéquation pour le Brésil en janvier 2026.

## 15.3 États-Unis : du Privacy Shield au Data Privacy Framework

### Ancien cadre

Le **Privacy Shield UE–États-Unis** a été invalidé par la CJUE dans l'arrêt **Schrems II** du 16 juillet 2020.

Il ne doit donc plus être présenté comme un mécanisme valide actuel.

### Cadre actuel

Le **10 juillet 2023**, la Commission européenne a adopté une décision d'adéquation relative au **EU-US Data Privacy Framework (DPF)**.

Les transferts vers les organisations américaines effectivement certifiées et couvertes peuvent bénéficier de cette décision dans les conditions applicables.

Il faut vérifier :

- que l'organisation est bien certifiée ;
- que la certification couvre les données et l'entité concernées ;
- que le statut est toujours valable.

La Commission a publié en octobre 2024 son premier examen périodique du fonctionnement de cette décision.

## 15.4 Clauses contractuelles types — SCC

Lorsque l'adéquation ne s'applique pas, les **clauses contractuelles types** de la Commission peuvent constituer une garantie appropriée.

Les SCC de 2021 utilisent plusieurs modules selon les rôles :

- responsable -> responsable ;
- responsable -> sous-traitant ;
- sous-traitant -> sous-traitant ;
- sous-traitant -> responsable.

## 15.5 Analyse du pays tiers et mesures supplémentaires

Depuis Schrems II, signer des SCC ne suffit pas toujours à clore l'analyse.

Il faut vérifier si les lois et pratiques du pays tiers permettent effectivement de respecter le niveau de protection requis et, lorsque nécessaire, envisager des mesures supplémentaires :

- chiffrement fort avec clés sous contrôle approprié ;
- pseudonymisation avant transfert ;
- séparation des données ;
- mesures organisationnelles et contractuelles.

Cette démarche est souvent appelée **Transfer Impact Assessment — TIA**.

## 15.6 BCR

Les **Binding Corporate Rules** peuvent encadrer certains transferts au sein d'un groupe international après approbation selon le mécanisme applicable.

## 15.7 Dérogations de l'article 49

Les dérogations ne doivent pas être transformées en mécanisme ordinaire pour des transferts massifs et répétitifs.

Elles couvrent des situations spécifiques, par exemple certains transferts occasionnels fondés sur un consentement explicite après information des risques, ou nécessaires à certains contrats, sous les conditions du texte.

---

# 16. Cookies, traceurs et ePrivacy

## 16.1 Cookies ≠ RGPD uniquement

En France, les règles sur la lecture et l'écriture d'informations dans le terminal proviennent notamment de l'**article 82 de la loi Informatique et Libertés**, qui transpose l'article 5(3) de la directive ePrivacy.

Le RGPD intervient notamment pour définir les exigences du consentement lorsque celui-ci est requis et pour les traitements de données personnelles qui en résultent.

## 16.2 Principe

Sauf exemption, un traceur soumis au consentement ne doit pas être déposé ou lu avant que l'utilisateur ait consenti.

Le consentement doit être :

- préalable ;
- libre ;
- éclairé ;
- spécifique ;
- univoque ;
- démontrable.

## 16.3 Refuser doit être aussi simple qu'accepter

Un bandeau du type :

```text
[ TOUT ACCEPTER ]

Pour refuser, ouvrez "paramètres", puis 4 sous-menus...
```

est difficilement compatible avec le principe d'un choix libre et simple.

Une interface plus équilibrée :

```text
[ TOUT REFUSER ]    [ PERSONNALISER ]    [ TOUT ACCEPTER ]
```

## 16.4 Traceurs exemptés

Certains traceurs strictement nécessaires peuvent être exemptés du consentement, par exemple dans certaines configurations :

- authentification ;
- panier d'achat ;
- équilibrage ou sécurité nécessaires ;
- certaines mesures d'audience répondant à des critères stricts.

L'exemption de certains traceurs de mesure d'audience n'est pas automatique : la CNIL impose des conditions précises, notamment limitation à la mesure d'audience pour le compte exclusif de l'éditeur et absence de suivi global inter-sites.

## 16.5 Consentement multi-terminaux

En janvier **2026**, la CNIL a publié des recommandations finales sur le **consentement multi-terminaux**.

L'idée est de permettre, dans certaines architectures liées à un compte utilisateur, de synchroniser un choix de consentement entre plusieurs terminaux tout en préservant :

- la compréhension du choix ;
- la liberté ;
- la preuve ;
- le retrait ;
- l'information sur la portée multi-terminaux.

Ce sujet illustre pourquoi une CMP (*Consent Management Platform*) n'est pas un simple widget graphique : elle fait partie de l'architecture de conformité.

## 16.6 Exemple de matrice

| Traceur | Finalité | Consentement ? | Durée | Fournisseur |
|---|---|---:|---|---|
| session | maintenir la session | selon nécessité/exemption | session | interne |
| panier | conserver le panier | selon nécessité/exemption | définie | interne |
| analytics | audience | selon configuration | définie | prestataire |
| publicité | ciblage | oui en principe | définie | régie |

---

# 17. RGPD et développement logiciel

Le RGPD doit être traduit en exigences techniques.

## 17.1 Modèle de données

Avant d'ajouter un champ :

```text
Quel besoin métier ?
Pourquoi cette donnée ?
Quelle base légale ?
Qui y accède ?
Combien de temps ?
Comment la supprimer ?
Comment la restituer ?
```

## 17.2 API de droits

Un système bien conçu permet de retrouver les données d'une personne à travers :

- base principale ;
- CRM ;
- stockage objet ;
- logs pertinents ;
- moteur de recherche ;
- outils marketing ;
- fournisseurs externes.

L'identité doit être résolue sans divulguer les données d'une autre personne.

## 17.3 Exemple conceptuel de politique de rétention

```yaml
customer_account:
  active: while_contract_active
  intermediate_archive: 5y_if_justified
  purge_job: monthly

support_ticket:
  active: 2y_after_closure
  purge_job: weekly
```

Les durées ne sont ici qu'illustratives : elles doivent être justifiées pour le traitement réel.

## 17.4 Logs

Les logs peuvent contenir :

- IP ;
- identifiants ;
- URL avec paramètres ;
- jetons ;
- contenu saisi ;
- événements métier.

Il faut éviter :

```text
Authorization: Bearer eyJ...
password=...
medical_note=...
```

Bonnes pratiques :

- masquer les secrets ;
- minimiser les payloads ;
- définir des durées ;
- limiter les accès ;
- séparer logs techniques et métier ;
- documenter les finalités.

## 17.5 Sauvegardes et droit à l'effacement

Le droit à l'effacement ne signifie pas nécessairement qu'un octet doit disparaître instantanément de toutes les sauvegardes immuables.

Il faut cependant :

- empêcher le retour des données supprimées dans le système actif ;
- fixer une rétention des sauvegardes ;
- limiter leur usage ;
- prévoir comment les suppressions sont réappliquées lors d'une restauration ;
- documenter l'architecture.

## 17.6 Données de test

Éviter de copier automatiquement une production contenant des données personnelles vers des environnements de développement.

Préférer :

- données synthétiques ;
- anonymisation robuste ;
- sous-ensemble minimisé et protégé lorsque cela est réellement nécessaire.

---

# 18. Cloud, SaaS et services externes

## 18.1 « Hébergé en Europe » ne suffit pas

Une analyse cloud doit examiner :

- entité contractante ;
- localisation des traitements ;
- accès support ;
- sous-traitants ;
- transferts ;
- clés de chiffrement ;
- télémétrie ;
- journaux ;
- sauvegardes ;
- mécanismes de sortie.

## 18.2 Questions pour un fournisseur

```text
Où sont les données ?
Qui peut y accéder ?
Quels sous-traitants utilisez-vous ?
Quels mécanismes de transfert ?
Comment gérez-vous les demandes de droits ?
Quel délai de notification d'incident ?
Comment supprimer toutes les données ?
Combien de temps les sauvegardes persistent-elles ?
Quelles certifications et quels rapports d'audit fournissez-vous ?
```

## 18.3 Chiffrement côté client

Le chiffrement côté client peut réduire les risques lorsque le fournisseur n'a pas accès aux clés.

Mais il ne résout pas automatiquement :

- les métadonnées ;
- les identifiants ;
- la télémétrie ;
- les transferts d'autres données ;
- les obligations de gouvernance.

## 18.4 Réversibilité

La protection des données implique aussi la maîtrise du cycle de vie :

- exporter ;
- migrer ;
- supprimer ;
- obtenir une attestation ;
- traiter les copies et sauvegardes.

---

# 19. Intelligence artificielle, modèles et AI Act

## 19.1 Le RGPD s'applique à l'IA lorsque des données personnelles sont traitées

Le fait qu'un système soit appelé « IA », « machine learning » ou « LLM » ne crée pas une exemption au RGPD.

Les traitements peuvent intervenir pendant :

- collecte du corpus ;
- préparation ;
- entraînement ;
- fine-tuning ;
- évaluation ;
- inférence ;
- journalisation ;
- amélioration continue.

## 19.2 Questions RGPD pour un projet IA

```text
Le corpus contient-il des données personnelles ?
Quelle est la finalité ?
Quelle base légale ?
Des données de l'article 9 sont-elles présentes ?
Les personnes sont-elles informées directement ou indirectement ?
Peut-on exercer les droits ?
Le modèle mémorise-t-il des données ?
Quels risques de réidentification ou extraction ?
Une AIPD est-elle nécessaire ?
Existe-t-il des transferts ?
```

## 19.3 Intérêt légitime et IA

La CNIL a publié en 2025 des recommandations détaillées sur le développement de systèmes d'IA. Elle souligne que l'intérêt légitime peut être pertinent dans certains projets privés, mais uniquement après une véritable analyse de nécessité, d'attentes raisonnables, d'impact et de garanties.

Il ne faut pas transformer :

```text
"nous voulons entraîner un modèle"
```

en :

```text
"donc intérêt légitime"
```

sans analyse.

## 19.4 Données publiques

Une donnée accessible publiquement reste potentiellement une donnée personnelle.

« C'était sur Internet » n'annule pas :

- la finalité ;
- la base légale ;
- l'information ;
- les droits ;
- la minimisation ;
- les obligations de sécurité.

## 19.5 Modèle et données personnelles

La question de savoir si un modèle entraîné constitue lui-même une donnée personnelle dépend du contexte et de la possibilité d'obtenir ou d'inférer des informations relatives à des personnes identifiables.

Il faut notamment examiner :

- mémorisation ;
- extraction ;
- membership inference ;
- inversion ;
- identifiants présents dans les sorties ;
- capacité réelle à isoler une personne.

## 19.6 RGPD et AI Act

Le **règlement européen sur l'IA — règlement (UE) 2024/1689** ne remplace pas le RGPD.

Les deux textes peuvent s'appliquer au même système.

Au 29 août 2026 :

- certaines dispositions de l'AI Act s'appliquent depuis le 2 février 2025 ;
- des dispositions relatives notamment aux modèles d'IA à usage général et à la gouvernance s'appliquent depuis le 2 août 2025 ;
- l'AI Act est applicable de manière générale depuis le **2 août 2026**, avec certaines obligations bénéficiant encore d'un calendrier spécifique, notamment celles visées à l'article 113.

Depuis le 2 août 2026, de nouvelles obligations de transparence prévues par le règlement commencent notamment à s'appliquer à certains systèmes d'IA.

## 19.7 Privacy engineering pour l'IA

Mesures possibles selon le risque :

- filtrage des corpus ;
- déduplication ;
- minimisation ;
- pseudonymisation ;
- differential privacy dans certains cas ;
- contrôle des accès ;
- limitation de la rétention des prompts ;
- redaction de données sensibles ;
- évaluation de mémorisation ;
- audit des sorties ;
- procédure de droits ;
- documentation de provenance.

---

# 20. Relations de travail, surveillance et vidéosurveillance

## 20.1 Salariés

Le lien de subordination rend le consentement souvent délicat comme base légale dans la relation de travail, car sa liberté réelle doit être évaluée.

Un traitement RH doit notamment respecter :

- nécessité ;
- proportionnalité ;
- information ;
- accès limité ;
- durée de conservation ;
- sécurité.

## 20.2 Surveillance

Une organisation ne peut pas conclure :

```text
ordinateur professionnel -> donc surveillance illimitée
```

Il faut évaluer la finalité, la proportionnalité et les droits fondamentaux.

La surveillance permanente et excessive de salariés peut constituer un manquement au principe de minimisation.

En juillet 2026, la CNIL indiquait que plusieurs sanctions simplifiées récentes concernaient notamment des dispositifs vidéo filmant en permanence des salariés.

## 20.3 Vidéosurveillance et vidéoprotection

Les termes et régimes peuvent varier selon :

- lieu accessible ou non au public ;
- finalité ;
- droit national applicable.

Il faut examiner à la fois :

- base légale ;
- information ;
- angles de caméra ;
- durée ;
- habilitations ;
- sécurité ;
- nécessité éventuelle d'une AIPD ;
- règles nationales spécifiques.

---

# 21. Sanctions, contrôles et contentieux

## 21.1 Pouvoirs des autorités

Une autorité de contrôle peut notamment :

- enquêter ;
- demander des informations ;
- réaliser des contrôles ;
- prononcer des injonctions ;
- limiter ou interdire des traitements ;
- prononcer des amendes administratives dans les limites prévues par le règlement.

## 21.2 Deux plafonds d'amendes

Selon les catégories de manquements, l'article 83 prévoit des plafonds pouvant atteindre :

### Niveau 1

- 10 millions d'euros ;
- ou, pour une entreprise, jusqu'à 2 % du chiffre d'affaires annuel mondial total de l'exercice précédent ;
- le montant le plus élevé étant retenu selon le texte.

### Niveau 2

- 20 millions d'euros ;
- ou jusqu'à 4 % du chiffre d'affaires annuel mondial total ;
- le montant le plus élevé étant retenu.

Une sanction n'est toutefois pas automatiquement égale au plafond.

## 21.3 Autres conséquences

La non-conformité peut également produire :

- injonction de supprimer des données ;
- arrêt d'un traitement ;
- contentieux ;
- demandes d'indemnisation ;
- perte de contrats ;
- coût d'incident ;
- perte de confiance ;
- obligation de reconstruire un système.

## 21.4 Sanctions : ne pas apprendre uniquement les montants

En 2025, la CNIL indique avoir prononcé **83 sanctions** pour un total d'environ **486,8 millions d'euros** d'amendes. Les thèmes récurrents comprenaient notamment les cookies, la surveillance des salariés et la sécurité des données.

En 2026, les décisions publiées montrent encore la diversité des manquements :

- sécurité ;
- conservation ;
- droits ;
- vidéosurveillance ;
- cookies ;
- AIPD ;
- relations responsable/sous-traitant ;
- décisions automatisées.

L'objectif d'un cours n'est donc pas de mémoriser une liste de records, mais d'identifier les obligations opérationnelles contrôlées.

---

# 22. Construire un programme de conformité

## 22.1 Étape 1 — Cartographier

Lister :

- applications ;
- bases ;
- flux ;
- fournisseurs ;
- données ;
- personnes concernées ;
- transferts ;
- finalités.

## 22.2 Étape 2 — Qualifier

Pour chaque traitement :

```text
finalité ?
responsable ?
base légale ?
données sensibles ?
destinataires ?
durée ?
transfert ?
AIPD ?
droits ?
mesures de sécurité ?
```

## 22.3 Étape 3 — Prioriser par risque

Exemples de priorité forte :

- données de santé ;
- biométrie ;
- données d'enfants ;
- surveillance ;
- profilage significatif ;
- grande échelle ;
- exposition Internet ;
- transferts complexes ;
- décision automatisée ;
- bases anciennes sans politique de purge.

## 22.4 Étape 4 — Corriger

Plan d'action possible :

```text
P0 : violation active ou exposition critique
P1 : traitement illicite / droits impossibles / sécurité majeure
P2 : documentation, contrats, durées, interfaces
P3 : optimisation continue
```

## 22.5 Étape 5 — Industrialiser

Automatiser :

- revues périodiques ;
- purges ;
- inventaire de fournisseurs ;
- détection d'incidents ;
- traitement des droits ;
- preuves de consentement ;
- contrôle de la configuration ;
- formation des équipes.

## 22.6 Privacy review dans le cycle produit

Exemple de jalons :

```text
idée
  -> qualification données
  -> revue architecture
  -> base légale
  -> AIPD si nécessaire
  -> sécurité
  -> textes d'information
  -> test des droits
  -> go-live
  -> revue périodique
```

---

# 23. Cas pratiques

## 23.1 Newsletter

Une entreprise souhaite envoyer une newsletter à des particuliers.

Questions :

- quelle règle de prospection électronique s'applique en plus du RGPD ?
- quel mécanisme permet de se désinscrire ?
- combien de temps conserver les informations ?
- comment documenter l'opposition ?

Le RGPD doit ici être articulé avec le droit des communications électroniques.

## 23.2 Application sportive

Une application collecte :

- identité ;
- fréquence cardiaque ;
- localisation ;
- sommeil.

Points d'analyse :

- certaines données concernent potentiellement la santé ;
- minimisation ;
- article 9 ;
- sécurité ;
- transfert vers le fournisseur cloud ;
- AIPD potentielle selon le traitement ;
- conservation ;
- droits.

## 23.3 SaaS B2B

Un client transmet sa base de salariés à un SaaS.

Le SaaS est-il sous-traitant ?

Souvent oui pour le service fourni selon les instructions du client, mais il faut analyser les opérations. Le SaaS peut être responsable de certains traitements distincts.

À examiner :

- DPA article 28 ;
- sous-traitants ;
- transferts ;
- sécurité ;
- rétention ;
- fin de contrat.

## 23.4 LLM externe

Une équipe copie des tickets clients dans un chatbot SaaS.

Questions immédiates :

- données personnelles dans les prompts ?
- secrets ?
- finalité ?
- rôle du fournisseur ?
- utilisation pour entraîner ses modèles ?
- rétention ?
- localisation ?
- transferts ?
- droits ?
- données sensibles ?

La première mesure peut être aussi simple que de ne pas envoyer les données inutiles.

## 23.5 Caméras dans un atelier

La direction souhaite filmer en continu tous les postes « pour la sécurité ».

Il faut distinguer :

- sécurité réelle ;
- surveillance de la performance ;
- nécessité ;
- zones filmées ;
- alternatives ;
- proportionnalité ;
- information ;
- durée ;
- AIPD éventuelle.

Une finalité vague ne justifie pas automatiquement une surveillance permanente.

---

# 24. Travaux pratiques

## TP 1 — Identifier les données personnelles

Classer :

```text
adresse IP
identifiant aléatoire stable
statistique agrégée
photo
numéro de série d'un équipement
coordonnées GPS
hash d'adresse e-mail
```

Pour chaque élément, expliquer **dans quel contexte** il peut ou non être personnel.

## TP 2 — Choisir une base légale

Pour cinq traitements, déterminer la base possible et justifier :

1. livraison d'une commande ;
2. newsletter optionnelle ;
3. conservation de factures ;
4. sécurité anti-fraude ;
5. traitement vital d'urgence.

## TP 3 — Test d'intérêt légitime

Rédiger :

```text
1. intérêt poursuivi
2. nécessité
3. attentes de la personne
4. impact
5. garanties
6. droit d'opposition
7. conclusion
```

## TP 4 — Registre

Construire un registre pour une petite application SaaS :

- création de compte ;
- facturation ;
- support ;
- analytics ;
- sécurité.

## TP 5 — Demande d'accès

Concevoir la procédure :

```text
réception
 -> identité
 -> périmètre
 -> collecte interne
 -> revue droits des tiers
 -> export
 -> réponse
 -> preuve de traitement
```

## TP 6 — Violation de données

Scénario : un bucket contenant 15 000 dossiers clients a été publiquement lisible pendant six heures.

Déterminer :

- nature de la violation ;
- données concernées ;
- risque ;
- actions techniques ;
- besoin de notification ;
- besoin d'informer les personnes ;
- documentation.

## TP 7 — AIPD

Réaliser le squelette d'une AIPD pour un système de contrôle d'accès biométrique.

## TP 8 — Transferts

Pour un fournisseur américain :

1. vérifier s'il est couvert par une décision d'adéquation applicable ;
2. sinon identifier le mécanisme de transfert ;
3. documenter la chaîne de sous-traitance ;
4. analyser les mesures supplémentaires nécessaires.

## TP 9 — Bandeau cookies

Analyser trois interfaces :

- accepter visible, refuser caché ;
- accepter/refuser symétriques ;
- aucune action mais cookies publicitaires déposés immédiatement.

Expliquer les problèmes.

## TP 10 — Revue RGPD d'une fonctionnalité IA

Fonctionnalité : résumé automatique des tickets de support avec un LLM externe.

Produire une fiche comprenant :

- finalité ;
- données ;
- base légale ;
- fournisseur ;
- transfert ;
- minimisation ;
- conservation ;
- sécurité ;
- information ;
- droits ;
- AIPD.

## TP 11 — Durées de conservation

Construire une matrice :

| Donnée | Base active | Archivage | Justification | Purge |
|---|---|---|---|---|
| compte | ... | ... | ... | ... |
| facture | ... | ... | ... | ... |
| logs | ... | ... | ... | ... |

## TP 12 — Audit d'un fournisseur SaaS

Préparer un questionnaire de 30 points couvrant :

- article 28 ;
- sécurité ;
- transferts ;
- rétention ;
- incidents ;
- droits ;
- sous-traitants ;
- fin de contrat.

---

# 25. Projet final — Mise en conformité d'un SaaS

## Contexte

Une entreprise fictive exploite **TaskFlow**, un SaaS de gestion de projets.

TaskFlow traite :

- comptes utilisateurs ;
- collaborateurs invités ;
- commentaires ;
- pièces jointes ;
- logs ;
- métriques d'utilisation ;
- tickets support ;
- facturation ;
- une fonction IA de résumé ;
- cookies analytics et marketing.

Fournisseurs :

- cloud européen ;
- support américain ;
- service e-mail ;
- LLM externe ;
- outil analytics.

## Livrables

### 1. Cartographie

Dessiner :

```text
navigateur
   |
   v
TaskFlow API ---> PostgreSQL
   |                |
   |                +--> backup
   |
   +--> e-mail
   +--> support
   +--> LLM
   +--> analytics
```

### 2. Registre

Créer au minimum les traitements :

- gestion du compte ;
- fourniture du service ;
- sécurité ;
- facturation ;
- support ;
- analytics ;
- marketing ;
- IA.

### 3. Bases légales

Justifier chaque finalité.

### 4. Sous-traitants

Créer une fiche par fournisseur :

```text
rôle
localisation
sous-traitants ultérieurs
transfert
sécurité
rétention
suppression
incident
```

### 5. Politique de conservation

Créer les règles et leur implémentation technique.

### 6. Gestion des droits

Décrire l'export et l'effacement d'un utilisateur.

### 7. AIPD

Décider si une AIPD est requise pour la fonctionnalité IA et documenter le raisonnement.

### 8. Incident

Écrire le playbook de violation de données avec chronologie des 72 premières heures.

### 9. Cookies

Produire une matrice des traceurs et une interface de choix conforme.

### 10. Rapport final

Classer les écarts :

```text
critique
élevé
moyen
faible
```

et produire un plan de remédiation.

---

# 26. Checklist de conformité

## Gouvernance

- [ ] responsables et sous-traitants identifiés ;
- [ ] DPO désigné lorsque nécessaire ;
- [ ] responsabilités internes définies ;
- [ ] registre à jour ;
- [ ] documentation centralisée.

## Finalités et bases légales

- [ ] finalités explicites ;
- [ ] base légale définie pour chaque finalité ;
- [ ] intérêt légitime documenté lorsque utilisé ;
- [ ] consentement démontrable lorsque nécessaire ;
- [ ] données sensibles encadrées.

## Minimisation et conservation

- [ ] champs réellement nécessaires ;
- [ ] politique de conservation ;
- [ ] purge automatique ;
- [ ] sauvegardes couvertes ;
- [ ] logs couverts.

## Transparence

- [ ] information article 13/14 ;
- [ ] langage clair ;
- [ ] politique réellement conforme au système ;
- [ ] fournisseurs/transferts correctement décrits ;
- [ ] changements de finalité gérés.

## Droits

- [ ] canal de demande ;
- [ ] procédure identité ;
- [ ] délai d'un mois suivi ;
- [ ] accès ;
- [ ] rectification ;
- [ ] effacement ;
- [ ] limitation ;
- [ ] opposition ;
- [ ] portabilité lorsque applicable ;
- [ ] décisions automatisées traitées.

## Fournisseurs

- [ ] contrat article 28 ;
- [ ] sous-traitants ultérieurs connus ;
- [ ] sécurité évaluée ;
- [ ] transferts documentés ;
- [ ] suppression en fin de contrat ;
- [ ] notification d'incident suffisamment rapide.

## Sécurité

- [ ] contrôle d'accès ;
- [ ] MFA lorsque pertinent ;
- [ ] chiffrement ;
- [ ] correctifs ;
- [ ] sauvegardes/restauration ;
- [ ] logs ;
- [ ] revue des habilitations ;
- [ ] tests de sécurité ;
- [ ] procédure d'incident ;
- [ ] registre des violations.

## Risques

- [ ] AIPD évaluée ;
- [ ] AIPD réalisée lorsque requise ;
- [ ] risques résiduels documentés ;
- [ ] changements réévalués.

## Transferts

- [ ] pays tiers identifiés ;
- [ ] adéquation vérifiée ;
- [ ] SCC/BCR/autre mécanisme si nécessaire ;
- [ ] TIA lorsque nécessaire ;
- [ ] mesures supplémentaires examinées.

## Web

- [ ] traceurs inventoriés ;
- [ ] aucun traceur soumis au consentement avant choix ;
- [ ] refus aussi simple que l'acceptation ;
- [ ] retrait accessible ;
- [ ] preuve du choix ;
- [ ] exemptions documentées.

## IA

- [ ] corpus qualifié ;
- [ ] données personnelles identifiées ;
- [ ] finalité/base légale ;
- [ ] droits ;
- [ ] risques de mémorisation ;
- [ ] fournisseur/transferts ;
- [ ] logs de prompts ;
- [ ] articulation AI Act ;
- [ ] AIPD évaluée.

---

# 27. Glossaire

**AIPD / DPIA**
Analyse d'impact relative à la protection des données.

**Anonymisation**
Transformation visant à rendre les personnes non identifiables de manière suffisamment robuste pour que les données sortent du champ des données personnelles.

**Autorité de contrôle**
Autorité publique indépendante chargée de surveiller l'application du droit de la protection des données. En France : CNIL.

**BCR**
*Binding Corporate Rules*, règles d'entreprise contraignantes permettant d'encadrer certains transferts internationaux au sein d'un groupe.

**Base légale**
Fondement juridique autorisant un traitement au regard de l'article 6.

**CNIL**
Commission nationale de l'informatique et des libertés.

**Consentement**
Une des bases légales de l'article 6 ; il doit remplir les conditions du RGPD lorsqu'il est utilisé.

**Donnée personnelle**
Information se rapportant à une personne physique identifiée ou identifiable.

**DPO**
Délégué à la protection des données.

**DPF**
EU-US Data Privacy Framework, cadre faisant l'objet d'une décision d'adéquation adoptée en 2023 pour les organisations américaines certifiées couvertes.

**EEE**
Espace économique européen.

**Pseudonymisation**
Traitement qui empêche d'attribuer les données à une personne sans informations supplémentaires conservées séparément ; les données restent personnelles.

**Responsable du traitement**
Acteur déterminant les finalités et les moyens essentiels du traitement.

**SCC / CCT**
Standard Contractual Clauses / clauses contractuelles types.

**Sous-traitant**
Acteur traitant les données pour le compte du responsable du traitement.

**TIA**
*Transfer Impact Assessment*, analyse du contexte juridique et pratique d'un transfert vers un pays tiers.

**Traitement**
Toute opération ou ensemble d'opérations portant sur des données personnelles.

---

# 28. Références officielles

## Textes

- Règlement (UE) 2016/679 — RGPD : <https://eur-lex.europa.eu/eli/reg/2016/679/oj/fra>
- Loi Informatique et Libertés : <https://www.cnil.fr/fr/le-cadre-national/la-loi-informatique-et-libertes>
- Règlement (UE) 2024/1689 — AI Act : <https://eur-lex.europa.eu/eli/reg/2024/1689/oj/fra>

## CNIL

- Site de la CNIL : <https://www.cnil.fr/>
- Principes et bases légales : <https://www.cnil.fr/fr/les-bases-legales>
- Consentement : <https://www.cnil.fr/fr/les-bases-legales/consentement>
- Violations de données : <https://www.cnil.fr/fr/violations-de-donnees-personnelles-les-regles-suivre>
- AIPD : <https://www.cnil.fr/fr/PIA-privacy-impact-assessment>
- Listes AIPD : <https://www.cnil.fr/fr/listes-des-traitements-pour-lesquels-une-aipd-est-requise-ou-non>
- Durées de conservation : <https://www.cnil.fr/fr/passer-laction/les-durees-de-conservation-des-donnees>
- Cookies et traceurs : <https://www.cnil.fr/fr/cookies-et-autres-traceurs/regles>
- Recommandations IA : <https://www.cnil.fr/fr/developpement-des-systemes-dia-les-recommandations-de-la-cnil-pour-respecter-le-rgpd>
- Sanctions : <https://www.cnil.fr/fr/les-sanctions-prononcees-par-la-cnil>

## EDPB / CEPD

- European Data Protection Board : <https://www.edpb.europa.eu/>
- Mesures supplémentaires pour les transferts : <https://www.edpb.europa.eu/documents/recommendation/recommendations-012020-on-measures-that-supplement-transfer-tools-to_en>

## Commission européenne

- Décisions d'adéquation : <https://commission.europa.eu/law/law-topic/data-protection/international-dimension-data-protection/adequacy-decisions_en>

---

# Conclusion

Le RGPD ne doit pas être traité comme une série de formulaires à remplir après le développement d'un produit.

La démarche correcte est plutôt :

```text
finalité
  -> nécessité
  -> base légale
  -> minimisation
  -> architecture
  -> sécurité
  -> transparence
  -> droits
  -> conservation
  -> fournisseurs et transferts
  -> documentation
  -> contrôle continu
```

Une organisation mature n'essaie pas seulement de démontrer qu'elle possède une politique de confidentialité. Elle doit être capable de démontrer que **le système réel**, ses données, son code, ses fournisseurs et ses processus correspondent effectivement à ce qui est déclaré.
