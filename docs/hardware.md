---
layout: default
title: Hardware
nav_order: 4
permalink: /hardware
description: Covers what devices are recommended for OpenMANET
---

# Hardware

This page lists the recommended parts and supported boards for building an OpenMANET node. The project is designed for Raspberry Pi–based devices running OpenWrt, using Wi‑Fi HaLow boards from Morse Micro (MM6108/MM8108).

---

## Supported Hardware (Firmware-Dependent)

### SBC

| Device | Status | Notes |
|--------|--------|-------|
| Raspberry Pi 4 / CM4 | ✅ Tested | Onboard Wi‑Fi works in AP mode on SPI-based builds |
| Raspberry Pi 3B | ✅ Supported | Requires selecting the correct image for your HaLow interface |
| Raspberry Pi 2W | ✅ Supported | Requires selecting the correct image for your HaLow interface |

### HaLow

| Device | Interface | MM Chipset | Notes |
|--------|-----------|------------|-------|
| Seeed WM1302 + Wio-WM6108 | SPI | 6108 | Common “Seeed board” setup |
| Silex SX-SDMAH | SDIO | 6108 | |
| Alfa AHPI6108E | SDIO | 6108 | |

---

## Recommended Parts List

| Item | Optional |
|------|----------|
| [Wio WM6180 Wi-Fi HaLow mini PCIe Module](https://www.seeedstudio.com/Wio-WM6180-Wi-Fi-HaLow-mini-PCIe-Module-p-6394.html) | No |
| [WM1302 Pi Hat](https://www.seeedstudio.com/WM1302-Pi-Hat-p-4897.html) | No |
| [External Antenna 868/915 MHz 2 dBi SMA Foldable](https://www.seeedstudio.com/External-Antenna-868-915MHZ-2dBi-SMA-L195mm-Foldable-p-5863.html) | No |
| [UF.L to SMA-K 1.13 mm Cable (120 mm)](https://www.seeedstudio.com/UF-L-SMA-K-1-13-120mm-p-5046.html) | No |
| Raspberry Pi (Pi 4 / CM4 / Pi 3B / Pi 2W) | No |
| [21700 Rechargeable Batteries](https://www.amazon.com/dp/B0D3GX96H6?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_4) | Yes |
| [WaveShare UPS B (18650 version)](https://www.amazon.com/gp/product/B0D39VDMDP/ref=ox_sc_saved_title_1?smid=A3B0XDFTVR980O&psc=1) | Yes |
| [Panda PAU06 USB Wi-Fi Adapter](https://www.amazon.com/dp/B00762YNMG?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_1) | Yes |
| [USB GPS Receiver (u-blox based)](https://www.amazon.com/dp/B01MTU9KTF?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_1) | Yes |

---

## Board Interface Types: SDIO vs SPI

HaLow modules connect to the Raspberry Pi through different interfaces depending on the board design:

| Interface | Description | Supported on |
|------------|-------------|--------------|
| SDIO | High-speed 4-bit data bus. Offers better throughput and lower latency. | Image-dependent (common on Pi 4 / Pi 3B / CM4) |
| SPI | Serial Peripheral Interface used by some HaLow HATs (for example Seeed boards). Easier to wire but typically slower than SDIO. | Image-dependent (Pi 4 / CM4 / Pi 3B / Pi 2W supported on current firmware) |

Notes:  
- Select firmware downloads carefully: the board type, Morse Micro chipset (MM6108 vs MM8108), and interface (SPI vs SDIO) are part of the firmware filename.
- On SDIO-based HaLow builds, onboard Wi‑Fi usually cannot be used due to SDIO bus conflicts.
- On SPI-based HaLow builds, onboard Wi‑Fi can be used for client access (AP mode).

---

## Radio Calibration (BCF Files)

Some HaLow radios use board configuration files (BCF) to set calibration and regulatory parameters.

- A BCF for the Alfa AHM26108D is included, but it is **not loaded by default** and must be applied manually for now.

---

## Optional / Advanced Parts

### Compute Module 4 (CM4) Builds

Most Raspberry Pi Compute Module 4 (CM4) carrier boards work with the OpenMANET image.  
A good option is the [WaveShare CM4 Dual ETH WiFi6 Base](https://www.waveshare.com/cm4-dual-eth-wifi6-base.htm), which includes:

- Two Ethernet ports for bridging or mesh uplink  
- An M.2 slot for a standard Wi-Fi card (AX200 or AX210)  
- Full GPIO header and USB ports for power and debug

CM4 boards are ideal for advanced builds, providing better expandability and efficiency for multi-interface mesh nodes.

<img src="pics/waveshare-cm4-wave/cm4.jpeg" alt="WaveShare CM4 build overview" width="360" />

<img src="pics/waveshare-cm4-wave/cm4_inside.jpeg" alt="WaveShare CM4 internals" width="360" />

#### Other tested CM4 Carrier Boards

**WaveShare CM4-IO-Base-X**
Version A and B have been tested and work as expected
[CM4-IO-BASE-A](https://www.waveshare.com/product/raspberry-pi/boards-kits/compute-module-4-4s-cat/cm4-io-base-a.htm)
[CM4-IO-BASE-B](https://www.waveshare.com/product/raspberry-pi/boards-kits/compute-module-4-4s-cat/cm4-io-base-b.htm)

Notes:
- A M.2 **M Key** slot for communcation cards
- Full GPIO Header
- Same form factor as a Pi4

**MCUZone CM4_WiFi6**
This is a slightly larger carrier board than the WaveShare boards.

Can be found on [AliExpress](https://www.aliexpress.us/item/3256803637327862.html)

Notes:
- A M.2 **A Key** slot for communications cards.  This is limited to the 2230 form factor.
- Full GPIO Header
- Better for height constrained use cases, but a larger length and width form factor.

---

### M.2 Wi-Fi Cards for CM4 Boards

| Module | Band Support | Current Use |
|---------|--------------|-------------|
| [Intel AX200](https://www.waveshare.com/Wireless-AX200.htm) | 2.4 / 5 GHz Wi-Fi 6 | Works as an access point |
| [Intel AX210](https://www.waveshare.com/Wireless-AX210.htm) | 2.4 / 5 / 6 GHz Wi-Fi 6E | Works as an access point |

These cards currently operate as normal Wi-Fi access points.  

### M.2 Wi-Fi Cards that support 802.11s

| Chipset     | Interface  | 802.11s | Notes |
|-------------|------------|-----|----------------------------|
| Intel AX2XX | M.2 AE Key | no  | Can only operate in AP mode|
| QCNA765     | M.2 E Key  | no  | |
| WCN6856     | M.2 E Key  | no  |  |
| QCA6174     | M.2 E Key  | yes | You can only have one wifi network defined when using 802.11s |
| MT7921      | M.2 E Key  | no  | |
| MT7915DAN   | M.2 BM Key | yes | Dual Band AX; 802.11s mesh works, but stability may vary by kernel/driver |
| MT7916AED   | M.2 AE Key | yes | Dual Band AX; 802.11s mesh works, but stability may vary by kernel/driver |


Work is underway to support bonding of HaLow (915 MHz) and 2.4 GHz links together using BATMAN-V for multi-band uplinks.

---

## Development Notes and Future Plans

- Separate firmware builds for SDIO and SPI boards simplify setup.  
- CM4 carrier boards are increasingly recommended for advanced configurations.  
- Future releases will expand multi-gateway mesh support and improve multicast reliability.  

---
