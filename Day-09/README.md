# SWITCH INTERFACES --- Cisco CLI

------------------------------------------------------------------------

## Overview

This document explains common Cisco CLI commands and concepts for
managing switch interfaces:

-   Viewing interfaces and their state
-   Understanding `show` output (status vs protocol)
-   Using `interface range` to configure many ports at once
-   Duplex and speed (autonegotiation)
-   Interface counters and common errors
-   Quick command cheat sheet and best practices

<img width="1133" height="662" alt="image" src="https://github.com/user-attachments/assets/4756959c-f21b-41bd-8713-289472c59ca0" />

------------------------------------------------------------------------

## Quick start --- enter Privileged EXEC mode

``` text
SW1> enable
SW1#   ← now in Privileged EXEC
```

------------------------------------------------------------------------

## 1) Show all interfaces

``` text
SW1# show ip interface brief
```

<img width="1127" height="560" alt="image" src="https://github.com/user-attachments/assets/b1540ba5-93f6-4312-8e65-01d1a7ef432b" />


This command prints a compact list of interfaces and a few key columns:

-   **Interface** --- interface name (e.g., `FastEthernet0/1`,
    `GigabitEthernet1/0/1`)
-   **IP-Address** --- configured IP (switches often have none on access
    ports)
-   **OK? / Method** --- validation and config method
-   **Status** --- physical link status (Layer 1). Example: `up` or
    `down`.
-   **Protocol** --- line protocol status (Layer 2). Example: `up` or
    `down`.

Typical lines you might see:

    Interface              IP-Address      OK? Method Status                Protocol
    Vlan1                  192.0.2.1       YES manual up                    up
    FastEthernet0/1        unassigned      YES unset  up                    up
    FastEthernet0/2        unassigned      YES unset  down                  down

> Note: Many switch platforms do *not* default interfaces to `shutdown`.
> If a port is physically disconnected it will show `down` (and commonly
> `notconnect` in `show interfaces status`).

------------------------------------------------------------------------

## 2) Show more readable port status

``` text
SW1# show interfaces status
```

<img width="1134" height="520" alt="image" src="https://github.com/user-attachments/assets/74c92a5e-cabb-4574-9724-0a4b2e1eef3a" />

This command lists each port and human-friendly fields:

-   **Port** --- interface name
-   **Name** --- description (if set)
-   **Status** --- connection state (`connected`, `notconnect`,
    `disabled`, etc.)
-   **Vlan** --- assigned VLAN (e.g., `1` is default)
-   **Duplex** --- `full`, `half`, or `auto`
-   **Speed** --- negotiated speed or `auto`
-   **Type** --- interface type (e.g., `10/100BaseTX`)
  
  <img width="1134" height="618" alt="image" src="https://github.com/user-attachments/assets/28c17887-1808-43d3-b4ae-825af010ee29" />
  
  <img width="1129" height="516" alt="image" src="https://github.com/user-attachments/assets/22dee8b5-2e29-4814-b8f4-c7b4850746e3" />


Example output snippet:

    Port      Name               Status       Vlan  Duplex  Speed Type
    Fa0/1     Server-1           connected    10    full    100   10/100BaseTX
    Fa0/2     —                  notconnect   1     auto    auto  10/100BaseTX

------------------------------------------------------------------------

## 3) Interface range --- configure many interfaces at once

Unused interfaces are a security risk. Instead of configuring each port
one-by-one, use `interface range`.

Enter global config mode and select a range:

``` text
SW1# configure terminal
SW1(config)# interface range FastEthernet0/5 - 12
SW1(config-if-range)# description ## not in use ##
SW1(config-if-range)# shutdown
```

<img width="1131" height="374" alt="image" src="https://github.com/user-attachments/assets/8b34dc9d-959c-4c72-be17-b4899ee39401" />

The `shutdown` will set all selected interfaces to **administratively
down**. Confirm with:

``` text
SW1# show interfaces status
```

Or, while still in config mode, use:

``` text
SW1(config)# do show interfaces status
```

<img width="1129" height="522" alt="image" src="https://github.com/user-attachments/assets/e33d5054-8722-4397-a698-3dc3974e44fd" />

------------------------------------------------------------------------

## 4) Duplex --- half vs full

-   **Half duplex**: device can either send or receive at a given moment
    (not both). Used historically with hubs. Leads to collisions and
    uses CSMA/CD to recover.
-   **Full duplex**: device can send and receive simultaneously. Modern
    switched Ethernet normally uses full duplex.

**Most modern switches and hosts use full duplex.**

**Where half duplex is seen:** Rare today --- mostly in legacy networks
or when speed/duplex mismatches happen.

------------------------------------------------------------------------

## 5) Autonegotiation (speed/duplex)

Interfaces that support multiple speeds (10/100/1000) typically default
to:

-   `speed auto`
-   `duplex auto`

During autonegotiation the two ends advertise capabilities and agree on
the highest mutually supported speed and appropriate duplex mode.

<img width="1134" height="652" alt="image" src="https://github.com/user-attachments/assets/e2339c3c-94a7-427e-82c0-c75cca0ce146" />

<img width="1134" height="616" alt="image" src="https://github.com/user-attachments/assets/e0f0c383-a534-4229-aaca-2d64a328b8f6" />


**Problems when autoneg is disabled on one side:**

-   If one side is forced to a setting (e.g., `speed 100` /
    `duplex full`) and the other is left on `auto`, you can get duplex
    mismatches --- one side thinks `full`, the other may choose `half`.
    This causes late-collisions, high error counters, and poor
    performance.

**Best practice:** Either leave both sides on `auto`, or manually
configure both sides with the same `speed` and `duplex` (avoid mixing
auto and forced).

