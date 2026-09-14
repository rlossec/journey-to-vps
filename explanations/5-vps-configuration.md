# 5. Configuration du VPS

Nos données ont traversé le DNS, routage et l'infrastructure OVH. Il arrive enfin sur **notre** machine.

Avant de rentrer dans le VPS, on a besoin d'un peu de théorie.

## Théorie

### Ports — le numéro d'appartement

On connait et on visualise bien ce que sont les ports physiques : USB, HDMI, VGA ou Ethernet.

Il existe aussi les ports logiques. Il ne sont pass palpables et sont représentés par des nombres de 0 à 65 535

On peut les diviser en trois grandes catégories :

- les ports systèmes
- les ports enregistrés
- les ports dynamiques ou privés

[Schéma types de ports](../excalidraw/5-vps-configuration/5-1-ports-type.excalidraw)

| Plage               | Nom                                         | Rôle                                                                                                                             | Exemples                                           |
| ------------------- | ------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| **0 – 1 023**       | _Well-known ports_ (ports système)          | Réservés aux services **standard**, connus de tous. Nécessitent souvent des droits admin.                                        | 22 (SSH), 80 (HTTP), 443 (HTTPS), 53 (DNS)         |
| **1 024 – 49 151**  | _Registered ports_ (ports enregistrés)      | Attribués à des **services spécifiques** par l'IANA, mais pas « système ».                                                       | 3306 (MySQL), 5432 (PostgreSQL), 8080 (proxy HTTP) |
| **49 152 – 65 535** | _Dynamic / private ports_ (ports éphémères) | Utilisés **temporairement** par le système pour les connexions sortantes du client. Jamais configurés manuellement côté serveur. | Choisis à la volée par l'OS                        |

### Socket — l'adresse complète de livraison

Un port seul ne suffit pas. Pour qu'une connexion existe, il faut trois informations :

1. **L'IP** — le bâtiment (`54.36.100.9`)
2. **Le port** — le numéro d'appartement (`443`)
3. **Le protocole de transport** — le **type de livraison** (TCP ou UDP)

L'ensemble forme une **socket** : `TCP 54.36.100.9:443`.

[Schéma socket éclatée](../excalidraw/5-vps-configuration/5-2-socket-splitted.excalidraw)



[Schéma socket](../excalidraw/5-vps-configuration/5-3-socket-analogy.excalidraw)

> **Analogie** : une adresse postale complète, c'est **le bâtiment** (IP) + **l'appartement** (port) + **le mode de livraison** (recommandé = TCP, simple dépôt boîte aux lettres = UDP). Sans l'un des trois, la livraison échoue ou arrive au mauvais endroit.

Un service **à l'écoute** (_listening socket_) : un process a dit au système « les paquets pour **cette socket**, c'est pour moi ». `ss` / `netstat` listent ces sockets.


**Interface** :

| Écoute sur                               | Visible depuis Internet ?                |
| ---------------------------------------- | ---------------------------------------- |
| `0.0.0.0` ou l'IP publique `54.36.100.9` | Oui, si le firewall laisse passer        |
| `127.0.0.1` (localhost)                  | **Non** — seulement la machine elle-même |

D'où : tout n'est pas exposé. Un backend sur `127.0.0.1:8080` n'est pas « un site public », même sans `iptables`. Le firewall coupe ce qui **pourrait** l'être ; l'interface **localhost** empêche déjà le reste du monde de parler au process.

### Proxy et Reverse Proxy

Qu’est-ce qu’un proxy, au juste ? Voyons cela.

Deux types courants de proxy sont le **forward proxy** et le **reverse proxy**.

####  Forward proxy

Un forward proxy est un serveur placé entre un groupe de machines clientes et Internet. Quand ces clients envoient des requêtes vers des sites, le forward proxy joue le rôle d’intermédiaire : il intercepte ces requêtes et parle aux serveurs web **au nom** de ces machines clientes.

**Pourquoi voudrait-on faire ça ?**

1. Protéger l’identité en ligne du client

En se connectant à un site via un forward proxy, l’adresse IP du client est masquée au serveur. Seule l’IP du proxy est visible. Il est plus difficile de remonter jusqu’au client.

