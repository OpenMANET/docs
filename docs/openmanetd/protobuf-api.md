---
layout: default
title: Protobuf API
parent: OpenMANETd Daemon
nav_order: 1
permalink: /openmanetd/protobuf-api
description: Getting started with the OpenMANET protobuf API using ConnectRPC SDKs.
---

# Protobuf API

The OpenMANETd daemon exposes a protobuf-based API via [ConnectRPC](https://connectrpc.com/). The schema is published on the Buf Schema Registry at [buf.build/openmanet/protobufs](https://buf.build/openmanet/protobufs), which provides auto-generated SDKs for multiple languages. For curl-based examples and a full endpoint reference, see the [OpenMANETd Daemon](../openmanetd.md) page.

---

## Overview

OpenMANETd provides a type-safe API for monitoring and controlling mesh nodes programmatically. The API uses [ConnectRPC](https://connectrpc.com/), which supports the Connect protocol, gRPC, and gRPC-Web over a single HTTP/2 endpoint. This means you can use standard gRPC clients or ConnectRPC's lighter-weight clients interchangeably.

### Available Services

All services are under the `openmanet.service.v1` package:

| Service | Key Methods | Description |
|---------|-------------|-------------|
| `NodeService` | `ListNodes`, `GetNode` | Mesh node information |
| `InterfaceService` | `ListWirelessInterfaces`, `GetWirelessInterface` | Wireless interface details |
| `MeshNeighborService` | `ListMeshNeighbors` | Directly connected mesh neighbors |
| `StationService` | `ListStations`, `GetStation` | Connected station statistics |
| `StatusService` | `GetServiceStatus` | Mesh health and connectivity status |
| `GNSSService` | `GetGNSSStatus`, `GetGNSSConfig`, `UpdateGNSSConfig` | GPS position and configuration |
| `CommsService` | `GetCommsStatus`, `StreamAudioTx`, `StreamAudioRx` | Voice communications and push-to-talk |

---

## API Server Details

| Setting | Value |
|---------|-------|
| Default port | `8087` |
| Protocol | HTTP/2 with h2c (unencrypted) |
| Supported wire formats | Connect, gRPC, gRPC-Web |
| Serialization | Protocol Buffers (binary) or JSON |
| Timeout | 30 seconds |

> The server uses h2c (HTTP/2 cleartext, no TLS). Clients must be configured for unencrypted HTTP/2 connections. The examples below show how to handle this for each language.

---

## Generated SDKs

The [Buf Schema Registry](https://buf.build/openmanet/protobufs) (BSR) auto-generates client SDKs from the published protobuf schema. This is the recommended approach — no local code generation tooling required.

### Go

```bash
go get buf.build/gen/go/openmanet/protobufs/protocolbuffers/go
go get buf.build/gen/go/openmanet/protobufs/connectrpc/go
```

The first package provides the generated protobuf message types. The second provides the ConnectRPC service client stubs.

### TypeScript / JavaScript

Configure npm to use the Buf registry for the `@buf` scope, then install the generated packages:

```bash
npm config set @buf:registry https://buf.build/gen/npm/v1
```

```bash
npm install @buf/openmanet_protobufs.bufbuild_es @buf/openmanet_protobufs.connectrpc_es
```

Install the ConnectRPC runtime dependencies:

```bash
# For Node.js
npm install @connectrpc/connect @connectrpc/connect-node

# For browser applications
npm install @connectrpc/connect @connectrpc/connect-web
```

### Python

Install the generated packages from the Buf Python registry:

```bash
pip install --extra-index-url https://buf.build/gen/python \
  openmanet-protobufs-protocolbuffers-python \
  openmanet-protobufs-connectrpc-python
```

Install the ConnectRPC Python runtime:

```bash
pip install connectrpc
```

---

## Using ConnectRPC Clients

With the generated SDKs installed, you can create typed clients to interact with the API. The examples below demonstrate querying the mesh status using `StatusService.GetServiceStatus`.

### Go

```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"

	"connectrpc.com/connect"
	servicev1 "buf.build/gen/go/openmanet/protobufs/protocolbuffers/go/openmanet/service/v1"
	"buf.build/gen/go/openmanet/protobufs/connectrpc/go/openmanet/service/v1/servicev1connect"
)

func main() {
	client := servicev1connect.NewStatusServiceClient(
		http.DefaultClient,
		"http://{MESH_NODE_IP}:8087",
	)

	res, err := client.GetServiceStatus(
		context.Background(),
		connect.NewRequest(&servicev1.GetServiceStatusRequest{}),
	)
	if err != nil {
		log.Fatal(err)
	}

	status := res.Msg.Status
	fmt.Printf("Connected: %v\n", status.IsConnected)
	fmt.Printf("Neighbors: %d\n", status.ConnectedNeighbors)
	fmt.Printf("Active Interfaces: %d\n", status.ActiveMeshInterfaces)
	fmt.Printf("Gateway: %v\n", status.IsMeshGateway)
}
```

> Go 1.24+ supports h2c with `http.DefaultClient` out of the box. For older Go versions, configure an HTTP/2 transport using `golang.org/x/net/http2`.

### TypeScript (Node.js)

```typescript
import { createClient } from "@connectrpc/connect";
import { createConnectTransport } from "@connectrpc/connect-node";
import { StatusService } from "@buf/openmanet_protobufs.connectrpc_es/openmanet/service/v1/status_service_pb";

const transport = createConnectTransport({
  baseUrl: "http://{MESH_NODE_IP}:8087",
  httpVersion: "2",
});

const client = createClient(StatusService, transport);

async function main() {
  const res = await client.getServiceStatus({});
  const status = res.status;
  console.log(`Connected: ${status?.isConnected}`);
  console.log(`Neighbors: ${status?.connectedNeighbors}`);
  console.log(`Active Interfaces: ${status?.activeMeshInterfaces}`);
  console.log(`Gateway: ${status?.isMeshGateway}`);
}

main();
```

> For browser-based applications, use `@connectrpc/connect-web` with `createConnectTransport` or `createGrpcWebTransport` instead of `connect-node`.

### Python

```python
import asyncio
from openmanet.service.v1 import status_service_pb2
from openmanet.service.v1.status_service_connect import StatusServiceClient

async def main():
    client = StatusServiceClient("http://{MESH_NODE_IP}:8087")
    response = await client.get_service_status(status_service_pb2.GetServiceStatusRequest())

    status = response.status
    print(f"Connected: {status.is_connected}")
    print(f"Neighbors: {status.connected_neighbors}")
    print(f"Active Interfaces: {status.active_mesh_interfaces}")
    print(f"Gateway: {status.is_mesh_gateway}")

asyncio.run(main())
```

---

## Local Code Generation

If you prefer to generate code locally instead of using BSR-hosted SDKs (for example, to customize generation or work offline), you can use the `buf` CLI.

### Install Buf

```bash
# macOS
brew install bufbuild/buf/buf

# Other platforms: https://buf.build/docs/installation
```

### Generate Code

Create a `buf.gen.yaml` in your project. Here is an example for Go:

```yaml
version: v2
plugins:
  - remote: buf.build/protocolbuffers/go
    out: gen/go
    opt: paths=source_relative
  - remote: buf.build/connectrpc/go
    out: gen/go
    opt: paths=source_relative
```

Then generate from the BSR:

```bash
buf generate buf.build/openmanet/protobufs
```

This pulls the schema directly from the BSR and generates code into `gen/go/`. Adapt the plugins for other languages:

- **TypeScript**: Use `buf.build/bufbuild/es` and `buf.build/connectrpc/es`
- **Python**: Use `buf.build/protocolbuffers/python` and `buf.build/connectrpc/python`

---

## Next Steps

- [OpenMANETd Daemon](/openmanetd) -- Full API endpoint reference with curl examples
- [ConnectRPC Documentation](https://connectrpc.com/docs/introduction) -- Protocol details and advanced client configuration
- [Buf Schema Registry](https://buf.build/openmanet/protobufs) -- Browse the schema and view generated documentation
- [Protocol Buffers Guide](https://protobuf.dev/) -- Protobuf language reference
