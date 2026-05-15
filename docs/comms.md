---
layout: default
title: Comms (Push-to-Talk)
nav_order: 11
has_children: true
permalink: /comms
description: Multicast push-to-talk voice comms for OpenMANET nodes — architecture overview and per-device setup.
---

## Overview

Comms is the OpenMANET push-to-talk (PTT) voice subsystem. It runs as part of `openmanetd` and carries live audio between nodes over the mesh using Opus-encoded RTP on multicast UDP. A node can run one or more parallel **talk groups** (each on its own multicast address/port), enforce half-duplex within each group, and source PTT events from any of four selectable control sources. A browser-only **web mode** is also available for nodes that have no sound card at all.

---

## At a glance

| Parameter | Value |
|---|---|
| Codec | Opus, 48 kHz, mono, VoIP profile |
| Bitrate | 32 kbps |
| Frame size | 20 ms (960 samples) |
| Encoder complexity | 5 (default; configurable 1–10) |
| In-band FEC | Enabled, adaptive (`packetLossPerc` floor = 20, ramps to 30 / 40 under loss) |
| DTX | Disabled |
| Transport | RTP over multicast UDP |
| RTP payload type | 111 (dynamic, Opus) |
| RTP MTU | 1400 bytes |
| Multicast TTL | 32 (sized for multi-hop batman-adv + VXLAN + Tailscale paths) |
| RTCP Sender Report interval | 5 s (outbound only; inbound RTCP not processed) |
| Default playback / capture latency hint | 60 ms each |

---

## Control sources

Comms supports four PTT control sources, selected by the `comms.controlSource` config key:

- **`openvlm`** *(default)* — OpenVLM USB HID dongle. GPIO3 on the dongle is the PTT button: press → `PTTDown`, release → `PTTUp` (hold-to-talk). VOL+ / VOL− buttons on the dongle also drive the system ALSA mixer via a built-in `alsa.Controller` aux event handler.
- **`web`** — Browser-driven PTT. The browser owns audio I/O and the malgo audio backend is bypassed entirely, so this mode works on nodes with no sound card.

---

## In this section

- [How comms works](comms/how-it-works.md) — operator-level walkthrough of the audio pipeline, talk groups, half-duplex, and the adaptive FEC loop.
- [Setting up comms](comms/setup.md) — enable comms on a node, pick a control source, wire the hardware, and verify it's working.

---

## See also

- [OpenMANETd Daemon](openmanetd.md) — the daemon that hosts the comms subsystem.
- [Networking](networking.md) — mesh interface and multicast routing context.
