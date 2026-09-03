# Why is Nginx called a reverse proxy?

What is a proxy anyway? Let’s take a look.

Two common types of proxy are **forward proxy** and **reverse proxy**.

## Forward proxy

A forward proxy is a server that sits between a group of client machines and the internet. When those clients make requests to websites on the internet, the forward proxy acts as a middleman: it intercepts those requests and talks to web servers on behalf of those client machines.

### Why would anyone want to do that?

#### 1. Protect the client’s online identity

By using a forward proxy to connect to a website, the IP address of the client is hidden from the server. Only the IP address of the proxy is visible. It is harder to trace back to the client.

#### 2. Bypass browsing restrictions

Some institutions (governments, schools, big businesses) use firewalls to restrict access to the internet. By connecting to a forward proxy outside the firewalls, the client machine can potentially get around these restrictions.

It does not always work: the firewalls themselves could block connections to the proxy.

#### 3. Block access to certain content

Schools and businesses often configure their networks so that all clients connect to the web through a proxy, with filtering rules that disallow sites like social networks.

A forward proxy normally requires a client to configure its application to point to it. Large institutions usually apply a technique called **transparent proxy** to streamline the process.

## Transparent proxy

A transparent proxy works with layer 4 switches to redirect certain types of traffic to the proxy automatically. There is no need to configure the client machines to use it.

It is difficult to bypass a transparent proxy when the client is on the institution’s network.

**In summary:** a forward proxy sits between the client and the internet and acts on behalf of the client.

---

## Reverse proxy

A reverse proxy sits between the internet and the web servers. It intercepts the requests from clients and talks to the web server on behalf of the clients.

### Why would a website use a reverse proxy?

#### 1. Protect a website

The website’s IP addresses are hidden behind the reverse proxy and are not revealed to the clients. This makes it much harder to target a DDoS attack against a website.

#### 2. Load balancing

A popular website handling millions of users every day is unlikely to handle the traffic with a single server. A reverse proxy can balance a large amount of incoming requests by distributing the traffic to a large pool of web servers, and effectively prevent any single one of them from becoming overloaded.

This assumes that the reverse proxy can handle the incoming traffic. Services like Cloudflare put reverse proxy servers in hundreds of locations all around the world. This puts the reverse proxy close to the users and at the same time provides a large amount of processing capacity.

#### 3. Cache static content

A piece of content can be cached on the reverse proxy for a period of time. If the same piece of content is requested again from the reverse proxy, the locally cached version can be quickly returned.

#### 4. Handle SSL encryption

SSL handshake is computationally expensive. A reverse proxy can free up the origin servers from these expensive operations. Instead of handling SSL for all clients, a website only needs to handle SSL handshake from a small number of reverse proxies.

---

## Layers of reverse proxy

Reverse proxies are everywhere. For a modern website, it is not uncommon to have many layers of reverse proxy.

1. **First layer:** an edge service like Cloudflare. The reverse proxies are deployed to hundreds of locations worldwide, close to the users.
2. **Second layer:** an API gateway or load balancer at the hosting provider.

Many cloud providers combine these two layers into a single ingress service. The user enters the cloud network at the edge, close to the user; from the edge, the reverse proxy connects over a fast fiber network to the load balancer, where the request is evenly distributed over a cluster of web servers.
