# 5. Ports et Apache — commandes

Objectif : **qui écoute**, sur quelle interface ; ce que voit l’extérieur ; reverse proxy.

Cible : `readresolve.tech` / `54.36.100.9`. `ss` et conf Apache : **sur le VPS**. `nmap` : vers **notre** VPS uniquement.

Réponses et analyses : [`response-analysis/5-apache-server.md`](response-analysis/5-apache-server.md)

## Lister les sockets en écoute

### Linux

```bash
sudo ss -tlnp
sudo ss -tlnp | grep -E ':80|:443|:22|:64483|:90'
```

Repérer : `*:80` / `*:443` (public) vs `127.0.0.1:…` (local seulement).

### Windows

Écoute **locale** du PC (pas le VPS) — pour comparer l’idée « qui écoute » :

```powershell
Get-NetTCPConnection -State Listen |
  Select-Object LocalAddress, LocalPort, OwningProcess
```

## Voir ce que l’Internet reçoit (en-têtes)

### Linux

```bash
curl -I https://readresolve.tech
```

### Windows

```powershell
curl.exe -I https://readresolve.tech
```

## Scanner les ports publics de notre VPS

### Linux

```bash
nmap -sV -p 22,80,443 54.36.100.9
```

### Windows

```powershell
nmap -sV -p 22,80,443 54.36.100.9
```

## Lire la conf Apache (VPS, sans modifier)

```bash
sudo apache2ctl -S
sudo apache2ctl -M
ls -la /etc/apache2/sites-enabled/
sudo grep -RniE 'ProxyPass|ProxyPassReverse|ServerName|VirtualHost|Listen' /etc/apache2/
```

Fichiers utiles : `ports.conf`, `sites-enabled/`, `mods-enabled/`, chemins certs dans le vhost SSL.
