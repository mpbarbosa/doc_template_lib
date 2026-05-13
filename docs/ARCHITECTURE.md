# Architecture Overview

## Purpose

doc_template_lib provides reusable Markdown documentation templates for software architecture and AI-assisted development conventions.

## Structure

- All guides are Markdown files at the repository root.
- No source code, build system, or test framework is included.
- `.workflow-config.yaml` configures documentation review and CI.

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
