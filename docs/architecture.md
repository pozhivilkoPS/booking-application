# Architecture

## Domain

Booking platform for events (concerts, shows). A user picks an event, holds
seats, pays, and receives a confirmed booking. Unpaid holds expire and the
seats return to sale.

## Services

| Service             | Responsibility                                       | Storage           |
|---------------------|------------------------------------------------------|-------------------|
| `api-gateway`       | Single entry point, routing, auth, rate limiting     | —                 |
| `inventory-service` | Events, seats, seat holds                            | PostgreSQL, Redis |
| `booking-service`   | Booking lifecycle, Saga orchestration                | PostgreSQL        |
| `payment-service`   | Payment processing (simulated, can be made to fail)  | PostgreSQL        |
| `search-service`    | Event search (CQRS read model)                       | Elasticsearch     |
| `notification-service` | Emails / tickets                                  | —                 |
| `frontend`          | React + TypeScript: seat map, search, booking flow   | —                 |

## Key decision: what Kafka is used for — and what it is NOT

**Kafka is used for asynchronous, eventually consistent flows:**

- Booking Saga: `BookingCreated` → `PaymentRequested` → `PaymentCompleted` /
  `PaymentFailed` → `BookingConfirmed` / `BookingCancelled` (with compensations).
- CQRS read model sync: inventory/booking changes → search index
  (optionally via Debezium CDC).
- Hold expiry: unpaid hold times out → `SeatReleased`.
- Notifications and analytics.

**Kafka is NOT used for grabbing a seat.** Two users clicking the same seat
need an immediate, strictly consistent answer, so this is synchronous:

- Optimistic locking (`@Version`) or `SELECT ... FOR UPDATE` in PostgreSQL.
- Unique constraint on `(event_id, seat_id)` for active bookings.
- Redis `SET NX` with TTL (10–15 min) as a temporary seat hold.

Availability checks are synchronous too (REST/gRPC).

## Kafka partitioning

Message key = `eventId`. All messages for one event land in the same partition
and are processed in order.

Topic auto-creation is disabled in the broker; topics are declared explicitly.

## Microservice patterns

- Database per Service (separate `inventory`, `booking`, `payment` databases)
- Saga with compensating actions
- Transactional Outbox (no lost events between DB commit and Kafka publish)
- Idempotent Consumer
- CQRS (write: PostgreSQL, read: Elasticsearch)
- API Gateway
- Circuit Breaker, Rate Limiter (Resilience4j)
- Distributed lock (Redis)
- Dead Letter Topic

## Delivery pipeline

GitHub Actions (build + tests) → Docker image → container registry →
ArgoCD (GitOps) → Kubernetes (k3s/kind locally) via Helm charts.

Observability: OpenTelemetry + Jaeger, Prometheus + Grafana, Loki.
Testing: JUnit 5, Testcontainers (real Kafka/Postgres), Pact (contract tests).
