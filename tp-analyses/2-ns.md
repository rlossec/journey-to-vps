← [Retour à la feuille TP](../tp-sheet.md#cmd-2-ns)

# Records NS (`readresolve.tech`)

**But :** qui est autoritaire pour la zone (vue « zone », pas délégation TLD).

## Commandes

**Linux**

```bash
dig NS readresolve.tech
```

**Windows**

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type NS
```

## Sortie attendue (Linux)

```
;; ANSWER SECTION:
readresolve.tech.       3600    IN      NS      ns13.ovh.net.
readresolve.tech.       3600    IN      NS      dns13.ovh.net.
```

## Sortie attendue (Windows)

```
Name               Type TTL  Section NameHost
----               ---- ---  ------- --------
readresolve.tech   NS   3600 Answer  ns13.ovh.net
readresolve.tech   NS   3600 Answer  dns13.ovh.net
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `dns13.ovh.net` / `ns13.ovh.net` | Paire NS **OVH** — cohérent avec l’étape 1. |
| `ANSWER: 2` | Cette fois les NS sont **dans Answer** (résolveur récursif a fini le travail). |
| TTL &lt; 3600 | Reste de cache, pas un TTL de zone à 145 s. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-2-ns)
