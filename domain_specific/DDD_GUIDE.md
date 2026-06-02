# DDD Guide

Domain-Driven Design (DDD) is a software design approach that centers the
project's architecture on the core business domain, its language, and its
rules. It is a useful design philosophy for this document template library and
for the projects that use it as reference.

> For projects that benefit from only a pragmatic subset of DDD, see
> [LIGHTWEIGHT_DDD_GUIDE.md](./LIGHTWEIGHT_DDD_GUIDE.md).

## Goal

Build software whose structure, vocabulary, and boundaries directly reflect the
business problem being solved. Keep the domain model at the center of every
architectural decision, and prevent external concerns — storage, transport, UI,
frameworks — from contaminating core logic.

## What DDD Means

DDD means that the primary driver of design is deep understanding of the domain,
expressed through a shared language between developers and domain experts, and
encoded in a model that lives at the heart of the system.

A well-applied DDD system can answer:

- "What are the core concepts this system owns, and where do they live?"
- "Which team or context is authoritative for each business rule?"
- "How do different parts of the system collaborate without sharing internals?"

DDD operates at two levels: **strategic** (how contexts and teams are
structured) and **tactical** (how domain objects are coded within a context).
Both levels must be addressed for DDD to deliver its benefits.

## Why It Matters

1. It keeps the software model aligned with how the business actually works.
2. It provides a shared language that reduces misunderstandings between
   developers and domain experts.
3. It keeps complex business rules isolated from infrastructure concerns.
4. It makes context boundaries explicit, reducing accidental coupling across
   teams and services.
5. It gives the system a stable core that survives changes to frameworks,
   databases, and external APIs.

## DDD and Code LLMs

DDD also improves the quality of LLM-assisted coding.

Code-focused models perform better when each file's purpose is derivable from
its domain position — the context it belongs to, the concept it encodes, the
invariant it protects. Explicit domain language, stable aggregate roots, and
clear context maps give a model reliable signals for generating focused edits
with minimal risk of breaking boundaries.

This does not replace engineering judgment. It means DDD is useful twice: it
helps humans reason about complex domains, and it gives code models a richer,
more stable signal for safer and more precise suggestions.

### Why LLMs Benefit

- Named domain concepts reduce ambiguous interpretations.
- Aggregate boundaries limit the scope of any single edit.
- Invariants encoded in the model are easier to preserve across changes.
- Anti-corruption layers stop external terminology from polluting generated
  code.
- Explicit context maps make the right home for new logic easy to locate.

### Where Missing or Broken DDD Hurts LLMs

- Business rules scattered across layers are hard to locate and easy to
  duplicate.
- Shared mutable state crossing context boundaries creates unpredictable
  side effects.
- External API shapes leaking into the core model make refactors expensive.
- Vague naming invites the model to make structurally wrong guesses.
- No clear aggregate root means edits may touch more state than necessary.

---

## Strategic Patterns

Strategic DDD shapes the large-scale structure of the system before any code is
written.

### Ubiquitous Language

Ubiquitous language is a single, consistent vocabulary shared by domain experts
and developers for every core concept the system owns.

Rules:

1. Every important concept must have one agreed name.
2. That name must appear in code, tests, documentation, and team conversations.
3. The language belongs to a bounded context — names may differ across contexts
   when the meaning genuinely differs.
4. Avoid generic terms (`manager`, `handler`, `engine`, `processor`) when a
   domain term exists.
5. When domain experts use a different word than developers, align on one.

Good ubiquitous language is **specific**, **stable**, **close to real business
concepts**, and **distinct from infrastructure or vendor terminology**.

### Bounded Contexts

A bounded context is an explicit boundary within which a specific model and
language apply consistently. The same word may mean different things in
different contexts — that difference is a signal that they belong to separate
bounded contexts.

Each bounded context should be owned clearly:

| Ownership signal | Implication |
| --- | --- |
| One team maintains it | Strong candidate for its own context |
| Separate deployment cadence | Context boundary likely needed |
| Concepts with the same name but different meanings | Separate contexts required |
| Different consistency or transaction needs | Strong case for context isolation |

