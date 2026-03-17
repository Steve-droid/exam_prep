# Three-Tier Application Architecture

## Question
Describe a three-tier application.

## Ideal One-Liner
A three-tier application separates concerns into three layers — Presentation (Frontend), Logic (Backend), and Data (Database) — each running independently and communicating through defined interfaces.

---

## Full Answer

### The Three Tiers

```
                    Internet
                       │
                  ┌────▼────┐
                  │  Nginx  │   ← Tier 1: Presentation / Reverse Proxy
                  └────┬────┘
                       │ (internal network only)
                  ┌────▼────┐
                  │ Backend │   ← Tier 2: Business Logic (Flask / Node / etc.)
                  └────┬────┘
                       │ (internal network only)
                  ┌────▼────┐
                  │   DB    │   ← Tier 3: Data (MySQL / Postgres / etc.)
                  └─────────┘
```

### Tier 1 — Frontend / Presentation Layer
- What the user interacts with: HTML/CSS/JS, React, etc.
- In production, Nginx typically serves static assets directly (no need to hit the backend for images, JS bundles, CSS).

### Tier 2 — Backend / Logic Layer
- Contains business logic: API routes, authentication, data processing.
- Receives requests forwarded by Nginx, queries the database, returns responses.
- Examples: Flask, Node.js/Express, Django.

### Tier 3 — Database Layer
- Persistent data storage: MySQL, PostgreSQL, MongoDB.
- Should **never** be directly reachable from the internet.

---

### Nginx's Role
Nginx sits at the edge and does three things:

1. **Reverse Proxy** — receives external HTTP requests and forwards them to backend containers. The client only ever talks to Nginx; it never knows the backend's address.
2. **Load Balancer** — distributes requests across multiple backend instances (round-robin, least connections, etc.).
3. **Static Asset Server** — serves files like `bundle.js`, `style.css`, and images directly from disk without touching the backend, dramatically reducing load.

---

### Networking and Security

| Layer    | Exposed to Internet? | How reached?                  |
|----------|----------------------|-------------------------------|
| Nginx    | Yes (port 80/443)    | Direct — it's the entry point |
| Backend  | No                   | Only via Nginx on internal net |
| Database | No                   | Only via Backend on internal net |

- Only Nginx has a port mapped to the host (or a public IP).
- Backend and DB live on an **internal Docker bridge network** — no external exposure.
- This limits the attack surface: even if someone probes the host, they can't reach the DB directly.

---

### Docker Compose Sketch

```yaml
version: "3.8"

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./static:/usr/share/nginx/html:ro
    networks:
      - frontend
    depends_on:
      - backend

  backend:
    build: ./backend
    networks:
      - frontend
      - backend
    environment:
      - DB_HOST=db
    depends_on:
      - db

  db:
    image: mysql:8.0
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - backend
    environment:
      - MYSQL_ROOT_PASSWORD=secret

volumes:
  db_data:

networks:
  frontend:   # Nginx ↔ Backend
  backend:    # Backend ↔ DB
```

Key points in the sketch:
- Nginx is on the `frontend` network only.
- DB is on the `backend` network only.
- Backend is on **both** networks — it bridges them.
- Named volume `db_data` ensures the database survives container restarts.

---

### Gan Shmuel Reference
In Gan Shmuel, the weight and billing services are Flask backends. They are not directly exposed to the internet — in a production hardened version, Nginx would sit in front of them. The MySQL databases (`weight-db`, `billing-db`) are on the internal compose network and are never port-mapped to the host. The bridge network lets Flask reach MySQL by hostname (`weight-db:3306`), exactly as the backend-to-DB tier pattern above.
