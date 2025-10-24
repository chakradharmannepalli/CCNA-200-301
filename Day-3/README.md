# ⚙️ INTERFACES AND CABLES

## 🔌 Switch Ports
- **Switches** provide many **ports** for connectivity (usually 24).  
- These ports are typically **RJ-45 (Registered Jack)** connectors.

---

## 🌐 What is Ethernet?
**Ethernet** is a collection of **network protocols and standards** used to enable communication between devices in a local area network (LAN).

### Why Do We Need Network Protocols and Standards?
- Provide **common communication standards** for all network devices.  
- Ensure **hardware compatibility** between manufacturers.  
- Define **connection speeds**, measured in **bits per second (bps)**.

| Unit | Bits |
|------|------:|
| 1 kilobit (Kb) | 1,000 |
| 1 megabit (Mb) | 1,000,000 |
| 1 gigabit (Gb) | 1,000,000,000 |
| 1 terabit (Tb) | 1,000,000,000,000 |

> 🧠 **Note:**  
> A **bit** = 0 or 1  
> A **byte** = 8 bits

---

## 📜 Ethernet Standards (Copper)

Defined in the **IEEE 802.3** standard (1983).  
**IEEE** = *Institute of Electrical and Electronics Engineers.*

| Speed | Common Name | Standard | Cable Type | Max Distance |
|--------|--------------|-----------|-------------|---------------:|
| 10 Mbps | Ethernet | 802.3i | 10BASE-T | 100m |
| 100 Mbps | Fast Ethernet | 802.3u | 100BASE-T | 100m |
| 1 Gbps | Gigabit Ethernet | 802.3ab | 1000BASE-T | 100m |
| 10 Gbps | 10 Gigabit Ethernet | 802.3an | 10GBASE-T | 100m |

- **BASE** = Baseband Signaling  
- **T** = Twisted Pair  

---

## 🧵 Twisted Pair (Copper) Cables
- Most Ethernet uses **copper cables**.  
- **UTP (Unshielded Twisted Pair)**: no metallic shield; twists protect against **EMI (Electromagnetic Interference)**.  
- Most use **8 wires (4 pairs)**.

| Standard | Wire Pairs Used |
|-----------|----------------:|
| 10/100BASE-T | 2 pairs (4 wires) |
| 1000BASE-T / 10GBASE-T | 4 pairs (8 wires) |

---

## 🔄 Communication via Ethernet Connections

Each Ethernet cable has an **RJ-45 plug** with **8 pins**.

### 📎 RJ-45 Pin Layout
+-------------------------+
| 1 2 3 4 5 6 7 8 |
+-------------------------+
|TX TX RX -- -- RX -- -- |
+-------------------------+
Pins 1–2: Transmit (TX)
Pins 3–6: Receive (RX)


### 💻 PC ↔ Switch Connection

| Device | Transmit (TX) | Receive (RX) |
|---------|----------------|---------------|
| PC | Pins 1–2 | Pins 3–6 |
| Switch | Pins 3–6 | Pins 1–2 |

➡️ Allows **Full-Duplex Transmission**.

---

## 🧭 Cable Types

### 🔹 Straight-Through Cable
Used to connect **different** device types:
- PC ↔ Switch  
- Router ↔ Switch  
- Firewall ↔ Switch  

**Pin Mapping:**
PIN 1 → PIN 1
PIN 2 → PIN 2
PIN 3 → PIN 3
PIN 6 → PIN 6

**Diagram:**
PC (TX 1–2, RX 3–6)
| Straight Cable |
Switch (RX 1–2, TX 3–6)

---

### 🔸 Crossover Cable
Used to connect **similar** device types:
- Switch ↔ Switch  
- Router ↔ Router  
- PC ↔ PC  

**Pin Mapping:**
PIN 1 → PIN 3
PIN 2 → PIN 6
PIN 3 → PIN 1
PIN 6 → PIN 2

**Diagram:**
Device A (TX 1–2, RX 3–6)
\ Crossed /
_________/
/
Device B (TX 1–2, RX 3–6)


| Device Type | TX Pins | RX Pins |
|--------------|----------|----------|
| Router | 1, 2 | 3, 6 |
| Firewall | 1, 2 | 3, 6 |
| PC | 1, 2 | 3, 6 |
| Switch | 3, 6 | 1, 2 |

> 💡 **Auto MDI-X:**  
> Most modern devices automatically detect transmit/receive pairs and adjust, so crossover cables are rarely needed today.

---

## ⚡ High-Speed Ethernet
- **1000BASE-T** and **10GBASE-T** use **4 pairs (8 wires)**.  
- Each wire pair is **bidirectional**, allowing simultaneous transmit and receive operations for higher throughput.

---

## 💡 Fiber-Optic Connections

Defined in the **IEEE 802.3ae** standard.  
Use **SFP (Small Form-Factor Pluggable)** transceivers to connect fiber cables to switches/routers.  
Fiber connections have **separate transmit (TX)** and **receive (RX)** cables.

**Fiber Connection Layout:**
[TX Port] =====> [RX Port]
[RX Port] <===== [TX Port]


---

## 🔍 Fiber-Optic Cable Types

### 🟢 Single-Mode Fiber (SMF)

Laser → | Narrow Core | → Receiver

- **Narrow core** — light enters at a single angle.  
- Uses **laser-based transmitters**.  
- Supports **longer distances** than multimode or UTP.  
- **More expensive** due to laser-based SFPs.

---

### 🟣 Multi-Mode Fiber (MMF)
LED → |==== Wide Core ====| → Receiver
↳ Multiple light angles (modes)

- **Wider core** — allows multiple light paths (modes).  
- Uses **LED-based transmitters**.  
- Supports **medium distances** (longer than UTP, shorter than SMF).  
- **Cheaper** than single-mode fiber.

---

## 📊 Fiber-Optic Standards

| Speed | Standard | Connection Speed | Mode Support | Max Distance |
|--------|-----------|------------------|---------------|---------------:|
| 1 Gbps | 1000BASE-LX | 802.3z | Multimode / Single | 550m (Multi) / 5km (Single) |
| 10 Gbps | 10GBASE-SR | 802.3ae | Multimode | 400m |
| 10 Gbps | 10GBASE-LR | 802.3ae | Single | 10km |
| 10 Gbps | 10GBASE-ER | 802.3ae | Single | 30km |

---

## 🧩 UTP vs Fiber-Optic Cabling

| Feature | 🧵 UTP (Copper) | 💡 Fiber-Optic |
|----------|------------------|----------------|
| 💰 Cost | Lower | Higher |
| 📏 Max Distance | ~100m | Up to 30km |
| ⚡ EMI Resistance | Vulnerable | Immune |
| 🔌 Port Type | RJ45 (cheaper) | SFP (costlier) |
| 🔒 Security | Emits weak signals (can be intercepted) | No signal leakage |
| 💡 Signal Type | Electrical | Light-based |

---

📘 **Summary:**
Ethernet interfaces and cables form the backbone of wired networking.  
Understanding cable types, pinouts, and standards ensures **reliable, secure, and efficient** data communication across networks.

---

🖼️ **Image Placeholders (optional for visual README):**
- RJ-45 Connector → `![RJ45 Diagram](images/rj45.png)`  
- Straight vs Crossover Cable → `![Ethernet Cable Types](images/ethernet-cables.png)`  
- Single-mode vs Multi-mode Fiber → `![Fiber Types](images/fiber-types.png)`
