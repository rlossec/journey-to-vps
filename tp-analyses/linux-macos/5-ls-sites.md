← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-ls-sites)

# `ls -la /etc/apache2/sites-enabled/`

**But :** quels vhosts sont **actifs** (liens vers `sites-available`).

```bash
ls -la /etc/apache2/sites-enabled/
```

## Sortie attendue (schéma)

```
lrwxrwxrwx … 000-default.conf -> ../sites-available/000-default.conf
lrwxrwxrwx … readresolve-le-ssl.conf -> ../sites-available/…
lrwxrwxrwx … …backend….conf -> ../sites-available/…
```

Noms exacts : capture formateur / votre VPS.

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `l` / `->` | **Lien symbolique** : le site est activé (`a2ensite`). |
| Plusieurs fichiers | Souvent frontend SSL + default + backends. |
| On ne **modifie** pas | Lecture seule pour le TP. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-ls-sites)
