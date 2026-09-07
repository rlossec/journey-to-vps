← [Retour à la feuille TP](../../tp-sheet.md#cmd-1-auth)

# Autoritaire : « Quelle est l’IP ? »

```powershell
nslookup -type=A readresolve.tech dns13.ovh.net
```

## Sortie attendue

```
Serveur :   UnKnown
Address:  2001:41d0:d00:f200::2

Nom :    readresolve.tech
Address:  54.36.100.9
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `Address: 54.36.100.9` | **IP publique du VPS.** Fin de la résolution. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-1-auth)
