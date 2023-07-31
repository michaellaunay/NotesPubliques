La solution consistant à téléverser ou transmettre par mail des numérisations de pièce d'identité n'est pas très satisfaisante, car elle envoie en clair par messagerie une pièce d'identité aux membres vérificateurs.
Ce n'est pas un processus de validation qui respecte la confidentialité à la fois de la pièce d'identité mais si elle se fait par mail en plus elle permet au candidat de connaître l'adresse mail de membres certifiés sans qu'on sache encore qui est cette personne.
Il y a donc un risque que le candidat puisse faire pression sur les vérificateurs.
Un vérificateur pourra nous reprocher d'avoir diffusé son adresse.

Pour régler cela nous avons plusieurs solutions :

- utiliser des proxy mail pour les adresses des vérificateurs et un chiffrement de la pièce par le candidat.

- chiffrer la pièce d'identité côté navigateur du candidat avec les clés publiques associés au compte des vérificateurs et déposer les documents chiffrés sur le portail (une variante consiste à utiliser des mécanismes d"enveloppes chiffrés contenant la clé de déchiffrement du document).

Il existe déjà des bibliothèques pour faire cela qui permettent de chiffrer des documents côté navigateur avant de les déposer sur un serveur web. Ces solutions utilisent généralement des bibliothèques de chiffrement en JavaScript pour effectuer le chiffrement localement, garantissant ainsi que les données restent sécurisées avant d'être envoyées au serveur. Plusieurs bibliothèques populaires permettent de chiffrer des documents côté navigateur :

1. CryptoJS : CryptoJS est une bibliothèque JavaScript de chiffrement qui prend en charge différents algorithmes de chiffrement, tels que AES, DES, Triple DES, etc. Vous pouvez l'utiliser pour chiffrer vos données avant de les envoyer au serveur.

Site web : [https://cryptojs.gitbook.io/docs/](https://cryptojs.gitbook.io/docs/)

2. sjcl (Stanford JavaScript Crypto Library) : C'est une bibliothèque JavaScript de chiffrement développée par Stanford University. Elle propose des implémentations de plusieurs algorithmes de chiffrement et de hachage.

Site web : [https://bitwiseshiftleft.github.io/sjcl/](https://bitwiseshiftleft.github.io/sjcl/)

3. Forge : Forge est une bibliothèque JavaScript complète qui prend en charge le chiffrement, le hachage, les signatures numériques, etc. Elle est souvent utilisée pour les tâches de chiffrement côté navigateur.

Site web : [https://github.com/digitalbazaar/forge](https://github.com/digitalbazaar/forge)

4. WebCrypto API : L'API WebCrypto est une spécification du World Wide Web Consortium (W3C) qui fournit une interface native pour le chiffrement en JavaScript. Elle permet d'accéder aux fonctionnalités de chiffrement directement via le navigateur sans avoir besoin de bibliothèques tierces.

Documentation : [https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API)

Avec ce système, on applique ce que l'on appelle un chiffrement de bout en bout.