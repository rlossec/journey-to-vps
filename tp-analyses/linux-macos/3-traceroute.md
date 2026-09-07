← [Retour à la feuille TP](../../tp-sheet.md#cmd-3-traceroute)

# `traceroute` vers `54.36.100.9`

**But :** voir **approximativement** le chemin (box → FAI → OVH → VPS).

Les chemins et le nombre de `*` **changent** selon le FAI. On cherche la **forme**, pas les mêmes IP.

```bash
traceroute 54.36.100.9
```

## Sortie attendue (souvent incomplète sous WSL)

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

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `192.168.x.x` | **Box** / LAN. |
| Noms opérateur | Sortie **FAI**. |
| `213.186…` / `be100.par-…` | Entrée **backbone OVH**. |
| `* * *` | Routeur **silencieux** aux sondes. **≠** lien cassé. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-3-traceroute)
