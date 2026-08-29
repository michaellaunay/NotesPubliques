---
schema_version: 1
uid: "01M05DHAX93FNQE9SWSM4H9VB8"
titre: "Index cours"
type: index
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - cours
resume: "Index thématique généré des notes du dossier cours."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2026-08-16
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: false
metadata_verifiees: false
---

# cours

Index régénéré par `make index` — ne pas éditer à la main. Vue graphique : [[_INDEX_cours.excalidraw]].

## Intelligence artificielle

- [[Du PDF scanné au corpus exploitable OCR multimodal local avec olmOCR 2 et Infinity-Parser2-Pro]] — Cours-atelier sur la construction d'une chaîne locale de numérisation : comparaison d'olmOCR 2 et d'Infinity-Parser2-Pro, architecture du pipeline, installation, backends d'inférence, ingestion des PDF et production d'un corpus exploitable.
- [[Hermes Agent]] — Cours approfondi sur Hermes Agent, agent IA persistant et extensible : mémoire persistante, recherche dans les conversations, création de skills, tâches planifiées, sous-agents isolés et compatibilité multi-fournisseurs.
- [[Les CNN et RNN]] — Cours de niveau master sur les réseaux convolutifs et récurrents : principe de la convolution, cartes d'activation, partage des poids, dimensions et stride, puis traitement des séquences.
- [[Les RAGs]] — Synthèse comparative des architectures RAG : RAG standard et ses limites, Graph RAG, Agentic RAG, requêtes single-hop et multi-hop, coûts et pièges de la similarité vectorielle.
- [[Les transformers]] — Cours de niveau master sur les Transformers : limites du traitement séquentiel et des RNN, dépendances longues, problème de parallélisation, modèles seq2seq puis mécanisme d'attention et architecture complète.
- [[cours/LLM|LLM]] — Cours complet sur les grands modèles de langage : histoire, tokenisation, Transformers, pré-entraînement et post-entraînement, prompting, RAG, outils et agents, raisonnement, multimodalité, évaluation, sécurité, déploiement et choix d'un modèle.
- [[Machine Learning]] — Fiche schématique présentant le flux de travail d'un projet d'analyse de données et la place du machine learning dans ce processus.
- [[Pytorch]] — Cours complet de PyTorch 2.x : tenseurs, appareils de calcul, autograd, nn.Module, fonctions de perte, optimisation, Dataset et DataLoader, boucles d'entraînement, validation, checkpoints, CNN, transfert d'apprentissage, AMP, torch.compile, profilage, entraînement distribué et export.
- [[RAG]] — Cours complet sur les systèmes RAG : limites d'un LLM seul, mémoire externe, embeddings et similarité cosinus, découpage documentaire, recherche hybride, reranking, Graph RAG, Agentic RAG et évaluation.
- [[Travailler avec Claude]] — Formation à Claude Code en cinq couches : écosystème Anthropic, différence avec claude.ai, workflow agentique, système de permissions, modes d'opération et mémoire projet CLAUDE.md.

## Systèmes et administration

- [[Apache]] — Fiche de synthèse sur le serveur HTTP Apache : présentation, installation, configuration, sécurisation et traçage.
- [[Docker]] — Cours sur Docker : définition et installation, terminologie (images, conteneurs), commandes de base, réseaux, volumes, Dockerfile et composition de services.
- [[GNULinux]] — Cours de fond sur GNU/Linux : histoire, licences et logiciel libre, notion de noyau, utilisation du système, shell et commandes, arborescence, droits, processus et administration courante.
- [[InetOrgPerson]] — Cours sur la classe d'objets inetOrgPerson dans LDAP : caractéristiques, attributs, exemples d'utilisation et survol de la RFC 2798.
- [[Initialisation système et des services]] — Cours sur le démarrage d'un système GNU/Linux : séquence de boot, GRUB 2, paramètres du noyau, comparaison systemd / SysV init / upstart, targets et niveaux d'exécution.
- [[Installation Ubuntu]] — Procédure illustrée d'installation d'Ubuntu en version bureau et en version serveur, avec partitionnement LVM chiffré et travaux pratiques.
- [[LDAP]] — Cours sur LDAP et OpenLDAP : structure de l'annuaire, opérations, format LDIF, classes d'objets et RFC de référence.
- [[Les distributions Linux]] — Fiche définissant la notion de distribution GNU/Linux et présentant les principales familles de distributions.
- [[Les namespaces Linux]] — Cours approfondi sur les namespaces du noyau Linux : pourquoi isoler des processus, appartenance multiple d'un processus, différence avec la virtualisation classique et étude de chaque type de namespace.
- [[Postfix]] — Cours sur le serveur de courrier Postfix : rôles de MTA, MDA et MUA, formats de boîtes aux lettres Mbox et Maildir, protocoles IMAP et POP3, configuration et sécurisation.
- [[proc]] — Cours approfondi sur le système de fichiers /proc sous GNU/Linux : rôle pédagogique et opérationnel, différences avec /sys, /dev et /run, et exploration guidée des principaux fichiers.
- [[Watchdog espace disque]] — Script de surveillance de l'occupation disque : paramétrage du seuil, récupération de l'utilisation et déclenchement d'une alerte.
- [[WSL2]] — Procédure d'installation et d'usage de WSL 2, avec ses limitations et l'ajout d'une interface graphique.

