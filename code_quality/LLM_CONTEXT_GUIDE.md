# LLM Context Efficiency Guide

Code structure determines how much an LLM must read to reason accurately about
any given change. This guide treats the context window as a finite resource and
describes code organization practices that minimize wasted context load.

## Goal

Structure code so that the minimum necessary context answers any question an
LLM is asked about it — reducing wasted tokens, keeping relevant code within
the context window, and making LLM-generated edits more accurate and correctly
scoped.

## What LLM Context Efficiency Means

LLM context efficiency means organizing code so that questions about a module
can be answered from that module alone, patterns are consistent enough that one
example generalizes to all, and names carry enough semantics that bodies are
optional reading.

In practice, it means:

1. Each file has a single, clearly named responsibility — its purpose is
   inferable without reading its body.
2. Import graphs are sparse — understanding one module does not require loading
   many others.
3. Patterns are consistent — the same structural convention appears across all
   modules of the same type.
4. Public names carry their semantics — identifiers describe what they do
   without requiring surrounding context.
5. The public API surface is declared in one place — an LLM can know what is
   exposed without scanning every file.
6. Side effects are isolated and explicit — importing a module does not
   silently consume context on unrelated behavior.

It does **not** mean splitting every file into the smallest possible unit.
Over-granular modules scatter related context across many files, increasing the
number of files an LLM must load to answer a single question. The goal is the
minimum context load per question, not the minimum file size.

## Why It Matters

1. Context windows are finite. Code that forces an LLM to load many files to
   answer a focused question may exceed the window or displace other relevant
   context.
2. LLM accuracy degrades as context fills with irrelevant code. A large
   mixed-concern file forces the model to sift signal from noise.
3. Inconsistent patterns require the LLM to re-learn conventions per instance
   rather than generalizing from one example — consuming context that could
   have been used for the actual task.
4. Poorly named identifiers shift semantic load from names to bodies: the LLM
   must read more to understand less.
5. Dense import graphs create collateral context cost: answering a question
   about module X requires loading modules A, B, and C that X imports.
6. The same properties that reduce context cost for LLMs reduce cognitive load
   for human reviewers working with bounded attention.

## Required Rules

1. Every file must have a single, nameable responsibility. The file name must
   describe that responsibility without ambiguity.
2. Every public identifier — function, class, constant — must be interpretable
   from its name alone, without reading its body.
3. Import graphs must be kept sparse. A module should depend on as few other
   modules as the design allows.
4. All modules of the same type must follow the same structural pattern.
   Pattern divergence within a module category is not permitted.
5. The public API surface of any package or layer must be declared in one
   place.
6. Side effects must not occur at module import time.

## Context Cost Reference

| Code property | Context cost | Why |
| --- | --- | --- |
| Large mixed-concern file (500+ lines) | High | Full file read required for any focused question |
| Small single-concern module | Low | Often answerable from imports and signatures alone |
| Generic name (`util`, `helper`, `manager`) | High | Body read required to determine purpose |
| Precise semantic name (`orderValidator`) | Low | Purpose clear from identifier |
| Dense import graph (10+ dependencies) | High | Many collateral files may enter context |
| Sparse import graph (2–3 dependencies) | Low | Few collateral files pulled in |
| Inconsistent patterns across similar modules | High | Each instance requires separate LLM re-learning |
| Consistent patterns across all similar modules | Low | One example generalizes to the rest |
| Scattered public API (exports spread across files) | High | LLM must scan many files to know what is exposed |
| Single declared public surface | Low | One file answers "what does this expose?" |
| Side effects at import time | High | Hidden behavior introduces surprise context cost |
| Pure modules, side effects isolated | Low | Import cost is predictable and bounded |

## Best Practices

### File Granularity

1. Size a file around a single answerable question: "what does this file do?"
   should have one answer.
2. When a file requires more than one sentence to describe, split it.
3. Do not split so granularly that answering a question requires loading more
   files than a single larger file would have cost. The goal is the minimum
   context per question, not the minimum file size.
4. Name every file after its responsibility. `orderValidator.js` is a context
   unit. `utils.js` is not.

### Naming Density

1. Write names that eliminate the need to read the body.
   `calculateRetryDelay(attempt, baseMs)` is self-describing. `process(n, b)`
   is not.
2. Prefer specific nouns and action verbs over generic ones. `UserRepository`
   is better than `DataManager`. `parseISODate` is better than `formatDate`.
3. Avoid abbreviations that require knowledge of the codebase to decode.
   Abbreviations shift semantic load from the name to the reader's memory.
4. Name boolean-returning functions as predicates: `isExpired()`,
   `hasPermission()` — not `check()` or `validate()`.

### Import Graph Sparsity

