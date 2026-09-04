# CCNA 2 (SRWE) — Practice PT Skills Assessment, Part 2

**Score: 97% (149/154 points)**

This repository documents my work on the CCNA 2: Switching, Routing, and Wireless
Essentials (SRWE) Practice PT Skills Assessment (PTSA) — Part 2, completed in Cisco
Packet Tracer. The activity simulates a small enterprise network (XYZ Corp.) with a
headquarters site, a remote office, and a remote branch connected over a WAN, and
asks for switch security hardening, IPv4/IPv6 static routing, DHCP, and an
enterprise wireless LAN built on a Wireless LAN Controller (WLC).

## Performance Summary

| Component                              | Earned | Max |
|-----------------------------------------|:------:|:---:|
| WLAN Controller Implementation           | 18     | 23  |
| IPv6 Static Routing Implementation       | 7      | 7   |
| DHCP and Addressing Configuration        | 10     | 10  |
| Other Switch Security Features           | 21     | 21  |
| Switch Port Security Configuration       | 93     | 93  |
| **Total**                                | **149**| **154** |

The only points dropped were on the WLC implementation section — likely a config
step I missed on the WLAN or interface setup rather than a conceptual gap, since
switch security, DHCP, and both IPv4/IPv6 routing sections scored full marks. Worth
revisiting the WLC portion in Packet Tracer to pin down exactly which check failed.

## Topology

- **RTR-HQ** — headquarters router, dual-stack (IPv4/IPv6), Ethernet + serial links
  to the branch, plus a link to a cloud/ISP router
- **RTR-Office** — remote office router, connected to RTR-HQ over a serial backup
  link and to the HQ router's Ethernet segment
- **RTR-Branch** — remote branch router, terminates a router-on-a-stick
  sub-interface for the WLAN user VLAN and hosts a DHCP-assigned WAN interface
- **SW-1** — access switch at HQ, hardened with port security, DHCP snooping,
  Dynamic ARP Inspection (DAI), and STP edge protection
- **WLC-10** — Wireless LAN Controller at the branch, providing an 802.1X/WPA2
  Enterprise WLAN authenticated against a RADIUS server
- **Cloud Router** — simulates the ISP/WAN edge

## What I Configured

### 1. Switch Security (SW-1)
- Created VLAN 10 (`users`) and VLAN 999 (`unused`) as a "parking VLAN" for
  disabled ports
- Set FastEthernet 0/1–0/5 and GigabitEthernet 0/1 as static access ports in
  VLAN 10
- Enabled port security: max 4 sticky MAC addresses per port, `restrict`
  violation mode (drop + log + alert without shutting the port), 10-minute MAC
  aging, and a manually configured static MAC on Fa0/1 for Host 1
- Enabled DHCP snooping globally and per-VLAN, rate-limited to 5 packets/sec on
  access ports, with the uplink to the router trusted
- Enabled Dynamic ARP Inspection on both VLANs, trusting the router uplink
- Applied PortFast + BPDU Guard on all active access ports to protect against
  rogue switches / STP manipulation
- Moved every unused port into VLAN 999 and administratively shut them down

### 2. Addressing & DHCP (RTR-Branch)
- Configured a router-on-a-stick sub-interface (`dot1q 10`) for the WLAN user
  VLAN with the addressing from the design table
- Built a DHCP pool (`WLAN-hosts`) for 192.168.10.0/24, excluding the router and
  WLC management addresses, and handing out default gateway + DNS
- Set the WAN-facing interface to obtain its address via DHCP from the cloud/ISP

### 3. Static Routing (RTR-HQ & RTR-Branch)
- IPv4 and IPv6 default routes preferring the Ethernet link between HQ and
  branch, with a floating (AD 10) backup default route over the serial link
- A static route to the branch's WLAN user subnet, again with a floating serial
  backup
- Host routes (IPv4 `/32` and IPv6 `/128`) to the office server, so HQ can reach
  that single host directly
- `ipv6 unicast-routing` enabled on both routers, since none of this works
  without it

### 4. Wireless LAN via WLC
- Created a dynamic interface (`WLAN 10`, VLAN 10, physical port 1) on the WLC,
  pointing at the DHCP pool configured on the branch router's sub-interface
- Added a RADIUS authentication server entry (shared secret) for enterprise
  auth
- Created WLAN `SSID-10`, bound to the new interface, secured with
  WPA2-Enterprise and 802.1X key management, and pointed at the RADIUS server
- Enabled FlexConnect Local Switching and Local Auth so the branch AP can keep
  authenticating/forwarding even if the controller link is briefly unavailable
- Added a DHCP scope for wired management devices (APs, admin hosts) in the
  192.168.100.0/24 range
- Configured an SNMP trap receiver so the WLC reports events back to the
  network management host
- Verified a wireless client (WPA2-Enterprise, PEAP, username/password) could
  associate to `SSID-10` and pull an address from the branch DHCP pool

## Key Takeaways

- Port security and STP edge protection (PortFast + BPDU Guard) are cheap
  insurance against both accidental loops and rogue devices on access ports —
  worth applying by default on any access-layer port that faces end users.
- Floating static routes are a clean way to get automatic WAN failover without
  a dynamic routing protocol, as long as you're careful with administrative
  distance on the backup path.
- WLC-based enterprise Wi-Fi has more moving parts than a standalone AP: the
  dynamic interface, the DHCP relationship with the upstream router, the RADIUS
  server entry, and the WLAN's security/AAA bindings all have to line up. That's
  also where I lost my only points this round, so it's the section I'd
  practice again first.
