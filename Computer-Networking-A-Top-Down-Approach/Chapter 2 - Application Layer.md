# Principles of Network Applications
In a _client-server architecture_, there is an always-on host, called the server, which services requests from many other hosts, called clients. A classic example is the Web application for which an always-on Web server services requests from browsers
running on client hosts. Another characteristic of the client-server architecture is that the server has fixed, well-known address, and because the server is always on, a client can always contact the server by sending a packet to the server's IP address.
In a _P2P architecture_, there is minimal (or no) reliance on dedicated servers in data centers. Instead the application exploits direct communication between pairs of intermittently connected hosts, called peers. The peers are not owned by the service provider, but are instead desktops and laptops controlled by users, with most of the peers residing in homes, universities, and offices. One of the most compelling features of P2P architectures is their self-scalability. For example, in a P2P file-sharing application, although each peer generates workload by requesting files, each peer also adds service capacity to the system by distributing files to other peers.  
## Transport Services Available to Applications
Many networks, including Internet, provide more than one transport-layer protocol. When you develop an application, you must choose one of the available transport-layer protocol. Possible services of transport-layer protocols:  


__Reliable Data Transfer__: For many applications -such as email, file transfer, remote host access- data loss can have devastating consequences. Thus, to support these applications, something has to be done to guarantee that the data sent by one end of the application is delivered correctly and completely to the other end of the application. If a protocol provides such a guaranteed data delivery service, it is said to provide reliable data transfer. When a transport-layer protocol doesn’t provide reliable data transfer, some of the data sent by the sending process may never arrive at the receiving process. This may be acceptable for loss-tolerant applications, most notably multimedia applications such as conversational audio/video that can tolerate some amount of data loss.  


__Throughput__: Another natural service that transport-layer protocol could provide, namely, guaranteed available throughput at some specified rate. With such a service, the application could request a guaranteed throughput of r bits/sec, and the transport protocol would then ensure that the available throughput is always at least r bits/sec. Applications that have throughput requirements are said to be bandwidth-sensitive applications. Many current multimedia applications are bandwidth sensitive. While bandwidth-sensitive applications have specific throughput requirements, elastic applications can make use of as much, or as little, throughput as happens to be available. Electronic mail, file transfer, and Web transfers are all elastic applications.  


__Timing__: An example guarantee might be that every bit that the sender pumps into the socket arrives at the receiver's socket no more than 100msec later. Such a service would be appealing to interactive real-time applications, such as Internet telephony, virtual environments, and multiplayer games, all of which require tight timing constraints on data delivery in order to be effective.  


__Security__: A transport protocol can provide an application with one or more security services. For example, in the sending host, a transport protocol can encrypt all data transmitted by the sending process, and in the receiving host, the transport-layer protocol can decrypt the data before delivering the data to the receiving process.  

The Internet makes two transport protocols available to applications, UDP and TCP. When you (as an application developer) create a new network application for the Internet, one of the first decision you have to make is whether to use UDP or TCP. TCP provides reliable end-to-end data transfer and we also know that TCP can be easily enhanced at the application later with SSL to provide security services. But throughput and timing services are not provided by today's Internet transport protocols. In summary, today’s Internet can often provide satisfactory service to time-sensitive applications, but it cannot provide any timing or throughput guarantees.
## Application Layer Protocols
An application-layer protocol defines how an application's processes, running on different end systems, pass messages to each other. In particular, an application-layer protocol defines:  

1. The types of messages exchanged, for example, request messages and response messages
2. The syntax of the various message types, such as the fields in the message and how the fields are delineated
3. The semantics of the fields, that is, the meaning of the information in the fields
4. Rules for determining when and how a process sends messages and respond to messages

