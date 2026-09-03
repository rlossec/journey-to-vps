# 4. Infrastructure de l'hébergeur (OVH)

Le routage (étape 3) a amené le paquet dans le réseau OVH. Mais il n'arrive pas **directement** sur le VPS. Plusieurs couches, **toutes gérées par OVH**, filtrent et aiguillent le trafic avant qu'il ne touche notre machine.

On n'a **pas la main** sur ces éléments — on les comprend pour savoir ce qui se passe **avant** notre configuration (étape 5).

## Attaques DDoS — pourquoi OVH filtre en amont

### DoS et DDoS

**DoS** (Denial of Service) : une source sature la cible.
**DDoS** (Distributed) : **beaucoup** de sources (botnet). Plus dur à couper : ce n'est plus « bloquer une IP ».

```
DoS                         DDoS
  attaquant ──► VPS          des milliers de machines
                             ──► ──► VPS
                             ──►
```

Trois familles, sans entrer dans les protocoles :

- **Volumétrique** — noyer la **bande passante** (le tuyau du VPS est minuscule face à un flood).
- **Surcharge de ressources** — trop de paquets / connexions, la machine s'épuise.
- **Exploitation** — viser un trou logiciel. Ce n'est plus seulement « trop de trafic ».

Un VPS tout juste créé, **IP publique déjà routée**, firewall local encore ouvert : cible facile. C'est arrivé **dès la mise en service** du serveur de formation. D'où l'intérêt du filtrage **en amont**, chez l'hébergeur.

On laisse de côté le cas particulier des **serveurs de jeux** (offre anti-DDoS GAME OVH).

## Étapes dans OVH

Une fois le trafic dans le réseau OVH, il traverse plusieurs étages, **tous côté OVH** :

1. **HCAP** (anti-DDoS) — premier filtre, parfois **avant** le cœur du datacenter.
2. **Backbone routers** — gros routeurs qui relient les points de présence et les DC.
3. **Edge Firewall** — filtrage d'entrée du réseau / du site.
4. **Datacenter routers** — dernier aiguillage vers l'hôte qui porte le VPS.

!image.png

> **Pourquoi plusieurs couches ?** Isoler les clients, absorber un flood **avant** qu'il n'atteigne une petite VM, ne pas exposer l'hyperviseur comme s'il était sur Internet nu. Rien de tout ça n'est configurable dans notre zone DNS ni dans notre VPS.

### HCAP, VAC

| Brique | Rôle | Qui configure |
| --- | --- | --- |
| **HCAP** (_Hardware Client Amplitude Policer_) | Premier filtre, aux **points de présence** (là où OVH se raccorde aux autres opérateurs). Coupe / bride le volumétrique **avant** le datacenter. Peut **décharger** les nœuds VAC. | OVH (automatique) |
| **VAC** | Centres de « lavage » du trafic : on tente de laisser passer le légitime, d'écarter le flood. | OVH (mitigation auto) |

Le HCAP protège **le tuyau et le réseau**. Il ne sait pas « notre appli n'écoute que 443 ». Le détail métier (SSH, HTTP, rien d'autre) reste à configurer **sur le VPS** — étape 5.

Ping OK + site mort : souvent **notre** configuration VPS (firewall ou Apache), pas « Internet cassé ». Inversement, un DDoS volumétrique peut tuer le lien **avant** que le VPS ne voie les paquets : d'où HCAP / VAC.

Doc OVH :

https://www.ovhcloud.com/fr/security/anti-ddos/ddos-attack-mitigation/

https://www.ovhcloud.com/fr/security/anti-ddos/

### Backbone routers

Gros routeurs internes qui relient les points de présence OVH et les datacenters entre eux.

### Edge Firewall (pare-feu de bordure)

Filtre **sans état** (ACL : IP, ports, protocoles) intégré à l'anti-DDoS. Peut s'activer automatiquement pendant une attaque. On peut y poser quelques règles dans l'espace client, mais c'est **côté OVH**, pas sur notre machine.

> Un firewall, c'est un **filtre de paquets** : selon l'adresse, le port, le sens, on **accepte**, **refuse** ou **ignore**. L'Edge Firewall fait ça **avant** que le paquet n'arrive au VPS. Le firewall **sur** le VPS (`iptables`) fera le même travail **après** — on le détaille à l'étape 5.

### Datacenter routers

Dernier aiguillage vers l'hôte physique qui porte le VPS.

## Questions à garder en tête

- Pourquoi le trafic ne va-t-il pas directement sur le VPS ?
- Pourquoi plusieurs couches de protection / d'aiguillage ?
- Lesquelles sont gérées par **OVH** ? Lesquelles par **nous** ?
- Qu'est-ce qui distingue l'Edge Firewall (OVH) du firewall VPS (`iptables`) ?

Le paquet a traversé l'infrastructure OVH. Il arrive enfin sur notre machine — étape 5 : la configuration qu'on maîtrise.
