# CI/CD Integration

Complete examples for automating ODIN Fleet deployments in CI/CD pipelines.

## Table of Contents

- [Complete Deployment Pipeline](#complete-deployment-pipeline)
- [GitHub Actions Example](#github-actions-example)
- [GitLab CI Example](#gitlab-ci-example)

---

## Complete Deployment Pipeline

**Prerequisites:** Install the ODIN CLI (see main SKILL.md)

```bash
#!/bin/bash
set -e  # Exit on error

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

---

## GitHub Actions Example

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

---

## GitLab CI Example

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
