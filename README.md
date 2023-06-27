# NotesPubliques

Ce répertoire contient toutes mes notes publiques, qui représentent un sous-ensemble de l'ensemble de mes notes. Il comprend toutes mes ressources pédagogiques, documents, articles, commentaires, ainsi que divers posts issus de réseaux sociaux tels que LinkedIn, tous au format Markdown. Les cours sont une réécriture complète et complémentaire des ressources disponibles sur les liens suivants : https://github.com/michaellaunay/CoursGNULinux et https://github.com/michaellaunay/CoursInformatique. Tous les documents que j'ai produits depuis 2005 et qui sont publics seront convertis en notes Obsidian. Je vous invite à utiliser `Obsidian` ou le module `Markdown Memo` pour `Visual Studio Code` pour parcourir ces notes.

Comme ce dépôt n'est pas le dépôt principal de mes notes, mais une copie de la partie publique, il se peut que vous rencontriez des commits ou des liens vers des ressources indisponibles.

# Un peu de technique pour les curieux

Voici comment j'ai créé ce dépôt :

```bash
git clone git@github.com:michaellaunay/Notes.git NotesPubliquesUp
cd NotesPubliquesUp
git log
git filter-repo --path cours --path informatique --path sciences --path réflexions --path '_INDEX PRINCIPAL.md'
git remote add NotesPubliques git@github.com:michaellaunay/NotesPubliques.git
git push NotesPubliques
cd ..
rm -rf NotesPubliquesUp
```

Création de la branche Notes qui sert de miroir partiel du dépôt Notes :

```bash
git checkout -b Notes
git push --set-upstream origin Notes
```

Ajout du fichier de Versions.md qui me permet de savoir à partir d'où je dois mettre à jour.

```bash
git clone git@github.com:michaellaunay/Notes.git NotesPubliquesUp
cd NotesPubliquesUp
mkdir /tmp/patches
git filter-repo --path cours --path informatique --path sciences --path réflexions --path '_INDEX PRINCIPAL.md'
git log
# trouver le hash correspondant au commit du dernier hash du fichier Version.md
export HASH_BASE=HASH_DU_PREMIER_COMMIT_A_PORTER
git format-patch -o /tmp/patches --root $HASH_BASE..HEAD
cd ../NotesPubliques
git am /tmp/patches/
```

La branche `Notes` de ce dépôt correspond à la dernière copie de mes notes privées, tandis que la branche `master` contient des modifications spécifiques au dépôt public, comme ce README.md.

Bien sûr, l'historique des `commits` permet de voir ce qui a été fait sur la version privée, mais c'est le meilleur compromis que j'ai trouvé pour l'instant.
