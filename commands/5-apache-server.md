# 5. Ports et Apache — commandes

Objectif : **qui écoute**, sur quelle interface ; ce que voit l’extérieur ; le site répond bien en Apache.

Cible : `readresolve.tech` / `54.36.100.9`. `ss` et la conf Apache : **sur le VPS** (`sudo`). `nmap` : vers **notre** VPS uniquement.

## Sur le VPS — sockets

```bash
sudo ss -tlnp
```

`-t` TCP · `-l` listening · `-n` ports numériques · `-p` process.

Repérer : `0.0.0.0:443` / `:80` (public) vs `127.0.0.1:…` (local seulement).

Variante :

```bash
sudo ss -tlnp | grep -E ':80|:443|:22'
```

## Depuis un PC — ce que l’Internet voit

**Linux / Windows** (en-têtes, sans démonter HTTP) :

```bash
curl -I https://readresolve.tech
```

Attendu : `Server: Apache`, statut 200.

Scan de **notre** machine (si `nmap` est installé) :

```bash
nmap -sV -p 22,80,443 54.36.100.9
```

## Windows (sans `ss`)

```powershell
Get-NetTCPConnection -State Listen |
  Select-Object LocalAddress, LocalPort, OwningProcess
curl.exe -I https://readresolve.tech
```

## Apache (captures formateur / root)

Fichiers typiques : `/etc/apache2/sites-enabled/` · directives `ProxyPass` / `ProxyPassReverse` · `Listen 80` / `Listen 443`. On **montre** la conf du cas d’étude, on ne la réécrit pas ici.
