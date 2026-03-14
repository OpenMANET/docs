---
layout: default
title: Gateworks Venice
parent: Initial Setup
nav_order: 3
permalink: /initial-setup/venice
description: Initial setup guide for Gateworks Venice devices on OpenMANET.
---

# Initial Setup — Gateworks Venice
**Please Note: Setup on Gateworks Venice cards is for advanced users.  We do not provide technical support if you brick your device, and do not take responsibility if you do not follow Gateworks recommendations.**

The image can be installed a few different ways.  If you already have OpenMANET on your Gateworks SBC, you can install it through the Web UI using the update firmware functionality.

### Initial Install on Gateworks SBC
If you have not already installed OpenMANET on a Gateworks SBC you will need a few things to easily install the image for the first time.

Gateworks does provide documentation for [Installing Firmware](https://trac.gateworks.com/wiki/venice/firmware).

After trying several different ways to install the firmware, here is the easiest method.
#### Requirements
- [Gateworks JTAG Programmer](https://trac.gateworks.com/wiki/venice/firmware) (Only needed if you did not buy a full development kit)
- [USB-C Flash Drive](https://www.amazon.com/dp/B09WB2NL8W)
- [USB-C Ethernet Adapter](https://www.amazon.com/dp/B082K62S48) (Only needed for GW7500)
- Software for a Serial interface

**IMPORTANT**
Follow all instructions from Gateworks to [prevent a group loop](https://trac.gateworks.com/wiki/gettingstarted#Power) with your JTAG device

Steps to get a serial console (Linux/MacOS)
1. Connect your JTAG device to a USB port
2. Use [Gateworks Instructions](https://trac.gateworks.com/wiki/jtag_instructions#SerialConsoleAccessonLinux) for Serial Console
3. Copy the OpenMANET squashfs firmware from your host computer to the USB-C Flash drive
4. Plug the USB-C flash drive into the Gateworks SBC
5. Once a `screen` session is setup, plug your USB JTAG device into the Gateworks SBC.
6. Power on the Gateworks SBC
7. You should get information in the `screen` session similar to this

```console
U-Boot 2024.10-00043-g977697bc9710 (May 27 2025 - 21:33:09 +0000)

CPU:   Freescale i.MX8MP[8] rev1.1 1600 MHz (running at 1200 MHz)
CPU:   Industrial temperature grade (-40C to 105C) at 20C
Reset cause: POR
Model: Gateworks Venice GW75xx-2x i.MX8MP Development Kit
DRAM:  4 GiB
Core:  257 devices, 30 uclasses, devicetree: separate
WDT:   Started watchdog@30280000 with servicing every 1000ms (60s timeout)
MMC:   FSL_SDHC: 1, FSL_SDHC: 2
Loading Environment from MMC... Reading from MMC(2)... OK
In:    serial@30890000
Out:   serial@30890000
Err:   serial@30890000
SEC0:  RNG instantiated
Net:   No ethernet found.
GSC     : boot watchdog disabled
Thermal protection:enabled at 96C
Hit any key to stop autoboot:  0
```

8. Hit any key to stop the boot process and drop into the uboot terminal.
9. You will enter the follow command into the uboot terminal which will begin copying the firmware from the USB flash drive onto the Gateworks SBC
```
usb start && load usb 0:1 $loadaddr <name of flash image on the USBC Flash Drive> && gzwrite mmc $dev $loadaddr $filesize
```
10. Unplug the SBC from power
11. Plug in a USBC Ethernet Adapter.  Connect the USBC ethernet adapter to your computer with an ethernet cable.
12. Power on the Gateworks SBC.  After the UBoot menu you will see the linux system starting up.
13. Open a browser to http://10.41.254.1 and complete the setup through the WebUI.
