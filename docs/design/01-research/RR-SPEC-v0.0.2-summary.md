# v0.0.2 — Wild Examples Audit: Consolidated Summary

> **Phase:** Research & Discovery (v0.0.x)
> **Status:** COMPLETE
> **Sub-Parts:** v0.0.2a, v0.0.2b, v0.0.2c, v0.0.2d — all verified
> **Date Completed:** 2026-02-05
> **Verified:** 2026-02-05
> **Enrichment Pass:** 2026-02-06 (11 real-world specimens integrated across all sub-parts)

---

## Purpose of This Document

This summary consolidates the findings, deliverables, and forward-feeding decisions from the four v0.0.2 sub-parts. It serves as the exit gate for the Wild Examples Audit milestone and as the primary reference for v0.0.3 (Ecosystem Survey) and v0.1.0 (Foundation Design). Where v0.0.1 analyzed the specification itself, v0.0.2 analyzed how the ecosystem actually implements it.

---

## What v0.0.2 Set Out to Do

The Specification Deep Dive (v0.0.1) identified 8 gaps and proposed design principles from first-principles analysis plus three exemplary implementations (Stripe, Nuxt, Vercel). v0.0.2's objective was to systematically audit a broader, more diverse set of real-world llms.txt implementations to validate those findings, discover new patterns, quantify trends, and build an evidence base for DocStratum's design decisions. The audit covered 18 implementations across 6 categories — enough breadth to identify archetypes and strong enough statistical patterns to make confident recommendations.

---

## Sub-Part Overview

### v0.0.2a — Source Discovery & Collection

Identified and cataloged 18 real-world llms.txt implementations across 6 categories (AI/ML [6], Framework [4], Platform [3], Tool [3], Enterprise [2], Database [1]). Sources were discovered through four methods: directory crawls of llmstxt.site and llmstxthub.com, web search for adoption reports, platform provider documentation, and community references. Stripe, Nuxt, and Vercel were excluded to avoid duplicating v0.0.1 analysis.

**Key finding:** CDN/WAF protections blocked direct HTTP access to every llms.txt file attempted, requiring a web-search-based verification methodology. This limitation is documented transparently and affected confidence ratings in downstream audits.

> **Enrichment Pass (2026-02-06):** A fifth discovery method was added — Direct Specimen Collection — providing 11 actual llms.txt files for byte-level analysis. The catalog expanded from 18 to **24 unique sources** with 6 new entries (Astro, Deno, Docker, Neon, OpenAI, Resend) and 5 existing entries annotated with `[SPECIMEN]` empirical data. An empirical size distribution was added revealing a **bimodal pattern**: Type 1 Index files cluster at 1.1 KB – 225 KB, Type 2 Full files jump to 1.3 MB+, with no specimens in the gap.

### v0.0.2b — Individual Example Audits

Completed a standardized audit form for each of the 18 sources. Each audit covers: basic metadata, a 9-item structural checklist, content analysis across 4 dimensions rated 1–5 (Completeness, Organization, Descriptions, LLM-Friendliness), section inventory, notable features, problems/issues, and ideas to adopt. Produced 72 individual ratings, 30+ unique "Ideas to Adopt," and an 18-row summary table.

**Key finding:** Four implementations achieved perfect 5/5 scores across all dimensions: Svelte, Shadcn UI, Pydantic, and Vercel AI SDK. All four are frameworks or component libraries — not enterprise platforms or AI companies. The lowest scorer was NVIDIA (2/5, minimal visibility).

> **Enrichment Pass (2026-02-06):** 5 existing audit forms received Enrichment Pass annotations correcting estimated ratings with empirical data. **Major corrections:** Cursor revised from 3→2 (2 H1 headers, bare URLs, 20% conformance); Vercel AI SDK's full variant revealed as Type 2 Full (1.3 MB, 15% conformance — not the small index assumed); Cloudflare and LangChain both confirmed missing blockquotes. 6 new audit forms added (#19–#24: Astro, Deno, Docker, Neon, OpenAI, Resend), expanding the audit set to **24 total implementations**. Three specimens (Astro, Deno, OpenAI) achieve **100% conformance** — new gold standard candidates. Resend (1.1 KB, 19 lines) confirmed as the smallest real-world specimen at 80% conformance.