#### Identifying context boundaries

1. Map the core nouns and verbs that appear in business conversations.
2. Note where the same term carries a meaningfully different responsibility.
3. Draw a boundary around the cluster of concepts that share a consistent
   model.
4. Name each context after the business capability it represents.

### Context Maps

A context map documents how bounded contexts relate to each other and how
knowledge flows between them. Common relationship patterns:

| Pattern | Description |
| --- | --- |
| **Partnership** | Two contexts evolve together; teams coordinate changes |
| **Shared Kernel** | A small shared model owned jointly by two teams |
| **Customer–Supplier** | Upstream context publishes a contract; downstream consumes it |
| **Conformist** | Downstream adopts the upstream model without translation |
| **Anti-Corruption Layer** | Downstream translates the upstream model before use |
| **Open Host Service** | Upstream publishes a stable, versioned integration API |
| **Published Language** | Shared, well-documented exchange format across contexts |

Prefer Anti-Corruption Layer over Conformist when the upstream model is
unstable or semantically different from the downstream domain. Prefer Open Host
Service when many consumers need to integrate with a context.

---

## Tactical Patterns

Tactical DDD provides the building blocks for expressing domain logic in code
within a single bounded context. Use these when the domain is rich enough to
justify them.

### Entities

An entity is an object with a distinct identity that persists across time and
state changes.

Characteristics:

- Defined by identity, not attribute values.
- Two entities with the same attributes are still different objects if they
  have different identities.
- Owns behavior that changes its internal state in controlled ways.

When to use:

- The concept has a lifecycle the system must track.
- The system needs to find, update, and reason about the same instance over
  time.

### Value Objects

A value object is an immutable object defined entirely by its attribute values.
Two value objects with the same attributes are interchangeable.

Characteristics:

- No identity beyond its values.
- Immutable by design — replacing is preferred over mutating.
- Encodes invariants and validation close to construction.

Good candidates:

- Monetary amounts, measurements, coordinates.
- Identifiers with format or validation rules.
- Date ranges, status transitions, policy inputs and outputs.

### Aggregates

An aggregate is a cluster of related objects (entities and value objects)
treated as a single unit for data changes. The **aggregate root** is the sole
entry point for all operations on the cluster.

Rules:

1. External objects may only hold a reference to the aggregate root, never to
   internal members.
2. All invariants that span the cluster must be enforced within the aggregate.
3. One transaction should modify at most one aggregate.
4. Keep aggregates small — only include objects that must change together to
   preserve consistency.

When to use:

- Several objects share a consistency rule that must always hold together.
- The system needs a transactional boundary that the database cannot enforce
  alone.

When not to use:

- The domain does not own mutable, long-lived state.
- Apparent clusters are really independent entities with eventual consistency
  between them.

### Repositories

A repository provides a collection-like interface for loading and persisting
aggregates. It hides storage details behind a domain-oriented contract.

Rules:

1. One repository per aggregate root.
2. The interface is defined in the domain layer; the implementation lives in
   infrastructure.
3. The repository contract speaks domain language, not SQL or SDK language.
4. Repositories should not expose query builders or raw storage primitives to
   callers.

### Domain Services

A domain service encodes a business rule or policy that does not belong
naturally to any single entity or value object.

Good candidates:

- Rules that involve multiple aggregates or concepts.
- Calculations or decisions that are owned by the domain but have no obvious
  home object.
- Policies with complex conditional logic that would clutter a single entity.

Domain services should be stateless and express pure policy logic — they should
not call infrastructure directly.

### Domain Events

A domain event is a record of something meaningful that happened in the domain.
It is named in past tense and represents a fact, not a command.

Good candidates:

- `OrderPlaced`, `PaymentConfirmed`, `AccountSuspended`.
- Events that trigger downstream reactions in other parts of the system.
- Facts that need to be recorded for audit, integration, or notification.

Rules:

1. Events are immutable once created.
2. Events carry enough information for listeners to act without querying back.
3. Raising an event is a domain decision, not an infrastructure concern.
4. Prefer explicit event types over generic `Event<T>` wrappers.

