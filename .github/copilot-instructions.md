# Copilot Instructions

This file provides durable, high-signal guidance for Copilot-assisted development in this repository.

- **Purpose**: Guide Copilot to make high-quality, context-aware edits. Focus on documentation consistency, actionable architecture principles, and cross-reference hygiene.
- **Scope**: This is a documentation-first repository. Most changes are to Markdown guides and configuration files.
- **Supporting workflow surfaces**: `.workflow-config.yaml` (project-local workflow configuration), `.ai_workflow/` (runtime artifacts, cache, and checkpoints).
- **Reference**: For project overview and details, see [README.md](../README.md).

## Guidance for Copilot

- Each guide should focus on a single principle or subject area. Avoid merging unrelated topics.
- Prefer cross-references between guides over repeated explanations. Treat duplication as a design smell.
- Write documentation to be reusable across projects. Avoid embedding repository-local implementation details unless explicitly required.
- Keep documents actionable: favor rules, signals, heuristics, and checklists over narrative discussion.
- When adding or updating guides, mirror the structure and style of existing documents for consistency.
- Ensure `.workflow-config.yaml` accurately reflects the repository's scope and usage.
