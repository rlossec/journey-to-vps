← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-a-aaaa)

# Records A et AAAA (`readresolve.tech`)

**But :** IPv4 publiée, et s’il existe une IPv6. Ici : A oui, AAAA non.

```powershell
Resolve-DnsName -Name "readresolve.tech" -Type A
Resolve-DnsName -Name "readresolve.tech" -Type AAAA
```

## Sortie attendue — A

```
Name                 Type  TTL   Section  IPAddress
----                 ----  ---   -------  ---------
readresolve.tech     A     2863  Answer   54.36.100.9
```

## Sortie attendue — AAAA

```
Name              Type TTL Section   PrimaryServer    NameAdministrator
----              ---- --- -------   -------------    -----------------
readresolve.tech  SOA  300 Authority dns13.ovh.net    tech.ovh.net
```

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `A` / `54.36.100.9` | Apex → IP du VPS. |
| `TTL` inférieur à 3600 | Reste de cache, pas un TTL de zone à 2863 s. |
| `SOA` en Authority (AAAA) | **Pas d’IPv6** : le nom existe, ce type n’a pas d’enregistrement. |
| ≠ nom introuvable | Ici le domaine existe, sans record AAAA. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-2-a-aaaa)
