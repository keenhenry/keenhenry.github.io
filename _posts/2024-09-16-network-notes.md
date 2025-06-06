---
layout: post
title:  Notes on Computer Network
date:   2024-09-01
categories: [technology, computer network, notes]
tags: [technology, computer network, notes]
description: Notes of basic computer network knowledge
---

## Network Models - OSI model and more

OSI stands for **Open System Interconnection**. OSI model has 7 layers. However, the more commonly (arguably)
used model is the 'five-layer' model[^tcp-ip-model], which includes the **physical layer** (layer 1), data
**link layer** (layer 2), **network layer** (layer 3), **transport layer** (layer 4) and **application
layer** (layer 5). OSI model adds two additional layers in between the transport layer and the application layer,
namely the **session layer** and the **presentation layer**.


### Encapsulation and Decapsulation

The key takeaway with the most of the popular network models is how data flows among the layers. Let's only use
the five-layer model to explain.

Some terminology first:

PDU
: Protocol Data Unit - each protocol defines a unit of data that belongs to it.


When an application (say, app `A`) in the application layer needs to send data to a service (say, service `S`)
running on the other node, here is what happens:

1. The data service `A` is sending gets **encapsulated** in the layers below. First, data
   (application layer PDU) is being taken as the payload of the **segment**s (transport layer PDU)
   in the transport layer when assembling transport layer protocol data (with protocols
   like TCP, UDP, etc.). Then the same thing is done in the network layer to wrap transport layer
   segments in the payload of the **packet**s (network layer PDU) according to some network layer protocol
   (a typical protocol is **IP**). This continues until (and including) the data link layer. In the end,
   **frame**s (link layer PDU) are assembled (according to protocols like Ethernet) before sending out
   on the network.
2. On the other side of the receiving end, the node where the service `S` is running receives the
   data from the bottom - the physical layer. Then it strips away data layer by layer until eventually
   reaching the top layer - the application layer - and hence reaching service `S`. The way the
   system strips away data is by examining the headers of the PDU and removing the header that belongs
   to the layer below it to extract only the payload, which is then the PDU that belongs to the current
   layer. This process of _unpacking_ data is called **decapsulation**.


Side notes: in a communication between two parties, we need a way of identifying entities in this
communication. In the computer networking world, we use _addresses_ to do so. To identify which application
is talking to what service on two ends, `port`s are used in the transport layer. Similarly, for the same
communication, `IP`s are used in the network layer to identify the nodes/computers where the application
and the service are running. That's why source and destination IP addresses are being put in the header
of a packet. Likewise, `MAC` addresses are used in the link layer to identify network interfaces on a network.

Here is a nice video about network models and exactly what I tried to explain 😃

{% include embed/youtube.html id='3b_TAYtzuho' %}


## Network Configuration

There are four necessary settings needed to be configured for a 'node' to run correctly on a network:

- IP address (can be configured via DHCP)
- Subnet mask (can be configured via `ip` command on Linux temporarily or edit `/etc/network/interfaces` file for permanent change)
- (Default) Gateway for a host (can also be configured via `ip` command for temporary change; for persistent change, edit certain system files, like `/etc/netplan/01-network-manager-all.yaml` in Ubuntu, different OSes use different files)
- DNS server to use, see [this post](/posts/dns-notes/) for more details

## Network Devices

### What is a Hub (rarely used today)

A hub is a **physical layer** (**layer 1**) device that allows for connections from many computers
at once. All the devices connected to a hub will end up talking to all other devices at the same time.
The devices connected to the hub will need to determine whether the message was
meant for them or ignore it if not. This way of communication creates lots of noise on the
network. In addition, only one device can talk at a given time, otherwise, the electric pulses
sent over the wires can interfere with each other. This is also known as a _Collision Domain_.

**collision domain**
: A network segment where only one device can communicate at a time.


### What is a Switch (a.k.a. switching hub)

