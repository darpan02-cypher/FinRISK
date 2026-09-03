# FinRisk

Real-time financial risk & investigation platform (portfolio project, not a production banking system). Evaluates synthetic transactions with a hybrid rules + ML + network risk engine, routes high-risk ones into investigation cases, and lets investigators ask an evidence-grounded RAG assistant to explain decisions.

## Where things live

- `plan.md` — the phase plan and current status. **Read this first, every session.** Work one phase at a time; don't start a phase whose checkbox above it isn't done.
- `docs/specs/*.md` — detailed requirements, one file per domain. Pull in only the spec file(s) relevant to the current phase.
- `docs/decisions/` — ADRs for major technology choices.
- `openapi/finrisk.yaml` — the API contract. Written before the endpoint that implements it, not after.

## Tech stack

- Python 3.12, FastAPI, Pydantic v2, SQLAlchemy + Alembic
- PostgreSQL, Redis, Kafka
- scikit-learn / XGBoost for the fraud model
- Docker Compose for local dev; AWS (ECS/Fargate, RDS, MSK) for the cloud version
- pytest, ruff, mypy, pre-commit
- GitHub Actions for CI/CD

(Update this list as real choices get made in Phase 0/1 — don't let it drift from what's actually in `pyproject.toml`.)

## Commands

> Placeholder — fill these in once they exist, and keep them exact (not "the conventional way to run tests").

```
make dev            # start docker-compose stack
make test            # unit + integration tests
make lint             # ruff + mypy
make migrate          # alembic upgrade head
python scripts/generate_transactions.py --scenario normal --count 1000
```

## Hard rules (do not drift from these)

1. **The LLM never makes the fraud/risk decision.** Rules + ML + network scoring produce `risk_score` and `decision`. The LLM only explains a decision that already exists, grounded in retrieved evidence — never invents evidence, never overrides a score.
2. **Every risk decision is versioned and reproducible.** Store `model_version`, `ruleset_version`, `policy_version` with every `RiskEvaluation`. Never overwrite historical model/rule metadata.
3. **No real financial data, ever.** Synthetic customers, accounts, transactions, tokenized payment identifiers only. Never log full card numbers, CVV, passwords, tokens, or API keys.
4. **Only report measured numbers.** Latency, throughput, accuracy, fraud recall — if we didn't run the test, it doesn't go in the README or resume bullets.
5. **Transaction decisions must survive Kafka/Redis/ML/LLM outages.** The RAG/investigation layer is allowed to degrade; the risk decision path is not.
6. **Don't introduce a technology without a stated reason in an ADR** (no Kubernetes, service mesh, Neo4j, Spark/Flink/Airflow, Terraform, multiple LLMs "just because"). See `docs/decisions/`.

## Working style

- Before an architecturally significant decision (new datastore, new service boundary, a modeling choice), state: the problem, the recommended option, 1-2 alternatives, the tradeoff — then implement. Don't silently decide these.
- Don't over-explain trivial implementation details.
- Prefer simple, explicit, testable code over abstractions added "for design pattern" reasons.
- At the start of a session, state which phase (from `plan.md`) you're on and what's left in it before writing code.
