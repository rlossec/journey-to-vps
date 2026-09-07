← [Retour à la feuille TP](../../tp-sheet.md#cmd-1-trace)

# `dig +trace readresolve.tech`

```bash
dig +trace readresolve.tech
```

## Sortie attendue raccourcie (VPS)

```
; <<>> DiG 9.20.24-1ubuntu0.2-Ubuntu <<>> +trace readresolve.tech
;; global options: +cmd
.                       86366   IN      NS      a.root-servers.net.
.                       86366   IN      NS      b.root-servers.net.
… (13 identités de root)
;; Received 525 bytes from 127.0.0.53#53(127.0.0.53) in 2 ms

;; communications error to …#53: timed out
tech.                   172800  IN      NS      ns01.trs-dns.com.
tech.                   172800  IN      NS      ns01.trs-dns.net.
tech.                   172800  IN      NS      ns10.trs-dns.org.
tech.                   172800  IN      NS      ns10.trs-dns.info.
;; Received 677 bytes from …#53(d.root-servers.net) in 4 ms

readresolve.tech.       900     IN      NS      ns13.ovh.net.
readresolve.tech.       900     IN      NS      dns13.ovh.net.
;; Received 239 bytes from …#53(ns10.trs-dns.org) in 6 ms

readresolve.tech.       3600    IN      A       54.36.100.9
;; Received 265 bytes from 5.39.112.241#53(ns13.ovh.net) in 2 ms
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `. IN NS a.root-servers.net.` … | Départ : liste des **root**. |
| `tech. IN NS ns*.trs-dns.*` | La racine délègue le TLD **`.tech`**. |
| `readresolve.tech. IN NS dns13/ns13.ovh.net` | Le TLD délègue le domaine à **OVH**. |
| `A 54.36.100.9` | Le serveur autoritaire donne **l’IP**. Fin de chaîne. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-1-trace)
