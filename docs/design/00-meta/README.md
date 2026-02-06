# 00-meta — Project Overview & Collaboration

> **Purpose**: High-level project vision, AI collaboration guidelines, and architectural overview

---

## 📄 Contents

### Core Documentation

- **`RR-META-llms-txt-architect.md`** — Master technical design document (v1.0)
  - Executive summary and "Writer's Edge" thesis
  - Complete tech stack justification
  - Implementation roadmap (v0.0 through v0.6)
  - Learning outcomes and resume bullet points

- **`RR-META-specs.md`** — Detailed specification breakdowns
  - v0.0.1a: Formal Grammar & Parsing Rules
  - v0.0.1b: Spec Gap Analysis & Implications
  - v0.0.1c: Processing & Expansion Methods
  - v0.0.1d: Standards Interplay & Positioning

### Engineering Standards

- **`RR-META-testing-standards.md`** — Testing standards
  - pytest framework, AAA pattern, fixtures, markers
  - Per-module coverage targets (aligned with NFR-010)
  - Mocking strategies for LLM calls, parametrized testing

- **`RR-META-logging-standards.md`** — Logging standards
  - Python stdlib `logging` only, `%s` formatting
  - Five-level contract with per-module INFO requirements (FR-067)
  - Secret masking, error logging patterns, `caplog` testing

- **`RR-META-commenting-standards.md`** — Commenting standards
  - Google-style docstrings, type hints, inline comments
  - Module/class/function docstring templates (NFR-007, NFR-013)
  - TODO format, enforcement tools (Ruff, pydocstyle, mypy)

- **`RR-META-documentation-requirements.md`** — Documentation requirements
  - README, ARCHITECTURE, CHANGELOG, DECISION_LOG templates
  - Spec document section standards and file naming convention
  - Writing style guide, terminology table, maintenance triggers

- **`RR-META-development-workflow.md`** — Development workflow
  - Spec-first methodology, 12-step development lifecycle
  - Phase transition rules, version numbering, AI session protocols
  - Branching/commit conventions, self-review checklist

### AI Collaboration

- **`RR-META-agentic-instructions-ai-collaborator-guide.md`** — AI collaborator guide
  - Prompt engineering standards
  - Context management strategies
  - Collaboration protocols

- **`RR-META-memory-log-session-001.md`** — Session memory
  - Project initialization notes
  - Research completion status
  - Decision history

---

## 🎯 Start Here

**New to the project?** Read the files in this order:

1. **RR-META-llms-txt-architect** — Get the big picture and understand the "why"
2. **RR-META-specs** — Understand the detailed specification analysis
3. **RR-META-agentic-instructions-ai-collaborator-guide** — Learn how to collaborate effectively (for AI agents)

---

## 🔑 Key Concepts

### The Problem: Context Collapse

AI agents browsing documentation face systematic loss of meaning when encountering:

- Unstructured content
- Navigation-heavy layouts
- Semantically ambiguous web pages

### The Solution: The DocStratum

A hand-crafted `/llms.txt` file providing:

1. **Canonical Concept Map** — Human-curated taxonomy
2. **Explicit Relationship Declarations** — Dependency graphs
3. **Few-Shot Training Examples** — Pre-written Q&A pairs

### The Differentiator: The Writer's Edge

> "A Technical Writer with strong Information Architecture skills can outperform a sophisticated RAG pipeline by simply writing better source material."

**Structure is a feature.**

---

## 📊 Project Status

**Phase**: Foundation (v0.1.x)  
**Status**: Active Development  
**Last Updated**: February 5, 2026

---

## 🗺️ Next Steps

After reviewing this directory, proceed to:

- **`01-research/`** — Understand the research foundation
- **`02-foundation/`** — Begin implementation with schema definition
