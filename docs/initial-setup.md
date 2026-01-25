---
layout: default
title: Initial Setup
nav_order: 3
description: Step-by-step guide to flashing, configuring, and deploying OpenMANET on Raspberry Pi HaLow HATs.
---

# Initial Setup

This page walks you through downloading, flashing, and configuring the OpenMANET image on your Raspberry Pi HaLow HATs, plus initial mesh configuration tips and topology examples.

---

## Setup Steps

1. **Download the latest OpenMANET image**  
   Go to the Releases section: [OpenMANET Firmware Releases](https://github.com/OpenMANET/firmware/releases) and download the image.

2. **Flash the image to an SD card**  
   Use the official Raspberry Pi guide for instructions on flashing the image:  
   [Raspberry Pi Getting Started Guide](https://www.raspberrypi.com/documentation/computers/getting-started.html)

   OpenMANET release assets are published as `sysupgrade` images (`...-squashfs-sysupgrade.img.gz`) and are the images we provide for SD card flashing.

   If you want to completely start over, we recommend fully formatting the SD card with SD Card Formatter before re-flashing (OpenWrt stores config in its writable overlay, and rewriting the partition table alone doesn’t always reset everything):
   - https://www.sdcard.org/downloads/formatter/

3. **Initial connection**  
   On a fresh flash, the node boots with a static IP of `10.41.254.1`.  
   Connect your computer directly via Ethernet and set your computer to obtain an IP automatically.  
   Your computer should get an IP address in that range from the node and you will be able to access it at `http://10.41.254.1` in a web browser.  
   The default username is `root`, and the default password is blank.  
   If your computer is connected to WiFi, you can plug the RPi into your Ethernet adapter and still stay connected to the internet at the same time.  
   *Note: after running the initial wizard, you will use the password you set.*

4. **Initial configuration**  
   Follow the wizard to configure the node type (mesh gate vs mesh point) and your radio settings.

   > **Important:** After initial configuration, the node performs address/DHCP reservation and will reboot to apply it. If it does not reboot on its own, reboot it manually.
   >
   > After that reboot, you may need to release/renew DHCP (or simply reconnect) on your end user device so it gets a new lease.


---

## Mesh Gate vs. Mesh Point

When configuring 802.11s with the Seeed HaLow HAT, there are two main node types:

### Mesh Gate
Think of a Mesh Gate as the “hub” of your mesh. It’s the point where your next-hop connection (like an upstream internet connection or Starlink Mini) is attached.

- **Router Mode (required)**  
  The Mesh Gate acts as a NAT router. Its Ethernet uplink (`wan`) uses DHCP from whatever upstream network you plug into. Your HaLow mesh and local Wi-Fi AP live on the mesh (`10.41.x.x`).

### Mesh Point
A Mesh Point is a node that connects to the 802.11s mesh. It can bridge its Ethernet or 2.4/5 GHz Wi-Fi interface into the HaLow mesh.
**Recommendation:** Create a mesh gate node first before creating a mesh point node.  This will make it easier to confirm connectivity on the mesh.

If your node does not connect over HaLow, you will not be able to connect without connecting physically.

---
### Mesh SSID/Channel Consistency
For the HaLow mesh to form reliably, every HaLow radio should use the same:
- Network name (SSID / mesh ID)
- Password (if configured)
- Channel number
- Channel bandwidth

If an end user device (EUD) gets a `10.41.x.x` IP, it is on the mesh.

---
## Connecting by Hostname (mDNS)

OpenMANET supports mDNS (`.local`) for easy discovery inside the mesh network. After names propagate, you can connect to a node by hostname instead of IP:

- Web UI: `https://manet01.local`
- SSH: `ssh root@manet01.local`

To list mDNS names from a node, use `ubus call umdns browse` or `avahi-browse -a`. Note that `nslookup` typically will not resolve mDNS names.

---
### B) Mesh Gate in ROUTER Mode (own subnet, NAT to upstream, works offline)

```
           (Optional Upstream Router / Internet)
                         |
                    [ Ethernet ]
                         |
              +------------------------------------+
              |  Mesh Gate (ROUTER / NAT / DHCP/DNS|
              |  LAN = 802.11s mesh subnet         |
              +------------------------------------+
                        ))))))  802.11s  (((((( 
                 ________/         |            \________
                /                  |                       \
       +----------------+   +----------------+      +----------------+
       | Mesh Point A   |   | Mesh Point B   |      | Mesh Point C   |
       | (bridge later) |   | (bridge later) |      | (bridge later) |
       +----------------+   +----------------+      +----------------+
            |     \               |     \                   |     \
        [EUD A]  [WiFi AP]   [EUD B]  [WiFi AP]       [EUD C]  [WiFi AP]
```

**Notes:**
- Mesh Gate supplies DHCP/DNS on the mesh subnet.
- Traffic from mesh NATs to the upstream (if present).
- Works well in disconnected/off-grid scenarios; clients still have local name resolution and services.

---

## GPS Range Testing Script

A range-test script is included in the `scripts` folder. It uses the GPS module listed in the parts list to measure ping, RSSI, and SNR. You can use SCP to transfer the file to the Pi.

```bash
cp scripts/rangetest.sh /root/
chmod +x /root/rangetest.sh
```

It is recommended to run it inside `tmux` so it continues running even if you disconnect.

---
