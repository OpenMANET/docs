---
layout: default
title: Networking
nav_order: 4
permalink: /networking
description: Explains the flat OpenMANET topology, addressing plan, and how mesh gates, mesh points, DHCP, MDNS, and BATMAN-V all fit together.
---

# Networking

OpenMANET purposely ships with an opinionated network design so that anyone—especially folks with limited networking experience,can assemble a working MANET without guesswork. The latest OpenMANET release moves every node onto a single `10.41.0.0/16` space, removes bridge mode from mesh gates, and layers **BATMAN-V** on top of **802.11s** to provide a true MANET. The result is a flat network for end users, even though the mesh is free to create multiple hops in the background.

---

## Flat Mesh Domain for Every Use

**All HaLow radios live in 10.41.0.0/16.** Each mesh point brings up `bat0` with a randomly selected static address like `10.41.113.1` and bridges it with Ethernet and the 2.4/5 GHz AP (`br-ahwlan`). The mesh gate reserves `10.41.1.1/16`.

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

## Mesh Gate = Router + NAT

**Router mode is mandatory.** Bridge mode and the HaLow AP wizard were removed in this release. The mesh gate always NATs the mesh into whatever upstream you plug into `eth0` (Starlink, LTE modem, hotel Wi-Fi, etc.).

**MANET vs. uplink separation.** The HaLow/BATMAN side stays on `10.41.0.0/16`. Your uplink continues to use whatever addressing the upstream network provides (DHCP on `wan`). The gate performs NAT and DNS forwarding between the two, keeping the MANET insulated from the outside network.

**Only one mesh gate today.** Multiple gates will return in a future release, but for now run a single router so default routes are predictable. Remember to reboot after running the initial configuration wizard so firewall, BATMAN, and Alfred services all restart cleanly.

---

## Client Access per Mesh Point

**Ethernet or Wi-Fi both work.** Every mesh point exposes `br-ahwlan` over the onboard Ethernet jack and an auxiliary 2.4/5 GHz AP. Pick whichever interface matches your end-user device.

**Unique SSIDs per radio.** Give each node a distinct SSID in the 2.4/5 GHz bands (e.g., `manet01-24G`, `manet02-24G`). That prevents clients from roaming to a distant node when you are diagnosing a specific radio.

**Always-on DHCP.** Because each node serves leases locally, clients retain connectivity even in a disconnected “dark” site where the mesh gate or upstream is unreachable.

**MDNS across the mesh.** Avahi and Alfred advertise hostnames via multicast, so `hostname.local` lookups work on any node. This is how OpenMANET keeps SSH and web access simple without manual host files.

---

## BATMAN-V Adds the MANET Brain

**True MANET routing.** BATMAN-V monitors link quality per hop, redistributes neighbors, and reroutes automatically as nodes move. Your clients keep a flat IP experience while the mesh dynamically rebuilds paths.

**Bonding-ready.** Switching from BATMAN_IV to BATMAN_V opens the door to uplink bonding (HaLow + Wi-Fi or multiple USB Ethernet adapters) in future builds.

**Multicast friendly.** New firewall rules plus the BATMAN multicast optimizations improve reliability for TAK, ADSBCOT, and MDNS traffic.

---

## Why This Design?

This scheme intentionally hides complexity:

- Flat addressing and MDNS avoid the need for static routes or manual DNS.
- Local DHCP on every mesh point ensures a client-friendly experience even when nodes are disconnected from the mesh gate.
- NAT on the single mesh gate keeps the MANET isolated from whatever upstream you plug into it, ensuring the upstream network does not need any config changes.

Flash **all nodes** when upgrading to the latest OpenMANET so every device shares the same addressing plan and BATMAN-V stack.

---

## Quick Reference

| Component   | Key Details |
|-------------|-------------|
| Mesh Gate   | Router-only, NAT enabled, static `10.41.1.1/16`, DHCP/DNS for the mesh, WAN via DHCP |
| Mesh Point  | Random static `10.41.xxx.1`, 16-client DHCP pool, bridges `bat0` + `eth0` + local AP |
| Client Link | Connect via Ethernet or unique 2.4/5 GHz SSID on each node |
| Discovery   | MDNS + Alfred broadcast hostnames/services across the mesh |
| Routing     | 802.11s + BATMAN-V for MANET resiliency and future uplink bonding |
