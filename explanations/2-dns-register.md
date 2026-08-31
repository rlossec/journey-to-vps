# 2. Enregistrement DNS

À l’étape 1, un serveur autoritaire (`dns13.ovh.net`) a répondu :

`readresolve.tech` → `54.36.100.9`

Cette réponse n’est pas calculée au moment de la requête. Quelqu’un l’a **écrite** avant. Ici on passe du côté **administrateur** : comment on déclare le lien entre le nom et le VPS.

Réserver le nom `readresolve.tech` ne suffit pas. Sans enregistrements, la chaîne de résolution aboutit à un serveur de noms qui n’a **rien** à renvoyer — ou une réponse qui ne pointe pas vers notre machine.

## De la réservation à la zone

Trois objets distincts, souvent confondus :

| Objet                  | Qu’est-ce que c’est                                  | Qui le donne / le tient                                    |
| ---------------------- | ---------------------------------------------------- | ---------------------------------------------------------- |
| **Nom de domaine**     | L’étiquette lisible (`readresolve.tech`)             | On la **réserve** (registrar, ici via OVH)                 |
| **IP publique du VPS** | L’adresse de la machine sur Internet (`54.36.100.9`) | **OVH** l’attribue au VPS à la création                    |
| **Zone DNS**           | La liste des enregistrements pour ce nom             | On la **remplit** (console OVH). C’est sous notre contrôle |

L’IP n’est pas « le nom de domaine ». C’est l’adresse du VPS. Le record **A** copie cette IP dans la zone pour que les résolveurs sachent où envoyer le trafic web.

Pourquoi cette IP identifie **ce** VPS : sur Internet, un paquet pour `54.36.100.9` est livré à **cette** machine, pas à une autre. Changer de VPS (ou d’IP) sans mettre à jour le A, c’est garder un nom qui pointe vers l’ancienne adresse — ou vers plus rien d’utile.

Un **sous-domaine** (`www.readresolve.tech`) n’est pas un second achat : c’est une **ligne de plus** dans la même zone. Ici, `www` a son propre **A** vers la **même** IP `54.36.100.9`. Plusieurs noms peuvent donc désigner le même serveur.

La zone vit chez les serveurs **autoritaires** déjà vus en lecture : `dns13.ovh.net` et `ns13.ovh.net`. On n’écrit pas sur la racine ni sur le TLD `.tech`. On écrit **le contenu** que ces deux serveurs serviront.

## Types d’enregistrements

Chaque ligne de la zone a un **type**. Le résolveur demande un type précis (`A`, `NS`, `MX`…). Un nom sans le bon type, c’est un nom qui existe mais ne sert pas à ce qu’on croit (un site sans A n’a pas d’IPv4).

### A et AAAA — le nom vers une IP

- **A** : nom → adresse **IPv4**. C’est **le** record du cas d’étude : `readresolve.tech` → `54.36.100.9`.
- **AAAA** : même idée en **IPv6**. Ici, il n’y en a pas : le VPS est annoncé en IPv4 seulement.

Sans A (et sans AAAA), le navigateur n’obtient pas d’IP : la résolution de l’étape 1 s’arrête avant le routage.

`www.readresolve.tech` a aussi un **A** vers `54.36.100.9`. Ce n’est pas un CNAME : deux enregistrements A, deux noms, une seule machine.

### NS — qui a le droit de répondre pour la zone

**NS** (Name Server) : quels serveurs sont **autoritaires** pour le domaine.

Ici : `dns13.ovh.net` et `ns13.ovh.net`.

On les a **lus** à l’étape 1 (le TLD `.tech` nous y envoie). On les **écrit** aussi dans la zone. Ce sont eux qui détiennent la vérité sur le A, le MX, etc.

### CNAME — un nom vers un autre nom

**CNAME** : alias. Il ne pointe **pas** vers une IP, mais vers **un autre nom**, qui devra lui-même être résolu (souvent jusqu’à un A).

Utile pour `blog.readresolve.tech` → `readresolve.tech`, par exemple. On ne met en général **pas** de CNAME sur l’apex (`readresolve.tech` lui-même) : l’apex a déjà NS, MX, etc., et un CNAME ne cohabite pas avec d’autres types sur le même nom.

Sur ce domaine, `www` n’est **pas** un CNAME : c’est un second A. Les deux approches « collent » `www` au site ; le CNAME dit « va voir cet autre nom », le A dit « voici l’IP ».

### MX — le courrier (bref)

**MX** (Mail eXchange) : où envoyer les **e-mails** du domaine, pas les pages web.

Ici OVH a posé, entre autres, `mx4.mail.ovh.net` (priorité 1) et `mx3.mail.ovh.net` (priorité 10). Le site et le mail peuvent vivre sur des machines **différentes**. Le A du web n’achemine pas le courrier.

### TXT — du texte, souvent pour prouver ou sécuriser (bref)

**TXT** : texte libre. Usages courants : prouver qu’on possède le domaine (Google, Let’s Encrypt…), ou des règles mail (**SPF**).

Ici : `v=spf1 include:mx.ovh.com ~all` — « les serveurs mail OVH ont le droit d’envoyer du courrier pour ce nom ». Ça n’a aucun effet sur l’affichage du site.

## TTL — durée de vie du cache

Chaque enregistrement porte un **TTL** (Time To Live) : combien de **secondes** un résolveur a le droit de **garder** la réponse sans redemander à l’autoritaire.

Valeurs observées sur `readresolve.tech` :

| Record                 | TTL typique  |
| ---------------------- | ------------ |
| A (`readresolve.tech`) | 3600 (1 h)   |
| NS, MX, A de `www`     | 3600         |
| TXT (SPF)              | 600 (10 min) |

C’est **nous** qui le choisissons (ou le registrar par défaut) **à l’écriture**. L’étape 1 a montré l’effet à la **lecture** : cache navigateur / OS / résolveur, et **propagation**.

Si on change le A (nouvelle IP de VPS) :

- l’autoritaire sert tout de suite la nouvelle valeur ;
- les résolveurs qui ont encore l’ancienne dans le cache attendent la fin du TTL.

D’où : un changement DNS **n’est pas immédiat partout**. Ce n’est pas « Internet met 48 h », c’est surtout le TTL (ici, jusqu’à une heure pour le A). Avant une bascule d’IP, on **abaisse** souvent le TTL à l’avance, on attend qu’il expire, puis on change l’IP.

## Questions à garder en tête

- [ ] Qu’est-ce que la propagation DNS ?
- [ ] Pourquoi un domaine a-t-il besoin d’enregistrements DNS ?
- [ ] D’où vient l’adresse IP ? Pourquoi identifie-t-elle **ce** VPS ?
- [ ] Plusieurs noms de domaine peuvent-ils pointer vers la même IP ?
- [ ] Que se passe-t-il si l’IP du VPS change ?


L’IP est connue **et** déclarée. Le navigateur peut l’utiliser pour **trouver le chemin** jusqu’à OVH — étape 3.
