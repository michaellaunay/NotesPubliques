TAL signifie Template Attribute Language, tandis que METAL signifie Macro Expansion TAL. Ensemble, ils offrent des fonctionnalités de réutilisation du code et de logique conditionnelle dans les templates.

# TAL : Template Attribute Language

TAL offre des attributs spéciaux que vous pouvez utiliser dans votre markup HTML pour ajouter de la logique de template. Voici quelques exemples :

- `tal:condition` : Il permet d'inclure ou d'exclure un élément en fonction d'une condition. Par exemple :

    ```html
    <p tal:condition="python: user.is_authenticated">
      Bienvenue, ${user.name}!
    </p>
    ```
    
    Dans cet exemple, le paragraphe n'est rendu que si l'utilisateur est authentifié.
    
- `tal:repeat` : Il permet de créer des boucles. Par exemple :

    ```html
    <ul>
      <li tal:repeat="item items">${item}</li>
    </ul>
    ```
    
    Dans cet exemple, un élément de liste est créé pour chaque élément dans la liste `items`.

# METAL : Macro Expansion TAL

METAL permet la réutilisation du code à travers les templates en utilisant des macros. Une macro est une partie de template que vous pouvez définir une fois et réutiliser à plusieurs endroits.

Par exemple, nous pouvons créer une macro pour un en-tête de site web qui est partagé entre plusieurs pages :

```html
<!-- Dans header.pt -->
<div metal:define-macro="site_header">
  <h1>Mon Site Web</h1>
  <nav>
    <a href="/">Accueil</a> | <a href="/about">À Propos</a>
  </nav>
</div>
```

Ensuite, vous pouvez utiliser cette macro dans un autre template :

```html
<!-- Dans index.pt -->
<html>
  <body>
    <div metal:use-macro="load: header.pt/site_header"></div>
    <p>Bienvenue sur mon site web!</p>
  </body>
</html>
```

Dans cet exemple, `load: header.pt/site_header` charge la macro `site_header` à partir du template `header.pt`.