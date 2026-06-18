# API Documentation

This repository contains only Markdown documentation templates and does not expose a programmatic API.

## Usage

- Copy and adapt the Markdown guides for your own projects.
- Each guide is self-contained and does not require code integration.

## Available Guides

Guides are organized into subject-scoped folders. Copy any guide by its full
path relative to the repository root.

### `code_quality/`

| File | Subject |
| --- | --- |
| `code_quality/HIGH_COHESION_GUIDE.md` | Single-responsibility design principle |
| `code_quality/LOW_COUPLING_GUIDE.md` | Explicit, minimal dependency design |
| `code_quality/REFERENTIAL_TRANSPARENCY.md` | Deterministic, side-effect-contained design |
| `code_quality/CLEAN_ARCHITECTURE_GUIDE.md` | Layered dependency-direction guidance |
| `code_quality/DRY_GUIDE.md` | Single-source-of-truth and duplication-avoidance guidance |
| `code_quality/CODE_QUALITY_CONTROL_GUIDE.md` | Change-quality and boundary review guidance |
| `code_quality/LLM_CONTEXT_GUIDE.md` | Code structure for LLM context window efficiency |
| `code_quality/INTERFACE_FIRST_GUIDE.md` | Contract-first design — define interfaces before implementations |
| `code_quality/NAMING_GUIDE.md` | Naming conventions for symbols, files, and tests |
| `code_quality/ERROR_HANDLING_GUIDE.md` | Error classification, propagation, and contract declaration |
| `code_quality/DEFENSIVE_CODING_GUIDE.md` | Boundary validation, invariant assertions, and fail-fast design |
| `code_quality/OBSERVABILITY_GUIDE.md` | Structured logs, metrics, and distributed traces for production visibility |
| `code_quality/SOLID_GUIDE.md` | Five SOLID design principles with emphasis on OCP, LSP, and ISP |
| `code_quality/INCREMENTAL_CHANGE_GUIDE.md` | Structuring AI-assisted work into single-concern, verifiable changes |
| `code_quality/CLAUDE_CODE_WORKFLOW_GUIDE.md` | Claude Code session discipline — scope, verification, tool-call review, and course correction |
| `code_quality/UNIT_TEST_GUIDE.md` | Fast, isolated, deterministic unit-testing guidance |
| `code_quality/INTEGRATION_TEST_GUIDE.md` | Multi-component, real-boundary integration testing guidance |
| `code_quality/E2E_TEST_GUIDE.md` | Full-stack, user-visible end-to-end testing guidance |

### `domain_specific/`

| File | Subject |
| --- | --- |
| `domain_specific/DOMAIN_DESIGN_CONTROL_GUIDE.md` | Domain model, API, and interface change-review guidance |
| `domain_specific/LIGHTWEIGHT_DDD_GUIDE.md` | Pragmatic Domain-Driven Design guidance |
| `domain_specific/DDD_GUIDE.md` | Full Domain-Driven Design methodology (strategic + tactical) |
| `domain_specific/MOBILE_FIRST_GUIDE.md` | Mobile-first interface design |
| `domain_specific/REST_API_GUIDE.md` | Resource-oriented HTTP API design guidance |
| `domain_specific/NODE_MODULE_GUIDE.md` | Layered Node.js module structure and dependency-direction guidance |

### `frontend/`

| File | Subject |
| --- | --- |
| `frontend/REACT_GUIDE.md` | React component design, state, hooks, and data-flow guidance |

### `meta/`

| File | Subject |
| --- | --- |
| `meta/GUIDE_AUTHORING_GUIDE.md` | How to plan, write, structure, and register a new guide in this library |
