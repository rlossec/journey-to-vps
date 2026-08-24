## Intro

Théorie : `explanations/0-intro.md` · Accroche pratique : `commands/0-intro.md`

Avec Dominga, Jonathan et Rayann on avait déjà parlé du navigateur et de son fonctionnement. Cette fois : à partir d’une URL, **readresolve.tech**, comment obtient-on un site — et quels systèmes **autres que le navigateur** interviennent.

Deux cas de figure, **un seul** parcours :

- **Accéder** au site (fil principal)
- **Héberger / configurer** (domaine, firewall VPS, Apache) — sans ça, l’accès ne marche pas

### Schéma à montrer en premier

Navigateur → résolveur DNS → DNS autoritaire → routage Internet → HCAP → backbone → pare-feu de bordure → routeur DC → firewall VPS → Apache frontend (reverse proxy) → Apache backend

Le navigateur ne connaît pas cette chaîne : un nom, puis une IP (`54.36.100.9`), puis il envoie vers cette IP.

### Qui maîtrise quoi (à dire à voix haute)

- **OVH / Internet** : résolution (infra DNS), routage, HCAP, backbone, pare-feu de bordure, datacenter
- **Nous** : zone DNS du domaine, firewall VPS, services en écoute, Apache frontend / backend

Tant que le paquet n’a pas passé **notre** firewall, Apache n’a rien reçu.

### Accroche

Ouvrir le site (ou `curl -I https://readresolve.tech`). DevTools → Réseau : on voit une requête, pas le chemin.

Puis les 5 étapes. Premier zoom : **sans cache**, ordinateur tout neuf.

1. Résolution DNS
2. Enregistrement DNS
3. Routage Internet (jusqu’à OVH, puis *dans* OVH)
4. Firewall OVH, puis firewall VPS
5. Ports, Apache, reverse proxy

## 1. Résolution DNS

Théorie : `explanations/1-dns-lookup.md` · Commandes : `commands/1-dns-lookup.md`  
Schéma à projeter : `excalidraw/dns-resolver-corrige.json` · Ancien (conservé) : `excalidraw/dns-resolver.json`

Fil : **sans cache**, ordi neuf. Le navigateur demande au **résolveur** (FAI / 8.8.8.8) : `readresolve.tech` → quelle IP ?

Le résolveur enchaîne (il ne connaît pas l’IP tout de suite) :

1. **Racine** (13 identités A–M, milliers d’instances) : « `.tech` ? » → `ns01.trs-dns.com` … — pas A puis B : course RTT / anycast
2. **TLD `.tech`** : « qui gère `readresolve` ? » → `dns13.ovh.net` / `ns13.ovh.net`
3. **Autoritaire** : A → `54.36.100.9` (TTL 3600)

Puis **cache** (navigateur, OS, résolveur) et **propagation** : deux users, deux réponses possibles tant que les TTL n’ont pas expiré.

Pratique : Linux `dig +trace` · Windows `nslookup` itératif (racine → TLD → autoritaire).

## 2. Enregistrement DNS

Théorie : `explanations/2-dns-register.md` · Commandes : `commands/2-dns-register.md`

Étape 1 = **lire**. Ici = **écrire**. Réserver le nom ≠ le faire pointer.

Trois objets : nom (on réserve) · IP du VPS (OVH l’attribue, `54.36.100.9`) · **zone** (on la remplit, chez `dns13.ovh.net`).

Records, cas réel :

- **A** : `readresolve.tech` et `www` → même IP (deux A, pas un CNAME)
- **AAAA** : absent
- **NS** : ceux déjà vus en lecture
- **MX** / **TXT** : mail OVH / SPF — pas le web

**TTL** : on le choisit à l’écriture (A = 3600). D’où un changement d’IP pas immédiat partout.

Pratique : `dig A/NS/MX/TXT` · Windows `Resolve-DnsName -Type …` · capture console OVH (formateur).

## 3. Routage Internet

Théorie : `explanations/3-routage.md` · Commandes : `commands/3-routage.md`

L’IP est connue. Le navigateur **n’a pas** la route : il envoie vers `54.36.100.9`.

**BGP** (une phrase) : les opérateurs s’annoncent les blocs d’IP ; OVH annonce le préfixe du VPS.

Pratique : `ping` (joignable ?) · `traceroute` / `tracert` (sauts). `* * *` ≠ paquet mort. On voit FAI puis plages OVH, pas les étiquettes HCAP / backbone.

**Dans OVH** (modèle, pas lu sur traceroute) : HCAP → backbone → pare-feu de bordure → routeur DC → VPS. Rien de ça n’est à nous. Détail anti-DDoS : étape 4.

DevTools Réseau : une latence, zéro saut.

## 4. Firewall

Théorie : `explanations/4-firewall.md` · Commandes : `commands/4-firewall.md`

Deux douanes. Apache n’a encore rien.

**OVH** — DoS vs DDoS (une source / des milliers). Volumétrique, surcharge, exploit (une phrase chacun). Attaque dès la mise en service du VPS, `iptables` pas prêt. **HCAP** (policer en bordure, avant le DC) · **VAC** (lavage) · **pare-feu de bordure** (ACL, console). Pas le cas « serveur de jeux ».

**VPS** — `iptables` : entrant / sortant, politique par défaut, **ordre**. TP : `DROP`/`REJECT` **443**, pas le 22. `curl` + DevTools. `ping` peut rester vert.

Captures console OVH + `iptables -L` : formateur / root.

## 5. Ports, Apache, reverse proxy

Théorie : `explanations/5-apache-server.md` · Commandes : `commands/5-apache-server.md`

Le paquet est sur le VPS. **Qui écoute ?** Port + interface : `0.0.0.0` = public (si firewall OK) · `127.0.0.1` = machine seule.

`ss -tlnp` sur le VPS · `curl -I` → `Server: Apache` · `nmap -sV` vers **notre** IP.

**Forward** vs **reverse** (une phrase / le schéma). Ici : Apache **frontend** (443 public) → Apache **backend** (local). Le client ne parle qu’au proxy. `ProxyPass` : idée, captures conf.

Boucle : on a parcouru tout le fil de l’intro.
