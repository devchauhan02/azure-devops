# Docker Cheat Sheet (Comprehensive)

Quick reference for Docker images, containers, volumes, networks, and system management.

---

## Image Commands

| Command | Description |
|---------|-------------|
| `docker pull nginx` | Pull image from registry |
| `docker pull node:18-alpine` | Pull specific tag |
| `docker image ls` | List all images |
| `docker images` | List all images (short) |
| `docker image rm nginx` | Remove image |
| `docker rmi <image_id>` | Remove image by ID |
| `docker build -t myapp:1.0 .` | Build image from Dockerfile |
| `docker tag myapp:1.0 user/myapp:1.0` | Tag image |
| `docker push user/myapp:1.0` | Push image to registry |
| `docker history <image>` | Show image layers |
| `docker image prune` | Remove unused images |
| `docker image prune -a` | Remove all unused images |

---

## Container Commands

### Run & Start

| Command | Description |
|---------|-------------|
| `docker run nginx` | Run container (foreground) |
| `docker run -d nginx` | Run in detached mode |
| `docker run -d -p 8080:80 nginx` | Run with port mapping |
| `docker run -d -p 8080:80 --name web nginx` | Run with name |
| `docker run -it ubuntu bash` | Run interactive |
| `docker run --rm nginx` | Auto-remove after exit |
| `docker run -e VAR=value nginx` | Set environment variable |
| `docker run -v mydata:/app nginx` | Mount volume |
| `docker run --network mynet nginx` | Connect to network |
| `docker start web` | Start stopped container |
| `docker restart web` | Restart container |

### List & Inspect

| Command | Description |
|---------|-------------|
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker ps -q` | List only container IDs |
| `docker inspect web` | Detailed container info |
| `docker stats` | Live resource usage |
| `docker top web` | Running processes |
| `docker port web` | Port mappings |

### Stop & Remove

| Command | Description |
|---------|-------------|
| `docker stop web` | Stop container |
| `docker stop $(docker ps -q)` | Stop all running |
| `docker kill web` | Force stop |
| `docker rm web` | Remove container |
| `docker rm -f web` | Force remove running |
| `docker container prune` | Remove all stopped |

---

## Logs & Execution

| Command | Description |
|---------|-------------|
| `docker logs web` | View logs |
| `docker logs -f web` | Follow logs |
| `docker logs --tail 100 web` | Last 100 lines |
| `docker logs --since 1h web` | Logs from last hour |
| `docker exec -it web bash` | Interactive shell |
| `docker exec web ls /app` | Run single command |
| `docker attach web` | Attach to running container |
| `docker cp web:/app/file.txt .` | Copy file from container |
| `docker cp file.txt web:/app/` | Copy file to container |

---

## Port Mapping

| Command | Description |
|---------|-------------|
| `docker run -p 8080:80 nginx` | Map host 8080 to container 80 |
| `docker run -p 3000:3000 node` | Map same port |
| `docker run -p 127.0.0.1:8080:80 nginx` | Bind to localhost only |
| `docker port web` | Show port mappings |

**Format:** `-p <host_port>:<container_port>`

---

## Volume Commands

| Command | Description |
|---------|-------------|
| `docker volume create mydata` | Create volume |
| `docker volume ls` | List volumes |
| `docker volume inspect mydata` | Volume details |
| `docker volume rm mydata` | Remove volume |
| `docker volume prune` | Remove unused volumes |
| `docker run -v mydata:/app nginx` | Mount named volume |
| `docker run -v $(pwd):/app nginx` | Bind mount (host path) |
| `docker run -v /app/node_modules nginx` | Anonymous volume |
| `docker run --mount source=mydata,target=/app nginx` | Mount (verbose) |

**Volume Types:**
- **Named Volume:** `mydata:/app`
- **Bind Mount:** `$(pwd):/app`
- **Anonymous Volume:** `/app/data`

---

## Network Commands

| Command | Description |
|---------|-------------|
| `docker network ls` | List networks |
| `docker network create mynet` | Create network |
| `docker network create --driver bridge mynet` | Create bridge network |
| `docker network inspect mynet` | Network details |
| `docker network connect mynet web` | Connect container to network |
| `docker network disconnect mynet web` | Disconnect from network |
| `docker network rm mynet` | Remove network |
| `docker network prune` | Remove unused networks |
| `docker run --network mynet nginx` | Run in specific network |

**Default Networks:**
- `bridge` - default
- `host` - use host networking
- `none` - no networking

---

## Docker System & Cleanup

| Command | Description |
|---------|-------------|
| `docker system df` | Disk usage |
| `docker system info` | System information |
| `docker system prune` | Remove unused data |
| `docker system prune -a` | Remove all unused (including images) |
| `docker system prune --volumes` | Include volumes |
| `docker container prune` | Remove stopped containers |
| `docker image prune` | Remove dangling images |
| `docker image prune -a` | Remove unused images |
| `docker volume prune` | Remove unused volumes |
| `docker network prune` | Remove unused networks |

---

## Monitoring & Stats

| Command | Description |
|---------|-------------|
| `docker stats` | Live container stats |
| `docker stats web` | Stats for specific container |
| `docker stats --no-stream` | Single snapshot |
| `docker top web` | Running processes |
| `docker events` | Real-time events |
| `docker inspect web` | Full container JSON |

---

## Dockerfile Instructions

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image | `FROM node:18-alpine` |
| `WORKDIR` | Set working directory | `WORKDIR /app` |
| `COPY` | Copy files | `COPY . .` |
| `ADD` | Copy + extract archives | `ADD app.tar.gz /app` |
| `RUN` | Execute build command | `RUN npm install` |
| `EXPOSE` | Document port | `EXPOSE 3000` |
| `CMD` | Default command | `CMD ["npm","start"]` |
| `ENTRYPOINT` | Fixed executable | `ENTRYPOINT ["node"]` |
| `ENV` | Environment variable | `ENV NODE_ENV=production` |
| `ARG` | Build argument | `ARG VERSION=1.0` |
| `VOLUME` | Define volume | `VOLUME /data` |
| `USER` | Set user | `USER node` |
| `LABEL` | Add metadata | `LABEL version="1.0"` |
| `HEALTHCHECK` | Health check | `HEALTHCHECK CMD curl localhost` |

---

## Example Dockerfiles

### Node.js App

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
USER node
CMD ["npm", "start"]
```

