# ZeroClaw AI — System Architecture Document v3.0

> Per-User Docker Container Isolation Pattern
> February 28, 2026

---

## 1. Architecture Overview

ZeroClaw AI uses a **per-user Docker container isolation pattern** where every user gets their own dedicated Docker container running an isolated ZeroClaw agent instance. This pattern is inspired by [Reed Dev's production Adola bot](https://dev.to/reeddev42/running-one-docker-container-per-user-on-a-35month-server-51hh) and adapted for our mobile-first AI agent platform.

### 1.1 Core Principle

One Docker container = one user = complete isolation. No user can ever see, access, or affect another user's data, memory, or agent state.

### 1.2 High-Level System Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│  MOBILE APP (Expo SDK 54 + React Native + NativeWind)                   │
│  ─ Chat UI  ─ Daily Planner  ─ Integrations Hub  ─ Voice Mode          │
│  ─ Settings  ─ Paywall (RevenueCat)  ─ Onboarding Flow                 │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │ HTTPS / WSS
                                ▼
┌───────────────────────────────────────────────────────────────────────┐
│  SUPABASE CLOUD                                                       │
│  ┌──────────┐ ┌──────────────┐ ┌──────────┐ ┌──────────┐            │
│  │   Auth   │ │  PostgreSQL  │ │  Edge Fn  │ │ Realtime │            │
│  │(email/   │ │  (users,     │ │(orchestr- │ │  (live   │            │
│  │ social)  │ │ subs, goals, │ │ ator, cron│ │  agent   │            │
│  │          │ │ tasks, etc.) │ │ webhooks) │ │  status) │            │
│  └──────────┘ └──────────────┘ └─────┬─────┘ └──────────┘            │
│                                       │ Storage (files/media)          │
└───────────────────────────────────────┼───────────────────────────────┘
                                        │ HTTPS (authenticated)
                                        ▼
┌───────────────────────────────────────────────────────────────────────┐
│  HETZNER VPS (Agent Runtime Server)                                   │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  GATEWAY CONTAINER (always running)                              │ │
│  │                                                                   │ │
│  │  ┌──────────────┐  ┌───────────────┐  ┌──────────────────────┐ │ │
│  │  │ HTTP Server  │  │ Container     │  │ Idle Cleanup Loop    │ │ │
│  │  │ (receives    │  │ Lifecycle Mgr │  │ (every 5 min, stops  │ │ │
│  │  │  requests    │  │ (create/start │  │  inactive containers │ │ │
│  │  │  from Edge   │  │  /stop via    │  │  after 10 min idle)  │ │ │
│  │  │  Functions)  │  │  Docker API)  │  │                      │ │ │
│  │  └──────────────┘  └───────────────┘  └──────────────────────┘ │ │
│  │                                                                   │ │
│  │  Mounts: /var/run/docker.sock (Docker-out-of-Docker pattern)     │ │
│  │  Mounts: /data (host filesystem for user volumes)                │ │
│  └───────────────────────────┬─────────────────────────────────────┘ │
│                               │ Docker API (via socket)               │
│                               ▼                                       │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐      │
│  │ZC-usr-a │ │ZC-usr-b │ │ZC-usr-c │ │ZC-usr-d │ │ZC-usr-e │ ... │
│  │ RUNNING │ │ RUNNING │ │ STOPPED │ │ RUNNING │ │ STOPPED │      │
│  │ ~50MB   │ │ ~50MB   │ │  0 MB   │ │ ~50MB   │ │  0 MB   │      │
│  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘      │
│       │           │           │           │           │              │
│  /data/users/a  /data/users/b  ...      ...         ...             │
│  (bind mounts — persistent user data on host disk)                   │
│                                                                       │
│  ┌────────────────────┐  ┌─────────┐  ┌──────────┐                  │
│  │  Docker Socket     │  │  Redis  │  │ Scrapling│                  │
│  │  Proxy (restricts  │  │ (user → │  │ Sidecar  │                  │
│  │  API surface)      │  │  cont.  │  │ (shared) │                  │
│  └────────────────────┘  │  map)   │  └──────────┘                  │
│                           └─────────┘                                │
│                                                                       │
│  Docker Network: zeroclaw-net (internal bridge, no external access)   │
└───────────────────────────────────────────────────────────────────────┘
                    │
                    ▼ (outbound HTTPS only)
        ┌───────────────────────┐
        │  External APIs        │
        │  ─ MiniMax M2.5 LLM  │
        │  ─ Composio (OAuth)   │
        │  ─ Deepgram (STT)     │
        │  ─ ElevenLabs (TTS)   │
        └───────────────────────┘
```

---

## 2. Per-User Container Pattern (Detailed)

### 2.1 The Docker-out-of-Docker Pattern

The Gateway container does not run Docker inside itself. Instead, it mounts the host's Docker socket (`/var/run/docker.sock`) and talks to the **host's Docker daemon** to create, start, stop, and remove sibling containers. This is the same battle-tested pattern used by CI/CD tools like Jenkins and GitLab Runner.

```
HOST MACHINE
├── Docker Daemon (dockerd)
│   ├── Gateway Container (always running, mounts docker.sock)
│   ├── ZC-user-abc (created by Gateway via Docker API)
│   ├── ZC-user-def (created by Gateway via Docker API)
│   ├── Redis Container
│   ├── Scrapling Container
│   └── Socket Proxy Container
│
├── /var/run/docker.sock  ← Gateway reads this to control siblings
│
└── /data/users/           ← Persistent storage on host disk
    ├── abc/               ← bind-mounted into ZC-user-abc
    ├── def/               ← bind-mounted into ZC-user-def
    └── ...
```

### 2.2 Container Lifecycle

```
USER SENDS MESSAGE
        │
        ▼
┌───────────────────┐
│  Supabase Edge    │
│  Function         │
│  (authenticates   │
│   user, checks    │
│   subscription)   │
└────────┬──────────┘
         │ HTTPS POST to Gateway
         ▼
┌───────────────────┐
│  Gateway checks   │
│  Redis:           │       ┌─────────────────────────────┐
│  container for    │──YES─▶│  Route request to container  │
│  this user_id     │       │  http://zc-{uid}:8080/chat  │
│  running?         │       │  Docker DNS resolves name    │
└────────┬──────────┘       │  Response in < 100ms        │
         │                  └─────────────────────────────┘
         NO
         │
         ▼
┌───────────────────┐
│  Container exists │       ┌─────────────────────────────┐
│  but stopped?     │──YES─▶│  docker start zc-{uid}      │
│                   │       │  Resume in ~1 second         │
└────────┬──────────┘       │  Volume already mounted      │
         │                  │  All state preserved         │
         NO                 └─────────────────────────────┘
         │
         ▼
┌───────────────────┐
│  Create new       │
│  container:       │
│                   │
│  docker create    │
│    --name zc-{uid}│
│    --volume       │
│    /data/users/   │
│    {uid}:/agent   │
│    --network      │
│    zeroclaw-net   │
│    --memory 50m   │
│    --cpus 0.5     │
│    zeroclaw:latest│
│                   │
│  docker start     │
│    zc-{uid}       │
│                   │
│  Time: ~2-3 sec   │
└───────────────────┘
```

### 2.3 Idle Cleanup Process

The Gateway runs a cleanup loop every 5 minutes. This is critical for keeping resource usage proportional to **concurrent** users, not total users.

```
EVERY 5 MINUTES:
┌─────────────────────────────────┐
│  For each entry in Redis:       │
│    user_id → {container, last}  │
│                                 │
│  If now - last_active > 10 min: │
│    docker stop zc-{uid}         │
│    Remove from active map       │
│    Container stays on disk      │
│    (can be restarted in ~1s)    │
│                                 │
│  If stopped > 24 hours:         │
│    docker rm zc-{uid}           │
│    Volume stays on disk         │
│    (fresh container on next     │
│     request, same user data)    │
└─────────────────────────────────┘
```

**Resource states:**

| State | CPU | RAM | Disk | Resume Time |
|-------|-----|-----|------|-------------|
| Running (active) | Up to 0.5 cores | ~50MB | Volume mounted | Instant |
| Stopped (idle) | 0 | 0 | ~100MB writable layer + volume | ~1 second |
| Removed (long idle) | 0 | 0 | Volume only (~10-500MB) | ~2-3 seconds |

### 2.4 Gateway Implementation

```javascript
// gateway/index.js — Core lifecycle management

const Docker = require('dockerode');
const docker = new Docker({ socketPath: '/var/run/docker.sock' });
const Redis = require('ioredis');
const redis = new Redis();

const CONTAINER_IMAGE = 'zeroclaw-agent:latest';
const NETWORK = 'zeroclaw-net';
const IDLE_TIMEOUT = 10 * 60 * 1000;  // 10 minutes
const CLEANUP_INTERVAL = 5 * 60 * 1000; // 5 minutes
const MAX_CONTAINERS = 500;

// Ensure a user has a running container
async function ensureContainer(userId) {
  const name = `zc-${userId.slice(0, 12)}`;

  try {
    const container = docker.getContainer(name);
    const info = await container.inspect();

    if (!info.State.Running) {
      await container.start();
    }

    await redis.hset(`user:${userId}`, 'container', name, 'last_active', Date.now());
    return name;

  } catch (err) {
    // Container doesn't exist — create it
    const userDir = `/data/users/${userId}`;
    await ensureUserDirectory(userDir);

    const container = await docker.createContainer({
      name,
      Image: CONTAINER_IMAGE,
      HostConfig: {
        Binds: [`${userDir}:/agent`],
        NetworkMode: NETWORK,
        Memory: 50 * 1024 * 1024,      // 50MB hard limit
        NanoCpus: 500000000,            // 0.5 CPU
        PidsLimit: 50,                  // Max 50 processes
        ReadonlyRootfs: true,           // Read-only container FS
        Tmpfs: { '/tmp': 'rw,noexec,nosuid,size=10m' },
        SecurityOpt: ['no-new-privileges:true'],
        CapDrop: ['ALL'],
        CapAdd: ['NET_RAW'],
      },
      User: '1000:1000',
    });

    await container.start();
    await redis.hset(`user:${userId}`, 'container', name, 'last_active', Date.now());
    return name;
  }
}

// Route a message to the user's container
async function routeMessage(userId, message) {
  const containerName = await ensureContainer(userId);
  const response = await fetch(`http://${containerName}:8080/chat`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ message, user_id: userId }),
  });
  return response.json();
}

