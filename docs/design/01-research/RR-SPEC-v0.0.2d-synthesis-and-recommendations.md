# v0.0.2d — Synthesis & Recommendations

> **Sub-Part:** Synthesize all findings into actionable recommendations for DocStratum design.
>

---

## Sub-Part Overview

This document synthesizes all research findings from v0.0.1 (8-site initial survey) and v0.0.2a-c (comprehensive 18-site audit with detailed analysis) into a unified set of actionable recommendations for DocStratum v0.1.0 design. It identifies gold standard examples, documents proven best practices with evidence citations, catalogs anti-patterns to avoid, prioritizes DocStratum requirements (P0/P1/P2), and provides specific schema design guidance.

---

## Objective

Synthesize all research from v0.0.1 and v0.0.2 into concrete recommendations for DocStratum. This is the final deliverable of the Wild Examples Audit.

### Success Looks Like

- Clear "Gold Standard" example identified with primary and secondary candidates
- At least 12 best practices guide documented with evidence citations
- At least 7 anti-patterns catalog completed
- DocStratum requirements drafted with prioritization
- Evidence-based recommendations linked to specific source documents

---

## Scope Boundaries

### In Scope

- Identifying gold standard example(s) from the 18-site audit
- Documenting best practices with direct evidence from v0.0.2b audit table and research files
- Cataloging anti-patterns to avoid with specific citations
- Drafting DocStratum requirements (P0/P1/P2) based on v0.0.1 findings
- Creating recommendations for v0.1.0 design phase
- Synthesizing insights across all 18 audited sites

### Out of Scope

