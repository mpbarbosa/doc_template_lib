# CONTEXT.md Format

## Structure

```md
# {Context Name}

{One or two sentence description of what this context is and why it exists.}

## Language

**Guide**:
{A one or two sentence description of the term}
_Avoid_: Document, template, article

**Principle**:
A single design rule that a guide is built around.
_Avoid_: Topic, subject, concept

**Cross-reference**:
A link from one guide to another authoritative guide, used instead of duplicating content.
_Avoid_: Reference, pointer, mention
```

## Rules

- **Be opinionated.** When multiple words exist for the same concept, pick the
  best one and list the others under `_Avoid_`.
- **Keep definitions tight.** One or two sentences max. Define what it IS, not
  what it does.
- **Only include terms specific to this project's context.** General programming
  or writing concepts don't belong even if the project uses them extensively.
  Before adding a term, ask: is this a concept unique to this documentation
  library's design, or a general concept? Only the former belongs.
- **Group terms under subheadings** when natural clusters emerge. If all terms
  belong to a single cohesive area, a flat list is fine.

## Single vs multi-context repos

**Single context (most repos):** One `CONTEXT.md` at the repo root.

**Multiple contexts:** A `CONTEXT-MAP.md` at the repo root lists the contexts,
where they live, and how they relate to each other:

```md
# Context Map

## Contexts

- [Authoring](./docs/authoring/CONTEXT.md) — conventions for writing guides
- [Workflow](./docs/workflow/CONTEXT.md) — AI workflow configuration and steps

## Relationships

- **Authoring → Workflow**: Authoring conventions define what counts as a valid
  guide; Workflow steps enforce those conventions automatically
```

The skill infers which structure applies:

- If `CONTEXT-MAP.md` exists, read it to find contexts.
- If only a root `CONTEXT.md` exists, single context.
- If neither exists, create a root `CONTEXT.md` lazily when the first term is
  resolved.

When multiple contexts exist, infer which one the current topic relates to. If
unclear, ask.
