# Docker Images and Layers

## Question
How are Docker images built, and what are image layers?

## Ideal One-Liner
A Docker image is built instruction-by-instruction from a Dockerfile — each filesystem-modifying instruction creates a new read-only layer, and the final image is the complete stack.

---

## Full Answer

### How Images Are Built
Docker reads the Dockerfile top-to-bottom. `RUN`, `COPY`, `ADD` create new layers. `ENV`, `EXPOSE`, `CMD`, `ENTRYPOINT` add metadata only (no layer).

```dockerfile
FROM ubuntu:22.04          # base layer
RUN apt-get install -y python3  # new layer
COPY app.py /app/          # new layer
CMD ["python3", "/app/app.py"]  # metadata only
```

### What Are Layers
Each layer is a **diff** — only what changed vs the layer below. Layers are:
- **Read-only** — immutable once built
- **Shared** — two images with the same base share that layer on disk
- **Cached** — unchanged layers are reused on rebuild

A running container gets one **writable layer** on top. It's lost when the container is removed.

### Why Order Matters (Caching)
```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt  # cached unless requirements.txt changes
COPY . .                             # changes often — put last
```
Putting `COPY . .` first busts the pip cache on every code change.

### Useful Commands
```bash
docker history myapp:1.0   # layers, sizes, instructions
docker inspect myapp:1.0   # full metadata
```

In Gan Shmuel: the CI Dockerfile's `apt-get install` layer is cached after the first build. MySQL images are shared between prod and test compose files — stored once on the EC2 host.
