← [Retour à la feuille TP](../tp-sheet.md#cmd-5-iptables-delete)

# `sudo iptables -D INPUT 1`

**But :** retirer la règle n° 1 (le DROP 443) et rétablir HTTPS.

**Windows :** pas d’équivalent (commande VPS).

## Commande

```bash
sudo iptables -D INPUT 1
```

À n’utiliser que si la ligne 1 est **bien** le DROP qu’on a ajouté. Vérifier avant avec `--line-numbers`.

## Sortie attendue

Aucune ligne. Contrôle : `curl -I https://readresolve.tech` redevient `200`.

## Lignes importantes (lecture de la commande)

| Fragment | Lecture |
| --- | --- |
| `-D INPUT 1` | Supprime la règle **numéro 1** de INPUT. |
| Pas `-D … 443` | On supprime par **numéro**, pas par port. Un mauvais numéro enlève autre chose. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-5-iptables-delete)