It is like a hub that can connect many network devices for them to communicate with each other.
However, a switch is a **(data) link layer** (**layer 2**) device, which means it can inspect
the contents of ethernet protocol data and determine which system the data was intended for and
only send that data to that one system.

This reduces a lot of noise and eliminates a lot of collision domains on a network! This makes
a network faster (than a network connected by a hub!).


### What is a Router

A device that forwards traffic depending on the destination address of that traffic. It is a
**network layer** (**layer 3**) device. While hubs and switches connect devices within a single
network, a.k.a LAN (Local Area Network), a router connects many LANs; it allows communication
among different LANs.

A router can inspect IP data to determine where to forward the data to. A router keeps a routing
table that keeps track of a bunch of IPs and their corresponding destinations. Routers themselves
share data with each other via a protocol called **BGP** (**Border Gateway Protocol**). This protocol
lets them learn about the most optimal paths to forward traffic.

A router has at least *TWO* network interfaces (two IPs!) since it has to connect to more than one
network to do its jobs.


### What is a Firewall

A device that blocks traffic that meets certain criteria. It can stop traffic you don't want entering
a network. Firewalls can operate at lots of different layers of the network. But it is most commonly
used at the **transport layer**. A typical firewall has a configuration that allows traffic for certain
ports while blocking traffic to other ports.

While a firewall can be a physical device that does the job, many firewalls are usually just a program
running on some devices or computers. For example, many routers have firewall programs built-in as well.


### What is a NAT

_Network Address Translation_ takes one IP address and translates it to another. To put it more technically,
it is a technology that allows a gateway, usually a router or firewall, to _rewrite_ the source IP of an
outgoing IP datagram while retaining the original IP to rewrite it into the response coming back.
In other words, the gateway is hiding the IP of the original sender from the networks beyond the gateway,
this is also called **IP masquerading**. In addition, the outbound source IP is usually replaced by the IP
(on the outbound network) of the gateway itself, this way, the IPs of the devices behind the gateway are
unknown to outside world. This is also called one-to-many NAT, where one NAT address corresponds
to many IP addresses behind it.

NAT exists for practical reasons. One is for security reasons, the other is to preserve the limited amount of
IPv4 addresses. It is not a protocol or standard, it is a technique that each OS vendor implements a bit
differently.

One thing to keep in mind is that NAT can be implemented in various layers of the network model, it does not
have to be always in the _network layer_. Many NATs also operate in the transport layer. For a NAT
to operate in the transport layer, it uses a technique called `port preservation` so that it knows which computer
to direct the response/traffic back.

Port Preservation
: A technique where the source port chosen by a client is the same port used by the router. This port is
  then used by the router to direct traffic back to the right computer (that sits behind the NAT).

Of course, it is still possible that the same ephemeral ports are chosen by different applications running
on different computers in the network, in that case, the router chooses random _unused_ ephermeral
ports for the original clients to avoid conflicts in ports. The information about the ports of the
applications behind the NAT is stored in a table in the router/NAT.


In connection with port preservation and NAT, there is another important technique called `port forwarding`.

Port Forwarding
: a technique where specific destination ports can be configured to always be delivered to specific nodes.

This technique allows the server/service to hide its IP, this is IP masquerading for the server side.
To elaborate a bit more with an example, let's say in a NAT/gateway/router (with IP `192.168.1.1`),
port `80` is configured to be always forwarding traffic to an IP (say `10.10.5.100`) where a web server
is running. Any client that sends traffic to port `80` with destination IP `192.168.1.1` is forwarded
to the web server. This way, the client does not need to know what the IP of the web server is, it only
needs to know the external IP of the router/gateway. When the web server responds, it will have the source
IP rewritten as the external IP of the router, therefore, the IP of the server is being masqueraded.

One benefit of using IP masquerading and port forwarding is they simplify how external users might
interact with lots of services all run by the same organization. Let's imagine, an external user
wants to access services (for example, a web service and a mail service) running by an organization.
Because of port forwarding, this user now only needs to know ONE DNS name to access all the services
running by the organization, because the destination ports are different for different services.

