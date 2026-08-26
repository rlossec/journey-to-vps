# Commandes, réponses et analyse — 5. Ports et Apache

Fiche atelier (commandes seules) : [`../5-apache-server.md`](../5-apache-server.md)

## Lister les sockets en écoute

### Linux (VPS)

#### Commande

```bash
sudo ss -tlnp
```

#### Réponse

```
State        Recv-Q        Send-Q               Local Address:Port                Peer Address:Port       Process
LISTEN       0             4096                       0.0.0.0:64483                    0.0.0.0:*
LISTEN       0             4096                 127.0.0.53%lo:53                       0.0.0.0:*
LISTEN       0             511                      127.0.0.1:9030                     0.0.0.0:*
LISTEN       0             511                      127.0.0.1:9050                     0.0.0.0:*
LISTEN       0             511                      127.0.0.1:9060                     0.0.0.0:*
LISTEN       0             511                      127.0.0.1:9130                     0.0.0.0:*
LISTEN       0             511                      127.0.0.1:9140                     0.0.0.0:*
LISTEN       0             511                      127.0.0.1:9150                     0.0.0.0:*
LISTEN       0             511                      127.0.0.1:9170                     0.0.0.0:*
LISTEN       0             4096                    127.0.0.54:53                       0.0.0.0:*
LISTEN       0             4096                          [::]:64483                       [::]:*
LISTEN       0             511                              *:80                             *:*
LISTEN       0             511                              *:443                            *:*
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| En-tête `LISTEN` | Socket en écoute (prêt à accepter des connexions). |
| `0.0.0.0:64483` | Port admin/SSH **sur toutes les IPv4** — aligné avec `iptables` dport 64483. |
| `[::]:64483` | Même service en IPv6. |
| `127.0.0.53%lo:53` / `127.0.0.54:53` | Résolveur DNS local (stub) — pas un service web. |
| `127.0.0.1:9030` … `:9170` | Services **localhost only** → typiques **backends** derrière reverse proxy. Invisible depuis Internet. |
| `*:80` / `*:443` | HTTP/HTTPS sur **toutes** les interfaces = point d’entrée public (Apache frontend attendu). |
| Colonne `Process` vide ici | Sortie sans nom de process visible (droits / truncation) — `ss -tlnp` en root devrait les afficher. |

### Windows

#### Commande

```powershell
Get-NetTCPConnection -State Listen |
  Select-Object LocalAddress, LocalPort, OwningProcess
```

#### Réponse

```
LocalAddress  LocalPort OwningProcess
------------  --------- -------------
::                49679          1788
::1               49671          5356
::                49668          4220
::                49667          3160
::                49666          2452
::                49665          1700
::                49664          1860
::                15150        138932
::                 7680         16260
::                 5432          7452
::                  445             4
::                  135          1944
127.0.0.1         63742         83988
127.0.0.1         63724         83988
0.0.0.0           59318        142372
127.0.0.1         51782          4828
127.0.0.1         51781          4828
127.0.0.1         51780          4828
127.0.0.1         51779          4828
0.0.0.0           49679          1788
0.0.0.0           49668          4220
0.0.0.0           49667          3160
0.0.0.0           49666          2452
0.0.0.0           49665          1700
0.0.0.0           49664          1860
127.0.0.1         46933          5552
127.0.0.1         19010        142372
127.0.0.1          9993        138932
0.0.0.0            6742         59044
127.0.0.1          6463         94880
0.0.0.0            5432          7452
0.0.0.0            5040         12888
0.0.0.0            2968        144916
192.168.1.172       139             4
172.27.0.1          139             4
0.0.0.0             135          1944
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| Liste longue de ports | Écoute **du PC Windows**, pas du VPS — comparaison d’idée seulement. |
| `127.0.0.1` vs `0.0.0.0` / `::` | Même distinction local vs toutes interfaces. |
| `OwningProcess` | PID du process (à croiser avec le Gestionnaire des tâches). |
| Ports 139 / 445 | Services Windows réseaux locaux — hors cas d’étude VPS. |

---

## Scanner les ports publics (`nmap`)

### Linux

#### Commande

```bash
nmap -sV -p 22,80,443 54.36.100.9
```

#### Réponse (WSL)

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-08-26 13:49 CEST
Nmap scan report for vps-3229ca35.vps.ovh.net (54.36.100.9)
Host is up (0.0088s latency).

PORT    STATE    SERVICE  VERSION
22/tcp  filtered ssh
80/tcp  open     http     Apache httpd
443/tcp open     ssl/http Apache httpd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.58 seconds
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `vps-3229ca35.vps.ovh.net` | Nom reverse DNS OVH de l’IP. |
| `Host is up` | La cible répond au scan. |
| `22/tcp filtered` | Pas de réponse claire (pas d’écoute utile sur 22 **ou** filtre) — SSH est sur **64483**, pas 22. |
| `80/tcp open … Apache` | HTTP public = Apache. |
| `443/tcp open ssl/http Apache` | HTTPS public = Apache (+ TLS détecté). |

#### Réponse (VPS)

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-26 11:48 +0000
Nmap scan report for vps-3229ca35.vps.ovh.net (54.36.100.9)
Host is up (0.000077s latency).

PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
443/tcp open   ssl/http Apache httpd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.67 seconds
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| Latence `0.000077s` | Scan **local** (même machine / réseau très proche). |
| `22/tcp closed` | Pas de service sur 22 **sur la machine** (RST) — distinct de `filtered` vu de l’extérieur. |
| 80 / 443 `open` Apache | Confirme `ss` et l’intro `Server: Apache`. |

---

## En-têtes `curl -I`

Pas de capture remplie dans `commands/5-apache-server.md` pour cette section — réutiliser l’analyse de [`0-intro.md`](0-intro.md) (`Server: Apache`, 200).

---

## Fichiers conf Apache

Pas de sorties collées encore. Quand tu auras `apache2ctl -S` / `grep ProxyPass`, on pourra ajouter ici le même format commande → réponse → analyse ligne à ligne.
