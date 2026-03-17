# Docker Container Lifecycle

## Question
Explain the Docker container lifecycle — covering build time, initialization time, and run time.

## Ideal One-Liner
A Docker container moves through three phases: image creation (build), container setup (init), and process execution (run) — each with distinct commands and failure points.

---

## Full Answer

### Build Time
`Dockerfile` is executed top-to-bottom. Each filesystem-modifying instruction (`RUN`, `COPY`, `ADD`) creates a new read-only layer. Layers are cached. Result: a tagged, immutable image.
```bash
docker build -t myapp:1.0 .
```
In Gan Shmuel: `ci/app.py` runs `docker compose build` after every git pull.

### Initialization Time
Between `docker run` and PID 1 starting: Docker creates a writable layer, sets up networking, mounts volumes, injects env vars, and resolves the entrypoint.
```bash
docker run ...    # create + start
docker create ... # create only
```
In Gan Shmuel: on deploy, Docker attaches the weight service to the bridge network, mounts the MySQL named volume, and maps 8082→8080.

### Run Time
Container lives as long as **PID 1** runs. PID 1's exit code becomes the container's exit code. Restart policies (`always`, `on-failure`) apply here.
```bash
docker logs <c>        # stdout/stderr
docker exec -it <c> bash
docker stop <c>        # SIGTERM → SIGKILL
```

| Phase | Trigger | Failure result |
|---|---|---|
| Build | `docker build` | Image not created |
| Init | `docker run` | Container won't start |
| Run | PID 1 starts | Container exits |
