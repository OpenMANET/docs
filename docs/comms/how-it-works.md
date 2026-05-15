---
layout: default
title: How comms works
parent: Comms (Push-to-Talk)
nav_order: 1
permalink: /comms/how-it-works
description: How the OpenMANET push-to-talk comms pipeline moves audio between nodes — talk groups, multicast RTP, Opus, half-duplex, and FEC adaptation.
---

## Pipeline at a glance

Comms runs the same audio pipeline symmetrically on every node. When you press PTT on one node, the local transmit side wakes up; every other node in the same talk group sits on the receive side:

```
                     ┌────────── Transmit side ──────────┐
 Mic ─► malgo capture ─► Opus encode ─► RTP packetize ─► multicast UDP
                                                           │
                                                           ▼
                                                       mesh subnet
                                                           │
                                                           ▼
 Speaker ◄─ malgo playback ◄─ Opus decode ◄─ jitter buffer ◄─ RTP parse
                     └────────── Receive side  ──────────┘
```

- Capture runs on the audio thread every 20 ms and hands frames to a dedicated encode goroutine (so Opus encoding and UDP sending never stall the audio thread).
- The capture device is **opened once at startup and stays open for the life of the comms run**. Per-PTT, an atomic TX gate (`SetTxEnabled`) decides whether captured frames reach the encoder; the device itself never opens or closes mid-session. This eliminates the first-cycle stall USB audio class devices used to suffer when the device was opened on every PTT press.
- On the receive side, each multicast port has its own UDP listener, RTP jitter buffer, and playback stream running independently of the others.
- The whole pipeline is codec-agnostic internally; today it's Opus, but only `buildCodec` knows that.

---

## Talk groups

Each entry in the global **multicast talk groups** configuration is one talk group. For each entry the comms subsystem opens:

- a UDP sender dialled out through the configured mesh interface, with multicast TTL set to 32 — generous enough to survive multi-hop batman-adv + VXLAN + Tailscale paths where each bridge decrements TTL,
- a separate RTCP sender on `port + 1` (5-second Sender Reports only; inbound RTCP is not processed because a multicast PTT topology has no single feedback path),
- a UDP receiver bound with `SO_REUSEPORT` and joined to the multicast group,
- its own **RTP session** (one local SSRC per node), **jitter buffer**, and **playback stream**.

Each talk group has independent **Send** and **Receive** flags that can be flipped at runtime:

- `Send = true` — the mic feeds this group when you PTT.
- `Receive = true` — audio arriving on this group is decoded and played on this node.

