← [Retour — atelier 443](../tp-sheet.md#cmd-5-curl-https) · [Retour — extérieur](../tp-sheet.md#cmd-5d-curl)

# `curl -I` HTTPS

**But :** le site répond-il en HTTPS ? Utilisé **avant / après** le blocage du 443, et à l’étape « ce que voit l’extérieur ».

## Commandes

**Linux**

```bash
curl -I --max-time 8 https://readresolve.tech
```

(`--max-time 8` : obligatoire pendant le TP DROP, sinon `curl` attend trop longtemps.)

**Windows**

```powershell
curl.exe -I --max-time 8 https://readresolve.tech
```

Sans `--max-time` pour l’observation « normale » (5d) :

```bash
curl -I https://readresolve.tech
```

## Sortie attendue (site OK)

```
HTTP/2 200
server: Apache
content-type: text/html
content-length: 454
```

Windows `curl.exe` peut afficher `HTTP/1.1 200 OK` et `Server: Apache` — même conclusion.

## Sortie attendue (443 bloqué par iptables)

Timeout / hang, puis erreur du type `Failed to connect` / `Operation timed out`. **Pas** un `200`.

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `200` + `Server: Apache` | Le chemin jusqu’au **frontend** Apache fonctionne. |
| Timeout après `DROP` | Le firewall VPS avale le trafic 443. |
| `ping` encore OK | ICMP n’est pas le port 443. |

← [Retour — atelier 443](../tp-sheet.md#cmd-5-curl-https) · [Retour — extérieur](../tp-sheet.md#cmd-5d-curl)
