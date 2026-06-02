# 🐳 Docker Interview Answers (0–3 YOE DevOps/SRE)

These are concise, interview-ready answers that you can speak confidently.

---

# 1. Docker Architecture

### Q. Explain Docker Architecture.

**Answer:**

Docker follows a client-server architecture.

* **Docker Client** (`docker`) is where users run commands.
* **Docker Daemon** (`dockerd`) receives those commands and manages containers, images, networks, and volumes.
* **Container Runtime** (`containerd`, `runc`) actually creates and runs containers.
* **Docker Registry** stores images such as Docker Hub, ECR, or GCR.

### Flow:

```text
Docker CLI
     │
     ▼
 Docker Daemon
     │
     ▼
 Container Runtime
     │
     ▼
 Container
```

Example:

```bash
docker run nginx
```

The client sends the request to the daemon, which pulls the image if needed and starts the container.

---

# 2. Images vs Containers

### Q. Difference between Docker Image and Container?

| Image              | Container              |
| ------------------ | ---------------------- |
| Blueprint          | Running instance       |
| Read-only          | Read-write layer added |
| Static             | Dynamic                |
| Stored in Registry | Runs on Host           |

### Example

Image:

```bash
docker pull nginx
```

Container:

```bash
docker run nginx
```

### Interview Answer

"A Docker image is a template containing application code, libraries, and dependencies. A container is a running instance of that image."

---

# 3. Dockerfile Instructions

### Q. What is a Dockerfile?

A Dockerfile is a text file containing instructions to build an image.

Example:

```dockerfile
FROM node:20

WORKDIR /app

COPY . .

RUN npm install

EXPOSE 3000

CMD ["npm","start"]
```

---

### Common Instructions

### FROM

Base image.

```dockerfile
FROM ubuntu:22.04
```

---

### RUN

Executes commands during image build.

```dockerfile
RUN apt update
```

---

### COPY

Copies files.

```dockerfile
COPY . .
```

---

### WORKDIR

Sets working directory.

```dockerfile
WORKDIR /app
```

---

### ENV

Sets environment variables.

```dockerfile
ENV APP_ENV=prod
```

---

### EXPOSE

Documents application port.

```dockerfile
EXPOSE 8080
```

---

### CMD

Default command.

```dockerfile
CMD ["nginx","-g","daemon off;"]
```

---

# 4. CMD vs ENTRYPOINT

### Q. Difference between CMD and ENTRYPOINT?

### CMD

Provides default arguments.

```dockerfile
CMD ["nginx"]
```

Can be overridden.

```bash
docker run image bash
```

---

### ENTRYPOINT

Defines the main executable.

```dockerfile
ENTRYPOINT ["nginx"]
```

Cannot be easily replaced.

---

### Combined Usage

```dockerfile
ENTRYPOINT ["ping"]
CMD ["google.com"]
```

Run:

```bash
docker run image
```

Result:

```bash
ping google.com
```

Override CMD:

```bash
docker run image yahoo.com
```

Result:

```bash
ping yahoo.com
```

### Interview One-liner

"ENTRYPOINT defines the main process while CMD provides default arguments to that process."

---

# 5. Volumes vs Bind Mounts

### Q. Difference between Volume and Bind Mount?

| Volume            | Bind Mount         |
| ----------------- | ------------------ |
| Managed by Docker | Managed by Host    |
| Preferred         | Mostly development |
| Portable          | Host dependent     |
| Safer             | Less secure        |

---

### Volume

```bash
docker volume create myvol

docker run -v myvol:/data nginx
```

Docker manages storage.

---

### Bind Mount

```bash
docker run -v /home/user/data:/data nginx
```

Directly maps host path.

---

### Interview Answer

"Volumes are Docker-managed persistent storage and are recommended for production. Bind mounts directly use host directories and are mostly used during development."

---

# 6. Bridge Networking

### Q. How does Docker networking work?

Docker provides isolated networks.

Default network:

```text
Bridge Network
```

Container gets:

* Private IP
* Virtual Ethernet Pair
* NAT through host

Example:

```bash
docker network ls
```

Create network:

```bash
docker network create app-net
```

Attach containers:

```bash
docker run --network app-net nginx
```

---

### Q. How do two containers communicate?

If both are on the same custom bridge network:

```bash
ping container-name
```

