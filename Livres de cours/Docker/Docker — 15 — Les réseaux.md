---
schema_version: 1
uid: 01M1BQ62DXRWXV78HW905GQW91
titre: "Docker — 15 — Les réseaux"
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
resume: "Chapitre 15 sur 17 du livre « Docker » : Les réseaux. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Docker » — chapitre 15/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 14 — Utilisation de Docker Compose|← 14 — Utilisation de Docker Compose]] · [[Docker — 16 — Gestion de clusters|16 — Gestion de clusters →]]

# Les réseaux
Par défaut docker propose 3 types de réseau Bridget, Host, Aucun. Et il permet d'en créer ce qui remplace l'ancienne commande "--link".
Pour des conteneurs qui fonctionnent nous pouvons intérogger leur adresse et numéro de port avec les commandes :
```bash
michaellaunay@leojag:~$ docker run -d --name c2 nginx:latest
f3ba0b7b5e5b77656675a146ca2ff677fa67bfab865470ba39814168ada106f4
michaellaunay@leojag:~$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
f3ba0b7b5e5b   nginx:latest   "/docker-entrypoint.…"   27 seconds ago   Up 26 seconds   80/tcp    c2
42f1fcfb86a5   nginx:latest   "/docker-entrypoint.…"   34 seconds ago   Up 33 seconds   80/tcp    c1
michaellaunay@leojag:~$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
64194948d69f   bridge    bridge    local
9b6750ca6345   host      host      local
542911f5334c   none      null      local
michaellaunay@leojag:~$ docker network inspect 64194948d69f
[
    {
        "Name": "bridge",
        "Id": "64194948d69f79fd8e79ae29cbb5fcf15025d0cbae9472b95542a06b3613e482",
        "Created": "2023-03-23T18:51:45.348300819+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "42f1fcfb86a5f8a07b90c7e5ec1a37b36af630e5dc65c4b1d2717f11d305eeb6": {
                "Name": "c1",
                "EndpointID": "59916226657d2b54ddfe23441ce80178984794441603fe065e6630876d62493c",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "f3ba0b7b5e5b77656675a146ca2ff677fa67bfab865470ba39814168ada106f4": {
                "Name": "c2",
                "EndpointID": "aba35f15ee3a1ef73ba316a0f10796fd690f45be8a37ad03119339a0b1571984",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
michaellaunay@leojag:~$ docker ps -q | xargs docker inspect --format '{{ .Name }} - {{ .NetworkSettings.Ports }}'
/c2 - map[80/tcp:[]]
/c1 - map[80/tcp:[]]
michaellaunay@leojag:~$ docker ps -q | xargs docker inspect --format '{{ .Name }} - {{ .NetworkSettings.IPAddress }}'
/c2 - 172.17.0.3
/c1 - 172.17.0.2
michaellaunay@leojag:~$ docker exec -ti c1 bash
root@42f1fcfb86a5:/# apt install iputils-ping
Reading package lists... Done
...
root@42f1fcfb86a5:/# ping c2
ping: c2: Name or service not known
root@42f1fcfb86a5:/# ping 172.17.0.3
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.082 ms
```
**Attention** Les adresses ip  sont dynamiques et par défaut il n'y a pas de résolution de nom (Pas de DNS privé).
Il faut donc créer un réseau personnel.

Pour cela il suffit de faire :
```bash
michaellaunay@leojag:~$ docker run -d --name c1 --network MonReseau0 nginx:latest
bfaaf1b13522caaeb6a83daf850ca3465ea3dad95b70c2e6e7df2487858b2698
michaellaunay@leojag:~$ docker network ls
NETWORK ID     NAME         DRIVER    SCOPE
02e68136c61c   MonReseau0   bridge    local
64194948d69f   bridge       bridge    local
9b6750ca6345   host         host      local
542911f5334c   none         null      local
michaellaunay@leojag:~$ docker run -d --name c2 --network MonReseau0 nginx:latest
91eaa40f1c654baa30e0a9e1eb43c66dd46e132a54151f1df76edfd4c3bc5a5e
michaellaunay@leojag:~$ docker run -d --name c1 --network MonReseau0 nginx:latest
7b22e62a808622ad44e796f5abff75fb700c8545aec5e02127342f35d57194e1
michaellaunay@leojag:~$ docker exec -ti c1 bash
root@7b22e62a8086:/# apt install iputils-ping
Reading package lists... Done
...
root@7b22e62a8086:/# ping C2
PING C2 (192.168.7.2) 56(84) bytes of data.
64 bytes from c2.MonReseau0 (192.168.7.2): icmp_seq=1 ttl=64 time=0.095 ms
```
**Attention** à ne pas créer un sous réseau avec les mêmes adresses que notre réseau privé !

---
> [!info] Livre « Docker » — chapitre 15/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 14 — Utilisation de Docker Compose|← 14 — Utilisation de Docker Compose]] · [[Docker — 16 — Gestion de clusters|16 — Gestion de clusters →]]
