# Docker & Kubernetes: Beginner to Intermediate Guide

> A structured, practical reference for learning containerization and container orchestration.

---

# SECTION 1: DOCKER

---

## 1. Core Concepts

---

### 1.1 Container vs Virtual Machine

**Definition**

A **Virtual Machine (VM)** emulates an entire physical computer, including hardware, OS kernel, and applications. A **Container** is a lightweight, isolated process that shares the host OS kernel but maintains its own filesystem, network, and process space.

**Explanation / Working**

| Attribute         | Virtual Machine                   | Container                        |
|-------------------|-----------------------------------|----------------------------------|
| OS Kernel         | Separate kernel per VM            | Shared host kernel               |
| Boot Time         | Minutes                           | Milliseconds                     |
| Size              | Gigabytes                         | Megabytes                        |
| Isolation         | Full hardware-level isolation     | Process-level isolation          |
| Use Case          | Full OS environments              | Microservices, app packaging     |
| Overhead          | High (hypervisor + full OS)       | Low (namespaces + cgroups)       |

**Why It Matters**

Containers allow consistent, reproducible environments across development, staging, and production. They eliminate the "works on my machine" problem and drastically reduce resource usage compared to VMs.

**Common Mistakes / Notes**

- Containers are not VMs. They do not boot a full OS.
- Containers share the host kernel. A Linux container cannot run natively on a Windows kernel without a compatibility layer (WSL2 on Windows).
- Security isolation in containers is weaker than in VMs. For strict isolation requirements, VMs remain preferable.

---

### 1.2 Images vs Containers

**Definition**

- **Image**: A read-only, immutable template that contains the application code, runtime, libraries, and configuration needed to run a container.
- **Container**: A running (or stopped) instance of an image. It is the live, writable execution of an image.

**Explanation / Working**

Think of an image as a class definition and a container as an object instantiated from that class. Multiple containers can be created from the same image. Each container gets its own writable layer on top of the shared image layers.

```
Image (read-only layers)
  └── Container 1 (writable layer)
  └── Container 2 (writable layer)
  └── Container 3 (writable layer)
```

**Why It Matters**

Images are the unit of distribution. You build an image once, push it to a registry, and run it anywhere Docker is installed. This guarantees environmental consistency.

**Common Mistakes / Notes**

- Data written inside a container's writable layer is lost when the container is removed. Use volumes for persistence.
- Do not confuse `docker stop` (pause) with `docker rm` (remove). A stopped container still exists and retains its writable layer.

---

### 1.3 Layers & Caching

**Definition**

Docker images are built in **layers**. Each instruction in a Dockerfile creates a new layer. Layers are stacked and each layer only stores the diff (changes) from the previous layer.

**Explanation / Working**

```
Layer 5: COPY ./app /app          ← your application code
Layer 4: RUN pip install -r ...   ← dependencies installed
Layer 3: COPY requirements.txt .  ← requirements file
Layer 2: RUN apt-get update       ← OS packages
Layer 1: FROM python:3.11-slim    ← base image
```

Docker caches each layer. If a layer and all layers before it are unchanged, Docker reuses the cached version, significantly speeding up builds.

**Why It Matters**

Efficient layering reduces build times, reduces image sizes, and improves pull/push performance from registries.

**Practical Example**

Incorrect order (cache busted frequently):

```dockerfile
COPY . .                    # Any code change invalidates all below
RUN pip install -r requirements.txt
```

Correct order (dependencies cached separately):

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt   # Cached unless requirements.txt changes
COPY . .                              # Only app code layer changes
```

**Common Mistakes / Notes**

- Always copy dependency manifests (`package.json`, `requirements.txt`, `pom.xml`) before copying source code.
- Any change to a layer invalidates all subsequent layer caches.
- Use `.dockerignore` to prevent unnecessary files from being included in the build context, which also triggers cache invalidation.

---

### 1.4 Immutable Infrastructure

**Definition**

**Immutable infrastructure** means that once a container is deployed, it is never modified in place. Instead, a new image is built with the required changes and a new container is deployed to replace the old one.

**Why It Matters**

- Eliminates configuration drift between environments.
- Every deployment is predictable and repeatable.
- Rollbacks are straightforward: re-deploy the previous image version.

**Common Mistakes / Notes**

- Never SSH into a running container to make changes. Any change must go through a new image build.
- Configuration that changes between environments (dev/staging/prod) should be injected via environment variables or config files at runtime, not baked into the image.

---

## 2. Docker CLI

**Definition**

The Docker CLI (`docker`) is the primary command-line interface for interacting with the Docker daemon to build, run, manage, and inspect containers and images.

---

### `docker build`

**Explanation / Working**

Builds an image from a Dockerfile. The build context (files sent to the daemon) is the specified path.

**Syntax**

```bash
docker build [OPTIONS] PATH
```

**Practical Examples**

```bash
# Build an image from the current directory with a tag
docker build -t myapp:1.0 .

# Build using a specific Dockerfile
docker build -t myapp:1.0 -f Dockerfile.prod .

# Build without using cache
docker build --no-cache -t myapp:latest .
```

**Common Mistakes / Notes**

- The `.` at the end specifies the build context, not the Dockerfile location.
- Use `-f` to point to a differently named Dockerfile.

---

### `docker run`

**Explanation / Working**

Creates a new container from an image and starts it.

**Syntax**

```bash
docker run [OPTIONS] IMAGE [COMMAND]
```

**Practical Examples**

```bash
# Run a container interactively with a terminal
docker run -it ubuntu:22.04 bash

# Run in detached (background) mode
docker run -d nginx:latest

# Map host port 8080 to container port 80
docker run -d -p 8080:80 nginx:latest

# Pass environment variables
docker run -d -e DB_HOST=localhost -e DB_PORT=5432 myapp:1.0

# Mount a volume
docker run -d -v mydata:/var/lib/mysql mysql:8

# Assign a name
docker run -d --name web-server nginx:latest

