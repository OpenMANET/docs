---
layout: default
title: ATAK Voice plugin interop
parent: Comms (Push-to-Talk)
nav_order: 3
permalink: /comms/atak-voice-plugin
description: Configure the ATAK Voice plugin (Vx) multicast channels to ride the same RTP/UDP flow as an OpenMANET comms talk group.
---

## Overview

The ATAK **Voice plugin** ("Vx", v1.7.8 and later) supports a **Multicast** channel type that sends voice as RTP over multicast UDP — the same transport family OpenMANET comms uses. With matching multicast address, port, and protocol selection, an Android TAK device can sit on the same talk group as a set of OpenMANET nodes without any gateway in between.

This page covers the **Multicast Only** path. The Voice plugin's Mumble path is a separate, server-mediated topology that has nothing to do with OpenMANET comms and is not documented here.

> **Codec parity is not verified in writing.** The Voice plugin user guide does not document its multicast codec, sample rate, frame size, or RTP payload type. OpenMANET comms uses Opus 48 kHz mono, 20 ms frames, RTP payload type 111. We have tested with the Vx plugin, and audio does work, but updates to the plugin may break compatability.

---

## 1. Prerequisites

- OpenMANET comms is already running on at least one mesh node — see [Setting up comms](setup.md). You need to know the **multicast address** and **port** of the talk group you want the ATAK device to join.
- ATAK is installed on an Android device with the Voice plugin v1.7.8 or later loaded.
- The Android device is joined to the OpenMANET mesh, either:
  - associated to a mesh point's 2.4/5 GHz AP (see [Networking](../networking.md#client-access-per-mesh-point)), or
  - attached via Ethernet to a mesh point.
- The Android device has an IP address inside `10.41.0.0/16`. If it doesn't, it isn't actually on the mesh — fix that first. See [Networking](../networking.md#flat-mesh-domain-for-every-use).

---

## 2. Match these values to your OpenMANET talk group

Before opening the plugin, write down the three values you'll be entering. They all come from the OpenMANET side:

| OpenMANET talk group field | Voice plugin field |
|---|---|
| Multicast address | Mission / Channel **Multicast IP** |
| Port | Mission / Channel **Port** |
| Transport (always RTP) | Mission / Channel **Default Protocol** → **RTP** |

| OpenMANET Talk Groups | RTP port |
|---|-------|
| 1 | 38801 |
| 2 | 38803 |
| 3 | 38805 |
| 4 | 38807 |
| 5 | 38809 |

The Voice plugin's protocol toggle has two positions, **UDP** and **RTP**. You **must** pick **RTP**. UDP mode sends raw audio with no RTP header and will not interoperate with OpenMANET's RTP receivers.

---

## 3. Create the mission

1. Open the Voice plugin and tap **+** in the mission list.
2. Enter a **Name** (e.g. `OpenMANET`).
3. Set **Type** to **Multicast**.
4. **IP Prefix** — the OpenMANET talk group's multicast address.
5. **Port** — the OpenMANET talk group's port.
6. **Default Protocol** — **RTP**.
7. Save. The new mission appears in the mission list.

---

## 4. Create a multicast channel inside the mission

1. Open the new mission and tap **+**.
2. **Channel Type** → **Multicast**.
3. **Channel Name** — anything meaningful to the operator.
4. **Multicast IP** and **Port** — the same values as the OpenMANET talk group.
5. **Protocol** — **RTP**.
6. Save.

The plugin auto-creates an **Engineering Channel** for every multicast mission. It is plugin-internal — it reaches all Voice-plugin users on the same mission but does not cross over to OpenMANET. Leave it alone or mute it on each device as you prefer.

---

## 5. Verify on the wire

Before doing a listening test, confirm packets are flowing in both directions. On any mesh node, with the talk group port (example: `5007`):

```bash
tcpdump -i br-ahwlan -nn -vv udp and portrange 38801-38809
```

- Press PTT on the ATAK device. You should see UDP packets with the **ATAK device's IP** as source, destined to the talk group's multicast address on port 5007.
- Press PTT on an OpenMANET node. You should see packets with the **node's IP** as source, on the same multicast address and port, plus RTCP Sender Reports on `port + 1` (5008) every 5 s.

To check the RTP payload type — the most likely place for a codec mismatch to bite — capture a few seconds with `-w` and open the file in Wireshark, or decode the RTP header manually with `tcpdump -X`. OpenMANET emits **RTP payload type 111**. If the Voice plugin emits a different PT, transport still works but the receiving side will likely fail to decode the audio.

---

## 6. Listening test

This is what actually proves interop:

- Transmit from an OpenMANET node and confirm the ATAK device plays the audio.
- Transmit from the ATAK device and confirm an OpenMANET node plays the audio.

If packets flow on the wire (step 5) but audio does not come through in one or both directions, the codec or RTP framing disagrees. Your two options at that point are:

- Confirm the Voice plugin's codec parameters (sample rate, channels, frame size, payload type) with its developer and reconcile the difference, or
- Bridge the two sides through a transcoding relay.

Do not assume audio interop until both directions pass this test.

---

## 7. Caveats

- **Use RTP, never UDP.** The Voice plugin's UDP mode sends raw audio with no RTP header. OpenMANET will not understand it.
- **Multicast TTL.** OpenMANET sets multicast TTL to 1. That's fine across a single BATMAN-V mesh but will not cross routed boundaries — the ATAK device must be on the same L2 broadcast domain (`br-ahwlan`) as a mesh point.
- **Half-duplex is one-sided.** OpenMANET enforces half-duplex within a talk group (see [How comms works](how-it-works.md#half-duplex)). The Voice plugin does not — multiple ATAK users can PTT at once, and those overlapping streams will collide on OpenMANET receivers. Apply normal voice discipline.
- **Engineering Channel does not bridge.** It only reaches Voice-plugin users on the same mission.

---

## See also

- [Setting up comms](setup.md) — bring comms up on the OpenMANET side first.
- [How comms works](how-it-works.md) — Opus pipeline, talk groups, half-duplex, jitter buffer.
- [Networking](../networking.md) — flat `10.41.0.0/16` mesh, BATMAN-V multicast, client access.
