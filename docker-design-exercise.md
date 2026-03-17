# Docker Architecture Design Exercise

## Question
Design an architecture with two containers demonstrating all three volume types, proper port mapping, network connectivity, data persistence, and container communication.

## Ideal One-Liner
Two containers — a Flask web app and a MySQL database — connected on a bridge network, with a named volume for DB persistence, a host volume for logs, and an anonymous volume for temp scratch space, with port mapping only on the Flask container.

---

## Full Answer

### Architecture Overview

```
Host Machine
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  Port 8080 ──► [Flask App Container]  ◄──► [MySQL Container]  │
│                     │                           │             │
│                /app/logs                 /var/lib/mysql        │
│                     │                           │             │
│              Host Volume            Named Volume (db_data)     │
│            /home/steve/logs                                    │
│                                                                │
│  Bridge Network: app_network                                   │
└────────────────────────────────────────────────────────────────┘
```

### Volume Usage

| Volume Type | Where used | Purpose |
|---|---|---|
| **Named volume** (`db_data`) | MySQL `/var/lib/mysql` | Persist database files across container restarts |
| **Host volume** (bind mount) | Flask `/app/logs` → `/home/steve/app-logs` | Access app logs from host without exec-ing in |
| **Anonymous volume** | Flask `/tmp/cache` | Temporary scratch space (cache) that doesn't need persistence |

---

### Docker Compose File

```yaml
version: "3.8"

services:

  # ── Flask Web Application ──────────────────────────────────────
  web:
    build: ./web                      # builds from ./web/Dockerfile
    container_name: flask_app
    ports:
      - "8080:5000"                   # host:container port mapping
    environment:
      - DB_HOST=db                    # use service name for DNS resolution
      - DB_USER=appuser
      - DB_PASSWORD=securepassword
      - DB_NAME=myapp
    volumes:
      # Host volume (bind mount): app logs accessible from host
      - /home/steve/app-logs:/app/logs
      # Anonymous volume: temp scratch space (Docker assigns random name)
      - /tmp/cache
    networks:
      - app_network
    depends_on:
      - db
    restart: on-failure:3

  # ── MySQL Database ────────────────────────────────────────────
  db:
    image: mysql:8.0
    container_name: mysql_db
    # No ports: exposed — NOT accessible from outside
    environment:
      - MYSQL_ROOT_PASSWORD=rootsecret
      - MYSQL_DATABASE=myapp
      - MYSQL_USER=appuser
      - MYSQL_PASSWORD=securepassword
    volumes:
      # Named volume: database files persist across container removal
      - db_data:/var/lib/mysql
    networks:
      - app_network
    restart: unless-stopped

# ── Volume Declarations ─────────────────────────────────────────
volumes:
  db_data:                            # Docker-managed named volume

# ── Network Declaration ─────────────────────────────────────────
networks:
  app_network:
    driver: bridge
```

---

### Key Design Decisions Explained

#### Port Mapping
- **Flask:** `8080:5000` — external users hit port 8080 on the host, Docker forwards to port 5000 in the container (where Flask listens).
- **MySQL:** No port mapping — the DB is internal-only. External access is blocked. `web` reaches it via `db:3306` on the internal network.

#### Network Connectivity
- Both services are on `app_network` (user-defined bridge).
- Flask connects to MySQL using the service name `db` → Docker DNS resolves this to the container's internal IP.
- Even if the DB container is recreated and gets a new IP, the Flask app still connects to `db:3306` — DNS handles it.

#### Data Persistence
- MySQL data in `db_data` (named volume) survives `docker compose down` and `docker compose up`.
- Data is only destroyed with `docker compose down -v` (which removes volumes too).

#### Container Communication
```python
# In Flask app:
import mysql.connector
conn = mysql.connector.connect(
    host="db",          # service name = hostname on the bridge network
    user="appuser",
    password="securepassword",
    database="myapp"
)
```

---

### Dockerfile for Flask Service (Reference)

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Create directories for volumes
RUN mkdir -p /app/logs /tmp/cache

COPY . .

EXPOSE 5000

CMD ["python3", "app.py"]
```

---

### Gan Shmuel Reference

Gan Shmuel's `docker-compose.yml` is essentially this pattern at a larger scale:
- **Named volumes:** `mysql_weight_data` and `mysql_billing_data` for the two databases.
- **Host volume:** The project repo is mounted into the CI container (`/home/steve/bootcamp/...:/repo`) so git pull results are visible inside the container without rebuilding the image.
- **Port mapping:** Weight (8080:8080), Billing (8081:8080), CI (8085:8085). Test stack uses different host ports (8082, 8083) while internal container ports stay the same.
- **Bridge network:** All services communicate by service name — `weight-db`, `billing-db`.
- **No exposed DB ports:** The MySQL containers have no `ports:` mapping — they're accessible only within the compose network.
