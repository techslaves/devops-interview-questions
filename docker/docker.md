# Docker Interview Questions and Answers

This document contains common interview questions related to Docker and their answers.

### 1. Explain Docker architecture in detail.
**Answer:**
Docker follows a client–server architecture:
*   **Docker Client:** CLI (`docker build`, `docker run`) that communicates with the daemon.
*   **Docker Daemon (`dockerd`):** Manages images, containers, networks, and volumes.
*   **containerd:** Handles container lifecycle (pulling images, starting/stopping containers).
*   **runc:** Creates containers using Linux namespaces & cgroups.
*   **OCI runtime spec:** Ensures portability.

**Key internals:**
*   **Namespaces:** Provide isolation (PID, NET, MNT, etc.).
*   **Cgroups:** Enforce resource limits (CPU, Memory).
*   **Union FS (overlay2):** Manages layered images.

### 2. Difference between Docker Engine, containerd, and runc?
**Answer:**

| Component | Responsibility |
| :--- | :--- |
| **Docker Engine** | High-level API, UX, and orchestration. |
| **containerd** | Container lifecycle management (image transfer, storage, execution). |
| **runc** | Low-level container execution (spawns and runs containers). |

### 3. How does Docker achieve process isolation?
**Answer:**
Docker uses Linux Namespaces to isolate resources:
*   **PID namespace:** Process isolation (container sees only its own processes).
*   **NET namespace:** Network stack isolation (own IP, ports, routing table).
*   **MNT namespace:** Filesystem isolation (mount points).
*   **UTS namespace:** Hostname isolation.
*   **IPC namespace:** Shared memory isolation.
*   **User namespace:** UID/GID mapping (root in container != root on host).

### 4. How do Docker image layers work?
**Answer:**
*   Each instruction in a `Dockerfile` (e.g., `RUN`, `COPY`) creates a read-only layer.
*   When a container starts, Docker adds a thin **writable layer** on top.
*   **OverlayFS** (or similar UnionFS) merges these layers at runtime, presenting a unified view.

### 5. How to reduce Docker image size?
**Answer:**
**Techniques:**
*   **Multi-stage builds:** Compile in one stage, copy only artifacts to a minimal runtime stage.
*   **Distroless or Alpine base images:** Use minimal OS images.
*   **Combine RUN commands:** Reduces the number of layers (e.g., `apt-get update && apt-get install ... && rm -rf /var/lib/apt/lists/*`).
*   **Remove build tools:** Don't include compilers in the final image.
*   **`.dockerignore`:** Exclude unnecessary files from the build context.

### 6. What is the difference between COPY and ADD?
**Answer:**

| Feature | COPY | ADD |
| :--- | :--- | :--- |
| **Function** | Simple file copy from host to image. | Supports URL download & auto-extraction of tarballs. |
| **Recommendation** | **Preferred** (Explicit and predictable). | Avoid unless specifically needed (e.g., extracting a local tar). |

### 7. Why is multi-stage build important?
**Answer:**
*   **Separation:** Separates build-time dependencies (Maven, GCC) from runtime dependencies (JRE, Libs).
*   **Security:** Reduces attack surface by removing build tools and source code from the final image.
*   **Size:** Significantly smaller final image size.

### 8. Explain Docker networking drivers.
**Answer:**

| Driver | Use Case |
| :--- | :--- |
| **bridge** | Default for single-host networking. Containers talk via IP/NAT. |
| **host** | Removes network isolation. Container shares host's network stack. High performance. |
| **overlay** | Multi-host networking (Swarm/Kubernetes). |
| **macvlan** | Assigns a MAC address to the container, making it appear as a physical device on the network. |
| **none** | Fully isolated networking (no network interface). |

### 9. How does Docker DNS work?
**Answer:**
*   Docker runs an embedded DNS server at `127.0.0.11`.
*   Container names resolve automatically to their internal IPs within the same user-defined network.
*   DNS resolution is scoped per network.

### 10. Difference between bridge vs host networking?
**Answer:**

