# The Autonomous Loop

Every other document in this repository describes the triangle as a *structure*: three vertices, nine dimensions, six levels. None of them say what happens, mechanically, when a machine is the one closing the triangle with nobody watching in real time. This document is the missing control layer — the algorithm that lets Level 4–5 run unattended without becoming Level 4–5 in name only.

It answers four questions a structural description cannot:

1. When evidence fails, what does the system do *next* — and how does it avoid looping forever?
2. Who verifies the verifier, once the verifier is what stands between generation and release?
3. What has to be true for a release verdict to be issued by software instead of a person?
4. Inside a single generation pass, how does an agent decide what to try, when to stop, and when to escalate?

> **Prerequisite.** This document assumes the triangle (spec, quality model, expected results) already exists for the change in flight. It governs the loop *inside* [development-process.md](development-process.md) step 5–6 — implementation through evidence — not the authoring of the triangle itself.

---

## 1. Convergence control — closing the loop without looping forever

[development-process.md](development-process.md) states: "a failing evidence check sends you back to the Quality Model or Specification, not into the generated code." That sentence hides a decision tree. Making it explicit is what turns a slogan into something an unattended system can execute.

### 1.1 The failure-routing table

When an evidence check fails, the loop must classify *why* before it can decide *where* to send the failure. Patching code blindly until tests pass is exactly the self-confirming circularity Principle 4 forbids — so routing, not patching, is the first move.