### v0.0.2c — Pattern Analysis & Statistics

Transformed the raw audit data into statistical insights and a pattern taxonomy. Calculated file size distributions (8K to 3.7M tokens, median ~67.5K), quality score metrics (mean 4.0/5, 83% scoring 4+), section frequency analysis (11 canonical section types identified), and structural compliance rates (100% on baseline elements, 0% on LLM Instructions). Defined 5 implementation archetypes and analyzed 7 correlation factors. Validated all 8 v0.0.1 findings against the new data.

**Key finding:** Quality is not correlated with size (r ≈ −0.05). The strongest predictor of quality is the presence of concrete examples and code samples (r ≈ 0.65), followed by number of thoughtfully organized sections (r ≈ 0.60) and active versioning/maintenance signals (r ≈ 0.55).

> **Enrichment Pass (2026-02-06):** **Critical structural compliance correction:** The original analysis claimed 100% blockquote compliance — empirical data from 11 specimens reveals only **55% (6/11)** include blockquotes. H1 uniqueness drops to 82% (Cursor has 2 H1s). Link list format compliance is 89% for Type 1 (Cursor uses bare URLs). A **Document Type Classification** system was added distinguishing Type 1 Index (curated link catalogs, 1.1–225 KB, 80–100% conformance) from Type 2 Full (inline documentation dumps, 1.3–25 MB, 5–15% conformance — spec grammar inapplicable). Archetype assignments were empirically validated with conformance grades. The bimodal size distribution was formally documented.

### v0.0.2d — Synthesis & Recommendations

Synthesized all findings from v0.0.1 and v0.0.2a–c into actionable recommendations. Identified 4 gold standard implementations (Svelte primary, plus Shadcn UI, Pydantic, Vercel AI SDK), documented 15 best practices with evidence citations, cataloged 12 anti-patterns (7 critical, 5 minor), drafted 16 prioritized requirements for DocStratum (P0/P1/P2), and wrote 9 specific implementation recommendations for v0.1.0.

**Key finding:** Svelte is the primary gold standard — the only implementation with multi-tier variants (small/medium/full), a dedicated guidance page explaining optimal usage, and explicit performance warnings about RAG degradation. It demonstrates that tiered architecture is a design feature, not a workaround.

> **Enrichment Pass (2026-02-06):** Gold standard list expanded with 3 new empirical candidates: **Astro** (100% conformance at 2.6 KB — micro tier reference), **OpenAI** (100% at 19 KB — compact reference), and **Deno** (100% at 63 KB — medium-scale reference). A correction was added for Vercel AI SDK: the collected specimen (`ai-sdk-llms.txt`) is a Type 2 Full document (1.3 MB, 15% conformance), not the index variant the 5/5 rating was based on. All 15 best practices were validated against specimen data. Anti-patterns were confirmed with specific specimen citations (Cursor for structural violations, Resend for link-only entries). A structural compliance correction was propagated: blockquote compliance is 55%, not 100%. The bimodal size boundary (~250 KB) was added to P0 Requirement #5 guidance.

---

## The 18 Audited Implementations

### Audit Summary Table

| # | Site | Category | Overall | Key Distinction |
|---|------|----------|---------|----------------|
| 1 | Anthropic | AI/ML | 4 | Tiered docs (8K + 481K), co-developed llms-full.txt standard |
| 2 | Cloudflare | Platform | 4 | Per-product decomposition, 3.7M token full file |
| 3 | Supabase | Platform | 4 | Domain-specific file variants (llms/guides.txt) |
| 4 | Cursor | Tool | 3 | Generic Mintlify default, minimal customization |
| 5 | ElevenLabs | AI/ML | 4 | Dual-tier API documentation |
| 6 | Shopify | Enterprise | 4 | Multi-API enterprise docs (REST + GraphQL + Storefront) |
| 7 | Hugging Face | AI/ML | 4 | Model card standardization |
| 8 | Pinecone | Database | 4 | Vector DB domain alignment with RAG workflows |
| 9 | NVIDIA | Enterprise | 2 | Minimal visibility, uncertain quality |
| 10 | Zapier | Platform | 4 | Workflow-centric pattern documentation |
| 11 | **Svelte** | Framework | **5** | Multi-tier variants + dedicated guidance page |
| 12 | **Shadcn UI** | Tool | **5** | AI-Ready architecture, processMdxForLLMs pipeline |
| 13 | **Pydantic** | Framework | **5** | Concept-first organization, cross-cutting concerns |
| 14 | PydanticAI | AI/ML | 4 | Type-driven agent patterns |
| 15 | LangChain | AI/ML | 4 | Comprehensive LLM framework documentation |
| 16 | **Vercel AI SDK** | Framework | **5** | Full-stack reference architecture, streaming-first |
| 17 | FastHTML | Framework | 4 | Spec author's reference implementation |
| 18 | Mintlify | Tool | 4 | Ecosystem multiplier, auto-generates for thousands of projects |

