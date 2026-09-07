← [Retour à la feuille TP](../tp-sheet.md#cmd-1-auth)

# Autoritaire : « Quelle est l’IP ? »

**But :** interroger **le serveur de noms du domaine** (`dns13.ovh.net`), qui a le record A.

## Commandes

**Linux**

```bash
dig A readresolve.tech @dns13.ovh.net
```

**Windows**

```powershell
nslookup -type=A readresolve.tech dns13.ovh.net
```

## Sortie attendue (Linux)

```
; <<>> DiG … <<>> A readresolve.tech @dns13.ovh.net
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, …
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, …

;; QUESTION SECTION:
;readresolve.tech.              IN      A

;; ANSWER SECTION:
readresolve.tech.       3600    IN      A       54.36.100.9

;; SERVER: …#53(dns13.ovh.net)
```

Le flag **`aa`** (authoritative answer) est le marqueur : la réponse vient de la zone, pas d’un cache de résolveur.

## Sortie attendue (Windows)

```
Serveur :   UnKnown
Address:  2001:41d0:d00:f200::2

Nom :    readresolve.tech
Address:  54.36.100.9
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `A 54.36.100.9` / `Address: 54.36.100.9` | **IP publique du VPS.** Fin de la résolution. |
| `aa` (dig) | Réponse **autoritaire**. |
| `TTL 3600` | 1 h : durée pendant laquelle un résolveur peut garder cette IP. |
| `SERVER` = `dns13.ovh.net` | On n’est plus chez le root ni le TLD. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-1-auth)
