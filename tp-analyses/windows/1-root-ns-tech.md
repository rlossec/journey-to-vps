← [Retour à la feuille TP](../../tp-sheet.md#cmd-1-root)

# Root : « Qui gère `.tech` ? »

```powershell
nslookup -type=NS tech. a.root-servers.net
```

## Sortie attendue

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

← [Retour à la feuille TP](../../tp-sheet.md#cmd-1-root)