### Factories

A factory encapsulates the logic required to create a valid aggregate or complex
value object, keeping construction details out of the caller.

Use factories when:

- Construction requires invariant enforcement too complex for a constructor.
- The object's valid initial state depends on multiple inputs or decisions.
- Callers should not need to know the concrete type being instantiated.

---

## Required Rules

1. Establish a ubiquitous language for every core concept and use it everywhere.
2. Keep bounded contexts explicit with visible boundaries in the codebase.
3. Keep domain logic (entities, aggregates, services) free from infrastructure
   dependencies.
4. Apply the aggregate pattern only where a real consistency boundary exists.
5. Define repository interfaces in the domain layer; implement them in
   infrastructure.
6. Use anti-corruption layers when integrating with external systems that speak
   a different language.
7. Name domain events in past tense; make them immutable.
8. Do not leak entity or aggregate internals through the aggregate root.
9. Prefer value objects over primitive types for domain-significant data.
10. Document the context map when the system has more than one bounded context.

## Positive Signals

- The core domain vocabulary appears consistently in code, tests, and docs.
- Each bounded context has a clear owner and a visible boundary.
- Infrastructure code imports domain types; domain code imports nothing from
  infrastructure.
- Aggregates are small and protect real invariants.
- External API shapes are translated at context edges, not mirrored inside.
- New features have an obvious home in an existing context or a well-reasoned
  new one.
- Business rules can be understood and tested without starting a database or
  server.

## Warning Signs

- Business rules are scattered across HTTP handlers, database queries, and
  background jobs.
- The same concept is named differently across modules, tests, and docs.
- Entities contain database column names or API field names directly.
- Aggregates grow large because convenience edits bypass the root.
- A "domain" layer that imports ORM decorators, HTTP types, or SDK clients.
- Transactions span multiple aggregates in a single call.
- No clear answer to "which context owns this rule?"

## Applying DDD by Component Type

| Component type | DDD approach |
| --- | --- |
| Aggregate root | Own invariant enforcement; be the sole mutation entry point |
| Entity | Maintain lifecycle state with identity-based equality |
| Value object | Encode invariants; enforce immutability |
| Domain service | Express stateless cross-entity policy; no infrastructure calls |
| Repository | Define the interface in the domain; implement in infrastructure |
| Domain event | Record past facts; carry sufficient data for listeners |
| Application service | Orchestrate use cases; delegate decisions to domain objects |
| Anti-corruption layer | Translate external concepts into the bounded context's language |
| Infrastructure adapter | Implement domain interfaces; never define domain concepts |

## Best Practices

### When Designing a New Bounded Context

1. Name the context after the business capability it represents.
2. Define the ubiquitous language before writing code.
3. Draw the context map showing how it relates to adjacent contexts.
4. Decide which integration pattern applies at each boundary.
5. Keep the first working model simple — let it evolve with domain understanding.

### When Creating Aggregates and Entities

1. Ask whether the concept truly needs identity and lifecycle tracking.
2. Prefer value objects; graduate to entities only when identity is required.
3. Keep the aggregate as small as the invariants allow.
4. Put all consistency checks in the aggregate root before persisting.
5. Emit domain events from aggregate methods when meaningful facts occur.

### When Writing Domain Services

1. Make the service stateless — inject only what it needs to compute the
   result.
2. Name it after the policy it expresses, not the technology it uses.
3. Keep it free from repository calls unless the rule requires loading related
   objects.
4. Prefer returning a result type over raising exceptions for expected outcomes.

### When Writing Documentation

1. Maintain a glossary of the ubiquitous language for each bounded context.
2. Document the context map when more than one context is active.
3. Cross-link to [LIGHTWEIGHT_DDD_GUIDE.md](./LIGHTWEIGHT_DDD_GUIDE.md) when
   the current project uses only a pragmatic subset of these patterns.
4. Keep tactical pattern usage justified — note why an aggregate or repository
   was introduced, not just that it was.

