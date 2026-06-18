# Observability Guide

Observability is the discipline of making a system's runtime behavior visible
through structured, queryable signals — without requiring code changes or
debugger attachment to understand what is happening in production.

## Goal

Ensure that every significant state transition, decision, failure, and
performance characteristic of a system component is captured in structured
signals that can be queried, correlated, and acted on in production without
access to the source code or a running debugger.

## What Observability Means

Observability means the system emits enough structured signal that any
production question — "what happened?", "how often?", "how long?", "why did
this fail?" — can be answered from the signals alone.

In practice, it means:

1. Structured logs record what happened — state transitions, decisions, errors
   — with enough field context to diagnose without the surrounding code.
2. Metrics capture how much, how many, and how long — counters, gauges, and
   histograms at operation boundaries.
3. Distributed traces record how a request flowed through the system and where
   time was spent.
4. Every error that is caught and not re-thrown produces a structured log entry
   with full context.
5. A correlation ID is present on every log entry and trace span for a given
   request, allowing all signals for one request to be joined.
6. No sensitive data — passwords, tokens, PII — appears in any signal.

It does **not** mean logging everything. High-volume noise makes logs
unsearchable and drives up observability costs. Observability is about signal
quality, not signal volume. A small number of well-structured, high-value
entries is more useful than a large volume of unstructured noise.

## Why It Matters

1. AI tools generate code that either has no logging or uses unstructured
   `console.log` strings. Neither is queryable in production. Without an
   observability convention, AI-generated code is production-blind.
2. Unstructured log strings cannot be queried, aggregated, or correlated.
   A log entry that says `"order failed"` is not diagnosable; one that
   includes `orderId`, `customerId`, `failureReason`, and `durationMs` is.
3. Missing error observability turns silent swallows into invisible failures
   — systems that appear healthy until a symptom surfaces far from the origin.
4. Without a correlation ID, tracing a single request across logs, metrics,
   and multiple services requires manual reconstruction from timestamps — a
   slow and error-prone process.
5. Metrics enable proactive monitoring: a rising error rate or p99 latency
   spike is visible before it reaches users, if the metrics exist.

## Observability and Code LLMs

Observability is among the most consistent gaps in AI-generated code.

LLMs generate implementations that satisfy the functional requirement. They
rarely add structured logging, metrics emission, or trace span creation unless
explicitly specified. When they do add logging, it is typically unstructured
string output — `console.log('Processing order: ' + orderId)` — which is not
queryable in production.

Explicit observability conventions — what to emit, where, and in what structure
— give the LLM a specification it can follow when generating any operation.

### Why LLMs Benefit

- A structured log field template gives the LLM a concrete format to produce
  at each log site rather than improvising string messages.
- Explicit rules for where to emit logs (on state transitions, on errors, on
  significant decisions) tell the LLM which sites require instrumentation
  without requiring judgment about what "significant" means.
- Correlation ID propagation rules prevent the LLM from generating functions
  that lose request context mid-chain.
- A metric naming convention lets the LLM generate correct metric names for
  new operations by pattern-matching existing ones.

### Where Missing Observability Hurts LLMs

- Without a logging convention, the LLM omits logging entirely or uses
  unstructured string output that cannot be queried.
- Without error observability rules, the LLM generates catch blocks that
  swallow errors silently or re-throw without producing any signal.
- Without a correlation ID convention, the LLM generates functions that
  lose request context, breaking the ability to trace a request end to end.

## Required Rules

1. Every structured log entry must include at minimum: a timestamp, a severity
   level, a message, and the correlation ID for the current request.
2. Log entries for domain events must include the entity identifier and
   the state transition that occurred.
3. Every `catch` block that does not re-throw must produce a structured log
   entry at `ERROR` or `WARN` level with the exception details and the
   operation context.
4. No sensitive data — passwords, secrets, tokens, PII — may appear in any
   log entry, metric label, or trace attribute.
5. Every operation that can fail must emit a metric increment on both success
   and failure paths.
6. A correlation ID must be propagated through every function call in a request
   chain and included in every log entry and trace span for that request.
7. Log entries must use structured fields, not string concatenation. Field
   names must be consistent across the codebase.

## Signal Reference

| Signal | What it answers | Granularity | Examples |
| --- | --- | --- | --- |
| Log | What happened, and why | Per event | Order created, payment failed, cache miss |
| Metric | How much, how many, how long | Aggregated over time | Request count, error rate, p99 latency |
| Trace | How a request flowed, where time was spent | Per request, per span | HTTP handler → service → repository → DB |

Use all three. Logs answer "what happened for this request?"; metrics answer
"is the system healthy across all requests?"; traces answer "why did this
request take 800ms?".

## Best Practices

### Structured Logging

1. Use a structured logger that emits machine-readable entries (JSON or
   equivalent). Never use string concatenation to build log messages.
2. Define a standard set of fields that every log entry must include:
   `timestamp`, `level`, `message`, `correlationId`, `service`. Add
   domain-specific fields per entry type.
3. Name fields consistently across the codebase. `orderId` in one place and
   `order_id` in another breaks log aggregation queries.
4. Keep `message` short and human-readable. Put queryable detail in fields,
   not in the message string.

### Log Levels

Use the narrowest accurate level:

| Level | When to use |
| --- | --- |
| `ERROR` | Unexpected failure requiring investigation or intervention |
| `WARN` | Expected failure or degraded operation: retry, fallback, circuit open |
| `INFO` | Significant domain event: order created, payment processed, user authenticated |
| `DEBUG` | Diagnostic detail useful during development; must be disabled in production by default |

Never use `ERROR` for expected failures the system handles automatically.
Never use `INFO` for high-frequency events that would produce noise at scale.

### What to Log

