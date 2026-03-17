# Alpine Image

## Question
What is an Alpine image and when should it be used?

## Ideal One-Liner
Alpine is a ~5MB minimal Linux distribution using `musl` `libc` and `BusyBox`, used as a lightweight, low attack-surface base image.

---

## Full Answer

Alpine is built on **`musl` `libc`** and `**BusyBox**` instead of `glibc` and GNU `coreutils` — no kernel included (containers share the host kernel). Result: ~5MB vs Ubuntu's ~80MB.

**Use when:**
- You want a minimal runtime base for a compiled binary or script
- Final stage of a multi-stage build — build in a full image, copy artifact, run in Alpine
- Quick shell environment: `docker run -it alpine sh`

**Trade-off:** `musl` `libc` can break binaries compiled against `glibc`. Use `debian:slim` in that case.

**Package manager:** `apk add <package>`
