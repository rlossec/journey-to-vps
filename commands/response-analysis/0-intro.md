# Commandes, réponses et analyse — Intro

Fiche atelier (commandes seules) : [`../0-intro.md`](../0-intro.md)

## Voir les en-têtes HTTP

### Linux

#### Commande

```bash
curl -I https://readresolve.tech
```

#### Réponse (WSL)

```
HTTP/2 200
last-modified: Thu, 16 Jul 2026 13:25:45 GMT
etag: "1c6-656ba609f8840"
accept-ranges: bytes
content-length: 454
vary: Accept-Encoding
content-type: text/html
date: Wed, 26 Aug 2026 08:21:28 GMT
server: Apache
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `HTTP/2 200` | Protocole HTTP/2 ; code **200** = succès. Le chemin jusqu’au serveur fonctionne. |
| `last-modified: … 16 Jul 2026` | Date de dernière modification de la ressource côté serveur. |
| `etag: "1c6-…"` | Identifiant de version du contenu (cache HTTP client). |
| `accept-ranges: bytes` | Le serveur accepte les téléchargements partiels. |
| `content-length: 454` | Taille du corps (~454 octets) — page légère. |
| `vary: Accept-Encoding` | La réponse peut changer selon compression (`gzip`, etc.). |
| `content-type: text/html` | On sert du HTML. |
| `date: …` | Horodatage de la réponse. |
| `server: Apache` | Logiciel qui répond : **Apache** (pas encore le détail frontend/backend). |

#### Réponse (VPS)

```
HTTP/2 200
last-modified: Thu, 16 Jul 2026 13:25:45 GMT
etag: "1c6-656ba609f8840"
accept-ranges: bytes
content-length: 454
vary: Accept-Encoding
content-type: text/html
date: Wed, 26 Aug 2026 08:21:00 GMT
server: Apache
```

#### Analyse

Même lecture que WSL. Seule la `date` change (horloge / instant de la requête). Depuis le VPS ou depuis WSL, on obtient le **même** site Apache — les étapes suivantes expliquent le chemin invisible ici.

### Windows

#### Commande

```powershell
curl.exe -I https://readresolve.tech
```

#### Réponse

```
HTTP/1.1 200 OK
Date: Wed, 26 Aug 2026 08:20:32 GMT
Server: Apache
Upgrade: h2,h2c
Connection: Upgrade
Last-Modified: Thu, 16 Jul 2026 13:25:45 GMT
ETag: "1c6-656ba609f8840"
Accept-Ranges: bytes
Content-Length: 454
Vary: Accept-Encoding
Content-Type: text/html
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `HTTP/1.1 200 OK` | Succès en HTTP/1.1 (ce `curl.exe` n’a pas négocié HTTP/2 comme WSL). |
| `Server: Apache` | Même constat : Apache répond. |
| `Upgrade: h2,h2c` | Le serveur propose de passer en HTTP/2. |
| `Connection: Upgrade` | Lié à la négociation d’upgrade. |
| Autres en-têtes | Même ressource (`ETag`, taille 454, `text/html`) que sous Linux. |

**Point pédagogique** : ces en-têtes ne montrent ni DNS, ni traceroute, ni firewall — seulement « ça marche, c’est Apache ».
