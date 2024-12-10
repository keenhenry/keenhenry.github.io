---
layout: post
title:  Computer Network Notes
date:   2024-09-01
categories: [technology, computer network, notes]
tags: [technology, computer network, notes]
description: Study notes of computer network technology
---

## OSI model

OSI stands for **Open System Interconnection** model. OSI model has 7 layers. But the most commonly (arguably) used
model is the 'five-layer' model, which includes **physical layer** (layer 1), data **link layer** (layer 2), **network layer** (layer 3),
**transport layer** (layer 4) and **application layer** (layer 5). OSI model adds two additional layers in between transport layer and
application layer, namely _session layer_ and _presentation layer_.

### Encapsulation and Decapsulation

The key take-away with the OSI model or the five-layer model is how data flows among the layers of the model. Let's only use
the five-layer model to explain.

When an service (say, service `A`) in the application layer needs to send data to the another application running on another node, here is
what happens:

1. The data service `A` is sending gets **encapsulated** in the layers below; first, data is being treated as the payload of the packets
   in transport layer when making transport layer packets. Then the same thing is done in the network layer to wrap transport layer
   packets in the payload of the IP datagrams. This continues until (and including) data link layer. In the end, Ethernet frames are
   created before sending out on the network.
2.


A video about OSI models and exactly what I tried to explain 😃

{% include embed/youtube.html id='3b_TAYtzuho' %}


## Network Configuration

There are four necessary settings needed to be configured for a 'node' to run correctly on a network:

- IP address (can be configured via DHCP)
- Subnet mask (can be configured via `ip` command on Linux temporarily or edit `/etc/network/interfaces` file for permanent change)
- (Default) Gateway for a host (can also be configured via `ip` command for temporary change)
- DNS server to use (TODO: add a link to your other post discussing DNS)

## Network Devices

### What is a Hub (rarely used today)

A hub is a **physical layer** (**layer 1**) device that allows for connections from many computers
at once. All the devices connected to a hub will end up talking to all other devices at the same time.
The devices connected to the hub will need to determine themselves whether the message was
meant for them or ignore if it's not. This way of communication creates lots of noises on the
network. In addition, only one device can talk at a given time, otherwise the electric pulses
sent over the wires can interfere with each other . This is also known as a _Collision Domain_.

**collision domain**
: A network segment where only one device can communicate at a time.


### What is a Switch (a.k.a. switching hub)

It is like a hub which can connect many network devices for them to communicate with each other.
However, a switch is a **(data) link layer** (**layer 2**) device, which means it can inspect
the contents of ethernet protocol data and determine which system the data was intended for and
only send that data to that one system.

This reduces a lot of noises and eliminates a lot of collision domains on a network! This makes
a network faster (than a network connected by a hub!).


### What is a Router

A device that forwards traffic depending on the destination address of that traffic. It is a
**network layer** (**layer 3**) device. While hubs and switches connects devices within a single
network, a.k.a LAN (Local Area Network), a router connects many LANs together; it allows communication
between different LANs.

A router can inspects IP data to determine where to forward the data to. A router keeps a routing
table that keeps track of a bunch of IPs and their corresponding destinations. Routers themselves
share data with each other via a protocol called **BGP** (**Border Gateway Protocol**). This protocol
lets them learn about the most optimal paths to forward traffic.

A router has at least *TWO* network interfaces (two IPs!), since it has to connect to two networks to
do its jobs.


### What is a Firewall

A device that blocks traffic that meets certain criteria. It can stop traffic you don't want entering
a network. Firewalls can operate at lots of different layers of the network. But it is most commonly
used at the **transport layer**. A typical firewall has a configuration that allows traffic for certain
ports while blocking traffic to other ports.

While a firewall can be a physical device that does the job, many firewalls are usually just a program
running on some devices or computers. For example, many routers have firewalls programs built-in as well.


### What is a NAT
#### NAT Masquerading
### What is a Bridge

## Network Protocols

### Ethernet

Ethernet is a **link layer** (**layer 2**) protocol. It abstracts away the physical layer details
like what wires and hardware are used and is the protocol to resolve collisions inside a collision
domain.


### MAC address

MAC address is the addressing method for the **(data) link layer** (**layer 2**).
MAC address is globally unique, each individual network interface gets a unique MAC address.
A MAC address is a **48** bit number, which usually represented by 6 groups of two hexadecimal numbers.

The reason for MAC address to exist is when devices are within a network segment where collisions are
possible, you need unique addresses for each device so that you know who exactly the sent data is meant
for.

A MAC address contains 2 parts. The first part (first 3 octets) is OUI (Organizationally Unique Identifier),
which is assigned by IEEE to each individual hardware manufacturer. That means you can identify the hardware
manufacturer of a network interface purely by its MAC address. The second part (last 3 octets) is assigned by
the manufacturer with the condition that they only assign each possible address once to keep the address
globally unique.


