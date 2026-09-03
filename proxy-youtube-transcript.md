Intro
Why is Nginx called a reverse proxy?
What is a proxy anyways?
Let’s take a look.
Forward Proxy
Two common types of proxy are forward proxy and reverse proxy.
A forward proxy is a server that sits between a group of client machines and the internet.
When those clients make requests to websites on the internet, the forward proxy acts as a  
middleman intercepts those requests and talks to web servers on behalf of those client machines.
Why would anyone want to do that?
Here are a few common reasons.
One, a forward proxy protects the client’s online identity.
By using a forward proxy to connect to a website,  
the IP address of the client is hidden from the server.
Only the IP address of the proxy is visible.
It would be harder to trace back to the client.
Two, a forward proxy can be used to bypass browsing restrictions.
Some institutions like governments, schools,  
and big businesses use firewalls to restrict access to the internet.
By connecting to a forward proxy outside the firewalls,  
the client machine can potentially get around these restrictions.
It does not always work because the firewalls  
themselves could block the connections to the proxy.
Three, a forward proxy can be used to block access to certain content.
This is not uncommon for schools and businesses to configure their networks to connect all clients  
to the web through a proxy and apply filtering rules to disallow sites like social networks.
It is worth noting that a forward proxy normally  
requires a client to configure its application to point to it.
For large institutions, they usually apply  
Transparent Proxy
a technique called transparent proxy to streamline the process.
A transparent proxy works with layer 4 switches to redirect certain types  
of traffic to the proxy automatically.
There is no need to configure the client machines to use it.
It is difficult to bypass a transparent proxy when the client is on the institution's network.
In summary, a forward proxy sits between the client and  
Reverse Proxy
the internet and acts on behalf of the client.
Now, let’s take a look at reverse proxy.
A reverse proxy sits between the internet and the web servers.
It intercepts the requests from clients and talks to the web server on behalf of the clients.
Why would a website use a reverse proxy?
Here are a few good reasons.
One, a reverse proxy could be used to protect a website.
The website’s IP addresses are hidden behind the reverse proxy and are not revealed to the clients.
This makes it much harder to target a DDoS attack against a website.
Second, a reverse proxy is used for load balancing.
A popular website handling millions of users every day is unlikely to be  
able to handle the traffic with a single server.
A reverse proxy can balance a large amount of  
incoming requests by distributing the traffic to a large pool of web servers,  
and effectively preventing any single one of them from becoming overloaded.
Note that this assumes that the reverse proxy can handle the incoming traffic.
Services like Cloudflare put reverse proxy servers in hundreds of locations all around the world.
This puts the reverse proxy close to the users and at the same time  
provides a large amount of processing capacity.
Third, a reverse proxy caches static content.
A piece of content could be cached on the reverse proxy for a period of time.
If the same piece of content is requested again from the reverse proxy,  
the locally cached version could be quickly returned.
Fourth, a reverse proxy can handle SSL encryption.
SSL handshake is computationally expensive.
A reverse proxy can free up the origin servers from these expensive operations.  
Instead of handling SSL for all clients,  
a website only needs to handle SSL handshake from a small number of reverse proxies.
Layers of Reverse Proxy
Reverse proxies are everywhere. For a modern website, it is not  
uncommon to have many layers of reverse proxy.
The first layer could be an edge service like Cloudflare.
The reverse proxies are deployed to hundreds of locations worldwide close to the users.
The second layer could be an API gateway or load balancer at the hosting provider.
Many cloud providers combine these two layers into a single ingress service.
The user would enter the cloud network at the edge close to the user,  
and from the edge, the reverse proxy connects over a fast fiber network to  
the load balancer where the request is evenly distributed over a cluster of web servers.
If you like our videos, you may like our weekly system design newsletter as well.
It covers topics and trends in large-scale system designs. Trusted by 150,000+ readers.
Subscribe at: blog.bytebytego.com
