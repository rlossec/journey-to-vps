← [Retour à la feuille TP](../tp-sheet.md#cmd-1-root)

# Root : « Qui gère `.tech` ? »

**But :** interroger **un serveur racine**, pas le résolveur du FAI.

## Commandes

**Linux**

```bash
dig NS tech. @a.root-servers.net
```

## Sortie attendue (Linux)

```
; <<>> DiG 9.20.24-1ubuntu0.3-Ubuntu <<>> NS tech. @a.root-servers.net
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15386
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 4, ADDITIONAL: 9
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;tech.                          IN      NS

;; AUTHORITY SECTION:
tech.                   172800  IN      NS      ns01.trs-dns.com.
tech.                   172800  IN      NS      ns10.trs-dns.org.
tech.                   172800  IN      NS      ns10.trs-dns.info.
tech.                   172800  IN      NS      ns01.trs-dns.net.

;; ADDITIONAL SECTION:
ns01.trs-dns.com.       172800  IN      A       64.96.1.1
ns01.trs-dns.com.       172800  IN      AAAA    2620:57:4001::1
… (A + AAAA des 3 autres NS)

;; SERVER: 2001:503:ba3e::2:30#53(a.root-servers.net) (UDP)
```

**Windows**

```powershell
nslookup -type=NS tech. a.root-servers.net
```

## Sortie attendue (Windows)

```
Serveur :   UnKnown
Address:  2001:503:ba3e::2:30

tech    nameserver = ns01.trs-dns.com
tech    nameserver = ns10.trs-dns.org
tech    nameserver = ns10.trs-dns.info
tech    nameserver = ns01.trs-dns.net
ns01.trs-dns.com        internet address = 64.96.1.1
…
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `ANSWER: 0` / `AUTHORITY: 4` | **Délégation**, pas l’IP du site. La racine dit *où* demander ensuite. |
| `NS ns01.trs-dns.com.` … | Les **TLD servers** de `.tech`. |
| `recursion requested but not available` | Normal : un root n’est pas un résolveur récursif. |
| Section Additional (`A` / `AAAA`) | **Glue** : IP des TLD, pour les joindre tout de suite. |
| Pas de `readresolve.tech` | Hors périmètre de la racine. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-1-root)
