← [Retour à la feuille TP](../../tp-sheet.md#cmd-3-traceroute-icmp)

# `traceroute -I`

**But :** même lecture que `traceroute`, avec sondes **ICMP** (comme `ping`) au lieu d’UDP.

```bash
traceroute -I 54.36.100.9
```

Parfois plus de sauts visibles si l’UDP est filtré.

## Sortie attendue

Même **squelette** que [traceroute](3-traceroute.md) : LAN → FAI → OVH → `54.36.100.9`, avec des `*`.

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `-I` | Sondes ICMP ; utile si l’UDP est filtré. |
| `*` | Routeur muet, pas une panne. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-3-traceroute-icmp)
