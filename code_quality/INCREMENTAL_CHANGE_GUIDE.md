# Incremental Change Guide

Incremental change is the practice of structuring AI-assisted development work
into units small enough that each one can be fully understood, reviewed, and
verified before the next begins.

## Goal

Ensure every change to a codebase targets a single behavioral concern,
is verifiable against a declared specification before the next change starts,
and produces a diff that a reviewer can assess independently — so that errors
introduced by AI-generated code are caught at the smallest possible scope.

## What Incremental Change Means

Incremental change means decomposing work before generating any code, so that
each prompt has a single target, each output has a verification condition, and
each step builds on a confirmed foundation.

In practice, it means:

1. Work is decomposed into an ordered sequence of changes before any code is
   generated. The sequence is the plan; generation follows the plan.
2. Each change targets one concern: one interface, one implementation, one
   test, one observability addition.
3. Each change has an explicit "done" condition — a test that passes, a type
   that checks, a contract that is satisfied — before the next change begins.
4. Prompts are scoped to one file or one concern. A prompt that asks for "the
   full feature" produces a diff too large to review correctly.
5. Each verified change is committed before the next begins. An unverified
   change is a hidden dependency for everything that follows it.

It does **not** mean that every trivial edit is a separate commit or that
every function is a separate prompt. The unit of incrementality is a
*behavioral concern* — one interface, one implementation decision, one
error-handling path — not a line count or file count. Related one-liners that
belong to the same concern are one change.

## Why It Matters

1. Reviewer attention is finite. A large AI-generated diff accumulates errors
   that individually would be caught but together exceed the reviewer's capacity
   to verify. Smaller diffs produce more reliable reviews.
2. Errors compound across unverified changes. An incorrect interface,
   generated and accepted without review, propagates incorrect assumptions into
   every implementation, test, and integration that follows.
3. Small changes make regressions locatable. When each change is verified
   before the next begins, a failing test identifies the exact change that
   introduced the failure. In a large unverified batch, the failure could
   originate anywhere.
4. Incremental commits enable surgical rollback. Reverting a single behavioral
   concern is a one-step operation. Reverting part of a large batch requires
   manual surgical editing.
5. The discipline of decomposition before generation produces better prompts.
   A prompt scoped to one concern is more specific, produces less ambiguous
   output, and requires less correction.

## Incremental Change and Code LLMs

Incremental change is not a constraint on LLMs — it is a constraint on the
workflow that uses them. LLMs are capable of generating large, multi-concern
outputs in one shot. The cost of doing so is borne by the reviewer, not the
model.

A single large prompt produces a diff that mixes interface decisions,
implementation choices, error handling, and observability into one unit that
cannot be verified in parts. Breaking the same work into a sequence of focused
prompts produces the same total code with a verification gate after each piece.

### Why Incremental Prompting Benefits LLMs

- A focused prompt produces a more accurate output. An LLM given one specific
  interface to define makes better decisions than one given an entire feature
  to implement.
- Verification feedback after each step corrects the model's direction before
  errors propagate. A correction at the interface stage costs one prompt; the
  same correction after full implementation costs a rewrite.
- Small, consistent diffs are easier to cross-reference against the declared
  contract, making AI review faster and more reliable.
- Established patterns from earlier steps are in context when later steps are
  generated, reducing inconsistency across a feature.

### Where Large-Batch Generation Hurts

- A diff that touches many files for a single logical change makes it
  impossible to verify any one part without understanding all parts.
- Mixed concerns in a single diff — interface, implementation, tests, logging
  — mean no single review criterion covers the whole change.
- An incorrect early decision (wrong interface shape, wrong error type)
  propagates silently through all subsequent generated code in the same batch.
- Large diffs increase the likelihood that a reviewer approves the overall
  shape while missing a specific error in one file.

## Required Rules

1. Work must be decomposed into a named sequence of changes before any
   generation begins.
2. Each change must target one behavioral concern.
3. Each change must have an explicit verification condition before the next
   change starts.
4. Prompts must be scoped to one file or one concern. Cross-concern prompts
   are not permitted.
5. Each verified change must be committed before the next change begins.
6. An interface must be declared and reviewed before its implementation is
   generated.

## The Stacked Change Pattern

The most reliable sequence for any new capability follows four stages. Each
stage has a concrete verification condition.

| Stage | Output | Verification condition |
| --- | --- | --- |
| 1. Interface | Declared types, function signatures, error cases | A reviewer can use the interface correctly without reading any implementation |
| 2. Tests | Tests against the interface — all failing | Tests run and fail for the right reason (no implementation yet) |
| 3. Implementation | Code that satisfies the interface | All declared tests pass; no new warnings |
| 4. Observability | Log entries, metrics, trace spans | Structured signals are emitted on success, failure, and significant decisions |

Do not proceed to stage N+1 until stage N is verified and committed. A test
suite that passes against a wrong interface has verified the wrong thing.

## Best Practices

### Decomposing Before Generating

1. Write the sequence of changes as a list before opening a prompt. Name each
   change as a single sentence: "Define the `OrderRepository` interface with
   `findById` and its error cases."
