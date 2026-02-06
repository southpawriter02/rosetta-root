You are implementing a specific version spec for the DocStratum project.

## Your Assignment

**Target spec:** [PASTE SPEC PATH, e.g. docs/design/02-foundation/RR-SPEC-v0.1.2-schema-definition.md]

Read the target spec thoroughly before writing any code.

## Context

- Design specs live in `docs/design/`, organized by phase (00-meta through 07-release)
- Spec filenames follow: `RR-SPEC-v{X}.{Y}.{Z}-{slug}.md` (versions) and `RR-SPEC-v{X}.{Y}.{Z}{letter}-{slug}.md` (sub-tasks)
- No implementation code exists yet — you may be creating the first files
- The tech stack is fixed: Python 3.11+, Pydantic, LangChain, LiteLLM, Streamlit, pytest, Black, Ruff. Do not propose alternatives.

## Rules

### Scope

1. **Only implement what the spec describes.** Do not add features, modules, utilities, or abstractions not explicitly documented in the target spec or its sub-documents.
2. **Do not pull work forward from future versions.** If the spec references a capability delivered later (e.g., "v0.3.1 will add streaming"), do not implement it now.
3. **If the spec is ambiguous, ask me.** Do not guess or interpret liberally. Flag the ambiguity and wait for clarification.
4. **Do not modify spec documents.** Specs are read-only. If you think a spec is wrong, tell me — don't edit it.
5. **Do not add dependencies beyond what the spec or existing requirements.txt calls for.** If you believe one is needed, ask first.
6. **Do not refactor existing code unless the spec explicitly calls for it.**

### Quality Standards

7. **Read the engineering standards before writing code.** They are in `docs/design/00-meta/`:
   - `RR-META-testing-standards.md` — pytest, AAA pattern, naming (`test_{method}_{scenario}_{result}`), per-module coverage targets
   - `RR-META-logging-standards.md` — stdlib `logging` only, `%s` formatting, level contract, no print() in src/
   - `RR-META-commenting-standards.md` — Google-style docstrings, type hints on all public signatures, TODO format: `# TODO (vX.Y.Z): description`
   - `RR-META-documentation-requirements.md` — CHANGELOG format, terminology table, writing style
   - `RR-META-development-workflow.md` — commit format, self-review checklist, phase transition rules

8. **Tests are not optional.** Every deliverable includes tests written alongside implementation. Follow AAA pattern (Arrange-Act-Assert).
9. **Logging is not optional.** Every module uses `logging.getLogger(__name__)` and logs key operations at INFO level. No `print()` in source modules.
10. **Docstrings are not optional.** Every public module, class, and function gets a Google-style docstring with Args, Returns, Raises, and Example sections.
11. **Format and lint.** Run `black` and `ruff check` on all changed files before committing.

### Process

12. **Check the spec's Dependencies section** and verify those prerequisites exist before starting.
13. **Work through Acceptance Criteria as a checklist.** Each checkbox is a unit of work. Satisfy them in order.
14. **Commit with version-prefixed messages:** `{type}(v{X}.{Y}.{Z}): imperative description`
    - Types: `feat`, `fix`, `docs`, `test`, `refactor`, `chore`
    - Example: `feat(v0.1.2): implement Pydantic schema for llms.txt`
15. **Create only the files and directories the spec defines.** If the spec says `schemas/llms_schema.py`, create exactly that path.

### Finishing

16. When done, **summarize**: which acceptance criteria are satisfied, which are not (and why), and what the next version should pick up.
17. Run the **self-review checklist** from `RR-META-development-workflow.md` before declaring the version complete.

## Start

Read the target spec now, then read its sub-documents (a, b, c, d) if they exist in the same directory. Tell me your implementation plan before writing any code.
