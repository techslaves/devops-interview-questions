# Docker Commands Interview Questions

This document contains common interview questions related to Docker commands and their usage.

### 1. List running containers and show resource usage
**Question:**
How do you list running containers and check CPU/memory usage?

**Commands:**
```shell
docker ps
docker stats
```
**🎯 Interview Tip:** `docker stats` reads cgroup metrics in real time.

### 2. View logs of a container
**Question:**
How do you check container logs and follow them?

**Commands:**
```shell
docker logs <container>
docker logs -f <container>           # Follow logs
docker logs --since 10m <container>  # Logs from last 10 minutes
```

### 3. Exec into a running container
**Question:**
How do you access a running container?

**Commands:**
```shell
docker exec -it <container> /bin/sh
docker exec -it <container> /bin/bash
```
**📌 Note:** Distroless images won’t have a shell, so this command will fail.

### 4. Find why a container stopped
**Question:**
How do you check why a container exited?

**Commands:**
```shell
docker ps -a
docker inspect <container>
```
**Look for:** `ExitCode`, `OOMKilled`, `Error`.

### 5. Stop, start, restart containers
**Commands:**
```shell
docker stop <container>
docker start <container>
docker restart <container>
```

### 6. Remove stopped containers and unused resources
**Question:**
How do you clean up Docker resources?

**Commands:**
```shell
docker container prune
docker image prune
docker volume prune
docker system prune
```
**⚠️ Interview trap:** `docker system prune -a` removes **all** unused images, not just dangling ones.

### 7. Build Docker image
**Question:**
How do you build an image and tag it?

**Command:**
```shell
docker build -t myapp:v1 .
```

### 8. Run a container with port mapping
**Question:**
Expose container port 8080 to host 80.

**Command:**
```shell
docker run -d -p 80:8080 myapp
```

### 9. Run container with environment variables
**Command:**
```shell
docker run -e ENV=prod -e DEBUG=false myapp
```
**Or from file:**
```shell
docker run --env-file .env myapp
```

### 10. Limit CPU and memory
**Question:**
How do you restrict container resources?

**Command:**
```shell
docker run --cpus="1.5" --memory="512m" myapp
```

### 11. Check image layers and history
**Question:**
How do you see how an image was built?

**Commands:**
```shell
docker history myapp
docker inspect myapp
```

### 12. Create and inspect Docker networks
**Commands:**
```shell
docker network ls
docker network create mynet
docker network inspect mynet
```
**Attach container:**
```shell
docker run --network mynet nginx
```

### 13. Connect containers to same network
**Command:**
```shell
docker network connect mynet container1
```
**Note:** Containers on the same user-defined network can resolve each other by name.

### 14. Docker volumes commands
**Commands:**
```shell
docker volume create myvol
docker volume ls
docker volume inspect myvol
```
**Run with volume:**
```shell
docker run -v myvol:/data myapp
```

### 15. Copy files between host and container
**Commands:**
```shell
docker cp file.txt container:/tmp/
docker cp container:/tmp/file.txt .
```

### 16. Tag and push image to registry
**Commands:**
```shell
docker tag myapp:v1 repo/myapp:v1
docker push repo/myapp:v1
```

### 17. Login to Docker registry
**Command:**
```shell
docker login
# Or specific registry
docker login registry.example.com
```

### 18. Save and load Docker images
**Question:**
How do you move images without a registry?

**Commands:**
```shell
docker save myapp > myapp.tar
docker load < myapp.tar
```

### 19. Run container as non-root user
**Command:**
```shell
docker run --user 1001:1001 myapp
```

### 20. View Docker disk usage
**Command:**
```shell
docker system df
```

### 21. Inspect container environment variables
**Command:**
```shell
docker inspect <container> | grep -i env
```

### 22. Debug networking inside container
**Commands:**
```shell
docker exec -it <container> ip addr
docker exec -it <container> netstat -tuln
```

### 23. Set restart policy
**Command:**
```shell
docker run --restart unless-stopped myapp
```

### 24. Build multi-arch images
**Command:**
```shell
docker buildx build --platform linux/amd64,linux/arm64 .
```

### 25. Docker healthcheck
**Command:**
```shell
docker inspect --format='{{.State.Health.Status}}' <container>
```

---

### ⚠️ Common Interview Traps

**❓ Difference between `docker stop` and `docker kill`?**
*   ✔️ `stop` sends `SIGTERM` → graceful shutdown.
*   ✔️ `kill` sends `SIGKILL` → force kill.

**❓ Why `docker exec` fails on distroless images?**
*   ✔️ No shell (`/bin/sh` or `/bin/bash`) is present in the image.
