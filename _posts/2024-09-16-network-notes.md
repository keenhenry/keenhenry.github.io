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


## What is a Hub (rarely used today)

A hub is a **physical layer** (**layer 1**) device that allows for connections from many computers
at once. All the devices connected to a hub will end up talking to all other devices at the same time.
The devices connected to the hub will need to determine themselves whether the message was
meant for them or ignore if it's not. This way of communication creates lots of noises on the
network. In addition, only one device can talk at a given time, otherwise the electric pulses
sent over the wires can interfere with each other . This is also known as a _Collision Domain_.

**collision domain**
: A network segment where only one device can communicate at a time.


## What is a Switch (a.k.a. switching hub)

It is like a hub which can connect many network devices for them to communicate with each other.
However, a switch is a **(data) link layer** (**layer 2**) device, which means it can inspect
the contents of ethernet protocol data and determine which system the data was intended for and
only send that data to that one system.

This reduces a lot of noises and eliminates a lot of collision domains on a network! This makes
a network faster (than a network connected by a hub!).


## What is an IP address

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


## Subnetting

- Subnetting is the process of splitting up a larger network into smaller subnetworks.
- Why we need subnets? A gateway router cannot keep too many IPs with it, it's just not
  practical, that's why we need subnetworks or subnets to split things into smaller chunks
  and let smaller gateway routers to do forwarding jobs.


### Subnet masks

- How does this work?

### CIDR (Classless Inter-Domain Routing)

## ARP protocol, ARP table

- how does it work
- what is it used for?
- What linux command to use to check such information?
- Which layer protocol it is?


## What is a Router

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
## Network ID