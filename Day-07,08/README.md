# IPv4 Addressing (Part 1)

Hey! Welcome back to the world of networking. If you're prepping for CCNA or just curious about how the internet knows where to send your cat videos, this is your spot. We'll unpack **IPv4 addressing** at the **OSI Network Layer (Layer 3)**—the "traffic cop" that connects distant networks. I'll keep it conversational, with simple examples, bite-sized math, and visuals described (since those diagrams are gold). No jargon overload; think of this as notes from a chill study session.

By the end, you'll nail why `192.168.1.100` and `192.168.1.105` are neighbors, but `192.168.2.50` lives next door.

## Layer 3: Bridging Worlds Beyond Your Wi-Fi Bubble

Picture your home network as a backyard BBQ (your **LAN**—Local Area Network). **Switches** (Layer 2 heroes) keep the party flowing: They connect devices *inside* the same network, expanding it without drama. But to invite the whole neighborhood (or the world)? Enter **Layer 3**.

- **Key Powers**:
  - Links end devices across *different* networks (bye-bye LAN limits!).
  - Hands out **logical addresses** (IP addresses—like unique party invites).
  - Chooses the smartest route from **source** (your laptop) to **destination** (that server in Seattle).
  - **Routers** rule here: They inspect IPs and forward packets like a GPS on steroids.

**Routing in Action**: Without a router, all your devices share one big network (e.g., `192.168.1.0/24`). Add a router between two switches? *Split city*. Now you've got two networks: `192.168.1.0/24` and `192.168.2.0/24`. It's like dividing the BBQ into two patios—still connected via the router's "door."
<img width="1133" height="387" alt="image" src="https://github.com/user-attachments/assets/2f8316ac-f658-4f9a-99cf-26784c27b8fd" />


*(Visual: Imagine a diagram with PCs (PC1-PC4) plugged into two switches (SW1, SW2), linked by a router (R1). Left side: Network `192.168.1.0/24` with PCs at .1 and .2. Right: `192.168.2.0/24` with .1 and .2. Router interfaces glow: G0/0 at `192.168.1.254/24`, G0/1 at `192.168.2.254/24`. Arrows show traffic flow.)*
<img width="1140" height="380" alt="image" src="https://github.com/user-attachments/assets/088559c1-0828-41bb-9681-6126af4220ed" />

Routers play both sides: Each interface gets an IP matching its attached LAN. Why `.254`? It's the "last house on the block"—easy default gateway for hosts to find the exit.

**Broadcast Bonus**: Shouts like ARP requests ("Who's got this MAC?") stay local. Routers block them from crossing borders, saving bandwidth.

## IPv4: The Packet's ID Card

IPv4 is the OG Layer 3 protocol—still powering most nets today. Addresses are 32 bits (4 octets), dotted like `192.168.1.254`.

- **Binary Breakdown**: `11000000.10101000.00000001.11111110` (each octet = 8 bits, 0-255 decimal).
- We use dotted decimal 'cause binary's a headache for humans.