2. Review the sequence for dependencies. If change 3 requires change 2 to be
   correct, commit change 2 before generating change 3.
3. When a change seems too large to verify as a unit, split it. A change is
   too large when its verification condition requires understanding more than
   one concern.

### Scoping Prompts

1. One file per prompt when possible. A prompt that modifies multiple files
   has multiple concerns.
2. Include the declared interface in the prompt context when generating an
   implementation. The interface is the specification; the prompt is asking
   for conformance.
3. Ask for the interface before the implementation. "Define the interface for
   X" is a better first prompt than "Implement X."
4. Ask for one error case at a time if the full error taxonomy is uncertain.
   It is easier to add an error case incrementally than to remove a wrong one.

### Verification Before Continuation

1. Define "done" for each change before generating it. The done condition is
   the verification gate.
2. Do not treat "the code looks right" as a verification condition. Runnable
   tests, type checks, and contract reviews are verification conditions.
3. If a generated change does not meet its verification condition, correct it
   before continuing. Do not carry a known issue into the next change.
4. Commit each verified change with a message that names the specific concern
   it addresses.

### Prompt Context Management

1. Include only the context the current change needs. Sending the entire
   codebase as context for a focused change dilutes the signal.
2. For implementation prompts: include the interface declaration and the
   failing tests. Exclude unrelated files.
3. For observability prompts: include the implementation and the structured
   log field template. The model can follow the established pattern without
   reading the entire codebase.

## Review Heuristics

### Concern Count Test

How many distinct behavioral concerns does this diff address? If more than one,
the change is not atomic. A change that adds a new interface, its
implementation, its tests, and logging in one diff cannot be reviewed
independently per concern.

### Verification Gate Test

Was there an explicit verification condition for the previous change before
this change was started? If the previous change was generated and immediately
extended without a verification step, errors may have propagated silently.

### Interface-First Test

Was the interface declared and reviewed before the implementation was
generated? A diff that shows an interface and its implementation appearing at
the same commit skipped the interface review stage.

### Commit Granularity Test

Does each commit correspond to one verified behavioral concern? Commits that
mix "add interface, add implementation, fix logging, update tests" are a
signal that changes were not incremental — they were batched and committed
together after the fact.

## Positive Signals

- Each commit message names one specific concern.
- The test suite has a commit where new tests appear and fail, followed by a
  commit where they pass — the interface-test-implementation sequence is
  visible in the log.
- Diffs are reviewable against a single criterion: does this implementation
  conform to this interface? Does this test cover this error case?
- Interface declarations appear in the commit log before their implementations.
- Rollback of any single change does not affect unrelated behavior.

## Warning Signs

- A single prompt that produces a diff touching more than three files for a
  logically single capability.
- Implementation and interface appearing in the same commit with no prior
  interface-only commit.
- Tests written after the implementation rather than against the interface
  first.
- A commit message that requires "and" to describe what it does.
- A review comment on change N that reveals an error introduced in change N-3
  — the verification gate failed.
- AI-generated code that is accepted before its verification condition is met,
  then extended immediately.
- A diff that cannot be summarized in one sentence without "also."

## Related Guides

- [CLAUDE_CODE_WORKFLOW_GUIDE.md](./CLAUDE_CODE_WORKFLOW_GUIDE.md) —
  session-level discipline for the Claude Code sessions that execute incremental
  changes: scope declaration, tool-call review, verification gates, and course
  correction. The session-level complement to this structural guide.
- [INTERFACE_FIRST_GUIDE.md](./INTERFACE_FIRST_GUIDE.md) for the first stage
  of the stacked change pattern — declare the contract before generating any
  implementation.
- [LLM_CONTEXT_GUIDE.md](./LLM_CONTEXT_GUIDE.md) for scoping prompt context
  to the minimum necessary for the current change — focused context produces
  more accurate generation.
- [ERROR_HANDLING_GUIDE.md](./ERROR_HANDLING_GUIDE.md) for declaring error
  cases in the interface stage so they are tested and implemented
  incrementally, not added reactively.
- [OBSERVABILITY_GUIDE.md](./OBSERVABILITY_GUIDE.md) for the fourth stage of
  the stacked change pattern — instrumentation added as a discrete, verifiable
  step after the implementation is confirmed.
- [CODE_QUALITY_CONTROL_GUIDE.md](./CODE_QUALITY_CONTROL_GUIDE.md) for the
  broader quality gates that incremental verification supports at each stage.

## Summary Checklist

- [ ] Work is decomposed into a named sequence of single-concern changes
      before any generation begins.
- [ ] Each change has an explicit verification condition defined before
      generation starts.
- [ ] Prompts are scoped to one file or one concern.
- [ ] The interface is declared and reviewed before its implementation is
      generated.
- [ ] Tests are written against the interface before the implementation
      exists.
- [ ] Each verified change is committed before the next change begins.
- [ ] No change is extended before its verification condition is met.
- [ ] Each commit message names one specific concern without "and."
