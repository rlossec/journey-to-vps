← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-cname)

# Record CNAME (`www.readresolve.tech`)

**But :** vérifier que `www` n’est **pas** un alias. Ici il n’y a pas de CNAME.

```powershell
Resolve-DnsName -Name "www.readresolve.tech" -Type CNAME
```

## Sortie attendue

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

← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-cname)