| Feature | Bridge | Host |
| :--- | :--- | :--- |
| **Isolation** | NAT-based isolation. | No NAT, shares host IP/Ports. |
| **Security** | Safer (ports must be exposed explicitly). | Less secure (all ports exposed). |
| **Performance** | Slight overhead due to NAT. | Best performance (native speed). |

### 11. Difference between volume, bind mount, tmpfs?
**Answer:**

| Type | Persistence | Use Case |
| :--- | :--- | :--- |
| **Volume** | Managed by Docker (`/var/lib/docker/volumes`). | Production data, backups, sharing between containers. |
| **Bind mount** | Host filesystem path. | Development (live code reloading), config files. |
| **tmpfs** | Memory (RAM). | Secrets, temporary data, security (never written to disk). |

### 12. Why are volumes preferred in production?
**Answer:**
*   **Managed:** Independent of container lifecycle and host directory structure.
*   **Portable:** Easier to back up, migrate, and manage via CLI.
*   **Performance:** Optimized for container I/O (especially on non-Linux hosts).

### 13. What are Docker security best practices?
**Answer:**
*   **Non-root user:** Run applications as a non-root user inside the container.
*   **User Namespaces:** Map container root to a non-privileged user on the host.
*   **Image Scanning:** Use tools like Trivy, Grype, or Clair to scan for vulnerabilities.
*   **Read-only filesystem:** Run containers with `--read-only` where possible.
*   **Drop Capabilities:** Drop unnecessary Linux capabilities (`--cap-drop ALL`).
*   **Seccomp & AppArmor:** Use security profiles to restrict syscalls.

### 14. How does Docker use Linux capabilities?
**Answer:**
Instead of giving a container full root access (which is dangerous), Docker allows granting only specific privileges.
**Example:**
```shell
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE nginx
```

### 15. What is a distroless image?
**Answer:**
*   **Definition:** An image containing only the application and its runtime dependencies.
*   **Features:** No shell, no package manager, no standard linux utilities.
*   **Benefit:** Minimal attack surface and very small size.

### 16. How does Docker enforce resource limits?
**Answer:**
Docker uses Linux **Cgroups (Control Groups)**:
*   **CPU:** `--cpus`, `--cpu-shares`.
*   **Memory:** `--memory` (OOM killer triggers if exceeded).
*   **IO:** `blkio` cgroups for disk throughput limits.

### 17. What happens when a container exceeds memory?
**Answer:**
The Linux kernel **OOM Killer (Out-Of-Memory Killer)** terminates the container process.
*   **Exit Code:** `137` (128 + 9 for SIGKILL).

### 18. How to debug high CPU usage in a container?
**Answer:**
*   `docker stats`: Real-time resource usage.
*   `docker top`: View running processes inside the container.
*   `perf`, `strace`: Advanced profiling tools (requires capabilities).
*   Check Cgroup metrics on the host.

### 19. Difference between ENTRYPOINT and CMD?
**Answer:**

| Feature | ENTRYPOINT | CMD |
| :--- | :--- | :--- |
| **Purpose** | Fixed executable. | Default arguments. |
| **Override** | Hard to override (requires `--entrypoint`). | Easily overridden by passing args to `docker run`. |

**Best Practice:**
```dockerfile
ENTRYPOINT ["java"]
CMD ["-jar", "app.jar"]
```

### 20. Why should containers run one process?
**Answer:**
*   **Lifecycle:** The container lives and dies with the main process (PID 1).
*   **Logs:** `stdout`/`stderr` are easily captured.
*   **Scaling:** Easier to scale individual components.
*   **Restart:** Orchestrators can restart failed processes cleanly.

### 21. Why is Docker alone not enough for production?
**Answer:**
Docker manages single containers, but lacks:
*   **Auto-healing:** Restarting failed containers automatically.
*   **Autoscaling:** Scaling based on load.
*   **Rolling updates:** Zero-downtime deployments.
*   **Service discovery:** Finding services across multiple nodes.
*   **Solution:** Use orchestrators like Kubernetes, ECS, or Nomad.

