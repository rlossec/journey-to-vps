← [Retour à la feuille TP](../tp-sheet.md#cmd-5-iptables-list)

# `sudo iptables -L -n -v --line-numbers`

**But :** lire le firewall **du VPS** (sous notre contrôle), pas l’Edge OVH.

**Windows :** pas d’équivalent (commande VPS).

## Commande

```bash
sudo iptables -L -n -v --line-numbers
```

## Sortie attendue (VPS)

```
Chain INPUT (policy DROP 9493 packets, 597K bytes)
num   pkts bytes target     prot opt in     out     source               destination
1     …          ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0
2     …          ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
3     …          ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:64483 ctstate NEW
4     …          ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80 ctstate NEW
5     …          ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:443 ctstate NEW
6     …          ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0

Chain FORWARD (policy DROP …)
Chain OUTPUT (policy ACCEPT …)
```

Les compteurs `pkts` / `bytes` changent en continu.

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `INPUT policy DROP` | Par défaut, l’entrant est **refusé**. |
| `dpt:64483` | SSH / admin (pas le 22). |
| `dpt:80` / `dpt:443` | HTTP / HTTPS publics. |
| `ACCEPT icmp` | Le `ping` de l’étape 3 peut passer. |
| `num` | Numéro de règle — sert à `-D INPUT N`. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-5-iptables-list)
