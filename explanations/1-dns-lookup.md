# 1. Résolution DNS

## 1.1. Théorie - Diapo 8

Les divers dispositifs informatiques fonctionnent avec des nombres. En l'occurence quand on accède à un site internet, on doit rapatrier les donnés pour le construire. Et avant même de les rapatrier on doit connaître l'endroit où sont ces données.  

Ainsi nous nous tapons l'url `https://google.com`, le navigateur doit déjà trouver l'IP du serveur qui héberge le site.  

> **_Définition_**  
> Quand le navigateur convertit l'`hostname` en IP on dit que l’on **résout le nom d'hôte ou hostname**.  

> **Hypothèse de départ**  
>  On imagine qu'on vient d'emménager, qu'on a acheté un nouvel ordinateur et qu'on consulte notre site que l'on vient de lancer à la seconde prêt.  

Quand nous validons l'url `https://google.com` dans le navigateur, la première mission du navigateur va être de **Résoudre le `hostname`** `google.com`.  
A ce moment le navigateur via notre box va faire intervenir un **DNS Resolver** qui aura la tâche de trouver l'IP associée au `hostname` que l'on a donné.

Le résolveur DNS va être central par la suite.

### 1.1.1. - Compréhension du `hostname`

Pour bien comprendre la suite des étapes il faut bien analyser le `hostname` et son découpage.

Prenons le nom d'hôte de notre cas d'étude : `https://mbr-me.readresolve.tech`

[Schema Découpage Hostname(../excalidraw/1-dns-lookup/1-1-url-explanations.excalidraw)

Dans `mbr-me.readresolve.tech`, on a plusieurs parties :

- `.tech` correspond au **Top Level Domain** : TLD
- `readresolve` correspond **nom de domaine**
- `mbr-me` enfin correspond à un **sous domaine**

Illustrons cela avec Google et ses services.

[!google-hostname-example](../img/1-google-hostname-example.png)
[Schema Arbre Url](../excalidraw/1-dns-lookup/1-2-url-tree.excalidraw)

On a cet arbre avec :

- en haut le root,
- puis `.com` le TLD ici.
- puis le nom de domaine : `google`
- et de multiples sous-domaines, qui représente les différences services Google.

### 1.1.2. Les acteurs de la résolution

Comme on l'a indiqué le personnage principal de la résolution est le Resolver DNS. Il va centralisé la Résolution et aller demander à chaque acteur secondaire, de l'aide pour résoudre le `hostname`.

Ces trois acteurs sont très hiérachiques :
- les **root servers** en haut
- les **TLD servers** au milieu
- le **serveur d'autorité** en bas

Chacun va renvoyer au suivant, de façon très administrative, sans avoir plus d'informations.

![Gif Astérix : Formulaire A38](../img/asterix-A38.gif)

### 1.1.3. Que savent ils chacun ?

Partons d'en bas, le serveur d'autorité pour notre exemple `mbr-me.readresolve.tech` est un serveur DNS d'OVH qui va contenir tous les associations domaine <-> IP des sites hébergés chez eux.
Pour trouver ce serveur d'autorité, un TLD server contient l'information de ce serveur d'autorité (parmi plein d'autres).
Et pour trouver le serveur TLD, il faut solliciter un root serveur.

### 1.1.4. L'orchestration du resolver

Ainsi, quand le DNS Resolver recoit `mbr-me.readresolve.tech`, il ne sait évidemment pas que c'est chez OVH, il n'a que ce nom de domaine.
Par contre il connait la procédure :

1.  le resolver doit d'abord solliciter les "patrons" : les root servers. Il leur dit "Alors là je cherche les responsables des `.tech`
2.  le root serveur le plus rapide lui répond "le responsables des .tech c'est le serveur TLD `ns01.trs-dns.com`"
3.  le resolveur sollicite donc le serveur TLD `ns01.trs-dns.com` : Qui s'occupe de `readresolve` ? (ou qui est le serveur d'autorité)
4.  le serveur TLD `ns01.trs-dns.com` répond: "Ah le serveur d'autorité pour `readresolve` c'est `ns13.ovh.net`"
5.  le resolveur sollicite donc le serveur d'autorité `ns13.ovh.net` et demande "Tu dois connaitre l'IP de mbr-me.readresolve.tech, c'est ton boulot"
6.  et enfin le serveur d'autorité donne l'IP du serveur qui héberge : `54.36.100.8` :sweat_smile:

[Schema Résolution DNS](../excalidraw/1-dns-lookup/1-5-dns-lookup.excalidraw)

### 1.1.5. Récap

Quatre rôles à distinguer :

| Rôle                   | Question à laquelle il répond             | L'acteur                                                           |
| ---------------------- | ----------------------------------------- | ------------------------------------------------------------------ |
| **Résolveur récursif** | Aucune, il trouve ceux qui répondent      | FAI                                                                |
| **Root**               | « Qui gère ce TLD ? »                     | 13 identités **A** à **M**, des milliers d’instances dans le monde |
| **TLD**                | « Qui est autoritaire pour ce domaine ? » | `.tech` → `ns01.trs-dns.com`, …                                    |
| **Autoritaire**        | « Quelle est l'ip pour ce nom ? »         | `dns13.ovh.net` / `ns13.ovh.net`                                   |

Le serveur root **ne connaît pas** l’IP de `readresolve.tech`. Il sait seulement où sont les serveurs `.tech`. Le TLD **ne connaît pas** forcément l’IP non plus : il pointe vers les serveurs faisant autorité.

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
