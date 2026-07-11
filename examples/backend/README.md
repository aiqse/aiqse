# AIQSE × Backend Services

Backend services are where all nine dimensions apply at full strength — and where the evidence pipeline pays for itself.

| Dimension | In backend terms |
|---|---|
| Input quality | Request schema validation at the boundary, size limits, encoding edge cases |
| Business quality | Domain invariants enforced in one place, money/time/rounding rules with worked examples |
| State quality | Transaction boundaries, outbox pattern, cross-store consistency, migration safety |
| Security | AuthN/AuthZ per endpoint, injection surfaces, tenant isolation, PII handling in logs |
| Concurrency | Idempotency keys, optimistic locking, exactly-once vs at-least-once per effect |
| Performance | p99 budgets per endpoint, query-count ceilings, connection-pool behavior under load |
| Side effects | Payments/email/webhooks: delivery guarantees and compensation on failure |
| Error handling | Failure-mode table per dependency (DB down, upstream timeout), consistent error contract |
| Observability | Structured logs with correlation IDs, RED metrics, alert thresholds tied to budgets |

**Evidence sources:** unit + property-based tests, integration tests with real dependencies (Testcontainers), contract tests (OpenAPI/Pact), golden master over a recorded request corpus, load tests against stated budgets, SAST/dependency scanning, production replay (shadow or canary).

**Start here:** [templates/api-quality.md](../../templates/api-quality.md) — the endpoint-level specialization of the Quality Model.

## Golden master, concretely

Backend services are the ideal golden-master target:

1. Record a corpus: requests + responses + resulting state + emitted events.
2. AI regenerates the service (new spec version, refactor, model upgrade).
3. Replay the corpus against the candidate; diff everything observable.
4. Intended differences must map to spec changes; everything else is a regression.

This turns "can we trust the regenerated implementation?" from a review problem into a diff.

*A full worked example is planned. Contributions welcome.*
