# backend-lab-oms — Project Introduction

> **A 12-week backend learning project.** Build a real Order Management System
> from scratch using Java, Spring Boot, MySQL, Redis, Kafka, Docker, and Kubernetes.

---

## What Is This Project?

`backend-lab-oms` is a **backend-only Order Management System (OMS)** built
as a deliberate learning vehicle. It is not production software — it is a
carefully scoped project that forces you to confront real backend problems:
concurrency, consistency, failure handling, scalability, and observability.

Every technology in this stack solves a **specific, concrete problem** that
you will encounter and understand through hands-on experimentation.

---

## The System in Plain Terms

Imagine a small e-commerce operation that needs a backend to:

1. **Manage a product catalog** — add products, update prices, track available stock.
2. **Accept orders** — customers order multiple products at once; each order has line items.
3. **Prevent overselling** — if 3 units remain, only 3 units can be sold. Concurrent requests must not break this.
4. **Track order state** — every order moves through a defined lifecycle:

   ```
   PENDING → CONFIRMED → PROCESSING → SHIPPED → DELIVERED
                 ↓
             CANCELLED
   ```

5. **Process background jobs** — when an order is placed, trigger async tasks
   (e.g., reserving inventory, sending notifications) without blocking the HTTP response.

6. **Push real-time updates** — notify connected clients when an order's status changes,
   without them needing to poll.

7. **Store files** — upload and retrieve order invoices or product images via
   object storage, keeping binary data out of the database.

8. **Survive failures** — the system must degrade gracefully when a dependency
   (Redis, Kafka, a database replica) becomes unavailable.

---

## Architecture Overview

```
Client (curl / Postman / Swagger)
       │
       ▼
  [ HAProxy ]  ← reverse proxy + load balancer
       │
  ┌────┴────┐
  │  API 1  │  API 2   ← Spring Boot instances (modular monolith)
  └────┬────┘
       │
  ┌────┴──────────────────────────────────┐
  │         Spring Boot Modules           │
  │  product │ inventory │ order │ notif  │
  └────┬─────────┬──────────┬──────┬─────┘
       │         │          │      │
    MySQL      Redis      Kafka  MinIO
  (primary     (cache,    (events (files,
  database)   idempotency) async)  invoices)
       │
   MySQL Replica  ← read scaling + HA failover
```

**Why modular monolith?**
Starting with microservices adds distributed system complexity before you
understand the domain. A modular monolith enforces clean boundaries between
modules while keeping deployment simple. Splitting into services is a future
option, not a starting point.

---

## Technology Stack

| Layer | Technology | Why It's Here |
|-------|-----------|---------------|
| Language | Java 17 (LTS) | Strong typing, mature ecosystem, industry standard |
| Framework | Spring Boot 3.x | DI container, REST, JPA, Kafka, WebSocket — all integrated |
| Security | Spring Security + JWT | Authentication filter chain, stateless token-based auth, role enforcement |
| Build | Maven 3.9.x | Dependency management, reproducible builds, CI compatibility |
| Primary Database | MySQL 8.x | Relational integrity, ACID transactions, foreign key enforcement |
| Cache | Redis 7.x | Reduce read latency, store idempotency keys, invalidate refresh tokens |
| Message Broker | Apache Kafka (KRaft) | Decouple order creation from downstream processing |
| Object Storage | MinIO | Store files outside the DB; S3-compatible API |
| Real-time | WebSocket (Spring) | Push order status to clients; full-duplex connection |
| Containerization | Docker + Compose | One-command local environment; identical across machines |
| Reverse Proxy | HAProxy | Route traffic to multiple API instances; health-check backends |
| HA | Keepalived + VRRP | Virtual IP failover when a proxy node goes down |
| CI/CD | GitHub Actions | Automate test → build → image → deploy on every push |
| Orchestration | Kubernetes (kind/Minikube) | Self-healing, rolling updates, resource management |
| Unit Testing | JUnit 5 + Mockito | Fast, isolated tests for domain and service logic |
| Load Testing | k6 or JMeter | Measure throughput and latency under realistic load |
| **Frontend** | **Vue 3 + Vite** | **SPA UI: login, product catalog, order management, real-time status** |

---

## What You Will NOT Build

This project is scoped intentionally. The following are out of scope for 12 weeks:

| Out of Scope | Reason |
|-------------|--------|
| Frontend UI | Use curl, Postman, or Swagger UI instead |
| Paid cloud (AWS, GCP, Azure) | Docker Compose, Minikube, local VMs only |
| Full microservices architecture | Modular monolith first; split later with clear reason |
| Production-grade HA | Lab environment only; demo ≠ production HA |
| Real payment processing | `PaymentStrategy` is a design exercise, not real integration |
| Technologies added "for the CV" | Every tool must solve a stated problem |

