# Commandes, réponses et analyse — 4. Firewall

Fiche atelier (commandes seules) : [`../4-firewall.md`](../4-firewall.md)

## Lire les règles iptables

### Linux (VPS)

#### Commande

```bash
sudo iptables -L -n -v --line-numbers
```

#### Réponse (VPS Admin)

```
Chain INPUT (policy DROP 9493 packets, 597K bytes)
num   pkts bytes target     prot opt in     out     source               destination
1     1759  215K ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0
2    1601K  514M ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
3      123  7344 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:64483 ctstate NEW
4    13824  732K ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80 ctstate NEW
5    27315 1551K ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:443 ctstate NEW
6     308K   10M ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0

Chain FORWARD (policy DROP 0 packets, 0 bytes)
…

Chain OUTPUT (policy ACCEPT 1907K packets, 309M bytes)
…
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `Chain INPUT (policy DROP …)` | Par défaut, le trafic **entrant** est **refusé**. Les compteurs `9493 packets` = déjà du bruit / scans droppés. |
| `num` | Numéro de règle (utile pour `-D INPUT N`). |
| `pkts` / `bytes` | Compteurs : combien cette règle a matché. |
| Règle 1 `ACCEPT … lo` | Tout le **localhost** passe (backends `127.0.0.1` OK en local). |
| Règle 2 `RELATED,ESTABLISHED` | Réponses aux connexions déjà ouvertes (ex. retour HTTPS). |
| Règle 3 `dpt:64483 … NEW` | Nouvelles connexions TCP vers **64483** (SSH / admin non standard). |
| Règle 4 `dpt:80 … NEW` | HTTP public autorisé. |
| Règle 5 `dpt:443 … NEW` | HTTPS public autorisé (plus de paquets que 80 ici). |
| Règle 6 `ACCEPT icmp` | Ping autorisé → cohérent avec l’étape routage. |
| `FORWARD policy DROP` | Pas de routage/forward de paquets entre interfaces (VPS simple). |
| `OUTPUT policy ACCEPT` | Sortie libre depuis le VPS. |

#### Commande (persistance)

```bash
sudo cat /etc/iptables/rules.v4
```

#### Réponse

```
*filter
:INPUT DROP …
:FORWARD DROP …
:OUTPUT ACCEPT …
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp … --dport 64483 … NEW -j ACCEPT
-A INPUT -p tcp … --dport 80 … NEW -j ACCEPT
-A INPUT -p tcp … --dport 443 … NEW -j ACCEPT
-A INPUT -p icmp -j ACCEPT
COMMIT
```

#### Analyse

| Ligne / fragment | Signification |
| --- | --- |
| `*filter` | Table filtre (filtrage classique). |
| `:INPUT DROP` | Même politique que ci-dessus, forme sauvegardée. |
| `-A INPUT …` | Append des règles dans le même ordre logique. |
| `COMMIT` | Fin du bloc à charger. |
| Horodatage `Jul 12 2026` | Date de génération du fichier — règles persistantes au reboot (si service associé actif). |

---

## TP — bloquer le port 443

#### Commande

```bash
sudo iptables -I INPUT 1 -p tcp --dport 443 -j DROP
```

#### Analyse (effet attendu — pas encore de capture « après »)

| Fragment | Signification |
| --- | --- |
| `-I INPUT 1` | **Insert** en position 1 (avant les ACCEPT 443). |
| `-p tcp --dport 443` | Cible HTTPS. |
| `-j DROP` | Silence / hang côté client (vs `REJECT` = refus explicite). |

Conséquence : `curl https://…` depuis le PC échoue ; `ping` peut rester OK (règle ICMP inchangée) ; SSH sur 64483 reste ouvert.
