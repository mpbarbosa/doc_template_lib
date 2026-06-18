# Architecture Overview

## Purpose

doc_template_lib provides reusable Markdown documentation templates for software architecture and AI-assisted development conventions.

## Structure

Guides are organized into four subject-scoped folders:

| Folder | Subject matter |
| --- | --- |
| `code_quality/` | General programming principles: design patterns, testing, naming, error handling, observability, AI-assisted workflow |
| `domain_specific/` | Domain model, API, and platform topics: DDD, REST, mobile-first, module architecture |
| `frontend/` | UI framework and browser-platform guidance: React, component design, state |
| `meta/` | Guidance about this library itself: authoring conventions, contribution process |

Repository meta-docs (not reusable guides) live in `docs/`. Configuration and
CI tooling live at the repository root alongside `CLAUDE.md` and
`.workflow-config.yaml`.

There is no source code, build system, or test framework. `.workflow-config.yaml`
configures the `ai-workflow` documentation review pipeline.

## Design Principles

- **Single Responsibility:** Each guide covers one principle or subject.
- **Cross-Reference:** Guides link to each other to avoid duplication.
- **Reusability:** Templates are designed for use across multiple projects.

## Integration

- Copy guides into your project as needed.
- Update `.workflow-config.yaml` in your project to include new guides for review.

## Maintenance

- Update guides to reflect evolving best practices.
- Keep documentation actionable and consistent.
