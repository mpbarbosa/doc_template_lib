---
name: bump-version-stage-commit-push
description: >
  Bump the project version, track and stage all intended files, generate an
  appropriate commit message from the staged diff, commit, and push the current
  branch. Use this skill when the user asks for a version bump plus full git
  release flow in one pass.
---

## Overview

This skill performs a lightweight release-style git workflow for this
repository:

1. Bump the version
2. Stage the intended files
3. Validate the repo state
4. Generate a commit message from the staged changes
5. Commit
6. Push the current branch

This is a docs-only repository with no build system. The canonical version
source is `CHANGELOG.md`. The version is also mirrored in `.workflow-config.yaml`
under `project.version` and must be kept in sync.

---

## Canonical version files

| File | Rule |
|------|------|
| `CHANGELOG.md` | Canonical version source — add a dated release section |
| `.workflow-config.yaml` | `project.version` must match the new version |

There is no `package.json`, no npm scripts, and no build step.

---

## Preconditions

Before committing:

1. Confirm the current branch is known.
2. Inspect `git status --short`.
3. Never discard unrelated user changes.

If the user asked to "track the files," stage with:

```bash
git add -A
```

---

## Execution flow

### Step 1 — Inspect repo state

```bash
git status --short
git branch --show-current
git rev-parse --abbrev-ref --symbolic-full-name '@{u}'
```

If upstream is missing, stop and report that push cannot proceed until the
branch has an upstream.

### Step 2 — Determine the bump level

Read the current version from `CHANGELOG.md` (the most recent `## [X.Y.Z]`
heading). Default to a **minor** bump unless the user specifies otherwise:

- `patch` — typo fixes, small corrections to existing guides
- `minor` — new guide added, significant guide revision
- `major` — breaking restructure of the library

### Step 3 — Update CHANGELOG.md

1. Rename the `## [Unreleased]` section to `## [NEW_VERSION] - YYYY-MM-DD`
   using today's date.
2. Add a fresh empty `## [Unreleased]` section above it for future entries.

Example — bumping from `1.0.0` to `1.1.0` on 2026-06-16:

```markdown
## [Unreleased]

## [1.1.0] - 2026-06-16
- Added frontend/REACT_GUIDE.md
- Added .claude/skills/bump-version-stage-commit-push

## [1.0.0] - 2026-05-13
...
```

### Step 4 — Update .workflow-config.yaml

Set `project.version` to the new version string:

```yaml
project:
  version: "1.1.0"
```

### Step 5 — Stage changes

```bash
git add -A
```

### Step 6 — Validate

Run the quick workflow stage to catch doc consistency or config issues:

```bash
ai-workflow run --stage quick
```

If validation fails, fix the issue before committing.

### Step 7 — Review staged scope

```bash
git diff --cached --stat --summary
```

Generate the commit message from the staged diff, not from guesswork.

### Step 8 — Generate commit message

Use a short conventional-style subject that reflects the staged scope.

Examples:

- `chore: bump version to 1.1.0`
- `docs: add REACT_GUIDE and bump to 1.1.0`
- `chore: release 1.2.0`

Heuristics:

- Use `chore:` for version bumps and repo maintenance.
- Use `docs:` when new or revised guides are the primary content of the release.
- Use `feat:` only if this repo gains new tooling or workflow capability.

### Step 9 — Commit

```bash
git commit -m "GENERATED_SUBJECT" -m "Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

### Step 10 — Push

```bash
git push origin "$(git branch --show-current)"
```

---

## Safety rules

- Do **not** amend previous commits unless the user explicitly asks.
- Do **not** use destructive git commands like `reset --hard`.
- Do **not** invent a version string — derive it by incrementing the most
  recent version in `CHANGELOG.md`.
- Do **not** claim success before push completes.
- Pushing to the remote is a shared, hard-to-reverse action — confirm with the
  user before Step 10 unless they have already authorized pushing in this
  conversation.

---

## Recommended one-pass command sequence

```bash
git status --short
# Edit CHANGELOG.md and .workflow-config.yaml with the new version
git add -A
ai-workflow run --stage quick
git diff --cached --stat --summary
git commit -m "GENERATED_SUBJECT" -m "Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
git push origin "$(git branch --show-current)"
```
