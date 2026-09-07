← [Retour à la feuille TP](../tp-sheet.md#cmd-2-aaaa)

# Record AAAA (`readresolve.tech`)

**But :** voir s’il existe une IPv6 publiée. Ici : **non**.

## Commandes

**Linux**

```bash
dig AAAA readresolve.tech
```

**Windows**

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type AAAA
```

## Sortie attendue (Linux)

```
;; flags: qr rd ra …; QUERY: 1, ANSWER: 0, AUTHORITY: 1, …

;; QUESTION SECTION:
;readresolve.tech.              IN      AAAA

;; AUTHORITY SECTION:
readresolve.tech.       300     IN      SOA     dns13.ovh.net. tech.ovh.net. …
```

## Sortie attendue (Windows)

```
Name              Type TTL Section   PrimaryServer    NameAdministrator
----              ---- --- -------   -------------    -----------------
readresolve.tech  SOA  300 Authority dns13.ovh.net    tech.ovh.net
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `ANSWER: 0` / pas de ligne AAAA | **Pas d’IPv6** dans la zone. |
| `SOA` en Authority | Réponse négative : le domaine existe, ce **type** n’a pas d’enregistrement. |
| ≠ `NXDOMAIN` | `NXDOMAIN` = le **nom** n’existe pas. Ici le nom existe, sans AAAA. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-2-aaaa)
