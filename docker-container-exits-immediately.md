# Container Exits Immediately After Starting

## Question
Why might a container exit immediately after starting? What would you check?

## Ideal One-Liner
A container exits when PID 1 exits — common causes are a missing/crashing entrypoint command, an app that starts and completes (expected exit), or a startup error like a missing config or env var.

---

## Full Answer

### Why Containers Exit

Docker containers run as long as their **PID 1** (the main process) runs. The moment PID 1 exits — for any reason — the container stops. There is no process manager keeping it alive (unlike a VM or a systemd service).

This means:
- If your command runs and completes (e.g., `echo hello`), the container exits cleanly — this is correct behavior.
- If your app crashes on startup, the container exits with an error code.
- If the entrypoint path is wrong, the container can't start at all.

---

### Common Causes

#### 1. The command finishes (expected exit)
```dockerfile
CMD ["echo", "hello world"]    # runs once, exits cleanly — exit code 0
```
Not a bug — this container was designed to run a task and exit. Use `docker run --rm` for these.

#### 2. The command or entrypoint doesn't exist
```dockerfile
ENTRYPOINT ["pythonn3", "app.py"]   # typo — "pythonn3" not found
```
Container starts, bash tries to exec `pythonn3`, fails immediately with "exec: no such file."

#### 3. The app crashes on startup
- Missing required environment variable.
- Config file not found (bad path, volume not mounted).
- Port already in use.
- Database connection fails on startup (app crashes instead of retrying).
- Missing Python import, syntax error in app code.

#### 4. Permission issues
```dockerfile
# App tries to write to /var/log/app/ but that directory is owned by root
USER appuser     # runs as non-root but /var/log/app/ isn't writable
```

#### 5. The container's process goes to background
```dockerfile
# Bad: Nginx forks into background → PID 1 exits → container exits
CMD ["nginx"]

# Good: Run in foreground (daemon off)
CMD ["nginx", "-g", "daemon off;"]
```
This is a common trap with services that daemonize by default.

---

### What to Check

```bash
# 1. See the container in the stopped state
docker ps -a
# Look at STATUS column — "Exited (1)" vs "Exited (0)" is meaningful

# 2. Read the logs — this is almost always where the answer is
docker logs <container>

# 3. Check the exit code
docker inspect <container> --format '{{.State.ExitCode}}'
# 0 = clean exit
# 1 = app error
# 126 = permission denied to execute
# 127 = command not found (typo in ENTRYPOINT/CMD)
# 137 = SIGKILL (OOM or manual kill)
# 139 = segmentation fault

# 4. Override the entrypoint to inspect the container manually
docker run -it --entrypoint bash myimage
# Or:
docker run -it --entrypoint sh myimage
# Then try running the original command yourself to see the error
```

---

### Debugging Workflow

```
Container exits immediately
│
├── docker ps -a → exit code?
│   ├── 0 → app ran to completion (is this expected?)
│   ├── 127 → command not found → check ENTRYPOINT/CMD spelling
│   ├── 1 → application error
│   └── 137 → OOM killed
│
├── docker logs <container> → any error message?
│   └── "No such file" / "ImportError" / "Connection refused" → fix the cause
│
└── docker run -it --entrypoint sh myimage → manual inspection
    └── run the app command manually to see the full error
```

---

### Gan Shmuel Reference
During Gan Shmuel development, a common issue was the CI container exiting immediately because the Docker socket (`/var/run/docker.sock`) was not mounted — the CI app would start, try to run `docker compose` commands, fail with permission denied, and exit. Checking `docker logs ci` immediately showed the Python traceback, revealing the socket mount was missing from the compose file.
