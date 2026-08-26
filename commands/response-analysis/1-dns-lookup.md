# Commandes, réponses et analyse — 1. Résolution DNS

Fiche atelier (commandes seules) : [`../1-dns-lookup.md`](../1-dns-lookup.md)

## Résoudre le record A

### Linux

#### Commande

```bash
dig readresolve.tech A
```

#### Réponse (WSL)

```
; <<>> DiG 9.18.39-0ubuntu0.24.04.5-Ubuntu <<>> readresolve.tech A
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63739
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;readresolve.tech.              IN      A

;; ANSWER SECTION:
readresolve.tech.       2866    IN      A       54.36.100.9

;; Query time: 0 msec
;; SERVER: 10.255.255.254#53(10.255.255.254) (UDP)
;; WHEN: Wed Aug 26 10:01:17 CEST 2026
;; MSG SIZE  rcvd: 61
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `status: NOERROR` | Requête réussie. |
| `flags: qr rd ra` | Réponse (`qr`) ; récursion demandée (`rd`) et acceptée (`ra`). |
| `ANSWER: 1` | Un record A. |
| `TTL 2866` | Secondes restantes en cache (origine 3600). |
| `A 54.36.100.9` | IP publique du VPS. |
| `Query time: 0 msec` | Réponse cache / locale. |
| `SERVER: 10.255.255.254` | Résolveur WSL (passerelle Hyper-V). |

#### Réponse (VPS)

```
; <<>> DiG 9.20.24-1ubuntu0.2-Ubuntu <<>> readresolve.tech A
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39978
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;readresolve.tech. IN A

;; ANSWER SECTION:
readresolve.tech. 3451 IN A 54.36.100.9

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Wed Aug 26 07:50:20 UTC 2026
;; MSG SIZE rcvd: 61
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| Même `A 54.36.100.9` | Même zone. |
| `TTL 3451` | Autre cache / autre instant. |
| `SERVER: 127.0.0.53` | Stub `systemd-resolved` sur le VPS. |

### Windows

#### Commande

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type A
```

#### Réponse

```
Name                                           Type   TTL   Section    IPAddress
----                                           ----   ---   -------    ---------
readresolve.tech                               A      2832  Answer     54.36.100.9
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `Type A` / `54.36.100.9` | Même record. |
| `TTL 2832` | Cache Windows déjà partiellement consommé. |

---

## Lister les serveurs de noms (NS)

### Linux

#### Commande

```bash
dig NS readresolve.tech
```

#### Réponse (WSL)

```
; <<>> DiG 9.18.39-0ubuntu0.24.04.5-Ubuntu <<>> NS readresolve.tech
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49371
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;readresolve.tech.              IN      NS

;; ANSWER SECTION:
readresolve.tech.       3600    IN      NS      ns13.ovh.net.
readresolve.tech.       3600    IN      NS      dns13.ovh.net.

;; Query time: 20 msec
;; SERVER: 10.255.255.254#53(10.255.255.254) (UDP)
;; WHEN: Wed Aug 26 10:02:09 CEST 2026
;; MSG SIZE  rcvd: 91
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `flags: … ad` | DNSSEC validé côté résolveur. |
| `NS ns13` / `dns13.ovh.net` | Serveurs autoritaires OVH. |
| `TTL 3600` | TTL « plein » pour ce type ici. |

#### Réponse (VPS)

```
; <<>> DiG 9.20.24-1ubuntu0.2-Ubuntu <<>> NS readresolve.tech
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16086
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;readresolve.tech.              IN      NS

;; ANSWER SECTION:
readresolve.tech.       3600    IN      NS      ns13.ovh.net.
readresolve.tech.       3600    IN      NS      dns13.ovh.net.

;; Query time: 113 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Wed Aug 26 07:50:35 UTC 2026
;; MSG SIZE  rcvd: 91
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| Mêmes NS | Cohérent avec WSL. |
| Pas de flag `ad` | Ce chemin ne signale pas la validation DNSSEC (≠ « pas de DNSSEC »). |
| `113 msec` | Aller-retour plus long qu’un hit cache immédiat. |

### Windows

#### Commande

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type NS
```

#### Réponse

```
Name                           Type   TTL   Section    NameHost
----                           ----   ---   -------    --------
readresolve.tech               NS     3600  Answer     ns13.ovh.net
readresolve.tech               NS     3600  Answer     dns13.ovh.net
```

#### Analyse

Même paire NS, TTL 3600.

---

## Rejouer la chaîne sans s’appuyer sur le cache (`+trace`)

### Linux

#### Commande

```bash
dig +trace readresolve.tech
```

#### Réponse (WSL)

```
;; communications error to 10.255.255.254#53: timed out
;; communications error to 10.255.255.254#53: timed out
;; communications error to 10.255.255.254#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.5-Ubuntu <<>> +trace readresolve.tech
;; global options: +cmd
;; no servers could be reached
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `timed out` vers `10.255.255.254#53` | Le stub DNS WSL ne permet pas (ici) la chaîne itérative `+trace`. |
| `no servers could be reached` | Échec total de la démo depuis WSL — pas un problème de zone. |
| Contournement | Utiliser la capture **VPS** ou les `nslookup` Windows ci-dessous. |

