# IPv4 Header and Routing Fundamentals

This repository contains notes and explanations on key networking concepts, focusing on the IPv4 Header structure and Routing Fundamentals. These materials are designed for educational purposes, covering Layer 3 (Network Layer) protocols and routing mechanisms in IP networks.

## Table of Contents
- [IPv4 Header](#ipv4-header)
  - [Overview](#overview)
  - [Header Fields](#header-fields)
- [Routing Fundamentals](#routing-fundamentals)
  - [What is Routing?](#what-is-routing)
  - [Viewing Routes](#viewing-routes)
  - [Route Types](#route-types)
  - [Static Routing](#static-routing)
    - [Configuration Examples](#configuration-examples)
    - [Default Route](#default-route)
- [References](#references)

## IPv4 Header

### Overview
The **Internet Protocol version 4 Header (IPv4 Header)** is used at **Layer 3** (Network Layer) to facilitate routing—sending data between devices on separate networks, even across the globe via the Internet.

The IPv4 Header **encapsulates** a TCP or UDP segment (Layer 4 PDU). It enables packet forwarding across routers.

<img width="1135" height="584" alt="image" src="https://github.com/user-attachments/assets/9fa94c22-870c-456e-963b-dc082f84dd21" />

**Key Review Points:**
- Minimum Header Length: 20 bytes
- Maximum Header Length: 60 bytes
- Total Packet Size: Up to 65,535 bytes (header + data)

### Header Fields
The IPv4 Header consists of fixed and variable fields. Below is a summary table of all fields:

<img width="1132" height="353" alt="image" src="https://github.com/user-attachments/assets/a83ec406-79d4-4aaa-ae66-8e908f5af864" />


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

<img width="1136" height="590" alt="image" src="https://github.com/user-attachments/assets/3f125c27-b92d-48ef-9c05-922cbd2cbb9f" />


- **Routers** maintain a **Routing Table** with known destinations.
- Upon receiving a packet, the router consults the table to forward it via the best path.
- **WAN (Wide Area Network)**: Networks spanning large geographic areas (e.g., internet connections between cities).

  <img width="1131" height="245" alt="image" src="https://github.com/user-attachments/assets/d06d099a-4a2e-41e6-84e8-41df78ed07a8" />

  <img width="1136" height="445" alt="image" src="https://github.com/user-attachments/assets/37ed821d-5a35-4339-8450-f05920268929" />

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

### Viewing Routes
To inspect the routing table on a Cisco router (or similar device), use the command:

  ```
    show ip route
  ```

<img width="1140" height="466" alt="image" src="https://github.com/user-attachments/assets/51c1193d-8259-4d56-8cf4-99fa03eebeb7" />


This displays all routes, including codes for route types (e.g., 'C' for connected, 'S' for static). Example output snippet:
  ```
    C    192.168.1.0/24 is directly connected, GigabitEthernet0/0
    S    10.0.0.0/24 [1/0] via 192.168.1.1
  ```

<img width="1133" height="613" alt="image" src="https://github.com/user-attachments/assets/b335b138-39d5-4923-861f-3ac8fe803a71" />


<img width="1136" height="642" alt="image" src="https://github.com/user-attachments/assets/a1ab0302-7fd4-40b3-9361-97fb3f0be659" />


- Interpret codes: 'C' (Connected), 'S' (Static), 'L' (Local), etc.
- Use `show ip route <network>` for specific routes.

### Route Types
Routing tables include different types of routes, each with distinct purposes and administrative distances (AD) for path selection (lower AD preferred):

| Route Type | Code | Description | Administrative Distance (AD) | Automatic? |
|------------|------|-------------|------------------------------|------------|
| **Connected** | C | Routes to networks directly attached to the router's interfaces. Automatically added upon interface configuration and up/up status. | 0 | Yes |
| **Local** | L | Routes for the router's own IP addresses on interfaces (used for local delivery, e.g., pinging the router itself). Automatically added. | 0 | Yes |
| **Static** | S | Manually configured routes. Do not adapt to topology changes; useful for stable, small networks. | 1 (default; adjustable) | No |

- **Connected vs. Local**: Connected routes point to entire subnets (e.g., 192.168.1.0/24); Local routes are host-specific for the interface's IP (e.g., 192.168.1.1/32).
- **Static vs. Connected/Local**: Statics require manual entry and have higher AD, so they are overridden by connected/local if overlapping.

### Static Routing

Static routes are manually added for simplicity in small/stable networks. They don't adapt to changes (e.g., link failures).

#### Configuration Examples
Static routes can specify a **next-hop IP**, an **exit interface**, or both.

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

**Static Route with Both Exit Interface and Next-Hop:** `ip route <destination_network> <subnet_mask> <exit_interface> <next_hop_ip>`

Example: Route to 192.168.2.0/24 via GigabitEthernet0/1 to next-hop 10.0.0.2
  ```
    ip route 192.168.2.0 255.255.255.0 GigabitEthernet0/1 10.0.0.2
  ```
- **Next-Hop vs. Exit Interface**:
  - **Next-Hop**: Specifies the IP address of the next router to forward the packet to. Requires ARP resolution to find the Layer 2 next-hop MAC. Useful for point-to-point or multi-access networks.
  - **Exit Interface**: Specifies the local outbound interface (e.g., port) to send the packet out. Bypasses ARP for the next-hop; directly uses the interface. Preferred for point-to-point links to avoid recursion.
  - **Both**: Combines them for explicit control (e.g., in multi-access networks); the interface must lead to the next-hop IP.

#### Default Route
A **default route** (0.0.0.0/0) forwards all unmatched traffic to a gateway (e.g., internet router).

**Configuration:** `ip route 0.0.0.0 0.0.0.0 <next_hop_ip_or_exit_interface>`

**Example:** `ip route 0.0.0.0 0.0.0.0 10.0.0.1`

This sends all non-local traffic to the default gateway at 10.0.0.1.

## References
- IANA [List of IP Protocol Numbers](https://en.wikipedia.org/wiki/List_of_IP_protocol_numbers)
- Cisco Networking Academy materials on IPv4 and Routing.
- RFC 791: Internet Protocol (IPv4 specification).