### Multi-Stage Build

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
CMD ["node", "dist/server.js"]
```

### Python App

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

---

## Docker Compose Commands

| Command | Description |
|---------|-------------|
| `docker compose up` | Start services |
| `docker compose up -d` | Start in background |
| `docker compose up --build` | Rebuild and start |
| `docker compose down` | Stop and remove |
| `docker compose down -v` | Also remove volumes |
| `docker compose start` | Start existing services |
| `docker compose stop` | Stop services |
| `docker compose restart` | Restart services |
| `docker compose ps` | List services |
| `docker compose logs` | View logs |
| `docker compose logs -f` | Follow logs |
| `docker compose logs web` | Logs for specific service |
| `docker compose exec web bash` | Execute in service |
| `docker compose build` | Build images |
| `docker compose pull` | Pull images |
| `docker compose config` | Validate and view config |

---

## Example docker-compose.yml

### Basic Web + Database

```yaml
version: "3.9"

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - db
    networks:
      - app-network

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

### Node.js + MongoDB + Redis

```yaml
version: "3.9"

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongo:27017/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongo
      - redis

  mongo:
    image: mongo:6
    volumes:
      - mongo-data:/data/db

  redis:
    image: redis:7-alpine

volumes:
  mongo-data:
```

---

## Save & Load Images

| Command | Description |
|---------|-------------|
| `docker save myapp:1.0 > myapp.tar` | Save to tar file |
| `docker save myapp:1.0 | gzip > myapp.tar.gz` | Save compressed |
| `docker load < myapp.tar` | Load from tar |
| `docker load < myapp.tar.gz` | Load compressed |
| `docker export web > web.tar` | Export container |
| `docker import web.tar myapp:latest` | Import container |

---

## Resource Limits

| Command | Description |
|---------|-------------|
| `docker run -m 512m nginx` | Limit memory to 512MB |
| `docker run --memory-swap 1g nginx` | Set swap limit |
| `docker run --cpus 2 nginx` | Limit to 2 CPUs |
| `docker run --cpus 0.5 nginx` | Limit to 50% of 1 CPU |
| `docker run --cpu-shares 512 nginx` | Relative CPU weight |
| `docker update --cpus 2 web` | Update running container |

---

## Restart Policies

