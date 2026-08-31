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
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: false
metadata_verifiees: false
---

# cours

Index régénéré par `make index` — ne pas éditer à la main. Vue graphique : [[_INDEX_cours.excalidraw]].

## Intelligence artificielle

- [[DeepSeek Harness]] — Cours complet sur DeepSeek Harness (dsh) : notion de harness agentique, architecture Cordis et tout-plugin, profils et presets, modèles, outils, skills, sandbox, MCP, extensions, automatisation, sécurité et développement de plugins.
- [[Du PDF scanné au corpus exploitable OCR multimodal local avec olmOCR 2 et Infinity-Parser2-Pro]] — Cours-atelier sur une chaîne locale, traçable et reproductible de transformation de PDF scannés en corpus exploitable : ingestion, rendu, olmOCR 2, Infinity-Parser2 Pro/Flash, routage, contrôle qualité, indexation et RAG.
- [[Hermes Agent]] — Cours approfondi et actualisé sur Hermes Agent (Nous Research) : architecture agentique, mémoire et recherche de sessions, skills auto-améliorés, cron, délégation multi-agent, profils, MCP, plugins, gateways multi-canaux, backends d'exécution, sécurité et migration OpenClaw.
- [[Les CNN et RNN]] — Cours de niveau master sur les CNN et RNN : convolution et champs réceptifs, architectures convolutionnelles modernes, récurrence et BPTT, dépendances longues, LSTM/GRU, Seq2Seq, attention, entraînement PyTorch et choix d'architecture en 2026.
- [[Les RAGs]] — Synthèse comparative des architectures RAG : RAG standard et ses limites, Graph RAG, Agentic RAG, requêtes single-hop et multi-hop, coûts et pièges de la similarité vectorielle.
- [[Les transformers]] — Cours de niveau master sur les Transformers, conçu comme une progression pédagogique : séquences et attention, Q/K/V et calcul matriciel, positions, multi-head attention, blocs modernes, entraînement, inférence et KV cache, GQA, FlashAttention, MoE, contexte long, multimodalité, adaptation, performances PyTorch, limites et travaux pratiques.
- [[LLM]] — Cours complet sur les grands modèles de langage : histoire, tokenisation, Transformers, pré-entraînement et post-entraînement, prompting, RAG, outils et agents, raisonnement, multimodalité, évaluation, sécurité, déploiement et choix d'un modèle.
- [[Machine Learning]] — Cours complet de machine learning classique : formulation du problème, préparation des données, pipelines scikit-learn, validation, métriques, algorithmes supervisés et non supervisés, interprétabilité, reproductibilité, déploiement et surveillance.
- [[Pytorch]] — Cours complet de PyTorch 2.x : tenseurs, appareils de calcul, autograd, nn.Module, fonctions de perte, optimisation, Dataset et DataLoader, boucles d'entraînement, validation, checkpoints, CNN, transfert d'apprentissage, AMP, torch.compile, profilage, entraînement distribué et export.
- [[RAG]] — Cours progressif sur les systèmes RAG : du besoin documentaire et de l'ingestion au retrieval hybride, au reranking, aux citations, à l'évaluation, au GraphRAG, aux architectures agentiques, à la sécurité et à la mise en production.
- [[Travailler avec Claude]] — Cours complet sur Claude et Claude Code : surfaces, installation, contexte, CLAUDE.md, auto memory, règles, permissions, sandbox, modèles, skills, subagents, agent teams, MCP, hooks, plugins, automatisation, CI/CD, Git, sécurité et exploitation en équipe.

## Systèmes et administration

