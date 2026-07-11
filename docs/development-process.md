# The Development Process

Developing under AIQSE means **closing the triangle** around each change:

```text
Requirements
      │   decide what problem to solve
      ▼
Specification          ┐
      │                │
      ▼                │
Quality Model          ├─ build the triangle
      │                │
      ▼                │
Expected Results       ┘
      │
      ▼
Implementation         ── fill the interior (generated)
      │
      ▼
Evidence               ── prove the triangle closed
```

The loop is iterative — a failing evidence check sends you back to the Quality Model or Specification, not into the generated code.

> **A note on "who":** every step below is agent-agnostic. This document describes **Level 2** (human-built triangle, AI-filled interior) — the level most teams practice today. At higher levels, AI drafts or owns more steps; the steps themselves do not change. See [autonomy-levels.md](autonomy-levels.md).

## Phase by phase

### 1. Requirements

Unchanged from any methodology: understand the problem, the users, the constraints. AIQSE adds one rule — requirements are never handed directly to a generator. They pass through the triangle first.

### 2. Specification

Turn the requirement into a precise behavioral contract: inputs, outputs, state changes, edge cases. The litmus test: **could two independent implementations of this spec be behaviorally equivalent?** If not, the spec has gaps the generator will fill arbitrarily.

### 3. Quality Model

Walk the nine dimensions ([quality-model.md](quality-model.md)) and write verifiable criteria for each. This is the highest-leverage hour in the process — it is where correctness gets decided. Much of it is derivation rather than invention: reorganizing constraints the specification already contains and enriching them with external norms — which is why AI should draft it and a human should judge it (see [quality-model.md](quality-model.md#where-a-quality-model-comes-from)).

### 4. Expected Results

Derive concrete checks from the spec and quality model *before implementation exists*: input/output tables, state assertions, latency budgets, required log events. Template: [templates/expected-result.md](../templates/expected-result.md).

Because they predate the implementation, expected results are independent of it — the gauge is not made by the part (Principle 4).

### 5. Implementation

Generate with full context: specification, quality model, expected results, codebase conventions. Two rules:

- **Regenerate, don't archaeologize.** When the triangle changes, prefer regenerating the interior to hand-patching it. Interchangeable parts, not repaired ones.
- **Surprises are spec bugs.** If the output surprises you, the specification was ambiguous. Fix it there.

### 6. Evidence

Run the verification pipeline ([test-strategy.md](test-strategy.md)) and collect the results into an evidence record ([evidence.md](evidence.md)). The definition of done: **every quality-model line has passing evidence — the triangle is closed.**

## Roles at Level 2

| Activity | Owner |
|---|---|
| Requirements, Specification | Human |
| Quality Model, Expected Results | Human (AI may draft, human decides) |
| Implementation | AI (human intervenes by exception) |
| Evidence pipeline | Automated |
| Verdict — does evidence close the triangle? | Human |

At Level 3 the human moves from author to approver; at Level 4–5, to standards-setter and auditor. The table's *rows* are permanent; the *owners* are not.

## Working incrementally

AIQSE is compatible with agile cadences. A "story" is a triangle delta: spec delta + quality-model delta + regenerated interior + fresh evidence. Small stories mean small triangles — the process scales down as well as up.
