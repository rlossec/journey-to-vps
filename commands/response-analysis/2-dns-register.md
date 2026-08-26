# Commandes, réponses et analyse — 2. Enregistrement DNS

Fiche atelier (commandes seules) : [`../2-dns-register.md`](../2-dns-register.md)

## Lire les records de la zone (par type)

### Linux

#### Commande

```bash
dig A readresolve.tech
```

#### Réponse (WSL)

```
; <<>> DiG 9.18.39-0ubuntu0.24.04.5-Ubuntu <<>> A readresolve.tech
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 56810
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;readresolve.tech.              IN      A

;; ANSWER SECTION:
readresolve.tech.       3600    IN      A       54.36.100.9

;; Query time: 19 msec
;; SERVER: 10.255.255.254#53(10.255.255.254) (UDP)
;; WHEN: Wed Aug 26 10:59:06 CEST 2026
;; MSG SIZE  rcvd: 61
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `A 54.36.100.9` | Record A de l’apex : IP du VPS. |
| `TTL 3600` | Durée de vie configurée dans la zone (1 h). |
| flag `ad` | Validation DNSSEC signalée par le résolveur. |

#### Réponse (VPS)

Pas encore de capture.

---

#### Commande

```bash
dig AAAA readresolve.tech
```

#### Réponse (WSL)

```
; <<>> DiG 9.18.39-0ubuntu0.24.04.5-Ubuntu <<>> AAAA readresolve.tech
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30257
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;readresolve.tech.              IN      AAAA

;; AUTHORITY SECTION:
readresolve.tech.       300     IN      SOA     dns13.ovh.net. tech.ovh.net. 2086516984 86400 3600 3600000 300

;; Query time: 9 msec
;; SERVER: 10.255.255.254#53(10.255.255.254) (UDP)
;; WHEN: Wed Aug 26 10:59:24 CEST 2026
;; MSG SIZE  rcvd: 99
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `ANSWER: 0` | **Pas** de record AAAA (pas d’IPv6 publié). |
| `AUTHORITY: 1` + `SOA` | Réponse négative autoritaire : la zone existe, mais ce type est absent. |
| `SOA … 300` | SOA (et TTL négatif / minimum) côté OVH — normal pour « NXRRSET » / no data. |

#### Réponse (VPS)

Pas encore de capture.

---

#### Commande

```bash
dig NS readresolve.tech
```

#### Réponse (WSL)

```
; <<>> DiG 9.18.39-0ubuntu0.24.04.5-Ubuntu <<>> NS readresolve.tech
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20653
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;readresolve.tech.              IN      NS

;; ANSWER SECTION:
readresolve.tech.       145     IN      NS      ns13.ovh.net.
readresolve.tech.       145     IN      NS      dns13.ovh.net.

;; Query time: 9 msec
;; SERVER: 10.255.255.254#53(10.255.255.254) (UDP)
;; WHEN: Wed Aug 26 10:59:44 CEST 2026
;; MSG SIZE  rcvd: 91
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `ns13` / `dns13.ovh.net` | NS de la zone chez OVH (cohérent étape 1). |
| `TTL 145` | TTL **restant** en cache (zone à 3600) — pas un TTL configuré à 145 s. |

#### Réponse (VPS)

Pas encore de capture.

---

### Windows

#### A

#### Commande

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type A
```

#### Réponse

```
Name                                           Type   TTL   Section    IPAddress
----                                           ----   ---   -------    ---------
readresolve.tech                               A      2863  Answer     54.36.100.9
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `A` / `54.36.100.9` | Même apex A que `dig`. |
| `TTL 2863` | Cache Windows (déjà consommé depuis 3600). |

#### AAAA

#### Commande

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type AAAA
```

#### Réponse

```
Name                        Type TTL   Section    PrimaryServer               NameAdministrator           SerialNumber
----                        ---- ---   -------    -------------               -----------------           ------------
readresolve.tech            SOA  300   Authority  dns13.ovh.net               tech.ovh.net                2086516984
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `Type SOA` / `Section Authority` | Équivalent PowerShell de `ANSWER: 0` + SOA : **pas d’AAAA**. |
| `dns13.ovh.net` / `SerialNumber 2086516984` | SOA de la zone OVH. |

#### NS

#### Commande

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type NS
```

