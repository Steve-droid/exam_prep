# What Happens When the Main Process Inside a Container Stops

## Question
What happens when the main process inside a container stops?

## Ideal One-Liner
When PID 1 exits, the container immediately stops and inherits its exit code — what happens next depends on the configured restart policy.

---

## Full Answer

### PID 1 is the Key
The container's main process is always **PID 1**. When PID 1 exits — cleanly, crashed, or killed — the container stops. It doesn't matter if other processes are still running inside.

### Exit Code
The container's exit code equals PID 1's exit code:
- `0` = clean/intentional exit
- Non-zero = error or crash

This matters for CI pipelines — `docker wait` returns this code, and tools use it to determine success/failure.

### Restart Policies
What happens after the container stops depends on the configured restart policy:

| Policy | Behavior |
|---|---|
| `no` (default) | Container stays stopped, nothing happens |
| `on-failure` | Restarts only if exit code ≠ 0 |
| `always` | Restarts regardless of exit code |
| `unless-stopped` | Like `always`, but respects manual `docker stop` |

Set with: `docker run --restart=always ...` or in `docker-compose.yml`:
```yaml
restart: always
```

### Stopped ≠ Removed
A stopped container **still exists**. Its writable layer (filesystem state) is preserved. You can:
- Restart it: `docker start <container>`
- Inspect it: `docker logs <container>`, `docker inspect <container>`
- Remove it: `docker rm <container>` — only now is the writable layer gone

### Gan Shmuel Reference
In `docker-compose.yml`, the MySQL containers use `restart: always` so they recover automatically if they crash during the pipeline.
