# Feuille TP — commandes

On va avoir besoin de 2 terminaux :
- un `bash` sur le VPS
- un `PowerShell` sur Windows (déso Hélène et Amine :broken_heart: )

Donc connectez vous au VPS :

```bash
ssh mbr-********@54.36.100.9 -p 64483
```

Il y a qq fois équivalence mais souvent la nature des commandes différent entre les OS.
Il y a du bon à prendre dans les deux.  

Après chaque commande, un bandeau **🐧 Linux / macOS** et **🪟 Windows** : le lien ouvre l’analyse de **votre** OS.

S’il n’y a pas d’équivalent, c’est indiqué en italique, sans fiche.

Chaque fiche contient :
- un rappel de la commande, 
- un exemple de sortie attendue,
- une analyse des lignes utiles,
- un lien vers cette page 

---

## Introduction

Pas de commandes

---

## 1. Résolution DNS

### 1.1. Résolution en 1 commande

<a id="cmd-1-trace"></a>

🐧 **Linux / macOS** — [Analyse](tp-analyses/linux-macos/1-dig-trace.md)

```bash
dig +trace readresolve.tech
```

🪟 **Windows** — *pas d’équivalent*

Le résultat est abrupt mais tout est là !

### 1.2. Etape par étape

#### 1.2.1. Demander à un root server : "Quel TLD gère .tech ?"

<a id="cmd-1-root"></a>

🐧 **Linux / macOS** — *pas d’équivalent*

🪟 **Windows** — [Analyse](tp-analyses/windows/1-root-ns-tech.md)

```powershell
nslookup -type=NS tech. a.root-servers.net
```

#### 1.2.2. Demander à un TLD server : "Quel est le serveur autoritaire ?"

<a id="cmd-1-tld"></a>

🐧 **Linux / macOS** — *pas d’équivalent*

🪟 **Windows** — [Analyse](tp-analyses/windows/1-tld-ns.md)

```powershell
nslookup -type=NS readresolve.tech ns01.trs-dns.com
```

#### 1.2.3. Demander à un serveur autoritaire : "Quelle est l'IP du serveur final ?"

<a id="cmd-1-auth"></a>

🐧 **Linux / macOS** — *pas d’équivalent*

🪟 **Windows** — [Analyse](tp-analyses/windows/1-auth-a.md)

```powershell
nslookup -type=A readresolve.tech dns13.ovh.net
```

---

## 2. Enregistrement DNS

Objectif : Découvrir les enregistrements sur le VPS

### 2.1. Records A et AAAA

<a id="cmd-2-a-aaaa"></a>

🐧 **Linux / macOS** — [Analyse](tp-analyses/linux-macos/2-a-aaaa.md)

```bash
dig A readresolve.tech
```
```bash
dig AAAA readresolve.tech
```

🪟 **Windows** — [Analyse](tp-analyses/windows/2-a-aaaa.md)

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type A
```
```powershell
Resolve-DnsName -Name "readresolve.tech" -Type AAAA
```

### 2.2. Records NS

<a id="cmd-2-ns"></a>

🐧 **Linux / macOS** — [Analyse](tp-analyses/linux-macos/2-ns.md)

```bash
dig NS readresolve.tech
```

🪟 **Windows** — [Analyse](tp-analyses/windows/2-ns.md)

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type NS
```

### 2.3. Record CNAME (`www`)

<a id="cmd-2-cname"></a>

🐧 **Linux / macOS** — [Analyse](tp-analyses/linux-macos/2-www-cname.md)

```bash
dig CNAME www.readresolve.tech
```

🪟 **Windows** — [Analyse](tp-analyses/windows/2-www-cname.md)

```powershell
Resolve-DnsName -Name "www.readresolve.tech" -Type CNAME
```

### 2.4. À creuser (sans fiche)

MX, TXT, A de `www`, whois : à lancer pour voir la zone, pas d’analyse dédiée.

