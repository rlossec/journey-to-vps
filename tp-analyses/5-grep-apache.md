← [Retour à la feuille TP](../tp-sheet.md#cmd-5-grep-apache)

# `grep` ProxyPass / VirtualHost / Listen

**But :** trouver dans la conf les preuves du **reverse proxy** (frontend → backend).

**Windows :** pas d’équivalent (commande VPS).

## Commande

```bash
sudo grep -RniE 'ProxyPass|ProxyPassReverse|ServerName|VirtualHost|Listen' /etc/apache2/
```

## Sortie attendue (schéma)

Beaucoup de lignes (mods, sites). Extraire :

```
/etc/apache2/ports.conf: Listen 80
/etc/apache2/ports.conf: Listen 443
/etc/apache2/sites-enabled/….conf: <VirtualHost *:443>
/etc/apache2/sites-enabled/….conf: ServerName readresolve.tech
/etc/apache2/sites-enabled/….conf: ProxyPass / http://127.0.0.1:90xx/
/etc/apache2/sites-enabled/….conf: ProxyPassReverse / http://127.0.0.1:90xx/
```

Ports backend (`90xx`) et chemins de fichiers : à coller depuis le VPS / capture formateur.

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `Listen 80` / `443` | Apache écoute le public. |
| `ServerName readresolve.tech` | Vhost du cas d’étude. |
| `ProxyPass` → `127.0.0.1:…` | Le client parle au **frontend** ; Apache relaie au **backend** local. |
| `ProxyPassReverse` | Réécrit les en-têtes de redirection du backend. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-5-grep-apache)