- [[Apache]] — Cours pratique sur Apache HTTP Server 2.4 : architecture, installation Debian/Ubuntu, VirtualHost, autorisation, TLS/ACME, HTTP/2, PHP-FPM, reverse proxy, journalisation, performances, sécurité et dépannage.
- [[Docker]] — Cours complet sur Docker : conteneurs, images OCI, Dockerfile, BuildKit/Buildx, Compose, réseaux, volumes, sécurité, rootless, supply chain et exploitation en production.
- [[GNULinux]] — Cours de fond sur GNU/Linux : histoire, licences et logiciel libre, notion de noyau, utilisation du système, shell et commandes, arborescence, droits, processus et administration courante.
- [[InetOrgPerson]] — Cours complet sur inetOrgPerson : schéma LDAP, héritage, attributs, syntaxes, nommage, contraintes, sécurité, interopérabilité et exploitation avec OpenLDAP.
- [[Initialisation système et des services]] — Cours complet sur le démarrage GNU/Linux et la gestion moderne des services : UEFI/BIOS, GRUB 2, noyau et initramfs, PID 1, unités systemd, dépendances, targets, journal, timers, activation, cgroups, durcissement et diagnostic.
- [[Installation Ubuntu]] — Installer proprement Ubuntu Desktop ou Server : choix de version, vérification de l'image, UEFI/Secure Boot, partitionnement, LVM/LUKS, réseau, SSH, post-installation, automatisation Subiquity/autoinstall et dépannage.
- [[LDAP]] — Cours complet sur LDAPv3 et OpenLDAP : DIT, DN, schémas, filtres, LDIF, TLS/SASL, ACL, mots de passe, réplication, sauvegarde, supervision et intégration applicative.
- [[Les distributions Linux]] — Cours de référence pour comprendre ce qu'est une distribution Linux, ses composants, les grandes familles Debian, Fedora/RHEL, SUSE, Arch, Gentoo, Alpine et NixOS, les modèles de publication et les critères permettant de choisir une distribution adaptée à un poste, un serveur, un conteneur ou un système spécialisé.
- [[Les namespaces Linux]] — Cours approfondi et progressif sur les namespaces Linux : modèle du noyau, API clone/clone3/unshare/setns, UTS/PID/mount/network/IPC/user/cgroup/time, /proc, rootless, idmapped mounts, sécurité, conteneurs et diagnostic.
- [[Postfix]] — Cours complet sur Postfix : architecture d'un MTA, SMTP et submission, configuration de main.cf et master.cf, relais, domaines locaux et virtuels, SASL, TLS, LMTP/Dovecot, SPF/DKIM/DMARC, sécurité, files d'attente, observabilité, délivrabilité et diagnostic en production.
- [[proc]] — Cours approfondi et progressif sur procfs sous GNU/Linux : modèle de procfs, processus, mémoire, descripteurs, namespaces, réseau, sysctl, sécurité, PSI, conteneurs et diagnostic de production.
- [[Watchdog espace disque]] — Cours pratique sur la supervision de la capacité des systèmes de fichiers Linux : blocs, espace disponible, inodes, script Bash robuste, journalisation, systemd.timer, diagnostic des causes de saturation, alertes et intégration Prometheus/node_exporter.
- [[WSL2]] — Cours complet sur WSL 2 : architecture, installation et cycle de vie des distributions, systemd, WSLg, interopérabilité Windows/Linux, systèmes de fichiers, réseau NAT et mirrored, configuration, sauvegarde, conteneurs, GPU, USB, sécurité et dépannage.

## Réseaux et sécurité

- [[HTTP]] — Cours complet sur HTTP : sémantique, HTTP/1.1, HTTP/2, HTTP/3 et QUIC, méthodes, statuts, en-têtes, cache, cookies, authentification, HTTPS/TLS, CORS, sécurité, API, performances et diagnostic.
- [[Identité numérique européenne]] — Cours sur le cadre européen d'identité numérique issu du règlement eIDAS révisé : portefeuille EUDI, acteurs, PID et attestations d'attributs, services de confiance, protection des données, architecture technique, OpenID4VCI/OpenID4VP, mdoc, SD-JWT VC, certification, parties utilisatrices et déploiement en France.
- [[IPFS]] — Cours complet sur IPFS : adressage par contenu, CID, Merkle DAG, UnixFS et IPLD, réseau libp2p/DHT/Bitswap, installation et exploitation d'un nœud Kubo, épinglage, MFS, CAR, IPNS et DNSLink, passerelles vérifiables, réseaux privés et clusters, sécurité, écosystème, limites et travaux pratiques.
- [[Les protocoles de communications]] — Cours complet sur les protocoles réseau : modèles OSI/TCP-IP, Ethernet et Wi-Fi, IPv4/IPv6, routage, TCP/UDP/QUIC, DNS, DHCP, TLS, HTTP, messagerie, VPN, IoT, protocoles décentralisés, diagnostic et sécurité.
- [[OAuth OpenID]] — Cours complet sur OAuth 2.0, l'évolution OAuth 2.1, OpenID Connect, JWT/JOSE, PKCE, DPoP, PAR/JAR, les architectures BFF et l'intégration moderne avec Keycloak et Pyramid.
- [[Sécurité avancée sous Linux]] — Cours avancé et pratique de sécurisation d'un système GNU/Linux : modèle de menace, permissions et capabilities, durcissement du noyau et de systemd, SELinux/AppArmor/Landlock/seccomp, SSH et réseau, LUKS2, journalisation et audit, réponse à incident, sécurité des conteneurs, chaîne d'approvisionnement et usages de l'IA.
- [[Sécurité avec Python]] — Cours de sécurité applicative en Python appuyé sur un projet Pyramid : authentification, sessions, cryptographie, injections SQL, XSS et CSRF, analyse de journaux, puis les vulnérabilités propres au langage (pickle, eval, sous-processus), la gestion des secrets et la sécurité de la chaîne d'approvisionnement.
- [[Sécurité des IOT en python avec SCADA]] — Cours avancé sur la cybersécurité IoT, OT et SCADA avec Python : architecture, segmentation, IEC 62443, MQTT, Modbus, OPC UA, BLE, PKI/TLS, supervision, réponse à incident, développement sécurisé et cadre réglementaire européen.

