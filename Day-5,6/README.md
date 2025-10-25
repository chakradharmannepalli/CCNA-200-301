# 🖧 Ethernet LAN Switching

## 📚 Table of Contents
1. [Part 1](#-part-1)
   - [What is a LAN?](#-what-is-a-lan)
   - [Ethernet Frame Structure](#-ethernet-frame-structure)
   - [Ethernet Header Fields](#️-ethernet-header-fields)
   - [MAC Address (48 bits)](#-mac-address-48-bits)
   - [Type / Length Field](#-type--length-field)
   - [Ethernet Trailer (FCS)](#-ethernet-trailer-fcs)
   - [Ethernet Frame Summary](#-ethernet-frame-summary)
   - [MAC Address Table (Switch Logic)](#-mac-address-table-switch-logic)
2. [Part 2](#-part-2)
   - [Ethernet Frame Recap](#-ethernet-frame-recap)
   - [ARP (Address Resolution Protocol)](#-arp-address-resolution-protocol)
   - [PING (ICMP Echo)](#-ping-icmp-echo)
   - [Useful Cisco IOS Commands](#-useful-cisco-ios-commands)
   - [Interface Naming](#-interface-naming)
   - [Summary](#-summary)
## 📘 Part 1
### 🌐 What is a LAN?
- A **LAN (Local Area Network)** is a network contained within a relatively small area.
- **Routers** are used to connect separate LANs.

### 🧱 Ethernet Frame Structure
Ethernet frame = Header + Data (Packet) + Trailer

### ⚙️ Ethernet Header Fields
| **Field** | **Length** | **Description** |
|------------|------------|------------------|
| **Preamble** | 7 bytes (56 bits) | Alternating 1s and 0s; used to synchronize devices. |
| **Start Frame Delimiter (SFD)** | 1 byte (8 bits) | Marks the start of the frame. |
| **Destination MAC Address** | 6 bytes (48 bits) | Receiver's MAC address. |
| **Source MAC Address** | 6 bytes (48 bits) | Sender's MAC address. |
| **Type / Length** | 2 bytes (16 bits) | Length or protocol type (e.g., IPv4). |

### 🧭 MAC Address (48 bits)
| **Attribute** | **Description** |
|----------------|------------------|
| **Length** | 6 bytes (48 bits) |
| **Format** | 12 hex chars e.g., `E8:BA:70:11:28:74` |
| **Structure** | First 3 bytes = OUI, last 3 bytes = Device ID |
| **Uniqueness** | Globally unique |
| **Type** | Burned-In Address (BIA) |

### 📦 Type / Length Field
| **Value Range** | **Meaning** | **Examples** |
|------------------|--------------|---------------|
| 0 – 1500 | Length of payload | — |
| 1536 – 65535 | Protocol type | IPv4: 0x0800, IPv6: 0x86DD |

### 🧮 Ethernet Trailer (FCS)
| **Field** | **Length** | **Description** |
|------------|------------|------------------|
| **Frame Check Sequence (FCS)** | 4 bytes (32 bits) | Error detection using CRC |

### 🧰 Ethernet Frame Summary
| **Component** | **Size** | **Description** |
|----------------|----------|------------------|
| Destination MAC | 6 bytes | Receiver’s hardware address |
| Source MAC | 6 bytes | Sender’s hardware address |
| Type / Length | 2 bytes | Protocol or length |
| Data (Payload) | 46–1500 bytes | Packet data |
| FCS | 4 bytes | CRC check |
| Total Frame Size | 64–1518 bytes | Minimum–Maximum Ethernet frame size |

### 🗂️ MAC Address Table (Switch Logic)
- Switches dynamically learn MAC addresses from source address of incoming frames.
- **Forwarding:** If destination MAC known → send to specific port.
- **Flooding:** If unknown → send to all ports except source.
- **Aging:** Dynamic MAC removed after 5 mins inactivity.

## 📘 Part 2
### 🧱 Ethernet Frame Recap
- Header: 14 bytes (Dest + Src + Type/Length)
- Data: 46–1500 bytes
- Trailer: 4 bytes (FCS)
- Minimum frame size: 64 bytes
- Padding added if data < 46 bytes

### 🌐 ARP (Address Resolution Protocol)
- Used to find MAC address of a known IP
- ARP Request: Broadcast (FF:FF:FF:FF:FF:FF)
- ARP Reply: Unicast to requester

### 📶 PING (ICMP Echo)
- Tests connectivity and measures round-trip time
- Echo Request → Echo Reply
- Cisco default: 5 packets, 100 bytes each, ! = success, . = fail

### 🧰 Useful Cisco IOS Commands
| Command | Description |
|---------|-------------|
| show arp | Show ARP table |
| show mac address-table | Show switch MAC table |
| clear mac address-table dynamic | Clear all dynamic MAC entries |
| clear mac address-table dynamic interface <iface> | Clear MACs for a specific interface |

### 🧾 Interface Naming
| Example | Meaning |
|---------|--------|
| F0/1, F0/2 | Fast Ethernet 100 Mbps |
| G0/1, G0/2 | Gigabit Ethernet 1 Gbps |

### 🧾 Summary
- LAN: Local network via switches
- Ethernet Frame: Header + Data + Trailer
- Min Frame Size: 64 bytes
- ARP: Resolves IP → MAC
- Ping: Test connectivity
- MAC Table: Dynamic learning, 5 min aging
"""
