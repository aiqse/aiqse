# Evidence

Evidence is the final artifact of the AIQSE pipeline: proof, collected and preserved, that an implementation satisfies its Quality Model.

> A quality claim without evidence is an assumption.

## What counts as evidence

| Evidence type | Proves | Typical source |
|---|---|---|
| Test results | Input, business, state, error-handling dimensions | CI test runs |
| Benchmark output | Performance budgets | Load/perf harness |
| Security scan reports | Security dimension | SAST, DAST, dependency audit |
| Golden master diffs | Behavioral equivalence across regenerations | Replay harness |
| Production replay results | Behavior under real traffic | Shadow/canary infrastructure |
| Observability assertions | Required events and metrics are emitted | Log/metric assertions in CI |

What does **not** count: "the AI said it handled that case," "it looks right," a green build with no mapping to the quality model, or line-coverage percentages by themselves.

## The evidence record

Each change carries an evidence record linking every quality-model line to its proof:

```yaml
feature: subscription-checkout
spec: specs/checkout.md          # version/commit pinned
quality_model: qm/checkout.yaml  # version/commit pinned
implementation: <commit sha>
generated_by: <model + prompt reference>
evidence:
  input_quality:
    - check: "invalid email → 422"
      source: ci/run-4812#test_email_validation
      result: pass
  security:
    - check: "card number never logged"
      source: ci/run-4812#log_scrub_assertions
      result: pass
  performance:
    - check: "checkout p99 < 800ms @ 100rps"
      source: perf/run-0921
      result: pass (p99 = 640ms)
  # ... every dimension, every line
verdict: accepted
verdict_by: <human>
date: 2026-07-09
```

## Rules

**Complete.** Every quality-model line has an entry — passing, or explicitly waived with a reason and an owner. The definition of done is a complete evidence record, not a merged PR.

**Current.** Evidence is bound to a specific implementation commit. Regenerated the code? The old evidence is void; run the pipeline again. (This is why the pipeline must be fully automated.)

**Preserved.** Evidence records are versioned alongside releases. When an incident occurs, the first question is answerable in minutes: *what did we prove about this behavior, and when?*

**Human-verdicted.** Automation produces evidence; a human issues the verdict. The sign-off is no longer "I read the code" — it is "I read the proof."

## Why evidence, not review

A reviewer reading 2,000 generated lines approves what they *hope* is true. An evidence record shows what *is* true, dimension by dimension, against criteria decided before the code existed. As generation volume grows, review scales linearly with human attention; evidence scales with automation. That asymmetry is the practical argument for AIQSE.
