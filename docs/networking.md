---
layout: default
title: Networking
nav_order: 5
permalink: /networking
description: Explains the flat OpenMANET topology, addressing plan, and how mesh gates, mesh points, DHCP, MDNS, and BATMAN-V all fit together.
---

# Networking

OpenMANET purposely ships with an opinionated network design so that anyone—especially folks with limited networking experience—can assemble a working MANET without guesswork. The latest OpenMANET release moves every node onto a single `10.41.0.0/16` space, removes bridge mode from mesh gates, and layers **BATMAN-V** on top of **802.11s** to provide a true MANET. The result is a flat network for end users, even though the mesh is free to create multiple hops in the background.

---

## Flat Mesh Domain for Every Use

**All HaLow radios live in 10.41.0.0/16.** Each mesh point brings up `bat0` with a reserved static address (for example `10.41.113.1`) and bridges it with Ethernet and the 2.4/5 GHz AP (`br-ahwlan`). Mesh gateways are always within `10.41.0.0/24` (commonly `10.41.1.1`).

**Every mesh point runs its own DHCP server.** Sixteen leases are handed out locally per node (`start 351`, `limit 16` by default). Tablets, TAK devices, or laptops plugged in via Ethernet or Wi-Fi still receive an address when the mesh gate is offline.

**The mesh is one broadcast domain.** Because `bat0`, `eth0`, and the local AP ports sit inside the same bridge, clients see a single flat LAN no matter which node they use. This makes service discovery painless and lets multicast applications work across hops.

Example from `/etc/config/network` on a mesh point:

```sh
config device
	option name 'br-ahwlan'
	option type 'bridge'
	list ports 'eth0'
	list ports 'bat0'
```

---

## Addressing (Fresh Flash vs. After Setup)

**Fresh flash address:** a newly flashed node starts at `10.41.254.1`.

**After the initial wizard:** `openmanetd` reconciles addressing by asking other nodes to announce their IP/DHCP ranges, then reserves a static IP (and DHCP range) that is not in use. The node will reboot to apply the reserved settings.

**Client hint:** if your end user device gets a `10.41.x.x` IP, it is on the mesh. After the node reboots post-setup, you may need to release/renew DHCP (or reconnect) so your device gets a fresh lease.

**Safe static ranges:** you can safely assign static IPs within `10.41.253.0/24` and `10.41.254.0/24`. Auto addressing will not use those ranges.

---

## Mesh Gate = Router + NAT

**Router mode is mandatory.** Bridge mode and the HaLow AP wizard were removed in this release. The mesh gate always NATs the mesh into whatever upstream you plug into `eth0` (Starlink, LTE modem, hotel Wi-Fi, etc.).

**MANET vs. uplink separation.** The HaLow/BATMAN side stays on `10.41.0.0/16`. Your uplink continues to use whatever addressing the upstream network provides (DHCP on `wan`). The gate performs NAT and DNS forwarding between the two, keeping the MANET insulated from the outside network.

**Only one mesh gate today.** Multiple gates will return in a future release, but for now run a single router so default routes are predictable. Remember to reboot after running the initial configuration wizard so firewall, BATMAN, and Alfred services all restart cleanly.

---

## Client Access per Mesh Point

**Ethernet or Wi-Fi both work.** Every mesh point exposes `br-ahwlan` over the onboard Ethernet jack and an auxiliary 2.4/5 GHz AP. Pick whichever interface matches your end-user device.

**Unique SSIDs per radio.** Give each node a distinct SSID in the 2.4/5 GHz bands (e.g., `manet01-24G`, `manet02-24G`). That prevents clients from roaming to a distant node when you are diagnosing a specific radio.

**Always-on DHCP.** Because each node serves leases locally, clients retain connectivity even in a disconnected “dark” site where the mesh gate or upstream is unreachable.

**mDNS across the mesh.** OpenMANET supports mDNS reflection so you can reach nodes by name from anywhere on the mesh (on clients that support mDNS): `https://manet01.local` or `ssh root@manet01.local`.

To list mDNS names from a node:
- `ubus call umdns browse`
- `avahi-browse -a`

Note: `nslookup` typically will not resolve mDNS names.

**Hostname-aware BATMAN tools.** `batctl` commands will resolve device hostnames (instead of only MAC addresses) after a few minutes for names to propagate.

---

## BATMAN-V Adds the MANET Brain

**True MANET routing.** BATMAN-V monitors link quality per hop, redistributes neighbors, and reroutes automatically as nodes move. Your clients keep a flat IP experience while the mesh dynamically rebuilds paths.

