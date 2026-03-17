# Saving Changes Made to a Running Container

## Question
How do you save changes made to a running container?

## Ideal One-Liner
Use `docker commit` to snapshot a container's current state into a new image — but this is a debugging tool, not a build mechanism; the correct production approach is to update the Dockerfile and rebuild.

---

## Full Answer

### How Container Storage Works

When a container starts, Docker adds a thin **writable layer** on top of the read-only image layers. All changes you make inside the running container (installing packages, editing files, writing data) go into this writable layer.

```
┌─────────────────────────┐  ← writable container layer (your changes go here)
├─────────────────────────┤
│    Image Layer 3         │  ← COPY . .
├─────────────────────────┤
│    Image Layer 2         │  ← RUN pip install
├─────────────────────────┤
│    Image Layer 1         │  ← FROM python:3.11-slim
└─────────────────────────┘
```

When the container is removed, the writable layer is **destroyed** — changes are lost unless you save them.

---

### Method 1: `docker commit` (Quick, Not Recommended for Production)

```bash
# Snapshot the container's current state into a new image
docker commit <container_name_or_id> myimage:v2

# With a commit message
docker commit -m "Installed curl and vim for debugging" web myimage:debug

# View the new image
docker images
docker history myimage:v2
```

This creates a new image containing all layers from the original image **plus** your changes as an additional layer.

**When `docker commit` is useful:**
- Debugging snapshots — freeze the state of a broken container to investigate later.
- Quick experiments — test a change without writing a Dockerfile.
- Saving a container you accidentally configured correctly and want to preserve.

**Why NOT to use it in production:**
- **Not reproducible** — no text record of what was changed or why.
- **Not version-controlled** — changes aren't tracked in Git.
- **Opaque layers** — `docker history` shows `<missing>` for committed changes.
- **Violates immutable infrastructure** principle — every deployment should build from a clean, known Dockerfile.

---

### Method 2: The Correct Approach — Update the Dockerfile

If the change is something that should be permanent and reproducible:

1. Identify what you changed inside the container.
2. Add those changes to the Dockerfile (e.g., add a `RUN apt-get install curl` line).
3. Rebuild: `docker build -t myimage:v2 .`
4. Redeploy.

```dockerfile
# Before:
FROM python:3.11-slim
COPY . /app
CMD ["python3", "app.py"]

# After (added curl for health checks):
FROM python:3.11-slim
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
COPY . /app
CMD ["python3", "app.py"]
```

This is declarative, version-controlled, and reproducible.

---

### For Persistent Data: Use Volumes

If the data you want to "save" is application data (database records, uploaded files, logs) — that's not a container snapshot problem. Use **volumes**:

```bash
docker run -v mydata:/app/data myimage
```

The data in `/app/data` persists in the `mydata` volume regardless of whether the container is stopped, removed, or recreated.

---

### Summary

| Approach | Use case | Production? |
|---|---|---|
| `docker commit` | Debugging, experiments, snapshots | No |
| Rebuild from Dockerfile | Any permanent change to the image | Yes, always |
| Volumes | Persistent runtime data (DB, files) | Yes |
