---
name: grill-with-docs
description: >
  Grilling session that stress-tests a plan against the project's existing
  documentation conventions, sharpens terminology, and updates documentation
  (CONTEXT.md, ADRs) inline as decisions crystallise. Use when asked to "grill"
  a proposal, guide structure, or documentation decision — especially before
  committing to a direction. Adapted from mattpocock/skills for Claude Code and
  GitHub Copilot agents.
---

<what-to-do>

Interview me relentlessly about every aspect of this plan until we reach a
shared understanding. Walk down each branch of the design tree, resolving
dependencies between decisions one-by-one. For each question, provide your
recommended answer.

Ask the questions one at a time, waiting for feedback on each question before
continuing.

If a question can be answered by exploring the repository, explore it instead.

</what-to-do>

<supporting-info>

## Domain awareness

Before the first question, orient yourself:

1. Check for a `CONTEXT.md` at the repo root (or a `CONTEXT-MAP.md` if the
   repo has multiple contexts — see [CONTEXT-FORMAT.md](./CONTEXT-FORMAT.md)).
2. Check `docs/adr/` for existing Architecture Decision Records.
3. Read `CLAUDE.md` and `docs/ARCHITECTURE.md` for the project's stated
   conventions, patterns, and terminology.
4. Scan the guide files at the repo root to understand the established
   documentation conventions and which principles already have authoritative
   guides.

Create files lazily — only when you have something to write. If no `CONTEXT.md`
exists, create one when the first term is resolved. If no `docs/adr/` exists,
create it only when the first ADR is warranted.

## During the session

### Challenge against the glossary

When the user uses a term that conflicts with the existing language in
`CONTEXT.md` (or in `CLAUDE.md` / architecture docs), call it out immediately.
"Your glossary defines 'cohesion' as X, but you seem to mean Y — which
is it?"

### Sharpen fuzzy language

When the user uses vague or overloaded terms, propose a precise canonical term.
"You're saying 'structure' — do you mean the heading hierarchy inside a guide,
or the layout of guides across the repository? Those are different things here."

### Discuss concrete scenarios

When documentation relationships are being discussed, stress-test them with
specific scenarios. Invent edge cases that force the user to be precise about
the scope and boundaries of a guide.

### Cross-reference with guides

When the user states how something should be structured or documented, check
whether the existing guides agree. If you find a contradiction or overlap,
surface it: "You said DDD tactical patterns belong in `LIGHTWEIGHT_DDD_GUIDE.md`,
but `DDD_GUIDE.md` already covers them — which is the authoritative home?"

### Update CONTEXT.md inline

When a term is resolved, update `CONTEXT.md` right there. Don't batch these
up — capture them as they happen. Use the format in
[CONTEXT-FORMAT.md](./CONTEXT-FORMAT.md).

`CONTEXT.md` is a **glossary only**. Keep it devoid of implementation details,
specs, and decisions. Those belong in ADRs or architecture docs.

### Offer ADRs sparingly

Only offer to create an ADR when all three are true:

1. **Hard to reverse** — the cost of changing your mind later is meaningful.
2. **Surprising without context** — a future reader will wonder "why did they
   do it this way?"
3. **The result of a real trade-off** — there were genuine alternatives and you
   picked one for specific reasons.

If any of the three is missing, skip the ADR. Use the format in
[ADR-FORMAT.md](./ADR-FORMAT.md).

### This project's doc locations

| Document | Path | Purpose |
|---|---|---|
| Domain glossary | `CONTEXT.md` (root) | Canonical terms; create lazily |
| Architecture overview | `docs/ARCHITECTURE.md` | Repo structure and design rationale |
| Contributing | `docs/CONTRIBUTING.md` | Contribution guidelines |
| Getting started | `docs/GETTING_STARTED.md` | Quick-start for consumers |
| ADRs | `docs/adr/` | Documentation design decisions; create lazily |
| Agent instructions | `CLAUDE.md` | Codebase conventions for AI agents |

</supporting-info>
