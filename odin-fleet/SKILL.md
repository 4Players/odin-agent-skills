---
name: odin-fleet
description: |
  ODIN Fleet platform for deploying and managing game servers globally. Use when: deploying dedicated
  servers, integrating with matchmaking systems (GameLift FlexMatch, Nakama), building server browsers,
  configuring Docker/Steamworks servers, managing resources and scaling, implementing CI/CD pipelines,
  or automating server lifecycle management via API/SDK. Engine-agnostic platform supporting any game engine.
license: MIT
---

# ODIN Fleet Platform

Global game server deployment and management platform optimized for real-time, stateful hosting (world-building games) and versatile for short-term, stateless tasks (match-based games).

## Architecture Overview

```
Images → Server Configs → Deployments → Servers
   ↓           ↓              ↓            ↓
Docker/     Resources,      Location,    Running
Steam       Ports, Env     Instances    Instances
```

### Core Layers

**1. Images (Binary)**: Docker container or Steamworks build

**2. Server Configurations**: Blueprint defining resources, ports, env vars, mounts, restart policy

**3. Deployments**: WHERE and HOW MANY instances run (location, count)

**4. Servers (Instances)**: Running containers with unique IP:Port

## Key Concepts

### Image Types

**Docker Images:**
```typescript
{
  name: "GameServer",
  version: "1.0.0",
  type: "dockerImage",
  os: "linux",
  dockerImage: { imageName: "myregistry/gameserver:1.0.0", registryId: 123 }
}
```

**Steamworks Images:**
```typescript
{
  name: "GameServer",
  version: "1.0.0",
  type: "steam",
  os: "windows",
  steamworks: { appId: 123456, branch: "public", executable: "/gameserver/server.exe" }
}
```

### Port Configuration

```typescript
{
  ports: [
    { name: "Game Port", targetPort: 7777, protocols: ["UDP"], publishMode: "ingress" }
  ],
  env: [
    { key: "GAME_PORT", type: "portMapping", value: "Game Port" }  // Dynamic port
  ]
}
```

### Environment Variables

```typescript
// Static value
{ key: "SERVER_NAME", type: "static", value: "Production Server" }

// System value (IP-Address or Instance-ID)
{ key: "SERVER_IP", type: "system", value: "IP-Address" }

// Port mapping (published port number)
{ key: "GAME_PORT", type: "portMapping", value: "Game Port" }
```

### Resource Management

```typescript
{
  resources: {
    limits: { cpu: 2, memory: 4096 },        // Maximum
    reservations: { cpu: 1, memory: 2048 }   // Guaranteed minimum
  }
}
```

### Persistent Storage

```typescript
{
  mounts: [{ target: "/data", readOnly: false }]
}
```

### Restart Policies

- **never**: Manual restart only
- **on-failure**: Restart on non-zero exit code
- **any**: Always restart (recommended for production)

## Node.js SDK

### Installation

```bash
npm install @4players/fleet-nodejs
```

### Basic Usage

```typescript
import { FleetApiClient } from '@4players/fleet-nodejs';

const client = new FleetApiClient("YOUR_API_TOKEN");
client.selectAppId(appId);

// Create image
const image = await client.createBinary({
  name: "GameServer", version: "1.0.0", type: "dockerImage", os: "linux",
  dockerImage: { imageName: "myregistry/server:1.0.0", registryId: 1 }
});

// Create config
const config = await client.createServerConfig({
  name: "Production", binaryId: image.id,
  resources: { limits: { cpu: 2, memory: 4096 }, reservations: { cpu: 1, memory: 2048 } },
  ports: [{ name: "Game Port", targetPort: 7777, protocols: ["UDP"] }],
  env: [{ key: "GAME_PORT", type: "portMapping", value: "Game Port" }],
  restartPolicy: { condition: "any" }
});

// Create deployment
const deployment = await client.createDeployment({
  name: "EU Production", serverConfigId: config.id, numInstances: 3,
  placement: { constraints: { country: "de", city: "frankfurt" } }
});

// List servers
const servers = await client.getServers();
servers.forEach(s => {
  console.log(`${s.addr}:${s.ports["Game Port"]?.publishedPort}`);
});
```

### Server Metadata

```typescript
await client.updateServerMetadata(serverId, {
  instance_state: "occupied",
  game_mode: "deathmatch",
  player_count: 8
});
```

## REST API

**Base URL:** `https://fleet.4players.io`
**Auth:** `Authorization: Bearer YOUR_API_TOKEN`

**Key Endpoints:**
- `GET/POST /apps/{appId}/binaries` - Images
- `GET/POST /apps/{appId}/server-configs` - Configs
- `GET/POST /apps/{appId}/deployments` - Deployments
- `GET /apps/{appId}/servers` - List servers
- `PATCH /apps/{appId}/servers/{id}/metadata` - Update metadata
- `GET /apps/{appId}/servers/{id}/logs` - Get logs

Full API docs: https://fleet.4players.io/docs/api

## CLI Tool

See the **odin-fleet-cli** skill for CLI-specific documentation.

```bash
odin login --api-key=$API_KEY
odin fleet images list
odin fleet configs list
odin fleet deployments list
odin fleet servers list --format="table(id,addr,status.state)"
```

## Additional Documentation

- **Matchmaking**: See [references/matchmaking.md](references/matchmaking.md) for GameLift FlexMatch, Nakama, custom matchmaking
- **Deployment Patterns**: See [references/deployment-patterns.md](references/deployment-patterns.md) for CI/CD, blue-green, best practices

## Troubleshooting

**Server Not Starting:**
- Check logs: `odin fleet servers logs <server-id>`
- Verify port configuration matches container
- Ensure image exists and is accessible

**Ports Not Accessible:**
- Verify port protocol (TCP vs UDP)
- Check dynamic port env var is used correctly

**Metadata Not Updating:**
- Verify API token has write permissions
- Check JSON format in update requests

## Resources

- **Dashboard:** https://console.4players.io
- **API Docs:** https://fleet.4players.io/docs/api
- **Main Docs:** https://docs.4players.io/fleet/
- **Node.js SDK:** https://github.com/4Players/fleet-sdk-nodejs
- **CLI:** https://github.com/4Players/fleet-cli
- **Discord:** https://4np.de/discord
