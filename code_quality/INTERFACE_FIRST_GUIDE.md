# Interface-First Guide

Interface-first design is the practice of declaring the complete contract of a
unit of behavior — its inputs, outputs, and failure modes — before writing any
implementation.

## Goal

Ensure every public function, class, and module has an explicit contract that
specifies what it accepts, what it guarantees, and what it can fail with — so
that implementations can be verified against a specification rather than
inferred from behavior.

## What Interface-First Design Means

Interface-first design means the interface is written first and serves as the
specification. The implementation is what fulfills it, not what defines it.

In practice, it means:

1. A function's name, parameter types, return type, and error cases are
   declared before any implementation line is written.
2. Error cases are part of the contract, not discovered during implementation.
3. Interfaces are defined in the layer that uses them, not the layer that
   implements them.
4. No implementation detail — infrastructure type, framework class, internal
   data shape — appears in a public interface.
5. The interface is the review target. An implementation review verifies
   conformance to the contract, not the other way around.
6. Tests are written against the interface. They remain valid when the
   implementation changes.

It does **not** mean writing formal interface declarations for every private
function or internal helper. The principle applies to public boundaries: the
surface a caller, a test, or an AI tool will interact with.

## Why It Matters

1. AI tools generate implementations that satisfy the happy path. Without a
   declared contract, there is no specification against which to verify that
   the generated code handles edge cases, errors, and boundary conditions
   correctly.
2. Interfaces are stable; implementations change. Designing the interface first
   separates the stable contract from the volatile implementation, reducing the
   cost of future changes.
3. Tests written against an interface survive implementation rewrites. Tests
   written against an implementation become obstacles when the implementation
   changes.
4. Contract-first design surfaces design problems before they are encoded in
   implementation. A hard-to-express interface is a signal that the abstraction
   is wrong.
5. Multiple implementations — production, test double, stub, alternative
   provider — become possible only when the interface is defined
   independently of any one implementation.

## Interface-First Design and Code LLMs

Interface-first design directly improves the quality and verifiability of
LLM-generated code.

An LLM given a precise interface — parameter types, return shape, declared
error cases — generates a conforming implementation. An LLM given nothing
generates a convenient implementation that may be wrong at boundaries, silent
on errors, or shaped around internal assumptions rather than the caller's
needs.

The interface also limits the review surface. A contract is shorter and more
scannable than an implementation. Reviewing AI-generated code against a
declared contract is faster and more reliable than reviewing it in isolation.

### Why LLMs Benefit

- A precise interface is a prompt in itself: it specifies exactly what the
  generated implementation must do.
- Error cases declared in the contract prevent AI from silently skipping them.
- Interfaces defined in inner layers prevent AI from leaking infrastructure
  types into domain logic.
- Contract-first code has a stable review target that does not change when the
  AI regenerates the implementation.

### Where Missing Contracts Hurt LLMs

- Without declared error cases, AI generates optimistic implementations that
  ignore failure modes.
- Without an interface boundary, AI couples callers to implementation details,
  making future regeneration disruptive.
- Without a specification, reviewing AI-generated code requires understanding
  the full implementation rather than checking conformance to a known contract.

## Required Rules

1. Every public function and class must have a declared interface before any
   implementation is written.
2. Error cases must be declared in the interface. They are part of the
   contract, not implementation notes.
3. Interfaces must be defined in the layer that uses them, not the layer that
   implements them.
4. No implementation detail — infrastructure type, ORM entity, HTTP response
   shape, framework class — may appear in a domain or use-case interface.
5. An implementation must fulfill the complete interface contract, including
   all declared error cases.
6. Callers must not need to read the implementation to use the interface
   correctly.

## Contract Elements Reference

| Element | What it declares | Example |
| --- | --- | --- |
| Name | What the operation does, in domain terms | `findOrderById` |
| Parameters | What the caller must provide and its shape | `orderId: OrderId` |
| Return type | What the caller receives on success | `Promise<Order>` |
| Preconditions | What must be true before calling | `orderId` must be a valid, non-empty identifier |
| Postconditions | What is guaranteed on success | Returned order belongs to the given `orderId` |
| Error cases | What can fail and what the caller receives | `OrderNotFoundError` if no order exists |

A contract is complete when a caller can use it correctly — including error
handling — without reading the implementation.

## Best Practices

### Writing the Contract

1. Start with the name. If naming is hard, the abstraction may be wrong.
2. Declare parameter types before deciding how they will be used internally.
   Internal use is an implementation concern.
3. Declare the return type as the domain concept the caller needs, not the
   shape your implementation produces.
4. List every error case the caller must handle. If you are unsure, ask: what
   can prevent this operation from succeeding?
