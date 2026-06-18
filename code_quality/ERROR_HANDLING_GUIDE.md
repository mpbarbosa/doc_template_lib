# Error Handling Guide

Error handling is the discipline of declaring, classifying, and propagating
failures as explicitly as successes. Errors are part of a function's contract,
not an afterthought discovered during implementation.

## Goal

Ensure every public boundary declares what it can fail with, every failure is
classified by type and recoverability, and every error propagation decision is
explicit — so that error behavior is as predictable and reviewable as success
behavior.

## What Consistent Error Handling Means

Consistent error handling means failures are first-class citizens of the
interface contract, typed precisely, propagated by explicit rules, and never
silently discarded.

In practice, it means:

1. Every public function declares its error cases as part of its contract, not
   as implementation notes.
2. Errors are classified — domain, validation, infrastructure, programming —
   and each class has explicit propagation rules.
3. Infrastructure errors never surface through domain boundaries as-is. They
   are wrapped with domain context at the adapter layer.
4. Every `catch` block makes an explicit decision: recover and continue, wrap
   and re-throw, or produce an observable output. Silent swallowing is
   prohibited.
5. Error types are domain-meaningful nouns, not generic `Error` instances with
   string messages.
6. Recovery strategy is decided at the catch site, not deferred to an
   upstream caller that has less context.

It does **not** mean catching every exception or wrapping every function in
error handling. Programming errors — assertion violations, null dereferences,
logic bugs — should not be caught and recovered from. They should fail fast and
surface immediately.

## Why It Matters

1. AI tools generate code that handles the happy path and skips or
   under-specifies error cases. Without explicit error contracts, generated
   code leaves failures as implicit, untested behavior.
2. Silent exception swallowing hides failures until they produce observable
   symptoms far from the point of origin, making root-cause diagnosis
   expensive.
3. Generic, untyped errors (`throw new Error('something went wrong')`) force
   callers to parse string messages to determine what happened, making
   error-handling code brittle and untestable.
4. Infrastructure errors that surface through domain boundaries couple
   callers to implementation details — a database timeout should not be
   visible to the caller of `findOrderById`.
5. Inconsistent error handling across a codebase means every component must be
   read independently to understand its failure behavior, multiplying review
   cost.

## Error Handling and Code LLMs

Error handling is one of the most consistent quality gaps in AI-generated code.

LLMs generate optimistic implementations: the body handles the expected case
and either omits error handling entirely or adds a generic catch block that
swallows or re-throws without context. Without an explicit error contract in
the interface, there is no specification for the LLM to conform to.

Explicit error taxonomy and propagation rules give reviewers a checklist that
applies uniformly to any AI-generated function, regardless of what it does.

### Why LLMs Benefit

- Declared error cases in the interface give the LLM a specification to
  implement against, not a gap to fill with generic handling.
- A typed error taxonomy makes generated error-handling code testable — each
  error type can be asserted in a test.
- Explicit propagation rules reduce the number of valid choices an LLM must
  make at each catch site, improving consistency.
- Uniform error shapes allow generated client-side error handling to be
  generic rather than endpoint-specific.

### Where Inconsistent Error Handling Hurts LLMs

- A catch block that swallows an exception gives the LLM no signal that error
  handling is expected at this site.
- Generic `Error` throws produce generated error-handling code that branches on
  string messages — brittle and hard to test.
- Infrastructure errors surfacing through domain boundaries cause the LLM to
  generate callers that depend on implementation details.
- Inconsistent patterns across the codebase prevent the LLM from generalizing
  the correct approach from existing examples.

## Required Rules

1. Every public function must declare its error cases. Error cases are part of
   the interface contract, not implementation notes.
2. Every error type must be a domain-meaningful named type. Generic `Error`
   instances with string discrimination are not acceptable for errors that
   callers must handle.
3. Infrastructure errors must be caught at the adapter boundary and wrapped
   with domain context before propagating.
4. Every `catch` block must make an explicit propagation decision: recover and
   continue, wrap and re-throw with context, or produce an observable output
   (structured log entry, metric, domain event). Silent swallowing is
   prohibited.
5. Programming errors — assertion violations, type violations, logic bugs —
   must not be caught and swallowed. They must fail fast.
6. The recovery strategy must be decided at the catch site. Do not propagate a
   raw exception to a caller that has less context about the appropriate
   recovery.

## Error Taxonomy

| Class | Definition | Propagation rule | Examples |
| --- | --- | --- | --- |
| Domain error | Expected failure within the problem domain | Declare in interface; caller decides recovery | `OrderNotFoundError`, `PaymentDeclinedError` |
| Validation error | Input violates the declared contract | Reject at the boundary; do not propagate inward | `InvalidEmailError`, `MissingFieldError` |
| Infrastructure error | External system failure | Catch at adapter boundary; wrap with domain context | `DatabaseTimeoutError`, `NetworkUnavailableError` |
| Programming error | Violated invariant; bug in the code | Fail fast; never catch to recover | Null dereference, assertion failure, type violation |

## Best Practices

### Declaring Error Cases

1. List every error case the caller must handle in the interface contract.
   If you are uncertain, ask: what can prevent this operation from succeeding?
2. Name error types after the domain event they represent:
   `OrderNotFoundError`, not `NotFoundError` or `Error404`.
3. Include enough context in the error for diagnosis without a debugger:
   the entity identifier, the operation attempted, the constraint violated.
