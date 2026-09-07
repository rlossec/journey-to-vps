← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-ss-grep)

# `ss` filtré (80, 443, 22, 64483, 90…)

**But :** isoler les sockets du cas d’étude dans une liste trop longue.

```bash
sudo ss -tlnp | grep -E ':80|:443|:22|:64483|:90'
```

## Sortie attendue (VPS)

```
LISTEN  …   0.0.0.0:64483
LISTEN  …  127.0.0.1:9030
LISTEN  …  127.0.0.1:9050
…
LISTEN  …          *:80
LISTEN  …          *:443
```

Pas de ligne `:22` (ou alors un faux positif du type `:2200` — regarder le numéro **exact**).

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `grep -E '…'` | Filtre texte : on ne change pas `ss`, on **réduit** l’affichage. |
| Toujours `*:80/:443` vs `127.0.0.1:90` | Public vs backend. |
| `:90` | Motif large (9030, 9050…) — vérifier que ce n’est pas un autre port. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-ss-grep)