#### Unicast, Multicast, Broadcast

These are transmission types under Ethernet protocol.

Broadcast
: a broadcast is sent to every single device on a LAN. The ethernet broadcast address is all `F`'s.

Multicast
: a multicast is sent to multiple devices (not all, nor single) on a LAN.
The least significant bit in the first octet of a destination (MAC) address is
set to one, then that means you're dealing with a multicast frame.
When a multicast message is sent, it is sent to all the devices on the collision
domain, but only the intended destination**s** will accept and process the message, the remaining
devices on the domain will simply disgard the message. Network interfaces can be configured
to accept lists of configured multicast addresses for this type of communication.

Unicast
: A unicast transmission is always meant for just one receiving address.
The least significant bit in the first octet of a destination (MAC) address is
set to zero, then it means the ethernet frame is intended for only the destination address.
When a unicast message is sent, it is sent to all the devices on the collision
domain, but only the intended destination will receive and process the message, the other
devices on the domain will simply disgard the message.


### ARP (Address Resolution Protocol)

**ARP** is a protocol used to discover the hardware (MAC) address of a node with a certain IP address.
It is a data **link layer** (**layer 2**) protocol.

#### Why?

When a IP datagram is fully formed, it needs to be encapsultaed into the Ethernet frame for
transmission. In doing so, the transmitting device needs to know the destination MAC address to
complete the Ethernet frame header (to be able to send the frame!). This is where ARP protocol comes
into play.

#### How it works

Before diving in the details, let's first introduce a terminology:

ARP table
: A table with a list of IP addresses and the MAC addresses associated with them. ARP table entries
  generally expire after a short amount of time to ensure changes in the network are accounted for.

Now, let's walk through the protocol:

