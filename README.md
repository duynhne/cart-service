# cart-service

Shopping cart microservice for managing user carts and items.

## Features

- Add/remove items
- Update quantities
- Cart totals calculation
- Cart count for badges

## API Endpoints

> **Browser callers** hit `https://gateway.duynhne.me/cart/v1/private/cart/…` (all routes private — JWT required); Kong rewrites to the cluster paths below. See [homelab naming convention](https://github.com/duynhlab/homelab/blob/main/docs/api/api-naming-convention.md).

| Method | Cluster path | Edge path (via gateway) |
|--------|--------------|-------------------------|
| `GET` | `/api/v1/cart` | `/cart/v1/private/cart` |
| `POST` | `/api/v1/cart` | `/cart/v1/private/cart` |
| `DELETE` | `/api/v1/cart` | `/cart/v1/private/cart` |
| `GET` | `/api/v1/cart/count` | `/cart/v1/private/cart/count` |
| `PATCH` | `/api/v1/cart/items/:id` | `/cart/v1/private/cart/items/:id` |
| `DELETE` | `/api/v1/cart/items/:id` | `/cart/v1/private/cart/items/:id` |

## Tech Stack

- Go + Gin framework
- PostgreSQL 18 (transaction-db cluster, HA, sync replication)
- PgCat connection pooling
- OpenTelemetry tracing

## Development

### Prerequisites

- Go 1.25+
- [golangci-lint](https://golangci-lint.run/welcome/install/) v2+

### Local Development

```bash
# Install dependencies
go mod tidy
go mod download

# Build
go build ./...

# Test
go test ./...

# Lint (must pass before PR merge)
golangci-lint run --timeout=10m

# Run locally (requires .env or env vars)
go run cmd/main.go
```

### Pre-push Checklist

```bash
go build ./... && go test ./... && golangci-lint run --timeout=10m
```

## License

MIT
