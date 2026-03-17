# Docker Volumes — Three Types

## Question
What are Docker volumes? Explain the three types and their use cases.

## Ideal One-Liner
Docker volumes provide persistent storage that survives container removal; the three types are named volumes (Docker-managed, best for databases), anonymous volumes (Docker-managed but untracked), and host volumes/bind mounts (map a host path directly, full control).

---

## Full Answer

### Why Volumes?

A container's writable layer is **ephemeral** — it disappears when the container is removed. Anything you want to survive container restarts or removal must live in a volume.

Volumes also enable:
- Data sharing between multiple containers.
- Access to data from the host.
- Consistent storage location regardless of container lifecycle.

---

### Type 1: Named Volumes

Docker creates and manages the volume. You give it a name. Docker stores the data at `/var/lib/docker/volumes/<name>/_data` on the host (you rarely need to know this path directly).

```bash
# Create explicitly
docker volume create mydata

# Or let compose create it
```

```yaml
# docker-compose.yml
services:
  db:
    image: mysql:8.0
    volumes:
      - db_data:/var/lib/mysql    # named volume

volumes:
  db_data:                        # declare the named volume
```

```bash
docker run -v db_data:/var/lib/mysql mysql:8.0
```

**Characteristics:**
- Managed by Docker — survives `docker compose down` (but not `docker compose down -v`).
- Easy to reference by name across multiple containers.
- Data persists across container restarts and removal.
- Can be backed up: `docker run --rm -v db_data:/data alpine tar czf - /data > backup.tar.gz`

**Best for:** Database storage, any critical persistent data.

---

### Type 2: Anonymous Volumes

Same as named volumes, but Docker assigns a random UUID as the name. You cannot easily reference them later.

```dockerfile
# In a Dockerfile — creates an anonymous volume at /tmp/cache
VOLUME ["/tmp/cache"]
```

```bash
docker run -v /app/logs myimage    # anonymous volume at /app/logs
```

**Characteristics:**
- Docker manages the storage location.
- No name → hard to find and reuse later.
- Typically removed when the container is removed (with `docker rm`).
- Listed in `docker volume ls` with a long random ID.

**Best for:** Temporary scratch space, caching layers in multi-stage builds, declaring "this path should be a volume" in a Dockerfile without specifying details.

**Note:** Avoid relying on anonymous volumes for important data — use named volumes instead.

---

### Type 3: Host Volumes (Bind Mounts)

Maps a **specific host filesystem path** directly into the container. You control exactly where the data lives on the host.

```bash
docker run -v /home/steve/data:/app/data myimage
# or using --mount (more explicit):
docker run --mount type=bind,source=/home/steve/data,target=/app/data myimage
```

```yaml
# docker-compose.yml
services:
  app:
    volumes:
      - ./config:/app/config      # relative path resolved to absolute
      - /var/log/myapp:/app/logs  # absolute host path
```

**Characteristics:**
- You choose the host path — data is exactly where you expect it.
- Changes on the host are immediately visible in the container, and vice versa.
- Ties the container to the host's filesystem structure.
- Useful for development: mount your source code so changes are reflected without rebuilding.
- Not portable: if the host path doesn't exist, Docker creates an empty directory (or the bind mount fails depending on configuration).

**Best for:**
- Development environments (mount source code for live reloading).
- Accessing host config files (e.g., Nginx config).
- Log directories you want to access from the host.
- CI containers that need to read/write the project repo.

---

### Comparison Table

| Feature | Named Volume | Anonymous Volume | Host Volume |
|---|---|---|---|
| Location | Docker-managed | Docker-managed | Host path you specify |
| Named/referenceable | Yes | No (random ID) | Yes (by path) |
| Persists after container removal | Yes | Sometimes | Yes (it's on the host) |
| Portable across hosts | Yes (movable) | Yes | No (path must exist) |
| Good for production DB | Yes | No | Possible but not ideal |
| Good for dev code mount | No | No | Yes |

---

### Gan Shmuel Reference

The project demonstrates all three types in practice:

1. **Named volumes** — `mysql_weight_data` and `mysql_billing_data` for the two MySQL databases. Defined in `docker-compose.yml`. Persist across `docker compose down && up`.

2. **Host volume** — the project repository (`/home/steve/bootcamp/Week2/gan-shmuel`) is mounted into the CI container at `/repo`. When `ci/app.py` runs `git pull`, the updated files are immediately visible inside the container without a rebuild.

3. **Anonymous volumes** — not explicitly used in Gan Shmuel, but could appear in dev stages or Dockerfile `VOLUME` declarations.