Once you understand how port forwarding works, you can then understand how [_SSH tunneling_][ssh-tunnel]
or _SSH port forwarding_ works.


### What is a Bridge

A bridge is a network device that is used to connect multiple LANs to a larger LAN. The mechanism of
network aggregation is known as _bridging_. It is a physical or hardware device that operates
at the **data link layer** (layer 2). Put it a bit differently, a bridge in computer networks
is used to divide network connections into sections, now each section has a separate collision
domain.


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
  Protocol**, an IP is assigned to a device when a device is connected to a network.
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
is all `0`'s. When doing an _bitwise logical_ `AND` operation of an IP address and its subnet mask,
you will get the subnet ID (or just network ID after CIDR was introduced), the ID of the network this
IP belongs to. For example:

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
variable-length subnet masking. This allows finer control of the sizes of subnetworks, avoiding allocating
larger subnets than needed.

CIDR notation is a compact representation of an IP address and its associated network mask. Use
the example above again (`192.168.5.85` with a `255.255.255.0` subnet mask); we can write its CIDR
notation like this: `192.168.5.95/24`.


#### Network ID

This is the ID to identify a _network_ on the internet or intranet. Back in the old days, when classful networks
were still in use, the network ID of an IP address was the most significant 1 byte (for **class A**), 2 bytes
(**class B**) or 3 bytes (**class C**) in the IP address. After CIDR was introduced, a network ID is of _variable
length_ and defined by the subnet mask.


### IPv6

An IPv6 address is a `128`-bit number. It is usually written out as 8 groups of 16 bits each.
Each one of these groups is represented as 4 hexadecimal numbers. This is a very long number,
so IPv6 has a notation method that further shortens the number. Here are two shortening rules:

- Any leading zeros are removed from a group.
- Any number of consecutive groups composed of just zeros can be replaced with two colons.

For example, the original IPv6 loopback address should be like this:

`0000:0000:0000:0000:0000:0000:0000:0001`

Applying the first rule, you get:

`0:0:0:0:0:0:0:1`

Applying the second rule, you get:

`::1`

An IPv6 address also has a network portion and host ID portion just like an IPv4 address. And
CIDR subnetting also works the same way; the only difference is the subnet mask is a bigger
number in IPv6!


### Routing Concepts

How does routing work (in a router)?

1. A router receives a packet.
2. It examines the destination IP in that packet. (It strips away Ethernet frame data and extracts only the IP datagram
   that was encapsulated inside it; this is decapsulation!)
4. It looks up the destination network of this IP in its routing table.
5. It forwards traffic to the interface that's closest to the remote network
   as determined by the additional information in the routing table.

These steps are repeated as often as needed until the traffic reaches its destination. Also note that a router itself
also has an ARP table, so it can look up MAC addresses of IPs that are directly connected to it and set destination MAC
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

What is the command to check the content of the local routing table?

```bash
$ ip route

# Or
$ netstat -rn

# Or
$ route -n
```


#### Routing Protocols

Routing protocols are used by routers to learn the world around them (so that they have the most up-to-date information about
the best routes/paths for packets). These protocols define how routers should speak to each other and share information with each other.

There are two categories of routing protocols:

1. **Interior Gateway Protocols**
2. **Exterior Gateway Protocols**


##### Interior Gateway Protocols

Interior gateway protocols are used by routers to share information within a single autonomous system. In networking terms, an autonomous
system is a collection of networks that all fall under the control of a single network operator.

An example of an autonomous system is a large corporation that needs to route data between their many offices, each of which might have their
own local area network.

Another example of an autonomous system is an Internet service provider (whose reaches are national in scale) that employs many routers.

There are two types of Interior Gateway Protocols:

- **Link State Routing protocols**
- **Distance Vector protocols** (older, outdated protocols, slower routing information propagation)

Nowadays, **Link State Routing protocols** are used among routers.


