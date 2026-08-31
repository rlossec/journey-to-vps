# 3. Routage Internet

L’étape 1 a donné une IP. L’étape 2 a expliqué comment elle a été **déclarée**. Le navigateur connaît maintenant `54.36.100.9`. Il n’a **pas** le plan du trajet : il envoie un paquet vers cette adresse, et le réseau se débrouille.

## Sur Internet : des réseaux qui se passent le paquet

Internet n’est pas un câble unique vers OVH. C’est un assemblage de **réseaux d’opérateurs** (FAI, OVH, transitaires…). Chaque réseau est un **système autonome** (AS) : il gère ses propres routeurs et ses propres adresses.

**BGP** (Border Gateway Protocol), en une phrase : les opérateurs s’annoncent _« je sais joindre tel bloc d’IP »_. OVH annonce, entre autres, le préfixe qui contient `54.36.100.9`. Les FAI apprennent un chemin vers ce préfixe. On n’entre pas dans les messages BGP ni dans les tables.

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

## Questions à garder en tête

- Une fois l’IP connue, comment le serveur est-il atteint ?
- Le navigateur connaît-il la route complète ?
- Pourquoi le trafic ne va-t-il pas **directement** sur le VPS ?
- Pourquoi plusieurs couches de protection / d’aiguillage ?
- Lesquelles sont gérées par **OVH** ?

Le paquet est (presque) arrivé. Il reste les **douanes** : d’abord OVH, puis les règles qu’on met sur le VPS — étape 4.
