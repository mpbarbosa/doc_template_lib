# SOLID Guide

SOLID is a set of five design principles — Single Responsibility, Open/Closed,
Liskov Substitution, Interface Segregation, and Dependency Inversion — that,
applied together, produce code that is stable under change, testable in
isolation, and safe to extend in AI-assisted workflows.

## Goal

Apply the five SOLID principles to produce modules where each has one reason to
change, new behavior is added through extension rather than modification,
subtypes honor their supertypes' contracts, interfaces are narrow and
role-specific, and high-level policy depends on abstractions rather than
implementations.

## What SOLID Means

SOLID means the structure of the code encodes its change characteristics.
A SOLID-compliant module is stable because the things most likely to change —
implementations, providers, new behavior — are isolated from the things that
must remain stable — interfaces, contracts, business rules.

In practice, it means:

1. **Single Responsibility (SRP):** each module has exactly one reason to
   change — one actor whose requirements drive its evolution.
2. **Open/Closed (OCP):** adding new behavior extends the existing structure;
   it does not modify it.
3. **Liskov Substitution (LSP):** every subtype is a correct behavioral
   substitute for its supertype — not just structurally compatible, but
   contractually equivalent.
4. **Interface Segregation (ISP):** each client depends on a narrow interface
   that contains only the methods it uses.
5. **Dependency Inversion (DIP):** high-level modules depend on abstractions
   defined in their own layer; low-level modules implement those abstractions.

The five principles are not independent. OCP is enabled by DIP (abstracting
what varies). LSP is the behavioral contract that makes OCP substitutions
correct. ISP is the structural form that SRP takes at the interface boundary.

## Why It Matters

1. Each SOLID violation seems minor in isolation but compounds. A module with
   mixed responsibilities, a growing switch statement, and a fat interface
   becomes progressively harder to change, test, and reason about as the
   codebase grows.
2. AI tools introduce specific SOLID violations: OCP violations via conditional
   branching instead of extension, LSP violations via behavioral contract drift
   in subclasses, ISP violations via fat interface generation. Knowing the
   pattern of violations makes AI-generated code faster to review.
3. SOLID-compliant code is more predictable for AI generation: stable
   interfaces, consistent extension patterns, and narrow dependencies give the
   model stronger signals for generating correctly scoped additions.
4. Tests written against SOLID-compliant code survive more changes because they
   depend on stable interfaces and contracts, not on volatile implementations.

## SOLID and Code LLMs

SOLID violations are a consistent pattern in AI-generated code, and each
principle maps to a specific failure mode.

OCP is the most commonly violated: the LLM adds a new `if`/`switch` branch to
an existing function rather than following the established extension pattern.
LSP is violated when the LLM generates a subclass that overrides a method with
subtly different behavior — satisfying the type checker but breaking the
behavioral contract. ISP is violated when the LLM models "everything this
object can do" in a single interface rather than what each client actually
needs.

Knowing these patterns makes SOLID review a targeted, fast pass rather than a
general code review.

### Why LLMs Benefit

- Stable interfaces are easier to generate correct implementations for.
- Established extension patterns (strategy, plugin, visitor) give the LLM a
  template to follow when adding new behavior.
- Narrow, role-specific interfaces reduce the context an LLM must load to
  generate a new implementation.
- DIP-compliant abstractions in inner layers prevent the LLM from generating
  implementation-specific imports in policy code.

### Where SOLID Violations Hurt LLMs

- An OCP violation (growing if/switch chain) causes the LLM to add more
  branches rather than suggesting an extension point.
- An LSP violation in an existing subclass is a silent incorrect example that
  the LLM may replicate when generating the next subclass.
- A fat interface forces the LLM to implement all methods for every new
  implementation, even those the specific client does not use.

## Principle Reference

| Principle | Rule | Primary existing guide | Novel content here |
| --- | --- | --- | --- |
| Single Responsibility | One reason to change per module | [HIGH_COHESION_GUIDE.md](./HIGH_COHESION_GUIDE.md) | Actor analysis, SRP at interface boundary |
| Open/Closed | Extend via new code; do not modify existing | — | Full coverage |
| Liskov Substitution | Subtypes are behavioral substitutes for supertypes | — | Full coverage |
| Interface Segregation | Clients depend only on methods they use | [INTERFACE_FIRST_GUIDE.md](./INTERFACE_FIRST_GUIDE.md) | Role interface pattern |
| Dependency Inversion | High-level policy depends on abstractions it defines | [LOW_COUPLING_GUIDE.md](./LOW_COUPLING_GUIDE.md), [CLEAN_ARCHITECTURE_GUIDE.md](./CLEAN_ARCHITECTURE_GUIDE.md) | Structural enforcement of DIP |

---

## Single Responsibility Principle

> A module should have one, and only one, reason to change.

