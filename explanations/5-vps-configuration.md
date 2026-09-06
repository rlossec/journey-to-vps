# 5. Configuration du VPS

Nos données ont traversé le DNS, routage et l'infrastructure OVH (étape 4). Il arrive enfin sur **notre** machine. Tout ce qui suit est **sous notre contrôle**.

Avant de voir qui écoute et comment, posons le vocabulaire. Les protocoles (TCP, HTTP, HTTPS, TLS) restent une **autre** présentation. On les croise : port **80** / **443**, `Server: Apache` dans les en-têtes.

## Théorie

### Ports — le numéro d'appartement

Un **port** est un numéro (0–65 535) qui identifie un service sur une machine. L'IP seule désigne **le bâtiment** (le serveur) ; le port désigne **l'appartement** dans ce bâtiment.

> **Analogie** : le facteur (Internet) livre un colis au **54 rue de la Formation** (l'IP `54.36.100.9`). Il regarde ensuite le **numéro d'appartement** sur l'étiquette pour savoir **à quelle porte** frapper. Port **22** → appartement SSH (souvent), port **443** → appartement HTTPS, etc.

Sans numéro de port, le paquet arrive au bâtiment mais personne ne sait à qui il est destiné.

#### Trois plages de ports

[Schéma types de ports](../excalidraw/5-vps-configuration/5-1-ports-type.excalidraw)

| Plage               | Nom                                         | Rôle                                                                                                                             | Exemples                                           |
| ------------------- | ------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| **0 – 1 023**       | _Well-known ports_ (ports système)          | Réservés aux services **standard**, connus de tous. Nécessitent souvent des droits admin.                                        | 22 (SSH), 80 (HTTP), 443 (HTTPS), 53 (DNS)         |
| **1 024 – 49 151**  | _Registered ports_ (ports enregistrés)      | Attribués à des **services spécifiques** par l'IANA, mais pas « système ».                                                       | 3306 (MySQL), 5432 (PostgreSQL), 8080 (proxy HTTP) |
| **49 152 – 65 535** | _Dynamic / private ports_ (ports éphémères) | Utilisés **temporairement** par le système pour les connexions sortantes du client. Jamais configurés manuellement côté serveur. | Choisis à la volée par l'OS                        |

En théorie, un serveur web expose surtout les well-known **80** et **443**. Le SSH est souvent sur **22**, mais on peut le déplacer — on verra ce que **notre** VPS fait vraiment en pratique.

### Socket — l'adresse complète de livraison

Un port seul ne suffit pas. Pour qu'une connexion existe, il faut trois informations :

1. **L'IP** — le bâtiment (`54.36.100.9`)
2. **Le port** — le numéro d'appartement (`443`)
3. **Le protocole de transport** — le **type de livraison** (TCP ou UDP)

L'ensemble forme une **socket** : `TCP 54.36.100.9:443`.

[Schéma socket](../excalidraw/5-vps-configuration/5-3-socket-analogy.excalidraw)

> **Analogie** : une adresse postale complète, c'est **le bâtiment** (IP) + **l'appartement** (port) + **le mode de livraison** (recommandé = TCP, simple dépôt boîte aux lettres = UDP). Sans l'un des trois, la livraison échoue ou arrive au mauvais endroit.

Un service **à l'écoute** (_listening socket_) : un process a dit au système « les paquets pour **cette socket**, c'est pour moi ». `ss` / `netstat` listent ces sockets.

[Schéma socket éclatée](../excalidraw/5-vps-configuration/5-2-socket-splitted.excalidraw)

**Interface** :

| Écoute sur                               | Visible depuis Internet ?                |
| ---------------------------------------- | ---------------------------------------- |
| `0.0.0.0` ou l'IP publique `54.36.100.9` | Oui, si le firewall laisse passer        |
| `127.0.0.1` (localhost)                  | **Non** — seulement la machine elle-même |

D'où : tout n'est pas exposé. Un backend sur `127.0.0.1:8080` n'est pas « un site public », même sans `iptables`. Le firewall coupe ce qui **pourrait** l'être ; l'interface **localhost** empêche déjà le reste du monde de parler au process.

### Proxy et Reverse Proxy

Les deux sont des **intermédiaires** entre un client et un serveur. La différence : **qui** les place et **de quel côté**.

#### Proxy forward — côté client

Le **client** passe par un intermédiaire pour **sortir**. Le serveur de destination ne voit pas l'IP réelle du poste (ou pas directement).

```
                      ┌──────────┐
  Poste A ──────────► │  Proxy   │ ──────────► Serveur web
  Poste B ──────────► │ (sortie) │             (google.com)
  Poste C ──────────► │          │
                      └──────────┘
          Réseau interne           Internet
```

> **Analogie** : un assistant qui va chercher le courrier **à la place** des employés. L'extérieur ne voit que l'assistant, pas qui a demandé quoi.

Usages : entreprise, VPN, filtrage de navigation.

#### Reverse proxy — côté serveur (notre cas)

L'intermédiaire est **devant nos applis**. Le **navigateur** ne parle **qu'à** lui. Les backends restent cachés.

```
                                     ┌───────────────────┐
  Clients Internet                   │      VPS          │
        │                            │                   │
        │  HTTPS (port 443, public)  │  ┌─────────────┐  │
        ▼                            │  │   Apache     │  │
  ──────────────────────────────────►│  │  frontend    │  │
                                     │  │ (rev. proxy) │  │
                                     │  └──────┬───────┘  │
                                     │         │          │
                                     │    HTTP local      │
                                     │   (127.0.0.1)      │
                                     │         │          │
                                     │  ┌──────▼───────┐  │
                                     │  │   Apache     │  │
                                     │  │  backend     │  │
                                     │  │  (appli)     │  │
                                     │  └──────────────┘  │
                                     └───────────────────┘
```

> **Analogie** : l'accueil d'un immeuble de bureaux. Le visiteur (client) ne monte pas directement dans les étages (backends). Il passe par l'accueil (reverse proxy) qui le redirige vers le bon bureau.

Le client ne connaît pas le port du backend, ni son nom interne. Il connaît `readresolve.tech` et le 443 du frontend.

#### Pourquoi un reverse proxy

- **Un** point d'entrée (TLS, nom, logs) pour plusieurs applis derrière.
- Le backend n'a pas à être joignable du monde : moins de surface d'attaque.
- Le proxy peut répartir, filtrer des chemins, servir des fichiers statiques.

Sans reverse proxy, chaque backend public = un port (ou une IP) de plus à ouvrir et à durcir. Avec : **un** Apache public, le reste en localhost.

### Firewall — filtre de paquets

Un firewall décide, pour chaque paquet : **accepter**, **refuser** ou **ignorer**, selon l'IP, le port, le protocole, le sens.

Deux couches utiles à distinguer (étape 4 vs ici) :

| Couche                           | Où                                      | Qui configure                 |
| -------------------------------- | --------------------------------------- | ----------------------------- |
| **Edge Network Firewall**        | Bordure du réseau OVH, **avant** le VPS | Console OVH (quelques règles) |
| **Firewall du VPS** (`iptables`) | **Sur** la machine                      | Nous (`root`)                 |

L'infra OVH protège le réseau en amont, mais elle ne connaît pas **toute** notre politique locale. Sans filtre sur le VPS, **tout port** où un process écoute (et que l'Edge laisse passer) est joignable.

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
