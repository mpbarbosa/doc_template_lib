# Guide Authoring Guide

A guide in this library is a reusable Markdown document that captures one
principle, practice, or method in a form that any project can adopt. This
guide describes how to plan, write, structure, and register a new guide so it
remains consistent with the rest of the library and useful to its consumers.

## Goal

Produce a guide that conveys one principle, stays actionable, cross-references
rather than duplicates, and integrates cleanly with the library so future
authors and AI tools can navigate and extend it without ambiguity.

## What a Well-Formed Guide Is

A well-formed guide is a single-subject reference document. It answers one
question — "how should we approach X?" — with enough specificity that a
reader knows what to do, what signals indicate they are doing it correctly,
and what to avoid.

In practice, a well-formed guide:

1. Has a subject that fits in a phrase without "and." If the title requires
   "and," the scope is two guides.
2. Leads with a goal that a reader can verify against the final checklist.
3. Defines its subject before defending it. Readers need the definition to
   evaluate the reasoning.
4. Favors rules, signals, heuristics, and checklists over narrative prose.
   Each section should leave the reader knowing what to do.
5. Links to authoritative guides rather than restating their content. Repeated
   explanations across guides diverge over time and mislead readers.
6. Is written to work across projects. A guide that embeds project-specific
   paths, tool versions, or team names is a local document, not a template.

## Why It Matters

1. Consistency across guides makes the library navigable. A reader who has
   used one guide should be able to orient immediately in any other. Structural
   drift makes every guide a new learning experience.
2. Single-subject scope keeps guides maintainable. A guide that covers two
   principles has two reasons to change. When either principle evolves, the
   whole document is in flux.
3. Cross-referencing over duplication keeps the library accurate. Duplicated
   explanations diverge. A single authoritative source can be updated in one
   place.
4. Actionable writing reduces interpretation overhead. A guide that tells the
   reader what to do in concrete terms requires less re-reading than one that
   argues a position without resolving it into decisions.
5. Project-agnostic scope maximizes reuse. A guide with embedded project
   details cannot be adopted by another team without editing. A guide with
   principles and patterns can be imported unchanged.

## Folder Structure

Place the new guide in the folder that best matches its subject:

| Folder | Subject matter |
| --- | --- |
| `code_quality/` | General programming principles: design patterns, testing, naming, error handling, observability, AI-assisted workflow |
| `domain_specific/` | Domain model, API, and platform topics: DDD, REST, mobile-first, module architecture |
| `frontend/` | UI framework and browser-platform guidance: React, component design, state |
| `meta/` | Guidance about this library itself: authoring conventions, contribution process |

If no existing folder fits, create a new one with a lowercase, hyphen-free
name that describes the category. Add the new folder to this table and to
`docs/ARCHITECTURE.md`.

## Guide Structure

Follow this section order. Every guide must have all starred sections (\*);
others are included when they add value.

| Order | Section heading | Required |
| --- | --- | --- |
| 1 | H1 title (e.g., `# Naming Guide`) | \* |
| 2 | Opening paragraph — one or two sentences stating what the guide covers and why | \* |
| 3 | `## Goal` — one paragraph, the outcome a reader should achieve | \* |
| 4 | `## What [Subject] Is` (or `## What [Subject] Means`) — definition before defense | \* |
| 5 | `## Why It Matters` — numbered list, reasons the principle is worth following | \* |
| 6 | Topic sections — H2 headings, H3 sub-headings for groupings | as needed |
| 7 | `## Required Rules` — numbered, non-negotiable constraints | when rules are strict |
| 8 | `## Best Practices` — grouped under H3, practical guidance | recommended |
| 9 | `## Review Heuristics` — named tests a reviewer can apply to a diff or design | recommended |
| 10 | `## Positive Signals` — bulleted list, what correct application looks like | \* |
| 11 | `## Warning Signs` — bulleted list, what incorrect application looks like | \* |
| 12 | `## Related Guides` — linked list with one-line descriptions of the relationship | \* |
| 13 | `## Summary Checklist` — checkboxes, one per decision a reader must make | \* |

Do not add sections that are not in this list without a clear reason. Extra
sections that restate content elsewhere in the guide are a scope signal.

## Writing Style

**Be specific over general.** "Validate at system boundaries" is actionable.
"Consider validation" is not.

**Use present tense and imperative mood** for rules and checklists: "Define
the interface before the implementation." Not "interfaces should be defined."

**Avoid narrative justification in rule statements.** Put the reasoning in
"Why It Matters." Rules read as rules; reasoning reads as reasoning. Mixing
them produces neither.

**Tables for structured comparisons.** Use a table when you are listing
items with consistent attributes (e.g., layer / verification method). Use a
bulleted list when items are homogeneous and attributes vary.

**One sentence per checklist item.** A checklist item that needs qualification
is two items or a rule statement with an exception — resolve it before writing
the checklist.

**No project-local details.** No file paths, tool version numbers, or team
names. If an example is needed, use a generic placeholder (`<module>`,
`<project-root>`).

**Cross-reference by filename.** Use the Markdown link form
`[GUIDE_NAME.md](./GUIDE_NAME.md)` with a one-line description of the
relationship. Do not paraphrase the content of the linked guide.

## Required Rules

1. One guide, one subject. A guide whose title requires "and" is two guides.
2. Define before defending. The definition section comes before "Why It
   Matters."