**The Header** (packet's "envelope"):
- More fields than Ethernet (Layer 2): Version (4), TTL (hop limit), Protocol (next layer, e.g., TCP=6).
- Stars: **Source IP** and **Destination IP** (32 bits each).
<img width="1137" height="459" alt="image" src="https://github.com/user-attachments/assets/a037cf37-173a-4707-9f72-92c9068af931" />

*(Visual: A detailed table of the IPv4 header. Rows for octets 0-5 (up to 60 bytes total). Columns: Bit positions 0-31. Highlights: Version/IHL (bits 0-7), Total Length (16-23), Source IP (96-127), Destination IP (128-159). Options pad if needed. Clean grid with labels like "Time to Live" at octet 8.)*

## Number Bases: Decimal, Hex, and Binary Basics (No Sweat)

Quick refresher—networks love these for conversions.

- **Decimal (Base 10)**: Everyday math. 3294 = 3×1000 + 2×100 + 9×10 + 4×1.
- **Hex (Base 16)**: Compact for big numbers (A=10, B=11, ..., F=15). 3294 = CDE (C×256=3072 + D×16=208 + E=14).
  <img width="1140" height="718" alt="image" src="https://github.com/user-attachments/assets/3da49da8-d982-442f-b4fa-abafc6a3cbd9" />


*(Visual: Side-by-side. Left: Decimal breakdown of 3294. Right: Hex CDE with multipliers: C(12)×256=3072, D(13)×16=208, E(14)×1=14. Totals match.)*

Now, binary (base 2)—IPs' secret sauce. Positions: 128, 64, 32, 16, 8, 4, 2, 1 (powers of 2, right to left).

### Binary → Decimal (Add the 1s)
`10001111`:
- 1×128 + 0×64 + 0×32 + 0×16 + 1×8 + 1×4 + 1×2 + 1×1 = **143**

`01110110` → **118**  
`11101100` → **236**

### Decimal → Binary (Subtract & Slot)
For 221: Start high.
- ≥128? Yes (1), remainder 93 → `1_______`
- ≥64? Yes (1), 29 → `11______`
- ≥32? No (0) → `110_____`
- ≥16? Yes (1), 13 → `1101____`
- ≥8? Yes (1), 5 → `11011___`
- ≥4? Yes (1), 1 → `110111__`
- ≥2? No (0) → `1101110_`
- ≥1? Yes (1) → `11011101`

127 = `01111111` (all 1s from 64 down).  
207 Hack: 255-207=48 (`00110000`), flip 0s/1s → `11001111`.

## /24 and Friends: Network vs. Host Magic

`192.168.1.254/24`? The `/24` = **CIDR prefix**—first 24 bits are **network** (shared "street"), last 8 = **host** (your "house number").

- All on `192.168.1.x` share the network—hosts vary (.100, .105, .205).
- Mask: `/24` = `255.255.255.0` (binary: 24 1s, 8 0s).
<img width="1132" height="524" alt="image" src="https://github.com/user-attachments/assets/8fe58f28-7189-4161-8753-2fb05c18b95e" />

<img width="1136" height="557" alt="image" src="https://github.com/user-attachments/assets/3fbabdcd-8d7f-42fe-9b55-4242bd48bbed" />

*(Visual: Binary string `11000000.10101000.00000001.11111110` split into octets, arrows to decimal `192.168.1.254/24`. Orange highlights network bits.)*

*(Another: Network diagram labeling all IPs: PCs at .1/.2, router at .254 per side. Shows how hosts fit into network portions.)*

**Binary to IP Practice**:
- `10011010010011100110111100100000` → Octets: 154 | 78 | 111 | 32 → `154.78.111.32/16` (Class B: Net=154.78, Host=111.32).
- `00001100100000001111101100010111` → 12 | 128 | 251 | 23 → `12.128.251.23/8` (Class A: Net=12, Host=128.251.23).

## Classes: IPv4's Family Tree

Old-school way to divvy addresses by first octet (leading bits). Focus: A-C, but let's touch on D and E too—they're special snowflakes.

| Class | Leading Bits | First Octet Range | Default Prefix | Networks | Hosts/Network | Vibe |
|-------|--------------|-------------------|----------------|----------|---------------|------|
| **A** | 0xxxxxxx    | 1-126*           | /8            | 128     | 16M          | Giant (ISPs) |
| **B** | 10xxxxxx    | 128-191          | /16           | 16K     | 65K          | Medium biz |
| **C** | 110xxxxx    | 192-223          | /24           | 2M      | 254          | Home/office |
| **D** | 1110xxxx    | 224-239          | N/A           | N/A     | N/A          | Multicast groups (e.g., video streams to multiple devices) |
| **E** | 1111xxxx    | 240-255          | N/A           | N/A     | N/A          | Experimental/research (future-proofing or lab tests) |

*Skip 0 (invalid); **127.x.x.x** = loopback (self-test, like `ping 127.0.0.1` checks your device's network stack—no wires needed).
<img width="1131" height="347" alt="image" src="https://github.com/user-attachments/assets/96ea6c05-2bac-457b-b65c-62b1fa237958" />
<img width="1120" height="615" alt="image" src="https://github.com/user-attachments/assets/966d4e06-4920-4d04-8a7c-5e35db92f527" />

- **Class D Deep Dive**: No network/host split here—these are for **multicasting**. Think one-to-many broadcasts without flooding everyone (e.g., `224.0.0.1` for all hosts on a subnet, or `239.x.x.x` for private groups). Routers use IGMP to manage who joins. Saves bandwidth for streaming or updates—your Netflix binge might use these under the hood.
- **Class E Deep Dive**: Reserved for R&D. Not routable on the public internet; they're like a sandbox for testing new protocols. If you're tinkering with wild ideas (e.g., quantum networking experiments), this is your range. No assignments, no drama.

Shorter prefix = bigger host pool (A: tiny net, huge party). Longer = cozy (C: many nets, few per net). D/E skip the party altogether.

*(Visual: Table with leading bits, ranges, prefixes. Another: Deeper stats—Class A: 2^7 networks, 2^24 hosts (minus 2 specials). Pie chart vibes for D/E: "Multicast" icon for D, lab beaker for E.)*

**Masks in Binary**:
- A (/8): `255.0.0.0` → `11111111.00000000.00000000.00000000`
- B (/16): `255.255.0.0` → `11111111.11111111.00000000.00000000`
- C (/24): `255.255.255.0` → `11111111.11111111.11111111.00000000`
  <img width="1138" height="635" alt="image" src="https://github.com/user-attachments/assets/78e0f020-6a26-4317-bea0-3dc25500bbe0" />


*(Visual: Clean list with binary under each dotted mask.)*

## Special IPs: The Untouchables

- **Network Address**: Host bits all 0s → Identifies the net (e.g., `192.168.1.0/24`). No host gets this—it's the "street sign."
  <img width="1135" height="529" alt="image" src="https://github.com/user-attachments/assets/b4d26b93-b8b4-4bd6-8b5e-65e119f0a6a7" />

- **Broadcast**: All 1s → "Hey team!" (e.g., `192.168.1.255`). MAC: `FF:FF:FF:FF:FF:FF`. Also off-limits.
<img width="1136" height="507" alt="image" src="https://github.com/user-attachments/assets/68280a3f-280e-46b2-b94c-a9f3f3afc6a6" />

Usable: `.1` to `.254` (256 total - 2 specials).

*(Visual: Diagram highlighting network addresses in red boxes: `192.168.1.0/24` and `192.168.2.0/24`. Text: "Host portion all 0s = Network Address (CANNOT assign to host).")*

*(Broadcast version: Same net, but focus on all-1s implication, with warning text.)*

## Level Up: Practice Time!

Grab a coffee and try: Convert `10101010.00001111.11000000.00110011` to decimal IP. (/16? Class B net.) Or sketch your home router setup.

This sets the stage—next: Subnetting deep dive. Questions? Hit me up. You've got the basics; now own the net! 🌐

# IPv4 Addressing (Part 2)

Hey again! If Part 1 got you hooked on how IPs organize the networking neighborhood (routers splitting streets, classes like family trees), welcome to **Part 2**. We're zooming in on the nitty-gritty: How many devices can crash a single network party? What's the "first and last seat" for hosts? And how do you actually *set this up* on a Cisco router using CLI commands—like giving your gear a street address and turning on the lights.

Think of this as upgrading from a house sketch to wiring the lights. We'll keep it breezy with examples, a handy formula, and step-by-step CLI walkthroughs (no copy-paste needed—just understand the flow). By the end, you'll confidently config a router interface.

Ready? Let's subnet like pros. (P.S. If you missed Part 1, hop back for the basics!)

## Party Size: Max Hosts Per Network

Every network has a "guest list limit"—determined by how many bits are left for **hosts** (after the network portion claims its share). More host bits = bigger bash. But subtract 2 seats: One for the network ID (the "venue address") and one for broadcast (the "last call" shout).

**The Magic Formula**:  
**Max Hosts = 2<sup>N</sup> - 2**  
*(N = number of host bits. Easy calc: Double the previous power of 2, minus 2.)*

Here's the breakdown by class (defaults from Part 1):

| Class | Default Prefix | Host Bits (N) | Total Addresses | Max Usable Hosts | Real-World Vibe |
|-------|----------------|---------------|-----------------|------------------|-----------------|
| **A** | /8            | 24            | 16,777,216     | 16,777,214      | Mega-corp HQ (think Google's campus—millions of devices? No sweat.) |
| **B** | /16           | 16            | 65,536         | 65,534          | University network (enough for every student, prof, and coffee machine). |
| **C** | /24           | 8             | 256            | 254             | Home office or small team (your Wi-Fi + smart fridge = cozy). |

**Quick Examples**:
- **Class C** (`192.168.1.0/24`): Full range `.0` to `.255` (256 spots). Minus network (`.0`) and broadcast (`.255`) → **254 hosts**. Perfect for a family router.
- **Class B** (`172.16.0.0/16`): Ranges from `172.16.0.0` to `172.16.255.255` → **65,534 hosts**. (2^16 = 65,536 - 2.)
- **Class A** (`10.0.0.0/8`): `10.0.0.0` to `10.255.255.255` → **16,777,214 hosts**. (2^24 is huge—ISPs love this.)

**Pro Tip**: In real life, we subnet (slice these further) to avoid wasting space. But for now? This is the raw capacity.

*(Visual: A fun bar graph—Class A as a skyscraper, B as a mid-rise, C as a bungalow. Labels: "2^N Total" and "-2 for Specials = Usable Hosts." Side note: "Why -2? Network ID can't be a host; broadcast is for group yells!")*

## VIP Seats: First and Last Usable Addresses

Hosts get the comfy middle chairs—*not* the ends. First usable? Network ID +1. Last? Broadcast -1. (All in binary: Flip the host bits from all-0s or all-1s.)

**Class C** (`192.168.1.0/24`):
- Network: `192.168.1.0` (host bits: `00000000` → Reserved).
- **First Usable**: `192.168.1.1` (host: `00000001`).
- Broadcast: `192.168.1.255` (host: `11111111` → Reserved).
- **Last Usable**: `192.168.1.254` (host: `11111110`).

**Class B** (`172.16.0.0/16`):
- Network: `172.16.0.0` (host bits: `00000000 00000000`).
- **First**: `172.16.0.1` (host: `00000000 00000001`).
- Broadcast: `172.16.255.255` (host: `11111111 11111111`).
- **Last**: `172.16.255.254` (host: `11111111 11111110`).

**Class A** (`10.0.0.0/8`):
- Network: `10.0.0.0` (host: `00000000 00000000 00000000`).
- **First**: `10.0.0.1` (host: `00000000 00000000 00000001`).
- Broadcast: `10.255.255.255` (host: `11111111 11111111 11111111`).
- **Last**: `10.255.255.254` (host: `11111111 11111111 11111110`).

**Hack**: To find these fast, just bump/decrement the last octet (or relevant ones) by 1. No binary needed unless subnetting gets wild.

*(Visual: Side-by-side timelines for each class. Dots for addresses: Red X on network/broadcast, green checks on first/last. Arrows: "+1 from Net" and "-1 from Broadcast." Example binary snippets under each.)*

## Hands-On: Configuring Cisco Devices with CLI

Theory's cool, but nothing beats typing commands. Cisco's CLI (Command Line Interface) is like the router's brain—text-based, efficient, and a CCNA must. We'll config a router interface (e.g., G0/0) with an IP, mask, and bring it online.

**Start Here**: Assume you're on a Cisco router (R1). Boot up in User EXEC mode (`R1>`).

### Quick Recon: `show ip interface brief`
- **What it shows**: A snapshot table—interfaces, IPs, assignment method (manual/DHCP?), **Status** (Layer 1: physical link), **Protocol** (Layer 2: logical up?).
- **Key Decodes**:
  - **Status**: `up` (good) or `administratively down` (shut off via command—*default for router ports*). Switch ports? Up by default.
  - **Protocol**: `up` if Layer 2 is chatting; down if physical (Status) fails.
- Example Output:
  
| Interface            | IP-Address  | OK? | Method | Status                  | Protocol |
|----------------------|-------------|-----|---------|--------------------------|-----------|
| GigabitEthernet0/0   | unassigned  | YES | unset  | administratively down    | down      |
| GigabitEthernet0/1   | unassigned  | YES | unset  | administratively down    | down      |

*(Visual: Clean table screenshot. Highlights: "Admin down? Wake it with 'no shutdown'!" Red for down, green for up. Notes: "Routers start sleepy—switches don't.")*

### Config Mode: Assign IP and Wake It Up
Enter **Global Config** mode, then drill to the interface. Shortcuts like `g0/0` save time.
```
R1# conf t          // Global config mode (short: configure terminal)
R1(config)# interface g0/0  // Target the port (GigabitEthernet0/0)
R1(config-if)# ip address 10.255.255.254 255.0.0.0  // IP + mask (Class A example—last usable!)
R1(config-if)# no shutdown  // Flip the switch: Up!
```

- **Output Magic**: You'll see `%LINK-3-UPDOWN: Interface GigabitEthernet0/0... changed state to up` (Status). Then `%LINEPROTO-5-UPDOWN: Line protocol on Interface... changed state to up` (Protocol). Boom—online!
  <img width="1125" height="295" alt="image" src="https://github.com/user-attachments/assets/6673a77c-f29d-48d5-8a05-2119aad6d1f5" />


**Verify**: From config mode, sneak a peek:
```
R1(config-if)# do show ip interface brief  // 'do' runs EXEC commands without exiting
```
(Now your table shows the IP, Status/Protocol: up/up. High five!)
<img width="1139" height="536" alt="image" src="https://github.com/user-attachments/assets/0bc9b06e-acf3-4a10-93ae-182d5dd10009" />


### Pro Tips & More Show Commands
- **Descriptions**: Label ports for sanity (e.g., "To the wild internet?").
  ```
    R1(config)# interface g0/0
    R1(config-if)# description ## Upstream to ISP ##
  ```
- Check: `show interfaces description` → Adds a "Description" column: `Gi0/0 | ## Upstream to ISP ##`.

- **Deeper Dives**:
- `show interfaces g0/0`: Full stats—MAC (hardware ID), IP, errors, speed/duplex. (Layer 1-3 intel.)
- `show running-config interface g0/0`: Your current setup (saved changes?).

*(Visual: CLI transcript screenshot—step-by-step commands with green success messages. Another: `show interfaces description` table: Interface | IP | Description | Status | Protocol. Example row: Gi0/0 | 10.255.255.254 | ## To SW1 ## | up | up.)*

<img width="833" height="796" alt="image" src="https://github.com/user-attachments/assets/d287918e-66cb-41fc-b2fe-fb19c5f3d8d8" />

**Safety Note**: `shutdown` to pause an interface (reverse of `no shutdown`). Always save with `wr` (write memory) to persist.

## Level Up: Your Turn!

Fire up Packet Tracer (free Cisco sim) and try: Config R1's G0/0 with `192.168.1.1/24`, no shutdown, then ping a fake host at `.2`. Spot the first/last usable? Calc hosts for /16.
