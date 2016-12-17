# Introduction
## Forwarding and Routing
The role of the network layer is thus deceptively simple -to move packets from a sending host to a receiving host. To do so, two important network-layer functions can be identified:

* _Forwarding_: When a packet arrives at a router's input link, the router must move the packet to the appropriate output link.
* _Routing_: The network layer must determine the route or path taken by packets as they flow from a sender to a receiver. The algorithms that calculate these paths are referred to as routing algorithms.

The terms forwarding and routing are often used interchangeably by authors discussing the network layer. Forwarding refers to the router-local action of transferring a packet from an input link interface to the appropriate output link interface. Routing refers to the network-wide process that determines the end-to-end paths that packets take from source to destination.  

Every router has a __forwarding table__. A router forwards a packet by examining the value of a field in the arriving packet's header, and then using this header value to index into the router's forwarding table. The value stored in the forwarding table entry for that header indicates the router's outgoing link interface to which that packet is to be forwarded. Depending on the network-layer protocol, the header value could be the destination address of the packet or an indication of the connection to which the packet belongs.  

You might now be wondering how the forwarding tables in the routers are configured. This is a crucial issue, one that exposes the important interplay between routing and forwarding. The routing algorithm determines the values that are inserted into the router's forwarding tables. The routing algorithm may be centralized (e.g., with an algorithm executing on a central site and downloading routing information to each of the routers) or decentralized (i.e., with a piece of the distributed routing algorithm running in each router). In either case, a router receives routing protocol messages, which are used to configure its forwarding table. We're thus fortunate that all networks have both a forwarding and a routing function!  

We'll reserve the term packet switch to mean a general packet-switching device that transfers a packet from input link interface to output link interface, according to the value in a field in the header of the packet. Some packet switches, called link-layer switches, base their forwarding decision on values in the fields of the link-layer frame; switches are thus referred to as link-layer (layer 2) devices. Other packet switches, called routers, base their forwarding decision on the value in the network-layer field. Routers are thus network-layer (layer 3) devices, but must also implement layer 2 protocols as well, since layer 3 devices require the services of layer 2 to implement their (layer 3) functionality. To confuse matters, marketing literature often refers to "layer 3 switches" for routers with Ethernet interfaces, but these are really layer 3 devices. Since our focus in this chapter is on the network layer, we use the term router in place of packet switch. We'll even use the term router when talking about packet switches in virtual-circuit networks (soon to be discussed).

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Connection Setup</h3>
We just said that the network layer has two important functions, forwarding and routing. But we'll soon see that in some computer networks there is actually a third important network-layer function, namely, connection setup. Recall from our study of TCP that a three-way handshake is required before data can flow from sender to receiver. This allows the sender and receiver to set up the needed state information (for example, sequence number and initial flow-control window size). In an analogous manner, some network-layer architectures -for example, ATM, frame relay, and MPL- require the routers along the chosen path from source to destination to handshake with each other in order to set up state before network-layer data packets within a given source-to-destination connection can begin to flow. In the network layer, this process is referred to as connection setup.

## Network Service Model
The network service model defines the characteristics of end-to-end transport of packets between sending and receiving end systems. The Internet's network layer provides a single service, known as best-effort service. From Table 4.1, it might appear that best-effort service is a euphemism for no service at all.  

