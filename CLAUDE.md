# BookingApplication

Learning / portfolio project: event booking platform on microservices.
GitHub repo name: `booking-application`. Java packages: `com.booking.*`.

Full architecture and decisions: [docs/architecture.md](docs/architecture.md).

## Goal of the project

The owner is a Java developer improving skills and learning vibe coding.
Must-haves: Kafka, microservices, microservice patterns, TypeScript UI,
CI/CD + Kubernetes.

How to work with the owner:
- Act as a mentor: explain *why* a solution is chosen, not just write code.
- Move in small steps; each step should leave the project runnable.
- The owner reviews every diff.

## Stack

- Backend: Java 21, Spring Boot 3, Spring Kafka, Spring Cloud Gateway,
  Resilience4j, PostgreSQL 17, Redis 8
- Kafka 4.1 in KRaft mode (no Zookeeper), topic auto-creation disabled
- Frontend: React + TypeScript
- Infra: Docker Compose (local), Kubernetes + Helm, GitHub Actions, ArgoCD

## Layout

```
infra/docker-compose.yml   # postgres, kafka, kafka-ui (:8090), redis
infra/postgres/init.sql    # databases: inventory, booking, payment
services/                  # one directory per Java service
frontend/                  # React + TS
docs/                      # architecture and decisions
.github/workflows/         # CI
```

Start infrastructure: `docker compose -f infra/docker-compose.yml up -d`

## Roadmap

1. [x] Repository skeleton + local infrastructure (Docker Compose)
2. [ ] `inventory-service`: events, seats, seat hold (optimistic lock + Redis TTL)
3. [ ] `booking-service` + Kafka, Transactional Outbox
4. [ ] `payment-service`, Booking Saga with compensations
5. [ ] Hold expiry (`SeatReleased`), Idempotent Consumer, Dead Letter Topic
6. [ ] `api-gateway`, Circuit Breaker, Rate Limiter
7. [ ] Frontend: event list, seat map, booking flow
8. [ ] `search-service` (CQRS read model, Elasticsearch)
9. [ ] CI: GitHub Actions, Testcontainers
10. [ ] Kubernetes + Helm + ArgoCD
11. [ ] Observability: OpenTelemetry, Prometheus, Grafana

## Current status / environment notes

- Java and Node.js are NOT installed yet on the dev machine (WSL2).
  Proposed: Java 21 + Maven via SDKMAN, Node LTS via nvm.
- Docker CLI is installed but the daemon was not running (needs Docker Desktop
  WSL integration). Compose file validated with `docker compose config`, but
  containers were not started yet.
- Next step: roadmap item 2 (`inventory-service`).
