---
name: odin-fleet-cli
description: |
  ODIN Fleet CLI tool for managing game servers, creating resources, and building CI/CD pipelines.
  Use when: automating fleet deployments, scripting server management, creating images/configs/deployments,
  filtering and formatting CLI output, building CI/CD pipelines, managing backups, viewing logs,
  or working with Fleet API via command line. Covers all CLI commands, output formatting, filters,
  and automation best practices.
---

# ODIN Fleet CLI

Command-line tool for managing ODIN Fleet resources, automating deployments, and building CI/CD pipelines.

## Installation

### Option 1: Download Pre-built Binary (Recommended)

Download the latest release for your platform from [GitHub Releases](https://github.com/4Players/fleet-cli/releases/latest):

**Linux:**
```bash
# Download and install
curl -L -o odin https://github.com/4Players/fleet-cli/releases/latest/download/odin-linux
chmod +x odin
sudo mv odin /usr/local/bin/
```

**macOS (Intel):**
```bash
# Download and install
curl -L -o odin https://github.com/4Players/fleet-cli/releases/latest/download/odin-macos-x64
chmod +x odin
sudo mv odin /usr/local/bin/
```

**macOS (Apple Silicon):**
```bash
# Download and install
curl -L -o odin https://github.com/4Players/fleet-cli/releases/latest/download/odin-macos-arm64
chmod +x odin
sudo mv odin /usr/local/bin/
```

**Windows:**
1. Download `odin-windows.exe` from [GitHub Releases](https://github.com/4Players/fleet-cli/releases/latest)
2. Rename to `odin.exe`
3. Add to your PATH or run from the download directory

### Option 2: Build from Source

**Requirements:**
- Deno runtime: [https://deno.land/](https://deno.land/)

```bash
git clone https://github.com/4Players/fleet-cli.git
cd fleet-cli
deno task build
chmod +x odin  # macOS/Linux only
```

The `odin` binary (or `odin.exe` on Windows) will be created in the project root. Move it to a directory in your PATH for global access.

### Option 3: Development Mode

```bash
git clone https://github.com/4Players/fleet-cli.git
cd fleet-cli
deno run --allow-all src/main.ts
```

## Authentication

### Login
```bash
# Interactive login
odin login

# Direct token login (CI/CD)
odin login --api-key=YOUR_API_KEY
```

The API key is stored in a local configuration file for subsequent commands. Get your API key from the [4Players Console](https://console.4players.io/settings/api-keys).

### App Selection
```bash
# List all apps
odin apps list

# Select app interactively
odin apps select

# Override app for specific command
odin <command> --app-id=123456
```

## Global Flags

These flags work with any ODIN CLI command:

```bash
--api-key=<string>        # Override stored API key (useful for CI/CD)
--app-id=<string>         # Override selected app
--format=<string>         # Output format: json, value, flattened, table
--force                   # Skip confirmation prompts
--quiet                   # Suppress informational output (errors only)
```

## App Management

### List Apps
```bash
odin apps list
odin apps list --format=json
odin apps list --format="table(id,name)"
```

### Get App Details
```bash
odin apps get
odin apps get --app-id=123456
```

### Create App
```bash
odin apps create
```

### Delete App
```bash
odin apps delete
odin apps delete --force  # Skip confirmation
```

## Image Management

### List Images
```bash
odin fleet images list
odin fleet images list --format="table(id,name,version,os,type)"
```

### Create Docker Image
```bash
# Interactive mode
odin fleet images create

# Non-interactive mode
odin fleet images create \
  --name="MyServer" \
  --version="1.0.0" \
  --os=linux \
  --type=dockerImage \
  --docker-image=myimage:latest \
  --registry-id=1 \
  --force

# Get only the ID (useful for scripts)
IMAGE_ID=$(odin fleet images create \
  --name="MyServer" \
  --version="1.0.0" \
  --os=linux \
  --type=dockerImage \
  --docker-image=myimage:latest \
  --registry-id=1 \
  --force \
  --format="value(id)")
```

**Docker Image Flags:**
- `--name`: Image name (required)
- `--version`: Image version
- `--os`: Operating system (linux, windows)
- `--type`: Image type (dockerImage or steam)
- `--docker-image`: Full Docker image path
- `--registry-id`: Docker registry ID (from console)

### Create Steamworks Image
```bash
odin fleet images create \
  --name="MyServer" \
  --version="1.0.0" \
  --os=windows \
  --type=steam \
  --steam-app-id=123456 \
  --branch=public \
  --steamcmd-username=user \
  --steamcmd-password=pass \
  --command=server.exe \
  --force
```

**Steamworks Flags:**
- `--steam-app-id`: Steam App ID
- `--branch`: Steam branch (e.g., beta, stable)
- `--password`: Branch password (if protected)
- `--steamcmd-username`: SteamCMD username
- `--steamcmd-password`: SteamCMD password
- `--command`: Launch command
- `--runtime`: Steam runtime environment
- `--headful`: Enable headful mode
- `--request-license`: Request license (for restricted games)
- `--unpublished`: For unpublished Steam apps

### Get Image Details
```bash
odin fleet images get
odin fleet images get --image-id=123456 --format=json
```

### Delete Image
```bash
odin fleet images delete
odin fleet images delete --image-id=123456 --force
```

## Configuration Management

Server configurations define resources, ports, environment variables, and persistent storage.

### List Configs
```bash
odin fleet configs list
odin fleet configs list --format="table(id,name,binaryId)"
```

### Create Config

Configs are complex JSON objects. Use interactive mode or `--dry-run` to generate the JSON payload.

```bash
# Interactive mode with dry-run to see JSON
odin fleet configs create --dry-run

# Create with JSON payload
odin fleet configs create --payload='{"name":"Production","binaryId":123,...}' --force

# Extract existing config and modify
CONFIG_JSON=$(odin fleet configs get --config-id=123 --format=json)
# Modify JSON with jq or other tools
echo "$CONFIG_JSON" | jq '.name = "New Config"' | \
  odin fleet configs create --payload="$(cat -)" --force
```

### Update Config
```bash
odin fleet configs update --config-id=123456 --payload='{"name":"Updated",...}'
```

### Get Config Details
```bash
odin fleet configs get
odin fleet configs get --config-id=123456 --format=json
```

### Delete Config
```bash
odin fleet configs delete
odin fleet configs delete --config-id=123456 --force
```

## Deployment Management

Deployments define where and how many server instances run.

### List Deployments
```bash
odin fleet deployments list
odin fleet deployments list --format="table(id,name,numInstances,country,city)"
```

### Create Deployment
```bash
# Interactive mode
odin fleet deployments create

# Non-interactive mode
odin fleet deployments create \
  --name="Production EU" \
  --config-id=123456 \
  --num-instances=3 \
  --country=de \
  --city=frankfurt \
  --force

# Get deployment ID for scripts
DEPLOYMENT_ID=$(odin fleet deployments create \
  --name="Production EU" \
  --config-id=123456 \
  --num-instances=3 \
  --country=de \
  --city=frankfurt \
  --force \
  --format="value(id)")
```

**Deployment Flags:**
- `--name`: Deployment name
- `--config-id`: Server configuration ID
- `--num-instances`: Number of server instances
- `--country`: Country code (e.g., de, us)
- `--city`: City name (e.g., frankfurt, newyork)

### Scale Deployment
```bash
odin fleet deployments update \
  --deployment-id=123456 \
  --num-instances=5 \
  --force
```

### Get Deployment Details
```bash
odin fleet deployments get
odin fleet deployments get --deployment-id=123456 --format=json
```

### Delete Deployment
```bash
odin fleet deployments delete
odin fleet deployments delete --deployment-id=123456 --force
```

### Get Available Locations
```bash
odin fleet locations list
odin fleet locations list --format="table(country,city)"
```

## Server Management

### List Servers
```bash
odin fleet servers list
odin fleet servers list --format="table(id,addr,status.state)"
odin fleet servers list --format="table(id,addr,ports['Game Port'].publishedPort)"
```

### Get Server Details
```bash
odin fleet servers get
odin fleet servers get --server-id=123456
odin fleet servers get --server-id=123456 --format=json
```

### View Server Logs
```bash
# Basic logs
odin fleet servers logs --server-id=123456

# Follow logs (live)
odin fleet servers logs --server-id=123456 --follow=true

# Last 100 lines with timestamps
odin fleet servers logs --server-id=123456 --tail=100 --timestamps=true

# Error logs only from last hour
odin fleet servers logs \
  --server-id=123456 \
  --since=60 \
  --stderr=true \
  --stdout=false \
  --timestamps=true
```

**Log Flags:**
- `--server-id`: Server ID
- `--details`: Show detailed logs
- `--follow`: Follow logs in real-time
- `--stdout`: Show stdout (default: true)
- `--stderr`: Show stderr (default: true)
- `--since`: Show logs from last N minutes
- `--timestamps`: Include timestamps
- `--tail`: Number of lines from end (or "all")

### Restart Server
```bash
odin fleet servers restart
odin fleet servers restart --server-id=123456 --force
```

### Backup Management

#### Create Backup
```bash
odin fleet servers backup create --server-id=123456 --name="Before Update"
```

Servers are paused briefly during backup creation.

#### Restore Backup
```bash
odin fleet servers backup restore --server-id=123456
```

#### Download Backup
```bash
odin fleet servers backup download-url --server-id=123456
```

Returns a download link for the backup.

## Output Formatting

The `--format` flag provides powerful output customization for scripts and automation.

### JSON Format
```bash
odin <command> --format=json
```

Returns API-compliant JSON output.

### Table Format
```bash
odin <command> --format="table(property1,property2,nested.property)"

# Examples
odin fleet servers list --format="table(id,addr,status.state)"
odin fleet configs list --format="table(id,name,resources.cpu,resources.memory)"
```

**Nested Properties:**
```bash
odin fleet servers list --format="table(id,serverConfig.name,location.city)"
```

**Array Indexing:**
```bash
odin fleet configs list --format="table(id,name,env[0].key,env[0].value)"
```

**Map Access:**
```bash
odin fleet servers list --format="table(id,ports['Game Port'].publishedPort)"
```

### Value Format
```bash
odin <command> --format="value[separator=':'](property1,property2)"

# Examples
odin fleet images create ... --format="value(id)"  # Returns just the ID
odin fleet servers list --format="value[separator=':'](id,addr)"
```

**Use in Scripts:**
```bash
IMAGE_ID=$(odin fleet images create \
  --name="MyImage" \
  --type=dockerImage \
  --docker-image=my/image:latest \
  --registry-id=1 \
  --force \
  --format="value(id)")

echo "Created image: $IMAGE_ID"
```

### Flattened Format
```bash
odin <command> --format="flattened[noPad,separator='->'](property1,property2)"

# Examples
odin fleet servers get --server-id=123 --format="flattened(id,addr,status.state)"
odin fleet servers get --server-id=123 --format="flattened[separator='->'](id,addr)"
```

**Output:**
```
id->12345
addr->1.2.3.4
```

### Functions

Functions operate on the entire dataset:

```bash
# Count items
odin fleet servers list --format="value(count(id))"

# Count filtered items
odin fleet servers list \
  --filter="status.state='running'" \
  --format="value(count(id))"
```

## Filtering

The `--filter` flag enables powerful server-side and client-side filtering.

### Basic Syntax
```bash
--filter="<field> <operator> <value>"
```

### Operators
- `=`: Equal to
- `!=`: Not equal to
- `>`: Greater than
- `<`: Less than
- `>=`: Greater than or equal to
- `<=`: Less than or equal to
- `~`: Contains (supports wildcards `*`)

### Examples

**Equal:**
```bash
odin fleet servers list --filter="status.state = 'running'"
```

**Contains:**
```bash
odin fleet servers list --filter="serverConfig.name ~ 'Minecraft*'"
```

**Greater than:**
```bash
odin fleet servers list --filter="id > 100"
```

### Logical Operators

**AND:**
```bash
odin fleet servers list --filter="status.state = 'running' AND location.country = 'de'"
```

**OR:**
```bash
odin fleet servers list --filter="location.city = 'frankfurt' OR location.city = 'limburg'"
```

**NOT:**
```bash
odin fleet servers list --filter="NOT (status.state = 'running')"
```

**Complex:**
```bash
odin fleet servers list --filter="(serverConfig.name ~ 'Prod*' AND status.state = 'running') OR id = 123"
```

### Nested Properties
```bash
odin fleet servers list --filter="serverConfig.status = 'ready'"
odin fleet servers list --filter="location.country = 'de' AND location.city = 'frankfurt'"
```

### Map Access
```bash
odin fleet servers list --filter="ports['Game Port'].publishedPort = 30097"
```

## CI/CD Integration

### Complete Deployment Pipeline

**Prerequisites:** Install the ODIN CLI (see Installation section above)

```bash
#!/bin/bash
set -e  # Exit on error

# Install ODIN CLI (if not already installed)
# curl -L -o odin https://github.com/4Players/fleet-cli/releases/latest/download/odin-linux
# chmod +x odin && sudo mv odin /usr/local/bin/

# Configuration
API_KEY="your-api-key"
APP_ID="your-app-id"
DOCKER_IMAGE="my/game-server:$CI_COMMIT_SHA"
REGISTRY_ID=1

# 1. Create image
echo "Creating image..."
IMAGE_ID=$(odin fleet images create \
  --api-key="$API_KEY" \
  --app-id="$APP_ID" \
  --name="GameServer" \
  --version="$CI_COMMIT_SHA" \
  --os=linux \
  --type=dockerImage \
  --docker-image="$DOCKER_IMAGE" \
  --registry-id=$REGISTRY_ID \
  --force \
  --quiet \
  --format="value(id)")

echo "Created image with ID: $IMAGE_ID"

# 2. Create or update config
echo "Creating server configuration..."
CONFIG_PAYLOAD=$(cat <<EOF
{
  "name": "Production Config",
  "binaryId": $IMAGE_ID,
  "restartPolicy": "Always",
  "resources": {
    "cpu": 2,
    "memory": 4096
  },
  "ports": [
    {
      "name": "Game Port",
      "containerPort": 7777,
      "protocol": "UDP"
    }
  ],
  "env": [
    {
      "key": "VERSION",
      "value": "$CI_COMMIT_SHA"
    }
  ]
}
EOF
)

CONFIG_ID=$(odin fleet configs create \
  --api-key="$API_KEY" \
  --app-id="$APP_ID" \
  --payload="$CONFIG_PAYLOAD" \
  --force \
  --quiet \
  --format="value(id)")

echo "Created config with ID: $CONFIG_ID"

# 3. Create deployment
echo "Creating deployment..."
DEPLOYMENT_ID=$(odin fleet deployments create \
  --api-key="$API_KEY" \
  --app-id="$APP_ID" \
  --name="Production EU" \
  --config-id=$CONFIG_ID \
  --num-instances=3 \
  --country=de \
  --city=frankfurt \
  --force \
  --quiet \
  --format="value(id)")

echo "Created deployment with ID: $DEPLOYMENT_ID"

# 4. Wait for servers to be running
echo "Waiting for servers to start..."
while true; do
  RUNNING_COUNT=$(odin fleet servers list \
    --api-key="$API_KEY" \
    --app-id="$APP_ID" \
    --filter="deployment.id = $DEPLOYMENT_ID AND status.state = 'running'" \
    --quiet \
    --format="value(count(id))")

  echo "Running servers: $RUNNING_COUNT / 3"

  if [ "$RUNNING_COUNT" -eq "3" ]; then
    echo "All servers are running!"
    break
  fi

  sleep 10
done

# 5. Get server IPs and ports
echo "Server details:"
odin fleet servers list \
  --api-key="$API_KEY" \
  --app-id="$APP_ID" \
  --filter="deployment.id = $DEPLOYMENT_ID" \
  --format="table(id,addr,ports['Game Port'].publishedPort,status.state)"

echo "Deployment complete!"
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

      - name: Install ODIN CLI
        run: |
          curl -L -o odin https://github.com/4Players/fleet-cli/releases/latest/download/odin-linux
          chmod +x odin
          sudo mv odin /usr/local/bin/

      - name: Deploy to Fleet
        env:
          ODIN_API_KEY: ${{ secrets.ODIN_API_KEY }}
          ODIN_APP_ID: ${{ secrets.ODIN_APP_ID }}
        run: |
          IMAGE_ID=$(odin fleet images create \
            --api-key="$ODIN_API_KEY" \
            --app-id="$ODIN_APP_ID" \
            --name="MyServer" \
            --version="${{ github.sha }}" \
            --os=linux \
            --type=dockerImage \
            --docker-image="ghcr.io/${{ github.repository }}:${{ github.sha }}" \
            --registry-id=1 \
            --force \
            --quiet \
            --format="value(id)")

          echo "Created image: $IMAGE_ID"
```

### GitLab CI Example

```yaml
deploy:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
    - curl -L -o odin https://github.com/4Players/fleet-cli/releases/latest/download/odin-linux
    - chmod +x odin
    - mv odin /usr/local/bin/
  script:
    - |
      IMAGE_ID=$(odin fleet images create \
        --api-key="$ODIN_API_KEY" \
        --app-id="$ODIN_APP_ID" \
        --name="MyServer" \
        --version="$CI_COMMIT_SHA" \
        --os=linux \
        --type=dockerImage \
        --docker-image="$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA" \
        --registry-id=1 \
        --force \
        --quiet \
        --format="value(id)")
      echo "Deployed image: $IMAGE_ID"
  only:
    - main
```

## Best Practices

### Error Handling
```bash
#!/bin/bash

# Exit on error
set -e

# Capture errors
if ! IMAGE_ID=$(odin fleet images create ... 2>&1); then
  echo "Failed to create image: $IMAGE_ID" >&2
  exit 1
fi
```

### Using --quiet for Scripts
```bash
# Suppress informational output, only show errors
odin fleet images create ... --quiet
```

### Using --force for Automation
```bash
# Skip confirmation prompts in CI/CD
odin fleet deployments delete --deployment-id=123 --force
```

### Validation with --dry-run
```bash
# Preview JSON payload before creating
odin fleet configs create --dry-run

# Save payload for reuse
odin fleet configs create --dry-run > config-template.json
```

### Combining Filters and Formatting
```bash
# Count running servers in Frankfurt
odin fleet servers list \
  --filter="status.state = 'running' AND location.city = 'frankfurt'" \
  --format="value(count(id))"

# Get IPs of all running servers
odin fleet servers list \
  --filter="status.state = 'running'" \
  --format="value(addr)" \
  | while read -r ip; do
      echo "Server IP: $ip"
    done
```

### Environment Variables
```bash
# Store credentials in environment
export ODIN_API_KEY="your-api-key"
export ODIN_APP_ID="your-app-id"

# Use in commands
odin fleet servers list --api-key="$ODIN_API_KEY" --app-id="$ODIN_APP_ID"
```

### Scripting with jq
```bash
# Get all servers as JSON
SERVERS_JSON=$(odin fleet servers list --format=json)

# Parse with jq
echo "$SERVERS_JSON" | jq '.[] | select(.status.state == "running") | .id'

# Modify config and update
odin fleet configs get --config-id=123 --format=json | \
  jq '.resources.cpu = 4' | \
  odin fleet configs update --config-id=123 --payload="$(cat -)" --force
```

## Advanced Use Cases

### Blue-Green Deployment
```bash
#!/bin/bash
set -e

# Create new deployment (green)
GREEN_DEPLOYMENT=$(odin fleet deployments create \
  --name="Production Green" \
  --config-id=$NEW_CONFIG_ID \
  --num-instances=3 \
  --country=de \
  --city=frankfurt \
  --force \
  --format="value(id)")

# Wait for green to be healthy
while [ $(odin fleet servers list \
  --filter="deployment.id = $GREEN_DEPLOYMENT AND status.state = 'running'" \
  --format="value(count(id))") -lt 3 ]; do
  sleep 10
done

# Delete old deployment (blue)
odin fleet deployments delete --deployment-id=$OLD_DEPLOYMENT_ID --force

echo "Blue-green deployment complete!"
```

### Health Monitoring Script
```bash
#!/bin/bash

# Monitor server health
while true; do
  TOTAL=$(odin fleet servers list --format="value(count(id))")
  RUNNING=$(odin fleet servers list \
    --filter="status.state = 'running'" \
    --format="value(count(id))")

  echo "$(date): $RUNNING / $TOTAL servers running"

  if [ "$RUNNING" -lt "$TOTAL" ]; then
    echo "WARNING: Some servers are not running!"
    odin fleet servers list \
      --filter="status.state != 'running'" \
      --format="table(id,status.state,serverConfig.name)"
  fi

  sleep 60
done
```

### Automated Backup Rotation
```bash
#!/bin/bash

# Get all running servers
SERVER_IDS=$(odin fleet servers list \
  --filter="status.state = 'running'" \
  --format="value(id)")

# Backup each server
for SERVER_ID in $SERVER_IDS; do
  echo "Backing up server $SERVER_ID..."
  odin fleet servers backup create \
    --server-id=$SERVER_ID \
    --name="Daily Backup $(date +%Y-%m-%d)" \
    --force
done

echo "All backups complete!"
```

## Command Reference

### Apps
```bash
odin apps list                    # List all apps
odin apps select                  # Select app interactively
odin apps get                     # Get app details
odin apps create                  # Create new app
odin apps delete                  # Delete app
```

### Images
```bash
odin fleet images list            # List all images
odin fleet images create          # Create new image
odin fleet images get             # Get image details
odin fleet images delete          # Delete image
```

### Configs
```bash
odin fleet configs list           # List all configs
odin fleet configs create         # Create new config
odin fleet configs get            # Get config details
odin fleet configs update         # Update config
odin fleet configs delete         # Delete config
```

### Deployments
```bash
odin fleet deployments list       # List all deployments
odin fleet deployments create     # Create new deployment
odin fleet deployments get        # Get deployment details
odin fleet deployments update     # Update deployment (scale)
odin fleet deployments delete     # Delete deployment
```

### Servers
```bash
odin fleet servers list           # List all servers
odin fleet servers get            # Get server details
odin fleet servers logs           # View server logs
odin fleet servers restart        # Restart server
odin fleet servers backup create  # Create backup
odin fleet servers backup restore # Restore backup
odin fleet servers backup download-url  # Get backup download link
```

### Locations
```bash
odin fleet locations list         # List available locations
```

## Resources

- **CLI Source Code**: [https://github.com/4Players/fleet-cli](https://github.com/4Players/fleet-cli)
- **API Documentation**: [https://fleet.4players.io/docs/api](https://fleet.4players.io/docs/api)
- **Console**: [https://console.4players.io](https://console.4players.io)
- **Main Documentation**: [https://docs.4players.io/fleet/](https://docs.4players.io/fleet/)

## Help and Support

```bash
# General help
odin --help

# Command-specific help
odin fleet images create --help
odin fleet servers logs --help
```

For issues or questions, visit the [ODIN Discord](https://4np.de/discord) or contact support@4players.io.