**Bonding-ready.** Switching from BATMAN_IV to BATMAN_V opens the door to uplink bonding (HaLow + Wi-Fi or multiple USB Ethernet adapters) in future builds.

**Multicast friendly.** New firewall rules plus the BATMAN multicast optimizations improve reliability for TAK, ADSBCOT, and MDNS traffic.

**Second batman-adv interface.** A second batman-adv interface is included for future link bonding work.

---

## Why This Design?

This scheme intentionally hides complexity:

- Flat addressing and MDNS avoid the need for static routes or manual DNS.
- Local DHCP on every mesh point ensures a client-friendly experience even when nodes are disconnected from the mesh gate.
- NAT on the single mesh gate keeps the MANET isolated from whatever upstream you plug into it, ensuring the upstream network does not need any config changes.

Flash **all nodes** when upgrading to the latest OpenMANET so every device shares the same addressing plan and BATMAN-V stack.

---

## Troubleshooting

### See What BATMAN Thinks Is On The Mesh

Use `batctl dc` to view the distributed ARP table. This is a quick way to confirm which nodes/clients are currently visible on the mesh, along with their IPs and “last seen” time (and, after hostname propagation, their hostnames):

```sh
root@manet01:~# batctl dc
[B.A.T.M.A.N. adv 2024.3-openwrt-6, MainIF/MAC: wlan0/2c:c6:82:8a:2b:ca (bat0/9a:c2:84:47:71:98 BATMAN_V)]
Distributed ARP Table (bat0):
          IPv4             MAC        VID   last-seen
 *       10.41.0.1 manet01_br-ahwlan   -1      0:09
 *    10.41.88.231 manet02_br-ahwlan   -1      1:54
 *     10.41.0.111 0e:12:80:92:f7:dc   -1      0:39
 *   10.41.254.229 manet03_br-ahwlan   -1      0:38
```

In the output above, entries ending in `_br-ahwlan` are mesh nodes; entries shown as raw MAC addresses are typically end user devices behind a node.

### Browse mDNS Services

Use `avahi-browse -a` to see mDNS-advertised hostnames and services currently visible on the mesh:

```sh
root@manet01:~# avahi-browse -a
+ br-ahwlan IPv6 manet02                                       _ssh._tcp            local
+ br-ahwlan IPv4 manet03                                       _ssh._tcp            local
+ br-ahwlan IPv4 manet02                                       _ssh._tcp            local
+ br-ahwlan IPv6 manet02                                       _http._tcp           local
+ br-ahwlan IPv4 manet03                                       _http._tcp           local
+ br-ahwlan IPv4 manet02                                       _http._tcp           local
+ br-ahwlan IPv6 manet02 [c6:2a:60:bd:a5:6e]                   _workstation._tcp    local
+ br-ahwlan IPv4 manet03 [b2:95:f1:69:d9:0a]                   _workstation._tcp    local
+ br-ahwlan IPv4 manet03 [b8:27:eb:6c:45:ac]                   _workstation._tcp    local
+ br-ahwlan IPv4 manet02 [c6:2a:60:bd:a5:6e]                   _workstation._tcp    local
+ br-ahwlan IPv6 MacBook Air (3)                               _companion-link._tcp local
+ br-ahwlan IPv4 MacBook Air (3)                               _companion-link._tcp local
+ br-ahwlan IPv6 7A55CB537445@MacBook Air (3)                  _raop._tcp           local
+ br-ahwlan IPv4 7A55CB537445@MacBook Air (3)                  _raop._tcp           local
+ br-ahwlan IPv6 MacBook Air (3)                               _airplay._tcp        local
+ br-ahwlan IPv4 MacBook Air (3)                               _airplay._tcp        local
```

---

## Quick Reference

| Component   | Key Details |
|-------------|-------------|
| Fresh Flash | Node starts at `10.41.254.1` before the wizard |
| Mesh Gate   | Router-only, NAT enabled, static IP in `10.41.0.0/24`, DHCP/DNS for the mesh, WAN via DHCP |
| Mesh Point  | Reserved static `10.41.xxx.1`, local DHCP, bridges `bat0` + `eth0` + local AP |
| Client Link | Connect via Ethernet or unique 2.4/5 GHz SSID on each node |
| Discovery   | mDNS (`hostname.local`) + service announcements across the mesh |
| Routing     | 802.11s + BATMAN-V for MANET resiliency and future uplink bonding |
