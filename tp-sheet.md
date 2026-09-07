# Feuille TP — commandes

On va avoir besoin de 2 terminaux :
- un `bash` sur le VPS
- un `PowerShell` sur Windows (déso Hélène et Amine :broken_heart: ) 

Donc connectez vous au VPS :

```bash
ssh mbr-********@54.36.100.9 -p 64483
```

Il y a qq fois équivalence mais souvent les commandes différent entre le os.
Il y a du bon à prendre dans les deux.

Après chaque commande : **deux liens** (Linux / macOS et Windows) — chacun ouvre l’analyse de **son** OS.
S’il n’y a pas d’équivalent, c’est indiqué, sans fiche.

Chaque fiche rappelle la commande, montre une sortie attendue, commente les lignes utiles, et ramène ici.

---

## Introduction

Pas de commandes

---

## 1. Résolution DNS

### Résolution en 1 commande

<a id="cmd-1-trace"></a>

**Linux / macOS**

```bash
dig +trace readresolve.tech
```

[Sortie + analyse](tp-analyses/linux-macos/1-dig-trace.md)

**Windows** : pas d’équivalent.

Le résultat est abrupt mais tout est là !

### Etape par étape

#### 1. Demander à un root server : "Quel TLD gère .tech ?"

<a id="cmd-1-root"></a>

**Linux / macOS** : pas d’équivalent.

**Windows**

```powershell
nslookup -type=NS tech. a.root-servers.net
```

[Sortie + analyse](tp-analyses/windows/1-root-ns-tech.md)

#### 2. Demander à un TLD server : "Quel est le serveur autoritaire ?"

<a id="cmd-1-tld"></a>

**Linux / macOS** : pas d’équivalent.

**Windows**

```powershell
nslookup -type=NS readresolve.tech ns01.trs-dns.com
```

[Sortie + analyse](tp-analyses/windows/1-tld-ns.md)

#### 3. Demander à un serveur autoritaire : "Quelle est l'IP du serveur final ?"

<a id="cmd-1-auth"></a>

**Linux / macOS** : pas d’équivalent.

**Windows**

```powershell
nslookup -type=A readresolve.tech dns13.ovh.net
```

[Sortie + analyse](tp-analyses/windows/1-auth-a.md)

---

## 2. Enregistrement DNS

Objectif : Découvrir les enregistrements sur le VPS

### Record A

<a id="cmd-2-a"></a>

**Linux / macOS**

```bash
dig A readresolve.tech
```

[Sortie + analyse](tp-analyses/linux-macos/2-a.md)

**Windows**

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type A
```

[Sortie + analyse](tp-analyses/windows/2-a.md)

### Record AAAA

<a id="cmd-2-aaaa"></a>

**Linux / macOS**

```bash
dig AAAA readresolve.tech
```

[Sortie + analyse](tp-analyses/linux-macos/2-aaaa.md)

**Windows**

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type AAAA
```

[Sortie + analyse](tp-analyses/windows/2-aaaa.md)

### Records NS

<a id="cmd-2-ns"></a>

**Linux / macOS**

```bash
dig NS readresolve.tech
```

[Sortie + analyse](tp-analyses/linux-macos/2-ns.md)

**Windows**

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type NS
```

[Sortie + analyse](tp-analyses/windows/2-ns.md)

### Records MX

<a id="cmd-2-mx"></a>

**Linux / macOS**

```bash
dig MX readresolve.tech
```

[Sortie + analyse](tp-analyses/linux-macos/2-mx.md)

**Windows**

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type MX
```

[Sortie + analyse](tp-analyses/windows/2-mx.md)

### Record TXT

<a id="cmd-2-txt"></a>

**Linux / macOS**

```bash
dig TXT readresolve.tech
```

[Sortie + analyse](tp-analyses/linux-macos/2-txt.md)