1. **State transitions**: every time an entity changes state, log the entity
   identifier, the previous state, the new state, and the trigger.
2. **Significant decisions**: when the system chooses a code path based on
   runtime data (cache hit vs. miss, retry vs. abort, feature flag value),
   log the decision and the data that drove it.
3. **All caught errors that are not re-thrown**: log at `ERROR` if unexpected,
   `WARN` if expected. Include the exception type, message, entity identifier,
   and operation name.
4. **External calls**: log the start and outcome of every call to an external
   dependency — database, API, queue — including duration and success/failure.

### What Not to Log

1. Passwords, authentication tokens, API keys, session identifiers.
2. PII — names, email addresses, phone numbers, payment card details — unless
   explicitly required and access-controlled.
3. High-frequency events that would produce thousands of entries per second
   in normal operation (inner loop iterations, cache reads for every field).
4. Implementation details with no diagnostic value in production.

### Metrics

1. Emit a counter on every operation entry point, split by outcome:
   `orders.processed.success`, `orders.processed.failure`.
2. Emit a histogram for every operation with variable latency: request
   duration, external call duration, queue processing time.
3. Use a consistent naming convention: `<domain>.<operation>.<outcome>` or
   `<service>.<resource>.<metric_type>`.
4. Include dimensions (labels/tags) that support filtering: `endpoint`,
   `status_code`, `error_type`. Do not use high-cardinality values (entity IDs,
   user IDs) as metric dimensions.

### Correlation IDs

1. Generate a correlation ID at the outermost entry point of every request
   (HTTP handler, message consumer, scheduled job entry).
2. Propagate the correlation ID explicitly through every function call in the
   request chain — as a parameter, a context object, or a request-scoped
   store.
3. Include the correlation ID in every log entry, trace span, and outbound
   request header for the duration of the request.
4. Never generate a new correlation ID mid-request. The ID is fixed for the
   lifetime of the request.

## Review Heuristics

### Error Visibility Test

Does every `catch` block that does not re-throw produce a log entry? Silently
swallowed exceptions are invisible failures. Review each catch site: if no
log entry is produced, the failure is unobservable.

### Structured Field Test

Do log entries use structured fields or string concatenation? Query a sample
of log output: can the `orderId`, `userId`, and `correlationId` be extracted
as first-class fields, or are they embedded in a string that requires parsing?

### Correlation Coverage Test

Can every log entry for a single request be joined by correlation ID? If a
function in the request chain does not receive or propagate the correlation ID,
that segment of the request is invisible.

### Sensitive Data Test

Does any log entry, metric label, or trace attribute contain a password, token,
API key, or PII field? Review all log sites at system boundaries where external
input enters the system — these are the highest-risk locations.

### Metric Coverage Test

Does every operation that can fail emit a metric on both success and failure
paths? A metric that only increments on success makes failure rates invisible
to dashboards and alerts.

## Positive Signals

- Log entries are structured JSON with consistent field names across the
  codebase.
- Every catch block that does not re-throw produces a structured log entry.
- Correlation IDs appear in every log entry for a given request and can be
  used to join all entries for that request.
- Metrics exist for both success and failure paths of every critical operation.
- No sensitive data appears in any log entry, metric dimension, or trace
  attribute.
- A new operation generated by an LLM includes log entries at state
  transitions and error catch sites by following existing patterns.

## Warning Signs

- `console.log('Processing order: ' + orderId)` — unstructured string output.
- A catch block with no log entry: `catch (e) { return null }`.
- Log entries that lack a correlation ID — impossible to trace to a request.
- Metric names that are inconsistent across similar operations.
- PII or tokens appearing in log output at system boundaries.
- DEBUG-level entries that would produce thousands of lines per second in
  production.
- A new service or module with no log entries and no metrics, generated or
  written without an observability review.
- Log entries where diagnostic detail is embedded in the `message` string
  rather than in queryable fields.

## Related Guides

- [ERROR_HANDLING_GUIDE.md](./ERROR_HANDLING_GUIDE.md) for the rule that every
  caught exception that is not re-thrown must produce a structured log entry —
  observability is the required output of a non-recovering catch block.
- [CLEAN_ARCHITECTURE_GUIDE.md](./CLEAN_ARCHITECTURE_GUIDE.md) for placing
  observability instrumentation in the infrastructure and adapter layers,
  keeping domain logic free of logging dependencies.
- [INTERFACE_FIRST_GUIDE.md](./INTERFACE_FIRST_GUIDE.md) for including
  observability requirements — which events to emit, which fields to include —
  as part of the operation contract before implementation begins.
- [INTEGRATION_TEST_GUIDE.md](./INTEGRATION_TEST_GUIDE.md) for verifying that
  observability signals are emitted correctly under real boundary conditions,
  not just under unit-test mocks.
- [DEFENSIVE_CODING_GUIDE.md](./DEFENSIVE_CODING_GUIDE.md) for the relationship
  between boundary validation and log coverage — every rejection at a system
  boundary should produce a structured log entry.

## Summary Checklist

- [ ] Every log entry includes timestamp, level, message, correlationId, and
      relevant entity identifiers as structured fields.
- [ ] Field names are consistent across all log entries in the codebase.
- [ ] Every `catch` block that does not re-throw produces a structured log
      entry at `ERROR` or `WARN`.
- [ ] No sensitive data appears in any log entry, metric label, or trace
      attribute.
- [ ] Metrics exist for both success and failure paths of every critical
      operation.
- [ ] Correlation IDs are generated at request entry points and propagated
      through the entire request chain.
- [ ] Log level discipline is enforced: `ERROR` for unexpected failures,
      `WARN` for expected degraded operation, `INFO` for significant domain
      events, `DEBUG` disabled in production.
- [ ] External dependency calls log start, outcome, and duration.
