# Advanced Patterns

Best practices and advanced use cases for ODIN Fleet CLI.

## Table of Contents

- [Error Handling](#error-handling)
- [Blue-Green Deployment](#blue-green-deployment)
- [Health Monitoring Script](#health-monitoring-script)
- [Automated Backup Rotation](#automated-backup-rotation)
- [Scripting with jq](#scripting-with-jq)

---

## Error Handling

```bash
#!/bin/bash
set -e  # Exit on error

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

---

## Blue-Green Deployment

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

---

## Health Monitoring Script

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

---

## Automated Backup Rotation

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

---

## Scripting with jq

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
