# 3. Routage Internet

L’étape 1 a donné une IP. L’étape 2 a expliqué comment elle a été **déclarée**. Le navigateur connaît maintenant `54.36.100.9`. Il n’a **pas** le plan du trajet : il envoie un paquet vers cette adresse, et le réseau se débrouille.

## Sur Internet : des réseaux qui se passent le paquet

Internet n’est pas un câble unique vers OVH. C’est un assemblage de **réseaux d’opérateurs** (FAI, OVH, transitaires…). Chaque réseau est un **système autonome** (AS) : il gère ses propres routeurs et ses propres adresses.

**BGP** (Border Gateway Protocol), en une phrase : les opérateurs s’annoncent _« je sais joindre tel bloc d’IP »_. OVH annonce, entre autres, le préfixe qui contient `54.36.100.9`. Les FAI apprennent un chemin vers ce préfixe. On n’entre pas dans les messages BGP ni dans les tables.

Le navigateur ne calcule pas la route. Il envoie vers l’IP ; **les routeurs** choisissent le prochain saut d’après ce qu’ils ont appris. Deux clients (Free, Orange, 4G) peuvent emprunter des chemins **différents** pour la même IP.

## Pratique

`traceroute` / `tracert` envoie des sondes avec un **TTL IP** qui augmente. Chaque routeur qui expire le TTL signale « je suis là ». On obtient une **liste de sauts**, pas une carte officielle OVH.

Exemple réel (poste derrière une box, FAI Free, vers `54.36.100.9`) :

```
 1  192.168.1.254      box (réseau local)
 2  194.149.174.96     premier saut opérateur
 3  212.27.35.6        réseau Free
 4  * * *              pas de réponse ICMP
 5  213.186.32.181     entrée visible du réseau OVH
 6–7  * * *
 8  57.130.3.80        encore OVH
 9  37.59.16.2
 …  d’autres sauts OVH, plusieurs * * *
19  54.36.100.9        le VPS
```

Les `* * *` ne veulent pas dire que le paquet est mort : beaucoup de routeurs **ignorent ICMP** (politique, charge, matériel). Le trafic utile (HTTPS) passe souvent alors que traceroute reste muet. Inversement, un ping OK ne garantit pas HTTPS.

On **n’identifie pas** sur cette liste « ceci est le HCAP » ou « ceci est le pare-feu de bordure ». Traceroute montre des **IP de routeurs**, pas les noms des briques commerciales OVH. Ces briques, on les pose par le **modèle** ci-dessous.

## Dans OVH : le paquet n’arrive pas « sur le VPS »

Une fois le trafic dans le réseau OVH, il ne saute pas du premier routeur OVH à `54.36.100.9`. Plusieurs étages, **tous côté OVH** :

1. **HCAP** (anti-DDoS) — premier filtre, parfois **avant** le cœur du datacenter. Détail des attaques et de la mitigation : **étape 4**.
2. **Backbone** — gros routeurs qui relient les points de présence et les DC.
3. **Pare-feu de bordure** — filtrage d’entrée du réseau / du site.
4. **Routeur de datacenter** — dernier aiguillage vers l’hôte qui porte le VPS.
5. **VPS** — enfin la machine. Encore un firewall **à nous** (`iptables`) avant Apache.

Pourquoi plusieurs couches : isoler les clients, absorber un flood **avant** qu’il n’atteigne une petite VM, ne pas exposer l’hyperviseur comme s’il était sur Internet nu. Rien de tout ça n’est configurable dans notre zone DNS ni dans Apache.

| Côté Internet / OVH                        | Côté formation                              |
| ------------------------------------------ | ------------------------------------------- |
| Tables de routage, BGP, chemin jusqu’à OVH | —                                           |
| HCAP, backbone, bordure, routeur DC        | Firewall du VPS (étape 4), Apache (étape 5) |

DevTools → Réseau : on voit une requête vers `readresolve.tech`, une latence totale. On ne voit **aucun** saut. Même information que le navigateur : nom, puis IP, puis « ça a répondu » ou non.

## Questions à garder en tête

- Une fois l’IP connue, comment le serveur est-il atteint ?
- Le navigateur connaît-il la route complète ?
- Pourquoi le trafic ne va-t-il pas **directement** sur le VPS ?
- Pourquoi plusieurs couches de protection / d’aiguillage ?
- Lesquelles sont gérées par **OVH** ?

Le paquet est (presque) arrivé. Il reste les **douanes** : d’abord OVH, puis les règles qu’on met sur le VPS — étape 4.
