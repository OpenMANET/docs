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
| Gateworks GW16167 | M.2 E-Key (USB) | MM8108 | Up to +26dBm Transmit Power |
| Gateworks GW16170 | M.2 E-Key (USB) | MM8108-M20 | Up to +28.5dBm Transmit Power |

### Interface Types at a Glance

| Interface | Throughput | Onboard Wi‑Fi | Typical Boards |
|-----------|------------|----------------|----------------|
| SPI | Lower | ✅ Available (AP mode) | Seeed WM1302 HAT |
| SDIO | Higher | ❌ Conflicts with HaLow bus | Silex, Alfa |
| USB | Higher | N/A | Gateworks GW16167 & GW16170 |

## Supported Cameras

Raspberry Pi firmware images support CSI cameras through libcamera and MediaMTX. The tested module is the [Arducam 8 MP IMX219 Camera Module V2-compatible camera](https://www.amazon.com/dp/B083BHJZ16). Drivers are also included for Raspberry Pi Camera Modules 1, 2, and 3 (OV5647, IMX219, and IMX708), plus the IMX477-based Raspberry Pi High Quality Camera.

Camera services start only when `cam -l` detects a compatible sensor. See [Cameras](./hardware/cameras) for compatibility, the RTSP address, service behavior, and ATAK Sensor CoT integration.

---

## Optional Accessories

| Item | Notes |
|------|-------|
| [WaveShare UPS HAT D (21700)](https://www.waveshare.com/ups-hat-d.htm) | Battery-backed power for field use |
| [Panda PAU06 USB Wi-Fi Adapter](https://www.amazon.com/dp/B00762YNMG) | Secondary Wi‑Fi interface (drivers included) |
| [USB GPS Receiver (u-blox)](https://www.amazon.com/dp/B01MTU9KTF) | Required for GNSS/range-testing features |
| [Arducam 8 MP IMX219 Camera](https://www.amazon.com/dp/B083BHJZ16) | Tested CSI camera; provides RTSP video and ATAK Sensor CoT on Raspberry Pi nodes |
| [21700 Rechargeable Batteries](https://www.amazon.com/dp/B0D3GX96H6) | For use with WaveShare UPS HAT D |

---

## Sub-pages

| Page | Description |
|------|-------------|
| [Raspberry Pi Variants](./hardware/raspberry-pi) | Detailed parts list, SDIO/SPI reference, CM4 carrier boards, and M.2 Wi‑Fi cards |
| [Cameras](./hardware/cameras) | Supported CSI cameras, MediaMTX RTSP streaming, and ATAK Sensor CoT integration |
| [HaLowLink2](./hardware/halowlink2) | HaLowLink2-specific hardware notes |
| [Heltec](./hardware/heltec) | Heltec hardware support |
| [Gateworks Venice](./hardware/venic) | Gateworks Venice hardware support |