##### Exterior Gateway Protocols

Exterior Gateway Protocols are used by routers for the exchange of information between independent autonomous system**s**.
To be more precise, Exterior Gateway Protocols are used to communicate data between routers representing the edges of an
autonomous system; routers use exterior gateway protocols when they need to share information across different organizations.

Exterior gateway protocols are the key to the Internet today. They are usually the concerns of **core Internet routers**.
Getting data to the _edge router_ of an autonomous system is the number one goal of **core Internet routers**.

For the core Internet routers, autonomous systems are known and defined. Each autonomous network has an **Autonomous System
Number** (ASN) assigned to it by IANA (Internet Assigned Numbers Authority; also the authority that assigns IP addresses!).
This information is kept in the core Internet routers to identify autonomous networks.

Nowadays, [**Border Gateway Protocol**][bgp] is the most commonly used Exterior Gateway Protocol.


#### Non-Routable Address Space

These are ranges of IPs set aside for use by anyone that cannot be routed to.

The reason for the creation of such address space was that IPv4 simply couldn't provide enough
IP addresses for all networked devices on earth, so one mitigation was to make certain IP ranges
**reusable** by many organizations or individuals _internally_, as long as these IP ranges are not
exposed on the internet (and hence no IP address conflicts). And for the devices having non-routable
IP addresses, they will use a **NAT** to translate their addresses to something that's routable
on the internet! Now, the details:

_RFC 1918_ defined three ranges of IP addresses that will never be routed anywhere by **core** routers:

- **10.0.0.0/8**
- **172.16.0.0/12**
- **192.168.0.0/16**

These ranges are free for anyone to use for their internal networks! Note that interior gateway protocols
_will_ route these address spaces, but exterior gateway protocols _will not_. They're only appropriate for use within
an autonomous system.

Do they look familiar? Exactly, your IP in the SOHO network and IPs in your offices and Docker networks are usually falling
into these ranges!

### Transport and Application Layers

The first three layers of the OSI model define how each individual node sends data to each other. But it does not mean much to
humans. What we humans actually need is '_computer programs_/_applications_' running on computers to be able to talk to each
other.

The transport layer protocols allow traffic to be directed to specific network applications, and the application layer allows
these applications to communicate in a way they understand.

Transport layer has the ability to multiplex and demultiplex. This makes it unique compared to all other layers. The transport
layer handles multiplexing and demultiplexing through **port**s. Different services run on a computer while listening on
specific ports for incoming requests.

Multiplex
: Nodes on a network can direct traffic to many different receiving services.

Demultiplex
: Taking traffic that is directing at the same node to the proper receiving services (think about different processes
  running on the same node).

Port
: A port is a 16-bit number that's used to direct traffic to specific services running on a networked computer.


#### TCP vs. UDP

TCP and UDP are transport layer protocols. They have different characteristics and have different use cases.

**TCP**

0. Connection-oriented protocol. Data is resent if needed when data is not acknowledged/lost. The information,
   like the **sequence number** in a TCP segment, is key to this capability.
1. Suitable for reliable data transfer.
2. Unicast communication.
3. Used by application layer protocols like `HTTP`, `FTP`, `Telnet`, `SMTP` etc.


**UDP**

0. Connectionless protocol.
1. Best effort data transfer: fast but not guaranteed delivery.
2. Can be used in _unicast_, _multicast_ and _broadcast_ communication.
3. Used by application layer protocols like VoIP, DNS, NFS, live streaming, video chats, online gaming etc.


Socket
: The instantiation of an endpoint in a potential TCP connection. TCP sockets require actual
  programs to instantiate them. Sockets and ports are different things. Ports are more virtual;
  a program needs to open a socket on a port to receive/send data to the other end.


**TCP socket states**

`LISTEN`
: A TCP socket is ready and listening for incoming connections. Only visible on the server side.


`SYNC_SENT`
: A synchronization request has been sent, but the connection has not been established yet. This
  is only visible on the client side.


