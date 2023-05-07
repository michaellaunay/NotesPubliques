En 2008 j'ai fait une boulette, j'avais effacé tout le contenu d'un serveur que je dé-commissionnais par "sudo rm -rf /*", j'avais copié son contenu sur mon nouveau serveur et fait des vérifications, mais il manquait un répertoire...

J'ai passé 15h à récupérer le contenu du disque qui était au format ext2, donc sans journalisation.

À l'époque, partout sur internet, on disait que c'était impossible.

Sauf que tant que nous n'écrivons pas sur le disque, les données restent présentes, mais ne sont plus accessibles, d'ailleurs si vous formatez un disque dur à plateaux, il reste possible de récupérer magnétiquement les données (jusqu'à trois fois).

Lorsque j'ai vu mon erreur, j'ai basculé immédiatement la machine en mode "rescue" sur un Ubuntu (j'étais chez Dedibox devenu online.fr puis Scalleway.fr).

J'ai récupéré un bout de code en C (une preuve de concept) que j'ai adapté, j'ai fait un "dump" (copie) de tout le contenu du disque vers un autre serveur, puis j'ai compilé et lancé mon bout de code C sur le fichier unique. Ce programme a essayé de relier les blocs de données et a reconstitué une partie du répertoire perdu. J'avais des numéros au lieu des noms de fichiers. J'ai mis 10h à renommer les fichiers et recréer l'arborescence en auditant chaque fichier et n'ai quasiment rien perdu. Ma chance est qu'il s'agissait de fichiers textes et images.

Aujourd'hui, avec la journalisation des systèmes de fichiers, c'est beaucoup plus facile de restaurer les données...

En avril 2020, ma société Ecréall a été hackée, deux jours avant qu'une alerte CERT indique une faille sur Salt l'outil de déploiement que nous utilisions https://www.linkedin.com/posts/michaellaunay19730119_les-serveurs-de-mon-entreprise-ecr%C3%A9all-ont-activity-6663350419386970112-jkSR .

J'ai mis en œuvre mon plan de reprise sur incident, car à Ecreall on changeait nos serveurs tous les deux ans ce qui nous forçait à maintenir à jour notre procédure de restauration/duplication.

Une bonne entreprise se doit de tester son plan de reprise sur incident dans des conditions proches du réel et l'installation "from scratch" de nos serveurs est toujours l'occasion de tester notre procédure et de la mettre à jour.

Aujourd'hui, personne ne cherche plus à être impénétrable face aux attaques, car cela est impossible, et le hacking de la NSA l'a bien montré https://www.silicon.fr/cybercriminels-chinois-russes-outils-de-hacking-nsa-173367.html .

Le hacking, la maladresse, ou le cas de force majeur sont trois raisons de mettre en place un plan de reprise sur incident. Et la maladresse est probablement la plus répandue, car la source numéro un de bogue et de grain de sable dans un mécanisme d'horlogerie est assise devant l'écran.

Même GitLab a détruit ses bases de données suite à incident "involontaire" en 2017 https://youtu.be/tLdRBsuvVKc . Ils ont bien géré la partie communication avec les utilisateurs/clients et ont indiqué à quelle étape ils en étaient tout au long de l'incident. Leur "postmortem" est instructif.

La conclusion de l'analyse après l'incident est qu'il faut toujours faire 3 sauvegardes, dont deux sur des supports et dans des endroits différents et surtout tester fréquemment les procédures de reprise sur incident. Seule cette pratique sauve l'entreprise.

Vérifiez bien que vos données ne sont pas dans le même datacenter. Ainsi quand le datacenter d'OVH a brûlé, beaucoup de boites ont perdu leurs backups parce que les serveurs de backup étaient sur le même site...

Les contrats d'OVH qui opposaient un recours pour les cas de force majeure et leur clause de limitation des indemnités aux sommes perçues ne vont pas les dédouaner de leurs responsabilités :

https://www.usine-digitale.fr/article/quels-impacts-contractuels-suite-a-l-incendie-dans-les-data-centers-d-ovhcloud.N2119346

L'entreprise WebEncheres où j'avais acheté d'occasion mon tracteur (c'est plus sexy qu'une Lamborghini) avait disparu pendant quelques mois suite à l'incendie et a depuis été rachetée par son concurrent (Agorastore), est-ce lié ? Ce qui est sûr c'est qu'une indisponibilité d'un site de commerce en ligne fragilise grandement l'entreprise qui est derrière.

Pour conclure, pour se protéger au mieux des trois principales sources de destructions de vos données, respectez la règle des trois sauvegardes dont deux sur des supports différents.

PS: Avant de se lancer dans la production de voitures de sport, Lamborghini était spécialisé dans la fabrication de tracteurs agricoles. La légende raconte que Ferruccio Lamborghini, fondateur de la marque, était mécontent d'une Ferrari qu'il possédait et a soulevé le problème auprès d'Enzo Ferrari. En réponse, Enzo aurait dédaigné les critiques de Ferruccio, ce qui aurait poussé ce dernier à créer sa propre entreprise de voitures de sport pour concurrencer Ferrari. Ainsi, l'arrogance d'Enzo Ferrari a allumé la flamme chez Lamborghini.