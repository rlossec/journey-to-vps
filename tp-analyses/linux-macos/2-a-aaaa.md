← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-a-aaaa)

# Records A et AAAA (`readresolve.tech`)

```bash
dig A readresolve.tech
```

```bash
dig AAAA readresolve.tech
```

## Sortie attendue — A

```
;; flags: qr rd ra …; QUERY: 1, ANSWER: 1, …
;; QUESTION SECTION:
;readresolve.tech.              IN      A

;; ANSWER SECTION:
readresolve.tech.       3600    IN      A       54.36.100.9
```

## Sortie attendue — AAAA

```
;; flags: qr rd ra …; QUERY: 1, ANSWER: 0, AUTHORITY: 1, …

;; QUESTION SECTION:
;readresolve.tech.              IN      AAAA

;; AUTHORITY SECTION:
readresolve.tech.       300     IN      SOA     dns13.ovh.net. tech.ovh.net. …
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `A 54.36.100.9` | Apex → IP du VPS. |
| `TTL 3600` (ou plus bas) | TTL **configuré** = 3600 ; plus bas = **reste** en cache. |
| `ANSWER: 0` + `SOA` (AAAA) | **Pas d’IPv6** : le nom existe, ce type n’a pas d’enregistrement. |
| ≠ `NXDOMAIN` | `NXDOMAIN` = le **nom** n’existe pas. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-a-aaaa)
