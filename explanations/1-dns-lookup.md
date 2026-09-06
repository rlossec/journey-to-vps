# 1. Résolution DNS

## 1.1. Théorie

A l'étape précédente, on a accédé à `https://mbr-raphael.readresolve.tech`.

Comme pour n’importe quel site, il va devoir transformer l'url en IP. Les outils informatiques fonctionnent avec des nombres et pour le Protocol Internet, c’est l’IP qui fait foi.

Quand on fait cette conversion, on dit que l’on résout le nom de domaine.

On imagine dans un premier temps qu'on vient d'emménager, qu'on a acheté un nouvel ordinateur et qu'on consulte notre site que l'on vient de lancer à la seconde prêt.
Un ensemble de coïncidence tout a fait classique.

Notre navigateur via notre box va faire intervenir un Résolveur DNS (de notre FAI) qui aura la tâche de trouver l'IP associée à l'url que l'on a donné.

Pour cela il va déjà analysé notre url et faire une requête DNS.

### 0 - Compréhension d'url

Pour bien comprendre la suite des étapes il faut bien analyser l'url et son découpage.

[Schema Découpage Url](../excalidraw/1-dns-lookup/1-1-url-explanations.excalidraw)

On a plusieurs parties,

- `.tech` correspond au **Top Level Domain** : TLD
- `readresolve` correspond **au nom de domaine**
- `mbr-me` enfin correspond à un sous domaine

Illustrons cela avec Google et ses services.

[Schema Arbre Url](../excalidraw/1-dns-lookup/1-2-url-tree.excalidraw)

On a cet arbre avec

- en haut le root,
- puis `.com` qui est l'extension la plus utilisé et donc le TLD ici.
- puis le nom de domaine : `google`
- et de multiples sous-domaines, qui représente les différences services google.

### Intro resolver

Avec cela en tête passons au fonctionnement du Resolver DNS.
Comme je l'avais indiqué, c'est propre à notre FAI, même si des alternatives existent.

[Schema Resolution DNS](../excalidraw/1-dns-lookup/1-5-dns-lookup.excalidraw)

### Etape 1 : Root Server

Pour la première étape notre résolveur va se concentrer sur la terminaison.
Son objectif, trouver le TLD server qui s'occupe de la terminaison `.tech`

Il doit pour cela faire la demande à ce qu'on appelle les Root servers.
Ils sont au nombre de 13 dans le monde en terme d'identité. Ils sont symbolisés par les lettres de A à M.

Evidemment, pour chacun de ces 13 identités, ils existent des centaines d'instance dans le monde. Internet ne peut pas dépendre d'uniquement 13 entités.

Parmi ces 13 root server, le plus rapide répondera donc à la requête DNS par l'ip du serveur TLD, c'est à dire ici celui qui gère le `.tech`. Les serveurs root n'ont pas d'autre informations.

A la fin de l'étape, retour donc au Résolveur DNS de notre FAI mais avec l'information ou **l'ip du TLD responsable des `.tech`.**

### Etape 2 : TLD Server

Evidemment maintenant qu'on a l'info du TLD Server, on va lui soumettre une requête, du type "Eh toi qui connait les .tech, tu saurais qui est responsable DNS de ce nom de domaine : `readresolve` ?
Et il va nous répondre pas avec l'ip finale du VPS mais avec ce qu'on appelle le serveur autoritaire de notre nom de domaine. Comme notre VPS est à OVH, il s'agira d'un serveur DNS d'OVH.

A la fin de l'étape retour donc au Résolveur DNS de notre FAI mais avec **l'ip du serveur autoritaire pour `readresolve.tech`**.

### Etape 3 : Serveur autoritaire

Et nous voilà à la dernière étape, on connait le serveur autoritaire qui lui a l'information de l'ip du VPS ! On demande donc cet ip au serveur autoritaire.

Et fin de la résolution !

Si on revient à notre url, on a pas parlé du sous domaine. En effet, ce n'est pas la responsabilité du Resolver DNS, on en parlera plus tard.

### Récap

Quatre rôles à distinguer :

| Rôle                   | Question à laquelle il répond               | Exemple ici                                                        |
| ---------------------- | ------------------------------------------- | ------------------------------------------------------------------ |
| **Résolveur récursif** | « Trouve-moi l’IP, je m’occupe du reste »   | FAI, Google `8.8.8.8`, Cloudflare `1.1.1.1`                        |
| **Racine (Root)**      | « Qui gère ce TLD ? »                       | 13 identités **A** à **M**, des milliers d’instances dans le monde |
| **TLD**                | « Qui est autoritaire pour ce domaine ? »   | `.tech` → `ns01.trs-dns.com`, …                                    |
| **Autoritaire**        | « Quelle est **la** réponse pour ce nom ? » | `dns13.ovh.net` / `ns13.ovh.net`                                   |

La racine **ne connaît pas** l’IP de `readresolve.tech`. Elle sait seulement où sont les serveurs `.tech`. Le TLD **ne connaît pas** forcément l’IP non plus : il pointe vers les serveurs de noms du domaine.

## 1.2. Pratique

Voilà pour la théorie.

Allons expérimenter via des commandes si on peut suivre cette résolution DNS.

On va se placer sur le serveur VPS.
Une commande permet de suivre toutes les étapes de la résolution pour

```bash
dig +trace mbr-me-readresolve.tech
```

