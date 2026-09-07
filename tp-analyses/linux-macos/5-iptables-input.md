← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-iptables-input)

# `sudo iptables -L INPUT --line-numbers`

**But :** confirmer que la règle DROP 443 est bien **n° 1**.

```bash
sudo iptables -L INPUT --line-numbers
```

## Sortie attendue (pendant l’atelier)

```
Chain INPUT (policy DROP)
num  target     prot opt source               destination
1    DROP       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:443
2    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
3    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
4    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:64483 ctstate NEW
…
     ACCEPT     tcp  --  …                    …                    tcp dpt:443 ctstate NEW
```

La règle `ACCEPT … dpt:443` est toujours là, **plus bas** : elle ne s’applique plus, l’ordre compte.

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| Ligne `1 DROP … dpt:443` | C’est celle qu’on vient d’insérer. |
| `ACCEPT 443` plus loin | Inatteignable tant que le DROP est au-dessus. |
| Numéro `1` | C’est celui à donner à `-D INPUT 1` pour restaurer. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-iptables-input)
