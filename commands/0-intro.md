# Intro — observer le résultat avant d’ouvrir les boîtes

Objectif : constater que le site répond, **sans encore expliquer comment**.

## Navigateur

Ouvrir `https://readresolve.tech`.

Onglet Réseau des outils développeur : une requête vers le domaine, une réponse. On ne voit ni le DNS, ni le routage, ni les firewalls.

## Terminal

**Linux / Windows**

```bash
curl -I https://readresolve.tech
```

On obtient des en-têtes HTTP. Le reste du parcours (DNS → OVH → VPS → Apache) est invisible depuis cette seule commande : c’est précisément ce que les 5 étapes vont rendre visible.