**Windows**

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type TXT
```

[Sortie + analyse](tp-analyses/windows/2-txt.md)

### `www` — A

<a id="cmd-2-www-a"></a>

**Linux / macOS**

```bash
dig A www.readresolve.tech
```

[Sortie + analyse](tp-analyses/linux-macos/2-www-a.md)

**Windows**

```powershell
Resolve-DnsName -Name "www.readresolve.tech" -Type A
```

[Sortie + analyse](tp-analyses/windows/2-www-a.md)

### `www` — CNAME

<a id="cmd-2-www-cname"></a>

**Linux / macOS**

```bash
dig CNAME www.readresolve.tech
```

[Sortie + analyse](tp-analyses/linux-macos/2-www-cname.md)

**Windows**

```powershell
Resolve-DnsName -Name "www.readresolve.tech" -Type CNAME
```

[Sortie + analyse](tp-analyses/windows/2-www-cname.md)

### Whois

<a id="cmd-2-whois"></a>

**Linux / macOS**

```bash
whois readresolve.tech
```

[Sortie + analyse](tp-analyses/linux-macos/2-whois.md)

**Windows** : pas d’équivalent.

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

Objectif : l’IP est-elle **joignable**, et **par où** (approximativement) ?

### Ping

<a id="cmd-3-ping"></a>

**Linux / macOS**

```bash
ping -c 4 54.36.100.9
```

[Sortie + analyse](tp-analyses/linux-macos/3-ping.md)

**Windows**

```powershell
ping 54.36.100.9
```

[Sortie + analyse](tp-analyses/windows/3-ping.md)

### Traceroute

<a id="cmd-3-traceroute"></a>

**Linux / macOS**

```bash
traceroute 54.36.100.9
```

[Sortie + analyse](tp-analyses/linux-macos/3-traceroute.md)

**Windows**

```powershell
tracert 54.36.100.9
```

[Sortie + analyse](tp-analyses/windows/3-traceroute.md)

### Variante sondes / affichage

<a id="cmd-3-traceroute-icmp"></a>

**Linux / macOS**

```bash
traceroute -I 54.36.100.9
```

[Sortie + analyse](tp-analyses/linux-macos/3-traceroute-icmp.md)

**Windows**

```powershell
tracert -d 54.36.100.9
```

[Sortie + analyse](tp-analyses/windows/3-traceroute-icmp.md)

**À reconnaître** : box (`192.168.x.x`) → FAI → plages OVH → `54.36.100.9`. Les `* * *` ≠ lien cassé.

---

## 4. Infrastructure OVH

**Pas de commande** sur cette partie : on n’a pas la main sur HCAP / VAC / edge FW. Observation sur captures console OVH.

---

## 5. Configuration du VPS

Objectif : firewall local (`iptables`), **qui écoute** (ports / sockets), reverse proxy frontend → backend.

Commandes **VPS** = bash Linux (même depuis un PC Windows, via SSH).

### 5a. Lire le firewall — VPS

<a id="cmd-5-iptables-list"></a>

```bash
sudo iptables -L -n -v --line-numbers
```

[Sortie + analyse](tp-analyses/linux-macos/5-iptables-list.md)

**Windows** : pas d’équivalent.

<a id="cmd-5-iptables-rules"></a>

```bash
sudo cat /etc/iptables/rules.v4
```

[Sortie + analyse](tp-analyses/linux-macos/5-iptables-rules.md)

**Windows** : pas d’équivalent.

### 5b. Atelier port 443 — Local puis VPS

**Local** (avant / après le blocage) :

<a id="cmd-5-curl-https"></a>

**Linux / macOS**

```bash
curl -I --max-time 8 https://readresolve.tech
```

[Sortie + analyse](tp-analyses/linux-macos/5-curl-https.md)

**Windows**

```powershell
curl.exe -I --max-time 8 https://readresolve.tech
```

[Sortie + analyse](tp-analyses/windows/5-curl-https.md)

**VPS** (ne pas toucher au SSH / ports 22 ou 64483) :

<a id="cmd-5-iptables-drop"></a>

```bash
sudo iptables -I INPUT 1 -p tcp --dport 443 -j DROP
```

[Sortie + analyse](tp-analyses/linux-macos/5-iptables-drop.md)

**Windows** : pas d’équivalent.

<a id="cmd-5-iptables-input"></a>

```bash
sudo iptables -L INPUT --line-numbers
```

[Sortie + analyse](tp-analyses/linux-macos/5-iptables-input.md)

**Windows** : pas d’équivalent.

<a id="cmd-5-iptables-delete"></a>

```bash
sudo iptables -D INPUT 1
```

[Sortie + analyse](tp-analyses/linux-macos/5-iptables-delete.md)

**Windows** : pas d’équivalent.

`ping 54.36.100.9` peut rester OK : ICMP ≠ HTTPS.

### 5c. Ports / sockets — VPS

<a id="cmd-5-ss"></a>

```bash
sudo ss -tlnp
```

[Sortie + analyse](tp-analyses/linux-macos/5-ss.md)

**Windows** : pas d’équivalent.

<a id="cmd-5-ss-grep"></a>

```bash
sudo ss -tlnp | grep -E ':80|:443|:22|:64483|:90'
```

[Sortie + analyse](tp-analyses/linux-macos/5-ss-grep.md)

**Windows** : pas d’équivalent.

Repérer : `*:80` / `*:443` (public) vs `127.0.0.1:…` (local seulement).

### 5d. Ce que voit l’extérieur — Local

<a id="cmd-5d-curl"></a>

**Linux / macOS**

```bash
curl -I https://readresolve.tech
```

[Sortie + analyse](tp-analyses/linux-macos/5-curl-https.md)

**Windows**

```powershell
curl.exe -I https://readresolve.tech
```

[Sortie + analyse](tp-analyses/windows/5-curl-https.md)

<a id="cmd-5-nmap"></a>

**Linux / macOS**

```bash
nmap -sV -p 22,80,443 54.36.100.9
```

[Sortie + analyse](tp-analyses/linux-macos/5-nmap.md)

**Windows**

```powershell
nmap -sV -p 22,80,443 54.36.100.9
```

[Sortie + analyse](tp-analyses/windows/5-nmap.md)

`nmap` uniquement vers **notre** VPS.

### 5e. Reverse proxy Apache — VPS (lecture seule)

<a id="cmd-5-apache-s"></a>

```bash
sudo apache2ctl -S
```

[Sortie + analyse](tp-analyses/linux-macos/5-apache-s.md)

**Windows** : pas d’équivalent.

<a id="cmd-5-apache-m"></a>

```bash
sudo apache2ctl -M
```

[Sortie + analyse](tp-analyses/linux-macos/5-apache-m.md)

**Windows** : pas d’équivalent.

<a id="cmd-5-ls-sites"></a>

```bash
ls -la /etc/apache2/sites-enabled/
```

[Sortie + analyse](tp-analyses/linux-macos/5-ls-sites.md)

**Windows** : pas d’équivalent.

<a id="cmd-5-grep-apache"></a>

```bash
sudo grep -RniE 'ProxyPass|ProxyPassReverse|ServerName|VirtualHost|Listen' /etc/apache2/
```

[Sortie + analyse](tp-analyses/linux-macos/5-grep-apache.md)

**Windows** : pas d’équivalent.

---

## Mémo rapide

| Partie               | Commandes clés                                            |
| -------------------- | --------------------------------------------------------- |
| 1 Résolution DNS     | `dig +trace` · `Resolve-DnsName` / `nslookup` · flush DNS |
| 2 Enregistrement DNS | `dig A/NS/MX/TXT` · `whois`                               |
| 3 Routage            | `ping` · `traceroute` / `tracert`                         |
| 4 Infra OVH          | captures formateur (pas de CLI élève)                     |
| 5 VPS                | `iptables` · `ss` · `curl` · `nmap` · conf Apache         |
