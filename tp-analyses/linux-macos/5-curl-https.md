← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-curl-https)

# `curl -I` HTTPS

**But :** le site répond-il en HTTPS ? Avant / après le blocage du 443, et à l’étape « extérieur ».

Aussi utilisé en 5d sans `--max-time` : `curl -I https://readresolve.tech`

← aussi [Ce que voit l’extérieur](../../tp-sheet.md#cmd-5d-curl)

```bash
curl -I --max-time 8 https://readresolve.tech
```

(`--max-time 8` : obligatoire pendant le TP DROP.)

## Sortie attendue (site OK)

```
HTTP/2 200
server: Apache
content-type: text/html
content-length: 454
```

## Sortie attendue (443 bloqué)

Timeout, puis `Failed to connect` / `Operation timed out`. **Pas** un `200`.

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `200` + `Server: Apache` | Le chemin jusqu’au **frontend** Apache fonctionne. |
| Timeout après `DROP` | Le firewall VPS avale le trafic 443. |
| `ping` encore OK | ICMP n’est pas le port 443. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-curl-https)
