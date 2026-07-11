# API Quality Template

A specialization of the Quality Model for HTTP/RPC endpoints. Copy per endpoint or endpoint group.

```yaml
endpoint: POST /v1/<resource>
spec: <link>

api_quality:

  contract:
    request_schema: <link or inline>       # types, required fields, constraints
    response_schema: <link or inline>      # success and error shapes
    status_codes:                          # every code this endpoint may return, and when
      200: ""
      422: ""
      403: ""
      409: ""
      500: ""

  input_quality:
    - "Unknown fields: <rejected | ignored>"
    - "Max payload size: <n> KB → 413 beyond"

  security:
    - "Auth: <scheme>; unauthenticated → 401"
    - "Authorization rule: <who may call>; violation → 403"
    - "Rate limit: <n>/min per <key> → 429 with Retry-After"

  concurrency:
    - "Idempotency: <Idempotency-Key header | naturally idempotent | not idempotent (why acceptable)>"

  performance:
    - "p99 < <n> ms at <n> rps"
    - "Max DB queries per request: <n>"

  error_handling:
    - "Error body shape: { code, message, field_errors[] } — consistent across all errors"
    - "Upstream timeout after <n> ms → <behavior>"
    - "Error messages never include: stack traces, SQL, internal hostnames"

  compatibility:
    - "Versioning scheme: <path | header>"
    - "Breaking-change policy: <rule>"

  observability:
    - "Access log: method, path, status, latency_ms, caller_id"
    - "Trace header <name> propagated to all downstream calls"
```
