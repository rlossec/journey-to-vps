# 3. Routage — commandes

Objectif : l’IP est-elle **joignable**, et **par où** (approximativement) ?

Cible : `54.36.100.9` (`readresolve.tech`)

Réponses et analyses : [`response-analysis/3-routage.md`](response-analysis/3-routage.md)

## Tester la joignabilité (ping)

### Linux

```bash
ping -c 4 54.36.100.9
```

### Windows

```powershell
ping 54.36.100.9
```

## Voir le chemin approximatif (traceroute)

### Linux

```bash
traceroute 54.36.100.9
```

Souvent plus parlant (sondes ICMP, comme Windows) :

```bash
traceroute -I 54.36.100.9
```

Variantes selon distro : `tracepath 54.36.100.9` · `mtr 54.36.100.9`

### Windows

```powershell
tracert 54.36.100.9
```

Sans résoudre les noms :

```powershell
tracert -d 54.36.100.9
```

## Ce qu’on doit reconnaître

| Observation                                            | Lecture                                             |
| ------------------------------------------------------ | --------------------------------------------------- |
| Premier saut en `192.168.x.x`                          | Box / passerelle locale                             |
| Puis IPs du FAI                                        | On n’est pas encore chez OVH                        |
| `213.186.x.x`, `37.59.x.x`, `91.121.x.x`, `57.130.x.x` | Plages typiques **OVH**                             |
| Dernier saut `54.36.100.9`                             | Le VPS                                              |
| `* * *`                                                | Routeur qui ne répond pas à traceroute ≠ lien cassé |
| Ping ~ quelques ms depuis la France                    | Machine joignable en ICMP                           |
