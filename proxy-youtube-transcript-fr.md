# Pourquoi Nginx est-il appelé un reverse proxy ?

Qu’est-ce qu’un proxy, au juste ? Voyons cela.

Deux types courants de proxy sont le **forward proxy** et le **reverse proxy**.

## Forward proxy

Un forward proxy est un serveur placé entre un groupe de machines clientes et Internet. Quand ces clients envoient des requêtes vers des sites, le forward proxy joue le rôle d’intermédiaire : il intercepte ces requêtes et parle aux serveurs web **au nom** de ces machines clientes.

### Pourquoi voudrait-on faire ça ?

#### 1. Protéger l’identité en ligne du client

En se connectant à un site via un forward proxy, l’adresse IP du client est masquée au serveur. Seule l’IP du proxy est visible. Il est plus difficile de remonter jusqu’au client.

#### 2. Contourner des restrictions de navigation

Des institutions (États, écoles, grandes entreprises) utilisent des firewalls pour limiter l’accès à Internet. En se connectant à un forward proxy situé *hors* de ces firewalls, le client peut parfois contourner ces restrictions.

Cela ne fonctionne pas toujours : le firewall peut aussi bloquer les connexions vers le proxy.

#### 3. Bloquer l’accès à certains contenus

Les écoles et les entreprises configurent souvent le réseau pour que tous les clients passent par un proxy, avec des règles de filtrage (réseaux sociaux, etc.).

Un forward proxy exige en général que le client configure son application pour le pointer. Les grandes institutions utilisent souvent un **transparent proxy** pour simplifier cela.

## Transparent proxy

Un transparent proxy s’appuie sur des commutateurs layer 4 pour rediriger automatiquement certains types de trafic vers le proxy. Il n’est pas nécessaire de configurer les machines clientes.

Lorsqu’on est sur le réseau de l’institution, il est difficile de le contourner.

**En résumé :** un forward proxy se place entre le client et Internet, et agit **au nom du client**.

---

## Reverse proxy

Un reverse proxy se place entre Internet et les serveurs web. Il intercepte les requêtes des clients et parle aux serveurs web **à leur place**.

### Pourquoi un site utiliserait-il un reverse proxy ?

#### 1. Protéger le site

Les adresses IP du site sont cachées derrière le reverse proxy et ne sont pas révélées aux clients. Il devient plus difficile de cibler le site avec une attaque DDoS.

#### 2. Load balancing

Un site très fréquenté ne peut généralement pas tout gérer avec un seul serveur. Le reverse proxy répartit les requêtes entrantes sur un parc de serveurs web, pour éviter qu’un seul d’entre eux ne sature.

Cela suppose que le reverse proxy lui-même tienne la charge. Des services comme Cloudflare déploient des reverse proxies dans des centaines de lieux dans le monde : plus proches des utilisateurs, et avec une grande capacité de traitement.

#### 3. Mettre en cache le contenu statique

Un contenu peut rester en cache sur le reverse proxy pendant un certain temps. Si la même ressource est redemandée, la copie locale peut être renvoyée rapidement.

#### 4. Gérer le chiffrement SSL

Le SSL handshake est coûteux en calcul. Le reverse proxy décharge les origin servers de ces opérations. Au lieu de gérer le SSL pour tous les clients, le site n’a plus qu’à gérer les SSL handshakes avec un petit nombre de reverse proxies.

---

## Layers of reverse proxy

Les reverse proxies sont partout. Pour un site moderne, il n’est pas rare d’en avoir plusieurs layers.

1. **Premier layer :** un service edge, type Cloudflare. Les reverse proxies sont déployés près des utilisateurs, dans des centaines de lieux.
2. **Deuxième layer :** une API gateway ou un load balancer chez l’hébergeur.

Beaucoup de fournisseurs cloud fusionnent ces deux layers en un seul service d’ingress. L’utilisateur entre dans le réseau cloud à l’edge, près de chez lui ; de là, le reverse proxy relie, via un réseau fibre rapide, le load balancer, qui répartit la requête sur un cluster de serveurs web.