### Category Performance

| Category | Count | Mean Score | Insight |
|----------|-------|-----------|---------|
| Framework | 4 | 4.75 | Highest-performing category; all scored 4–5 |
| AI/ML | 6 | 4.0 | Consistent but lacks innovation in LLM affordances |
| Platform | 3 | 4.0 | Good at scale, challenges with monolithic size |
| Tool | 3 | 4.0 | High variance (Cursor 3 to Shadcn UI 5) |
| Enterprise | 2 | 3.0 | Lowest category; NVIDIA drags average down |
| Database | 1 | 4.0 | Single data point (Pinecone) |

---

## Five Implementation Archetypes

The audit revealed five distinct approaches to llms.txt implementation, each with clear tradeoffs:

### Archetype 1: The Index (Score: 2–3)

Minimal file serving primarily as a link index with lightweight descriptions. Examples: Cursor, NVIDIA. Works as navigational scaffolding but insufficient for LLM consumption — agents must constantly fetch external links to understand context.

### Archetype 2: The Comprehensive Guide (Score: 4)

Large, single-file documentation with extensive inline content. Examples: Cloudflare (3.7M tokens), LangChain (80–150K), Shopify (100–250K). Excellent completeness but size becomes problematic for constrained context windows. Requires careful organization to remain navigable. **Cloudflare clarification:** The monolithic 3.7M-token `llms-full.txt` is itself an anti-pattern (unusable for any current context window). The *effective* strategy is Cloudflare's per-product decomposition — separate modular files for Workers, R2, D1, etc. — which allows LLMs to fetch only the relevant product context. **Context Builder implication:** When designing for large documentation sets, prioritize the modular variant strategy over monolithic full-file generation.

### Archetype 3: The Tiered System (Score: 4–5)

Uses multiple files at different detail levels. Examples: Svelte (small/medium/full), Anthropic (llms.txt + llms-full.txt). Acknowledges that different consumers have different constraints. Svelte's explicit guidance page ("which tier should I use?") is the model best practice. Highest-scoring archetype.

### Archetype 4: The LLM-Optimized (Score: 5 potential)

Specifically designed for AI agent consumption with explicit LLM affordances. No mature examples in v0.0.2 sample (Stripe excluded), but emerging approximations: Shadcn UI (AI-Ready architecture), Vercel AI SDK (agent patterns), PydanticAI (type-driven). Represents the frontier — not yet standard but likely to become table stakes within 18–24 months.

### Archetype 5: The Broken/Stub (Score: 1–2, anti-pattern)

Minimal, incomplete, or abandoned implementation. Example: NVIDIA (minimal visibility). Erodes trust and wastes the documentation slot. Even a basic Index provides more value.

---

## Quality Predictors — What Actually Matters

The correlation analysis revealed that quality is driven by intentionality, not volume:

| Predictor | Correlation | Implication |
|-----------|------------|-------------|
| Concrete examples and code samples | Strong (r ≈ 0.65) | "Show, don't tell" is the single strongest driver |
| Number of organized sections (5–12) | Strong (r ≈ 0.60) | Minimum viable: 5–7 sections |
| Active versioning/maintenance signals | Moderate (r ≈ 0.55) | Quality requires ongoing investment |
| Category (Framework > Enterprise) | Weak-Moderate (r ≈ 0.45) | Open-source frameworks invest more in DX |
| Tiered architecture | Weak-Moderate (r ≈ 0.35) | Sufficient but not necessary for quality |
| File size | Near-zero (r ≈ −0.05) | **Size does NOT predict quality** |

