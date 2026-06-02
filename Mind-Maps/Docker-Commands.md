# 🐳 Docker Commands Mind Map (Production-Focused | DevOps/SRE 0–3 YOE)

```text
DOCKER COMMANDS
│
├── 1. Environment Information
│   │
│   ├── docker version
│   ├── docker info
│   └── docker system df
│
├── 2. Image Management
│   │
│   ├── docker pull <image>
│   ├── docker images
│   ├── docker image ls
│   ├── docker build -t app:v1 .
│   ├── docker tag image new-image
│   ├── docker push image
│   ├── docker rmi image
│   └── docker image inspect image
│
├── 3. Container Lifecycle
│   │
│   ├── docker run
│   ├── docker start
│   ├── docker stop
│   ├── docker restart
│   ├── docker kill
│   ├── docker pause
│   ├── docker unpause
│   ├── docker rm
│   └── docker rename
│
├── 4. Container Monitoring (MOST USED)
│   │
│   ├── docker ps
│   ├── docker ps -a
│   ├── docker logs
│   ├── docker logs -f
│   ├── docker stats
│   ├── docker top
│   ├── docker inspect
│   └── docker events
│
├── 5. Container Debugging (DAILY USE)
│   │
│   ├── docker exec -it
│   ├── docker inspect
│   ├── docker logs
│   ├── docker cp
│   ├── docker diff
│   └── docker port
│
├── 6. Networking
│   │
│   ├── docker network ls
│   ├── docker network create
│   ├── docker network inspect
│   ├── docker network connect
│   ├── docker network disconnect
│   └── docker network rm
│
├── 7. Volumes
│   │
│   ├── docker volume create
│   ├── docker volume ls
│   ├── docker volume inspect
│   ├── docker volume rm
│   └── docker volume prune
│
├── 8. Docker Compose
│   │
│   ├── docker compose up -d
│   ├── docker compose down
│   ├── docker compose ps
│   ├── docker compose logs
│   ├── docker compose restart
│   ├── docker compose config
│   └── docker compose pull
│
├── 9. Cleanup Commands
│   │
│   ├── docker system prune
│   ├── docker image prune
│   ├── docker container prune
│   ├── docker network prune
│   └── docker volume prune
│
├── 10. Registry Operations
│   │
│   ├── docker login
│   ├── docker logout
│   ├── docker pull
│   ├── docker push
│   └── docker search
│
└── 11. Production Troubleshooting Flow
    │
    ├── docker ps
    ├── docker logs
    ├── docker inspect
    ├── docker exec
    ├── docker stats
    ├── docker top
    ├── docker network inspect
    └── docker events
```

---

# 🚀 Daily Production Commands (80% of Real Work)

## 1. Check Running Containers

```bash
docker ps
```

Output:

```text
CONTAINER ID   IMAGE      STATUS
abc123         nginx      Up 5 hours
```

Used for:

* Checking running containers
* Validating deployments
* Incident troubleshooting

---

## 2. View All Containers

```bash
docker ps -a
```

Shows:

* Running
* Exited
* Failed containers

---

## 3. View Logs

### Last logs

```bash
docker logs nginx
```

### Live logs

```bash
docker logs -f nginx
```

Production use:

```bash
docker logs -f payment-service
```

Check:

* Application errors
* Database connectivity issues
* Startup failures

---

## 4. Enter a Container

```bash
docker exec -it nginx bash
```

or

```bash
docker exec -it nginx sh
```

Production use:

```bash
docker exec -it api-container bash
```

Check:

```bash
ps aux
env
cat /etc/hosts
curl localhost:8080
```

---

## 5. Inspect Container Details

```bash
docker inspect nginx
```

Shows:

* IP Address
* Mounts
* Volumes
* Network
* Environment Variables

Useful:

```bash
docker inspect nginx | grep IPAddress
```

---

## 6. Monitor Resource Usage

```bash
docker stats
```

Output:

```text
CPU %
MEM USAGE
NET I/O
BLOCK I/O
```

Production use:

Detect:

* Memory leaks
* CPU spikes
* Resource starvation

---

## 7. View Processes Inside Container

```bash
docker top nginx
```

Equivalent to:

```bash
ps aux
```

inside the container.

---

## 8. Copy Files

### Host → Container

```bash
docker cp config.yaml nginx:/tmp/
```

### Container → Host

```bash
docker cp nginx:/var/log/app.log .
```

Useful during incidents.

---

# 🌐 Networking Commands

## List Networks

```bash
docker network ls
```

---

## Inspect Network

```bash
docker network inspect app-network
```

Shows:

* Connected containers
* Subnet
* Gateway
* IP addresses

---

## Create Network

```bash
docker network create app-network
```

---

# 💾 Volume Commands

## List Volumes

```bash
docker volume ls
```

---

## Inspect Volume

```bash
docker volume inspect mysql-data
```

---

## Create Volume

```bash
docker volume create mysql-data
```

---

# 🧹 Cleanup Commands

## Remove Stopped Containers

```bash
docker container prune
```

---

## Remove Unused Images

```bash
docker image prune
```

---

## Complete Cleanup

```bash
docker system prune -a
```

⚠️ Be careful in production.

---

# 📦 Image Management

## Build Image

```bash
docker build -t myapp:v1 .
```

---

## Pull Image

```bash
docker pull nginx
```

---

## Push Image

```bash
docker push myrepo/myapp:v1
```

---

# 🔥 Docker Compose Commands

## Start Application

```bash
docker compose up -d
```

---

## Check Containers

```bash
docker compose ps
```

---

## View Logs

```bash
docker compose logs -f
```

---

## Restart Service

```bash
docker compose restart web
```

---

## Stop Everything

```bash
docker compose down
```

---

# 🚨 Real Production Debugging Flow

When someone says:

> "Application is down"

Follow this order:

### Step 1

```bash
docker ps
```

Is container running?

---

### Step 2

```bash
docker logs <container>
```

Any errors?

---

### Step 3

```bash
docker stats
```

CPU or Memory issue?

---

### Step 4

```bash
docker inspect <container>
```

Network? Mounts? Env vars?

---

### Step 5

```bash
docker exec -it <container> bash
```

Check application manually.

---

### Step 6

```bash
docker network inspect app-network
```

Can containers communicate?

---

### Step 7

```bash
docker events
```

See real-time Docker events:

```text
container start
container stop
container die
```

---

# 🎯 Top 15 Commands Every DevOps Engineer Should Memorize

```bash
docker ps
docker ps -a
docker logs -f
docker exec -it
docker inspect
docker stats
docker top
docker build
docker pull
docker push
docker network inspect
docker volume ls
docker compose up -d
docker compose logs -f
docker system prune
```

If you're preparing for DevOps/SRE interviews, these commands alone cover most Docker troubleshooting and operational scenarios encountered in daily production work.
