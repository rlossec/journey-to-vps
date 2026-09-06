# 3. Routage internet

À l'étape précédente, notre navigateur a réussi à obtenir une information essentielle :

> `readresolve.tech` → `54.36.100.9`

On connaît donc maintenant **la destination**. Mais il nous manque encore une information importante :

> **Comment est-ce que notre ordinateur va rejoindre cette adresse IP ?**

Parce que connaître l'adresse d'une maison ne suffit pas pour y arriver. Il faut encore savoir **par quelles routes passer**. C'est justement le rôle du routage.

## 3.1. Théorie — Routage

Le routage, c'est le mécanisme qui permet aux données de déterminer **vers quel prochain équipement ils doivent être envoyés** pour se rapprocher de leur destination.

Notre ordinateur ne connaît évidemment pas Internet dans son ensemble. Il ne possède pas une carte complète du chemin.

À la place, chaque équipement réseau possède une **table de routage**.
Cette table lui permet de répondre à une question beaucoup plus simple :

> **« Pour cette adresse IP de destination, par où est-ce que j'envoie le paquet ? »**

Et ce fonctionnement se répète de routeur en routeur.

## 3.2. Pratique : ip route

Revenons à notre ordinateur. Il veut envoyer des données à : `54.36.100.9`

Mais cette adresse n'est évidemment pas dans notre réseau local. Notre ordinateur va donc l'envoyer à ce qu'on appelle sa **passerelle par défaut**. Dans une installation classique, cette passerelle est notre box ou notre routeur.

Il possède donc une table de routage. On peut la consulter.

### Commandes

```
ip route
```

Le mot important ici est :

**`default`**

Cela signifie en quelque sorte :

> « Pour toutes les destinations pour lesquelles je n'ai pas de route plus précise, utilise cette passerelle. »

On peut donc avoir plusieurs routes :

```
192.168.1.0/24     → réseau local
default            → 192.168.1.1
```

Si je veux joindre une machine de mon réseau local, je peux lui parler directement.

Si je veux joindre `54.36.100.9`, ce n'est pas mon réseau → je passe par la route par défaut.

---

## 3.3. Choix

Évidemment, un routeur peut avoir énormément de routes. Il faut donc une règle pour déterminer laquelle utiliser. Une notion importante est celle du **préfixe réseau**.

Par exemple :

```
54.36.0.0/16
54.36.100.0/24
```

La deuxième route est plus précise que la première.

Si on cherche :

```
54.36.100.9
```

elle correspond aux deux.

Mais le routeur choisira la route la plus précise :

```
54.36.100.0/24
```

C'est ce qu'on appelle le principe du **longest prefix match**.

On ne va pas rentrer beaucoup plus loin dans les détails pour l'instant.

L'idée à retenir est simplement :

> **Un routeur regarde l'adresse de destination et cherche la route la plus précise qu'il possède.**

## 3.4. Pratique — Traceroute

Linux

```
traceroute 54.36.100.9
```

Sous Windows :

```
tracert 54.36.100.9
```

La commande va afficher une succession de **sauts**.

---

## Transition

Notre paquet a maintenant trouvé son chemin jusqu'au réseau qui héberge notre VPS.

Mais on vient de dire quelque chose d'important :

> **« réseau OVH »**

Et notre histoire ne s'arrête pas là.

Notre VPS est hébergé dans une infrastructure OVH, avec ses propres équipements réseau et ses propres mécanismes de sécurité.

Alors une nouvelle question apparaît :

> **Une fois arrivé chez OVH, qu'est-ce qui se passe exactement avant que le paquet atteigne notre VPS ?**

C'est ce que nous allons regarder maintenant.

## Questions à garder en tête

- [x] Une fois l’IP connue, comment le serveur est-il atteint ?
- [x] Le navigateur connaît-il la route complète ?
- [x] Pourquoi le trafic ne va-t-il pas **directement** sur le VPS ?
- [x] Pourquoi plusieurs couches de protection / d’aiguillage ?
- [x] Lesquelles sont gérées par **OVH** ?