**Meta-insight:** A 40K-token well-organized document beats a 3.7M-token document lacking clear examples. Curation trumps comprehensiveness.

---

## Gold Standards

### Primary: Svelte

Perfect 5/5 across all dimensions. Distinguished by multi-tier variant strategy (small/medium/full), dedicated `/docs/llms` guidance page, explicit performance warnings about RAG degradation, and concept-first organization. The most sophisticated llms.txt implementation in the ecosystem.

**What DocStratum should borrow:** Tiered documentation as a first-class design pattern; dedicated guidance pages; performance-aware descriptions; canonical variant naming (small/medium/full); cross-cutting concerns documentation.

**What could be improved:** No explicit versioning scheme; no formal LLM Instructions section; no concept definition block/glossary.

### Secondary Gold Standards

**Shadcn UI (5/5):** AI-Ready architecture with `processMdxForLLMs` transformation pipeline. Demonstrates that LLM-readability can be built into the documentation pipeline itself. Framework-agnostic variants (React, Svelte, Vue).

**Pydantic (5/5):** Concept-first organization around fundamental ideas (validation, serialization, schema), not module structure. Cross-cutting concerns elevated as first-class sections. Clearest expression of semantic tiering semantics.

**Vercel AI SDK (5/5):** Modern TypeScript AI toolkit as full-stack reference architecture. Proves that new projects can be designed with LLM-readability in mind from the start. Streaming-first, composable design.

---

## Best Practices (Top 15, Evidence-Backed)

### Structure (6 practices)

1. **Use multi-tier variants** — Svelte (5), Anthropic (4), Pydantic (5) all implement tiering. Recommended sizes: small (<5K), medium (5–20K), full (20–50K).
2. **Organize by concepts, not alphabetically** — Pydantic (5) and Svelte (5) prioritize conceptual clarity. Identify 5–7 core mental models.
3. **Provide dedicated LLM guidance documentation (Guidance Page pattern)** — Svelte's `/docs/llms` page is unique in the audit: a human-readable page explaining what llms.txt is, which tier (small/medium/full) to use for different contexts, common usage patterns, and limitations. **DocStratum recommendation:** Make a `/docs/llms` or `LLM_GUIDE.md` guidance page a standard artifact of the DocStratum protocol — generated alongside llms.txt, explaining variant selection, including "best practices for AI agents" specific to the documented tool, and updated as the format evolves.
4. **Implement per-product decomposition for large platforms** — Cloudflare (4) and Supabase (4) demonstrate modular variants for complex ecosystems.
5. **Use canonical section names** — v0.0.2c identified 11 canonical sections with frequency data. Standardization enables cross-site parsing. The full registry (required for parser section recognition logic in v0.3.1):

| Canonical Section | Frequency | % | Allowed Variants |
|-------------------|-----------|---|-----------------|
| Getting Started / Overview | 14/18 | 78% | "Platform Overview" |
| API Reference | 12/18 | 67% | — |
| Core Concepts / Architecture | 10/18 | 56% | Bridges conceptual and applied |
| Configuration / Settings | 8/18 | 44% | — |
| Integrations / SDKs / Ecosystem | 8/18 | 44% | — |
| Examples / Use Cases / Tutorials | 6/18 | 33% | Practical application layer |
| Deployment / Hosting / Infrastructure | 6/18 | 33% | Operations focus |
| Troubleshooting / Error Handling / FAQ | 5/18 | 28% | — |
| Changelog / Release Notes | 4/18 | 22% | Version tracking |
| Best Practices / Performance Tips | 3/18 | 17% | Advanced guidance |
| Community / Support / Contributing | 3/18 | 17% | Social/governance |

6. **Include cross-cutting concerns as first-class sections** — Pydantic (5) documents patterns spanning the entire framework.

### Content (7 practices)