# Auto-remove container after it exits
docker run --rm ubuntu:22.04 echo "hello"
```

**Common Mistakes / Notes**

- Without `-d`, the container runs in the foreground.
- Without `--name`, Docker assigns a random name.
- Port mapping format is `HOST_PORT:CONTAINER_PORT`.

---

### `docker ps`

**Explanation / Working**

Lists running containers.

**Syntax**

```bash
docker ps [OPTIONS]
```

**Practical Examples**

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Show only container IDs
docker ps -q

# Show all containers with IDs only
docker ps -aq
```

---

### `docker stop` and `docker rm`

**Explanation / Working**

- `docker stop`: Sends a SIGTERM signal to the main process inside the container. Waits 10 seconds (default) before sending SIGKILL.
- `docker rm`: Removes a stopped container. Does not remove the image.

**Syntax**

```bash
docker stop [OPTIONS] CONTAINER [CONTAINER...]
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

**Practical Examples**

```bash
# Stop a named container
docker stop web-server

# Stop all running containers
docker stop $(docker ps -q)

# Remove a stopped container
docker rm web-server

# Force remove a running container (SIGKILL)
docker rm -f web-server

# Remove all stopped containers
docker container prune
```

---

### `docker images`

**Explanation / Working**

Lists images stored locally on the Docker host.

**Practical Examples**

```bash
# List all local images
docker images

# List only image IDs
docker images -q

# Remove an image
docker rmi myapp:1.0

# Remove all unused images
docker image prune -a
```

---

### `docker logs`

**Explanation / Working**

Fetches the stdout and stderr output of a container.

**Syntax**

```bash
docker logs [OPTIONS] CONTAINER
```

**Practical Examples**

```bash
# View logs of a container
docker logs web-server

# Follow log output in real time
docker logs -f web-server

# Show last 100 lines
docker logs --tail 100 web-server

# Show logs with timestamps
docker logs -t web-server
```

---

### `docker exec`

**Explanation / Working**

Runs a command inside a running container. Used for debugging and inspection.

**Syntax**

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

**Practical Examples**

```bash
# Open an interactive bash shell inside a running container
docker exec -it web-server bash

# Run a single command inside the container
docker exec web-server cat /etc/nginx/nginx.conf

