---
sidebar_position: 5
title: Docker
description: Run Hermes Agent in Docker for persistent, self-updating deployments
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Docker Deployment

Run Hermes Agent as a Docker container for persistent, always-on deployments. The official image comes pre-configured with all dependencies and supports automatic updates.

## Quick Start

```sh
docker run -it --rm \
  -v ~/.hermes:/root/.hermes \
  -e ANTHROPIC_API_KEY=your-key \
  nousresearch/hermes-agent:latest
```

This starts an interactive CLI session. For persistent deployments, see [Background Deployment](#background-deployment) below.

## Image Details

| Property | Value |
|----------|-------|
| **Image** | `nousresearch/hermes-agent:latest` |
| **Base** | Python 3.12 slim |
| **Size** | ~500MB |
| **Platforms** | `linux/amd64`, `linux/arm64` |
| **Registry** | [Docker Hub](https://hub.docker.com/r/nousresearch/hermes-agent) |

## Configuration

### Environment Variables

Pass your API keys and configuration via environment variables:

```sh
docker run -it --rm \
  -v ~/.hermes:/root/.hermes \
  -e ANTHROPIC_API_KEY=your-key \
  -e OPENAI_API_KEY=your-key \
  -e OPENROUTER_API_KEY=your-key \
  nousresearch/hermes-agent:latest
```

Or use an env file:

```sh
# Create .env file
cat > hermes.env << 'EOF'
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
OPENROUTER_API_KEY=sk-or-...
EOF

docker run -it --rm \
  -v ~/.hermes:/root/.hermes \
  --env-file hermes.env \
  nousresearch/hermes-agent:latest
```

### Volume Mounts

Mount `~/.hermes` to persist configuration, memory, skills, and sessions across container restarts:

```sh
-v ~/.hermes:/root/.hermes
```

| Host Path | Container Path | Purpose |
|-----------|---------------|---------|
| `~/.hermes/config.yaml` | `/root/.hermes/config.yaml` | Agent configuration |
| `~/.hermes/.env` | `/root/.hermes/.env` | API keys (alternative to env vars) |
| `~/.hermes/memory/` | `/root/.hermes/memory/` | Persistent memory |
| `~/.hermes/skills/` | `/root/.hermes/skills/` | Custom skills |
| `~/.hermes/sessions.db` | `/root/.hermes/sessions.db` | Session history |

### Custom Config

If you have an existing `config.yaml`, it will be picked up automatically from the mounted volume:

```sh
# Edit config on host
vim ~/.hermes/config.yaml

# Container uses it automatically
docker run -it --rm \
  -v ~/.hermes:/root/.hermes \
  nousresearch/hermes-agent:latest
```

## Background Deployment

### Running as a Gateway

For messaging platform integrations (Discord, Telegram, Slack), run the gateway in the background:

```sh
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/root/.hermes \
  --env-file hermes.env \
  nousresearch/hermes-agent:latest \
  gateway
```

### Docker Compose

For production deployments, use Docker Compose:

```yaml
# docker-compose.yml
version: '3.8'

services:
  hermes:
    image: nousresearch/hermes-agent:latest
    container_name: hermes
    restart: unless-stopped
    volumes:
      - ~/.hermes:/root/.hermes
    env_file:
      - hermes.env
    command: gateway
    # Optional: resource limits
    deploy:
      resources:
        limits:
          memory: 2G
```

```sh
docker compose up -d
```

## Management

### Common operations

```sh
docker logs --tail 50 hermes          # Recent logs
docker logs -f hermes                 # Follow logs
docker restart hermes                 # Restart
docker stop hermes                    # Stop
docker rm hermes                      # Remove container
```

### Executing commands

```sh
docker exec -it hermes hermes status  # Check agent status
docker exec -it hermes bash           # Shell into container
```

### Updating

```sh
docker pull nousresearch/hermes-agent:latest
docker stop hermes && docker rm hermes
# Re-run your docker run command or docker compose up -d
```

### Checking container health

```sh
docker logs --tail 50 hermes          # Recent logs
docker run -it --rm nousresearch/hermes-agent:latest version     # Verify version
docker stats hermes                    # Resource usage
```
