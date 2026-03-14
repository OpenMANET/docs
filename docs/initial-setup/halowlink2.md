---
layout: default
title: HaLow Link 2 Initial Setup
parent: Initial Setup
nav_order: 2
permalink: /initial-setup/halowlink2
description: How to flash OpenMANET onto the HaLow Link 2
---

# How to flash OpenMANET on HaLow Link 2

![HaLow Link 2](../pics/halowlink2-setup/product.jpg)

---

### 1. Connect to the HaLow Link 2 Wi-Fi network

The SSID and password are on the label on the side of the device.

![Step 1 screenshot](../pics/halowlink2-setup/step01.png)

### 2. Click on properties

![Step 2 screenshot](../pics/halowlink2-setup/step02.png)

### 3. Remember the gateway IP address

![Step 3 screenshot](../pics/halowlink2-setup/step03.png)

### 4. In your browser type in the gateway IP address. If you get this warning hit Advanced

![Step 4 screenshot](../pics/halowlink2-setup/step04.jpg)

### 5. Then click Continue

![Step 5 screenshot](../pics/halowlink2-setup/step05.jpg)

### 6. Enter the root password

The Device Password can be found on the label on the side of the HaLow Link 2.

![Step 6 screenshot](../pics/halowlink2-setup/step06.jpg)

### 7. Click Login

![Step 7 screenshot](../pics/halowlink2-setup/step07.png)

### 8. Click on Advanced Config

![Step 8 screenshot](../pics/halowlink2-setup/step08.jpg)

### 9. Click on System

![Step 9 screenshot](../pics/halowlink2-setup/step09.jpg)

### 10. Click on Backup/Flash Firmware

![Step 10 screenshot](../pics/halowlink2-setup/step10.png)

### 11. Click on Flash Image

![Step 11 screenshot](../pics/halowlink2-setup/step11.png)

### 12. Click Browse

![Step 12 screenshot](../pics/halowlink2-setup/step12.jpg)

### 13. Select the image you want to flash

The latest images can be found at [https://github.com/OpenMANET/firmware/releases](https://github.com/OpenMANET/firmware/releases). Look for the file with `halowlink2` in the filename.

![Step 13 screenshot](../pics/halowlink2-setup/step13.png)

### 14. Click Open

![Step 14 screenshot](../pics/halowlink2-setup/step14.png)

### 15. Click Upload

![Step 15 screenshot](../pics/halowlink2-setup/step15.jpg)

### 16. Wait for the image to be verified

![Step 16 screenshot](../pics/halowlink2-setup/step16.jpg)

### 17. Uncheck "Keep settings and retain config"

![Step 17 screenshot](../pics/halowlink2-setup/step17.png)

### 18. Scroll down and check "Force upgrade"

![Step 18 screenshot](../pics/halowlink2-setup/step18.png)

### 19. Click Continue

This will start the flash. Wait at least 10 minutes and keep the device plugged into power.

![Step 19 screenshot](../pics/halowlink2-setup/step19.png)

### 20. Connect to the openmanet Wi-Fi network

Once flashing is complete, a Wi-Fi network called **openmanet** will appear. The password is `openmanet`.

![Step 20 screenshot](../pics/halowlink2-setup/step20.png)

### 21. Access the OpenMANET web interface

A freshly flashed device can be found at `10.41.254.1`.

![Step 21 screenshot](../pics/halowlink2-setup/step21.png)

### 22. Click Login

There is no password set on a fresh flash, so just click Login.

![Step 22 screenshot](../pics/halowlink2-setup/step22.png)

### 23. Complete the setup wizard

Select your country and finish the device setup.

![Step 23 screenshot](../pics/halowlink2-setup/step23.jpg)
