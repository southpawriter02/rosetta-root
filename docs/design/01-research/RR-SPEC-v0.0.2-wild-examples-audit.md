# v0.0.2 — Wild Examples Audit

> **Task:** Find and analyze real-world llms.txt implementations to understand how others are using the specification.
> 

---

## Task Overview

> **Phase:** Research & Discovery (v0.0.x)
> **Status:** COMPLETE
> **Sub-Parts:** v0.0.2a, v0.0.2b, v0.0.2c, v0.0.2d — all verified
> **Date Completed:** 2026-02-05
> **Verified:** 2026-02-05

---

## 🎯 Purpose

This task expands upon the preliminary example analysis done in v0.0.1. While the Specification Deep Dive included a [Wild Examples Analysis](RR-SPEC-v0.0.0-wild-examples-analysis.md) of Stripe, Nuxt, and Vercel, this task conducts a **systematic audit** of a broader range of implementations to:

1. **Validate patterns** discovered in v0.0.1
2. **Discover new patterns** from diverse sources
3. **Quantify trends** (file sizes, structures, common sections)
4. **Build an evidence base** for DocStratum design decisions

---

## 📋 Sub-Parts Overview

This task is divided into four sequential sub-parts. Each must be completed before starting the next.

---

## 🔗 Dependency Chain

```
v0.0.1 — Specification Deep Dive ✅
    │
    ├── Wild Examples Analysis (Stripe, Nuxt, Vercel)
    │   └── Provides: Initial patterns, audit approach
    │
    └── Stripe LLM Instructions Pattern
        └── Provides: Framework for evaluating LLM guidance sections
            │
            ▼
v0.0.2 — Wild Examples Audit (THIS TASK)
    │
    ├── v0.0.2a — Source Discovery & Collection
    │   └── Output: Catalog of 15-20 llms.txt URLs with metadata
    │
    ├── v0.0.2b — Individual Example Audits
    │   └── Input: Catalog from v0.0.2a
    │   └── Output: Completed audit forms for each source
    │
    ├── v0.0.2c — Pattern Analysis & Statistics
    │   └── Input: Audits from v0.0.2b
    │   └── Output: Statistical summary, pattern taxonomy
    │
    └── v0.0.2d — Synthesis & Recommendations
        └── Input: Analysis from v0.0.2c + v0.0.1 findings
        └── Output: Best practices guide, DocStratum requirements
            │
            ▼
v0.0.3 — Ecosystem Survey (NEXT TASK)
```

---

## 📂 Sub-Part Pages

[v0.0.2a — Source Discovery & Collection](RR-SPEC-v0.0.2a-source-discovery-and-collection.md)

[v0.0.2b — Individual Example Audits](RR-SPEC-v0.0.2b-individual-example-audits.md)

[v0.0.2c — Pattern Analysis & Statistics](RR-SPEC-v0.0.2c-pattern-analysis-and-statistics.md)

[v0.0.2d — Synthesis & Recommendations](RR-SPEC-v0.0.2d-synthesis-and-recommendations.md)

---

## 🧠 Context from v0.0.1

### Already Analyzed (Do Not Re-Audit)

These examples were analyzed in [Wild Examples Analysis](RR-SPEC-v0.0.0-wild-examples-analysis.md):

### Patterns Already Identified

From v0.0.1, we know to look for:

- LLM Instructions sections (Stripe pattern)
- Tiered expansion (llms.txt + llms-full.txt)
- Hierarchical organization
- Version separation
- Anti-pattern documentation

### Gaps to Fill

v0.0.2 should specifically seek:

- **Smaller projects** — How do minimal implementations look?
- **Non-tech companies** — Do non-developer docs use llms.txt?
- **API-heavy sites** — How is API documentation structured?
- **Open source projects** — Community-maintained examples
- **Broken/poor examples** — What NOT to do

---

## ✅ Overall Acceptance Criteria

All sub-parts must be complete before v0.0.2 is considered done:

- [x]  **v0.0.2a:** 15-20 unique sources cataloged with metadata (18 sources across 6 categories)
- [x]  **v0.0.2b:** Audit form completed for each source (18/18 audits with 72 ratings)
- [x]  **v0.0.2c:** Statistical summary and pattern taxonomy complete (5 archetypes, 7 correlation factors)
- [x]  **v0.0.2d:** Best practices document and DocStratum requirements drafted (15 practices, 7 anti-patterns, 16 requirements)

---

## 📊 Deliverables Summary

[v0.0.2a — Source Discovery & Collection](RR-SPEC-v0.0.2a-source-discovery-and-collection.md)

[v0.0.2b — Individual Example Audits](RR-SPEC-v0.0.2b-individual-example-audits.md)

[v0.0.2c — Pattern Analysis & Statistics](RR-SPEC-v0.0.2c-pattern-analysis-and-statistics.md)

[v0.0.2d — Synthesis & Recommendations](RR-SPEC-v0.0.2d-synthesis-and-recommendations.md)