#### Réponse

```
Name                           Type   TTL   Section    NameHost
----                           ----   ---   -------    --------
readresolve.tech               NS     2932  Answer     ns13.ovh.net
readresolve.tech               NS     2932  Answer     dns13.ovh.net
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| Deux `NS` OVH | Zone déléguée à `ns13` / `dns13`. |
| `TTL 2932` | Reste de cache (pas la valeur zone fixe). |

#### MX

#### Commande

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type MX
```

#### Réponse

```
Name                                     Type   TTL   Section    NameExchange                              Preference
----                                     ----   ---   -------    ------------                              ----------
readresolve.tech                         MX     3600  Answer     mx4.mail.ovh.net                          1
readresolve.tech                         MX     3600  Answer     mx3.mail.ovh.net                          10
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `mx4` priorité `1` | Serveur mail primaire OVH. |
| `mx3` priorité `10` | Secondaire (utilisé si le 1 échoue). |
| `TTL 3600` | TTL zone pour les MX. |

#### TXT

#### Commande

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type TXT
```

#### Réponse

```
Name                                     Type   TTL   Section    Strings
----                                     ----   ---   -------    -------
readresolve.tech                         TXT    600   Answer     {v=spf1 include:mx.ovh.com ~all}
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `v=spf1 … ~all` | Politique **SPF** : autorise les MX OVH ; `~all` = softfail pour le reste. |
| `TTL 600` | TTL plus court que A/MX (10 min). |

#### `www` (A)

#### Commande

```powershell
Resolve-DnsName -Name "www.readresolve.tech" -Type A
```

#### Réponse

```
Name                                           Type   TTL   Section    IPAddress
----                                           ----   ---   -------    ---------
www.readresolve.tech                           A      3333  Answer     54.36.100.9
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `www` → `A 54.36.100.9` | Sous-domaine `www` pointe **directement** vers la même IP que l’apex. |
| `TTL 3333` | Cache restant. |

#### `www` (CNAME)

#### Commande

```powershell
Resolve-DnsName -Name "www.readresolve.tech" -Type CNAME
```

#### Réponse

```
Name                        Type TTL   Section    PrimaryServer               NameAdministrator           SerialNumber
----                        ---- ---   -------    -------------               -----------------           ------------
readresolve.tech            SOA  300   Authority  dns13.ovh.net               tech.ovh.net                2086516984
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| SOA en Authority | Pas de CNAME pour `www` — même pattern que AAAA. |
| Conclusion | `www` est un **A**, pas un alias vers l’apex. |

---

## Infos registrar / dates (`whois`)

### Linux

#### Commande

```bash
whois readresolve.tech
```

#### Réponse (WSL)

```
Domain Name: readresolve.tech
Registry Domain ID: D58141208-CNIC
Registrar WHOIS Server: whois.ovh.com
Registrar URL: https://www.ovhcloud.com/fr/
Updated Date: 2026-02-01T10:40:15.000Z
Creation Date: 2017-12-17T15:57:18.000Z
Registry Expiry Date: 2026-12-17T23:59:59.000Z
Registrar: OVH sas
Registrar IANA ID: 433
Registrar Abuse Contact Email: abuse@ovh.net
Registrar Abuse Contact Phone:
Domain Status: clientDeleteProhibited https://icann.org/epp#clientDeleteProhibited
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
Name Server: dns13.ovh.net
Name Server: ns13.ovh.net
DNSSEC: signedDelegation
URL of the ICANN RDDS Inaccuracy Complaint Form: https://icann.org/wicf

>>> Last update of WHOIS database: 2026-08-26T09:19:29.751Z <<<

For more information on domain status codes, please visit https://icann.org/epp

The WHOIS information provided in this page has been redacted
in compliance with ICANN's Temporary Specification for gTLD
Registration Data.

