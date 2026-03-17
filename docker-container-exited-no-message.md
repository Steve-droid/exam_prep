# Troubleshooting a Container That Exited Without a Message

## Question
A Docker container exited unexpectedly without a message. How do you troubleshoot?

## Ideal One-Liner
Start with `docker ps -a` for the exit code, then `docker logs` for last output, then `docker inspect` for OOMKilled and state details, then escalate to host-level tools like `dmesg` for OOM events.

---

## Full Answer

### Systematic Troubleshooting Approach

Think of it as moving from the most accessible information (Docker layer) to deeper system-level events (host layer).

---

### Step 1: Exit Code — `docker ps -a`

```bash
docker ps -a
# CONTAINER ID  IMAGE   STATUS              ...
# abc123        myapp   Exited (137) 2m ago ...
```

The exit code tells you a lot before you read a single log line:

| Exit Code | Meaning |
|---|---|
| 0 | Clean exit — process ran to completion (may be intentional) |
| 1 | Generic application error |
| 2 | Shell misuse or bash error |
| 126 | Permission denied — cannot execute the command |
| 127 | Command not found — typo in ENTRYPOINT/CMD |
| 128+N | Killed by signal N. E.g., 137 = 128+9 = SIGKILL (OOM or force kill) |
| 137 | OOM kill or `docker kill` |
| 139 | Segmentation fault (128+11=SIGSEGV) |
| 143 | SIGTERM (128+15) — graceful shutdown signal received |

---

### Step 2: Logs — `docker logs`

```bash
docker logs <container>          # all output since start
docker logs --tail 50 <container>  # last 50 lines
docker logs --timestamps <container>  # with timestamps to see when it died
```

Even if the app didn't print an error to stderr, the **last lines before exit** often reveal the cause. If logs are completely empty, the app may have crashed before it could write anything — likely a startup issue.

---

### Step 3: Inspect State — `docker inspect`

```bash
docker inspect <container>

# Key fields to extract:
docker inspect <container> --format '{{.State.ExitCode}}'
docker inspect <container> --format '{{.State.OOMKilled}}'     # true/false
docker inspect <container> --format '{{.State.Error}}'         # runtime error message
docker inspect <container> --format '{{.State.FinishedAt}}'    # when it died
docker inspect <container> --format '{{.RestartCount}}'        # crash loop count
```

If `OOMKilled: true` → the container was killed by the Linux OOM (Out of Memory) killer because it exceeded its memory limit or the host ran out of memory.

---

### Step 4: Docker Events — `docker events`

```bash
docker events --since 30m    # system events from the last 30 minutes
```

Output might show:
```
container die  (exitCode=137, name=myapp)
container oom  (name=myapp)
```

This gives you a timeline of what happened to the container.

---

### Step 5: Host-Level — OOM Killer and System Logs

If the exit code suggests OOM (137) and Docker's OOMKilled is false (host-level OOM, not container-level):

```bash
# Linux kernel OOM killer log
dmesg | grep -i "oom\|kill"
dmesg | grep -i "out of memory"

# systemd journal (on systemd-based hosts)
journalctl -k --since "1 hour ago" | grep -i oom

# Check current memory pressure
free -h
cat /proc/meminfo
```

---

### Step 6: Resolve Based on What You Found

| Finding | Resolution |
|---|---|
| Exit 0 | Process completed normally — add `restart: unless-stopped` or rethink the service design |
| Exit 1, logs show app error | Fix the application bug or missing config |
| OOMKilled: true | Increase memory limit (`--memory=512m`) or optimize the app's memory usage |
| Host OOM | Add more RAM to the host, or reduce container count/limits |
| Exit 127 | Fix typo in ENTRYPOINT or CMD |
| No logs at all | App crashed before logging — `docker run -it --entrypoint sh` to debug manually |
| Frequent restarts | Use `restart: on-failure:3` to limit restart loops and investigate |

---

### Gan Shmuel Reference
In a crash scenario during Gan Shmuel, the troubleshooting flow was: `docker ps -a` to confirm the container was stopped → `docker logs <service>` to see the Flask traceback → `docker inspect <service>` to verify it wasn't OOM killed. Most issues traced back to misconfigured environment variables or the Docker socket not being available for the CI container.
