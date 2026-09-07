← [Retour à la feuille TP](../tp-sheet.md#cmd-5-iptables-drop)

# `iptables -I` : bloquer le 443

**But :** insérer un `DROP` **en tête** de INPUT, avant l’`ACCEPT` 443.

**Windows :** pas d’équivalent (commande VPS). Ne pas toucher au SSH (22 / 64483).

## Commande

```bash
sudo iptables -I INPUT 1 -p tcp --dport 443 -j DROP
```

## Sortie attendue

Aucune ligne : succès silencieux. Vérifier avec `sudo iptables -L INPUT --line-numbers`.

## Lignes importantes (lecture de la commande)

| Fragment | Lecture |
| --- | --- |
| `-I INPUT 1` | **Insert** en position 1 (prioritaire). |
| `-p tcp --dport 443` | Uniquement HTTPS. |
| `-j DROP` | Paquet ignoré (timeout côté client), pas un `REJECT` explicite. |

Effet : `curl https://…` échoue ; `ping` et SSH **64483** restent OK.

← [Retour à la feuille TP](../tp-sheet.md#cmd-5-iptables-drop)