`SYNC_RECEIVED`
: A socket previously in a `LISTEN` state has received a synchronization request and sent a `SYN/ACK`
  back. But it hasn't received the final ACK from the client yet. This is only visible on the server
  side.


`ESTABLISHED`
: The TCP connection is in working order, and both sides are free to send each other data. This is
  visible for both the client and server sides of the connection.


`FIN_WAIT`
: A `FIN` has been sent, but the corresponding `ACK` from the other end hasn't been received yet. This
  is also visible for both the client and server sides.


`CLOSE_WAIT`
: The connection has been closed at the TCP layer, but the application that opened the socket hasn't
  released its hold on the socket yet. Visible for both the client and server sides.


`CLOSED`
: The connection has been fully terminated, and no further communication is possible. Visible for both
  the client and server sides.

When troubleshooting the TCP layer protocol problems, you will need to understand the meaning of these
socket states. The definitions are not defined by the TCP protocol itself; instead, they are different
per OS vendor. You should check the documentation of the socket state definitions from the OS you're
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

### DHCP

Dynamic Host Configuration Protocol. This is an _application layer_ protocol. The basic idea is
whenever a device is connected to a network, it requests/queries the DHCP server to ask for an
IP that is available on the network to use that IP. So, DHCP actually follows the client-server
way of communication.

There are three ways of IP allocation in DHCP:

Dynamic Allocation
: The most common way of allocation. A range of IP addresses is set aside for client devices.
  One of these IPs is issued to those devices when they request one.

Automatic Allocation
: Same as dynamic allocation, the only difference is that in this mode the DHCP server also keeps
  track of what IPs were assigned to which devices, and tries to assign the same IP to the same
  device each time they are connected, if possible.

Fixed Allocation
: a manually specified list of MAC addresses and their corresponding IPs is configured.
  IP allocation is according to this list. When the MAC address of a device cannot be found
  in this list, the DHCP server then falls back to automatic allocation or dynamic allocation or
  even refuses to allocate at all.

DHCP is usually used to assign IPs to devices, including computers, laptops, mobile devices
and also primary gateways. In addition to this function, it can also be used to assign other
things on the network, like NTP (Network Time Protocol) servers.

#### How it works

1. DHCP discovery
  a. A **DHCP discover message** is sent by the client without an IP (`0.0.0.0:68`) to the broadcast address `255.255.255.255:67`.
  b. The DHCP server responds (`source IP:port` = `DHCP server IP:67`) with a **DHCP offer message** also to
     the broadcast address `255.255.255.255:67`. This offer message contains the IP address that the
     server offers for the assignment.
  c. The DHCP client (`0.0.0.0:68`) then sent a **DHCP request message** to the DHCP server (`255.255.255.255:67`)
     to accept the offer.
  d. The DHCP server again responds (`destination IP:port` = `255.255.255.255:68`; and the MAC address of the client
     is included in one of the fields in the message, so the client knows this is intended for itself) with a
     **DHCP ACK message**.
2. DHCP client now uses the information provided by the DHCP server to configure its network stack.
   **DHCP lease** is now complete. DHCP lease has an expiration time, when the lease is expired or is
   released by the client, the assigned IP can then be returned to the pool of available IPs.


### VPN

Virtual Private Network is a technology that allows for the extension of a private or local network
to hosts that might not work on that same local network.

This is not a protocol or standard, so each vendor implements this technology differently.

VPN is a tunneling technique, similar to [SSH tunneling][ssh-tunnel] but more complex and robust.
It provisions access to something not locally available.

How things work:

> A VPN client establishes a VPN tunnel (an encrypted channel) to a private network. This would provision
> the client computer with what's known as a _virtual interface_ with an IP that matches the address space
> of the network they've established a VPN connection to. By sending data out of this virtual interface,
> the computer could access internal resources just like if it was physically connected to the private
> network.

