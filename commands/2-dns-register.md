# 2. Enregistrement DNS — commandes

Objectif : **lire** ce qu’on a **écrit** dans la zone (types + TTL). La chaîne racine → TLD est l’étape 1.

Domaine : `readresolve.tech`

Réponses et analyses : [`response-analysis/2-dns-register.md`](response-analysis/2-dns-register.md)

## Lire les records de la zone (par type)

### Linux

```bash
dig A readresolve.tech
dig AAAA readresolve.tech
dig NS readresolve.tech
dig MX readresolve.tech
dig TXT readresolve.tech
dig A www.readresolve.tech
dig CNAME www.readresolve.tech
```

### Windows

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type A
Resolve-DnsName -Name "readresolve.tech" -Type AAAA
Resolve-DnsName -Name "readresolve.tech" -Type NS
Resolve-DnsName -Name "readresolve.tech" -Type MX
Resolve-DnsName -Name "readresolve.tech" -Type TXT
Resolve-DnsName -Name "www.readresolve.tech" -Type A
Resolve-DnsName -Name "www.readresolve.tech" -Type CNAME
```

## Infos registrar / dates (`whois`)

Pas le contenu de la zone — registrar, dates, contacts.

### Linux

```bash
whois readresolve.tech
```

### Windows

```powershell
# Si whois n’est pas installé : fiche registrar, ou WSL.
whois readresolve.tech
```

## Ce qu’on doit reconnaître

| Type  | Attendu ici                                               |
| ----- | --------------------------------------------------------- |
| A     | `54.36.100.9`, TTL 3600                                   |
| AAAA  | pas d’enregistrement                                      |
| NS    | `dns13.ovh.net`, `ns13.ovh.net`                           |
| MX    | `mx4.mail.ovh.net` (prio 1), `mx3.mail.ovh.net` (prio 10) |
| TXT   | SPF `v=spf1 include:mx.ovh.com ~all`                      |
| `www` | **A** vers la même IP — pas un CNAME                      |

Vue d’ensemble externe : [dnschecker — tous les records](https://dnschecker.org/all-dns-records-of-domain.php?query=readresolve.tech&rtype=ALL&dns=dnsauth).

La zone dans la **console OVH** : capture fournie par le formateur.
