---
layout: post
title:  Computer Network Notes
date:   2024-09-01
categories: [technology, computer network, notes]
tags: [technology, computer network, notes]
description: Study notes of computer network technology
---


## Network Configuration

There are four things that need to be specifically configured for a 'node' to run on a network:

- IP address (can be configured via DHCP)
- Subnet mask
- Gateway for a host
- DNS server to use


## What is a Switch
## What is an IP address

- First of all, this is a Network Layer concept.
- IP addresses belong to networks, not to the devices attached to those networks.
- **Dynamic IP addresses** are assigned to a device automatically when that device
  is connected to a network. This is done via a protocol called DHCP (Dynamic Host
  Configuration Protocol).
- **Static IP addresses**, on the other hand, need to be configured manually on a node.

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
## What is a NAT
### NAT Masquerading
## What is a Bridge
## Network ID