#Expérimentation #Informatique 
Cours: Udemi 
links: https://www.udemy.com/course/enterprise-oauth-for-developers/learn/lecture/24162340#overview
Survol en français :
https://www.youtube.com/watch?v=IzZF7WdBp4o&ab_channel=PoitouJug
https://www.youtube.com/watch?v=PhQJKKrV5i0


## Remarques
Ne pas confondre Authentification (Qui est le membre) et Autorisation (Quels sont les groupes et les permissions du membre).

OAuth est un framework en version 2 (Protocole en version 1) qui permet la délégation d'autorisation mais n'est pas conçu pour l'authentification.

OpenID Connect est utilisé pour l'authentification.

Authorization Code Flow : Le client se connecte au AS et lui fournit l'URL à laquelle il souhaite accéder. L'AS vérifie l'identité et s'assure qu'il fait confiance à l'adresse de redirection (enregistrement préalable des serveurs de ressources) et effectue la redirection en passant un code d'authentification qui n'est valable que pour une courte période. Ce code dans l'URL de redirection permettra de récupérer le token via un POST (ainsi, on évite le problème du token dans l'URL qui est dans les historiques et les logs). Cependant, pour vérifier qu'il n'y a pas eu d'interception (man-in-the-middle) entre le GET et le POST, il y a signature avec PKCE (pixi).

Authorization code + PKCE est l'OAuth 2.1, recommandation de sécurité en 2021.

Pour des raisons de sécurité et même si OAuth le permet, ne pas utiliser les * pour les URL et les paramètres de redirection.

L'IETF propose des bibliothèques standard pour l'implémentation de OAuth 2.1 dans tous les langages.

Les clés ne sont pas uniques, on a un jeu de clés et le token possède un champ "kid" qui indique quelle clé utiliser. Il y a un Keys End Point qui permet de récupérer les jeux de clés.

Attention, comme le JWT contient les infos client, si celui-ci les met à jour sur l'OIDC, alors celles du JWT ne sont plus à jour jusqu'à l'expiration du token.

Il y a deux types de tokens entre le client, l'Authorization Server et le Resource Server : Bearer token (jeton au porteur) et JWT token. Le Bearer token est juste une chaîne qui doit être protégée, et le Resource Server contacte l'Authorization Server pour vérifier le Bearer token. Dans le cas du JWT token, le token contient les données du scope. Il est signé par l'Authorization Server avec une clé privée, le Resource Server doit donc vérifier la validité du token en le vérifiant avec la clé publique. La clé publique est accessible sur un endpoint du Serveur d'Authorization. Le JWT token est un JSON encodé, une fois décodé on y trouve une entrée "exp" qui est la date d'expiration du token, l'entrée "scp" est le scope (seulement le nom des clés, pas les valeurs), l'entrée "sub" représente l'utilisateur ayant fait la demande.

Dans OAuth, PKCE (prononcer pixi) Proof Key for Code Exchange remplace les grants pour ne pas passer par le navigateur, il n'y a plus de credentials.

Vocabulaire : en OAuth, on parle de token, tandis qu'en OpenID, on parle d'ID_token.

Les utilisateurs sont enregistrés sur l'OpenID Connect avec le Dynamic Registration End Point, qui permet la création, la consultation et la modification.

Le Discovery End Point permet de découvrir les données de configuration, les endpoints, les scopes supportés, les algorithmes de chiffrement et les signatures.

Remarques : Google cherche à décentraliser la gestion des autorisations (ACL), voir le projet Google's Zanzibar.

Les SPA (Single Page Applications) gèrent des cookies qui leur permettent, lors de la redirection, de renouveler le token sans donner un token refresh et de rediriger vers l'application sans redemander les credentials.

Voir recommandations OAuth 2 OpenID Connect de l'IETF.


links :
https://www.ietf.org/archive/id/draft-ietf-oauth-security-topics-17.html

https://www.ssi.gouv.fr/uploads/2020/09/anssi-guide-recommandations_pour_la_securisation_de_la_mise_en_oeuvre_du_protocole_openid_connect-v1.0.pdf