## Refactoring Toward DDD

When a system has accumulated tangled logic and inconsistent naming, migrate
incrementally rather than rewriting.

1. Identify the most valuable or most volatile business rules.
2. Name them using the domain language; rename existing code to match.
3. Extract those rules into focused domain objects, services, or value types.
4. Move infrastructure concerns (queries, HTTP calls, serialization) behind
   interfaces.
5. Add anti-corruption layers at external boundaries that currently leak.
6. Establish one aggregate root per consistency cluster; remove direct access
   to internals.
7. Add domain events at the points where important facts need to be
   communicated.
8. Defer introducing repositories, factories, and context maps until the model
   stabilizes enough to warrant them.

## Review Heuristics

### Language Consistency Test

Does every important concept have one name, used consistently in code, tests,
docs, and conversations? If reviewers frequently clarify terminology, the
ubiquitous language is incomplete.

### Dependency Direction Test

Does infrastructure depend on domain, or does domain depend on infrastructure?
Any import of a framework, ORM, or HTTP library inside a domain object is a
boundary violation.

### Invariant Enforcement Test

Can an aggregate be left in an invalid state by any caller? If yes, the
aggregate root is not enforcing its invariants.

### Context Ownership Test

For any business rule, is there one clear context that owns it? Rules that
live "between" contexts without an owner will drift and diverge.

### Transaction Scope Test

Does any single transaction modify more than one aggregate? If yes, consider
whether eventual consistency via domain events is the correct design.

### Event Naming Test

Are domain events named in past tense and do they carry enough information for
listeners to act without calling back? Events that describe commands or require
the listener to re-query the emitter are design smells.

## Preferred Fixes

1. Rename types, modules, and methods to match the ubiquitous language.
2. Extract scattered business rules into named domain services or value types.
3. Move infrastructure imports out of domain objects behind interfaces.
4. Introduce anti-corruption layers at boundaries where external models leak.
5. Shrink large aggregates by splitting them at natural consistency sub-groups.
6. Replace direct infrastructure calls with repository interfaces defined in
   the domain layer.
7. Introduce domain events where side effects are currently triggered
   procedurally across layers.
8. Delete abstractions that mirror external shapes without adding domain value.

## Related Guides

- [LIGHTWEIGHT_DDD_GUIDE.md](./LIGHTWEIGHT_DDD_GUIDE.md) for a pragmatic,
  selective application of DDD suited to projects where full tactical patterns
  add more ceremony than value.
- [CLEAN_ARCHITECTURE_GUIDE.md](../code_quality/CLEAN_ARCHITECTURE_GUIDE.md) for the
  dependency-direction rules that keep domain logic infrastructure-free.
- [HIGH_COHESION_GUIDE.md](../code_quality/HIGH_COHESION_GUIDE.md) for keeping each domain
  object focused on one clear responsibility.
- [LOW_COUPLING_GUIDE.md](../code_quality/LOW_COUPLING_GUIDE.md) for keeping context
  boundaries explicit and dependency surfaces minimal.
- [REFERENTIAL_TRANSPARENCY.md](../code_quality/REFERENTIAL_TRANSPARENCY.md) for expressing
  domain services and value computations as pure, testable logic.

## Summary Checklist

- [ ] The project has a documented ubiquitous language for each bounded
      context.
- [ ] Bounded contexts have visible boundaries in the codebase and a named
      owner.
- [ ] The context map documents how contexts integrate and which pattern each
      integration uses.
- [ ] Domain objects (entities, aggregates, value objects) are free from
      infrastructure imports.
- [ ] Aggregate roots enforce all invariants for their cluster; no external
      object bypasses them.
- [ ] Repository interfaces are defined in the domain layer and implemented in
      infrastructure.
- [ ] Domain events are named in past tense and carry sufficient data for
      listeners.
- [ ] Anti-corruption layers translate external concepts at context boundaries.
- [ ] Business rules can be tested without starting a database or server.
- [ ] A reviewer or code-focused LLM could locate any business rule from the
      domain language without searching infrastructure or transport code.
