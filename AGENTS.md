# cart-service

> AI Agent context for understanding this repository

## 📋 Overview

Shopping cart microservice. Manages user carts, items, and quantities.

## 🏗️ Architecture

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

## 🔌 API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/cart` | Get user cart |
| `POST` | `/api/v1/cart` | Add item to cart |
| `DELETE` | `/api/v1/cart` | Clear cart |
| `GET` | `/api/v1/cart/count` | Get cart item count |
| `PATCH` | `/api/v1/cart/items/:itemId` | Update item quantity |
| `DELETE` | `/api/v1/cart/items/:itemId` | Remove item |

## 📐 3-Layer Architecture

| Layer | Location | Responsibility |
|-------|----------|----------------|
| **Web** | `internal/web/v1/handler.go` | HTTP, validation, error translation |
| **Logic** | `internal/logic/v1/service.go` | Business rules (❌ NO SQL) |
| **Core** | `internal/core/` | Domain models, repositories |

## 🗄️ Database

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

## 🚀 Graceful Shutdown

**VictoriaMetrics Pattern:**
1. `/ready` → 503 when shutting down
2. Drain delay (5s)
3. Sequential: HTTP → Database → Tracer

## 🔧 Tech Stack

| Component | Technology |
|-----------|------------|
| **Framework** | Gin |
| **Database** | PostgreSQL 18 via pgx/v5 |
| **Tracing** | OpenTelemetry |
| **Metrics** | Prometheus |

## 🛠️ Development

```bash
go mod download && go test ./... && go build ./cmd/main.go
```
