# Docker Basic Operations

## Question
How do you run a container in the background, access a running container, check running containers, stop a container, and remove an image?

## Ideal One-Liner
The core Docker operations are: `docker run -d` (background), `docker exec -it bash` (shell access), `docker ps` (list running), `docker stop` (graceful shutdown), and `docker rmi` (remove image).

---

## Full Answer

### Run a Container

```bash
# Run in foreground (blocks your terminal, logs stream to stdout)
docker run nginx

# Run in background / detached mode (-d)
docker run -d nginx

# Run with a name (makes it easier to reference later)
docker run -d --name my-nginx nginx

# Run with port mapping (host:container)
docker run -d -p 8080:80 --name my-nginx nginx

# Run and auto-remove when it exits (good for one-off tasks)
docker run --rm alpine echo "hello"

# Run with environment variables
docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql:8.0

# Run with a volume
docker run -d -v mydata:/var/lib/mysql mysql:8.0

# Run interactively (get a shell immediately)
docker run -it ubuntu bash
```

---

### Access a Running Container

```bash
# Get an interactive shell inside a running container
docker exec -it <container_name_or_id> bash

# If the container has no bash, try sh
docker exec -it <container> sh

# Run a single command without interactive shell
docker exec my-nginx nginx -t     # test nginx config

# Run as root even if the container runs as another user
docker exec -it --user root <container> bash
```

`-i` = interactive (keep stdin open)
`-t` = allocate a pseudo-TTY (makes it feel like a real terminal)

---

### Check Running Containers

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Show only container IDs (useful for scripting)
docker ps -q

# Filter by status
docker ps --filter "status=exited"

# Pretty format output
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

---

### Stop a Container

```bash
# Graceful stop: sends SIGTERM, waits 10s, then sends SIGKILL
docker stop <container>

# Immediate stop: sends SIGKILL directly (no graceful shutdown)
docker kill <container>

# Stop all running containers
docker stop $(docker ps -q)
```

---

### Remove a Container

```bash
# Remove a stopped container
docker rm <container>

# Force-remove a running container (equivalent to kill + rm)
docker rm -f <container>

# Remove all stopped containers
docker container prune
```

---

### Remove an Image

```bash
# Remove a specific image by name or ID
docker rmi nginx
docker rmi nginx:1.25.3
docker rmi abc123def456

# Force remove (even if a container is using it)
docker rmi -f nginx

# Remove all unused images (not referenced by any container)
docker image prune

# Remove ALL images (use with caution)
docker image prune -a
```

Note: you cannot `docker rmi` an image if a running (or even stopped) container was created from it. Remove the container first, or use `-f`.

---

### Quick Reference Card

```bash
docker run -d --name web -p 8080:80 nginx    # run detached with name and port
docker exec -it web bash                      # shell into running container
docker ps                                     # list running containers
docker ps -a                                  # include stopped containers
docker logs web                               # view stdout/stderr logs
docker logs -f web                            # follow logs live
docker stop web                               # graceful stop
docker kill web                               # immediate stop
docker rm web                                 # remove stopped container
docker rm -f web                              # stop and remove
docker rmi nginx                              # remove image
docker run --rm alpine echo "done"            # auto-remove after exit
```

---

### Gan Shmuel Reference
In `ci/app.py`, the pipeline uses `docker compose` commands (not raw `docker run`) to manage the services. But the underlying operations are the same: build images, start containers in detached mode, check health endpoints, stop test containers after pytest, and promote to prod containers. Understanding these primitive Docker commands is what makes the compose workflow comprehensible.
