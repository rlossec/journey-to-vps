**Commands Linux**

1. `curl`
2. `whois`
3. `ss`
4. `ping`
5. `dig`
6. `host`
7. `traceroute`
8. `tracepath`
9. `mtr`
10. `nmap`

**Commandes Windows**

1. `curl`
2. 404
3. 404
4. `ping`
5. 404
6. 404
7. `tracert`
8. 404
9. 404
10. 404

https://www.ionos.fr/digitalguide/serveur/outils/commandes-netstat/

## Linux / VPS

### resolvectl

`resolvectl status | head -20`

**Réponse** :

```
Global
         Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub

Link 2 (ens3)
    Current Scopes: DNS
         Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 213.186.33.99
       DNS Servers: 213.186.33.99
        DNS Domain: openstacklocal
     Default Route: yes
```
