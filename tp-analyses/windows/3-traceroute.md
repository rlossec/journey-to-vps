← [Retour à la feuille TP](../../tp-sheet.md#cmd-3-traceroute)

# `tracert` vers `54.36.100.9`

**But :** voir **approximativement** le chemin (box → FAI → OVH → VPS).

Les chemins et le nombre de `*` **changent** selon le FAI. On cherche la **forme**, pas les mêmes IP.

```powershell
tracert 54.36.100.9
```

## Sortie attendue (souvent plus lisible que Linux/WSL)

```
  1     1 ms     192.168.1.254
  2     3 ms     194.149.174.96
  3     3 ms     ns1.online.net [212.27.35.6]
  4     *        Délai d’attente de la demande dépassé.
  5     4 ms     be100.par-th2-pb1-nc5.fr.eu [213.186.32.181]
  8     5 ms     par3-cch01-vac-1-firewall.fr [57.130.3.80]
 …
 19     7 ms     vps-3229ca35.vps.ovh.net [54.36.100.9]
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `192.168.x.x` | **Box** / LAN. |
| Noms opérateur (`online.net`, …) | Sortie **FAI**. |
| `213.186…` / `be100.par-…` | Entrée **backbone OVH**. |
| `…vac…firewall…` | Douane hébergeur (VAC / firewall) — pont vers l’étape 4. |
| `*` / délai dépassé | Routeur **silencieux** aux sondes. **≠** lien cassé. |
| Dernier saut `54.36.100.9` | VPS atteint (quand la trace va au bout). |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-3-traceroute)
