# Docker Bridge Networks

## Question
How do you create a bridge network in Docker?

## Ideal One-Liner
A bridge network is a private virtual network on the host that containers join to communicate with each other by container name; create one with `docker network create` or define it in docker-compose under `networks:`.

---

## Full Answer

### What Is a Bridge Network?

A bridge network is a **software bridge** (a virtual network switch) that Docker creates on the host. Containers connected to the same bridge network can communicate with each other directly, while remaining isolated from containers on other networks.

```
Host machine:
┌──────────────────────────────────────┐
│  Bridge network: my_network          │
│  172.18.0.0/16                       │
│  ┌──────────┐    ┌──────────┐        │
│  │ web      │◄──►│  db      │        │
│  │ 172.18.0.2│    │172.18.0.3│       │
│  └──────────┘    └──────────┘        │
└──────────────────────────────────────┘
```

---

### Creating a Bridge Network

#### CLI:
```bash
# Create a user-defined bridge network
docker network create mynetwork

# Create with a specific subnet
docker network create --subnet 172.20.0.0/16 mynetwork

# Attach a container to the network at run time
docker run --network mynetwork --name web nginx
docker run --network mynetwork --name db mysql:8.0

# web can now reach db by hostname "db"
```

#### Docker Compose (most common in practice):
```yaml
services:
  web:
    image: nginx
    networks:
      - app_network

  db:
    image: mysql:8.0
    networks:
      - app_network

networks:
  app_network:          # Docker creates this bridge network
    driver: bridge      # "bridge" is the default, so this line is optional
```

When you run `docker compose up`, Docker automatically creates the `app_network` bridge and connects both services to it.

---

### Default Bridge Network vs User-Defined Bridge

Docker has a **default bridge network** (`docker0`) that all containers join if no network is specified. However, it has a critical limitation:

| Feature | Default bridge | User-defined bridge |
|---|---|---|
| Container DNS by name | **No** — you can only use IP addresses | **Yes** — containers find each other by name |
| Isolation | All containers share it | Only containers you add are connected |
| Recommended? | No (legacy) | Yes |

```bash
# On default bridge: web CANNOT reach db by name
docker run --name db mysql:8.0

# On user-defined bridge: web CAN reach db by hostname "db"
docker network create mynet
docker run --network mynet --name db mysql:8.0
docker run --network mynet --name web nginx    # web → http://db:3306 works
```

---

### Inspecting Networks

```bash
docker network ls                        # list all networks
docker network inspect mynetwork         # details: subnet, connected containers, IPs
docker network connect mynetwork web     # add a running container to a network
docker network disconnect mynetwork web  # remove a container from a network
```

---

### Other Network Drivers

| Driver | Use case |
|---|---|
| `bridge` | Default for single-host container communication |
| `host` | Container shares host's network namespace (no isolation) |
| `none` | No networking — fully isolated |
| `overlay` | Multi-host communication (Docker Swarm, Kubernetes) |
| `macvlan` | Container gets its own MAC address, appears as physical device |

For most use cases (including all of Gan Shmuel), the `bridge` driver is the right choice.

---

### Gan Shmuel Reference

In `docker-compose.yml`, all services (CI, weight, billing, weight-db, billing-db) are on the same compose-defined bridge network. This is how:
- The weight Flask app can connect to `weight-db:3306` by hostname.
- The billing Flask app can connect to `billing-db:3306` by hostname.
- The CI container can reach the weight and billing health endpoints to check deployment status.

Without a user-defined bridge network, none of this name-based DNS resolution would work.
