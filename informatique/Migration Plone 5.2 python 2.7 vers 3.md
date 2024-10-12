Traduction en français de https://community.plone.org/t/best-practice-documentation-on-zodb-debugging/12778

### Identifier ce qui est cassé  
**Vérifiez l'intégralité de votre base de données**

Utilisez l'outil **zodbverify** (<https://github.com/plone/zodbverify>) pour vérifier une base de données ZODB en itérant et en chargeant tous les enregistrements. **zodbverify** est disponible sous forme de script autonome et comme add-on pour *plone.recipe.zope2instance*. Utilisez la version la plus récente !

Exécution basique :

```bash
$ bin/zodbverify -f var/filestorage/Data.fs
```

Ce qui vous retournera :
- une liste des types d'erreurs
- le nombre d'occurrences
- tous les *oid* (identifiants d'objets) qui provoquent ces erreurs lors du chargement

**zodbverify** est disponible uniquement pour Plone 5.2 et versions ultérieures. Pour les versions plus anciennes de Plone, utilisez les scripts `fstest.py` et `fsrefs.py` du package ZODB :

```bash
$ ./bin/zopepy ./parts/packages/ZODB/scripts/fstest.py var/filestorage/Data.fs
$ ./bin/zopepy ./parts/packages/ZODB/scripts/fsrefs.py var/filestorage/Data.fs
```

L'exemple de sortie de **zodbverify** pourrait ressembler à ceci, tiré d'un intranet de taille moyenne (1 Go pour Data.fs, 5 Go pour blobstorage) qui a commencé avec Plone 4 sur Archetypes et a migré vers Plone 5.2 sur Python 3 et Dexterity :

```bash
$ ./bin/zodbverify -f var/filestorage/Data.fs

[...]

INFO:zodbverify:Terminé ! 163 955 enregistrements scannés.  
Trouvé 1 886 enregistrements qui ne pouvaient pas être chargés.  
Exceptions, fréquence d'occurrence et *oid* affectés :

ModuleNotFoundError: No module named 'Products.Archetypes': 1 487  
0x0e00eb 0x0e00ee 0x0e00ef [...]

ModuleNotFoundError: No module named 'Products.ResourceRegistries': 1  
0x3b1311
```

Vous verrez tous les différents types d'erreurs et les objets qui les provoquent, référencés par leur *oid* dans ZODB. Consultez l'annexe pour savoir comment traiter les *oid*.

### Examiner un objet cassé

Dans cet exemple, l'objet avec l'oid `0x376b77` semble être une instance de **FormThanksPage** provenant de **Products.PloneFormGen**. Mais attendez ! Vous avez supprimé tous ces objets, alors où se trouve-t-il sur le site ?

Si l'objet incriminé est un contenu normal, la solution est souvent simple : vous pouvez appeler `obj.getPhysicalPath()` pour savoir où il se trouve. Dans de nombreux cas, éditer et sauvegarder l'objet corrigera le problème. Dans d'autres cas, vous devrez peut-être copier le contenu vers un nouvel élément et supprimer l'objet cassé.

Cependant, il est souvent question d'un autre type d'objet, comme :
- Une annotation sur un objet ou sur le portail
- Une valeur de relation dans le catalogue de relations
- Un élément dans le catalogue IntId
- Une ancienne révision de contenu dans CMFEditions
- Une entrée de configuration dans **portal_properties** ou **portal_registry**

La partie la plus difficile est de déterminer **ce qu'est l'objet cassé** et **où il se trouve** avant de pouvoir le supprimer ou le corriger.

### Inspecter un objet

Pour examiner l'objet, passez l'*oid* et l'indicateur `-D` à zodbverify comme ceci :

```bash
$ ./bin/zodbverify -f var/filestorage/Data.fs -o 0x376b77 -D
```

L'outil affichera des informations telles que :

```bash
INFO:zodbverify:Inspection de 0x376b77 :  
<persistent broken Products.PloneFormGen.content.thanksPage.FormThanksPage instance b'\x00\x00\x00\x00\x007kw'>
```

Cela vous permet de savoir qu'il s'agit d'une instance de classe "cassée" (**persistent broken**), un mécanisme de ZODB qui vous donne accès aux objets dont la classe ne peut plus être importée.

Vous pouvez maintenant inspecter l'objet, par exemple :

```bash
(Pdb++) obj
<persistent broken Products.PloneFormGen.content.thanksPage.FormThanksPage instance b'\x00\x00\x00\x00\x007kw'>
```

En inspectant son dictionnaire interne, vous pourriez voir des détails sur l'état de l'objet. Vous pouvez aussi utiliser le débogueur pour examiner en profondeur les données stockées et tenter de réparer l'objet.

### Inspecter le chemin des références

Maintenant que vous savez que l'objet **FormThanksPage** est cassé, vous ne savez toujours pas où il se trouve réellement dans la base de données.

**zodbverify** peut vous aider à trouver ce chemin en construisant un arbre de références des objets dans la ZODB :

```bash
INFO:zodbverify:Construction d'un arbre de références pour ZODB...
[...]
INFO:zodbverify:Création d'un dictionnaire de références pour 163955 objets.
INFO:zodbverify:
Cet oid est référencé par :
INFO:zodbverify:0x376ada BTrees.IOBTree.IOBucket au niveau 1
INFO:zodbverify:0x28018c BTrees.IOBTree.IOBTree au niveau 2
INFO:zodbverify:0x280184 five.intid.intid.IntIds au niveau 3
INFO:zodbverify:0x1e five.localsitemanager.registry.PersistentComponents au niveau 4
INFO:zodbverify:0x11 Products.CMFPlone.Portal.PloneSite au niveau 5
INFO:zodbverify:0x01 OFS.Application.Application au niveau 6
INFO:zodbverify: 8< --------------- >8 Arrêt aux objets racine
```

Vous pouvez voir que **FormThanksPage** se trouve dans un **IOBucket**, qui est dans un **IOBTree**, lui-même dans un objet de la classe **five.intid.intid.IntIds**, qui fait partie du registre des composants de votre site Plone.

Cela signifie qu'il existe une référence à un objet cassé dans l'outil **IntId**. Pour résoudre ce problème, vous devrez supprimer les références aux objets cassés de cet outil. (La manière de traiter ces références est couverte plus loin dans la section "Coupables fréquents".)

---

### Décider de la manière de résoudre le problème

Dans ce cas précis, la solution est claire : il faut supprimer les références aux objets cassés dans l'outil **IntId**. Cependant, il existe plusieurs approches pour résoudre ces problèmes, selon la situation. Voici quelques options :

---

#### Option 1 : Ignorer les erreurs

Il est possible d'ignorer ces erreurs, surtout dans le cas de bases de données anciennes migrées depuis Plone 2 ou 3. Si ces erreurs ne se manifestent jamais lors de l'utilisation du site et que le client n'a ni le budget ni l'intérêt de les corriger, vous pouvez les laisser telles quelles. Si elles n'ont aucun impact (par exemple, si vous pouvez toujours compacter la base de données ou si aucune fonctionnalité ne tombe en panne), il est parfois plus simple de les ignorer.

---

#### Option 2 : Migrer/Modifier une base de données avec **zodbupdate**

Utilisez cette option lorsque des modules ou des classes ont été déplacés ou renommés.

Documentation : [zodbupdate](https://github.com/zopefoundation/zodbupdate)

Cet outil vous permet de changer des objets dans la base de données en fonction de certaines règles :

- Si un import a changé de place, vous pouvez utiliser un *mapping* de renommage.
- Pour spécifier si un objet doit être décodé, utilisez un *mapping* de décodage.

**Exemple de règles de décodage** tirées de Zope/src/OFS/__init__.py :

```python
zodbupdate_decode_dict = {
    'OFS.Image File data': 'binary',
    'OFS.Image Image data': 'binary',
    'OFS.Application Application title': 'utf-8',
    'OFS.DTMLDocument DTMLDocument title': 'utf-8',
    'OFS.userfolder UserFolder title': 'utf-8',
}
```

**Exemple de règles de renommage** :

```python
zodbupdate_rename_dict = {
    'webdav.LockItem LockItem': 'OFS.LockItem LockItem',
}
```

Vous pouvez définir vos propres mappings dans vos packages et les enregistrer dans le fichier `setup.py` pour que **zodbupdate** les prenne en compte lors de l'exécution.

- **Exemple de mapping de renommage** : [Zope commit](https://github.com/zopefoundation/Zope/commit/f677ed7)
- **Exemple de mapping de décodage** : [ZopeVersionControl commit](https://github.com/zopefoundation/Products.ZopeVersionControl/commit/138cf39)


### Option 3 : Contourner avec un patch

Il est possible d'injecter un module pour contourner les classes ou modules manquants ou déplacés. 

L'intérêt de cette approche est que cela vous permet de supprimer les objets concernés de manière sécurisée par la suite, sans affecter les performances.

Voici un exemple de contournement dans un fichier `__init__.py` :

```python
# -*- coding: utf-8 -*-
from OFS.SimpleItem import SimpleItem
from plone.app.upgrade.utils import alias_module
from plone.app.upgrade import bbb
from zope.interface import Interface

class IBBB(Interface):
    pass

class BBB(object):
    pass

SlideshowDescriptor = SimpleItem

# Interfaces
try:
    from collective.z3cform.widgets.interfaces import ILayer
except ImportError:
    alias_module('collective.z3cform.widgets.interfaces.ILayer', IDummy)

[...]

# SimpleItem
try:
    from collective.easyslideshow.descriptors import SlideshowDescriptor
except ImportError:
    alias_module('collective.easyslideshow.descriptors.SlideshowDescriptor', SlideshowDescriptor)

try:
    from Products.CMFPlone import UndoTool
except ImportError:
    sys.modules['Products.CMFPlone.UndoTool'] = bbb
```

En utilisant `alias_module` ou en modifiant directement le dictionnaire des modules, vous pouvez contourner des classes ou modules manquants, permettant ainsi de supprimer ou traiter ces objets plus tard.

Plus d'exemples :  
- [collective.migrationhelpers/patches.py](https://github.com/collective/collective.migrationhelpers/blob/master/src/collective/migrationhelpers/patches.py)
- Plone utilise fréquemment cette technique, voir par exemple : [plone.app.upgrade](https://github.com/plone/plone.app.upgrade/blob/master/plone/app/upgrade/__init__.py)

---

### Option 4 : Remplacer les objets cassés par des objets factices (dummies)

Si un objet est manquant (ex. : vous obtenez une erreur **POSKeyError**) ou s'il est trop corrompu pour être réparé, vous pouvez choisir de le remplacer par un objet factice.

Voici un exemple d'implémentation :

```python
from persistent import Persistent
from ZODB.utils import p64
import transaction

app = self.context.__parent__
broken_oids = [0x2c0ab6, 0x2c0ab8]

for oid in broken_oids:
    dummy = Persistent()
    dummy._p_oid = p64(oid)
    dummy._p_jar = app._p_jar
    app._p_jar._register(dummy)
    app._p_jar._added[dummy._p_oid] = dummy
transaction.commit()
```

**Attention :** L'objet manquant ou cassé sera définitivement supprimé après cette opération. Avant d'opter pour cette solution, assurez-vous d'avoir identifié clairement ce qu'était l'objet en question.

---

### Option 5 : Supprimer les objets cassés de la base de données

Vous pouvez également choisir de supprimer les objets cassés directement de la base de données :

```python
from persistent import Persistent
from ZODB.utils import p64
import transaction

app = self.context.__parent__
broken_oids = [0x2c0ab6, 0x2c0ab8]

for oid in broken_oids:
    root = connection.root()
    del app._p_jar[p64(oid)]
transaction.commit()
```

Cependant, cette approche présente des risques. En supprimant l'objet, vous enlevez son pickle mais pas nécessairement toutes les références qui y sont associées. Cela peut entraîner des erreurs **POSKeyError** par la suite. Il est donc recommandé d'utiliser cette méthode avec précaution.

### Option 6 : Réparation manuelle

C'est généralement la meilleure méthode pour résoudre la majorité des problèmes.

#### Étapes à suivre :

1. Utilisez **zodbverify** pour identifier tous les objets cassés.
2. Concentrez-vous sur un type d'erreur à la fois.
3. Utilisez **zodbverify** avec l'option `-o <OID> -D` pour inspecter un objet et déterminer où cet objet est référencé.
4. Si nécessaire, utilisez le script **fsoids.py** pour suivre les références jusqu'à ce que vous trouviez où se situe l'objet dans l'arbre des objets. **zodbverify** essaiera de faire cela pour vous.
5. Supprimez ou corrigez l'objet (en utilisant une étape de mise à jour, un débogueur comme **pdb** ou un mapping de renommage).

#### Identification des objets cassés

La version la plus récente de **zodbverify** possède une fonctionnalité qui automatise cette tâche (discutée dans l'exemple 1). En attendant que cette fonctionnalité soit intégrée, vous devez utiliser la branche `show_references` du pull-request : <https://github.com/plone/zodbverify/pull/8>.

Lorsqu'il inspecte un *oid*, **zodbverify** crée un dictionnaire de toutes les références pour une recherche inversée. Ensuite, il suit récursivement le chemin des références jusqu'aux objets de référence, jusqu'à la racine. Pour éviter les entrées redondantes ou sans importance, **zodbverify** arrête l'analyse après 600 niveaux et à certains objets racine, car ceux-ci ont souvent de nombreuses références qui pourraient encombrer les résultats.

Le résultat vous donnera une idée claire de l'emplacement de l'objet cassé dans l'arborescence des objets, et vous indiquera comment y accéder et le corriger.

#### Exemple

Si l'*oid* `0x3b1d06` est cassé, vous pouvez l'inspecter avec **zodbverify** :

```bash
$ ./bin/instance zodbverify -o 0x3b1d06 -D
```

Voici un extrait de la sortie typique que vous pourriez obtenir :

```bash
2020-08-24 12:19:32,441 INFO    [Zope:45][MainThread] Prêt à gérer les requêtes
2020-08-24 12:19:32,442 INFO    [zodbverify:222][MainThread]
L'objet est 'obj'
L'instance Zope est 'app'
[4] > /Users/pbauer/workspace/dipf-intranet/src-mrd/zodbverify/src/zodbverify/verify_oid.py(230)verify_oid()
-> pickle, state = storage.load(oid)

(Pdb++) obj
<BTrees.OIBTree.OITreeSet object at 0x110b97ac0 oid 0x3b1d06 in <Connection at 10c524040>>
```

Dans cet exemple, l'objet est un **BTrees.OIBTree.OITreeSet**, mais il n'a pas d'attribut `__parent__`, ce qui rend difficile de savoir avec certitude de quoi il s'agit exactement.

Si vous continuez (`c` pour **continue**), **zodbverify** essaiera de charger le pickle et fournira plus d'informations sur les erreurs rencontrées :

```bash
2020-08-24 12:20:50,784 INFO    [zodbverify:68][MainThread]
Impossible de traiter <class 'BTrees.OIBTree.OITreeSet'> enregistrement 0x3b1d06 (b'\x00\x00\x00\x00\x00;\x1d\x06'):
2020-08-24 12:20:50,784 INFO    [zodbverify:69][MainThread] b'\x80\x03cBTrees.OIBTree\nOITreeSet\n[...]'
```

Ensuite, **zodbverify** construira un arbre de références pour cet objet :

```bash
2020-08-24 12:22:42,596 INFO    [zodbverify:234][MainThread] ModuleNotFoundError: No module named 'webdav.interfaces'; 'webdav' is not a package: 0x3b1d06
2020-08-24 12:22:42,597 INFO    [zodbverify:43][MainThread] Construction d'un arbre de références de ZODB...
[...]
2020-08-24 12:22:49,037 INFO    [zodbverify:61][MainThread] Création d'un dictionnaire de références pour 163955 objets.

2020-08-24 12:22:49,424 INFO    [zodbverify:40][MainThread] L'oid 0x3b1d06 est référencé par :

0x3b1d06 (BTrees.OIBTree.OITreeSet) est référencé par 0x3b1d01 (BTrees.OOBTree.OOBucket) au niveau 1
0x3b1d01 (BTrees.OOBTree.OOBucket) est référencé par 0x11c284 (BTrees.OOBTree.OOBTree) au niveau 2
0x11c284 (BTrees.OOBTree.OOBTree) est _reltoken_name_TO_objtokenset pour 0x11c278 (z3c.relationfield.index.RelationCatalog) au niveau 3
0x11c278 (z3c.relationfield.index.RelationCatalog) est relations pour 0x1e (five.localsitemanager.registry.PersistentComponents) au niveau 4
0x1e (five.localsitemanager.registry.PersistentComponents) est référencé par 0x11 (Products.CMFPlone.Portal.PloneSite) au niveau 5
0x11 (Products.CMFPlone.Portal.PloneSite) est Plone pour 0x01 (OFS.Application.Application) au niveau 6
8< --------------- >8 Arrêt aux objets racine
```

Grâce à cette sortie, vous pouvez constater que l'objet cassé fait partie du **RelationCatalog** de **zc.relation**. Consultez la section "Coupables fréquents" pour savoir comment traiter ces objets.

### Exemple 1 d'utilisation de **fsoids.py**

Dans cet exemple, j'utilise le script **fsoids.py** pour localiser où se trouve réellement un objet cassé, afin que je puisse le supprimer ou le corriger. Bien que **zodbverify** soit plus facile à utiliser, cette approche est discutée ici car elle était la meilleure option avant l'extension de **zodbverify** et permet de mieux comprendre le fonctionnement des références dans ZODB.

---

1. **Première étape** : Utilisation de **zodbverify** pour détecter les objets cassés.

```bash
$ ./bin/zodbverify -f var/filestorage/Data.fs

INFO:zodbverify:Terminé ! 120 797 enregistrements scannés.
Trouvé 116 enregistrements qui ne pouvaient pas être chargés.
Exceptions et fréquence d'occurrence :
AttributeError: Cannot find dynamic object factory for module plone.dexterity.schema.generated: 20
AttributeError: module 'plone.app.event.interfaces' has no attribute 'IEventSettings': 3
ModuleNotFoundError: No module named 'Products.ATContentTypes': 4
ModuleNotFoundError: No module named 'Products.Archetypes': 5
ModuleNotFoundError: No module named 'Products.CMFDefault': 20
[...]
ModuleNotFoundError: No module named 'webdav.EtagSupport'; 'webdav' is not a package: 16
```

Nous avons identifié plusieurs erreurs, dont celles concernant des modules manquants, comme **Products.Archetypes** ou **webdav.EtagSupport**. Il faut maintenant suivre les références pour localiser les objets cassés.

---

2. **Suivre les références avec **fsoids.py**

Commencez par suivre l'objet cassé, dans ce cas, `0x35907d` :

```bash
./bin/zopepy ./parts/packages/ZODB/scripts/fsoids.py var/filestorage/Data.fs 0x35907d

oid 0x35907d BTrees.OIBTree.OISet 1 révision
    tid 0x03c425bfb4d8dcaa offset=282340 2017-12-15 10:07:42.386043
        tid user=b'Plone xxx@xxx.de'
        tid description=b'/Plone/it-service/hilfestellungen-anleitungen-faq/outlook/content-checkout'
        nouvelle révision BTrees.OIBTree.OISet à 282469
    tid 0x03d3e83a045dd700 offset=421126 2019-11-19 15:54:01.023413
        tid user=b''
        tid description=b''
        référencé par 0x35907b BTrees.OIBTree.OITreeSet à 911946038
```

Nous voyons ici qu'il est référencé par l'oid `0x35907b`. Suivons cette référence.

---

3. **Continuer à suivre les références**

Suivons maintenant l'oid `0x35907b` :

```bash
./bin/zopepy ./parts/packages/ZODB/scripts/fsoids.py var/filestorage/Data.fs 0x35907b

[...]
référencé par 0x3c5790 BTrees.OOBTree.OOBucket
```

Ensuite, suivez l'oid `0x3c5790` :

```bash
./bin/zopepy ./parts/packages/ZODB/scripts/fsoids.py var/filestorage/Data.fs 0x3c5790

[...]
référencé par 0x11c284 BTrees.OOBTree.OOBTree
```

Puis, suivez l'oid `0x11c284` :

```bash
./bin/zopepy ./parts/packages/ZODB/scripts/fsoids.py var/filestorage/Data.fs 0x11c284

[...]
référencé par 0x3d0bd6 BTrees.OOBTree.OOBucket
```

Enfin, suivez l'oid `0x3d0bd6` :

```bash
./bin/zopepy ./parts/packages/ZODB/scripts/fsoids.py var/filestorage/Data.fs 0x3d0bd6

[...]
référencé par 0x11c278 z3c.relationfield.index.RelationCatalog
```

---

**Trouvé !** L'objet cassé est finalement situé dans le **RelationCatalog** de **z3c.relationfield.index**.

### Exemple 2 d'utilisation de **fsoids.py**

Dans cet exemple, **zodbverify** a détecté une trace de **Products.PloneFormGen**, même si vous pensez avoir désinstallé correctement l'addon (par exemple, en utilisant [collective.migrationhelpers](https://github.com/collective/collective.migrationhelpers/blob/master/src/collective/migrationhelpers/addons.py#L11)).

Nous allons maintenant suivre la piste des objets qui font référence à cet élément dans l'arborescence pour le localiser et le corriger.

#### Étape 1 : Suivi des références de l'objet cassé

```bash
./bin/zopepy ./parts/packages/ZODB/scripts/fsoids.py var/filestorage/Data.fs 0x372d00
oid 0x372d00 Products.PloneFormGen.content.thanksPage.FormThanksPage 1 révision
    tid 0x03d3e83a045dd700 offset=421126 2019-11-19 15:54:01.023413
        tid user=b''
        tid description=b''
        nouvelle révision Products.PloneFormGen.content.thanksPage.FormThanksPage à 912841984
        référencé par 0x372f26 BTrees.OIBTree.OIBucket à 912930339
        références 0x372e59 Products.Archetypes.BaseUnit.BaseUnit à 912841984
        références 0x372e5a Products.Archetypes.BaseUnit.BaseUnit à 912841984
        [...]
    tid 0x03d40a3e52a41633 offset=921078960 2019-11-25 17:02:19.368976
        tid user=b'Plone pbauer'
        tid description=b'/Plone/rename_file_ids'
        référencé par 0x2c1b51 BTrees.IOBTree.IOBucket à 921653012
```

Nous voyons ici que cet objet est référencé par l'oid `0x2c1b51`. Poursuivons notre enquête.

#### Étape 2 : Suivre les autres références

Ensuite, examinons l'oid `0x2c1b51` :

```bash
./bin/zopepy ./parts/packages/ZODB/scripts/fsoids.py var/filestorage/Data.fs 0x2c1b51
oid 0x2c1b51 BTrees.IOBTree.IOBucket 1 révision
    [...]
```

Nous sautons quelques étapes similaires jusqu'à trouver l'oid `0x280184`, qui est le catalogue **IntIds** :

```bash
./bin/zopepy ./parts/packages/ZODB/scripts/fsoids.py var/filestorage/Data.fs 0x280184
oid 0x280184 five.intid.intid.IntIds 1 révision
    tid 0x03d3e83a045dd700 offset=421126 2019-11-19 15:54:01.023413
        tid user=b''
        tid description=b''
        nouvelle révision five.intid.intid.IntIds à 8579054
        références 0x28018c <inconnu> à 8579054
        références 0x28018d <inconnu> à 8579054
    tid 0x03d3e90c4d3aed55 offset=915868610 2019-11-19 19:24:18.100824
        tid user=b' adminstarzel'
        tid description=b'/Plone/portal_quickinstaller/installProducts'
```

Ceci est le catalogue **IntId** de **zope.intid**. Le problème est similaire à celui du catalogue de relations (**zc.relation**), où des références à des objets cassés peuvent rester dans le catalogue et doivent être supprimées manuellement.

#### Étape 3 : Suppression manuelle des objets cassés dans une session **pdb**

Voici un exemple de code pour supprimer tous les objets cassés du catalogue **IntIds** dans une session **pdb** :

```python
(Pdb++) from zope.intid.interfaces import IIntIds
(Pdb++) from zope.component import getUtility
(Pdb++) intid = getUtility(IIntIds)
(Pdb++) broken_keys = [i for i in intid.ids if 'broken' in repr(i.object)]
(Pdb++) for broken_key in broken_keys: intid.unregister(broken_key)
(Pdb++)
(Pdb++) import transaction
(Pdb++) transaction.commit()
```

Après avoir compacté la base de données, le problème devrait être résolu. 🎉

---

### Autres options pour l'inspection de ZODB

- **zodbbrowser** : Un outil Zope3 pour naviguer dans une base de données ZODB via un navigateur. Cependant, il peut être difficile à faire fonctionner avec une base de données ZODB Plone.
- **zc.zodbdgc** : Cet outil valide les bases de données distribuées en partant de la racine et en vérifiant que tous les objets référencés sont accessibles.
- **collective.zodbdebug** : Un excellent outil pour créer et inspecter les cartes de références et rétro-références d'une base de données ZODB. Cependant, il n'est pas encore compatible avec Python 3. Certaines fonctionnalités de **zodbverify** couvrent aussi ce domaine.

---

### Coupables fréquents : **IntIds** et **Relations**

Les outils **IntId** et les catalogues de relations sont souvent à l'origine des problèmes, surtout si vous avez migré d'**Archetypes** vers **Dexterity**.

Il peut y avoir de nombreuses valeurs de relations (**RelationValues**) dans ces outils, qui font encore référence à des objets supprimés et non chargés correctement. Vous pouvez utiliser le code ci-dessous pour nettoyer le catalogue **IntId** et les relations tout en maintenant les relations intactes :

```python
from collective.relationhelpers.api import cleanup_intids
from collective.relationhelpers.api import purge_relations
from collective.relationhelpers.api import restore_relations
from collective.relationhelpers.api import store_relations

def remove_relations(context=None):
    # stocker toutes les relations dans une annotation sur le portail
    store_relations()
    # vider le catalogue des relations
    purge_relations()
    # supprimer toutes les valeurs de relations et références aux objets cassés dans IntId
    cleanup_intids()
    # recréer toutes les relations à partir de l'annotation sur le portail
    restore_relations()
```

Pour plus de détails, consultez le code source : [collective.relationhelpers](https://github.com/collective/collective.relationhelpers/blob/master/src/collective/relationhelpers/api.py).

---

### Annotations

De nombreux addons et fonctionnalités de Plone stockent des données dans des annotations sur le portail ou sur du contenu. Il est conseillé de vérifier les annotations après une migration pour voir si certaines peuvent être supprimées en toute sécurité.

Exemple : **wicked** (une fonctionnalité maintenant supprimée d'édition de style wiki) stockait ses paramètres dans une annotation.

```python
def cleanup_wicked_annotation(context=None):
    ann = IAnnotations(portal)
    if 'plone.app.controlpanel.wicked' in ann:
        del ann['plone.app.controlpanel.wicked']
```

Autre exemple : des fichiers provenant de téléchargements échoués stockés par **plone.formwidget.namedfile** dans une annotation.

```python
def cleanup_upload_annotation(context=None):
    # supprimer les traces des téléchargements avortés
    ann = IAnnotations(portal)
    if ann.get('file_upload_map', None) is not None:
        for uuid in ann['file_upload_map']:
            del ann['file_upload_map'][uuid]
```

### TODO

à finir et merger avec  [https://github.com/plone/zodbverify/pull/8 9](https://github.com/plone/zodbverify/pull/8)

### Annexe : Migration d'une ZODB de Python 2 à Python 3

Étant donné que les utilisateurs rencontrent souvent des problèmes après la migration de leur ZODB, voici un aperçu rapide de la migration d'une base de données ZODB de Python 2 vers Python 3.

#### Étapes de la migration

1. **Exécuter zodbupdate avec l'option de conversion vers Python 3** :
   Utilisez le script **zodbupdate** en Python 3 avec le paramètre `--convert-py3`.

   ```bash
   $ ./bin/zodbupdate --convert-py3
   ```

2. **Passer les paramètres de localisation de la base de données, l'encodage par défaut (utf8) et un encodage de secours (latin1)** pour les éléments où le décodage en utf8 échoue.

   Exemple :

   ```bash
   $ ./bin/zodbupdate --convert-py3 --file=var/filestorage/Data.fs --encoding=utf8 --encoding-fallback=latin1
   ```

   Exemple de sortie :

   ```bash
   Mise à jour du marqueur magique pour var/filestorage/Data.fs
   Ignorer l'index pour /Users/pbauer/workspace/projectx/var/filestorage/Data.fs
   Chargement de 2 règles de décodage depuis AccessControl:decodes
   Chargement de 12 règles de décodage depuis OFS:decodes
   [...]
   Validation des modifications (n°1).
   ```

   Après cela, votre ZODB devrait être utilisable en Python 3.

---

#### Processus de migration résumé

1. **D'abord, exécutez `zodbupdate` en Python 2** sans utiliser l'option de conversion vers Python 3 pour détecter et appliquer plusieurs règles de renommage explicites et implicites.

   ```bash
   $ ./bin/zodbupdate -f var/filestorage/Data.fs
   ```

2. **Exécutez `zodbverify`** pour vérifier la base de données. Si vous rencontrez encore des avertissements ou des exceptions, vous devrez peut-être définir plus de règles et les appliquer avec `zodbupdate`. Vous pouvez néanmoins choisir de migrer vers Python 3 même si des erreurs sont signalées.

   ```bash
   $ ./bin/instance zodbverify
   ```

3. **Utilisez Python 3** et exécutez de nouveau `zodbupdate` avec l'option `--convert-py3`.

   ```bash
   $ ./bin/zodbupdate --convert-py3 --file=var/filestorage/Data.fs --encoding=utf8
   ```

4. **Pour être sûr, exécutez encore `zodbverify` avec Python 3** après la migration.

   ```bash
   $ ./bin/instance zodbverify
   ```

---

#### Manipulation des oids

Voici quelques exemples utiles pour transformer les oids entre différents formats (entier, hexadécimal, texte) :

- **Convertir un oid en entier en bytes** :

  ```python
  from ZODB.utils import p64
  oid = 0x2c0ab6
  p64(oid)
  # Résultat : b'\x00\x00\x00\x00\x00,\n\xb6'
  ```

- **Convertir un oid en bytes en représentation hexadécimale** :

  ```python
  from ZODB.utils import oid_repr
  oid = b'\x00\x00\x00\x00\x00,\n\xb6'
  oid_repr(oid)
  # Résultat : '0x2c0ab6'
  ```

- **Convertir un oid en représentation hexadécimale en bytes** :

  ```python
  from ZODB.utils import repr_to_oid
  oid = '0x2c0ab6'
  repr_to_oid(oid)
  # Résultat : b'\x00\x00\x00\x00\x00,\n\xb6'
  ```

---

#### Obtenir le chemin des blobs

Pour obtenir le chemin d'un blob en utilisant un oid :

```python
from ZODB.blob import BushyLayout
if isinstance(oid, int):
    from ZODB.utils import p64
    oid = p64(oid)
return BushyLayout.oid_to_path(None, oid)
```

---

#### Charger un objet à partir d'un oid dans une session **pdb**

Pour charger un objet à partir de son oid dans une session **pdb** :

```python
oid = 0x2c0ab6
from ZODB.utils import p64
app = self.context.__parent__
obj = app._p_jar.get(p64(oid))
```

---

### Liens utiles

De nombreuses personnes de la communauté Plone/Zope ont écrit sur ce sujet. Voici quelques ressources utiles :

- [Guide définitif pour le POSKeyError](https://plonechix.blogspot.com/2009/12/definitive-guide-to-poskeyerror.html)
- [Correction des références cassées dans ZODB](https://www.nathanvangheem.com/posts/2011/05/25/fixing-broken-zodb-object-references.html)
- [Réparation de ZODB](http://www.derstappen-it.de/tech-blog/zodb-repair)
- [Débogage et correction du POSKeyError](https://sixfeetup.com/blog/poskeyerror-debugging-and-fixing-cookbook)
- [Garbage collection multi-ZODB](http://www.zodb.org/en/latest/articles/multi-zodb-gc.html)

Ces ressources fournissent des solutions et des astuces pour déboguer et corriger les problèmes rencontrés dans ZODB, notamment lors des migrations vers Python 3.