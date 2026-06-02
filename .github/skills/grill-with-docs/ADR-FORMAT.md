# ADR Format

ADRs live in `docs/adr/` and use sequential numbering: `0001-slug.md`,
`0002-slug.md`, etc.

Create the `docs/adr/` directory lazily — only when the first ADR is needed.

## Template

```md
# {Short title of the decision}

{1-3 sentences: what's the context, what did we decide, and why.}
```

That's it. An ADR can be a single paragraph. The value is in recording *that*
a decision was made and *why* — not in filling out sections.

## Optional sections

Only include these when they add genuine value. Most ADRs won't need them.

- **Status** frontmatter (`proposed | accepted | deprecated | superseded by ADR-NNNN`) — useful when decisions are revisited.
- **Considered Options** — only when the rejected alternatives are worth
  remembering.
- **Consequences** — only when non-obvious downstream effects need to be called
  out.

## Numbering

Scan `docs/adr/` for the highest existing number and increment by one.

## When to offer an ADR

All three of these must be true:

1. **Hard to reverse** — the cost of changing your mind later is meaningful.
2. **Surprising without context** — a future reader will look at the repo and
   wonder "why on earth did they do it this way?"
3. **The result of a real trade-off** — there were genuine alternatives and you
   picked one for specific reasons.

If a decision is easy to reverse, skip it — you'll just reverse it. If it's not
surprising, nobody will wonder why. If there was no real alternative, there's
nothing to record beyond "we did the obvious thing."

### What qualifies

- **Structural conventions.** "Each guide covers exactly one principle — no
  combined guides." "Guides follow a goal → definition → why it matters →
  signals/checklists structure."
- **Naming and layout decisions.** "Guide files are named `<TOPIC>_GUIDE.md`
  at the repo root." "Supporting skill formats live in `.github/skills/`."
- **Cross-reference strategy.** "Guides link to each other rather than
  duplicating content." The explicit no-s are as valuable as the yes-s.
- **Scope boundaries between guides.** "Lightweight DDD covers a pragmatic
  subset; `DDD_GUIDE.md` covers the full methodology. There is no merging."
- **Deliberate deviations from the obvious path.** "We keep both
  `LIGHTWEIGHT_DDD_GUIDE.md` and `DDD_GUIDE.md` rather than one canonical DDD
  guide because..." Anything where a reader would assume the opposite.
- **Constraints not visible in the documents.** "The `eligible_docs` list in
  `.workflow-config.yaml` must be updated manually whenever a new guide is
  added — the workflow does not auto-discover new files."
- **Rejected alternatives when the rejection is non-obvious.** If you
  considered merging related guides and chose to keep them separate for subtle
  reasons, record it — otherwise someone will suggest merging again.
