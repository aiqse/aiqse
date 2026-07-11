# Concept

AIQSE rests on a single observation:

> **When machines produce the implementations, trust can no longer live in the artifact. It must live in the process — exactly as manufacturing discovered a century ago.**

## The Triangle

```text
                Specification
               what it must do
                 ╱         ╲
                ╱           ╲
               ╱   ( code )  ╲
              ╱               ╲
        Quality ──────────── Verification
   what "correct" means      what is proven
```

Three artifacts, mutually constraining:

### Specification — *what it must do*

A precise statement of expected behavior: inputs, outputs, state changes, edge cases. Precise enough that two independent implementations of it would be behaviorally equivalent. A specification without the other two vertices is a wish.

### Quality — *what "correct" means*

The Quality Model decomposes correctness into nine explicit dimensions:

```yaml
input_quality:      # validation, boundaries, malformed data
business_quality:   # domain rules, invariants
state_quality:      # consistency, transitions, persistence
security:           # authn/authz, injection, data exposure
concurrency:        # races, ordering, idempotency
performance:        # latency, throughput, resource budgets
side_effects:       # external calls, notifications, writes
error_handling:     # failure modes, recovery, messaging
observability:      # logs, metrics, traces
```

Behavior (the spec) tells you what the system does on the happy path. Quality tells you what it must do everywhere else — under attack, under load, under partial failure, under concurrency.

The Quality Model is not a second design document. It is a **derived view**: the quality-relevant facts already scattered through the design (input constraints, business rules, performance notes, security requirements) re-projected along the quality dimensions — then **enriched with external norms** the design never states (RFC 5322, security baselines, the organization's latency budgets). Because it is largely derivable, AI should draft it and humans review it. Because it is enriched from outside the design, it remains an independent vertex rather than a restatement — a design error does not automatically propagate into what quality demands, and a dimension that *cannot* be filled from the design is a detected design gap. See [docs/quality-model.md](docs/quality-model.md).

### Verification — *what is proven*

Concrete checks derived from Specification and Quality **before implementation exists**, executed continuously, and collected as evidence. The critical property is independence: verification derives from the other two vertices, never from the implementation it judges. The gauge is not made by the part. See [docs/evidence.md](docs/evidence.md).

## Code is the interior

Code is not a vertex. It is what gets produced *inside* the triangle — and the triangle is what makes it trustworthy:

- An implementation is **acceptable** exactly when the triangle closes around it: spec satisfied, every quality dimension evidenced.
- An implementation is **interchangeable**: any other implementation that closes the same triangle is equally acceptable. This is Whitney's interchangeable parts, applied to software — the gauges (verification) define conformance, so the part (code) becomes replaceable and regenerable.

## The triangle is agent-agnostic

Nothing about the triangle says *who* produces each vertex. Today a human typically writes the spec and quality model while AI fills the interior — that is Level 2 of six. Every vertex can progressively be produced by AI, with humans moving from authors, to approvers, to auditors of the system itself. The triangle does not change; only the agents do. See [docs/autonomy-levels.md](docs/autonomy-levels.md).

This is the difference between a methodology about *how humans should use AI* and an engineering discipline that survives full automation.

## The flow

```text
Requirements  →  Specification  →  Quality Model  →  Expected Results
                                                            │
                                              implementation is generated
                                                            │
                                                        Evidence
                                              triangle closed? → ship
```

The traversal is described in [docs/development-process.md](docs/development-process.md).
