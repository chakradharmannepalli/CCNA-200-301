# Day 16 — VLANs (Part 1)

> **Course:** Jeremy's IT Lab — *CCNA 200-301 Complete Course* (Free)
> **Topic:** Broadcast domains, what a VLAN is, and how to configure VLANs + access ports
> **Video reference:** `Free CCNA | VLANs (Part 1) | Day 16` + `Day 16 Lab`
> **Series:** Part 1 of 3 → continues in [Day 17](../Day-17-VLANs-Part-2/README.md) and [Day 18](../Day-18-VLANs-Part-3/README.md)

---

## 📑 Table of Contents

1. [Learning Objectives](#-learning-objectives)
2. [Before VLANs: LANs and Broadcast Domains](#1-before-vlans-lans-and-broadcast-domains)
3. [The Problem VLANs Solve](#2-the-problem-vlans-solve)
4. [What Is a VLAN?](#3-what-is-a-vlan)
5. [Switchport Types](#4-switchport-types)
6. [Default VLANs on a Cisco Switch](#5-default-vlans-on-a-cisco-switch)
7. [The Lab Topology Used in This Series](#6-the-lab-topology-used-in-this-series)
8. [Step-by-Step Configuration](#7-step-by-step-configuration)
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

- Explain what a **broadcast domain** is and count how many exist in a given topology.
- Explain, in plain language, **why** a single flat LAN becomes a problem as a network grows.
- Define a **VLAN** and explain how it solves that problem at Layer 2.
- Describe the difference between an **access port** and a **trunk port** (conceptually — full trunk configuration is Day 17).
- List which VLANs exist **by default** on a Cisco switch and which ones can never be deleted.
- Create VLANs, name them, and assign **access ports** to them using the Cisco IOS CLI.
- Verify your configuration with `show vlan brief`.

---

## 1. Before VLANs: LANs and Broadcast Domains

### 1.1 What Is a LAN?

A **Local Area Network (LAN)** is a group of devices — PCs, servers, printers, switches — connected together in a small geographical area (an office, a floor, a building) that can communicate directly with one another at Layer 2.

> **Key definition:** A LAN is a single broadcast domain, including every device inside it.

### 1.2 What Is a Broadcast Domain?

A **broadcast domain** is the set of devices that will receive a **broadcast frame** — a frame whose destination MAC address is `FFFF.FFFF.FFFF` — sent by any single member of that group.

Two rules govern broadcast domains:

| Device | Forwards broadcast frames? |
|---|---|
| **Switch (Layer 2)** | ✅ Yes — floods the broadcast out of *every other* interface in the same VLAN |
| **Router (Layer 3)** | ❌ No — a router receives a broadcast but does **not** forward it out of another interface |

This single fact — *routers stop broadcasts, switches don't* — is the foundation for everything else in this lesson.

### 1.3 Worked Example — Counting Broadcast Domains

![Broadcast domain example with two routers and three switches](images/01-broadcast-domains.png)

In the diagram above:

- `PC6`, `PC7`, `PC8`, and `SW3` all sit behind `R2`. If `PC6` sends a broadcast, `SW3` floods it to `PC7` and `PC8`, but **`R2` does not forward it onward to `R1`.** → **1 broadcast domain.**
- The point-to-point link between `R1` and `R2` is its own separate broadcast domain (only those two router interfaces are in it).
- `SW1` with `PC1`/`PC2` behind `R1` is another broadcast domain.
- `SW2` with `PC3`/`PC4`/`PC5` behind `R1` is another broadcast domain.

**Total: 4 broadcast domains.** Notice that switches never reduced the count — only the routers created new boundaries.

---

## 2. The Problem VLANs Solve

Imagine a small company with three departments — **Engineering**, **HR**, and **Sales** — all plugged into **one switch**, using the address block `192.168.1.0/24`.

If a PC in Engineering sends a broadcast, the switch floods it out of **every** port — HR and Sales receive it too, along with the router. This hurts:

- **Performance** — unnecessary broadcast traffic is flooded everywhere, which wastes bandwidth and CPU cycles on every host, even as the network grows.
- **Security** — because every device sits in the same Layer 2 domain, hosts can talk to each other directly without ever passing through a router/firewall where you could apply security policies.

### "Just subnet it" doesn't fully fix this

A very common misunderstanding: *if I put each department in its own IP subnet, doesn't that solve the problem?*

**No — not by itself.** Subnetting only separates devices at **Layer 3**. A switch only understands **Layer 2** (MAC addresses); it has no idea that `192.168.1.1` and `192.168.1.129` are supposed to be in different subnets. If both are connected to plain access ports on the same switch with no VLANs configured, a broadcast frame (destination MAC `FFFF.FFFF.FFFF`) will still be flooded to **every** port on that switch, regardless of IP subnet.

So even after subnetting:

- Unicast traffic between subnets is forced through the router (good — that's where you can apply ACLs).
- But broadcast/unknown-unicast traffic is **still flooded to everyone** at Layer 2, because the switch is still one big broadcast domain.

**The real fix has to happen at Layer 2 — and that's exactly what a VLAN does.**

---

## 3. What Is a VLAN?

### 3.1 Definition

A **Virtual LAN (VLAN)** is a logical grouping of switch ports that creates **multiple separate broadcast domains** on top of a **single physical switch** (or across multiple switches).

> Put simply: **VLANs logically separate a switch's LANs at Layer 2** — the same way subnets logically separate networks at Layer 3.

### 3.2 Key Properties

| Property | Detail |
|---|---|
| Layer | VLANs operate at **Layer 2** of the OSI model |
| Where configured | On Layer 2 (or Layer 3) **switches**, on a **per-interface** basis |
| Effect | Each VLAN behaves as its own isolated broadcast domain |
| Cross-VLAN traffic | A switch will **never** forward traffic directly between two different VLANs — it must be sent to a router (or a Layer 3 switch — see [Day 18](../Day-18-VLANs-Part-3/README.md)) |

### 3.3 Benefits

- **Performance:** broadcasts/unknown-unicasts sent inside VLAN 10 stay inside VLAN 10 — they never reach VLAN 20 or VLAN 30.
- **Security:** because hosts in different VLANs cannot reach each other without passing through a router, you can apply security policies (ACLs, firewalls) at that routing point — and they will actually be effective, unlike in a single flat broadcast domain.
- **Logical organization:** you can group users by department, function, or traffic type (e.g. a dedicated voice VLAN) regardless of their physical location or cabling.

### 3.4 A Small Proof: `ping 255.255.255.255`

If you send a **directed local broadcast** (`ping 255.255.255.255`) from a PC in VLAN 10, only the *other* hosts configured in VLAN 10 receive it — hosts in VLAN 20 and VLAN 30, even though they're plugged into the exact same physical switch, never see that broadcast. This is the clearest possible demonstration that VLANs create real, separate broadcast domains.

---

## 4. Switchport Types

Every switch interface that participates in VLANs is configured as one of two basic types:

| Type | Also known as | Belongs to | Tags frames? | Typically connects to |
|---|---|---|---|---|
| **Access port** | *Untagged port* | A **single** VLAN | ❌ No | End hosts (PCs, printers, servers) |
| **Trunk port** | *Tagged port* | **Multiple** VLANs | ✅ Yes (except the native VLAN) | Other switches, routers (ROAS), firewalls |

> 📌 **This lesson (Day 16) only covers access ports.** Trunk ports, 802.1Q tagging, and the native VLAN are covered in full in **[Day 17](../Day-17-VLANs-Part-2/README.md)**.

An **access port**:

- Belongs to exactly one VLAN.
- Does **not** insert a VLAN tag into frames — the connected device (a normal PC's NIC) doesn't need to understand VLANs at all.
- Gives the end host "access" to the network — hence the name.

---

## 5. Default VLANs on a Cisco Switch

Run `show vlan brief` on a factory-default switch and you'll already see VLANs defined:

| VLAN ID | Default Name | Purpose | Can it be deleted? |
|---|---|---|---|
| **1** | `default` | Every switchport is a member of VLAN 1 until you configure it otherwise | ❌ No |
| **1002** | `fddi-default` | Legacy FDDI | ❌ No |
| **1003** | `token-ring-default` | Legacy Token Ring | ❌ No |
| **1004** | `fddinet-default` | Legacy FDDI | ❌ No |
| **1005** | `trnet-default` | Legacy Token Ring | ❌ No |

> ⚠️ **Important:** VLANs 1002–1005 exist for legacy technologies (FDDI and Token Ring) that you will never encounter in a modern network and do not need to know in detail for the CCNA. Just remember: **VLAN 1 and VLANs 1002–1005 exist by default and cannot be deleted.**

VLAN 1 is not just "a VLAN" — by default it is also the **default native VLAN** on trunk ports and is used for switch management traffic on many devices out of the box. Because of this, best practice in real networks is to **avoid using VLAN 1 for user data** (more on this in Day 17).

### A quick preview of the VLAN numbering range

The full range of VLAN IDs is discussed in depth in Day 17 (it's tied to the size of the 802.1Q tag field), but as a preview:

- Usable VLAN range: **1 – 4094**
- **Normal range:** 1 – 1005
- **Extended range:** 1006 – 4094

---

## 6. The Lab Topology Used in This Series

To keep these notes concrete, Days 16–18 all build on **one running example** — a small company LAN using `192.168.1.0/24`, split into three VLANs using **VLSM** (see the earlier Subnetting days):

| Department | VLAN ID | VLAN Name | Subnet | Usable Range | Gateway (last usable IP) |
|---|---|---|---|---|---|
| Engineering | **10** | `ENGINEERING` | `192.168.1.0/26` | `.1 – .62` | `192.168.1.62` |
| HR | **20** | `HR` | `192.168.1.64/26` | `.65 – .126` | `192.168.1.126` |
| Sales | **30** | `SALES` | `192.168.1.128/26` | `.129 – .190` | `192.168.1.190` |

*(The fourth `/26` block, `192.168.1.192/26`, is reserved for later use — see Day 18, where it's further subnetted into a `/30` for a router-to-switch link.)*

**Today's (Day 16) topology** is intentionally simple: **one switch (`SW1`)**, three VLANs, all interfaces are **access ports**, and — for now — there are three separate physical links from the switch up to the router (one per VLAN). This is inefficient (and gets replaced by a single trunk link in Day 17), but it's the clearest way to first understand what a VLAN actually does.

![One switch split into three VLANs with legacy per-VLAN router links](images/02-vlan-segmentation.png)

---

## 7. Step-by-Step Configuration

### 7.1 Check the Current State

Always start by seeing what already exists:

```
SW1# show vlan brief
```

On a fresh switch, you'll only see the default VLANs (1, 1002–1005), with every interface listed under VLAN 1.

### 7.2 Assign Access Ports to a VLAN (this also creates the VLAN)

You do **not** have to manually create a VLAN before assigning ports to it. If you assign an interface to a VLAN that doesn't exist yet, the switch automatically creates it for you.

```
SW1(config)# interface range fa0/1 - 2
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 10
% Access VLAN does not exist. Creating vlan 10
```

Repeat for the other departments:

```
SW1(config)# interface range fa0/3 - 4
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 20

SW1(config)# interface range fa0/5 - 6
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 30
```

> 💡 **Tip:** `switchport mode access` and `switchport access vlan <id>` are **two different commands that do two different things**:
> - `switchport mode access` → sets the **port type** (access vs. trunk).
> - `switchport access vlan <id>` → sets **which VLAN** that access port belongs to.
>
> You can technically skip `switchport mode access` on a port whose default negotiated mode is already access, but Jeremy's IT Lab (and Cisco best practice) recommends **always configuring it explicitly** rather than relying on auto-negotiation.

### 7.3 Manually Create a VLAN and Give It a Name

Because VLANs 10, 20, and 30 were auto-created above, they currently have generic default names like `VLAN0010`. Let's fix that by entering VLAN configuration mode:

```
SW1(config)# vlan 10
SW1(config-vlan)# name ENGINEERING
SW1(config-vlan)# exit

SW1(config)# vlan 20
SW1(config-vlan)# name HR
SW1(config-vlan)# exit

SW1(config)# vlan 30
SW1(config-vlan)# name SALES
SW1(config-vlan)# exit
```

> Note: `vlan <id>` is also the command you'd use to create a VLAN **before** assigning any interfaces to it — that's a perfectly valid (and often cleaner) order of operations too.

### 7.4 Verify

```
SW1# show vlan brief
```

```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0, Gi0/1, ...
10   ENGINEERING                      active    Fa0/1, Fa0/2
20   HR                               active    Fa0/3, Fa0/4
30   SALES                            active    Fa0/5, Fa0/6
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

---

## 8. Full Configuration Listing

Putting it all together for `SW1`:

```
hostname SW1
!
vlan 10
 name ENGINEERING
!
vlan 20
 name HR
!
vlan 30
 name SALES
!
interface range fa0/1 - 2
 switchport mode access
 switchport access vlan 10
!
interface range fa0/3 - 4
 switchport mode access
 switchport access vlan 20
!
interface range fa0/5 - 6
 switchport mode access
 switchport access vlan 30
!
end
```

---

## 9. Verification Commands

| Command | What it shows |
|---|---|
| `show vlan brief` | Every VLAN on the switch, its name, and the **access ports** assigned to it (does **not** show trunk ports or their allowed VLANs — see Day 17) |
| `show vlan` | The same info as above, plus more detail (MTU, SAP, STP type, etc.) |
| `show interfaces status` | Port-by-port status, speed/duplex, and which VLAN each access port belongs to |
| `show running-config interface <if>` | Confirms the exact `switchport` configuration applied to one interface |

---

## 10. Common Mistakes & Gotchas

- ❌ **Forgetting `switchport mode access`.** If a port's mode is ambiguous or influenced by Dynamic Trunking Protocol (DTP, covered Day 19), it might not behave as a plain access port. Always set it explicitly.
- ❌ **Confusing `switchport access vlan` with `vlan <id>`.** The first command is run in **interface** configuration mode and assigns a port to a VLAN. The second is run in **global** configuration mode and creates/edits the VLAN itself. They are not interchangeable.
- ❌ **Trying to delete VLAN 1 or VLANs 1002–1005.** The switch will not allow this — these VLANs are permanent.
- ❌ **Assuming subnetting alone isolates broadcast traffic.** As explained in [Section 2](#2-the-problem-vlans-solve), without VLANs, a switch floods broadcasts to every port regardless of IP subnet.
- ❌ **Expecting `show vlan brief` to show trunk ports.** It only lists **access** ports per VLAN. To see trunk configuration, you need `show interfaces trunk` (Day 17).
- ❌ **Forgetting that hosts in different VLANs can never talk to each other without a router/L3 device** — even if, for some reason, they happened to be configured in the same IP subnet. VLAN separation always wins; a switch will never bridge traffic across VLANs on its own.

---

## 11. Command Cheat Sheet

| Command | Mode | Purpose |
|---|---|---|
| `show vlan brief` | Privileged EXEC | List VLANs and their access ports |
| `vlan <vlan-id>` | Global config | Create a VLAN / enter VLAN config mode |
| `name <vlan-name>` | VLAN config | Name the current VLAN |
| `interface <type><number>` | Global config | Enter config mode for one interface |
| `interface range <type><start> - <end>` | Global config | Enter config mode for several interfaces at once |
| `switchport mode access` | Interface config | Set the port as an access port |
| `switchport access vlan <vlan-id>` | Interface config | Assign the access port to a VLAN (auto-creates the VLAN if needed) |

---

## 12. Key Takeaways

- A **broadcast domain** is the group of devices that receive a broadcast frame from any one member. **Routers stop broadcasts; switches flood them.**
- Subnetting alone (Layer 3) does **not** stop broadcast flooding on a switch — the switch still floods at Layer 2 regardless of IP addressing.
- A **VLAN** logically splits a single physical switch (or group of switches) into multiple separate Layer 2 broadcast domains.
- VLANs improve both **network performance** (less unnecessary broadcast traffic) and **security** (forces inter-VLAN traffic through a router where policy can be applied).
- **Access ports** belong to exactly one VLAN and do not tag frames — they're for end hosts.
- VLAN **1** and VLANs **1002–1005** exist by default on every Cisco switch and cannot be deleted.
- Assigning an interface to a VLAN that doesn't exist **auto-creates** that VLAN.
- `show vlan brief` only shows **access** port membership, not trunk configuration.

---

## 13. Quick Self-Check Questions

<details>
<summary><strong>1. What is the difference between a broadcast domain and a collision domain?</strong></summary>

A broadcast domain is bounded by Layer 3 devices (routers) and defines how far a broadcast frame travels. A collision domain (relevant mainly to old hub-based/half-duplex networks) is the set of devices that could cause a collision if they transmit at the same time — every switch port is its own collision domain in a modern full-duplex network.
</details>

<details>
<summary><strong>2. If a switch has no VLANs configured, how many broadcast domains does it represent?</strong></summary>

One. All ports are members of the default VLAN (VLAN 1) unless configured otherwise.
</details>

<details>
<summary><strong>3. True or False: Subnetting a switch's hosts into separate IP subnets is enough to stop broadcast traffic from one department reaching another.</strong></summary>

False. The switch operates at Layer 2 and has no awareness of IP subnets. Without VLANs, it will still flood broadcast/unknown-unicast frames out of every port.
</details>

<details>
<summary><strong>4. Which command both creates VLAN 30 and assigns an interface to it in a single step?</strong></summary>

`switchport access vlan 30`, run in interface configuration mode — if VLAN 30 doesn't already exist, the switch creates it automatically.
</details>

<details>
<summary><strong>5. Can VLAN 1 be deleted from a Cisco switch?</strong></summary>

No. VLAN 1, along with VLANs 1002–1005, is a default VLAN that cannot be deleted.
</details>

---

## 14. What's Next

➡️ **[Day 17 — VLANs (Part 2)](../Day-17-VLANs-Part-2/README.md):** trunk ports, 802.1Q tagging, the native VLAN, and Router on a Stick (ROAS) for inter-VLAN routing.

---

<sub>Notes based on Jeremy's IT Lab — *Free CCNA 200-301 Complete Course*, Day 16: "VLANs (Part 1)". This is an independent study summary created for personal revision; all credit for the original teaching content goes to Jeremy's IT Lab.</sub>