1. Count a module's direct imports as its context radius. Each import is a
   file that may enter context alongside the importing module. Prefer fewer.
2. Extract shared logic into a utility that multiple modules import rather than
   having modules import each other — this keeps the graph directed and sparse.
3. Do not import a module for a single small helper that could be inlined or
   extracted to a shared constant.
4. Before adding an import, check whether renaming or restructuring would
   eliminate the need for it.

### Pattern Consistency

1. Define one structural template per module type (service, repository,
   handler, validator) and apply it uniformly. When all validators look the
   same, loading one covers all of them.
2. Codify conventions in a template or style guide file in the repository.
   Consistency that lives only in developers' heads degrades over time and
   cannot be loaded into context.
3. Treat structural divergence as a code smell. A module that breaks the
   established pattern requires extra context to explain itself.
4. Include a representative example of each module type in the codebase so an
   LLM can generate conforming modules without reading every existing one.

### Public API Surface

1. Declare the complete public surface of any package or layer in one
   entry-point file. An LLM should answer "what does this expose?" by reading
   one file.
2. Keep the entry point free of logic — re-exports only.
3. Do not let consumers import from internal paths. Each internal import is an
   undeclared API surface that increases context ambiguity.

## Review Heuristics

### 10-Line Scan Test

Read only a file's import block and its first declaration. Is the file's
complete responsibility clear? If not, the file is not self-declaring and will
require fuller context reads every time an LLM interacts with it.

### Cold Name Test

Given only a symbol's name and its parameter names, can an LLM predict what it
does without reading its body? If the name could describe several different
behaviors, it is not carrying its semantic load and the body will consume
context on every encounter.

### Dependency Radius Test

Count the files that must be loaded to fully understand the behavior of a
single function or class. More than three or four is a signal that coupling is
too dense for efficient context use.

### Pattern Recognition Test

Pick two modules of the same type. Do they follow an identical structural
convention — same sections in the same order, same naming patterns, same export
shape? If an LLM would need to re-learn the pattern for each, consistency is
missing and each module is paying full context cost independently.

### Question Boundary Test

Name a specific question about a module ("what does this validate?", "what does
this return on error?"). How many files must be in context to answer it
correctly? A well-structured module should answer most questions about itself
from itself alone.

## Positive Signals

- Every file name answers "what is this?" without ambiguity.
- Any function's behavior is predictable from its name and parameter names.
- The import list of any module fits on one screen and reveals its full
  dependency surface.
- All modules of the same category follow the same structural template.
- The package entry point answers "what is public?" in its entirety.
- An LLM generating a new module from one example produces a conforming result
  without correction.
- Changing behavior in one module does not pull unrelated modules into context
  for review.

## Warning Signs

- Files named `utils`, `helpers`, `common`, `shared`, or `manager` with no
  qualifying noun.
- Functions named `process`, `handle`, `do`, `run`, or `execute` without a
  specific subject.
- An LLM-generated edit touches files unrelated to the stated task — a sign
  the context radius is too wide.
- Modules of the same type that look structurally different from each other.
- Internal paths appearing in consumer imports.
- Logic inside a package entry point.
- `import` statements that trigger observable behavior (timers, network calls,
  logging).
- A file that takes more than one sentence to describe its responsibility.
- Abbreviations or acronyms in public identifiers that are not universally
  known in the domain.

## Related Guides

- [HIGH_COHESION_GUIDE.md](./HIGH_COHESION_GUIDE.md) for the
  single-responsibility principle that produces low-cost context units.
- [LOW_COUPLING_GUIDE.md](./LOW_COUPLING_GUIDE.md) for sparse import graphs
  that minimize collateral context load.
- [CLEAN_ARCHITECTURE_GUIDE.md](./CLEAN_ARCHITECTURE_GUIDE.md) for layer
  boundaries that limit how far a question's context radius can expand.
- [DRY_GUIDE.md](./DRY_GUIDE.md) for the pattern consistency that enables LLM
  generalization from a single example.
- [NODE_MODULE_GUIDE.md](../domain_specific/NODE_MODULE_GUIDE.md) for
  single-entry-point design that makes the public API surface explicit and
  bounded.

## Summary Checklist

- [ ] Every file name describes its single responsibility without ambiguity.
- [ ] Every public identifier is interpretable from its name alone.
- [ ] Import graphs are sparse; each module has the minimum necessary
      dependencies.
- [ ] All modules of the same type follow an identical structural template.
- [ ] The public API surface is declared in one entry-point file with no logic.
- [ ] No module produces side effects at import time.
- [ ] An LLM generating a new module from one example produces a conforming
      result without correction.
- [ ] Changing one module does not pull unrelated modules into context for
      review.
