# Linking Containers

## Question
How do you link one container with another? What are the modern alternatives?

## Ideal One-Liner
The old `--link` flag is deprecated; the modern approach is to place containers on the same user-defined bridge network, where Docker's built-in DNS lets them reach each other by container name automatically.

---

## Full Answer

### The Old Way: `--link` (Deprecated)

```bash
docker run -d --name db mysql:8.0
docker run -d --name web --link db:database nginx
```

What `--link` did:
1. Added an entry to the web container's `/etc/hosts`: `<db IP> database db`
2. Injected environment variables into web like: `DATABASE_PORT=tcp://172.17.0.2:3306`
3. Only worked on the default bridge network.

**Why it was deprecated:**
- Static — only worked in one direction (web knows about db, but db doesn't know about web).
- No DNS — just `/etc/hosts` entries that could go stale.
- Did not scale — managing links across many containers was complex.
- Bound to the default bridge network, which doesn't support modern features.
- The `--link` flag is still technically in Docker, but it's considered legacy and may be removed.

---

### The Modern Way: User-Defined Bridge Networks

```bash
# Create a network
docker network create mynetwork

# Run containers on the same network
docker run -d --name db --network mynetwork mysql:8.0
docker run -d --name web --network mynetwork nginx
```

Now `web` can reach `db` by hostname:
```bash
# From inside the web container:
curl http://db:80
mysql -h db -u root -p
```

Docker's embedded DNS server automatically resolves container names to their current IPs.

---

### With Docker Compose (Best Practice)

In docker-compose, services on the same compose file are **automatically on the same default network** and can reach each other by service name. No explicit linking or network definition required (though defining a network is still good practice for clarity).

```yaml
services:
  web:
    image: nginx
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: secret
```

`web` can connect to `db:3306` — Docker compose creates a bridge network and connects both services automatically.

```yaml
# Explicit network definition (optional but clearer):
services:
  web:
    networks:
      - backend
  db:
    networks:
      - backend

networks:
  backend:
```

---

### Summary: Old vs New

| Aspect | `--link` (old) | User-defined network (modern) |
|---|---|---|
| DNS by name | Partial (hosts entry) | Full DNS resolution |
| Bidirectional | No | Yes |
| Scalable | No | Yes |
| Works with compose | Limited | Native |
| Status | Deprecated | Recommended |
| Environment variable injection | Yes (--link side effect) | No (use explicit env vars) |

---

### Gan Shmuel Reference
In Gan Shmuel, there are no `--link` flags anywhere. The `docker-compose.yml` defines services that all join the same compose-managed bridge network. The billing service connects to `billing-db:3306` and the weight service connects to `weight-db:3306` — pure DNS by service name, no linking required. This is the correct modern approach.
