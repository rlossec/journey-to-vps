# 5. Ports, Apache, reverse proxy

Le paquet a passé DNS, routage et firewalls. Il arrive sur **une interface, un port**. Si personne n’écoute, rien ne s’affiche. Si un process écoute, c’est encore à lui de **traiter** la requête.

Ici : qui écoute, **où** (public vs local), puis pourquoi Apache est souvent **deux fois** — un **frontend** (reverse proxy) et un **backend**.

Les protocoles (TCP, HTTP, HTTPS, TLS) restent une **autre** présentation. On les croise : port **80** / **443**, `Server: Apache` dans les en-têtes.

## Qui écoute — sockets et ports

Un **port** est un numéro (0–65535) pour distinguer les services sur **la même IP**. 22 → SSH, 80 → HTTP, 443 → HTTPS, etc.

Un service **à l’écoute** (*listening socket*) : un process a dit au système « les paquets pour **cette IP + ce port**, c’est pour moi ». `ss` / `netstat` listent ces sockets.

**Interface** :

| Écoute sur | Visible depuis Internet ? |
| --- | --- |
| `0.0.0.0` ou l’IP publique `54.36.100.9` | Oui, si le firewall laisse passer |
| `127.0.0.1` (localhost) | **Non** — seulement la machine elle-même |

D’où : tout n’est pas exposé. Un backend sur `127.0.0.1:8080` n’est pas « un site public », même sans `iptables`. Le firewall (étape 4) coupe ce qui **pourrait** l’être ; l’interface **localhost** empêche déjà le reste du monde de parler au process.

Sur **ce** VPS, on s’attend au moins à Apache (ou un proxy) sur 80/443, et souvent SSH sur 22. Le détail réel : sortie de `ss -tlnp` (capture `root`, formateur). Tout ce qui écoute en public **et** n’est pas filtré est une surface d’attaque — on n’ouvre pas « tous les ports par principe ».

`nmap -sV` depuis **notre** poste vers **notre** IP : ce qu’un client voit de l’extérieur (ports ouverts + indice de logiciel). Ça complète `ss`, qui se lit **sur** la machine.

En-tête observé aujourd’hui : `Server: Apache` sur `https://readresolve.tech` — le client parle à **Apache**, pas « à Internet en général ».

## Reverse proxy — Apache devant Apache

**Proxy forward** (idée, une phrase) : le **client** passe par un intermédiaire pour sortir (entreprise, VPN). Le serveur web de destination ne voit pas l’IP réelle du poste, ou pas directement.

**Reverse proxy** (notre cas) : l’intermédiaire est **devant nos applis**. Le **navigateur** ne parle **qu’à** lui. Les backends restent cachés.

```
Clients Internet
        │
        │  HTTPS (port 443, public)
        ▼
Apache frontend     ←  reverse proxy, écoute 0.0.0.0:443
        │
        │  HTTP en local (ex. 127.0.0.1)
        ▼
Apache backend      ←  appli / vhost interne, pas exposé
```

Le client ne connaît pas le port du backend, ni son nom interne. Il connaît `readresolve.tech` et le 443 du frontend.

Pourquoi ce schéma :

- **Un** point d’entrée (TLS, nom, logs) pour plusieurs applis derrière.
- Le backend n’a pas à être joignable du monde : moins de surface.
- Le proxy peut répartir, filtrer des chemins, servir des fichiers statiques — sans en faire un catalogue de modules Apache.

`ProxyPass` / `ProxyPassReverse` (idée) : « ce qui arrive sur `/` (ou un vhost), **relaye-le** vers tel serveur local ». La conf exacte : captures sur le VPS. On n’écrit pas un tutorial Apache complet.

Sans reverse proxy, chaque backend public = un port (ou un IP) de plus à ouvrir et à durcir. Avec : **un** Apache public, le reste en localhost.

## Questions à garder en tête

- Quels services sont exposés ?
- Pourquoi tous les services ne sont-ils pas publics ?
- Pourquoi un proxy **devant** les applis ?
- Pourquoi ne pas exposer chaque backend directement ?
- Que peut prendre en charge le proxy ?

Le navigateur a demandé `https://readresolve.tech`. DNS → IP, routage → OVH, firewalls, Apache. La page peut s’afficher. C’est tout le fil depuis l’intro.
