# Assigning a Specific IP to a Container

## Question
How can you assign a specific IP address to a container?

## Ideal One-Liner
Use `--ip` with a user-defined network (`docker run --network mynet --ip 172.18.0.10 myimage`), or set `ipv4_address` in docker-compose — the network must have an explicit subnet defined via `ipam` config.

---

## Full Answer

### Why You'd Want to Assign a Specific IP

Most of the time you don't need to — Docker DNS by name handles container-to-container communication. But specific IPs are useful when:
- External systems (outside Docker) need to connect to a container at a known IP.
- You're integrating with legacy systems that rely on IPs.
- Testing network configurations.
- Firewall rules are IP-based.

---

### Using `--ip` with Docker CLI

**Requirements:**
1. Must be a **user-defined network** (not the default bridge — it doesn't support `--ip`).
2. The IP must be within the network's subnet.
3. The network must have a defined subnet.

```bash
# Step 1: Create network with explicit subnet
docker network create \
  --subnet 172.20.0.0/16 \
  --gateway 172.20.0.1 \
  mynetwork

# Step 2: Run container with specific IP
docker run -d \
  --name web \
  --network mynetwork \
  --ip 172.20.0.10 \
  nginx

# Step 3: Verify
docker inspect web --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
# → 172.20.0.10
```

---

### Using docker-compose with Static IP

```yaml
version: "3.8"

services:
  web:
    image: nginx
    networks:
      mynetwork:
        ipv4_address: 172.20.0.10

  db:
    image: mysql:8.0
    networks:
      mynetwork:
        ipv4_address: 172.20.0.20

networks:
  mynetwork:
    driver: bridge
    ipam:                        # IP Address Management config
      driver: default
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1
```

---

### Important Constraints

- The IP must be **within the subnet** of the network.
- The IP must not be the **gateway address** (usually `.1`).
- The IP must not conflict with another container on the same network.
- If you try to assign an IP on the **default bridge network**, Docker will reject it.

---

### When NOT to Use Static IPs

In practice, **static IPs in Docker are rarely the right solution.** They create fragile configurations that break when:
- You move the stack to a different host.
- Another service is already using that IP.
- You scale with multiple containers (each would need a unique IP).

Prefer DNS by name:
```yaml
environment:
  - DB_HOST=db          # Let Docker DNS resolve "db" → whatever IP it has
```

Reserve static IP assignment for specific infrastructure/networking needs where an external system requires a predictable address.