4. Distinguish errors the caller can act on (`OrderNotFoundError` — try a
   different ID) from errors the caller can only report
   (`PaymentGatewayUnavailableError` — retry or surface to the user).

### Propagation Decisions

1. At every catch site, choose one of three outcomes:
   - **Recover**: handle the error locally and continue normally.
   - **Wrap and re-throw**: add context and propagate to the caller.
   - **Observe and re-throw**: log a structured entry and propagate unchanged.
2. Never catch an exception purely to re-throw it unchanged with no log or
   context added. That catch block does nothing useful.
3. Wrap infrastructure errors at the adapter boundary — the outermost layer
   that owns the infrastructure dependency. Never let a database error,
   network error, or SDK exception propagate through a domain interface
   unchanged.

### Error Types and Structure

1. Create a named error type for every distinct failure the caller must
   handle differently.
2. Include structured fields, not string concatenation:
   `new OrderNotFoundError({ orderId })` not
   `new Error('Order ' + id + ' not found')`.
3. Organize error types by domain concept, not by technical cause.
   `PaymentDeclinedError` is better than `ExternalServiceError` even if both
   originate from the payment gateway.

### Recovery vs. Fail-Fast

1. Recover only from errors the current site has enough context to resolve
   correctly. If recovery requires knowledge the current layer does not have,
   propagate.
2. Programming errors — null dereferences, assertion violations, type
   mismatches — must not be caught to recover. They signal a bug; surfacing
   them immediately is the correct behavior.
3. Use assertions for invariants that must always hold. Let them throw.
   Do not surround invariant checks with `try-catch`.

## Review Heuristics

### Contract Completeness Test

Does the interface declare every error case the caller must handle? A contract
that describes only the success path is incomplete. Review each public function:
what can fail? Is each failure named, typed, and declared?

### Swallow Test

Does every `catch` block produce at least one of: a re-throw, a wrapped
re-throw, or an observable output (structured log, metric, event)? Any `catch`
block that does none of these is a silent swallow — a prohibited pattern.

### Leakage Test

Do any infrastructure error types (`SQLException`, `HTTPError`, SDK exceptions)
appear in domain layer throw statements or interface declarations? If yes, an
adapter boundary is missing or porous.

### Context Test

Does the error contain enough information to diagnose the failure without
attaching a debugger or reading the call stack? At minimum: what was attempted,
on what entity, and what constraint was violated.

### Taxonomy Test

Is each caught or thrown error classified by type? Would a reviewer know —
from the error type name alone — whether this is a domain error, a validation
error, an infrastructure error, or a programming error?

## Positive Signals

- Error types are domain-meaningful named types; no generic `new Error('...')`
  for errors callers must handle.
- Every public interface declaration includes its error cases alongside its
  return type.
- Infrastructure errors are caught at adapter boundaries and re-thrown as
  domain errors with context.
- Every `catch` block has an explicit, visible decision: recover, wrap, or
  observe.
- Each error type includes structured fields that support diagnosis without
  string parsing.
- Test suites include at least one test per declared error case.

## Warning Signs

- `catch (e) { }` — silent swallow with no recovery, no log, no re-throw.
- `catch (e) { throw e }` — re-throw with no added context; the catch block
  contributes nothing.
- `throw new Error('something went wrong')` — untyped, unstructured, not
  testable by type.
- Database exceptions, HTTP errors, or SDK exceptions surfacing through domain
  interfaces unchanged.
- Error handling added only on the happy path; failure cases left without
  catch blocks.
- String parsing to determine error type: `if (e.message.includes('not found'))`.
- A single `catch (e)` at the top of a function that handles all failure
  modes identically regardless of cause.
- Error cases discovered during implementation and added to the interface
  retroactively, without updating tests.

## Related Guides

- [INTERFACE_FIRST_GUIDE.md](./INTERFACE_FIRST_GUIDE.md) for declaring error
  cases as part of the interface contract before writing any implementation.
- [DEFENSIVE_CODING_GUIDE.md](./DEFENSIVE_CODING_GUIDE.md) for enforcing
  input contracts at boundaries, which determines what validation errors are
  raised and where.
- [OBSERVABILITY_GUIDE.md](./OBSERVABILITY_GUIDE.md) for structured logging of
  error context at every catch site that does not recover.
- [CLEAN_ARCHITECTURE_GUIDE.md](./CLEAN_ARCHITECTURE_GUIDE.md) for placing
  infrastructure error wrapping at adapter boundaries, keeping domain
  interfaces clean.
- [REST_API_GUIDE.md](../domain_specific/REST_API_GUIDE.md) for translating
  classified error types into consistent HTTP error response shapes.
- [UNIT_TEST_GUIDE.md](./UNIT_TEST_GUIDE.md) for testing every declared error
  case, not just the success path.

## Summary Checklist

- [ ] Every public function declares its error cases as part of its interface
      contract.
- [ ] Every error type is a domain-meaningful named type with structured
      fields.
- [ ] Each error is classified: domain, validation, infrastructure, or
      programming.
- [ ] Infrastructure errors are caught at adapter boundaries and wrapped before
      propagating.
- [ ] Every `catch` block makes an explicit decision: recover, wrap, or
      observe. No silent swallows.
- [ ] Programming errors are not caught to recover — they fail fast.
- [ ] Every declared error case has a corresponding test.
- [ ] No error type discrimination is done by parsing string messages.