## Réseaux et sécurité

- [[HTTP]] — Cours sur le protocole HTTP : contexte historique, architecture client-serveur, cycle requête-réponse, méthodes, en-têtes, codes de statut, HTTPS et sécurité.
- [[Identité numérique européenne]] — Fiche de présentation du portefeuille d'identité numérique européen et du règlement eIDAS.
- [[IPFS]] — Cours sur IPFS : enjeux de la décentralisation du stockage, comparaison avec le modèle client-serveur, Merkle DAG, adressage par contenu et fonctionnement technique du réseau.
- [[Les protocoles de communications]] — Cours d'introduction aux réseaux : notion de protocole, histoire d'ARPANET et d'Internet, adresses MAC et IP, principaux protocoles et ports.
- [[OAuth OpenID]] — Cours sur l'authentification et l'autorisation modernes : principes d'OAuth 2.0 et d'OpenID Connect, jetons JWT, puis mise en œuvre avec Keycloak comme serveur d'autorisation.
- [[Sécurité avancée sous Linux]] — Cours avancé et pratique de sécurisation d'un système GNU/Linux : modèle de menace, permissions et capabilities, durcissement du noyau et de systemd, SELinux/AppArmor/Landlock/seccomp, SSH et réseau, LUKS2, journalisation et audit, réponse à incident, sécurité des conteneurs, chaîne d'approvisionnement et usages de l'IA.
- [[Sécurité avec Python]] — Cours de sécurité applicative en Python appuyé sur un projet Pyramid : authentification, sessions, cryptographie, injections SQL, XSS et CSRF, analyse de journaux, puis les vulnérabilités propres au langage (pickle, eval, sous-processus), la gestion des secrets et la sécurité de la chaîne d'approvisionnement.
- [[Sécurité des IOT en python avec SCADA]] — Cours sur la sécurité des objets connectés et des systèmes SCADA en Python : vecteurs d'attaque de la périphérie au nuage, sécurisation des communications, cryptographie, SSL/TLS et VPN, avec une étude de cas Bluetooth.

## Programmation

- [[Algorithmes avancés en Python]] — Cours complet d'algorithmique avancée en Python : tris, structures de données et graphes, plus courts chemins et arbres couvrants, programmation dynamique, retour sur trace et gloutons, chaînes de caractères, géométrie computationnelle, optimisation combinatoire, parallélisme et verrou global, k-moyennes et arbres de décision, et projet de fin de cours.
- [[Anaconda]] — Distribution Anaconda : histoire, licence commerciale depuis 2024 et alternatives libres (Miniforge, conda-forge), Jupyter et le format IPYNB, installation sous Ubuntu, cycle de vie d'un environnement conda et comparaison avec pip, uv et pixi.
- [[C++]] — Cours complet de C++ : histoire et environnement de développement, bases du langage, programmation orientée objet, et notions avancées.
- [[Google Colab]] — Fiche sur Google Colab : notebook Jupyter hébergé, GPU et TPU, lien avec Google Cloud Platform, puis l'intégration de Gemini et du Data Science Agent, le modèle des unités de calcul, les règles de confidentialité et les alternatives.
- [[Jupyter Notebook et Google Colab]] — Cours retraçant l'histoire d'IPython et de Jupyter, l'architecture des notebooks, leur rôle central en science des données et en enseignement, ainsi que leurs limites.
- [[Mathplotlib]] — Astuce pour produire des graphiques Matplotlib au format vectoriel SVG, dans un script comme dans un notebook.
- [[Numpy]] — Cours pratique sur NumPy : création de tableaux, fonctions usuelles, slices et vues, tableaux multidimensionnels, statistiques et notion d'axe, puis le tirage aléatoire moderne, le broadcasting et les ruptures de NumPy 2.
- [[Pandas]] — Cours sur la bibliothèque Pandas : structures Series et DataFrame, sélection, filtrage, valeurs manquantes, groupement, concaténation et jointures, puis les ruptures de pandas 3.0 — Copy-on-Write, type str adossé à Arrow, pd.col() et migration.
- [[Python]] — Cours de fond sur Python : historique et cycle de vie des versions, ressources de python.org, PEP, syntaxe, types, structures de contrôle, fonctions et classes, annotations de type, dataclasses, pathlib, programmation asynchrone, tests, modules et outillage.
- [[Regex]] — Cours sur les expressions régulières : histoire, usages actuels, fonctionnement, caractères spéciaux, capture et substitution, avec un exemple concret sous Vim.
- [[SQLAchemy]] — Fiche de prise en main de SQLAlchemy : installation, création d'un moteur de connexion, déclaration de modèles et opérations CRUD.
- [[ZC.Buildout]] — Cours sur zc.buildout : assemblage reproductible d'applications, parts et recipes, installation moderne, fichiers buildout.cfg, gestion et verrouillage des versions, héritage de configurations, développement local, commandes de diagnostic et place de Buildout dans l'écosystème Python actuel.