# The Web and HTTP
The HyperText Transfer Protocol (HTTP), the Web's application-layer protocol, is at the heart of the Web. It is defined in [RFC 1945](https://tools.ietf.org/html/rfc1945) and [RFC 2616](https://www.ietf.org/rfc/rfc2616.txt). This is implemented in two programs: a client program and a server program. The client program and server program, executing on different end systems, talk to each other by exchanging HTTP messages. HTTP defines the structure of these messages and how the client and server exchange the messages. Because an HTTP server maintains no information about the clients, HTTP is said to be stateless protocol.  
## Non-Persistent and Persistent Connections
When client-server interaction is taking place over TCP, the application developer needs to make an important decision -should each request/response pair be sent over a separate TCP connection, or should all of the requests and their corresponding responses be sent over the same TCP connection? In the former approach, the application is said to use __non-persistent connections__; and in the latter approach, __persistent connections__.  


<h3>&nbsp;&nbsp;&nbsp;&nbsp;HTTP with Non-Persistent Connections</h3>
Let's walk through the steps of transferring a Web page from server to client for the case of non-persistent connections. Let's suppose the page contains of a base HTML file and 10 JPEG images reside on the same server. Here is what happens:  

1. Client initiates a TCP connection to the server
2. Client sends an HTTP request message to retrieve base HTML file
3. Server sends the response message to client
4. Server tells TCP to close the TCP connection. (But TCP doesn’t actually terminate the connection until it knows for sure that the client has received the response message intact.)
5. Client receives the response message. The TCP connection terminates.
6. First four steps are repeated for each of the referenced JPEG images. 

These steps illustrate the use of non-persistent connections, where each TCP connection is closed after the server sends the object. Thus, in this example, when a user requests the Web page, 11 TCP connections are generated. However, users can configure modern browsers to control the degree of parallelism, so some of the JPEG images are obtained over parallel TCP connections.  
We define __round-trip time (RTT)__, which is the time it takes for a small packet to travel from client to server and then back to client. Now consider what happens when a user clicks on a hyperlink. This causes the browser to initiate a TCP connection between the browser and the Web server; this involves a "three-way handshake" -the client sends a small TCP segment to the server, the server acknowledges and responds with a small TCP segment, and finally, the client acknowledges back to server. The first two parts of the three-way handshake take one RTT. After completing the first two parts of the handshake, the client sends the HTTP request message combined with the third part the three-way handshake (ACK) into the TCP connection. Once
the request message arrives at the server, the server sends the HTML file into the TCP connection. This HTTP request/response eats up another RTT. Thus, roughly, the total response time is two RTTs plus the transmission time at the server of the
HTML file.  

> No parallel connections: 2RTT + TransmissionTime<sub>base</sub> + (2RTT + TransmissionTime<sub>object</sub>) * n  
> Parallel connections: 2RTT + TransmissionTime<sub>base</sub> + 2RTT + TransmissionTime<sub>object</sub>


<h3>&nbsp;&nbsp;&nbsp;&nbsp;HTTP with Persistent Connections</h3>
With persistent connections, the server leaves the TCP connection open after sending a response. Subsequent requests and responses between the same client and server can be sent over the same connection. In particular, an entire Web page (in
the example above, the base HTML file and the 10 images) can be sent over a single persistent TCP connection. Moreover, multiple Web pages residing on the same server can be sent from the server to the same client over a single persistent TCP connection. These requests for objects can be made back-to-back, without waiting for replies to pending requests (pipelining). When the server receives the back-to-back requests, it sends the objects back-to-back. Typically, HTTP server closes a connection when it isn't used for a certain time (timeout interval).  

> Without pipelining: 2RTT + TransmissionTime<sub>base</sub> + (RTT + TransmissionTime<sub>object</sub>) * n  
> With pipelining: 2RTT + TransmissionTime<sub>base</sub> + RTT + (TransmissionTime<sub>object</sub>) * n  

## HTTP Message Format
There are two types of HTTP messages, request messages and response messages.  


__HTTP Request Message__: These messages can contain many lines or as few as one line. Each line is followed by a carriage return and line feed. The first line of an HTTP request message is called the request line. Request line has three fields: the method field, the URL field and the HTTP version field. The method field can take on several different values including GET, POST, HEAD, PUT, and DELETE. The URL field contains the requested object. The subsequent lines in the HTTP message is called header lines. The header line _Host:_ specifies the host on which the object resides. By including the _Connection: close_ header line, the browser is telling the server to close the connection after sending the requested object. The _User-agent:_ line specifies the browser type that is making the request to the server. The _Accept-language : en_ indicates that the user prefers to receive an English version of the object.  