2. Contourner des restrictions de navigation

Des institutions (États, écoles, grandes entreprises) utilisent des firewalls pour limiter l’accès à Internet. En se connectant à un forward proxy situé *hors* de ces firewalls, le client peut parfois contourner ces restrictions.

Cela ne fonctionne pas toujours : le firewall peut aussi bloquer les connexions vers le proxy.

3. Bloquer l’accès à certains contenus

Les écoles et les entreprises configurent souvent le réseau pour que tous les clients passent par un proxy, avec des règles de filtrage (réseaux sociaux, etc.).

Un forward proxy exige en général que le client configure son application pour le pointer. Les grandes institutions utilisent souvent un **transparent proxy** pour simplifier cela.

**En résumé :** un forward proxy se place entre le client et Internet, et agit **au nom du client**.

#### Reverse proxy

Un reverse proxy se place entre Internet et les serveurs web. Il intercepte les requêtes des clients et parle aux serveurs web **à leur place**.

**Pourquoi un site utiliserait-il un reverse proxy ?**

1. Protéger le site

Les adresses IP du site sont cachées derrière le reverse proxy et ne sont pas révélées aux clients. Il devient plus difficile de cibler le site avec une attaque DDoS.

2. Load balancing

Un site très fréquenté ne peut généralement pas tout gérer avec un seul serveur. Le reverse proxy répartit les requêtes entrantes sur un parc de serveurs web, pour éviter qu’un seul d’entre eux ne sature.

Cela suppose que le reverse proxy lui-même tienne la charge. Des services comme Cloudflare déploient des reverse proxies dans des centaines de lieux dans le monde : plus proches des utilisateurs, et avec une grande capacité de traitement.

3. Mettre en cache le contenu statique

Un contenu peut rester en cache sur le reverse proxy pendant un certain temps. Si la même ressource est redemandée, la copie locale peut être renvoyée rapidement.

4. Gérer le chiffrement SSL

Le SSL handshake est coûteux en calcul. Le reverse proxy décharge les origin servers de ces opérations. Au lieu de gérer le SSL pour tous les clients, le site n’a plus qu’à gérer les SSL handshakes avec un petit nombre de reverse proxies.



### Firewall — filtre de paquets

Un firewall décide, pour chaque paquet : **accepter**, **refuser** ou **ignorer**, selon l'IP, le port, le protocole, le sens.

Deux couches utiles à distinguer (étape 4 vs ici) :

| Couche                           | Où                                      | Qui configure                 |
| -------------------------------- | --------------------------------------- | ----------------------------- |
| **Edge Network Firewall**        | Bordure du réseau OVH, **avant** le VPS | Console OVH (quelques règles) |
| **Firewall du VPS** (`iptables`) | **Sur** la machine                      | Nous (`root`)                 |



#### Sens du filtrage (`iptables`)

- **Entrant** (`INPUT`) — Internet → le VPS. C'est le plus critique.
- **Sortant** (`OUTPUT`) — le VPS → l'extérieur (mises à jour, DNS). Souvent plus permissif en formation.

#### Politique par défaut

Si **aucune** règle ne matche, on **ACCEPT** ou on **DROP**. En durcissement : `INPUT` en DROP, puis on ouvre ce qu'il faut (SSH, 80, 443). Une politique ACCEPT + « j'ai oublié une règle » = tout passe.

#### Ordre des règles

Première rule qui matche **gagne**. Une `DROP` trop haut dans la liste peut cacher une `ACCEPT` plus bas. D'où les captures `iptables -L` : on lit de **haut en bas**.

`iptables` (table `filter`, chaînes `INPUT` / `OUTPUT` / `FORWARD`) : l'outil classique Linux. On n'en fait pas un cours complet : lire une liste, comprendre politique + ordre, savoir qu'une règle **DROP 443** suffit à « casser le HTTPS » pour le TP.

**Chaîne d'arrivée sur le VPS** (rappel formateur) :

