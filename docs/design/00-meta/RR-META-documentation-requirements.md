# Documentation Requirements — DocStratum

<aside>

**Scope:** All phases (v0.1.x through v0.6.x)

**Status:** Active

**Applies To:** All project documentation — README files, architecture docs, decision logs, changelogs, and spec documents

**Deliverable:** Templates, content standards, and maintenance rules for every documentation artifact in the project

</aside>

---

## Purpose

This document defines what documentation the DocStratum project must produce, what each document must contain, and how documentation is maintained over the project lifecycle. Code comments and docstrings are covered separately in [Commenting Standards](RR-META-commenting-standards.md); this document covers everything *outside* the code itself.

---

## Documentation Philosophy

### Core Principles

1. **Documentation is a deliverable, not an afterthought.** Each phase includes documentation tasks alongside code tasks.
2. **Write for the newcomer.** Every document should be understandable by someone encountering the project for the first time.
3. **One source of truth.** Every fact lives in exactly one place. Other documents link to it rather than duplicating it.
4. **Keep it current or delete it.** Outdated documentation is worse than no documentation. If a document can't be maintained, remove it.
5. **Show, don't just tell.** Include code examples, command-line invocations, and expected output wherever possible.

---

## Documentation Inventory

### Required Documents

```
docstratum/
├── README.md                      # Project overview and quick start
├── ARCHITECTURE.md                # System design and module relationships
├── CHANGELOG.md                   # Version history and release notes
├── DECISION_LOG.md                # Architectural decision records
├── LICENSE                        # MIT license
├── docs/
│   ├── README.md                  # Documentation navigation hub
│   ├── 00-meta/                   # Standards and project-level docs
│   │   ├── README.md
│   │   ├── RR-META-testing-standards.md
│   │   ├── RR-META-logging-standards.md
│   │   ├── RR-META-commenting-standards.md
│   │   ├── RR-META-documentation-requirements.md  (this file)
│   │   ├── RR-META-development-workflow.md
│   │   ├── RR-META-llms-txt-architect.md
│   │   ├── RR-META-agentic-instructions-ai-collaborator-guide.md
│   │   ├── RR-META-memory-log-session-001.md
│   │   └── RR-META-specs.md
│   ├── 01-research/               # Research phase specs (v0.0.x)
│   ├── 02-foundation/             # Foundation phase specs (v0.1.x)
│   ├── 03-data-preparation/       # Data preparation specs (v0.2.x)
│   ├── 04-logic-core/             # Logic core specs (v0.3.x)
│   ├── 05-demo-layer/             # Demo layer specs (v0.4.x)
│   ├── 06-testing/                # Testing & validation specs (v0.5.x)
│   └── 07-release/                # Release phase specs (v0.6.x)
```

### Document Ownership by Phase

| Document | Created | Updated | Owner Phase |
|----------|---------|---------|-------------|
| README.md | v0.1.x (stub) | v0.6.1 (final) | Phase 7 (Release) |
| ARCHITECTURE.md | v0.3.0 (initial) | v0.6.2 (final) | Phase 7 (Release) |
| CHANGELOG.md | v0.2.0 (first entry) | Every release | All phases |
| DECISION_LOG.md | v0.1.0 (started) | Ongoing | All phases |
| Standards docs (00-meta/) | v0.1.x (this effort) | As needed | All phases |
| Spec docs (per-phase/) | Per phase | Within phase | Each phase |

---

## README.md — Project Overview

### Purpose

The README is the first thing anyone sees. It must answer in under 60 seconds: *What is this project? How do I run it? Where do I learn more?*

### Required Sections

```markdown
# DocStratum

One-paragraph description: A semantic translation layer for AI agents
browsing documentation. Hand-crafted /llms.txt architecture that
eliminates context collapse.

## Quick Start

Step-by-step instructions to get running in under 5 minutes.
Must include: clone, install, configure, run.

## What It Does

Brief explanation of the three-layer architecture with a
before/after example showing context collapse → structured output.

## Architecture Overview

High-level diagram (ASCII or Mermaid) showing the pipeline:
llms.txt → Loader → Context Builder → Agent → Response

## Installation

### Prerequisites
- Python 3.11+
- OpenAI or Anthropic API key

### Setup
Exact commands to install and configure.

## Usage

### CLI
How to run from the command line.

### Streamlit Demo
How to launch and use the web interface.

### Python API
How to import and use programmatically.

## Project Structure

Directory tree showing key files and their purposes.

## Results

Summary table: response quality metrics, baseline vs. DocStratum
comparison, with and without /llms.txt context.

## Contributing

Brief instructions (solo project, but document the process).

## License

MIT — link to LICENSE file.
```

