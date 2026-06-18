# Claude Code Workflow Guide

A Claude Code session is most effective when it is treated as a single-concern
unit: one question answered, one behavior added, one fix applied — scoped,
verified, and committed before the next unit begins. This guide covers how to
structure sessions, what to check before approving tool calls, how to correct
course, and what signals indicate a session is working well or going wrong.

For code structure patterns that keep sessions efficient, see
[LLM_CONTEXT_GUIDE.md](./LLM_CONTEXT_GUIDE.md). This guide covers the session
itself, not the code it produces.

## Goal

Produce small, verifiable, independently reviewable diffs — one concern at a
time — with each change confirmed before the next begins.

## What a Claude Code Workflow Session Means

A well-formed session has a declared scope before the first prompt is sent, a
known verification command before the first tool call executes, and a commit
gate before the next session begins. The conversation is the vehicle; the
verified, committed change is the product.

In practice, it means:

1. The session's one concern is named before opening the conversation. It fits
   in a single sentence without "and."
2. The verification command for that concern is known in advance — not
   discovered after the change is generated.
3. Each tool call is read before approving. A file write that touches more
   files than the task requires is a signal to redirect, not to approve.
4. Verification passes before the commit is made. A commit made before
   verification is an unverified foundation for everything that follows.
5. Course corrections are made immediately and reference a specific rule, not
   a vague re-statement of the goal.

A session that drifts across multiple concerns, accumulates unapproved scope,
or defers verification is not an efficient session — it is a large batch of
unreviewed changes with a session wrapper around it.

## Why It Matters

1. Reviewable diffs require reviewable scope. A diff that touches five files
   for a single-concern task cannot be reviewed with confidence in any one of
   them. Small scope enables reliable review.
2. Verification before commit prevents propagation. An incorrect change that
   is committed and extended in the next session cannot be isolated without
   understanding both sessions. An incorrect change caught before commit is
   corrected in one step.
3. Immediate course correction limits waste. Letting Claude continue past an
   out-of-scope addition accumulates unrequested changes that must be undone
   rather than redirected. The cost of one redirect is a sentence; the cost
   of undoing five tool calls is a session.
4. Explicit pre-session scoping produces better prompts. Knowing the one
   concern before writing the prompt forces the specificity that generates
   accurate, narrow output.
5. CLAUDE.md accuracy compounds across sessions. A context file that describes
   a module layout that no longer exists will mislead every future session that
   reads it. Accurate context is a shared infrastructure.

## Before a Session Starts

Answer three questions before opening a conversation:

1. **What is the one concern?** A single file, a single route, a single test,
   a single guide update. Not a file update and a bug fix.
2. **What is the verification command?** Know which command confirms the change
   is correct before starting.
3. **Which context applies?** The per-directory `CLAUDE.md` for the target
   location loads automatically and surfaces the relevant guides and rules.

If you cannot answer all three, the task needs more scoping before starting.

## Prompt Construction

A good prompt includes:

- **Desired behavior, not implementation**: "The function should return an
  error when the input is empty" — not "call `validate()` with these
  arguments."
- **A reference pattern**: paste the header or signature of one existing
  similar function. This anchors the output to established conventions.
- **Explicit scope**: "Add only X. Do not refactor Y. Do not add error
  handling for cases not described here."
- **The guide that applies** (by name): "Following the stacked change pattern
  in INCREMENTAL_CHANGE_GUIDE."

Omitting scope leads to "while I'm in here" additions — refactors, extra
error handling, new abstractions — none of which were asked for.

## Verification by Layer

Run the command that matches the files changed before moving to the next task.
Map each changed layer to its own verification gate:

| Layer | Verification method |
| --- | --- |
| Syntax | Language-specific syntax check (e.g., `bash -n`, `tsc --noEmit`) |
| Lint | Static analysis at the configured severity for that directory |
| Unit tests | Test runner scoped to the changed module |
| Integration tests | Test runner covering the component boundary crossed |
| Contract review | Manual: does the output satisfy the declared interface? |
| End-to-end | Full-stack test or smoke test targeting the changed behavior |

A change is not verified by type-checking or linting alone — run the test
suite for the relevant layer. Define the verification command before
generating the change, not after.

## Reviewing Tool Calls Before Approving

Claude Code shows a description before executing each tool call. Read it.

| Tool call type | What to check |
| --- | --- |
| File read | Is this file in scope for the stated task? |
| File write / edit | Does the description match the one concern? Is the scope narrow? |
| `git add` | Does the staged set match only the intended files? |
| `git commit` | Does the message name one concern without "and"? |
| `git push` | Is this the intended branch? Is now the right time to push? |
| Shell command | Is the command reversible? Does it match the task? |

**Read diffs before approving writes.** A diff that touches many files for a
single-concern task should prompt a redirect, not an approve.

Watch for these additions that were not asked for:
- New error handling for cases the task does not describe
- Refactored surrounding code ("while I'm in here…")
- New abstraction layers or helper functions
- Extra imports or dependencies

