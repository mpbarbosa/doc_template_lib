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
- [Lightweight DDD Guide](./LIGHTWEIGHT_DDD_GUIDE.md)

## Goal

Catch quality regressions early by checking that new code:

1. lands in the correct bounded context
2. keeps public APIs clear and intentional
3. isolates third-party SDK details at adapter boundaries
4. preserves deterministic helper logic where practical
5. stays covered by repository validation and focused tests

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
  modules and documents.
- Prefer value-style modeling for requests, responses, configs, and parsed
  outputs.
- Avoid adding abstractions whose main effect is ceremony rather than clarity.

### 4. Purity gate

- Keep pure mapping, parsing, normalization, and validation logic in small
  reusable helpers where practical.
- Keep filesystem, process, environment, network, and SDK session work in
  explicit runtime-facing modules.
- Do not hide side effects behind utility-sounding names.

### 5. Test gate

- Changes to public behavior require focused tests at the affected boundary.
- Extracted helper logic should gain direct unit coverage when its behavior is
  significant enough to regress independently.
- Split tests along responsibility seams when a refactor separates execution
  logic from administration or translation logic.

### 6. Documentation gate

- Update user-facing docs when public API behavior, exports, or recommended
  usage changes.
- Cross-link to related design guides instead of restating them.
- Call out intentional breaking cleanup in `CHANGELOG.md`.

### 7. Architecture gate

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

### 8. Validation gate

Run the repository validation commands for substantive code changes. Typical
examples:

1. Lint: `npm run lint` / `ruff check .` / equivalent
2. Test: `npm test` / `pytest` / equivalent
3. Build: `npm run build` / `tsc` / equivalent

Adapt these to the project's actual toolchain.

## Review checklist

- [ ] The change belongs to the correct module boundary.
- [ ] Public names reflect library concepts rather than accidental SDK naming.
- [ ] Compatibility shims are isolated from business-facing APIs.
- [ ] Pure helpers are separated from runtime orchestration where practical.
- [ ] New abstractions improve clarity more than they increase indirection.
- [ ] Tests cover the changed boundary and any newly extracted critical helper.
- [ ] Docs and changelog reflect any meaningful API or behavior change.
- [ ] Repository validation commands still pass.
- [ ] No inner-layer file (domain, use case) imports from an outer layer.
- [ ] Domain and use-case files have no framework, database, or SDK imports.
- [ ] Use-case logic delegates I/O through inner-layer interfaces, not by constructing infrastructure directly.
- [ ] Adapter translation code does not leak into domain types or use-case logic.
- [ ] Infrastructure wiring is confined to the outermost layer or composition root.

## Summary

Good quality control is mostly about keeping boundaries clear, abstractions
small, and public APIs intentional. Favor thinner adapters, focused helpers, and
explicit documentation over broad wrappers and hidden dependency leakage.