```
Edge Network Firewall (OVH)
        │
        ▼
Firewall VPS (iptables)
        │
        ▼
Apache frontend (reverse proxy)  ← 80 / 443 publics
        │
        ▼
Apache backend (localhost)       ← pas exposé
```

Le firewall laisse passer (ou non). **Ensuite**, c'est le reverse proxy qui accueille le client et parle aux backends.

## Pratique

Voilà pour la théorie. On revient sur **notre** VPS `54.36.100.9` / `readresolve.tech`, dans l'ordre du chemin réel.

### 1. Edge Network Firewall — ce que OVH laisse arriver

Capture console OVH pour **notre** IP (étape 4 : Edge Firewall). Règles actives, politique du type « default deny » en fin de liste :

![Edge Network Firewall — règles pour 54.36.100.9](../img/5-vps-firewall.png)

Ce qu'on lit concrètement :

| Priorité | Effet                                                                                        |
| -------- | -------------------------------------------------------------------------------------------- |
| 0        | TCP `established` — trafic de retour (firewall **sans état** : pas de suivi de session réel) |
| 1        | TCP **64483** — accès admin (SSH déplacé ; pas le 22)                                        |
| 2–3      | TCP **80** et **443** — le site                                                              |
| 17       | ICMP — `ping` possible                                                                       |
| 19       | **Refuse** tout le reste IPv4                                                                |

L'UI le rappelle : cette protection de bordure se **couple** avec le firewall **du serveur** (`iptables` / `ufw`…). L'Edge ne remplace pas la douane locale.

### 2. Firewall du VPS — `iptables`

Sur la machine, **notre** politique : ce que `iptables` accepte en `INPUT`. Aligné avec l'Edge : 80, 443, 64483 — pas « tous les ports ».

#### TP — couper le 443

### 3. Reverse proxy — après le firewall

Le paquet a passé Edge + `iptables`. Il arrive sur un process qui **écoute**. Sur **ce** VPS : Apache **frontend** en reverse proxy (ports **80** / **443** publics) → Apache **backend** en localhost.

Le client parle à `https://readresolve.tech` → Apache frontend. Pas directement au backend. En-tête observé : `Server: Apache` — le client parle à **Apache**, pas « à Internet en général ».

(Captures conf `ProxyPass` / vhosts : formateur.)

### 4. Qui écoute — `ss`

Sur le VPS, on liste les sockets en écoute :

```bash
ss -tlnp
```

On s'attend à croiser la théorie (port + interface) avec le concret :

| Ce qu'on voit                   | Lecture                                                                               |
| ------------------------------- | ------------------------------------------------------------------------------------- |
| `*:80` / `*:443` (ou `0.0.0.0`) | Point d'entrée public — reverse proxy                                                 |
| `0.0.0.0:64483`                 | SSH / admin, pas le well-known 22                                                     |
| `127.0.0.1:90xx` (plusieurs)    | Backends **localhost** — invisibles depuis Internet, même si le firewall était ouvert |

Tout ce qui écoute en public **et** n'est pas filtré est une surface d'attaque — on n'ouvre pas « tous les ports par principe ».

Complément depuis **notre** poste :

```bash
nmap -sV -p 22,80,443,64483 54.36.100.9
```

`ss` se lit **sur** la machine ; `nmap` montre ce qu'un client voit **de l'extérieur**. Souvent : 80/443 ouverts, **22** fermé ou filtré, **64483** ouvert — cohérent avec l'Edge et `iptables`.

## Questions à garder en tête

- Quels services sont exposés ?
- Quelle est la différence entre un port well-known et un port éphémère ?
- Qu'est-ce qu'une socket ? Pourquoi trois composants ?
- Quelle est la différence entre proxy forward et reverse proxy ?
- Pourquoi tous les services ne sont-ils pas publics ?
- Pourquoi un proxy **devant** les applis ?
- Pourquoi ne pas exposer chaque backend directement ?
- Que distingue l'Edge Firewall (OVH) de `iptables` sur le VPS ?

Le navigateur a demandé `https://readresolve.tech`. DNS → IP, routage → OVH, Edge Firewall, `iptables`, Apache (reverse proxy). La page peut s'afficher. C'est tout le fil depuis l'intro.
