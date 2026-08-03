# Day 17 — VLANs (Part 2)

> **Course:** Jeremy's IT Lab — *CCNA 200-301 Complete Course* (Free)
> **Topic:** Trunk ports, 802.1Q VLAN tagging, the native VLAN, and Router on a Stick (ROAS)
> **Video reference:** `Free CCNA | VLANs (Part 2) | Day 17` + `Day 17 Lab`
> **Series:** Part 2 of 3 → follows [Day 16](chakradharmannepalli/Day-16/README.md), continues in [Day 18](chakradharmannepalli/Day-18/README.md)

---

## 📑 Table of Contents

1. [Learning Objectives](#-learning-objectives)
2. [Recap: Why We Need Trunk Ports](#1-recap-why-we-need-trunk-ports)
3. [VLAN Tagging: ISL vs. 802.1Q](#2-vlan-tagging-isl-vs-8021q)
4. [The 802.1Q Tag in Detail](#3-the-8021q-tag-in-detail)
5. [VLAN ID Ranges](#4-vlan-id-ranges)
6. [The Native VLAN](#5-the-native-vlan)
7. [Configuring Trunk Ports](#6-configuring-trunk-ports)
8. [Controlling Which VLANs Are Allowed on a Trunk](#7-controlling-which-vlans-are-allowed-on-a-trunk)
9. [Changing the Native VLAN](#8-changing-the-native-vlan)
10. [Router on a Stick (ROAS)](#9-router-on-a-stick-roas)
11. [Full Configuration Listing](#10-full-configuration-listing)
12. [Verification Commands](#11-verification-commands)
13. [Access Port vs. Trunk Port — Comparison](#12-access-port-vs-trunk-port--comparison)
14. [Common Mistakes & Gotchas](#13-common-mistakes--gotchas)
15. [Command Cheat Sheet](#14-command-cheat-sheet)
16. [Key Takeaways](#15-key-takeaways)
17. [Quick Self-Check Questions](#16-quick-self-check-questions)
18. [What's Next](#17-whats-next)

---

## 🎯 Learning Objectives

By the end of this day you should be able to:

- Explain what a **trunk port** is and why it's needed as a network grows past a handful of VLANs.
- Name the two VLAN trunking protocols and know which one actually matters for the exam (and real life).
- Draw and label the fields of an **802.1Q tag** from memory: TPID, PCP, DEI, VID.
- Explain the **native VLAN** — what it is, its default value, why it exists, and what happens on a **mismatch**.
- Configure a Cisco switchport as a trunk, including encapsulation, allowed VLANs, and native VLAN.
- Explain and configure **Router on a Stick (ROAS)** for inter-VLAN routing using subinterfaces.

---

## 1. Recap: Why We Need Trunk Ports

Continuing the [Day 16](../Day-16-VLANs-Part-1/README.md) example: Engineering (VLAN 10), HR (VLAN 20), and Sales (VLAN 30) are now separated into different broadcast domains. But real networks rarely fit on a single switch, and a department's users aren't always plugged into the same switch either.

Picture VLAN 10 (Engineering) split across **two** switches, `SW1` and `SW2`, with `SW2` also connecting up to the router `R1` for inter-VLAN routing. If we only had **access ports** available, we'd need:

- A **separate physical link, in the matching VLAN**, for every VLAN that needs to cross from `SW1` to `SW2`.
- A **separate physical link, in the matching VLAN**, for every VLAN that needs to reach `R1` from `SW2`.

This "one cable per VLAN" approach technically works, but it doesn't scale — real networks might have dozens of VLANs, and routers/switches simply don't have that many spare interfaces. This is exactly the inefficient legacy design shown at the end of Day 16.

**The solution is a trunk port** — a single physical link that carries traffic for **multiple VLANs simultaneously**, by tagging each frame with the VLAN it belongs to.

| | Access port | Trunk port |
|---|---|---|
| Belongs to | 1 VLAN | Multiple VLANs |
| Frames tagged? | No (*untagged port*) | Yes (*tagged port*), except the native VLAN |
| Typical use | Connect to end hosts | Connect switch↔switch or switch↔router |

---

## 2. VLAN Tagging: ISL vs. 802.1Q

There are two trunking protocols that add VLAN identification to frames sent over a trunk:

| Protocol | Type | Status |
|---|---|---|
| **ISL** (Inter-Switch Link) | Cisco proprietary | Legacy — predates 802.1Q. Not supported on modern Cisco equipment. **You will not use this in the real world.** |
| **IEEE 802.1Q** (a.k.a. "dot1q") | Open industry standard | The only protocol you need for the CCNA and for real networks today |

> 📌 **Exam tip:** For the CCNA 200-301, you only need to know **802.1Q** in depth. ISL is mentioned mainly so you recognize the name and know it's obsolete.

---

## 3. The 802.1Q Tag in Detail

802.1Q works by inserting an extra 4-byte (32-bit) field into the Ethernet frame — right **between the Source MAC address field and the Type/Length field**.

![802.1Q tag structure inside an Ethernet frame](images/01-dot1q-tag-structure.png)

The tag is made up of two main fields:

### 3.1 TPID — Tag Protocol Identifier (16 bits / 2 bytes)

- Always set to the hexadecimal value **`0x8100`**.
- This is how the receiving switch recognizes "this frame is 802.1Q tagged" in the first place.

### 3.2 TCI — Tag Control Information (16 bits / 2 bytes)

The TCI field itself is broken into three sub-fields:

| Sub-field | Length | Purpose |
|---|---|---|
| **PCP** — Priority Code Point | 3 bits | Used for **CoS** (Class of Service) to prioritize important traffic (e.g. voice) during congestion |
| **DEI** — Drop Eligible Indicator | 1 bit | Marks frames that are safe to drop first if the network becomes congested |
| **VID** — VLAN Identifier | 12 bits | **The most important field** — identifies which VLAN the frame belongs to |

> 💡 **Memory tip:** *TPID tells you it's tagged. TCI tells you which VLAN (and how important) the traffic is.*

---

## 4. VLAN ID Ranges

The VID field is 12 bits long, which gives 2¹² = **4096** possible values, numbered 0–4095.

- **VLAN 0** and **VLAN 4095** are reserved by the 802.1Q standard itself and cannot be used.
- That leaves a usable range of **1 – 4094**.
- This usable range is further split into two sub-ranges on Cisco switches:

| Range name | VLAN IDs | Notes |
|---|---|---|
| **Normal range** | 1 – 1005 | Stored in `vlan.dat`; advertised by VTP versions 1 & 2 (VTP is covered on Day 19) |
| **Extended range** | 1006 – 4094 | Supported on modern switches; historically had some VTP limitations |

*(Recall from [Day 16](../Day-16-VLANs-Part-1/README.md) that VLAN 1 and VLANs 1002–1005 are Cisco default/reserved VLANs within this normal range and can't be deleted — that's a separate, additional restriction on top of the 802.1Q-level reservation of 0 and 4095.)*

---

## 5. The Native VLAN

IEEE 802.1Q has one more important feature that **ISL does not have**: the **native VLAN**.

> **The native VLAN is the VLAN used for *untagged* traffic on a trunk port.**

Behaviour:

- A switch does **not** add an 802.1Q tag to frames belonging to the native VLAN — they're sent over the trunk exactly as a normal, untagged frame.
- When a switch **receives** an untagged frame on a trunk port, it assumes that frame belongs to the native VLAN.
- The native VLAN is **VLAN 1 by default** on every trunk port, and it must be configured **per trunk port** (it is not a switch-wide global setting).

### 5.1 Why does the native VLAN matter?

**The native VLAN configured on one end of a trunk must match the native VLAN configured on the other end.** If it doesn't, you get a **native VLAN mismatch**.

**Example of a mismatch:**

- `SW2`'s trunk port has its native VLAN set to **VLAN 10**.
- `SW1`'s trunk port (the other end of the same link) has its native VLAN set to **VLAN 30**.
- `SW2` sends an untagged frame (because, from its point of view, it's VLAN 10 = native, so no tag is added).
- `SW1` receives an **untagged** frame and — because *its own* native VLAN is configured as VLAN 30 — assumes the frame belongs to VLAN 30.
- If the actual destination is in VLAN 10, `SW1` will not forward the frame correctly (it may be dropped or, worse, leak into the wrong VLAN).

> ⚠️ **Security note:** A native VLAN mismatch is also the basis of a class of attack known as **VLAN hopping**. Because of this, real-world best practice is to change the native VLAN on trunk ports to an **unused** VLAN ID (never VLAN 1, and never a VLAN that's actively carrying user traffic).

---

## 6. Configuring Trunk Ports

Continuing the running example, here's the topology we'll configure (matching the Day 16 addressing):

![Router on a Stick topology with SW1, SW2, and R1](images/02-roas-topology.png)

We'll configure:

- `SW1`'s `G0/0` → trunk to `SW2` (needs VLAN 10 and VLAN 30 — there are no VLAN 20 hosts on `SW1`)
- `SW2`'s `G0/0` → trunk to `SW1` (same as above)
- `SW2`'s `G0/1` → trunk to `R1` (needs **all three** VLANs, since `R1` must route between all of them)

### 6.1 Basic Trunk Configuration

```
SW1(config)# interface g0/0
SW1(config-if)# switchport mode trunk
% Command rejected: An interface whose trunk encapsulation is "Auto"
  can not be configured to "trunk" mode.
```

> ⚠️ On switches that support **both** ISL and 802.1Q, the trunk encapsulation defaults to **Auto**, and you must manually pick one before you can force trunk mode. (Switches that only support 802.1Q skip this step entirely.)

```
SW1(config-if)# switchport trunk encapsulation dot1q
SW1(config-if)# switchport mode trunk
```

Confirm with:

```
SW1# show interfaces trunk
```

```
Port      Mode             Encapsulation  Status        Native vlan
Gi0/0     on                802.1q         trunking      1

Port      Vlans allowed on trunk
Gi0/0     1-4094

Port      Vlans allowed and active in management domain
Gi0/0     1,10,30
```

By default, **all** VLANs (1–4094) are allowed across a new trunk.

---

## 7. Controlling Which VLANs Are Allowed on a Trunk

For both performance and security, you usually want to restrict a trunk to only the VLANs it actually needs to carry. The command is `switchport trunk allowed vlan`, with several keyword options:

| Keyword | Effect |
|---|---|
| `switchport trunk allowed vlan <list>` | **Replaces** the allowed list entirely with the VLANs specified |
| `switchport trunk allowed vlan add <list>` | **Adds** VLAN(s) to the existing allowed list |
| `switchport trunk allowed vlan remove <list>` | **Removes** VLAN(s) from the existing allowed list |
| `switchport trunk allowed vlan all` | Allows **all** VLANs (the default state) |
| `switchport trunk allowed vlan except <list>` | Allows **all** VLANs *except* the ones listed |
| `switchport trunk allowed vlan none` | Allows **no** VLANs at all — the trunk carries nothing |

*(Multiple VLANs in `add`, `remove`, or `except` are comma-separated, e.g. `1-5,10`.)*

### Applying it to our topology

On `SW1`'s `G0/0` (only VLAN 10 and VLAN 30 exist on `SW1`):

```
SW1(config)# interface g0/0
SW1(config-if)# switchport trunk allowed vlan 10,30
```

On `SW2`'s `G0/0` (the link to `SW1` — same two VLANs):

```
SW2(config)# interface g0/0
SW2(config-if)# switchport trunk encapsulation dot1q
SW2(config-if)# switchport mode trunk
SW2(config-if)# switchport trunk allowed vlan 10,30
```

On `SW2`'s `G0/1` (the link to `R1` — needs **all three** VLANs):

```
SW2(config)# interface g0/1
SW2(config-if)# switchport trunk encapsulation dot1q
SW2(config-if)# switchport mode trunk
SW2(config-if)# switchport trunk allowed vlan 10,20,30
```

> 💡 **Verification reminder:** `show vlan brief` will **not** show `G0/0` or `G0/1` under any VLAN, even though they carry that VLAN's traffic — because `show vlan brief` only lists **access** ports. Use `show interfaces trunk` to confirm trunk configuration and allowed VLANs.

---

## 8. Changing the Native VLAN

As discussed in [Section 5](#5-the-native-vlan), best practice is to move the native VLAN off VLAN 1 to an unused VLAN ID (here, VLAN 1001 is used as an example — pick any VLAN that carries no real traffic):

```
SW1(config-if)# switchport trunk native vlan 1001
```

Do this **identically** on the other end of the same trunk link, or you'll create a native VLAN mismatch (see Section 5.1).

---

## 9. Router on a Stick (ROAS)

### 9.1 The Idea

Instead of using a separate physical router interface for every VLAN (the Day 16 legacy approach), **Router on a Stick (ROAS)** — sometimes nicknamed the *"one-armed router"* — uses:

- **One** physical trunk link between the router and the switch, and
- **Multiple logical subinterfaces** on the router, one per VLAN, each with its own 802.1Q tag and its own IP address.

The switch side just needs a normal trunk port, configured exactly like in Sections 6–7 above — **no extra configuration is needed on the switch for ROAS to work.**

### 9.2 Configuring the Router

On `R1`, the physical interface itself gets **no IP address** — only the subinterfaces do:

```
R1(config)# interface g0/0
R1(config-if)# no shutdown
!
R1(config-if)# interface g0/0.10
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# ip address 192.168.1.62 255.255.255.192
!
R1(config)# interface g0/0.20
R1(config-subif)# encapsulation dot1q 20
R1(config-subif)# ip address 192.168.1.126 255.255.255.192
!
R1(config)# interface g0/0.30
R1(config-subif)# encapsulation dot1q 30
R1(config-subif)# ip address 192.168.1.190 255.255.255.192
```

> 💡 **Naming convention:** The subinterface number (`.10`, `.20`, `.30`) does **not** technically have to match the VLAN number configured with `encapsulation dot1q`. But matching them is strongly recommended — it makes the configuration far easier to read and troubleshoot.

> ⚠️ **Don't forget `no shutdown` on the physical interface** — router interfaces are administratively shut down by default, and if the physical interface is down, **none** of its subinterfaces will work either, no matter how correctly they're configured.

### 9.3 How It Works

- Any frame arriving on `G0/0` tagged with VLAN 10 is treated by the router **as if it arrived on subinterface `G0/0.10`**.
- When `R1` needs to send a packet back out towards the VLAN 20 subnet, it sends it out of the physical `G0/0` interface, tagged with VLAN 20 (because that's what's configured on subinterface `G0/0.20`).
- The router now behaves, logically, as if it had three separate physical interfaces — one per VLAN — despite only using one physical port.

### 9.4 Verification

```
R1# show ip interface brief
```

```
Interface              IP-Address       OK? Method Status                Protocol
GigabitEthernet0/0     unassigned       YES unset  up                    up
GigabitEthernet0/0.10  192.168.1.62     YES manual up                    up
GigabitEthernet0/0.20  192.168.1.126    YES manual up                    up
GigabitEthernet0/0.30  192.168.1.190    YES manual up                    up
```

```
R1# show ip route
```

```
     192.168.1.0/26 is directly connected, GigabitEthernet0/0.10
     192.168.1.64/26 is directly connected, GigabitEthernet0/0.20
     192.168.1.128/26 is directly connected, GigabitEthernet0/0.30
```

### 9.5 Worked Traffic Example

`PC-D` in VLAN 10 (attached to `SW2`) wants to reach `PC-C` in VLAN 30 (attached to `SW1`):

1. `PC-D` sends the frame to its default gateway, `192.168.1.62` (R1's `G0/0.10`).
2. `SW2` receives it on an access port in VLAN 10, and forwards it out its trunk to `R1`, tagged **VLAN 10**.
3. `R1` receives the tagged frame on `G0/0` → treats it as arriving on `G0/0.10` → routes it (Layer 3 lookup) → determines the destination subnet, `192.168.1.128/26`, is reachable via `G0/0.30`.
4. `R1` sends the packet back out `G0/0`, this time tagged **VLAN 30** (because that's configured on `G0/0.30`).
5. `SW2` receives the VLAN 30-tagged frame on the trunk and forwards it to `SW1` over the `SW1↔SW2` trunk, still tagged VLAN 30.
6. `SW1` forwards it out the correct access port to `PC-C`.

**Key insight:** even though `PC-D` and `PC-C` are only two switch-hops apart physically, the traffic must travel all the way to `R1` and back, because inter-VLAN routing can only happen at a Layer 3 device. *(Day 18 introduces a faster alternative to this — the multilayer switch.)*

---

## 10. Full Configuration Listing

**`SW1`:**

```
interface g0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,30
 switchport trunk native vlan 1001
```

**`SW2`:**

```
interface g0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,30
 switchport trunk native vlan 1001
!
interface g0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

**`R1`:**

```
interface g0/0
 no shutdown
!
interface g0/0.10
 encapsulation dot1q 10
 ip address 192.168.1.62 255.255.255.192
!
interface g0/0.20
 encapsulation dot1q 20
 ip address 192.168.1.126 255.255.255.192
!
interface g0/0.30
 encapsulation dot1q 30
 ip address 192.168.1.190 255.255.255.192
```

---

## 11. Verification Commands

| Command | Where | Shows |
|---|---|---|
| `show interfaces trunk` | Switch | Trunk ports, encapsulation, native VLAN, allowed VLANs, active VLANs |
| `show vlan brief` | Switch | **Access** port membership only — trunks won't appear here |
| `show ip interface brief` | Router | IP addresses of the physical interface and every subinterface |
| `show ip route` | Router | Connected routes for each subinterface's subnet |
| `show running-config` | Both | Full applied configuration |

---

## 12. Access Port vs. Trunk Port — Comparison

| | Access Port | Trunk Port |
|---|---|---|
| Alternate name | Untagged port | Tagged port |
| VLAN membership | Exactly one | Multiple (all by default, restrict with `allowed vlan`) |
| Tags frames | No | Yes — except native VLAN traffic |
| Default native VLAN | N/A | VLAN 1 |
| Typical connection | End host (PC, printer) | Switch↔switch, switch↔router |
| Key config command | `switchport mode access` | `switchport mode trunk` |

---

## 13. Common Mistakes & Gotchas

- ❌ **Trying `switchport mode trunk` before setting the encapsulation** on switches that support both ISL and 802.1Q — you'll get `Command rejected`. Set `switchport trunk encapsulation dot1q` first (not needed on switches that only support dot1q).
- ❌ **Mismatched native VLANs** between the two ends of a trunk. Always configure the *same* native VLAN on both sides.
- ❌ **Leaving the native VLAN as VLAN 1** in a production network — a well-known security weakness that enables VLAN hopping attacks; change it to an unused VLAN.
- ❌ **Checking `show vlan brief` to verify trunk ports.** It won't show them — use `show interfaces trunk` instead.
- ❌ **Forgetting `no shutdown` on the router's physical interface** when configuring ROAS — all subinterfaces will stay down until the parent physical interface is enabled.
- ❌ **Mismatching subinterface number and VLAN tag on purpose "to save typing."** It works, but it makes troubleshooting much harder — always match them where possible.
- ❌ **Forgetting to allow the correct VLANs on each trunk segment.** `SW1↔SW2` in our example only needs VLANs 10 and 30 — allowing VLAN 20 there too isn't wrong, but it's unnecessary and against best practice (restrict trunks to only what's needed).

---

## 14. Command Cheat Sheet

| Command | Mode | Purpose |
|---|---|---|
| `switchport trunk encapsulation {dot1q \| isl \| negotiate}` | Interface config | Set trunking protocol (only needed on switches that support both) |
| `switchport mode trunk` | Interface config | Force the port into trunk mode |
| `switchport trunk allowed vlan {list \| add \| remove \| all \| except \| none}` | Interface config | Control which VLANs cross this trunk |
| `switchport trunk native vlan <id>` | Interface config | Set the native (untagged) VLAN for this trunk |
| `show interfaces trunk` | Priv. EXEC | Verify trunk ports, encapsulation, native VLAN, allowed VLANs |
| `interface <phys-if>.<subif-id>` | Global config | Create/enter a router subinterface |
| `encapsulation dot1q <vlan-id>` | Subinterface config | Tag this subinterface's traffic with a VLAN |
| `ip address <ip> <mask>` | Subinterface config | Assign the subinterface's gateway IP address |

---

## 15. Key Takeaways

- A **trunk port** carries traffic for multiple VLANs over one physical link by tagging frames.
- **802.1Q** is the only trunking protocol you need for the CCNA (ISL is obsolete).
- The 802.1Q tag is **4 bytes**: a **TPID** (always `0x8100`) plus a **TCI**, which itself contains **PCP** (3 bits), **DEI** (1 bit), and **VID** (12 bits — the actual VLAN ID).
- Usable VLAN range is **1–4094** (0 and 4095 are reserved by the protocol).
- The **native VLAN** carries untagged traffic on a trunk; it defaults to VLAN 1 and **must match on both ends** of the link.
- `switchport trunk allowed vlan` restricts which VLANs may cross a trunk — good practice for security and performance.
- **ROAS (Router on a Stick)** performs inter-VLAN routing using a single physical router interface split into **subinterfaces**, each with its own `encapsulation dot1q <vlan>` and IP address.
- Every packet routed between VLANs via ROAS makes a round trip through the router — switch → router → switch.

---

## 16. Quick Self-Check Questions

<details>
<summary><strong>1. Which field of the 802.1Q tag actually identifies the VLAN?</strong></summary>

The **VID (VLAN Identifier)** field, 12 bits, inside the TCI.
</details>

<details>
<summary><strong>2. What hexadecimal value does the TPID field always contain?</strong></summary>

`0x8100`.
</details>

<details>
<summary><strong>3. By default, is the native VLAN's traffic tagged when sent across a trunk?</strong></summary>

No — native VLAN traffic is sent **untagged**. That's the defining characteristic of the native VLAN.
</details>

<details>
<summary><strong>4. What command restricts a trunk to allow only VLANs 10, 20, and 99?</strong></summary>

`switchport trunk allowed vlan 10,20,99`
</details>

<details>
<summary><strong>5. In ROAS, does the physical router interface itself need an IP address?</strong></summary>

No — the IP addresses go on the **subinterfaces**. The physical interface only needs `no shutdown` to bring the whole thing up.
</details>

<details>
<summary><strong>6. What happens if two switches on either end of a trunk have mismatched native VLANs?</strong></summary>

Untagged frames sent by one switch will be assumed by the other switch to belong to its own configured native VLAN, which can cause traffic to be dropped or forwarded into the wrong VLAN — a "native VLAN mismatch," and a known VLAN-hopping security risk.
</details>

---

## 17. What's Next

➡️ **[Day 18 — VLANs (Part 3)](../Day-18-VLANs-Part-3/README.md):** configuring the native VLAN directly on a router, and inter-VLAN routing using a **multilayer (Layer 3) switch** with Switched Virtual Interfaces (SVIs) — a faster alternative to ROAS.

⬅️ **[Back to Day 16 — VLANs (Part 1)](../Day-16-VLANs-Part-1/README.md)**

---

<sub>Notes based on Jeremy's IT Lab — *Free CCNA 200-301 Complete Course*, Day 17: "VLANs (Part 2)". This is an independent study summary created for personal revision; all credit for the original teaching content goes to Jeremy's IT Lab.</sub>
