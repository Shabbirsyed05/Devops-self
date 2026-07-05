# Docker Interview Guide

> **Source:** [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `07-docker/` folder
> **Purpose:** Master Docker concepts — containers, images, volumes, networking, and debugging for interviews

---

## Table of Contents

| # | Topic |
|---|-------|
| [1](#1-container-exits-immediately) | Container Exits Immediately |
| [2](#2-purpose-of-expose-in-dockerfile) | Purpose of `EXPOSE` in Dockerfile |
| [3](#3-port-not-accessible-on-localhost) | Port Not Accessible on Localhost |
| [4](#4-data-lost-on-container-restart) | Data Lost on Container Restart |
| [5](#5-code-changes-not-reflected-in-new-image) | Code Changes Not Reflected in New Image |
| [6](#6-app-crashes-with-permission-denied) | App Crashes with Permission Denied |
| [7](#7-host-ran-out-of-disk-space) | Host Ran Out of Disk Space |
| [8](#8-debug-a-live-container) | Debug a Live Container |
| [9](#9-container-registry) | Container Registry |
| [10](#10-cmd-vs-entrypoint) | CMD vs ENTRYPOINT |
| [11](#11-docker-commands) | Docker Commands |
| [12](#12-forcefully-remove-a-container) | Forcefully Remove a Container |

---

## 1. Container Exits Immediately

> **Q: Docker container exits immediately — how will you troubleshoot?**

### Answer

Inspect container logs, verify the Dockerfile `CMD`/`ENTRYPOINT`, and confirm the container runs a **long-lived foreground process**. Containers exit when their main process finishes or crashes.

---

### Step-by-Step Troubleshooting

#### Step 1 — Check container logs

```bash
docker logs <container_id_or_name>
```

#### Step 2 — Inspect the Dockerfile CMD

```dockerfile
CMD ["python", "app.py"]
```

> If `app.py` exits immediately, the container will too.

#### Step 3 — Run in interactive mode

```bash
docker run -it <image> /bin/bash
```

#### Step 4 — Override CMD/ENTRYPOINT temporarily

```bash
docker run -it --entrypoint /bin/sh <image>
```

#### Step 5 — Check the exit code

```bash
docker inspect <container_id> --format='{{.State.ExitCode}}'
```

| Exit Code | Meaning |
|---|---|
| `0` | Process exited cleanly (no crash, but short-lived) |
| `1` | Application error |
| `137` | Killed (OOM or `SIGKILL`) |
| `126` | Permission denied on CMD |
| `127` | Command not found |

> **Key takeaway:** A Docker container must run a **long-running foreground process**. If it finishes or crashes, the container exits. Use `docker logs`, `-it` mode, and inspect `CMD`/`ENTRYPOINT` to debug.

---

## 2. Purpose of `EXPOSE` in Dockerfile

> **Q: What is the purpose of the `EXPOSE` instruction in a Dockerfile?**

### Answer

`EXPOSE` indicates which port the containerized application listens on at runtime. It acts as **documentation** and a **signal** to tools like Docker Compose — but it does **not** actually publish the port to the host.

To make a port accessible on the host, use the `-p` flag at runtime:

```bash
docker run -p 8080:80 myapp
```

### Example

```dockerfile
FROM nginx:alpine
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

| Instruction | Effect |
|---|---|
| `EXPOSE 80` | Documents the port; used by Docker Compose and orchestration tools |
| `docker run -p 8080:80` | Actually publishes port 80 inside → 8080 on host |

> **Key takeaway:** `EXPOSE` is **documentation** inside the image. It does not open ports on its own — you still need `-p` at runtime.

---

## 3. Port Not Accessible on Localhost

> **Q: Port is not accessible on `localhost` even after using the `-p` flag. How do you troubleshoot?**

### Answer

The most common cause: the app inside the container is listening on `127.0.0.1` (loopback only) instead of `0.0.0.0` (all interfaces).

### Fix — Bind to all interfaces

```python
# BAD — not accessible externally
app.run(host='127.0.0.1', port=5000)

# GOOD — accepts connections from any interface
app.run(host='0.0.0.0', port=5000)
```

### Additional Checks

```bash
# Confirm port is published
docker ps

# Check what the app is actually listening on inside the container
docker exec -it <container> netstat -tulnp

# Test connectivity from inside the container itself
docker exec -it <container> curl http://localhost:5000
```

| Check | Command |
|---|---|
| Port mapping active | `docker ps` |
| App listening interface | `netstat -tulnp` inside container |
| Firewall/security group | Check cloud console or `iptables` |
| Host port conflict | `netstat -tulnp` on the host |

> **Key takeaway:** Publishing ports with `-p` is only half the story. The app **inside** the container must listen on `0.0.0.0` to accept external traffic.

---

## 4. Data Lost on Container Restart

> **Q: Your application container loses data when it stops and restarts. How do you fix this?**

### Answer

Containers are **ephemeral by design** — the container filesystem is lost when a container is removed. Persist data by mounting a **Docker volume** or **bind mount** outside the container lifecycle.

### Option 1 — Docker Volumes (recommended)

```bash
# Create a named volume
docker volume create mydata

# Mount it at runtime
docker run -v mydata:/app/data myapp
```

### Option 2 — Bind Mounts

```bash
# Mount a host directory into the container
docker run -v /host/data:/app/data myapp
```

### Verify Persistence

```bash
docker stop myapp
docker rm myapp
docker run -v mydata:/app/data myapp
# Data is still there
```

| Method | Best For |
|---|---|
| **Named Volume** | Databases, app state — managed by Docker |
| **Bind Mount** | Config files, source code during development |

> **Key takeaway:** Use Docker volumes or bind mounts for databases, logs, and uploaded files. Without them, all data is lost when the container is removed.

---

## 5. Code Changes Not Reflected in New Image

> **Q: You made a change in your code, rebuilt the image, but the container still shows old behavior. What could be the issue?**

### Answer

Usually caused by **Docker build cache** serving a stale layer, or a **volume mount overwriting** the updated image content.

### Cause 1 — Stale build cache

```bash
# Force a clean rebuild ignoring all cache
docker build --no-cache -t myapp:latest .
```

Use correct layer ordering in your Dockerfile — put rarely changing steps first:

```dockerfile
# GOOD: install dependencies first, then copy app code
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

### Cause 2 — Volume masking image content

```bash
# If you mount the local directory, the container sees LOCAL files — not image files
docker run -v "$(pwd)":/app myapp
```

Remove the volume mount when testing the built image, or verify what's inside:

```bash
docker run -it myapp:latest cat /app/main.py
```

| Root Cause | Fix |
|---|---|
| Docker caching old layers | `docker build --no-cache` |
| Volume overriding image content | Remove bind mount; verify with `cat` inside container |
| Running old container | `docker rm` old container, re-run with new image |

> **Key takeaway:** If Docker changes aren't reflected, check for caching in `docker build` and ensure no local volume is masking your image content.

---

## 6. App Crashes with Permission Denied

> **Q: App runs fine locally but crashes with "Permission Denied" inside a Docker container. What could be the issue?**

### Answer

### Cause 1 — Script not executable

```dockerfile
COPY start.sh /app/start.sh
RUN chmod +x /app/start.sh
```

### Cause 2 — Running as non-root user without proper ownership

```dockerfile
RUN useradd -m appuser
RUN chown -R appuser:appuser /app
USER appuser
```

### Cause 3 — Volume mount with restrictive host permissions

```bash
# Fix permissions on the host directory before mounting
chmod -R 755 ./data/
```

### Debugging Commands

```bash
# Check file permissions inside the container
docker exec -it <container> ls -la /app

# Drop into a shell as root to investigate
docker run -it --user root myapp /bin/sh
```

> **Key takeaway:** Always check file permissions and user context in your Dockerfile. The container user must have read/execute access to the files it needs.

---

## 7. Host Ran Out of Disk Space

> **Q: Your Docker host is running out of disk space. How do you clean up?**

### Answer

### Step 1 — Understand what is using space

```bash
docker system df
```

### Step 2 — Quick cleanup

```bash
# Remove stopped containers, unused networks, dangling images
docker system prune

# Deep cleanup — also removes all unused images and volumes
docker system prune -a --volumes
```

### Step 3 — Targeted cleanup

```bash
# Remove dangling (untagged) images only
docker image prune

# Remove unused volumes only
docker volume prune

# Remove unused networks only
docker network prune
```

### Step 4 — Investigate the Docker storage directory

```bash
sudo du -sh /var/lib/docker/*
```

### Prevention

| Practice | Detail |
|---|---|
| Scheduled prune | Cron job: `docker system prune -a --volumes -f` |
| Smaller base images | Use `alpine` or `distroless` variants |
| Clean up in Dockerfile | `RUN apt-get install -y pkg && rm -rf /var/lib/apt/lists/*` |
| Image lifecycle policy | Use ECR/ACR lifecycle rules to auto-delete old images |

> **Key takeaway:** Docker does **not** auto-clean. Regularly prune stopped containers, dangling images, and unused volumes.

---

## 8. Debug a Live Container

> **Q: Your application is running inside a Docker container but showing abnormal behavior. How would you debug it without stopping it?**

### Answer

### Method 1 — `docker exec` — interactive shell access

```bash
# Open a shell inside the running container
docker exec -it <container-id> /bin/sh

# Run specific commands
docker exec -it myapp cat /var/log/app.log
docker exec -it myapp env
```

### Method 2 — `docker logs` — view stdout/stderr

```bash
# Stream live logs
docker logs -f <container-id>

# View last 100 lines
docker logs --tail=100 <container-id>
```

### Method 3 — `docker inspect` — metadata and config

```bash
docker inspect <container-id>
docker inspect -f '{{.Config.Env}}' <container-id>
docker inspect -f '{{.NetworkSettings.IPAddress}}' <container-id>
```

### Method 4 — `docker top` — running processes

```bash
docker top <container-id>
```

### Method 5 — Network diagnostics from inside

```bash
docker exec -it <container-id> netstat -tulnp
docker exec -it <container-id> curl http://localhost:<port>
```

### Method 6 — `docker attach` (use with caution)

```bash
docker attach <container-id>
# Detach safely with: Ctrl+P then Ctrl+Q
```

| Tool | Best For |
|---|---|
| `docker exec` | Interactive debugging, file inspection |
| `docker logs` | App stdout/stderr output |
| `docker inspect` | Config, env vars, network settings |
| `docker top` | Process list inside the container |
| `docker attach` | Viewing live stdin/stdout (use carefully) |

> **Key takeaway:** Use `exec` for shell access, `logs` for output, `inspect` for config, and `top` for processes — all **without stopping the container**.

---

## 9. Container Registry

> **Q: Which container registry does your organization use?**

### Answer

We primarily use **Amazon Elastic Container Registry (ECR)**.

### Amazon ECR — Key Features

| Feature | Detail |
|---|---|
| **Managed** | Fully managed by AWS — no registry infrastructure to maintain |
| **IAM integration** | Fine-grained access control via AWS IAM policies |
| **Image scanning** | Built-in vulnerability scanning on push |
| **Lifecycle policies** | Auto-delete old/untagged images to save storage costs |
| **High availability** | Regional redundancy built-in |

### ECR Workflow

```bash
# Authenticate Docker with ECR
aws ecr get-login-password --region <region> \
  | docker login --username AWS --password-stdin \
    <aws_account_id>.dkr.ecr.<region>.amazonaws.com

# Build, tag, and push
docker build -t myapp .
docker tag myapp:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/myapp:latest
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/myapp:latest
```

### Other Common Registries

| Registry | Best For |
|---|---|
| **Docker Hub** | Public images, open-source projects |
| **GitHub Container Registry (GHCR)** | Projects hosted on GitHub |
| **Azure Container Registry (ACR)** | Azure-based workloads |
| **Google Artifact Registry** | GCP workloads |
| **Harbor** | Self-hosted, enterprise on-premise |

> **Key takeaway:** Choose your registry based on cloud provider, CI/CD integration needs, and security/compliance requirements.

---

## 10. CMD vs ENTRYPOINT

> **Q: What is the difference between `CMD` and `ENTRYPOINT` in a Dockerfile?**

### Answer

### `CMD` — Default command (easily overridable)

```dockerfile
FROM ubuntu
CMD ["echo", "Hello from CMD"]
```

```bash
docker run myimage                  # Output: Hello from CMD
docker run myimage echo "Override"  # Output: Override  (CMD replaced)
```

### `ENTRYPOINT` — Fixed executable (not easily replaced)

```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
```

```bash
docker run myimage "Hello"          # Output: Hello  (appended to ENTRYPOINT)
docker run --entrypoint /bin/sh myimage   # Override requires --entrypoint flag
```

### Using `ENTRYPOINT` + `CMD` Together (best practice)

```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Hello from CMD"]
```

```bash
docker run myimage                  # Output: Hello from CMD  (CMD as default args)
docker run myimage "Custom message" # Output: Custom message  (CMD overridden)
```

### Summary

| Feature | `CMD` | `ENTRYPOINT` |
|---|---|---|
| **Purpose** | Default command or arguments | Fixed executable that always runs |
| **Overridable at runtime** | Yes — any argument after image name | Only with `--entrypoint` flag |
| **Used together** | Provides default args to `ENTRYPOINT` | Defines the base command |
| **Common use case** | Default flags, help text | App binary e.g. `nginx`, `python`, `node` |

> **Key takeaway:** Use `ENTRYPOINT` for the main executable and `CMD` for its default arguments. This gives both a fixed command and flexible defaults.

---

## 11. Docker Commands

> **Q: What Docker commands do you use on a day-to-day basis?**

### Answer

### Container Management

```bash
docker ps               # List running containers
docker ps -a            # List all containers (including stopped)
docker run -d -p 8080:80 myapp        # Run container in background
docker stop <container>               # Graceful stop
docker start <container>              # Start a stopped container
docker rm <container>                 # Remove a stopped container
docker rm -f <container>              # Force remove a running container
```

### Image Management

```bash
docker build -t myapp:latest .        # Build image from Dockerfile
docker images                         # List all local images
docker rmi <image>                    # Remove an image
docker pull nginx:alpine              # Pull image from registry
docker push myrepo/myapp:latest       # Push image to registry
docker tag myapp:latest myrepo/myapp:v1.0  # Tag an image
```

### Debugging & Inspection

```bash
docker logs -f <container>            # Stream container logs
docker exec -it <container> /bin/sh   # Open shell inside container
docker inspect <container>            # Full container metadata
docker top <container>                # Running processes inside container
docker stats                          # Live resource usage (CPU, memory)
```

### Cleanup

```bash
docker system prune -a                # Remove all unused resources
docker volume prune                   # Remove unused volumes
docker image prune                    # Remove dangling images
docker system df                      # Show disk usage
```

### Docker Compose (common daily use)

```bash
docker compose up -d                  # Start services in background
docker compose down                   # Stop and remove containers
docker compose logs -f                # Stream all service logs
docker compose ps                     # List compose services
```

> **Key takeaway:** Mastering `build`, `run`, `exec`, `logs`, `inspect`, and `prune` covers 90% of daily Docker work.

---

## 12. Forcefully Remove a Container

> **Q: Have you ever had to forcefully remove a Docker container? When and how?**

### Answer

Yes — when containers are stuck in a `dead`, `paused`, or unresponsive state and a graceful stop is not working.

### Always Try Graceful Stop First

```bash
# Check state first
docker ps -a

# Attempt graceful shutdown
docker stop <container-id>
```

### Force Remove

```bash
docker rm -f <container-id>
```

This sends `SIGKILL` immediately and then removes the container — no cleanup, no graceful shutdown hooks.

### Investigate Before Forcing

```bash
docker inspect <container-id>              # Check state and exit code
docker logs <container-id>                 # Check what it was doing
```

### Caution

| Risk | Detail |
|---|---|
| **Data loss** | Any non-persisted data is gone immediately |
| **No shutdown hooks** | Application cleanup scripts won't run |
| **Production risk** | Avoid in production unless absolutely necessary |

> **Key takeaway:** Use `docker rm -f` only when a container is stuck or unresponsive. Always try `docker stop` first, and ensure important data is stored in volumes before force-removing.

---

## Quick Reference Cheatsheet

```
=========================================================
         DOCKER INTERVIEW CHEATSHEET
=========================================================

  CONTAINER LIFECYCLE
  -------------------
  run → start → stop → rm
  docker run -d -p host:container image
  docker exec -it <id> /bin/sh
  docker rm -f <id>   (force kill + remove)

  DEBUGGING
  ---------
  docker logs -f <id>
  docker inspect <id>
  docker top <id>
  docker stats
  Exit codes: 0=clean, 1=error, 137=killed, 127=not found

  STORAGE
  -------
  Named volume:  docker run -v mydata:/app/data
  Bind mount:    docker run -v /host/path:/container/path
  Containers are ephemeral — volumes persist data

  EXPOSE vs -p
  ------------
  EXPOSE = documentation only (no port opened)
  -p 8080:80 = actually publishes port to host

  CMD vs ENTRYPOINT
  -----------------
  ENTRYPOINT = fixed executable
  CMD        = default args (overridable)
  Together:  ENTRYPOINT runs, CMD provides default args

  CLEANUP
  -------
  docker system prune -a --volumes
  docker system df    (check usage first)

=========================================================
```

---

## Interview Tips

| Do This | Not This |
|---|---|
| For "exits immediately" → check exit code + `docker logs` first | Just say "restart it" |
| For "permission denied" → check `USER` and `chmod` in Dockerfile | Blindly run as root |
| For "port not accessible" → mention app must bind `0.0.0.0` | Only mention `-p` flag |
| For `CMD` vs `ENTRYPOINT` → explain combining both with a concrete example | Give just a one-line definition |
| For "data lost" → explain volumes vs bind mounts with use cases | Just say "use a volume" |
| For "disk space" → run `docker system df` first, then prune selectively | Jump straight to `prune -a` |

---

## Resources

- [Docker Official Documentation](https://docs.docker.com/)
- [Dockerfile Reference](https://docs.docker.com/reference/dockerfile/)
- [Docker Volumes](https://docs.docker.com/engine/storage/volumes/)
- [Amazon ECR Documentation](https://docs.aws.amazon.com/ecr/)
- [Original Source: devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide)

---

> Star this repo if it helped you prepare for your DevOps interview!
> Drop the next topic's raw notes and they will be formatted and added here.
