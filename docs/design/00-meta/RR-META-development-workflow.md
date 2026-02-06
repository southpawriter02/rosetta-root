# Development Workflow Standards — DocStratum

<aside>

**Scope:** All phases (v0.1.x through v0.6.x)

**Status:** Active

**Applies To:** All development activities, AI coding sessions, version transitions, and contributions

**Deliverable:** Enforceable development lifecycle rules, AI session protocols, phase transition criteria, and contribution standards for spec-first development

</aside>

---

## Purpose

This document establishes the development workflow for the DocStratum project. It codifies the spec-first methodology, defines how AI coding sessions operate within the version structure, and provides guardrails that prevent premature implementation, skipped phases, or context loss between sessions.

---

## Development Philosophy

### Core Principles

1. **Spec first, code second.** Every implementation begins by reading the spec for the target version.
2. **Phases are gates, not suggestions.** A phase is not complete until all exit criteria are satisfied.
3. **Versions are sequential.** v0.1.1 must be complete before v0.1.2 begins.
4. **Acceptance criteria are contracts.** A version is "done" only when every checkbox is checked.
5. **Small, verifiable steps.** Each version produces a specific deliverable that can be independently validated.
6. **Documentation is a deliverable.** Documentation tasks are part of each version, not deferred.
7. **AI sessions are stateless.** Every session starts from scratch; context must be recoverable from the repo.
8. **The repo is the single source of truth.** If it's not committed, it doesn't exist.
9. **Ask before deviating.** Any change to scope, structure, or specs requires explicit user approval.
10. **Quality gates prevent debt.** Do not advance to the next version with known defects in the current one.

---

## Spec-First Development Methodology

### What Is a Spec?

A spec is a version directory in `docs/{phase-dir}/` containing:

- **README.md** — Phase overview with objective and acceptance criteria
- **RR-SPEC-v{X}.{Y}.{Z}-{slug}.md** — Version spec with detailed requirements
- **RR-SPEC-v{X}.{Y}.{Z}{letter}-{slug}.md** — Sub-document specs for ordered sub-tasks

Every spec includes these standard sections:

| Section | Purpose |
|---------|---------|
| Objective | What this version accomplishes and why |
| Acceptance Criteria | Checkbox list of verifiable completion criteria |
| Limitations & Constraints | Known boundaries |
| Dependencies | What must be completed first |
| Troubleshooting | Common issues with solutions |
| User Story | Who benefits and how |
| Inputs from Previous Sub-Parts | What this version receives |
| Outputs to Next Sub-Part | What this version produces |
| Decision Log | Architectural choices made |

### The Spec-to-Code Workflow

```
┌──────────────────────────────────────────────────┐
│ 1. Read Phase Overview                           │
│    docs/{phase-dir}/README.md                    │
└──────────────────────┬───────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────┐
│ 2. Read Version Spec                             │
│    docs/{phase-dir}/RR-SPEC-v{X}.{Y}.{Z}-*.md   │
└──────────────────────┬───────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────┐
│ 3. Read Sub-Documents (a, b, c, d)               │
│    RR-SPEC-v{X}.{Y}.{Z}{letter}-*.md            │
└──────────────────────┬───────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────┐
│ 4. Identify Acceptance Criteria                  │
│    (these define "done")                          │
└──────────────────────┬───────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────┐
│ 5. Check "Inputs from Previous Sub-Parts"        │
│    (verify prerequisites are complete)            │
└──────────────────────┬───────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────┐
│ 6. Implement to Satisfy Criteria                 │
└──────────────────────┬───────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────┐
│ 7. Verify Each Criterion                         │
│    (check every box or document why not)          │
└──────────────────────┬───────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────┐
│ 8. Commit with Version Reference                 │
│    feat(v0.1.2): implement Pydantic schema       │
└──────────────────────────────────────────────────┘
```

---

## Development Lifecycle

Every unit of work follows this sequence:

```
 1. Identify target version ─────────────────────────────────────────
         │
         ▼
 2. Read the spec (README.md + sub-documents) ───────────────────────
         │
         ▼
 3. Check "Inputs from Previous Sub-Parts" ──────────────────────────
    (verify prerequisites are complete)
         │
         ▼
 4. Implement the deliverable ───────────────────────────────────────
         │
         ▼
 5. Write tests (per RR-META-testing-standards.md) ──────────────────
         │
         ▼
 6. Add logging (per RR-META-logging-standards.md) ──────────────────
         │
         ▼
 7. Add docstrings and comments (per RR-META-commenting-standards.md)
         │
         ▼
 8. Run test suite: pytest --cov=src --cov=schemas ──────────────────
         │
         ▼
 9. Check ALL acceptance criteria (every box must be checkable) ─────
         │
         ▼
10. Update CHANGELOG.md ─────────────────────────────────────────────
         │
         ▼
11. Commit with version-prefixed message ────────────────────────────
         │
         ▼
12. Check "Outputs to Next Sub-Part" ────────────────────────────────
    (confirm downstream dependencies are satisfied)
```

Steps 5-7 happen alongside step 4, not after. Tests, logging, and documentation are part of implementation, not a separate phase.

---

## Phase Transition Rules

### Phase Exit Criteria

| Phase | Directory | Version Range | Key Gates |
|-------|-----------|---------------|-----------|
| 0: Research | 01-research/ | v0.0.x | Spec deep dive, wild examples audit, ecosystem survey, best practices, requirements defined |
| 1: Foundation | 02-foundation/ | v0.1.x | Python environment configured, Pydantic schema defined, sample data created |
| 2: Data Preparation | 03-data-preparation/ | v0.2.x | Source audit complete, concepts extracted, YAML authored, validation pipeline working |
| 3: Logic Core | 04-logic-core/ | v0.3.x | Loader, context builder, agents implemented and tested |
| 4: Demo Layer | 05-demo-layer/ | v0.4.x | Streamlit UI working, side-by-side view, metrics display |
| 5: Testing | 06-testing/ | v0.5.x | Test execution, evidence capture, metrics analysis |
| 6: Release | 07-release/ | v0.6.x | README finalized, docs folder organized, code cleanup, demo recording |

### Transition Rules

1. All acceptance criteria in the **phase overview** must be checked
2. All acceptance criteria in **every version within the phase** must be checked
3. The user **explicitly confirms** phase transition — AI does not auto-advance
4. If a criterion cannot be met, document the reason in the Decision Log and get user approval to proceed
5. Update `CLAUDE.md` "Current Phase" and "Active Version" when transitioning

---

## Version Numbering Rules

### Format

```
v{major}.{minor}.{patch}

major = 0 (pre-release, always 0 until v1.0.0 release)
minor = phase number (0 = Research, 1 = Foundation, 2 = Data Prep, etc.)
patch = sequential version within the phase (0 = overview, 1+ = work items)
```

### Sub-Document Lettering

Within a patch version, sub-documents are lettered a, b, c, d and represent ordered sub-tasks:

```
RR-SPEC-v0.1.2-schema-definition.md          # Version overview
RR-SPEC-v0.1.2a-core-models.md               # Sub-task a
RR-SPEC-v0.1.2b-validation-rules.md          # Sub-task b
RR-SPEC-v0.1.2c-example-data.md              # Sub-task c
RR-SPEC-v0.1.2d-test-coverage.md             # Sub-task d
```

### Rules

- The `.0` patch (v0.0.0, v0.1.0, etc.) is always the phase overview, never implementation
- Sub-documents are completed in alphabetical order (a before b before c)
- Version numbers in the directory structure match commit message references
- Do not create version directories that aren't in the roadmap without user approval

---

## AI Session Workflow

### Starting a Session

```
┌───────────────────────────────────────────────────────┐
│ 1. Read CLAUDE.md (automatic for Claude Code)         │
│    → Note "Current Phase" and "Active Version"        │
├───────────────────────────────────────────────────────┤
│ 2. Read the spec for the active version               │
│    docs/{phase-dir}/RR-SPEC-v{X}.{Y}.{Z}-*.md        │
├───────────────────────────────────────────────────────┤
│ 3. Read sub-documents if they exist                   │
│    (a, b, c, d in the version directory)              │
├───────────────────────────────────────────────────────┤
│ 4. Check "Inputs from Previous Sub-Parts"             │
│    (are prerequisites complete?)                      │
├───────────────────────────────────────────────────────┤
│ 5. Check acceptance criteria                          │
│    (which boxes are already checked?)                 │
├───────────────────────────────────────────────────────┤
│ 6. Begin work on the first unchecked criterion        │
└───────────────────────────────────────────────────────┘
```