## Développement web

- [[CSS]] — Cours de CSS : historique, modes d'intégration, syntaxe, unités de mesure, sélecteurs, mise en forme et mise en page moderne.
- [[Deform]] — Cours moderne sur Deform 3 sous Pyramid : schémas Colander, widgets, validation, rendu, CSRF, fichiers, formulaires dynamiques et personnalisation.
- [[HTML]] — Cours d'introduction au HTML : bases du langage, organisation et sémantique du contenu, liens, images et ressources, bonnes pratiques et projet final.
- [[Javascript]] — Cours d'introduction à JavaScript : origines, rôle dans la pile technologique, historique d'ECMAScript, variables, types, opérateurs et concepts fondamentaux.
- [[Pyramid]] — Cours sur le framework web Pyramid : installation et structure d'une application, routes et vues, templates Chameleon, formulaires Deform, persistance et sécurité.
- [[Selenium]] — Cours complet sur Selenium 4 avec Python : WebDriver, Selenium Manager, sélecteurs, interactions, attentes, fenêtres et iframes, actions avancées, Page Object Model, pytest, Grid 4, WebDriver BiDi et bonnes pratiques.
- [[Tal et Metal]] — Fiche sur les langages de gabarits TAL (Template Attribute Language) et METAL (Macro Expansion TAL) issus de l'écosystème Zope.

## Génie logiciel

- [[Agile Unified Process (AUP)]] — Présentation de l'Agile Unified Process : origine, principes, disciplines, phases, avantages et limites de cette version allégée du Rational Unified Process.
- [[Architecture des logiciels]] — Cours complet d'architecture logicielle : définitions, attributs de qualité, décisions et vues architecturales, styles d'architecture (monolithique, en couches, microservices, etc.) et documentation.
- [[Design patterns]] — Cours sur les patrons de conception : définition et intérêt, catégorisation (création, structure, comportement) et étude détaillée des principaux patterns.
- [[Les méthodes agiles]] — Cours sur les méthodes agiles : historique et Manifeste Agile, comparaison avec les méthodes traditionnelles, puis Scrum, XP, Kanban, DSDM, Crystal, FDD et Lean Software Development.
- [[Open Spec]] — Cours sur OpenSpec et le développement piloté par les spécifications : limites du développement piloté par prompts, principes du Spec-Driven Development et mise en œuvre avec un agent IA.
- [[Outils de modélisation textuels]] — Liste commentée d'outils de génération de diagrammes à partir de texte : D2, Pikchr, Diagon, Typograms, Markdeep et autres.
- [[Principes SOLID en COO]] — Fiche sur les cinq principes SOLID de conception orientée objet : responsabilité unique, ouvert/fermé, substitution de Liskov, ségrégation des interfaces et inversion des dépendances.
- [[Projet Encadré]] — Sujet de projet encadré : conception et architecture d'une bibliothèque numérique décentralisée, réalisée en binômes hétérogènes avec évaluation croisée entre équipes.
- [[TOGAF]] — Cours sur TOGAF : fondements de l'architecture d'entreprise, structure du cadre, méthode ADM, artefacts, livrables et gouvernance, mise en pratique et certifications.

## Données et web sémantique

- [[Bases de données relationnelles]] — Cours sur le modèle relationnel : modélisation des données, langage de manipulation SQL, vues, index, contraintes, transactions et isolation, sécurité et droits d'accès.
- [[Data Mining en Python]] — Cours de fouille de données en Python : concepts et processus, préparation et nettoyage, fouille de texte avec NLTK et spaCy, techniques et modèles, évaluation, puis la fuite de données, les chaînes scikit-learn, l'écosystème Arrow et le cadre juridique RGPD et règlement IA.
- [[SOLID]] — Cours sur SOLID, le projet de web décentralisé de Tim Berners-Lee : principe des pods, structuration des données, RDF et SPARQL, et mise en place d'un environnement de développement.

## Connaissances et outils

