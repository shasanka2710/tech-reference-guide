# 🐳 Docker Quick Reference

## Core Concepts

| Concept     | Description |
|-------------|-------------|
| Image       | Read-only template for creating containers |
| Container   | Running instance of an image |
| Dockerfile  | Script to build a Docker image |
| Registry    | Repository for Docker images (e.g., Docker Hub) |
| Volume      | Persistent data storage for containers |
| Network     | Communication layer between containers |

## Common Commands

```bash
# Image management
docker pull nginx:latest
docker build -t myapp:1.0 .
docker build -t myapp:1.0 -f Dockerfile.prod .
docker images
docker rmi myapp:1.0
docker tag myapp:1.0 myregistry/myapp:1.0
docker push myregistry/myapp:1.0

# Container lifecycle
docker run nginx
docker run -d --name web -p 8080:80 nginx          # detached, named, port mapped
docker run -it ubuntu:22.04 bash                    # interactive
docker run --rm alpine echo "hello"                 # auto-remove after exit
docker run -e ENV_VAR=value myapp                   # env variable
docker run -v /host/path:/container/path myapp      # volume mount

docker start   mycontainer
docker stop    mycontainer
docker restart mycontainer
docker rm      mycontainer
docker rm -f   mycontainer        # force remove running container

# Inspect & Debug
docker ps                         # running containers
docker ps -a                      # all containers
docker logs mycontainer
docker logs -f mycontainer        # follow logs
docker inspect mycontainer
docker stats                      # live resource usage
docker top mycontainer            # processes inside container
docker exec -it mycontainer bash  # shell inside running container

# Cleanup
docker system prune               # remove all stopped containers, dangling images
docker system prune -a            # also remove unused images
docker volume prune
docker network prune
```

## Dockerfile

```dockerfile
# Start from base image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy dependency files first (layer caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application source
COPY . .

# Build step
RUN npm run build

# Set environment variable
ENV NODE_ENV=production
ENV PORT=3000

# Expose port
EXPOSE 3000

# Create non-root user (security best practice)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD wget -qO- http://localhost:3000/health || exit 1

# Start command
CMD ["node", "dist/server.js"]
```

### Multi-stage Build

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production image (smaller)
FROM node:20-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

## Docker Compose

```yaml
# docker-compose.yml
version: "3.9"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_URL=mongodb://mongo:27017/mydb
    depends_on:
      mongo:
        condition: service_healthy
    volumes:
      - ./logs:/app/logs
    networks:
      - app-network
    restart: unless-stopped

  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: secret
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - app-network

volumes:
  mongo_data:

networks:
  app-network:
    driver: bridge
```

```bash
# Docker Compose commands
docker compose up -d              # start in background
docker compose up --build         # rebuild images before starting
docker compose down               # stop and remove containers
docker compose down -v            # also remove volumes
docker compose logs -f app        # follow logs for a service
docker compose exec app bash      # shell into service
docker compose ps                 # status
docker compose pull               # pull latest images
```

## Volumes

```bash
# Named volumes
docker volume create mydata
docker volume ls
docker volume inspect mydata
docker volume rm mydata

# Mount types
# Named volume
docker run -v mydata:/data nginx

# Bind mount
docker run -v /host/path:/container/path nginx

# Read-only mount
docker run -v /host/config:/app/config:ro nginx
```

## Networking

```bash
# Network management
docker network create mynet
docker network create --driver bridge mynet
docker network ls
docker network inspect mynet
docker network rm mynet

# Connect containers
docker network connect    mynet mycontainer
docker network disconnect mynet mycontainer

# Run on specific network
docker run --network mynet nginx
```

## Best Practices

- Use specific image tags, not `latest` in production
- Use multi-stage builds to reduce final image size
- Add `.dockerignore` to exclude unnecessary files
- Run containers as non-root users
- Use `COPY` instead of `ADD` unless you need URL/tar auto-extraction
- Order Dockerfile instructions to maximize layer cache hits (static deps first)
- Set resource limits with `--memory` and `--cpus`
- Use health checks for production containers
