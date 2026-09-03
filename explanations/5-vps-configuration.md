# 5. Configuration du VPS

Le paquet a traversé DNS, routage et l'infrastructure OVH (étape 4). Il arrive enfin sur **notre** machine. Tout ce qui suit est **sous notre contrôle**.

Avant de voir qui écoute et comment, posons le vocabulaire. Les protocoles (TCP, HTTP, HTTPS, TLS) restent une **autre** présentation. On les croise : port **80** / **443**, `Server: Apache` dans les en-têtes.

## Vocabulaire

### Ports — le numéro d'appartement

Un **port** est un numéro (0–65 535) qui identifie un service sur une machine. L'IP seule désigne **le bâtiment** (le serveur) ; le port désigne **l'appartement** dans ce bâtiment.

> **Analogie** : le facteur (Internet) livre un colis au **54 rue de la Formation** (l'IP `54.36.100.9`). Il regarde ensuite le **numéro d'appartement** sur l'étiquette pour savoir **à quelle porte** frapper. Port **22** → appartement SSH, port **443** → appartement HTTPS, etc.

Sans numéro de port, le paquet arrive au bâtiment mais personne ne sait à qui il est destiné.

#### Trois plages de ports

[Schéma types de ports](../excalidraw/5-vps-configuration/5-1-ports-type.excalidraw)

| Plage | Nom | Rôle | Exemples |
| --- | --- | --- | --- |
| **0 – 1 023** | *Well-known ports* (ports système) | Réservés aux services **standard**, connus de tous. Nécessitent souvent des droits admin. | 22 (SSH), 80 (HTTP), 443 (HTTPS), 53 (DNS) |
| **1 024 – 49 151** | *Registered ports* (ports enregistrés) | Attribués à des **services spécifiques** par l'IANA, mais pas « système ». | 3306 (MySQL), 5432 (PostgreSQL), 8080 (proxy HTTP) |
| **49 152 – 65 535** | *Dynamic / private ports* (ports éphémères) | Utilisés **temporairement** par le système pour les connexions sortantes du client. Jamais configurés manuellement côté serveur. | Choisis à la volée par l'OS |

Sur **notre VPS**, on s'attend à voir les well-known **22**, **80**, **443** ouverts — et rien d'autre en public.

### Socket — l'adresse complète de livraison

Un port seul ne suffit pas. Pour qu'une connexion existe, il faut trois informations :

1. **L'IP** — le bâtiment (`54.36.100.9`)
2. **Le port** — le numéro d'appartement (`443`)
3. **Le protocole de transport** — le **type de livraison** (TCP ou UDP)

L'ensemble forme une **socket** : `TCP 54.36.100.9:443`.

[Schéma socket](../excalidraw/5-vps-configuration/5-3-socket-analogy.excalidraw)

> **Analogie** : une adresse postale complète, c'est **le bâtiment** (IP) + **l'appartement** (port) + **le mode de livraison** (recommandé = TCP, simple dépôt boîte aux lettres = UDP). Sans l'un des trois, la livraison échoue ou arrive au mauvais endroit.

Un service **à l'écoute** (*listening socket*) : un process a dit au système « les paquets pour **cette socket**, c'est pour moi ». `ss` / `netstat` listent ces sockets.

[Schéma socket éclatée](../excalidraw/5-vps-configuration/5-2-socket-splitted.excalidraw)

**Interface** :

| Écoute sur | Visible depuis Internet ? |
| --- | --- |
| `0.0.0.0` ou l'IP publique `54.36.100.9` | Oui, si le firewall laisse passer |
| `127.0.0.1` (localhost) | **Non** — seulement la machine elle-même |

D'où : tout n'est pas exposé. Un backend sur `127.0.0.1:8080` n'est pas « un site public », même sans `iptables`. Le firewall (étape 4) coupe ce qui **pourrait** l'être ; l'interface **localhost** empêche déjà le reste du monde de parler au process.

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

## Firewall du VPS — `iptables`

L'infrastructure OVH (étape 4) protège le réseau en amont, mais elle ne connaît pas **notre** politique : SSH depuis où, 80/443 ouverts, le reste fermé. Sans filtre local, **tout port** où un process écoute est joignable.

### Sens du filtrage

- **Entrant** (`INPUT`) — Internet → le VPS. C'est le plus critique.
- **Sortant** (`OUTPUT`) — le VPS → l'extérieur (mises à jour, DNS). Souvent plus permissif en formation.

### Politique par défaut

Si **aucune** règle ne matche, on **ACCEPT** ou on **DROP**. En durcissement : `INPUT` en DROP, puis on ouvre 22 / 80 / 443. Une politique ACCEPT + « j'ai oublié une règle » = tout passe.

### Ordre des règles

Première règle qui matche **gagne**. Une `DROP` trop haut dans la liste peut cacher une `ACCEPT` plus bas. D'où les captures `iptables -L` : on lit de **haut en bas**.

`iptables` (table `filter`, chaînes `INPUT` / `OUTPUT` / `FORWARD`) : l'outil classique Linux. On n'en fait pas un cours complet : lire une liste, comprendre politique + ordre, savoir qu'une règle **DROP 443** suffit à « casser le HTTPS » pour le TP.

### TP — couper le 443

Sur **ce** VPS (on est admin), sans toucher au **22** :

1. Depuis un PC : `curl -I https://readresolve.tech` → ça répond.
2. Sur le VPS : poser un `DROP` (ou `REJECT`) sur le port **443** en `INPUT`.
3. Re-tester : navigateur / DevTools → échec ; `curl` → **timeout** (`DROP`) ou **connection refused** (`REJECT`). Le DNS et `ping` peuvent encore marcher.
4. Retirer la règle. Le site revient.

Ça isole la douane VPS : le routage (étape 3) et l'infra OVH (étape 4) n'ont pas bougé ; Apache n'a simplement **plus** le droit de recevoir le HTTPS.

## Pratique — qui écoute sur le VPS

Sur **ce** VPS, on s'attend au moins à Apache (ou un proxy) sur 80/443, et souvent SSH sur 22. Le détail réel : sortie de `ss -tlnp` (capture `root`, formateur). Tout ce qui écoute en public **et** n'est pas filtré est une surface d'attaque — on n'ouvre pas « tous les ports par principe ».

`nmap -sV` depuis **notre** poste vers **notre** IP : ce qu'un client voit de l'extérieur (ports ouverts + indice de logiciel). Ça complète `ss`, qui se lit **sur** la machine.

En-tête observé aujourd'hui : `Server: Apache` sur `https://readresolve.tech` — le client parle à **Apache**, pas « à Internet en général ».

## Reverse proxy — Apache devant Apache

`ProxyPass` / `ProxyPassReverse` (idée) : « ce qui arrive sur `/` (ou un vhost), **relaye-le** vers tel serveur local ». La conf exacte : captures sur le VPS. On n'écrit pas un tutorial Apache complet.

## Questions à garder en tête

- Quels services sont exposés ?
- Quelle est la différence entre un port well-known et un port éphémère ?
- Qu'est-ce qu'une socket ? Pourquoi trois composants ?
- Quelle est la différence entre proxy forward et reverse proxy ?
- Pourquoi tous les services ne sont-ils pas publics ?
- Pourquoi un proxy **devant** les applis ?
- Pourquoi ne pas exposer chaque backend directement ?

Le navigateur a demandé `https://readresolve.tech`. DNS → IP, routage → OVH, firewalls, Apache. La page peut s'afficher. C'est tout le fil depuis l'intro.