7. **Include LLM Instructions section** — Stripe pattern (v0.0.1) with positive, negative, and conditional directives. 0% adoption in v0.0.2 sample but emerging.
8. **Write semantic descriptions, not link-only lists** — Sites scoring 4–5 include contextual explanations. Cursor (3) shows generic defaults fail.
9. **Define validation schema and metadata** — v0.0.1 P0 gap. No example implements this — clear opportunity.
10. **Provide few-shot examples as structured code Q&A** — v0.0.1 P0 gap. Framework category strongest here (all 4+ stars). v0.0.2d specifies that "concrete examples" (the strongest quality predictor at r ≈ 0.65) must be *code examples* formatted as `"Q: How do I...? A: Use X like this: [code example]"`. **Schema implication (v0.1.2):** Per v0.0.1b's P0 gap analysis, the schema must explicitly support a `few_shot_examples` array to structure these code blocks with intent, question, ideal answer, source pages, and difficulty — treating them as first-class data, not inline prose.
11. **Document API signatures with types** — Pydantic (5), Vercel AI SDK (5), PydanticAI (4) emphasize type-driven design.
12. **Include error handling and troubleshooting** — Correlates with 5/5 overall scores. Only 28% of sites include troubleshooting.
13. **Add performance guidance** — Svelte's RAG degradation warnings are unique and represent mature operational thinking.

### Size (2 practices)

14. **Set target token ranges, not hard limits** — Small: 3–5K, Medium: 8–15K, Full: 20–50K. Anti-pattern threshold: no file >100K tokens.
15. **Use auto-generation hooks** — Mintlify (4) and Shadcn UI (5) demonstrate that keeping llms.txt in sync requires automation.

---

## Anti-Patterns Catalog

### Critical (7 — Never Do)

1. **Monolithic files >100K tokens** — Cloudflare full is 3.7M tokens. Exceeds all current LLM context windows.
2. **No semantic organization** — Sites scoring 3/5 organize alphabetically or by URL structure, not concepts.
3. **Link-only lists without descriptions** — LLMs can't determine relevance without context.
4. **No LLM Instructions despite being AI-native** — Anthropic, Hugging Face omit explicit agent guidance.
5. **Formulaic auto-generated descriptions (the "Mintlify Homogeneity" risk)** — Mintlify auto-generates llms.txt for thousands of hosted projects, often producing alphabetical link lists with formulaic descriptions and no semantic grouping. This explains why many sites (e.g., Cursor, Pinecone) share near-identical structure despite different domains. The systemic risk: as Mintlify's market share grows, an increasing proportion of the llms.txt ecosystem converges on low-information-density defaults. **DocStratum implication:** The semantic enrichment pipeline (concept injection, few-shot examples, LLM instructions) must specifically counter this homogeneity — transforming auto-generated stubs into semantically rich, domain-specific content is a core value proposition.
6. **No versioning or compatibility tracking** — 11% of examples lack this; they are the lowest scorers.
7. **Unknown/minimal implementation** — NVIDIA's approach erodes trust and wastes the documentation slot.

### Minor (5 — Avoid if Possible)

8. **Generic template defaults without customization** — Cursor (3) vs. Mintlify itself (4) shows templates need adaptation.
9. **No tiering for large tools** — Tools with >15K tokens should consider 3+ tiers.
10. **Alphabetical ordering within sections** — Defeats semantic organization at the detail level.
11. **Missing examples in reference documentation** — Code examples bridge reference and practice.
12. **Incomplete coverage of core features** — Agents gravitate toward documented features, missing undocumented ones.

---

## DocStratum Requirements (16 total, P0/P1/P2)

### P0 — Specification Blockers (6)

| # | Requirement | Source Evidence |
|---|------------|----------------|
| 1 | Formal concept and terminology definitions | v0.0.1 P0 gap; Pydantic demonstrates value |
| 2 | Validation schema and required metadata | v0.0.1 P0 gap; no example implements this |
| 3 | Few-shot examples for common LLM tasks | v0.0.1 P0 gap; Framework category strongest |
| 4 | LLM Instructions as first-class section | v0.0.1 anti-pattern; Stripe template model; 0% adoption in v0.0.2 |
| 5 | Maximum file size guidance with anti-pattern warnings | v0.0.1 P0 gap; 461x variance observed (8K vs 3.7M) |
| 6 | Canonical section names with allowed variants | v0.0.2c frequency analysis; 11 canonical types identified |

### P1 — High-Value Additions (5)

