# 1. Résolution DNS — commandes

Objectif : voir la chaîne **racine → TLD → autoritaire → IP**, puis le **cache**.

Domaine : `readresolve.tech` · IP attendue : `54.36.100.9`

## Linux

Réponse complète (résolveur local, souvent **avec** cache) :

```bash
dig readresolve.tech A
dig NS readresolve.tech
```

Rejouer la chaîne comme un résolveur **sans** s’appuyer sur son cache (le plus parlant pour le schéma) :

```bash
dig +trace readresolve.tech
```

Forcer un résolveur public :

```bash
dig +trace @8.8.8.8 readresolve.tech
```

Sans récursion : « as-tu déjà la réponse en cache ? » — souvent vide si le serveur n’est pas autoritaire et n’a rien mémorisé.

```bash
dig +norecurse @8.8.8.8 readresolve.tech
```

Variante :

```bash
host readresolve.tech
```

## Windows (PowerShell / `nslookup`)

`dig` n’est en général **pas** installé. Équivalents :

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type A
Resolve-DnsName -Name "readresolve.tech" -Type NS
```

Chaîne **itérative** (équivalent pédagogique de `dig +trace`) :

```powershell
nslookup -type=NS tech. a.root-servers.net
nslookup -type=NS readresolve.tech ns01.trs-dns.com
nslookup -type=A readresolve.tech dns13.ovh.net
```

Sans récursion vers un résolveur public :

```powershell
Resolve-DnsName -Name "readresolve.tech" -Server "8.8.8.8" -NoRecursion
```

## Cache local

**Windows** — voir / vider le cache OS :

```powershell
ipconfig /displaydns
ipconfig /flushdns
```

**Linux** (selon le système) :

```bash
resolvectl flush-caches
```

Navigateur : vider le cache DNS interne (Chrome : `chrome://net-internals/#dns`) pour se rapprocher du scénario « ordinateur neuf ».

## Ce qu’on doit reconnaître dans la sortie

| Étape | On doit voir |
| --- | --- |
| Racine | délégation `.tech` vers `ns01.trs-dns.com` (et autres `trs-dns`) |
| TLD | `readresolve.tech` NS → `dns13.ovh.net` / `ns13.ovh.net` |
| Autoritaire | A → `54.36.100.9` |
| TTL | ici **3600** |
