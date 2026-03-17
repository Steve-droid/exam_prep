# Docker Image Shared Layers

## Question
If an Nginx image weighs 100MB and you deploy 3 containers from it, will the total disk usage be 300MB?

## Ideal One-Liner
No — image layers are read-only and shared; Docker stores them once, so 3 containers from a 100MB image costs ~100MB on disk, not 300MB.

---

## Full Answer

The 100MB image layers are **read-only and shared** — Docker stores them once on disk regardless of how many containers run from the image. Each container gets its own thin **writable layer** on top, but that's near-zero unless the container actively writes data.

Total: **~100MB**, not 300MB.

This is Docker's **copy-on-write** storage model — a file from the image is only copied into a container's writable layer if that container modifies it.
