# Container Keeps Crashing — Troubleshooting by Lifecycle Stage

## Question
Your container keeps crashing. Walk through your troubleshooting process based on the lifecycle stages.

## Ideal One-Liner
Work through the three lifecycle stages: confirm the image builds correctly (build time), check startup conditions and config (init time), then diagnose the runtime error from logs, exit codes, and resource usage (run time).

---

## Full Answer

### The Three Lifecycle Stages

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Build Time  │ → │  Init Time   │ → │  Run Time    │
│  (image)     │    │  (start)     │    │  (execution) │
└─────────────┘    └─────────────┘    └─────────────┘
```

---

### Stage 1: Build Time — Is the Image Correct?

Before a container can crash, the image must build and be valid.

```bash
# Does the image build at all?
docker build -t myapp .
# If this fails, the Dockerfile has an error (bad syntax, missing file, failed RUN command)

# Is the base image valid?
docker pull python:3.11-slim    # can Docker pull it?

# Inspect image layers
docker history myapp

# Check the image exists
docker images | grep myapp
```

Common build-time issues:
- Typo in base image name → `docker build` fails at FROM
- `COPY` references a file that doesn't exist in build context → fails at COPY
- `RUN pip install` fails → package not found or network issue

---

### Stage 2: Init Time — Does It Start?

The container was created and is attempting to start.

```bash
# Attempt to start manually and watch what happens
docker run --name test myapp

# If it exits immediately:
docker logs test
docker inspect test --format '{{.State.ExitCode}}'
```

Check these conditions:

**Environment variables:**
```bash
# Did you pass all required env vars?
docker run -e DB_HOST=db -e DB_PASSWORD=secret myapp
# Missing env var → app throws error on startup → exits
```

**Volume mounts:**
```bash
# Does the host path exist?
docker run -v /does/this/path/exist:/app/config myapp
# If it doesn't exist, Docker creates an empty dir → app finds no config → crashes
```

**Port conflicts:**
```bash
# Is port 8080 already in use on the host?
ss -tlnp | grep 8080
docker run -p 8080:5000 myapp  # fails if 8080 is taken
```

**Permissions:**
```bash
# Is the entrypoint script executable?
docker run -it --entrypoint ls myapp /app
# Check: is app.py / start.sh executable?
```

---

### Stage 3: Run Time — What's Happening During Execution?

The container started but crashes after some time.

```bash
# What does the container output before dying?
docker logs myapp
docker logs -f myapp        # follow in real time

# What's the exit code?
docker ps -a                # shows Exited (N)

# Is it OOM?
docker inspect myapp --format '{{.State.OOMKilled}}'
# If true → increase memory limit or fix memory leak

# Is it in a restart loop?
docker inspect myapp --format '{{.RestartCount}}'
# High restart count → crash loop (CrashLoopBackOff equivalent)

# Resource usage before crash
docker stats myapp           # live CPU/memory — is something spiking?
```

Runtime crash causes and fixes:

| Symptom | Likely Cause | Fix |
|---|---|---|
| App crashes after DB query | DB connection timeout or bad query | Check DB logs, fix query or connection config |
| OOMKilled: true | App consuming too much memory | Increase `--memory` limit or fix memory leak |
| Exit 137 (no OOMKilled) | External `docker kill` or host OOM | Check host memory, check restart policy |
| Frequent brief restart loop | App retrying a failed dependency | Add retry logic, fix the dependency |
| Crash after N minutes | Memory leak / goroutine leak | Profile the app |

---

### Restart Policies

```yaml
# docker-compose.yml
services:
  web:
    restart: on-failure:3     # restart up to 3 times on failure, then give up
    # restart: unless-stopped # always restart unless manually stopped
    # restart: always         # always restart (even on clean exit)
    # restart: "no"           # never restart (default)
```

When debugging, set `restart: "no"` so the container doesn't loop and you can read the last logs before it would restart.

---

### Gan Shmuel Reference

A concrete crash example from Gan Shmuel: the CI container was configured without mounting the Docker socket (`/var/run/docker.sock`). On startup, `ci/app.py` initialized fine, but the first time the CI pipeline received a webhook and tried to run `docker compose up`, it failed with "Cannot connect to the Docker daemon." The container then exited with code 1. Troubleshooting path: `docker logs ci` → saw the connection error → `docker inspect ci` → confirmed no Docker socket in volumes → added the bind mount in `docker-compose.yml` → fixed.