| Command | Description |
|---------|-------------|
| `docker run --restart no nginx` | Never restart |
| `docker run --restart always nginx` | Always restart |
| `docker run --restart on-failure nginx` | Restart on error |
| `docker run --restart unless-stopped nginx` | Restart unless manually stopped |
| `docker run --restart on-failure:3 nginx` | Max 3 restart attempts |
| `docker update --restart always web` | Update policy |

---

## Environment Variables

| Command | Description |
|---------|-------------|
| `docker run -e VAR=value nginx` | Single variable |
| `docker run -e VAR1=val1 -e VAR2=val2 nginx` | Multiple variables |
| `docker run --env-file .env nginx` | From file |

**.env file example:**
```
NODE_ENV=production
PORT=3000
DATABASE_URL=postgres://localhost/db
```

---

## Health Checks

### In Dockerfile

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl --fail http://localhost:3000 || exit 1
```

### In Docker Run

```bash
docker run --health-cmd "curl --fail http://localhost || exit 1" \
  --health-interval=30s \
  --health-timeout=3s \
  --health-retries=3 \
  nginx
```

### Check Health Status

```bash
docker ps  # Shows health in STATUS column
docker inspect --format='{{.State.Health.Status}}' web
```

---

## Common DevOps Workflows

### Debug Container

```bash
# Enter shell
docker exec -it web sh

# View logs
docker logs -f web

# Check processes
docker top web

# Inspect config
docker inspect web
```

### Rebuild & Restart

```bash
# Docker Compose
docker compose down
docker compose up -d --build

# Individual container
docker stop web
docker rm web
docker build -t myapp .
docker run -d -p 8080:80 --name web myapp
```

### Find Issues

```bash
# Check all container logs
docker logs $(docker ps -aq)

# See resource usage
docker stats

# Check disk usage
docker system df

# See events
docker events --since 1h
```

### Cleanup Everything

```bash
# Nuclear option - remove everything
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
docker rmi $(docker images -q)
docker volume rm $(docker volume ls -q)
docker network rm $(docker network ls -q)

# Or use prune
docker system prune -a --volumes
```

---

## Registry & Hub

| Command | Description |
|---------|-------------|
| `docker login` | Login to Docker Hub |
| `docker login registry.example.com` | Login to private registry |
| `docker logout` | Logout |
| `docker search nginx` | Search Docker Hub |
| `docker pull nginx` | Pull from Docker Hub |
| `docker pull registry.example.com/app` | Pull from private registry |
| `docker tag app user/app:1.0` | Tag for push |
| `docker push user/app:1.0` | Push to registry |

---

## Best Practices Summary

✅ **Images**
- Use official base images
- Use specific tags, avoid `latest`
- Use minimal images (`alpine`, `slim`)
- Use multi-stage builds
- Add `.dockerignore`

✅ **Containers**
- One process per container
- Run as non-root user
- Use health checks
- Set resource limits
- Use restart policies

✅ **Volumes**
- Use volumes for persistent data
- Use named volumes over bind mounts
- Never store data in containers

✅ **Networks**
- Use custom networks for isolation
- Don't expose unnecessary ports
- Use internal networks when possible

✅ **Security**
- Don't store secrets in images
- Scan images for vulnerabilities
- Keep images updated
- Use read-only file systems when possible

✅ **Performance**
- Layer caching - copy dependencies first
- Combine RUN commands
- Clean up in same layer
- Use .dockerignore

---

## Quick Reference Card

```
# Life Cycle
docker run          # Create & start
docker start        # Start existing
docker stop         # Stop gracefully
docker restart      # Restart
docker rm           # Remove

# Info
docker ps           # Running
docker ps -a        # All
docker logs         # View logs
docker inspect      # Full details
docker stats        # Resources

# Execute
docker exec -it     # Interactive
docker attach       # Attach to running

# Images
docker build        # Build
docker pull         # Download
docker push         # Upload
docker images       # List
docker rmi          # Remove

# Cleanup
docker system prune # Clean unused
docker volume prune # Clean volumes
docker network prune # Clean networks
```

---

## Troubleshooting Quick Tips

**Container won't start:**
```bash
docker logs <container>
docker inspect <container>
```

**Can't connect to container:**
```bash
docker port <container>
docker network inspect bridge
```

**High disk usage:**
```bash
docker system df
docker system prune -a
```

**Permission errors:**
```bash
docker run --user $(id -u):$(id -g) ...
```

**Container keeps restarting:**
```bash
docker logs <container>
docker update --restart=no <container>
```