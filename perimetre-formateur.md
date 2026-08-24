# Case study: how a request reaches a VPS (OVH)

Design and conduct workshops for each section.

Select the most appropriate approach based on the topic:

- Local or VPS-based commands (`curl`, `whois`, `ss`, `ping`, `dig`, `host`, `traceroute`, `tracepath`, `mtr`, `nmap` etc). If using a local machine, remember to check for OS-specificity
- Browser developer tools
- Requests to the VPS

For your case study, you will need information available only via the OVH customer console or data requiring `root` access on the VPS. I plan to provide demonstrations and supply all the screenshots needed for your final report.

## 1. Introduction

**Topics** :

- Overview of the complete path
- Roles of the different actors
- What happens before the VPS receives a request?

Diagram :

```
Browser / curl
      │
      ▼
DNS resolver
      │
      ▼
Authoritative DNS
      │
      ▼
Internet routing
      │
      ▼
HCAP (Anti-DDoS)
      │
      ▼
Backbone router
      │
      ▼
Edge Network Firewall
      │
      ▼
Datacenter router
      │
      ▼
VPS firewall
      │
      ▼
Apache (frontend) - as reverse proxy
      │
      ▼
Apache (backend)
```

**Guiding questions**:

- What systems are involved before reaching the VPS?
- Which components belong to OVH?
- Which components are under your control?

## 2. Domain names and DNS

**Topics**:

- Domain registration
- Subdomains
- DNS zones
- Public IP addresses
- IP attribution to a VPS
- Common DNS records:
    - `A`
    - `AAAA`
    - `CNAME`
    - `MX` (briefly)
    - `TXT` (briefly)
- `TTL` - Time To Live

**Guiding questions**:

- Why does a domain need DNS records?
- Where does the IP address come from?
- Why does this IP identify my VPS?
- Can several domain names point to the same IP?
- What happens if the IP changes?
- Why doesn't a DNS change take effect immediately?

## 3. DNS propagation and resolution

**Topics**:

- Recursive resolvers
- Authoritative servers
- Caching
- Propagation

**Guiding questions**:

- What is DNS propagation?
- Why may two users obtain different answers ?
- How does caching improve performance ?

## 4. From DNS to the OVH Network

**Topics**:

- Internet routing
- BGP - Border Gateway Protocol (concept only)
- How traffic reaches OVH

**Guiding questions**:

- Once the IP is known, how is the server reached?
- Does the browser know the complete route?

## 5. Inside the OVH Infrastructure

**Topics**:

- HCAP (Anti-DDoS)
- Backbone routers
- Edge firewall
- Datacenter router
- Datacenter
- VPS

**Guiding questions**:

- Why doesn't traffic go directly to the VPS?
- Why are several protection layers used?
- Which layers are managed by OVH?

## 6. The VPS firewall

**Topics**:

- Purpose of a firewall
- Packet filtering
- Incoming vs outgoing traffic
- Default policies
- Rule ordering
- Introduction to `iptables`

**Guiding questions**:

- Why use a firewall?
- What happens if no rule matches?
- Why is rule order important?

## 7. Listening services and ports

**Topics**:

- Open ports
- Listening sockets
- Public vs local interfaces

**Guiding questions**:

- Which services are exposed ?
- Why isn't every service publicly accessible ?

## 8. Reverse proxy concept

**Topics**:

- Forward proxy vs reverse proxy (brief comparison)
- A reverse proxy receives client requests and forwards them to backend services
- The client communicates only with the reverse proxy
- The backend remains hidden

Diagram:

```
Internet clients
      │
      │ HTTP/HTTPS
      │
      ▼
Reverse Proxy - Apache (frontend)
      │
      │ HTTP
      │
      ▼
Apache (backend services)
```

**Guiding questions**:

- Why put a proxy in front of applications?
- Why not expose every backend directly?
- What responsibilities can the proxy handle?