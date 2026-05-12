# FinRelay Architecture

## 1. What the project does

FinRelay is a fintech operations webhook reliability platform.

Its purpose is to receive events from external systems, validate them, store them safely, process them asynchronously, recover from failures, and give operators visibility into event delivery health.

The system is designed around reliability, replayability, observability, and operational control.

The project focuses on fintech-style events such as:
- payment.succeeded
- payment.failed
- refund.created
- payout.failed
- chargeback.created
- settlement.received

---

## 2. Problem statement

Webhook-driven systems are common in fintech, but they become hard to manage when:
- events are delivered more than once
- downstream services fail temporarily
- requests time out
- payloads need to be replayed
- failures need auditing
- operators need visibility into delivery behavior
- teams need analytics around retries, latency, and failure patterns

FinRelay is meant to solve that reliability and observability layer.

---

## 3. System goals

### Functional goals
- receive incoming webhook events
- verify event authenticity
- persist raw payloads
- deduplicate repeated events
- queue events for async processing
- process events with worker services
- retry failures with backoff
- isolate poison messages in a DLQ
- replay failed events manually
- expose event delivery analytics
- provide searchable payload and log inspection
- send alerts for abnormal behavior

### Non-functional goals
- reliability
- auditability
- security
- debuggability
- scalability
- observability
- modularity
- clean separation between ingestion, processing, analytics, and operator experience

---

## 4. High-level event flow

1. An external fintech system sends a webhook to FinRelay.
2. The API ingress layer accepts the request.
3. The webhook signature and timestamp are validated.
4. The raw payload is persisted.
5. The event is checked for duplicates.
6. The event is placed onto an SQS queue.
7. A worker consumes the message.
8. The worker processes the event and updates delivery state.
9. Success or failure is written back to PostgreSQL.
10. Failed events are retried with backoff.
11. Poison messages are moved to the DLQ.
12. Raw payloads are archived in S3.
13. Event and delivery data are indexed for analytics and search.
14. Metrics and traces are exported to the observability stack.
15. Alerts are triggered when abnormal conditions are detected.
16. Operators can inspect, search, and replay events from the dashboard.

---

## 5. Service breakdown

### Frontend
- Next.js
- TypeScript
- Tailwind CSS
- shadcn/ui
- Recharts

Used for:
- login
- tenant overview
- endpoint management
- event stream
- delivery timeline
- DLQ browser
- replay console
- charts and dashboards
- audit log viewer
- search pages

### API
- Node.js
- TypeScript

Used for:
- webhook ingestion
- signature verification
- endpoint configuration
- replay actions
- alert rules
- auth-protected admin APIs
- data retrieval for dashboard screens

### Database
- PostgreSQL

Used for:
- tenants
- users
- endpoints
- signing secrets metadata
- webhook events
- delivery attempts
- retry state
- replay jobs
- alert rules
- audit logs
- endpoint configs

### Cache
- Redis

Used for:
- dedupe keys
- idempotency keys
- rate limiting
- short-lived retry state
- in-flight status
- locks
- quick dashboard lookups

### Queue
- AWS SQS
- DLQ

Used for:
- async processing
- decoupling ingestion from workers
- smoothing spikes
- retry routing
- poison message isolation

### Object storage
- AWS S3

Used for:
- raw payload archival
- replay snapshots
- long-term event storage
- exports
- audit retention

### Analytics
- ClickHouse

Used for:
- latency aggregates
- success/failure rates
- retry counts
- endpoint performance summaries
- event trends
- dashboard charts

### Search
- OpenSearch

Used for:
- payload text search
- failure investigation
- event filtering
- log-style query exploration

### Observability
- OpenTelemetry
- Prometheus
- Grafana
- Loki

Used for:
- traces
- metrics
- logs
- latency charts
- retry charts
- DLQ metrics
- worker health
- alerting dashboards

### Deployment
- Docker
- AWS ECS/Fargate

Used for:
- API container
- worker container
- frontend container
- optional analytics processor container
- optional replay processor container

### CI/CD
- GitHub Actions

Used for:
- linting
- testing
- building Docker images
- pushing images
- deploying to ECS/Fargate

### Auth
- JWT or Clerk or Auth.js