When a networked device tries to send an IP datagram to a node with IP address `10.5.15.100`,
it first needs to know the MAC address of that corresponding IP (so that the information can
be filled in in the Ethernet frame during data encapsulation). It looks up its ARP table and finds
that there is no entry in the table for this IP. Then, it sends out an **ARP message** to the
**MAC broadcast address** (which is all `F`s) on the network. This message is delivered to
_ALL_ computers on the local network. The computer on the network with IP `10.5.15.100` receives
this ARP message and sends out an **ARP response** which contains the MAC address of the network
interface in question. The other computers on the network simply discard the ARP message (because
they don't have the IP address in question). Now the transmitting device can fill in the Ethernet
frame header with the correct destination MAC address and thus make the frame complete and subsequently
send out the frame to the device with IP `10.5.15.100`. Additionally, the transmitting device will
likely also add one entry to its local ARP table so that it will not need to resend an ARP message
next time when it needs to communicate with the device with this IP.


#### Linux Command

The command to use on Linux to manipulate local ARP cache or lookup ARP information is simply `arp`.

```bash
$ man arp

# Print current content of ARP table:
$ arp

# Delte and entry
$ arp -d <address>

# Add a new table entry
$ arp -s <address> <hw_addr>
```


### What is an IP address

- First of all, this is a **network layer** (**layer 3**) property.
- IP addresses belong to networks, not to the devices attached to those networks. They are used
  as IDs for devices connected to the network. For devices that communicate with the **Internet
  Protocol**. An IP is assigned to a device when a device is connected to a network.
- **Dynamic IP addresses** are assigned to a device automatically when that device
  is connected to a network. This is done via a protocol called DHCP (Dynamic Host
  Configuration Protocol).
- **Static IP addresses**, on the other hand, need to be configured manually on a node. Static
  IPs are more common for servers, where you might need a fixed IP in certain situations. To
  make a permanent static IP on a Linux node, we usually need to edit some configuration files
  in the filesystem. Different Linux distribution uses different files for configuring such
  information.


### Subnetting

- Subnetting is the process of splitting up a larger network into smaller subnetworks.
- Why we need subnets? A gateway router for a network (for example, class A network) cannot
  keep too many IPs with it, it's just not practical, that's why we need subnetworks or subnets
  to split things into smaller chunks and let smaller gateway routers to do forwarding jobs.


#### Subnet masks

Subnet masks are 32-bit numbers that are used to calculate subnet IDs from IP addresses. They are
usually written out as 4 octets in decimal.

A subnet mask contains two parts, the first part is a sequence of `1`'s and the remaining part
is all `0`'s. When doing an bitwise logical AND operation with and IP address and its subnet mask,
you will get the subnet ID (or just network ID after CIDR was introduced), the ID of the network this
IP belongs go. For example:

```
IP: 192.168.5.85
Subnet mask: 255.255.255.0 (/24 in CIDR notation)

Convert numbers to binary format:

192.168.5.85  = 11000000.10101000.00000101.01010101
255.255.255.0 = 11111111.11111111.11111111.00000000

Apply bitwise logical AND operation:

11000000.10101000.00000101.01010101 ^
11111111.11111111.11111111.00000000
===================================
11000000.10101000.00000101.00000000

Covert numbers back to decimal format:

192.168.5.0 => network ID for the subnetwork!!!
85          => host ID for the host!!!

With this information, we can further derive other important attributes
of the network / subnetwork:

1. Broadcast address in the subnet: 192.168.5.255
2. Host addresses: 192.168.5.1 ~ 192.168.5.254 (254 available addresses)
```


#### CIDR (Classless Inter-Domain Routing)

CIDR is a way of allocating IP addresses for IP routing. It replaces the traditional classful network
method and allows an address to be defined entirely by just two numbers: network ID and host ID. It was
created to slow the growth of routing tables and slow the exhaustion of IPv4 addresses.

CIDR is simply much more flexible than the classful way of network demarcation because it is based on
variable length subnet masking. This allows finer control of the sizes of subnetworks, avoiding allocating
larger subnets than needed.

CIDR notation is a compact representation of an IP address and its associated network mask. use
the example above again (`192.168.5.85` with `255.255.255.0` subnet mask), we can write its CIDR
notation like this: `192.168.5.95/24`.


#### Network ID

This is the ID to identify a network on the internet or intranet. Back in the old days, when classful network
was still in use, the network ID of an IP address is the most significant 1 byte (for **class A**), 2 bytes
(**class B**) or 3 bytes (**class C**) in the IP address. After CIDR was introduced, a network ID is of _variable
length_ and is defined by the subnet mask.


### Routing Concepts

How does routing work (in a router)?

1. A router receives a packet.
2. It examines destination IP in that packet. (It strips away Ethernet frame data and extracts only the IP datagram
   that was encapsulated inside it; this is decapsultaion!)
4. It looks up the destination network of this IP in its routing table.
5. It forwards traffic to through the interface that's closest to the remote network
   as determined by the additional informaiton in the routing table.

These steps are repeated as often as needed until the traffic reaches its destination. Also note that a router itself
has also ARP table, so it can look up MAC addresses of IPs that's directly connected to it and sets destination MAC
addresses of the Ethernet frames it is sending out.


#### Basic Routing table

| Destination Network |   Next Hop  | Total Hops |   Interface   |
| ------------------- | ----------- | ---------- | ------------- |
| 192.168.1.1/24      | 192.173.0.1 |     5      | 192.173.0.254 |
| 101.66.27.0/24      | 10.11.0.1   |     3      | 10.11.0.25    |

Destination Network
: This is just the definition of the remote network, a network ID and a net mask. This definition for a network
  allows the router to know what IP addresses might live on that network. When the router receives an incoming packet,
  it examines the destination IP address and determines which network it belongs to.

Next Hop
: This is the IP address of the next router that should receive data intended for the destination network in question.

Total hops
: The router keeps track of how far away that destination currently is; this number is frequently updated.

Interface
: Which of its interfaces it should forward traffic matching the destination network out of.

**Linux Command**

What is the command to check the content of local routing table?

```bash
$ ip route

# Or
$ netstat -rn

# Or
$ route -n
```


#### Routing Protocols

Routing protocols are used by routers to learn the world around them (so that they have most up-to-date information about
best routes/paths for packets). These protocols define how routers should speak to each other and share information with each other.

There are two categories of routing protocols:

1. **Interior Gateway Protocols**
2. **Exterior Gateway Protocols**


##### Interior Gateway Protocols

Interior gateway protocols are used by routers to share information within a single autonomous system. In networking terms, an autonomous
system is a collection of networks that all fall under the control of a single network operator.

An example autonomous system is a large corporation that needs to route data between their many offices and each of which might have their
own local area network.

Another example of autonomous system is an Internet service provider (whose reaches are national in scale) employs many routers.

There are two types of Interior Gateway Protocols:

- **Link State Routing protocols**
- **Distance Vector protocols** (older, outdated protocols, slower routing information propagation)

Nowadays, **Link State Routing protocols** are used among routers.


##### Exterior Gateway Protocols

Exterior Gateway Protocols are used by routers for the exchange of information between independent autonomous system**s**.
To be more precise, Exterior Gateway Protocols are used to communicate data between routers representing the edges of an
autonomous system, routers use exterior gateway protocols when they need to share information across different organizations.

Exterior gateway protocols are the key to the Internet today. They are usually the concerns of **core Internet routers**.
Getting data to the _edge router_ of an autonomous system is the number one goal of **core Internet routers**.

For the core Internet routers, autonomous systems are known and defined. Each autonomous network has a **Autonomous System
Number** (ASN) assigned to it by IANA (Internet Assigned Numbers Authority; also the authority that assigns IP addresses!).
This information is kept in the core Internet routers to identified autonomous networks.

Nowadays, [**Border Gateway Protocol**][bgp] is the most commonly used Exterior Gateway Protocol.


#### Non-Routable Address Space

These are ranges of IPs set aside for use by anyone that cannot be routed to.

The reason for the creation of such address space was that IPv4 simply couldn't provide enough
IP addresses for all networked devices on earth so one mitigation was to make certain IP ranges
**reusable** by many organizations or individuals internally, as long as these IP ranges are not
exposed on the internet (and hence no IP address conflicts). And for the devices having non-Routable
IP addresses, they will use a **NAT** to translate their addresses to something that's routable
on the internet! Now, the details:

_RFC 1918_ defined three ranges of IP addresses that will never be routed anywhere by **core** routers:

- **10.0.0.0/8**
- **172.16.0.0/12**
- **192.168.0.0/16**

These ranges are free for anyone to use for their internal networks! However, note that interior gateway protocols
_will_ route these address spaces. They're appropriate for use within an autonomous system, but exterior gateway protocols
_will not_.

Do they look familiar? Exactly, your IP in the SOHO network and IPs in your offices and Docker networks are usually falling
into these ranges!

### Transport and Application Layers

The first three layers of OSI model define how each individual node sends data to each other. But it does not mean much to
humans. What we humans actually need is '_computer programs_/_applications_' running on computers to be able to talk to each
other.

Transport layer protocols allows traffic to be directed to specific network applications, and the application layer allows
these applications to communicate in a way they understand.

Transport layer has the ability to multiplex and demultiplex. This makes it unique compared to all other layers. The transport
layer handles multiplexing and demultiplexing through **port**s. Different services run on a computer while listening on
specific ports for incomming requests.

Multiplex
: Nodes on a network can direct traffic to many different receiving services.

Demultiplex
: Taking traffic that is directing at the same node to the proper receiving services (think about different processes
  running on the same node).

Port
: A port is a 16-bit number that's used to direct traffic to specific services running on a networked computer.


#### TCP vs. UDP

TCP and UDP are transport layer protocols. They have different characteristics and have different use-cases.

**TCP**

0. Connection-Oriented protocol. Data is resent if needed when data is not acknowledged/lost. The information
   like **sequence number** in a TCP segment is key to this capability.
1. Suitable for reliable data transfer.
2. Unicast communication.
3. Used by application layer protocols like `HTTP`, `FTP`, `Telnet` and `SMTP` etc.


**UDP**

0. Connectionless protocol.
1. Best effort data transfer, fast but not guaranteed delivery.
2. Can be used in Unicast, Multicast and Broadcast communication.
3. Used by application layer protocols like VoIP, DNS, NFS, live streamming, video chats
   and online gaming etc.


Socket
: The instantiation of an endpoint in a potential TCP connection. TCP sockets require actual
  programs to instantiate them. Sockets and ports are different things. Ports are more virtual;
  a program needs to open a socket on a port to receive / send data to the other end.


**TCP socket states**

`LISTEN`
: A TCP socket is ready and listening for incoming connections. Only visible on server side.


`SYNC_SENT`
: A synchronization request has been sent, but the connection has not been established yet. This
  is only visible on the client side.


`SYNC_RECEIVED`
: A socket previously in a `LISTEN` state has receieved a synchronization request and sent a `SYN/ACK`
  back. But it hasn't received the final ACK from the client yet. This is only visible on the server
  side.


`ESTABLISHED`
: The TCP connection is in working order and both sides are free to send each other data. This is
  visible for both the client and server sides of the connection.


`FIN_WAIT`
: A `FIN` has been sent, but the corresponding `ACK` from the other end hasn't been received yet. This
  is also visible for both the client and server sides.


`CLOSE_WAIT`
: The connection has been closed at the TCP layer, but the application that opened the socket hasn't
  released its hold on the socket yet. Visible for both the client and server sides.


`CLOSED`
: The connection has been fully terminated and that no further communication is possible. Visible for both
  the client and server sides.

When troubleshooting the TCP layer protocol problems, you will need to understand the meaning of these
socket states. The definitions are not defined by the TCP protocol itself, instead, they are different
per OS vendors. You should check the documentation of the socket state definitions from the OS you're
working on.

The Linux command to view the socket states is `ss`:

```bash
# Display socket summary
$ ss -s
Total: 1078
TCP:   14 (estab 3, closed 4, orphaned 0, timewait 4)

Transport Total     IP        IPv6
RAW       1         0         1
UDP       13        5         8
TCP       10        7         3
INET      24        12        12
FRAG      0         0         0

# Display all open network ports
$ ss -l
```


### WIFI

WIFI is a **link layer** (**layer 2**) protocol.


[bgp]: https://en.wikipedia.org/wiki/Border_Gateway_Protocol
