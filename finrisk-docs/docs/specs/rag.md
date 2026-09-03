# RAG / Investigation Assistant Spec

This is the intelligence layer only — it explains decisions the risk engine already made. It never makes or overrides a risk decision (see `CLAUDE.md` hard rule #1).

## Knowledge sources

- **Internal policies** (synthetic, written for this project): AML Policy, Fraud Escalation Policy, Transaction Monitoring Policy, Investigation SOP, Risk Threshold Policy.
- **Regulatory/reference documents**: appropriately licensed/public guidance only (e.g. FinCEN, FFIEC style material) — never scrape copyrighted documents.
- **Historical cases**: synthetic case data (Case 1001, 1002, ...).
- **Current transaction evidence**: structured DB data (rules triggered, scores, signals) must also be retrievable, not just documents.

## Architecture

```
Investigator Question → Query Processing → Retriever
   → [Relevant Documents + Transaction Evidence + Historical Cases]
   → Context Builder → LLM → Grounded Answer → Citations/Evidence References
```

Responses must distinguish **FACT / POLICY / INFERENCE / RECOMMENDATION**, e.g.:

```
FACT: The transaction amount is 4.7x the customer's 30-day average.
FACT: The customer used a previously unseen device.
POLICY: Transactions with multiple high-risk signals require manual review.
INFERENCE: The combination of amount anomaly and device novelty increases
           the likelihood of account compromise.
RECOMMENDATION: Route the case to manual review.
```

No unsupported claims. Every FACT/POLICY statement must trace to a retrieved source or DB record; the LLM must never invent evidence.

## Supported questions (v1 target set)

Why was this transaction flagged? / What signals contributed most to the score? / Has this customer done this before? / Which rules triggered? / What policy applies? / Are there similar historical cases? / Why HIGH/CRITICAL? / What else should an investigator review? / Summarize this case. / What evidence supports blocking this transaction?

Endpoint: `POST /api/v1/investigations/{id}/ask` (see `api.md`). Answers must return citations/evidence references alongside the text.

## Evaluation

Don't just claim "implemented RAG" — build a small eval set: `question / expected evidence / expected answer characteristics`. Measure retrieval relevance, answer groundedness, citation correctness, hallucination rate, and latency. Automate it (Phase 9 exit criteria in `plan.md`).

## Failure mode

If the LLM/vector store is unavailable: risk decisions still work; the investigation assistant returns a clear "temporarily unavailable" response. This must be tested, not assumed (see `nonfunctional.md`).
