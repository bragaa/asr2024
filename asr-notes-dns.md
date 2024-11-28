# Why DNS?
- Using names instead of numeric addresses to designate network devices in configuration files and for humans makes human easily manageable and rememberable. It also introduces a *level of indirection* (i.e. a server's IP address can be changed without impacting client computers).
- The TCP/IP stack protocols refer to network devices with their IP addresses. To introduce names, techniques were used to map names to addresses (forward resolution) and vice versa (reverse name resolution).
  - One old technique is to resolve names from *local* files: ```/etc/hosts``` for unix-like systems or ```%systemroot%/system32/drivers/etc/hosts``` for Windows.
    ```
    127.0.0.1       localhost
    192.168.32.128  dev

    # The following lines are desirable for IPv6 capable hosts
    ::1     ip6-localhost ip6-loopback
    fe00::0 ip6-localnet
    ff00::0 ip6-mcastprefix
    ff02::1 ip6-allnodes
    ```
    This works like this: in a managed network, new mapping, updates to existing mappings, etc. are sent to the network admin, which updates the file and then send back (using FTP or mail) the file to all the network devices. The solution was in ARPanet, and is still used in computers to for small configs. However, the solution will not scale up: centralized load and single point of failure.
  - An alternative is the **Domain Name System** (DNS). The DNS allows a computer to automatically find the resource record (RR) of a given type and associated to a given name from different DNS servers distributed across the Internet.
- The **Domain Name System** is a *distributed* and *automated* solution, making it more *scalable*. It has the following features:
  - hierarchical name space, allowing to name domains and divide them into subdomains, up to 128 levels,
  - ability to store *resources records* of various types (A, AAAA, etc.) for names; a common resource record type is the IPv4 address (A)
  - delegation scheme to make the resource records distributed among multiple DNS servers
  - protocol allowing clients to query servers (for resolutions) and servers to cooperate with each others.

# DNS name space organization
- DNS defines hiearchical name space, and allows independant entities, e.g. organizations, companies, ISP, universities, governemental offices, individuals, etc. to have their own DNS domain, where they can create names and resource records for them.
- Here is an excerpt of the DNS name space: 
  ![Organisation de l'espace de noms DNS](../imgs/dns-namespace.svg)
- A hierarchical name space has the advantage of allowing organizations create and manage their own domain names, independently from a central authority, while avoiding name collisions. The DNS name space is made up of the following levels:
  - the root DNS name space (or domain) is managed by the Internet corporation for assigned names and numbers (ICANN).
  - the ICANN partitions the root domain into standard subdomains called the Top level domains (TLD), and delegates their administration to relevant organizations called **registries**. These TLDs are created for countries (country-code TLDs, ccTLD, like `tn.`, `uk.`, `fr.` etc.) or based on activity (generic TLDs gTLDs, like `.com`, `.org`, `tv.`). gTLDs are managed by companies designated by the ICANN, while ccTLDs are managed relevant governemental offices in the relevant country (ATI in tunisia).
  - finally, second level domains are created by having an organization obtaining a unique domain name like `entreprise.tn.` from the relevant registry. This organisation is then able to create other names in this domain, associate resources to these names, etc.
- DNS allows up to 128 levels in the hierarchy.
- Notation: to avoid confusion, you have to type the entire domain name, up to the root domain (it is thus called *fully qualified domin name*. For instance, `serveur.entreprise.com.` is an FQDN while `serveur.entreprise.com` leads to confusion,: is it `serveur.entreprise.com.tn.` or `serveur.entreprise.com.`
- DNS domain names are not free, may be revoked, reserved, etc.

# Ressource records
- DNS stores information associated to names (called ressrouces, like addresses, mail server, etc.) in what it calls **Ressource records**, RR.
- RR format: `name [class] [type] [ttl] [rdata]`. For example, `mailServer.mydomain.uk.  IN  A 12.168.74.3` states that *the IPv4 address* (A) of `mailServer.mydomain.uk.` is `12.168.74.3`. The RR class defaults to Internet (`IN`) if omitted.
- The `rdata` (record data) semantics and syntax depends on the resource `type` (check those of SRV, SoA.)
- Common RR types are: A, AAAA, CNAME (Canonical name), NS (Name service), SOA (Start of Authority), MX (Mail exchange), SRV, PTR (for inverse resolution), DNSKEY, DS (DigitalSigner)
- A DNS server stores RRs in ascii-encoded files called **zone files**. Here is a commented example to illustrate common RR types and their meanings; one RR per line. More on zone later. Comments starts with semi-colon.

```
; this is file-wide variable definition
$TTL 604800
; a Start of Authority RR, serving to introduce domain metadata. It must be the first RR in a zone file and occurs only once. The RR reads as follows: authoritative server for domain entreprise.tn. is serveur.entreprise.tn., is managed by admin@entreprise.tn. and have replication information as described between parenthesis.
entreprise.tn.    SOA serveur.entreprise.tn.  admin.entreprise.tn. (
    2   ; serial number (version number of the zone file)
    604800   ; zone refresh interval
    86400 ; retry interval
    2419200  ; expire in
    1)       ; negative cache TTL

; domain authoritative name server(s) (many servers can be mentioned) and IPv4 address
entreprise.tn.      NS  serveur.entreprise.tn.
serveur.entreprise.tn.  A  193.95.1.1

; name server(s) authoritative on factory.entreprise.tn. and IPv4 address. this RR stands for the deleguation.
factory.entreprise.tn.      NS  serveur1.entreprise.tn.
serveur1.entreprise.tn.  A  193.95.1.11

; IPv4 address of serveur.entreprise.tn. is 193.95.1.2
serveur.entreprise.tn.  A 193.95.1.2

; The same, but with IPv6 addresses (Do you see why 4 'A' are used?)
serveur6.entreprise.tn.  AAAA  de::12:2e:de:22:22:23:60

; www is a Canonical name for serveur.entreprise.tn., simpler, straighthrough name
www    CNAME      serveur.entreprise.tn.

; Mail eXchange server hosting mailboxes for mail in domain entreprise.tn (foulen@entreprise.tn) is srv.ovh.fr.
entreprise.tn.   MX  srv.ovh.fr.

; LDAP service over TCP is hosted on serveur.entreprise.tn. and reachable on port 389.
_ldap._tcp.entreprise.tn.  SRV 1 90 389 serveur.entreprise.tn.      
```

# Zones and Delegation
- It is obvious that RRs of all the names of the root domain name will not be stored on the same server! Each domain owner (like the ATI for the `tn.` domain) manages its own DNS servers which host the RR associated to names in its domain and thus is the only entity able to respond to queries related to them. Furthermore, when a domain owner delegates a subdomains say, entreprise.tn. to a different entity (the `entreprise` company), it has to create delegation information in the form of RR of type NS. It is common to say that a server has authority over a name domain, when RR for names of this domain are stored within the server.
- Delegation marks the start of a new zone. It is defined by an RR of type NS telling the delegated subdomain, and the authoritative server where zone file can be fetched.
- It is up to an authority to decide whether to delegate or not. Typically, they will delegate upon paying a fee for the subdomain. In other scenarios, delegation were not envisioned (is this the case for `.com.tn. `and `.gouv.tn.`?)
- A **DNS zone** is the set of names of a domain hosted on the same DNS server. In different words, zones are partitions of the domain namespace and associated RRs. A zone file is an ascii-encoded file containing the corresponding RRs. The following figure illustrates the zone and authoritative servers concepts.
![DNS Zones](../imgs/dns-system.png)
- It is common for many zones to be hosted on the same server, typical for DNS registries, and this makes out the business model of DNS registries and the Managed DNS services
- Following is an excerpt of the root zone file, showing the delegations made by the ICANN to some registries on some TLDs
```
. IN  SOA A.ROOT-SERVERS.NET. NSTLD.VERISIGN-GRS.COM. (
     ; truncated
      )
; com and net zones are hosted in server managed by Network Solutions, LLC. (Information obtained from the WHOIS database)
com.  IN  NS  A.GTLD-SERVERS.NET.
net.  IN  NS  A.GTLD-SERVERS.NET.
A.GTLD-SERVERS.NET. IN  A 192.5.6.30
; the tn zone is hosted in the following servers
tn.   IN  NS  ns2.nic.fr.
ns2.nic.fr. IN  A 192.93.0.4
tn.   IN  NS  ns.eu.net.
ns.eu.net.  IN  A 192.16.202.11
tn.   IN  NS  NS.ATI.TN.
ns.ati.tn. IN A 193.95.66.10
```
- Note: The ATI, registrar for the tn. zone typically outsource the commcercial operations of subscription, update of domain information, to ISPs, and other entities without introducing any delegation. The workflow for these sub-registrars will always involve quering and interaction with the ATI servers (to chack name uniqueness, update informations ,etc.)

- Each registrant (like entreprise) typically maintains DNS information regarding its domain content (names, association information, delegation, etc.) and makes it queriable by deploying it on one or more servers. It is the unique entity responsible for responding to queries regarding ressources in its namespace (not considering caching or malicious activities).

# The DNS protocol
- The protocol defines two operations: name resolution and zone transfer.
- **Name resolution** is offered on port udp/53 and is meant to ask the server to find the RR of a specific type for a given DNS name. 
- An secondary server will send **Zone transfer** requests to the primary server to according to the timing parameters to ask for updated zone file (if changes were happened on the zone file, detecte using the serial number). Of course, we are considering servers authoritative on the same zone!

# Name resolution process
- What the web browser typically does before opening a web page `http://www.somesite.tn/index.html` is to query the pre-configured DNS server for the IPv4 address of `www.somesite.tn` by using the services of system component (shared library) called resolver, offering the DNS client and client feature, commonly called the **resolver**; it will:
  - find the name server to contact from the system configuration
  - send a query to the server asking for the RR of type A associated with name `www.somesite.tn`
- Client makes either iterative or recursive queries, depending on the situation; servers may accept both or iterative only.
- In an **iterative** query, the DNS server will respond with information by using its own zones files only or its cache, it will throw an error if a name is invalid (not in zone files) or the if it cannot find the name it its cache.
- A **recursive** query is more complicated in that the contacted server has to find information by issuing (iterative) DNS queries to other servers. The client states that recursion is desired (set the flag `rd`) in the query.
  - The server accepting recursion will typically start search by following the domain name components from the root server down to the authoritative server authoritative on `somesite.tn.` domain having the A RR associated with `www.somesite.tn.`.
  - If a server accepting recursive queries do nto return information, this means that the name is invalid (not assigned).
- Servers accepting recursive queries are called resolvers; the system client is a resolver able to make iterative queries until finding the name. (note however that it still starts by making recursive queries to a local newtork-wide resolver).
- In general, OS-based clients sends recursive requests to the network access provider-provided DNS server and is combined with caching for efficiency
- ISPs (and any managed network in general) will offer their customers (and restrict them to using it) a recursive server, called resolver, and implementing *caching* and *forwarding* roles. This is done for bandwidth efficiency and to enhance user experience.
- Here is an illustration of what happens when you want to make `ping www.hp.uk.`. This example did not take into account the forwarder's cache which will make the things happens differently and optimized. Just think about how the forwarder would act if it already know the `uk.` authoritative server from its cache.
![DNS name resolution example](../imgs/dns-qurey.png)
  
# DNS Fault tolerance
- DNS is fault tolerant on various levels:
  - the zone file is held by different servers, (primary and multiple secondaries) synchronizing using zone transfer/replication protocol usint the tcp/53 and whole settings are define in the SoA RR.
  - the same server can be geographically distributed (ditinct instances) thanks to BGP anycast routing. We end with different servers having the same IP.
  
# Zones replication
- A DNS server can be a master or slave for a given zone.
  - They use AXFR, IXFR and NOTIFY protocols for zone data synchronization (i.e. replication)
  - They may use other mechanisms (LDAP-based as in AD, replication of relational databases)

# DNS Server roles
- Authoritative server can be either master or slave *for a given zone*. **Master** servers holds the master zone file copy, disseminating zone updates to the **secondary** ones using the AXFR, IXFR and NOTIFY operations.
- A non authoritative server can be configured:
  - to forward requests,
  - to accept recursion,
  - to cache results carried by requests,
  - with a split-horizon
  - with other non-standard capabilities

# Inverse resolution
- With DNS, the search criteria is always the name. In some situations we need find the name associated with an address; this is called the **inverse resolution** (as opposed to the **forward resolution**).
-  Inverse resolution is sometimes desired, e.g. to make human friendly output in various situations, like showing the routing table. Just try the `route` and `route -n` commands.
- The DNS mappings of names to addresses are organized based on hiearchizing name space, making it non suitable to inverse resolution, unless you are willing to search all the DNS zones!
- To make inverse resolutions happens using existing DNS infrastructure (in a deterministic way), a special subdomain is allocated by the ICANN, called `in-addr.arpa.` and holding information obtained from the IP address allocation scheme as done by the RIR hierarchy. For instance, the name 4.3.2.1.in-addr.arpa. is semantically equivalent to address 1.2.3.4
- Authority on the various domains is dictated by the IP addresses allocations from RIR.
![Inverse name resolution hierarchy](../imgs/dns-inverse-namespace.png)
- Here is an example of an inverse zone file.
```
; ORIGIN will be the default name 
$ORIGIN 1.0.10.in-addr.arpa.
@ IN  SOA serv1.mydomain.uk.  jl.mydomain.uk.(
  2005112501 ; serial number
  21600 ;updates in 6h
  3600 ;attempt every 1h
  604800 ;expire after 7 days
  86400  ;default TimeToLive of the RRs in this zone file
  )
    
1 IN  PTR serv1.mydomain.uk.
2 IN  PTR serv2.mydomain.uk.
10  IN  PTR mail.mydomain.uk.
; ...
100 IN  PTR asterix.mydomain.uk.
```

# RR Caching

# Misc
- A root zone hint file is distributed via OS or DNS Servers software
- the client (aka resolver) is implemented as shared system-wide library
- LLMNR (RFC4795) and mDNS (RFC6762) can be used to perform link local resolutions (A, SRV, PTR, etc.)


# DNS Query examples
- RR information query on a specific DNS server
```me@mymachine ~ $ dig @8.8.8.8 www.ati.tn A IN

  ; <<>> DiG 9.9.5-3ubuntu0.15-Ubuntu <<>> @8.8.8.8 www.ati.tn A IN
  ; (1 server found)
  ;; global options: +cmd
  ;; Got answer:
  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45303
  ;; flags: qr rd ra ad; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

  ;; OPT PSEUDOSECTION:
  ; EDNS: version: 0, flags:; udp: 512
  ;; QUESTION SECTION:
  ;www.ati.tn.			IN	A

  ;; ANSWER SECTION:
  www.ati.tn.		5958	IN	CNAME	ati.tn.
  ati.tn.			5958	IN	A	193.95.68.142

  ;; Query time: 91 msec
  ;; SERVER: 8.8.8.8#53(8.8.8.8)
  ;; WHEN: Tue Sep 26 10:30:03 CET 2017
  ;; MSG SIZE  rcvd: 69
```

- Tracing DNS Query
  ```me@mymachine ~ $ dig  sontaya.dusit.ac.th  +trace +nodnssec

  ; <<>> DiG 9.9.5-3ubuntu0.15-Ubuntu <<>> sontaya.dusit.ac.th +trace +nodnssec
  ;; global options: +cmd
  .			18267	IN	NS	f.root-servers.net.
      ....truncated....
  .			18267	IN	NS	a.root-servers.net.
  ;; Received 811 bytes from 127.0.1.1#53(127.0.1.1) in 96 ms

  th.			172800	IN	NS	b.thains.co.th.
  th.			172800	IN	NS	a.thains.co.th.
  th.			172800	IN	NS	sfba.sns-pb.isc.org.
  th.			172800	IN	NS	ams.sns-pb.isc.org.
  th.			172800	IN	NS	c.thains.co.th.
  th.			172800	IN	NS	ns.thnic.net.
  ;; Received 419 bytes from 202.12.27.33#53(m.root-servers.net) in 2284 ms

  dusit.ac.th.		7200	IN	NS	ns2.dusit.ac.th.
      ... truncated ...
  dusit.ac.th.		7200	IN	NS	ns4.dusit.ac.th.
  ;; Received 267 bytes from 202.28.0.1#53(ns.thnic.net) in 2225 ms

  sontaya.dusit.ac.th.	3600	IN	CNAME	dusithost.dusit.ac.th.
  dusithost.dusit.ac.th.	3600	IN	A	202.29.83.152
  dusit.ac.th.		3600	IN	NS	suphan-ns.dusit.ac.th.
      ... truncated ...
  dusit.ac.th.		3600	IN	NS	suphan-ns2.dusit.ac.th.
  ;; Received 457 bytes from 202.29.82.151#53(ns.dusit.ac.th) in 616 ms
```

# References
- [http://www.zytrax.com/books/dns/]
- [https://whois.icann.org/en/primer] (a whois primer)