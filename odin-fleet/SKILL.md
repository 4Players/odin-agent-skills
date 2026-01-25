---
name: odin-fleet
description: |
  ODIN Fleet for deploying and managing game servers across a global network.
  Use when: deploying game servers, creating Docker images or Steamworks builds, configuring server
  settings (CPU, memory, ports, environment variables), managing deployments, using the Fleet CLI,
  or automating server management via API. Engine-agnostic - works with any game server.
---

# ODIN Fleet

Game server deployment and management platform by 4Players.

## Core Concepts

### Architecture Layers

```
Images → Server Configs → Deployments → Servers
   ↓           ↓              ↓            ↓
Docker/     Resources,      Location,    Running
Steam       Ports, Env     Instances    Instances
```

| Layer | Description |
|-------|-------------|
| **Image** | Docker container or Steamworks build with server software |
| **Server Config** | Blueprint: resources, ports, env vars, persistence |
| **Deployment** | Where and how many servers run |
| **Server** | Individual running container instance |

## Deployment Workflow

```
1. Create Image
   └─ Docker registry (Docker Hub, GitHub, GitLab, AWS, Azure, GCP)
   └─ Steamworks (app ID, branch, credentials)

2. Create Server Configuration
   ├─ Define ports and protocols
   ├─ Set environment variables
   ├─ Configure resources (CPU, memory)
   ├─ Mount persistent folders
   └─ Add config/secret files

3. Create Deployment
   ├─ Select location (country, city)
   ├─ Choose configuration
   └─ Set instance count

4. Monitor & Manage
   ├─ View logs and metrics
   ├─ Manage backups
   └─ Scale up/down
```

## CLI Tool

### Installation
```bash
git clone https://github.com/4Players/fleet-cli.git
cd fleet-cli && deno task build
chmod +x odin  # macOS/Linux
```

### Authentication
```bash
odin login                           # Interactive
odin login --accessToken=<token>     # Direct token
```

### App Selection
```bash
odin apps list                       # List all apps
odin apps select                     # Interactive selection
```

### Image Management
```bash
# List images
odin fleet images list

# Create Docker image
odin fleet images create \
  --name="MyServer" \
  --version="1.0.0" \
  --os=linux \
  --type=dockerImage \
  --docker-image=myimage:latest \
  --registry-id=1

# Create Steamworks image
odin fleet images create \
  --name="MyServer" \
  --version="1.0.0" \
  --os=windows \
  --type=steam \
  --steam-app-id=123456 \
  --branch=public \
  --steamcmd-username=user \
  --steamcmd-password=pass \
  --command=server.exe

# Delete image
odin fleet images delete <image-id>
```

### Configuration Management
```bash
# List configs
odin fleet configs list

# Create config (interactive)
odin fleet configs create

# Create with JSON payload
odin fleet configs create --payload='<json>'

# Preview without executing
odin fleet configs create --dry-run

# Update config
odin fleet configs update <config-id> --payload='<json>'

# Delete config
odin fleet configs delete <config-id>
```

### Deployment Management
```bash
# List deployments
odin fleet deployments list

# Create deployment
odin fleet deployments create \
  --name="Production" \
  --config-id=<config-id> \
  --num-instances=3 \
  --country=DE \
  --city=Frankfurt

# Scale instances
odin fleet deployments update <deployment-id> --num-instances=5

# Delete deployment
odin fleet deployments delete <deployment-id>
```

### Server Management
```bash
# List servers
odin fleet servers list

# Get server details
odin fleet servers get <server-id>

# View logs
odin fleet servers logs <server-id>

# Restart server
odin fleet servers restart <server-id>

# Manage backups
odin fleet servers backups <server-id>
```

### Global Flags
```bash
--appId <id>           # Override selected app
--accessToken <token>  # Override stored token
--format <format>      # Output format (value(id), json)
--dry-run              # Preview without executing
--force                # Skip confirmation
```

## Server Configuration

### Port Configuration
```json
{
  "ports": [
    {
      "name": "Game",
      "containerPort": 7777,
      "protocol": "UDP"
    },
    {
      "name": "Query",
      "containerPort": 27015,
      "protocol": "TCP"
    }
  ]
}
```
Ports are dynamically mapped to available host ports.

### Environment Variables

**Types:**
- **Static**: Fixed values (server name, game mode)
- **System**: Runtime values (`IP-Address`, `Instance-ID`)
- **Port**: Published port numbers (e.g., `GAME_PORT`)

```json
{
  "env": [
    { "key": "SERVER_NAME", "value": "My Server" },
    { "key": "GAME_MODE", "value": "deathmatch" },
    { "key": "DIFFICULTY", "value": "hard" }
  ]
}
```

### Resources
```json
{
  "resources": {
    "cpu": 1,           // CPU cores (shared or dedicated)
    "memory": 2048      // Memory in MB
  }
}
```

### Persistent Storage
```json
{
  "mounts": [
    { "path": "/data" },
    { "path": "/logs" }
  ]
}
```
Data survives restarts with automatic backup/restore.

### Restart Policies
- `Never`: Manual restart only
- `OnFailure`: Restart on non-zero exit
- `Always`: Always restart (recommended for production)

## Node.js SDK

### Installation
```bash
npm install @4players/fleet-nodejs
```

### Usage
```typescript
import { FleetApiClient } from '@4players/fleet-nodejs';

const client = new FleetApiClient("YOUR_ACCESS_TOKEN");
client.selectAppId(appId);

// Create image
await client.createBinary({
  name: "MyServer",
  version: "1.0.0",
  os: "linux",
  type: "dockerImage",
  dockerImage: "myimage:latest"
});

// Create config
await client.createServerConfig({
  name: "Production",
  binaryId: imageId,
  restartPolicy: "Always",
  ports: [{ name: "Game", containerPort: 7777, protocol: "UDP" }],
  resources: { cpu: 1, memory: 2048 }
});

// Create deployment
await client.createDeployment({
  name: "EU Production",
  serverConfigId: configId,
  numInstances: 3,
  locationConstraint: { country: "DE" }
});

// List servers
const servers = await client.getServers();

// Get locations
const locations = await client.getLocations();
```

## Supported Registries

- Docker Hub (public/private)
- GitHub Container Registry
- GitLab Container Registry
- AWS ECR
- Azure Container Registry
- Google Cloud Artifact Registry
- Self-hosted registries
- Custom registries

## Example: Minecraft Server

```bash
# 1. Create image
odin fleet images create \
  --name="Minecraft" \
  --version="2024.6.1" \
  --os=linux \
  --type=dockerImage \
  --docker-image=itzg/minecraft-server:2024.6.1 \
  --registry-id=1

# 2. Create config (use interactive or JSON)
odin fleet configs create

# 3. Create deployment
odin fleet deployments create \
  --name="Minecraft EU" \
  --config-id=<config-id> \
  --num-instances=1 \
  --country=DE \
  --city=Frankfurt

# 4. Get server IP and port
odin fleet servers list
```

## Documentation

Dashboard: https://console.4players.io
API Docs: https://fleet.4players.io/docs/api
CLI GitHub: https://github.com/4Players/fleet-cli
SDK GitHub: https://github.com/4Players/fleet-sdk-nodejs
