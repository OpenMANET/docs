---
layout: default
title: Heltec
parent: Hardware
nav_order: 3
permalink: /hardware/heltec
description: Heltec HT-HD01 V2 hardware support, specifications, and configuration for OpenMANET.
---

# Heltec

OpenMANET supports Heltec Wi-Fi HaLow devices. These are compact, self-contained gateways with integrated HaLow radios.

---

## Supported Hardware

| Device | Status | Notes |
|--------|--------|-------|
| Heltec HT-HD01 V2 | ✅ Tested | MT7628AN + MM6108 (SDIO), onboard 2.4 GHz Wi-Fi |

---

## HT-HD01 V2

The HT-HD01 V2 is a compact Wi-Fi HaLow gateway based on the MediaTek MT7628AN SoC with a Morse Micro MM6108 HaLow radio connected via SDIO.

### Specifications

| Feature | Details |
|---------|---------|
| SoC | MediaTek MT7628AN (MIPS 24KEc, 580 MHz) |
| RAM | 128 MB DDR2 |
| Flash | 32 MB SPI NOR |
| HaLow Radio | Morse Micro MM6108 (SDIO interface) |
| HaLow Band | 915 MHz (US) |
| 2.4 GHz Wi-Fi | MT7628AN built-in (802.11b/g/n) |
| Ethernet | 1x 100 Mbps |
| Power | USB-C, 5V |
| Antenna | External SMA (sub-GHz), onboard 2.4 GHz |

---

### Build Instructions

To compile the OpenMANET firmware for the HT-HD01 V2:

```bash
./scripts/openmanet_setup.sh -i -b ht-hd01-v2
make download
make -j$(nproc)
```

The output image will be located at:

```
bin/targets/ramips/mt76x8/openmanet-*-ramips-mt76x8-heltec_ht-hd01-v2-squashfs-sysupgrade.bin
```

---

### Flashing

Flash via the OpenWrt web interface: navigate to **System > Backup / Flash Firmware**, upload the sysupgrade image, and uncheck "Keep settings" for a clean install.

---

### Radio Calibration (BCF Files)

The HT-HD01 V2 requires the board-specific calibration file `bcf_HD01_v2.bin` for the MM6108 HaLow radio. This BCF is included in the OpenMANET firmware image and is configured automatically.

---

### Default Configuration

| Setting | Value |
|---------|-------|
| HaLow Channel | 28 (916 MHz, 8 MHz BW) |
| 2.4 GHz SSID | openmanet |
| 2.4 GHz Password | openmanet |
| 2.4 GHz Encryption | WPA2-PSK |
| Mesh Routing | batman-adv (BATMAN_V) |

---

### Known Limitations

- Only the HT-HD01 V2 is supported. The HT-HD01 V1 is not supported.
- The 2.4 GHz radio operates in AP mode only.
- Ethernet port is 100 Mbps (no gigabit).

---