> Most VPNs work by using the payload section of the transport layer to carry an encrypted payload that
> actually contains an entire second set of packets, the network, the transport, and the application layers
> of a packet intended to traverse the remote network. This payload is carried to the VPN's endpoint,
> where all the other layers are stripped away and discarded. Then the payload is unencrypted, leaving
> the VPN server with the top three layers of a new packet. This gets encapsulated with the proper data
> link layer information and sent out across the network. This process is completed in the inverse, in the
> opposite direction.


### Wi-Fi and Cellular Network

One important concept regarding wireless networks is that both Wi-Fi (`802.11`) and cellular/mobile networks
are operating over *radio waves*, but they operate in different _frequencies_.

Because they are operating in radio waves, that means they can be interfered with by other radio waves if other
radio waves happen to be in the same range of frequencies.

Wi-Fi is basically a **link layer** protocol.


## Troubleshooting Network


### Testing Connectivity

1. Check connectivity:

```bash
$ ping <ip or FQDN>
```

`ping` uses ICMP protocol for probing connectivity issues.

2. Check the path along the route:

```bash
$ traceroute <FQDN or ip>
$ mtr <ip or domain name>  # Long running traceroute!
```

3. Test port connectivity (transport layer connectivity!)

```bash
# `nc` tries to establish a connection with `google.com`; if the connection fails, the command will exit
$ nc google.com 80  # netcat!

# Not sending data (only port scanning!) and in verbose mode
$ nc -z -v google.com 80
Connection to google.com (2a00:1450:400e:801::200e) 80 port [tcp/http] succeeded!
```


### Digging DNS

```bash
# interactive mode
$ nslookup
>

# non-interactive mode
$ nslookup twitter.com
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   twitter.com
Address: 104.244.42.193
Name:   twitter.com
Address: 104.244.42.1
Name:   twitter.com
Address: 104.244.42.129
Name:   twitter.com
Address: 104.244.42.65

# Similar to nslookup
$ dig <domain name>
```

If your own DNS server is not functioning properly, you might want to consider using some publicly
available DNS service (for free) provided by certain organizations for troubleshooting or more
permanently. Most public DNS servers are available globally through **anycast**.

#### Level 3 Communications

- `4.2.2.1`
- `4.2.2.2`
- `4.2.2.3`
- `4.2.2.4`
- `4.2.2.5`
- `4.2.2.6`

#### Google

- `8.8.8.8`
- `8.8.4.4`

Most of these servers also respond to ICMP echo requests, so they're also a great choice for testing
general Internet connectivity with `ping`.


#### Host Files

Since DNS is now everywhere, host files are not used much anymore. They are old technology, but
they're functioning as they were before. Most modern operating systems still support the use of
host files. A host file is simply a plain text file with lines mapping IP addresses to (human-readable)
host names.

Host files are being _evaluated_/examined by the network stack of operating systems before a DNS
resolution attempt occurs. In other words, you can use a host file to force an individual computer to
think a certain domain name always points to a certain IP address (like when troubleshooting network
problems). The reason why host files still exist today is because of the `loopback` address, which
is always present in a host file.

On Linux, it is common to see entries in `/etc/hosts` file like the following:

```text
127.0.0.1       localhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
```

That's why in your own browser, when you type `localhost` in the address bar, it gets translated to
`127.0.0.1` which is the local address of the machine the web browser is running on! `::1` is the
loopback address for IPv6!


## Footnotes

[^tcp-ip-model]: which is a model very similar to the [**TCP/IP model**][tcp-model].
                 The TCP/IP model is also used a lot in practice. Here are some [comparisons][vs] between the OSI
                 model and the TCP/IP model.


[bgp]: https://en.wikipedia.org/wiki/Border_Gateway_Protocol
[tcp-model]: https://simple.wikipedia.org/wiki/TCP/IP_model
[vs]: https://www.geeksforgeeks.org/difference-between-osi-model-and-tcp-ip-model/
[ssh-tunnel]: https://linuxize.com/post/how-to-setup-ssh-tunneling/
