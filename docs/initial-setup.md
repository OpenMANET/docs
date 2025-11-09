---
layout: default
title: Initial Setup
nav_order: 2
description: Step-by-step guide to flashing, configuring, and deploying OpenMANET on Raspberry Pi HaLow HATs.
---

# Initial Setup

This page walks you through downloading, flashing, and configuring the OpenMANET image on your Raspberry Pi HaLow HATs, plus initial mesh configuration tips and topology examples.

---

## Setup Steps

1. **Download the latest OpenMANET image**  
   Go to the Releases section: [OpenMANET OpenWrt Releases](https://github.com/OpenMANET/openwrt/releases) and download the image.

2. **Flash the image to an SD card**  
   Use the official Raspberry Pi guide for instructions on flashing the image:  
   [Raspberry Pi Getting Started Guide](https://www.raspberrypi.com/documentation/computers/getting-started.html)

3. **Initial connection**  
   When first powered on, the Pi boots with a static IP of `10.42.0.1`.  
   Connect your computer directly via Ethernet and set your computer to obtain an IP automatically.  
   Your computer should get an IP address in that range from the Pi and you will be able to access the Pi at `10.42.0.1` in a web browser.  
   The default username is `root`, and the default password is blank.  
   If your computer is connected to WiFi, you can plug the RPi into your Ethernet adapter and still stay connected to the internet at the same time.  
   *Note: after running the initial wizard, you will use the password you set.*

4. **Initial configuration**  
   Folow the wizard to cofigure the mesh gate and mesh points.


   *Reboot the Pi after flashing the image to fix multicast issues*


---

## Mesh Gate vs. Mesh Point

When configuring 802.11s with the Seeed HaLow HAT, there are two main node types:

### Mesh Gate
Think of a Mesh Gate as the “hub” of your mesh. It’s the point where your next-hop connection (like an upstream internet connection or Starlink Mini) is attached. Mesh Gates have two operating modes:

- **Router Mode**  
  The Mesh Gate acts as its own NAT router. The mesh network gets its own subnet, and traffic is NAT’d to the upstream network. The Mesh Gate also runs DHCP and DNS, allowing your Raspberry Pis to resolve each other by hostname. This option is best for disconnected environments where there’s no upstream network or internet.

### Mesh Point
A Mesh Point is a node that connects to the 802.11s mesh. It can bridge its Ethernet or 2.4/5 GHz Wi-Fi interface into the HaLow mesh.
**Recommendation:** Create a mesh gate node first before creating a mesh point node.  This will make it easier to confirm connectivity on the mesh.

If your node does not connect over HaLow, you will not be able to connect without connecting physically.

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