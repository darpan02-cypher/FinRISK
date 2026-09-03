# FinRisk Build Plan

Work top to bottom. Don't start a phase until the previous one's boxes are checked. Each phase lists which spec file(s) to load into context — don't pull in specs you don't need yet.

Status legend: `[ ]` not started · `[~]` in progress · `[x]` done

---

## Phase 0 — Requirements & Architecture Review
*Specs: all of `docs/specs/*.md` (read-only pass, no code)*

- [ ] Confirm problem statement, target users, functional/non-functional requirements
- [ ] System architecture diagram (`docs/architecture/system-overview.md`)
- [ ] Data flow diagram (`docs/architecture/data-flow.md`)
- [ ] Data model sketch (entities from `docs/specs/data-events.md`)
- [ ] Event model sketch (topics from `docs/specs/data-events.md`)
- [ ] Risk scoring design confirmed (`docs/specs/risk-engine.md`)
- [ ] ML architecture confirmed (`docs/specs/risk-engine.md`)
- [ ] RAG architecture confirmed (`docs/specs/rag.md`)
- [ ] API surface drafted (`docs/specs/api.md`)
- [ ] Repo structure decided
- [ ] Initial ADRs written (FastAPI, Kafka, Redis, Postgres, hybrid risk engine, RAG, "LLM not used for transaction decisions", spec-driven API)
- [ ] **Stop. Human reviews before Phase 1.**

## Phase 1 — Repository + OpenAPI + Domain Model
*Specs: `docs/specs/api.md`, `docs/specs/data-events.md`*

- [ ] Repo scaffolding, `pyproject.toml`, pre-commit, Makefile
- [ ] `openapi/finrisk.yaml` v0: transactions, risk, investigations endpoints
- [ ] SQLAlchemy models for core entities (Customer, Account, Transaction, Device, Merchant)
- [ ] Alembic initial migration

## Phase 2 — FastAPI + PostgreSQL
*Specs: `docs/specs/api.md`*

- [ ] `POST /api/v1/transactions`, `GET /api/v1/transactions/{id}` working against Postgres
- [ ] Repository/service layer separation
- [ ] `/health`, `/ready`
- [ ] Idempotency key handling on transaction ingest

## Phase 3 — Rule Engine
*Specs: `docs/specs/risk-engine.md`*

- [ ] Rule interface + structured `RuleResult` (rule_id, triggered, severity, score, evidence)
- [ ] Implement: large transaction, amount anomaly, velocity, new device, geo anomaly, suspicious recipient, failed auth
- [ ] Configurable thresholds (not hardcoded)
- [ ] Unit tests per rule

## Phase 4 — Redis + Velocity Detection
*Specs: `docs/specs/data-events.md`*

- [ ] `customer:{id}:transactions:last_5_minutes`, known devices, recent history — with TTLs
- [ ] Velocity rule wired to Redis instead of Postgres
- [ ] Document why Redis here vs. querying Postgres per-check (ADR)

## Phase 5 — Kafka Event Pipeline
*Specs: `docs/specs/data-events.md`*

- [ ] Topics: `transactions`, `risk-evaluations`, `risk-decisions`, `investigation-events`, `audit-events`
- [ ] Producer on transaction ingest, consumer for risk evaluation
- [ ] Idempotency, retries, dead-letter queue, duplicate handling
- [ ] Failure-mode test: consumer restart, duplicate delivery

## Phase 6 — ML Training + Inference
*Specs: `docs/specs/risk-engine.md`*

- [ ] Synthetic/public dataset documented as such
- [ ] Feature engineering → train/val/test split → class-imbalance handling
- [ ] Evaluate: precision, recall, F1, PR-AUC, ROC-AUC, confusion matrix (not accuracy)
- [ ] Model serialization + `model_version`
- [ ] `ModelInferenceService` wired into risk pipeline, latency measured

## Phase 7 — Network Risk
*Specs: `docs/specs/risk-engine.md`*

- [ ] Shared device/IP/recipient/address graph signals via Postgres traversal (no graph DB yet)
- [ ] `network_risk` component feeding into final score
- [ ] ADR: when (if ever) Neo4j becomes justified

## Phase 8 — Investigation Cases
*Specs: `docs/specs/data-events.md`, `docs/specs/security-audit.md`*

- [ ] `InvestigationCase`, `CaseEvidence`, `InvestigatorDecision` entities + endpoints
- [ ] High-risk decisions auto-create cases
- [ ] Investigator decision recording

## Phase 9 — RAG
*Specs: `docs/specs/rag.md`*

- [ ] Ingest sources: internal policies, public regulatory guidance, synthetic historical cases, structured transaction evidence
- [ ] Retriever → context builder → LLM → FACT/POLICY/INFERENCE/RECOMMENDATION structured answer
- [ ] `POST /api/v1/investigations/{id}/ask` with citations
- [ ] Small eval suite: retrieval relevance, groundedness, citation correctness, hallucination rate, latency
- [ ] Confirm graceful degradation: risk decisions unaffected if RAG/LLM is down

## Phase 10 — Observability
*Specs: `docs/specs/nonfunctional.md`*

- [ ] Structured JSON logs + correlation/request IDs
- [ ] Metrics: `transactions_processed_total`, `risk_evaluation_latency_ms`, `ml_inference_latency_ms`, `rag_latency_ms`, `kafka_consumer_lag`, etc.
- [ ] Prometheus/Grafana reproducible via docker-compose

## Phase 11 — Testing
*Specs: `docs/specs/nonfunctional.md`*

- [ ] Unit: rules, features, scoring, ML adapter, RAG retrieval, decision logic
- [ ] Integration: FastAPI+Postgres, Kafka, Redis, full pipeline (Testcontainers)
- [ ] Contract tests: OpenAPI ↔ implementation
- [ ] E2E: transaction → Kafka → risk engine → case → investigator query → RAG answer

## Phase 12 — Load Testing
*Specs: `docs/specs/nonfunctional.md`*

- [ ] Run at 100 / 500 / 1000 txn/sec, record actual p50/p95/p99 latency + error rate
- [ ] Document hardware/environment used
- [ ] No invented numbers — this becomes the resume bullet

## Phase 13 — Docker
*Specs: `docs/specs/nonfunctional.md`*

- [ ] `docker-compose.yml`: FastAPI, Postgres, Redis, Kafka, (optional Kafka UI, Prometheus, Grafana)
- [ ] One-command local bring-up documented in README

## Phase 14 — AWS
*Specs: `docs/specs/nonfunctional.md`*

- [ ] Deploy: API Gateway → ECS/Fargate → RDS/Redis/Kafka(MSK) → S3 logs
- [ ] Document tradeoffs and what was skipped

## Phase 15 — CI/CD
*Specs: `docs/specs/nonfunctional.md`*

- [ ] GitHub Actions: lint → unit → integration → contract → security scan → build → push → deploy

## Phase 16 — Documentation + Resume Optimization

- [ ] README complete (all sections from spec §44)
- [ ] Architecture diagrams + ADRs finalized
- [ ] Resume bullets derived only from measured results (Phase 12 numbers, actual test coverage, etc.)

---

## Definition of done

See `docs/specs/nonfunctional.md` §"Definition of done" for the full checklist — don't mark the project complete until every box there is checked, not just "it runs."
