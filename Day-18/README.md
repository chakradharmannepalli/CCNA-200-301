# Day 18 — VLANs (Part 3)

> **Course:** Jeremy's IT Lab — *CCNA 200-301 Complete Course* (Free)
> **Topic:** Native VLAN on a router, multilayer (Layer 3) switching, and inter-VLAN routing using SVIs
> **Video reference:** `Free CCNA | VLANs (Part 3) | Day 18` + `Day 18 Lab`
> **Series:** Part 3 of 3 → follows [Day 16](../Day-16-VLANs-Part-1/README.md) and [Day 17](../Day-17-VLANs-Part-2/README.md)

---

## 📑 Table of Contents

1. [Learning Objectives](#-learning-objectives)
2. [The Native VLAN on a Router (ROAS)](#1-the-native-vlan-on-a-router-roas)
3. [Wireshark View: Tagged vs. Untagged Frames](#2-wireshark-view-tagged-vs-untagged-frames)
4. [Layer 3 Switching / Multilayer Switching](#3-layer-3-switching--multilayer-switching)
5. [Inter-VLAN Routing Methods — Full Comparison](#4-inter-vlan-routing-methods--full-comparison)
6. [Inter-VLAN Routing via SVIs](#5-inter-vlan-routing-via-svis)
7. [Step-by-Step SVI Configuration](#6-step-by-step-svi-configuration)
8. [Conditions for an SVI to Be Up/Up](#7-conditions-for-an-svi-to-be-upup)
9. [Full Configuration Listing](#8-full-configuration-listing)
10. [Verification Commands](#9-verification-commands)
11. [Common Mistakes & Gotchas](#10-common-mistakes--gotchas)
12. [Command Cheat Sheet](#11-command-cheat-sheet)
13. [Key Takeaways](#12-key-takeaways)
14. [Quick Self-Check Questions](#13-quick-self-check-questions)
15. [What's Next](#14-whats-next)

---

## 🎯 Learning Objectives

By the end of this day you should be able to:

- Configure the **native VLAN on a router's ROAS interface** using two different methods.
- Read a Wireshark capture and identify a dot1q-tagged frame versus an untagged native-VLAN frame.
- Explain what a **multilayer switch (Layer 3 switch)** is and how it differs from a regular Layer 2 switch.
- Compare all three inter-VLAN routing methods covered across Days 16–18.
- Configure a **routed port** on a Layer 3 switch.
- Configure **Switched Virtual Interfaces (SVIs)** to perform inter-VLAN routing directly on a switch.
- List the four conditions required for an SVI to reach an up/up state.

---

## 1. The Native VLAN on a Router (ROAS)

In [Day 17](../Day-17-VLANs-Part-2/README.md) we configured `R1` with three subinterfaces (`G0/0.10`, `G0/0.20`, `G0/0.30`), each tagging its own VLAN. But what if we want VLAN 10 to be the **native VLAN** on that trunk — i.e., sent/received untagged?

There are **two valid ways** to configure this on a router:

![Two methods for configuring the native VLAN on a router](images/01-native-vlan-router-methods.png)

### Method 1 — Native VLAN on a Subinterface

VLAN 10 still gets its own subinterface, `G0/0.10` — but we add the keyword **`native`** to the `encapsulation dot1q` command:

```
R1(config)# interface g0/0.10
R1(config-subif)# encapsulation dot1q 10 native
```

This tells `R1`: *"treat `G0/0.10` as the native VLAN — send its traffic untagged, and assume any untagged traffic received on `G0/0` belongs to VLAN 10."* Everything else about the subinterface (its IP address, for example) stays the same as before.

### Method 2 — IP Address Directly on the Physical Interface

Alternatively, skip the subinterface for VLAN 10 entirely, and assign the IP address straight onto the **physical** interface — which is inherently untagged:

```
R1(config)# no interface g0/0.10
R1(config)# interface g0/0
R1(config-if)# ip address 192.168.1.62 255.255.255.192
```

Both methods behave identically from the network's point of view: `SW2` sends VLAN 10 traffic untagged to `R1`, and `R1` sends VLAN 10 traffic untagged back to `SW2`.

> ⚠️ **Whichever method you choose, the native VLAN configured on the router must match the native VLAN configured on the switch's trunk port** (`switchport trunk native vlan`, from Day 17) — otherwise you get the same native VLAN mismatch problem discussed in Day 17.

---

## 2. Wireshark View: Tagged vs. Untagged Frames

Understanding this at the packet level makes native VLAN behavior click. Imagine capturing traffic on the link between `SW2` and `R1` while a PC in VLAN 20 pings a PC in VLAN 10 (which is the native VLAN in this example).

**Request — VLAN 20 (non-native) PC → R1:**

```
Ethernet II, Src: PC-E_MAC, Dst: R1_MAC
    Type: 802.1Q Virtual LAN (0x8100)
802.1Q Virtual LAN
    .000 0000 0000 0000 = Priority: 0
    .... .... .... .... = DEI: Not set
    .... 0000 0001 0100 = ID: 20
    Type: IPv4 (0x0800)
Internet Protocol Version 4
```

The Ethernet **Type** field reads `0x8100`, confirming this frame is 802.1Q-tagged, and the **VID** inside the tag reads **20** — because VLAN 20 is not the native VLAN, it must be tagged.

**Reply — R1 → VLAN 10 (native) PC:**

```
Ethernet II, Src: R1_MAC, Dst: PC-A_MAC
    Type: IPv4 (0x0800)
Internet Protocol Version 4
```

Notice there's **no 802.1Q layer at all** in this frame — the Type field goes straight to `0x0800` (IPv4). Because VLAN 10 is the native VLAN on this trunk, neither `R1` nor `SW2` bothers tagging it.

> 💡 This is also *why* the native VLAN feature exists in the first place: untagged native VLAN frames are 4 bytes smaller than tagged frames, which is a tiny efficiency gain — though the *main* reason it exists is for backward compatibility with devices that don't understand 802.1Q tagging at all.

---

## 3. Layer 3 Switching / Multilayer Switching

So far, every switch we've used (`SW1`, `SW2`) has been a plain **Layer 2 switch** — it forwards frames based on MAC addresses only, and it has no concept of IP addresses or routing.

Meet the **multilayer switch**, also called a **Layer 3 switch**:

| | Layer 2 Switch | Multilayer (Layer 3) Switch |
|---|---|---|
| Forwards frames using | MAC address table | MAC address table **and** an IP routing table |
| IP-aware? | No | Yes |
| Can run static/dynamic routing? | No | Yes — just like a router |
| Can perform inter-VLAN routing on its own? | No — must send traffic to a router | Yes, using **SVIs** or **routed ports** |

A multilayer switch does everything a regular switch does (Layer 2 forwarding, VLANs, trunking) **plus** it can route packets between subnets/VLANs at very high speed, without ever needing to send the traffic to an external router.

There are two ways to give a multilayer switch a Layer 3 (IP-routable) presence:

1. **Switched Virtual Interface (SVI):** a logical, software-based interface tied to a specific VLAN (`interface vlan <id>`). Hosts in that VLAN use the SVI's IP address as their default gateway.
2. **Routed port:** converts a **physical** switch interface from an L2 switchport into an L3, router-style interface (using `no switchport`) that you can assign an IP address to directly — just like a router's physical interface. Typically used for point-to-point links to another router or Layer 3 device, not for VLANs.

---

## 4. Inter-VLAN Routing Methods — Full Comparison

Having now covered all three approaches across this three-part series, here's the complete picture:

| # | Method | Covered in | Physical links needed | Scalability | How it works |
|---|---|---|---|---|---|
| 1 | **Legacy (multiple router interfaces)** | [Day 16](../Day-16-VLANs-Part-1/README.md) | One **access** link per VLAN | ❌ Poor — runs out of interfaces fast | Each router interface sits on an access port in a different VLAN and acts as that VLAN's default gateway |
| 2 | **Router on a Stick (ROAS)** | [Day 17](../Day-17-VLANs-Part-2/README.md) | **One trunk link** total | ⚠️ Better, but all inter-VLAN traffic funnels through one router link, which can become a bottleneck | Router subinterfaces, each tagged with `encapsulation dot1q <vlan>`, route between VLANs |
| 3 | **Multilayer switch with SVIs** | Day 18 (this page) | No dedicated router link needed for local VLANs | ✅ Best — the standard for real, growing networks | The switch itself routes between VLANs at wire speed using SVIs |

---

## 5. Inter-VLAN Routing via SVIs

### 5.1 Updating the Topology

To use SVIs, `SW2` is upgraded from a Layer 2 switch to a **multilayer switch**. We also replace the trunk link between `SW2` and `R1` with a **routed point-to-point link** — `R1` is now only used to reach *outside* the LAN (the Internet / rest of the network), not for inter-VLAN routing.

![Multilayer switch performing inter-VLAN routing with SVIs and a routed port to R1](images/02-svi-multilayer-topology.png)

We'll carve the previously-unused fourth `/26` block from Day 16 (`192.168.1.192/26`) down into a `/30` using VLSM, just for this point-to-point link:

| Link | Subnet | `SW2` G0/1 | `R1` G0/0 |
|---|---|---|---|
| `SW2` ↔ `R1` (routed) | `192.168.1.192/30` | `192.168.1.193` | `192.168.1.194` |

The SVIs on `SW2` reuse the **exact same gateway IPs** that `R1`'s subinterfaces used in Day 17 — so **no reconfiguration is needed on any PC**; their default gateway addresses stay identical:

| VLAN | SVI | IP Address |
|---|---|---|
| 10 | `interface vlan 10` | `192.168.1.62 /26` |
| 20 | `interface vlan 20` | `192.168.1.126 /26` |
| 30 | `interface vlan 30` | `192.168.1.190 /26` |

### 5.2 Worked Traffic Example — Fully Local Routing

`PC-D` (VLAN 10, on `SW2`) pings `PC-B` (VLAN 30, attached to `SW1`):

1. `PC-D` sends the frame to its gateway `192.168.1.62` — which now lives on `SW2`'s own **VLAN 10 SVI**, not on `R1`.
2. `SW2` looks up `192.168.1.128/26` in **its own routing table**, sees it's directly connected via the VLAN 30 SVI.
3. `SW2` re-encapsulates the packet as an Ethernet frame tagged VLAN 30 and forwards it out its trunk toward `SW1` (if the destination MAC isn't already known, it floods within VLAN 30 first).
4. `SW1` delivers the frame to `PC-B`.

**Notice `R1` is never involved.** This is the core advantage of SVIs over ROAS — routing between local VLANs happens entirely inside the switch, at hardware speed, with zero extra hops out to a router and back.

`R1` now comes into play **only** when a host needs to reach something outside the LAN (the Internet cloud in the diagram) — `SW2` is the default gateway for all local hosts, and it forwards anything destined outside its own subnets to `R1` via the routed point-to-point link, using a default route.

---

## 6. Step-by-Step SVI Configuration

### 6.1 Configure the Point-to-Point Link (Router Side)

First, remove the old ROAS subinterfaces from `R1` and reset the physical interface back to a plain routed interface:

```
R1(config)# no interface g0/0.10
R1(config)# no interface g0/0.20
R1(config)# no interface g0/0.30
R1(config)# default interface g0/0
R1(config)# interface g0/0
R1(config-if)# ip address 192.168.1.194 255.255.255.252
R1(config-if)# no shutdown
```

> 💡 `default interface <name>` resets an interface back to its factory-default configuration in one step — handy for clearing out old trunk/subinterface configuration before repurposing an interface.

### 6.2 Configure the Point-to-Point Link (Switch Side) + Enable Routing

```
SW2(config)# default interface g0/1
SW2(config)# ip routing
SW2(config)# interface g0/1
SW2(config-if)# no switchport
SW2(config-if)# ip address 192.168.1.193 255.255.255.252
```

> ⚠️ **`ip routing` is easy to forget — and critical.** By default, IP routing is **disabled** on a Cisco multilayer switch, even if it's fully licensed/capable of it. Without this command, the switch will not build or use a routing table, and inter-VLAN routing (even with SVIs configured) simply will not work.
>
> **`no switchport`** is the command that converts a physical switch interface from a Layer 2 switchport into a Layer 3 **routed port** — only after this can you assign it an IP address the way you would on a router interface.

### 6.3 Configure a Default Route Toward R1

So that traffic destined outside the LAN gets forwarded correctly:

```
SW2(config)# ip route 0.0.0.0 0.0.0.0 192.168.1.194
```

### 6.4 Configure the SVIs

```
SW2(config)# interface vlan 10
SW2(config-if)# ip address 192.168.1.62 255.255.255.192
SW2(config-if)# no shutdown
!
SW2(config)# interface vlan 20
SW2(config-if)# ip address 192.168.1.126 255.255.255.192
SW2(config-if)# no shutdown
!
SW2(config)# interface vlan 30
SW2(config-if)# ip address 192.168.1.190 255.255.255.192
SW2(config-if)# no shutdown
```

> ⚠️ **SVIs are administratively shut down by default** — just like router interfaces — so `no shutdown` is required on every single one, or it will never come up.

---

## 7. Conditions for an SVI to Be Up/Up

An SVI won't simply come up just because you've assigned it an IP address and typed `no shutdown`. **All four** of the following conditions must be true:

| # | Condition | Explanation |
|---|---|---|
| 1 | **The VLAN must already exist** in the switch's VLAN database | Unlike assigning an *access port* to a new VLAN (which auto-creates it — see [Day 16](../Day-16-VLANs-Part-1/README.md)), creating an SVI for a VLAN that doesn't exist does **not** auto-create the VLAN. Create it first with `vlan <id>`. |
| 2 | **At least one port in that VLAN must be up/up** | Either an access port assigned to the VLAN, or a trunk port that allows the VLAN — as long as one of them is physically up. |
| 3 | **The VLAN itself must not be administratively shut down** | You can `shutdown` a VLAN from VLAN configuration mode (`vlan <id>` → `shutdown`) — this is different from shutting down the SVI, and it will also prevent the SVI from coming up. |
| 4 | **The SVI itself must not be shut down** | SVIs are disabled by default; always run `no shutdown` after creating one. |

**Example of condition #1 failing:** if you type `interface vlan 40` and VLAN 40 was never created, the SVI's line protocol will sit in a **down/down** state indefinitely, no matter what IP address you assign or how many times you type `no shutdown` — because the underlying VLAN simply doesn't exist.

---

## 8. Full Configuration Listing

**`R1`:**

```
interface g0/0
 ip address 192.168.1.194 255.255.255.252
 no shutdown
```

**`SW2`:**

```
ip routing
!
interface g0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,30
!
interface g0/1
 no switchport
 ip address 192.168.1.193 255.255.255.252
!
interface vlan 10
 ip address 192.168.1.62 255.255.255.192
 no shutdown
!
interface vlan 20
 ip address 192.168.1.126 255.255.255.192
 no shutdown
!
interface vlan 30
 ip address 192.168.1.190 255.255.255.192
 no shutdown
!
ip route 0.0.0.0 0.0.0.0 192.168.1.194
```

**`SW1`:** *(unchanged from Day 17 — still a Layer 2 switch, still trunking VLANs 10 and 30 up to `SW2`)*

```
interface g0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,30
```

---

## 9. Verification Commands

| Command | Where | Shows |
|---|---|---|
| `show ip route` | `SW2` | Connected routes for each SVI + the default route pointing to `R1` |
| `show interfaces status` | `SW2` | The VLAN column shows **`routed`** instead of a VLAN number for a routed port |
| `show ip interface brief` | `SW2` / `R1` | Up/down status and IP address of every Layer 3 interface, including SVIs |
| `show vlan brief` | `SW2` | Confirms the VLAN exists (required for its SVI to work) |
| `show run \| section interface Vlan` | `SW2` | Quickly review just the SVI configuration block |

---

## 10. Common Mistakes & Gotchas

- ❌ **Forgetting `ip routing`.** This is the single most common reason "SVIs are configured but inter-VLAN routing still doesn't work." Without it, the switch never builds/uses a Layer 3 routing table.
- ❌ **Forgetting `no shutdown` on an SVI.** SVIs, like router interfaces, are shut down by default.
- ❌ **Assuming creating an SVI auto-creates the VLAN.** It does **not** — this is the opposite behavior from assigning an access port, which *does* auto-create a missing VLAN. Always `vlan <id>` first if you're not sure the VLAN exists.
- ❌ **Forgetting `no switchport` on a routed port.** Without it, the interface stays a Layer 2 switchport and won't accept an IP address the way a router interface does.
- ❌ **Native VLAN mismatch between the router and the switch's ROAS trunk** (if still using ROAS elsewhere) — both ends must agree on which VLAN is native.
- ❌ **Forgetting that at least one port must be up/up in a VLAN for its SVI to come up.** If every access/trunk port carrying that VLAN goes down, the SVI will follow it down too.
- ❌ **Confusing a routed port with an SVI.** A routed port is a single **physical** interface acting like a router port (point-to-point, no VLAN). An SVI is a **logical** interface tied to a VLAN, used as the gateway for many hosts in that VLAN.

---

## 11. Command Cheat Sheet

| Command | Mode | Purpose |
|---|---|---|
| `ip routing` | Global config | Enables Layer 3 routing on a multilayer switch (off by default) |
| `no switchport` | Interface config | Converts a switchport into a Layer 3 **routed port** |
| `interface vlan <id>` | Global config | Creates/enters config mode for an **SVI** |
| `no shutdown` | SVI / interface config | Required to bring the interface up (disabled by default) |
| `ip route 0.0.0.0 0.0.0.0 <next-hop>` | Global config | Configures a default route |
| `default interface <name>` | Global config | Resets an interface to its factory-default configuration |
| `encapsulation dot1q <vlan> native` | Subinterface config | Marks a router subinterface as the native VLAN for ROAS |
| `show interfaces status` | Priv. EXEC | Confirms whether a port is `routed`, `trunk`, or in a specific VLAN |

---

## 12. Key Takeaways

- A router's native VLAN can be configured two ways: with `encapsulation dot1q <id> native` on a subinterface, or by placing the IP address directly on the **physical** interface.
- In Wireshark, native VLAN traffic shows **no 802.1Q layer at all**; tagged VLAN traffic shows `Type: 802.1Q Virtual LAN (0x8100)` with a visible VID.
- A **multilayer (Layer 3) switch** can both switch (Layer 2) and route (Layer 3) — unlike a regular Layer 2 switch.
- Three inter-VLAN routing methods exist: **legacy per-VLAN links**, **ROAS**, and **SVIs on a multilayer switch** — SVIs are the standard, scalable solution used in real networks.
- **`ip routing`** must be explicitly enabled on a multilayer switch before it will route anything — it's off by default.
- **`no switchport`** turns a physical switchport into a Layer 3 **routed port**, usable for point-to-point links (e.g., to another router).
- **SVIs** (`interface vlan <id>`) give a VLAN a routable gateway IP directly on the switch — and unlike access ports, they do **not** auto-create a missing VLAN.
- An SVI needs **all four** conditions to reach up/up: VLAN exists, at least one port in that VLAN is up, the VLAN isn't shut down, and the SVI itself isn't shut down.
- With SVIs, inter-VLAN traffic between local hosts never has to leave the switch — a major performance advantage over ROAS.

---

## 13. Quick Self-Check Questions

<details>
<summary><strong>1. Name the two ways to configure the native VLAN on a router in a ROAS setup.</strong></summary>

(1) `encapsulation dot1q <vlan> native` on a subinterface, or (2) configure the IP address directly on the physical interface with no subinterface for that VLAN.
</details>

<details>
<summary><strong>2. In a Wireshark capture, how can you tell a frame belongs to the native VLAN rather than a tagged VLAN?</strong></summary>

The frame has no 802.1Q layer at all — the Ethernet Type field goes straight to the next protocol (e.g. `0x0800` for IPv4) with no `0x8100` tag present.
</details>

<details>
<summary><strong>3. What command enables Layer 3 routing on a multilayer switch?</strong></summary>

`ip routing`, entered in global configuration mode. It is disabled by default.
</details>

<details>
<summary><strong>4. Does creating an SVI for a VLAN that doesn't exist automatically create that VLAN?</strong></summary>

No. This is different from access ports. You must create the VLAN yourself first (`vlan <id>`) before its SVI can come up.
</details>

<details>
<summary><strong>5. What command converts a Layer 2 switchport into a Layer 3 routed port?</strong></summary>

`no switchport`, entered in interface configuration mode.
</details>

<details>
<summary><strong>6. List all four conditions required for an SVI to be up/up.</strong></summary>

(1) The VLAN exists in the VLAN database. (2) At least one access or trunk port carrying that VLAN is up/up. (3) The VLAN itself is not administratively shut down. (4) The SVI itself is not shut down.
</details>

<details>
<summary><strong>7. Compared to ROAS, why is SVI-based inter-VLAN routing generally faster?</strong></summary>

With ROAS, every inter-VLAN packet has to leave the switch, travel to the router, get routed, and travel back to the switch. With SVIs, the routing decision happens inside the multilayer switch itself, so local inter-VLAN traffic never has to make that extra round trip.
</details>

---

## 14. What's Next

➡️ **Day 19 — DTP & VTP:** Dynamic Trunking Protocol (automatic trunk negotiation) and VLAN Trunking Protocol (propagating VLAN databases across multiple switches).

⬅️ **[Back to Day 17 — VLANs (Part 2)](# Day 18 — VLANs (Part 3)

> **Course:** Jeremy's IT Lab — *CCNA 200-301 Complete Course* (Free)
> **Topic:** Native VLAN on a router, multilayer (Layer 3) switching, and inter-VLAN routing using SVIs
> **Video reference:** `Free CCNA | VLANs (Part 3) | Day 18` + `Day 18 Lab`
> **Series:** Part 3 of 3 → follows [Day 16](../Day-16-VLANs-Part-1/README.md) and [Day 17](../Day-17-VLANs-Part-2/README.md)

---

## 📑 Table of Contents

1. [Learning Objectives](#-learning-objectives)
2. [The Native VLAN on a Router (ROAS)](#1-the-native-vlan-on-a-router-roas)
3. [Wireshark View: Tagged vs. Untagged Frames](#2-wireshark-view-tagged-vs-untagged-frames)
4. [Layer 3 Switching / Multilayer Switching](#3-layer-3-switching--multilayer-switching)
5. [Inter-VLAN Routing Methods — Full Comparison](#4-inter-vlan-routing-methods--full-comparison)
6. [Inter-VLAN Routing via SVIs](#5-inter-vlan-routing-via-svis)
7. [Step-by-Step SVI Configuration](#6-step-by-step-svi-configuration)
8. [Conditions for an SVI to Be Up/Up](#7-conditions-for-an-svi-to-be-upup)
9. [Full Configuration Listing](#8-full-configuration-listing)
10. [Verification Commands](#9-verification-commands)
11. [Common Mistakes & Gotchas](#10-common-mistakes--gotchas)
12. [Command Cheat Sheet](#11-command-cheat-sheet)
13. [Key Takeaways](#12-key-takeaways)
14. [Quick Self-Check Questions](#13-quick-self-check-questions)
15. [What's Next](#14-whats-next)

---

## 🎯 Learning Objectives

By the end of this day you should be able to:

- Configure the **native VLAN on a router's ROAS interface** using two different methods.
- Read a Wireshark capture and identify a dot1q-tagged frame versus an untagged native-VLAN frame.
- Explain what a **multilayer switch (Layer 3 switch)** is and how it differs from a regular Layer 2 switch.
- Compare all three inter-VLAN routing methods covered across Days 16–18.
- Configure a **routed port** on a Layer 3 switch.
- Configure **Switched Virtual Interfaces (SVIs)** to perform inter-VLAN routing directly on a switch.
- List the four conditions required for an SVI to reach an up/up state.

---

## 1. The Native VLAN on a Router (ROAS)

In [Day 17](../Day-17-VLANs-Part-2/README.md) we configured `R1` with three subinterfaces (`G0/0.10`, `G0/0.20`, `G0/0.30`), each tagging its own VLAN. But what if we want VLAN 10 to be the **native VLAN** on that trunk — i.e., sent/received untagged?

There are **two valid ways** to configure this on a router:

![Two methods for configuring the native VLAN on a router](images/01-native-vlan-router-methods.png)

### Method 1 — Native VLAN on a Subinterface

VLAN 10 still gets its own subinterface, `G0/0.10` — but we add the keyword **`native`** to the `encapsulation dot1q` command:

```
R1(config)# interface g0/0.10
R1(config-subif)# encapsulation dot1q 10 native
```

This tells `R1`: *"treat `G0/0.10` as the native VLAN — send its traffic untagged, and assume any untagged traffic received on `G0/0` belongs to VLAN 10."* Everything else about the subinterface (its IP address, for example) stays the same as before.

### Method 2 — IP Address Directly on the Physical Interface

Alternatively, skip the subinterface for VLAN 10 entirely, and assign the IP address straight onto the **physical** interface — which is inherently untagged:

```
R1(config)# no interface g0/0.10
R1(config)# interface g0/0
R1(config-if)# ip address 192.168.1.62 255.255.255.192
```

Both methods behave identically from the network's point of view: `SW2` sends VLAN 10 traffic untagged to `R1`, and `R1` sends VLAN 10 traffic untagged back to `SW2`.

> ⚠️ **Whichever method you choose, the native VLAN configured on the router must match the native VLAN configured on the switch's trunk port** (`switchport trunk native vlan`, from Day 17) — otherwise you get the same native VLAN mismatch problem discussed in Day 17.

---

## 2. Wireshark View: Tagged vs. Untagged Frames

Understanding this at the packet level makes native VLAN behavior click. Imagine capturing traffic on the link between `SW2` and `R1` while a PC in VLAN 20 pings a PC in VLAN 10 (which is the native VLAN in this example).

**Request — VLAN 20 (non-native) PC → R1:**

```
Ethernet II, Src: PC-E_MAC, Dst: R1_MAC
    Type: 802.1Q Virtual LAN (0x8100)
802.1Q Virtual LAN
    .000 0000 0000 0000 = Priority: 0
    .... .... .... .... = DEI: Not set
    .... 0000 0001 0100 = ID: 20
    Type: IPv4 (0x0800)
Internet Protocol Version 4
```

The Ethernet **Type** field reads `0x8100`, confirming this frame is 802.1Q-tagged, and the **VID** inside the tag reads **20** — because VLAN 20 is not the native VLAN, it must be tagged.

**Reply — R1 → VLAN 10 (native) PC:**

```
Ethernet II, Src: R1_MAC, Dst: PC-A_MAC
    Type: IPv4 (0x0800)
Internet Protocol Version 4
```

Notice there's **no 802.1Q layer at all** in this frame — the Type field goes straight to `0x0800` (IPv4). Because VLAN 10 is the native VLAN on this trunk, neither `R1` nor `SW2` bothers tagging it.

> 💡 This is also *why* the native VLAN feature exists in the first place: untagged native VLAN frames are 4 bytes smaller than tagged frames, which is a tiny efficiency gain — though the *main* reason it exists is for backward compatibility with devices that don't understand 802.1Q tagging at all.

---

## 3. Layer 3 Switching / Multilayer Switching

So far, every switch we've used (`SW1`, `SW2`) has been a plain **Layer 2 switch** — it forwards frames based on MAC addresses only, and it has no concept of IP addresses or routing.

Meet the **multilayer switch**, also called a **Layer 3 switch**:

| | Layer 2 Switch | Multilayer (Layer 3) Switch |
|---|---|---|
| Forwards frames using | MAC address table | MAC address table **and** an IP routing table |
| IP-aware? | No | Yes |
| Can run static/dynamic routing? | No | Yes — just like a router |
| Can perform inter-VLAN routing on its own? | No — must send traffic to a router | Yes, using **SVIs** or **routed ports** |

A multilayer switch does everything a regular switch does (Layer 2 forwarding, VLANs, trunking) **plus** it can route packets between subnets/VLANs at very high speed, without ever needing to send the traffic to an external router.

There are two ways to give a multilayer switch a Layer 3 (IP-routable) presence:

1. **Switched Virtual Interface (SVI):** a logical, software-based interface tied to a specific VLAN (`interface vlan <id>`). Hosts in that VLAN use the SVI's IP address as their default gateway.
2. **Routed port:** converts a **physical** switch interface from an L2 switchport into an L3, router-style interface (using `no switchport`) that you can assign an IP address to directly — just like a router's physical interface. Typically used for point-to-point links to another router or Layer 3 device, not for VLANs.

---

## 4. Inter-VLAN Routing Methods — Full Comparison

Having now covered all three approaches across this three-part series, here's the complete picture:

| # | Method | Covered in | Physical links needed | Scalability | How it works |
|---|---|---|---|---|---|
| 1 | **Legacy (multiple router interfaces)** | [Day 16](../Day-16-VLANs-Part-1/README.md) | One **access** link per VLAN | ❌ Poor — runs out of interfaces fast | Each router interface sits on an access port in a different VLAN and acts as that VLAN's default gateway |
| 2 | **Router on a Stick (ROAS)** | [Day 17](../Day-17-VLANs-Part-2/README.md) | **One trunk link** total | ⚠️ Better, but all inter-VLAN traffic funnels through one router link, which can become a bottleneck | Router subinterfaces, each tagged with `encapsulation dot1q <vlan>`, route between VLANs |
| 3 | **Multilayer switch with SVIs** | Day 18 (this page) | No dedicated router link needed for local VLANs | ✅ Best — the standard for real, growing networks | The switch itself routes between VLANs at wire speed using SVIs |

---

## 5. Inter-VLAN Routing via SVIs

### 5.1 Updating the Topology

To use SVIs, `SW2` is upgraded from a Layer 2 switch to a **multilayer switch**. We also replace the trunk link between `SW2` and `R1` with a **routed point-to-point link** — `R1` is now only used to reach *outside* the LAN (the Internet / rest of the network), not for inter-VLAN routing.

![Multilayer switch performing inter-VLAN routing with SVIs and a routed port to R1](images/02-svi-multilayer-topology.png)

We'll carve the previously-unused fourth `/26` block from Day 16 (`192.168.1.192/26`) down into a `/30` using VLSM, just for this point-to-point link:

| Link | Subnet | `SW2` G0/1 | `R1` G0/0 |
|---|---|---|---|
| `SW2` ↔ `R1` (routed) | `192.168.1.192/30` | `192.168.1.193` | `192.168.1.194` |

The SVIs on `SW2` reuse the **exact same gateway IPs** that `R1`'s subinterfaces used in Day 17 — so **no reconfiguration is needed on any PC**; their default gateway addresses stay identical:

| VLAN | SVI | IP Address |
|---|---|---|
| 10 | `interface vlan 10` | `192.168.1.62 /26` |
| 20 | `interface vlan 20` | `192.168.1.126 /26` |
| 30 | `interface vlan 30` | `192.168.1.190 /26` |

### 5.2 Worked Traffic Example — Fully Local Routing

`PC-D` (VLAN 10, on `SW2`) pings `PC-B` (VLAN 30, attached to `SW1`):

1. `PC-D` sends the frame to its gateway `192.168.1.62` — which now lives on `SW2`'s own **VLAN 10 SVI**, not on `R1`.
2. `SW2` looks up `192.168.1.128/26` in **its own routing table**, sees it's directly connected via the VLAN 30 SVI.
3. `SW2` re-encapsulates the packet as an Ethernet frame tagged VLAN 30 and forwards it out its trunk toward `SW1` (if the destination MAC isn't already known, it floods within VLAN 30 first).
4. `SW1` delivers the frame to `PC-B`.

**Notice `R1` is never involved.** This is the core advantage of SVIs over ROAS — routing between local VLANs happens entirely inside the switch, at hardware speed, with zero extra hops out to a router and back.

`R1` now comes into play **only** when a host needs to reach something outside the LAN (the Internet cloud in the diagram) — `SW2` is the default gateway for all local hosts, and it forwards anything destined outside its own subnets to `R1` via the routed point-to-point link, using a default route.

---

## 6. Step-by-Step SVI Configuration

### 6.1 Configure the Point-to-Point Link (Router Side)

First, remove the old ROAS subinterfaces from `R1` and reset the physical interface back to a plain routed interface:

```
R1(config)# no interface g0/0.10
R1(config)# no interface g0/0.20
R1(config)# no interface g0/0.30
R1(config)# default interface g0/0
R1(config)# interface g0/0
R1(config-if)# ip address 192.168.1.194 255.255.255.252
R1(config-if)# no shutdown
```

> 💡 `default interface <name>` resets an interface back to its factory-default configuration in one step — handy for clearing out old trunk/subinterface configuration before repurposing an interface.

### 6.2 Configure the Point-to-Point Link (Switch Side) + Enable Routing

```
SW2(config)# default interface g0/1
SW2(config)# ip routing
SW2(config)# interface g0/1
SW2(config-if)# no switchport
SW2(config-if)# ip address 192.168.1.193 255.255.255.252
```

> ⚠️ **`ip routing` is easy to forget — and critical.** By default, IP routing is **disabled** on a Cisco multilayer switch, even if it's fully licensed/capable of it. Without this command, the switch will not build or use a routing table, and inter-VLAN routing (even with SVIs configured) simply will not work.
>
> **`no switchport`** is the command that converts a physical switch interface from a Layer 2 switchport into a Layer 3 **routed port** — only after this can you assign it an IP address the way you would on a router interface.

### 6.3 Configure a Default Route Toward R1

So that traffic destined outside the LAN gets forwarded correctly:

```
SW2(config)# ip route 0.0.0.0 0.0.0.0 192.168.1.194
```

### 6.4 Configure the SVIs

```
SW2(config)# interface vlan 10
SW2(config-if)# ip address 192.168.1.62 255.255.255.192
SW2(config-if)# no shutdown
!
SW2(config)# interface vlan 20
SW2(config-if)# ip address 192.168.1.126 255.255.255.192
SW2(config-if)# no shutdown
!
SW2(config)# interface vlan 30
SW2(config-if)# ip address 192.168.1.190 255.255.255.192
SW2(config-if)# no shutdown
```

> ⚠️ **SVIs are administratively shut down by default** — just like router interfaces — so `no shutdown` is required on every single one, or it will never come up.

---

## 7. Conditions for an SVI to Be Up/Up

An SVI won't simply come up just because you've assigned it an IP address and typed `no shutdown`. **All four** of the following conditions must be true:

| # | Condition | Explanation |
|---|---|---|
| 1 | **The VLAN must already exist** in the switch's VLAN database | Unlike assigning an *access port* to a new VLAN (which auto-creates it — see [Day 16](../Day-16-VLANs-Part-1/README.md)), creating an SVI for a VLAN that doesn't exist does **not** auto-create the VLAN. Create it first with `vlan <id>`. |
| 2 | **At least one port in that VLAN must be up/up** | Either an access port assigned to the VLAN, or a trunk port that allows the VLAN — as long as one of them is physically up. |
| 3 | **The VLAN itself must not be administratively shut down** | You can `shutdown` a VLAN from VLAN configuration mode (`vlan <id>` → `shutdown`) — this is different from shutting down the SVI, and it will also prevent the SVI from coming up. |
| 4 | **The SVI itself must not be shut down** | SVIs are disabled by default; always run `no shutdown` after creating one. |

**Example of condition #1 failing:** if you type `interface vlan 40` and VLAN 40 was never created, the SVI's line protocol will sit in a **down/down** state indefinitely, no matter what IP address you assign or how many times you type `no shutdown` — because the underlying VLAN simply doesn't exist.

---

## 8. Full Configuration Listing

**`R1`:**

```
interface g0/0
 ip address 192.168.1.194 255.255.255.252
 no shutdown
```

**`SW2`:**

```
ip routing
!
interface g0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,30
!
interface g0/1
 no switchport
 ip address 192.168.1.193 255.255.255.252
!
interface vlan 10
 ip address 192.168.1.62 255.255.255.192
 no shutdown
!
interface vlan 20
 ip address 192.168.1.126 255.255.255.192
 no shutdown
!
interface vlan 30
 ip address 192.168.1.190 255.255.255.192
 no shutdown
!
ip route 0.0.0.0 0.0.0.0 192.168.1.194
```

**`SW1`:** *(unchanged from Day 17 — still a Layer 2 switch, still trunking VLANs 10 and 30 up to `SW2`)*

```
interface g0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,30
```

---

## 9. Verification Commands

| Command | Where | Shows |
|---|---|---|
| `show ip route` | `SW2` | Connected routes for each SVI + the default route pointing to `R1` |
| `show interfaces status` | `SW2` | The VLAN column shows **`routed`** instead of a VLAN number for a routed port |
| `show ip interface brief` | `SW2` / `R1` | Up/down status and IP address of every Layer 3 interface, including SVIs |
| `show vlan brief` | `SW2` | Confirms the VLAN exists (required for its SVI to work) |
| `show run \| section interface Vlan` | `SW2` | Quickly review just the SVI configuration block |

---

## 10. Common Mistakes & Gotchas

- ❌ **Forgetting `ip routing`.** This is the single most common reason "SVIs are configured but inter-VLAN routing still doesn't work." Without it, the switch never builds/uses a Layer 3 routing table.
- ❌ **Forgetting `no shutdown` on an SVI.** SVIs, like router interfaces, are shut down by default.
- ❌ **Assuming creating an SVI auto-creates the VLAN.** It does **not** — this is the opposite behavior from assigning an access port, which *does* auto-create a missing VLAN. Always `vlan <id>` first if you're not sure the VLAN exists.
- ❌ **Forgetting `no switchport` on a routed port.** Without it, the interface stays a Layer 2 switchport and won't accept an IP address the way a router interface does.
- ❌ **Native VLAN mismatch between the router and the switch's ROAS trunk** (if still using ROAS elsewhere) — both ends must agree on which VLAN is native.
- ❌ **Forgetting that at least one port must be up/up in a VLAN for its SVI to come up.** If every access/trunk port carrying that VLAN goes down, the SVI will follow it down too.
- ❌ **Confusing a routed port with an SVI.** A routed port is a single **physical** interface acting like a router port (point-to-point, no VLAN). An SVI is a **logical** interface tied to a VLAN, used as the gateway for many hosts in that VLAN.

---

## 11. Command Cheat Sheet

| Command | Mode | Purpose |
|---|---|---|
| `ip routing` | Global config | Enables Layer 3 routing on a multilayer switch (off by default) |
| `no switchport` | Interface config | Converts a switchport into a Layer 3 **routed port** |
| `interface vlan <id>` | Global config | Creates/enters config mode for an **SVI** |
| `no shutdown` | SVI / interface config | Required to bring the interface up (disabled by default) |
| `ip route 0.0.0.0 0.0.0.0 <next-hop>` | Global config | Configures a default route |
| `default interface <name>` | Global config | Resets an interface to its factory-default configuration |
| `encapsulation dot1q <vlan> native` | Subinterface config | Marks a router subinterface as the native VLAN for ROAS |
| `show interfaces status` | Priv. EXEC | Confirms whether a port is `routed`, `trunk`, or in a specific VLAN |

---

## 12. Key Takeaways

- A router's native VLAN can be configured two ways: with `encapsulation dot1q <id> native` on a subinterface, or by placing the IP address directly on the **physical** interface.
- In Wireshark, native VLAN traffic shows **no 802.1Q layer at all**; tagged VLAN traffic shows `Type: 802.1Q Virtual LAN (0x8100)` with a visible VID.
- A **multilayer (Layer 3) switch** can both switch (Layer 2) and route (Layer 3) — unlike a regular Layer 2 switch.
- Three inter-VLAN routing methods exist: **legacy per-VLAN links**, **ROAS**, and **SVIs on a multilayer switch** — SVIs are the standard, scalable solution used in real networks.
- **`ip routing`** must be explicitly enabled on a multilayer switch before it will route anything — it's off by default.
- **`no switchport`** turns a physical switchport into a Layer 3 **routed port**, usable for point-to-point links (e.g., to another router).
- **SVIs** (`interface vlan <id>`) give a VLAN a routable gateway IP directly on the switch — and unlike access ports, they do **not** auto-create a missing VLAN.
- An SVI needs **all four** conditions to reach up/up: VLAN exists, at least one port in that VLAN is up, the VLAN isn't shut down, and the SVI itself isn't shut down.
- With SVIs, inter-VLAN traffic between local hosts never has to leave the switch — a major performance advantage over ROAS.

---

## 13. Quick Self-Check Questions

<details>
<summary><strong>1. Name the two ways to configure the native VLAN on a router in a ROAS setup.</strong></summary>

(1) `encapsulation dot1q <vlan> native` on a subinterface, or (2) configure the IP address directly on the physical interface with no subinterface for that VLAN.
</details>

<details>
<summary><strong>2. In a Wireshark capture, how can you tell a frame belongs to the native VLAN rather than a tagged VLAN?</strong></summary>

The frame has no 802.1Q layer at all — the Ethernet Type field goes straight to the next protocol (e.g. `0x0800` for IPv4) with no `0x8100` tag present.
</details>

<details>
<summary><strong>3. What command enables Layer 3 routing on a multilayer switch?</strong></summary>

`ip routing`, entered in global configuration mode. It is disabled by default.
</details>

<details>
<summary><strong>4. Does creating an SVI for a VLAN that doesn't exist automatically create that VLAN?</strong></summary>

No. This is different from access ports. You must create the VLAN yourself first (`vlan <id>`) before its SVI can come up.
</details>

<details>
<summary><strong>5. What command converts a Layer 2 switchport into a Layer 3 routed port?</strong></summary>

`no switchport`, entered in interface configuration mode.
</details>

<details>
<summary><strong>6. List all four conditions required for an SVI to be up/up.</strong></summary>

(1) The VLAN exists in the VLAN database. (2) At least one access or trunk port carrying that VLAN is up/up. (3) The VLAN itself is not administratively shut down. (4) The SVI itself is not shut down.
</details>

<details>
<summary><strong>7. Compared to ROAS, why is SVI-based inter-VLAN routing generally faster?</strong></summary>

With ROAS, every inter-VLAN packet has to leave the switch, travel to the router, get routed, and travel back to the switch. With SVIs, the routing decision happens inside the multilayer switch itself, so local inter-VLAN traffic never has to make that extra round trip.
</details>

---

## 14. What's Next

➡️ **Day 19 — DTP & VTP:** Dynamic Trunking Protocol (automatic trunk negotiation) and VLAN Trunking Protocol (propagating VLAN databases across multiple switches).

⬅️ **[Back to Day 17 — VLANs (Part 2)](https://github.com/chakradharmannepalli/CCNA-200-301/blob/main/Day-17/README.md)**
⬅️ **[Back to Day 16 — VLANs (Part 1)](https://github.com/chakradharmannepalli/CCNA-200-301/blob/main/Day-16/README.md)**

---

<sub>Notes based on Jeremy's IT Lab — *Free CCNA 200-301 Complete Course*, Day 18: "VLANs (Part 3)". This is an independent study summary created for personal revision; all credit for the original teaching content goes to Jeremy's IT Lab.</sub>)**
⬅️ **[Back to Day 16 — VLANs (Part 1)](../Day-16-VLANs-Part-1/README.md)**

---

<sub>Notes based on Jeremy's IT Lab — *Free CCNA 200-301 Complete Course*, Day 18: "VLANs (Part 3)". This is an independent study summary created for personal revision; all credit for the original teaching content goes to Jeremy's IT Lab.</sub>
