← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-apache-m)

# `sudo apache2ctl -M`

**But :** modules chargés — repérer **`proxy_module`** (reverse proxy).

```bash
sudo apache2ctl -M
```

## Sortie attendue (extrait)

Liste longue. Les lignes utiles :

```
 proxy_module (shared)
 proxy_http_module (shared)
 ssl_module (shared)
 rewrite_module (shared)
```

Pas de capture collée à ce jour : l’essentiel est la présence de **`proxy_`**.

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `proxy_module` / `proxy_http_module` | Apache peut **relayer** vers un backend (`ProxyPass`). |
| `ssl_module` | HTTPS sur le frontend. |
| Autres modules | Hors périmètre (on ne fait pas un cours Apache). |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-apache-m)
