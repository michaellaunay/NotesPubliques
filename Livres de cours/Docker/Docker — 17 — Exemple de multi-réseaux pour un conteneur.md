---
schema_version: 1
uid: 01M1BQ62DXH2Y6P4Y7RDRNS96E
titre: "Docker — 17 — Exemple de multi-réseaux pour un conteneur"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - administration-systeme
  - conteneurisation
  - docker
  - devops
resume: "Chapitre 17 sur 17 du livre « Docker » : Exemple de multi-réseaux pour un conteneur. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
niveau: intermediaire
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-01-27
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre « Docker » — chapitre 17/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 16 — Gestion de clusters|← 16 — Gestion de clusters]] · [[Docker — Compléments 2026|Compléments 2026 →]]

# Exemple de multi-réseaux pour un conteneur
Supposons que nous développons un conteneur docker pour Postfix et souhaitons l'associer à une adresse "failover" et un réseau interne de plusieurs conteneur. Le conteneur docker jouant alors le rôle de "relay"  entre les autres conteneurs et internet.

La solution la plus souple (et la plus « Docker‑friendly ») pour qu’un conteneur dispose à la fois :

1. d’un accès au réseau interne Docker (`mon_network`) pour communiquer avec vos autres conteneurs,
2. **et** d’une IP publique (la failover) directement attachée à l’interface du conteneur (pour Postfix et autorisée dans l'entrée SPF du domaine),

est de connecter notre conteneur **à deux réseaux** :

- **`meshs_network`** (bridge Docker classique) pour l’interaction conteneur‑vers‑conteneur,
- **un réseau Docker de type macvlan** (sur l’interface physique de l’hôte) pour lui attribuer l’IP failover.

Ainsi, le conteneur aura deux interfaces réseau virtuelles :

- **eth0** : connectée à `meshs_network`, avec une IP interne (ex. `172.18.0.x`),
- **eth1** : connectée au réseau macvlan, avec l’IP publique failover (ex. `212.83.163.229`).

Ensuite, dans la configuration de Postfix, vous lui indiquez de se « lier » à l’adresse failover (via `smtp_bind_address = 212.83.163.229`). Postfix sortira alors sur le réseau macvlan, tandis que l’ensemble du trafic interne au conteneur (communication avec Zope ou autres services) continuera de passer par le réseau Docker interne `meshs_network`.

---

## Étapes principales

3. **Créer le réseau macvlan**  
    En supposant que votre interface hôte est `eno1` et que vous avez la failover `212.83.163.229/32`, vous pouvez (à adapter selon votre plage réseau) :
    
    ```bash
    docker network create -d macvlan \
      --subnet=212.83.163.224/29 \
      --gateway=212.83.163.225 \
      -o parent=eno1 \
      failover_macvlan
    ```
    
    _Ici, on suppose que vous avez un petit subnet /29 qui inclut 212.83.163.229, et 212.83.163.225 comme gateway (à affiner selon votre infra)._
    
4. **Lancer (ou connecter) votre conteneur Postfix sur les 2 réseaux**
    
    - Déjà, vous lancez votre conteneur sur `meshs_network` comme vous le faites aujourd’hui.
        
    - Puis vous le connectez **en plus** à `failover_macvlan` en spécifiant l’IP failover :
        
        ```bash
        docker network connect \
          --ip 212.83.163.229 \
          failover_macvlan \
          meshs_postfix_container
        ```
        
    
    À ce stade, le conteneur aura deux interfaces :
    
    - **eth0** (bridge Docker `meshs_network`)
    - **eth1** (macvlan `failover_macvlan`, IP = `212.83.163.229`)
5. **Configurer Postfix pour écouter/envoyer via l’IP failover**  
    Dans votre `/etc/postfix/main.cf` du conteneur, assurez-vous d’avoir la ligne :
    
    ```postfix
    smtp_bind_address = 212.83.163.229
    ```
    
    Postfix utilisera alors l’interface associée à cette IP (eth1) pour l’envoi.  
    _Si vous exécutez Postfix en mode chroot, veillez à ce que la résolution DNS et la configuration réseau dans le chroot soient correctes, comme abordé précédemment._
    
6. **Vérifier le routing et la résolution**
    
    - **DNS** : Assurez-vous que l’hôte (ou le conteneur) résout correctement le nom de domaine _plateforme.meshs.net_.
    - **SPF** : Vérifiez que votre enregistrement SPF pour _plateforme.meshs.net_ inclut bien `ip4:212.83.163.229`.
    - **Connectivité** : Testez l’accès au conteneur sur `212.83.163.229` (si besoin) et vérifiez dans vos logs Postfix que l’adresse source des mails envoyés est bien la failover.

---

### Pourquoi cette stratégie fonctionne ?

- **macvlan** donne à votre conteneur la possibilité de s’annoncer directement sur le LAN (ou sur Internet s’il s’agit d’une IP publique) en utilisant sa propre adresse MAC et son IP failover.
- Le conteneur est toujours joignable par d’autres conteneurs via `meshs_network` (comme avant), sans perturber la topologie interne de votre application.
- Postfix, grâce à `smtp_bind_address`, utilisera l’interface macvlan pour toute émission SMTP, respectant ainsi votre configuration SPF et garantissant que les mails sortent depuis la failover.

### Alternative : mode `--network host`

Vous pourriez lancer Postfix en mode `host`, ce qui permettrait d’utiliser directement l’IP failover de l’hôte, mais vous perdriez l’isolation Docker sur le plan réseau (et ne bénéficieriez plus de `meshs_network`). C’est moins flexible que la solution macvlan + bridge.

---

**Conclusion** : La meilleure approche pour répondre à votre besoin (rester dans le réseau Docker interne tout en exposant l’adresse failover pour Postfix) est de créer un **second réseau macvlan** et d’y connecter votre conteneur Postfix avec l’IP publique. Vous bénéficiez ainsi d’une isolation propre, de la communication interne sur `meshs_network` et de l’envoi SMTP depuis l’IP failover.

## Liens
> [Premier Pas & Installation](https://youtu.be/AWJG-AbGFik)
> [Le Dockerfile et ses instructions](https://youtu.be/BUJAI1ptUu8)
> [Les layers Docker](https://youtu.be/_iUjrWhzx9o)
> [Les différents types de volumes docker](https://youtu.be/vHGwkA9fsIs)
> [Les volumes](https://youtu.be/lstTLSM5494)
> [Le UserID pour les volumes](https://youtu.be/SI1mhq-f0rg)
> [ENTRYPOINT vs CMD](https://youtu.be/kfyDu5R4VrM)
> [Docker pour Gitlab](https://youtu.be/Wbaczrx-US0)
> [Documentation officielle](https://docs.docker.com/engine/install/#server)
> [Article NextInpact](https://www.nextinpact.com/article/48913/docker-et-conteneurisation-par-exemple)
> [Network, réseau personnalisé, ip et driver host](https://youtu.be/wQIyczrZNQM)

---
> [!info] Livre « Docker » — chapitre 17/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 16 — Gestion de clusters|← 16 — Gestion de clusters]] · [[Docker — Compléments 2026|Compléments 2026 →]]
