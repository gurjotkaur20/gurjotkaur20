# Docker vs containerd: Complete Guide

## Quick Overview

**Docker:** Complete container platform (build, run, manage, CLI)

**containerd:** Lightweight container runtime (run containers only)

**Key Point:** Docker now uses containerd internally. Kubernetes uses containerd directly.

---

## Evolution of Docker

### Initial Architecture (Pre-2015)

```
Docker CLI → Docker Engine (dockerd) → Linux Kernel
```

Docker Engine handled everything:
- Building images
- Pulling images
- Running containers
- Networking
- Storage
- Logging

### Current Architecture (Post-2015)

```
Docker CLI
    │
    ▼
Docker Engine (dockerd)
    │
    ▼
containerd
    │
    ▼
runc
    │
    ▼
Linux Kernel (Namespaces + Cgroups)
```

**Why the split?**
- Docker was too monolithic
- Separated concerns (build vs run)
- containerd became reusable by other projects
- Kubernetes moved to containerd directly

---

## What is Docker?

**Docker** is a complete developer platform for:

```
✅ Building container images (Dockerfile → Image)
✅ Pushing/pulling images to registries
✅ Running containers (docker run)
✅ Managing networks (docker network)
✅ Managing volumes (docker volume)
✅ Orchestration tools (Docker Compose)
✅ User-friendly CLI (docker ps, docker logs, etc.)
```

**Example Workflow:**
```bash
docker build -t myapp:1.0 .       # Build image
docker push myapp:1.0             # Push to registry
docker run -d myapp:1.0           # Run container
docker ps                         # List containers
docker logs <container-id>        # View logs
docker exec <id> bash             # Execute in container
```

---

## What is containerd?

**containerd** is an industry-standard, lightweight container runtime focused solely on:

```
✅ Pulling images from registries
✅ Storing and managing images
✅ Managing image layers (snapshots)
✅ Starting containers
✅ Stopping containers
✅ Managing container lifecycle
```

**What it does NOT provide:**
```
❌ Dockerfile builds
❌ Docker Compose
❌ Docker CLI (high-level)
❌ Networking management
❌ Volume management
❌ Developer tooling
```

**Focus:** Run containers efficiently and nothing else.

---

## Architecture Comparison

### Running a Container with Docker

```
User: docker run nginx

     ↓

Docker CLI
(Parses command, validates input)

     ↓

dockerd (Docker Engine)
(Manages containers, coordinates operations)

     ↓

containerd
(Pulls image, prepares filesystem, manages snapshots)

     ↓

runc
(Creates namespaces, cgroups, chroot)

     ↓

Linux Kernel
(Allocates resources, enforces isolation)

     ↓

Container runs
```

**Key Point:** Docker itself doesn't run containers; it orchestrates containerd, which orchestrates runc.

---

## Kubernetes and Container Runtimes

### Old Kubernetes (Before 1.20)

```
Kubernetes (kubelet)
     ↓
Docker (via Dockershim)
     ↓
containerd
     ↓
runc
     ↓
Linux Kernel
```

**Problem:** Kubernetes had to maintain Dockershim (compatibility layer).

### Modern Kubernetes (1.24+)

```
Kubernetes (kubelet)
     ↓
CRI (Container Runtime Interface)
     ↓
containerd (or CRI-O)
     ↓
runc
     ↓
Linux Kernel
```

**Benefits:**
- Direct communication via CRI
- No Docker dependency
- Lighter and faster
- Cleaner architecture

---

## Container Runtime Interface (CRI)

**CRI** is the standard API that Kubernetes uses to communicate with container runtimes.

```
As long as a runtime implements CRI, Kubernetes can use it.

Examples:
✅ containerd (with CRI plugin)
✅ CRI-O
✅ Docker (via cri-dockerd)
✅ Mirantis Container Runtime
```

**CRI Operations:**
```
PullImage()
CreateContainer()
StartContainer()
StopContainer()
RemoveContainer()
ListContainers()
GetContainerStats()
```

Kubernetes doesn't call Docker-specific commands; it uses these CRI methods.

---

## Docker vs containerd Comparison

| Feature | Docker | containerd |
|---------|--------|-----------|
| **Purpose** | Complete platform | Runtime only |
| **Use Case** | Developer laptops | Kubernetes, servers |
| **Builds Images** | ✅ Yes | ❌ No |
| **Manages Networks** | ✅ Yes | ❌ No |
| **Manages Volumes** | ✅ Yes | ❌ No |
| **Docker Compose** | ✅ Yes | ❌ No |
| **CLI** | ✅ docker ps, etc. | ❌ Limited (ctr, crictl) |
| **Size** | Large (~100 MB) | Small (~30 MB) |
| **Dependency Chain** | Docker → containerd → runc | containerd → runc |
| **Used by Kubernetes** | ✅ Via cri-dockerd | ✅ Directly |
| **OCI Compliant** | ✅ Yes | ✅ Yes |

---

## Why Kubernetes Removed Docker Support

**Common Misconception:** "Kubernetes removed Docker"

**Correct Statement:** "Kubernetes removed Dockershim (the compatibility layer)"

**What Actually Happened:**

```
Before (Kubernetes 1.20):
- Kubernetes used Dockershim to communicate with Docker
- Dockershim had to be maintained by Kubernetes

After (Kubernetes 1.24+):
- Kubernetes removed Dockershim
- Kubernetes now uses CRI directly
- Docker images still work (OCI standard)
```

**Why the change?**
- Simpler maintenance
- Better performance
- containerd is more efficient
- No need for intermediate layer

---

## OCI Standards Matter

**OCI** (Open Container Initiative) defines:
- **Image Format**: How images are structured
- **Runtime Spec**: How containers are executed

```
Docker Image (built with docker build)
     ↓
OCI Image Format
     ↓
containerd (any OCI runtime)
     ↓
Runs perfectly
```

**Key Insight:**

Even though Kubernetes removed Docker, Docker-built images work perfectly because they follow the OCI standard.

---

## Real-World Architecture

### Developer's Perspective

```
Developer Laptop
├─ Docker Desktop
├─ docker build (Dockerfile)
├─ docker run (testing locally)
└─ docker push (push to registry)
```

### Production Kubernetes Cluster

```
Kubernetes Node
├─ kubelet (Kubernetes agent)
├─ CRI (interface)
├─ containerd (runtime)
├─ runc (actual container executor)
└─ Linux Kernel
```

**Workflow:**
```
1. Developer: docker build → docker push
2. Production: Kubernetes deploys pod
3. kubelet: Tell containerd to run image
4. containerd: Pull image, run with runc
5. runc: Execute container in Linux kernel
```

---

## Working with containerd

### Check Images

```bash
# Using ctr (containerd CLI)
ctr images list

# Using crictl (CRI CLI - Kubernetes-friendly)
crictl images
```

### Check Running Containers

```bash
# Using ctr
ctr containers list

# Using crictl
crictl ps
```

### Run a Container (Advanced)

```bash
ctr run -d docker.io/library/nginx:latest my-nginx
```

**Note:** This is rarely done manually; Kubernetes handles it.

---

## Key Takeaways

✅ **Docker** = Platform (build, run, manage, CLI)

✅ **containerd** = Runtime (run containers only)

✅ **Docker uses containerd internally** for container execution

✅ **Kubernetes uses containerd directly** via CRI

✅ **Docker images are OCI-compliant** and work with any OCI runtime

✅ **Kubernetes removed Dockershim**, not Docker images

✅ **containerd is lighter and faster** than Docker

✅ **Both are industry standards** for containerization

---
