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

## Networking Model
- The mesh exposes a flat `10.41.0.0/16` LAN to end users, even though BATMAN-V may send frames over multiple HaLow hops in the background.
- A single Mesh Gate now runs strictly in router mode, handing out `10.41.1.1` and NATing the MANET into whatever uplink you connect it to, your upstream network stays separate.
- Every mesh point keeps its own DHCP scope and unique 2.4/5 GHz SSID, so clients can join over Ethernet or Wi-Fi and still get a lease during disconnected operations.
- MDNS and Alfred advertise hostnames mesh-wide, so you can reach `hostname.local` from anywhere without touching DNS servers.

This design is deliberately opinionated to reduce the amount of networking knowledge you need to bring a cluster online. See the dedicated [Networking](./networking) page for the full breakdown.


## Advantages vs. the Seeed image
- Different BCF radio file increases TX power (≈21 dBm → **27 dBm**)  
- Newer build than the Seeed image 
- Includes **802.11s** and **batman-adv** support  

---

## 📡 Range Testing  
Want to see how OpenMANET performs in the field?  
Check out the dedicated **[Wi-Fi HaLow Range Testing](./range-testing.html)** page for detailed results, images, and notes from real-world testing.

---
