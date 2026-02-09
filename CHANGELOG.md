# Changelog

All notable changes to this project will be documented in this file.

Format based on [Keep a Changelog](https://keepachangelog.com/).

## [0.2.0] - 2026-02-09

### Fixed

- Replace SELECT+INSERT/UPDATE with atomic UPSERT (`INSERT ... ON CONFLICT DO UPDATE`) in `AddItem()` to fix intermittent `SQLSTATE 25006` errors caused by PgCat routing writes to read-only replicas.

## [0.1.0] - 2026-01-07

### Added

- Shopping cart microservice with 3-layer architecture (Web / Logic / Core).
- Cart API endpoints:
  - `GET /api/v1/cart` -- get user cart.
  - `POST /api/v1/cart` -- add item to cart.
  - `DELETE /api/v1/cart` -- clear cart.
  - `GET /api/v1/cart/count` -- get cart item count.
  - `PATCH /api/v1/cart/items/:itemId` -- update item quantity.
  - `DELETE /api/v1/cart/items/:itemId` -- remove item.
- PostgreSQL 18 via pgx/v5 with PgCat connection pooling.
- OpenTelemetry tracing and Prometheus metrics.
- Graceful shutdown with readiness drain delay.
