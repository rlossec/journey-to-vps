# 4. Firewall — commandes

Objectif : voir les **règles VPS**, puis le **TP 443**. HCAP / VAC : captures console OVH (formateur).

À faire **sur le VPS** (`sudo`) pour `iptables`. Ne pas `DROP` le port **22** / **64483** depuis une session SSH.

Réponses et analyses : [`response-analysis/4-firewall.md`](response-analysis/4-firewall.md)

## Lire les règles iptables

### Linux

```bash
sudo iptables -L -n -v --line-numbers
sudo cat /etc/iptables/rules.v4
```

`-n` : pas de résolution DNS. `-v` : compteurs. `--line-numbers` : utile pour supprimer une règle.

### Windows

Sans objet sur le PC de formation — les règles du cas d’étude se lisent **sur le VPS**.

## TP — vérifier HTTPS avant / après (client)

### Linux

```bash
curl -I --max-time 8 https://readresolve.tech
```

### Windows

```powershell
curl.exe -I --max-time 8 https://readresolve.tech
```

## TP — bloquer / rétablir le port 443 (VPS)

```bash
sudo iptables -I INPUT 1 -p tcp --dport 443 -j DROP
# variante pédagogique :
# sudo iptables -I INPUT 1 -p tcp --dport 443 -j REJECT

sudo iptables -L INPUT --line-numbers
sudo iptables -D INPUT 1
```

`ping 54.36.100.9` peut rester OK : ICMP n’est pas le 443.
