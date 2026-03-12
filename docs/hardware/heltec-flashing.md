---
layout: default
title: Flashing the HT-HD01 V2
parent: Hardware
nav_order: 4
permalink: /hardware/heltec-flashing
description: Step-by-step guide to flash OpenMANET firmware onto the Heltec HT-HD01 V2.
---

# How to flash OpenMANET onto the HT-HD01 V2

This walkthrough is geared toward a stock flashed HT-HD01 V2.

![HT-HD01 V2](https://heltec.org/wp-content/uploads/2024/12/7-3-768x768.png)

---

### Download the firmware

Download the latest firmware from the [OpenMANET releases page](https://github.com/OpenMANET/firmware/releases).

Look for the file matching this pattern:

`openmanet-*-ramips-mt76x8-heltec_ht-hd01-v2-squashfs-sysupgrade.bin`

The key identifiers to look for are `mt76x8`, `heltec`, and `ht-hd01-v2` in the filename.

---

### 1. Connect to the HT-HD01 V2's Wi-Fi network

The default password to the network is `heltec.org`

![Step 1 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/4acd1601-2f7a-4909-beec-2ca7d09d4301/1e15eac6-a0ec-476f-9fc8-84f112a86b8c.png?crop=focalpoint&fit=crop&fp-x=0.7503&fp-y=0.7513&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 2. Once connected click on properties

![Step 2 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/9bcfdb4e-6ba2-4246-b4cf-1d085012e23b/402e7926-68b6-4e85-8456-a78fca5f2519.png?crop=focalpoint&fit=crop&fp-x=0.7503&fp-y=0.7513&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 3. You will need the default gateway value

![Step 3 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/c89042f2-2afd-4fff-ad41-1ccf40eed2fe/0801a3f3-929a-4ffc-b920-2f8b11794c10.png?crop=focalpoint&fit=crop&fp-x=0.5680&fp-y=0.6601&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8&mark-x=171&mark-y=333&m64=aHR0cHM6Ly9pbWFnZXMudGFuZ28udXMvc3RhdGljL2JsYW5rLnBuZz9tYXNrPWNvcm5lcnMmYm9yZGVyPTYlMkNGRjc0NDImdz0zMjkmaD0yOCZmaXQ9Y3JvcCZjb3JuZXItcmFkaXVzPTEw)

### 4. In your web browser go to the gateway IP address and login

The default password should be `heltec.org`

![Step 4 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/64804f41-1bdc-4362-a95c-e57fd189ee41/636f1c42-c51b-416f-85d9-d9f7224e13f6.jpg?q=100&crop=focalpoint&fit=crop&fp-x=0.2503&fp-y=0.3971&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 5. Click on the X in the top right

![Step 5 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/c85c6e1c-1c4f-4e23-97ce-c37aeb27f462/b855c63a-58cc-476b-9db6-f5db61c4dca4.jpg?q=100&crop=focalpoint&fit=crop&fp-x=0.5000&fp-y=0.5000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 6. Navigate to System > Reset / Flash Firmware

![Step 6 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/8e89fff3-a5af-4673-baf7-c7cd3581edca/b13ddd1b-ffcd-4a6a-a451-8dd3ceca0855.jpg?q=100&crop=focalpoint&fit=crop&fp-x=0.0937&fp-y=0.5173&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 7. Click Flash Image

![Step 7 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/7bca090f-3fcf-425d-add1-5df2c987a3eb/6bdc5df8-0561-4a9d-ad1c-9eacff911524.jpg?q=100&crop=focalpoint&fit=crop&fp-x=0.4136&fp-y=0.3658&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 8. Click Browse

![Step 8 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/0a909078-df56-46e0-b185-4b525deed117/f8f7aaca-2fc1-4a9f-aa20-9a5952771e29.jpg?q=100&crop=focalpoint&fit=crop&fp-x=0.4531&fp-y=0.2500&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 9. Select the bin file you want to flash

**IMPORTANT: You need to be flashing the correct bin for this board. Please double check.**

`openmanet-24.10-1.6.4-ramips-mt76x8-heltec_ht-hd01-v2-squashfs-sysupgrade.bin`

![Step 9 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/d45a2cea-2e8c-4f1b-81d1-8d493a61b777/fec02933-3c00-450f-8c6b-f55322de2702.png?crop=focalpoint&fit=crop&fp-x=0.4699&fp-y=0.5273&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8&mark-x=207&mark-y=148&m64=aHR0cHM6Ly9pbWFnZXMudGFuZ28udXMvc3RhdGljL2JsYW5rLnBuZz9tYXNrPWNvcm5lcnMmYm9yZGVyPTYlMkNGRjc0NDImdz00NTcmaD0zNiZmaXQ9Y3JvcCZjb3JuZXItcmFkaXVzPTEw)

### 10. Click Open once you have confirmed the correct bin has been selected

![Step 10 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/e904f7f1-f54c-439e-b46a-c624f4f186c3/ec5888ac-e79a-4235-ad3e-ec717081f2aa.png?crop=focalpoint&fit=crop&fp-x=0.5241&fp-y=0.5924&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 11. Click Upload

![Step 11 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/4aa5ed49-eaf1-4462-852b-068923079fad/010ee0cb-f382-4ea1-9f5f-c59f2c0d88c9.jpg?q=100&crop=focalpoint&fit=crop&fp-x=0.6027&fp-y=0.2368&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 12. Wait as the bin uploads

![Step 12 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/f2023023-6825-408d-b804-766cbbdc38b8/beb7e5ac-1da3-4976-8bd8-a9b0dec2ef82.jpg?q=100&crop=focalpoint&fit=crop&fp-x=0.5652&fp-y=0.1354&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 13. Uncheck "Keep settings"

![Step 13 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/eaec75b3-c951-4ec5-8121-3e412138714a/19b46532-cd16-4897-b518-63b099e4bd19.jpg?q=100&crop=focalpoint&fit=crop&fp-x=0.5058&fp-y=0.3020&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 14. Click Continue

![Step 14 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/6a892272-a041-4b01-8656-bbe016424e95/96cbb66b-6454-40a5-930a-4d6368c0ec1b.jpg?q=100&crop=focalpoint&fit=crop&fp-x=0.5175&fp-y=0.3515&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 15. Wait for the device to flash and reboot

You will need to wait about 10 minutes until the device has been flashed and booted up. You can tell once it's booted when a Wi-Fi network called **openmanet** appears.

![Step 15 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/86a33afc-a198-4255-a490-71c8e4560bec/8c49b8ef-8cfe-45c0-8bc0-6deb62dfa865.jpg?q=100&crop=focalpoint&fit=crop&fp-x=0.5562&fp-y=0.1652&fp-z=2.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 16. Connect to the openmanet Wi-Fi network

The password is `openmanet`

![Step 16 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/f30c02d7-4098-41fb-bb07-42bfc0762895/c35a95aa-2a75-47ea-b49f-3d86dea9c8d7.png?crop=focalpoint&fit=crop&fp-x=0.8333&fp-y=0.8107&fp-z=3.0000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 17. Log in to the OpenMANET web interface

![Step 17 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/860f21a3-dca1-408b-9ba5-aa730debe1ba/c8148f0f-c63f-4af4-a489-a243d609ba68.png?crop=focalpoint&fit=crop&fp-x=0.5000&fp-y=0.5000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)

### 18. Go through the wizard to configure your device

![Step 18 screenshot](https://images.tango.us/workflows/b86ae31f-8e7e-48f1-9f5b-885d3f25dda7/steps/604681a3-b4b7-45c6-92b6-60fe1adb06c7/70e02d6d-d0aa-4e74-94a0-b24fc6fa234b.png?crop=focalpoint&fit=crop&fp-x=0.5000&fp-y=0.5000&w=1200&border=2%2CF4F2F7&border-radius=8%2C8%2C8%2C8&border-radius-inner=8%2C8%2C8%2C8)
