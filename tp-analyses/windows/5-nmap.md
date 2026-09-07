← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-nmap)

# `nmap -sV -p 22,80,443 54.36.100.9`

**But :** ce que voit **l’extérieur**. Uniquement vers **notre** VPS.

```powershell
nmap -sV -p 22,80,443 54.36.100.9
```

`nmap` doit être installé. Sans nmap : s’appuyer sur `curl.exe` + la démo formateur.

## Sortie attendue (depuis un PC)

```
Nmap scan report for vps-3229ca35.vps.ovh.net (54.36.100.9)
Host is up (0.0088s latency).

PORT    STATE    SERVICE  VERSION
22/tcp  filtered ssh
80/tcp  open     http     Apache httpd
443/tcp open     ssl/http Apache httpd
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `80` / `443` `open` Apache | Point d’entrée public = Apache. |
| `22 filtered` | Pas de SSH utile sur 22 (il est sur **64483**). |
| `filtered` vs `closed` | Filtre / silence vs « port fermé, machine a répondu ». |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-nmap)