## Programmation

- [[Algorithmes avancés en Python]] — Cours complet d'algorithmique avancée en Python : tris, structures de données et graphes, plus courts chemins et arbres couvrants, programmation dynamique, retour sur trace et gloutons, chaînes de caractères, géométrie computationnelle, optimisation combinatoire, parallélisme et verrou global, k-moyennes et arbres de décision, et projet de fin de cours.
- [[Anaconda]] — Distribution Anaconda : histoire, licence commerciale depuis 2024 et alternatives libres (Miniforge, conda-forge), Jupyter et le format IPYNB, installation sous Ubuntu, cycle de vie d'un environnement conda et comparaison avec pip, uv et pixi.
- [[C++]] — Cours complet de C++ moderne : fondamentaux, RAII, STL, templates, concurrence, C++20/C++23, aperçu C++26, CMake, tests, sécurité mémoire et projets.
- [[Google Colab]] — Fiche sur Google Colab : notebook Jupyter hébergé, GPU et TPU, lien avec Google Cloud Platform, puis l'intégration de Gemini et du Data Science Agent, le modèle des unités de calcul, les règles de confidentialité et les alternatives.
- [[Internationalisation]] — Cours sur l'internationalisation et la localisation des applications en Python (2026) : concepts et formats, gettext (marquage, pluriels, contexte, extraction, catalogues PO/MO), Babel (catalogues et formatage CLDR des dates, nombres, monnaies), intégration complète dans Pyramid 2.1 avec Chameleon (TranslationStringFactory, localizer, négociation de langue, extraction avec pybabel et lingua), bonnes pratiques et outillage ; tous les exemples ont été exécutés.
- [[Jupyter Notebook et Google Colab]] — Cours complet sur Jupyter, JupyterLab et Google Colab : architecture des notebooks, kernels, environnements, reproductibilité, sécurité, collaboration, GPU/TPU, Git et bonnes pratiques.
- [[Mathplotlib]] — Cours complet sur Matplotlib : modèle Figure/Axes/Artist, API orientée objet, graphiques usuels, mises en page, styles, couleurs, annotations, export vectoriel et raster, notebooks, performances, accessibilité et intégration NumPy/Pandas.
- [[Numpy]] — Cours pratique sur NumPy : création de tableaux, fonctions usuelles, slices et vues, tableaux multidimensionnels, statistiques et notion d'axe, puis le tirage aléatoire moderne, le broadcasting et les ruptures de NumPy 2.
- [[Pandas]] — Cours sur la bibliothèque Pandas : structures Series et DataFrame, sélection, filtrage, valeurs manquantes, groupement, concaténation et jointures, puis les ruptures de pandas 3.0 — Copy-on-Write, type str adossé à Arrow, pd.col() et migration.
- [[Python]] — Cours de fond sur Python : historique et cycle de vie des versions, ressources de python.org, PEP, syntaxe, types, structures de contrôle, fonctions et classes, annotations de type, dataclasses, pathlib, programmation asynchrone, tests, modules et outillage.
- [[Regex]] — Cours complet sur les expressions régulières : théorie, syntaxes, moteurs, Unicode, captures, lookarounds, substitutions, Python, JavaScript, POSIX, Vim, PCRE2, performances, ReDoS, sécurité et bonnes pratiques.
- [[SQLAchemy]] — Fiche de prise en main de SQLAlchemy : installation, création d'un moteur de connexion, déclaration de modèles et opérations CRUD.
- [[ZC.Buildout]] — Cours sur zc.buildout : assemblage reproductible d'applications, parts et recipes, installation moderne, fichiers buildout.cfg, gestion et verrouillage des versions, héritage de configurations, développement local, commandes de diagnostic et place de Buildout dans l'écosystème Python actuel.