// Idle cleanup loop
setInterval(async () => {
  const keys = await redis.keys('user:*');
  for (const key of keys) {
    const data = await redis.hgetall(key);
    if (Date.now() - parseInt(data.last_active) > IDLE_TIMEOUT) {
      try {
        await docker.getContainer(data.container).stop();
        await redis.del(key);
      } catch (err) {
        // Container already stopped or removed
      }
    }
  }
}, CLEANUP_INTERVAL);
```

### 2.5 Networking

All user containers join a single internal Docker bridge network (`zeroclaw-net`). No container is exposed to the internet directly.

```
zeroclaw-net (Docker bridge network)
│
├── gateway          (only service with external port: 8080)
├── redis            (internal only, port 6379)
├── scrapling        (internal only, port 8001)
├── socket-proxy     (internal only, port 2375)
│
├── zc-user-abc123   (internal only, port 8080)
├── zc-user-def456   (internal only, port 8080)
└── zc-user-ghi789   (internal only, port 8080)
```

The Gateway calls user containers **by name** using Docker's built-in DNS resolution:

```
http://zc-user-abc123:8080/chat
```

No port mapping is needed. No container is reachable from outside the Docker network. Only the Gateway has an exposed port, and even that sits behind Caddy/Nginx with TLS.

---

## 3. Per-User Data Volume

### 3.1 Volume Structure

Every user's data lives in a **bind-mounted directory** on the host filesystem. Bind mounts (not Docker named volumes) are used so the Gateway can read user files directly without `docker exec`.

```
/data/users/{user_id}/
│
├── config.toml                  ← Agent configuration
│   [agent]
│   name = "Alia's Assistant"
│   autonomy_level = "supervised"    # readonly | supervised | full
│   default_provider = "minimax"
│   wake_time = "07:00"
│   timezone = "Asia/Karachi"
│
│   [memory]
│   backend = "sqlite"
│   auto_save = true
│
│   [secrets]
│   encrypt = true
│
│   [workspace]
│   workspace_only = true
│
├── memory.db                    ← SQLite hybrid search database
│   - Conversation history
│   - Vector embeddings for semantic search
│   - Keyword index for exact match
│   - Learned user preferences
│   - Bookmarked information
│
├── secrets.enc                  ← Encrypted credentials file
│   - Composio entity ID
│   - Per-user API tokens
│   - Connected app OAuth tokens
│   - Encryption key derived from user-specific salt
│
├── soul.md                      ← Agent personality & rules
│   - Custom behavioral instructions
│   - User-defined constraints ("never schedule before 10am")
│   - Tone and language preferences
│   - Domain-specific knowledge
│
├── skills/                      ← Installed skills from marketplace
│   ├── inbox-digest/
│   │   ├── skill.toml           (manifest: name, version, tools)
│   │   └── SKILL.md             (instructions for the agent)
│   ├── fitness-planner/
│   │   ├── skill.toml
│   │   └── SKILL.md
│   └── daily-brief/
│       ├── skill.toml
│       └── SKILL.md
│
├── schedules/                   ← Cron job definitions
│   ├── morning-brief.json       (7am daily digest)
│   ├── inbox-digest.json        (8pm newsletter summary)
│   ├── weekly-report.json       (Sunday 6pm recap)
│   └── meeting-prep.json        (15min before calendar events)
│
└── workspace/                   ← Agent's working directory
    ├── drafts/                  (email drafts, social posts)
    ├── research/                (scrapling reports, summaries)
    ├── exports/                 (CSV exports, generated docs)
    └── temp/                    (scratch space, cleaned weekly)
