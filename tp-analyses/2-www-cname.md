← [Retour à la feuille TP](../tp-sheet.md#cmd-2-www-cname)

# Record CNAME (`www.readresolve.tech`)

**But :** vérifier que `www` n’est **pas** un alias. Ici il n’y a pas de CNAME.

## Commandes

**Linux**

```bash
dig CNAME www.readresolve.tech
```

**Windows**

```powershell
Resolve-DnsName -Name "www.readresolve.tech" -Type CNAME
```

## Sortie attendue (Linux)

```
;; flags: … ANSWER: 0, AUTHORITY: 1, …

;; QUESTION SECTION:
;www.readresolve.tech.          IN      CNAME

;; AUTHORITY SECTION:
readresolve.tech.       300     IN      SOA     dns13.ovh.net. …
```

## Sortie attendue (Windows)

```
Name              Type TTL Section   PrimaryServer
----              ---- --- -------   -------------
readresolve.tech  SOA  300 Authority dns13.ovh.net
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `ANSWER: 0` + `SOA` | Pas de CNAME pour `www` (même schéma que l’AAAA absent). |
| Conclusion | `www` est un **A** vers `54.36.100.9`, pas un alias vers l’apex. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-2-www-cname)
