# Docker: Image vs Container

## Question
What is the difference between a Docker image and a Docker container?

## Ideal One-Liner
A Docker image is a read-only, layered filesystem built from a Dockerfile or pulled from a registry; a container is a running instance of that image with an added writable layer and its own isolated process, network, and filesystem namespace.

## Full Elaboration

### Docker Image
- A **read-only** layered filesystem. Each instruction in a Dockerfile creates a new immutable layer.
- Layers are stacked using a union filesystem (overlay2 on most Linux systems).
- Can be built from a Dockerfile (`docker build`) or pulled from a registry (`docker pull nginx`).
- Acts as a template — it never changes when containers run from it.

### Docker Container
- A **running instance** of an image.
- Docker adds a thin **writable layer** on top of the image's read-only layers when a container starts.
- Has its own isolated: network namespace, PID namespace, and filesystem view.
- **Ephemeral by default** — the writable layer is lost when the container is removed (which is why volumes exist).

### Key Relationship
- One image → many containers simultaneously. Each container shares the read-only image layers and gets its own writable layer.
- This is why 3 containers from a 100MB Nginx image do NOT consume 300MB — the image layers are shared on the host.

### Gan Shmuel Example
In `docker-compose.yml`, the `weight` and `billing` services are each built from their own images. Multiple test and prod containers run from the same images without duplicating the base layer storage.
