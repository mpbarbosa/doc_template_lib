# Node.js Module Structure Guide

Node.js module structure is an architectural discipline for organizing library
and application code into layers with explicit, unidirectional dependencies and
a controlled public API surface.

## Goal

Organize a Node.js codebase so that any module's responsibilities are obvious
from its location, dependencies flow in one direction only, and the public API
surface is controlled from a single entry point — without reading every file to
understand what is exposed or how things connect.

## What Node.js Module Structure Means

Node.js module structure means code is partitioned into distinct layers with
clear purposes, a single entry point controls what is public, and lower layers
never depend on higher ones.

In practice, it means:

1. A single `index.js` (or `index.ts`) re-exports the complete public API and
   contains no business logic of its own.
2. Core business logic lives in a dedicated directory, isolated from utilities
   and configuration.
3. Reusable helpers and supporting classes live in a utility directory, free of
   domain logic.
4. Configuration constants and version information occupy their own layer with
   no runtime dependencies.
5. Dependencies flow strictly downward: entry → core → utils → config. No lower
   layer imports from a higher one.
6. Each module has a single responsibility. If its purpose requires the word
   "and", it covers too much.
7. Dependencies are injected rather than created internally, making them
   explicit and replaceable.

It does **not** mean enforcing exactly these four directories for every Node.js
project. The principle is the dependency direction and the single public API
surface, not the specific folder names.

## Why It Matters

1. A single entry point gives consumers one place to find everything the package
   exposes — and ensures nothing internal is accidentally public.
2. Strict downward dependency flow prevents circular imports, a common source of
   initialization bugs and unexpected coupling in Node.js.
3. Isolated core logic is independently testable without configuring utilities,
   infrastructure, or a full application environment.
4. Single-responsibility modules reduce the cognitive load of changes: a
   modification to one module should not require understanding unrelated ones.
5. Injected dependencies make coupling between modules visible in constructor
   signatures rather than hidden in global lookups or singletons.

## Node.js Module Structure and Code LLMs

Node.js module structure also improves the quality of LLM-assisted coding.

Code-focused models work better when each file's responsibility is obvious from
its layer position. When business logic is isolated from infrastructure, and
the public surface is declared in one place, a model generating a new module or
editing an existing one is less likely to introduce misplaced logic, hidden
dependencies, or unintended exports.

This does not replace engineering review. It means a well-layered Node.js
codebase helps twice: it keeps human reasoning clear about where a change
belongs, and it gives code models stronger signals for making correctly scoped
edits.

### Why LLMs Benefit

- Layer-appropriate files have narrower, more predictable responsibilities.
- Unidirectional imports reduce ambiguity about who depends on what.
- Isolated core logic is easier to reason about without full environment setup.
- A single entry point makes the intended public API surface explicit.
- Named exports make every import traceable to a specific declaration.

### Where Mixed Layers Hurt LLMs

- Business logic entangled with infrastructure details is harder to extract and
  reason about.
- Classes that create their own dependencies hide coupling that a model cannot
  see from the call site.
- Module-scope side effects triggered by `import` make execution order hard to
  predict when a model generates new module graphs.
- Default exports obscure what is being imported, making refactors error-prone.

## Required Rules

1. One entry point must export the entire public API. Consumers must not import
   from internal paths.
2. Dependencies must flow strictly downward. A lower-layer module must never
   import from a layer above it.
3. Each module must have one clear responsibility, describable in a single
   sentence without "and".
4. Dependencies must be injected, not created internally. A class that
   instantiates its own dependencies cannot be tested in isolation.
5. Named exports must be used throughout. Default exports obscure what is being
   imported and make refactoring harder.
6. Module scope must be free of side effects. Importing a module must not
   trigger network calls, filesystem access, or global state mutations.

## Layer Reference

| Layer | Typical location | Responsibility | May import from |
| --- | --- | --- | --- |
| Entry | `index.js` | Public API re-exports only, no logic | Core, Utils, Config |
| Core | `core/` | Business logic, domain classes, orchestration | Utils, Config |
| Utils | `utils/` | Reusable helpers, no domain logic | Config |
| Config | `config/` | Constants, version information | Nothing |

Adjust layer names to match the project's vocabulary. The rule that matters is
dependency direction, not the folder names.

## Best Practices

### Entry Point Design

1. Re-export from the entry point everything the consumer needs. Nothing else.
2. Put no logic in the entry point. It should consist entirely of re-export
   statements.
3. Use explicit named re-exports:
   `export { MyClass } from './core/MyClass.js'`. Avoid barrel `export *`
   unless the internal module is intentionally a namespace.

### Module Design

1. Name module files after the class or concept they contain: `OrderCache.js`,
   not `helpers.js`.
