# ENTRYPOINT vs CMD in a Dockerfile

## Question
What is the difference between ENTRYPOINT and CMD in a Dockerfile? How can you overwrite each?

## Ideal One-Liner
ENTRYPOINT defines the fixed executable that always runs; CMD provides default arguments to ENTRYPOINT (or the default command if no ENTRYPOINT exists), and can be overridden at `docker run` time without the `--entrypoint` flag.

---

## Full Answer

### CMD — The Default Command

`CMD` specifies the **default command or arguments** to run when a container starts. It can be overridden by simply passing arguments to `docker run`.

```dockerfile
CMD ["python3", "app.py"]
```

```bash
docker run myimage                     # runs: python3 app.py
docker run myimage python3 other.py    # overrides CMD, runs: python3 other.py
```

---

### ENTRYPOINT — The Fixed Executable

`ENTRYPOINT` defines the executable that **always runs** — it cannot be bypassed by passing arguments to `docker run` (only by using the `--entrypoint` flag explicitly).

```dockerfile
ENTRYPOINT ["python3"]
```

```bash
docker run myimage              # runs: python3   (nothing useful — no args)
docker run myimage app.py       # runs: python3 app.py  ← args appended to ENTRYPOINT
```

---

### Using Both Together (The Common Pattern)

When both are set:
- **ENTRYPOINT** = the executable
- **CMD** = the default arguments

```dockerfile
ENTRYPOINT ["python3"]
CMD ["app.py"]
```

```bash
docker run myimage               # runs: python3 app.py   (default)
docker run myimage other.py      # runs: python3 other.py (CMD overridden, ENTRYPOINT fixed)
```

This is the idiomatic pattern: ENTRYPOINT locks in the binary, CMD provides swappable defaults.

---

### How to Override Each

| What | How | Flag needed? |
|---|---|---|
| Override CMD | `docker run myimage <new args>` | No flag — just pass args |
| Override ENTRYPOINT | `docker run --entrypoint bash myimage` | Requires `--entrypoint` |

```bash
# Override CMD only
docker run myimage other_script.py

# Override ENTRYPOINT (get a shell for debugging)
docker run --entrypoint bash myimage -c "ls /app"
docker run --entrypoint /bin/sh myimage
```

---

### Shell Form vs Exec Form

**Exec form (recommended):** `["executable", "arg1", "arg2"]`
- Runs the process directly as PID 1.
- Signals (SIGTERM, SIGINT) are delivered directly to the process.
- No shell interpretation.

**Shell form:** `executable arg1 arg2`
- Docker prepends `/bin/sh -c`.
- The process is a child of `/bin/sh`, not PID 1.
- SIGTERM goes to the shell, not your app — graceful shutdown may not work.

```dockerfile
# Bad: shell form — your process won't receive signals directly
ENTRYPOINT python3 app.py

# Good: exec form — your process is PID 1, receives signals directly
ENTRYPOINT ["python3", "app.py"]
```

---

### Common Patterns

```dockerfile
# Pattern 1: Locked executable, overridable script
ENTRYPOINT ["python3"]
CMD ["app.py"]

# Pattern 2: Docker as a CLI tool (e.g., curl image)
ENTRYPOINT ["curl"]
CMD ["--help"]
# docker run myimage http://example.com → runs curl http://example.com

# Pattern 3: Entrypoint script (common for init logic)
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["python3", "app.py"]
# entrypoint.sh does setup then: exec "$@"
```

---

### Gan Shmuel Reference
The Flask services in Gan Shmuel (weight and billing) use the ENTRYPOINT/CMD pattern: the ENTRYPOINT ensures `python3` (or `flask run`) always executes, and CMD provides the script name. This allows the containers to be started with different scripts during testing vs. production without changing the base image — only the CMD argument changes.
