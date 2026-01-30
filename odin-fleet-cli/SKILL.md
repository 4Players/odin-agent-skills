---
name: odin-fleet-cli
description: |
  ODIN Fleet CLI tool for managing game servers, creating resources, and building CI/CD pipelines.
  Use when: automating fleet deployments, scripting server management, creating images/configs/deployments,
  filtering and formatting CLI output, building CI/CD pipelines, managing backups, viewing logs,
  or working with Fleet API via command line. Covers all CLI commands, output formatting, filters,
  and automation best practices.
license: MIT
---

# ODIN Fleet CLI

Command-line tool for managing ODIN Fleet resources, automating deployments, and building CI/CD pipelines.

## Installation

### Download Pre-built Binary (Recommended)

**Linux:**
```bash
curl -L -o odin https://github.com/4Players/fleet-cli/releases/latest/download/odin-linux
chmod +x odin && sudo mv odin /usr/local/bin/
```

**macOS (Apple Silicon):**
```bash
curl -L -o odin https://github.com/4Players/fleet-cli/releases/latest/download/odin-macos-arm64
chmod +x odin && sudo mv odin /usr/local/bin/
```

**macOS (Intel):**
```bash
curl -L -o odin https://github.com/4Players/fleet-cli/releases/latest/download/odin-macos-x64
chmod +x odin && sudo mv odin /usr/local/bin/
```

**Windows:** Download `odin-windows.exe` from [GitHub Releases](https://github.com/4Players/fleet-cli/releases/latest)

## Authentication

```bash
odin login                          # Interactive login
odin login --api-key=YOUR_API_KEY   # Direct token login (CI/CD)
odin apps list                      # List all apps
odin apps select                    # Select app interactively
```

Get your API key from the [4Players Console](https://console.4players.io/settings/api-keys).

## Global Flags

```bash
--api-key=<string>   # Override stored API key
--app-id=<string>    # Override selected app
--format=<string>    # Output: json, value, flattened, table
--force              # Skip confirmation prompts
--quiet              # Suppress informational output
```

## Image Management

```bash
odin fleet images list
odin fleet images get --image-id=123456
odin fleet images delete --image-id=123456 --force

# Create Docker image
odin fleet images create \
  --name="MyServer" --version="1.0.0" --os=linux \
  --type=dockerImage --docker-image=myimage:latest \
  --registry-id=1 --force

# Create Steamworks image
odin fleet images create \
  --name="MyServer" --version="1.0.0" --os=windows \
  --type=steam --steam-app-id=123456 --branch=public \
  --steamcmd-username=user --steamcmd-password=pass \
  --command=server.exe --force

# Get only the ID (for scripts)
IMAGE_ID=$(odin fleet images create ... --format="value(id)")
```

## Configuration Management

```bash
odin fleet configs list
odin fleet configs get --config-id=123456
odin fleet configs delete --config-id=123456 --force

# Create with JSON payload (use --dry-run to preview)
odin fleet configs create --payload='{"name":"Config","binaryId":123,...}' --force
odin fleet configs update --config-id=123 --payload='{"name":"Updated"}'
```

## Deployment Management

```bash
odin fleet deployments list
odin fleet deployments get --deployment-id=123456
odin fleet deployments delete --deployment-id=123456 --force

# Create deployment
odin fleet deployments create \
  --name="Production EU" --config-id=123456 \
  --num-instances=3 --country=de --city=frankfurt --force

# Scale deployment
odin fleet deployments update --deployment-id=123456 --num-instances=5 --force

# List available locations
odin fleet locations list
```

## Server Management

```bash
odin fleet servers list
odin fleet servers get --server-id=123456
odin fleet servers start --server-id=123456 --force
odin fleet servers stop --server-id=123456 --force
odin fleet servers restart --server-id=123456 --force
odin fleet servers start-all --force
odin fleet servers stop-all --force

# View logs
odin fleet servers logs --server-id=123456
odin fleet servers logs --server-id=123456 --follow=true --tail=100

# Backups
odin fleet servers backup create --server-id=123456 --name="Before Update"
odin fleet servers backup restore --server-id=123456
odin fleet servers backup download-url --server-id=123456
```

## Output Formatting

### JSON Format
```bash
odin fleet servers list --format=json
```

### Table Format
```bash
odin fleet servers list --format="table(id,addr,status.state)"
odin fleet configs list --format="table(id,name,resources.cpu)"
odin fleet servers list --format="table(id,ports['Game Port'].publishedPort)"
```

### Value Format (for scripts)
```bash
IMAGE_ID=$(odin fleet images create ... --format="value(id)")
odin fleet servers list --format="value[separator=':'](id,addr)"
odin fleet servers list --format="value(count(id))"
```

### Flattened Format
```bash
odin fleet servers get --server-id=123 --format="flattened(id,addr,status.state)"
```

## Filtering

```bash
# Operators: = != > < >= <= ~ (contains with wildcards)
odin fleet servers list --filter="status.state = 'running'"
odin fleet servers list --filter="serverConfig.name ~ 'Minecraft*'"
odin fleet servers list --filter="id > 100"

# Logical operators: AND, OR, NOT
odin fleet servers list --filter="status.state = 'running' AND location.country = 'de'"
odin fleet servers list --filter="location.city = 'frankfurt' OR location.city = 'limburg'"
odin fleet servers list --filter="NOT (status.state = 'running')"

# Nested properties and map access
odin fleet servers list --filter="location.country = 'de' AND location.city = 'frankfurt'"
odin fleet servers list --filter="ports['Game Port'].publishedPort = 30097"
```

## Command Reference

### Apps
```bash
odin apps list | select | get | create | delete
```

### Fleet Resources
```bash
odin fleet images list | create | get | update | delete
odin fleet configs list | create | get | update | delete
odin fleet deployments list | create | get | update | delete
odin fleet servers list | get | start | stop | restart | start-all | stop-all | logs
odin fleet servers backup create | restore | download-url
odin fleet locations list
```

## Additional Documentation

- **CI/CD Integration**: See [references/cicd-integration.md](references/cicd-integration.md) for GitHub Actions, GitLab CI examples
- **Advanced Patterns**: See [references/advanced-patterns.md](references/advanced-patterns.md) for blue-green deployments, monitoring scripts

## Resources

- **CLI Source**: https://github.com/4Players/fleet-cli
- **API Docs**: https://fleet.4players.io/docs/api
- **Console**: https://console.4players.io
- **Help**: `odin --help` or `odin fleet images create --help`