![4_1](https://github.com/opwid/Library/blob/master/Computer-Networking-A-Top-Down-Approach/Images/4_1.png) 

Other network architectures have defined and implemented service models that go beyond the Internet's best-effort service. For example, the ATM network architecture provides for multiple service models, meaning that different connections can be provided with different classes of service within the same network.

# Virtual Circuit and Datagram Networks
A network layer can provide connectionless service or connection service between two hosts. Network-layer connection and connectionless services in many ways parallel transport-layer connection-oriented and connectionless services. For example, a network-layer connection service begins with handshaking between the source and destination hosts; and a network-layer connectionless service does not have any handshaking preliminaries.  

Although the network-layer connection and connectionless services have some parallels with transport-layer connection-oriented and connectionless services, there are crucial differences:

* In the network layer, these services are host-to-host services provided by the network layer for the transport layer. In the transport layer these services are process-to-process services provided by the transport layer for the application layer.
* In all major computer network architectures to date (Internet, ATM, frame relay, and so on), the network layer provides either a host-to-host connectionless service or a host-to-host connection service, but not both. Computer networks that provide only a connection service at the network layer are called __virtual-circuit (VC) networks__; computer networks that provide only a connectionless service at the network layer are called __datagram networks__.
* The implementations of connection-oriented service in the transport layer and the connection service in the network layer are fundamentally different. We'll see shortly that the network-layer connection service is implemented in the routers in the network core as well as in the end systems. 
## Virtual Circuits
While the Internet is a datagram network, many alternative network architectures -including those of ATM and frame relay- are virtual-circuit networks and, therefore, use connections at the network layer. These network-layer connections are called virtual circuits (VCs). Let's now consider how a VC service can be implemented in a computer network.  
A VC consists of (1) a path (that is, a series of links and routers) between the source and destination hosts, (2) VC numbers, one number for each link along the path, and (3) entries in the forwarding table in each router along the path. A packet belonging to a virtual circuit will carry a VC number in its header. Because a virtual circuit may have a different VC number on each link, each intervening router must replace the VC number of each traversing packet with a new VC number. The new VC number is obtained from the forwarding table.  
To illustrate the concept, consider the network shown in Figure 4.3. The numbers next to the links of R1 in Figure 4.3 are the link interface numbers. Suppose now that Host A requests that the network establish a VC between itself and Host B. Suppose also that the network chooses the path A-R1-R2-B and assigns VC numbers 12, 22, and 32 to the three links in this path for this virtual circuit. In this case, when a packet in this VC leaves Host A, the value in the VC number field in the packet header is 12; when it leaves R1, the value is 22; and when it leaves R2, the value is 32.  
![4_3](https://github.com/opwid/Library/blob/master/Computer-Networking-A-Top-Down-Approach/Images/4_3.png)  

How does the router determine the replacement VC number for a packet traversing the router? For a VC network, each router's forwarding table includes VC number translation; for example, the forwarding table in R1 might look something like this:  
![4_3_1](https://github.com/opwid/Library/blob/master/Computer-Networking-A-Top-Down-Approach/Images/4_3_1.png)  

Whenever a new VC is established across a router, an entry is added to the forwarding table. Similarly, whenever a VC terminates, the appropriate entries in each table along its path are removed.  
You might be wondering why a packet doesn't just keep the same VC number on each of the links along its route. The answer is twofold. First, replacing the number from link to link reduces the length of the VC field in the packet header. Second, and more importantly, VC setup is considerably simplified by permitting a different VC number at each link along the path of the VC.  
In a VC network, the network's routers must maintain __connection state information__ for the ongoing connections. Specifically, each time a new connection is established across a router, a new connection entry must be added to the router's forwarding table; and each time a connection is released, an entry must be removed from the table. Note that even if there is no VC-number translation, it is still necessary to maintain connection state information that associates VC numbers with output interface numbers.  
There are three identifiable phases in a virtual circuit:

* VC Setup: During the setup phase, the sending transport layer contacts the network layer, specifies the receiver's address, and waits for the network to set up the VC. The network layer determines the path between sender and receiver, that is, the series of links and routers through which all packets of the VC will travel. The network layer also determines the VC number for each link along the path. Finally, the network layer adds an entry in the forwarding table in each router along the path. During VC setup, the network layer may also reserve resources (for example, bandwidth) along the path of the VC.
* Data Transfer: Once the VC has been established, packets can begin to flow along the VC.
* VC Teardown: This is initiated when the sender (or receiver) informs the network layer of its desire to terminate the VC. The network layer will then typically inform the end system on the other side of the network of the call termination and update the forwarding tables in each of the packet routers on the path to indicate that the VC no longer exists.

In a VC network layer, routers along the path between the two end systems are involved in VC setup, and each router is fully aware of all the VCs passing through it.  
The messages that the end systems send into the network are known as __signaling messages__, and the protocols used to exchange these messages are often referred to as __signaling protocols__.

## Datagram Networks
In a datagram network, each time an end system wants to send a packet, it stamps the packet with the address of the destination end system and then pops the packet into the network.  

As a packet is transmitted from source to destination, it passes through a series of routers. Each of these routers uses the packet's destination address to forward the packet.  Specifically, each router has a forwarding table that maps destination addresses to link interfaces; when a packet arrives at the router, the router uses the packet's destination address to look up the appropriate output link interface in the forwarding table. The router then intentionally forwards the packet to that output link interface.  

To get some further insight into the lookup operation, let's look at a specific example. Suppose that all destination addresses are 32 bits (which just happens to be the length of the destination address in an IP datagram). A brute-force implementation of the forwarding table would have one entry for every possible destination address. Since there are more than 4 billion possible addresses, this option is totally out of the question.  
Now let's further suppose that our router has four links, numbered 0 through 3, and that packets are to be forwarded to the link interfaces as follows:  

![4_5_1](https://github.com/opwid/Library/blob/master/Computer-Networking-A-Top-Down-Approach/Images/4_5_1.png)  

Clearly, for this example, it is not necessary to have 4 billion entries in the router's forwarding table. We could, for example, have the following forwarding table with just four entries:  

![4_5_2](https://github.com/opwid/Library/blob/master/Computer-Networking-A-Top-Down-Approach/Images/4_5_2.png)  

With this style of forwarding table, the router matches a prefix of the packet's destination address with the entries in the table; if there's a match, the router forwards the packet to a link associated with the match. When there are multiple matches, the router uses the longest prefix matching rule; that is, it finds the longest matching entry in the table and forwards the packet to the link interface associated with the longest prefix match.  

The time scale at which this forwarding state information changes is relatively slow. Indeed, in a datagram network the forwarding tables are modified by the routing algorithms, which typically update a forwarding table every one-to-five minutes or so. Because forwarding tables in datagram networks can be modified at any time, a series of packets sent from one end system to another may follow different paths through the network and may arrive out of order.

# What's Inside a Router?
Four router components can be identified:

* _Input ports_: An input port performs several key functions. It performs the physical layer function of terminating an incoming physical link at a router; this is shown in the leftmost box of the input port and the rightmost box of the output port. An input port also performs link-layer functions needed to interoperate with the link layer at the other side of the incoming link; this is represented by the middle boxes in the input and output ports. Perhaps most crucially, the lookup function is also performed at the input port; this will occur in the rightmost box of the input port. It is here that the forwarding table is consulted to determine the router output port to which an arriving packet will be forwarded via the switching fabric. Control packets (for example, packets carrying routing protocol information) are forwarded from an input port to the routing processor.
* _Switching fabric_: The switching fabric connects the router's input ports to its output ports. This switching fabric is completely contained within the router -a network inside of a network router!
* _Output ports_: An output port stores packets received from the switching fabric and transmits these packets on the outgoing link by performing the necessary link-layer and physical-layer functions. When a link is bidirectional, an output port will typically be paired with the input port for that link on the same line card (a printed circuit board containing one or more input ports, which is connected to the switching fabric).
* _Routing processor_: The routing processor executes the routing protocols, maintains routing tables and attached link state information, and computes the forwarding table for the router. It also performs the network management functions that we'll study in Chapter 9.