```
; <<>> DiG 9.20.24-1ubuntu0.2-Ubuntu <<>> +trace readresolve.tech
;; global options: +cmd
.                       86366   IN      NS      e.root-servers.net.
.                       86366   IN      NS      f.root-servers.net.
.                       86366   IN      NS      g.root-servers.net.
.                       86366   IN      NS      h.root-servers.net.
.                       86366   IN      NS      i.root-servers.net.
.                       86366   IN      NS      j.root-servers.net.
.                       86366   IN      NS      k.root-servers.net.
.                       86366   IN      NS      l.root-servers.net.
.                       86366   IN      NS      m.root-servers.net.
.                       86366   IN      NS      a.root-servers.net.
.                       86366   IN      NS      b.root-servers.net.
.                       86366   IN      NS      c.root-servers.net.
.                       86366   IN      NS      d.root-servers.net.
.                       86366   IN      RRSIG   NS 8 0 518400 20260908050000 20260826040000 57780 . U0SVQzV1Q05Q4r0zFT8ZucZmAR+VYExL4MqcfKGDu6phZV/rus3jZrmH mIYFFmp6BuJ0u43DOrrw2eSMSDaZNEFHgj2rbHNh+4QMA+/R+0oP7sc5 j1aCRerpoChqNABHAOlqGwkEM8DRBNIN2NBJEJQoR1kLvgE0j9zvE1do +O0F0gAVyYQEmq1fBJhvhhNtJLYlcrCmkwAJ166TwslGjmAhGX1PDHQv xrU+8N/PdqWNvN09PTSj1WvGJscQ7wduadDaIr6uWyEgXGHKoQw9fKco 62sjGvq7U3IS7U3ExE+lPCsgJcNmQaVQTwVnunCw9+RVMlt3i+Uyst9o F2YJiA==
;; Received 525 bytes from 127.0.0.53#53(127.0.0.53) in 2 ms

;; communications error to 192.58.128.30#53: timed out
;; communications error to 192.58.128.30#53: timed out
;; communications error to 192.58.128.30#53: timed out
;; communications error to 199.7.83.42#53: timed out
tech.                   172800  IN      NS      ns01.trs-dns.com.
tech.                   172800  IN      NS      ns01.trs-dns.net.
tech.                   172800  IN      NS      ns10.trs-dns.org.
tech.                   172800  IN      NS      ns10.trs-dns.info.
tech.                   86400   IN      DS      2185 13 2 E796AB04119E87F72A094522E281F7D125E730B990B638BE308133E2 F5512752
tech.                   86400   IN      RRSIG   DS 8 1 86400 20260908050000 20260826040000 57780 . SC/qWpw0aKuhJX5x+ZJYkwOtgzVfUiHutUk4NENpD1uLwD9ByVC7zpjL hB0oXK6G4NxQiqsORhdsUkcvy4OYmgq58yNlMrjmjd2PFiUr2TLzOSM0 nSyHl5UF4sNEoq3Od6Ble3tGT1GMp3kVymmNyFyXsM+X10FdPJCPgOhY lKbbXEH/hNmKEgqqDDctnrX5LehuKBIJuQno5HOZ3tv1a55h4rjRDnFJ JElznvLqrntH/IiUBr7u9eVfoLlODbj31eq94CR3icVx6Kh77FhyfFBN utaSlyO9LNd+3HTdHs2bKAiu0NtcI4/leu047hAHQxUMmNJb7RHz7hPL DviBhg==
;; Received 677 bytes from 2001:500:2d::d#53(d.root-servers.net) in 4 ms

;; communications error to 64.78.205.1#53: timed out
readresolve.tech.       900     IN      NS      ns13.ovh.net.
readresolve.tech.       900     IN      NS      dns13.ovh.net.
readresolve.tech.       900     IN      DS      4496 8 2 2EF6CD16781522CF1FE57E87B7D9984358E32BE85B21F2C20BF4FF23 7584FB29
readresolve.tech.       900     IN      RRSIG   DS 13 2 900 20260918030447 20260819104906 6357 tech. rgzAcXLxIzIMj3WBPFHRMUnNFqeozbXtePFTebPA2wYy2BMYMg+SDkuf vGJdluIcQB8OfarvyEf2QTKbU9muBg==
;; Received 239 bytes from 2620:171:813:1534:8::1#53(ns10.trs-dns.org) in 6 ms

readresolve.tech.       3600    IN      A       54.36.100.9
readresolve.tech.       3600    IN      RRSIG   A 8 2 3600 20260911054305 20260812054305 2590 readresolve.tech. RI6UDGtAH7C7xuuE5AjszBzNRTGs94pR1ospbPgXLyS6P2vZ4cjCf8W0 im/4xy6AKuoWX+IEEqjKjOMDrucw2u30SN71beHXLTVgrYfphd7Ob/z4 UluahMPfGlaj7tbEvyW6yf9QAINB4ZbKxCf8EyZOlgV6TTx9mqbpesFH f8E=
;; Received 265 bytes from 5.39.112.241#53(ns13.ovh.net) in 2 ms
```

## 1.3. Cache

Maintenant revenons un peu sur notre cas particulier : nouvel appartement, nouvel ordinateur, nouveau site.

En vrai, la résolution DNS prend des raccourci. Pour quasi chaque intervenant, navigateur, OS, resolveur, les serveurs DNS, ils ont un cache qui peut contenir l'information et permettre d'éviter des étapes.

Le cache **accélère** (moins de allers-retours) et **soulage** les serveurs root et les TLD. Contrepartie : une modification DNS n’est pas visible partout au même moment.

## Transition

Mais comment lorsqu'on créé son serveur et son site, on va communiquer l'information à ces DNS ?

## Questions à garder en tête

- [x] Pourquoi deux utilisateurs peuvent-ils obtenir des réponses différentes ?
- [x] En quoi le cache améliore-t-il les performances ?
- [x] Pourquoi une modification DNS n’est-elle pas visible tout de suite ?