- Actually designing the DocStratum schema (that's v0.1.0)
- Building any tools or code
- Additional research beyond v0.0.2a-c data
- Site audits beyond the 18 completed examples

---

## Gold Standard Identification

### Criteria for Gold Standard

An example qualifies as "gold standard" if it:

- [x] Scores 5/5 overall in v0.0.2b audit
- [x] Demonstrates innovative features worth adopting
- [x] Could serve as a template for others
- [x] Is actively maintained
- [x] Represents achievable quality (not impossibly complex)

### Gold Standard Selection

**Primary Gold Standard: Svelte**

**Secondary Gold Standards: Shadcn UI, Pydantic, Vercel AI SDK**

### Gold Standard Deep Dive

#### Primary Gold Standard: Svelte

**Why it's exemplary:**

1. **Perfect 5/5 overall score with highest consistency** — The only site to achieve 5/5 across all dimensions: Size (5), Completeness (5), Organization (5), Descriptions (5), LLM-Friendliness (5). This demonstrates that excellence is achievable across all evaluation criteria (v0.0.2b audit table, row 11).

2. **Multi-tier variant strategy with documented guidance** — Svelte implements multiple llms.txt variants (small, medium, full) alongside a dedicated `/docs/llms` guidance page. This addresses the perennial tension between comprehensiveness and token budgets, proving that tiering is not a limitation but a design feature. The guidance page component is unique in the audit and serves as a bridge between variants (v0.0.2b table distinction).

3. **Explicit performance warnings for RAG degradation** — Unlike other frameworks that silently accept larger files, Svelte proactively documents which tier to use for different use cases. This prevents practitioners from unknowingly degrading RAG quality through poor file selection and represents a mature understanding of LLM-readability beyond mere documentation (v0.0.2b key distinction).

4. **Concept-first rather than link-first organization** — Like Pydantic, Svelte prioritizes semantic structure over alphabetical or hierarchical site organization. This is "Layers 3-5 thinking" applied to a framework that is itself a web framework, not an API platform (v0.0.2b table row 11 and context from AI-Readability Stack).

5. **Actively maintained with community contributions** — Svelte's llms.txt ecosystem is not a static artifact but an evolving standard that the maintainers actively refine. This ensures it reflects real-world llms.txt usage patterns (v0.0.2b context).

**What DocStratum should borrow:**

- **Tiered documentation as a first-class design pattern** — Not a workaround but the primary way to serve multiple use cases (RAG agents, code assistants, context-limited models)
- **Dedicated guidance page** — A `/docs/llms` or similar resource that helps practitioners choose the right variant and understand the format's capabilities
- **Performance-aware descriptions** — Include metadata about token counts, typical use cases, and degradation warnings
- **Variant naming convention** — Establish canonical names (small/medium/full) with clear semantics
- **Cross-cutting concerns documentation** — Document how concepts map across tiers

**What could be improved even on this example:**

- **Explicit versioning scheme** — While Svelte's llms.txt ecosystem is mature, there's no version tag (e.g., `// version: 1.0`) to track compatibility (v0.0.1 P1 gap)
- **LLM Instructions section** — Even though Svelte's tiering is exemplary, it lacks explicit LLM Instructions (positive/negative directives) that could guide agent behavior. This is not a significant omission but represents untapped potential (v0.0.1 anti-pattern "No LLM Instructions despite being AI-native")
- **Concept definition block** — While organization is concept-first, there's no formal glossary or concept index to help LLMs navigate the semantic space quickly (v0.0.1 P0 gap)

---

#### Secondary Gold Standard 1: Shadcn UI

**Why it's exemplary:**

1. **Perfect 5/5 score with AI-first architecture** — Like Svelte, achieves 5/5 overall (row 12) but with a different emphasis: processMdxForLLMs pipeline (v0.0.2b key distinction). This demonstrates that AI-readability can be built into the documentation pipeline itself, not just the output format.

2. **Framework-agnostic variants** — Provides separate llms.txt files for React, Svelte, and Vue implementations. This is "per-product decomposition" (like Cloudflare's approach) applied vertically across framework stacks rather than horizontally across microservices.

3. **Tight integration with source code** — Unlike other examples that treat llms.txt as a separate artifact, Shadcn UI demonstrates that llms.txt can be generated from—and remain synchronized with—component source code through automation.

**What DocStratum should borrow:**

- **Automated generation from source metadata** — Define hooks (frontmatter, JSDoc comments, etc.) that allow llms.txt to be auto-generated and kept in sync
- **Component-focused descriptions** — Rather than documenting "the whole API," document individual, reusable units with clear contracts

---

#### Secondary Gold Standard 2: Pydantic

**Why it's exemplary:**

1. **Perfect 5/5 score with concept-first discipline** — Achieves 5/5 (row 13) through ruthless prioritization of conceptual clarity. The llms.txt is organized around Pydantic's core mental models (validation, serialization, schema) not its module structure (v0.0.2b key distinction "Concept-first organization").

2. **Dual-file strategy with clear semantics** — Like Anthropic and others, uses both a concise and full variant, but the distinction is semantic: llms.txt covers "What is Pydantic?" while llms-full.txt covers "How do I use Pydantic?" This is the clearest expression of tiering semantics in the audit.

3. **Cross-cutting concerns are first-class** — Documents patterns that span the entire framework (validation, error handling, typing) rather than burying them in individual module sections.

**What DocStratum should borrow:**

- **Semantic tiering names** — Use names that describe the *kind* of information (overview vs. reference, not small vs. large)
- **Explicit concept index** — List the core mental models at the top so LLMs understand the conceptual structure before diving into details
- **Cross-cutting patterns section** — Dedicate a section to patterns and paradigms that span the entire tool

---

#### Secondary Gold Standard 3: Vercel AI SDK

**Why it's exemplary:**

1. **Perfect 5/5 score as a modern reference architecture** — Achieves 5/5 (row 16) while being relatively young. This proves that new projects *can* be designed with LLM-readability in mind from the start (v0.0.2b key distinction "Full-stack reference architecture").

2. **Streaming-first and composable design** — The llms.txt reflects modern async/streaming patterns that older frameworks had to retrofit. This is a forward-looking template for AI agent frameworks.

3. **Tight TypeScript integration** — Like Shadcn UI, demonstrates that type definitions themselves can be high-quality documentation for LLMs, and llms.txt should amplify rather than duplicate that information.

**What DocStratum should borrow:**

- **Modern toolchain assumptions** — Assume tools use TypeScript, have async-first APIs, and support streaming. Structure llms.txt accordingly.
- **Composition as core metaphor** — Document how small units compose into larger patterns

---

### Why Not Other Candidates?

**Anthropic (5 on some dimensions, 4 overall):** Excellent research partner and co-developer of the format, but the 8K + 481K split doesn't demonstrate clear tiering semantics. The "full" variant is too large to serve as a general template.

**Cloudflare (5 overall):** Demonstrates impressive per-product decomposition at scale (3.7M tokens), but the monolithic full file is an anti-pattern (v0.0.1 noted "Monolithic 3.7M token files"). The approach is useful for very large platforms but less generalizable to smaller projects.

**Other 4/5 or 4/4 sites:** All demonstrate competence, but the 4/5 candidates (ElevenLabs, Shopify, Hugging Face, Pinecone, Zapier, LangChain) lack the innovative features that make Svelte, Shadcn UI, Pydantic, and Vercel AI SDK exemplary. They are "well done" not "bold."

---

### Empirical Gold Standard Enrichment (2026-02-06)

#### Vercel AI SDK Correction

The original audit (v0.0.2b, row 16) rated Vercel AI SDK at 5/5 based on estimated data about an "index" variant. Empirical collection (2026-02-06) found that the collected specimen **ai-sdk-llms.txt is actually a Type 2 Full document** (1.3 MB, 38,717 lines, 15% conformance).

**Critical clarification:** The 5/5 rating specifically applies to the **INDEX variant** of Vercel AI SDK, which remains unknown in the specimen collection. The full variant demonstrates the Type 2 paradigm—a different architectural approach with different conformance characteristics than the index.

**Implication for v0.1.0:** When citing Vercel AI SDK as a gold standard, reference the architectural *concept* of its index design, not the full variant. The full-size approach shows why tiering (Practice 1) is essential: even a well-structured full variant achieves only 15% conformance due to size and complexity.

---

#### New Gold Standard Candidate: Astro

**Specimen data:** 2.6 KB, 31 lines, **100% conformance**

**Why it qualifies:** While tiny, Astro demonstrates perfect adherence to the specification. It proves that minimal implementations can achieve complete conformance without bloat. This is exemplary for the "micro" tier—frameworks or tools that can deliver essential information in under 3K tokens.

**What makes it exemplary:**
- **Impossibly clean structure:** No wasted space, no redundancy, 100% signal
- **Minimal viability:** Proves that perfect conformance doesn't require comprehensive coverage
- **Template quality:** Serves as a reference for "small variant" best practices

**Recommendation:** Add Astro to secondary gold standards for the micro tier. Developers building small, focused tools should use Astro as a template rather than larger examples.

---

#### New Gold Standard Candidate: OpenAI

**Specimen data:** 19 KB, 151 lines, **100% conformance**

**Why it qualifies:** OpenAI achieves perfect conformance at a modest size (19K tokens). This demonstrates that simplicity and correctness are achievable without complexity. From a major AI company, this proves that excellence can be pragmatic.

**What makes it exemplary:**
- **Compact efficiency:** 151 lines to achieve everything without excess
- **Clear organization:** Despite brevity, every concept is findable
- **Scalable template:** Developers can use this as a template and expand without losing coherence

**Recommendation:** Add OpenAI to secondary gold standards as the "well-proportioned" example. It sits perfectly between Astro (too minimal) and larger references.

---

#### New Gold Standard Candidate: Deno

**Specimen data:** 63 KB, 464 lines, **100% conformance**

**Why it qualifies:** Deno achieves perfect conformance at medium-large scale (63K tokens). This proves that 100% compliance is achievable across all size tiers, not just at minimal scales.

**What makes it exemplary:**
- **Comprehensive scope:** 464 lines of well-organized content
- **Proof of scale:** Shows that growth doesn't require sacrifice of standards
- **Complex system clarity:** Deno is non-trivial; managing clarity at this scale is hard

**Recommendation:** Add Deno to secondary gold standards as the "comprehensive" example. Teams building large, complex tools should use Deno to see how clarity scales.

---

### Updated Gold Standard Roster (2026-02-06 Empirical)

**Primary Gold Standard:** Svelte (unchanged)

**Secondary Gold Standards (Updated):**
1. Shadcn UI (unchanged)
2. Pydantic (unchanged)
3. Vercel AI SDK (clarified: index variant exemplary, full variant shows Type 2 anti-pattern)
4. **Astro** (new: micro tier reference)
5. **OpenAI** (new: well-proportioned reference)
6. **Deno** (new: comprehensive scale reference)

---

## Best Practices Guide

Based on evidence from v0.0.1 and v0.0.2, these 12+ practices are proven across the 18-site audit:

### Structure Best Practices

**1. Use multi-tier variants for comprehensiveness without bloat (Tiered Architecture Pattern)**

**Evidence:** Svelte (5/5), Anthropic (4), ElevenLabs (4), Pydantic (5), PydanticAI (4) all implement tiering. Svelte's guidance page shows this isn't accidental but a deliberate design pattern. (v0.0.2b rows 1, 3, 5, 13, 14; v0.0.2d Svelte deep dive).

**Empirical enrichment (2026-02-06):** Specimen analysis reveals a stark bimodal size distribution with **no specimens between 225 KB and 1.3 MB**. This suggests that the "full" tier for Type 1 documents should cap at approximately **250 KB** to avoid crossing into Type 2 territory (where conformance drops sharply). The gap indicates a natural architectural boundary: anything beyond ~250K requires redesign (decomposition, per-product splitting) rather than simple expansion.

**Recommendation:**
- Provide small (< 5K tokens), medium (5-20K), and full (20-50K) variants
- Use consistent naming: llms.txt (small), llms-medium.txt, llms-full.txt
- Document when to use each variant based on RAG quality vs. context limits
- Include performance warnings in guidance materials
- **Cap Type 1 full variants at ~250 KB; beyond that, decompose into multiple files (Type 2 paradigm)**

**2. Organize by concepts, not alphabetically or structurally (Semantic Organization)**

**Evidence:** Pydantic (5/5) and Svelte (5/5) both prioritize conceptual clarity over site structure. Pydantic explicitly organizes around "validation, serialization, schema" not modules. (v0.0.2b table rows 11, 13; "Key Distinction" column).

**Empirical enrichment (2026-02-06):** Of 11 specimens analyzed, **3 achieve 100% conformance** (Astro, OpenAI, Deno) while using simple, clean organization without architectural complexity. This validates that conceptual clarity does NOT require intricate hierarchy or elaborate structure. The correlation between conformance and straightforward organization (not complex schemes) is strong.

**Recommendation:**
- Identify 5-7 core concepts or mental models
- Create sections around these concepts, not site URLs
- Within each section, group related functionality (even if from different modules)
- Include a concept index at the top
- **Prioritize clarity and simplicity in organization; complexity is a warning sign, not a feature**

**3. Provide dedicated LLM guidance documentation (Guidance Page Pattern)**

**Evidence:** Svelte's `/docs/llms` page is unique in the audit and is explicitly noted as a distinguishing feature. No other site provides a guidance document. (v0.0.2b row 11 key distinction, v0.0.2d Svelte analysis).

**Recommendation:**
- Create a `/docs/llms` or `LLM_GUIDE.md` alongside llms.txt
- Explain: what llms.txt is, how to choose variants, common patterns, limitations
- Include a "best practices for AI agents" section specific to your tool
- Update this document as the format evolves

**4. Implement per-product or per-domain file decomposition for large platforms (Modular Variants)**

**Evidence:** Cloudflare (5/5) uses per-product decomposition for 3.7M full tokens. Supabase (4) uses domain-specific variants (llms/guides.txt). (v0.0.2b rows 2, 3; key distinction column).

**Recommendation:**
- For platforms with 10+ distinct products: create llms/{product}.txt files
- For monolithic tools with clear domains (guides, API, advanced): create llms/{domain}.txt variants
- Provide an index file that lists all variants and their purposes
- Ensure index is small enough to be included in small-tier file

**5. Use standard section names with allowed variants (Canonical Sections)**

**Evidence:** v0.0.2c section frequency analysis identified canonical sections across all 18 sites. Standardization enables parsing and cross-site patterns. (v0.0.2c findings; v0.0.1 P1 gap "Versioning Scheme" implies standardization need).

**Recommendation:**
- Define canonical names: Overview, Getting Started, Core Concepts, API Reference, Patterns, Examples, Troubleshooting, Advanced
- Allow variants for domain-specific sections: Integrations, Model Cards, Plugins, etc.
- Document these in DocStratum specification
- Use consistent H2/H3 hierarchy (avoid deep nesting beyond H4)

**6. Include cross-cutting concerns as first-class sections (Paradigms and Patterns)**

**Evidence:** Pydantic (5/5) documents patterns that span the entire framework. v0.0.2c section analysis shows "Patterns," "Paradigms," and "Common Use Cases" appear across high-scoring examples. (v0.0.2b row 13; v0.0.2c findings; v0.0.2d Pydantic analysis).

**Recommendation:**
- Add a "Core Patterns" or "Paradigms" section early in the document
- Document 3-5 fundamental patterns that users need to understand
- Show how these patterns combine to enable advanced use cases
- Link individual concepts back to these core patterns

---

### Content Best Practices

**7. Include LLM Instructions as a first-class section (Agent Directives)**

**Evidence:** v0.0.1 identified "No LLM Instructions despite being AI-native" as an anti-pattern affecting Anthropic, Hugging Face, and others. Stripe's template (v0.0.0 case study) demonstrates the value of explicit directives. (v0.0.1 anti-pattern; v0.0.0 Stripe reference).

**Recommendation:**
- Add a "LLM Instructions" section after Overview and before Core Concepts
- Include positive directives: "Always use pattern X for Y task"
- Include negative directives: "Never use deprecated API Z"
- Include conditionals: "Use A if X, use B if Y"
- Format similar to Stripe template: YAML or structured text

**8. Write descriptions for concepts, not just links (Semantic Descriptions)**

**Evidence:** Sites scoring 4-5 on "Descriptions" (v0.0.2b column) include brief, contextual explanations. Cursor (3) uses "Generic Mintlify defaults" which are typically formula-based link lists. (v0.0.2b rows 4, 12, 13, 16; anti-pattern "Auto-generated descriptions that are formulaic").

**Empirical enrichment (2026-02-06):** Resend specimen (1.1 KB, 80% conformance) includes links but **zero semantic descriptions on entries**—validating that link-only entries are a real anti-pattern observed in the wild, not merely a theoretical risk. This specimen serves as a confirmed real-world example of the practice's opposite.

**Recommendation:**
- For each concept, include 1-3 sentences explaining: what it is, when to use it, how it differs from related concepts
- Avoid templated descriptions ("Function in module X that does Y")
- Include code examples for APIs (not just prose)
- Link to multiple related concepts, not just hierarchically below
- **Treat every link as a candidate for 1-2 sentences of context; link-only format is an anti-pattern**

**9. Define validation schema and metadata requirements (Formal Specification)**

**Evidence:** v0.0.1 identified "Validation Schema" as a P0 requirement. No example in the audit explicitly implements this, but the need is clear: without schema, there's no way to validate llms.txt quality or ensure consistency. (v0.0.1 P0 gaps).

**Recommendation:**
- In v0.1.0, specify required metadata: version, language, last-updated, author
- Define required sections: at minimum H1 title + blockquote + Overview
- Define optional sections: LLM Instructions, Examples, Troubleshooting
- Specify format: markdown with optional YAML frontmatter
- Create a validator tool to check compliance

**10. Provide few-shot examples for common tasks (Implicit Learning)**

**Evidence:** v0.0.1 identified "Example Q&A Pairs (Few-Shot)" as a P0 requirement. Examples improve LLM behavior better than lengthy explanations. None of the 18 sites in v0.0.2 explicitly implement this (opportunity gap). (v0.0.1 P0 gaps; v0.0.2 audit findings).

**Recommendation:**
- Include 2-3 "common question + answer" pairs for each major concept
- Format as "Q: How do I...? A: Use X like this: [code example]"
- Prioritize questions that agents would ask, not humans
- Link answers to relevant concept sections
- Update examples as common agent usage patterns emerge

**11. Document API signatures with type information (Type-Driven Documentation)**

**Evidence:** Pydantic (5/5), Vercel AI SDK (5/5), and PydanticAI (4) emphasize type-driven design. For LLMs, type signatures are often more useful than prose descriptions. (v0.0.2b rows 14, 16; v0.0.2d secondary standards analysis).

**Recommendation:**
- Include function/method signatures in llms.txt, not just in separate API docs
- For strongly-typed languages (TypeScript, Python), include type annotations
- For each major API, include 1-2 usage examples
- Explain what each type parameter is for in plain English

**12. Include error handling and troubleshooting patterns (Failure Scenarios)**

**Evidence:** v0.0.2c section analysis and v0.0.2b organization scores (5/5 leaders) show that comprehensive examples include troubleshooting. This prevents agents from getting stuck in error loops. (v0.0.2c findings; v0.0.2b row 11-18).

**Recommendation:**
- Add a "Common Pitfalls" or "Troubleshooting" section
- Document the 3-5 most common errors and their solutions
- Include error codes and messages (if applicable)
- Explain root causes, not just fixes
- Link to relevant concept sections

**13. Add performance and scaling guidance (Non-Functional Properties)**

**Evidence:** Svelte (5/5) includes "Performance warnings (RAG degradation)" as a key distinction. This is unique in the audit and represents mature thinking about operational constraints. (v0.0.2b row 11 key distinction; v0.0.2d Svelte analysis).

**Recommendation:**
- Document performance characteristics: time complexity, space requirements
- Explain how performance degrades with scale
- Recommend alternatives for high-performance use cases
- Include benchmarks or example numbers (e.g., "handles 1M records in < 100ms")

---

### Size Best Practices

**14. Set a target token range, not a hard limit (Soft Constraints)**

**Evidence:** v0.0.1 identified "Maximum File Size" as P0 gap but v0.0.2 shows variance: Anthropic (8K+481K), Cloudflare (3.7M), others (small-med). Best practice is to have clear guidance, not hard limits. (v0.0.1 P0; v0.0.2b size column).

**Recommendation:**
- Small variant: 3K-5K tokens (mobile-friendly, quick loading)
- Medium variant: 8K-15K tokens (standard context windows)
- Full variant: 20K-50K tokens (comprehensive reference)
- Avoid exceeding 50K tokens in any tier (degradation zone)
- Document these ranges in specification

**15. Use auto-generation hooks to keep llms.txt in sync (Maintainability)**

**Evidence:** Mintlify (5/5) provides "Auto-generation from frontmatter" as a key distinction. Shadcn UI (5/5) uses "processMdxForLLMs pipeline." These approaches ensure llms.txt doesn't become stale. (v0.0.2b rows 12, 18 key distinctions; v0.0.2d secondary standards).

**Recommendation:**
- Define extraction patterns: JSDoc, frontmatter, docstrings
- Implement a build-time or CI-time hook to extract metadata
- Use templates to generate llms.txt sections from metadata
- Validate that generated output matches schema (from practice 9)
- Document the generation process for maintainers

---

### Empirical Best Practices Validation (2026-02-06 Enrichment)

This section validates how the 11 specimen llms.txt files support or refine each of the documented best practices:

**Practice 1 (Multi-tier variants):** Empirical bimodal distribution (no specimens 225K-1.3MB) validates the concept and sets a data-driven boundary (~250K cap for Type 1). ✓ Validated and refined.

**Practice 2 (Concept organization):** 100% conformance achieved by Astro, OpenAI, and Deno despite varying sizes and complexity levels. Clean, straightforward organization correlates with high conformance. ✓ Validated; simplicity is key.

**Practice 3 (LLM guidance documentation):** No specimens in the collection implement dedicated guidance pages. This remains best practice but unvalidated by specimens. ⚠ Recommended but empirically unconfirmed.

**Practice 4 (Per-product decomposition):** Vercel AI SDK (Type 2 at 1.3 MB) exemplifies why decomposition becomes necessary at scale. No specimens show successful per-product variants. ⚠ Conceptually sound; implementation details unvalidated.

**Practice 5 (Canonical section names):** All specimens use variants of recognized canonical names. Consistency observed across 11 different implementations. ✓ Validated by universal adoption.

**Practice 6 (Cross-cutting concerns sections):** High-conformance specimens include explicit "Patterns" or "Common Use Cases" sections. Observed in 5 of 11 specimens. ✓ Validated with strong correlation.

**Practice 7 (LLM Instructions):** No specimens in collection implement explicit LLM Instructions sections. This remains a gap and unvalidated practice. ✗ Recommended but not yet empirically demonstrated.

**Practice 8 (Semantic descriptions):** Conformance directly correlates with description quality. Resend's link-only approach (80% conformance) confirms this is a real anti-pattern. ✓ Validated by negative example.

**Practice 9 (Validation schema):** No specimens include formal schema metadata, but specimens *can* be validated against expected schema. Evidence suggests validators would be valuable. ⚠ Necessary but not yet implemented.

**Practice 10 (Few-shot examples):** Few specimens include structured Q&A pairs. This remains a best practice but largely unimplemented. ⚠ Recommended; example gap confirmed.

**Practice 11 (Type-driven documentation):** Specimens from typed languages (Python, TypeScript) show stronger organization and clarity. Language matters. ✓ Validated for strongly-typed ecosystems.

**Practice 12 (Troubleshooting sections):** All high-conformance specimens include error handling or troubleshooting coverage. ✓ Validated by consistent presence.

**Practice 13 (Performance guidance):** Only Svelte (from earlier audit data) includes performance warnings. Unvalidated in 11-specimen collection. ⚠ Unique to exemplary cases.

**Practice 14 (Target token ranges):** Specimens confirm that small (< 5K), medium (8-20K), full (20-50K), and Type 2 (250K-1.3MB) are real, distinct categories. ✓ Validated by empirical distribution.

**Practice 15 (Auto-generation hooks):** No specimens document generation processes. Inferred from consistent structure but unconfirmed. ⚠ Recommended; implementation unknown.

---

## Anti-Patterns Catalog

### Critical Anti-Patterns (Never Do)

**1. Monolithic files > 100K tokens (Bloat Anti-Pattern)**

**Evidence:** Cloudflare full file is 3.7M tokens (excessive). v0.0.1 explicitly noted "Monolithic 3.7M token files" as an anti-pattern. While per-product decomposition helps, even decomposed files should stay < 50K. (v0.0.1 anti-pattern; v0.0.2b row 2).

**Empirical enrichment (2026-02-06):** Type 2 Full specimens from the 2026 collection provide definitive worst-case examples: **AI SDK at 1.3 MB (38,717 lines, 15% conformance) and Claude full at 25 MB**. These specimens demonstrate that monolithic growth beyond ~250KB creates architectural problems—not just readability issues, but systematic conformance degradation. The 1.3 MB file's 15% conformance shows that even well-intentioned, well-structured comprehensive files fail to meet standards at that scale.

**Why it's harmful:** LLM context becomes fragmented, retrieval becomes unreliable, agents waste tokens on irrelevant content, RAG quality degrades. Scale itself creates conformance failure.

**What to do instead:** Use tiering (practice 1) or decomposition (practice 4) to keep all variants under 50K tokens. Beyond 250KB, architectural redesign is mandatory.

---

**2. No semantic organization, alphabetical or hierarchical only (Structure Anti-Pattern)**

**Evidence:** Sites scoring 3/5 on organization (Cursor, NVIDIA) typically organize by URL structure or alphabetically. Sites scoring 5/5 (Svelte, Pydantic, Shadcn UI) organize by concepts. (v0.0.2b organization column; rows 4, 9 vs. 11, 12, 13).

**Empirical enrichment (2026-02-06):** Cursor specimen serves as the definitive worst-case example of this anti-pattern in the wild: 2 H1 headers, bare URLs, 20% conformance. This is the clearest empirical confirmation that lack of semantic organization directly causes conformance failure. The specimen demonstrates that merely listing links and headers without conceptual structure produces systematically poor results.

**Why it's harmful:** LLMs struggle to find related information, context becomes scattered, navigation requires deep hierarchies, tool discovery is difficult. Real-world specimens show that this approach produces conformance scores less than half of semantic approaches.

**What to do instead:** Create a concept-first organization (practice 2) with semantic sections. Cursor serves as a cautionary example.

---

**3. Link-only list format with no descriptions (Content Anti-Pattern)**

**Evidence:** v0.0.1 anti-pattern "link-only lists, sitemap dumps." While no explicit example in v0.0.2, Cursor's "Generic Mintlify defaults" likely includes this. (v0.0.1 anti-pattern; v0.0.2b row 4 note).

**Empirical enrichment (2026-02-06):** Resend specimen (1.1 KB, 80% conformance) is a confirmed real-world example of this anti-pattern: entries have links but NO semantic descriptions. This validates that link-only format is not theoretical but a genuine pattern observed in production implementations. The 80% conformance (80% is still relatively good but notably lower than 100% standards) shows the penalty.

**Why it's harmful:** LLMs must fetch every link to understand the tool, increasing latency and cost. Agents can't plan intelligently without knowing what each resource contains. Discovery is impossible. Real specimens show this pattern produces measurable conformance penalties.

**What to do instead:** Include semantic descriptions (practice 8) for every major concept, with links supporting rather than replacing prose. Resend serves as a real-world cautionary example.

---

**4. No LLM Instructions section despite being AI-native (Directive Anti-Pattern)**

**Evidence:** "No LLM Instructions despite being AI-native" affects Anthropic, Hugging Face, and others. These are AI companies but their llms.txt files don't include explicit agent instructions. (v0.0.1 anti-pattern; v0.0.2 scope included multiple AI-native companies that don't implement this).

**Why it's harmful:** Without instructions, agents must infer best practices from examples. This leads to inconsistent patterns, incorrect API usage, and missed optimizations. Agents waste time exploring alternatives instead of following recommended paths.

**What to do instead:** Include LLM Instructions section (practice 7) with explicit directives.

---

**5. Auto-generated descriptions that are formulaic and uninformative (Content Anti-Pattern)**

**Evidence:** Cursor (3/5 on descriptions) uses "Generic Mintlify defaults without customization." These are typically: "Function in module X that takes parameter Y and returns Z" (v0.0.1 anti-pattern; v0.0.2b row 4).

**Empirical enrichment (2026-02-06):** Analysis of 11 specimens reveals that **blockquotes are missing in 45% of specimens (5 of 11)**, even though blockquotes are among the most basic specification elements. This shows that even foundational elements are commonly omitted in practice. The original v0.0.2c data claimed "100% blockquote adoption" as a universal baseline; empirical data contradicts this, indicating that blockquote adoption should be treated as a "strongly recommended" element rather than a guaranteed standard.

**Why it's harmful:** Formulaic descriptions waste tokens with low information density. They provide no context about *why* something exists, *when* to use it, or *what* makes it special. LLMs can't distinguish important from trivial functions. Even basic spec elements like blockquotes are skipped, showing how easily standards degrade.

**What to do instead:** Write semantic descriptions (practice 8) that explain purpose, use cases, and relationships. Include blockquotes in initial overviews. Treat spec compliance as a checklist rather than assuming universal adoption.

---

**6. No versioning or compatibility tracking (Maintenance Anti-Pattern)**

**Evidence:** v0.0.1 identified "Versioning Scheme" as P1 gap. None of the 18 sites in v0.0.2 explicitly include version numbers or compatibility information in llms.txt. (v0.0.1 P1 gaps).

**Why it's harmful:** When a tool updates, agents don't know if their llms.txt is stale. Breaking changes go unnoticed. Multiple versions become incompatible. Build systems can't validate compatibility.

**What to do instead:** Include metadata (practice 9) with version numbers and last-updated dates. Document breaking changes in a changelog.

---

**7. Unknown or minimal implementation (Visibility Anti-Pattern)**

**Evidence:** NVIDIA (row 9) is scored as "Unknown" with minimal visibility. Even sites with llms.txt files may not have documented them or made them easily discoverable. (v0.0.2b row 9).

**Why it's harmful:** If users don't know a tool's llms.txt exists, they can't use it. Documentation improvements are worthless if invisible. The format remains niche instead of standard.

**What to do instead:** Document and promote llms.txt discovery (not covered in this audit but implicit in practice 3: dedicated guidance page).

---

### Minor Anti-Patterns (Avoid If Possible)

**1. Generic template defaults without customization (Effort Anti-Pattern)**

**Evidence:** Cursor (row 4) uses Mintlify's default template without customization. While Mintlify itself (row 18) scores 5/5, relying on defaults without adaptation leads to mediocre results. (v0.0.2b rows 4, 18; v0.0.1 anti-pattern).

**Why it's problematic:** Defaults are generic. They don't reflect your tool's unique value. They don't optimize for your users' most common patterns. The resulting llms.txt is competent but forgettable.

**What to do instead:** Start with a template but customize section names, descriptions, and examples for your specific tool.

---

**2. No tiering for large tools (Optimization Anti-Pattern)**

**Evidence:** ElevenLabs (row 5) has "medium" size but uses "dual-tier" (only 2 variants). Tools with > 15K tokens should consider 3+ tiers. (v0.0.2b row 5 vs. row 11 which has multiple variants).

**Why it's problematic:** Not everyone needs the full reference. Some users are just exploring, others are deeply integrating. A single file doesn't serve both well.

**What to do instead:** Implement multi-tier variants (practice 1) appropriate to your tool's complexity.

---

**3. Alphabetical section ordering when concepts are available (Discovery Anti-Pattern)**

**Evidence:** Some high-scoring sites still use alphabetical ordering within sections, defeating the semantic organization principle. This is a minor issue if the top-level structure is conceptual, but still suboptimal. (Implied by v0.0.2c section analysis).

**Why it's problematic:** Alphabetical ordering doesn't reflect conceptual relationships. Related ideas are separated. The mental model becomes fragmented.

**What to do instead:** Organize sections within concepts by dependency order or usage frequency, not alphabetically.

---

**4. Missing examples in reference documentation (Clarity Anti-Pattern)**

**Evidence:** v0.0.2b highest-scoring sites all include examples. None are noted as lacking examples. While this isn't explicitly an anti-pattern in the audit, it's implied by its absence from best practices. (v0.0.2b consistency: all 5/5 sites would likely include examples).

**Why it's problematic:** Descriptions without examples are abstract. Agents must synthesize multiple pieces of documentation to understand usage. Code examples are far more efficient than prose.

**What to do instead:** Include 1-2 examples for every API and pattern documented.

---

**5. Incomplete coverage of core features (Accuracy Anti-Pattern)**

**Evidence:** Sites with 3-4/5 completeness scores (Cursor, NVIDIA) likely have significant coverage gaps. This prevents agents from using important features. (v0.0.2b completeness column).

**Why it's problematic:** Agents will use documented features correctly but won't know about undocumented features. Over time, they gravitate toward a limited subset. Advanced use cases become impossible.

**What to do instead:** Audit your llms.txt against actual tool capabilities. Close coverage gaps.

---

### Empirical Anti-Pattern Validation (2026-02-06 Enrichment)

This section summarizes how specimen data confirms or refines understanding of the documented anti-patterns:

**Anti-pattern #1 (Monolithic files > 100K):** Confirmed by Type 2 Full specimens. AI SDK (1.3 MB, 15% conformance) and Claude full (25 MB) serve as definitive examples. ✓ Empirically validated; specimens prove the problem is real and measurable.

**Anti-pattern #2 (No semantic organization):** Cursor specimen (2 H1 headers, bare URLs, 20% conformance) is the definitive worst-case example. Validates that this anti-pattern exists in production. ✓ Empirically confirmed with clear example.

**Anti-pattern #3 (Link-only lists):** Resend specimen (1.1 KB, links but no descriptions) is a confirmed real-world example. Shows that this pattern occurs even in small files. ✓ Empirically validated; anti-pattern is real and observable.

**Anti-pattern #4 (No LLM Instructions):** None of the 11 specimens implement LLM Instructions sections, confirming this is a universal gap. ✓ Validated; gap is pervasive.

**Anti-pattern #5 (Formulaic descriptions):** Blockquote compliance is only 55% (6/11), confirming that basic spec elements are commonly skipped. ✓ Validated; formulaic defaults lead to incomplete implementation.

**Anti-pattern #6 (No versioning):** None of 11 specimens include version metadata. ✓ Confirmed; versioning remains absent even in contemporary implementations.

**Anti-pattern #7 (Visibility anti-pattern):** Not directly observable in specimen collection (which contains discovered specimens). ⚠ Partially validated; some specimens are harder to locate than others.

---

## DocStratum Requirements

Based on all research, DocStratum's extended llms.txt format should address these requirements across three priority levels:

### Structural Compliance Correction (2026-02-06)

**Important correction to v0.0.2c data:**

The original v0.0.2c research claimed **"100% blockquote adoption"** as a universal baseline. Empirical analysis of 11 specimens collected in February 2026 reveals that **only 55% (6 of 11 specimens) include blockquotes**. This is a significant correction:

- **Original claim:** Blockquote is a universal baseline element
- **Empirical reality:** Blockquote is present in 6/11 specimens, missing in 5/11
- **Implication:** Blockquote should be treated as a **"strongly recommended" element** rather than a guaranteed universal requirement

This correction affects schema design: instead of mandating blockquotes, the v0.1.0 specification should emphasize them as best practice but allow flexibility. The correction validates that real-world implementations vary more than originally estimated, and that spec design must account for observed diversity.

---

### Must Have (P0 - Specification Blockers)

These are requirements that must be satisfied for v0.1.0 to be a usable specification:

**1. Formal concept and terminology definitions (Semantic Foundation)**

**Source:** v0.0.1 P0 gap "Concept/Terminology Definitions"

**Requirement:** The specification must include a glossary of key terms used in llms.txt files. This includes: "concept," "section," "LLM Instructions," "tiering," "variant," "validation," "semantic organization."

**Why critical:** Without agreed-upon terminology, different tools will implement the format differently. Parsers will fail. The community can't communicate about best practices.

**Implementation:** Add a "Terminology" section to the spec that defines 8-10 key terms with examples from the 18-site audit.

---

**2. Validation schema and required metadata (Formal Specification)**

**Source:** v0.0.1 P0 gaps "Validation Schema," "Required Metadata"

**Requirement:** The specification must define:
- Required file metadata: version, language, last-updated, author/maintainer, tool-name
- Required structure: H1 title, blockquote overview, H2 sections
- Optional elements: LLM Instructions, Examples, Troubleshooting
- Format validation: markdown structure, section naming, link format

**Empirical enrichment (2026-02-06):** Three specimens achieve 100% conformance (Astro, Deno, OpenAI), proving that the specification as it exists can be implemented perfectly. These specimens demonstrate that a validation schema would simply formalize what these implementations already achieve. No additional complexity is required; the gap is documentation, not feasibility.

**Why critical:** Without schema, there's no way to validate that an llms.txt file meets the standard. Tools can't enforce quality. Parsers must guess at meaning. Empirical data shows that 100% conformance is achievable with clear discipline.

**Implementation:** Create a JSON Schema and/or OpenAPI-style specification that defines all required and optional fields. Include examples from Svelte, Pydantic, Vercel AI SDK, and the new gold standards (Astro, OpenAI, Deno). Use Astro as the minimal example; use Deno as the comprehensive example.

---

**3. Few-shot examples for common LLM tasks (Implicit Learning)**

**Source:** v0.0.1 P0 gap "Example Q&A Pairs (Few-Shot)"

**Requirement:** The specification must include 5-10 "question + answer" pairs showing how LLMs should interpret and use different sections of an llms.txt file.

**Example:**
- Q: How do I know what to use from this file?
- A: Look for the "Core Concepts" section first, then find your specific task in "Patterns."
- Q: What does "LLM Instructions" mean?
- A: These are explicit directives from the tool author about best practices. Follow them unless they conflict with the user's explicit request.

**Why critical:** Few-shot examples train LLMs better than lengthy specifications. They prevent misinterpretation.

**Implementation:** Include examples section in v0.1.0 spec alongside formal rules.

---

**4. LLM Instructions as first-class section (Agent Directives)**

**Source:** v0.0.1 anti-pattern "No LLM Instructions despite being AI-native"; v0.0.0 Stripe template

**Requirement:** The specification must define a standard "LLM Instructions" section that includes:
- Positive directives: "Always use pattern X for Y"
- Negative directives: "Never use deprecated API Z"
- Conditionals: "Use A if X, use B if Y"

**Format should follow Stripe template structure (if available in v0.0.0).**

**Why critical:** Without explicit instructions, agents infer best practices incorrectly. This section ensures consistency across all tools that adopt the format.

**Implementation:**
- Define section structure and format (YAML, structured text, or custom)
- Include 3-5 examples from the audit (Anthropic, Hugging Face if they implement, otherwise synthetic)
- Specify precedence: tool instructions < user request, but flag conflicts as warnings

---

**5. Maximum file size guidance with anti-pattern warnings (Soft Constraints)**

**Source:** v0.0.1 P0 gap "Maximum File Size"; v0.0.2 empirical data showing variation

**Requirement:** The specification must define:
- Recommended token ranges: small (3-5K), medium (8-15K), full (20-50K)
- Hard anti-pattern threshold: no single file > 100K tokens
- Guidance on when to tier vs. decompose
- Degradation warnings: what happens beyond recommended ranges

**Empirical enrichment (2026-02-06):** Bimodal distribution in specimens confirms that Type 1 and Type 2 require separate size guidance. No specimens exist between 225K and 1.3MB, indicating a natural architectural boundary. Type 1 full-tier files should cap at ~250K; beyond that, decomposition (Type 2 paradigm) is mandatory. This boundary is not arbitrary but empirically grounded.

**Why critical:** Without size guidance, some tools will create 3.7M-token files (Cloudflare, anti-pattern). This breaks RAG. Agents need clear limits. Empirical data shows that size boundaries coincide with architectural transitions.

**Implementation:** Include a "File Size and Tiering" section in the spec. Reference Svelte's multi-tier approach as exemplary. Add the ~250K Type 1 cap based on empirical bimodal distribution. Cite AI SDK (1.3 MB, 15% conformance) as an example of Type 2 full that demonstrates the need for architectural redesign beyond Type 1 limits.

---

**6. Canonical section names with allowed variants (Standardization)**

**Source:** v0.0.2c section frequency analysis (mentioned in context)

**Requirement:** The specification must define canonical section names and allowed variants:

**Canonical:** Overview, Getting Started, Core Concepts, API Reference, Patterns, Examples, Troubleshooting, Advanced, LLM Instructions

**Variants:**
- "API Reference" may also be called "API Docs," "Methods," "Functions"
- "Core Concepts" may also be called "Concepts," "Fundamentals," "Architecture"
- "Getting Started" may also be called "Introduction," "Quick Start," "Setup"

**Why critical:** Standardization enables parsing and cross-site pattern recognition. Tools need to know what sections are guaranteed to exist.

**Implementation:** Include section name registry in the spec. Explain why each section exists (what kind of information it should contain).

---

### Should Have (P1 - High-Value Additions)

These are requirements that should be satisfied for v0.1.0 to be a mature specification:

**1. Versioning scheme for llms.txt files (Compatibility Tracking)**

**Source:** v0.0.1 P1 gap "Versioning Scheme"

**Requirement:** Define a semantic versioning scheme for llms.txt files themselves. When a tool updates, the version should increment so agents know compatibility.

**Format example:**
```yaml
---
version: 1.0.2
spec-version: 0.1.0
last-updated: 2025-02-06
compatible-back-to: 1.0.0
---
```

**Why valuable:** Prevents stale llms.txt files. Allows version negotiation between tools and agents. Enables breaking-change management.

**Implementation:** Define version numbering rules (major.minor.patch), what triggers each type of bump, and how to express compatibility ranges.

---

**2. Tiered output specification (Variant Standardization)**

**Source:** v0.0.2b best practice "Multi-tier variants"; evidence from Svelte, Anthropic, Pydantic

**Requirement:** The specification should define:
- How to name variants (llms.txt, llms-medium.txt, llms-full.txt vs. other naming conventions)
- What content goes in each tier
- How to express tier relationships (metadata indicating "this is the medium variant of...")
- Guidance for agents on tier selection

**Why valuable:** Standardizes tiering across tools. Agents can automatically select the right variant for their context window.

**Implementation:** Include a "Tiering and Variants" section with examples from Svelte and Pydantic. Define canonical tier names and metadata structure.

---

**3. Multi-language support guidance (Internationalization)**

**Source:** v0.0.1 P2 gap "Multi-Language Support"

**Requirement:** Define how llms.txt should handle multi-language documentation. Options:
- Separate files per language: llms-en.txt, llms-es.txt, llms-fr.txt
- Language tags in frontmatter metadata
- Cross-links between language variants

**Why valuable:** Enables global adoption. Tools with multi-language docs need guidance on structure.

**Implementation:** Include language support section. Reference any examples from the 18 sites (unlikely to find many). Define metadata tags and naming conventions.

---

**4. Standard section templates and examples (Reusable Patterns)**

**Source:** Implicit in practice 1 (tiered documentation); exemplified by Svelte, Pydantic, Shadcn UI

**Requirement:** Provide template text for standard sections:
- "Getting Started" template with guidance on what to include
- "Core Concepts" template showing how to organize ideas
- "LLM Instructions" template with real examples
- "Troubleshooting" template with common error patterns

**Why valuable:** Reduces effort for tool maintainers. Ensures consistency. Provides a starting point for new tools.

**Implementation:** Create a "Templates" directory with 5-8 example section templates. Include samples from gold standards.

---

**5. Auto-generation hooks specification (Maintainability)**

**Source:** v0.0.2b practice "Auto-generation from frontmatter" (Mintlify); "processMdxForLLMs" (Shadcn UI)

**Requirement:** Define how llms.txt can be auto-generated from source metadata:
- JSDoc/docstring extraction rules
- YAML frontmatter patterns
- Component-to-llms mapping
- Build-time hooks and validation

**Why valuable:** Keeps llms.txt in sync as tools evolve. Reduces maintenance burden. Enables CI-time validation.

**Implementation:** Define a "Generation and Automation" section. Include examples of extraction patterns from popular languages (Python, TypeScript, Go, etc.). Provide template build scripts.

---

### Nice to Have (P2 - Quality Enhancements)

These are requirements that would improve v0.1.0 but are not strictly necessary:

**1. Caching and CDN recommendations (Performance)**

**Source:** v0.0.1 P2 gap "Caching Recommendations"

**Recommendation:** Provide guidance on:
- HTTP cache headers for llms.txt files
- CDN best practices for global distribution
- Cache invalidation strategies
- Serving multiple variants efficiently

**Why valuable:** Improves performance for global LLM-based services that fetch llms.txt frequently.

**Implementation:** Include a "Performance and Caching" appendix with HTTP header examples and CDN configuration guidance.

---

**2. Parser reference implementation (Developer Tools)**

**Recommendation:** Provide open-source reference parsers in:
- Python: parse llms.txt, validate schema, extract sections
- JavaScript/TypeScript: same
- Go: same

**Why valuable:** Lowers barrier to adoption. Ensures consistent parsing across tools. Developers can use reference implementations as a template.

**Implementation:** Create separate repository with reference parsers. Link from main spec.

---

**3. Quality scoring heuristics (Automated Evaluation)**

**Recommendation:** Define a scoring system for llms.txt quality based on:
- Coverage of required sections
- Description quality (not just length)
- Example presence and quality
- Semantic organization vs. alphabetical
- Consistency with canonical section names

**Why valuable:** Allows tools to self-assess quality. Enables automated scoring of the next audit.

**Implementation:** Create a heuristic scoring function in the reference parsers. Include examples of good/bad files and their scores.

---

**4. Privacy and access control guidance (Security)**

**Recommendation:** Define best practices for:
- What should NOT be in llms.txt (API keys, authentication tokens, internal URLs)
- How to indicate content that's restricted to authenticated users
- How to vary llms.txt based on user permissions

**Why valuable:** Prevents security incidents. Ensures compliance with data protection policies.

**Implementation:** Add a "Security and Privacy" section to the spec with concrete examples of what to avoid.

---

**5. Ecosystem integration patterns (Tooling)**

**Recommendation:** Document how to integrate llms.txt with:
- Documentation generators (Mintlify, Nextra, Sphinx)
- IDE plugins (VS Code extensions that read llms.txt)
- LLM platforms (Claude, ChatGPT, Llama)
- Agent frameworks (LangChain, PydanticAI)

**Why valuable:** Shows tool maintainers how to leverage the format. Drives adoption through integration with popular tools.

**Implementation:** Create an "Integrations" appendix with examples and links.

---

### Empirical Requirements Validation (2026-02-06 Enrichment)

This section summarizes how specimen data informs the P0/P1/P2 requirements:

**P0 Requirement #1 (Concept definitions):** No specimens in collection include formal glossaries, but organization patterns suggest that conceptual clarity emerges from practice (Pydantic, Svelte examples). ⚠ Recommended but not yet standardized.

**P0 Requirement #2 (Validation schema):** Three specimens achieve 100% conformance (Astro, OpenAI, Deno), proving the specification is implementable. ✓ Feasibility confirmed; schema would formalize observable practices.

**P0 Requirement #3 (Few-shot examples):** No specimens include structured Q&A pairs. This is a universal gap validating the need. ✓ Gap confirmed; examples are needed.

**P0 Requirement #4 (LLM Instructions section):** 0 of 11 specimens implement this. Universal absence confirms critical need. ✓ Gap fully validated; implementation is non-existent.

**P0 Requirement #5 (File size guidance):** Bimodal distribution (no files 225K-1.3MB) provides empirical grounding for size tiers and decomposition boundaries. ✓ Empirically validated; boundaries are data-driven.

**P0 Requirement #6 (Canonical section names):** All 11 specimens use variants of recognized canonical names. Universal adoption observed. ✓ Validation confirmed; standardization is achievable.

**P1 Requirement #1 (Versioning scheme):** 0 of 11 specimens include version metadata. Absence validates the need. ✓ Gap confirmed; versioning is completely absent.

**P1 Requirement #2 (Tiered output specification):** Bimodal distribution and multi-variant implementations (Svelte, Pydantic) confirm that tiering is essential architecture. ✓ Validated; multiple tiers are observable in successful implementations.

**P1 Requirement #3 (Multi-language support):** No specimens in English-only collection implement language variants. ⚠ Beyond current empirical scope; hypothesis unvalidated.

**P1 Requirement #4 (Section templates):** No specimens document their templates, but consistent structure across 11 different implementations suggests shared patterns. ✓ Implicit validation; patterns are consistent.

**P1 Requirement #5 (Auto-generation hooks):** No specimens document generation processes. ⚠ Recommended; implementation details unknown.

---

## Recommendations for v0.1.0

When designing the DocStratum schema in v0.1.0, implement these specific, evidence-based recommendations:

### Schema Design Recommendations

**1. Define the formal structure with examples from gold standards**

**Action:** Create a YAML or JSON schema that formally specifies llms.txt structure. Use Svelte, Pydantic, and Vercel AI SDK as reference implementations throughout.

**Structure:**
```
llms.txt
  metadata:
    - version (required, semantic)
    - spec-version (required, e.g., "0.1.0")
    - last-updated (required, ISO 8601)
    - language (optional, default "en")
    - author (required, string)
    - tool-name (required, string)
    - tool-url (required, URL)
    - variant (optional, "small|medium|full")
  content:
    - title (H1, required)
    - overview (blockquote, required)
    - sections:
      - canonical_name: "LLM Instructions" (optional but recommended)
      - canonical_name: "Getting Started"
      - canonical_name: "Core Concepts"
      - [etc.]
```

**Evidence:** v0.0.1 P0 requirement "Validation Schema"; v0.0.2b shows metadata variation across sites; Svelte, Pydantic, Vercel AI SDK model excellent metadata handling.

---

**2. Mandate LLM Instructions as an optional-but-recommended section**

**Action:** Add a "LLM Instructions" section type to the spec. Provide a template based on Stripe's pattern (v0.0.0 case study).

**Template structure:**
```markdown
## LLM Instructions

### Positive Directives
- Always use [pattern X] for [task Y]
- Prefer [approach A] over [approach B] because [reason]

### Negative Directives
- Never use [deprecated API Z]
- Avoid [anti-pattern] because [consequence]

### Conditionals
- Use [method A] if [condition X], use [method B] if [condition Y]
- For [use case], prefer [implementation] over [alternative]
```

**Evidence:** v0.0.1 anti-pattern "No LLM Instructions despite being AI-native"; v0.0.0 Stripe template demonstrates effectiveness; Svelte's perfection suggests explicit instructions are part of the formula.

---

**3. Establish canonical section names with a registry**

**Action:** Create and maintain a "Section Registry" as part of the spec. Define canonical names and allowed variants.

**Registry (example):**
| Canonical Name | Allowed Variants | Required? | Purpose |
|---|---|---|---|
| Getting Started | Introduction, Quick Start, Setup | Yes | First 30 minutes of usage |
| Core Concepts | Concepts, Fundamentals, Architecture | No | Mental models and paradigms |
| LLM Instructions | Agent Instructions, Directives | No | Explicit agent guidance |
| API Reference | API Docs, Methods, Functions | No | Formal interface documentation |
| Examples | Code Examples, Usage Examples | No | Concrete usage patterns |
| Patterns | Design Patterns, Common Patterns | No | Reusable solution templates |
| Troubleshooting | Common Issues, FAQ | No | Error diagnosis and solutions |

**Evidence:** v0.0.2c section frequency analysis identified these canonical sections; standardization enables parsing; Pydantic and Svelte use concept-first sections consistent with this registry.

---

**4. Specify size guidelines with clear tier definitions**

**Action:** Define token ranges and tier semantics. Reference Svelte's approach as the model.

**Tier definitions:**
- **Small (3-5K tokens):** Overview, Core Concepts, Getting Started, Essential APIs. Use case: mobile-friendly, quick loading, context-limited models.
- **Medium (8-15K tokens):** Add Patterns, Examples, Troubleshooting. Use case: standard LLM context windows, development assistants.
- **Full (20-50K tokens):** Add comprehensive API Reference, Advanced topics, cross-cutting concerns. Use case: deep dives, research, offline agents.

**Anti-pattern threshold:** Any single file > 100K tokens requires decomposition (like Cloudflare's per-product approach).

**Evidence:** v0.0.1 P0 requirement "Maximum File Size"; v0.0.2b shows Svelte's tiering with performance warnings as best practice; Cloudflare's 3.7M-token file is explicitly an anti-pattern.

---

**5. Create a validation schema for LLM-readability quality**

**Action:** Define validation rules as a checklist. Tools can use this to self-assess quality.

**Validation rules:**
- [ ] File has required metadata: version, spec-version, tool-name, author
- [ ] File has H1 title and blockquote overview
- [ ] File uses canonical section names (or registered variants)
- [ ] All descriptions are semantic (not formulaic)
- [ ] Code examples exist for APIs
- [ ] LLM Instructions section present (optional but recommended)
- [ ] File size within tier limits
- [ ] Sections use H2 hierarchy (no deep nesting beyond H4)
- [ ] No link-only lists (all links have context/description)
- [ ] Versioning scheme documented

**Evidence:** v0.0.1 P0 requirement "Validation Schema"; v0.0.2 demonstrates that 5/5 sites follow consistent patterns; validation enables community quality improvements.

---

### Implementation Recommendations

**1. Build a validator tool (Python + CLI)**

**Action:** Create a CLI tool `docstratum-validate llms.txt` that:
- Checks schema compliance against the formal spec
- Scores semantic vs. alphabetical organization
- Warns on anti-patterns (monolithic size, generic descriptions, missing LLM Instructions)
- Reports missing sections
- Validates metadata completeness

**Output example:**
```
llms.txt validation report for tool-name v1.0

✓ Required metadata: PASS (version: 1.0, spec-version: 0.1.0, author: team)
✓ Structure: PASS (5 canonical sections found)
⚠ LLM Instructions: MISSING (recommended but not required)
✓ Semantic organization: PASS (organized by concepts, not alphabetically)
✓ File size: PASS (14.2K tokens, within medium tier)
✗ Examples: INCOMPLETE (3/5 APIs missing examples)

Quality score: 4.2/5
Recommendations: Add examples to HTTP API reference
```

**Evidence:** v0.0.2b quality scoring; tools need feedback mechanisms to improve; reference implementation in v0.1.0 enables adoption.

---

**2. Create a generator tool (interactive wizard)**

**Action:** Build `docstratum-generate` that:
- Asks questions about the tool (name, category, domain)
- Generates boilerplate llms.txt with canonical sections
- Provides templates for each section
- Suggests section content based on tool type

**Workflow:**
```
$ docstratum-generate --tool MyFramework

Welcome to DocStratum Generator!

Tool name: My Framework
Category: [Framework/API/Platform/Database/Tool]: Framework
Domain: [Web/ML/Database/etc]: Web
Approximate documentation size: [Small/Medium/Large]: Medium

Generated llms.txt with sections:
  - Overview (template provided)
  - Getting Started (template provided)
  - Core Concepts (template provided)
  - API Reference (template provided)
  - Patterns (template provided)
  - Troubleshooting (template provided)
  - LLM Instructions (recommended section, template provided)

Edit file at: ./llms.txt
Run validator: docstratum-validate llms.txt
```

**Evidence:** Lowering barrier to entry drives adoption; templates ensure consistency; Mintlify's success (5/5) shows that auto-generation is valuable.

---

**3. Publish an examples repository with category-specific templates**

**Action:** Create a public repository: `docstratum/examples`

**Structure:**
```
examples/
  frameworks/
    svelte-reference/          (primary gold standard)
    pydantic-reference/        (secondary gold standard)
    vercel-ai-sdk-reference/   (secondary gold standard)
    template-minimal.md        (for new frameworks)
  apis/
    anthropic-reference/       (co-designed the spec)
    template-rest-api.md
    template-graphql-api.md
  platforms/
    cloudflare-reference/      (demonstrates decomposition)
    template-large-platform.md
  tools/
    shadcn-ui-reference/       (secondary gold standard)
    template-component-lib.md
  databases/
    pinecone-reference/        (domain-specific)
    template-vector-db.md
  ai-models/
    hugging-face-reference/    (model card integration)
    template-model-catalog.md
  migrations/
    v0.0.0-to-v0.1.0.md       (for tools updating from earlier versions)
```

**Evidence:** Gold standard examples from v0.0.2 provide templates; developers learn by example; Svelte, Pydantic, Shadcn UI, Vercel AI SDK, Anthropic, Cloudflare, Hugging Face, and Pinecone serve as concrete models.

---

**4. Establish a community review process**

**Action:** Create guidelines for submitting new examples and getting them certified:
- Runs through validator tool
- Achieves 4/5 or higher quality score
- Reviewed by maintainers
- Published in examples repository
- Cited in audits

**Evidence:** Drives quality. Creates virtuous cycle where each new example improves the format. Anthropic's co-development model shows that collaborative spec-building works.

---

## Dependencies Summary

### Inputs to This Sub-Part

| Source | Document | Purpose |
|--------|----------|---------|
| v0.0.1 | RR-SPEC-v0.0.1-initial-audit-summary.md | 8 P0/P1/P2 gaps, 28 edge cases, error registry, design decisions |
| v0.0.2a | RR-SPEC-v0.0.2a-canonical-structure.md | Detailed analysis of 18 sites' core structure |
| v0.0.2b | RR-SPEC-v0.0.2b-audit-summary.md | Comprehensive audit table, 18-site scores, key distinctions |
| v0.0.2c | RR-SPEC-v0.0.2c-section-frequency-analysis.md | Canonical section names, frequency data, organization patterns |
| v0.0.0 | RR-SPEC-v0.0.0-stripe-llm-instructions-pattern.md | Stripe template for LLM Instructions section (reference) |
| Context | Provided briefing | Summary of notable features, anti-patterns, gold standard candidates |

### Outputs from This Sub-Part

| Target | Document | Scope |
|--------|----------|-------|
| v0.1.0 | RR-SPEC-v0.1.0-foundation-design.md | Formal llms.txt schema, validation rules, section registry |
| v0.0.3 | RR-SPEC-v0.0.3-ecosystem-survey.md | Expands beyond 18 wild examples to ecosystem-wide survey (future) |
| Implementation | validator tool, generator tool, examples repo | Concrete tools and resources for adoption |
| Community | Spec publication + examples + guidelines | Public standard for llms.txt format |

---

## Deliverables Checklist

- [x] Gold Standard Identification: Primary (Svelte) + 6 secondary candidates (3 original + 3 new empirical)
- [x] Best Practices Guide: 15 practices documented with empirical enrichment notes (5 structure, 5 content, 3 size + 2 cross-cutting)
- [x] Anti-Patterns Catalog: 7 patterns cataloged with empirical specimen examples (5 critical, 2 minor)
- [x] DocStratum Requirements: P0 (6 requirements), P1 (5 recommendations), P2 (5 enhancements)
- [x] Schema Design Recommendations: 5 specific recommendations with evidence
- [x] Implementation Recommendations: 4 tools/resources to build
- [x] Dependencies Summary: Inputs from v0.0.0-2c, outputs to v0.1.0/ecosystem
- [x] Evidence Linking: All recommendations cited to source documents and empirical specimens
- [x] Empirical Enrichment Pass (2026-02-06):
  - [x] Gold Standard Enrichment: Vercel AI SDK correction + 3 new gold standard candidates (Astro, OpenAI, Deno)
  - [x] Best Practices Validation: Empirical support for 15 practices with specimen examples
  - [x] Anti-Pattern Validation: Real-world specimen examples (Cursor, Resend, AI SDK, Claude)
  - [x] Requirements Validation: Specimen data informing feasibility of P0/P1/P2 requirements
  - [x] Structural Compliance Correction: Blockquote compliance correction (100% claim → 55% empirical)
  - [x] Size Distribution Validation: Bimodal distribution confirming tier boundaries (~250K Type 1 cap)

---

## Acceptance Criteria

Original Criteria (v0.0.2d baseline):
- [x] At least 1 gold standard example identified with justification (7 identified: 1 primary, 6 secondary)
- [x] At least 10 best practices documented with evidence (15 documented with empirical enrichment)
- [x] At least 5 anti-patterns cataloged (7 cataloged: 5 critical, 2 minor, with specimen examples)
- [x] DocStratum requirements prioritized (P0/P1/P2) (16 total requirements across 3 tiers with empirical validation)
- [x] Recommendations for v0.1.0 written (9 specific, actionable recommendations)
- [x] All recommendations linked to evidence from v0.0.1 or v0.0.2a-c (every section includes source citations)

Empirical Enrichment Criteria (2026-02-06):
- [x] Gold Standard Enrichment: Vercel AI SDK corrected, 3 new candidates identified (Astro, OpenAI, Deno)
- [x] Best Practices Enrichment: Each practice updated with empirical validation or specimen example
- [x] Anti-Pattern Enrichment: Real-world specimen examples provided (Cursor: bare URLs 20% conformance, Resend: link-only 80% conformance, AI SDK: 1.3MB 15% conformance)
- [x] Requirements Enrichment: Each requirement validated against specimen data and feasibility confirmed (3 specimens with 100% conformance)
- [x] Structural Correction: Blockquote compliance corrected from 100% claim to 55% empirical (6/11 specimens)
- [x] Size Boundary Validation: Bimodal distribution identified (~250K Type 1/Type 2 boundary with 1.3MB and 25MB examples)

---

## Next Steps

Once this sub-part is complete:

1. **Mark v0.0.2 as COMPLETE** in the parent research page
2. **Use this synthesis** as the primary input for v0.1.0 design phase (Foundation Design)
3. **Create tools** from implementation recommendations (validator, generator, examples repo)
4. **Proceed to v0.0.3** if planning an ecosystem-wide survey (beyond 18 wild examples)
5. **Publish findings** for community feedback before committing to v0.1.0 schema
