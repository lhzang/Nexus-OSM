# Git Workflow

This document defines how Git is used in `backend-lab-oms` —
branch strategy, commit message format, daily flow, and the rules
for when to branch and when to commit.

---

## Branch Strategy

```
main ─────────────────────────────────────────  (stable, always green)
  └── develop ─────────────────────────────────  (integration target)
        ├── feature/week01-domain-model
        ├── feature/week02-spring-api
        ├── feature/week02-ci-pipeline
        ├── feature/week03-mysql-schema
        └── ...
```

### Branch Rules

| Branch | Rule |
|--------|------|
| `main` | Always compiles. Always passes tests. Never commit directly to main. |
| `develop` | Integration target. All feature branches merge here. |
| `feature/xxx` | One per roadmap topic or logical unit. Branch from `develop`; merge back to `develop`. |

---

## When to Create a New Branch

Create a new `feature/` branch when you start:
- A new week topic
- A distinct piece of work that can be reviewed independently

```bash
# New week = new branch
git checkout -b feature/week03-mysql-schema

# Same week, different concern = separate branch
git checkout -b feature/week02-spring-api
git checkout -b feature/week02-ci-pipeline
```

**Do not** create a branch for every file or every small edit.
A branch represents a coherent unit of work, not a single line change.

---

## When to Commit Within a Branch

Commit whenever you complete a **small, verifiable unit of work**.
Do not wait until the end of the session to commit everything.

**Daily KPI: 2–3 commits per session.**

Plan your session around three natural checkpoints:

| Commit # | Type | Contains |
|----------|------|----------|
| 1 | `chore` / `docs` | Setup, scaffolding, config, initial files |
| 2 | `feat` / `refactor` | Core implementation — the thing you set out to build |
| 3 | `test` / `docs` | Tests proving it works + learning log update |

---

## Commit Message Convention — Conventional Commits

```
<type>(<scope>): <short description>

[optional body — use when the "why" is not obvious]

[optional footer — e.g., Refs #12, Breaking change]
```

### Types

| Type | When to Use |
|------|-------------|
| `feat` | New feature or business capability added |
| `fix` | Bug fix — something was broken and is now correct |
| `chore` | Setup, config, tooling, build scripts — no logic change |
| `docs` | Documentation only — no code change |
| `test` | Adding or correcting tests — no production code change |
| `refactor` | Code restructure with no behavior change |
| `perf` | Performance improvement with measurable evidence |
| `style` | Formatting, whitespace — no logic change |

### Scopes (use the module or area)

`product`, `inventory`, `order`, `payment`, `notification`, `shared`,
`db`, `redis`, `kafka`, `websocket`, `minio`, `docker`, `ci`, `k8s`, `docs`

### Examples

```bash
# Day 1 — Setup
git commit -m "chore: initialize Maven project structure with Java 17"
git commit -m "chore(git): add .gitignore for Java, Maven, and IntelliJ"
git commit -m "docs: add README, introduction.md, and environment.md"

# Week 1 — Domain model
git commit -m "feat(product): add Product class with encapsulated price and stock"
git commit -m "test(product): verify Product rejects negative price and zero quantity"
git commit -m "docs: add learning log entry for domain model session"

# Week 2 — Spring Boot
git commit -m "feat(product): add ProductService and ProductRepository stubs"
git commit -m "feat(product): expose GET /products/{id} endpoint with DTO mapping"
git commit -m "test(product): add integration test for 404 on missing product"

# Week 3 — Database
git commit -m "chore(db): add Flyway migration V1 for products and orders schema"
git commit -m "perf(db): add composite index on orders(customer_id, status, created_at)"
git commit -m "docs(db): record EXPLAIN output and index selection in query-benchmarks.md"

# Architecture decisions
git commit -m "docs(adr): ADR-0001 — choose cache-aside over write-through for product cache"
git commit -m "docs(adr): ADR-0002 — transactional outbox for reliable Kafka publish"
```

### Rules for Good Commit Messages

- **Imperative mood:** "add", "fix", "remove" — not "added", "fixed"
- **Under 72 characters** for the subject line
- **Lowercase** after the type prefix
- **No period** at the end of the subject line
- **No vague messages:** `fix bug`, `update code`, `wip` are not acceptable
- **Body explains WHY**, not what (the diff shows what)

---

## Full Daily Git Flow

```bash
# ── Start of session ──────────────────────────────────────────────
git checkout develop
git status                        # make sure you're clean

git checkout -b feature/weekNN-topic

# ── During session — commit as you go ────────────────────────────
git add <files>
git commit -m "chore: scaffold product module package structure"

git add <files>
git commit -m "feat(product): add Product domain class with invariant checks"

git add <files>
git commit -m "test(product): add unit tests for price and stock validation"

git add docs/learning-log.md
git commit -m "docs: update learning log for week01 day02"

# ── End of session — merge back to develop ───────────────────────
git checkout develop
git merge --no-ff feature/weekNN-topic -m "merge: weekNN topic description"
git branch -d feature/weekNN-topic

# ── End of week — promote to main ────────────────────────────────
# Only when: mvn test passes + all week deliverables done
git checkout main
git merge --no-ff develop -m "release: weekNN — short description of what was built"
git tag -a v0.N.0-weekNN -m "Week NN complete: description"
```

---

## Week-by-Week Branch Reference

| Week | Branch(es) | Topic |
|------|-----------|-------|
| 1 | `feature/week01-domain-model` | Java OOP, domain model, unit tests |
| 2 | `feature/week02-spring-api`, `feature/week02-ci-pipeline` | Spring Boot REST, JPA, validation, CI |
| 3 | `feature/week03-auth` | Spring Security, JWT, BCrypt, RBAC |
| 4 | `feature/week04-mysql-schema` | MySQL schema, transactions, ACID |
| 5 | `feature/week05-db-tuning` | Index tuning, partition, locking |
| 6 | `feature/week06-concurrency` | Threads, ExecutorService, CompletableFuture |
| 7 | `feature/week07-redis-cache` | Cache-aside, invalidation, idempotency |
| 8 | `feature/week08-kafka` | Kafka events, retry, transactional outbox |
| 9 | `feature/week09-websocket-minio` | WebSocket push, MinIO, reliability patterns |
| 10 | `feature/week10-docker` | Docker, Compose, HAProxy, networking |
| 11 | `feature/week11-cicd` | Full CI/CD pipeline, load testing |
| 12 | `feature/week12-ha-replication` | MySQL replication, Keepalived, HA |
| 13 | `feature/week13-kubernetes` | Kubernetes, observability, System Design review |

---

## Definition of Done (Per Topic)

A topic is only complete when you can:

- [ ] Explain the mechanism in your own words (not just recite the definition)
- [ ] Write a working demo without copying a tutorial verbatim
- [ ] Name at least **two failure modes** or edge cases
- [ ] Show a test, benchmark, or log output proving the behavior
- [ ] Document the trade-offs and limitations in writing

---

## Things to Never Do

| ❌ Never | ✅ Instead |
|----------|-----------|
| `git commit -m "fix"` | `git commit -m "fix(order): reject state transition from SHIPPED to PENDING"` |
| Commit `.env` files | Add `.env` to `.gitignore`; commit `.env.example` only |
| Commit to `main` directly | Always go through `develop` |
| `git push --force` on shared branches | Only allowed on your own feature branches |
| Giant commit at end of day | 2–3 small commits throughout the session |
| Leave `target/` in the repo | Covered by `.gitignore` |
