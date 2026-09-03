# Risk Engine Spec

Covers: rule engine, ML component, real-time inference, network/graph risk. See `product.md` for how these combine into the final score.

## Rule engine

Deterministic, built first. Each rule returns a structured result:

```json
{
  "rule_id": "AMOUNT_ANOMALY",
  "triggered": true,
  "severity": "HIGH",
  "score": 25,
  "evidence": {
    "transaction_amount": 8500,
    "historical_average": 1800,
    "ratio": 4.72
  }
}
```

Initial rule set:

- **Large transaction**: `amount > configurable_threshold`
- **Amount anomaly**: `amount > 3 × customer's historical average`
- **Velocity**: more than N transactions within T minutes (backed by Redis, see `data-events.md`)
- **New device**: `device_id` not previously associated with customer
- **Geographic anomaly**: current location significantly differs from recent locations
- **Suspicious recipient**: recipient associated with previous high-risk transactions
- **Failed authentication**: multiple failed login/auth events before the transaction

## Machine learning component

Realistic but honestly scoped — synthetic/public dataset, clearly documented as such, never claimed to be trained on real bank data.

Model candidates, start simple: Logistic Regression → Random Forest → XGBoost.

Pipeline: Data → Feature Engineering → Train/Val/Test split → Class Imbalance Handling → Model Evaluation → Serialization → Real-Time Inference API.

Fraud datasets are highly imbalanced — evaluate Precision, Recall, F1, PR-AUC, ROC-AUC, Confusion Matrix. **Do not lead with accuracy**; document why it's misleading here. Track Fraud Recall and False Positive Rate specifically.

### Real-time inference

```
RiskService → FeatureService → ModelInferenceService → FraudProbability
```

e.g. `fraud_probability = 0.91` → normalized into the ML risk component of the final score. Expose via internal call or `POST /risk/evaluate`. Must be fast enough for real-time evaluation — measure and record inference latency, feature-generation latency, and total risk-evaluation latency separately (feeds the metrics in `nonfunctional.md`).

## Network / graph risk

Purpose: catch relationships not visible from a single transaction, e.g.:

```
Customer A → Account X → Recipient Y → Account Z → previously-flagged Customer B
```

**Start with Postgres relationship/graph traversal, not a graph database.** Only evaluate Neo4j later if there's a concrete justification (query patterns Postgres genuinely can't express well at the required latency) — write that as an ADR before adding it, per the "don't add tech without a stated reason" rule in `CLAUDE.md`.

Signals: shared devices, shared IPs, shared recipients, shared addresses, shared accounts, suspicious transaction clusters. Output e.g. `network_risk = 35`, one component of the final score.

## Versioning

Every risk evaluation must record `model_version`, `ruleset_version`, `policy_version` alongside the score — see `security-audit.md` for the full auditability requirement. When the model changes (v1.0 → v1.1 → v1.2), old decisions stay reproducible with their original version metadata; never overwrite it.
