← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-ss)

# `sudo ss -tlnp`

**But :** qui **écoute**, et sur quelle interface (public vs localhost).

```bash
sudo ss -tlnp
```

## Sortie attendue (VPS)

```
State   Recv-Q Send-Q  Local Address:Port   Process
LISTEN  0      4096          0.0.0.0:64483  …
LISTEN  0      511         127.0.0.1:9030   …
LISTEN  0      511         127.0.0.1:9050   …
… (autres 127.0.0.1:90xx)
LISTEN  0      511                 *:80     …
LISTEN  0      511                 *:443    …
```

Les ports `90xx` exacts peuvent varier ; le motif `127.0.0.1` vs `*` compte.

## Lignes importantes

| Fragment | Lecture |
| --- | --- |
| `*:80` / `*:443` | HTTP/HTTPS **publics** = frontend Apache. |
| `0.0.0.0:64483` | Admin / SSH, toutes IPv4. |
| `127.0.0.1:90xx` | **Backends** localhost — invisibles depuis Internet. |
| Pas de `:22` en LISTEN | SSH n’est pas sur le well-known 22. |

← [Retour à la feuille TP](../../tp-sheet.md#cmd-5-ss)