__HTTP Response Message__: Response message has three sections: an initial status line, six header lines, and then the entity body. The entity body of the message -it contains the requested object itself. The status line has three fields: the protocol version, a status code, and a corresponding status message. The server uses the _Connection: close_ header line to tell the client that it is going to close the TCP connection after sending the message. The _Date:_ header line indicates the time and date when the HTTP response was created and sent by the server. The _Server:_ header line indicates that the message was generated by which server.  

## User-Server Interaction: Cookies
We mentioned that an HTTP server is stateless. However, it is often desirable for a Web site to identify users, either because the server wishes to restrict user access or because it want to serve content as a function of the user identity. For these purposes, HTTP uses cookies. Cookie technology has four components:  a cookie header line in the HTTP response message, a cookie header line in the HTTP request message, a cookie file kept on the user's end system and managed by the user's browser, a back-end database at the Web site. Cookies can thus be used to create a user session layer on top of stateless HTTP. 


## Web Caching
A Web cache, also called proxy server, is a network entity that satisfies HTTP requests on the behalf of an origin Web server. The Web cache has its own disk storage and keeps copies of recently requested objects in this storage. Note that a cache is both server and a client at the same time. When it receives requests from and sends responses to a browser, it is a server. When it sends requests to and receives responses from an origin server, it is a client.  
Web caching has seen deployment in the Internet for two reasons. First, a Web cache can substantially reduce the response time for a client request, particularly if the bottleneck bandwidth between the client and the origin server is much less than the bottleneck bandwidth between the client and the cache. If there is a high-speed connection between the client and the cache and if the cache has the requested object, then the cache will be able to deliver the object rapidly to the client. Second, Web caches can substantially reduce traffic on access links to the Internet. Furthermore, Web caches can reduce Web traffic in the Internet as a whole, thereby improving performance for all applications.  
Through the use of Content Distribution Networks, Web caches are increasingly playing an important role in the Internet. A CDN company installs many geographically distributed caches throughout Internet, thereby localizing much of the traffic.  


_How cache lower response time: Page 113_ 

## The Conditional GET
Although caching can reduce user-perceived response times, it introduces a new problem -a copy of an object residing in the cache may be stale. In other words, the object housed in the Web server may have been modified since the copy was cached at the client. Fortunately, HTTP has a mechanism that allows a cache to verify that its objects are up to date. This mechanism is called conditional GET. An HTTP request message is a so-called conditional GET message if the request message uses the GET method and the request message includes and _If-Modified-Since:_ header line.  

Let's walk through and example. First, on the behalf of a requesting browser, a proxy cache sends a request message to a Web server.
```
GET /fruit/kiwi.gif HTTP/1.1
Host: www.exotiquecuisine.com
```
Second, the Web server sends a response message with the requested object to the cache.
```
HTTP/1.1 200 OK
Date: Sat, 8 Oct 2011 15:39:29
Server: Apache/1.3.0 (Unix)
Last-Modified: Wed, 7 Sep 2011 09:23:24
Content-Type: image/gif

(data data data data data ...)
```
The cache forwards the object to the requesting browser but also caches the object locally. Importantly, cache also stores the last-modified date along with the object. Third, one week later, another browser requests the same object via the cache, and the object is still in the cache. Since this object may have been modified at the Web server in the past week, the cache performs and up-to-date check by issuing a conditional GET.
```
GET /fruit/kiwi.gif HTTP/1.1
Host: www.exotiquecuisine.com
If-Modified-Since: Wed, 7 Sep 2011 09:23:24
```
Note that the value of the _If-Modified-Since:_ header line is exactly equal to the value of the _Last-Modified:_ header line that was send by the server one week ago. This conditional GET is telling the server to send the object only if the object has been modified since the specified date. Suppose the object has not been modified. Then, the Web server sends a response message to the cache.
```
HTTP/1.1 304 Not Modified
Date: Sat, 15 Oct 2011 15:39:29
Server: Apache/1.3.0 (Unix)

(empty entity body)
```
We see that in response to the conditional GET, the Web server still sends a response message but does not include the requested object. Including the requested object would only waste bandwidth and increase user-perceived response time, particularly if the object is large. Note that this last response message has _304 Not Modified_ in the status line, which tells the cache that it can go ahead and forwards its cached copy of the object to the requesting browser. 