---

## Learning Outcomes

After 12 weeks, you will be able to:

**Java & Spring Boot**
- Explain IoC, DI, transaction management, and the Spring request lifecycle
- Design a domain model with enforced business invariants
- Build REST APIs with proper validation, error handling, and API contracts
- Write unit tests and integration tests with real coverage rationale

**Authentication & Authorization**
- Implement registration and login with BCrypt password hashing
- Issue and validate JWT access tokens and refresh tokens
- Secure endpoints with Spring Security filter chain
- Enforce role-based access control (`ROLE_USER`, `ROLE_ADMIN`) declaratively
- Handle token expiry, refresh flow, and logout (refresh token invalidation)

**Databases**
- Design a relational schema with normalization and constraints
- Read `EXPLAIN` output and understand index selection
- Reason about ACID, isolation levels, and locking strategies
- Prevent overselling under concurrent load

**Distributed Systems**
- Implement cache-aside with TTL and explicit invalidation strategy
- Produce and consume Kafka events with retry, idempotency, and DLT
- Explain the dual-write problem and implement the transactional outbox pattern
- Push real-time state over WebSocket and handle reconnection

**Infrastructure & Operations**
- Write a multi-stage Dockerfile and tag images to Git commits
- Compose a multi-service local environment that starts in one command
- Build a CI/CD pipeline that blocks on test failure
- Explain RPO, RTO, SPOF, and test failover in a lab

**Frontend (Vue 3 + Vite)**
- Set up a Vite project with Vue Router and Pinia state management
- Call REST APIs with Axios, handle JWT tokens in the client
- Build login, register, product list, and order management pages
- Receive real-time order status updates over WebSocket
- Upload files from the browser to the backend (MinIO)
- Containerize the frontend with Nginx and wire it in Docker Compose

**System Design**
- Draw C4 context and container diagrams for the system
- Articulate trade-offs: why this design, what it sacrifices, when it breaks
- Present an architecture with failure modes, bottlenecks, and scale strategy

---

## Project Principles

1. **No learning without evidence.** Every session must produce something verifiable:
   code, a passing test, a benchmark result, a diagram, or a written decision record.

2. **Documentation is part of the work.** If it's not written down, it didn't happen.
   The repo must be understandable by a stranger — or by you six months from now.

3. **Small, honest commits.** 2–3 commits per day. Each commit message describes
   what actually changed, not what you planned to change.

4. **Fail deliberately.** Break things on purpose. A failure lab with a written
   incident report is worth more than a system that "seems to work."

5. **No cargo cult.** Every technology must earn its place. If you cannot state
   the problem it solves, remove it.

---

## Repository Conventions

| Convention | Rule |
|-----------|------|
| Branch naming | `feature/weekNN-topic`, e.g. `feature/week01-domain-model` |
| Commit format | Conventional Commits: `feat(order): add state transition logic` |
| Secrets | Never committed. Use `.env.example` with placeholders only |
| Docker image tags | Always pin to version, never use `latest` in `compose.yaml` |
| Definition of Done | See [git-workflow.md](./git-workflow.md) — 5-point checklist |

---

## Documentation Map

| File | Purpose |
|------|---------|
| `docs/introduction.md` | ← You are here. Project overview. |
| `docs/requirements.md` | Business and non-functional requirements |
| `docs/environment.md` | Exact tool versions and local setup steps |
| `docs/git-workflow.md` | Branch strategy, commit conventions, daily flow |
| `docs/auth.md` | Auth design: JWT flow, token lifecycle, Spring Security setup |
| `docs/frontend.md` | Frontend architecture: Vue 3, routing, state, API client |
| `docs/architecture/context.md` | C4 Context diagram — system boundary |
| `docs/architecture/containers.md` | C4 Container diagram — services and connections |
| `docs/database/schema.md` | Tables, columns, indexes, FK relationships |
| `docs/api/` | REST endpoint contracts per module |
| `docs/adr/` | Architecture Decision Records |
| `docs/runbooks/` | Operational playbooks for common failure scenarios |
| `docs/incidents/` | Post-mortems from failure lab exercises |
| `docs/test-reports/` | Benchmark results with environment context |
| `docs/learning-log.md` | Daily journal — what was learned, tested, broken |
| `docs/weekly-reviews/` | End-of-week self-assessment documents |

---

*Last updated: Week 0 — project initialization.*