# Run as a specific user
docker exec -u root -it web-server bash
```

**Common Mistakes / Notes**

- The container must be running. `docker exec` does not start stopped containers.
- Changes made inside a container via `exec` are lost when the container is removed.

---

### Docker CLI Cheat Sheet

| Command                            | Description                              |
|------------------------------------|------------------------------------------|
| `docker build -t name:tag .`       | Build image from current directory       |
| `docker run -d -p 8080:80 image`   | Run container in background, map port    |
| `docker ps`                        | List running containers                  |
| `docker ps -a`                     | List all containers                      |
| `docker stop <name>`               | Gracefully stop a container              |
| `docker rm <name>`                 | Remove a stopped container               |
| `docker rm -f <name>`              | Force remove a running container         |
| `docker images`                    | List local images                        |
| `docker rmi <image>`               | Remove an image                          |
| `docker logs -f <name>`            | Stream container logs                    |
| `docker exec -it <name> bash`      | Open shell inside running container      |
| `docker inspect <name>`            | Full JSON details of container/image     |

---

## 3. Dockerfile

**Definition**

A **Dockerfile** is a text file containing a series of instructions that Docker reads to build an image. Each instruction creates a new layer in the image.

---

### Instructions Reference

#### `FROM`

**Definition**: Specifies the base image to build upon. Every Dockerfile must begin with a `FROM` instruction.

```dockerfile
FROM ubuntu:22.04
FROM python:3.11-slim
FROM node:20-alpine
```

**Notes**: Prefer minimal base images (e.g., `-slim`, `-alpine`) to reduce image size and attack surface.

---

#### `RUN`

**Definition**: Executes a command during the image build process. The result is committed as a new layer.

```dockerfile
RUN apt-get update && apt-get install -y curl vim && rm -rf /var/lib/apt/lists/*
```

**Notes**:
- Chain multiple commands with `&&` to reduce layer count.
- Always clean up package manager caches in the same `RUN` instruction.

---

#### `COPY` vs `ADD`

**Definition**:
- `COPY`: Copies files/directories from the build context into the image.
- `ADD`: Similar to `COPY` but also supports remote URLs and auto-extracts tar archives.

```dockerfile
COPY ./src /app/src
COPY requirements.txt /app/

ADD https://example.com/file.tar.gz /tmp/   # ADD with remote URL
ADD archive.tar.gz /app/                    # ADD with auto-extract
```

**Notes**: Prefer `COPY` over `ADD` unless you specifically need URL fetching or auto-extraction. `COPY` is explicit and predictable.

---

#### `CMD` vs `ENTRYPOINT`

| Feature           | `CMD`                                  | `ENTRYPOINT`                                    |
|-------------------|----------------------------------------|-------------------------------------------------|
| Purpose           | Default command/arguments              | Fixed executable that always runs               |
| Override          | Easily overridden via `docker run`     | Cannot be overridden without `--entrypoint`     |
| Use Case          | Default behavior, flexible             | Container behaves like a specific executable    |

```dockerfile
# CMD — can be overridden
CMD ["python", "app.py"]

# ENTRYPOINT — always executed
ENTRYPOINT ["python", "app.py"]

# Combined — ENTRYPOINT sets executable, CMD sets default arguments
ENTRYPOINT ["python"]
CMD ["app.py"]
```

**Notes**:
- Always use the JSON array (exec) form: `["executable", "arg1"]` instead of the shell form: `executable arg1`. The shell form spawns a shell process (`/bin/sh -c`) which does not receive Unix signals correctly.

---

#### `ENV`

**Definition**: Sets environment variables inside the container. Accessible during build and at runtime.

```dockerfile
ENV APP_ENV=production
ENV PORT=8080
```

---

#### `WORKDIR`

**Definition**: Sets the working directory for subsequent instructions (`RUN`, `COPY`, `CMD`, etc.).

```dockerfile
WORKDIR /app
```

**Notes**: Use `WORKDIR` instead of `RUN cd /app`. If the directory does not exist, Docker creates it.

---

#### `EXPOSE`

**Definition**: Documents which network port the container listens on at runtime. It is informational only — it does not actually publish the port.

```dockerfile
EXPOSE 8080
```

**Notes**: Actual port publishing requires `-p` flag in `docker run`.

---

### Complete Dockerfile Example

```dockerfile
# Stage 1: Builder
FROM python:3.11-slim AS builder

WORKDIR /app

# Copy dependency manifest first (layer caching)
COPY requirements.txt .

# Install dependencies into a separate directory
RUN pip install --prefix=/install --no-cache-dir -r requirements.txt

# Stage 2: Final image
FROM python:3.11-slim

WORKDIR /app

# Copy installed packages from builder
COPY --from=builder /install /usr/local

# Copy application source
COPY ./src .

# Set environment variables
ENV APP_ENV=production
ENV PORT=8080

# Document the port
EXPOSE 8080

# Create non-root user for security
RUN adduser --disabled-password --gecos '' appuser
USER appuser

# Run the application
CMD ["python", "main.py"]
```

---

### Multi-Stage Builds

**Definition**: A build technique using multiple `FROM` statements in a single Dockerfile to produce a smaller final image by discarding build-time dependencies.

**Why It Matters**

Without multi-stage builds, your production image contains compilers, build tools, and intermediate files that are not needed at runtime, significantly increasing image size.

```dockerfile
# Stage 1: Build the Go binary
FROM golang:1.21 AS builder
WORKDIR /src
COPY . .
RUN go build -o /app/server .

# Stage 2: Minimal runtime image
FROM alpine:3.18
COPY --from=builder /app/server /app/server
CMD ["/app/server"]
```

The final image is based on `alpine:3.18` (~7 MB) instead of `golang:1.21` (~800 MB).

---

### Dockerfile Cheat Sheet

| Instruction           | Purpose                                      |
|-----------------------|----------------------------------------------|
| `FROM image:tag`      | Set base image                               |
| `RUN command`         | Execute command at build time                |
| `COPY src dest`       | Copy files into the image                    |
| `ADD src dest`        | Copy with URL/tar support                    |
| `CMD ["exec", "arg"]` | Default command (overridable)                |
| `ENTRYPOINT ["exec"]` | Fixed entrypoint executable                  |
| `ENV KEY=VALUE`       | Set environment variable                     |
| `WORKDIR /path`       | Set working directory                        |
| `EXPOSE port`         | Document container port                      |
| `USER username`       | Set runtime user                             |
| `ARG NAME=default`    | Build-time variable                          |

---

## 4. Volumes & Persistence

**Definition**

By default, all data written inside a container is stored in its writable layer, which is ephemeral — it is lost when the container is removed. **Volumes** are the Docker-managed mechanism for persisting data outside the container lifecycle.

---

### The Data Persistence Problem

When you run:

```bash
docker rm my-database
```

All data stored inside the container is permanently deleted. For stateful applications like databases, this is unacceptable. Volumes solve this by storing data on the host filesystem, managed by Docker.

---

### Named Volumes

**Definition**: Volumes created and managed by Docker, stored in Docker's storage directory (typically `/var/lib/docker/volumes/`).

**Practical Examples**

```bash
# Create a named volume
docker volume create pgdata

# Use a named volume when running a container
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15

# List volumes
docker volume ls

# Inspect a volume
docker volume inspect pgdata

# Remove a volume
docker volume rm pgdata

# Remove all unused volumes
docker volume prune
```

---

### Bind Mounts

**Definition**: Maps a specific directory or file on the **host machine** into the container. The host path must exist.

```bash
# Mount current directory into container
docker run -d \
  -v /home/user/myapp:/app \
  -p 8080:8080 \
  myapp:dev

# Read-only bind mount
docker run -d \
  -v /etc/config:/app/config:ro \
  myapp:1.0
```

**Named Volumes vs Bind Mounts**

| Feature            | Named Volume                          | Bind Mount                              |
|--------------------|---------------------------------------|-----------------------------------------|
| Managed by         | Docker                                | Host filesystem                         |
| Path               | Docker-managed (`/var/lib/docker/`)   | Explicit host path                      |
| Portability        | High (works on any Docker host)       | Low (depends on host directory)         |
| Use Case           | Database data, app state              | Development (live code reload)          |
| Performance        | Slightly better on Docker Desktop     | Direct host I/O                         |

**Common Mistakes / Notes**

- Use named volumes in production for database storage.
- Use bind mounts in development to reflect live code changes without rebuilding.
- Do not store secrets in volumes without encryption. Use Docker Secrets or Kubernetes Secrets instead.

---

## 5. Docker Networking

**Definition**

Docker networking controls how containers communicate with each other and with the outside world. Docker provides several built-in network drivers.

---

### Bridge Network (Default)

**Definition**: The default network driver. Containers on the same bridge network can communicate using IP addresses. Containers on the default bridge cannot resolve each other by name.

```bash
# Inspect the default bridge network
docker network inspect bridge

# Run two containers on the default bridge
docker run -d --name app1 nginx
docker run -d --name app2 nginx

# app1 and app2 cannot resolve each other by name on the default bridge
```

---

### Custom Networks

**Definition**: User-defined bridge networks that provide automatic DNS-based service discovery — containers can reach each other by container name.

```bash
# Create a custom network
docker network create my-network

# Connect containers to the custom network
docker run -d --name db --network my-network postgres:15
docker run -d --name app --network my-network myapp:1.0

# Inside the app container, you can reach the database by name:
# psql -h db -U postgres
```

---

### Container Communication

**Why It Matters**

In a microservices architecture, containers need to communicate. Custom networks provide hostname-based resolution without exposing ports to the host.

**Practical Example**

```bash
# Create an isolated network for an application stack
docker network create app-stack

# Database - only accessible inside the network, not on the host
docker run -d \
  --name postgres \
  --network app-stack \
  -e POSTGRES_DB=mydb \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Application - connect to DB by hostname "postgres"
docker run -d \
  --name api \
  --network app-stack \
  -p 8080:8080 \
  -e DATABASE_URL=postgres://postgres:secret@postgres:5432/mydb \
  myapi:1.0
```

**Common Mistakes / Notes**

- The default bridge network does not support DNS resolution by container name. Always use custom networks.
- Avoid publishing database ports to the host (`-p 5432:5432`) in production. Keep databases on internal networks only.

---

### Docker Networking Cheat Sheet

| Command                                    | Description                              |
|--------------------------------------------|------------------------------------------|
| `docker network ls`                        | List all networks                        |
| `docker network create <name>`             | Create a custom bridge network           |
| `docker network inspect <name>`            | Inspect network details                  |
| `docker network connect <net> <container>` | Connect a running container to a network |
| `docker network rm <name>`                 | Remove a network                         |

---

## 6. Docker Compose

**Definition**

Docker Compose is a tool for defining and running multi-container Docker applications. All services are defined in a single `docker-compose.yml` YAML file. Compose manages the lifecycle of all defined services together.

---

### docker-compose.yml Structure

```yaml
version: "3.9"

services:
  <service-name>:
    image: <image> | build: <path>
    ports:
      - "HOST:CONTAINER"
    environment:
      - KEY=VALUE
    volumes:
      - <volume>:<path>
    depends_on:
      - <other-service>
    networks:
      - <network>

volumes:
  <volume-name>:

networks:
  <network-name>:
```

---

### Multi-Container Example: App + Database + Cache

```yaml
version: "3.9"

services:

  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: api
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgres://appuser:secret@db:5432/appdb
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
    networks:
      - backend
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    container_name: postgres
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - backend
    restart: unless-stopped

  cache:
    image: redis:7-alpine
    container_name: redis
    volumes:
      - redisdata:/data
    networks:
      - backend
    restart: unless-stopped

volumes:
  pgdata:
  redisdata:

networks:
  backend:
    driver: bridge
```

---

### Docker Compose Commands

```bash
# Start all services in detached mode
docker compose up -d

# Stop and remove all services
docker compose down

# Stop and remove services, including volumes
docker compose down -v

# View logs of all services
docker compose logs -f

# View logs of a specific service
docker compose logs -f api

# List running services
docker compose ps

# Rebuild images and restart
docker compose up -d --build

# Scale a service to 3 instances
docker compose up -d --scale api=3

# Execute a command in a running service container
docker compose exec api bash
```

**Common Mistakes / Notes**

- `depends_on` only waits for the container to start, not for the application inside to be ready (e.g., database accepting connections). Use health checks or retry logic in your application.
- Always define named volumes for stateful services. Removing compose stacks with `-v` will delete all associated volumes.
- Use `.env` files for sensitive environment variables instead of hardcoding them in the YAML.

---

## 7. Image Management

---

### Docker Registries

**Definition**: A **registry** is a storage and distribution system for Docker images. Docker Hub is the default public registry. Private registries (AWS ECR, GCR, GitHub Container Registry, Harbor) are used for proprietary images.

```bash
# Log in to Docker Hub
docker login

# Log in to a private registry
docker login registry.example.com
```

---

### Tagging Strategy

**Definition**: Image tags identify specific versions of an image. A tag is appended to the image name with a colon: `image:tag`.

**Best Practice Tagging**

```bash
# Tag with version, git commit, and latest
docker build -t myorg/myapp:1.2.0 .
docker build -t myorg/myapp:1.2.0-abc1234 .
docker build -t myorg/myapp:latest .
```

**Tag Strategy Guidelines**

| Tag Pattern                | Use Case                          |
|----------------------------|-----------------------------------|
| `latest`                   | Most recent build (use cautiously)|
| `1.2.0`                    | Semantic version                  |
| `1.2.0-abc1234`            | Version + git commit hash         |
| `stable`, `production`     | Environment-specific tags         |

---

### Push / Pull Images

```bash
# Tag a locally built image for a registry
docker tag myapp:1.0 myorg/myapp:1.0

# Push to Docker Hub
docker push myorg/myapp:1.0

# Pull an image from Docker Hub
docker pull myorg/myapp:1.0

# Pull from a private registry
docker pull registry.example.com/myorg/myapp:1.0
```

**Common Mistakes / Notes**

- Never tag production images as `latest` only. Always include a versioned tag. `latest` is mutable and provides no traceability.
- Avoid pushing images with embedded secrets. Use environment variables or secret managers.

---

## 8. Debugging Containers

---

### `docker logs`

**When to Use**: When a container fails to start, crashes, or behaves unexpectedly. Provides stdout/stderr output.

```bash
docker logs --tail 200 -f my-container
```

---

### `docker exec`

**When to Use**: To inspect the container's filesystem, running processes, environment variables, or test network connectivity from inside the container.

```bash
# Open interactive shell
docker exec -it my-container sh

# Check environment variables
docker exec my-container env

# Test internal DNS resolution
docker exec my-container nslookup db

# Check open ports inside container
docker exec my-container ss -tlnp
```

---

### `docker inspect`

**Definition**: Returns detailed JSON metadata about a container or image, including network settings, volume mounts, environment variables, and more.

```bash
# Inspect a container
docker inspect my-container

# Extract specific fields using Go template
docker inspect --format='{{.NetworkSettings.IPAddress}}' my-container

# Extract environment variables
docker inspect --format='{{.Config.Env}}' my-container
```

---

### Debugging Cheat Sheet

| Command                              | When to Use                              |
|--------------------------------------|------------------------------------------|
| `docker logs -f <name>`              | Container not behaving, check output     |
| `docker exec -it <name> sh`          | Inspect filesystem, environment, network |
| `docker inspect <name>`              | Full metadata, IP, volumes, config       |
| `docker stats`                       | Real-time CPU/memory/network usage       |
| `docker events`                      | Stream of Docker daemon events           |
| `docker diff <name>`                 | Files changed inside container           |

---

---

# SECTION 2: KUBERNETES

---

## 1. Core Architecture

**Definition**

**Kubernetes (K8s)** is an open-source container orchestration platform that automates the deployment, scaling, scheduling, and management of containerized applications.

---

### Cluster

**Definition**: A Kubernetes **cluster** is the complete Kubernetes deployment. It consists of a set of machines (nodes) that run containerized workloads and the control plane components that manage them.

```
Kubernetes Cluster
├── Control Plane
│   ├── API Server
│   ├── Scheduler
│   ├── Controller Manager
│   └── etcd
└── Worker Nodes
    ├── Node 1 (kubelet, kube-proxy, container runtime)
    ├── Node 2
    └── Node 3
```

---

### Nodes

**Definition**: A **node** is a physical or virtual machine in the cluster that runs containerized workloads. Each node is managed by the control plane.

**Node Components**:

| Component           | Role                                                              |
|---------------------|-------------------------------------------------------------------|
| `kubelet`           | Agent that ensures containers described in PodSpecs are running   |
| `kube-proxy`        | Maintains network rules and enables communication to/from Pods   |
| Container Runtime   | Runs containers (containerd, CRI-O)                              |

---

### Control Plane

**Definition**: The **control plane** is the set of components that manage the overall state of the cluster. It makes global decisions about scheduling, responds to events, and exposes the Kubernetes API.

| Component              | Role                                                                 |
|------------------------|----------------------------------------------------------------------|
| `kube-apiserver`       | Front-end for the Kubernetes control plane. All operations go here. |
| `kube-scheduler`       | Watches for unscheduled Pods and assigns them to nodes.             |
| `kube-controller-manager` | Runs controller loops (node controller, replication controller). |
| `cloud-controller-manager` | Interfaces with underlying cloud provider APIs.              |

---

### etcd

**Definition**: **etcd** is a distributed, consistent key-value store used as Kubernetes' backing store for all cluster state data. All cluster state (Pod definitions, ConfigMaps, Secrets, etc.) is persisted in etcd.

**Why It Matters**

- If etcd data is lost, the cluster loses all state.
- In production clusters, etcd must be backed up regularly.
- etcd requires an odd number of members (3, 5, 7) for quorum.

**Common Mistakes / Notes**

- Never run production workloads on control plane nodes.
- Always back up etcd before cluster upgrades.
- etcd performance directly impacts API server responsiveness.

---

## 2. Pods

**Definition**

A **Pod** is the smallest deployable unit in Kubernetes. A Pod represents one or more containers that share the same network namespace (same IP address and port space) and the same storage volumes.

---

### Pod Concept

**Explanation / Working**

- Each Pod gets a unique IP address within the cluster.
- Containers within the same Pod communicate via `localhost`.
- Pods are ephemeral — they are created, scheduled, and terminated. They are not rescheduled to a new node if they fail; the controller (Deployment/ReplicaSet) creates a new Pod.

```yaml
# Basic Pod definition
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.25
      ports:
        - containerPort: 80
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"
```

**Why It Matters**

Pods are the unit of scheduling. Kubernetes decides which node a Pod runs on based on resource requests, node capacity, and scheduling policies.

**Common Mistakes / Notes**

- Do not create bare Pods in production. Use Deployments or StatefulSets to manage Pods. Bare Pods are not restarted if the node fails.
- The multi-container pattern (sidecar) is useful for logging agents, proxies (like Envoy in service meshes), and secret injection.

---

## 3. Workloads

---

### Deployments

**Definition**: A **Deployment** is the recommended way to run stateless applications. It manages a ReplicaSet, ensures the desired number of Pod replicas are running, and handles rolling updates and rollbacks.

**Explanation / Working**

```
Deployment
  └── ReplicaSet
        ├── Pod 1
        ├── Pod 2
        └── Pod 3
```

When you update the image in a Deployment, Kubernetes performs a rolling update: gradually replacing old Pods with new ones, ensuring zero downtime.

**Example Deployment YAML**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  labels:
    app: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: myorg/myapi:1.2.0
          ports:
            - containerPort: 8080
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: url
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 20
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
```

---

### ReplicaSets

**Definition**: A **ReplicaSet** ensures that a specified number of identical Pod replicas are running at any given time. It is rarely used directly — Deployments manage ReplicaSets automatically.

**Why It Matters**

If a Pod crashes or is deleted, the ReplicaSet controller creates a replacement to maintain the desired count.

---

### StatefulSets (Basic)

**Definition**: A **StatefulSet** is a workload controller for stateful applications that require stable network identities, stable persistent storage, and ordered deployment and scaling.

**When to Use StatefulSets**

- Databases (PostgreSQL, MySQL, MongoDB)
- Distributed systems requiring peer discovery (Kafka, ZooKeeper, Elasticsearch)

**Key Differences from Deployments**

| Feature              | Deployment                    | StatefulSet                         |
|----------------------|-------------------------------|-------------------------------------|
| Pod Identity         | Random names (`pod-abc123`)   | Stable names (`pod-0`, `pod-1`)     |
| Storage              | Shared or none                | Dedicated PVC per Pod               |
| Scaling Order        | Parallel                      | Sequential (pod-0 before pod-1)     |
| Use Case             | Stateless apps                | Databases, stateful distributed apps|

**Common Mistakes / Notes**

- Do not use Deployments for databases in Kubernetes. Use StatefulSets.
- StatefulSets require a Headless Service for pod identity.
- Deleting a StatefulSet does not delete associated PersistentVolumeClaims. This is intentional to prevent data loss.

---

## 4. Services

**Definition**

A **Service** is a Kubernetes abstraction that provides a stable, permanent IP address and DNS name to a set of Pods. Since Pods are ephemeral and their IPs change, Services provide a consistent endpoint.

Services use **label selectors** to identify which Pods they route traffic to.

---

### ClusterIP (Default)

**Definition**: Exposes the Service on an internal IP address within the cluster. Only accessible from within the cluster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  type: ClusterIP
  selector:
    app: api
  ports:
    - protocol: TCP
      port: 80        # Port the Service listens on
      targetPort: 8080 # Port on the Pod
```

**Use Case**: Internal communication between microservices.

---

### NodePort

**Definition**: Exposes the Service on each node's IP at a static port (range: 30000–32767). Accessible from outside the cluster using `<NodeIP>:<NodePort>`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-nodeport
spec:
  type: NodePort
  selector:
    app: api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30080  # Optional: if omitted, auto-assigned
```

**Use Case**: Development and testing. Not recommended for production.

---

### LoadBalancer

**Definition**: Provisions an external load balancer from the cloud provider (AWS ELB, GCP GLB, Azure LB) and exposes the Service externally.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

**Use Case**: Production external-facing services on cloud-managed Kubernetes (EKS, GKE, AKS).

---

### Service Type Comparison

| Type           | Accessible From     | Use Case                             |
|----------------|---------------------|--------------------------------------|
| `ClusterIP`    | Inside cluster only | Internal service-to-service traffic  |
| `NodePort`     | External via node   | Dev/test, direct node access         |
| `LoadBalancer` | External via LB     | Production, cloud-managed clusters   |
| `ExternalName` | DNS alias           | Access external services by name     |

---

## 5. kubectl

**Definition**

`kubectl` is the official command-line tool for interacting with the Kubernetes API server. It allows you to create, inspect, update, and delete Kubernetes resources.

---

### `kubectl get`

```bash
# List all pods in the current namespace
kubectl get pods

# List pods with more details (IP, Node)
kubectl get pods -o wide

# List pods in a specific namespace
kubectl get pods -n kube-system

# List all resources of multiple types
kubectl get pods,services,deployments

# Watch resource changes in real time
kubectl get pods -w

# List all pods across all namespaces
kubectl get pods --all-namespaces
```

---

### `kubectl describe`

```bash
# Describe a pod (events, status, resource limits)
kubectl describe pod my-pod

# Describe a deployment
kubectl describe deployment api-deployment

# Describe a node
kubectl describe node worker-1
```

**When to Use**: Use `describe` as the first step in debugging. It shows Events, which reveal why a Pod is failing.

---

### `kubectl logs`

```bash
# View logs from a pod
kubectl logs my-pod

# Follow logs in real time
kubectl logs -f my-pod

# View logs from a specific container in a multi-container pod
kubectl logs my-pod -c sidecar-container

# View previous container's logs (if it crashed)
kubectl logs my-pod --previous

# Tail last 100 lines
kubectl logs my-pod --tail=100
```

---

### `kubectl apply -f`

```bash
# Apply a resource from a YAML file
kubectl apply -f deployment.yaml

# Apply all YAML files in a directory
kubectl apply -f ./manifests/

# Apply from a remote URL
kubectl apply -f https://example.com/manifest.yaml
```

**Notes**: `kubectl apply` is declarative — it creates the resource if it doesn't exist or updates it if it does. Prefer `apply` over `create` for GitOps workflows.

---

### `kubectl delete`

```bash
# Delete a pod
kubectl delete pod my-pod

# Delete using a manifest file
kubectl delete -f deployment.yaml

# Delete a deployment (also deletes associated ReplicaSet and Pods)
kubectl delete deployment api-deployment

# Delete all pods in a namespace
kubectl delete pods --all -n staging
```

---

### Additional Useful kubectl Commands

```bash
# Scale a deployment
kubectl scale deployment api-deployment --replicas=5

# Update a deployment's image (rolling update)
kubectl set image deployment/api-deployment api=myorg/myapi:1.3.0

# View rollout history
kubectl rollout history deployment/api-deployment

# Undo the last rollout
kubectl rollout undo deployment/api-deployment

# Check rollout status
kubectl rollout status deployment/api-deployment

# Open shell in a running pod
kubectl exec -it my-pod -- bash

# Port-forward from local machine to a pod
kubectl port-forward pod/my-pod 8080:8080

# Port-forward to a service
kubectl port-forward service/api-service 8080:80

# Copy files to/from a pod
kubectl cp my-pod:/app/logs/app.log ./app.log

# Get cluster info
kubectl cluster-info

# View kubeconfig contexts
kubectl config get-contexts

# Switch context
kubectl config use-context production-cluster
```

---

### kubectl Cheat Sheet

| Command                                        | Description                              |
|------------------------------------------------|------------------------------------------|
| `kubectl get pods`                             | List pods in current namespace           |
| `kubectl get pods -o wide`                     | List pods with node and IP info          |
| `kubectl describe pod <name>`                  | Full details and events for a pod        |
| `kubectl logs -f <pod>`                        | Stream pod logs                          |
| `kubectl apply -f <file>`                      | Apply/update resource from YAML          |
| `kubectl delete -f <file>`                     | Delete resource from YAML                |
| `kubectl exec -it <pod> -- bash`               | Open shell in pod                        |
| `kubectl scale deployment <name> --replicas=N` | Scale deployment                         |
| `kubectl rollout undo deployment/<name>`       | Rollback deployment                      |
| `kubectl port-forward <pod> 8080:8080`         | Forward local port to pod                |
| `kubectl get events --sort-by=.lastTimestamp`  | View recent cluster events               |

---

## 6. YAML Configuration

**Definition**

Kubernetes resources are defined as YAML manifests. Each manifest describes the desired state of a resource. The API server accepts these manifests and works to maintain the defined state.

---

### Required Fields

Every Kubernetes resource YAML must include:

| Field        | Description                                      | Example                  |
|--------------|--------------------------------------------------|--------------------------|
| `apiVersion` | API group and version for the resource           | `apps/v1`, `v1`          |
| `kind`       | Type of Kubernetes resource                      | `Pod`, `Deployment`, `Service` |
| `metadata`   | Identifying information                          | `name`, `namespace`, `labels` |
| `spec`       | Desired state / configuration of the resource   | Varies per resource type |

---

### Complete Annotated YAML Example

```yaml
# API version and resource type
apiVersion: apps/v1
kind: Deployment

# Resource metadata
metadata:
  name: frontend               # Unique name in the namespace
  namespace: production        # Target namespace (defaults to "default")
  labels:
    app: frontend
    version: "2.1.0"
  annotations:
    deployment.kubernetes.io/revision: "1"

# Desired state specification
spec:
  replicas: 3                  # Number of Pod replicas

  # Selector must match pod template labels
  selector:
    matchLabels:
      app: frontend

  # Pod template
  template:
    metadata:
      labels:
        app: frontend          # Must match selector.matchLabels

    spec:
      # Container definitions
      containers:
        - name: frontend
          image: myorg/frontend:2.1.0
          imagePullPolicy: IfNotPresent

          ports:
            - containerPort: 3000

          # Environment variables
          env:
            - name: API_URL
              valueFrom:
                configMapKeyRef:
                  name: frontend-config
                  key: api_url

          # Resource requests and limits
          resources:
            requests:
              memory: "64Mi"
              cpu: "100m"
            limits:
              memory: "128Mi"
              cpu: "200m"

          # Liveness probe: restart container if unhealthy
          livenessProbe:
            httpGet:
              path: /healthz
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 15

          # Readiness probe: stop sending traffic if not ready
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10
```

**Common Mistakes / Notes**

- `selector.matchLabels` must exactly match `template.metadata.labels`. Mismatches cause the Deployment to fail.
- Always define `resources.requests` and `resources.limits`. Without requests, the scheduler cannot make informed placement decisions. Without limits, a container can consume unlimited node resources.
- `imagePullPolicy: Always` forces pulling the image on every Pod start. Use `IfNotPresent` in most cases.
- YAML is indentation-sensitive. Always use spaces, never tabs.

---

## 7. Scaling & Self-Healing

---

### Auto-Restart (Self-Healing)

**Definition**: Kubernetes automatically restarts containers that fail, replaces Pods that are terminated, and reschedules Pods from failed nodes.

**Explanation / Working**

The `kubelet` monitors Pod health via probes and restarts containers as needed:

- **Liveness Probe**: Determines if a container is alive. If it fails, Kubernetes restarts the container.
- **Readiness Probe**: Determines if a container is ready to receive traffic. If it fails, Kubernetes removes the Pod from Service endpoints.
- **Startup Probe**: Allows slow-starting containers time to initialize before liveness checks begin.

```yaml
containers:
  - name: app
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 30
      failureThreshold: 3
      periodSeconds: 10

    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      failureThreshold: 3
      periodSeconds: 5

    startupProbe:
      httpGet:
        path: /healthz
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
```

---

### Horizontal Scaling

**Definition**: Horizontal scaling means increasing (or decreasing) the number of Pod replicas. This distributes load across multiple instances.

**Manual Scaling**

```bash
# Scale a deployment to 5 replicas
kubectl scale deployment api-deployment --replicas=5
```

**Horizontal Pod Autoscaler (HPA)**

**Definition**: HPA automatically scales the number of Pods based on observed CPU/memory utilization or custom metrics.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

```bash
# Apply the HPA
kubectl apply -f hpa.yaml

# View HPA status
kubectl get hpa
```

**Common Mistakes / Notes**

- HPA requires the Metrics Server to be installed in the cluster.
- HPA does not scale to zero by default. Use KEDA for scale-to-zero behavior.
- Always define `resources.requests` in your Deployment; HPA cannot function without them.

---

## 8. Config & Secrets

---

### ConfigMaps

**Definition**: A **ConfigMap** stores non-confidential configuration data as key-value pairs. ConfigMaps decouple configuration from container images.

**Creating a ConfigMap**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
  API_BASE_URL: https://api.example.com
  config.json: |
    {
      "timeout": 30,
      "retries": 3
    }
```

**Using a ConfigMap as Environment Variables**

```yaml
containers:
  - name: app
    envFrom:
      - configMapRef:
          name: app-config
```

**Using a ConfigMap as a Mounted File**

```yaml
containers:
  - name: app
    volumeMounts:
      - name: config-volume
        mountPath: /app/config

volumes:
  - name: config-volume
    configMap:
      name: app-config
```

---

### Secrets

**Definition**: A **Secret** stores sensitive data (passwords, API keys, TLS certificates) in base64-encoded form. Secrets prevent sensitive values from being embedded in YAML manifests.

**Important Note**: Base64 encoding is not encryption. Secrets are only as secure as your cluster's RBAC and etcd encryption configuration.

**Creating a Secret**

```bash
# Create a secret from literal values
kubectl create secret generic db-credentials \
  --from-literal=username=appuser \
  --from-literal=password=s3cret!

# Create a TLS secret
kubectl create secret tls my-tls \
  --cert=server.crt \
  --key=server.key
```

```yaml
# Secret YAML (values must be base64 encoded)
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YXBwdXNlcg==     # base64 of "appuser"
  password: czNjcmV0IQ==     # base64 of "s3cret!"
```

**Using a Secret as Environment Variables**

```yaml
containers:
  - name: app
    env:
      - name: DB_USER
        valueFrom:
          secretKeyRef:
            name: db-credentials
            key: username
      - name: DB_PASS
        valueFrom:
          secretKeyRef:
            name: db-credentials
            key: password
```

**Common Mistakes / Notes**

- Never commit Secret YAML files containing actual secret values to version control.
- Use tools like Sealed Secrets, External Secrets Operator, or HashiCorp Vault for production secret management.
- Use `stringData` instead of `data` when writing Secrets by hand to avoid manual base64 encoding:

```yaml
stringData:
  password: s3cret!   # Kubernetes encodes automatically
```

---

## 9. Debugging in Kubernetes

---

### Step-by-Step Debugging Workflow

**Step 1**: Check Pod status

```bash
kubectl get pods
```

Look for `STATUS`: `CrashLoopBackOff`, `ImagePullBackOff`, `Pending`, `Error`.

---

**Step 2**: Describe the Pod to view events

```bash
kubectl describe pod <pod-name>
```

The `Events` section at the bottom shows the exact failure reason (image not found, OOMKilled, failed scheduling, etc.).

---

**Step 3**: View container logs

```bash
kubectl logs <pod-name>

# If the container has crashed, view previous container logs
kubectl logs <pod-name> --previous
```

---

**Step 4**: Execute commands inside the container

```bash
kubectl exec -it <pod-name> -- sh

# Test database connectivity
kubectl exec -it api-pod -- sh -c "nc -zv db-service 5432"

# Check DNS resolution
kubectl exec -it api-pod -- nslookup db-service
```

---

### Common Pod Failure States

| Status               | Cause                                        | Resolution                                      |
|----------------------|----------------------------------------------|-------------------------------------------------|
| `ImagePullBackOff`   | Image not found or wrong credentials         | Check image name/tag, registry credentials      |
| `CrashLoopBackOff`   | Container crashes repeatedly on start        | Check `kubectl logs --previous`                 |
| `Pending`            | No node available with sufficient resources  | Check `kubectl describe pod` for scheduling     |
| `OOMKilled`          | Container exceeded memory limit              | Increase memory limit or fix memory leak        |
| `Error`              | Container exited with non-zero code          | Check application logs                          |
| `Terminating` (stuck)| PVC or finalizer blocking deletion           | Check `kubectl describe` and finalizers         |

---

## 10. Namespaces

**Definition**

A **Namespace** is a logical partition within a Kubernetes cluster that isolates resources between teams, environments, or applications. Resources in one namespace are not visible to another namespace by default (though networking can span namespaces).

---

**Explanation / Working**

Namespaces allow multiple teams or environments (dev, staging, production) to share the same cluster without interfering with each other.

```bash
# List namespaces
kubectl get namespaces

# Create a namespace
kubectl create namespace staging

# Run commands in a specific namespace
kubectl get pods -n staging

# Apply a manifest to a specific namespace
kubectl apply -f deployment.yaml -n staging

# Set default namespace for current context
kubectl config set-context --current --namespace=staging
```

---

**Namespace YAML**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: staging
  labels:
    environment: staging
```

**Default Namespaces**

| Namespace       | Purpose                                      |
|-----------------|----------------------------------------------|
| `default`       | Resources created without specifying a namespace |
| `kube-system`   | Kubernetes internal components               |
| `kube-public`   | Publicly accessible cluster information      |
| `kube-node-lease` | Node heartbeat leases                      |

**Common Mistakes / Notes**

- Not all Kubernetes resources are namespaced. Nodes, PersistentVolumes, and ClusterRoles are cluster-scoped.
- Use ResourceQuotas to limit CPU, memory, and object counts per namespace to prevent a single team from consuming all cluster resources.

---

## 11. Ingress

**Definition**

An **Ingress** is a Kubernetes resource that manages external HTTP and HTTPS access to Services within the cluster. It acts as an HTTP-aware reverse proxy, providing host-based and path-based routing, TLS termination, and load balancing.

**Why Not Just Use LoadBalancer Services?**

A `LoadBalancer` Service provisions one external IP per Service. This is expensive at scale. Ingress uses a single LoadBalancer with an **Ingress Controller** to route traffic to multiple internal services based on request rules.

---

### Ingress Controller

**Definition**: An **Ingress Controller** is the software component that implements the Ingress resource. It must be installed in the cluster. Common controllers: **NGINX Ingress Controller**, Traefik, AWS ALB Ingress Controller, HAProxy.

```bash
# Install NGINX Ingress Controller (example)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

---

### Ingress YAML Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx

  # TLS configuration
  tls:
    - hosts:
        - api.example.com
        - app.example.com
      secretName: tls-certificate

  rules:
    # Route traffic based on hostname and path

    - host: api.example.com
      http:
        paths:
          - path: /v1
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80

    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

---

### Routing Concepts

| Rule Type     | Description                                      | Example                           |
|---------------|--------------------------------------------------|-----------------------------------|
| Host-based    | Route based on the `Host` HTTP header            | `api.example.com` → api-service   |
| Path-based    | Route based on the URL path                      | `/api/` → api, `/` → frontend     |
| TLS           | Terminate HTTPS at the Ingress controller        | Cert stored as a K8s Secret       |

**Common Mistakes / Notes**

- An Ingress resource has no effect without an Ingress Controller deployed in the cluster.
- For TLS, the referenced Secret must exist in the same namespace as the Ingress.
- `pathType: Prefix` matches any path starting with the specified string. `pathType: Exact` requires an exact match.
- Use cert-manager to automate TLS certificate provisioning from Let's Encrypt.

---

---

# Quick Reference Summary

## Docker at a Glance

| Concept         | Summary                                                   |
|-----------------|-----------------------------------------------------------|
| Image           | Immutable template for containers                         |
| Container       | Running instance of an image                             |
| Dockerfile      | Instructions to build an image                           |
| Layer           | Each Dockerfile instruction creates a cached layer        |
| Volume          | Persistent data storage outside container lifecycle       |
| Network         | Isolated communication channel between containers         |
| Docker Compose  | Multi-container application definition in YAML            |
| Registry        | Remote storage and distribution for images                |

## Kubernetes at a Glance

| Concept         | Summary                                                   |
|-----------------|-----------------------------------------------------------|
| Cluster         | Full K8s deployment (control plane + nodes)              |
| Pod             | Smallest unit; one or more containers with shared network |
| Deployment      | Manages stateless Pods with rolling updates              |
| StatefulSet     | Manages stateful Pods with stable identity               |
| Service         | Stable endpoint for accessing Pods                       |
| ConfigMap       | Stores non-sensitive configuration                       |
| Secret          | Stores sensitive configuration                           |
| Namespace       | Logical cluster partition                                |
| Ingress         | HTTP/S routing to internal services                      |
| HPA             | Auto-scales Pod replicas based on metrics                |

---

*End of Document*