A "reason to change" is an actor — a stakeholder, a team, a business concern —
whose requirements drive the module's evolution. A module that satisfies two
different actors has two reasons to change and will be pulled in two directions
by unrelated requirements.

See [HIGH_COHESION_GUIDE.md](./HIGH_COHESION_GUIDE.md) for the full principle.

### SRP at the Interface Boundary

SRP applies to interfaces as well as implementations. An interface that exposes
methods serving two different client roles has two reasons to change — when
one client's needs change, the interface changes for all clients. Split the
interface by role (see Interface Segregation below).

### SRP Review Signal

Name the actors who could require this module to change. If there is more than
one, the module has mixed responsibilities.

---

## Open/Closed Principle

> Software entities should be open for extension and closed for modification.

Adding new behavior should require writing new code, not changing existing
code. When new behavior requires modifying a function or class that already
works, every existing caller is at risk.

The OCP is enabled structurally by defining an extension point — an interface,
an abstract class, a plugin hook — that new implementations can fill without
touching the code that uses it.

### Required Rules

1. When new behavior is added to a module, the change must be an addition
   (a new implementation, a new strategy, a new handler) not a modification
   (a new branch in an existing function).
2. Every place where behavior varies by type or case is an extension point.
   Represent it as an interface or an abstract type, not as a switch statement.
3. The code that uses the extension point must not change when a new
   implementation is added.

### Extension Patterns

| Pattern | When to use |
| --- | --- |
| Strategy | Behavior that varies by algorithm or policy |
| Plugin / Handler registry | Behavior that varies by type, registered at startup |
| Decorator | Behavior extended by wrapping, not inheritance |
| Template method | Fixed algorithm structure with variable steps |

### OCP Review Heuristics

**Growth test:** Does adding new behavior of this type require modifying an
existing function or class? If yes, the extension point is missing or
bypassed.

**Switch test:** Does the module contain a `switch` or `if/else if` chain that
discriminates on a type, kind, or category? If yes, that chain will grow with
every new variant — an OCP violation waiting to happen. Replace it with
polymorphism or a dispatch table.

### OCP Warning Signs

- A function with a growing `switch (type)` or `if (type === 'A') ... else if
  (type === 'B')` chain.
- A commit that adds a new variant by modifying an existing function rather
  than adding a new implementation.
- A comment like `// Add new types here` inside a function body.
- AI-generated code that adds a new `case` to an existing switch rather than
  creating a new strategy implementation.

---

## Liskov Substitution Principle

> Subtypes must be substitutable for their base types without altering the
> correctness of the program.

Substitutability is behavioral, not structural. A subtype that has all the
same methods as its supertype but changes their meaning, narrows their
accepted inputs, or weakens their guarantees violates LSP even if it compiles.

### Behavioral Contract Rules

1. **Precondition rule:** a subtype must not strengthen the preconditions of
   an overridden method. If the supertype accepts any positive integer, the
   subtype cannot require a positive even integer.
2. **Postcondition rule:** a subtype must not weaken the postconditions of an
   overridden method. If the supertype guarantees the result is sorted, the
   subtype cannot return an unsorted result.
3. **Invariant rule:** a subtype must maintain all invariants the supertype
   establishes.
4. **Exception rule:** a subtype must not throw exception types that the
   supertype does not declare.

### LSP Review Heuristics

**Substitution test:** Replace every use of the supertype with the subtype
in the test suite. Do all tests still pass? If any test fails that was not
intended to verify subtype-specific behavior, LSP is violated.

**Contract comparison test:** For each overridden method, compare the
subtype's preconditions and postconditions to the supertype's. Has any
precondition been strengthened? Has any postcondition been weakened?

**Exception test:** Does the subtype throw any exception that does not appear
in the supertype's declared error cases?

### LSP Warning Signs

- A subtype overrides a method and throws an exception the supertype does not
  declare.
- A subtype's override requires a more specific input type than the supertype
  accepts.
- Callers of the supertype use `instanceof` checks to handle specific subtypes
  differently — a sign the subtypes are not true behavioral substitutes.
- Overridden methods that call `throw new NotImplementedError()` — a subtype
  that does not implement part of the contract it declares.
- AI-generated subclasses that override methods with `// TODO: implement` or
  empty bodies.

---

## Interface Segregation Principle

> Clients should not be forced to depend on methods they do not use.

A fat interface forces every client to depend on the full surface of the
interface, including methods it never calls. When the interface changes for one
client's needs, all other clients are affected. Split interfaces by role — by
what each distinct client actually needs.

See [INTERFACE_FIRST_GUIDE.md](./INTERFACE_FIRST_GUIDE.md) for the full
principle on contract-first design.

### Required Rules

1. Define a separate interface for each distinct client role. A cache client
   needs `get` and `set`. An eviction monitor needs `size` and `evict`. These
   are two interfaces, not one.
2. An implementation may implement multiple role interfaces. A single class
   can satisfy both `Readable` and `Writable`. Clients depend on the narrow
   role, not the full implementation.
