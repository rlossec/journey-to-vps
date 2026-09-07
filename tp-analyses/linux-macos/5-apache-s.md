← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-apache-s)

# `sudo apache2ctl -S`

**But :** quels **vhosts** Apache (frontend public vs backend local). Lecture seule.

```bash
sudo apache2ctl -S
```

## Sortie attendue (schéma)

Les noms de fichiers varient. On cherche **Listen** / `*:80` / `*:443` vs `127.0.0.1`.

```
VirtualHost configuration:
*:80                   is a NameVirtualHost
         default server readresolve.tech (/etc/apache2/sites-enabled/…)
         port 80 namevhost readresolve.tech (/etc/apache2/sites-enabled/…)
*:443                  is a NameVirtualHost
         port 443 namevhost readresolve.tech (/etc/apache2/sites-enabled/…)
127.0.0.1:90xx         … (backends locaux, si déclarés ainsi)
```

Pas de capture collée à ce jour : confronter à **votre** sortie VPS / capture formateur.

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `*:443` + `readresolve.tech` | Vhost **public** HTTPS (frontend). |
| `127.0.0.1:…` | Vhost **local** = backend, pas exposé. |
| Chemin `sites-enabled/…` | Fichier à ouvrir ensuite (sans le modifier). |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-apache-s)
