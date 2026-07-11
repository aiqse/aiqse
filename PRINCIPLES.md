# Principles

Agile began as four values. SOLID is five principles. AIQSE is five principles. Everything else in this repository is commentary.

---

## Principle 1 — Quality is engineered into the process, not inspected into the code

Manufacturing learned this in 1924: when production volume exceeds what inspectors can examine, inspecting each artifact stops working — you must control the process that produces them. AI-generated code has crossed the same threshold. Reading every generated line is inspection-too-late; designing the generation-and-verification process so defects cannot pass is quality engineering.

**Practice:** invest review effort in specifications, quality models, and the verification pipeline — not in line-by-line reading of generated output.

## Principle 2 — The triangle is the primary artifact; code is its interior

Specification (what it must do), Quality (what correct means), Verification (what is proven) — these three retain value across every regeneration. Code is produced inside them, cheap to make and cheap to replace, like an interchangeable part conforming to a gauge. What you version, review, and protect is the triangle.

**Practice:** treat code as a build output. A change to the triangle is significant; a change to the interior is routine.

## Principle 3 — Every vertex is agent-agnostic

Nothing in the triangle requires a human author. A specification drafted by AI, a quality model proposed by AI, a verdict issued by an automated gate — all are legitimate, provided the triangle closes. Human involvement is a configuration of the current autonomy level, not a load-bearing wall of the discipline.

**Practice:** define each vertex by its acceptance criteria, never by who produces it. When an AI can meet the criteria, let it.

## Principle 4 — Verification must be independent of the implementation

The gauge is not made by the part. Checks derive from Specification and Quality, written or generated before and apart from the implementation they judge. When the same generation pass produces both code and its tests, the tests inherit the code's assumptions — including its bugs — and verification becomes circular self-confirmation.

**Practice:** derive expected results from the spec and quality model before implementation exists. Prefer implementation-independent instruments: golden masters, production replay, property-based tests.

## Principle 5 — Autonomy advances only as fast as verification strengthens

Toyota ran machines unattended only when the machines could detect their own defects and stop — jidoka. The same rule governs AIQSE's path from Level 0 (craft) to Level 5 (autonomous engineering): each reduction in human involvement must be preceded by verification strong enough to catch the failures that reduction would otherwise let through. Autonomy is earned by the verification system, never assumed of the generator.

**Practice:** to remove a human checkpoint, first demonstrate that the automated verification at that point catches what the human caught. Advance one level at a time; a quality escape demotes the level.

---

*These principles are a draft. If your experience contradicts one, that is exactly the feedback this project needs — open an issue.*
