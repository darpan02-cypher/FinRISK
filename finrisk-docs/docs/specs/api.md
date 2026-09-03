# API Spec

## Stack

FastAPI + Pydantic, dependency injection, async where it matters, structured error handling, validation, authn/authz, service/repository separation. Versioned under `/api/v1/`.

## Endpoints (v1 target)

```
POST   /api/v1/transactions
GET    /api/v1/transactions/{id}

POST   /api/v1/risk/evaluate
GET    /api/v1/risk/{transaction_id}

GET    /api/v1/investigations
POST   /api/v1/investigations
GET    /api/v1/investigations/{id}

POST   /api/v1/investigations/{id}/ask
GET    /api/v1/investigations/{id}/evidence

GET    /api/v1/audit/{entity_id}

GET    /health
GET    /ready
```

## Spec-driven development — this is a deliberate constraint, not optional

```
OpenAPI specification → API contract → Schemas → Implementation → Contract tests
```

`openapi/finrisk.yaml` is written **before** the endpoint it describes, and defines endpoints, request/response schemas, error responses, auth, validation, and status codes. Generated docs come from it. Where practical, generate client/server types from it rather than hand-writing duplicates. Contract tests assert the implementation matches the spec (Phase 11 in `plan.md`) — a passing implementation with a stale OpenAPI file is a failed contract test, not a documentation nit.

Why spec-driven here specifically: it's one of the things this project is meant to demonstrate (see resume objective), and it forces API design decisions to happen before implementation details leak into them.