# File Transfer: FTP(Incomplete)

# Electronic Mail in the Internet(Incomplete)





# DNS—The Internet’s Directory Service
## Services Provided by DNS
There are two ways to identify a host, by a hostname and by an IP address. The main tasks of the Internet's __domain name system (DNS)__ is to provide a directory service that translates hostnames to IP addresses. The DNS is a distributed database implemented in a hierarchy of DNS servers, and an application layer protocol that allows hosts to query the distributed database. The DNS protocol runs over UDP and uses port 53. DNS is commonly employed by other application layer protocols including HTTP, SMTP and FTP.  

DNS provides a few other important services in addition to translating hostnames to IP addresses:

* _Host aliasing_: A host with a complicated hostname can have one or more alias names. For example, a hostname such as _relay1.west-coast.enterprise.com_ could have, say, two aliases such as _enterprise.com_ and _www\.enterprise.com_. In this case, the hostname _relay1.west-coast.enterprise.com_ is said to be a __canonical hostname__. Alias hostnames, when present, are typically more mnemonic than canonical hostnames. 
* _Mail server aliasing_: For obvious reasons, it is highly desirable that e-mail addresses are mnemonic. DNS can be invoked by a mail application to obtain the canonical hostname for a supplied alias hostname as well as the IP address of the host. In fact, the MX records permits a company's mail server and Web server to have identical hostnames; for example, a company's Web server and mail server can both be called enterprise.com
* _Load distribution_: DNS is also used to perform load distribution among replicated servers, such as replicated Web servers. Busy sites are replicated over multiple servers, with each server running on a different end system and each having a different IP address. For replicated Web servers, a set of IP addresses is thus associated with one canonical hostname. The DNS database contains this set of IP addresses. When client make a DNS query for a name mapped to a set of addresses, the server responds with the entire set of IP addresses, but rotates the ordering of the addresses within each reply. Because a client typically send its HTTP request message to the IP address that is listed first in the set, DNS rotation distributed the traffic among the replicated servers. DNS rotation is also used for e-mail so that multiple mail servers can have the same alias name. 

## Overview of How DNS Works

Suppose that some application (such as a Web browser or a mail reader) running in a user's host needs to translate a hostname to an IP address. The application will invoke the client side of DNS, specifying the hostname that needs to be translated. On many UNIX-based machines _gethostbyname()_ is the function call that an application calls in order to perform the translation. DNS in the user's host then takes over, sending a query message into the network. After a delay, DNS in the user's host receives a DNS reply message that provides the desired mapping. This mapping is then passed to the invoking application.  

A simple design for DNS would have one DNS server that contains all the mappings. In this centralized design, client simply direct all queries to the single DNS server, and the DNS server responds directly to the querying clients. Although the simplicity of this design is attractive, it is in appropriate for today's Internet, with its vast number of hosts. The problems with a centralized design include:

* __Single point of failure__: If the DNS server crashes, so does the entire Internet
* __Traffic volume__: A single DNS server would have to handle all DNS queries
* __Distant centralized database__: A single DNS server cannot be close to all the querying clients. Long distance between a client and a DNS server will lead to significant delays
* __Maintenance__: The single DNS server would have to keep records for all Internet hosts. Not only would this centralized database be huge, but it would have to be updated frequently to account for every new host.

A single DNS server doesn't scale. Consequently, the DNS is distributed by design. In fact, the DNS is a wonderful example of how a distributed database can be implemented in the Internet.

