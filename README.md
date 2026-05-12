# FinRelay

FinRelay is a fintech-focused webhook reliability and analytics platform.

It receives incoming webhook events from external systems, verifies their authenticity, stores the raw payload safely, queues processing asynchronously, retries failures with backoff, isolates poison messages in a dead-letter queue, and exposes delivery health through an operator dashboard.

## What problem it solves

In real fintech workflows, webhook delivery is often unreliable at scale. Events may arrive more than once, be delayed, fail temporarily, or fail permanently because of downstream issues.

FinRelay helps teams:
- ingest webhook events safely
- verify signatures and freshness
- prevent duplicate processing
- retry transient failures
- route poison events into a DLQ
- replay failed events from a dashboard
- view reliability metrics and failure trends
- inspect payloads, attempts, and traces

## Domain

Fintech operations

Example event types:
- payment.succeeded
- payment.failed
- refund.created
- payout.failed
- chargeback.created
- settlement.received

## High-level architecture

Webhook source → API ingress → signature verification → raw storage → dedupe → queue → worker processing → transactional state update → archival → analytics indexing → search indexing → observability → alerts → replay console

## Core stack

- Frontend: Next.js, TypeScript, Tailwind CSS, shadcn/ui, Recharts
- API: Node.js, TypeScript
- Main DB: PostgreSQL
- Cache: Redis
- Queue: AWS SQS + DLQ
- Storage: AWS S3
- Observability: OpenTelemetry, Prometheus, Grafana, Loki
- Deployment: Docker, AWS ECS/Fargate
- CI/CD: GitHub Actions
- Auth: JWT / Clerk / Auth.js
- Notifications: Slack / Email
- Analytics: ClickHouse
- Search: OpenSearch

## Phase plan

### Phase 1
Foundation setup:
- repository
- cloud accounts
- naming conventions
- resource inventory
- access strategy

### Phase 2
Documentation and architecture:
- scope definition
- system design
- entity mapping
- event lifecycle mapping
- environment variable planning

### Phase 3
Local development setup:
- app skeleton
- service skeletons
- Docker Compose
- config and secrets handling

### Phase 4
Webhook ingestion MVP:
- public ingest endpoint
- signature verification
- raw event storage
- dedupe
- queue dispatch

### Phase 5
Async processing and reliability:
- worker service
- retries
- DLQ
- replay flow
- delivery attempt logging

### Phase 6
Dashboard and analytics:
- operator views
- event timelines
- delivery metrics
- failure trends
- replay controls

### Phase 7
Observability and hardening:
- traces
- metrics
- logs
- alerts
- role-based access
- production deployment

## Current status

- Planning stage
- No production code committed yet
- Documentation skeleton created first

## Notes

This project is intentionally scoped as a focused, production-style fintech operations system rather than a general-purpose webhook company clone.
