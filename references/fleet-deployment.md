# Deployment Patterns

Best practices, CI/CD integration, and deployment strategies.

## Table of Contents

- [Server Lifecycle Management](#server-lifecycle-management)
- [CI/CD Integration](#cicd-integration)
- [Best Practices](#best-practices)
- [Common Integration Scenarios](#common-integration-scenarios)

---

## Server Lifecycle Management

### Monitoring

```typescript
// Get server stats
const server = await client.getServer(serverId);
console.log(server.status);  // running, starting, stopped

// Get logs
const logs = await client.getServerLogs(serverId, {
  tail: 100,
  timestamps: true,
  stdout: true,
  stderr: true
});
```

### Backup Management

```typescript
// Create backup
await client.createServerBackup(serverId, { name: "Pre-update backup" });

// Restore from backup
await client.restoreServerBackup(serverId, backupId);

// Download backup
const downloadUrl = await client.getBackupDownloadUrl(serverId, backupId);
```

### Scaling

```typescript
// Scale deployment
await client.updateDeployment(deploymentId, { numInstances: 10 });

// Create deployment in new region
const usDeployment = await client.createDeployment({
  name: "US West Production",
  serverConfigId: config.id,
  numInstances: 5,
  placement: { constraints: { country: "us", city: "seattle" } }
});
```

---

## CI/CD Integration

### Deployment Pipeline Example

```bash
#!/bin/bash
# Build and deploy game server

# 1. Build Docker image
docker build -t myregistry/gameserver:$VERSION .
docker push myregistry/gameserver:$VERSION

# 2. Create Fleet image
IMAGE_ID=$(odin fleet images create \
  --name="GameServer" \
  --version="$VERSION" \
  --os=linux \
  --type=dockerImage \
  --docker-image="myregistry/gameserver:$VERSION" \
  --registry-id=$REGISTRY_ID \
  --format="value(id)")

# 3. Create/update server config
CONFIG_ID=$(odin fleet configs create \
  --payload="$(cat config.json)" \
  --force \
  --format="value(id)")

# 4. Update existing deployments
odin fleet deployments update $DEPLOYMENT_ID \
  --config-id=$CONFIG_ID --force

# 5. Wait for rollout
while [ $(odin fleet servers list \
  --filter="deployment.id = $DEPLOYMENT_ID AND status.state = 'running'" \
  --format="value(count(id))") -lt $NUM_INSTANCES ]; do
  sleep 10
done
```

### GitHub Actions Example

```yaml
name: Deploy to ODIN Fleet

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Build and push Docker image
        run: |
          docker build -t ghcr.io/${{ github.repository }}:${{ github.sha }} .
          docker push ghcr.io/${{ github.repository }}:${{ github.sha }}

      - name: Deploy to Fleet
        env:
          ODIN_API_KEY: ${{ secrets.ODIN_API_KEY }}
          ODIN_APP_ID: ${{ secrets.ODIN_APP_ID }}
        run: node deploy.js
```

---

## Best Practices

### Resource Sizing

1. **Start high, optimize down:** Begin with higher resources, monitor usage, adjust
2. **Monitor stats:** Use Fleet dashboard to track CPU, memory, network, disk
3. **Shared vs Dedicated:**
   - Use shared for dev/testing
   - Use dedicated for production with predictable performance needs

### Configuration Strategy

1. **Separate configs for environments:**
   - `GameServer-Dev` (smaller resources, debug enabled)
   - `GameServer-Staging` (production-like resources)
   - `GameServer-Prod` (optimized resources, restart=always)

2. **Version images explicitly:**
   - Use `myserver:1.0.0` not `myserver:latest`
   - Track changes through versioning

### Deployment Strategy

**Blue-Green Deployments:**

```typescript
// Create new deployment (green)
const greenDeployment = await client.createDeployment({
  name: "Production Green",
  serverConfigId: newConfigId,
  numInstances: 3,
  placement: { constraints: { country: "de" } }
});

// Wait for health
await waitForHealthy(greenDeployment.id);

// Delete old deployment (blue)
await client.deleteDeployment(oldDeploymentId);
```

**Geographic distribution:**
- Deploy to multiple regions for low latency
- Use location constraints in deployments

### Monitoring & Alerts

1. **Track key metrics:**
   - Running vs total instances
   - Server utilization (CPU, memory)
   - Player distribution across servers

2. **Implement health checks:**
   - Regular server list polling
   - Metadata state validation
   - Alert on instance failures

---

## Common Integration Scenarios

### Scenario 1: Match-Based Game with Dedicated Servers

1. Player requests match → Backend matchmaking service
2. Service finds/creates available server via Fleet API
3. Service updates server metadata: `instance_state="occupied"`
4. Players connect to server IP:port
5. Match ends → Server updates metadata: `instance_state="idle"`

### Scenario 2: Persistent World Servers

1. Create deployments with persistent storage mounted
2. Regular backups of world data
3. Server metadata tracks world state, player count
4. Server browser lists available worlds
5. Auto-restart on crashes to maintain uptime

### Scenario 3: Development/QA Test Servers

1. CI/CD creates temporary deployment on PR
2. QA team tests build
3. Deployment deleted after testing
4. Cost-effective: only pay for active test time
