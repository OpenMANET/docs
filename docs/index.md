---
layout: home
title: OpenMANET
nav_order: 1
permalink: /
description: Raspberry Pi–based MANET radio built on Wi-Fi HaLow (802.11ah) with 802.11s + batman-adv, GPS-driven range testing, and PTT support.
---

# OpenMANET Project

**OpenMANET** is a Raspberry Pi–based MANET (Mobile Ad-Hoc Network) radio built on **Wi-Fi HaLow (802.11ah)**.  
It’s designed around Raspberry Pi HATs and currently built specifically for the Seeed HaLow board, with plans to add support for other devices later.

---

## Description
This project aims to provide a flexible HaLow mesh radio using Raspberry Pi hardware and HaLow HATs. A number of optional components are supported; current testing includes the WaveShare 1850 UPS for power and the Panda Wireless PAU06 USB Wi-Fi adapter (additional drivers are included but not yet fully tested).

> Note: On most Raspberry Pi models, the onboard Wi-Fi shares the SDIO address with HaLow modules and cannot be used for client networking in this setup. Use a USB Wi-Fi adapter or the Ethernet port on the Raspberry Pi 4 to bridge connectivity.

---

## Roadmap
- Enclosure design  
- Investigate USB OTG/Ethernet Gadget mode to allow EUDs to connect without USB-to-Ethernet adapters  
- Test **batman-adv** mesh networking

---

## In Progress
- GPS-based range-testing script using GPSD, logging GPS location, RSSI, and SNR for analysis  
- PTT (Push-to-Talk) application so the radio is functional without an EUD  
- Support for Seeed HaLow HATs now; other boards will be added later  
- Raspberry Pi 3B+ and 2W support  
- Testing the GPS module included on the Seeed 40-pin board

---

## Advantages vs. the Seeed image
- Custom BCF configuration increases TX power (≈21 dBm → **27 dBm**)  
- Newer build than the Seeed image  
- Includes **802.11s** and **batman-adv** support  

---

## 📡 Range Testing  
Want to see how OpenMANET performs in the field?  
Check out the dedicated **[Wi-Fi HaLow Range Testing](./range-testing.html)** page for detailed results, images, and notes from real-world testing.

---
