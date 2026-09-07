← [Retour à la feuille TP](../../tp-sheet.md#cmd-1-tld)

# TLD : « Qui est autoritaire pour `readresolve.tech` ? »

```powershell
nslookup -type=NS readresolve.tech ns01.trs-dns.com
```

## Sortie attendue

```
Serveur :   UnKnown
Address:  2620:57:4001::1

readresolve.tech        nameserver = dns13.ovh.net
readresolve.tech        nameserver = ns13.ovh.net
```

← [Retour à la feuille TP](../../tp-sheet.md#cmd-1-tld)