## Développement web

- [[CSS]] — Cours complet de CSS moderne : cascade, sélecteurs, responsive design, Flexbox, Grid, container queries, nesting, scope, couleurs modernes, animations, accessibilité, performance et architecture CSS.
- [[Deform]] — Cours moderne sur Deform 3 sous Pyramid : schémas Colander, widgets, validation, rendu, CSRF, fichiers, formulaires dynamiques et personnalisation.
- [[HTML]] — Cours complet de HTML moderne : structure, sémantique, formulaires, médias, accessibilité, SEO, performance, sécurité et fonctionnalités natives du HTML Living Standard.
- [[Javascript]] — Cours complet de JavaScript moderne : ECMAScript, types, fonctions, objets, modules, asynchronisme, DOM, Web APIs, Node.js, tests, sécurité, performance et pratiques de développement actuelles.
- [[Pyramid]] — Cours approfondi sur Pyramid 2.1 : architecture WSGI et cycle de requête, configuration, routes, vues, traversal, sécurité moderne, sessions/CSRF, Deform, SQLAlchemy/ZODB, LDAP/OIDC, tests, migration et déploiement.
- [[Selenium]] — Cours complet sur Selenium 4 avec Python : WebDriver, Selenium Manager, sélecteurs, interactions, attentes, fenêtres et iframes, actions avancées, Page Object Model, pytest, Grid 4, WebDriver BiDi et bonnes pratiques.
- [[Tal et Metal]] — Cours complet sur Zope Page Templates : TAL pour la logique de présentation, TALES pour les expressions, METAL pour les macros et les slots, avec les différences entre Zope et Chameleon, l'échappement HTML, l'i18n, Pyramid et les bonnes pratiques d'architecture.

## Génie logiciel

- [[Agile Unified Process (AUP)]] — Cours complet sur l'Agile Unified Process (AUP) : origines, phases, disciplines, développement itératif piloté par les risques, artefacts légers, architecture, tests, déploiement, adaptation moderne avec DevOps et comparaison avec Scrum, Kanban, XP, RUP et Disciplined Agile.
- [[Architecture des logiciels]] — Cours complet et moderne d'architecture logicielle : décisions et compromis, attributs de qualité, styles architecturaux, modularité, DDD, systèmes distribués, résilience, sécurité, observabilité, documentation C4/ADR et évolution des systèmes.
- [[Design patterns]] — Cours complet sur les patrons de conception : patterns GoF, idiomes Python modernes, patterns d'architecture et d'entreprise, événements, résilience, concurrence, anti-patterns et méthode de choix pragmatique.
- [[Les méthodes agiles]] — Cours complet et actualisé sur l'agilité en développement logiciel : Manifeste Agile, Scrum 2020, XP, Kanban 2025, Lean, discovery produit, estimation et prévision, métriques de flux et DORA, pratiques d'ingénierie, DevOps, leadership, mise à l'échelle, IA et anti-patterns.
- [[Open Spec]] — Cours sur OpenSpec et le développement piloté par les spécifications : limites du développement piloté par prompts, principes du Spec-Driven Development et mise en œuvre avec un agent IA.
- [[Outils de modélisation textuels]] — Cours de choix et d'utilisation des outils de modélisation textuelle : Mermaid, PlantUML, D2, Graphviz, Structurizr, Kroki, DBML, WaveDrom, BPMN Sketch Miner, State Machine Cat, Svgbob, Pikchr et Penrose, avec workflows Git/CI et critères de reproductibilité.
- [[Principes SOLID en COO]] — Cours complet sur les cinq principes SOLID de conception logicielle : SRP, OCP, LSP, ISP et DIP, avec exemples Python, implications architecturales, relations avec les design patterns, les tests, l'injection de dépendances et les erreurs d'interprétation courantes.
- [[Projet Encadré]] — Sujet de projet encadré : conception et architecture d'une bibliothèque numérique décentralisée, réalisée en binômes hétérogènes avec évaluation croisée entre équipes.
- [[TOGAF]] — Cours complet sur le TOGAF Standard, 10th Edition : architecture d’entreprise, ADM, gestion des exigences, contenu et repository d’architecture, gouvernance, capacités métier, roadmaps, pratiques Agile/produit/cloud, ArchiMate et certifications.
- [[UML Ecore EMF Plantuml QVT Mermaid PyEcore]] — Cours d'ingénierie dirigée par les modèles (2026) : UML 2.5.1 et SysML v2, MDA et les quatre niveaux MOF, Ecore et EMF, transformations de modèles (QVT, ATL, Acceleo), grammaires et langages dédiés (BNF, EBNF, PEG, Xtext, Langium, textX), sémantiques formelles, pratique complète avec pyecore et pyecoregen (métamodèle, instances, XMI, export PlantUML, génération de code), diagrammes textuels PlantUML et Mermaid ; tous les exemples ont été exécutés.

