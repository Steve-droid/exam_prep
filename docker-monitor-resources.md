# Monitoring Docker Container Resource Utilization

## Question
How can you monitor resource utilization of a container?

## Ideal One-Liner
Use `docker stats` for live CPU/memory/network/disk metrics, `docker top` for processes inside the container, and `docker inspect` for full metadata; in production, use Prometheus + cAdvisor or a similar observability stack.

---

## Full Answer

### Built-in Docker Commands

#### `docker stats` — Live Resource Usage

```bash
# Live stats for all running containers (refreshes automatically)
docker stats

# Stats for a specific container
docker stats web db

# Non-streaming snapshot (useful in scripts)
docker stats --no-stream

# Custom format
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

Output columns:
| Column | What it shows |
|---|---|
| CPU % | CPU usage relative to total host CPU |
| MEM USAGE / LIMIT | Current memory used / memory limit |
| MEM % | Memory as % of limit |
| NET I/O | Total network bytes sent/received |
| BLOCK I/O | Total disk read/write bytes |
| PIDS | Number of processes inside the container |

#### `docker top` — Processes Inside the Container

```bash
docker top <container>          # list processes (like ps aux on the host)
docker top <container> aux      # pass flags to ps
```

Useful for seeing what's running inside without exec-ing into the container.

#### `docker inspect` — Detailed Configuration and State

```bash
docker inspect <container>

# Extract specific fields
docker inspect web --format '{{.State.Status}}'
docker inspect web --format '{{.HostConfig.Memory}}'    # memory limit in bytes
docker inspect web --format '{{.State.OOMKilled}}'       # was it OOM killed?
```

Use `docker inspect` when you need: IP addresses, volume mounts, network config, resource limits, restart policy, environment variables, start/finish times.

#### `docker logs` — Stdout/Stderr Output

```bash
docker logs <container>         # all logs since container started
docker logs -f <container>      # follow (tail -f style)
docker logs --tail 50 web       # last 50 lines
docker logs --since 1h web      # logs from last hour
docker logs --timestamps web    # include timestamps
```

---

### Setting Resource Limits

You can also set limits at run time to prevent a container from consuming excessive resources:

```bash
# Limit CPU to 0.5 cores and memory to 512MB
docker run -d \
  --cpus="0.5" \
  --memory="512m" \
  --name web \
  nginx
```

In docker-compose:
```yaml
services:
  web:
    image: nginx
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 512M
```

---

### Production Monitoring

`docker stats` is for quick local debugging. For production:

| Tool | What it does |
|---|---|
| **cAdvisor** | Container-level metrics exporter (Google) |
| **Prometheus** | Time-series metrics collector — scrapes cAdvisor |
| **Grafana** | Dashboards and visualization for Prometheus data |
| **Datadog** | SaaS observability platform with Docker integration |
| **ELK Stack** | Elasticsearch + Logstash + Kibana for log aggregation |

The standard production stack: **Prometheus + cAdvisor + Grafana** — open source, widely used, good Docker support.

---

### Gan Shmuel Reference
In Gan Shmuel, monitoring was not explicitly built out (the focus was on CI/CD). But `docker stats` would be used to check that the weight and billing containers aren't consuming excessive resources after deployment, and `docker logs` is used throughout `ci/app.py` implicitly — the Flask CI service captures stdout from subprocesses to detect test failures and build errors.
