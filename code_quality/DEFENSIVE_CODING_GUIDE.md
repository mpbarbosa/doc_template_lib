# Defensive Coding Guide

Defensive coding is the practice of validating all external inputs at system
boundaries and asserting all invariants at domain boundaries, so that inner
layers can trust their inputs and focus entirely on business logic.

## Goal

Ensure that invalid data is rejected at the point of entry and invalid internal
state is impossible to construct, so that failures surface where they originate
rather than propagating silently into business logic where they cause misleading
errors.

## What Defensive Coding Means

Defensive coding means that trust is established at boundaries and honored
internally — not re-negotiated at every function call throughout the system.

In practice, it means:

1. All external inputs are validated at system boundaries before entering any
   business logic.
2. Validated data is trusted internally. Inner layers do not re-validate what
   a boundary already accepted.
3. Domain invariants are enforced with assertions at construction time.
   A domain object that can be created in an invalid state is a design flaw.
4. Null and undefined values are resolved at the boundary where they originate.
   They do not propagate into inner layers.
5. Functions declare their preconditions explicitly. The body executes only
   when the preconditions are met.
6. Programming errors — violated invariants, impossible states — fail
   immediately and are never caught to recover from.

It does **not** mean placing defensive checks throughout every layer of the
system. Validation logic scattered across controllers, services, and repositories
creates duplication, inconsistency, and the false impression that each layer is
safe to call with arbitrary input. One thorough validation pass at the boundary
is correct; repeated partial checks throughout the system are not.

## Why It Matters

1. AI tools generate optimistic implementations. Without explicit boundary
   validation, AI-generated code assumes inputs are valid and skips the checks
   that prevent invalid data from corrupting internal state.
2. Validation errors discovered deep in business logic are hard to diagnose.
   An invalid input that passes a boundary and fails three layers later
   produces an error message with no relationship to the original cause.
3. Domain objects that can be constructed in an invalid state require every
   consumer to defensively check validity — a cost that compounds across the
   codebase.
4. Null propagation turns a missing value at the boundary into a null
   dereference anywhere downstream. Resolving null at the source eliminates
   an entire class of runtime errors.
5. Consistent boundary validation gives AI-generated code a clear structural
   model: validate here, trust everywhere else.

## Defensive Coding and Code LLMs

Defensive coding failures are among the most common quality issues in
AI-generated code.

LLMs generate happy-path implementations by default. Without an explicit
convention for where validation belongs, AI-generated code places checks
inconsistently — sometimes in controllers, sometimes in services, sometimes
not at all. The result is a system where no layer is reliably safe to call
and the same validation logic is partially duplicated in multiple places.

Explicit boundary rules give the LLM a structural constraint: all validation
belongs at the boundary, and inner layers can be generated without defensive
checks.

### Why LLMs Benefit

- A clear boundary taxonomy tells the LLM exactly where to generate validation
  code and where to omit it.
- The "parse, don't validate" pattern gives the LLM a reusable model: convert
  raw input to a typed domain value at the boundary; pass the typed value
  everywhere else.
- Assertion-enforced invariants in constructors prevent the LLM from
  generating code that creates objects in invalid states.
- Explicit preconditions at function entry prevent the LLM from generating
  guard clauses scattered through the body.

### Where Missing Boundaries Hurt LLMs

- Without boundary validation, the LLM adds `if (!value)` checks throughout
  inner layers, producing duplicated and inconsistent defensive logic.
- Without invariant enforcement in constructors, the LLM generates domain
  objects that can be constructed in invalid states, requiring callers to
  handle validity themselves.
- Without null resolution at the source, the LLM propagates null checks
  throughout the codebase — a symptom, not a fix.

## Required Rules

1. All inputs from external sources — user input, external APIs, message
   queues, file I/O, database reads — must be validated at the system boundary
   before entering any business logic.
2. Validated inputs must not be re-validated in inner layers. Inner layers
   receive valid data by contract.
3. Domain objects must enforce their invariants in their constructors or
   factory methods. A domain object must be valid at construction or must not
   be constructed at all.
4. Null and undefined values must be resolved at the boundary where they
   originate. They must not propagate past the boundary into inner layers.
5. Functions must declare preconditions for inputs they cannot handle.
   Preconditions are enforced with assertions, not ignored.
6. Programming errors — invariant violations, impossible states, assertion
   failures — must fail fast and must not be caught to recover from.

## Boundary Reference

| Boundary type | What crosses it | Validation responsibility | Trust level inside |
| --- | --- | --- | --- |
| System boundary | User input, external API responses, message queue payloads, file contents | Full input validation — types, formats, ranges, required fields | Trusted — no re-validation |
| Domain boundary | Service method calls, entity constructors, value object factories | Invariant assertions — enforce domain rules at construction | Trusted — domain rules hold |
| Internal (same layer) | Component-to-component calls within a validated flow | None — preconditions are documented, not re-checked | Trusted by design |

## Best Practices

### Parse, Don't Validate

Transform raw input into a typed domain value at the boundary. Pass the typed
value — not the raw input — into all inner layers.

```
// At the boundary: parse raw input into a domain type
const orderId = OrderId.parse(rawInput.orderId);  // throws ValidationError if invalid

// In inner layers: receive the domain type, no validation needed
function findOrder(orderId: OrderId): Promise<Order> { ... }
```

This eliminates the need for inner layers to know what "valid" means. The type
carries the guarantee.

### Guard Clauses

Declare preconditions at the top of a function. The function body executes
only in the valid state.

