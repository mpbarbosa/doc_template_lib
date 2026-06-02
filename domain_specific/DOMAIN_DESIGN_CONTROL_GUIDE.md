# Domain Design Control Guide

This guide defines domain-design review expectations for changes that touch
domain models, bounded contexts, public APIs, or user-facing interfaces.

It is intentionally narrow: use it to review the domain-design quality of
implementation changes, not as a replacement for the code quality and
architecture guides.

## Source of truth

Use this guide together with:

- [DDD Guide](./DDD_GUIDE.md)
- [Lightweight DDD Guide](./LIGHTWEIGHT_DDD_GUIDE.md)
- [REST API Guide](./REST_API_GUIDE.md)
- [Mobile-First Guide](./MOBILE_FIRST_GUIDE.md)
- [Clean Architecture Guide](../code_quality/CLEAN_ARCHITECTURE_GUIDE.md)
- [Code Quality Control Guide](../code_quality/CODE_QUALITY_CONTROL_GUIDE.md)

## Goal

Catch domain design regressions early by checking that new code:

1. uses the project's established ubiquitous language consistently
2. respects bounded context boundaries without leaking internal concepts
3. applies tactical DDD patterns only where they solve real complexity
4. exposes HTTP APIs that are resource-oriented, predictable, and versioned
5. delivers interfaces that are mobile-first and progressively enhanced
6. keeps domain logic free from infrastructure and transport concerns

## Domain Design Control and Code LLMs

Domain design discipline also improves the quality of LLM-assisted coding.

Code-focused models produce better domain-level edits when the ubiquitous
language is consistent, bounded contexts have visible boundaries, and API
contracts are predictable. When those conditions hold, a model adding a new
domain concept, endpoint, or UI flow is less likely to import the wrong
vocabulary, leak a context boundary, or introduce an API shape that breaks
existing consumers.

This does not replace design review. It means rigorous domain design is useful
twice: it keeps human reasoning grounded in the real business model, and it
gives code models stable, well-named signals for generating correctly scoped
edits.

### Why LLMs Benefit

- Consistent ubiquitous language removes ambiguity about which concept a file
  or symbol belongs to.
- Explicit bounded context boundaries prevent models from generating cross-context
  imports or shared mutable state where none should exist.
- Predictable REST URL patterns and status codes make new endpoint generation
  less error-prone.
- Mobile-first base styles give models a clear default to reason from before
  adding responsive enhancements.
- Separation of domain logic from infrastructure means generated domain edits
  stay within the correct layer.

### Where Weak Domain Design Hurts LLMs

- Inconsistent terminology forces the model to guess which name is canonical,
  producing ambiguous edits.
- Leaky bounded contexts mean a change in one area silently affects another.
- Verb-based or inconsistent API URLs make new route generation unpredictable.
- Desktop-first CSS defaults scatter overrides across breakpoints, making
  responsive edits harder to scope.
- Domain logic tangled with infrastructure makes it impossible to edit one
  without risking the other.

## Review gates

Every substantive change that touches domain models, APIs, or interfaces should
satisfy these gates.

### 1. Language gate

- Use the project's established ubiquitous language consistently in code,
  tests, documentation, and API contracts. See [DDD Guide](./DDD_GUIDE.md)
  for the full strategic vocabulary; see
  [Lightweight DDD Guide](./LIGHTWEIGHT_DDD_GUIDE.md) for the pragmatic subset.
- Every new concept introduced by a change must have one agreed name. That name
  must appear consistently wherever the concept is referenced.
- Vendor or framework terminology must not replace domain terms inside
  module names, type names, or API fields.
- When a change introduces a term that conflicts with existing language, resolve
  the conflict before merging — do not leave two names for the same concept.

### 2. Context boundary gate

- Each change must land in the correct bounded context. Concepts from one
  context must not bleed into another without an explicit anti-corruption layer.
- Domain objects must not be shared directly across context boundaries. Use
  translation at the boundary instead.
- If a change requires a new concept that spans contexts, decide whether it
  belongs to a shared kernel or whether it signals a new context, and document
  that decision.
- Context boundaries must be visible in the file structure and naming, not only
  implied by comments.

### 3. Domain model gate

- Apply tactical DDD patterns only where they solve real complexity. See
  [DDD Guide](./DDD_GUIDE.md) for guidance on entities, value objects,
  aggregates, repositories, domain services, and domain events.
- Prefer value objects over primitives for domain-significant data with
  invariants.
- Aggregate roots must enforce all invariants for their cluster. No external
  object may bypass the root to modify internal state.
- Domain services must be stateless and express pure policy — they must not
  call infrastructure directly.
- Avoid introducing aggregates, repositories, or domain events unless they
  solve a concrete complexity problem. Ceremony without real benefit is a
  design smell.

### 4. REST API gate

- URLs must identify resources as plural nouns. HTTP methods carry the
  operation semantics. See [REST API Guide](./REST_API_GUIDE.md) for the full
  rule set.
- HTTP methods must match their safety and idempotency guarantees: GET for
  read, POST for create, PUT for full replace, PATCH for partial update, DELETE
  for removal.
