← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-ns)

# Records NS (`readresolve.tech`)

**But :** qui est autoritaire pour la zone (vue « zone », pas délégation TLD).

```bash
dig NS readresolve.tech
```

## Sortie attendue

```
;; ANSWER SECTION:
readresolve.tech.       3600    IN      NS      ns13.ovh.net.
readresolve.tech.       3600    IN      NS      dns13.ovh.net.
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `dns13.ovh.net` / `ns13.ovh.net` | Paire NS **OVH** — cohérent avec l’étape 1. |
| TTL inférieur à 3600 | Reste de cache, pas un TTL de zone à 145 s. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-ns)
