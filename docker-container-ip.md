# How a Container Gets an Internal IP Address

## Question
How does a container obtain an internal IP address?

## Ideal One-Liner
When a container joins a Docker network, Docker's built-in DHCP assigns it an IP from the network's subnet, and for user-defined bridge networks, Docker also registers the container name in its internal DNS so containers reach each other by name rather than IP.

---

## Full Answer

### The Mechanism

1. When you create a Docker network (e.g., `docker network create mynet`), Docker allocates a **subnet** for it (e.g., `172.18.0.0/16`).
2. When a container starts and joins that network, Docker's internal **DHCP-like allocator** assigns the container an available IP from that subnet (e.g., `172.18.0.2`).
3. Docker's embedded **DNS server** (running at `127.0.0.11` inside every container) maps the container name to its IP.

```
Network: my_network (172.18.0.0/16)

Container "web"  → 172.18.0.2
Container "db"   → 172.18.0.3

"web" container resolves "db" → 172.18.0.3 via Docker's internal DNS
```

---

### How DNS Resolution Works Inside a Container

When a container on a user-defined network does:
```python
import socket
socket.gethostbyname("db")    # → 172.18.0.3
```

Docker's embedded DNS server (`127.0.0.11`) handles the lookup. This is configured automatically in `/etc/resolv.conf` inside the container:

```bash
# Inside a container:
cat /etc/resolv.conf
# nameserver 127.0.0.11
# options ndots:0
```

This is why you can write `mysql:3306` in an app's database config instead of `172.18.0.3:3306`.

---

### IPs Are Ephemeral — Use DNS by Name

Container IPs change when containers restart or are recreated. You should **never hardcode a container IP** in application config. Always use the container name or service name (in compose):

```python
# Bad — hardcoded IP, breaks on restart
DB_HOST = "172.18.0.3"

# Good — resolved by Docker DNS
DB_HOST = "db"
DB_HOST = "weight-db"   # Gan Shmuel style
```

---

### Checking a Container's IP

```bash
# Inspect the container for network details
docker inspect <container_name> | grep IPAddress

# More specific:
docker inspect <container_name> \
  --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'

# From inside the container:
hostname -I
ip addr show
cat /etc/hosts
```

---

### Default Bridge vs User-Defined Bridge

On the **default bridge network** (`docker0`):
- Containers get IPs in the `172.17.0.0/16` range.
- **No DNS by name** — containers can only reach each other by IP.
- This is a legacy limitation.

On a **user-defined bridge network**:
- Containers get IPs in whatever subnet was configured.
- **DNS by name works automatically** — this is the key advantage.
- Recommended for all modern Docker usage.

---

### Gan Shmuel Reference
In Gan Shmuel, `weight-db` and `billing-db` are container names on a compose-defined bridge network. The Flask weight app connects using `DB_HOST=weight-db` — Docker's DNS resolves `weight-db` to whatever IP that container has at runtime. This means the app config doesn't change even if the container is recreated with a different IP.
