← [Retour à la feuille TP](../tp-sheet.md#cmd-2-whois)

# `whois readresolve.tech`

**But :** infos **registre / registrar**, pas le contenu de la zone DNS (A, MX, …).

**Windows :** pas d’équivalent dans la feuille TP (`whois` n’est en général pas installé).

## Commande

```bash
whois readresolve.tech
```

## Sortie attendue (extrait utile)

La suite du texte est une licence registry ; on peut l’ignorer.

```
Domain Name: readresolve.tech
Registrar WHOIS Server: whois.ovh.com
Registrar URL: https://www.ovhcloud.com/fr/
Updated Date: 2026-02-01T10:40:15.000Z
Creation Date: 2017-12-17T15:57:18.000Z
Registry Expiry Date: 2026-12-17T23:59:59.000Z
Registrar: OVH sas
Name Server: dns13.ovh.net
Name Server: ns13.ovh.net
DNSSEC: signedDelegation
Domain Status: clientDeleteProhibited …
Domain Status: clientTransferProhibited …
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `Registrar: OVH sas` | Le nom de domaine est chez **OVH** (comme le VPS). |
| `Name Server: dns13` / `ns13` | NS publiés au **registre** = ceux de `dig NS`. |
| `DNSSEC: signedDelegation` | Délégation signée. |
| Dates création / expiry | Cycle de vie du domaine, pas le TTL d’un record. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-2-whois)
