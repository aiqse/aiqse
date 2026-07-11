# AIQSE × React

How the nine dimensions translate to a React component or feature.

| Dimension | In React terms |
|---|---|
| Input quality | Props contract, form validation, controlled-input edge cases (IME, paste, rapid typing) |
| Business quality | Derived state correctness, memoized computation invariants |
| State quality | Reducer transitions, stale-closure hazards, cache consistency (React Query/SWR) |
| Security | XSS via `dangerouslySetInnerHTML`, sanitizing user content, no secrets in client bundles |
| Concurrency | Race between rapid interactions and async responses, request cancellation, double-submit |
| Performance | Re-render budgets, bundle-size budget, interaction latency (INP) |
| Side effects | `useEffect` firing rules, analytics events, cleanup on unmount |
| Error handling | Error boundaries, failed-fetch UI states, retry affordances |
| Observability | Error reporting (Sentry etc.), web-vitals metrics, user-event logging |

**Evidence sources:** Testing Library (unit/integration), Playwright (E2E), Storybook + visual regression (golden master for UI), Lighthouse CI (performance budgets).

## Example: Quality Model for a search box

```yaml
feature: product-search-box
quality_model:
  input_quality:
    - "Query trimmed; empty query clears results, fires no request"
    - "Unicode / IME composition input handled without firing partial queries"
  state_quality:
    - "Results always correspond to the latest submitted query"
  concurrency:
    - "Out-of-order responses discarded (stale request cancellation)"
  performance:
    - "Debounce 300ms; no more than 1 request per pause"
  error_handling:
    - "Fetch failure → inline retry UI, previous results preserved"
  observability:
    - "search.performed event: query_length, result_count, latency_ms"
  security: { status: not-applicable, reason: "read-only public data, output rendered as text" }
  business_quality: { status: not-applicable, reason: "ranking owned by backend spec" }
  side_effects: { status: not-applicable, reason: "no external effects beyond the search call" }
```

*A full worked example (spec → quality model → expected results → generated component → evidence) is planned. Contributions welcome.*
