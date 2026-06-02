# Clean Architecture Guide

Clean architecture is a useful structural design rule for this document template
library and for the projects that use it as reference.

## Goal

Organize the system into concentric layers where inner layers hold stable
business rules and outer layers hold volatile implementation details. All
dependencies must point inward. No inner layer knows anything about the outer
layers that surround it.

## What Clean Architecture Means

Clean architecture means the most important code — the business rules and domain
logic — is the most isolated. Infrastructure, frameworks, databases, and
user-facing adapters are pushed to the edges and depend on stable inner
abstractions, not the other way around.

In practice, it means:

1. Core business rules do not import from frameworks, databases, or transport
   libraries.
2. Use-case logic orchestrates domain rules and delegates I/O to adapters.
3. Adapters translate between external shapes and the internal model.
4. Infrastructure wiring (HTTP, file system, database clients, SDKs) is
   confined to the outermost layer.
5. Every cross-layer call crosses a defined boundary.

It does **not** mean introducing all four canonical layers for every project.
Use as many layers as the actual complexity warrants.

## Why It Matters

1. Domain logic can be tested without spinning up a database, HTTP server, or
   external service.
2. Infrastructure can be replaced without rewriting business rules.
3. Use-case logic and domain policy have the longest useful life — protecting
   them from churn keeps the core stable.
4. Teams can work on UI, adapters, or domain logic without stepping on each
   other.
5. The system can support multiple entry points — REST API, CLI, message queue,
   tests — without duplicating core logic.

## Clean Architecture and Code LLMs

Clean architecture also improves the quality of LLM-assisted coding.

Code-focused models work better when the responsibility of each file is obvious
from its layer position. When domain rules live in isolated modules with no
framework imports, and adapters stay at the edges, an LLM is less likely to
leak infrastructure assumptions into business logic, or propose changes in the
wrong layer.

This does not replace engineering judgment. It means a well-layered codebase
helps twice: it keeps human reasoning clear about where changes belong, and it
gives code models stronger signals for making safer, correctly scoped edits.

### Why LLMs Benefit

- Layer-appropriate files have narrower, more predictable responsibilities.
- Inward-only dependencies reduce hidden side effects.
- Isolated domain logic is easier to reason about without full environment
  setup.
- Adapter boundaries make provider-specific details easy to identify and
  contain.
- Use-case files describe what the system does, not how it connects to
  infrastructure.

### Where Mixed Layers Hurt LLMs

- Business rules entangled with HTTP or database code are harder to extract and
  reason about.
- Framework imports scattered across domain logic create hidden coupling.
- One change in infrastructure may cascade into domain and use-case files.
- Generic utility files that cross layers make the dependency direction
  ambiguous.

## Required Rules

1. Dependencies only point inward. Outer layers depend on inner layers. Inner
   layers never import from outer layers.
2. Business rules and domain logic must be free of framework, database, SDK, and
   transport imports.
3. Use-case logic orchestrates the domain and delegates all I/O through
   interfaces or ports defined in the inner layers.
4. Adapters own the translation between external shapes and the internal model.
5. Wiring, composition, dependency injection, and framework bootstrap belong in
   the outermost layer.
6. Each layer should contain only the code that fits its responsibility.
7. Interfaces and ports for external collaborators must be defined inside, not
   in the outer layer that implements them.

## Layer Reference

| Layer | Typical contents | Allowed dependencies |
| --- | --- | --- |
| Domain (entities) | Core business rules, value types, invariants, domain errors | None — no external imports |
| Use cases | Application-specific workflows, orchestration, policy decisions | Domain layer only |
| Interface adapters | Controllers, presenters, translators, use-case facades | Use cases and domain |
| Frameworks & drivers | HTTP handlers, database clients, SDKs, CLI entry points, configuration | Adapters and use cases |

Adjust layer names and boundaries to match the project's vocabulary. The rule
that matters is dependency direction, not the names.

## Best Practices

### When Creating a New File

1. Decide which layer the file belongs to before naming it.
2. Check that its imports point inward, not sideways or outward.
3. Do not let infrastructure details define the shape of an inner-layer type.
4. Domain and use-case files should read without infrastructure context.

### When Creating Functions or Classes

1. Keep domain rules free of I/O.
2. Define ports (interfaces) for external collaborators inside the use-case or
   domain layer.