### During a Session

- Work on **one version at a time**
- Implement, test, and document in the same session when possible
- Do not start a new version until the current one's acceptance criteria are all met
- If a task requires information from another spec, read that spec
- Ask the user before making architectural decisions not covered by the spec
- Commit at logical checkpoints — not just at the end

### Ending a Session

1. **Summarize** what was accomplished (which acceptance criteria were satisfied)
2. **Note** any uncompleted criteria and blockers
3. **Suggest** the next action for the following session
4. **Commit** all work with appropriate messages
5. If `CLAUDE.md` "Active Version" needs updating, tell the user

---

## Session Handoff Protocol

### The Repo Is the Handoff Document

Context between sessions is preserved through committed artifacts, not memory:

| Mechanism | Purpose | Persistence |
|-----------|---------|-------------|
| `CLAUDE.md` "Active Version" field | Which version to work on next | Permanent until updated |
| Acceptance criteria checkboxes in specs | What's done vs. remaining | Permanent |
| `CHANGELOG.md` | What shipped in each version | Permanent |
| Commit history | Detailed record of all changes | Permanent |
| Claude Code auto-memory | Session notes and patterns | Persistent across sessions |

### What the User Updates Between Sessions

The only thing the user needs to maintain:

1. `CLAUDE.md` → Update "Current Phase" and "Active Version" when transitioning
2. Review Claude Code auto-memory for accuracy

Everything else is tracked automatically through specs and commits.

---

## Branching and Commit Conventions

### Branch Strategy

| Context | Branch | Example |
|---------|--------|---------|
| Solo development (Phase 1-2) | `main` | Work directly on main |
| Feature development (Phase 3+) | `phase-{N}/v{X}.{Y}.{Z}-brief` | `phase-3/v0.3.1-loader-module` |
| Bugfix | `fix/v{X}.{Y}.{Z}-brief` | `fix/v0.3.1-parse-boundary-error` |

### Commit Message Format

```
{type}(v{X}.{Y}.{Z}): imperative description

Optional body with details:
- Detail bullet 1
- Detail bullet 2
```

### Commit Types

| Type | When to Use | Example |
|------|------------|---------|
| `feat` | New functionality | `feat(v0.1.2): implement Pydantic schema for llms.txt` |
| `fix` | Bug fix | `fix(v0.3.1): correct section boundary detection` |
| `docs` | Documentation only | `docs(v0.0.1a): add formal grammar analysis` |
| `test` | Test additions/changes | `test(v0.3.1): add unit tests for Loader` |
| `refactor` | Code restructuring (no behavior change) | `refactor(v0.3.0): extract token estimation into helper` |
| `chore` | Build, config, tooling | `chore(v0.1.1): create project scaffolding` |

### Commit Frequency

- At least once per sub-document completion (a, b, c, d)
- At logical checkpoints within large sub-documents
- Never with empty or generic messages ("update", "fix", "WIP", "stuff")

---

## Self-Review Checklist

Run this checklist before considering any version complete.

### Code Quality

- [ ] All new/changed functions have Google-style docstrings
- [ ] All public function signatures have type hints
- [ ] No `print()` calls in `src/` or `schemas/` modules (use `logging`)
- [ ] No commented-out code blocks in `src/`
- [ ] No hardcoded file paths or API keys
- [ ] `# TODO` comments include version reference: `# TODO (vX.Y.Z): description`
- [ ] Black formatting applied: `black src/ schemas/ tests/`
- [ ] Ruff linting passes: `ruff check src/ schemas/ tests/`

### Testing

- [ ] All existing tests pass: `pytest`
- [ ] New tests written for new/changed code
- [ ] Test names follow `test_{method}_{scenario}_{result}` convention
- [ ] Coverage has not decreased: `pytest --cov=src --cov=schemas`

### Documentation

- [ ] Module docstrings present on all new `.py` files
- [ ] CHANGELOG.md updated with version entries
- [ ] Acceptance criteria reviewed — all boxes checked

### Git

- [ ] Commit message uses version prefix: `{type}(v{X}.{Y}.{Z}): description`
- [ ] No `.env` or credential files staged
- [ ] All changes are committed (no uncommitted work left behind)

---

## What to Do When Stuck

### Decision Tree

