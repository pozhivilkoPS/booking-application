# BookingApplication

Event-driven booking platform (concerts / events) built with microservices.

## Architecture

| Service             | Responsibility                                  | Storage          |
|---------------------|-------------------------------------------------|------------------|
| `inventory-service` | Events, seats, seat holds                       | PostgreSQL, Redis |
| `booking-service`   | Booking lifecycle, Saga orchestration           | PostgreSQL       |
| `payment-service`   | Payment processing (simulated)                  | PostgreSQL       |
| `frontend`          | React + TypeScript UI                           | —                |

Services communicate asynchronously via **Apache Kafka**.

## Patterns

- Database per Service
- Saga (with compensations)
- Transactional Outbox
- CQRS
- API Gateway
- Circuit Breaker

## Tech stack

Java 21, Spring Boot 3, Spring Kafka, PostgreSQL, Redis, React, TypeScript,
Docker, Kubernetes, Helm, GitHub Actions, ArgoCD.

## Local development

```bash
docker compose -f infra/docker-compose.yml up -d
```

| Component | URL                     |
|-----------|-------------------------|
| Kafka UI  | http://localhost:8090   |
| Kafka     | localhost:9092          |
| Postgres  | localhost:5432 (booking/booking) |
| Redis     | localhost:6379          |