5. Write the preconditions as assertions the implementation will enforce, not
   assumptions it will silently rely on.

### Contract Placement

1. Define interfaces in the inner layer that depends on them. A repository
   interface belongs to the use-case layer, not the database layer.
2. Keep interfaces free of import dependencies on infrastructure modules. If
   a parameter type requires importing a framework class, the interface is
   leaking implementation.
3. Group contracts by the domain concept they describe, not by the
   implementation that fulfills them.

### Contract and Tests

1. Write the test before the implementation. The test is the first consumer
   of the contract and will surface design problems before a line of
   implementation exists.
2. Test every declared error case. An untested error case is an undeclared
   one.
3. Test double implementations (mocks, stubs, fakes) should implement the
   same interface as the production implementation. If they do not, the test
   is not testing the contract.

## Review Heuristics

### Contract Completeness Test

Does the interface declare what happens on failure, not just on success? A
contract that only describes the happy path is incomplete. Every public function
that can fail must declare what the caller receives when it does.

### Leakage Test

Does the interface reference any infrastructure type — a database entity, an
HTTP client, a framework class, an ORM annotation? If yes, the contract couples
callers to implementation details and will break when the implementation
changes.

### Caller Sufficiency Test

Can a caller use this interface correctly — including error handling — without
reading the implementation? If a caller must read the implementation to know
what to expect or how to handle errors, the contract is incomplete.

### Stability Test

Would changing the implementation require changing the interface? If yes, the
interface exposes implementation decisions rather than domain contracts. The
interface should change only when the domain contract changes, not when the
implementation changes.

### Inversion Test

Is the interface defined in the layer that uses it, or the layer that
implements it? An interface defined in the infrastructure layer and imported
by the domain layer violates the dependency direction. Interfaces belong to
their consumers.

## Positive Signals

- Interfaces are shorter than their implementations and readable without
  surrounding context.
- Error cases are typed and named, not generic `Error` instances with string
  messages.
- Tests reference only the interface — no implementation import, no
  `instanceof`, no private method access.
- Multiple implementations can be swapped without changing any caller.
- The interface contains no import from an infrastructure module.
- Generating a new implementation from the interface alone produces a
  correct result.

## Warning Signs

- The interface and implementation are written at the same time, in the same
  file, without a prior declaration step.
- Error cases are discovered during implementation and added retroactively to
  the interface.
- Infrastructure types (`DatabaseRow`, `HTTPResponse`, `ORMEntity`) appear in
  domain interface parameters or return types.
- Callers read the implementation to understand what errors to handle.
- The test mocks the implementation class rather than implementing the
  interface.
- AI-generated code has no declared interface — only an implementation class
  or function.
- The interface changes every time the implementation changes.

## Related Guides

- [CLAUDE_CODE_WORKFLOW_GUIDE.md](./CLAUDE_CODE_WORKFLOW_GUIDE.md) — session
  discipline for the Claude Code session that declares and implements the
  interface; contract-first declaration is the first stage of any well-formed
  session.
- [CLEAN_ARCHITECTURE_GUIDE.md](./CLEAN_ARCHITECTURE_GUIDE.md) for the
  principle that interfaces belong to inner layers, not the outer layers that
  implement them.
- [LOW_COUPLING_GUIDE.md](./LOW_COUPLING_GUIDE.md) for keeping callers
  dependent on stable contracts rather than volatile implementations.
- [ERROR_HANDLING_GUIDE.md](./ERROR_HANDLING_GUIDE.md) for declaring and
  classifying the error cases that are part of every contract.
- [UNIT_TEST_GUIDE.md](./UNIT_TEST_GUIDE.md) for writing tests that verify
  contract conformance rather than implementation details.
- [DEFENSIVE_CODING_GUIDE.md](./DEFENSIVE_CODING_GUIDE.md) for enforcing
  preconditions at the interface boundary at runtime.
- [SOLID_GUIDE.md](./SOLID_GUIDE.md) for Interface Segregation — the
  principle that each interface should serve one client role — and for
  Liskov Substitution, which governs the behavioral contracts that
  implementations must honor.

## Summary Checklist

- [ ] Every public function and class has a declared interface before any
      implementation exists.
- [ ] The interface declares parameter types, return type, and all error cases.
- [ ] No infrastructure type appears in any domain or use-case interface.
- [ ] The interface is defined in the layer that uses it, not the layer that
      implements it.
- [ ] Every declared error case has a corresponding test.
- [ ] A caller can use the interface correctly without reading the
      implementation.
- [ ] Test doubles implement the same interface as the production
      implementation.
- [ ] The interface has not changed since the implementation was first written,
      unless the domain contract changed.
