# cart-service

Shopping cart microservice for managing user carts and items.

## Features

- Add/remove items
- Update quantities
- Cart totals calculation
- Cart count for badges

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/cart` | Get user cart |
| `POST` | `/api/v1/cart` | Add item |
| `DELETE` | `/api/v1/cart` | Clear cart |
| `GET` | `/api/v1/cart/count` | Get item count |
| `PATCH` | `/api/v1/cart/items/:id` | Update quantity |
| `DELETE` | `/api/v1/cart/items/:id` | Remove item |

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
