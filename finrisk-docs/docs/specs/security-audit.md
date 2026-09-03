# Security & Auditability Spec

## Security baseline

- JWT authentication; role-based access control (investigator, admin roles); service-to-service auth where appropriate
- Input validation everywhere at the boundary
- Secrets only via environment variables — never committed to git
- Secure password handling if passwords are implemented
- Audit logging + sensitive data masking

**Never log**: full card numbers, CVV, passwords, access tokens, API keys, sensitive auth credentials. All simulated payment data is tokenized/synthetic (see `CLAUDE.md` hard rule #3).

## Auditability

Every risk decision must be explainable after the fact. Store on every `RiskEvaluation`:

```
transaction_id, risk_score, risk_level, decision, rules_triggered,
ML_score, network_score, behavioral_score,
model_version, policy_version, timestamp, service_version
```

And on every investigator action:

```
case_id, investigator_id, decision, reason, timestamp
```

The system must always be able to answer: *"Why did the system make this decision at that point in time?"* — that means version metadata is captured at decision time and never mutated retroactively.

## Model & risk-engine versioning

Track independently: `model_version`, `ruleset_version`, `policy_version`, `service_version`. Example:

```json
{
  "risk_score": 87,
  "model_version": "fraud-model-1.2",
  "ruleset_version": "rules-3.1",
  "policy_version": "aml-policy-2.0"
}
```

When any of these change, previously stored decisions keep their original version metadata — this is what makes the audit trail actually reproducible rather than just a log line.
