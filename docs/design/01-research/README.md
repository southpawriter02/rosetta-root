# 01-research — Research & Discovery (v0.0.x)

> **Purpose**: Foundational research, specification analysis, and ecosystem survey

This phase establishes the theoretical and empirical foundation for the DocStratum project through systematic analysis of the `/llms.txt` specification, real-world implementations, and ecosystem tooling.

---

## 📚 Phase Structure

### v0.0.0 — Initial Research

- **Research & Discovery** — Project kickoff and scope definition
- **Wild Examples Analysis** — Analysis of real-world `/llms.txt` implementations
- **Stripe LLM Instructions Pattern** — Case study of industry best practices

### v0.0.1 — Specification Deep Dive

Formal analysis of the `/llms.txt` specification:

- **v0.0.1a** — Formal Grammar & Parsing Rules (ABNF grammar, parsing pseudocode)
- **v0.0.1b** — Spec Gap Analysis & Implications (8 undefined areas and their real-world impact)
- **v0.0.1c** — Processing & Expansion Methods (Concatenation, XML wrapping, selective inclusion, summarization)
- **v0.0.1d** — Standards Interplay & Positioning (robots.txt, sitemap.xml, humans.txt integration)

### v0.0.2 — Wild Examples Audit

Systematic audit of existing `/llms.txt` implementations:

- **v0.0.2a** — Source Discovery & Collection
- **v0.0.2b** — Individual Example Audits
- **v0.0.2c** — Pattern Analysis & Statistics
- **v0.0.2d** — Synthesis & Recommendations

### v0.0.3 — Ecosystem & Tooling Survey

Comprehensive survey of the `/llms.txt` ecosystem:

- **v0.0.3a** — Tools & Libraries Inventory
- **v0.0.3b** — Key Players & Community Pulse
- **v0.0.3c** — Related Standards & Competing Approaches
- **v0.0.3d** — Gap Analysis & Opportunity Map

### v0.0.4 — Best Practices Synthesis

Distillation of patterns and anti-patterns:

- **v0.0.4a** — Structural Best Practices
- **v0.0.4b** — Content Best Practices
- **v0.0.4c** — Anti-Patterns Catalog
- **v0.0.4d** — DocStratum Differentiators & Decision Log

### v0.0.5 — Requirements Definition

Formal requirements specification:

- **v0.0.5a** — Functional Requirements Specification
- **v0.0.5b** — Non-Functional Requirements & Constraints
- **v0.0.5c** — Scope Definition & Out-of-Scope Registry
- **v0.0.5d** — Success Criteria & MVP Definition

---

## 🔍 Key Findings

### Specification Gaps

The original `/llms.txt` spec leaves 8 critical areas undefined:

1. Max file size
2. Versioning
3. File type restrictions
4. Context scope
5. Link validation
6. File priority
7. Language specification
8. Change notification mechanism

### Processing Methods

Four primary methods identified:

- **Concatenation** — Simple but loses document boundaries
- **XML Wrapping** — Preserves structure, slight token overhead
- **Selective Inclusion** — Reduces noise, requires pre-processing
- **Summarization** — Compresses content, risk of detail loss

### Ecosystem Insights

- FastHTML's `llms_txt2ctx` uses basic concatenation
- Sophisticated tools (LlamaIndex, Pinecone) use XML wrapping
- Internal tools at Stripe/Anthropic use selective inclusion
- No standard tooling for summarization pipelines

---

## 📊 Deliverables

**Research Artifacts**:

- Formal ABNF grammar for `/llms.txt`
- Comparative analysis of 4 processing methods
- Catalog of anti-patterns from wild examples
- Gap analysis justifying schema extensions

**Requirements Documentation**:

- Functional requirements specification
- Non-functional requirements and constraints
- Success criteria and MVP definition
- Scope boundaries and out-of-scope registry

---

## 🎯 Success Criteria

This research phase is complete when:

- ✅ Specification gaps are documented with real-world impact
- ✅ Ecosystem survey identifies all major tools and players
- ✅ Best practices are synthesized from wild examples
- ✅ Requirements are formally defined and approved
- ✅ MVP scope is clearly bounded

---

## 🗺️ Next Phase

After completing research, proceed to:

- **`02-foundation/`** — Environment setup and schema definition
