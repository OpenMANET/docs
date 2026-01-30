---
layout: default
title: GNSS
nav_order: 9
permalink: /gnss
description: GPS/GNSS integration for position tracking, ATAK CoT messaging, and NMEA streaming to end-user devices.
---

# GNSS

OpenMANET includes integrated GPS/GNSS support for position tracking, location sharing, and ATAK integration. When enabled, nodes automatically share their position across the mesh and stream NMEA data to connected end-user devices (EUDs).

---

## Overview

The GNSS module connected or included with your mesh radio provides:

- **Automatic position tracking** from TPV (Time-Position-Velocity) reports
- **NMEA GGA sentence streaming** to connected EUD clients
- **ATAK CoT (Cursor on Target) messaging** for tactical situational awareness
- **Position data in mesh announcements** for topology visualization
- **Position data in the API** for extendable functionality

---

## Supported Hardware
**Seeed WM1302 + Wio-WM6108**
The GPS receiver is automatically enabled on OpenMANET hardware.  The device appears at `/dev/ttyACM0`.

Any **u-blox based USB GPS receiver** that works with gpsd should also function with OpenMANET. This may require manual configuration of `/etc/config/gpsd`.

---

## How It Works

### Connection & Monitoring

The GPS service in OpenMANET establishes a persistent connection to the GPS an generates both position reports and can stream NMEA data:

- **Latitude/Longitude** (decimal degrees)
- **Altitude** (meters above sea level)
- **Speed** (meters per second)
- **Track/Course** (degrees)
- **GPS fix mode** (2 = 2D fix, 3 = 3D fix)
- **Timestamp** (UTC)

The service maintains automatic reconnection with exponential backoff (up to 3 attempts) if the GPS daemon becomes unavailable.

### Position Validity

A position is considered valid when:
- GPS fix mode is **2 (2D)** or **3 (3D)**
- Position data has been received from gpsd

Invalid positions (mode 0 or 1) are not shared with EUDs or included in mesh announcements.

### NMEA Streaming to EUDs

When a valid GPS position is available, the service automatically:

1. Converts the position to **NMEA GGA format** (Global Positioning System Fix Data)
2. Queries current DHCP leases to find connected EUD clients
3. Sends the NMEA sentence via UDP to each EUD on **port 4349** (Standard Port in ATAK)

**NMEA GGA Format Example:**
```
$GPGGA,123045.00,3746.4946,N,12225.1640,W,1,08,1.0,50.0,M,0.0,M,,*47
```

This allows tactical applications like ATAK to receive real-time GPS position from the mesh node.

### ATAK CoT Integration

If **no DHCP leases are found** (no EUDs connected), the GPS service falls back to sending **Cursor on Target (CoT) protobuf messages** to the ATAK Situational Awareness multicast address:

- **Multicast Address:** `239.2.3.1:6969`
- **Message Type:** `G-U-U-S-R` (Ground Unit / Radio Unit)
- **Callsign:** `<hostname>-manet`
- **Platform:** Derived from board model (e.g., "Raspberry Pi 4 Model B")

The CoT message includes:
- **Position:** Latitude, longitude, altitude (HAE - Height Above Ellipsoid)
- **Group:** "MANET" with role "Radio Unit"
- **Track data:** Speed and course
- **Device info:** OS ("OpenMANET"), hostname, and platform name

This enables ATAK clients on the mesh to see OpenMANET nodes as radio units on the tactical map.

### Position Sharing in Mesh Announcements

When GPS is enabled and a valid position is available, nodes include their position in **node data announcements** sent periodically across the mesh.

This allows:
- Web UI map visualization of node locations
- Topology tracking with geographic context
- Range testing with position logging

The node data worker checks GPS validity before including position data (mode > 1) and logs the current position at the debug level.

---

## API / Position Access


## Automatic Reconnection

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
- Confirm GPS is enabled in `/etc/openmanet/config.yaml`
- Check logs: `logread | grep gps`
- Verify gpsd connection: `telnet localhost 2947` (type `?WATCH={"enable":true,"json":true}`)
- Restart openmanetd: `/etc/init.d/openmanetd restart`

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
- **Position updates** are sent whenever the GPS service receives a new TPV report with a valid fix
- **Checksums** are automatically calculated for NMEA sentences to ensure data integrity
