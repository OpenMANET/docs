---
layout: default
title: Gateworks Venice
parent: Hardware
nav_order: 4
permalink: /hardware/venice
description: Gateworks Venice hardware support for OpenMANET.
---

# Gateworks Venice
OpenMANET is able to run on Gateworks Venice based SBCs.  The offer many different models, and while OpenMANET should run on all of them, it has only been tested on a few models.

## Supported Hardware

### SBC

| Device      | Status       | Notes |
|-------------|--------------|----------------------------------------------------------------------------------|
| Venice 7100 | ✅ Supported | 1 ethernet and 1 mPCIE full height (for HaLow)                                   |
| Venice 7200 | ✅ Tested    | Dual mPCIE Slots for HaLow and 802.11ax with 2 ethernet.                         |
| Venice 7500 | ✅ Tested    | 1 full size mPCIE (For 802.11ax Wifi) 1 half-size mPCIE (For HaLow), no ethernet |

### HaLow

| Device | Interface | MM Chipset | Notes |
|--------|-----------|------------|-------|
| Gateworks GW16167 | M.2 E-Key (USB) | 8108 | Only tested HaLow card for Gateworks |

### Wi-Fi Cards that support 802.11s

These are the only 2 WiFi cards that have been tested with Gateworks Venice, and are the best option to "do everything".

| Chipset     | Interface  | 802.11s | Notes |
|-------------|------------|---------|----------------------------|
| MT7915DAN   | mPCIE      | yes     | Dual Band AX; Power consumption maximum is 9W, average is 4 – 8W. |
| MT7916AN    | mPCIE      | yes     | Dual Band AX; Power consumption maximum is 9W, average is 4 – 8W. |

