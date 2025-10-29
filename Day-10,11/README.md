# IPv4 Header and Routing Fundamentals

This repository contains notes and explanations on key networking concepts, focusing on the IPv4 Header structure and Routing Fundamentals. These materials are designed for educational purposes, covering Layer 3 (Network Layer) protocols and routing mechanisms in IP networks.

## Table of Contents
- [IPv4 Header](#ipv4-header)
  - [Overview](#overview)
  - [Header Fields](#header-fields)
- [Routing Fundamentals](#routing-fundamentals)
  - [What is Routing?](#what-is-routing)
  - [Static Routing](#static-routing)
    - [Configuration Examples](#configuration-examples)
    - [Default Route](#default-route)
- [References](#references)

## IPv4 Header

### Overview
The **Internet Protocol version 4 Header (IPv4 Header)** is used at **Layer 3** (Network Layer) to facilitate routing—sending data between devices on separate networks, even across the globe via the Internet.

The IPv4 Header **encapsulates** a TCP or UDP segment (Layer 4 PDU). It enables packet forwarding across routers.

**Key Review Points:**
- Minimum Header Length: 20 bytes
- Maximum Header Length: 60 bytes
- Total Packet Size: Up to 65,535 bytes (header + data)

### Header Fields
The IPv4 Header consists of fixed and variable fields. Below is a summary table of all fields:

| Field                  | # of Bits | Description |
|------------------------|-----------|-------------|
| **Version**           | 4        | Identifies the IP version (IPv4 = 0100 binary / Decimal 4; IPv6 = 0110 binary / Decimal 6). |
| **IHL (Internet Header Length)** | 4 | Specifies header length in 4-byte increments. Minimum: 5 (20 bytes, no options). Maximum: 15 (60 bytes). |
| **DSCP (Differentiated Services Code Point)** | 6 | Used for Quality of Service (QoS) to prioritize delay-sensitive traffic (e.g., voice/video streaming). |
| **ECN (Explicit Congestion Notification)** | 2 | Notifies endpoints of network congestion without dropping packets. Requires support from endpoints and infrastructure. |
| **Total Length**      | 16       | Total packet size in bytes (header + data). Minimum: 20 bytes. Maximum: 65,535 bytes (2^16 - 1). |
| **Identification**    | 16       | Identifies fragments of a larger packet for reassembly. Same value across all fragments of one packet. |
| **Flags**             | 3        | Controls fragmentation:<br>- Bit 0: Reserved (always 0).<br>- Bit 1: Don't Fragment (DF) flag.<br>- Bit 2: More Fragments (MF) flag (1 if more fragments; 0 for last/no fragments). |
| **Fragment Offset**   | 13       | Position of this fragment in the original packet (in 8-byte units). Enables out-of-order reassembly. |
| **Time to Live (TTL)** | 8       | Prevents infinite loops by decrementing per hop (router). Drop if TTL=0. Default: 64. Originally in seconds, now hop count. |
| **Protocol**          | 8        | Identifies encapsulated Layer 4 protocol:<br>- 1: ICMP<br>- 6: TCP<br>- 17: UDP<br>- 89: OSPF<br>(See [List of IP Protocol Numbers](https://en.wikipedia.org/wiki/List_of_IP_protocol_numbers) for more.) |
| **Header Checksum**   | 16       | Error-checking for header only. Recalculated per hop. Data errors handled by encapsulated protocol (e.g., TCP/UDP checksums). |
| **Source Address**    | 32       | IPv4 address of the sender. |
| **Destination Address** | 32     | IPv4 address of the receiver. |
| **Options**           | 0-320    | Variable (rarely used). Present if IHL > 5. Padded to 32-bit boundary. |

**Fragmentation Notes:**
- Occurs if packet > MTU (typically 1500 bytes for Ethernet).
- Fragments are reassembled at the destination host.

## Routing Fundamentals

### What is Routing?
**Routing** is the process routers use to determine the path IP packets take across networks to reach their destination.

- **Routers** maintain a **Routing Table** with known destinations.
- Upon receiving a packet, the router consults the table to forward it via the best path.
- **WAN (Wide Area Network)**: Networks spanning large geographic areas (e.g., internet connections between cities).

**Routing Methods:**
- **Dynamic Routing**: Routers automatically exchange info using protocols like OSPF to build tables.
- **Static Routing**: Manually configured by admins (no auto-updates).

A **route** instructs the router:
- Forward to **next-hop** IP for destination X.
- Send directly if destination is **directly connected**.
- Process locally if destination is the router's own IP.

**Review: Switching vs. Routing**
- **Switches**: Forward within LANs (Layer 2).
- **Routers**: Forward between LANs/WANs (Layer 3).

### Static Routing

Static routes are manually added for simplicity in small/stable networks. They don't adapt to changes (e.g., link failures).

#### Configuration Examples
Static routes can specify a **next-hop IP** or an **exit interface**.

**Basic Static Route (Next-Hop):** `ip route <destination_network> <subnet_mask> <next_hop_ip>`

Example: Route to 192.168.2.0/24 via next-hop 10.0.0.2
  ```
    ip route 192.168.2.0 255.255.255.0 10.0.0.2
  ```

**Static Route with Exit Interface:** `ip route <destination_network> <subnet_mask> <exit_interface>`

Example: Route to 192.168.2.0/24 via GigabitEthernet0/1
  ```
    ip route 192.168.2.0 255.255.255.0 GigabitEthernet0/1
    or
    ip route 192.168.2.0 255.255.255.0 g0/1
  ```

