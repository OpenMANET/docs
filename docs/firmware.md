---
layout: default
title: Firmware & Releases
nav_order: 2
permalink: /firmware
description: Firmware download guidance, naming conventions, and notable changes in recent OpenMANET releases.
---

# Firmware & Releases

Firmware images are published on GitHub: [OpenMANET OpenWrt Releases](https://github.com/OpenMANET/openwrt/releases)

This page covers how to choose the correct image and summarizes notable changes in the `1.6.x` line.

---

## Choosing the Right Image

OpenMANET firmware filenames include the information you need to select the correct image:

- SBC/target (example: `rpi4`, `rpi3`)
- Morse Micro chipset family (example: `mm6108`, `mm8108`)
- HaLow interface (example: `spi`, `sdio`)

> **Important:** Select your firmware downloads carefully. If you are using a Seeed Studio HaLow board, you typically want an image with `spi` in the name.

### HaLow Interface Guidance (SPI vs SDIO)

- `spi` images are intended for SPI-based HaLow HATs (most commonly the Seeed WM1302 + Wio-WM6108 setups).
- `sdio` images are intended for SDIO-based HaLow modules (for example Silex or Alfa SDIO boards).

> Note: SDIO images can work across different SDIO boards, but the board configuration file (BCF) must match your specific radio/module. The default BCF is tuned for an Alfa board; for other SDIO boards you will need to obtain the correct BCF from the manufacturer and apply it.

### SBC Target Mapping

Use this as a quick guide for the `rpiX` part of the filename:

| SBC | Use firmware with |
|-----|-------------------|
| Raspberry Pi 4 / CM4 | `rpi4` |
| Raspberry Pi 3B | `rpi3` |
| Raspberry Pi Zero 2 W (Pi2W) | `rpi3` |

---

## Image Type: `sysupgrade`

OpenMANET release assets are published as `sysupgrade` images (for example: `...-squashfs-sysupgrade.img.gz`).

- You can use these images for upgrades from within OpenWrt (LuCI or `sysupgrade`), and they are also what we publish for flashing to an SD card.
- OpenWrt stores configuration in its writable overlay; changing/rewriting the SD card partition table alone may not remove all prior state. If you are trying to completely start over, fully wipe the SD card before re-flashing.

### Starting Over (Recommended SD Card Wipe)

If you want a truly fresh install, we recommend fully formatting the SD card with SD Card Formatter before re-flashing:

- https://www.sdcard.org/downloads/formatter/

---

## Notable Changes (1.6.x)

- OpenWrt `24.10` base
- Linux kernel `6.6.104`
- `mac80211` `6.12.6` (backported Wi‑Fi drivers)
- Morse Micro drivers `1.16`
- `openmanetd` (beta) manages low-level configuration and auto addressing
- Expanded filesystem increased to `4GB`
- Morse Micro MM6108 and MM8108 drivers enabled on all targets
- Built-in Wi‑Fi support on SPI-based nodes (client AP mode)
- Most Wi‑Fi driver kernel modules included by default; additional packages available via `opkg`
- Custom packages migrated to `openmanet/packages` and included via `feed.conf`
- Improved automatic GPSd configuration for Seeed-based nodes

### Networking & Discovery

- mDNS support with reflection for easy mesh-wide discovery
  - Use `https://<hostname>.local` or `ssh root@<hostname>.local`
  - Example: `https://manet01.local`, `ssh root@manet01.local`
  - mDNS resolution only works inside the OpenMANET network and on clients that support mDNS
  - `nslookup` typically will not resolve mDNS names
  - To browse names from a node: `ubus call umdns browse` or `avahi-browse -a`
- `batctl` commands resolve hostnames (not only MAC addresses) once names propagate (can take a few minutes)
- Added a second batman-adv interface for future link bonding

### Setup Workflow Change (Reboot Required)

After the initial setup wizard, the node performs address/DHCP reservation and **must reboot** to apply it. It should reboot automatically; if it does not, reboot it manually.

### Hardware Notes

- A BCF for the Alfa AHM26108D is included, but it is **not loaded by default** and must be applied manually for now.

---

## Addressing Changes

- Fresh flash default address: `10.41.254.1` (was `10.42.0.1` on older releases)
- If your end user device gets a `10.41.x.x` IP, it is on the mesh
- Mesh gateways will always be within `10.41.0.0/24`
- Safe static ranges: `10.41.253.0/24` and `10.41.254.0/24` (auto addressing will not use these ranges)

After the post-wizard reboot, you may need to release/renew DHCP (or reconnect) on your device to obtain a fresh lease.

---

## `openmanetd` (Beta)

`openmanetd` is a new binary that manages low level configurations on OpenMANET:

- Automatic static IP assignment and DHCP range reservation
- Automatic gateway selection (matches gateway announcements with batman-adv best gateway)
- Periodic node announcements for future features (GPS, signal quality, battery life, etc.)

---

## Example Image Names (1.6.0)

Always verify SHA256 checksums on the GitHub release page for the version you are installing. These examples reflect recent `1.6.0` assets:

| Filename | SHA256 |
|----------|--------|
| `openmanet-1.6.0-mm8108-ekh01-spi-squashfs-sysupgrade.img.gz` | `afa1b4a258326d4a01409e4ce99009257421bb6a31a336c071aa09d8dec4a88c` |
| `openmanet-1.6.0-rpi4-mm6108-sdio-squashfs-sysupgrade.img.gz` | `541ef7bea159bf30ae3c23c48280ce81d432269b180bb3738e742c48f31cb87d` |
| `openmanet-1.6.0-rpi4-mm6108-spi-squashfs-sysupgrade.img.gz` | `6c7087520e60b825b32d57dcedcf81d2b539c95f97fc99a9353d7c6ab566243c` |
| `openmanet-1.6.0-rpi3-mm6108-sdio-squashfs-sysupgrade.img.gz` | `f90f1b698a9466bb73f12e18522fbf04db134923761cfee6a273090abc52c5f1` |
| `openmanet-1.6.0-rpi3-mm6108-spi-squashfs-sysupgrade.img.gz` | `f82274fe7c6863901aa0a400af89ecb31acaed429094c3486b499f508b863b87` |
