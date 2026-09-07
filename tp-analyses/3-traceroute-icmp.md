← [Retour à la feuille TP](../tp-sheet.md#cmd-3-traceroute-icmp)

# `traceroute -I` / `tracert -d`

**But :** variante de sondes / d’affichage. Même lecture pédagogique que `traceroute` / `tracert`.

## Commandes

**Linux**

```bash
traceroute -I 54.36.100.9
```

`-I` : sondes **ICMP** (comme `ping`) au lieu d’UDP. Parfois plus de sauts visibles selon les filtres du chemin.

**Windows**

```powershell
tracert -d 54.36.100.9
```

`-d` : **pas** de résolution inverse des IP (plus rapide, noms absents).

## Sortie attendue

Même **squelette** que [traceroute / tracert](3-traceroute.md) : LAN → FAI → OVH → `54.36.100.9`, avec des `*`.

Windows `-d` : des IP brutes à la place des noms (`be100.par-…`, `vps-3229ca35…`).

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| Linux `-I` | Autre type de sonde ; utile si l’UDP est filtré. |
| Windows `-d` | Moins de DNS inverse : on lit les **IP**, pas les FQDN. |
| `*` | Même signification : routeur muet, pas une panne. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-3-traceroute-icmp)
