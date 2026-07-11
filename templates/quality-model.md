# Quality Model Template

Copy this file per feature. Fill every dimension, or mark it `not-applicable` with a reason. See [docs/quality-model.md](../docs/quality-model.md) for guidance and a worked example.

```yaml
feature: <feature name>
spec: <link to specification>
author: <who defined this model>
date: <yyyy-mm-dd>

quality_model:

  input_quality:
    # Valid inputs, boundaries, behavior on invalid/malformed/adversarial input
    - ""

  business_quality:
    # Domain rules as invariants; calculations with worked examples
    - ""

  state_quality:
    # Valid/forbidden transitions, cross-store consistency, partial-failure state
    - ""

  security:
    # AuthN/AuthZ per operation, injection surfaces, data-exposure rules
    - ""

  concurrency:
    # Races, double-submits, ordering guarantees, idempotency
    - ""

  performance:
    # Latency budgets (p50/p99), throughput, resource ceilings
    - ""

  side_effects:
    # External calls, delivery guarantees, compensation on failure
    - ""

  error_handling:
    # Failure modes, user-visible behavior, retry/timeout/fallback policy
    - ""

  observability:
    # Required log events, metrics, alerts, trace propagation
    - ""

# Any dimension may instead be:
#   <dimension>: { status: not-applicable, reason: "<why>" }
```
