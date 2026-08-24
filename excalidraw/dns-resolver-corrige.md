# Schéma corrigé — résolution DNS

Fichier Excalidraw : `dns-resolver-corrige.json`  
Ancien schéma (conservé) : `dns-resolver.json`

Même contenu, lisible ici sans ouvrir Excalidraw.

```mermaid
flowchart LR
  subgraph client["Chez le client"]
    C["Cache navigateur / OS<br/>vide si ordi neuf"]
    N["Navigateur / curl<br/>https://readresolve.tech"]
    C --> N
  end

  subgraph reso["Résolveur récursif"]
    CR["Cache du résolveur<br/>vide au 1er passage"]
    R["FAI / 8.8.8.8 / 1.1.1.1"]
    CR --> R
  end

  subgraph dns["Le résolveur enchaîne — il n’a pas l’IP tout de suite"]
    R1["1. Racine A–M<br/>Q : qui gère .tech ?<br/>R : ns01.trs-dns.com …<br/>≠ IP du site"]
    R2["2. TLD .tech<br/>Q : qui est autoritaire<br/>pour readresolve.tech ?<br/>R : dns13.ovh.net<br/>≠ IP du site"]
    R3["3. DNS autoritaire<br/>dns13.ovh.net<br/>Q : A de readresolve.tech ?<br/>R : 54.36.100.9<br/>cette machine ≠ le VPS"]
  end

  VPS["VPS 54.36.100.9 — autre machine<br/>étape 3 routage, pas le DNS"]

  N -->|"readresolve.tech → IP ?"| R
  R -->|"1"| R1
  R -->|"2"| R2
  R -->|"3"| R3
  R -.->|"54.36.100.9"| N
  R3 -.- VPS
```

**Une idée par étage :** 1 et 2 délèguent (noms de serveurs). Seul 3 donne l’IP. Le cache est sur le client et le résolveur, pas sur chaque racine.
