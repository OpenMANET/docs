---
layout: default
title: Troubleshooting
nav_order: 6
permalink: /troubleshooting
description: Quick checks for mesh connectivity, hostname access via mDNS, and a recommended recovery path if setup gets messy.
---

# Troubleshooting

This page covers quick checks for common issues and the recommended “reset” workflow.

---

## Don’t Hand-Tune Settings (Reset Instead)

If something feels off after setup (odd addressing, multicast issues, services not coming up, etc.), the fastest path is usually to **start over clean** rather than changing settings outside the wizard.

OpenWrt stores configuration in its writable overlay. If you want a truly fresh install, fully wipe the SD card and re-flash:

1. Format the SD card with **SD Card Formatter**: https://www.sdcard.org/downloads/formatter/
2. In SD Card Formatter, select **Overwrite format**.
3. Re-flash the latest OpenMANET image and rerun the wizard.

---

## Connect By Hostname (mDNS / `.local`)

OpenMANET uses mDNS for easy discovery inside the mesh network (on clients that support mDNS). The simplest way to connect to different nodes is by hostname:

- If your hostname is `meshgate01`, use `meshgate01.local`
- Web UI: `https://meshgate01.local`
- SSH: `ssh root@meshgate01.local`

To browse mDNS names from a node:
- `ubus call umdns browse`
- `avahi-browse -a`

Note: `nslookup` typically will not resolve mDNS names.

---

## Power Problems Look Like “Network” Problems

A weak power supply, thin USB cable, or low batteries can cause strange, hard-to-diagnose issues (random reboots, radios dropping, services failing to start, flaky USB devices, or SD card corruption).

If you see weird behavior:
- Try a known-good power supply and cable (short, thick cable recommended).
- If you’re running on batteries/UPS, make sure they can provide enough current under load and are fully charged.

---

## Quick Mesh Sanity Checks

- If your end user device gets a `10.41.x.x` IP, it is on the mesh.
- After running the wizard, the node will reboot after address/DHCP reservation; reconnect or renew DHCP on your end user device if needed.
