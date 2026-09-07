← [Retour à la feuille TP](../tp-sheet.md#cmd-5-iptables-rules)

# `sudo cat /etc/iptables/rules.v4`

**But :** voir les règles **persistantes** (fichier), pas seulement la table en mémoire.

**Windows :** pas d’équivalent (commande VPS).

## Commande

```bash
sudo cat /etc/iptables/rules.v4
```

## Sortie attendue (VPS)

```
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 64483 -m conntrack --ctstate NEW -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -m conntrack --ctstate NEW -j ACCEPT
-A INPUT -p tcp -m tcp --dport 443 -m conntrack --ctstate NEW -j ACCEPT
-A INPUT -p icmp -j ACCEPT
COMMIT
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `*filter` | Table de filtrage. |
| `:INPUT DROP` | Même politique que `iptables -L`. |
| `-A INPUT … --dport 443` | HTTPS autorisé **au boot** (si le service iptables charge ce fichier). |
| `COMMIT` | Fin du bloc à appliquer. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-5-iptables-rules)
