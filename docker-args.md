# Docker ARG — Build-Time Variables

## Question
How do ARGs work in a Dockerfile and how are they used?

## Ideal One-Liner
ARG defines a build-time variable that can be passed via `docker build --build-arg`, is available only during the build (not at runtime), and should never be used for secrets since it appears in `docker history`.

---

## Full Answer

### What Is ARG?

`ARG` declares a variable that can be passed into the Dockerfile **at build time** using `--build-arg`. It is available to instructions that execute during the build (like `RUN`, `COPY`, `FROM`).

```dockerfile
ARG APP_VERSION=1.0.0           # declare with optional default value
RUN echo "Building version $APP_VERSION"
```

```bash
docker build --build-arg APP_VERSION=2.5.1 -t myapp:2.5.1 .
```

---

### ARG vs ENV — Key Difference

| Feature | ARG | ENV |
|---|---|---|
| Available during build | Yes | Yes |
| Available at runtime (inside running container) | **No** | Yes |
| Visible in `docker history` | Yes | Yes |
| Safe for secrets? | **No** | No (also visible in inspect) |

`ARG` values exist **only during the build process**. Once the image is built, the ARG value is gone — it is not stored in the container's environment. If you need a value at runtime, use `ENV`.

---

### Common Use Cases

#### 1. Parameterize base image version
```dockerfile
ARG PYTHON_VERSION=3.11
FROM python:${PYTHON_VERSION}-slim
```

```bash
docker build --build-arg PYTHON_VERSION=3.12 .
```

#### 2. Pass version/tag information
```dockerfile
ARG APP_VERSION=dev
ARG GIT_COMMIT=unknown

RUN echo "$APP_VERSION ($GIT_COMMIT)" > /app/version.txt
```

#### 3. Conditional package installation
```dockerfile
ARG INSTALL_DEBUG_TOOLS=false

RUN if [ "$INSTALL_DEBUG_TOOLS" = "true" ]; then \
      apt-get install -y curl vim; \
    fi
```

#### 4. Registry or image path
```dockerfile
ARG REGISTRY=registry.mycompany.com
FROM ${REGISTRY}/base-python:3.11
```

---

### ARG Scope

**ARG before FROM:** Available only in the `FROM` instruction.

```dockerfile
ARG BASE_VERSION=3.11
FROM python:${BASE_VERSION}-slim      # works

RUN echo $BASE_VERSION                # empty — ARG before FROM not available after FROM
```

To use it after FROM, re-declare it:
```dockerfile
ARG BASE_VERSION=3.11
FROM python:${BASE_VERSION}-slim
ARG BASE_VERSION                      # re-declare to make it available again
RUN echo $BASE_VERSION                # now works
```

**ARG scope in multi-stage builds:** An ARG declared in one stage is not available in subsequent stages. Re-declare in each stage.

---

### ARG and the Build Cache

ARG affects the build cache. If you change an `--build-arg` value, Docker invalidates the cache at the point where that ARG is first used:

```dockerfile
ARG APP_VERSION
RUN echo $APP_VERSION     # cache invalidated if APP_VERSION changes
COPY . .                  # also invalidated (downstream)
```

Place ARG declarations as late as possible if you want to maximize cache reuse.

---

### Why ARG Is NOT Safe for Secrets

```dockerfile
ARG DB_PASSWORD=super_secret
RUN ./setup.sh           # DB_PASSWORD used here
```

```bash
docker history myimage
# shows: ARG DB_PASSWORD=super_secret  ← visible to anyone with the image
```

For build-time secrets, use Docker BuildKit's `--secret` flag instead:
```bash
docker build --secret id=mysecret,src=./secret.txt .
```
```dockerfile
# syntax=docker/dockerfile:1
RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret
```
This does not persist the secret in any image layer.