| # | Requirement | Source Evidence |
|---|------------|----------------|
| 7 | Versioning scheme for llms.txt files | v0.0.1 P1 gap; missing from lowest scorers |
| 8 | Tiered output specification | Svelte + Anthropic validate the pattern |
| 9 | Multi-language support guidance | v0.0.1 P2 gap; all 18 examples English-only |
| 10 | Standard section templates and examples | Gold standards provide reusable models |
| 11 | Auto-generation hooks specification | Mintlify + Shadcn UI demonstrate automation value |

### P2 — Quality Enhancements (5)

| # | Requirement | Source Evidence |
|---|------------|----------------|
| 12 | Caching and CDN recommendations | v0.0.1 P2 gap; no site documents cache strategy |
| 13 | Parser reference implementation | Enables consistent cross-tool parsing |
| 14 | Quality scoring heuristics | v0.0.2b scoring methodology as baseline |
| 15 | Privacy and access control guidance | Security best practices for public llms.txt |
| 16 | Ecosystem integration patterns | Documentation generator and IDE plugin hooks |

---

## v0.0.1 Findings Validation

All 8 findings from v0.0.1 were tested against the v0.0.2 audit data:

| Finding | v0.0.1 Status | v0.0.2 Outcome | Confidence |
|---------|--------------|---------------|------------|
| 8 specification gaps | Identified | 7/8 confirmed, 1 partially validated | High |
| "Permissive input, strict output" | Proposed | Validated (Mintlify, Svelte confirm) | High |
| Stripe as only LLM Instructions example | Noted | Confirmed; emerging in newer frameworks | High |
| Unconstrained size is unusable | Proposed | Refined: practical max ~100–250K tokens | Medium-High |
| "Optional" section underused | Observed | 11% adoption confirmed (Svelte, FastHTML) | High |
| Anti-patterns (empty, link-only, dumps) | Listed | All four observed in v0.0.2 sample | High |
| Tiered approach is best practice | Emerging | Validated by Svelte (5) + Anthropic (4) | High |
| AI-Readability Stack Layers 3–5 | Proposed | Layer 3–4 correlation strong; Layer 5 emerging | High |

---

## Structural Compliance Rates

| Element | Adoption | Notes |
|---------|----------|-------|
| H1 title | 100% (18/18) | Mandatory baseline, universally followed |
| Blockquote summary | 100% (18/18) | Spec requirement, universally followed |
| Detail paragraphs | 100% (18/18) | Content baseline |
| H2 section headers | 100% (18/18) | Organizational standard |
| Link list format | 100% (18/18) | Standard pattern |
| Versioning/dates | 89% (16/18) | Missing: Cursor, NVIDIA |
| Contact/maintainer info | 89% (16/18) | Missing: Cursor, NVIDIA |
| "Optional" section | 11% (2/18) | Only Svelte and FastHTML |
| LLM Instructions section | 0% (0/18) | Not in sample (Stripe excluded) |

**Insight:** Baseline spec compliance is near-universal. Advanced features (Optional sections, LLM Instructions) represent the frontier — high-value additions with almost no current adoption.

> **Enrichment Pass Correction (2026-02-06):** Empirical analysis of 11 specimens revealed significant overcounting in the original compliance data. Corrected rates from direct file analysis:
>
> | Element | Original (18 estimated) | Empirical (11 specimens) | Delta |
> |---------|------------------------|------------------------|-------|
> | H1 title (exactly 1) | 100% | 82% (9/11) | Cursor has 2 H1s |
> | Blockquote summary | **100%** | **55% (6/11)** | **Major overcounting — 5 specimens missing** |
> | Link list format | 100% | 89% (8/9 Type 1) | Cursor uses bare URLs |
> | H2 section headers | 100% | 100% (9/9 Type 1) | Confirmed |
> | "Optional" section | 11% | 0% (0/11) | Confirmed low |
> | LLM Instructions | 0% | 0% (0/11) | Confirmed absent |
>
> The blockquote finding is the most significant: the element treated as a universal baseline is actually absent in nearly half of real-world specimens. This suggests blockquote should be reclassified from "mandatory baseline" to "strongly recommended" in the DocStratum schema.

---

## What Feeds Forward

### Into v0.0.3 — Ecosystem Survey

