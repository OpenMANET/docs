---
layout: default
title: Hardware
nav_order: 4
has_children: true
permalink: /hardware
description: Overview of all supported hardware for OpenMANET nodes.
---

# Hardware

OpenMANET runs on Raspberry Pi–based devices paired with Wi‑Fi HaLow (802.11ah) boards from Morse Micro. The tables below provide a quick reference to every supported platform. For detailed setup, parts lists, and build-specific guidance, see the sub-pages.

---

## Supported Single-Board Computers

| Device | Status | Notes |
|--------|--------|-------|
| Raspberry Pi 4 | ✅ Tested | Onboard Wi‑Fi works in AP mode on SPI-based builds |
| Raspberry Pi CM4 | ✅ Tested | Recommended for advanced/multi-interface builds; CM4 carrier boards add M.2, dual Ethernet, etc. |
| Raspberry Pi 3B | ✅ Supported | Requires selecting the correct image for your HaLow interface |
| Raspberry Pi Zero 2 W | ✅ Supported | Uses `rpi3` firmware images; suitable for lightweight/portable nodes |
| HaLowLink2 | ✅ Supported | Integrated HaLow device — see [HaLowLink2](./hardware/halowlink2) |
| Heltec | ✅ Supported  | See [Heltec](./hardware/heltec) for current status |
| Gateworks Venice | ✅ Supported | See [Gateworks Venice](./hardware/venice) |

---

## Supported HaLow Modules

| Device | Interface | MM Chipset | Notes |
|--------|-----------|------------|-------|
| Seeed WM1302 + Wio-WM6108 | SPI | MM6108 | Common "Seeed board" setup; works on all supported Pi variants |
| Silex SX-SDMAH | SDIO | MM6108 | |
| Alfa AHPI6108E | SDIO | MM6108 | |
| Gateworks GW16167 | M.2 E-Key (USB) | MM8108 | |

### Interface Types at a Glance

| Interface | Throughput | Onboard Wi‑Fi | Typical Boards |
|-----------|------------|----------------|----------------|
| SPI | Lower | ✅ Available (AP mode) | Seeed WM1302 HAT |
| SDIO | Higher | ❌ Conflicts with HaLow bus | Silex, Alfa |
| USB | Higher | N/A | Gateworks GW16167 |

---

## Optional Accessories

| Item | Notes |
|------|-------|
| [WaveShare UPS HAT D (21700)](https://www.waveshare.com/ups-hat-d.htm) | Battery-backed power for field use |
| [Panda PAU06 USB Wi-Fi Adapter](https://www.amazon.com/dp/B00762YNMG) | Secondary Wi‑Fi interface (drivers included) |
| [USB GPS Receiver (u-blox)](https://www.amazon.com/dp/B01MTU9KTF) | Required for GNSS/range-testing features |
| [21700 Rechargeable Batteries](https://www.amazon.com/dp/B0D3GX96H6) | For use with WaveShare UPS HAT D |

---

## Sub-pages

| Page | Description |
|------|-------------|
| [Raspberry Pi Variants](./hardware/raspberry-pi) | Detailed parts list, SDIO/SPI reference, CM4 carrier boards, and M.2 Wi‑Fi cards |
| [HaLowLink2](./hardware/halowlink2) | HaLowLink2-specific hardware notes |
| [Heltec](./hardware/heltec) | Heltec hardware support |
| [Gateworks Venice](./hardware/venic) | Gateworks Venice hardware support |