| Failure signature | Likely cause | Route to | Action |
|---|---|---|---|
| Implementation contradicts an explicit, unambiguous quality-model line | Generation defect | Implementation | Regenerate the interior with the failing check added to context as a negative example |
| Two independent implementations of the same spec line produce different evidence | Specification is underdetermined | Specification | Halt; flag the spec line as ambiguous; do not regenerate until resolved |
| A dimension has no corresponding quality-model line, yet a check fails against it | Quality Model gap | Quality Model | Halt; the gap is real (see [quality-model.md](quality-model.md#where-a-quality-model-comes-from)); draft the missing line, route back through review at the current autonomy level |
| Check itself references code structure, not behavior | Verification defect | Verification | Discard or rewrite the check; a test that must change when the implementation changes is not evidence ([test-strategy.md](test-strategy.md)) |
| Evidence intermittently flips pass/fail on an unchanged triangle | Non-determinism (flaky test, race in the harness) | Verification | Quarantine the check, do not let it gate; a flaky gate is a gap in the Quality Model's concurrency dimension until proven otherwise |

The classification itself must be evidenced, not guessed — the same rule that governs everything else in this pipeline. A generation agent proposing "this is a spec ambiguity, not a code bug" needs to show the two divergent readings, not merely assert one.

### 1.2 Regeneration budget: bounding the retry loop

An unattended system that regenerates on every failure needs an explicit stopping rule, or a bad spec produces an infinite loop of confident, wrong regenerations.

```yaml
regeneration_policy:
  max_attempts_per_check: 3        # same check, same triangle version
  max_attempts_per_change: 8       # across all checks, one triangle delta
  escalation_on_exhaustion: halt_and_flag   # never: relax the check
  attempt_diversity_required: true # attempt N+1 must differ in approach from attempt N,
                                    # not just be a re-roll with the same prompt
```

**Why bound by attempts, not by time.** A generation loop that retries until a timeout can still be making the same mistake every time; bounding by *distinct attempts* forces the loop to actually try something different, and makes exhaustion a legible, countable event rather than a silent timeout.

**Why diversity is required, not optional.** Re-running an identical prompt against a stochastic generator and hoping for a different output is not convergence, it is gambling on temperature. Attempt N+1 must change something structural: a different decomposition of the problem, a different library/pattern, or — if the first two attempts agree with each other but disagree with the check — a formal escalation to "the check may be wrong" (route 4 in §1.1) rather than a third blind attempt.

**Why exhaustion halts rather than relaxes.** The one move that must never be automatic is loosening the quality-model line or the check itself to make a failing implementation pass. That is quality escaping through the back door. Exhaustion is a jidoka event (§2.3): the line stops, and a human or a higher-privileged process is paged with the full attempt history attached as evidence of *why* it stopped.

### 1.3 Detecting divergence vs. progress

A bounded retry count prevents infinite loops, but says nothing about whether attempts 1 through 3 were converging toward a fix or wandering. Track evidence delta across attempts:

```yaml
attempt_1: { checks_passing: 14/20, new_failures: [] }
attempt_2: { checks_passing: 17/20, new_failures: [] }        # converging — continue
attempt_3: { checks_passing: 12/20, new_failures: [state_quality.3] }  # diverging — stop early, don't spend attempt budget on a regression
```

If an attempt's passing-check count drops below the previous attempt's, or introduces a failure in a dimension that previously passed, the loop stops immediately regardless of remaining budget and escalates as a regression, not a routine retry. Spending the full attempt budget on a diverging trajectory wastes the budget and — worse — the final "attempt 3" artifact is not automatically the best one seen; the loop must keep and offer the best-scoring attempt, not the last one.

---

## 2. The meta-loop — verification that audits and strengthens itself

Levels 4–5 ([autonomy-levels.md](autonomy-levels.md)) require "a verification system that itself improves without human prompting" and "self-improving" quality models. Naming that requirement is not the same as specifying it. This section is the mechanism.

### 2.1 What the meta-loop watches

The inner loop (§1) closes one triangle. The meta-loop runs continuously across *all* closed triangles and asks whether the verification system itself is still trustworthy:

```yaml
meta_loop_signals:
  escape_rate:            # incidents traceable to a passing evidence record
    window: rolling_90d
    threshold: domain-specific, set at Level 4 entry
  false_halt_rate:        # loop halted, human found no real defect
    signals_over-strict or miscalibrated checks
  regeneration_attempt_distribution:
    signals: specs that are chronically ambiguous (high attempts-to-converge)
  quarantined_check_count:
    signals: verification debt accumulating (flaky checks parked, never fixed)
  quality_model_gap_rate:  # dimensions marked not-applicable without
                            # a reviewed reason, or gaps found post-incident
    signals: derivation (quality-model.md) is being rubber-stamped, not reviewed
```

These are the same instruments Shewhart's control chart pattern calls for ([industrial-lessons.md](industrial-lessons.md)): not judging one artifact, but watching whether the *process* is in control.

### 2.2 The self-strengthening cycle

When an escape occurs — an incident that a passing evidence record should have prevented — it is not just a bug, it is a **verification gap report**, and it drives a mandatory cycle before the affected domain can return to its prior autonomy level:

```text
Incident occurs
      │
      ▼
Root-cause: which quality-model dimension should have caught this?
      │
      ├─ Dimension existed, check was wrong  → fix/strengthen the check
      ├─ Dimension existed, no check for it   → author the missing check
      └─ Dimension itself was missing/wrong   → amend the Quality Model
                                                 (propagates to every quality
                                                 model sharing that domain's
                                                 template — not just this feature)
      │
      ▼
New check added to the regression corpus (golden master / replay set)
      │
      ▼
Re-run the full evidence pipeline against a sample of recently-closed
triangles in the domain, retroactively — did any of them have latent
instances of the same gap?
      │
      ▼
Demotion lifted only when the strengthened check has run clean across
that retroactive sample plus N subsequent real changes (N set per risk tier)
```

The retroactive replay step is what makes this a *strengthening* loop rather than a one-off patch: a gap found once is assumed to be latent everywhere the same pattern was generated, and the loop must go check, not wait for a second incident to prove the point.

### 2.3 Jidoka thresholds — when the meta-loop halts autonomy itself

Individual failures halt individual changes (§1.2). The meta-loop's job is to halt *autonomy at a domain*, automatically, when its aggregate signals cross a line — this is jidoka applied one level up, to the factory's control system rather than a single machine:

| Trigger | Automatic response |
|---|---|
| Escape rate exceeds the domain's Level-4/5 entry threshold | Domain demotes one level immediately (Principle 5); re-entry requires the cycle in §2.2 to complete |
| Two consecutive quality escapes trace to the *same* quality-model dimension | That dimension is frozen for AI-authored edits across the domain until a human re-derives it |
| Quarantined-check count exceeds a configured ceiling | New feature work in the domain pauses; loop redirects capacity to verification debt before generating more interior |
| Regeneration-attempt distribution shows rising median attempts-to-converge | Flag as a leading indicator — specs in this domain are degrading in precision — before it becomes an escape |

The meta-loop is not permitted to *raise* a domain's autonomy level on its own signals — only lower it. Advancement is a Principle-5 decision made deliberately (§3.2), never a side effect of a good quarter of metrics; a control system that promotes itself on its own dashboard is exactly the unaccountable loop this document exists to prevent.

---

## 3. Verdict automation — when release requires zero human checkpoints

[evidence.md](evidence.md) states "Human-verdicted. Automation produces evidence; a human issues the verdict." That rule is correct for Level 2 and remains the default. This section specifies the conditions under which the verdict step itself may be automated — Level 4's "automated gates" and Level 5's "automated" verdict column, made concrete rather than asserted.

### 3.1 The automatic-verdict gate

An automated `accepted` verdict may be issued, with no human in the loop for that specific change, only when **all** of the following hold simultaneously:

```yaml
auto_verdict_conditions:
  triangle_completeness:
    every_quality_model_line_has_evidence: true
    no_lines_waived: true            # a waived line always forces human verdict
  evidence_freshness:
    bound_to_current_implementation_commit: true    # evidence.md "Current" rule
  domain_autonomy_level:
    minimum: 4                       # Level 2-3 domains always require human verdict
  regression_instruments_clean:
    golden_master_diff: no_unexplained_deltas
    production_replay: within_declared_tolerance     # where replay infra exists
  meta_loop_state:
    domain_escape_rate: below_threshold              # §2.1 — no open jidoka halt
    no_dimension_frozen: true                        # §2.3
  change_risk_tier:
    within_declared_bounded_autonomy_domain: true     # autonomy-levels.md,
                                                       # "levels are per-domain"
  novelty_check:
    triangle_delta_pattern_seen_before: true          # see §3.3
```

Any single condition failing routes the verdict to a human — this list is a set of gates in series, not a score to average. There is no partial credit: a change that is behaviorally perfect but touches a dimension whose check is currently quarantined (§2.1) does not get auto-verdicted, because the instrument measuring it is not currently trusted.

### 3.2 Novelty is the gate the metrics can't see

A domain can have a pristine escape rate and still contain a change that should not auto-ship: the first change of a genuinely new kind. §2.3 explicitly forbids the meta-loop from self-promoting on aggregate metrics; this is the corresponding rule for individual changes.

**Definition of novel, for this purpose:** a triangle delta whose quality-model shape (which dimensions changed, and how) has no sufficiently similar precedent among previously auto-verdicted changes in the domain. "Sufficiently similar" is domain-configured, not universal — a payments domain sets a tighter similarity bar than a CRUD-endpoints domain.

Novel changes always route to human verdict once, regardless of how clean their evidence is, specifically so the *next* similar change has a precedent to be compared against. This is the mechanism by which Level 4 domains accumulate the track record that Level 5 requires — autonomy is earned change-by-change, not declared.

### 3.3 What "automated verdict" actually issues

An automated verdict is not silence — it produces the same evidence record [evidence.md](evidence.md) specifies, with the human field replaced by the gate configuration that fired:

```yaml
verdict: accepted
verdict_by: automated-gate
gate_config_version: <commit sha of auto_verdict_conditions>
conditions_satisfied: [triangle_completeness, evidence_freshness, ...]
domain: checkout-crud
autonomy_level: 4
human_audit_status: pending    # see §3.4
```

### 3.4 Audit is not optional, it is deferred

Level 4's human role is "audits exceptions" — but a domain running well produces no exceptions to audit, which is exactly the condition under which oversight silently evaporates if nothing forces it. The loop must generate its own audit sample:

```yaml
audit_sampling:
  rate: max(1_in_20_changes, all_novel_changes, all_near-threshold_changes)
  near_threshold: any change where a gate condition passed with < 10% margin
                  (e.g. golden master diff rate just under the cutoff)
  audit_deadline: 5_business_days   # audit happens after ship, not before —
                                     # this is post-hoc calibration of the gate,
                                     # not a checkpoint the change waits on
  audit_finds_defect: treated as an escape → §2.2 cycle triggers
```

Auditing after ship rather than before is deliberate: gating on human review before release would just reintroduce the human checkpoint Level 4 exists to remove. The sample exists to calibrate the *gate*, not to re-approve each change — its output is meta-loop input (§2.1), not a second verdict.

---

## 4. Agentic execution — loop control inside a single generation pass

Sections 1–3 govern the loop *across* attempts and across time. This section governs the loop *inside* one agent's execution of step 5 (Implementation) — the actual tool-call loop a coding agent runs to go from "here is the triangle" to "here is a candidate interior."

This is the one section where "agent-agnostic" (Principle 3) meets a concrete reasoning-loop shape, because unlike the other vertices, code generation today is usually performed by an agent making a sequence of tool calls, not a single completion.

### 4.1 Scope discipline: the agent's context is the triangle, not the codebase

The agent's loop is seeded with the specification, quality model, and expected results for the change — never with "go improve this area" as an open-ended mandate. An agent that is allowed to widen its own scope mid-loop (refactor adjacent code, "while I'm here" changes) is an agent generating an interior that the triangle was never built to close around; those changes fall outside the evidence record and are, by construction, unverified.

```yaml
agent_context_bounds:
  in_scope: [specification_delta, quality_model_delta, expected_results,
             codebase_conventions, interfaces_the_change_must_satisfy]
  out_of_scope: everything else — a wider change requires a wider triangle first
  scope_expansion_mid_loop: requires halting and returning to Requirements/
                            Specification (development-process.md step 1-2),
                            never a silent decision inside the generation loop
```

### 4.2 Subtask decomposition follows quality-model dimensions, not code structure

When an agent breaks a generation task into subtasks, the decomposition that keeps evidence traceable is one subtask per quality dimension's requirements, not one subtask per file or class:

- decomposing by dimension keeps each subtask's output mappable back to the check that will verify it (§3.1's `conditions_satisfied` needs this traceability to exist at all)
- decomposing by file/class structure produces exactly the kind of implementation-shaped artifact [test-strategy.md](test-strategy.md) warns against: something that dies with the next regeneration and verifies nothing across it

### 4.3 Self-evaluation inside the loop is scaffolding, not evidence

An agent that runs its own tests mid-loop to decide whether to keep iterating is doing legitimate, valuable work — [test-strategy.md](test-strategy.md)'s "in-process feedback for the generation loop." The loop-control rule is narrow but load-bearing: **a check the agent ran on itself, mid-loop, to decide its own next move, can advance the agent to its next attempt, but cannot appear in the evidence record that produces a verdict.** The exit gauge (§3.1) is always the triangle-derived suite run independently of the generation agent's own session — same rule as test-strategy.md's TDD section, applied to the tool-call loop specifically.

### 4.4 Stopping conditions for the agent's own loop

Independent of the outer regeneration budget (§1.2), the agent's inner loop needs its own bounded stopping rule, or a single "attempt" silently becomes an unbounded tool-call session:

```yaml
inner_loop_bounds:
  max_tool_calls_per_attempt: <budget, domain-configured>
  max_self_correction_cycles: <budget — agent runs own tests, fixes, reruns>
  stop_on: [expected_results_locally_satisfied,   # ready for independent evidence pass
            budget_exhausted,                     # → counts as one attempt in §1.2,
                                                    #   not a silent retry inside it
            scope_expansion_needed]                # → halt per §4.1, don't self-expand
```

An inner loop that exhausts its budget consumes exactly one slot of the outer `max_attempts_per_check` (§1.2) — the two budgets are not independent pools an agent can use to multiply its effective retry count.

---

## How the four sections compose

```text
                    ┌─────────────────────────────────────────┐
                    │              Meta-loop (§2)              │
                    │  watches escape rate, false halts,       │
                    │  quarantine debt — across all domains    │
                    │  can only DEMOTE autonomy, never promote │
                    └───────────────┬───────────────────────────┘
                                    │ sets thresholds/conditions for
                                    ▼
┌──────────────┐   ┌──────────────────────────┐   ┌───────────────────┐
│ Agentic exec  │──▶│   Convergence control    │──▶│  Verdict gate (§3)  │──▶ ship /
│    (§4)       │   │        (§1)              │   │  auto or human     │    halt
│ one attempt   │   │ bounded retries, routes   │   │  gated on meta-loop │
│               │◀──│ failures to spec/QM/impl  │   │  state + novelty    │
└──────────────┘   └──────────────────────────┘   └───────────────────┘
       ▲                                                      │
       │                    escalation on exhaustion           │
       └──────────────────────────────────────────────────────┘
                    (jidoka: halt and page, never silently relax a check)
```

Section 1 is the loop that closes one triangle. Section 4 is what runs inside one of its attempts. Section 3 is the gate the loop exits through. Section 2 is what watches the gate itself and is the only piece of this document with authority to pull a domain backward — a deliberate asymmetry: nothing in this loop is allowed to promote its own autonomy, only to demote it. Promotion is a human act of standard-setting (Principle 5), by design.

---

## What this document does not solve

Naming these mechanisms is not the same as having working implementations of them — golden master corpora, replay infrastructure, and the statistical thresholds in §2.1 all require investment most teams have not made yet. This document specifies the *shape* the control loop must have for Level 4–5 to be more than a label; it does not certify that any particular pipeline has built it. The gap between "specified" and "operating in production" is exactly the gap [autonomy-levels.md](autonomy-levels.md) means when it says a level is *earned* by the verification system, not assumed of the generator.