A single node can monitor several groups while transmitting on only one. On first startup only the **first** talk group is send/receive-enabled; the others are dormant until you turn them on at runtime (see [Setting up comms](setup.md#runtime-talk-group-toggles)).

---

## Half-duplex

Within each talk group, comms enforces half-duplex via a per-port **half-duplex gate**. Every time a packet arrives from a remote talker on a send-enabled port, that port's gate is stamped and a `RemoteRxActive` cache is primed. A PTT attempt that hits the cache is rejected with a "channel busy" log — you cannot transmit on top of somebody else.

A background 100 ms decay loop clears the cache once every send-enabled port's gate has gone quiet, so the next PTT press after the remote talker finishes succeeds immediately.

---

## Jitter buffer & packet loss

Every receive-capable talk group has its own jitter buffer:

- **Prebuffer**: 5 packets (~100 ms) before playout begins, so a brief head-of-stream reorder does not cause a start-of-call glitch.
- **Ring depth**: 24 slots. Packets arriving when the ring is full are counted as overflows and dropped.
- **Packet Loss Concealment**: short gaps are bridged by Opus PLC. Sustained loss (more than ~200 ms of consecutive missing frames) falls through to clean silence rather than an endless hiss.
- **New talker handling**: when the RTP SSRC changes, the buffer is reset cleanly and an `RTP SSRC changed; jitter buffer reset` log is emitted. This is the expected signal that a new talker has started on the group.

A 2-second idle reset also catches edge cases where a sender restarts without rotating its SSRC.

---

## Adaptive FEC

The Opus encoder's `packetLossPerc` knob controls how much in-band FEC redundancy it adds to every encoded frame. Comms runs a small control loop called **FECAdapter** that ticks every 2 seconds, reads each receive-capable port's gap-run histogram from the jitter buffer, and moves the encoder's FEC level through three discrete steps (`20` → `30` → `40`).

The configured `comms.packetLossPerc` value (default 30, clamped to `[10, 40]`) is the **floor** — the adapter only ramps up above it, never below.

State machine specifics:

- **Loss estimate**: per-tick raw loss ratio, smoothed by an EWMA with α = 0.2. At 2-second ticks the 63 % response time is roughly 10 seconds — long enough to ride out one noisy window, short enough to catch sustained loss.
- **Upgrade thresholds**: 20 → 30 at loss EWMA ≥ 0.08, 30 → 40 at ≥ 0.20. Each transition needs **2 consecutive ticks** above threshold (~4 s).
- **Downgrade thresholds**: 40 → 30 at loss EWMA < 0.10, 30 → 20 at < 0.03. Each transition needs **15 consecutive ticks** below (~30 s) — much longer than the upgrade dwell so the adapter does not flap on a brief recovery window.
- **Idle stall reset**: after 30 ticks (~60 s) with zero pushed packets and zero gap activity the EWMA is reset to zero, so a stale loss estimate from the previous PTT does not bleed into the next one.

The design assumes link symmetry — every node reads its own RX loss and applies the result to its own TX encoder. On omnidirectional mesh links that holds because link quality is effectively reciprocal.

**Users don't tune this** — it's automatic. Just know that it's running, and that brief RX loss on one node quietly raises the FEC overhead on its next transmission.

---

## Audio latency layers

Audio stutter can come from four different sources. Each is absorbed by a different buffer, and each has a different tuning knob:

| Class of stutter | Heard by | Absorbed by | Config knob |
|---|---|---|---|
| Network arrival jitter (reorder, late packets) | Local listener | Jitter prebuffer | — (fixed at ~100 ms) |
| Brief packet loss bursts | Local listener | Opus PLC + jitter buffer | — (automatic) |
| Playback-side OS scheduling stalls | **Local** listener | malgo playback period buffer | `comms.playbackLatencyMs` |
| Capture-side OS scheduling stalls | **Remote** listeners | malgo capture period buffer | `comms.captureLatencyMs` |

Both `playbackLatencyMs` and `captureLatencyMs` default to **60 ms** (three 20 ms Opus frames of slack), which is the right starting point for typical hardware.

> **Capture-side stalls are heard by the people you're talking to, not by you.** If the capture audio thread is preempted, the ADC device buffer overruns and samples are silently dropped *before* they ever hit the encoder. Your local monitor sounds fine; every remote listener hears stutter. This is the least obvious failure mode — always ask a peer, not just yourself.

The per-PTT cycle stats log line surfaces the relevant counters so you don't have to guess which buffer is short:

- `encode_dur_max` / `encode_dur_avg` vs `frame_budget` (20 ms) — if max approaches budget, libopus is starving the audio thread; lower `encoderComplexity`.
- `capture_gap_max` and `capture_late` — the largest inter-arrival between successive capture callbacks and the count of callbacks ≥ 2× budget late. Both signal capture-thread preemption; raise `captureLatencyMs` or hunt the CPU contender.
- `dropped` — frames the producer dropped because the encode channel was full (consumer fell more than 200 ms behind cumulatively).

---

## Volume buttons (OpenVLM - Not Active Yet)

The `openvlm` control source decodes more than just the PTT button. The OpenVLM dongle's VOL+ and VOL− buttons emit aux events on a separate channel, and a built-in `alsa.Controller` translates them into `Master` mixer steps on the ALSA card the OpenVLM was auto-detected as. The card index comes from the `ALSA_CARD` environment variable that comms set during startup; if `ALSA_CARD` is unset or non-numeric, volume events are silently ignored. The default control name is `Master` (one raw step ≈ 1 dB on CM108B). Errors from ALSA are swallowed — a missing mixer never crashes the daemon.

This is automatic in `openvlm` mode and uses a pure-Go ALSA binding, so it works without `alsa-utils` installed on the target.

---

## Web mode

When `controlSource` is set to `web`, the malgo audio backend is not initialized at all:

- No capture stream is opened, no playback streams are opened, no sound card is touched.
- The browser is the audio device. It captures mic audio, encodes Opus in-browser, and posts frames to the daemon via an RPC stream; the daemon just fans them out to the multicast sockets.
- Incoming RTP is parsed and its raw Opus payloads are forwarded back to the browser for decoding and playback.

This makes comms usable on nodes that have no sound card or whose audio hardware isn't supported — as long as the node can serve the OpenMANET web UI, it can participate in PTT voice.