Docker provides internal DNS resolution.

---

# 7. Docker Compose

### Q. What is Docker Compose?

Docker Compose manages multi-container applications using a YAML file.

Example:

```yaml
services:
  web:
    image: nginx

  db:
    image: mysql
```

Start:

```bash
docker compose up -d
```

Stop:

```bash
docker compose down
```

---

### Use Cases

* Application
* Database
* Redis
* Monitoring stack

all managed together.

---

# 8. Multi-stage Builds

### Q. What is a Multi-stage Build?

Used to reduce image size.

Example:

```dockerfile
FROM node:20 AS builder

WORKDIR /app

COPY . .

RUN npm install
RUN npm run build

FROM nginx

COPY --from=builder /app/build /usr/share/nginx/html
```

---

### Benefits

* Smaller image
* Better security
* Faster deployments

### Interview Answer

"Multi-stage builds separate build dependencies from runtime dependencies, significantly reducing image size."

---

# 9. Namespaces and Cgroups

### Q. How Docker provides isolation?

Docker uses Linux kernel features.

---

## Namespaces

Provide isolation.

Types:

### PID Namespace

Process isolation.

```bash
ps aux
```

inside container only shows container processes.

---

### Network Namespace

Separate network stack.

Each container gets:

* IP
* Routing Table
* Interfaces

---

### Mount Namespace

Separate filesystem view.

---

### UTS Namespace

Separate hostname.

---

### IPC Namespace

Separate shared memory.

---

## Cgroups

Control resource usage.

Limit CPU:

```bash
docker run --cpus=1 nginx
```

Limit Memory:

```bash
docker run -m 512m nginx
```

---

### Interview Answer

"Namespaces provide isolation while Cgroups provide resource control. Together they enable containerization."

---

# 10. Troubleshooting Commands

### Q. Container is failing. What will you do?

### Step 1

Check status

```bash
docker ps -a
```

---

### Step 2

Check logs

```bash
docker logs container-id
```

---

### Step 3

Inspect container

```bash
docker inspect container-id
```

---

### Step 4

Enter container

```bash
docker exec -it container-id bash
```

---

### Step 5

Monitor resources

```bash
docker stats
```

---

### Step 6

Check processes

```bash
docker top container-id
```

---

# 11. Image Optimization

### Q. How do you reduce Docker image size?

### Use Small Base Images

```dockerfile
FROM alpine
```

instead of Ubuntu.

---

### Multi-stage Builds

Removes build tools.

---

### Use .dockerignore

Example:

```text
.git
node_modules
logs
```

---

### Combine RUN Commands

Bad:

```dockerfile
RUN apt update
RUN apt install curl
```

Good:

```dockerfile
RUN apt update && apt install -y curl
```

---

### Remove Unnecessary Files

```dockerfile
RUN rm -rf /var/lib/apt/lists/*
```

---

### Interview Answer

"I optimize images using Alpine images, multi-stage builds, .dockerignore, layer caching, and removing unnecessary files."

---

# 12. Docker Security Best Practices

### Q. How do you secure Docker containers?

### Run as Non-root

Bad:

```dockerfile
USER root
```

Good:

```dockerfile
RUN useradd appuser
USER appuser
```

---

### Read-only Filesystem

```bash
docker run --read-only nginx
```

---

### Limit Resources

```bash
docker run -m 512m --cpus=1 nginx
```

---

### Scan Images

Examples:

* Trivy
* Clair
* Docker Scout

---

### Use Trusted Images

Prefer:

* Official images
* Verified publishers

---

### Store Secrets Securely

Avoid:

```dockerfile
ENV PASSWORD=admin123
```

Use:

* Docker Secrets
* Kubernetes Secrets
* AWS Secrets Manager

---

# 🎯 Most Common Docker Interview Question

### Q. How does Docker work internally?

**Answer:**

"When I run `docker run nginx`, the Docker client sends the request to the Docker daemon. The daemon checks whether the image exists locally. If not, it pulls it from Docker Hub. Then Docker uses containerd and runc to create the container. Linux namespaces provide isolation, Cgroups control resource usage, and OverlayFS creates the container filesystem. Finally, the container process starts and runs independently."

This answer alone can impress many 0–3 YOE DevOps interviewers because it covers architecture, runtime, namespaces, cgroups, and storage in one flow.