### 22. Difference between Docker Swarm and Kubernetes?
**Answer:**

| Feature | Docker Swarm | Kubernetes |
| :--- | :--- | :--- |
| **Complexity** | Simple, easy to setup. | Complex, steep learning curve. |
| **Features** | Basic orchestration. | Rich ecosystem, advanced scheduling. |
| **Adoption** | Limited, declining. | Industry standard. |
| **Philosophy** | Docker-centric. | Cloud-native, API-driven. |

### 23. Container keeps restarting in production. How will you debug?
**Answer:**
I first check container logs and exit code. If it’s 137, it’s memory-related. If the app exits immediately, I verify ENTRYPOINT/CMD. I also check healthcheck configuration and dependency readiness.

**Commands:**
```shell
docker ps -a
docker logs <container>
docker inspect <container>
```

**Common Causes:**
*   Application crash (code error).
*   Wrong `ENTRYPOINT` / `CMD`.
*   Healthcheck failure.
*   OOMKilled (Exit Code 137).
*   Missing dependency (DB not ready).

### 24. Docker image is too large (2GB+). How will you fix?
**Answer:**
*   **Multi-stage builds:** Discard build artifacts.
*   **Base Image:** Switch to `distroless` or `alpine`.
*   **Cleanup:** Remove build tools and cache (`rm -rf /var/cache/apt`).
*   **Context:** Use `.dockerignore` to exclude files.

### 25. Need zero-downtime deployment using Docker. How can we achieve this?
**Answer:**
Docker alone can’t do zero-downtime easily. I use an orchestrator or a reverse proxy to gradually shift traffic.
**Solutions:**
*   **Blue-Green Deployment:** Spin up new version (Green), switch traffic, kill old (Blue).
*   **Rolling Update:** Supported natively by Swarm/Kubernetes.
*   **Reverse Proxy:** Use Nginx/Traefik to reload config dynamically.

### 26. High CPU usage in a container.
**Answer:**
I identify the process, add CPU limits, and profile the app. Containers without limits can starve the host.
```shell
docker stats
docker top <container_id>
```

### 27. Container cannot resolve DNS names.
**Answer:**
Docker provides built-in DNS per network. Issues usually arise from wrong network attachment or DNS override.
*   **Check:** Is the container on the correct user-defined network?
*   **Internal DNS:** `127.0.0.11`.
*   **Override:** Check `/etc/resolv.conf` or `--dns` flags.

### 28. Logs are filling disk space.
**Answer:**
I configure log rotation or stream logs to ELK/CloudWatch instead of storing them locally.
**Fix (daemon.json):**
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

### 29. Database container lost data after restart.
**Answer:**
Containers are ephemeral. Persistent data must be stored in Docker volumes or external storage.
**Fix:**
```shell
docker volume create db-data
docker run -v db-data:/var/lib/mysql ...
```

### 30. Docker host disk fills unexpectedly.
**Answer:**
Unused images, volumes, and containers accumulate. I automate pruning carefully.

**Check Usage:**
```shell
docker system df
```
**Cleanup:**
```shell
docker system prune
# WARNING: Removes stopped containers, unused networks, dangling images.
```

### 31. Multi-architecture image support needed.
**Answer:**
I use Docker Buildx to create multi-arch images and push a single manifest list.
```shell
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest . --push
```

### 32. What is the difference between docker stats and docker top? 
docker stats is used to monitor real-time container resource usage using cgroups, while docker top helps inspect the processes running inside a container using the PID namespace. Both are complementary tools used together for troubleshooting.

| Feature           | docker stats   | docker top    |
| ----------------- | -------------- | ------------- |
| Shows             | Resource usage | Processes     |
| CPU usage         | ✅ Yes          | ❌ No          |
| Memory usage      | ✅ Yes          | ❌ No          |
| Network / Disk IO | ✅ Yes          | ❌ No          |
| Process details   | ❌ No           | ✅ Yes         |
| Uses              | cgroups        | PID namespace |
| Real-time         | ✅ Yes          | Snapshot      |
