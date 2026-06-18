# Code Quality Control Guide

This guide defines quality-control expectations for implementation changes in a
project, with focus on boundary-heavy integration code such as adapters, wrappers,
and third-party SDK boundaries.

It is intentionally narrow: use it to review the quality of implementation
changes, not as a replacement for the architecture and design guides.

## Source of truth

Use this guide together with:

- [Clean Architecture Guide](./CLEAN_ARCHITECTURE_GUIDE.md)
- [High Cohesion Guide](./HIGH_COHESION_GUIDE.md)
- [Low Coupling Guide](./LOW_COUPLING_GUIDE.md)
- [Referential Transparency Guide](./REFERENTIAL_TRANSPARENCY.md)
- [DRY Guide](./DRY_GUIDE.md)
- [Lightweight DDD Guide](../domain_specific/LIGHTWEIGHT_DDD_GUIDE.md)
- [DDD Guide](../domain_specific/DDD_GUIDE.md)
- [Unit Test Guide](./UNIT_TEST_GUIDE.md)
- [Integration Test Guide](./INTEGRATION_TEST_GUIDE.md)
- [End-to-End Test Guide](./E2E_TEST_GUIDE.md)

## Goal

Catch quality regressions early by checking that new code:

1. lands in the correct bounded context
2. keeps public APIs clear and intentional
3. isolates third-party SDK details at adapter boundaries
4. preserves deterministic helper logic where practical
5. stays covered by repository validation and focused tests

## Code Quality Control and Code LLMs

Quality control discipline also improves the quality of LLM-assisted coding.

Code-focused models produce better edits when each component has a clear
boundary, duplication is avoided, and tests lock down the intended contract.
When those conditions hold, a model generating or modifying code is less likely
to place logic in the wrong layer, duplicate knowledge that already exists, or
produce a change that passes individual unit tests but breaks the broader system.

This does not replace engineering review. It means rigorous quality control is
useful twice: it keeps human reasoning accurate, and it gives code models a
well-structured codebase to reason against.

### Why LLMs Benefit

- Clear responsibility boundaries make it obvious which file an edit belongs in.
- Explicit adapter boundaries prevent infrastructure assumptions from leaking
  into generated domain logic.
- A single source of truth for each rule means a model edits the right copy,
  not a stale duplicate.
- Consistent naming in the ubiquitous language reduces ambiguity in code
  generation.
- A full test suite at unit, integration, and end-to-end levels gives reliable
  feedback after each generated change.

### Where Weak Quality Control Hurts LLMs

- Broad, mixed-responsibility files make it hard to scope a safe edit.
- SDK shapes leaking into domain types cause generated code to couple business
  logic to external providers.
- Duplicated logic means a model may update one copy and silently leave others
  stale.
- Missing or flaky tests make it impossible to confirm whether a generated
  change preserved the intended behavior.
- Infrastructure wiring scattered across layers causes generated changes to
  introduce unwanted side effects in the wrong layer.

## Quality gates

Every substantive code change should satisfy these gates.

### 1. Responsibility gate

- A file, class, or document should keep one clear primary job.
- Wrapper modules should orchestrate runtime behavior, not become generic
  buckets for parsing, mapping, compatibility glue, and policy logic at once.
- If a component description needs repeated "and", split or extract.

### 2. Boundary gate

- Public APIs should expose library-owned concepts by default.
- Third-party SDK shapes should cross into public APIs only when the leak is
  explicit, justified, and documented.
- Dependency quirks, dynamic imports, and version compatibility workarounds
  should stay in narrow internal adapters.

### 3. DDD-alignment gate

- Use the project's established ubiquitous language consistently across all
  modules and documents. See [DDD Guide](../domain_specific/DDD_GUIDE.md) for strategic and
  tactical modeling patterns; see [Lightweight DDD Guide](../domain_specific/LIGHTWEIGHT_DDD_GUIDE.md)
  for a pragmatic subset.
- Prefer value-style modeling for requests, responses, configs, and parsed
  outputs.
- Keep aggregate roots responsible for enforcing invariants; do not spread
  validation across services or controllers.
- Avoid adding abstractions whose main effect is ceremony rather than clarity.

### 4. Purity gate

- Keep pure mapping, parsing, normalization, and validation logic in small
  reusable helpers where practical.
- Keep filesystem, process, environment, network, and SDK session work in
  explicit runtime-facing modules.
- Do not hide side effects behind utility-sounding names.

### 5. DRY gate

- Every piece of knowledge — logic, configuration, validation rule, or
  documentation — should have one authoritative representation. See
  [DRY Guide](./DRY_GUIDE.md) for the full rule set.
- Extract logic that appears more than once into a named abstraction.
- Keep configuration in one place and reference it everywhere it is needed.
- Do not copy documentation sections — write them once and cross-link.
- Flag "keep in sync" comments as unresolved duplication that must be
  consolidated before merging.

### 6. Test gate

- Changes to public behavior require focused tests at the affected boundary.
- Extracted helper logic should gain direct unit coverage when its behavior is
  significant enough to regress independently.
