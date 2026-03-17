# Dockerfile and Best Practices

## Question
What is a Dockerfile and what are the best practices for writing one?

## Ideal One-Liner
A Dockerfile is a text file with sequential instructions that Docker executes layer by layer to build an image; best practices focus on minimizing image size, maximizing cache efficiency, and avoiding security pitfalls.

---

## Full Answer

### What Is a Dockerfile?

A Dockerfile is a recipe for building a Docker image. Docker reads it top to bottom, executing each instruction and creating a new **image layer** for each one. The final image is the stack of all layers.

```dockerfile
FROM python:3.11-slim          # base image layer
WORKDIR /app                   # set working directory
COPY requirements.txt .        # copy file (new layer)
RUN pip install -r requirements.txt  # run command (new layer)
COPY . .                       # copy app code (new layer)
CMD ["python3", "app.py"]      # default command (metadata, no new layer)
```

---

### Best Practices

#### 1. Use a Minimal Base Image
```dockerfile
# Bad
FROM ubuntu:22.04

# Good
FROM python:3.11-slim
# or
FROM python:3.11-alpine
```
Smaller base = smaller final image, fewer CVEs, faster pulls.

#### 2. Combine RUN Commands to Reduce Layers
```dockerfile
# Bad — creates 3 layers
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

# Good — creates 1 layer
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```
Each `RUN` creates a layer. Combining them reduces layer count and total image size. **Critically: clean up in the same `RUN` layer** — if you delete files in a later layer, they're still in the earlier layer and counted in the image size.

#### 3. Use .dockerignore
```
# .dockerignore
.git
__pycache__
*.pyc
node_modules
.env
tests/
```
Prevents the build context from sending unnecessary files to the Docker daemon. Speeds up builds and prevents secrets leaking in.

#### 4. Order Instructions for Cache Efficiency
Docker caches layers. If a layer changes, all layers after it are invalidated.

```dockerfile
# Good: dependencies first (change rarely), code last (changes often)
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .                          # this line busts cache on every code change,
                                  # but the pip install layer is cached ✓

# Bad: everything copies at once, pip reinstalls on every code change
COPY . .
RUN pip install -r requirements.txt
```

#### 5. Use COPY Not ADD (Unless You Need tar Extraction)
```dockerfile
# Use COPY for simple file copying
COPY ./app /app

# Only use ADD if you need automatic tar extraction
ADD archive.tar.gz /app
```

#### 6. Use Specific Image Tags
```dockerfile
# Bad — :latest can change unexpectedly, breaks reproducibility
FROM python:latest

# Good
FROM python:3.11.9-slim
```

#### 7. Run as Non-Root User
```dockerfile
RUN useradd --create-home appuser
USER appuser
```
Containers run as root by default. If an attacker breaks out of the app, running as root gives them more power on the host.

#### 8. Use Multi-Stage Builds for Compiled Languages
```dockerfile
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o server .

FROM alpine:3.19
COPY --from=builder /app/server /server
CMD ["/server"]
```
Final image has only the binary — no Go toolchain.

#### 9. Set WORKDIR Explicitly
```dockerfile
WORKDIR /app    # creates directory if it doesn't exist, cleaner than cd
```

---

### Example Flask Dockerfile Using These Practices

```dockerfile
FROM python:3.11-slim

# Non-root user
RUN useradd --create-home appuser

WORKDIR /app

# Dependencies first (cached separately from code)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# App code (changes most often — goes last)
COPY . .

USER appuser

EXPOSE 5000

CMD ["python3", "app.py"]
```

---

### Gan Shmuel Reference
The CI service in Gan Shmuel (`ci/Dockerfile`) installs git and Docker CE CLI in a single `RUN` layer combining `apt-get update`, `apt-get install`, and cleanup — following the best practice of combining commands and cleaning up in the same layer to minimize image size.
