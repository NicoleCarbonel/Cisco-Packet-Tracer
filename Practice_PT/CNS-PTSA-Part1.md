# CCNA 2: SRWE Practice PT Skills Assessment (PTSA) — Part 1

**Course:** CCNA 2 – Switching, Routing, and Wireless Essentials (SRWE) v7.0
**Tool:** Cisco Packet Tracer
**Score:** 81% (151 / 186 points)

## Overview

This lab simulates a small enterprise network with two LANs connected through an edge router and a Layer 3 switch. The goal was to configure EtherChannel, VLANs and trunking, inter-VLAN routing (both router-on-a-stick and Layer 3 switch SVI routing), and basic device hardening — then verify end-to-end connectivity across all VLANs.

## Skills Practiced

- Initial router and switch configuration (hostnames, passwords, banners, encrypted passwords)
- SSH remote access configuration on a switch
- VLAN creation and access port assignment
- EtherChannel configuration using LACP (`channel-group ... mode active`)
- Static trunking with DTP disabled (`switchport nonegotiate`)
- Inter-VLAN routing on a Layer 3 switch (SVI-based)
- Router-on-a-stick inter-VLAN routing (subinterfaces with 802.1Q encapsulation)
- Native VLAN configuration for management traffic
- Default gateway configuration on end devices

## Topology Summary

| Device | Role |
|---|---|
| Edge-Router | Router-on-a-stick, WAN link to Internet |
| L3-SW1 | Layer 3 switch, inter-VLAN routing for Sciences LAN |
| Sw-A / Sw-B | Access layer switches, Sciences LAN (VLANs 10/20/30) |
| Sw-C | Access layer switch, Arts LAN (VLANs 40/50/60) + Management VLAN 99 |

## Addressing Table

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| Edge-Router | G0/0/0 | 192.168.0.1 | 255.255.255.252 |
| Edge-Router | G0/0/1.40 | 192.168.40.1 | 255.255.255.0 |
| Edge-Router | G0/0/1.50 | 192.168.50.1 | 255.255.255.0 |
| Edge-Router | G0/0/1.60 | 192.168.60.1 | 255.255.255.0 |
| Edge-Router | G0/0/1.99 | 192.168.99.17 | 255.255.255.240 |
| Edge-Router | S0/1/0 | 209.165.201.2 | 255.255.255.252 |
| L3-SW1 | G1/1/1 | 192.168.0.2 | 255.255.255.252 |
| L3-SW1 | VLAN10 | 192.168.10.1 | 255.255.255.0 |
| L3-SW1 | VLAN20 | 192.168.20.1 | 255.255.255.0 |
| L3-SW1 | VLAN30 | 192.168.30.1 | 255.255.255.0 |
| Sw-C | VLAN99 | 192.168.99.18 | 255.255.255.240 |

## VLAN Table

| VLAN | Name | Network |
|---|---|---|
| 10 | FL1 | 192.168.10.0/24 |
| 20 | FL2 | 192.168.20.0/24 |
| 30 | FL3 | 192.168.30.0/24 |
| 40 | FAC | 192.168.40.0/24 |
| 50 | BDG5 | 192.168.50.0/24 |
| 60 | BDG6 | 192.168.60.0/24 |
| 99 | Management | 192.168.99.16/28 |

## Configuration Steps

### 1. Basic Router Configuration (Edge-Router)
- Disabled DNS lookup, set `enable secret`, console and VTY passwords, MOTD banner
- Encrypted all clear-text passwords with `service password-encryption`
- Configured IP addressing and descriptions on G0/0/0 and S0/1/0

### 2. Basic Switch Configuration (Sw-C)
- Configured SVI 99 for remote management, reachable via `ip default-gateway`
- Enabled SSH: generated 1024-bit RSA keys, set domain name `acad.pt`, created a local `admin` account, restricted VTY lines to `transport input ssh`

### 3. VLAN Configuration
- Created VLANs 10/20/30 on L3-SW1, Sw-A, Sw-B; VLANs 40/50/60/99 on Sw-C
- Assigned static access ports per the Port-to-VLAN table (e.g., Sw-A F0/7–10 → VLAN 10, F0/11–15 → VLAN 20, F0/16–24 → VLAN 30)

### 4. EtherChannel and Trunking
- Built three LACP EtherChannels (`channel-group X mode active`):
  - Channel 1: L3-SW1 (G1/0/1–2) ↔ Sw-A (G0/1–2)
  - Channel 2: L3-SW1 (G1/0/3–4) ↔ Sw-B (G0/1–2)
  - Channel 3: Sw-A (F0/5–6) ↔ Sw-B (F0/5–6)
- Set each port-channel to static trunk mode with `switchport nonegotiate` to disable DTP
- Configured Sw-C's uplink to Edge-Router as a static trunk with VLAN 99 as the native VLAN

### 5. Inter-VLAN Routing
- **L3-SW1 (SVI routing):** enabled `ip routing`, converted G1/1/1 to a routed port with `no switchport`
- **Edge-Router (router-on-a-stick):** created subinterfaces G0/0/1.40, .50, .60, .99 with 802.1Q encapsulation matching each VLAN, native VLAN 99 tagged accordingly
- Configured default gateways on all end devices (WS-1.1–1.6, WS-2.1–2.3, Management PC)

## Results

| Component | Earned | Max |
|---|---|---|
| EtherChannel Configuration | 26 | 26 |
| VLAN and Trunk Configuration | 73 | 87 |
| Inter-VLAN Routing Configuration | 30 | 38 |
| Basic Device Configuration | 22 | 25 |
| Default Gateway Configuration | 0 | 10 |
| **Total** | **151** | **186** |

## Lessons Learned

- EtherChannel and trunking configuration came out fully correct — LACP negotiation and `switchport nonegotiate` on port-channel interfaces are solid.
- Lost the full 10 points on **default gateway configuration on hosts** — the addressing table noted host IPs were preconfigured, but each end device's default gateway still needs to be set manually in the Desktop → IP Configuration tab. This is an easy point to miss under time pressure since it's a GUI step rather than a CLI command.
- Partial loss on VLAN/Trunk and Inter-VLAN Routing sections suggests double-checking `switchport trunk allowed vlan` lists and subinterface encapsulation values against the addressing table before submitting.
- **Takeaway for next attempt:** always do a final host-by-host gateway check before submission, in addition to switch/router CLI verification (`show vlan brief`, `show etherchannel summary`, `show ip route`).
