← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-mx)

# Records MX (`readresolve.tech`)

**But :** qui reçoit le mail du domaine (hors fil « site web », mais présent dans la zone).

```bash
dig MX readresolve.tech
```

## Sortie attendue

```
;; ANSWER SECTION:
readresolve.tech.       3600    IN      MX      1 mx4.mail.ovh.net.
readresolve.tech.       3600    IN      MX      10 mx3.mail.ovh.net.
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `mx4` priorité **1** | Serveur mail **primaire** OVH. |
| `mx3` priorité **10** | Secondaire (plus le chiffre est petit, plus c’est prioritaire). |
| Pas l’IP du VPS | Le mail n’est pas forcément sur la même machine que le site. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-mx)
