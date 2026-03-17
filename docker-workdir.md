# WORKDIR in a Dockerfile

## Question
What is the purpose of WORKDIR in a Dockerfile and what happens if the specified directory doesn't exist?

## Ideal One-Liner
WORKDIR sets the working directory for all subsequent instructions in the Dockerfile (RUN, CMD, ENTRYPOINT, COPY, ADD), and Docker creates the directory automatically if it doesn't exist.

---

## Full Answer

### What WORKDIR Does

Every instruction in a Dockerfile that interacts with the filesystem uses a current working directory. `WORKDIR` sets that directory — equivalent to `mkdir -p /app && cd /app`, but cleaner and more explicit.

```dockerfile
WORKDIR /app
COPY . .         # copies into /app/
RUN ls           # runs ls from /app/
CMD ["python3", "app.py"]  # runs as if you're in /app/
```

If no WORKDIR is set, the default working directory is `/` — the root of the filesystem, which is messy and error-prone.

---

### What Happens If the Directory Doesn't Exist?

**Docker creates it automatically.** You do not need a separate `RUN mkdir -p /app` before using `WORKDIR /app`. This is one of the advantages over using `RUN cd /app` in shell commands.

```dockerfile
# This is fine — /app will be created automatically
WORKDIR /app
COPY requirements.txt .
```

---

### Why Not Just Use `cd` in RUN Commands?

```dockerfile
# Bad — this does NOT work as expected
RUN cd /app
COPY . .          # still copies to / because each RUN is a new shell

# The "cd" only persists within the same RUN command
RUN cd /app && ls    # this works, but only for this one command
```

Each `RUN` instruction starts a fresh shell. `cd` inside a `RUN` does not persist to the next instruction. `WORKDIR` is the only way to set a persistent working directory across instructions.

---

### WORKDIR Can Be Called Multiple Times

You can use WORKDIR multiple times to navigate to different directories:

```dockerfile
WORKDIR /app
COPY . .

WORKDIR /app/scripts    # changes working dir for subsequent instructions
RUN chmod +x setup.sh
```

Using relative paths with WORKDIR builds on the previous WORKDIR:
```dockerfile
WORKDIR /app
WORKDIR src         # now at /app/src
WORKDIR ..          # back to /app
```

---

### Best Practice

Always set WORKDIR early in the Dockerfile and keep your app code in a dedicated directory like `/app`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app                        # set once, early

COPY requirements.txt .             # → /app/requirements.txt
RUN pip install -r requirements.txt

COPY . .                            # → /app/<all your files>

CMD ["python3", "app.py"]           # runs python3 /app/app.py
```

This keeps the container filesystem clean and avoids accidentally placing files at the root level.