```
┌────────────────────────────────────┐
│ What's blocking you?               │
└────────────┬───────────────────────┘
             │
     ┌───────┼───────┬───────────────┐
     ▼       ▼       ▼               ▼
  Spec is  Previous  External       Don't know
  unclear  version   dependency     where to
           missing   broken         start
     │       │       │               │
     ▼       ▼       ▼               ▼
  Re-read  Go back  Document      Read CLAUDE.md
  Objective & complete issue in   "Active Version"
  User Story it first Troubleshoot then read
  sections            section      that spec
     │       │       │               │
     ▼       ▼       ▼               ▼
  Still     Still    Still          Still
  stuck?    stuck?   stuck?         stuck?
     │       │       │               │
     └───────┴───────┴───────────────┘
                     │
                     ▼
          ┌──────────────────────┐
          │ Ask the user.        │
          │ Document the blocker │
          │ in the Decision Log. │
          └──────────────────────┘
```

### Specific Remedies

| Situation | Action |
|-----------|--------|
| Spec is ambiguous | Re-read the Objective and User Story sections. If still unclear, ask the user. |
| Prerequisite from previous version is missing | Go back and complete it before proceeding. |
| An acceptance criterion seems impossible | Document why, propose an alternative, get user approval. |
| External dependency (API, library) is broken | Check the spec's Troubleshooting section. If no entry, add one. |
| Test suite is failing | Fix failing tests before writing new code. |
| Don't know which version to work on | Check CLAUDE.md "Current Phase" and "Active Version". |
| Session is ending with work in progress | Commit what you have, summarize status, note next step. |

---

## Dos and Don'ts

### Do

- Read the spec before writing any code
- Follow acceptance criteria as your definition of done
- Ask the user before deviating from the spec
- Commit work at logical checkpoints with version-prefixed messages
- Write tests alongside implementation, not after
- Use existing patterns from the codebase as templates
- Check "Inputs from Previous Sub-Parts" before starting a version
- Update CHANGELOG.md with every version's deliverables
- Keep CLAUDE.md "Current Phase" and "Active Version" current
- Reference version numbers in TODO comments, commit messages, and log messages
- End sessions with a clear summary and next-step recommendation

### Don't

- Implement features from a future phase
- Create files outside the established directory structure
- Modify spec documents without user approval
- Skip acceptance criteria or mark them done prematurely
- Add dependencies not in requirements.txt without asking
- Make architectural decisions not covered by the spec without asking
- Leave uncommitted work at session end without documenting it
- Use generic commit messages ("update", "fix stuff")
- Assume context from a previous session — re-read the spec
- Ignore test failures and proceed to the next version
- Auto-advance to the next phase without explicit user confirmation
- Create version directories not in the roadmap without approval

---

## Acceptance Criteria (for this document)

- [ ] Spec-first methodology defined with workflow diagram
- [ ] Development lifecycle steps documented (12 steps from read-spec through commit)
- [ ] Phase transition rules with exit criteria for all 7 phases
- [ ] AI session workflow covers start, during, and end protocols
- [ ] Session handoff protocol uses CLAUDE.md as primary mechanism
- [ ] Branching and commit conventions with type prefixes and examples
- [ ] Self-review checklist covers code quality, testing, documentation, and git
- [ ] Version numbering rules align with directory structure and file naming convention
- [ ] "What to do when stuck" section provides actionable guidance with decision tree
- [ ] Dos and Don'ts are specific to AI-assisted, spec-first development
- [ ] All five standards documents cross-reference each other

---

## Related Documents

- [Testing Standards](RR-META-testing-standards.md) — pytest, coverage, fixtures, markers
- [Logging Standards](RR-META-logging-standards.md) — Logging framework, levels, patterns
- [Commenting Standards](RR-META-commenting-standards.md) — Docstrings, type hints, TODOs
- [Documentation Requirements](RR-META-documentation-requirements.md) — README, ARCHITECTURE, CHANGELOG templates
- [NFR Specification](../01-research/RR-SPEC-v0.0.5b-non-functional-requirements-and-constraints.md) — Quality targets and constraints
- [FR Specification](../01-research/RR-SPEC-v0.0.5a-functional-requirements-specification.md) — Functional requirements
- [AI Collaborator Guide](RR-META-agentic-instructions-ai-collaborator-guide.md) — AI session context and collaboration rules
- [Master TDD](RR-META-llms-txt-architect.md) — Project vision and implementation roadmap