### README Quality Checklist

- [ ] First sentence explains what the project does (no jargon)
- [ ] Quick Start section works for a fresh clone (tested)
- [ ] At least one code example showing input → output
- [ ] Installation instructions include all prerequisites
- [ ] All commands are copy-pasteable (no placeholders without explanation)
- [ ] No broken links

---

## ARCHITECTURE.md — System Design

### Purpose

Explains *how* the system works at a level above individual modules. Target audience: a developer who needs to understand the design before contributing.

### Required Sections

```markdown
# Architecture

## System Overview

One paragraph + high-level diagram showing the full pipeline.

## Three-Layer Architecture

Diagram showing the three context layers:
- Layer 1: Master Index ("What exists?")
- Layer 2: Concept Map ("How do things relate?")
- Layer 3: Few-Shot Bank ("How should I answer?")

## Module Relationships

Diagram showing which modules depend on which:

  loader.py
    ├── parser.py
    └── normalizer.py

  context/
    ├── builder.py
    ├── ranker.py
    └── token_manager.py

  agent/
    ├── baseline.py
    └── docstratum.py

  validator.py (independent)
  app.py (integrates all)

## Data Flow

Step-by-step description of how data moves through the system:
1. User provides llms.txt file path
2. Loader reads and normalizes the file
3. Parser extracts sections, entries, concepts
4. Context Builder selects relevant sections within token budget
5. Agent receives query + context → calls LLM → returns response
6. A/B Harness compares baseline vs. DocStratum agent responses
7. Demo layer visualizes results side-by-side

## Key Data Structures

Table of Pydantic models and what they represent:
- LlmsTxt: Root document model
- CanonicalPage: Single page entry with URL and metadata
- Concept: Extracted concept with relationships
- FewShotExample: Pre-written Q&A pair
- Context: Assembled context within token budget

## Design Decisions

Link to DECISION_LOG.md for detailed ADRs.
Summarize the most important decisions here:
- Why three-layer architecture over flat indexing
- Why LangChain over direct API calls
- Why Pydantic over plain dicts for schema
- Why Streamlit over React/Vue for demo

## Configuration

How configuration flows from .env → Config → modules.

## External Dependencies

What external services are required (OpenAI/Anthropic APIs)
and how the system handles their failure.
```

### Architecture Diagrams

Use Mermaid syntax (renders on GitHub):

```markdown
## Pipeline Diagram (Mermaid)

​```mermaid
graph LR
    A[llms.txt] --> B[Loader]
    B --> C[Parser]
    C --> D[Context Builder]
    D --> E[Agent]
    E --> F[Response]
    F --> G[A/B Harness]
    G --> H[Metrics]
​```
```

Include ASCII fallback for environments that don't render Mermaid:

```
llms.txt FILE
     │
     ▼
┌──────────┐
│  Loader  │──▶ Document
└──────────┘
     │
     ▼
┌──────────────┐
│Context Builder│──▶ Context (within token budget)
└──────────────┘
     │
     ▼
┌──────────┐
│  Agent   │──▶ Response
└──────────┘
```

---

## CHANGELOG.md — Version History

### Purpose

Tracks what changed in each version. Essential for understanding project progression and release history.

### Format: Keep a Changelog

