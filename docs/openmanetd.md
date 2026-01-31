---
layout: default
title: OpenMANETd Daemon
nav_order: 10
permalink: /openmanetd
description: openmanetd is the core management daemon for the OpenMANET mesh network.
---

## Overview

`openmanetd` is the core management daemon for the OpenMANET mesh network. It handles low-level configuration tasks, node discovery, IP address management, and provides both internal mesh communication and an external API for monitoring and control.

### Key Responsibilities

- **Node Discovery**: Publishes and receives mesh node information across the network
- **IP Address Management**: Handles static IP assignment and DHCP address reservations
- **Gateway Management**: Updates gateway routes and manages gateway mode operations
- **Position Tracking**: Distributes GPS position data when enabled
- **API Services**: Provides REST/gRPC API for monitoring mesh status and nodes

---

## Configuration

### Configuration File

The daemon uses a YAML configuration file located at `/etc/openmanetd/config.yml` by default.

### Configuration Structure

Below is the **default** `config.yml` with all available options:

```yaml
# Log level: "debug", "info", "warn", "error" (default: "info")
logLevel: info

# Mesh network interface (default: "br-ahwlan")
meshNetInterface: br-ahwlan

# Database file path (default: "/etc/openmanetd/openmanetd.db")
# DO NOT CHANGE
dbFile: /etc/openmanetd/openmanetd.db

# GNSS/GPS configuration
gnss:
  # Enable GNSS functionality (default: false)
  enable: true

  # External GNSS source configuration
  sendAsExternalGNSSSource:
    # Send position as NMEA sentences (default: false)
    sendAsNMEA: true
    # Send position as Cursor-on-Target (CoT) (default: false)
    sendAsCoT: true

# Alfred configuration
alfred:
  # Alfred mode: "primary" or "secondary" (default: "primary")
  mode: "primary"

  # BATMAN-adv interface (default: "bat0")
  batInterface: "bat0"

  # Alfred socket path (default: "/var/run/alfred.sock")
  socketPath: "/var/run/alfred.sock"

  # Data types to enable
  dataTypes:
    gateway: true          # Gateway announcements (default: true)
    node: true            # Node information (default: true)
    position: true        # GPS position data (default: true)
```

### Configuration Defaults

| Option | Default Value | Description |
|--------|--------------|-------------|
| `logLevel` | `info` | Logging level (debug, info, warn, error) |
| `meshNetInterface` | `br-ahwlan` | Primary mesh network interface |
| `dbFile` | `/etc/openmanetd/openmanetd.db` | SQLite database location |
| `resetDBOnStart` | `false` | Clear database on daemon startup |
| `gnss.enable` | `true` | Enable GNSS/GPS functionality |
| `gnss.sendAsExternalGNSSSource.sendAsNMEA` | `true` | Send position as NMEA sentences |
| `gnss.sendAsExternalGNSSSource.sendAsCoT` | `true` | Send position as CoT messages |
| `alfred.mode` | `primary` | Alfred synchronization mode |
| `alfred.batInterface` | `bat0` | BATMAN-adv interface name |
| `alfred.socketPath` | `/var/run/alfred.sock` | Alfred Unix socket |
| `alfred.dataTypes.gateway` | `true` | Publish gateway information |
| `alfred.dataTypes.node` | `true` | Publish node information |
| `alfred.dataTypes.position` | `true` | Publish GPS positions |

### Hot Reload

The configuration file is watched for changes and automatically reloaded when modified. This allows you to adjust settings without restarting the daemon. Changes are logged to the console:

```
Using config file: /etc/openmanetd/config.yml
```

---

## Protocol Buffer Specifications

OpenMANETd uses Protocol Buffers (protobuf) for two distinct purposes:
1. **Internal Mesh Communication**: Data exchange via ALFRED across the mesh
2. **External API**: REST/gRPC API using ConnectRPC

### Protobuf Repository

The protobuf schema is hosted with Buf Registry:

