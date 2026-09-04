# SRWE (Version 7.00) — Final PT Skills Assessment (PTSA)

**Score: 100% (224/224 points)**

This repository documents my work on the CCNA 2: Switching, Routing, and Wireless
Essentials (SRWE) Final Packet Tracer Skills Assessment. Unlike the earlier practice
assessments, this one is built entirely in **PT Physical Mode** — no logical
topology view, so every device has to be racked, cabled, and configured through a
direct console connection, the way it would be done on real hardware.

## Performance Summary

| Component                              | Earned | Max |
|-----------------------------------------|:------:|:---:|
| Network Building                         | 13     | 13  |
| VLAN and Trunk Configuration             | 67     | 67  |
| EtherChannel Configuration               | 8      | 8   |
| Initial IOS Device Configuration         | 63     | 63  |
| Device Security Configuration            | 8      | 8   |
| Secure Remote Access Configuration       | 29     | 29  |
| Host Computer Addressing Configuration   | 2      | 2   |
| Router Subinterface Configuration        | 13     | 13  |
| Router Interface Configuration           | 9      | 9   |
| Static Route Configuration               | 2      | 2   |
| Router DHCP Server Configuration         | 10     | 10  |
| **Total**                                | **224**| **224** |

## Topology

- **R1** — single router with a Loopback0 interface and a trunked G0/0/1 link
  carrying four sub-interfaces (VLANs 2, 3, 4, and the native VLAN 6)
- **S1** — access switch, connects to R1's trunk, to S2 via an EtherChannel, and
  to PC-A
- **S2** — access switch, connects to S1 via the same EtherChannel and to PC-B
- **PC-A / PC-B** — end hosts, one per switch, each in a different data VLAN

| VLAN | Name       | Router Sub-interface |
|------|------------|-----------------------|
| 2    | Bikes      | G0/0/1.2               |
| 3    | Trikes     | G0/0/1.3               |
| 4    | Management | G0/0/1.4               |
| 5    | Parking    | — (no router interface)|
| 6    | Native     | G0/0/1.6                |

## What I Configured

### 1. Physical Build (PT Physical Mode)
- Racked R1, S1, and S2; placed PC-A and PC-B on the table
- Cabled everything with copper straight-through, matching the topology diagram
- Powered on all devices and connected via the rear console port to begin
  configuration — no logical-view shortcuts

### 2. Device Hardening (R1, S1, S2)
- Disabled DNS lookup of mistyped commands (`no ip domain lookup`)
- Set hostnames and a MOTD banner warning against unauthorized access
- Console password + `login`, `enable secret`, and `service password-encryption`
  to protect against clear-text password exposure
- Enforced a minimum password length of 10 characters on R1

### 3. Secure Remote Access (SSH, all three devices)
- Created a local admin user (secret, not clear-text password)
- Set the IP domain name and generated a 1024-bit RSA key pair
- Forced SSHv2 only, and restricted the vty lines to `login local` +
  `transport input ssh` — no Telnet fallback

### 4. Router Interfaces & Sub-interfaces (R1)
- Configured Loopback0 with both an IPv4 address and an IPv6 GUA + explicit
  link-local
- Enabled `ipv6 unicast-routing`
- Built router-on-a-stick sub-interfaces for VLANs 2, 3, and 4, each with
  IPv4 + IPv6 addressing and a description
- Configured the native VLAN (6) sub-interface with 802.1Q native tagging, and
  brought up the parent physical interface

### 5. VLANs, Trunking & EtherChannel (S1, S2)
- Created VLANs 2–6 with matching names on both switches
- Trunked F0/1–F0/2 (and F0/5 on S1) as 802.1Q trunks, restricting allowed
  VLANs and setting VLAN 6 as native
- Bundled F0/1–F0/2 into a Layer 2 EtherChannel (LACP `active` on both ends)
  for redundant, load-shared inter-switch bandwidth
- Set the host-facing ports (S1 → PC-A in VLAN 2, S2 → PC-B in VLAN 3) as
  static access ports with port security capped at 3 learned MAC addresses
- Swept every unused port into the VLAN 5 "parking" VLAN, labeled them, and
  shut them down

### 6. Switch Management (SVIs)
- Configured a VLAN 4 SVI on each switch with the management addressing from
  the design table, plus an `ip default-gateway` pointing at R1's management
  sub-interface

### 7. Static Routing & DHCP (R1)
- IPv4 and IPv6 default routes pointed at the Loopback0 interface (a common
  PTSA pattern for simulating an "exit to the rest of the world" without a
  second router)
- Two DHCP pools (`CCNA-A` for VLAN 2, `CCNA-B` for VLAN 3), each scoped to
  the last 10 host addresses of its subnet, with the correct default gateway
  and a domain name
- Verified PC-A and PC-B could each pull an IPv4 address via DHCP while using
  statically assigned IPv6 GUAs and gateways

## Key Takeaways

- Working entirely in PT Physical Mode is a good discipline check — you can't
  fall back on the logical topology to double-check cabling, so getting the
  physical connections right the first time matters.
- EtherChannel plus 802.1Q trunking is a clean way to get both redundancy and
  aggregate bandwidth between two switches without needing STP to block a
  redundant link — LACP handles the negotiation as long as both ends agree on
  mode and allowed VLANs.
- Pointing a default route at a loopback interface is a PTSA-specific trick
  for simulating "somewhere out there" without a real upstream router — worth
  remembering it's a lab convention, not something you'd do in production.
- Locking down remote access (SSH-only, local auth, RSA key, no Telnet) and
  encrypting stored passwords are cheap, repeatable steps that should be part
  of every device's baseline config, not just assessments.

## Files

- `SRWE_Final_PTSA.pkt` — Packet Tracer source file *(add when uploading)*
- Config excerpts for R1, S1, and S2 *(optional: add as separate `.txt` files
  alongside this README if you want the raw CLI output preserved)*

---
*Part of my CCNA v7 (SRWE) coursework, Polytechnic University of the Philippines.*
