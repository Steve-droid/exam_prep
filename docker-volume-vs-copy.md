# Volume vs COPY in a Dockerfile

## Question
When should you use a volume vs COPY in a Dockerfile?

## Ideal One-Liner
Use COPY for application code and static configs that are baked into the image at build time; use volumes for data that must persist beyond a container's lifecycle or that changes at runtime without requiring a rebuild.

---

## Full Answer

### COPY — Build-Time, Static, Immutable

`COPY` bakes files into the image during `docker build`. Once built, those files are part of the image layer — they are immutable unless you rebuild the image.

```dockerfile
COPY app.py /app/app.py
COPY config.yml /app/config.yml
```

- The file is embedded in the image.
- Every container started from this image has the same file.
- To change the file, you must rebuild the image.
- Data does not persist between container runs — but since it's in the image, it's always there when the container starts.

**Best for:**
- Application source code.
- Static configuration files that are part of the deployment.
- Compiled binaries.
- Anything that should be versioned with the image.

---

### Volumes — Run-Time, Dynamic, Persistent

Volumes mount external storage into the container at a specific path. The data exists outside the container's writable layer and survives container removal.

```yaml
# docker-compose.yml
services:
  db:
    image: mysql:8.0
    volumes:
      - db_data:/var/lib/mysql    # named volume

  app:
    image: myapp
    volumes:
      - /host/logs:/app/logs      # host volume (bind mount)
```

- The mount point path exists in the container, but the data lives on the host (or in Docker-managed storage).
- Data persists even after `docker rm` — the volume is separate from the container.
- Multiple containers can share a volume.
- Data can be updated on the host without rebuilding the image.

**Best for:**
- Database files (MySQL, Postgres data directories).
- Log files that need to persist and be accessible from outside.
- User-uploaded content.
- Configuration that changes per-deployment environment without rebuilding.
- The CI container in Gan Shmuel — the repo directory is bind-mounted so git pull updates the code without a rebuild.

---

### They Can Appear to Do the Same Thing

Both of these put a `config.yml` at `/app/config.yml`:

```dockerfile
# Approach A: COPY (build-time)
COPY config.yml /app/config.yml
```

```yaml
# Approach B: volume (run-time)
volumes:
  - ./config.yml:/app/config.yml
```

The difference:
- **COPY:** Config is baked in. Change it → rebuild the image → redeploy.
- **Volume:** Config lives on the host. Change it → restart the container (or sometimes just reload).

For per-environment config (dev vs staging vs prod), volumes are often better — you don't want to rebuild the image just to change a config value.

---

### Decision Guide

| Use Case | COPY or Volume? |
|---|---|
| App source code | COPY — part of the image |
| Python/Node dependencies | COPY via RUN install — baked in |
| MySQL data directory | Named volume — persists data |
| Logs you need to access from host | Host volume (bind mount) |
| Per-environment config file | Host volume or environment variable |
| Repo code in CI container | Host volume — CI needs latest git pull without rebuild |
| Static assets (HTML, CSS) | COPY — part of the image |
| User uploads / file storage | Named or host volume — persists beyond container |

---

### Gan Shmuel Reference

Two concrete examples from the project:

1. **Named volume for MySQL:** `mysql_data:/var/lib/mysql` — database files persist across container restarts and removal. If you do `docker compose down` and `up` again, the database is still there.

2. **Host volume for the repo in CI:** The project repository is mounted into the CI container at `/repo`. When `ci/app.py` runs `git pull`, the code on the host filesystem is updated — the CI container sees the changes immediately without needing to rebuild the CI image.
