# Commandes, réponses et analyse — 3. Routage

Fiche atelier (commandes seules) : [`../3-routage.md`](../3-routage.md)

## Tester la joignabilité (ping)

### Linux

#### Commande

```bash
ping -c 4 54.36.100.9
```

#### Réponse (WSL)

```
PING 54.36.100.9 (54.36.100.9) 56(84) bytes of data.
64 bytes from 54.36.100.9: icmp_seq=1 ttl=51 time=9.03 ms
64 bytes from 54.36.100.9: icmp_seq=2 ttl=51 time=9.35 ms
64 bytes from 54.36.100.9: icmp_seq=3 ttl=51 time=8.11 ms
64 bytes from 54.36.100.9: icmp_seq=4 ttl=51 time=8.45 ms

--- 54.36.100.9 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 8.114/8.735/9.348/0.481 ms
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `PING 54.36.100.9 … 56(84) bytes` | ICMP echo vers l’IP du VPS ; taille de sonde. |
| `64 bytes from 54.36.100.9` | La cible a répondu (joignable en ICMP). |
| `icmp_seq=1…4` | Quatre sondes numérotées. |
| `ttl=51` | TTL IP restant à l’arrivée — indice de distance (pas le nombre exact de sauts). |
| `time≈8–9 ms` | RTT depuis le PC (France → OVH), cohérent. |
| `0% packet loss` | Aucune perte sur cet essai. |
| `rtt min/avg/max` | Synthèse des latences. |

#### Réponse (VPS)

```
PING 54.36.100.9 (54.36.100.9) 56(84) bytes of data.
64 bytes from 54.36.100.9: icmp_seq=1 ttl=64 time=0.135 ms
64 bytes from 54.36.100.9: icmp_seq=2 ttl=64 time=0.065 ms
64 bytes from 54.36.100.9: icmp_seq=3 ttl=64 time=0.066 ms
64 bytes from 54.36.100.9: icmp_seq=4 ttl=64 time=0.066 ms

--- 54.36.100.9 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3051ms
rtt min/avg/max/mdev = 0.065/0.083/0.135/0.030 ms
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `ttl=64` | Valeur « locale » typique (Linux) : on ping **soi-même** / la même machine. |
| `time≈0,06–0,13 ms` | Latence quasi nulle — pas le chemin Internet. |

### Windows

#### Commande

```powershell
ping 54.36.100.9
```

#### Réponse

```
Réponse de 54.36.100.9 : octets=32 temps=8 ms TTL=52
Réponse de 54.36.100.9 : octets=32 temps=8 ms TTL=52
Réponse de 54.36.100.9 : octets=32 temps=8 ms TTL=52
Réponse de 54.36.100.9 : octets=32 temps=8 ms TTL=52

Statistiques Ping pour 54.36.100.9:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 8ms, Maximum = 8ms, Moyenne = 8ms
```

#### Analyse

Même conclusion que WSL : IP joignable, ~8 ms, TTL ~52. Ping ≠ HTTPS, mais valide la couche IP avant traceroute.

---

## Voir le chemin approximatif (traceroute)

### Linux

#### Commande

```bash
traceroute 54.36.100.9
```

#### Réponse (WSL)

```
traceroute to 54.36.100.9 (54.36.100.9), 30 hops max, 60 byte packets
 1  ACER-FIXE-RTX-4.mshome.net (172.27.0.1)  0.352 ms  0.330 ms *
 2  192.168.1.254 (192.168.1.254)  2.778 ms *  2.764 ms
 3  station27.multimania.isdnet.net (194.149.174.124)  2.772 ms * 194.149.174.96 (194.149.174.96)  3.845 ms
 4  ns1.online.net (212.27.35.6)  2.717 ms  2.712 ms  3.836 ms
 5  * * *
 6  be100.par-th2-pb1-nc5.fr.eu (213.186.32.181)  3.814 ms  4.335 ms  4.328 ms
 7  * * *
 8  * * *
 9  * * *
10  * * *
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  * * *
20  * * *
21  * * *
22  * * *
23  * * *
24  * * *
25  * * *
26  * * *
27  * * *
28  * * *
29  * * *
30  * * *
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `30 hops max` | Limite de sauts sondés. |
| Saut 1 `172.27.0.1` / `mshome.net` | Passerelle virtuelle WSL / Hyper-V — pas encore la box « physique » seule. |
| Saut 2 `192.168.1.254` | Box / passerelle LAN. |
| Saut 3 `194.149.…` / multimania | Premiers routeurs côté opérateur / peering. |
| Saut 4 `ns1.online.net` | Réseau opérateur (Online / Scaleway-like selon le chemin). |
| Saut 5 `* * *` | Pas de réponse ICMP/UDP au traceroute — **≠** lien cassé. |
| Saut 6 `213.186.32.181` / `be100.par-…` | Plage / nom **OVH** (entrée backbone Paris). |
| Sauts 7–30 `*` | Fin de visibilité : les routeurs suivants ne répondent pas aux sondes ; le ping prouve pourtant que la cible est up. |

#### Réponse (VPS)

```
traceroute to 54.36.100.9 (54.36.100.9), 30 hops max, 60 byte packets
 1  vps-3229ca35.vps.ovh.net (54.36.100.9)  0.098 ms  0.014 ms  0.011 ms
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| Un seul saut vers `54.36.100.9` | On est déjà sur (ou immédiatement devant) la cible — traceroute depuis le VPS vers lui-même. |

### Windows

#### Commande

```powershell
tracert 54.36.100.9
```

#### Réponse

```
  1     1 ms    <1 ms     1 ms  192.168.1.254
  2     3 ms     3 ms     3 ms  194.149.174.96
  3     4 ms     3 ms     3 ms  ns1.online.net [212.27.35.6]
  4     *        *        *     Délai d’attente de la demande dépassé.
  5     4 ms     3 ms     3 ms  be100.par-th2-pb1-nc5.fr.eu [213.186.32.181]
  6     *        *        *     Délai d’attente de la demande dépassé.
  7     *        *        *     Délai d’attente de la demande dépassé.
  8     5 ms     3 ms     3 ms  par3-cch01-vac-1-firewall.fr [57.130.3.80]
  9     8 ms     5 ms     6 ms  37.59.16.2
 10     *        *        *     Délai d’attente de la demande dépassé.
 11     8 ms     8 ms     8 ms  be102.lil2-gra1-sbb1-nc5.fr.eu [91.121.215.176]
 12     9 ms    10 ms     9 ms  37.59.16.40
 13     *        *        *     Délai d’attente de la demande dépassé.
 14     *        *        *     Délai d’attente de la demande dépassé.
 15     *        *        *     Délai d’attente de la demande dépassé.
 16     *        *        *     Délai d’attente de la demande dépassé.
 17     *        *        *     Délai d’attente de la demande dépassé.
 18     *        *        *     Délai d’attente de la demande dépassé.
 19     7 ms     7 ms     8 ms  vps-3229ca35.vps.ovh.net [54.36.100.9]
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| Sauts 1–3 | LAN → opérateur (même idée que WSL). |
| `*` / délai dépassé | Routeur silencieux aux sondes. |
| `213.186…` `be100.par-…` | Entrée **OVH**. |
| `…vac-1-firewall.fr` `57.130.3.80` | Zone **VAC / anti-DDoS / firewall** OVH — douane hébergeur. |
| `37.59…`, `91.121…` | Suite du chemin interne OVH (datacenter / backbone). |
| Dernier saut `54.36.100.9` | Le VPS atteint. |

Plus parlant que la trace WSL pour l’étape « infra OVH » : le nom `…vac…firewall…` rend la douane visible.