🐧 **Linux / macOS**

```bash
dig MX readresolve.tech
dig TXT readresolve.tech
dig A www.readresolve.tech
whois readresolve.tech
```

🪟 **Windows**

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type MX
Resolve-DnsName -Name "readresolve.tech" -Type TXT
Resolve-DnsName -Name "www.readresolve.tech" -Type A
```

🪟 **Windows** — *pas d’équivalent `whois`*

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

Objectif : l’IP est-elle **joignable**, et **par où** ?

### Ping

<a id="cmd-3-ping"></a>

🐧 **Linux / macOS** — [Analyse](tp-analyses/linux-macos/3-ping.md)

```bash
ping -c 4 54.36.100.9
```

🪟 **Windows** — [Analyse](tp-analyses/windows/3-ping.md)

```powershell
ping 54.36.100.9
```

### Traceroute

<a id="cmd-3-traceroute"></a>

🐧 **Linux / macOS** — [Analyse](tp-analyses/linux-macos/3-traceroute.md)

```bash
traceroute 54.36.100.9
```

🪟 **Windows** — [Analyse](tp-analyses/windows/3-traceroute.md)

```powershell
tracert 54.36.100.9
```

### Variante sondes / affichage

<a id="cmd-3-traceroute-icmp"></a>

🐧 **Linux / macOS** — [Analyse](tp-analyses/linux-macos/3-traceroute-icmp.md)

```bash
traceroute -I 54.36.100.9
```

🪟 **Windows** — [Analyse](tp-analyses/windows/3-traceroute-icmp.md)

```powershell
tracert -d 54.36.100.9
```

---

## 4. Infrastructure OVH

**Pas de commande** sur cette partie : on n’a pas la main sur HCAP / VAC / edge FW. Observation sur captures console OVH.

---

## 5. Configuration du VPS

Objectif : firewall local (`iptables`), **qui écoute** (ports / sockets), reverse proxy frontend → backend.

Commandes **VPS** = bash Linux (même depuis un PC Windows, via SSH).

### 5a. Lire le firewall — VPS

<a id="cmd-5-iptables-list"></a>

🐧 **Linux (VPS)** — [Analyse](tp-analyses/linux-macos/5-iptables-list.md)

```bash
sudo iptables -L -n -v --line-numbers
```

🪟 **Windows** — *pas d’équivalent*

<a id="cmd-5-iptables-rules"></a>

🐧 **Linux (VPS)** — [Analyse](tp-analyses/linux-macos/5-iptables-rules.md)

```bash
sudo cat /etc/iptables/rules.v4
```

🪟 **Windows** — *pas d’équivalent*

### 5b. Atelier port 443 — Local puis VPS

**Local** (avant / après le blocage) :

<a id="cmd-5-curl-https"></a>

🐧 **Linux / macOS** — [Analyse](tp-analyses/linux-macos/5-curl-https.md)

```bash
curl -I --max-time 8 https://readresolve.tech
```

🪟 **Windows** — [Analyse](tp-analyses/windows/5-curl-https.md)

```powershell
curl.exe -I --max-time 8 https://readresolve.tech
```

**VPS** (ne pas toucher au SSH / ports 22 ou 64483) :

<a id="cmd-5-iptables-drop"></a>

🐧 **Linux (VPS)** — [Analyse](tp-analyses/linux-macos/5-iptables-drop.md)

```bash
sudo iptables -I INPUT 1 -p tcp --dport 443 -j DROP
```

🪟 **Windows** — *pas d’équivalent*

<a id="cmd-5-iptables-input"></a>

🐧 **Linux (VPS)** — [Analyse](tp-analyses/linux-macos/5-iptables-input.md)

```bash
sudo iptables -L INPUT --line-numbers
```

🪟 **Windows** — *pas d’équivalent*

<a id="cmd-5-iptables-delete"></a>

🐧 **Linux (VPS)** — [Analyse](tp-analyses/linux-macos/5-iptables-delete.md)

```bash
sudo iptables -D INPUT 1
```

🪟 **Windows** — *pas d’équivalent*

`ping 54.36.100.9` peut rester OK : ICMP ≠ HTTPS.

### 5c. Ports / sockets — VPS

<a id="cmd-5-ss"></a>

🐧 **Linux (VPS)** — [Analyse](tp-analyses/linux-macos/5-ss.md)

```bash
sudo ss -tlnp
```

🪟 **Windows** — *pas d’équivalent*

<a id="cmd-5-ss-grep"></a>

🐧 **Linux (VPS)** — [Analyse](tp-analyses/linux-macos/5-ss-grep.md)

```bash
sudo ss -tlnp | grep -E ':80|:443|:22|:64483|:90'
```

🪟 **Windows** — *pas d’équivalent*

Repérer : `*:80` / `*:443` (public) vs `127.0.0.1:…` (local seulement).

### 5d. Ce que voit l’extérieur — Local

<a id="cmd-5d-curl"></a>

🐧 **Linux / macOS** — [Analyse](tp-analyses/linux-macos/5-curl-https.md)

```bash
curl -I https://readresolve.tech
```

🪟 **Windows** — [Analyse](tp-analyses/windows/5-curl-https.md)

```powershell
curl.exe -I https://readresolve.tech
```

<a id="cmd-5-nmap"></a>

🐧 **Linux / macOS** — [Analyse](tp-analyses/linux-macos/5-nmap.md)

```bash
nmap -sV -p 22,80,443 54.36.100.9
```

🪟 **Windows** — [Analyse](tp-analyses/windows/5-nmap.md)

```powershell
nmap -sV -p 22,80,443 54.36.100.9
```

`nmap` uniquement vers **notre** VPS.

### 5e. Reverse proxy Apache — VPS (lecture seule)

<a id="cmd-5-apache-s"></a>

🐧 **Linux (VPS)** — [Analyse](tp-analyses/linux-macos/5-apache-s.md)

```bash
sudo apache2ctl -S
```

🪟 **Windows** — *pas d’équivalent*

<a id="cmd-5-apache-m"></a>

🐧 **Linux (VPS)** — [Analyse](tp-analyses/linux-macos/5-apache-m.md)

```bash
sudo apache2ctl -M
```

🪟 **Windows** — *pas d’équivalent*

<a id="cmd-5-ls-sites"></a>

🐧 **Linux (VPS)** — [Analyse](tp-analyses/linux-macos/5-ls-sites.md)

```bash
ls -la /etc/apache2/sites-enabled/
```

🪟 **Windows** — *pas d’équivalent*

<a id="cmd-5-grep-apache"></a>

🐧 **Linux (VPS)** — [Analyse](tp-analyses/linux-macos/5-grep-apache.md)

```bash
sudo grep -RniE 'ProxyPass|ProxyPassReverse|ServerName|VirtualHost|Listen' /etc/apache2/
```

🪟 **Windows** — *pas d’équivalent*

---

## Mémo rapide

| Partie               | 🐧 Linux / macOS                                   | 🪟 Windows                                           |
| -------------------- | -------------------------------------------------- | ---------------------------------------------------- |
| 1 Résolution DNS     | `dig +trace`                                       | `nslookup` (root → TLD → autoritaire)                |
| 2 Enregistrement DNS | `dig A/AAAA` · `NS` · `CNAME`                      | `Resolve-DnsName` A/AAAA · NS · CNAME                |
| 3 Routage            | `ping` · `traceroute` · `traceroute -I`            | `ping` · `tracert` · `tracert -d`                    |
| 4 Infra OVH          | captures formateur (pas de CLI)                    | captures formateur (pas de CLI)                      |
| 5 VPS                | `iptables` · `ss` · `curl` · `nmap` · conf Apache  | local : `curl.exe` · `nmap` — le reste via SSH/bash  |
