# Monolithic vs Microservices Architecture

## Question
When would you choose a monolithic architecture over microservices?

## Ideal One-Liner
Choose a monolith when the team is small, the domain is simple, or the project is early-stage — microservices add significant operational overhead that only pays off at scale.

---

## Full Answer

### What Is a Monolith?
A single deployable unit where all functionality (auth, billing, UI, business logic) lives in one codebase and is deployed together.

### What Are Microservices?
The application is split into independently deployable services, each responsible for one domain (e.g., user service, billing service, notifications service). They communicate over HTTP/gRPC/message queues.

---

### Microservices Complexity Overhead
Before choosing microservices, you inherit all of these costs:

| Concern | What it means |
|---|---|
| **Network calls** | Instead of in-process function calls, services talk over HTTP — adds latency and failure modes |
| **Distributed tracing** | A request spans multiple services; you need tools like Jaeger or Zipkin to trace it |
| **Service discovery** | Services need to find each other dynamically (Consul, Kubernetes DNS, etc.) |
| **Deployment complexity** | Each service has its own CI/CD pipeline, Docker image, versioning, rollback strategy |
| **Data consistency** | No shared DB means eventual consistency, distributed transactions (saga pattern) |
| **Operational overhead** | More containers, more logs, more alerts, more things to go wrong |

### When a Monolith Makes Sense

1. **Small team** — a 2–5 person team can't afford to maintain 10 independent services. A monolith lets everyone move fast.
2. **Early-stage project** — domain boundaries aren't clear yet. Splitting too early leads to wrong service boundaries that are expensive to fix.
3. **Simple domain** — if the business logic is not highly complex and doesn't have truly independent scaling needs, a monolith is simpler.
4. **Faster iteration** — one deploy, one test suite, one repo. No API contract negotiations between teams.
5. **YAGNI** — "You Ain't Gonna Need It." Don't build for future scale that may never come.

> "A well-structured monolith is better than a poorly designed set of microservices."

---

### Migration Path — Strangler Fig Pattern
When a monolith needs to be broken up:

1. **Identify a seam** — find a bounded context (e.g., billing) that has clear inputs and outputs.
2. **Build the new service** alongside the monolith.
3. **Route traffic** to the new service (via a proxy/API gateway) while the monolith still handles the rest.
4. **Strangle the old code** — once the new service is stable, remove the equivalent code from the monolith.
5. Repeat for each domain, incrementally.

```
Before:                    During:                    After:
┌──────────────┐           ┌──────────────┐           ┌─────────────┐  ┌─────────┐
│   Monolith   │           │   Monolith   │           │  Monolith   │  │ Billing │
│  - auth      │    →      │  - auth      │    →      │  - auth     │  │ Service │
│  - billing   │           │  - orders    │           │  - orders   │  └─────────┘
│  - orders    │           │  [billing ──►┼──► Billing│             │
└──────────────┘           └──────────────┘  Service] └─────────────┘
```

---

### Gan Shmuel Reference
Gan Shmuel is itself a small-scale microservices project: weight service and billing service are separate Flask apps with separate databases. This worked well for the bootcamp because the domain boundaries were pre-defined by the exercise. In a real startup scenario, this same system might have started as a monolith until the team understood the domain well enough to split it cleanly.
