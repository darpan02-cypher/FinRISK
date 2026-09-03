# Data & Events Spec

Covers: Kafka event pipeline, Redis real-time state, PostgreSQL persistent store.

## Event-driven architecture (Kafka)

```
Transaction API → Kafka Producer → transactions
   → Risk Evaluation Consumer → Feature Generation → Risk Engine → risk-decisions
        → Investigation                    → Analytics
```

Topics: `transactions`, `risk-evaluations`, `risk-decisions`, `investigation-events`, `audit-events`. Use explicit event schemas.

Design for: idempotency, retries, dead-letter queues, duplicate events, consumer failures. **Transaction idempotency is non-negotiable in a financial system** — every event carries an `idempotency_key`; a duplicate returns the existing result rather than reprocessing. Document this decision (ADR).

Why Kafka over direct HTTP / SQS / RabbitMQ: needed for async processing and *replayable* transaction events — that replayability is specifically what demonstrates stream-processing chops for this project. Write the full tradeoff as an ADR; don't just assert it.

## Redis (real-time state)

Used where per-request Postgres queries would be too slow for velocity-style checks:

- `customer:{id}:transactions:last_5_minutes` — velocity window
- `customer:{id}:known_devices` — new-device detection
- `customer:{id}:recent_transactions` — recent history for amount-anomaly comparisons

Use TTLs appropriately. Document (ADR) why Redis beats querying Postgres for every velocity calculation — this is a real latency argument, not a "Redis is standard" argument.

## PostgreSQL (system of record)

Primary persistent store. Suggested entities — don't over-normalize beyond what's needed:

```
Customer, Account, Transaction, RiskEvaluation, RiskSignal, RiskRule,
InvestigationCase, CaseEvidence, InvestigatorDecision, AuditEvent,
Device, Merchant
```

Use Alembic (or equivalent) migrations from day one — never hand-edit schema.

Why Postgres over MongoDB: transactions/risk data are relational and need strong consistency + joins for the network-risk traversal (see `risk-engine.md`); document as ADR.