- [[Excalidraw]] — Fiche de présentation du greffon Excalidraw pour Obsidian : usages, mode présentation, insertion de médias et apports pour l'illustration des notes.
- [[git]] — Cours sur Git : historique, installation et configuration du client, création d'un dépôt et principales commandes du quotidien.
- [[Markdown]] — Cours sur Markdown : formatage de base, apports des variantes GitHub et Obsidian, extensions, écriture de formules mathématiques et syntaxe étendue.
- [[Mermaid pour Obsidian]] — Cours sur Mermaid dans Obsidian : origines et adoption, installation, utilisation et principaux types de diagrammes (flux, séquence, Gantt, classes).
- [[MindMap sous Obsidian]] — Procédure d'installation et d'utilisation du greffon de cartes mentales dans Obsidian, avec quelques astuces.
- [[Obsidian OSIA Construire son système d'exploitation personnel augmenté par l'IA]] — Cours en trente chapitres et trois annexes sur la construction d'un système d'information personnel souverain : vault Obsidian, modèle de données par frontmatter, schéma et modèle métier, graphe sémantique, Git, puis couche agentique (constitution, domaines, skills, pipelines, mémoire, sécurité, tests, observabilité). L'annexe B traite l'application du modèle à un coffre existant, l'annexe C fournit une implémentation de référence testée, installée dans scripts/osia.
- [[Obsidian]] — Cours d'introduction à Obsidian mis à jour en 2026 : philosophie Local First, Markdown, liens, Properties, recherche, templates, Bases, Canvas, extensions, synchronisation, Web Clipper, CLI et automatisation.
- [[PlantUML pour Obsidian]] — Procédure d'installation d'un serveur PlantUML local pour Obsidian, avec mise à jour du système, installation de Java et présentation de modeleurs graphiques générant du PlantUML.
- [[Visual studio code]] — Cours de prise en main de Visual Studio Code : historique, installation, interface et barre d'activités, panneaux, extensions utiles et réglages du quotidien.

## Histoire et matériel informatique

- [[Histoire des langages de programmation]] — Cours d'histoire des langages de programmation : concepts de syntaxe et de sémantique, classification des langages, précurseurs, Fortran, COBOL, LISP, langages structurés et évolutions modernes.
- [[Historique Linux]] — Fiche chronologique sur les racines d'UNIX, la naissance de la Free Software Foundation, la GPL et le projet GNU.
- [[Informatique]] — Carte mentale de présentation générale de l'informatique : histoire du matériel, génie logiciel, cycle en V et architecture de von Neumann.
- [[Parsing Expression Grammars PEG]] — Fiche sur les Parsing Expression Grammars : non-terminaux, terminaux, règles de production, opérateurs de séquence, choix, répétition et prédicats, et gestion de la récursivité.

## Droit et conformité

- [[Droits d'auteur]] — Cours sur la propriété intellectuelle et le droit d'auteur : distinction avec la propriété industrielle, brevets, œuvres originales, collectives et posthumes, licences informatiques et chronologie des durées de protection.
- [[Règlement Général sur la Protection des Données (RGPD)]] — Cours sur le RGPD : contexte et cadre juridique, principes fondamentaux (licéité, minimisation, exactitude, limitation de conservation, intégrité) et droits des personnes concernées.

## Mathématiques et logique

- [[Logique]] — Cours de logique appliquée à l'informatique : logique propositionnelle, opérations de base, tautologies et inférence, puis logique des prédicats du premier ordre.
- [[notions d algèbre linéaire et de dérivation]] — Prérequis mathématiques du deep learning : vecteurs, matrices et produit matriciel, formes et diffusion, dérivées partielles et gradient, règle de la chaîne, jacobienne — et ce que chaque notion devient dans le moteur Autograd de PyTorch.
- [[Systèmes numériques]] — Cours sur les systèmes de numération : analyse des systèmes décimal et sexagésimal, critères d'un « meilleur » système et étude du bibi-binaire de Boby Lapointe.

## Langues et linguistique

- [[Esperanto]] — Cours d'espéranto orienté ingénierie : histoire et objectifs de la langue internationale, alphabet et prononciation, grammaire fondamentale et vocabulaire de base.

## Pédagogie et apprentissage

- [[Apprendre]] — Synthèse d'une vidéo de Science étonnante sur les mécanismes de l'apprentissage : mémoire de travail et à long terme, répétition espacée, auto-tests, diversification, apprentissage génératif et cartes mentales.

## Économie et entreprise

- [[SEO]] — Cours complet sur le référencement naturel moderne : fonctionnement des moteurs de recherche, stratégie éditoriale, SEO on-page et technique, indexation, performances, données structurées, popularité, mesure et visibilité dans les fonctionnalités de recherche générative.

## Management et productivité

- [[Getting things done]] — Fiche de synthèse sur la méthode Getting Things Done de David Allen : collecte, clarification, organisation, revue et engagement.
