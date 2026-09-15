# Intro

## D'un nom de domaine à une page web

Lorsque nous saisissons une URL dans notre navigateur, une page web apparaît généralement en quelques instants.

Mais que s'est-il passé entre le moment où nous avons tapé cette adresse et celui où le serveur nous a renvoyé la page ?

Derrière cette action très simple se cache une succession de systèmes, de protocoles et de configurations. 
L'objectif est de comprendre ce voyage, de la saisie de l'URL jusqu'à l'affichage de la page web.

## Le cas d'étude

Pour rendre cette exploration concrète, nous allons nous appuyer sur le VPS mis à disposition dans le cadre de la formation.

Nous allons suivre une requête réelle vers un site que nous hébergeons :

`https://mbr-raphael.readresolve.tech`

Vous pouvez remplacer raphael par votre username sur le VPS.

À chaque étape, nous alternerons entre :

- **Comprendre** les notions théoriques nécessaires.
- **Observer** ce qui se passe réellement sur Internet ou sur notre infrastructure.
- **Expérimenter** avec des outils comme `curl`, `dig`, `ss` ou les commandes disponibles sur notre VPS.

L'objectif n'est pas seulement de savoir qu'un site fonctionne, mais de comprendre pourquoi il fonctionne et où intervenir lorsqu'il ne fonctionne plus.

## Explications globales des étapes

Schéma Architecture, avec grands domaines /responsabilités : [0-introduction](../excalidraw/0-introduction.excalidraw)

- Internet
- OVH
- Notre VPS (Chez OVH)

On va découper ce voyage en 5 étapes :

1. **Résolution DNS** — Comment l’url `readresolve.tech` devient une IP.
2. **Enregistrement DNS** — comment on a déclaré ce lien entre `readresolve.tech` et l’ip
3. **Routage** — comment les données trouvent le chemin jusqu'au VPS
4. **Infrastructure OVH** - de même le chemin mais aussi les sécurités dans OVH
5. **Configuration du VPS : Ports et Apache** : enfin l'arrivée dans le VPS et la configuration que l'on gère



## Questions directrices

- [x] Quels systèmes interviennent **avant** que le VPS reçoive la requête ?
- [x] Qu’est-ce qui appartient à **OVH** ?
- [x] Qu’est-ce qui est **sous notre contrôle** ?