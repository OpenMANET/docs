---
layout: default
title: Cameras
parent: Hardware
nav_order: 5
permalink: /hardware/cameras
description: Raspberry Pi camera support, MediaMTX RTSP streaming, and ATAK Sensor CoT integration.
---

# Cameras

OpenMANET supports Raspberry Pi CSI cameras on its Raspberry Pi 4/CM4, Raspberry Pi 3B, and Raspberry Pi Zero 2 W images. When a compatible camera is connected, the node automatically provides an RTSP stream through MediaMTX. If the node also has a valid GPS fix, `openmanetd` advertises the camera to ATAK with Cursor-on-Target (CoT) messages.

## Supported Cameras

The following sensor drivers are included in the Raspberry Pi firmware images:

| Camera | Sensor | Support status |
|--------|--------|----------------|
| [Arducam Camera Module V2-compatible 8 MP camera](https://www.amazon.com/dp/B083BHJZ16) | Sony IMX219 | ✅ Tested with OpenMANET |
| Raspberry Pi Camera Module 1 / NoIR | OmniVision OV5647 | Driver included |
| Raspberry Pi Camera Module 2 / NoIR | Sony IMX219 | Driver included |
| Raspberry Pi Camera Module 3, including Wide and NoIR variants | Sony IMX708 | Driver and autofocus support included |
| Raspberry Pi High Quality Camera | Sony IMX477 | Driver included |

Raspberry Pi identifies Camera Modules 1, 2, and 3 by the OV5647, IMX219, and IMX708 sensors respectively. See the [official Raspberry Pi camera documentation](https://www.raspberrypi.com/documentation/accessories/camera.html) for the module variants and specifications.

Third-party CSI modules based on one of the supported sensors may also work if they are compatible with the Raspberry Pi libcamera stack. The automatic OpenMANET camera workflow currently targets CSI/libcamera cameras. Although the firmware includes a generic USB UVC driver, USB cameras are not automatically configured or advertised through MediaMTX, ONVIF, or ATAK.

> Connect or disconnect a CSI camera only while the Raspberry Pi is powered off. Attach the camera before boot so the firmware can detect it.

## Automatic Startup

The `rpi-camera-services` startup service checks for a libcamera-compatible sensor by running `cam -l`.

- If no camera is detected, MediaMTX and `camera-onvif-server` remain stopped. This avoids running camera services on nodes that do not have a camera.
- If a camera is detected, MediaMTX starts automatically.
- `camera-onvif-server` starts after the configured `ahwlan` interface is up and has an IPv4 address.

Confirm that libcamera detects the sensor:

```sh
cam -l
```

A working camera appears under `Available cameras`, for example:

```text
Available cameras:
1: 'imx219' (/base/soc/i2c0mux/i2c@1/imx219@10)
```

After attaching a camera and rebooting, you can resynchronize the services manually if needed:

```sh
/etc/init.d/rpi-camera-services restart
```

## RTSP Stream

MediaMTX listens for RTSP clients on port `554`. OpenMANET uses the IPv4 address assigned to the `ahwlan` network, normally the `br-ahwlan` bridge, and publishes the camera at this address:

```text
rtsp://<node-ip>:554/rpicamera
```

For example, a node whose bridge address is `10.41.16.254` provides:

```text
rtsp://10.41.16.254:554/rpicamera
```

You can open this URL from an EUD on the OpenMANET network with an RTSP client such as VLC. Check the address and configured RTSP port on the node with:

```sh
ip -4 addr show br-ahwlan
uci -q get mediamtx.@mediamtx[0].rtsp_address
```

The default MediaMTX setting is `:554`, which listens on port 554 and allows the stream to be reached through the node's OpenMANET address.

## ATAK Sensor CoT

Camera advertisement in ATAK requires all of the following:

- A camera detected by `cam -l`
- GNSS enabled in `openmanetd`
- CoT output enabled with `gnss.sendAsExternalGNSSSource.sendAsCoT: true`
- A valid 2D or 3D GPS fix (`mode` 2 or 3)
- An IPv4 address on `br-ahwlan`

When these conditions are met, `openmanetd` sends a camera video event and a linked Sensor CoT event to the ATAK Situational Awareness multicast group at `239.2.3.1:6969`. Updates are rate-limited to once every 30 seconds. The sensor appears in ATAK at the node's GPS position, and its video entry points to the node's `/rpicamera` RTSP stream.

If the camera works over RTSP but GPS reports `mode: 0` or `mode: 1`, the camera CoT is not sent because there is no valid position for the sensor marker.

Verify GNSS and outgoing CoT traffic with:

```sh
gpspipe -w -n 10 2>/dev/null | grep '"class":"TPV"'
tcpdump -ni br-ahwlan 'udp dst 239.2.3.1 and port 6969'
```

ATAK must be connected to the OpenMANET network and listening for SA multicast traffic on `239.2.3.1:6969`.

## Troubleshooting

### `cam -l` shows no available cameras

- Power down the Raspberry Pi and reseat the CSI ribbon cable.
- Confirm that the cable orientation is correct at both connectors.
- Confirm that the camera uses one of the supported sensors listed above.
- Reboot after connecting the camera.
- Check camera-related kernel messages with `dmesg | grep -iE 'camera|imx|ov5647|unicam'`.

### Camera is detected but RTSP is unavailable

```sh
ip -4 addr show br-ahwlan
/etc/init.d/rpi-camera-services restart
logread | grep -E 'rpi-camera-services|mediamtx|camera-onvif-server'
```

MediaMTX will not remain running without a detected camera, and the ONVIF service waits for the `ahwlan` interface to have an IPv4 address.

### RTSP works but no camera appears in ATAK

- Confirm the GPS TPV report has `"mode":2` or `"mode":3` and includes latitude and longitude.
- Confirm CoT output is enabled in `/etc/openmanetd/config.yml`.
- Wait at least 30 seconds for the next multicast update.
- Use `tcpdump` as shown above to confirm packets leave `br-ahwlan`.
- Confirm ATAK is receiving multicast traffic from the OpenMANET network.
