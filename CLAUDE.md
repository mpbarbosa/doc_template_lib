# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Repository Is

`doc_template_lib` is a collection of reusable Markdown guide templates for architecture and AI-assisted development conventions. It contains no source code, no build system, and no test framework. All substantive files are Markdown guides consumed by other projects as reference.

**Current guides:**

| File | Subject |
|------|---------|
| `HIGH_COHESION_GUIDE.md` | Single-responsibility design principle |
| `LIGHTWEIGHT_DDD_GUIDE.md` | Pragmatic Domain-Driven Design guidance |
| `LOW_COUPLING_GUIDE.md` | Explicit, minimal dependency design |
| `MOBILE_FIRST_GUIDE.md` | Mobile-first interface design |
| `REFERENTIAL_TRANSPARENCY.md` | Deterministic, side-effect-contained design |
| `.github/copilot-instructions.md` | Durable Copilot guidance for this repo |

There are no build, lint, or test commands to run.

---

## Document Authoring Conventions

These rules come from `.github/copilot-instructions.md` and are enforced by the ai_workflow.js pipeline:

- **One principle per guide.** Each document has a single clear subject. If a description needs repeated "and", the scope is too broad.
- **Cross-reference, don't duplicate.** Link to the authoritative guide rather than restating it. Treat repeated explanations as a design smell.
- **Write for reuse.** Guides must work as reference material across projects. Avoid embedding repository-local implementation details unless explicitly required.
- **Stay actionable.** Favor rules, signals, heuristics, and checklists over narrative prose. Each section should leave the reader knowing what to do.
- **Mirror existing structure and style** when adding a new guide. Match the heading hierarchy and tone of the existing documents.

---

## AI Workflow Integration

This repository uses [ai_workflow.js](https://github.com/mpbarbosa/ai_workflow.js) for CI/quality runs. Configuration is in `.workflow-config.yaml`.

### Running the workflow

```bash
ai-workflow run --stage quick    # doc review + consistency + config validation
ai-workflow run --stage full     # full pipeline (adds context, markdown lint, summary)
ai-workflow resume               # resume an interrupted run from checkpoint
```

### Key config rules

**Do not add a `steps:` key under `workflow.stages.<stage>`.**

```yaml
# CORRECT — enabled/disabled only
workflow:
  stages:
    quick:
      enabled: true

# WRONG — fails preflight immediately
workflow:
  stages:
    quick:
      enabled: true
      steps: [step_00, step_01, ...]  # ← prohibited
```

To disable a step, set `enabled: false` under `workflow.steps`, not here. The `workflow.steps` section already correctly scopes which steps run for this docs-only project (steps 06–10 and 14–23 are disabled because there is no source code or test framework).

**Locked dependency edges** (cannot be overridden via `dependency_comment`):

- `step_11 ← step_13` (context requires markdown lint)
- `step_0f ← step_17` (artifact commit follows summary)
- `step_12 ← step_0f` (git finalization is last)

The terminal chain `step_17 → step_0f → step_12` must always close the pipeline.

### `step_overrides.step_01.task_scope.eligible_docs`

Controls which documents step_01 (Documentation Updates) reviews. Update this list when adding a new guide so the AI review includes it.

```yaml
step_overrides:
  step_01:
    task_scope:
      eligible_docs:
        - README.md
        - HIGH_COHESION_GUIDE.md
        - LIGHTWEIGHT_DDD_GUIDE.md
        - LOW_COUPLING_GUIDE.md
        - MOBILE_FIRST_GUIDE.md
        - REFERENTIAL_TRANSPARENCY.md
        - .github/copilot-instructions.md
```

---

## Adding a New Guide

1. Create `<NAME>_GUIDE.md` at the repository root.
2. Mirror the structure of an existing guide (goal → definition → why it matters → signals/checklists).
3. Add it to `step_overrides.step_01.task_scope.eligible_docs` in `.workflow-config.yaml`.
4. Cross-reference it from related existing guides rather than duplicating shared content.
5. Run `ai-workflow run --stage quick` to validate consistency and config.
