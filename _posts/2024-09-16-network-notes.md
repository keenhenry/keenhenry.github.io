---
layout: post
title:  Computer Network Notes
date:   2024-09-01
categories: [technology, computer network, notes]
tags: [technology, computer network, notes]
description: Study notes of computer network technology
---


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

A device that forwards data in between networks. It is a **network layer** (**layer 3**) device.
While hubs and switches connects devices within a single network, a.k.a LAN (Local Area Network), a
router connects many LANs together; it allows communication between different LANs.

A router can inspects IP data to determine where to forward the data to. A router keeps a routing
table that keeps track of a bunch of IPs and their corresponding destinations. Routers themselves
share data with each other via a protocol called **BGP** (**Border Gateway Protocol**). This protocol
lets them learn about the most optimal paths to forward traffic.


## What is a NAT
### NAT Masquerading
## What is a Bridge

## Network Protocols

### Ethernet

Ethernet is a **link layer** (**layer 2**) protocol. It abstracts away the physical layer details
like what wires and hardware are used and is the protocol to resolve collisions inside a collision
domain.

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


### WIFI

WIFI is a **link layer** (**layer 2**) protocol.


### ARP protocol, ARP table

- how does it work
- what is it used for?
- What linux command to use to check such information?
- Which layer protocol it is?

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
the example above again (`192.168.5.85` with `255.255.255.0` subnet mask), we can write its CIDR notation
as follows: `192.168.5.95/24`.


### Network ID

This the ID to identify a network on internet or intranet. When classful network was still in use, a network
ID is other the most significant 1 byte (for class A), 2 bytes (class B) or 3 bytes (class C) in an IP
address. After CIDR was introduced, a network ID is of variable length, and is defined by the subnet mask.