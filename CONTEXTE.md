# Contexte du projet

Présentation pédagogique réalisée dans le cadre d’une formation **CDA** (Concepteur Développeur d’Applications).

Objectif : montrer, avec des **schémas** puis des **commandes**, le fil d’un cas d’étude réel : comment un site devient accessible sur Internet, depuis la mise en place du serveur jusqu’à l’arrivée d’une requête dans le navigateur.

## Cas d’étude

| Élément | Détail |
| --- | --- |
| Hébergement | VPS fourni par le formateur, chez **OVH** |
| Nom de domaine | **readresolve.tech** |
| Fil à suivre | créer / comprendre le serveur → lier le nom de domaine → y accéder |

Deux angles complémentaires, sur le **même** parcours :

1. **Accéder** à un site déjà en ligne (ce que voit le client).
2. **Héberger et configurer** son propre site (ce que fait l’administrateur).

## Périmètre formateur

Le cadrage d’origine est dans `perimetre-formateur.md` (8 sections). Notre plan en **5 étapes** le regroupe, il ne le remplace pas : chaque point du formateur doit rester visible dans une étape.

Avant les 5 étapes : une **intro courte** en deux temps — (1) les **acteurs** sans cas d’étude, (2) le **cas d’étude** et le **plan** des 5 étapes.

| Formateur | Notre plan |
| --- | --- |
| 1. Introduction | Intro (hors numérotation) |
| 2. Noms de domaine et DNS | 2. Enregistrement DNS |
| 3. Propagation et résolution DNS | 1. Résolution DNS |
| 4. Du DNS au réseau OVH | 3. Routage Internet |
| 5. Intérieur de l’infra OVH | 4. Infrastructure OVH |
| 6. Firewall du VPS | 5. Notre configuration — firewall VPS |
| 7. Services à l’écoute et ports | 5. Notre configuration — ports |
| 8. Reverse proxy | 5. Notre configuration — reverse proxy |

## Périmètre de cette présentation

Cinq grandes étapes, dans cet ordre (fil client : on tape l’URL, puis on explique comment c’est configuré) :

1. **Résolution DNS** — on soumet un nom dans le navigateur : comment trouve-t-on l’IP ? (résolveur récursif, autoritaires, cache, **propagation**).
2. **Enregistrement DNS** — d’où vient le nom : réservation, sous-domaines, zone DNS, IP publique du VPS, enregistrements (`A`, `AAAA`, `CNAME`, `MX` et `TXT` brièvement), TTL.
3. **Routage Internet** — une fois l’IP connue, comment les paquets trouvent le chemin jusqu’à OVH (**BGP** en concept seulement).
4. **Infrastructure OVH** — chemin **dans** OVH avant notre machine : HCAP / anti-DDoS, backbone, pare-feu de bordure, routeur de datacenter (le trafic n’arrive pas « directement » sur le VPS).
5. **Notre configuration** — firewall du VPS (`iptables` : filtrage, entrant/sortant, politique par défaut, ordre des règles), ports et services en écoute, **reverse proxy** Apache (frontend → backend). Comparaison brève proxy forward vs reverse.

Points du formateur à ne pas oublier **dans** ces étapes :

- Distinguer ce qui est **OVH** et ce qui est **sous notre contrôle**.
- Ateliers / TP (ex. couper le port 443) ; commandes locales **et** sur le VPS ; **outils développeur du navigateur**.
- Infos console OVH / `root` : captures fournies par le formateur.

## Format pédagogique

Chaque étape suit le même déroulement :

1. **Théorie** — explication courte, appuyée par des schémas.
2. **Expérimentation** — terminal et lignes de commande (Linux et, quand c’est utile, Windows), pour observer le comportement réel.

On reste concret : le domaine, l’IP du VPS et les commandes du cas d’étude sont le fil conducteur, pas un cours abstrait.

## Hors périmètre (volontaire)

Une **autre présentation** traitera les protocoles (**TCP**, **UDP**, **HTTP**, **HTTPS**).

Ici, on les **évoque de loin** uniquement quand c’est indispensable pour comprendre une étape (par exemple : un port 443, un reverse proxy HTTP). On n’explique pas le handshake, les en-têtes, ni la pile OSI en profondeur.

## Ce qui appartient à OVH vs ce que l’on maîtrise

| Côté OVH / Internet | Côté formation / VPS |
| --- | --- |
| Résolveurs, serveurs racine, TLD, serveurs autoritaires | Zone DNS du domaine `readresolve.tech` |
| Routage Internet (BGP) jusqu’au réseau OVH | Pare-feu du VPS (`iptables`) |
| HCAP, backbone, pare-feu de bordure, routeur DC | Services en écoute, Apache frontend / backend |

## Structure des fichiers

| Dossier / fichier | Rôle |
| --- | --- |
| `CONTEXTE.md` | Cadre de la présentation (ce document) |
| `perimetre-formateur.md` | Cadrage d’origine du formateur (8 sections) |
| `presentation_fr.md` | Fil de la présentation (brouillon de déroulé) |
| `explanations/` | Théorie par étape |
| `commands/` | Commandes d’atelier par étape (**sans** sorties) — par utilité, Linux / Windows |
| `commands/response-analysis/` | Mêmes étapes : **commandes + réponses** (WSL / VPS / Windows) + analyse ligne à ligne |
| `ressources/` | Liens, captures, références |
| `excalidraw/` | Schémas |

Intro (hors les 5 étapes) : `explanations/0-intro.md`, `commands/0-intro.md`.

Les étapes sont numérotées de `1` à `5` de la même façon partout :

1. `dns-lookup` — résolution DNS  
2. `dns-register` — enregistrement DNS  
3. `routage` — routage Internet  
4. `firewall` — firewall OVH + firewall VPS  
5. `apache-server` — Apache (ports + reverse proxy)

## Principes de rédaction

- Français, niveau formation CDA : précis, sans jargon inutile.
- Une idée par schéma ; le schéma précède les commandes.
- Les commandes illustrent la théorie, elles ne la remplacent pas.
- Distinguer clairement **cache** (navigateur, OS, FAI) et **requête DNS réelle**.
- Ne pas empiéter sur la présentation protocoles.
