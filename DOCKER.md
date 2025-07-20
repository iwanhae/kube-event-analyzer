# Docker Usage Guide

## Building the Docker Image

To build the Docker image locally:

```bash
docker build -t kube-event-analyzer:latest .
```

## Running the Container

### Basic Usage

```bash
docker run -d \
  --name kube-event-analyzer \
  -p 8080:8080 \
  -v $(pwd)/data:/app/data \
  kube-event-analyzer:latest
```

### With Kubernetes Configuration

If you need to connect to a Kubernetes cluster, mount your kubeconfig:

```bash
docker run -d \
  --name kube-event-analyzer \
  -p 8080:8080 \
  -v $(pwd)/data:/app/data \
  -v ~/.kube/config:/home/appuser/.kube/config:ro \
  -e KUBECONFIG=/home/appuser/.kube/config \
  kube-event-analyzer:latest
```

### Environment Variables

- `KUBECONFIG`: Path to Kubernetes configuration file (optional)
- `PORT`: API server port (defaults to 8080)

## Health Check

The container includes a built-in health check that monitors the `/health` endpoint:

```bash
curl http://localhost:8080/health
```

Expected response:
```json
{
  "status": "healthy",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

## Using Pre-built Images

Pre-built images are available from GitHub Container Registry:

```bash
docker pull ghcr.io/iwanhae/kube-event-analyzer:latest
```

### Available Tags

- `latest`: Latest stable version from the main branch
- `main`: Latest build from the main branch
- `v1.0.0`: Specific version tags
- `develop`: Latest build from the develop branch

## Multi-Architecture Support

The Docker images support multiple architectures:
- `linux/amd64` (x86_64)
- `linux/arm64` (ARM64/AArch64)

Docker will automatically pull the appropriate image for your platform.

## Data Persistence

The application stores data in `/app/data/events.db`. To persist data across container restarts:

```bash
# Create a named volume
docker volume create kube-events-data

# Run with the volume
docker run -d \
  --name kube-event-analyzer \
  -p 8080:8080 \
  -v kube-events-data:/app/data \
  kube-event-analyzer:latest
```

## Docker Compose

Example `docker-compose.yml`:

```yaml
version: '3.8'

services:
  kube-event-analyzer:
    image: ghcr.io/iwanhae/kube-event-analyzer:latest
    container_name: kube-event-analyzer
    ports:
      - "8080:8080"
    volumes:
      - kube-events-data:/app/data
      - ~/.kube/config:/home/appuser/.kube/config:ro
    environment:
      - KUBECONFIG=/home/appuser/.kube/config
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 5s

volumes:
  kube-events-data:
```

Run with:
```bash
docker-compose up -d
```

## Troubleshooting

### Check Container Logs

```bash
docker logs kube-event-analyzer
```

### Debug Container

```bash
docker exec -it kube-event-analyzer /bin/sh
```

### Health Check Status

```bash
docker inspect kube-event-analyzer --format='{{.State.Health.Status}}'
```