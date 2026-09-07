← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-txt)

# Record TXT (`readresolve.tech`)

**But :** lire un TXT typique de zone OVH — ici la politique **SPF**.

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type TXT
```

## Sortie attendue

```
Name               Type TTL Section Strings
----               ---- --- ------- -------
readresolve.tech   TXT  600 Answer  {v=spf1 include:mx.ovh.com ~all}
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `v=spf1` | Enregistrement **SPF** (qui a le droit d’envoyer du mail pour ce domaine). |
| `include:mx.ovh.com` | Délègue la liste aux MX OVH. |
| `~all` | Softfail pour le reste (pas un refus dur `-all`). |
| TTL `600` | Plus court que le A (10 min). |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-txt)