3. Cross-reference, do not duplicate. If content belongs to another guide,
   link to it.
4. Write for reuse. No project-specific paths, tool names, or team references.
5. Follow the canonical section order. Do not reorder or rename required
   sections.
6. Register the guide in all four locations before the work is complete (see
   "Adding a New Guide" below).

## Adding a New Guide

Complete all four registration steps before considering the guide done.

### 1. Create the file

Name the file `<SUBJECT>_GUIDE.md` in uppercase, in the appropriate folder.
Follow the canonical section order. Mirror the heading style and section
depth of an existing guide in the same folder.

### 2. Update `CLAUDE.md`

Add one row to the guide table under "What This Repository Is":

```markdown
| `<folder>/<NAME>_GUIDE.md` | One-line subject description |
```

Add the same path to `step_overrides.step_01.task_scope.eligible_docs`. Both
lists must stay in sync — the table is for human navigation, the YAML list is
for the AI review pipeline.

### 3. Update `.workflow-config.yaml`

Add the path under `step_overrides.step_01.task_scope.eligible_docs`:

```yaml
        - <folder>/<NAME>_GUIDE.md
```

Bump the `project.version` field (minor increment for a new guide).

### 4. Update `CHANGELOG.md`

Add an entry under `## [Unreleased]` or a new version block:

```markdown
## [X.Y.0] - YYYY-MM-DD
- Added <folder>/<NAME>_GUIDE.md — one-sentence description of the guide's
  subject and its relationship to adjacent guides
```

### 5. Cross-reference from related guides

Identify one to three existing guides whose "Related Guides" section should
link to the new guide. Add a link entry to each. Cross-references are
bidirectional: if guide A references guide B, guide B should reference guide A
where the relationship is non-trivial.

## Review Heuristics

### Single-Subject Test

Can the guide's subject be stated in a phrase without "and"? If not, split
the guide before proceeding.

### Definition-First Test

Does the "What [Subject] Is" section appear before "Why It Matters"? A guide
that defends a principle before defining it forces the reader to accept the
reasoning before they understand the claim.

### Duplication Test

Does any section explain something that is the primary subject of another
guide? If yes, replace the explanation with a cross-reference link.

### Actionability Test

Can a reader complete the Summary Checklist without re-reading the guide?
If the checklist items are too abstract to act on, the guide's rules and
practices need more specificity.

### Reusability Test

Does any sentence name a project path, a specific tool version, or a team
name? If yes, generalize or remove it.

### Registration Test

Is the guide registered in `CLAUDE.md` (both table and eligible_docs),
`.workflow-config.yaml`, and `CHANGELOG.md`? A guide that is not registered
will not be included in AI review pipelines.

## Positive Signals

- The guide title is a phrase without "and."
- The Summary Checklist maps directly to decisions made in the guide body.
- No section restates content that is the primary subject of another guide.
- Every cross-reference names the guide by filename and describes the
  relationship in one line.
- The guide works as a standalone reference without reading any project's
  local CLAUDE.md.
- Positive Signals and Warning Signs are concrete enough that a reviewer can
  apply them to a real diff or document without interpretation.

## Warning Signs

- The guide title contains "and" or covers two principles that could each
  stand alone.
- "Why It Matters" appears before the definition of the subject.
- A section explains a concept whose authoritative home is another guide.
- The Summary Checklist contains items that cannot be verified without reading
  the guide body.
- The guide references a specific project path, tool version, or team name.
- The guide is not listed in `CLAUDE.md`, `.workflow-config.yaml`, or
  `CHANGELOG.md`.
- The "Related Guides" section is empty, or links without describing the
  relationship.

## Related Guides

- [INCREMENTAL_CHANGE_GUIDE.md](../code_quality/INCREMENTAL_CHANGE_GUIDE.md)
  — apply the same single-concern, verify-before-continuing discipline when
  adding a new guide: draft, register, cross-reference as three distinct steps.
- [CLAUDE_CODE_WORKFLOW_GUIDE.md](../code_quality/CLAUDE_CODE_WORKFLOW_GUIDE.md)
  — session discipline for the AI-assisted writing session that produces the
  guide; prompt construction and course-correction patterns apply directly.
- [LLM_CONTEXT_GUIDE.md](../code_quality/LLM_CONTEXT_GUIDE.md) — structure
  the guide so AI tools reading it as context can locate the relevant section
  without loading the entire file.
- [DRY_GUIDE.md](../code_quality/DRY_GUIDE.md) — the cross-referencing rule
  (link, don't duplicate) is an application of the single-source-of-truth
  principle documented there.

## Summary Checklist

- [ ] The guide subject fits in a phrase without "and."
- [ ] "What [Subject] Is" appears before "Why It Matters."
- [ ] All required sections are present in canonical order.
- [ ] No section duplicates content whose authoritative home is another guide.
- [ ] Every cross-reference names the guide by filename and describes the
      relationship in one line.
- [ ] No project-specific paths, tool versions, or team names appear.
- [ ] The guide is added to the table in `CLAUDE.md`.
- [ ] The guide path is added to `eligible_docs` in `CLAUDE.md`.
- [ ] The guide path is added to `eligible_docs` in `.workflow-config.yaml`.
- [ ] `project.version` is bumped in `.workflow-config.yaml`.
- [ ] An entry is added to `CHANGELOG.md`.
- [ ] Related existing guides have been updated with a cross-reference back
      to this guide.
