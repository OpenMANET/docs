---
layout: default
title: GNSS
nav_order: 9
permalink: /gnss
description: GPS/GNSS integration for position tracking, ATAK CoT messaging, and NMEA streaming to end-user devices.
---

# GNSS

OpenMANET includes integrated position tracking, location sharing, and ATAK integration. A node can use either its own GPS/GNSS receiver or position CoT broadcast by a directly connected EUD such as ATAK.

---

## Overview

The OpenMANET position subsystem provides:

- **Automatic position tracking** from TPV (Time-Position-Velocity) reports
- **External CoT position input** from a directly connected ATAK EUD
- **NMEA GGA sentence streaming** to connected EUD clients
- **Position data in mesh announcements** for topology visualization
- **Position data in the API** for extendable functionality

---

## Supported Hardware
**Seeed WM1302 + Wio-WM6108**
The GPS receiver is automatically enabled on OpenMANET hardware.  The device appears at `/dev/ttyACM0`.
Please Note:
- The GPS antenna that comes with the Wio-WM6108 is not that great.  Consider getting a better antenna.
- Do not expect to get any or much signal indoors.  GNSS signals are fairly week unless you have a really good antenna.

Any **u-blox based USB GPS receiver** that works with gpsd should also function with OpenMANET. This may require manual configuration of `/etc/config/gpsd`.

A physical receiver is optional when the node uses the external CoT position source described below.

---

## How It Works

### Position Source

Select the position source from the **GNSS Source** field on the Web UI GPS page:

- **Internal · Local Receiver** is the default. The node reads position and satellite data from a receiver managed by gpsd.
- **External · EUD (ATAK) via CoT** lets a node without a local receiver adopt position CoT broadcast by a directly connected EUD.

External mode listens on the ATAK SA multicast group at `239.2.3.1:6969`. It accepts position only from an EUD found in the node's active DHCP leases, so CoT relayed from elsewhere on the mesh is ignored. Because the position comes from the EUD, satellite and sky-plot data are not available in this mode.

The equivalent `/etc/openmanetd/config.yml` setting is:

```yaml
gnss:
  enable: true
  source: external_cot
```

Use `source: internal` to select the local gpsd receiver. Source changes take effect without restarting `openmanetd`.

### Connection & Monitoring

In internal mode, the GPS service establishes a persistent connection to gpsd, generates position reports, and can stream NMEA data:

- **Latitude/Longitude** (decimal degrees)
- **Altitude** (meters above sea level)
- **Speed** (meters per second)
- **Track/Course** (degrees)
- **GPS fix mode** (2 = 2D fix, 3 = 3D fix)
- **Timestamp** (UTC)

The service maintains automatic reconnection with exponential backoff (up to 3 attempts) if the GPS daemon becomes unavailable.

### Position Validity

A position is considered valid when it comes from the selected source:

- In **internal** mode, GPS fix mode is **2 (2D)** or **3 (3D)** and position data has been received from gpsd.
- In **external CoT** mode, a valid position event has been received from a directly connected EUD.

Invalid positions are not shared with EUDs or included in mesh announcements.

### NMEA Streaming to EUDs

When a valid internal GPS position is available, the service automatically:

1. Converts the position to **NMEA GGA format** (Global Positioning System Fix Data)
2. Queries current DHCP leases to find connected EUD clients
3. Sends the NMEA sentence via UDP to each EUD on **port 4349** (Standard Port in ATAK)

**NMEA GGA Format Example:**
```
$GPGGA,123045.00,3746.4946,N,12225.1640,W,1,08,1.0,50.0,M,0.0,M,,*47
```

This allows tactical applications like ATAK to receive real-time GPS position from the mesh node.

### Position Sharing in Mesh Announcements

When GNSS tracking is enabled and the selected source has a valid position, nodes include their position in **node data announcements** sent periodically across the mesh.

This allows:
- Web UI map visualization of node locations
- Topology tracking with geographic context
- Range testing with position logging

The node data worker checks position validity before including position data and logs the current position at the debug level.

### Camera Sensor CoT

On Raspberry Pi nodes with a detected camera, a valid position allows `openmanetd` to place the camera on the ATAK map. That position can come from either the internal GPS receiver or the external CoT source. The node sends a linked camera video event and Sensor CoT event to the ATAK SA multicast address at `239.2.3.1:6969`. The advertised video URL uses the node's OpenMANET address and the MediaMTX RTSP path `rtsp://<node-ip>:554/rpicamera`.

Both the camera and a valid position are required. A camera can still provide RTSP video without a position, but it will not appear as a positioned sensor in ATAK. See [Camera Support](./hardware/cameras) for supported sensors and troubleshooting.

---

## Internal Receiver Reconnection

The GPS service includes robust connection handling:

- **Initial connection:** Connects to gpsd on startup
- **Automatic reconnection:** Retries up to **3 times** with a **5-second delay** between attempts
- **Connection loss recovery:** Automatically detects connection drops and re-establishes the session
- **Graceful shutdown:** Closes connections cleanly when the service stops

If the maximum reconnection attempts are reached, the service gives up and logs an error. Restarting `openmanetd` will reset the reconnection counter.

---

## Integration with Other Services

### Range Testing
The built-in range testing script (`/usr/bin/rangetest.sh`) logs GPS position alongside signal strength metrics when GPS is enabled, allowing you to correlate range performance with geographic location.

### Web UI
Future releases may include a map view showing node positions on the topology page when GPS data is available.

---

## Troubleshooting

### GPS not acquiring fix
- Ensure the GPS receiver has a clear view of the sky
- Wait 30-60 seconds for cold start (first fix after power-on)
- Check that gpsd is running: `ps | grep gpsd`
- Verify the GPS device is detected: `ls /dev/ttyUSB*`

### No position data in mesh
- Confirm GNSS is enabled in `/etc/openmanetd/config.yml`
- Confirm `gnss.source` matches the intended position source
- Check logs: `logread | grep gps`
- Verify gpsd connection: `telnet localhost 2947` (type `?WATCH={"enable":true,"json":true}`)
- Restart openmanetd: `/etc/init.d/openmanetd restart`

### External CoT position is not being adopted

- Confirm `gnss.source` is `external_cot` or select **External · EUD (ATAK) via CoT** in the Web UI.
- Confirm the ATAK device is directly connected to this node and has an active DHCP lease: `cat /tmp/dhcp.leases`.
- Confirm ATAK is broadcasting position CoT to `239.2.3.1:6969`.
- Check incoming traffic: `tcpdump -ni any 'udp dst 239.2.3.1 and port 6969'`.
- Remember that the node ignores CoT from devices that are not in its local DHCP leases.

### EUD not receiving NMEA data
- Confirm the EUD has a DHCP lease: `cat /tmp/dhcp.leases`
- Verify the EUD application is listening for NMEA on the correct port

### ATAK not showing node position
- Verify multicast is working across the mesh
- Ensure ATAK is subscribed to the SA multicast group `239.2.3.1:6969`

---

## Notes

- **NMEA streaming** uses UDP, so delivery is not guaranteed if network conditions are poor
- **CoT multicast** requires multicast routing to be enabled across the mesh (BATMAN-V handles this automatically)
- **Position updates** are sent whenever the selected source provides a valid position
- **Checksums** are automatically calculated for NMEA sentences to ensure data integrity
