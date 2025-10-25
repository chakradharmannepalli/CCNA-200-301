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
  <img width="1137" height="591" alt="image" src="https://github.com/user-attachments/assets/28c41997-cb17-4e8b-8ac1-93506427c8fe" />
  <img width="1139" height="591" alt="image" src="https://github.com/user-attachments/assets/8eada14d-204e-4ac8-b076-aba8824692de" />


- **Routers** are used to connect separate LANs.

### 🧱 Ethernet Frame Structure
Ethernet frame = Header + Data (Packet) + Trailer

<img width="1136" height="606" alt="image" src="https://github.com/user-attachments/assets/47f911a2-c6e2-4ccb-9096-1a57b3b424d0" />

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

<img width="1131" height="728" alt="image" src="https://github.com/user-attachments/assets/bb4d3f6e-47b9-451f-85bf-792c3a10e304" />

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

<img width="1132" height="727" alt="image" src="https://github.com/user-attachments/assets/76e9e626-d359-4f9a-9a5f-5d4f5d83a6aa" />

### 🗂️ MAC Address Table (Switch Logic)
- Switches dynamically learn MAC addresses from source address of incoming frames.
- **Forwarding:** If destination MAC known → send to specific port.
- **Flooding:** If unknown → send to all ports except source.
- **Aging:** Dynamic MAC removed after 5 mins inactivity.

 <img width="1134" height="673" alt="image" src="https://github.com/user-attachments/assets/92cc9755-e25e-4494-b377-623c08c1deda" />
 <img width="1133" height="642" alt="image" src="https://github.com/user-attachments/assets/0a028883-45a1-4925-9ad3-5c55a5913210" />


## 📘 Part 2
### 🧱 Ethernet Frame Recap
- Header: 14 bytes (Dest + Src + Type/Length)
- Data: 46–1500 bytes
- Trailer: 4 bytes (FCS)
- Minimum frame size: 64 bytes
- Padding added if data < 46 bytes
<img width="1134" height="319" alt="image" src="https://github.com/user-attachments/assets/6f48eea8-98ad-4c1e-bea1-83c49f942819" />
### Header + Trailer Size

- **Header + Trailer** = 18 bytes  
  - 6 bytes (Destination) + 6 bytes (Source) + 2 bytes (Type/Length) + 4 bytes (FCS)

### Minimum Ethernet Frame Size

- Minimum Ethernet frame = 64 bytes (Header + Payload + Trailer)  
- Minimum data payload (packet) size:
        - 64 bytes (min frame) - 18 bytes (header + trailer) = 46 bytes
  > If the payload is **less than 46 bytes**, padding bytes (series of 0s) are added until it reaches 46 bytes.


### 🌐 ARP (Address Resolution Protocol)
- Used to find MAC address of a known IP
- ARP Request: Broadcast (FF:FF:FF:FF:FF:FF)
- ARP Reply: Unicast to requester

<img width="1134" height="636" alt="image" src="https://github.com/user-attachments/assets/be15a2d4-ebb2-4606-9734-bda8cb338a0c" />
<img width="1135" height="637" alt="image" src="https://github.com/user-attachments/assets/88d387d7-676c-421e-9bb7-47112b8c514f" />
<img width="1128" height="593" alt="image" src="https://github.com/user-attachments/assets/e9cd7795-3569-472c-84f5-1b7838e440f4" />


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
| clear mac address-table dynamic address <mac-address | Clear MACs for a specific MAC address |
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
