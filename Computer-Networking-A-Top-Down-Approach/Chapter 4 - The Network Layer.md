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