3. Do not add a method to an existing interface because one client needs it
   and the implementation can support it. Add a new role interface for that
   client.

### ISP Review Heuristics

**Unused method test:** Does any client import an interface but only use a
subset of its methods? If yes, the interface covers more than one client role
and should be split.

**Change impact test:** If one client's needs change, does the interface change
in ways that require updating unrelated client implementations? If yes, the
interface is too broad.

### ISP Warning Signs

- An interface with more than five or six methods where different clients use
  different subsets.
- Client implementations that fulfill interface methods with empty bodies or
  `throw new NotImplementedError()` because they do not need those methods.
- AI-generated interfaces that model "everything this class can do" rather
  than what each client role actually requires.

---

## Dependency Inversion Principle

> High-level modules should not depend on low-level modules. Both should depend
> on abstractions. Abstractions should not depend on details. Details should
> depend on abstractions.

High-level policy — business rules, use cases — should not import low-level
implementation — databases, HTTP clients, file systems. Instead, high-level
policy defines the abstraction (an interface or port) it needs, and low-level
implementation satisfies it.

See [LOW_COUPLING_GUIDE.md](./LOW_COUPLING_GUIDE.md) for dependency management
and [CLEAN_ARCHITECTURE_GUIDE.md](./CLEAN_ARCHITECTURE_GUIDE.md) for the
architectural enforcement of dependency direction.

### DIP as the Structural Enforcement of OCP

DIP is what makes OCP possible at scale. An extension point is only closed for
modification if the code that depends on it references an abstraction, not a
concrete implementation. Without DIP, every new OCP extension requires the
high-level module to import the new concrete type — a modification.

### DIP Review Signal

Does any inner-layer file (domain, use case) import from an outer-layer file
(infrastructure, adapter, framework)? If yes, the dependency direction is
inverted — the high-level module depends on the low-level detail.

---

## Review Heuristics Summary

| Principle | Key test |
| --- | --- |
| SRP | Name the actors who could require this module to change. More than one? Split it. |
| OCP | Does adding new behavior of this type require modifying an existing function? |
| LSP | Replace the supertype with the subtype in all tests. Do they all still pass? |
| ISP | Does any client use only a subset of this interface's methods? |
| DIP | Does any inner-layer file import from an outer-layer file? |

## Positive Signals

- New behavior is added by creating new files, not by modifying existing ones.
- Subtypes can replace their supertypes in every test without failures.
- No client imports an interface for methods it does not call.
- Inner-layer files have no imports from framework, database, or transport
  modules.
- Switch statements on type or kind do not appear in business logic.
- Each module's responsibility can be described as one actor's concern.

## Warning Signs

- `switch (type)` or growing `if/else if` chains that require modification
  for each new variant.
- Subtype overrides that throw undeclared exceptions or impose stronger
  preconditions than the supertype.
- `instanceof` checks in code that should be polymorphic.
- Interface methods fulfilled with empty bodies or `NotImplementedError`.
- Inner-layer files that import from HTTP libraries, ORMs, or external SDKs.
- AI-generated subclasses with `// TODO` overrides or no-op method bodies.
- An interface that grows whenever any one of its several clients needs a new
  method.

## Related Guides

- [HIGH_COHESION_GUIDE.md](./HIGH_COHESION_GUIDE.md) for Single Responsibility
  — one responsibility per module, one reason to change.
- [INTERFACE_FIRST_GUIDE.md](./INTERFACE_FIRST_GUIDE.md) for Interface
  Segregation and the discipline of defining narrow contracts before
  implementations.
- [LOW_COUPLING_GUIDE.md](./LOW_COUPLING_GUIDE.md) for Dependency Inversion —
  minimal, explicit, directed dependencies.
- [CLEAN_ARCHITECTURE_GUIDE.md](./CLEAN_ARCHITECTURE_GUIDE.md) for the
  architectural enforcement of dependency direction across layers.
- [ERROR_HANDLING_GUIDE.md](./ERROR_HANDLING_GUIDE.md) for the Liskov
  connection — subtypes must not declare exception types beyond those in
  the supertype's contract.

## Summary Checklist

- [ ] Each module has one actor whose requirements drive its evolution (SRP).
- [ ] Adding a new variant or behavior requires new code, not modification of
      existing code (OCP).
- [ ] Every switch or if/else-if chain on type or kind is backed by a
      polymorphic extension point (OCP).
- [ ] Every subtype can replace its supertype in all tests without failures
      (LSP).
- [ ] No subtype strengthens preconditions or weakens postconditions of
      inherited methods (LSP).
- [ ] No subtype throws exception types not declared by its supertype (LSP).
- [ ] Each interface represents one client role; no client imports methods it
      does not use (ISP).
- [ ] No inner-layer file imports from an outer-layer implementation (DIP).
- [ ] Abstractions are defined in the layer that uses them, not the layer that
      implements them (DIP).