## Données et web sémantique

- [[Bases de données relationnelles]] — Cours complet sur les bases de données relationnelles : modèle relationnel, SQL moderne, modélisation, normalisation, contraintes, transactions, concurrence, index, optimisation, sécurité, migrations, sauvegarde et exploitation.
- [[Data Mining en Python]] — Cours de fouille de données en Python : concepts et processus, préparation et nettoyage, fouille de texte avec NLTK et spaCy, techniques et modèles, évaluation, puis la fuite de données, les chaînes scikit-learn, l'écosystème Arrow et le cadre juridique RGPD et règlement IA.
- [[SOLID]] — Cours complet sur Solid : Pods, WebID, RDF/Linked Data, protocole HTTP, Solid-OIDC, WAC/ACP, Community Solid Server, développement d'applications, notifications, sécurité et interopérabilité.

## Connaissances et outils

- [[Excalidraw]] — Cours complet sur Excalidraw et son intégration dans Obsidian : dessin, fichiers Markdown Excalidraw, liens et transclusions, frames, bibliothèques, templates, export, OCR, automatisation avec ExcalidrawAutomate, IA, sécurité, performances et workflows de visual thinking.
- [[git]] — Cours complet sur Git : modèle de données, workflow quotidien, branches, remotes, restauration, réécriture d'historique, worktrees, Git LFS, Git Xet/Hugging Face, sécurité, collaboration et administration.
- [[Markdown]] — Cours complet sur Markdown : CommonMark, GitHub Flavored Markdown, extensions GitHub et Obsidian, portabilité, accessibilité, sécurité, mathématiques, diagrammes et automatisation.
- [[Mermaid pour Obsidian]] — Cours complet sur Mermaid dans Obsidian : syntaxe, flowcharts, séquences, classes, états, ER, Gantt, GitGraph, mindmaps, graphiques récents, thèmes, accessibilité, liens internes, sécurité, compatibilité de versions et bonnes pratiques.
- [[MindMap sous Obsidian]] — Cours complet sur les cartes mentales dans Obsidian : choix entre Markdown/MarkMind, Canvas, Mermaid et Excalidraw, migration de l'ancien Enhanced MindMap, conception de cartes utiles, intégration au graphe, automatisation, export, synchronisation, accessibilité et bonnes pratiques de pérennité.
- [[Obsidian OSIA Construire son système d'exploitation personnel augmenté par l'IA]] — Cours en trente chapitres et trois annexes sur la construction d'un système d'information personnel souverain : vault Obsidian, modèle de données par frontmatter, schéma et modèle métier, graphe sémantique, Git, puis couche agentique (constitution, domaines, skills, pipelines, mémoire, sécurité, tests, observabilité). L'annexe B traite l'application du modèle à un coffre existant, l'annexe C fournit une implémentation de référence testée, installée dans scripts/osia.
- [[Obsidian]] — Cours d'introduction à Obsidian mis à jour en 2026 : philosophie Local First, Markdown, liens, Properties, recherche, templates, Bases, Canvas, extensions, synchronisation, Web Clipper, CLI et automatisation.
- [[PlantUML pour Obsidian]] — Procédure d'installation d'un serveur PlantUML local pour Obsidian, avec mise à jour du système, installation de Java et présentation de modeleurs graphiques générant du PlantUML.
- [[Visual studio code]] — Cours complet et actualisé sur Visual Studio Code : interface, configuration, navigation, Git, terminal, tâches, débogage, Python, développement distant, Dev Containers, sécurité, Copilot, agents et MCP.