```

### 3.2 Why Bind Mounts Over Docker Volumes

| Aspect | Bind Mounts (our choice) | Docker Named Volumes |
|--------|--------------------------|---------------------|
| Gateway access | Direct filesystem read — no `docker exec` | Requires `docker exec` or volume sharing |
| Backup | Simple `rsync` or `tar` of `/data/users/` | Need `docker volume` commands |
| Scheduling | Gateway reads `schedules/*.json` while container is stopped | Container must be running |
| Migration | Copy directory to new server, done | Export/import volume lifecycle |
| Visibility | `ls /data/users/abc/` from host | Need Docker commands to inspect |

The Gateway needs to read user schedule files to trigger cron jobs (like the morning daily brief) even when the user's container is stopped. Bind mounts make this trivial.

---

## 4. Docker Compose (Production)

### 4.1 Core Services

```yaml
# docker-compose.yml
version: '3.8'

networks:
  zeroclaw-net:
    driver: bridge
    internal: true          # No external access
  gateway-proxy:
    driver: bridge          # Gateway + Socket Proxy only

services:

  # ─── REVERSE PROXY ───────────────────────────────────
  caddy:
    image: caddy:2-alpine
    restart: always
    ports:
      - "443:443"
      - "80:80"             # Redirect to HTTPS
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
    networks:
      - zeroclaw-net

  # ─── DOCKER SOCKET PROXY ─────────────────────────────
  # Restricts which Docker API calls the Gateway can make
  socket-proxy:
    image: tecnativa/docker-socket-proxy:latest
    restart: always
    environment:
      CONTAINERS: 1         # Allow container operations
      NETWORKS: 0           # Block network changes
      IMAGES: 0             # Block image operations
      VOLUMES: 0            # Block volume operations
      POST: 1               # Allow POST (create/start/stop)
      LOG_LEVEL: warning
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - gateway-proxy

  # ─── GATEWAY ──────────────────────────────────────────
  gateway:
    build: ./gateway
    restart: always
    environment:
      DOCKER_HOST: tcp://socket-proxy:2375
      REDIS_URL: redis://redis:6379
      CONTAINER_IMAGE: zeroclaw-agent:latest
      NETWORK: zeroclaw-net
      DATA_DIR: /data/users
      MAX_CONTAINERS: 500
      IDLE_TIMEOUT_MINUTES: 10
      CONTAINER_MEMORY_LIMIT: 50m
      CONTAINER_CPU_LIMIT: "0.5"
    volumes:
      - /data:/data                  # Access to user volumes
    depends_on:
      - socket-proxy
      - redis
    networks:
      - zeroclaw-net
      - gateway-proxy

  # ─── REDIS ───────────────────────────────────────────
  redis:
    image: redis:7-alpine
    restart: always
    command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru
    volumes:
      - redis_data:/data
    networks:
      - zeroclaw-net

  # ─── SCRAPLING SIDECAR ──────────────────────────────
  scrapling:
    build: ./scrapling-sidecar
    restart: always
    environment:
      MAX_CONCURRENT_REQUESTS: 10
      REQUEST_TIMEOUT: 30
    networks:
      - zeroclaw-net

  # ─── SCHEDULER ──────────────────────────────────────
  # Reads user schedule files and triggers agent actions
  scheduler:
    build: ./scheduler
    restart: always
    environment:
      GATEWAY_URL: http://gateway:8080
      DATA_DIR: /data/users
    volumes:
      - /data:/data:ro               # Read-only access to user schedules
    networks:
      - zeroclaw-net

volumes:
  caddy_data:
  redis_data:
```

### 4.2 User Container Template

User containers are NOT defined in docker-compose. They are created dynamically by the Gateway via the Docker API. The image is pre-built and pulled once.

```dockerfile
# Dockerfile.zeroclaw-agent
FROM rust:1.77-slim AS builder

# Build ZeroClaw from source or use pre-built binary
COPY zeroclaw /usr/local/bin/zeroclaw
RUN chmod +x /usr/local/bin/zeroclaw

# Install Python for Scrapling skill integration
FROM python:3.12-slim

COPY --from=builder /usr/local/bin/zeroclaw /usr/local/bin/zeroclaw

# Minimal Python packages for tool skills
RUN pip install --no-cache-dir requests httpx

# Create non-root user
RUN useradd -u 1000 -m agent
USER agent

# Agent data mount point
VOLUME /agent
WORKDIR /agent

# ZeroClaw HTTP server port (internal only)
EXPOSE 8080

ENTRYPOINT ["zeroclaw", "serve", "--config", "/agent/config.toml", "--port", "8080"]
```

---

## 5. Security Architecture

### 5.1 Security Layers

```
Layer 1: Supabase Auth (JWT tokens, email/social login)
    │
Layer 2: Supabase RLS (users only access their own rows)
    │
Layer 3: Edge Function validation (subscription checks, rate limits)
    │
Layer 4: Gateway authentication (Edge Function → Gateway bearer token)
    │
Layer 5: Docker Socket Proxy (restricts API surface)
    │
Layer 6: Container isolation (separate process/filesystem/network per user)
    │
Layer 7: Container hardening (non-root, read-only FS, dropped capabilities)
    │
Layer 8: Encrypted secrets (ZeroClaw secrets.enc per user)
    │
Layer 9: Composio SOC 2 (managed OAuth, AI never sees raw credentials)
```

### 5.2 Container Hardening Checklist

| Setting | Value | Purpose |
|---------|-------|---------|
| `User` | `1000:1000` | Non-root execution — prevents privilege escalation |
| `ReadonlyRootfs` | `true` | Container filesystem is immutable — only /agent and /tmp writable |
| `Memory` | `50MB` | Hard RAM limit — prevents memory bombs |
| `NanoCpus` | `500000000` | 0.5 CPU — prevents CPU hogging |
| `PidsLimit` | `50` | Max 50 processes — prevents fork bombs |
| `SecurityOpt` | `no-new-privileges:true` | Blocks SUID/SGID escalation |
| `CapDrop` | `ALL` | Drop all Linux capabilities |
| `CapAdd` | `NET_RAW` | Add back only what's needed (HTTP requests) |
| `Tmpfs /tmp` | `rw,noexec,nosuid,size=10m` | Temp space: no executables, 10MB max |
| `NetworkMode` | `zeroclaw-net` | Internal Docker network only, no host access |

### 5.3 Docker Socket Proxy

Mounting `docker.sock` directly into the Gateway would give it full root access to the host. Instead, we use [Tecnativa docker-socket-proxy](https://github.com/Tecnativa/docker-socket-proxy) to expose only the API calls the Gateway needs:

| API Endpoint | Allowed | Purpose |
|-------------|---------|---------|
| `POST /containers/create` | Yes | Create new user containers |
| `POST /containers/{id}/start` | Yes | Resume stopped containers |
| `POST /containers/{id}/stop` | Yes | Stop idle containers |
| `DELETE /containers/{id}` | Yes | Remove long-idle containers |
| `GET /containers/{id}/json` | Yes | Inspect container state |
| `POST /images/*` | **No** | Blocked — prevent image manipulation |
| `POST /networks/*` | **No** | Blocked — prevent network changes |
| `POST /volumes/*` | **No** | Blocked — prevent volume manipulation |
| `POST /exec/*` | **No** | Blocked — prevent arbitrary command execution |

### 5.4 Network Isolation

```
INTERNET
    │
    ▼
┌──────────┐
│  Caddy   │  ← Only service with external ports (443, 80)
│  (TLS)   │  ← Terminates HTTPS, forwards to Gateway
└────┬─────┘
     │
     ▼
┌──────────────────────────────────────────────────────────┐
│  zeroclaw-net (internal: true — no external routing)     │
│                                                          │
│  gateway ←→ redis                                        │
│  gateway ←→ scrapling                                    │
│  gateway ←→ zc-user-abc (by container name)              │
│  gateway ←→ zc-user-def (by container name)              │
│                                                          │
│  zc-user-abc ←✕→ zc-user-def  (ICC can be disabled)     │
│  zc-user-* ←✕→ INTERNET  (outbound via Gateway proxy)   │
└──────────────────────────────────────────────────────────┘
```

User containers make outbound API calls (MiniMax, Composio) through the Gateway as a proxy, or directly if the network allows outbound HTTPS. Inter-container communication between user containers is disabled.

---

## 6. Scheduler Architecture

The Scheduler is a separate service that handles time-based triggers (morning briefs, cron jobs, meeting prep) without requiring user containers to be running 24/7.

```
┌─────────────────────────────────────────────────┐
│  SCHEDULER SERVICE                               │
│                                                   │
│  Every 60 seconds:                                │
│    1. Scan /data/users/*/schedules/*.json         │
│    2. For each schedule due now:                  │
│       - POST to Gateway with user_id + action    │
│       - Gateway spins up container if needed      │
│       - Container executes the scheduled task     │
│       - Container sends result (push notification,│
│         email digest, etc.)                       │
│       - Container goes idle → stopped after 10min │
│                                                   │
│  Schedule file example (morning-brief.json):      │
│  {                                                │
│    "type": "cron",                                │
│    "expression": "0 7 * * *",                     │
│    "timezone": "Asia/Karachi",                    │
│    "action": "daily_brief",                       │
│    "payload": {                                   │
│      "include_weather": true,                     │
│      "include_calendar": true,                    │
│      "include_tasks": true,                       │
│      "news_sources": ["hackernews", "bbc"]        │
│    }                                              │
│  }                                                │
└─────────────────────────────────────────────────┘
```

The Scheduler reads files from bind mounts (read-only). It never needs the container to be running. When a schedule fires, it tells the Gateway, which handles the container lifecycle.

---

## 7. Scaling Strategy

### 7.1 Capacity Planning

| Metric | Value |
|--------|-------|
| ZeroClaw container RAM | ~50MB active, 0MB stopped |
| Stopped container disk | <100MB writable layer |
| Cold start (create + start) | ~2-3 seconds |
| Warm restart (start stopped) | ~1 second |
| Concurrent user ratio | 10-15% of total users |

### 7.2 Scaling Phases

> **Note:** Docker Swarm has been removed from the scaling plan. While Swarm is maintained through 2030 (Mirantis), its ecosystem is declining, it lacks autoscaling (no HPA equivalent), and adopting it would require a second migration to Kubernetes later. Going directly from Docker Compose to k3s eliminates one migration step.

```
PHASE 1: Single VPS with Docker Compose (0-1,000 users)
────────────────────────────────────────────────────────
┌──────────────────────┐
│  Hetzner CX42        │   ~150 concurrent containers
│  16GB RAM, 8 vCPU    │   Orchestration: Docker Compose + Gateway
│                      │   No Swarm, no K8s
│  Gateway + Redis     │
│  + Scrapling         │
│  + N user containers │
└──────────────────────┘


PHASE 2: Multi-VPS with k3s (1,000-10,000+ users)
──────────────────────────────────────────────────
              ┌────────────────┐
              │  Hetzner LB    │
              │  (~€6/mo)      │
              └───────┬────────┘
                      │
         ┌────────────┼────────────┐
         │            │            │
   ┌─────┴─────┐ ┌───┴─────┐ ┌──┴──────┐
   │  VPS #1   │ │ VPS #2  │ │ VPS #3  │
   │  k3s      │ │ k3s     │ │ k3s     │
   │  Server   │ │ Agent   │ │ Agent   │
   │  CX42     │ │ CX42    │ │ CX42    │
   │  Control  │ │ Users   │ │ Users   │
   │  + Users  │ │ 3K-6K   │ │ 6K-10K  │
   └───────────┘ └─────────┘ └─────────┘

   k3s features used:
   - HPA (Horizontal Pod Autoscaler) for container count
   - PersistentVolumes for user data
   - Secrets management with external integration
   - Rolling deployments for zero-downtime updates
   - Same container image as Phase 1

   User routing: hash(user_id) → node assignment
   Supabase stores: user_id → node_endpoint mapping

   Optional: Syself Autopilot for managed k3s on Hetzner
```

### 7.3 Cost Table

> **Note:** Hetzner increased prices up to 37% on some plans in April 2026. The costs below should be verified against [hetzner.com/cloud](https://www.hetzner.com/cloud) before finalizing budgets.

| Stage | Total Users | Concurrent | Servers | Monthly Cost (estimate) |
|-------|------------|-----------|---------|------------------------|
| Launch | 0–300 | ~45 | 1× CX32 (8GB) | ~€20-25/mo |
| Growth | 300–1,000 | ~150 | 1× CX42 (16GB) | ~€35-45/mo |
| Scale | 1K–5K | ~750 | 2× CX42 + LB | ~€80-100/mo |
| Expand | 5K–10K | ~1,500 | 4× CX42 + LB | ~€170-190/mo |
| Big | 10K+ | 1,500+ | k3s cluster (5+ nodes) | ~€250+/mo |

### 7.4 Why This Scaling Order

**Phase 1 (Docker Compose)** — Docker Compose on a single Hetzner VPS. You already know Docker. No new tools to learn. Handles 1,000 users easily. Same `docker-compose.yml` from development to production.

**Phase 2 (k3s)** — When one VPS isn't enough, k3s is a single ~100MB binary that implements the full Kubernetes API. It works with the same Docker container images and can start as a single-node "cluster" before adding workers. Key advantages over Docker Swarm:
- **Horizontal Pod Autoscaling (HPA):** automatically scales container count based on CPU/memory metrics — Swarm has no equivalent
- **Secrets management:** integrates with HashiCorp Vault, sealed-secrets, and external KMS — Swarm secrets are basic with no rotation
- **Kubernetes operators:** production-ready patterns for databases, monitoring, and certificate management — Swarm has no operator ecosystem
- **Growing ecosystem:** CNCF-certified, actively developed, massive community — Swarm is in maintenance mode with declining adoption
- **Portainer Dashboard:** visual management for teams who prefer a UI over kubectl

Adding a k3s agent node to the cluster is a single command: `k3s agent --server https://server:6443 --token <token>`.

---

## 8. Request Flow (End to End)

> **IMPORTANT — Async Pattern:** Supabase Edge Functions have a **2-second CPU time limit**. They must NOT be used as synchronous orchestrators. The Edge Function acts as a **thin auth proxy** that validates the user and fires-and-forgets to the Gateway. The Gateway handles all orchestration and pushes responses back asynchronously via Supabase Realtime.

```
1. User taps "Send" in mobile app
   │
2. Expo app sends HTTPS POST to Supabase Edge Function
   │  Headers: Authorization: Bearer {supabase_jwt}
   │  Body: { message: "Summarize my emails today" }
   │
3. Edge Function (THIN AUTH PROXY — must complete in <2s CPU):
   │  a. Validates JWT token (Supabase Auth)
   │  b. Quick subscription tier lookup (cached or single DB read)
   │  c. Fire-and-forget POST to Gateway (do NOT await response):
   │     URL: https://agent.zeroclaw.app/api/chat
   │     Headers: Authorization: Bearer {gateway_secret}
   │     Body: { user_id: "abc123", message: "...", tier: "pro" }
   │  d. Returns 202 Accepted immediately to the mobile app
   │     { "status": "processing", "task_id": "task-xyz" }
   │
4. Gateway (ASYNC ORCHESTRATOR — no CPU time limits):
   │  a. Validates gateway_secret
   │  b. Calls ensureContainer("abc123")
   │     - Checks Redis for running container
   │     - If not running: docker start or docker create + start
   │  c. Routes request: POST http://zc-abc123:8080/chat
   │  d. Updates last_active timestamp in Redis
   │  e. Logs task to Supabase tasks table via Supabase client
   │
5. ZeroClaw Container (zc-abc123):
   │  a. Loads user config from /agent/config.toml
   │  b. Loads conversation context from /agent/memory.db
   │  c. Determines tools needed (email = Composio Gmail tool)
   │  d. Calls LLM API with context + tools (model selected by routing layer)
   │  e. Executes tool: Composio Gmail → reads inbox
   │  f. LLM synthesizes email summary
   │  g. Saves to memory.db
   │  h. Returns response JSON to Gateway
   │
6. Gateway pushes response directly:
   │  a. Writes response to Supabase Realtime channel (user-specific)
   │  b. Updates task status in tasks table
   │  c. Optionally streams response chunks for incremental rendering
   │
7. Mobile app receives response via Supabase Realtime (WebSocket)
   │  Displays email summary in chat UI
   │
   │  Note: The app subscribes to a Realtime channel on send
   │  and renders responses as they arrive — no polling needed.
```

**Why this pattern:**
- Edge Functions have a 2-second CPU limit — waiting for LLM + tool execution would timeout
- The Gateway has no CPU time limits and can handle long-running agent tasks (30-120s for research)
- Supabase Realtime handles 10,000+ concurrent WebSocket connections
- The mobile app gets an immediate 202 Accepted and renders responses as they stream in
- Task logging moves to the Gateway, keeping the Edge Function minimal

---

## 9. Backup & Disaster Recovery

### 9.1 What to Back Up

| Data | Location | Strategy | Frequency |
|------|----------|----------|-----------|
| User volumes | `/data/users/` | Incremental rsync to Hetzner Storage Box | Every 6 hours |
| Supabase PostgreSQL | Supabase Cloud | Automatic daily backups (included in plan) | Daily |
| Redis state | In-memory (ephemeral) | Reconstructable from Docker inspect | Not backed up |
| Docker images | Docker Hub / private registry | Versioned tags, immutable | On each release |
| Compose + config | Git repository | Version controlled | On each commit |

### 9.2 Recovery Procedure

```
1. Provision new Hetzner VPS
2. Install Docker, pull images
3. Restore /data/users/ from Storage Box backup
4. docker compose up -d
5. Gateway auto-creates containers on next user request
6. All user data, memory, skills intact from restored volumes
```

Recovery time: ~15 minutes for a full server replacement.

---

## 10. Monitoring & Observability

### 10.1 Key Metrics

| Metric | Source | Alert Threshold |
|--------|--------|----------------|
| Running container count | Docker API | > 80% of MAX_CONTAINERS |
| Host RAM usage | Node exporter | > 85% |
| Host CPU usage | Node exporter | > 90% sustained 5min |
| Container cold start time | Gateway logs | > 5 seconds |
| Gateway response time (p95) | Gateway logs | > 3 seconds |
| Failed container starts | Gateway logs | > 5 per hour |
| Idle cleanup rate | Gateway logs | Informational |
| Disk usage (/data) | Node exporter | > 80% |

### 10.2 Monitoring Stack

For Phase 1 (simple), use Hetzner's built-in monitoring + a lightweight approach:

```
Gateway → structured JSON logs → Loki (or just journald)
Docker → container metrics → Prometheus (node-exporter + cAdvisor)
Alerts → Grafana or simple webhook to Telegram/Slack
```

---

## 11. Technology Reference

| Component | Technology | Version | License | Link |
|-----------|-----------|---------|---------|------|
| Agent Runtime | ZeroClaw | v0.1.6 | Apache-2.0/MIT | [github.com/zeroclaw-labs/zeroclaw](https://github.com/zeroclaw-labs/zeroclaw) |
| Web Scraping | Scrapling | v0.4 | BSD-3-Clause | [github.com/D4Vinci/Scrapling](https://github.com/D4Vinci/Scrapling) |
| Mobile App | Expo SDK 54 | 54.x | MIT | [expo.dev](https://expo.dev) |
| Boilerplate | expo-supabase-ai-template | latest | MIT | [github.com/abdulhaseeb7603/expo-supabase-ai-template](https://github.com/abdulhaseeb7603/expo-supabase-ai-template) |
| Backend | Supabase | Cloud | Apache-2.0 | [supabase.com](https://supabase.com) |
| LLM | MiniMax M2.5 | latest | Proprietary | [minimax.io](https://minimax.io) |
| Integrations | Composio | latest | Proprietary | [composio.dev](https://composio.dev) |
| Payments | RevenueCat | latest | Proprietary | [revenuecat.com](https://revenuecat.com) |
| Reverse Proxy | Caddy | 2.x | Apache-2.0 | [caddyserver.com](https://caddyserver.com) |
| Socket Proxy | docker-socket-proxy | latest | Apache-2.0 | [github.com/Tecnativa/docker-socket-proxy](https://github.com/Tecnativa/docker-socket-proxy) |
| Voice STT | Deepgram | latest | Proprietary | [deepgram.com](https://deepgram.com) |
| Voice TTS | ElevenLabs | latest | Proprietary | [elevenlabs.io](https://elevenlabs.io) |

---

*— End of Architecture Document —*
