# Docker Multi-Stage Builds

## Question
What is a multi-stage build and when do you use it?

## Ideal One-Liner
A multi-stage build uses multiple FROM instructions in a single Dockerfile so you can build artifacts in a fat image and copy only what's needed into a minimal final image — eliminating build tools and intermediate files from production.

---

## Full Answer

### The Problem Multi-Stage Builds Solve

Without multi-stage builds, you face a choice:
- Include build tools in the image → large, insecure production image.
- Use a separate build script and manual artifact copying → complex, error-prone workflow.

Multi-stage builds solve this in one Dockerfile.

---

### Structure

```dockerfile
# Stage 1: Build stage (fat image with tools)
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o server .

# Stage 2: Production stage (minimal image)
FROM alpine:3.19
WORKDIR /app
COPY --from=builder /app/server .    ← copies only the binary from stage 1
CMD ["/app/server"]
```

Key points:
- Each `FROM` starts a new stage.
- You can name stages with `AS <name>` for clarity.
- `COPY --from=<stage>` pulls specific files from a previous stage.
- The final image only contains what's in the **last stage** — everything from earlier stages is discarded.

---

### Size Comparison Example (Go App)

| Approach | Image size |
|---|---|
| Single stage with golang:1.21 | ~800MB |
| Multi-stage: final on alpine:3.19 | ~10MB |
| Multi-stage: final on distroless | ~6MB |

The Go toolchain, source code, and build artifacts from stage 1 never appear in the final image.

---

### Python Example

```dockerfile
# Stage 1: Install dependencies
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Production runtime
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local   ← installed packages
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python3", "app.py"]
```

Final image: ~130MB instead of ~900MB.

---

### When to Use Multi-Stage Builds

| Scenario | Benefit |
|---|---|
| Compiled languages (Go, Rust, C, Java) | Build binary in fat image, deploy only binary |
| Python/Node with heavy build deps (native extensions) | Build in fat image, install in slim |
| Any production image that needs build tools (gcc, make) | Remove build tools from final image |
| Security-sensitive deployments | No compiler = no compiler exploits |
| CI/CD where image size matters | Smaller images pull faster |

---

### Advanced: Multiple Build Stages

```dockerfile
FROM node:20 AS frontend-builder
WORKDIR /frontend
COPY frontend/ .
RUN npm install && npm run build

FROM python:3.11 AS backend-builder
WORKDIR /backend
COPY requirements.txt .
RUN pip install --user -r requirements.txt

FROM python:3.11-slim AS production
WORKDIR /app
COPY --from=frontend-builder /frontend/dist /app/static
COPY --from=backend-builder /root/.local /root/.local
COPY backend/ .
CMD ["python3", "app.py"]
```

You can have as many stages as needed. Docker only builds the stages needed for the final target.

---

### Targeting a Specific Stage

```bash
# Build only the builder stage (useful for CI caching)
docker build --target builder -t myapp:builder .

# Build the full image
docker build -t myapp:latest .
```

---

### Gan Shmuel Reference
The weight and billing Flask services in Gan Shmuel use single-stage builds. If optimized with multi-stage builds: Stage 1 would use `python:3.11` to install dependencies (including any native extensions), Stage 2 would use `python:3.11-slim` with only the installed packages and application code — reducing the image from ~900MB to ~150MB. The CI container (`ci/Dockerfile`) intentionally stays fat because it needs the Docker CLI and git available at runtime, not just build time.