## Histoire et matériel informatique

- [[Histoire des langages de programmation]] — Cours d'histoire des langages de programmation : concepts fondamentaux, paradigmes, grandes familles de langages, évolution des années 1950 à 2026, runtimes, systèmes de types, WebAssembly, sûreté mémoire et tendances contemporaines.
- [[Historique Linux]] — Cours historique et conceptuel sur les racines d'UNIX, BSD, GNU et Linux, la naissance du logiciel libre et de l'open source, l'émergence des distributions, le modèle de développement du noyau et l'évolution de l'écosystème jusqu'en 2026.
- [[Informatique]] — Cours d'introduction générale à l'informatique : représentation de l'information, histoire, matériel, systèmes, réseaux, programmation, algorithmique, données, génie logiciel, sécurité, IA et enjeux contemporains.
- [[Parsing Expression Grammars PEG]] — Cours complet sur les Parsing Expression Grammars : sémantique déterministe et choix ordonné, opérateurs PEG, backtracking et mémoïsation, récursion gauche, construction d'AST, erreurs, tests, performances, sécurité et mise en œuvre avec des outils modernes comme CPython/pegen, TatSu, Peggy, Ohm et pest.

## Droit et conformité

- [[Droits d'auteur]] — Cours actualisé sur le droit d'auteur français et européen : originalité, droits moraux et patrimoniaux, logiciels, bases de données, licences libres, domaine public, exceptions, contrats, IA générative et fouille de textes et de données.
- [[Règlement Général sur la Protection des Données (RGPD)]] — Cours complet et actualisé sur le RGPD : champ d'application, données personnelles, bases légales, droits, accountability, sous-traitance, sécurité, violations, AIPD, transferts internationaux, cookies, IA et mise en conformité opérationnelle.

## Mathématiques et logique

- [[Logique]] — Cours complet de logique appliquée à l’informatique : logique propositionnelle et du premier ordre, méthodes de preuve, SAT/SMT, logique temporelle et modale, logique floue, programmation logique, vérification formelle et assistants de preuve.
- [[notions d algèbre linéaire et de dérivation]] — Prérequis mathématiques du deep learning : vecteurs, matrices et produit matriciel, formes et diffusion, dérivées partielles et gradient, règle de la chaîne, jacobienne — et ce que chaque notion devient dans le moteur Autograd de PyTorch.
- [[Systèmes numériques]] — Cours complet sur les systèmes de numération : bases positionnelles, conversions, calcul en binaire/octal/hexadécimal, entiers signés, virgule flottante IEEE 754, bases alternatives et système bibi-binaire de Boby Lapointe.

## Langues et linguistique

- [[Esperanto]] — Cours d'espéranto orienté ingénierie : histoire et objectifs de la langue internationale, alphabet et prononciation, grammaire fondamentale et vocabulaire de base.

## Pédagogie et apprentissage

- [[Apprendre]] — Cours fondé sur les sciences de l'apprentissage : mémoire de travail et à long terme, récupération active, espacement, entrelacement, exemples résolus, auto-explication, feedback, métacognition, sommeil, prise de notes et usage raisonné de l'IA.

## Économie et entreprise

- [[SEO]] — Cours complet sur le référencement naturel moderne : fonctionnement des moteurs de recherche, stratégie éditoriale, SEO on-page et technique, indexation, performances, données structurées, popularité, mesure et visibilité dans les fonctionnalités de recherche générative.

## Management et productivité

- [[Getting things done]] — Cours pratique sur Getting Things Done (GTD) : capture, clarification, organisation, réflexion et engagement, listes de projets et prochaines actions, revue hebdomadaire, horizons de focus, planification naturelle et mise en œuvre dans un système papier ou numérique.