#### Réponse (VPS)

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

#### Analyse (par blocs)

| Bloc | Signification |
| --- | --- |
| **Racine** (`. IN NS` + RRSIG) | Liste des root servers ; départ de la délégation (via `127.0.0.53`). |
| **Timeouts** | Certains root / NS ne répondent pas à temps — dig réessaie ; ≠ panne du domaine. |
| **TLD `.tech`** | Délégation vers `ns*.trs-dns.*`. |
| **Délégation domaine** | NS → `ns13` / `dns13.ovh.net` (TTL délégation 900). |
| **Autoritaire A** | `A 54.36.100.9` depuis `ns13.ovh.net` — fin de chaîne, TTL 3600. |

### Windows

#### Commande

```powershell
nslookup -type=NS tech. a.root-servers.net
nslookup -type=NS readresolve.tech ns01.trs-dns.com
nslookup -type=A readresolve.tech dns13.ovh.net
```

#### Réponse

```
nslookup -type=NS tech. a.root-servers.net
```

```
Serveur :   UnKnown
Address:  2001:503:ba3e::2:30

tech    nameserver = ns01.trs-dns.com
tech    nameserver = ns10.trs-dns.org
tech    nameserver = ns10.trs-dns.info
tech    nameserver = ns01.trs-dns.net
ns01.trs-dns.com        internet address = 64.96.1.1
…
```

```
nslookup -type=NS readresolve.tech ns01.trs-dns.com
```

```
Serveur :   UnKnown
Address:  2620:57:4001::1

readresolve.tech        nameserver = dns13.ovh.net
readresolve.tech        nameserver = ns13.ovh.net
```

```
nslookup -type=A readresolve.tech dns13.ovh.net
```

```
Serveur :   UnKnown
Address:  2001:41d0:d00:f200::2

Nom :    readresolve.tech
Address:  54.36.100.9
```

#### Analyse

| Étape | Signification |
| --- | --- |
| 1. Racine → `tech` NS | Équivalent pédagogique du début de `+trace`. |
| 2. TLD → NS OVH | Délégation du domaine. |
| 3. Autoritaire → A | IP `54.36.100.9`. |

---

## Forcer un résolveur public

### Windows

#### Commande

```powershell
Resolve-DnsName -Name "readresolve.tech" -Server "8.8.8.8"
```

#### Réponse

```
Name                                           Type   TTL   Section    IPAddress
----                                           ----   ---   -------    ---------
readresolve.tech                               A      3600  Answer     54.36.100.9
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `TTL 3600` | Souvent « frais » vs TTL partiels du cache local. |
| `-Server 8.8.8.8` | Contourne le résolveur Windows par défaut (quand UDP/53 n’est pas filtré). |

---

## Interroger sans récursion

### Linux

#### Commande

```bash
dig +norecurse readresolve.tech
```

#### Réponse (WSL) — label fichier ; serveur `127.0.0.53` (typique VPS)

```
; <<>> DiG 9.20.24-1ubuntu0.2-Ubuntu <<>> +norecurse readresolve.tech
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 52132
;; flags: qr ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;readresolve.tech.              IN      A

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Wed Aug 26 08:26:45 UTC 2026
;; MSG SIZE  rcvd: 45
```

#### Réponse (VPS) — label fichier ; serveur `10.255.255.254` (typique WSL)

```
; <<>> DiG 9.18.39-0ubuntu0.24.04.5-Ubuntu <<>> +norecurse readresolve.tech
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 63651
;; flags: qr ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;readresolve.tech.              IN      A

;; Query time: 9 msec
;; SERVER: 10.255.255.254#53(10.255.255.254) (UDP)
;; WHEN: Wed Aug 26 10:
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `status: REFUSED` | Le résolveur refuse une requête non récursive (il n’est pas autoritaire). |
| `ANSWER: 0` | Pas de record renvoyé. |
| Labels WSL/VPS | Probablement **inversés** (serveur `127.0.0.53` vs `10.255.255.254`) — à recoller proprement. |
| Variante utile | `dig +norecurse @dns13.ovh.net readresolve.tech` → attendre flag `aa`. |

---

## Variante simple (`host`)

### Linux

#### Commande

```bash
host readresolve.tech
```

#### Réponse (WSL)

```
readresolve.tech has address 54.36.100.9
readresolve.tech mail is handled by 1 mx4.mail.ovh.net.
readresolve.tech mail is handled by 10 mx3.mail.ovh.net.
```

#### Réponse (VPS)

```
readresolve.tech has address 54.36.100.9
readresolve.tech mail is handled by 1 mx4.mail.ovh.net.
readresolve.tech mail is handled by 10 mx3.mail.ovh.net.
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `has address` | Record A. |
| `mail is handled by 1` / `10` | MX avec priorités (aperçu ; détail étape 2). |

---

## Voir / vider le cache DNS local

Pas encore de capture de sortie. Commandes : `resolvectl flush-caches` · `ipconfig /displaydns` · `ipconfig /flushdns`.