3. Implement ports in adapter or infrastructure files, not inside business logic.
4. Pass infrastructure dependencies in from the outside rather than constructing
   them inside core logic.
5. Keep use-case orchestration separate from domain invariant enforcement.

### When Writing Documentation

1. Reference this guide when a component crosses a layer boundary in ways that
   need explanation.
2. Use cross-links instead of repeating layer rules across many documents.
3. Document adapters: describe what external shape they translate from, and what
   internal shape they produce.
4. Keep architecture notes separate from feature-level or usage-level guides.

## Refactoring for Clean Architecture

When a codebase has mixed layers, separate them deliberately.

1. Identify which behavior represents a real business rule versus an
   infrastructure detail.
2. Move business rules into framework-free domain or use-case modules.
3. Define an interface in the inner layer for each external dependency the
   inner layer needs.
4. Implement those interfaces in adapter or infrastructure modules.
5. Move framework, HTTP, and database imports outward until no inner file
   imports them.
6. Wire everything at the composition root or entry point.
7. Verify that each layer can be tested in isolation.

## Review Heuristics

### Dependency Direction Test

Trace the imports of any file in the system. Do they all point inward, toward
more stable layers? If a domain file imports a framework module, the rule is
violated.

### Framework Import Test

Do any files in the domain or use-case layers import from a framework, database
client, HTTP library, or external SDK? If yes, those imports should move
outward.

### Isolation Test

Can the domain and use-case layers be exercised in tests without starting a
server, opening a database connection, or mocking a framework? If no, the
boundary between use cases and infrastructure is too weak.

This matters even more in LLM-assisted work: the ability to reason about domain
rules in isolation makes AI-generated edits much less likely to introduce
unwanted infrastructure side effects.

### Adapter Translation Test

Does each adapter translate between one external shape and one internal
concept? Adapters that know too much about business rules, or domain modules
that know too much about external formats, indicate a boundary leak.

### Composition Root Test

Is infrastructure wiring concentrated in one or a few entry-point files? If
wiring is scattered across use-case and domain files, the composition boundary
is not clean.

## Positive Signals

- Domain and use-case files have no external SDK or framework imports.
- Each layer's files have consistent, predictable responsibilities.
- A new adapter can be added without touching domain or use-case logic.
- Tests for business rules do not require environment setup.
- The composition root is the only place that knows the full dependency graph.
- A new entry point (CLI, HTTP, message consumer) reuses use-case logic without
  changes.

## Warning Signs

- Domain files import from HTTP libraries, ORM clients, or external SDKs.
- Business rules are spread across controller handlers.
- Use-case logic constructs its own infrastructure dependencies instead of
  receiving them.
- A single file orchestrates, validates, persists, and formats a response.
- Adapter translation code leaks into domain types.
- Tests for domain logic require spinning up external services.
- Infrastructure configuration is read inside domain functions.

## Related Guides

- [HIGH_COHESION_GUIDE.md](./HIGH_COHESION_GUIDE.md) for keeping each module
  focused on one clear responsibility within its layer.
- [LOW_COUPLING_GUIDE.md](./LOW_COUPLING_GUIDE.md) for keeping dependencies
  explicit and directional boundaries enforced.
- [LIGHTWEIGHT_DDD_GUIDE.md](../domain_specific/LIGHTWEIGHT_DDD_GUIDE.md) for naming core domain
  concepts and separating policy logic from infrastructure.
- [REFERENTIAL_TRANSPARENCY.md](./REFERENTIAL_TRANSPARENCY.md) for keeping
  domain and use-case logic side-effect-free and testable.

## Summary Checklist

- [ ] Dependencies only point inward — no inner layer imports from an outer
      layer.
- [ ] Domain files have no framework, database, or transport imports.
- [ ] Use-case files orchestrate domain logic through defined interfaces.
- [ ] Adapters handle translation at each layer boundary.
- [ ] Infrastructure wiring is concentrated near the composition root.
- [ ] Domain and use-case logic can be tested without environment setup.
- [ ] Interfaces for external collaborators are defined in the inner layer that
      uses them, not the outer layer that implements them.
- [ ] A reviewer or code-focused LLM could identify the layer and responsibility
      of any file from its imports and contents alone.
