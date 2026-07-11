# Test Strategy

In AIQSE the question is not **"did we write tests?"** but **"can we prove quality?"**

Tests are one instrument among several. The strategy is a pipeline that converts a Quality Model into evidence:

```text
Quality Model
      │
      ▼
Unit Tests            ── input quality, business rules, error handling
      │
      ▼
Integration Tests     ── state quality, side effects, security
      │
      ▼
E2E Tests             ── user-visible behavior end to end
      │
      ▼
Golden Master         ── behavioral equivalence across regenerations
      │
      ▼
Production Replay     ── real traffic against the new implementation
      │
      ▼
Evidence
```

## Two rules that change everything

**1. Checks derive from the Quality Model, not from the code.**
Traditional test-writing reads the implementation and asks "what should I cover?" In AIQSE, checks are derived from the Quality Model and Expected Results — written before or independently of the implementation. Every quality-model line maps to at least one check; a line with no check is an unverified quality claim.

**2. The AI that wrote the code must not be the sole author of its verification.**
When one generation pass produces both code and tests, the tests tend to encode the code's assumptions — including its bugs. Break the circularity with any of: human-authored expected results (the default), a separate generation pass given only the spec and quality model, or implementation-independent checks (golden master, replay, property-based tests).

## Where this leaves TDD

TDD contained one insight AIQSE keeps: **checks exist before the implementation.** The rest of it was cognitive scaffolding for a *human* implementer — red-green-refactor paced a person's thinking and gave the author fast feedback. TDD was always a design discipline wearing a testing name; its tests were the developer's own verification, never the QA gate.

With a machine implementer, the scaffolding loses its purpose but the insight gets promoted. An agent that writes tests and runs them to confirm its own output is the craftsman checking his own work — legitimate *in-process* feedback for the generation loop, inadmissible as evidence. The exit gauge is the triangle-derived suite. Same file format, different artifact: what a check verifies is decided by **what it was derived from**, not by what framework runs it.

The corollary is a filter for test value: **a test that must change whenever the implementation changes is part of the implementation.** Tests pinned to class structure die with each regeneration and therefore verify nothing across regenerations. Instruments attach to behavior boundaries — contracts, state transitions, invariants — which is exactly what lets them survive when the interior is regenerated. Surviving regeneration unchanged, and still passing, is the test of a test.

## Layers

### Unit tests
Verify input quality, business calculations, and error handling in isolation. Property-based tests are particularly valuable here: they check invariants the AI never saw as examples.

### Integration tests
Verify state quality (transactions, consistency, partial failure), side effects (external calls fire correctly and compensate on failure), and security rules (authorization enforced at the boundary that matters).

### E2E tests
A thin layer proving the user-visible flows in the specification actually work assembled. Keep them few; they are evidence of assembly, not of dimension-level quality.

### Golden Master
Record the full observable behavior of the current implementation (responses, state changes, emitted events) across a broad input corpus. When the AI regenerates the implementation, replay the corpus and diff. **This is the natural regression instrument for AIQSE** — regeneration replaces patching, so "is the new implementation behaviorally equivalent where it should be?" becomes the central regression question.

### Production Replay
Capture real (sanitized) production traffic and replay it against the candidate implementation — shadow traffic, replay-and-diff, or canary. Synthetic checks prove what you thought to check; replay proves behavior under what actually happens.

### Non-functional evidence
Performance (benchmarks against the stated budgets), security (SAST/DAST, dependency audit — AI-generated code inherits training-data habits, so automated scanning is non-negotiable), and observability (assert that required log events and metrics are actually emitted — observability is a testable requirement, not a hope).

## Coverage, redefined

Line coverage measures how much code the tests touch. AIQSE measures **quality-model coverage**: what fraction of quality-model lines have passing, current evidence. 100% line coverage with an incomplete quality model is circular self-confirmation; the inverse is what correctness looks like.
