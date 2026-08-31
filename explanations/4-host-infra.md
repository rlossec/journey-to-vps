# 4. Firewall

## Attaques DDoS

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

## Etapes dans OVH

Une fois le trafic dans le réseau OVH, il ne saute pas du premier routeur OVH à `54.36.100.9`. Plusieurs étages, **tous côté OVH** :

1. **HCAP** (anti-DDoS) — premier filtre, parfois **avant** le cœur du datacenter.
2. **Backbone** routers — gros routeurs qui relient les points de présence et les DC.
3. **Edge Firewall**— filtrage d’entrée du réseau / du site.
4. **Datacenter routers** — dernier aiguillage vers l’hôte qui porte le VPS.

!image.png

> **Pourquoi plusieurs couches ?** isoler les clients, absorber un flood **avant** qu’il n’atteigne une petite VM, ne pas exposer l’hyperviseur comme s’il était sur Internet nu. Rien de tout ça n’est configurable dans notre zone DNS ni dans Apache.

### HCAP

### HCAP, VAC

| Brique                                            | Rôle                                                                                                                                                                                                       | Qui configure               |
| ------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- |
| **HCAP** (_Hardware Client Amplitude Policer_)    | Premier filtre, aux **points de présence** (là où OVH se raccorde aux autres opérateurs). Coupe / bride le volumétrique **avant** le datacenter. Peut **décharger** les nœuds VAC.                         | OVH (automatique)           |
| **VAC**                                           | Centres de « lavage » du trafic : on tente de laisser passer le légitime, d’écarter le flood.                                                                                                              | OVH (mitigation auto)       |
| **Pare-feu de bordure** (_Edge Network Firewall_) | Filtre **sans état** (ACL : IP, ports, protocoles) intégré à l’anti-DDoS. Peut s’activer tout seul pendant une attaque. On peut y poser quelques règles dans l’espace client pour **soulager** `iptables`. | OVH + un peu nous (console) |

Le HCAP ne remplace pas `iptables` : il ne sait pas « notre appli n’écoute que 443 ». Il protège **le tuyau et le réseau**. Le détail métier (SSH, HTTP, rien d’autre) reste **sur le VPS** — et, si on veut, recopiée en partie au bord.

Ping OK + site mort : souvent **notre** firewall (ou Apache), pas « Internet cassé ». Inversement, un DDoS volumétrique peut tuer le lien **avant** que `iptables` ne voie les paquets : d’où HCAP / VAC.

Doc OVH :

https://www.ovhcloud.com/fr/security/anti-ddos/ddos-attack-mitigation/

https://www.ovhcloud.com/fr/security/anti-ddos/

### Backbone router

### Edge Firewall

Un firewall, c’est un **filtre de paquets** : selon l’adresse, le port, le sens (entrant / sortant), on **accepte**, on **refuse**, ou on **ignore**. Tant qu’une règle refuse le HTTPS, le navigateur n’atteint pas Apache — même si le DNS et le routage sont justes.

### Datacenters router

## Questions à garder en tête

- [ ] Pourquoi un firewall (OVH **et** VPS) ?
- [ ] Que se passe-t-il si **aucune** règle ne correspond ?
- [ ] Pourquoi l’**ordre** des règles compte-t-il ?
- [ ] Pourquoi le trafic ne va-t-il pas directement sur le VPS ? Quelles couches sont **OVH** ?
