# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

## [1.5.0] - 2026-06-17
- Added code_quality/SOLID_GUIDE.md — five SOLID principles with full coverage
  of OCP, LSP, and ISP; cross-references for SRP and DIP (Phase 3 of AI code
  quality roadmap)
- Added code_quality/INCREMENTAL_CHANGE_GUIDE.md — stacked change pattern and
  single-concern verification workflow for AI-assisted development
- Cross-referenced new guides from HIGH_COHESION_GUIDE, CODE_QUALITY_CONTROL_GUIDE,
  and INTERFACE_FIRST_GUIDE

## [1.4.0] - 2026-06-17
- Added code_quality/DEFENSIVE_CODING_GUIDE.md — boundary validation,
  invariant assertions, and fail-fast design (Phase 2 of AI code quality
  roadmap)
- Added code_quality/OBSERVABILITY_GUIDE.md — structured logs, metrics, and
  distributed traces for production visibility
- Cross-referenced new guides from LOW_COUPLING_GUIDE and
  INTEGRATION_TEST_GUIDE

## [1.3.0] - 2026-06-17
- Added code_quality/INTERFACE_FIRST_GUIDE.md — contract-first design before
  implementation (Phase 1 of AI-supported code quality roadmap)
- Added code_quality/NAMING_GUIDE.md — naming conventions for symbols, files,
  and tests across all layers
- Added code_quality/ERROR_HANDLING_GUIDE.md — error classification,
  propagation rules, and contract declaration
- Cross-referenced new guides from CLEAN_ARCHITECTURE_GUIDE, UNIT_TEST_GUIDE,
  and REST_API_GUIDE

## [1.2.0] - 2026-06-17
- Added domain_specific/NODE_MODULE_GUIDE.md — layered Node.js module structure
  and dependency-direction guidance (imported and adapted from ibira.js)
- Added code_quality/LLM_CONTEXT_GUIDE.md — code structure for LLM context
  window efficiency
- Updated CLAUDE.md: fixed eligible_docs example (missing REACT_GUIDE.md),
  strengthened Adding a New Guide checklist, registered both new guides
- Updated .workflow-config.yaml: registered new guides in key_docs and
  eligible_docs

## [1.1.0] - 2026-06-16
- Added frontend/REACT_GUIDE.md (new frontend/ folder for framework guides)
- Added UNIT_TEST_GUIDE.md as a reusable unit-testing template.
- Synced guide inventories to include CLEAN_ARCHITECTURE_GUIDE.md and
  CODE_QUALITY_CONTROL_GUIDE.md consistently across repository docs.
- Updated CLAUDE.md: corrected eligible_docs paths and Adding a New Guide instructions
- Added .claude/skills/bump-version-stage-commit-push

## [1.0.0] - 2026-05-13
- Added HIGH_COHESION_GUIDE.md
- Added LIGHTWEIGHT_DDD_GUIDE.md
- Added LOW_COUPLING_GUIDE.md
- Added MOBILE_FIRST_GUIDE.md
- Added REFERENTIAL_TRANSPARENCY.md
- Added .github/copilot-instructions.md
- Added project-level documentation files
