# Docker Interview Guide

> Source: [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `07-docker/` folder

---

# Table of Contents

1. [Container Exits Immediately](#1-container-exits-immediately)
2. [Purpose of EXPOSE in Dockerfile](#2-purpose-of-expose-in-dockerfile)
3. [Port Not Accessible on Localhost](#3-port-not-accessible-on-localhost)
4. [Data Lost on Container Restart](#4-data-lost-on-container-restart)
5. [Code Changes Not Reflected in New Image](#5-code-changes-not-reflected-in-new-image)
6. [App Crashes with Permission Denied](#6-app-crashes-with-permission-denied)
7. [Host Ran Out of Disk Space](#7-host-ran-out-of-disk-space)
8. [Debug a Live Container](#8-debug-a-live-container)
9. [Container Registry](#9-container-registry)
10. [CMD vs ENTRYPOINT](#10-cmd-vs-entrypoint)
11. [Docker Commands](#11-docker-commands)
12. [Forcefully Remove a Container](#12-forcefully-remove-a-container)

---

# 1. Container Exits Immediately

## Question
Docker container exits immediately, how will you troubleshoot?

## ✅ Answer
I would first inspect the container logs, verify the Dockerfile entrypoint or command, and check if the container runs a long-lived process. Often, containers exit if the main process completes or crashes.

### 🧪 Step-by-step Troubleshooting

#### 1. Check logs of the container
```bash
docker logs <container_id_or_name>
```

#### 2. Inspect the Dockerfile
```Dockerfile
CMD ["python", "app.py"]
```
If `app.py` exits immediately, the container will too.

#### 3. Run container in interactive mode
```bash
docker run -it <image> /bin/bash
```

#### 4. Override CMD/ENTRYPOINT temporarily
```bash
docker run -it <image> /bin/sh
```

#### 5. Check the exit status
```bash
docker inspect <container_id> --format='{{.State.ExitCode}}'
```

> **Key takeaway:** A Docker container must run a long-running foreground process. If it finishes or crashes, the container exits. Use `docker logs`, `-it` mode, and inspect `CMD/ENTRYPOINT` to debug.

---

# 2. Purpose of EXPOSE in Dockerfile

## Question
What is the purpose of the `EXPOSE` instruction in a Dockerfile?

## ✅ Answer
`EXPOSE` indicates which port the containerized application will listen on at runtime. It serves as **documentation** and a **signal** to tools like Docker and Docker Compose, but it does **not** actually publish the port.

To make a port accessible to the host, use `-p`:
```bash
docker run -p 8080:80 myapp
```

### 🔧 Example
```Dockerfile
FROM nginx:alpine
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

> **Key takeaway:** `EXPOSE` is a form of **documentation** inside the image. It doesn't open ports on its own.

---

# 3. Port Not Accessible on Localhost

## Question
Port is not accessible on `localhost` even after using `-p` flag. How do you troubleshoot?

## ✅ Answer
Often the app is listening on `127.0.0.1` instead of `0.0.0.0` inside the container.

### 🔧 Example
```python
# BAD - not accessible externally
app.run(host='127.0.0.1', port=5000)

# GOOD - accepts connections from any interface
app.run(host='0.0.0.0', port=5000)
```

### 🔍 Additional checks:
- Run `docker ps` to confirm port is published
- Use `docker exec -it <container> netstat -tulnp` to check listening
- Check firewall or security group rules
- Ensure no other service on host uses that port

> **Key takeaway:** Publishing ports with Docker is only part of the setup. The application inside the container must listen on `0.0.0.0` for external traffic.

---

# 4. Data Lost on Container Restart

## Question
Your application container loses data when it stops and restarts. How do you fix this?

## ✅ Answer
Mount a Docker volume or bind mount to persist data outside the container.

### 🗂️ 1. Docker Volumes
```bash
docker volume create mydata
docker run -v mydata:/app/data myapp
```

### 🧷 2. Bind Mounts
```bash
docker run -v /host/data:/app/data myapp
```

### 🔍 Verify Persistence
```bash
docker stop myapp && docker rm myapp
docker run -v mydata:/app/data myapp
# Data will still be there!
```

> **Key takeaway:** Containers are ephemeral by design. Use Docker volumes or bind mounts for databases, logs, or uploaded files.

---

# 5. Code Changes Not Reflected in New Image

## Question
You made a change in your code, rebuilt the image, but the container still shows old behavior. What could be the issue?

## ✅ Answer
Usually caused by **build caching** or a **volume overwriting** the updated code.

### 🧱 1. Docker build cache
```bash
docker build --no-cache -t myapp:latest .
```

Ensure proper `COPY` order in Dockerfile:
```Dockerfile
# GOOD: Dependencies first, then app code
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

### 📦 2. Volume overwriting image content
```bash
docker run -v "$(pwd)":/app myapp
```
If rebuilding the image but still mounting old source, the container sees **local files**, not image files.

### 🔍 Verify
```bash
docker run -it myapp:latest cat /app/main.py
```

> **Key takeaway:** If Docker changes aren't reflected, check for caching in `docker build` and make sure no local volumes are masking your image content.

---

# 6. App Crashes with Permission Denied

## Question
App runs fine locally but crashes with "Permission Denied" inside a Docker container. What could be the issue?

## ✅ Answer

### 🔒 1. Permissions not preserved during COPY
```Dockerfile
COPY start.sh /app/start.sh
RUN chmod +x /app/start.sh
```

### 👤 2. Running as non-root user
```Dockerfile
USER appuser
RUN chown -R appuser:appuser /app
```

### 🧪 3. Volume mounts with restricted permissions
```bash
chmod -R 755 data/
```

### 🔍 Debugging
```bash
docker exec -it <container> ls -l /app
docker run -it myapp /bin/sh
```

> **Key takeaway:** Always check file permissions and user context in your Dockerfile.

---

# 7. Host Ran Out of Disk Space

## Question
Your Docker host is running out of disk space. How do you clean up?

## ✅ Answer

### 🧹 Cleanup Steps

```bash
docker system df          # Overview of space used
docker system prune       # Remove stopped containers, unused networks, dangling images
docker system prune -a --volumes  # Deep cleanup (all unused images + volumes)
```

### 📦 Remove unused volumes
```bash
docker volume ls -f dangling=true
docker volume prune
```

### 🧱 Remove unused images
```bash
docker images --format "{{.Repository}}:{{.Tag}}\t{{.Size}}"
docker rmi <image-id>
```

### 🧊 Investigate storage
```bash
sudo du -sh /var/lib/docker/*
```

### 🛡️ Prevention
- Run a cron job: `docker system prune -a --volumes -f`
- Use smaller base images (`alpine`, `distroless`)
- Clean up in Dockerfiles:
  ```Dockerfile
  RUN apt-get update && apt-get install -y something \
   && rm -rf /var/lib/apt/lists/*
  ```

> **Key takeaway:** Docker doesn't auto-clean. Regularly prune unused containers, images, and volumes.

---

# 8. Debug a Live Container

## Question
Your application is running inside a Docker container but showing abnormal behavior. How would you debug it without stopping it?

## ✅ Answer

### 🛠️ 1. `docker exec` — run commands inside
```bash
docker exec -it <container-id> /bin/sh
docker exec -it myapp cat /var/log/app.log
docker exec -it myapp env
```

### 🔌 2. `docker logs` — view stdout/stderr
```bash
docker logs -f <container-id>
```

### 🧾 3. `docker inspect` — metadata and config
```bash
docker inspect <container-id>
docker inspect -f '{{.Config.Env}}' <container-id>
```

### 👁️ 4. `docker top` — running processes
```bash
docker top <container-id>
```

### 📊 5. Check network settings
```bash
docker exec -it <container-id> netstat -tulnp
docker exec -it <container-id> curl http://localhost:port
```

### ⚠️ 6. `docker attach` (with caution)
```bash
docker attach <container-id>
```
Use `Ctrl+P + Ctrl+Q` to detach safely.

> **Key takeaway:** Use `exec` for shell access, `logs` for output, `inspect` for config, and `top` for processes — all without downtime.

---

# 9. Container Registry

## Question
Which container registry does your organization use?

## ✅ Answer
We primarily use **Amazon Elastic Container Registry (ECR)**.

### 🐳 Amazon ECR
- Fully managed Docker container registry by AWS
- Integrated with **IAM** for permissions
- Built-in **image vulnerability scanning**
- Supports **lifecycle policies** to clean up old images

```bash
aws ecr get-login-password | docker login --username AWS --password-stdin <aws_account>.dkr.ecr.<region>.amazonaws.com
docker build -t myapp .
docker tag myapp:latest <repo-url>:latest
docker push <repo-url>:latest
```

### 🧰 Other Common Registries
- **Docker Hub** – public images
- **GitHub Container Registry (GHCR)** – GitHub projects
- **Azure Container Registry (ACR)** – Azure projects
- **Google Artifact Registry** – GCP users
- **Harbor** – self-hosted enterprise

> **Key takeaway:** Choose based on your cloud provider, CI/CD integration, and security policies.

---

# 10. CMD vs ENTRYPOINT

## Question
What is the difference between `CMD` and `ENTRYPOINT` in a Dockerfile?

## ✅ Answer

### 🔸 CMD — Default command (overridable)
```Dockerfile
FROM ubuntu
CMD ["echo", "Hello from CMD"]
```
```bash
docker run myimage               # Output: Hello from CMD
docker run myimage echo "Override"  # Output: Override
```

### 🔸 ENTRYPOINT — Fixed command
```Dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
```
```bash
docker run myimage Hello from ENTRYPOINT  # Output: Hello from ENTRYPOINT
```

### 🔄 Using ENTRYPOINT + CMD Together
```Dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Hello from CMD"]
```
```bash
docker run myimage              # Output: Hello from CMD
docker run myimage Custom msg   # Output: Custom msg
```

### 📝 Summary Table

| Feature | CMD | ENTRYPOINT |
|---|---|---|
| Purpose | Default command or args | Fixed executable |
| Overridable | ✅ Easily | ❌ Only with `--entrypoint` |
| Combination | Acts as args to ENTRYPOINT | Works with CMD as default args |

> **Key takeaway:** Use ENTRYPOINT for the main command, CMD for default arguments.

---

# 11. Docker Commands

## Question
What Docker commands do you use on a day-to-day basis?

## ✅ Answer

### 1. `docker ps` — Show running containers
```bash
docker ps       # Running only
docker ps -a    # All (including exited)
```

### 2. `docker build` — Build image from Dockerfile
```bash
docker build -t myapp:latest .
```

### 3. `docker run` — Run a container
```bash
docker run -d -p 8080:80 myapp
```

### 4. `docker exec` — Execute command inside container
```bash
docker exec -it myapp_container /bin/bash
```

### 5. `docker logs` — View container logs
```bash
docker logs -f myapp_container
```

### 6. `docker stop` / `docker start`
```bash
docker stop myapp_container
docker start myapp_container
```

### 7. `docker images` — List all images
```bash
docker images
```

### 8. `docker rm` / `docker rmi` — Remove container/image
```bash
docker rm myapp_container
docker rmi myapp:latest
```

### 9. `docker system prune` — Clean up unused resources
```bash
docker system prune -a
```

### 10. `docker inspect` — Inspect metadata
```bash
docker inspect myapp_container
```

> **Key takeaway:** Mastering core Docker commands like `build`, `run`, `exec`, and `logs` makes daily container work efficient.

---

# 12. Forcefully Remove a Container

## Question
Have you ever had to forcefully remove a Docker container? When and how?

## ✅ Answer
Yes, when containers are stuck in a dead or unresponsive state.

### 🧨 Force remove
```bash
docker rm -f <container-id or container-name>
```
This sends `SIGKILL` and then removes the container.

### 📋 Before force removing — check state
```bash
docker ps -a
docker inspect <container-id>
docker logs <container-id>
docker stop <container-id>     # Try graceful first
```

### 🔐 Caution
- Force removal **kills** the container immediately — no cleanup
- Any unsaved or non-persisted data is lost
- Use with caution in production

> **Key takeaway:** Use `docker rm -f` only when a container is stuck or unresponsive. Always try graceful stop first, and ensure important data is stored in volumes.

---
