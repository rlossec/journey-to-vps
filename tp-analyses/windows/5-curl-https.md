← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-curl-https)

# `curl.exe -I` HTTPS

**But :** le site répond-il en HTTPS ? Avant / après le blocage du 443, et à l’étape « extérieur ».

Aussi utilisé en 5d sans `--max-time` : `curl.exe -I https://readresolve.tech`

← aussi [Ce que voit l’extérieur](../../tp-sheet.md#cmd-5d-curl)

```powershell
curl.exe -I --max-time 8 https://readresolve.tech
```

(`--max-time 8` : obligatoire pendant le TP DROP.)

## Sortie attendue (site OK)

```
HTTP/1.1 200 OK
Server: Apache
Content-Type: text/html
Content-Length: 454
```

(`curl.exe` reste souvent en HTTP/1.1 — même conclusion que Linux.)

## Sortie attendue (443 bloqué)

Timeout, puis erreur de connexion. **Pas** un `200`.

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `200` + `Server: Apache` | Le chemin jusqu’au **frontend** Apache fonctionne. |
| Timeout après `DROP` | Le firewall VPS avale le trafic 443. |
| `ping` encore OK | ICMP n’est pas le port 443. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-curl-https)
