# 4. Firewall — commandes

Objectif : voir les **règles VPS**, puis le **TP 443**. HCAP / VAC : pas de CLI sur la machine — captures console OVH (formateur).

À faire **sur le VPS** (souvent `root` / `sudo`). Ne pas `DROP` le port **22** depuis une session SSH.

## Lire les règles

```bash
sudo iptables -L -n -v
sudo iptables -L INPUT -n -v --line-numbers
```

`-n` : pas de résolution DNS. `-v` : compteurs. `--line-numbers` : utile pour supprimer une règle.

Politique de la chaîne (`policy ACCEPT` ou `DROP`) : tout en haut de `INPUT`.

## TP — couper HTTPS, puis rétablir

Depuis un **PC** (avant / après) :

```bash
curl -I --max-time 8 https://readresolve.tech
```

Windows : la même ligne `curl`. Navigateur : DevTools → Réseau, recharger.

**Sur le VPS** — bloquer 443 (en tête de `INPUT`, sans toucher SSH) :

```bash
sudo iptables -I INPUT 1 -p tcp --dport 443 -j DROP
```

Variante pédagogique (`REJECT` → « connection refused » plutôt qu’un hang) :

```bash
sudo iptables -I INPUT 1 -p tcp --dport 443 -j REJECT
```

Retirer la règle n°1 (revérifier le numéro avant) :

```bash
sudo iptables -L INPUT --line-numbers
sudo iptables -D INPUT 1
```

`ping 54.36.100.9` peut rester OK : ICMP n’est pas le 443.

## OVH

Espace client : mitigation anti-DDoS, **Edge Network Firewall** si une config existe. Captures fournies par le formateur — pas une commande du VPS.