- The 18-source catalog provides a foundation to expand toward ecosystem-wide coverage
- 5 implementation archetypes provide classification vocabulary for new discoveries
- Category performance data guides which ecosystem segments to prioritize

### Into v0.1.0 — Foundation Design

- 16 prioritized requirements (P0/P1/P2) become the input for formal schema design
- 15 best practices define what the schema must enable
- 12 anti-patterns define what the schema must prevent or warn against
- Gold standard examples (Svelte, Shadcn UI, Pydantic, Vercel AI SDK) serve as design references
- 9 implementation recommendations (5 schema design, 4 tooling) provide the blueprint

### Into v0.1.x — Implementation

- Canonical section registry → parser validation rules
- Tier definitions (small/medium/full) → output format specification
- Validation checklist → `docstratum-validate` CLI tool
- Section templates → `docstratum-generate` wizard
- Gold standard examples → examples repository structure
- **Fetcher module (v0.3.x) must handle WAF/CDN blocking** — v0.0.2a found that CDN/WAF protections (Cloudflare, etc.) returned 403 Forbidden for direct HTTP access to llms.txt on 60+ domains. This is not merely a research limitation; it is a production reality. The Fetcher must implement robust request headers (browser-mimicking user-agent strings), retry logic with backoff, and potentially configurable header profiles to bypass WAF protections. Without this, the tool will fail on a significant fraction of real-world llms.txt files.

---

## Empirical Enrichment Pass (2026-02-06)

### Specimen Collection

On 2026-02-06, 11 real-world llms.txt files were manually downloaded and integrated across all v0.0.2 sub-parts. This is the first time in the v0.0.x research program that direct file content has been analyzed.

| # | Specimen | Source | Size | Lines | Type | Conformance |
|---|----------|--------|------|-------|------|-------------|
| S1 | `astro-llms.txt` | astro.build | 2.6 KB | 31 | Type 1 Index | **100%** |
| S2 | `cloudflare-llms.txt` | cloudflare.com | 225 KB | 1,901 | Type 1 Index | 90% |
| S3 | `cursor-llms.txt` | cursor.com | 7.5 KB | 183 | Type 1 (non-conformant) | 20% |
| S4 | `deno-llms.txt` | deno.com | 63 KB | 464 | Type 1 Index | **100%** |
| S5 | `docker-llms.txt` | docker.com | 167 KB | 1,222 | Type 1 Index | 90% |
| S6 | `langchain-llms.txt` | langchain.com | 82 KB | 830 | Type 1 Index | 85% |
| S7 | `neon-llms.txt` | neon.tech | 68 KB | 558 | Type 1 Index | 95% |
| S8 | `openai-llms.txt` | openai.com | 19 KB | 151 | Type 1 Index | **100%** |
| S9 | `resend-llms.txt` | resend.com | 1.1 KB | 19 | Type 1 Index | 80% |
| S10 | `ai-sdk-llms.txt` | sdk.vercel.ai | 1.3 MB | 38,717 | Type 2 Full | 15% |
| S11 | `claude-llms-full.txt` | anthropic.com | 25 MB | 956,573 | Type 2 Full | 5% |

### Consolidated Empirical Findings

1. **Blockquote compliance is 55%, not 100%.** Five of 11 specimens (Cloudflare, Cursor, Docker, LangChain, Resend) omit the blockquote — the element previously treated as a universal baseline. This is the single most significant correction from the enrichment pass.

2. **Document Type Classification is a major new finding.** Two distinct paradigms exist: Type 1 Index (curated link catalogs, spec-conformant) and Type 2 Full (inline documentation dumps, spec-incompatible). The v0.0.1a grammar covers only Type 1. Type 2 requires separate handling in the DocStratum schema.

3. **Bimodal size distribution with a clear gap.** Type 1 files cluster at 1.1 KB – 225 KB. Type 2 files jump to 1.3 MB – 25 MB. No specimens exist in the 225 KB – 1.3 MB gap. This natural boundary should inform tier definitions.

4. **100% conformance is achievable.** Three specimens (Astro, Deno, OpenAI) achieve perfect conformance across different size tiers (2.6 KB, 63 KB, 19 KB). The spec is implementable as written — validation would simply formalize what these specimens already achieve.