- Status codes must accurately reflect the actual outcome. Never return 200 for
  an error. Never return 500 for a client mistake.
- Every error response must follow a single consistent envelope structure with
  a machine-readable `code` field.
- Breaking changes — removed fields, changed types, removed endpoints — require
  a new API version before shipping to consumers.
- Every collection endpoint that can return an unbounded number of items must
  paginate.

### 5. Mobile-first gate

- New UI features must define the base layout, typography, spacing, and
  interaction model for the smallest supported viewport first. See
  [Mobile-First Guide](./MOBILE_FIRST_GUIDE.md) for the full rule set.
- Responsive enhancements must use `min-width` breakpoints, not `max-width`
  overrides.
- Interactive controls must be touch-safe by default. Hover-only behavior is an
  optional enhancement, never a requirement for the primary flow.
- Performance decisions must be biased toward mobile: defer heavy assets and
  enhancements that are not needed for the base experience.

### 6. Separation gate

- Domain logic must remain free from infrastructure, transport, and framework
  imports. See [Clean Architecture Guide](../code_quality/CLEAN_ARCHITECTURE_GUIDE.md)
  for the full layer rules.
- Anti-corruption layers must translate external concepts into the domain
  language at the boundary — not inside domain objects or use-case logic.
- Repository interfaces must be defined in the domain layer and implemented in
  infrastructure. Domain code must not reference storage libraries directly.
- Application services may orchestrate domain objects and call repositories;
  they must not contain business rules that belong in the domain layer.

### 7. Documentation gate

- Update domain glossaries and context maps when a new concept, boundary, or
  integration pattern is introduced.
- Cross-link to the relevant domain guide instead of restating shared
  principles.
- Record architectural decisions that are hard to reverse, surprising without
  context, or the result of a real trade-off. Routine additions do not need an
  ADR.
- Call out intentional breaking API changes in `CHANGELOG.md`.

### 8. Validation gate

Run the repository validation commands for substantive changes. Adapt to the
project's actual toolchain:

**Source-code projects** — typical examples:

1. Lint: `npm run lint` / `ruff check .` / equivalent
2. Test: `npm test` / `pytest` / equivalent
3. Build: `npm run build` / `tsc` / equivalent

**Documentation-only projects** using [ai_workflow.js](https://github.com/mpbarbosa/ai_workflow.js):

1. Quick review: `ai-workflow run --stage quick`
2. Full pipeline: `ai-workflow run --stage full`

## Positive Signals

- Every new concept has one name used consistently across code, tests, docs,
  and API contracts.
- Bounded context boundaries are visible in the file structure and respected
  at every cross-context call.
- Domain objects are free of infrastructure imports; adapters live at the edges.
- Tactical DDD patterns are used only where they protect a real invariant or
  boundary.
- API URLs are plural nouns; status codes match actual outcomes; errors share
  one envelope structure.
- The base UI layout works well on a small screen without any media query.
- Breaking API changes are versioned before shipping; additive changes are not.
- Domain decisions that are hard to reverse are recorded as ADRs.

## Warning Signs

- The same concept is named differently across modules, tests, or API fields.
- Domain objects import from HTTP libraries, ORM clients, or external SDKs.
- An aggregate can be mutated by bypassing the root.
- A domain service calls a repository or sends an HTTP request directly.
- API URLs contain verbs: `/getUser`, `/processPayment`, `/createOrder`.
- A 200 status code is returned for an error condition.
- Different endpoints return different error shapes with no consistent envelope.
- A breaking API change ships without a version bump.
- The base CSS layout assumes desktop width and overrides for mobile with
  `max-width` rules.
- A new tactical DDD pattern is introduced without a concrete complexity
  problem to justify it.
- Cross-context imports appear without an anti-corruption layer.
- Domain decisions are undocumented or left implicit in implementation details.

## Summary Checklist

- [ ] Every new concept has one agreed name used consistently everywhere.
- [ ] The change lands in the correct bounded context with no boundary leak.
- [ ] Cross-context interactions go through an explicit anti-corruption layer.
- [ ] Tactical DDD patterns are applied only where they solve real complexity.
- [ ] Aggregate roots enforce all invariants; no external code bypasses them.
- [ ] Domain services are stateless and call no infrastructure directly.
- [ ] Repository interfaces are defined in the domain layer.
- [ ] API URLs use plural nouns with no verbs in path segments.
- [ ] HTTP methods match their safety and idempotency guarantees.
- [ ] Status codes accurately reflect the outcome on every endpoint.
- [ ] All error responses share the same envelope structure with a `code` field.
- [ ] Breaking API changes are shipped under a new version.
- [ ] Collection endpoints paginate results.
- [ ] The base UI layout is defined for the smallest viewport without media
      queries.
- [ ] Responsive enhancements use `min-width` breakpoints.
- [ ] Domain logic is free from framework, database, and transport imports.
- [ ] Hard-to-reverse domain decisions are recorded as ADRs.
- [ ] Docs and changelog reflect any meaningful API or domain model change.
- [ ] Repository validation commands still pass.