- Repository: [OpenMANET Buf Schema Registry](https://buf.build/openmanet/protobufs)

### Internal Mesh Messages

These protobuf messages are serialized and distributed across the mesh network using ALFRED.

#### Node Message

Announces node presence, configuration, and network details:

```protobuf
message Node {
  string mac = 1;               // Hardware MAC address
  string hostname = 2;          // Node hostname
  string ipaddr = 3;            // Assigned IP address
  Position position = 4;        // GPS coordinates (optional)
  string uci_dhcp_start = 5;    // DHCP pool start IP
  string uci_dhcp_limit = 6;    // DHCP pool size limit
}
```

**Distribution**: Sent every 60 seconds by all nodes

**Usage Example**:
```go
nodeData := &proto.Node{
    Mac:      "aa:bb:cc:dd:ee:ff",
    Hostname: "mesh-node-01",
    Ipaddr:   "10.41.1.5",
    Position: &proto.Position{
        Latitude:  37.7749,
        Longitude: -122.4194,
        Altitude:  10.5,
    },
}
```

#### Gateway Message

Announces gateway nodes that provide internet connectivity:

```protobuf
message Gateway {
  string gateway_address = 1;   // Gateway IP address
  string gateway_mac = 2;       // Gateway MAC address
  int32 bandwidth_down = 3;     // Download bandwidth (kbit/s)
  int32 bandwidth_up = 4;       // Upload bandwidth (kbit/s)
}
```

**Distribution**:
- Send: Every 60 seconds (gateway nodes only)
- Receive: Every 10 seconds (all nodes)

**Usage**: Client nodes use this data to select and route to internet gateways.

---

## ConnectRPC API

The daemon provides an HTTP/2 REST/gRPC API using [ConnectRPC](https://connectrpc.com/), compatible with gRPC clients and standard HTTP clients.

The API specification is available on [Buf Build](https://buf.build/openmanet/protobufs) where you can get code generation SDKs.

### API Server Configuration

- **Address**: `0.0.0.0:8087` (all interfaces, port 8087)
- **Protocol**: HTTP/2 with h2c (unencrypted HTTP/2)
- **Timeout**: 30 seconds for read/write operations
- **Format**: Protocol Buffers (binary) or JSON

### Service Endpoints

#### NodeService

Manages mesh node information stored in the local database.

**ListNodes** - Get all known mesh nodes

```bash
# Using curl with JSON
curl -X POST http://{MESH_NODE_IO}:8087/openmanet.service.v1.NodeService/ListNodes \
  -H "Content-Type: application/json" \
  -d '{}'

# Response
{
  "nodes": [
    {
      "mac": "aa:bb:cc:dd:ee:ff",
      "hostname": "mesh-node-01",
      "ipaddr": "10.41.1.5",
      "position": {
        "latitude": 37.7749,
        "longitude": -122.4194,
        "altitude": 10.5
      }
    }
  ]
}
```

**GetNode** - Get specific node by hostname

```bash
curl -X POST http://{MESH_NODE_IO}:8087/openmanet.service.v1.NodeService/GetNode \
  -H "Content-Type: application/json" \
  -d '{"hostname": "mesh-node-01"}'

# Response
{
  "node": {
    "mac": "aa:bb:cc:dd:ee:ff",
    "hostname": "mesh-node-01",
    "ipaddr": "10.41.1.5"
  }
}
```

#### InterfaceService

Provides wireless interface information.

**ListWirelessInterfaces** - Get all wireless interfaces

```bash
curl -X POST http://{MESH_NODE_IO}:8087/openmanet.service.v1.InterfaceService/ListWirelessInterfaces \
  -H "Content-Type: application/json" \
  -d '{}'

# Response
{
  "interfaces": [
    {
      "index": 3,
      "name": "wlan0",
      "hardwareAddress": "aa:bb:cc:dd:ee:ff",
      "phy": 0,
      "device": 0,
      "interfaceType": "MESH",
      "frequency": 2437,
      "channelWidth": 20
    }
  ]
}
```

**GetWirelessInterface** - Get specific interface details

```bash
curl -X POST http://{MESH_NODE_IO}:8087/openmanet.service.v1.InterfaceService/GetWirelessInterface \
  -H "Content-Type: application/json" \
  -d '{"name": "wlan0"}'
```

#### MeshNeighborService

Shows directly connected mesh neighbors.

**ListMeshNeighbors** - Get all mesh neighbors

```bash
curl -X POST http://{MESH_NODE_IO}:8087/openmanet.service.v1.MeshNeighborService/ListMeshNeighbors \
  -H "Content-Type: application/json" \
  -d '{}'

# Response
{
  "neighbors": [
    {
      "neighbor": "mesh-node-02",
      "hardwareAddress": "11:22:33:44:55:66",
      "signalStrength": -45,
      "signal": -45,
      "throughput": 54000
    }
  ]
}
```

#### StatusService

Provides mesh network status and health information.

**GetServiceStatus** - Get overall mesh status

```bash
curl -X POST http://{MESH_NODE_IO}:8087/openmanet.service.v1.StatusService/GetServiceStatus \
  -H "Content-Type: application/json" \
  -d '{}'

# Response
{
  "status": {
    "isConnected": true,
    "connectedNeighbors": 3,
    "activeMeshInterfaces": 2,
    "isMeshGateway": false
  }
}
```

### Using gRPC Clients

The API is fully compatible with standard gRPC tooling:

```bash
# Using grpcurl
grpcurl -plaintext -d '{}' \
  {MESH_NODE_IO}:8087 \
  openmanet.service.v1.NodeService/ListNodes

# Using buf curl (with Connect protocol)
buf curl --http2-prior-knowledge \
  --protocol connect \
  http://{MESH_NODE_IO}:8087/openmanet.service.v1.NodeService/ListNodes
```

---

## Management Workers

The daemon uses a worker-based architecture for mesh operations. Each worker runs in its own goroutine with periodic intervals.

### Node Data Worker

**Purpose**: Announces node presence and collects information from other nodes

**Send Interval**: 60 seconds

**Receive Interval**: Continuous listening

**Operations**:
- Publishes local node information (MAC, hostname, IP, position)
- Receives node announcements from other mesh nodes
- Updates local database with discovered nodes
- Handles duplicate node detection and cleanup

### Gateway Worker

**Purpose**: Manages gateway announcements and route updates

**Send Interval**: 60 seconds (gateway mode only)

**Receive Interval**: 10 seconds (all nodes)

**Operations**:
- Gateway nodes: Announce availability and bandwidth
- Client nodes: Discover gateways and update routing tables
- Monitor gateway connectivity and failover

**ALFRED Data Type**: Type 1 (standard BATMAN-adv gateway)

---

## Troubleshooting

### Common Issues

#### API Server Not Responding

**Symptoms**: Connection refused on port 8087

**Checks**:
```bash
# Verify daemon is running
ps aux | grep openmanetd

# Check port binding
netstat -tulpn | grep 8087

# Test local connection
curl http://127.0.0.1:8087/openmanet.service.v1.StatusService/GetServiceStatus
```

**Solutions**:
- Verify no firewall blocking port 8087
- Check daemon logs for startup errors
- Ensure no other service using port 8087

#### No Mesh Nodes Discovered

**Symptoms**: Empty response from ListNodes API

**Checks**:
```bash
# Check ALFRED is receiving data
alfred -r <node-data-type>

# Verify mesh connectivity
batctl n

# Check worker status in logs
grep "NodeDataWorker" /var/log/openmanetd.log
```

**Solutions**:
- Verify `alfred.dataTypes.node: true` in config
- Check mesh network connectivity
- Ensure other nodes are running openmanetd
- Verify BATMAN-adv interface is bridged correctly


#### GPS Position Not Publishing

**Symptoms**: Position data null in node announcements

**Checks**:
```bash
# Verify GPS service
/etc/init.d/gpsd status

# Check GPS fix
gpsmon
```

**Solutions**:
- Verify gpsd is running and has GPS fix
- Check gpsd socket connection
- See [GNSS Documentation](gnss.md) for GPS troubleshooting

### Debug Mode

Enable debug logging by modifying the daemon startup or configuration:

```bash
# Set log level via environment
export LOG_LEVEL=debug
openmanetd
```

### Monitoring

Check daemon health:

```bash
# System service status
/etc/init.d/openmanetd status

# View recent logs
logread | grep openmanetd

# API health check
curl http://127.0.0.1:8087/openmanet.service.v1.StatusService/GetServiceStatus
```

---

## See Also

- [GNSS/GPS Configuration](gnss.md) - GPS integration details
- [Networking](networking.md) - Mesh networking setup
- [Hardware](hardware.md) - Supported hardware platforms
- [Initial Setup](initial-setup.md) - Device configuration guide

## External Resources

- [ConnectRPC Documentation](https://connectrpc.com/docs/introduction)
- [Protocol Buffers Guide](https://protobuf.dev/)
- [BATMAN-adv Documentation](https://www.open-mesh.org/projects/batman-adv/wiki)
- [ALFRED Protocol](https://www.open-mesh.org/projects/alfred/wiki)
- [OpenMANET Protobufs Repository](https://buf.build/openmanet/protobufs)