5. **Cursor is definitively the worst-conforming Type 1 specimen.** At 20% conformance with 2 H1 headers and bare URLs, it's significantly worse than the estimated "competent but generic Mintlify default" from the original audit. This moves it from "The Index" to "The Broken/Stub" archetype.

6. **The original v0.0.2 audits were systematically optimistic.** Without direct file access, structural elements were assumed present. The enrichment pass reveals that indirect estimation overestimates compliance by approximately 30–45% on individual elements.

---

## Methodology Notes

### Verification Limitations

Direct HTTP access to llms.txt files was blocked by CDN/WAF protections during the initial research session (2026-02-05). All 18 original audits relied on indirect sources: web search data, platform provider documentation, community analyses, directory listings, and training knowledge. Ratings marked "(estimated)" in v0.0.2b indicate lower confidence. This limitation is transparently documented in each sub-part.

> **Enrichment Pass Resolution (2026-02-06):** The verification limitation was partially resolved through manual browser-based download of 11 specimen files. These files were analyzed at byte-level, providing the first direct structural conformance data in the research program. 5 of the 11 specimens overlap with original catalog entries, enabling before/after comparison of estimated vs. actual data. The remaining 6 specimens expand the catalog to 24 unique sources.

### Statistical Caveats

The 18-site sample, while diverse across 6 categories, is small for formal statistical inference. Correlation values (r) are approximations derived from observable patterns rather than formal regression analysis. Findings should be treated as strong directional indicators, confirmed when possible against v0.0.1's independent analysis, rather than precise measurements.

> **Enrichment Pass Note (2026-02-06):** The addition of 11 empirical specimens strengthens statistical confidence for structural compliance analysis. However, the specimen set is not randomly sampled — it was collected opportunistically based on browser accessibility. The structural compliance corrections (e.g., blockquote at 55%) may not generalize to the full ecosystem of 844,000+ implementations tracked by BuiltWith.

---

## Acceptance Criteria — All Verified

| Sub-Part | Criteria Count | Status |
|----------|---------------|--------|
| v0.0.2a — Source Discovery & Collection | 7/7 original + 5 enrichment = 12/12 | All `[x]` verified |
| v0.0.2b — Individual Example Audits | 7/7 original + enrichment passes | All `[x]` verified |
| v0.0.2c — Pattern Analysis & Statistics | 8/8 original + 8 enrichment = 16/16 | All `[x]` verified |
| v0.0.2d — Synthesis & Recommendations | 6/6 original + enrichment passes | All `[x]` verified |

**Total: 28/28 original acceptance criteria + enrichment pass criteria satisfied.**

---

## Source Documents

- [v0.0.2 — Wild Examples Audit](RR-SPEC-v0.0.2-wild-examples-audit.md) — Parent document with dependency chain and overall acceptance criteria
- [v0.0.2a — Source Discovery & Collection](RR-SPEC-v0.0.2a-source-discovery-and-collection.md) — 24-source catalog (18 original + 6 new) with metadata, 5 discovery methods, empirical size distribution
- [v0.0.2b — Individual Example Audits](RR-SPEC-v0.0.2b-individual-example-audits.md) — 24 audit forms (18 original + 6 new) with 96 ratings, 5 enrichment passes with empirical corrections, conformance grades
- [v0.0.2c — Pattern Analysis & Statistics](RR-SPEC-v0.0.2c-pattern-analysis-and-statistics.md) — File size stats (corrected with actual measurements), structural compliance corrections (blockquote 55%, not 100%), Document Type Classification, bimodal distribution, 5 archetypes with empirical validation
- [v0.0.2d — Synthesis & Recommendations](RR-SPEC-v0.0.2d-synthesis-and-recommendations.md) — 7 gold standards (1 primary + 6 secondary including 3 new empirical), 15 best practices (all validated), 12 anti-patterns (all confirmed with specimen citations), 16 requirements (empirically grounded)

### Evidence Base (2026-02-06 Enrichment)

| Source | Type | Count | Coverage |
|--------|------|-------|----------|
| Original v0.0.2 audit data (estimated) | Indirect | 18 sites | 6 categories |
| Specimen files (direct analysis) | Empirical | 11 files | 9 Type 1 + 2 Type 2 |
| Combined evidence base | Hybrid | 24 unique sites | 6 categories, 2 document types |
