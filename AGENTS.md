# cart-service

> AI Agent context for understanding this repository

## 📋 Overview

Shopping cart microservice. Manages user carts, items, and quantities.

## 🏗️ Architecture Guidelines

### 3-Layer Architecture

| Layer | Location | Responsibility |
|-------|----------|----------------|
| **Web** | `internal/web/v1/handler.go` | HTTP handling, validation, error translation |
| **Logic** | `internal/logic/v1/service.go` | Business rules (❌ NO SQL) |
| **Core** | `internal/core/` | Domain models, repositories, database |

### 3-Layer Coding Rules

**CRITICAL**: Strict layer boundaries. Violations will be rejected in code review.

#### Layer Boundaries

| Layer | Location | ALLOWED | FORBIDDEN |
|-------|----------|---------|-----------|
| **Web** | `internal/web/v1/` | HTTP handling, JSON binding, DTO mapping, call Logic, aggregation | SQL queries, direct DB access, business rules |
| **Logic** | `internal/logic/v1/` | Business rules, call repository interfaces, domain errors | SQL queries, `database.GetPool()`, HTTP handling, `*gin.Context` |
| **Core** | `internal/core/` | Domain models, repository implementations, SQL queries, DB connection | HTTP handling, business orchestration |

#### Dependency Direction

```
Web -> Logic -> Core (one-way only, never reverse)
```

- Web imports Logic and Core/domain
- Logic imports Core/domain and Core/repository interfaces
- Core imports nothing from Web or Logic

#### DO

- Put HTTP handlers, request validation, error-to-status mapping in `web/`
- Put business rules, orchestration, transaction logic in `logic/`
- Put SQL queries in `core/repository/` implementations
- Use repository interfaces (defined in `core/domain/`) for data access in Logic layer
- Use dependency injection (constructor parameters) for all service dependencies

#### DO NOT

- Write SQL or call `database.GetPool()` in Logic layer
- Import `gin` or handle HTTP in Logic layer
- Put business rules in Web layer (Web only translates and delegates)
- Call Logic functions directly from another service (use HTTP aggregation in Web layer)
- Skip the Logic layer (Web must not call Core/repository directly)

### Directory Structure

```
cart-service/
├── cmd/main.go
├── config/config.go
├── db/migrations/sql/
├── internal/
│   ├── core/
│   │   ├── database.go
│   │   └── domain/
│   ├── logic/v1/service.go
│   └── web/v1/handler.go
├── middleware/
└── Dockerfile
```

## 🛠️ Development Workflow

### Code Quality

**MANDATORY**: All code changes MUST pass lint before committing.

- Linter: `golangci-lint` v2+ with `.golangci.yml` config (60+ linters enabled)
- Zero tolerance: PRs with lint errors will NOT be merged
- CI enforces: `go-check` job runs lint on every PR

#### Commands (run in order)

```bash
go mod tidy              # Clean dependencies
go build ./...           # Verify compilation
go test ./...            # Run tests
golangci-lint run --timeout=10m  # Lint (MUST pass)
```

#### Pre-commit One-liner

```bash
go build ./... && go test ./... && golangci-lint run --timeout=10m
```

### Common Lint Fixes

- `perfsprint`: Use `errors.New()` instead of `fmt.Errorf()` when no format verbs
- `nosprintfhostport`: Use `net.JoinHostPort()` instead of `fmt.Sprintf("%s:%s", host, port)`
- `errcheck`: Always check error returns (or explicitly `_ = fn()`)
- `goconst`: Extract repeated string literals to constants
- `gocognit`: Extract helper functions to reduce complexity
- `noctx`: Use `http.NewRequestWithContext()` instead of `http.NewRequest()`

## 🔧 Tech Stack

| Component | Technology |
|-----------|------------|
| Framework | Gin |
| Database | PostgreSQL 18 via pgx/v5 |
| Tracing | OpenTelemetry |
| Metrics | Prometheus |

## 🏗️ Infrastructure Details

### Database

| Component | Value |
|-----------|-------|
| **Cluster** | transaction-db (CloudNativePG) |
| **PostgreSQL** | 18 |
| **HA** | 3 instances (1 primary + 2 replicas) |
| **Pooler** | PgCat HA (2 replicas) |
| **Endpoint** | `pgcat.cart.svc.cluster.local:5432` |
| **Pool Mode** | Transaction |
| **Replication** | **Synchronous** (zero data loss) |
| **Shared Cluster** | Yes (with order-service) |

**Query Routing (PgCat):**
- `SELECT` → `transaction-db-r` (replicas, load balanced)
- `INSERT/UPDATE/DELETE` → `transaction-db-rw` (primary)

### Graceful Shutdown

**VictoriaMetrics Pattern:**
1. `/ready` → 503 when shutting down
2. Drain delay (5s)
3. Sequential: HTTP → Database → Tracer

## 🔌 API Reference

All cart routes are **private** — JWT middleware is applied at the `/cart/v1/private` router group.

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/cart/v1/private/cart` | Get user cart |
| `POST` | `/cart/v1/private/cart` | Add item to cart |
| `DELETE` | `/cart/v1/private/cart` | Clear cart (also called by `order-service` after successful checkout with user's forwarded `Authorization`) |
| `GET` | `/cart/v1/private/cart/count` | Get cart item count (badge) |
| `PATCH` | `/cart/v1/private/cart/items/:itemId` | Update item quantity |
| `DELETE` | `/cart/v1/private/cart/items/:itemId` | Remove item |

Full convention + inventory: [`homelab/docs/api/api-naming-convention.md`](https://github.com/duynhlab/homelab/blob/main/docs/api/api-naming-convention.md).
