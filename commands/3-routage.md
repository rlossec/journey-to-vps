# 3. Routage — commandes

Objectif : l’IP est-elle **joignable**, et **par où** (approximativement) ? Pas le détail TCP/HTTP.

Cible : `54.36.100.9` (`readresolve.tech`)

## Linux

```bash
ping -c 4 54.36.100.9
traceroute 54.36.100.9
```

Souvent plus parlant (sondes ICMP, comme Windows) :

```bash
traceroute -I 54.36.100.9
```

Variantes selon la distro : `tracepath 54.36.100.9` · `mtr 54.36.100.9` (vue temps réel).

## Windows

```powershell
ping 54.36.100.9
tracert 54.36.100.9
```

Sans résoudre les noms (plus rapide, comme la capture de l’étape) :

```powershell
tracert -d 54.36.100.9
```

## Ce qu’on doit reconnaître

| Observation | Lecture |
| --- | --- |
| Premier saut en `192.168.x.x` | Box / passerelle locale |
| Puis IPs du FAI | On n’est pas encore chez OVH |
| `213.186.x.x`, `37.59.x.x`, `91.121.x.x`, `57.130.x.x` | Plages typiques **OVH** |
| Dernier saut `54.36.100.9` | Le VPS |
| `* * *` | Routeur qui ne répond pas à traceroute ≠ lien cassé |
| Ping ~ quelques ms depuis la France | Machine joignable en ICMP |

Comparer deux postes (box différente, 4G) : chemins **différents**, même destination. Le navigateur, lui, n’affiche aucune de ces lignes.
