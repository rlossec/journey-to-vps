← [Retour à la feuille TP](../../tp-sheet.md#cmd-3-traceroute-icmp)

# `tracert -d`

**But :** même lecture que `tracert`, **sans** résolution inverse des IP (plus rapide).

```powershell
tracert -d 54.36.100.9
```

## Sortie attendue

Même **squelette** que [tracert](3-traceroute.md) : LAN → FAI → OVH → `54.36.100.9`, avec des `*`.
Des **IP brutes** à la place des noms (`be100.par-…`, `vps-3229ca35…`).

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `-d` | Pas de DNS inverse : on lit les **IP**, pas les FQDN. |
| `*` | Routeur muet, pas une panne. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-3-traceroute-icmp)
