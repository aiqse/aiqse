# AIQSE × Next.js

Next.js adds server/client duality, rendering modes, and caching — which mostly means **more state-quality and side-effect surface**.

| Dimension | In Next.js terms |
|---|---|
| Input quality | Route params / searchParams validation, Server Action input schemas (zod etc.) |
| Business quality | Same rules enforced server-side regardless of client checks |
| State quality | Cache correctness: `revalidatePath`/`revalidateTag` after every mutation; ISR staleness bounds |
| Security | Server Actions are public endpoints — authz inside each action; secrets stay server-side; middleware auth coverage |
| Concurrency | Parallel route fetches, Server Action double-invocation, optimistic-update reconciliation |
| Performance | TTFB/LCP budgets per rendering mode, streaming boundaries, image optimization |
| Side effects | Mutations in Server Actions only; effects never fire during render/prerender |
| Error handling | `error.tsx` / `not-found.tsx` coverage, Server Action error contracts to the client |
| Observability | Server-side logging with request correlation, RUM web-vitals |

**Evidence sources:** Vitest + Testing Library, Playwright against a production build (`next build && next start` — dev mode hides caching behavior), Lighthouse CI, log assertions on Server Action paths.

## The two questions every Next.js Quality Model must answer

1. **For every piece of data: when is it allowed to be stale, and by how much?** (state_quality — this is the cache model, decided by a human, not defaulted by the framework)
2. **For every Server Action: who may call it, and what happens if it's called twice?** (security + concurrency)

*A full worked example is planned. Contributions welcome.*
