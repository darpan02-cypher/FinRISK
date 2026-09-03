# Product Spec

## Problem

Financial institutions need to evaluate transactions for fraud/financial crime in real time while minimizing false positives, and give investigators explainable, auditable evidence for every risk decision.

## What FinRisk is (and isn't)

A real-time risk & investigation platform: transaction → risk score → decision (ALLOW/REVIEW/BLOCK) → investigation case → RAG-assisted investigation → audit trail.

It is **not** an "AI fraud chatbot." The LLM never makes the fraud decision — see the hard rule in root `CLAUDE.md`. Decision layer (rules, ML, network, velocity) is fully separate from the intelligence layer (retrieval + explanation).

```
Transaction → Event Processing → Feature Generation
   → Rules + ML + Behavioral + Network Risk → Risk Score
   → Decision (ALLOW / REVIEW / BLOCK)
         → Investigation Case → Evidence Collection
         → RAG Investigation → Grounded Explanation → Audit Trail
```

## Target users

**Fraud/Risk Investigator** (initial, only user role that matters for v1):
view suspicious transactions and scores; see triggered rules, behavioral anomalies, network relationships, transaction history; open a case; ask questions and get evidence-grounded answers; review recommended actions; record decisions; view the audit trail.

Future (not built now): risk analysts, AML investigators, compliance, payment processors.

## Core use case

Example inbound transaction:

```json
{
  "transaction_id": "txn_123",
  "customer_id": "cust_456",
  "amount": 8500.00,
  "currency": "USD",
  "merchant_id": "merchant_42",
  "device_id": "device_99",
  "ip_address": "203.0.113.10",
  "timestamp": "2026-09-02T18:00:00Z"
}
```

Signals evaluated: amount, historical average, frequency, velocity, device change, IP/location change, unusual merchant/recipient, account age, prior fraud history, failed auth attempts, geographic anomaly, network relationships to flagged accounts.

Example output:

```
Risk Score: 87/100
Risk Level: HIGH
Decision: REVIEW

1. Transaction amount is 4.7x customer's 30-day average.
2. Five transactions occurred within 90 seconds.
3. New device detected.
4. Transaction originates from an unusual location.
5. Recipient account has connections to previously flagged accounts.
```

All signals are preserved as structured evidence (see `data-events.md` for `RiskSignal`/`CaseEvidence`).

## Risk decision model

Hybrid — never rely on ML alone:

```
Final Risk Score = Rule Risk + Behavioral Risk + ML Risk + Velocity Risk + Network Risk
```

Normalized to 0–100:

| Range | Level | Default decision |
|---|---|---|
| 0–29 | LOW | ALLOW |
| 30–59 | MEDIUM | ALLOW / REVIEW depending on which rules fired |
| 60–79 | HIGH | REVIEW |
| 80–100 | CRITICAL | BLOCK |

Thresholds and the ALLOW/REVIEW/BLOCK mapping must be **configurable**, not hardcoded — they change with `ruleset_version`/`policy_version` (see `security-audit.md`).