- Cross-boundary interactions — database, HTTP, queue, filesystem — require
  integration tests against real boundaries, not mocks. See
  [Integration Test Guide](./INTEGRATION_TEST_GUIDE.md).
- Critical user-visible flows require end-to-end coverage against the
  assembled stack. See [End-to-End Test Guide](./E2E_TEST_GUIDE.md).
- Split tests along responsibility seams when a refactor separates execution
  logic from administration or translation logic.

### 7. Documentation gate

- Update user-facing docs when public API behavior, exports, or recommended
  usage changes.
- Cross-link to related design guides instead of restating them.
- Call out intentional breaking cleanup in `CHANGELOG.md`.

### 8. Architecture gate

See [Clean Architecture Guide](./CLEAN_ARCHITECTURE_GUIDE.md) for the full layer
reference. The key checks at review time:

- Dependencies must point inward. No inner-layer file (domain, use case) may
  import from an outer layer (adapters, infrastructure, frameworks).
- Domain and use-case files must be free of framework, database, SDK, and
  transport imports.
- Use-case logic should delegate all I/O through interfaces or ports defined
  in the inner layers, not by constructing infrastructure dependencies
  directly.
- Adapter translation code must stay at the boundary; it must not leak into
  domain types or use-case files.
- Infrastructure wiring (HTTP, database clients, SDKs, DI containers) belongs
  in the outermost layer or composition root only.
- Domain and use-case logic should be exercisable in tests without starting a
  server or opening a real database connection.

### 9. Validation gate

Run the repository validation commands for substantive code changes. Adapt
these to the project's actual toolchain:

**Source-code projects** — typical examples:

1. Lint: `npm run lint` / `ruff check .` / equivalent
2. Test: `npm test` / `pytest` / equivalent
3. Build: `npm run build` / `tsc` / equivalent

**Documentation-only projects** using [ai_workflow.js](https://github.com/mpbarbosa/ai_workflow.js):

1. Quick review: `ai-workflow run --stage quick`
2. Full pipeline: `ai-workflow run --stage full`

## Positive Signals

- Files and classes have one clear job that can be described in a single
  sentence without "and".
- Public API names reflect the domain language, not the names of the underlying
  SDK or framework.
- Third-party adapter details are isolated to the outermost layer.
- Pure helpers are short, stateless, and directly testable.
- Each piece of logic has one home; changing it requires editing one file.
- Unit, integration, and end-to-end tests are clearly separated and all pass.
- Docs and changelog are updated as part of the change, not in a follow-up.
- A reviewer can identify the layer and responsibility of any changed file from
  its imports and contents alone.

## Warning Signs

- A file orchestrates behavior, maps data, handles errors, and manages
  configuration all at once.
- SDK types appear in public APIs or domain models with no justification.
- Compatibility workarounds are mixed in with business logic.
- The same calculation or validation appears in more than one place.
- A "keep in sync" comment guards duplicated logic.
- New behavior is only covered by the test that already existed, with no
  dedicated focused test.
- Documentation changes are missing from a PR that modifies public behavior.
- An inner-layer file imports from a framework, database client, or SDK.
- Infrastructure wiring is spread across domain or use-case files.

## Related Guides

- [INCREMENTAL_CHANGE_GUIDE.md](./INCREMENTAL_CHANGE_GUIDE.md) for structuring
  AI-assisted changes so each one is verifiable independently — incremental
  change is the workflow that makes quality control gates effective.
- [SOLID_GUIDE.md](./SOLID_GUIDE.md) for the structural principles that make
  quality control reviewable — SOLID-compliant code has clear extension points,
  stable interfaces, and predictable change patterns.
- [INTERFACE_FIRST_GUIDE.md](./INTERFACE_FIRST_GUIDE.md) for the contract
  review gate — the interface is the specification that quality control checks
  each change against.

## Summary Checklist

- [ ] The change belongs to the correct module boundary.
- [ ] Public names reflect domain concepts rather than accidental SDK naming.
- [ ] Compatibility shims are isolated from business-facing APIs.
- [ ] Pure helpers are separated from runtime orchestration where practical.
- [ ] No logic or knowledge is duplicated — each fact has one authoritative home.
- [ ] New abstractions improve clarity more than they increase indirection.
- [ ] Unit tests cover the changed boundary and any newly extracted critical helper.
- [ ] Cross-boundary behavior is covered by integration tests against real boundaries.
- [ ] Critical user-visible flows are covered by end-to-end tests.
- [ ] Docs and changelog reflect any meaningful API or behavior change.
- [ ] Repository validation commands still pass.
- [ ] No inner-layer file (domain, use case) imports from an outer layer.
- [ ] Domain and use-case files have no framework, database, or SDK imports.
- [ ] Use-case logic delegates I/O through inner-layer interfaces, not by constructing infrastructure directly.
- [ ] Adapter translation code does not leak into domain types or use-case logic.
- [ ] Infrastructure wiring is confined to the outermost layer or composition root.
