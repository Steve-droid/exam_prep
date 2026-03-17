# Debugging a Docker Container

## Question
How can you debug a Docker container and view its processes both from inside and outside the container?

## Ideal One-Liner
From outside: use `docker logs`, `docker top`, `docker stats`, and `docker inspect`; from inside: use `docker exec -it bash` to get a shell and run standard Linux tools.

---

## Full Answer

### From Outside the Container

These commands work from the host without entering the container:

```bash
# --- LOGS ---
docker logs <container>         # all stdout/stderr output
docker logs -f <container>      # follow logs in real time (like tail -f)
docker logs --tail 100 <container>   # last 100 lines
docker logs --timestamps <container> # include timestamps
docker logs --since 30m <container>  # logs from last 30 minutes

# --- PROCESSES ---
docker top <container>          # list processes running inside (like ps aux)
docker top <container> aux      # pass ps flags

# --- RESOURCE USAGE ---
docker stats <container>        # live CPU, memory, network, disk I/O
docker stats --no-stream <container>  # single snapshot

# --- FULL METADATA ---
docker inspect <container>      # complete JSON: config, state, IPs, volumes, env vars

# Useful inspect extractions:
docker inspect <container> --format '{{.State.Status}}'
docker inspect <container> --format '{{.State.ExitCode}}'
docker inspect <container> --format '{{.State.OOMKilled}}'
docker inspect <container> --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'

# --- COPY FILES OUT ---
docker cp <container>:/app/logfile.txt ./logfile.txt   # extract a file for inspection
```

---

### From Inside the Container

```bash
# Get an interactive shell
docker exec -it <container> bash     # bash shell
docker exec -it <container> sh       # sh (for alpine or minimal images)

# Once inside:
ps aux                      # all running processes
ps -ef                      # full process list with parent PIDs
cat /proc/1/cmdline         # see what PID 1 is running
ls /proc/1/fd               # open file descriptors of PID 1
top                         # interactive process monitor
htop                        # better top (if installed)

# Network debugging:
netstat -tlnp               # listening ports
ss -tlnp                    # same (modern replacement for netstat)
curl localhost:5000          # test if the app responds locally

# Environment and config:
env                         # all environment variables
cat /etc/hosts              # hostname → IP mappings
cat /etc/resolv.conf        # DNS config

# File system:
ls /app                     # verify app files are there
cat /app/config.yml         # check config values
df -h                       # disk space
```

---

### When the Container Has No Shell (Distroless, Minimal Images)

If `docker exec bash` fails with "executable not found":

```bash
# Option 1: Extract files you want to inspect
docker cp <container>:/app/logfile.txt .

# Option 2: Add a temporary debug tool via override at run time
docker run -it --entrypoint sh myimage    # try sh if bash not present

# Option 3: Sidecar container sharing the same pid namespace
docker run -it --pid=container:<target> busybox sh
# From inside busybox, you can see the target container's processes

# Option 4: nsenter (advanced — enter container namespaces from host)
docker inspect <container> --format '{{.State.Pid}}'
nsenter -t <pid> -m -u -i -n -p -- bash
```

---

### Debugging a Crash (Container Won't Stay Running)

```bash
# See exit code and last logs
docker ps -a                    # exit code in STATUS column
docker logs <container>         # what happened before crash

# Override entrypoint to get a shell before the crash
docker run -it --entrypoint bash myimage
# Then manually run the app to see the error:
python3 app.py
```

---

### Gan Shmuel Reference
In Gan Shmuel, `docker logs` was the primary debugging tool during CI pipeline development. When the weight or billing service failed to start, `docker logs weight` showed the Flask traceback. When the CI container itself had issues (e.g., failing to run docker commands because the socket wasn't mounted), `docker logs ci` captured the Python exception from `ci/app.py`. The `docker exec -it ci bash` command was used to manually test git pull and docker compose commands inside the CI container during debugging.
