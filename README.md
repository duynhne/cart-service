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

```bash
go mod download
go test ./...
go run cmd/main.go
```

