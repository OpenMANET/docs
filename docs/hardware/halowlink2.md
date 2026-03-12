---
layout: default
title: HaLowLink2
parent: Hardware
nav_order: 2
permalink: /hardware/halowlink2
description: MorseMicro HaLow Link 2 hardware support, specifications, and configuration for OpenMANET.
---

# HaLow Link 2

The HaLow Link 2 is a Wi-Fi HaLow router, access point, and extender made by MorseMicro. It is based on the MediaTek MT7621 SoC with a Morse Micro MM8108 HaLow radio connected via SDIO.

---

## Supported Hardware

| Device | Status | Notes |
|--------|--------|-------|
| MorseMicro HaLow Link 2 | ✅ Tested | MT7621 + MM8108 (SDIO), 2.4 GHz Wi-Fi (MT7603) |

---

## Specifications

| Feature | Details |
|---------|---------|
| SoC | MediaTek MT7621 (MIPS 1004Kc, dual-core, 880 MHz) |
| RAM | 256 MB DDR3 |
| Flash | 32 MB SPI NOR |
| HaLow Radio | Morse Micro MM8108 (SDIO interface) |
| HaLow Band | 915 MHz (US) |
| 2.4 GHz Wi-Fi | MediaTek MT7603 (802.11b/g/n, PCIe) |
| Ethernet | 1x WAN + 1x LAN (Gigabit) |
| Power | USB-C, 5V |
| LEDs | 3x RGB (Wi-Fi, Status, HaLow) |
| Antenna | External SMA (sub-GHz), onboard 2.4 GHz |

---

### Radio Calibration (BCF Files)

The HaLow Link 2 uses the board-specific calibration file `bcf_mm_hl2_ext.bin` for the MM8108 HaLow radio. This BCF is included in the OpenMANET firmware image and is configured automatically.

---

### Default Configuration

| Setting | Value |
|---------|-------|
| HaLow Channel | 28 (916 MHz, 8 MHz BW) |
| 2.4 GHz SSID | openmanet |
| 2.4 GHz Password | openmanet |
| Mesh Routing | batman-adv (BATMAN_V) |

---
