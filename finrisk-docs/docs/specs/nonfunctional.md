# Non-Functional & Delivery Spec

## Observability

Structured JSON logs with correlation/request IDs; metrics; distributed tracing where practical. Metrics to expose (Prometheus-compatible):

```
transactions_processed_total, risk_evaluations_total, fraud_predictions_total,
transactions_blocked_total, transactions_reviewed_total,
risk_evaluation_latency_ms, ml_inference_latency_ms, rag_latency_ms,
kafka_consumer_lag, failed_events_total
```

Every transaction should be traceable end to end.

## Error handling & degradation

Handle: Kafka down, Redis down, Postgres down, ML model unavailable, LLM unavailable, vector store unavailable, invalid/duplicate/malformed events. Degrade gracefully — the specific case that matters most: **if the LLM is unavailable, risk decisions still work; only the investigation assistant goes down.** This is a hard rule (`CLAUDE.md` #5), not a nice-to-have, and should be exercised in a failure-injection test, not just asserted in docs.

## Idempotency

See `data-events.md` — `idempotency_key` on transaction/event ingest; duplicate delivery returns the original result instead of reprocessing. Document the mechanism as an ADR.

## Testing

- **Unit**: rule engine, feature generation, risk scoring, ML inference adapter, RAG retrieval, decision logic
- **Integration**: FastAPI+Postgres, Kafka, Redis, full risk pipeline (Testcontainers)
- **Contract**: OpenAPI ↔ implementation (see `api.md`)
- **E2E**: transaction → Kafka → risk engine → decision → case creation → investigator query → RAG → grounded answer

## Load testing

Run and record actual numbers at 100 / 500 / 1000 transactions/sec: throughput, p50/p95/p99 latency, error rate. Document the hardware/environment. **Never invent a number** — this is where the honest resume bullet (e.g. "94ms p95 under 500 txn/sec synthetic load") comes from.

## Docker

`docker-compose.yml` with FastAPI, PostgreSQL, Redis, Kafka, and optionally Kafka UI, Prometheus, Grafana. Everything reproducible locally with one command. Don't containerize things that don't need it.

## AWS

Local Docker → AWS progression. Suggested shape: API Gateway → FastAPI on ECS/Fargate → RDS + Redis + Kafka(MSK) → S3 for logs. Choose services based on actual requirements, not completeness; document tradeoffs and what was deliberately skipped.

## CI/CD

GitHub Actions: lint → unit tests → integration tests → contract tests → security/dependency scan → build image → push → deploy.

## Code quality

Type hints, clean architecture, small functions, DI, environment-based config, structured logging, meaningful exceptions. Tooling: ruff, pytest, mypy, pre-commit. Add abstractions only where they earn their keep — no interfaces/classes purely for pattern's sake.

## Repository structure

```
finrisk/
├── README.md, LICENSE, .env.example, docker-compose.yml, Makefile
├── openapi/finrisk.yaml
├── services/{api,risk-engine,ml-service,investigation-service}/
├── src/{api,domain,services,repositories,models,schemas,rules,risk,rag,infrastructure}/
├── ml/{data,notebooks,training,evaluation,artifacts}/
├── tests/{unit,integration,contract,e2e}/
├── docs/{architecture,decisions,api,ml,rag}/
├── infra/aws/
└── .github/workflows/
```

Start modular; extract into separate services only where actually justified — don't force microservices prematurely.

## Synthetic data & fraud simulator

All data (customers, accounts, transactions, devices, merchants, locations, recipients, historical cases) is synthetic and must be clearly labeled as such — never implies real banking data. Simulator (`scripts/generate_transactions.py`) supports scenarios: `normal`, `account_takeover`, `velocity_attack`, `synthetic_identity`, `suspicious_network`. Demo flow: generate N transactions, inject a known number of suspicious scenarios, stream through Kafka, observe resulting decisions.

## Frontend (optional, P2)

If built: a minimal investigator dashboard (transaction list + risk breakdown, case detail with the AI assistant and citations) — it demonstrates the backend, it doesn't become the project. Don't let it delay P0 work.

## Non-functional requirements summary

Reliability (risk decisions don't depend on LLM uptime) · Performance (measured, not claimed) · Scalability (consumers/API horizontally scalable) · Security (no sensitive data in logs) · Observability (every transaction traceable) · Auditability (every decision reproducible) · Maintainability (business logic isolated from infrastructure).

## Tradeoffs to document as ADRs

Kafka vs. HTTP/SQS/RabbitMQ · Redis vs. Postgres/in-memory cache · Postgres vs. MongoDB · hybrid rules+ML vs. ML-only · why not let the LLM detect fraud · RAG vs. fine-tuning · FastAPI choice · spec-driven development · when graph tech is justified · Kafka-down / model-down / RAG-down behavior · duplicate-transaction prevention · model versioning approach.

## Definition of done

Not "it runs." Complete when every item below is true — treat this as the literal checklist to tick off before calling Phase 16 finished:

```
[ ] Architecture documented              [ ] Idempotency implemented
[ ] OpenAPI spec exists & matches impl   [ ] Unit tests implemented
[ ] FastAPI API works                    [ ] Integration tests implemented
[ ] PostgreSQL persistence works         [ ] Contract tests implemented
[ ] Redis velocity detection works       [ ] E2E flow implemented
[ ] Kafka pipeline works                 [ ] Load test executed (real numbers)
[ ] Rule engine works                    [ ] Docker setup works
[ ] ML model trained & integrated        [ ] CI pipeline works
[ ] Risk score + decision generated      [ ] AWS deployment documented/implemented
[ ] Investigation case generated         [ ] README complete
[ ] Evidence stored                      [ ] Architecture diagrams complete
[ ] Network risk implemented             [ ] ADRs complete
[ ] RAG implemented, answers grounded    [ ] Resume bullets from measured results only
[ ] Citations shown
[ ] Audit trail implemented
[ ] Structured logging + metrics implemented
[ ] Error handling implemented
```