Used for:
- dashboard login
- role-based access control
- replay permissions
- alert configuration access

### Notifications
- Slack
- Email

Used for:
- DLQ spikes
- retry spikes
- latency alerts
- endpoint failure alerts
- replay failure alerts

---

## 6. Core entities

### Tenant
Represents a business/customer account using the system.

Fields:
- id
- name
- status
- created_at
- updated_at

### User
Represents a dashboard user.

Fields:
- id
- tenant_id
- name
- email
- role
- auth_provider_id
- created_at
- updated_at

### Endpoint
Represents a webhook destination or monitored source.

Fields:
- id
- tenant_id
- name
- url
- event_filters
- signing_secret_id
- status
- retry_policy
- created_at
- updated_at

### Webhook event
Represents a single incoming webhook payload.

Fields:
- id
- tenant_id
- endpoint_id
- external_event_id
- event_type
- payload_path
- payload_hash
- status
- received_at
- processed_at
- replay_count

### Delivery attempt
Represents one attempt to process or forward an event.

Fields:
- id
- event_id
- attempt_number
- status
- response_code
- error_message
- duration_ms
- created_at

### Replay job
Represents a manual replay action.

Fields:
- id
- tenant_id
- event_id or filter criteria
- requested_by
- replay_status
- created_at
- finished_at

### Alert rule
Represents an operational threshold.

Fields:
- id
- tenant_id
- rule_type
- threshold
- window
- enabled
- created_at

### Audit log
Represents an operator action or system action that should be traceable.

Fields:
- id
- tenant_id
- actor_type
- actor_id
- action_type
- metadata
- created_at

---

## 7. Event lifecycle states

Proposed event states:
- received
- verified
- persisted
- queued
- processing
- succeeded
- failed_retryable
- retry_scheduled
- moved_to_dlq
- replay_requested
- replay_processing
- replay_succeeded
- replay_failed

These states should be explicit in the data model so the dashboard can show a clear timeline.

---

## 8. Reliability patterns used

- signature verification
- timestamp validation
- deduplication
- idempotent processing
- retry with backoff
- poison message isolation
- dead-letter queue handling
- manual replay
- audit logging
- event archival

---

## 9. Observability patterns used

- trace propagation using correlation IDs
- API span instrumentation
- queue processing spans
- worker spans
- DB query timing
- failure reason tracking
- Grafana dashboards
- log aggregation in Loki
- metrics collection in Prometheus

---

## 10. Analytics strategy

Transactional data stays in PostgreSQL.

Analytics-friendly aggregates are pushed to ClickHouse.

Examples:
- events per minute
- success rate by endpoint
- failure rate by event type
- retry histogram
- p95 / p99 latency
- DLQ trends
- replay success rate

---

## 11. Search strategy

OpenSearch is used for:
- full-text search across payloads
- failed event investigation
- event type filtering
- error message lookup
- operator notes
- incident investigation

---

## 12. Security strategy

- tenant isolation
- role-based dashboard access
- signed webhook verification
- timestamp freshness checks
- rate limiting
- encrypted secrets storage
- payload redaction where needed
- restricted replay permissions
- private infrastructure access where possible

---

## 13. Phase plan

### Phase 1
Foundation setup:
- repo
- docs
- cloud accounts
- naming strategy
- resource inventory
- access strategy

### Phase 2
Architecture and planning:
- service boundaries
- entity model
- event state model
- environment variable design
- flow diagrams

### Phase 3
Local development setup:
- app skeleton
- Docker setup
- service connectivity
- config strategy

### Phase 4
Webhook ingestion MVP:
- incoming webhook API
- signature validation
- raw storage
- dedupe
- queue push

### Phase 5
Worker and reliability layer:
- async processing
- retries
- DLQ
- replay
- delivery state updates

### Phase 6
Dashboard and analytics:
- event list
- event details
- retry history
- replay console
- analytics charts

### Phase 7
Observability and hardening:
- traces
- metrics
- logs
- alerts
- security
- deployment

---

## 14. Success criteria

The project is successful if it can:
- receive fintech-style webhooks reliably
- avoid duplicate processing
- recover from transient failures
- isolate unrecoverable failures
- support manual replay
- provide usable analytics
- expose searchable operational history
- deploy cleanly in a production-like environment