```
function applyDiscount(order, discountRate) {
  assert(order !== null, 'order must not be null');
  assert(discountRate >= 0 && discountRate <= 1, 'discountRate must be between 0 and 1');

  // body runs only when preconditions hold
}
```

Prefer guard clauses that return early or throw immediately over conditions
that wrap the entire body.

### Invariant Assertions

Use assertions for conditions that must always be true — conditions that, if
violated, indicate a programming error rather than a user error.

1. Assert in constructors and factory methods that the resulting object is
   valid.
2. Assert at the start of operations that depend on a specific internal state.
3. Never surround an assertion with a `try-catch` to recover from its failure.
   An assertion failure is a bug; recovering silently hides the bug.

### Optional and Null Hygiene

1. Resolve optionals and nulls at the boundary — convert them to explicit
   domain values (`None`, `Optional.empty()`, a default value, or a thrown
   validation error).
2. Do not pass null into a function that does not accept null. Declare
   parameter nullability explicitly.
3. Avoid returning null from functions that return a domain value. Return an
   explicit empty result (`[]`, `Optional.empty()`, a `NotFound` domain error)
   instead.

### Validation Error vs. Assertion Error

| Scenario | Mechanism | Class |
| --- | --- | --- |
| User-provided value is malformed | Validation at the system boundary | `ValidationError` (expected, declared in interface) |
| Domain invariant is violated | Assertion in constructor or factory | Programming error (fail fast, never catch) |
| Precondition violated by a caller | Assertion at function entry | Programming error (fail fast, never catch) |
| Optional value from external source is absent | Null resolution at boundary | `ValidationError` or explicit default |

## Review Heuristics

### Boundary Consistency Test

Is validation performed at every system entry point — every HTTP handler, every
message consumer, every file reader, every external API response processor? A
validation gap at any one entry point allows invalid data to enter the system.

### Re-validation Test

Does validation logic appear in inner layers — services, domain classes,
repositories? If yes, the boundary validation is incomplete or the inner layer
does not trust the boundary's guarantee. Fix the boundary; remove the inner
check.

### Assertion vs. Condition Test

Does each check distinguish between a user error (validation, expected failure)
and a programming error (assertion, impossible state)? A `if (!value) return`
inside domain logic is masking a design problem; an `assert(value !== null)` is
declaring a contract.

### Null Propagation Test

Can a null or undefined value silently reach an inner layer function? If yes,
null is not being resolved at the source. Trace the null to its origin and
resolve it there.

### Invariant Test

Can any domain object be constructed in an invalid state? If yes, domain
invariants are not being enforced at construction time and every consumer must
defensively check validity.

## Positive Signals

- Validation logic is concentrated at system boundaries — not scattered across
  services, repositories, or domain classes.
- Domain constructors and factory methods enforce invariants; no domain object
  can be created in an invalid state.
- Inner layer functions accept typed domain values; no raw strings, unvalidated
  IDs, or nullable primitives appear as inner layer parameters.
- Null checks do not appear inside business logic; optional handling is at the
  boundary.
- Assertions in constructors and operation entry points describe preconditions
  in code, not in comments.
- AI-generated inner layer code contains no validation logic because the
  boundary contract makes it unnecessary.

## Warning Signs

- Null checks scattered throughout business logic:
  `if (order !== null && order.items !== null)` inside a domain service.
- Validation logic duplicated in both a controller and the service it calls.
- Silent coercion of invalid input: converting `"abc"` to `0`, trimming
  instead of rejecting oversized values.
- Domain objects constructed without invariant checks in the constructor.
- `try-catch` blocks surrounding assertions to prevent them from failing.
- An inner layer that must re-read the raw input because it does not trust
  the value it received.
- A function body that begins with a chain of defensive null checks on
  parameters it received from an internal caller.

## Related Guides

- [INTERFACE_FIRST_GUIDE.md](./INTERFACE_FIRST_GUIDE.md) for declaring
  preconditions and validation contracts before writing any implementation.
- [ERROR_HANDLING_GUIDE.md](./ERROR_HANDLING_GUIDE.md) for classifying
  validation errors (user errors, recoverable) versus assertion errors
  (programming errors, fail fast).
- [CLEAN_ARCHITECTURE_GUIDE.md](./CLEAN_ARCHITECTURE_GUIDE.md) for placing
  validation at system boundaries (outer layers) and keeping domain logic
  (inner layers) free of infrastructure assumptions.
- [LOW_COUPLING_GUIDE.md](./LOW_COUPLING_GUIDE.md) for the "parse, don't
  validate" pattern — typed domain values reduce the coupling between layers
  caused by repeated validation of raw inputs.
- [UNIT_TEST_GUIDE.md](./UNIT_TEST_GUIDE.md) for testing boundary validation
  explicitly — every invalid input class must have a corresponding rejection
  test.

## Summary Checklist

- [ ] All external inputs are validated at system boundaries before entering
      business logic.
- [ ] Validation logic does not appear in inner layers (services, domain,
      repositories).
- [ ] Domain objects enforce their invariants in constructors or factory
      methods; invalid construction is impossible.
- [ ] Null and undefined values are resolved at the boundary; they do not
      propagate into inner layers.
- [ ] Every function precondition is enforced with an assertion, not a
      silent fallback.
- [ ] Programming errors (assertion failures) are not caught to recover from.
- [ ] Validation errors and assertion errors are distinguished by type and
      handling.
- [ ] Every invalid input class has a corresponding rejection test.
