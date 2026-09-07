← [Retour à la feuille TP](../tp-sheet.md#cmd-3-ping)

# `ping` vers `54.36.100.9`

**But :** l’IP est-elle **joignable** (ICMP) ? Ce n’est pas un test HTTPS.

## Commandes

**Linux**

```bash
ping -c 4 54.36.100.9
```

**Windows**

```powershell
ping 54.36.100.9
```

## Sortie attendue (Linux, depuis un PC)

```
PING 54.36.100.9 (54.36.100.9) 56(84) bytes of data.
64 bytes from 54.36.100.9: icmp_seq=1 ttl=51 time=9.03 ms
64 bytes from 54.36.100.9: icmp_seq=2 ttl=51 time=9.35 ms
64 bytes from 54.36.100.9: icmp_seq=3 ttl=51 time=8.11 ms
64 bytes from 54.36.100.9: icmp_seq=4 ttl=51 time=8.45 ms

--- 54.36.100.9 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss
```

Depuis **le VPS vers lui-même** : `ttl=64` et `time≈0,1 ms` — ce n’est plus le chemin Internet.

## Sortie attendue (Windows)

```
Réponse de 54.36.100.9 : octets=32 temps=8 ms TTL=52
… (4 lignes)
Paquets : envoyés = 4, reçus = 4, perdus = 0
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `64 bytes from` / `Réponse de 54.36.100.9` | La machine **répond** en ICMP. |
| `0% packet loss` | Aucune perte sur cet essai. |
| `time≈8–9 ms` | RTT typique France → OVH. |
| ICMP ≠ HTTPS | Un ping OK n’implique pas que le port 443 est ouvert. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-3-ping)
