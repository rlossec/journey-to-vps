← [Retour à la feuille TP](../../tp-sheet.md#cmd-3-ping)

# `ping` vers `54.36.100.9`

**But :** l’IP est-elle **joignable** (ICMP) ? Ce n’est pas un test HTTPS.

```powershell
ping 54.36.100.9
```

## Sortie attendue

```
Réponse de 54.36.100.9 : octets=32 temps=8 ms TTL=52
… (4 lignes)
Paquets : envoyés = 4, reçus = 4, perdus = 0
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `Réponse de 54.36.100.9` | La machine **répond** en ICMP. |
| `perdus = 0` | Aucune perte sur cet essai. |
| `temps=8 ms` | RTT typique France → OVH. |
| ICMP ≠ HTTPS | Un ping OK n’implique pas que le port 443 est ouvert. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-3-ping)