<h3>&nbsp;&nbsp;&nbsp;&nbsp;A Distributed, Hierarchical Database</h3>
The DNS uses a large number of servers, organized in a hierarchical fashion and distributed around the world. No single DNS server has all of the mappings for all of the hosts in the Internet. Instead, the mappings are distributed across the DNS servers. To a first approximation, there are three classes of DNS servers -root DNS servers, top-level domain (TLD) DNS servers, and authoritative DNS servers- organized in a hierarchy.  
Suppose a DNS client wants to determine the IP address for a hostname www.amazon.com. The client first contacts one of the root servers, which returns IP addresses for TLD servers for the top-level domain com. The client then contacts one of these TLD servers, which returns the IP address of an authoritative server for  amazon.com. Finally, the client contact one of the authoritative server for amazon.com, which returns the IP address for the hostname www.amazon.com. 

* __Root DNS servers__: In the Internet there are 13 root DNS servers (labeled A through M). Although we have referred to each of the 13 root DNS servers as if it were a single server, each "server" is actually a network of replicated servers, for both security and reliability purposes. All together, there are 247 root servers as of fall 2011.
* __Top-level domain (TLD)__: Those servers are responsible for top-level domains such as com, org, net, edu, and gov, and all of the country top-level domains such as uk, fr, ca, and jp. 
* __Authoritative DNS servers__: Every organization with publicly accessible hosts on the Internet must provide publicly accessible DNS records that map the names of those hosts to IP addresses. An organization's authoritative DNS server houses these DNS records. An organization can choose to implement its own authoritative DNS server to hold these records; alternatively, the organization can pay to have these records stored in an authoritative DNS server of some service provider. Most universities and large companies implement and maintain their own primary and secondary (backup) authoritative DNS server.

There is another type of DNS server called the __local DNS server__. A local DNS server does not strictly belong to the hierarchy of servers but is nevertheless central to the DNS architecture. Each ISP -such as a university, an academic department or a residential ISP- has a local DNS server (also called a default name server). When a host connects to an ISP, the ISP provides the host with the IP addresses of one or more of its local DNS servers (typically through DHCP). When a host makes a DNS query, the query is sent to the local DNS server, which acts as a proxy, forwarding the query into the DNS server hierarchy. Let's take a look at simple example:  
Suppose the host cis.poly.edu desires the IP address of gaia.cs.umass.edu. Also suppose that local DNS server is called dns.poly.edu and that an authoritative DNS server for gaia.cs.umass.edu is called dns.umass.edu. First, host sends a DNS query message to its local DNS server, dns.poly.edu. The local DNS server forwards the query message to a root DNS server. The root DNS server takes note of the edu suffix and returns to the local DNS server a list of IP addresses for TLD servers responsible for edu. The local DNS server then resends the query message to one of these TLD servers. The TLD server takes note of the umass.edu suffix and responds with the IP address of the authoritative DNS server for the University of Massachusetts, namely, dns.umass.edu. Finally, the local DNS server resends the query directly to dns.umass.edu, which responds with the IP address of gaia.cs.umass.edu.  

