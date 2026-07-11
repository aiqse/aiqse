# The Quality Model

The Quality Model is the core artifact of AIQSE. It decomposes the vague claim "this software is correct" into nine explicit, individually verifiable dimensions.

Every feature gets a Quality Model before implementation begins. Every dimension is either specified or explicitly marked `not-applicable` with a reason — never silently skipped. A ready-to-use template is at [templates/quality-model.md](../templates/quality-model.md).

## The nine dimensions

### 1. Input Quality

What inputs are valid, and what happens to everything else?

- Type, format, range, and length constraints for every input
- Boundary values (empty, zero, max, unicode, null)
- Behavior on malformed, missing, and adversarial input

### 2. Business Quality

Do domain rules and invariants hold?

- Business rules stated as invariants ("an order total never goes negative")
- Calculations with worked examples, including rounding and currency rules
- Domain edge cases (leap years, timezone boundaries, plan downgrades mid-cycle)

### 3. State Quality

Is state consistent before, during, and after the operation?

- Valid state transitions, and which transitions are forbidden
- Consistency across stores (DB, cache, search index)
- Partial-failure behavior: what state remains if step 3 of 5 fails?

### 4. Security

Who may do this, and what must never leak?

- Authentication and authorization requirements per operation
- Injection, tampering, and privilege-escalation surfaces
- Data exposure rules: what never appears in responses, logs, or errors

### 5. Concurrency

What happens when two things happen at once?

- Race conditions: concurrent writes, double-submits, retries
- Required ordering guarantees, idempotency of operations
- Locking / optimistic-concurrency expectations

### 6. Performance

How fast, how much, at what scale?

- Latency budgets (p50/p99), throughput targets
- Resource ceilings: memory, connections, query counts (N+1 budgets)
- Behavior at expected scale and at 10× expected scale

### 7. Side Effects

What does this operation touch beyond its return value?

- External calls (payments, email, webhooks) — and their failure handling
- At-most-once / at-least-once guarantees for each effect
- Compensation: how effects are undone when the operation fails after they fire

### 8. Error Handling

How does it fail?

- Every anticipated failure mode and its user-visible behavior
- Retry, timeout, and fallback policies
- Error messages: actionable for users, non-leaky for attackers

### 9. Observability

Can we see it working — and see it failing?

- Required log events with structure and level
- Metrics and alert thresholds
- Traces/correlation IDs across service boundaries

## Format

```yaml
feature: <name>
spec: <link to specification>
quality_model:
  input_quality:
    - "Email must be RFC 5322 valid; invalid → 422 with field-level error"
    - "Name: 1–100 unicode chars; empty or >100 → 422"
  business_quality:
    - "A user may hold at most one active subscription (invariant)"
  state_quality:
    - "On payment failure, no subscription row is created (atomicity)"
  security:
    - "Only the account owner or an admin may cancel (403 otherwise)"
    - "Card numbers never appear in logs or error payloads"
  concurrency:
    - "Double-submit of checkout is idempotent via idempotency-key"
  performance:
    - "Checkout p99 < 800 ms at 100 rps"
  side_effects:
    - "Confirmation email sent at-most-once, after commit"
  error_handling:
    - "Payment-provider timeout → user-visible retry prompt, no charge"
  observability:
    - "checkout.completed event logged with order_id, amount, latency_ms"
```

## Where a Quality Model comes from

The Quality Model is not a second specification, and it is not written from scratch.

A specification describes **behavior** — inputs, outputs, state changes, edge cases. A Quality Model describes **assurance** — what must hold while that behavior is delivered. Yet most of what a Quality Model contains is already present in the specification (and, in organizations that keep them, the surrounding design documents), scattered: an input constraint here, a business rule there, a performance note in an appendix, security requirements in a separate document. The Quality Model reorganizes those facts along the nine dimensions.

In database terms, it is a **view**. Specifications are written per-feature, per-screen, per-API; the Quality Model cuts across them and re-projects the same facts along the quality axis, into the one form verification can consume.

But it is not a *pure* projection. Two other kinds of information enter during derivation:

1. **External norms.** The specification says "email"; the Quality Model says "RFC 5322 valid." That constraint comes from standards, security baselines, and organizational budgets — from outside the specification. This is the relationship aircraft safety requirements have to the aircraft's design: "must stop with one brake failed" is not derived from the brake drawings.
2. **Absence made explicit.** Every dimension must be specified or marked `not-applicable` with a reason — so when derivation cannot fill a dimension, because no performance budget exists anywhere, that is not a gap in the Quality Model. It is a **detected gap in the specification**. Deriving the Quality Model doubles as a specification audit.

So: *Quality Model = projection of the specification + enrichment from external norms, with specification gaps surfaced.*

### Doesn't derivation collapse the triangle?

If the Quality Model derives from the specification, and the implementation is generated from both, hasn't the triangle folded back into the line that [philosophy.md](philosophy.md) rejects? No — because the quality vertex's independence never came from being written by a different hand. It comes from three things derivation preserves:

- **Outside information.** The external norms are precisely what the specification does not contain. A specification error cannot rewrite RFC 5322 or the organization's latency budget.
- **Forced completeness.** The nine-dimension walk interrogates the specification from angles it was never written in. A specification can be silent about concurrency; a Quality Model cannot.
- **A reviewed gate.** A human judges the two non-mechanical parts of the derivation: whether the right norms were applied, and whether the surfaced gaps are real.

The gauge is not made by the part — and not made *only* from the part's drawing, either.

### Who writes it

The projection step is mechanical enough that AI should perform it. Humans transcribing constraints that already exist in the specification is exactly the kind of work that belongs to the machine:

```text
specification → AI derives quality model → human review → quality model
```

The Level 2 role table in [development-process.md](development-process.md) already anticipates this ("AI may draft, human decides"). Making derivation the default moves this vertex toward Level 3 of [autonomy-levels.md](autonomy-levels.md) — AI drafts, human approves — and the review is not a courtesy pass: it is where the norms and the gaps get judged.

### Model vs. instance

Strictly speaking, "Quality Model" names the nine-dimension schema above — the fixed frame, analogous to ISO 25010. Each feature's YAML is an **instance** of it: a quality view over that feature's specification. This repository uses "Quality Model" for both where context makes it unambiguous.

## Why this matters

Without a Quality Model, an AI-generated implementation's behavior on all of these dimensions is **whatever the model happened to generate**. Someone decided your concurrency semantics — and it wasn't you.

The Quality Model is also the contract that makes evidence meaningful: each line above maps to at least one evidence source in [test-strategy.md](test-strategy.md).
