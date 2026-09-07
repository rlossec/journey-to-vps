← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-www-a)

# Record A (`www.readresolve.tech`)

**But :** le sous-domaine `www` pointe-t-il vers la même IP que l’apex ?

```powershell
Resolve-DnsName -Name "www.readresolve.tech" -Type A
```

## Sortie attendue

```
Name                    Type TTL  Section IPAddress
----                    ---- ---  ------- ---------
www.readresolve.tech    A    3333 Answer  54.36.100.9
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `A 54.36.100.9` | **Même IP** que `readresolve.tech`. |
| C’est un **A**, pas un CNAME | Confirmé par la commande CNAME suivante. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-www-a)
