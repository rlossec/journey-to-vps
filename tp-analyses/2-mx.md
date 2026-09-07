← [Retour à la feuille TP](../tp-sheet.md#cmd-2-mx)

# Records MX (`readresolve.tech`)

**But :** qui reçoit le mail du domaine (hors fil « site web », mais présent dans la zone).

## Commandes

**Linux**

```bash
dig MX readresolve.tech
```

**Windows**

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type MX
```

## Sortie attendue (Linux)

```
;; ANSWER SECTION:
readresolve.tech.       3600    IN      MX      1 mx4.mail.ovh.net.
readresolve.tech.       3600    IN      MX      10 mx3.mail.ovh.net.
```

## Sortie attendue (Windows)

```
Name               Type TTL  Section NameExchange       Preference
----               ---- ---  ------- ------------       ----------
readresolve.tech   MX   3600 Answer  mx4.mail.ovh.net   1
readresolve.tech   MX   3600 Answer  mx3.mail.ovh.net   10
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `mx4` priorité **1** | Serveur mail **primaire** OVH. |
| `mx3` priorité **10** | Secondaire (plus le chiffre est petit, plus c’est prioritaire). |
| Pas l’IP du VPS | Le mail n’est pas forcément sur la même machine que le site. |

← [Retour à la feuille TP](../tp-sheet.md#cmd-2-mx)