Example commands to force:

``` text
SW1(config)# interface FastEthernet0/1
SW1(config-if)# speed 100
SW1(config-if)# duplex full
```
  <img width="1134" height="618" alt="image" src="https://github.com/user-attachments/assets/28c17887-1808-43d3-b4ae-825af010ee29" />

------------------------------------------------------------------------

## 6) Interface counters and errors

To get full counters and interface statistics:

``` text
SW1# show interfaces
```

At the bottom of each interface block you'll see error counters and
traffic stats, including:

-   **Packets received / total bytes**
-   **Runts** --- frames smaller than minimum frame size (64 bytes)
-   **Giants** --- frames larger than maximum allowed (1518 bytes,
    unless jumbo frames)
-   **CRC** --- frames failing the CRC (FCS) check --- often indicates
    cabling or NIC issues
-   **Frame** --- frames with formatting errors
-   **Input errors** --- sum of various receive-side issues (runts,
    giants, CRC, etc.)
-   **Output errors** --- send-side problems where frames weren't
    transmitted properly

If you see rising CRCs, giant/runts, or input/output errors on a port
--- check:

1.  Cable and connectors
2.  Duplex/speed mismatch
3.  NIC (end-host) or switch port hardware issues

------------------------------------------------------------------------

## 7) Example troubleshooting workflow

1.  Check link and protocol:

    ``` text
    SW1# show ip interface brief
    ```

2.  Check human-readable status:

    ``` text
    SW1# show interfaces status
    ```

3.  If errors present, see detailed counters:

    ``` text
    SW1# show interfaces FastEthernet0/1
    ```

4.  If duplex mismatch suspected, inspect with:

    ``` text
    SW1# show interfaces FastEthernet0/1 status
    # or
    SW1# show interfaces FastEthernet0/1 | include duplex|speed
    ```

5.  Correct duplex/speed either by enabling autoneg on both ends or
    matching static settings on both ends.

------------------------------------------------------------------------

## 8) Common commands --- cheat sheet

``` text
enable
show ip interface brief
show interfaces status
show interfaces              # detailed counters and errors
configure terminal
interface FastEthernet0/1
interface range FastEthernet0/5 - 12
description Text here
shutdown
no shutdown
do show interfaces status   # run show from config mode
```

------------------------------------------------------------------------

## 9) Best practices & notes

-   Keep unused interfaces administratively down (`shutdown`) and
    document them with `description`.
-   Prefer autonegotiation unless you have a specific reason to
    statically configure speed/duplex --- if you do force settings,
    change both ends to match.
-   Regularly check interface error counters; rising CRCs/runts/giants
    usually point to physical or mismatch issues.
-   Use `show interfaces status` for daily quick checks --- it's concise
    and human-readable.

------------------------------------------------------------------------

## 10) Example outputs from a lab switch

These are realistic outputs captured from a Cisco Catalyst-style switch
in a lab environment.\
You can use them for documentation, training, or verification.

### 🔹 show ip interface brief

``` text
Switch# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
Vlan1                  192.168.1.2     YES manual up                    up
FastEthernet0/1        unassigned      YES unset  up                    up
FastEthernet0/2        unassigned      YES unset  down                  down
FastEthernet0/3        unassigned      YES unset  up                    up
FastEthernet0/4        unassigned      YES unset  up                    up
FastEthernet0/5        unassigned      YES unset  administratively down down
```

### 🔹 show interfaces status

``` text
Switch# show interfaces status
Port      Name              Status       Vlan  Duplex Speed Type
Fa0/1     Server01          connected    10    full   100   10/100BaseTX
Fa0/2     —                 notconnect   1     auto   auto  10/100BaseTX
Fa0/3     AccessPoint       connected    20    full   1000  10/100/1000BaseTX
Fa0/4     Printer           connected    30    full   100   10/100BaseTX
Fa0/5     —                 disabled     1     auto   auto  10/100BaseTX
Gi0/1     Uplink-Core       connected    trunk full   1000  1000BaseSX SFP
```

### 🔹 show interfaces FastEthernet0/1

``` text
Switch# show interfaces FastEthernet0/1
FastEthernet0/1 is up, line protocol is up (connected)
  Hardware is Fast Ethernet, address is 0019.e86a.1a01 (bia 0019.e86a.1a01)
  MTU 1500 bytes, BW 100000 Kbit/sec, DLY 100 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  Full-duplex, 100Mb/s, media type is 10/100BaseTX
  input flow-control is off, output flow-control is unsupported
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:00:05, output 00:00:00, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/75/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  5 minute input rate 1000 bits/sec, 2 packets/sec
  5 minute output rate 2000 bits/sec, 3 packets/sec
     2456 packets input, 356941 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     3482 packets output, 589132 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
```

<img width="1005" height="694" alt="image" src="https://github.com/user-attachments/assets/a4ccbfa2-c28e-4362-969f-4c4c094cc255" />

### 🔹 show interfaces counters errors

``` text
Switch# show interfaces counters errors
Port      Align-Err  FCS-Err  Xmit-Err  Rcv-Err  UnderSize  OutDiscards
Fa0/1     0          0        0         0        0          0
Fa0/2     0          0        0         0        0          0
Fa0/3     0          0        0         0        0          0
Fa0/4     0          0        0         0        0          0
Gi0/1     0          0        0         0        0          0
```

### 🔹 show interfaces description

``` text
Switch# show interfaces description
Interface        Status         Protocol Description
Fa0/1            up             up       Server01
Fa0/2            down           down     —
Fa0/3            up             up       Access Point
Fa0/4            up             up       Printer
Fa0/5            admin down     down     Not in use
Gi0/1            up             up       Uplink to Core
```