If any of these appear, stop and redirect immediately — do not let them
accumulate across multiple tool calls.

## Commit Discipline

Each commit should name one concern. Use the conventional prefix that matches:

| Prefix | Use for |
| --- | --- |
| `feat:` | New user-visible behavior or new shipped capability |
| `fix:` | Bug fix in existing behavior |
| `test:` | Adding or fixing tests |
| `docs:` | Guide or documentation changes only |
| `refactor:` | Structural change with no behavioral change |
| `chore:` | Version bumps, CI config, repo operations |
| `ci:` | CI/CD workflow changes |

A commit message requiring "and" to describe its scope is two commits.

## Course Correction

When Claude produces something outside the stated scope, correct immediately —
do not let a wrong direction continue across multiple tool calls.

Effective corrections are specific:

- "Do not add error handling for that case here — it is handled by the caller.
  Remove those lines and try again."
- "That refactors `X` which is outside scope. Revert that change and only
  add the new file."
- "The commit message says 'and'. Split this into two separate commits: one
  for the feature, one for the cleanup."

Vague corrections ("that's wrong, try again") produce vague re-attempts.
Reference the specific guide or rule that was violated when possible.

## Keeping CLAUDE.md Files Accurate

The per-directory `CLAUDE.md` files are the primary context for each layer.
If they describe a pattern that no longer exists, future sessions will follow
the wrong model.

Review `CLAUDE.md` files whenever:
- A refactor changes module boundaries or introduces a new layer.
- A new pattern type is added that differs from existing patterns in that
  directory.
- A component is extracted, merged, or renamed.
- New guides are imported that replace or extend an existing referenced guide.

CLAUDE.md inaccuracy is silent — no lint catches it. Treat it as
infrastructure: accurate context is a shared resource that degrades quietly
if not maintained.

## Session Anti-Patterns

### The "while I'm in here" drift

Claude refactors adjacent code, adds error handling for uncovered cases, or
cleans up style — none of which were asked for. Approve only what was
requested. Redirect with an explicit scope statement.

### The growing session

A single conversation accumulates many file changes across multiple concerns.
Each new task in the same session inherits context from the previous ones,
making it harder to isolate failures. Use separate sessions for separate
concerns, or commit and verify between tasks within the same session.

### Skipping verification

Approving a commit before running the relevant verification command. If the
verification fails after commit, the fix becomes a separate commit and the
history becomes noisy. Verify first, commit after.

### Approving a broad `git add` blindly

Staging everything at once includes generated files, credentials, and
unrelated work-in-progress. Always confirm `git status` before committing to
verify what will be included.

### Over-specifying implementation

Prompts that dictate exact function names, variable names, and call order
produce output that looks like what was asked for but may not be correct.
Describe behavior; let Claude choose implementation. Review the diff to
confirm the behavior is right.

## Positive Signals

- Each session addresses one concern with one verification gate.
- Diffs are readable in under 30 seconds.
- Verification passes before the commit is made.
- Commit messages name one concern without "and."
- CLAUDE.md files are updated when architecture changes.
- Course corrections are made immediately and reference a specific rule.
- The session ends with a verified, committed change.

## Warning Signs

- A diff touching many files for a stated single-concern task.
- A commit message with "and" in it.
- Verification run after the commit rather than before.
- A session that continues past the point where Claude produced something
  outside scope, without redirecting.
- CLAUDE.md files describing a module layout that was refactored or renamed.
- A prompt that describes implementation details rather than desired behavior.

## Related Guides

- [INCREMENTAL_CHANGE_GUIDE.md](./INCREMENTAL_CHANGE_GUIDE.md) — stacked
  change pattern and verification conditions for each stage; the structural
  complement to this session-level guide.
- [LLM_CONTEXT_GUIDE.md](./LLM_CONTEXT_GUIDE.md) — structure code so sessions
  load less context per change, reducing the surface area a session must
  reason about.
- [INTERFACE_FIRST_GUIDE.md](./INTERFACE_FIRST_GUIDE.md) — declare the
  contract before generating any implementation; the first stage of any
  well-formed session.
- [CODE_QUALITY_CONTROL_GUIDE.md](./CODE_QUALITY_CONTROL_GUIDE.md) — review
  expectations for what constitutes a complete, correct change; applies at
  the verification gate of each session.
- [NAMING_GUIDE.md](./NAMING_GUIDE.md) — naming rules that make prompts and
  diffs unambiguous; reduces correction loops caused by ambiguous identifiers.

## Summary Checklist

- [ ] The session's one concern is defined before starting.
- [ ] The verification command for that concern is known before starting.
- [ ] The relevant `CLAUDE.md` context has been confirmed accurate.
- [ ] Every file write is reviewed before approving.
- [ ] Out-of-scope additions are redirected immediately with a specific rule.
- [ ] Verification passes before committing.
- [ ] Commit message names one concern without "and."
- [ ] CLAUDE.md files are updated if architecture changed during this session.