The data in this record is provided by Tucows Registry for informational
purposes only, and it does not guarantee its accuracy. Tucows Registry is
authoritative for whois information in top-level domains it operates
under contract with the Internet Corporation for Assigned Names and
Numbers. Whois information from other top-level domains is provided by
a third-party under license to Tucows Registry.

This service is intended only for query-based access. By using this
service, you agree that you will use any data presented only for lawful
purposes and that, under no circumstances will you use (a) data
acquired for the purpose of allowing, enabling, or otherwise supporting
the transmission by e-mail, telephone, facsimile or other
communications mechanism of mass  unsolicited, commercial advertising
or solicitations to entities other than your existing  customers; or
(b) this service to enable high volume, automated, electronic processes
that send queries or data to the systems of any Registrar or any
Registry except as reasonably necessary to register domain names or
modify existing domain name registrations.

Tucows Registry reserves the right to modify these terms at any time. By
submitting this query, you agree to abide by this policy. All rights
reserved.
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `Domain Name: readresolve.tech` | Domaine du cas d’étude. |
| `Registrar: OVH sas` | Registrar = **OVH** (cohérent VPS + zone DNS). |
| `Creation Date: 2017-12-17` | Date d’enregistrement. |
| `Updated Date: 2026-02-01` | Dernière mise à jour registry. |
| `Registry Expiry Date: 2026-12-17` | Expiration (à renouveler). |
| `Name Server: dns13` / `ns13.ovh.net` | NS publiés au registre = ceux vus par `dig NS`. |
| `DNSSEC: signedDelegation` | Délégation signée — aligné avec les flags `ad` / RRSIG de l’étape 1. |
| `clientDeleteProhibited` / `clientTransferProhibited` | Verrous EPP : freinent suppression / transfert non voulu. |

#### Réponse (VPS)

```
Domain Name: readresolve.tech
Registry Domain ID: D58141208-CNIC
Registrar WHOIS Server: whois.ovh.com
Registrar URL: https://www.ovhcloud.com/fr/
Updated Date: 2026-02-01T10:40:15.000Z
Creation Date: 2017-12-17T15:57:18.000Z
Registry Expiry Date: 2026-12-17T23:59:59.000Z
Registrar: OVH sas
Registrar IANA ID: 433
Registrar Abuse Contact Email: abuse@ovh.net
Registrar Abuse Contact Phone:
Domain Status: clientDeleteProhibited https://icann.org/epp#clientDeleteProhibited
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
Name Server: dns13.ovh.net
Name Server: ns13.ovh.net
DNSSEC: signedDelegation
URL of the ICANN RDDS Inaccuracy Complaint Form: https://icann.org/wicf

>>> Last update of WHOIS database: 2026-08-26T09:15:54.659Z <<<

For more information on domain status codes, please visit https://icann.org/epp

The WHOIS information provided in this page has been redacted
in compliance with ICANN's Temporary Specification for gTLD
Registration Data.

The data in this record is provided by Tucows Registry for informational
purposes only, and it does not guarantee its accuracy. Tucows Registry is
authoritative for whois information in top-level domains it operates
under contract with the Internet Corporation for Assigned Names and
Numbers. Whois information from other top-level domains is provided by
a third-party under license to Tucows Registry.

This service is intended only for query-based access. By using this
service, you agree that you will use any data presented only for lawful
purposes and that, under no circumstances will you use (a) data
acquired for the purpose of allowing, enabling, or otherwise supporting
the transmission by e-mail, telephone, facsimile or other
communications mechanism of mass  unsolicited, commercial advertising
or solicitations to entities other than your existing  customers; or
(b) this service to enable high volume, automated, electronic processes
that send queries or data to the systems of any Registrar or any
Registry except as reasonably necessary to register domain names or
modify existing domain name registrations.

Tucows Registry reserves the right to modify these terms at any time. By
submitting this query, you agree to abide by this policy. All rights
reserved.
```

#### Analyse

Mêmes champs clés que WSL (whois registry, pas la zone DNS). Seul l’horodatage « Last update of WHOIS database » change entre les deux captures.