2. Keep files focused. If a class has more than one reason to change, split it.
3. Prefer static factory methods over constructor overloading for multiple
   creation pathways. Factory method names make the intended configuration
   explicit: `Fetcher.withCache(url)` is clearer than a constructor with an
   optional boolean flag.

### Dependency Injection

1. Accept dependencies as constructor parameters or factory method arguments.
   Do not instantiate dependencies inside a constructor.
2. Use duck typing for dependency contracts. A cache is anything with `has`,
   `get`, `set`, and `delete` — this lets consumers swap implementations
   without a formal interface declaration.
3. Provide sensible defaults for optional dependencies at the factory or call
   site, not deep inside the implementation.

### Pure Functional Core

Separate pure logic from side effects. A pure core accepts all its inputs as
parameters and returns a description of what should happen:

```js
// Pure core — no side effects
async function fetchPure(cacheState, timestamp, networkProvider) {
  // Returns a result object; never reads or writes external state
  return { data, newCacheState, events };
}

// Imperative shell — applies side effects
async function fetch() {
  const result = await fetchPure(this.cache, Date.now(), globalFetch);
  applyResult(result, this.cache);
  return result.data;
}
```

The pure core is easy to test deterministically; the shell contains all I/O.
See [REFERENTIAL_TRANSPARENCY.md](../code_quality/REFERENTIAL_TRANSPARENCY.md)
for the broader principle.

## Review Heuristics

### Dependency Direction Test

Do any modules in lower layers (`utils/`, `config/`) import from `core/`? Does
`core/` import from `index.js`? Any upward import indicates a layer violation
that will eventually create circular dependencies.

### Single Entry Test

Can a consumer import every public class, function, and constant through the
package entry point? If some symbols require reaching into internal paths
(`import { X } from 'pkg/src/core/X.js'`), the public API surface is leaking.

### Hidden Dependency Test

Does constructing a class trigger network calls, filesystem access, or global
state mutations? If yes, the class is not testable in isolation and its
dependencies are hidden rather than injected.

### Module Focus Test

Can the purpose of each module be described in a single sentence without "and"?
If not, the module covers multiple responsibilities and should be split.

### Pure Core Test

Can the core business logic be exercised by passing inputs and checking outputs
without any mocks for network, filesystem, or timing? If no, side effects are
mixed into the core and have not been isolated to the shell.

## Positive Signals

- The entry point contains only re-exports; it is the single import path for
  all public symbols.
- Lower-layer modules have no imports from higher layers; grepping for upward
  imports finds nothing.
- Constructors list all dependencies as parameters; no hidden instantiations
  (`new GlobalX()`) appear inside them.
- Core business logic functions are pure: same inputs, same outputs, no
  observable side effects.
- Named exports are used throughout; no default exports.
- A circular dependency check (e.g., `madge`) reports a clean graph.

## Warning Signs

- Consumers import from internal paths:
  `import { X } from 'pkg/src/core/X.js'`.
- A utility or config module imports from core.
- Classes instantiate their own dependencies internally
  (`this.cache = new Cache()`).
- Global singletons or module-level state that accumulates across calls.
- Default exports that obscure what is being imported.
- Side effects triggered by `import` statements (network, timers, file I/O).
- A single large file that mixes entry, core, utility, and configuration
  concerns.

## Related Guides

- [CLEAN_ARCHITECTURE_GUIDE.md](../code_quality/CLEAN_ARCHITECTURE_GUIDE.md)
  for the broader principle of layered, inward-facing dependencies across an
  entire application.
- [LOW_COUPLING_GUIDE.md](../code_quality/LOW_COUPLING_GUIDE.md) for keeping
  each module's dependencies minimal and explicit.
- [HIGH_COHESION_GUIDE.md](../code_quality/HIGH_COHESION_GUIDE.md) for
  ensuring each module has a single, focused responsibility.
- [REFERENTIAL_TRANSPARENCY.md](../code_quality/REFERENTIAL_TRANSPARENCY.md)
  for the pure functional core pattern used inside the Core layer.
- [UNIT_TEST_GUIDE.md](../code_quality/UNIT_TEST_GUIDE.md) for writing fast,
  isolated tests that depend injection makes possible.

## Summary Checklist

- [ ] A single entry point re-exports the complete public API with no logic of
      its own.
- [ ] Dependencies flow strictly downward; no lower-layer module imports from a
      higher one.
- [ ] Each module has one responsibility, describable in a single sentence.
- [ ] All dependencies are injected; no class creates its own dependencies
      internally.
- [ ] Named exports are used throughout; no default exports.
- [ ] Module scope is free of side effects at import time.
- [ ] Core business logic is pure and testable without mocks for
      infrastructure.
- [ ] Circular dependency analysis reports a clean graph.