Follow the [Keep a Changelog](https://keepachangelog.com/) convention:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- Items that are new

### Changed
- Items that changed behavior

### Fixed
- Bug fixes

## [0.3.0] - 2026-XX-XX

### Added
- Loader module with file parsing and normalization (v0.3.1)
- Context Builder with token budgeting (v0.3.2)
- Baseline and DocStratum agent implementations (v0.3.3–v0.3.4)
- A/B testing harness (v0.3.5)

## [0.2.0] - 2026-XX-XX

### Added
- Source audit and content inventory (v0.2.1)
- Concept extraction pipeline (v0.2.2)
- YAML authoring for llms.txt (v0.2.3)
- Validation pipeline (v0.2.4)

## [0.1.0] - 2026-02-XX

### Added
- Project scaffolding and directory structure (v0.1.1)
- Pydantic schema definition for llms.txt (v0.1.2)
- Sample data and examples (v0.1.3)
```

### Rules

- **Added** for new features
- **Changed** for changes in existing functionality
- **Deprecated** for features that will be removed
- **Removed** for features that have been removed
- **Fixed** for bug fixes
- **Security** for vulnerability fixes
- Update `[Unreleased]` section as work progresses
- Move `[Unreleased]` items to a versioned section on release

---

## DECISION_LOG.md — Architectural Decision Records

### Purpose

Records significant technical decisions, their context, and their rationale. Prevents re-litigating settled decisions and helps future contributors understand *why* things are the way they are.

### Template: Lightweight ADR

```markdown
# Decision Log

## ADR-001: Three-Layer Architecture for llms.txt

**Date:** 2026-02-05
**Status:** Accepted
**Phase:** v0.0.4 (Best Practices Synthesis)

### Context
We needed a structure for the llms.txt file that supports multiple
agent use cases: navigation, concept lookup, and few-shot learning.

### Options Considered
1. **Flat index** — single list of URLs
2. **Two-layer** — index + concepts
3. **Three-layer** — Master Index + Concept Map + Few-Shot Bank

### Decision
Use three layers (option 3) because:
- Each layer serves a distinct agent need
- Layers can be selected independently within token budgets
- Matches the research findings from v0.0.4a–d

### Consequences
- Must define schema for all three layers (v0.1.2)
- Context builder must support per-layer selection
- More complex than flat indexing but significantly more useful

---

## ADR-002: LangChain for Agent Integration

**Date:** 2026-02-05
**Status:** Accepted
**Phase:** v0.1.1

### Context
...
```

### When to Write an ADR

Write a decision record when:

- Choosing between multiple valid approaches
- Selecting a library or framework
- Defining a data format or protocol
- Making a trade-off that affects multiple modules
- Reversing or amending a previous decision

### When NOT to Write an ADR

- Implementation details within a single function
- Formatting or style choices (covered by standards docs)
- Temporary workarounds (use TODO comments instead)

---

## Spec Document Standards

### Existing Convention

The project uses spec documents organized by phase in `docs/phase-{N}/`. The following rules codify and enforce the established convention.

### File Naming

All spec files follow the naming convention:

```
RR-SPEC-v{X}.{Y}.{Z}-{kebab-case-slug}.md       # Version specs
RR-SPEC-v{X}.{Y}.{Z}{letter}-{kebab-case-slug}.md  # Sub-documents
RR-META-{kebab-case-slug}.md                       # Meta documents
README.md                                          # Directory indexes
```

### Required Spec Document Sections

Every version spec must include:

```markdown
# vX.Y.Z — Brief Title

<aside>
**Phase:** N — Phase Name
**Version:** vX.Y.Z
**Status:** Not Started | In Progress | Complete
**Duration:** Estimated time
**Deliverable:** One-sentence description of what this version produces
</aside>

---

## Objective
What this version accomplishes and why.

## [Technical Content]
Main body — varies by document type.

## Acceptance Criteria
- [ ] Checkbox list of verifiable completion criteria

## Limitations & Constraints
Numbered list of known limitations.

## Dependencies
What must be completed before this version.

## Troubleshooting
Common issues with symptoms and solutions.

## User Story
> As a [role], I want to [action] so that [benefit].

## Inputs from Previous Sub-Parts
What this version receives from prior work.

## Outputs to Next Sub-Part
What this version produces for subsequent work.

## Decision Log
Table of decisions made during this version.
```

### Sub-Document Standards

For sub-documents (e.g., `RR-SPEC-v0.3.1a-file-loading.md`):

```markdown
# vX.Y.Za — Sub-Part Title

<aside>
**Version:** vX.Y.Za
**Parent:** vX.Y.Z — Parent Title
**Status:** Not Started
**Duration:** Estimated time
**Deliverable:** What this sub-part produces
</aside>

## Objective
## [Technical Content]
## Acceptance Criteria
## Limitations & Constraints
## Dependencies
## Troubleshooting
## User Story
## Inputs from Previous Sub-Parts
## Outputs to Next Sub-Part
## Decision Log
```

---

## Documentation Maintenance

### Update Triggers

| Event | Documents to Update |
|-------|-------------------|
| New feature implemented | CHANGELOG.md, relevant spec README |
| Architecture change | ARCHITECTURE.md, DECISION_LOG.md |
| Bug fix | CHANGELOG.md |
| New dependency added | README.md (prerequisites), requirements.txt |
| API change | README.md (usage), ARCHITECTURE.md |
| Version released | CHANGELOG.md (move Unreleased → version) |
| Standard changed | Relevant RR-META-* standards doc |

### Staleness Prevention

- Review ARCHITECTURE.md at the start of each new phase
- Review README.md Quick Start section when dependencies change
- Run all Quick Start commands on a fresh clone before release (v0.6.1)
- Delete documents that no longer apply (don't just mark "deprecated")

---

## Documentation Quality Standards

### Writing Style

- **Active voice:** "The loader parses documents" not "Documents are parsed by the loader"
- **Present tense:** "The function returns a string" not "The function will return a string"
- **Second person for instructions:** "Run `pytest`" not "The user should run pytest"
- **Concise:** Remove filler words. "In order to" → "To". "Basically" → delete.
- **Consistent terminology:** Use the same term for the same concept everywhere

### Terminology Table

| Term | Meaning | Don't Use |
|------|---------|-----------|
| llms.txt | The structured file that DocStratum processes | sitemap, index file |
| Layer | One of three architectural tiers (Master Index, Concept Map, Few-Shot Bank) | level, section (when referring to architecture) |
| Section | A `##`-delimited block within an llms.txt file | layer, part, chunk |
| Entry | A single URL or content item within a section | page, link, item |
| Concept | An extracted term with definitions and relationships | entity, keyword, topic |
| Context | Assembled information within a token budget for LLM consumption | prompt, input, payload |
| Token budget | Maximum tokens allocated for context assembly | token limit, max tokens |
| Baseline agent | Agent without DocStratum context (control group) | plain agent, simple agent |
| DocStratum agent | Agent enhanced with DocStratum context (treatment group) | enhanced agent, smart agent |

### Formatting Rules

- Use `code blocks` for file paths, commands, function names, and config values
- Use **bold** for key terms being defined
- Use tables for comparisons and option matrices
- Use numbered lists for sequential steps
- Use bullet lists for unordered items
- Use `> blockquotes` for user stories and important callouts
- Include language identifier on fenced code blocks (```python, ```bash, etc.)

---

## Dos and Don'ts

### Do

- Write documentation alongside code, not after
- Include a Quick Start that works on a fresh clone
- Test every command you include in documentation
- Use templates for consistency across documents
- Link to related documents rather than duplicating content
- Include diagrams for system-level concepts
- Update CHANGELOG.md with every version
- Write decision records for significant technical choices

### Don't

- Write documentation you won't maintain
- Duplicate the same information in multiple places
- Use screenshots for command-line output (use text blocks)
- Write walls of text without structure (use headers and lists)
- Assume the reader has context (link to prerequisites)
- Leave placeholder sections empty ("TBD", "TODO: fill in later")
- Use relative dates ("recently", "soon") — use version numbers
- Create documentation files not listed in the project structure

---

## Acceptance Criteria (for this document)

- [ ] README.md template defined with all required sections
- [ ] ARCHITECTURE.md requirements specified with diagram standards
- [ ] CHANGELOG.md format defined (Keep a Changelog convention)
- [ ] DECISION_LOG.md ADR template provided
- [ ] Spec document section requirements match existing conventions
- [ ] File naming convention documented (RR-SPEC-/RR-META- patterns)
- [ ] Documentation maintenance triggers defined
- [ ] Writing style guide with terminology table provided
- [ ] Formatting rules documented
- [ ] All five standards documents cross-reference each other

---

## Related Documents

- [Commenting Standards](RR-META-commenting-standards.md) — In-code documentation (docstrings, comments)
- [Testing Standards](RR-META-testing-standards.md) — Test documentation and naming conventions
- [Logging Standards](RR-META-logging-standards.md) — Operational documentation via log output
- [Development Workflow](RR-META-development-workflow.md) — Documentation as part of the development lifecycle
- [NFR Specification](../01-research/RR-SPEC-v0.0.5b-non-functional-requirements-and-constraints.md) — NFR-007 (documentation coverage), NFR-013 (documentation-to-code ratio)
- [v0.6.1 — README Polish](../07-release/RR-SPEC-v0.6.1-readme-polish.md) — Phase where README is finalized
- [v0.6.2 — Docs Folder](../07-release/RR-SPEC-v0.6.2-docs-folder.md) — Phase where documentation is finalized
