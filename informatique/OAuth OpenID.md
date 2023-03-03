#Expérimentation #Informatique 
Cours: Udemi 
links: https://www.udemy.com/course/enterprise-oauth-for-developers/learn/lecture/24162340#overview
Survol en français :
https://www.youtube.com/watch?v=IzZF7WdBp4o&ab_channel=PoitouJug
https://www.youtube.com/watch?v=PhQJKKrV5i0


Remarques: 
Ne pas confondre Authentification (Qui est le membre) et Autorisation (Quels sont les groupes et les permissions du membre)

AOuth est un framework en version 2 (Protocole en version 1) qui permet la délégation d'autorisation mais n'est pas là pour l'authentification.

OpenID Connect sert à l'authentification.

Authorization Code Flow : 
Le client se connecte au AS lui passe l'url sur laquelle il veut des accès.
L'AS vérifie identité et vérifie s'il fait confiance à l'adresse de redirection (enregistrement préalable des serveurs de ressources) et fait la redirection en passant un code d'authent qui n'est valable que de façon courte.
Ce code dans l'url de redirection permettra de récupérer le token via un post (comme ça on a plus le problème du token dans l'url qui est dans les historiques et les log).
Par contre pour vérifier qu'il n'y a pas eu un men in the middle entre le get et le post il y a signature avec PKCE (pixi)

Authorization code + Pixi est lOAuth2.1 a recommandation de sécurité 2021 

Pour des raisons de sécurité et même si OAuth le permet ne pas utiliser les * pour les url et les paramettres de redirection.

IETF propose des biblithèques standards ppour l'implémentation de AOth2.1 dans tous les langages.

Les clés ne sont pas unique on a un jeu de clé et le token possède un champ "kid" qui dit quelle clé utiliser. Il y a un Keys End Point qui permet de récupérer les jeux de clés.

Attention comme le JWT contient les infos client si celui-ci les met à jours sur l'oidc allors celles du jwt ne sont plus à jour le temps de l'expiration du token...

Il y a deux types de tokens entre le client, Authorization Server et le Ressource Server : Baerer token (jeton au porteur) et jwt token.
Le Baerer token est juste une chaine qui doit être protégée, et le Ressource Server contacte le Authorization Server pour vérifier le Baerer token. Dans le cas du jwt token, le token contient les données du scope. Il est signé par l'Authorization server avec une clé privée, le Ressource server doit donc vérifier la validité du token en le vérifiant avec la clé publique. La clé publique est accessible sur un endpoint du Serveur d'Authorization.
Le jwt token est un json encodé, une fois décodé on y trouve une entré "exp" qui est la date d'expiration du token, l'entrée "scp" est le scope (seulement le nom des clés pas les valeurs), l'entrée "sub" représente l'utilisateur ayant fait la demande



Dans OAuth ,PKCE (prononcer pixi) Proof Key for code exchange qui remplace les grants pour ne pas passer par le navigateur, il n'y a plus de crédential 

Vocab : en OAuth on par de token en OpenId on parle ID_token

On enregistre les utilisateurs sur l'OpenIDConnect avec le Dynamic Registration End Point qui permet la création la consultation et la modification.

Discovery EndPoint permet de découvrire les données de configuration, les endpoints, les scopes supportés, les algo de chiffrement et les signatures

Remarques :
Google cherche à décentralisation la gestion des authorisations (ACL) voir le projet Google's Zanzibar

Les SPA gère des Cookies qui leur permettent sur redirection de renouveller le token sans donner un token refresh et de rediriger vers l'appli sans redemander les credentials.

Voir recommandations OAuth 2 OpenId Connect de l'IETF

links :
https://www.ietf.org/archive/id/draft-ietf-oauth-security-topics-17.html

https://www.ssi.gouv.fr/uploads/2020/09/anssi-guide-recommandations_pour_la_securisation_de_la_mise_en_oeuvre_du_protocole_openid_connect-v1.0.pdf