# Expected Results Template

Write these **before implementation exists**. They must be derivable from the specification and quality model alone — if you need to see the code to write them, the spec is incomplete.

```markdown
# Expected Results: <feature name>

Spec: <link>    Quality Model: <link>

## Input / Output

| # | Given (state) | When (input) | Then (output) | QM dimension |
|---|---|---|---|---|
| 1 | no active subscription | valid checkout request | 201, subscription created | business_quality |
| 2 | — | email = "not-an-email" | 422, field error on `email` | input_quality |
| 3 | active subscription exists | valid checkout request | 409, no new row | business_quality |

## State Assertions

| # | After operation | Assert | QM dimension |
|---|---|---|---|
| 1 | payment failure | zero subscription rows created | state_quality |

## Non-functional

| # | Condition | Budget / rule | QM dimension |
|---|---|---|---|
| 1 | 100 rps sustained | p99 < 800 ms | performance |
| 2 | any request | card number absent from all logs | security |

## Required Events

| # | Trigger | Event | Fields | QM dimension |
|---|---|---|---|---|
| 1 | successful checkout | checkout.completed | order_id, amount, latency_ms | observability |
```