![DNS_SERVER](https://github.com/opwid/Library/blob/master/Computer-Networking-A-Top-Down-Approach/Images/dns-server.png) 

<h3>&nbsp;&nbsp;&nbsp;&nbsp;DNS Caching</h3>
DNS extensively exploits DNS caching in order to improve the delay performance and to reduce the number of DNS messages ricocheting around the Internet. In query chain, when a DNS server receives a DNS reply (containing, for example, a mapping from a hostname to an IP address), it can cache the mapping in its local memory. If a hostname/IP address pair is cached in a DNS server and another query arrives to the DNS server for the same hostname, the DNS server can provide the desired IP address,
even if it is not authoritative for the hostname. Because hosts and mappings between hostnames and IP addresses are by no means permanent, DNS servers discard cached information after a period of time (often set to two days).

## DNS Records and Messages
The DNS servers that together implement the DNS distributed database store __resource records__ (__RRs__), includings RRs that provide hostname-to-IP address mappings.  

A resource record is a four-tuple that contains the following fields:
```
(Name, Value, Type, TTL)
```
TTL is the time to live value of the resource record; it determines when a resource should be removed from a cache.  
If Type=A, then _Name_ is a hostname and _Value_ is the IP address for the hostname. Thus, Type A record provides the standard hostname-to-IP address mapping.  
If Type=NS, then _Name_ is a domain and _Value_ is the hostname of an authoritative DNS server that knows how to obtain the IP addresses for hosts in the domain. This record is used to route DNS queries further along in the query chain.  
If Type=CNAME, then _Value_ is a canonical hostname for the alias hostname _Name_. This record can provide querying hosts the canonical name for a hostname.  
If Type=MX, then _Value_ is the canonical name of mail server that has an alias hostname _Name_. MX records allow the hostnames of mail servers to have simple aliases. Note that by using the MX record, a company can have the same aliased name for its mail server and for a Web server. 

<h3>&nbsp;&nbsp;&nbsp;&nbsp;DNS Messages</h3>

Both query and reply messages have the same format. The semantics of the various fields in a DNS message are as follows:

* The first 12 bytes is the header section, which has a number of fields. The first field is a 16-bit number that identifies the query. A 1-bit query/reply flag indicates whether the message is a query(0) or a reply(1). A 1-bit authoritative flag is set in a reply message when a DNS server is an authoritative server for a queried name. A 1-bit recursion-desired flag is set when a client desires that the DNS server perform recursion when it doesn't have the record. In the header, there are also four number-of fields. These fields indicate the number of occurrences of the four types of data sections that follow the header.
* The question section contains information about the query that is being made. This section includes (1) a name field that contains the name that is being queried, and (2) a type field that indicates the type of question being asked about the name—for example, a host address associated with a name (Type A) or the mail server for a name (Type MX).
* In a reply from a DNS server, the answer section contains the resource records for the name that was originally queried. Recall that in each resource record there is the Type (for example, A, NS, CNAME, and MX), the Value, and the TTL. A reply can return multiple RRs in the answer, since a hostname can have multiple IP addresses.
* The authority section contains records of other authoritative servers.
* The additional section contains other helpful records. For example, the answer field in a reply to an MX query contains a resource record providing the canonical hostname of a mail server. The additional section contains a Type A record providing the IP address for the canonical hostname of the mail server.

# Peer-to-Peer Applications
The applications such as Web, e-mail, and DNS all employ client-server architectures with significant reliance on always-on infrastructure servers. In P2P architecture, there is minimal (or no) reliance on always-on infrastructure servers. Instead, pairs of intermittently connected hosts, called peers, communicate directly with each other.
## P2P File Distribution
In client-server file distribution, the server must send a copy of the file to each of the peers -placing an enormous burden on the server and consuming a large amount of server bandwidth. In P2P file distribution, each peer can redistribute any portion of the file it has received to any other peers, thereby assisting the server in the distribution process.

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Scalability of P2P Architectures</h3>
Denote the upload rate of the server's access link by u<sub>s</sub>, the upload rate of the ith peer's access link by u<sub>i</sub>, and the download rate of the ith peer's access link by d<sub>i</sub>. Aldo denote the size of the file to be distributed (in bits) by F and the number of peers that want to obtain a copy of the file by N. The distribution time is the time it takes to get a copy of the file to all N peers.  


Let's first determine the distribution time for the client-server architecture, which denoted by D<sub>cs</sub>.

* The server must transmit one copy of the file to each of the N peers. Thus the server must transmit NF bits. Since the server's upload rate is u<sub>s</sub>, the time to distribute the file must be at least NF/u<sub>i</sub>.
* Let d<sub>min</sub> denote the download rate of the peer with the lowest download rate, that is, d<sub>min</sub> = min{d<sub>1</sub>, d<sub>2</sub>, d<sub>3</sub>,..., d<sub>N</sub>}. The peer with the lowest download rate cannot obtain all F bits of the file in less than F/d<sub>min</sub> seconds. Thus the minimum distribution time is at least F/d<sub>min</sub>.

Putting these two observations together, we obtain D<sub>cs</sub> >= max{NF/u<sub>s</sub> , F/d<sub>min</sub>}.  
So for N large enough, the client-server distribution time is given by NF/u<sub>s</sub>. Thus, the distribution time increases linearly with the number of peers N. So, for example, if the number of peers from one week to the next week increases from a thousand to a million, the time required to distribute the file to all perrs increases by 1,000.  


Let's now go through a similar analysis for the P2P architecture. In particular, when a peer receives some file data, it can use its own upload capacity to redistribute the data to other peers. 

* At the beginning of the distribution, only the server has the file. To get this file into the community of peers, the server must send each bit of the file at least once into its access link. Thus, the minimum distribution time is at least F/u<sub>s</sub>. 
* As with the client-server architecture, the peer with the lowest download rate cannot obtain all F bits of the file in less than F/d<sub>min</sub>.
* Observe that the total capacity of the system as a whole is equal to the upload rate of the server plus the upload rates of each of the individual peers, that is, u<sub>total</sub> = u<sub>s</sub> + u<sub>1</sub> + ... + u<sub>N</sub>. The system must deliver F bits to each of the N peers, thus uploading a total of NF bits. This cannot be done at a rate faster than u<sub>total</sub>. Thus, the minimum distribution time is also at least NF/(u<sub>s</sub>+u<sub>1</sub>+...+u<sub>N</sub>)

Putting these two observations together, we obtain D<sub>p2p</sub> >= max{F/u<sub>s</sub> , F/d<sub>min</sub> , NF/u<sub>total</sub>}.  

## Distributed Hash Tables (DHTs)

In this section, we will consider how to build a distributed, P2P database that will store the (key,value) pairs over millions of peers. In the P2P system, each peer will only hold a small subset of the totality of the (key,value) pairs. We'll allow any peer to query the distributed database with a particular key. The distributed database will then locate the peers that have the corresponding (key, value) pairs and return the key-value pairs to the querying peer. Any peer will also be allowed to insert new key-value pairs into the database. Such a distributed database is referred to as a distributed hash table (DHT).  

Let's first describe a specific example DHT service in the context of P2P file sharing. In this case, a key is the content name and the value is the IP address of a peer that has a copy of the content. So, if Bob and Charlie each have a copy of the latest Linux distribution, then the DHT database will include the following two key-value pairs: (Linux, IP<sub>Bob</sub>) and (Linux, IP<sub>Charlie</sub>). More specifically, since the DHT database is distributed over the peers, some peer, say Dave, will be responsible for the key "Linux" and will have the corresponding key-value pairs. Now suppose Alice wants to obtain a copy of
Linux. Clearly, she first needs to know which peers have a copy of Linux before she can begin to download it. To this end, she queries the DHT with "Linux" as the key. The DHT then determines that the peer Dave is responsible for the key "Linux". The
DHT then contacts peer Dave, obtains from Dave the key-value pairs (Linux, IP<sub>Bob</sub>) and (Linux, IP<sub>Charlie</sub>), and passes them on to Alice. Alice can then download the latest Linux distribution from either IP<sub>Bob</sub> or IP<sub>Charlie</sub>.  

Now let’s return to the general problem of designing a DHT for general key-value pairs. One naïve approach to building a DHT is to randomly scatter the (key,value) pairs across all the peers and have each peer maintain a list of the IP addresses of all participating peers. In this design, the querying peer sends its query to all other peers, and the peers containing the (key, value) pairs that match the key can respond with their matching pairs. Such an approach is completely unscalable,of course, as it would require each peer to not only know about all other peers but even worse, have each query sent to all peers.  

We now describe an elegant approach to designing a DHT. First, assign an identifier to each peer in the range [0, 2<sup>n</sup>-1] for some fixed n. Note that each such identifier can be expressed by an n-bit representation. Let's also require each key to be an integer in the same range by using a hash function.  
Let's now consider the problem of storing the (key,value) pairs in the DHT. The central issue here is defining a rule for assigning keys to pairs. Given that each peer has an integer identifier and that each key is also an integer in the same range, a natural approach is to assign each (key,value) pair to the peer whose identifier is the closest to the key. For convenience, let's define the closest peer as the closest successor of the key. Suppose n=4 so that all the peer and the key identifiers are in the range [0,15]. Further suppose that there are eight peers in the system with identifiers 1,3,4,5,8,10,12 and 15. Finally, suppose we want to store the (key, value) pair (11, Johnny Wu) in one of the eight peers. But in which peer? Using our closest convention, since peer 12 is the closest successor for key 11, we therefore store the pair (11, Johnny Wu) in the peer 12. Now suppose a peer, Alice, wants to insert a (key, value) pair into the DHT. Conceptually, this is straightforward: She first determines the peer whose identifier is closest to the key; she then sends a message to that peer, instructing it to store the (key, value) pair. But how does Alice determine the peer that is closest to the key? If Alice were to keep track of all the peers in the system (peer IDs and corresponding IP addresses), she could locally determine the closest peer. But such an approach requires each peer to keep track of all other peers in the DHT—which is completely impractical for a large-scale system with millions of peers. 

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Circular DHT</h3>
To address this problem of scale, let's now consider organizing the peers into a circle. In this circular arrangement, each peer only keeps track of its immediate successor and immediate predecessor (modulo 2<sup>n</sup>).  
![CIRCULAR_DHT](https://github.com/opwid/Library/blob/master/Computer-Networking-A-Top-Down-Approach/Images/circular_dht.png)   

Now suppose that peer 3 wants to determine which peer in the DHT is responsible for key 11. Using the circular overlay, the origin peer (peer 3) creates a message saying "Who is responsible for key 11?" and sends this message clockwise around the circle. Whenever a peer receives such a message, because it knows the identifier of its successor and predecessor, it can determine whether it is responsible for (that is, closest to) the key in question. If a peer is not responsible for the key, it simply sends the message to its successor. So, for example, when peer 4 receives the message asking about key 11, it determines that it is not responsible for the key (because its successor is closer to the key), so it just passes the message along to peer 5. This process continues until the message arrives at peer 12, who determines that it is the closest peer to key 11. At this point, peer 12 can send a message back to the querying peer, peer 3, indicating that it is responsible for key 11.  
The circular DHT provides a very elegant solution for reducing the amount of overlay information each peer must manage. Thus, in designing a DHT, there is tradeoff between the number of neighbors each peer has to track and the number of messages that the DHT needs to send to resolve a single query. On one hand, if each peer tracks all other peers (mesh overlay), then only one message is sent per query, but each peer has to keep track of N peers. On the other hand, with a circular DHT, each peer is only aware of two peers, its immediate successor and its immediate predecessor, but N/2 messages are sent on average for each query. Fortunately, we can refine our designs of DHTs so that the number of neighbors per peer as well as the number of messages per query is kept to an acceptable size. One such refinement is to use the circular overlay as a foundation, but add "shortcuts" so that each peer not only keeps track of its immediate successor and predecessor, but also of a relatively small number of shortcut peers scattered about the circle.

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Peer Churn</h3>
In P2P systems, a peer can come or go without warning. Thus, when designing a DHT, we also must be concerned about maintaining the DHT overlay in the presence of such peer churn. To handle peer churn, we will now require each peer to track (that is, know the IP address of) its first and second successors; for example, peer 4 now tracks both peer 5 and peer 8. We also require each peer to periodically verify that its two successors are alive (for example, by periodically sending ping messages to them and asking for responses). Now suppose peer 5 departed, since it no longer respond to ping messages. Peers 4 and 3 need to update their successor state information. Let's consider how peer 4 updates its state:

* Peer 4 replaces its first successor (peer 5) with its second successor (peer 8).
* Peer 4 then asks its new first successor (peer 8) for identifier and IP address of its immediate successor (peer 10). Peer 4 then makes peer 10 its second successor.  


Now consider what happens when a peer wants to join DHT. Let’s say a peer with identifier 13 wants to join the DHT, and at the time of joining, it only knows about peer 1’s existence in the DHT. Peer 13 would first send peer 1 a message, saying "what will be 13’s predecessor and successor?". This message gets forwarded through the DHT until it reaches peer 12, who realizes that it will be 13's predecessor and that its current successor, peer 15, will become 13's successor. Next, peer 12 sends this predecessor and successor information to peer 13. Peer 13 can now join the DHT by making peer 15 its successor and by notifying peer 12 that it should change its immediate successor to 13.
