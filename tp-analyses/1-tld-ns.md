← [Retour à la feuille TP](../tp-sheet.md#cmd-1-tld)

# TLD : « Qui est autoritaire pour `readresolve.tech` ? »

**But :** interroger **un serveur TLD** `.tech` (pas le résolveur local).

## Commandes

**Linux**

```bash
dig NS readresolve.tech @ns01.trs-dns.com
```

**Windows**

```powershell
nslookup -type=NS readresolve.tech ns01.trs-dns.com
```

## Sortie attendue (Linux)

Même logique que l’étape root : souvent `ANSWER: 0` et les NS dans **Authority** (referral), plus éventuellement de la glue.

```
; <<>> DiG … <<>> NS readresolve.tech @ns01.trs-dns.com
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 2, …
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;readresolve.tech.              IN      NS

;; AUTHORITY SECTION:
readresolve.tech.       900     IN      NS      ns13.ovh.net.
readresolve.tech.       900     IN      NS      dns13.ovh.net.
```

## Sortie attendue (Windows)

```
Serveur :   UnKnown
Address:  2620:57:4001::1

readresolve.tech        nameserver = dns13.ovh.net
readresolve.tech        nameserver = ns13.ovh.net
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `SERVER` / `Address: 2620:57:4001::1` | On a bien parlé au TLD (`ns01.trs-dns.com`), pas à `8.8.8.8`. |
| `NS dns13.ovh.net` / `ns13.ovh.net` | Serveurs **autoritaires** OVH du domaine. |
| Toujours pas d’IP `54.36.100.9` | Le TLD délègue ; il ne publie pas le record A. |
| Warning récursion | Même cause qu’au root : serveur autoritaire, pas résolveur. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-1-tld)
