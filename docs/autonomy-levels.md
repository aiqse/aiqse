# Autonomy Levels

Human involvement in the triangle is a stage, not the design. These six levels map the journey from craft to autonomous engineering — deliberately modeled on driving-automation levels, and governed by one rule imported from a century of manufacturing:

> **Autonomy advances only as fast as verification strengthens.**
> A level is earned by the verification system, never assumed of the generator.

## The levels

| Level | Name | Specification | Quality Model | Implementation | Verdict | Human role |
|---|---|---|---|---|---|---|
| **0** | Craft | human | implicit | human | human review | makes everything |
| **1** | Assisted | human | implicit | human + AI suggestions | human review | makes, with autocomplete |
| **2** | Directed generation | human | **human, explicit** | AI | human, reading evidence | defines quality, judges proof |
| **3** | Supervised triangle | AI drafts, human approves | AI drafts, human approves | AI | auto-accept routine, human for exceptions | approves vertices |
| **4** | Bounded autonomy | AI (within domains) | AI (within domains) | AI | automated gates | sets standards, audits exceptions |
| **5** | Autonomous engineering | AI | AI (self-improving) | AI | automated | expresses intent, audits the system |

## Level by level

### Level 0–1 — Craft and assistance

The pre-industrial baseline. Quality lives in individual skill and colleague inspection. Nothing in this repository is needed at these levels — and nothing in this repository is possible from them either, because correctness has never been made explicit.

### Level 2 — Directed generation

The first AIQSE level: humans write the specification and quality model, AI generates the interior, an automated pipeline produces evidence, a human issues the verdict by reading proof rather than code. Most teams practicing serious AI development are here or arriving here. **The rest of this repository's process documentation describes Level 2 as its default.**

*To advance:* the evidence pipeline must be complete (every quality dimension instrumented) and trusted (verdicts from evidence alone, with escapes rare and analyzed).

### Level 3 — Supervised triangle

AI drafts all three vertices — specification and quality model from requirements, verification derived from both. Humans stop authoring and start approving. Routine changes with fully green evidence auto-accept; humans handle exceptions and novel territory.

*To advance:* demonstrated record that human approval of AI-drafted vertices is no longer catching material defects — i.e., the human checkpoint has become a formality. (Manufacturing analog: the inspector who hasn't rejected a part in a year.)

### Level 4 — Bounded autonomy

Within declared domains — a service, a change class, a risk tier — AI owns the full triangle and ships on automated gates. This is the lights-out factory for routine engineering: unattended, self-halting (jidoka), mistake-proofed (poka-yoke). Humans set the standards the gates enforce, audit samples and every exception, and decide which domains qualify.

*To advance:* process-level statistics (escape rate, replay diff rate, incident rate) stable across domains and durations, plus a verification system that itself improves without human prompting.

### Level 5 — Autonomous engineering

The full journey: humans express **intent and constraints** — what should exist, what must never happen, what trade-offs are acceptable. AI maintains the entire triangle, including improving the quality models and strengthening the verification system as it learns. The human role is the one manufacturing converged on: **the standards body.** Humans calibrate the instruments, audit the system's self-reports, and answer for outcomes. They no longer inspect parts; they certify the factory.

Level 5 is a design goal, not a current capability — the point of naming it is that every artifact in this repository is written to survive the trip. The triangle needs no rewriting between Level 2 and Level 5; only the agents change.

## Rules of movement

**Advance one level at a time.** Each level's verification must catch what the retiring human checkpoint caught. Skipping levels is automation ahead of process control — the failure mode manufacturing history warns about most consistently ([industrial-lessons.md](industrial-lessons.md)).

**Quality escapes demote.** An incident that evidence should have caught drops the affected domain a level until the verification gap is closed and re-proven. Demotion is the system working, not failing — the andon principle.

**Levels are per-domain, not per-organization.** A team may run Level 4 on CRUD endpoints and Level 2 on payment logic. Risk tier decides, and the quality model's security and side-effect dimensions are where that risk is declared.

**Accountability never automates.** At every level including 5, a human or institution answers for outcomes. What changes is what they answer *with*: at Level 0, "I wrote it"; at Level 2, "I read the proof"; at Level 5, "I certify the system that produced the proof."
