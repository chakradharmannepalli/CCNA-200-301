# Jeremy's IT Lab - Day 7 & 8 (Cisco 200-301)

> Lab exercises and notes from **Day 7 & 8** of Jeremy's IT Lab, focusing on **IPv4 addressing, subnetting, binary/decimal conversions, host calculations, and Cisco CLI configuration**.

---

## 📊 Table of Contents
1. [Binary ↔ Decimal Conversion](#binary-↔-decimal-conversion)
2. [Decimal ↔ Binary Conversion](#decimal-↔-binary-conversion)
3. [IPv4 Addressing](#ipv4-addressing)
4. [Subnet Masks & Maximum Hosts](#subnet-masks--maximum-hosts)
5. [First & Last Usable IP Addresses](#first--last-usable-ip-addresses)
6. [Cisco CLI Configuration](#cisco-cli-configuration)
7. [References](#references)

---

## 💻 Binary ↔ Decimal Conversion

| Binary       | Decimal | ✅ |
|-------------|--------|---|
| 10001111    | 143    | ✔ |
| 01110110    | 118    | ✔ |
| 11101100    | 236    | ✔ |

> **Tip:** Sum powers of 2 where there’s a `1` in the binary position (128, 64, 32, 16, 8, 4, 2, 1).

---

## 🔄 Decimal ↔ Binary Conversion

| Decimal     | Binary     | ✅ |
|------------|-----------|---|
| 221        | 11011101  | ✔ |
| 127        | 01111111  | ✔ |
| 207        | 11001111  | ✔ |

> **Tip:** Subtract the largest possible powers of 2 from left to right to get the binary number.

---

## 🌐 IPv4 Addressing

### Binary → IPv4 Conversion

| Binary                                  | IPv4 Address       | Network Portion |
|----------------------------------------|-----------------|----------------|
| 10011010010011100110111100100000       | 154.78.111.32   | /16 ✅          |
| 00001100100000001111101100010111       | 12.128.251.23   | /8 ✅           |

### Address Classes

| Class | First Octet Range | Usage |
|-------|-----------------|-------|
| A     | 0 - 127          | Large networks |
| B     | 128 - 191        | Medium networks |
| C     | 192 - 223        | Small networks |
| D     | 224 - 239        | Multicast |
| E     | 240 - 255        | Experimental |

> Note: 127.x.x.x reserved for loopback.

---

## 🛡️ Subnet Masks & Maximum Hosts

**Formula:** `Hosts = 2^N - 2`  
*(N = number of host bits)*

| Class | Prefix | Subnet Mask       | Host Bits | Maximum Hosts |
|-------|--------|-----------------|-----------|---------------|
| C     | /24    | 255.255.255.0   | 8         | 254 ✅         |
| B     | /16    | 255.255.0.0     | 16        | 65,534 ✅      |
| A     | /8     | 255.0.0.0       | 24        | 16,777,214 ✅  |

---

## 🏁 First & Last Usable IP Addresses

| Network         | First Usable IP | Last Usable IP |
|----------------|----------------|----------------|
| 192.168.1.0/24 | 192.168.1.1 ✅  | 192.168.1.254 ✅ |
| 172.16.0.0/16  | 172.16.0.1 ✅   | 172.16.255.254 ✅ |
| 10.0.0.0/8     | 10.0.0.1 ✅     | 10.255.255.254 ✅ |

> **Tip:** Network address = all host bits `0`  
> Broadcast address = all host bits `1`  

---

## 💻 Cisco CLI Configuration

### 1️⃣ Basic Router & Switch Setup

```bash
enable                # Enter privileged mode
configure terminal    # Enter global configuration mode
hostname R1           # Set hostname (R1)
no ip domain-lookup    # Disable DNS lookup
enable secret cisco    # Set privileged EXEC password
line console 0
 password cisco
 login
 logging synchronous
exit
line vty 0 4
 password cisco
 login
 transport input ssh
exit
```
