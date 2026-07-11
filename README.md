# AI Quality-Driven Software Engineering (Draft)

> **Status: Draft / Request for Comments.**
> This is not a finished discipline. It is a proposal about how software engineering works when machines write the code. Issues and pull requests are welcome — we want to grow this with the community.

---

## The AIQSE Manifesto

We are uncovering better ways of building software by letting machines produce implementations while quality is engineered into the process. Through this work we have come to value:

- **Quality definitions** over code reviews
- **Executable specifications** over implementation details
- **Evidence** over assumptions
- **Process control** over artifact inspection
- **Continuous verification** over one-time testing

That is, while there is value in the items on the right, we value the items on the left more.

---

## The Triangle

AIQSE is built on a triangle, not a pipeline:

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

- **Specification** defines expected behavior.
- **Quality** defines correctness — nine explicit dimensions, from input handling to concurrency to observability.
- **Verification** produces proof that both hold.

**Code is not a vertex. Code is the interior of the triangle** — generated, disposable, regenerable. An implementation is acceptable exactly when the triangle closes around it. It does not matter who — or what — produced any vertex: the triangle is agent-agnostic, which is what makes full automation a design goal rather than a contradiction.

## This is not an extension of Spec-Driven Development

SDD is a line: specification → implementation. A line can only ask *"did we build what the spec says?"* — and its verification is derived from the same spec it checks, by the same agent that implemented it. Circular.

The triangle adds what the line is missing: an independent definition of correctness (Quality) and proof derived from *it*, not from the implementation (Verification).

| | Shape | Correctness is | Verification comes from |
|---|---|---|---|
| TDD | loop | whatever the tests assert | the implementer's own tests |
| BDD | line | expected behavior | scenarios (one quality dimension) |
| SDD | line | implicit in the spec | the spec itself |
| **AIQSE** | **triangle** | **explicit, nine-dimensional** | **the Quality vertex, independent of the implementation** |

## Learning from a century of industrial quality

Software is going through its industrial revolution: implementation has become machine work. Manufacturing faced the identical trust problem a hundred years ago and solved it — Shewhart (1924): quality is a property of the process, not the artifact; Deming: inspection after the fact is too late; Toyota: machines that stop themselves when quality deviates. AIQSE imports those lessons directly. See [docs/industrial-lessons.md](docs/industrial-lessons.md).

## The destination is autonomy

Human involvement in the triangle is a stage, not the design. AIQSE defines six levels, from craft to autonomous engineering:

| Level | Name | Human role |
|---|---|---|
| 0 | Craft | writes everything |
| 1 | Assisted | writes with AI suggestions |
| 2 | Directed generation | defines spec + quality, verdicts evidence |
| 3 | Supervised triangle | approves AI-drafted vertices |
| 4 | Bounded autonomy | sets standards, audits exceptions |
| 5 | Autonomous engineering | expresses intent, audits the system |

Governing rule: **autonomy may advance only as fast as verification strengthens.** See [docs/autonomy-levels.md](docs/autonomy-levels.md).

## The five principles

1. **Quality is engineered into the process, not inspected into the code.**
2. **The triangle — Specification, Quality, Verification — is the primary artifact; code is its interior.**
3. **Every vertex is agent-agnostic; the triangle must close regardless of who produces it.**
4. **Verification must be independent of the implementation.**
5. **Autonomy advances only as fast as verification strengthens.**

See [PRINCIPLES.md](PRINCIPLES.md).

## Explore

| Document | What it covers |
|---|---|
| [CONCEPT.md](CONCEPT.md) | The triangle and its vertices, in one page |
| [PRINCIPLES.md](PRINCIPLES.md) | The five principles, expanded |
| [docs/philosophy.md](docs/philosophy.md) | Why the pipeline became a triangle |
| [docs/industrial-lessons.md](docs/industrial-lessons.md) | What a century of manufacturing quality teaches AI-era software |
| [docs/autonomy-levels.md](docs/autonomy-levels.md) | Levels 0–5: the path to autonomous engineering |
| [docs/quality-model.md](docs/quality-model.md) | The nine quality dimensions |
| [docs/development-process.md](docs/development-process.md) | Closing the triangle, step by step |
| [docs/test-strategy.md](docs/test-strategy.md) | From "writing tests" to "proving quality" |
| [docs/evidence.md](docs/evidence.md) | What counts as proof, and how to record it |
| [templates/](templates/) | Quality Model, Expected Result, and API Quality templates |
| [examples/](examples/) | React, Next.js, and backend mappings |

## Contributing

This proposal will only become useful through real-world application. If you try AIQSE on a project — or disagree with any part of it — please [open an issue](../../issues) or send a pull request. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Documentation is licensed under [CC BY 4.0](LICENSE).
