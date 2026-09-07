← [Retour à la feuille TP](../tp-sheet.md#cmd-3-traceroute)

# `traceroute` / `tracert`

**But :** voir **approximativement** le chemin (box → FAI → OVH → VPS).

## Commandes

**Linux**

```bash
traceroute 54.36.100.9
```

**Windows**

```powershell
tracert 54.36.100.9
```

Les chemins et le nombre de `*` **changent** selon le FAI, l’heure, IPv4/IPv6. On cherche la **forme**, pas les mêmes IP.

## Sortie attendue (Linux / WSL — souvent incomplète)

```
traceroute to 54.36.100.9, 30 hops max
 1  … 172.27.0.1 …
 2  192.168.1.254 …
 3  … opérateur …
 6  be100.par-th2-pb1-nc5.fr.eu (213.186.32.181)
 7  * * *
 …
30  * * *
```

WSL + traceroute UDP : beaucoup de `*` après l’entrée OVH. Le ping prouve pourtant que la cible est up.

## Sortie attendue (Windows — souvent plus lisible)

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
| `* * *` / délai dépassé | Routeur **silencieux** aux sondes. **≠** lien cassé. |
| Dernier saut `54.36.100.9` | VPS atteint (quand la trace va au bout). |

← [Retour à la feuille TP](../tp-sheet.md#cmd-3-traceroute)
