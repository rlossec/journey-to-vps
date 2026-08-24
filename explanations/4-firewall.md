# 4. Firewall

Le paquet a trouvé OVH. Il n’est **toujours pas** dans Apache. Deux douanes encore :

1. **Chez OVH** — anti-DDoS (HCAP, VAC) et pare-feu de bordure. On ne les installe pas sur le VPS.
2. **Sur le VPS** — `iptables` (ou équivalent). **Nous.**

Un firewall, c’est un **filtre de paquets** : selon l’adresse, le port, le sens (entrant / sortant), on **accepte**, on **refuse**, ou on **ignore**. Tant qu’une règle refuse le HTTPS, le navigateur n’atteint pas Apache — même si le DNS et le routage sont justes.

## Côté OVH — avant la machine

### DoS et DDoS

**DoS** (Denial of Service) : une source sature la cible.  
**DDoS** (Distributed) : **beaucoup** de sources (botnet). Plus dur à couper : ce n’est plus « bloquer une IP ».

```
DoS                         DDoS
  attaquant ──► VPS          des milliers de machines
                             ──► ──► VPS
                             ──►
```

Trois familles, sans entrer dans les protocoles :

- **Volumétrique** — noyer la **bande passante** (le tuyau du VPS est minuscule face à un flood).
- **Surcharge de ressources** — trop de paquets / connexions, la machine s’épuise (souvent ce qu’on range dans les couches réseau / transport).
- **Exploitation** — viser un trou logiciel. Ce n’est plus seulement « trop de trafic ».

Un VPS tout juste créé, **IP publique déjà routée**, firewall local encore ouvert : cible facile. C’est arrivé **dès la mise en service** du serveur de formation, avant qu’on pose `iptables`. D’où l’intérêt du filtrage **en amont**, chez l’hébergeur — et d’un firewall VPS **tout de suite** après.

On laisse de côté le cas particulier des **serveurs de jeux** (offre anti-DDoS GAME OVH).

### HCAP, VAC, pare-feu de bordure

Tout ça est **dans le réseau OVH**, pas dans la VM.

| Brique | Rôle | Qui configure |
| --- | --- | --- |
| **HCAP** (*Hardware Client Amplitude Policer*) | Premier filtre, aux **points de présence** (là où OVH se raccorde aux autres opérateurs). Coupe / bride le volumétrique **avant** le datacenter. Peut **décharger** les nœuds VAC. | OVH (automatique) |
| **VAC** | Centres de « lavage » du trafic : on tente de laisser passer le légitime, d’écarter le flood. | OVH (mitigation auto) |
| **Pare-feu de bordure** (*Edge Network Firewall*) | Filtre **sans état** (ACL : IP, ports, protocoles) intégré à l’anti-DDoS. Peut s’activer tout seul pendant une attaque. On peut y poser quelques règles dans l’espace client pour **soulager** `iptables`. | OVH + un peu nous (console) |

Le HCAP ne remplace pas `iptables` : il ne sait pas « notre appli n’écoute que 443 ». Il protège **le tuyau et le réseau**. Le détail métier (SSH, HTTP, rien d’autre) reste **sur le VPS** — et, si on veut, recopiée en partie au bord.

Ping OK + site mort : souvent **notre** firewall (ou Apache), pas « Internet cassé ». Inversement, un DDoS volumétrique peut tuer le lien **avant** que `iptables` ne voie les paquets : d’où HCAP / VAC.

## Côté VPS — `iptables`

### Pourquoi encore un firewall

OVH ne connaît pas notre politique : SSH depuis où, 80/443 ouverts, le reste fermé. Sans filtre local, **tout port** où un process écoute est joignable (étape 5 : qui écoute vraiment).

Sens :

- **Entrant** (`INPUT`) — Internet → le VPS. C’est le plus critique.
- **Sortant** (`OUTPUT`) — le VPS → l’extérieur (mises à jour, DNS). Souvent plus permissif en formation.

**Politique par défaut** : si **aucune** règle ne matche, on **ACCEPT** ou on **DROP**. En durcissement : `INPUT` en DROP, puis on ouvre 22 / 80 / 443. Une politique ACCEPT + « j’ai oublié une règle » = tout passe.

**Ordre** : première règle qui matche **gagne**. Une `DROP` trop haut dans la liste peut cacher une `ACCEPT` plus bas. D’où les captures `iptables -L` : on lit de **haut en bas**.

`iptables` (table `filter`, chaînes `INPUT` / `OUTPUT` / `FORWARD`) : l’outil classique Linux. On n’en fait pas un cours complet : lire une liste, comprendre politique + ordre, savoir qu’une règle **DROP 443** suffit à « casser le HTTPS » pour le TP.

### TP — couper le 443

Sur **ce** VPS (on est admin), sans toucher au **22** :

1. Depuis un PC : `curl -I https://readresolve.tech` → ça répond.
2. Sur le VPS : poser un `DROP` (ou `REJECT`) sur le port **443** en `INPUT`.
3. Re-tester : navigateur / DevTools → échec ; `curl` → **timeout** (`DROP`) ou **connection refused** (`REJECT`). Le DNS et `ping` peuvent encore marcher.
4. Retirer la règle. Le site revient.

Ça isole la douane VPS : le routage (étape 3) n’a pas bougé ; Apache n’a simplement **plus** le droit de recevoir le HTTPS.

## Questions à garder en tête

- Pourquoi un firewall (OVH **et** VPS) ?
- Que se passe-t-il si **aucune** règle ne correspond ?
- Pourquoi l’**ordre** des règles compte-t-il ?
- Pourquoi le trafic ne va-t-il pas directement sur le VPS ? Quelles couches sont **OVH** ?

Le paquet est accepté. Reste à voir **qui écoute**, et sur quels ports — étape 5.
