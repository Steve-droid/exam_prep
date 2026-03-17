# ADD vs COPY in a Dockerfile

## Question
What is the difference between ADD and COPY in a Dockerfile?

## Ideal One-Liner
COPY simply copies files from the build context into the image; ADD does the same but also auto-extracts tar archives and can fetch from URLs — prefer COPY unless you specifically need those extra features.

---

## Full Answer

### COPY — Simple and Explicit

`COPY` copies files or directories from the **build context** (the directory you pass to `docker build`) into the image. That's all it does.

```dockerfile
COPY ./app /app              # copies ./app directory into /app in the image
COPY requirements.txt .      # copies requirements.txt to WORKDIR
COPY src/ /app/src/          # copies a subdirectory
```

**What COPY cannot do:**
- Does not extract archives.
- Cannot fetch from remote URLs.
- Cannot copy files from outside the build context.

---

### ADD — COPY Plus Extra Powers

`ADD` does everything `COPY` does, plus:

1. **Auto-extracts tar archives:**
```dockerfile
ADD archive.tar.gz /app/     # extracts the tar into /app/ automatically
```

2. **Fetches from URLs:**
```dockerfile
ADD https://example.com/file.tar.gz /tmp/
```

---

### Best Practice: Prefer COPY

The Docker documentation explicitly recommends preferring COPY over ADD unless you specifically need the extra functionality. Why?

1. **COPY is explicit and predictable.** You always know exactly what happens.
2. **ADD's URL fetching is a bad practice:**
   - The downloaded file is not cached in a useful way (cache is busted by URL, not file content).
   - You can't verify the file's integrity easily.
   - Better to use `RUN curl` or `RUN wget` for URL fetching — more transparent, better cache control.

```dockerfile
# Bad — using ADD to fetch a URL
ADD https://example.com/app.tar.gz /tmp/

# Good — explicit and cache-efficient
RUN curl -sSL https://example.com/app.tar.gz -o /tmp/app.tar.gz && \
    tar -xzf /tmp/app.tar.gz -C /app && \
    rm /tmp/app.tar.gz
```

3. **ADD's auto-extraction can be surprising:**
   - If you `ADD` a `.tar.gz` file intending to just copy it (not extract), ADD will extract it anyway.
   - This can silently do more than you intended.

---

### When ADD Is Acceptable

The only legitimate use case for ADD over COPY:

```dockerfile
# Extracting a local tar archive into the image
ADD app-source.tar.gz /app/
```

This is cleaner than:
```dockerfile
COPY app-source.tar.gz /tmp/
RUN tar -xzf /tmp/app-source.tar.gz -C /app && rm /tmp/app-source.tar.gz
```

Even then, consider whether it's worth the implicit behavior.

---

### Summary

| Feature | COPY | ADD |
|---|---|---|
| Copy local files | Yes | Yes |
| Auto-extract tar | No | Yes |
| Fetch from URL | No | Yes (not recommended) |
| Recommended for? | All normal file copying | Local tar extraction only |
| Explicit / predictable | Yes | Less so |
