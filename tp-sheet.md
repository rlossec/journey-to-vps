# Feuille TP — commandes

Cas d’étude : **readresolve.tech** → IP **54.36.100.9** (OVH)

À faire pendant la présentation. Sur chaque partie : taper la commande, observer la sortie, noter ce que vous retenez.

| Où ?      | Signification                                         |
| --------- | ----------------------------------------------------- |
| **Local** | votre PC (Windows / Linux / WSL)                      |
| **VPS**   | session SSH sur le serveur (fournie par le formateur) |

---

## Intro — le site répond, sans encore expliquer comment

**Local**

```bash
curl -I https://readresolve.tech
```

Navigateur : ouvrir `https://readresolve.tech` → DevTools → onglet **Réseau**.

---

## 1. Résolution DNS

Objectif : voir la chaîne **racine → TLD → autoritaire → IP**, puis le **cache**.

### Linux / Mac / WSL — Local

```bash
dig +trace readresolve.tech
dig readresolve.tech A
dig NS readresolve.tech
```

Résolveur public, sans cache local du résolveur :

```bash
dig +trace @8.8.8.8 readresolve.tech
dig +norecurse @8.8.8.8 readresolve.tech
```

Vider le cache DNS local (si disponible) :

```bash
resolvectl flush-caches
```

### Windows — Local

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type A
Resolve-DnsName -Name "readresolve.tech" -Type NS
```

Chaîne itérative (équivalent de `dig +trace`) :

```powershell
nslookup -type=NS tech. a.root-servers.net
nslookup -type=NS readresolve.tech ns01.trs-dns.com
nslookup -type=A readresolve.tech dns13.ovh.net
```

Cache OS :

```powershell
ipconfig /displaydns
ipconfig /flushdns
```

Navigateur : `chrome://net-internals/#dns` (vider le cache DNS Chrome).

**À reconnaître** : délégation `.tech` → NS OVH → A `54.36.100.9` · TTL **3600**.

---

## 2. Enregistrement DNS

Objectif : **lire** ce qui a été **écrit** dans la zone (types + TTL).

### Linux / Mac / WSL — Local

```bash
dig A readresolve.tech
dig AAAA readresolve.tech
dig NS readresolve.tech
dig MX readresolve.tech
dig TXT readresolve.tech
dig A www.readresolve.tech
dig CNAME www.readresolve.tech
whois readresolve.tech
```

### Windows — Local

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type A
Resolve-DnsName -Name "readresolve.tech" -Type AAAA
Resolve-DnsName -Name "readresolve.tech" -Type NS
Resolve-DnsName -Name "readresolve.tech" -Type MX
Resolve-DnsName -Name "readresolve.tech" -Type TXT
Resolve-DnsName -Name "www.readresolve.tech" -Type A
Resolve-DnsName -Name "www.readresolve.tech" -Type CNAME
```

**À reconnaître**

| Type  | Attendu                                                   |
| ----- | --------------------------------------------------------- |
| A     | `54.36.100.9`, TTL 3600                                   |
| AAAA  | absent                                                    |
| NS    | `dns13.ovh.net`, `ns13.ovh.net`                           |
| MX    | `mx4.mail.ovh.net` (prio 1), `mx3.mail.ovh.net` (prio 10) |
| TXT   | SPF OVH                                                   |
| `www` | **A** (pas un CNAME) vers la même IP                      |

Zone dans la console OVH : capture / démo formateur.

---

## 3. Routage Internet

Objectif : l’IP est-elle **joignable**, et **par où** (approximativement) ? BGP = concept (pas de commande BGP).

### Linux / Mac / WSL — Local

```bash
ping -c 4 54.36.100.9
traceroute 54.36.100.9
traceroute -I 54.36.100.9
```

### Windows — Local

```powershell
ping 54.36.100.9
tracert 54.36.100.9
tracert -d 54.36.100.9
```

**À reconnaître** : box (`192.168.x.x`) → FAI → plages OVH → `54.36.100.9`. Les `* * *` ≠ lien cassé.

---

## 4. Infrastructure OVH

Objectif : comprendre le chemin **dans** OVH (HCAP, backbone, edge firewall, routeurs DC).

**Pas de commande élève** sur cette partie : on n’a pas la main sur HCAP / VAC / edge FW. Observation sur captures console OVH (formateur).

Repère : ce qui est **OVH** (amont) vs ce qui sera **notre** VPS (partie 5).

---

## 5. Configuration du VPS

Objectif : firewall local (`iptables`), **qui écoute** (ports / sockets), reverse proxy frontend → backend.

### 5a. Lire le firewall — VPS

```bash
sudo iptables -L -n -v --line-numbers
sudo cat /etc/iptables/rules.v4
```

### 5b. Atelier port 443 — Local puis VPS

**Local** (avant / après le blocage) :

```bash
# Linux / Mac / WSL
curl -I --max-time 8 https://readresolve.tech
```

```powershell
# Windows
curl.exe -I --max-time 8 https://readresolve.tech
```

**VPS** (ne pas toucher au SSH / ports 22 ou 64483) :

```bash
sudo iptables -I INPUT 1 -p tcp --dport 443 -j DROP
sudo iptables -L INPUT --line-numbers
sudo iptables -D INPUT 1
```

`ping 54.36.100.9` peut rester OK : ICMP ≠ HTTPS.

### 5c. Ports / sockets — VPS

```bash
sudo ss -tlnp
sudo ss -tlnp | grep -E ':80|:443|:22|:64483|:90'
```

Repérer : `*:80` / `*:443` (public) vs `127.0.0.1:…` (local seulement).

### 5d. Ce que voit l’extérieur — Local

```bash
# Linux / Mac / WSL
curl -I https://readresolve.tech
nmap -sV -p 22,80,443 54.36.100.9
```

```powershell
# Windows
curl.exe -I https://readresolve.tech
nmap -sV -p 22,80,443 54.36.100.9
```

`nmap` uniquement vers **notre** VPS.

### 5e. Reverse proxy Apache — VPS (lecture seule)

```bash
sudo apache2ctl -S
sudo apache2ctl -M
ls -la /etc/apache2/sites-enabled/
sudo grep -RniE 'ProxyPass|ProxyPassReverse|ServerName|VirtualHost|Listen' /etc/apache2/
```

---

## Mémo rapide

| Partie               | Commandes clés                                            |
| -------------------- | --------------------------------------------------------- |
| 1 Résolution DNS     | `dig +trace` · `Resolve-DnsName` / `nslookup` · flush DNS |
| 2 Enregistrement DNS | `dig A/NS/MX/TXT` · `whois`                               |
| 3 Routage            | `ping` · `traceroute` / `tracert`                         |
| 4 Infra OVH          | captures formateur (pas de CLI élève)                     |
| 5 VPS                | `iptables` · `ss` · `curl` · `nmap` · conf Apache         |
