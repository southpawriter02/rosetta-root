# v0.0.1b — Spec Gap Analysis & Implications

> **Sub-Part:** Systematically analyze every gap in the official llms.txt specification, document real-world consequences, and propose schema extensions for DocStratum.

---

## Sub-Part Overview

This sub-part conducts a systematic analysis of the eight identified gaps in the official llms.txt specification, examining real-world consequences with concrete evidence from wild examples and community patterns. For each gap, it documents workarounds currently in use and proposes concrete schema extensions with priority rankings (P0/P1/P2) to guide DocStratum's roadmap.

---

## Objective

The official llms.txt specification is intentionally minimal. v0.0.1 identified eight areas the spec does NOT define. This sub-part explores each gap in depth: what real-world problems it causes, what workarounds the community has adopted, and what schema extensions DocStratum should introduce to fill each gap.

### Success Looks Like

- All 8 spec gaps analyzed with real-world evidence
- Community workarounds documented for each gap
- Schema extension proposals drafted with field-level detail
- Priority ranking (P0/P1/P2) assigned to each extension
- Clear traceability from gap to proposed solution to implementation module

---

## Scope Boundaries

### In Scope

- Deep analysis of each gap identified in v0.0.1
- Documenting real-world consequences with evidence from wild examples
- Proposing concrete schema fields/extensions for each gap
- Prioritizing extensions for DocStratum's roadmap

### Out of Scope

- Implementing schema extensions (that's v0.1.2 — Schema Definition)
- Building validation for extensions (that's v0.2.4)
- Re-reading the spec itself (already done in v0.0.1)
- Analyzing individual implementations (that's v0.0.2b)

---

## Dependencies

```
v0.0.1 — Specification Deep Dive (COMPLETED)
    │
    ├── Identified 8 spec gaps:
    │   1. Maximum file size / token limit
    │   2. Required metadata fields
    │   3. Versioning scheme
    │   4. Validation schema
    │   5. Caching recommendations
    │   6. Multi-language support
    │   7. Concept/terminology definitions
    │   8. Example Q&A pairs
    │
    ├── Wild Examples Analysis (evidence base)
    └── Stripe LLM Instructions Pattern (extension precedent)
            │
            v
v0.0.1b — Spec Gap Analysis & Implications (THIS TASK)
            │
            v
v0.1.2 — Schema Definition (FUTURE — implements extensions)
v0.2.4 — Validation Pipeline (FUTURE — validates extensions)
```

---

## 1. Gap Analysis: Maximum File Size / Token Limit

### The Gap

The spec provides no guidance on how large an llms.txt file should be. No maximum file size, token count, or line count is specified.

### Real-World Consequences

| Consequence | Severity | Evidence |
|---|---|---|
| Context window overflow — LLMs cannot ingest files exceeding their context limit | Critical | HMSAAB Movies: 107M tokens (people), 66M tokens (titles) — completely unusable by any current LLM |
| Memory exhaustion in consumer tools | High | Tools like `llms_txt2ctx` load entire files into memory; multi-MB files cause OOM errors |
| No signal for when to use tiered approach | Medium | Nuxt and Vite independently invented `llms.txt` + `llms-full.txt` pattern because single files grew too large |
| Token budget uncertainty for agent designers | Medium | Agent builders have no way to predict how many tokens an llms.txt will consume from their budget |

### Community Workarounds

1. **Tiered files:** Nuxt, Vite use `llms.txt` (summary) + `llms-full.txt` (complete). No standard naming convention.
2. **"Optional" section:** The spec's one concession — consumers CAN skip the Optional section to reduce size. But no guidance on target size.
3. **Manual truncation:** Some implementers manually limit link counts per section.

### Proposed Schema Extension

```yaml
# Top-level metadata
meta:
  token_estimate: 4500          # Approximate token count (GPT-4 tokenizer)
  size_bytes: 12800             # File size in bytes
  has_full_version: true        # Whether a companion llms-full.txt exists
  full_version_url: "/llms-full.txt"
  recommended_context: "standard"  # "minimal" | "standard" | "full"
```

### Size Guidelines (Proposed)

| Category | Token Range | Use Case |
|---|---|---|
| Minimal | < 2,000 tokens | Small projects, single-product docs |
| Standard | 2,000 - 15,000 tokens | Most documentation sites |
| Large | 15,000 - 50,000 tokens | Major platforms (Stripe, Vercel scale) |
| Full | 50,000+ tokens | Requires tiered approach (llms.txt + llms-full.txt) |

### Empirical Validation (11 Specimens)

The size guidelines above are now validated against real-world data. The 11 collected specimens fall into a clear bimodal distribution that aligns with the proposed tiers:

| Specimen | Raw Size | Estimated Tokens | Proposed Tier | Type |
|---|---|---|---|---|
| Resend | 1.1 KB | ~300 | Minimal | Type 1 Index |
| Astro | 2.6 KB | ~700 | Minimal | Type 1 Index |
| Cursor | 7.5 KB | ~2,000 | Standard | Type 1 Index |
| OpenAI | 19 KB | ~5,000 | Standard | Type 1 Index |
| Deno | 63 KB | ~17,000 | Large | Type 1 Index |
| Neon | 68 KB | ~18,000 | Large | Type 1 Index |
| LangChain | 82 KB | ~22,000 | Large | Type 1 Index |
| Docker | 167 KB | ~45,000 | Large | Type 1 Index |
| Cloudflare | 225 KB | ~60,000 | Full (borderline) | Type 1 Index |
| AI SDK | 1.3 MB | ~38,000 lines | Full | Type 2 Full |
| Claude full | 25 MB | ~481,000 | Full (extreme) | Type 2 Full |

**Key observation:** No specimens fall in the 225 KB–1.3 MB gap. This suggests a natural boundary: files that outgrow Type 1 Index format typically jump directly to Type 2 Full inline format rather than creating ever-larger index files. The `llms-full.txt` tiered convention (co-developed by Mintlify and Anthropic) formalizes this split.

### Priority: **P0** (Critical for usability)

---

## 2. Gap Analysis: Required Metadata Fields

### The Gap

The only "required" element is the H1 title. There is no standard for version, last-updated date, maintainer, site URL, or any other metadata.

### Real-World Consequences

| Consequence | Severity | Evidence |
|---|---|---|
| No way to detect stale files | High | Files with no date could be years old; LLMs treat outdated info as current |
| No attribution or contact info | Medium | When an LLM encounters errors, there's no way to report or trace back to a maintainer |
| No machine-readable site identification | Medium | Without a canonical site URL, relative links in the file are unresolvable |
| Caching systems have no invalidation signal | Medium | HTTP caches have no `Last-Modified` equivalent in the file itself |

### Community Workarounds

1. **HTTP headers:** Some sites rely on `Last-Modified` or `ETag` HTTP headers — but these don't survive file copying, GitHub hosting, or CDN caching.
2. **Inline comments:** Some implementers add HTML comments (`<!-- Last updated: 2026-01-15 -->`) — not machine-parseable.
3. **Git metadata:** Relying on git commit dates — not available to HTTP consumers.

### Proposed Schema Extension

```yaml
meta:
  schema_version: "1.0"           # DocStratum schema version
  site_name: "Example API Docs"   # Human-readable site name
  site_url: "https://docs.example.com"  # Canonical base URL
  last_updated: "2026-02-05"      # ISO 8601 date
  maintainer:
    name: "Documentation Team"
    contact: "docs@example.com"   # Optional contact
  license: "CC-BY-4.0"           # Optional content license
```

### Backward Compatibility Strategy

The meta block would be encoded as YAML frontmatter at the top of the Markdown file, delimited by `---` fences. This is a well-established convention (Jekyll, Hugo, Obsidian) and does not break existing parsers that ignore frontmatter.

```markdown
---
schema_version: "1.0"
site_name: "Example API Docs"
last_updated: "2026-02-05"
---

# Example API Docs

> A comprehensive API for building payment integrations.

## Getting Started
...
```

### Empirical Validation (11 Specimens)

Zero of 11 specimens include any structured metadata — no YAML frontmatter, no inline comments with dates, no version fields, no maintainer information. The only identifying information available is the H1 title text, which varies in informativeness:

| Specimen | H1 Title | Machine-Identifiable Site? | Date Signal? |
|---|---|---|---|
| Astro | `# Astro` | Yes (matches site name) | No |
| Deno | `# Deno Docs` | Yes | No |
| OpenAI | `# OpenAI API Documentation` | Yes | No |
| Cloudflare | `# Cloudflare Developer Documentation` | Yes | No |
| Docker | `# Docker Docs` | Yes | No |
| LangChain | `# Docs by LangChain` | Yes | No |
| Cursor | `# Cursor` | Yes | No |
| Neon | `# Neon Documentation` | Yes | No |
| Resend | `# Resend` | Yes | No |
| AI SDK | (Multiple H1s) | Partially (Type 2) | No |
| Claude full | (Multiple H1s) | Partially (Type 2) | No |

**Key observation:** While all Type 1 specimens have informative H1 titles that allow human identification, none provide machine-readable metadata. This validates the P0 priority — every consuming tool must currently infer context from the URL it fetched, not from the file content itself.

### Priority: **P0** (Critical for trust and freshness)

---

## 3. Gap Analysis: Versioning Scheme

### The Gap

No mechanism for versioning the llms.txt file itself. No way to indicate which version of the documented product the file covers. No migration path between versions.

### Real-World Consequences

| Consequence | Severity | Evidence |
|---|---|---|
| API version confusion — LLMs recommend wrong API version | Critical | Stripe explicitly addresses this: "always default to the latest version of the API" — needed because the spec provides no versioning |
| Documentation drift — llms.txt diverges from actual docs | High | No mechanism to signal "this file describes v3.x" vs "this file describes v4.x" |
| No change detection for consumers | Medium | Tools that cache llms.txt content have no version field to compare |
| Multiple product versions undifferentiated | Medium | Nuxt serves v3 and v4 docs; without versioning, an LLM may mix guidance |

### Community Workarounds

1. **Separate files:** Nuxt uses separate URL paths for v3 vs v4 documentation.
2. **Inline instructions:** Stripe's LLM Instructions section manually specifies version preferences.
3. **Git tags:** Some projects version the file through git — not visible to HTTP consumers.

### Proposed Schema Extension

```yaml
meta:
  file_version: "2.1.0"          # Version of this llms.txt file itself
  product_version: "4.0"         # Version of the product being documented
  api_version: "2026-01-15"      # Date-based API version (Stripe pattern)
  previous_version: "/llms-v1.txt"  # Link to previous version
  changelog_url: "/changelog"    # Where to find what changed
```

### Version Compatibility Matrix (Proposed)

```
file_version  → Tracks changes to the llms.txt structure/content
product_version → The product release being documented
api_version → For API-centric products, the API version

Example:
  file_version: "3.0" (third rewrite of the llms.txt)
  product_version: "2.1" (docs cover product v2.1)
  api_version: "2026-01-15" (API version date)
```

### Priority: **P1** (Important for accuracy, not blocking for MVP)

---

## 4. Gap Analysis: Validation Schema

### The Gap

No formal schema (JSON Schema, DTD, ABNF) is provided for validating llms.txt files. There is no reference validator or conformance test suite.

### Real-World Consequences

| Consequence | Severity | Evidence |
|---|---|---|
| No way to programmatically verify compliance | High | Tools must write ad-hoc parsers; each interprets the spec differently |
| Inconsistent implementations | High | From v0.0.2 wild examples: wide variation in structure, some files barely recognizable as llms.txt |
| No CI/CD integration possible | Medium | Can't add "validate llms.txt" as a build step without a schema |
| No quality scoring | Low | Without defined rules, "good" vs "bad" is subjective |

### Community Workarounds

1. **Manual review:** Most implementers eyeball their files.
2. **Ad-hoc linters:** A few tools check basic structure, but no standard.
3. **Community directories:** Sites like llmstxt.site perform basic validation (URL reachable, non-empty) but not structural.

### Proposed Schema Extension

This gap is addressed by the work in v0.0.1a (formal grammar) and will be implemented in v0.2.4 (Validation Pipeline). The key deliverable is a Pydantic model:

```python
# Validation levels map to increasing strictness:
# Level 0: Parseable (no E-series errors from v0.0.1a grammar)
# Level 1: Structurally complete (H1, blockquote, H2 sections, entries)
# Level 2: Content quality (descriptions present, URLs valid)
# Level 3: Best practices (size appropriate, no anti-patterns)
# Level 4: DocStratum Extended (concepts, few-shot, LLM instructions)
```

### Empirical Validation (11 Specimens)

The conformance analysis of 11 specimens starkly illustrates the need for a validation schema:

| Conformance Band | Specimens | Count |
|---|---|---|
| **100% (PASS)** | Astro, Deno, OpenAI | 3 |
| **80–95% (MOSTLY PASS)** | Neon (95%), Cloudflare (90%), Docker (90%), LangChain (85%), Resend (80%) | 5 |
| **≤20% (FAIL)** | Cursor (20%), AI SDK (15%), Claude full (5%) | 3 |

**Key observations:** Only 3 of 11 specimens (27%) achieve full grammar conformance. The "MOSTLY PASS" band represents files that a human would consider valid but that a strict parser would flag — primarily due to missing blockquotes (a stylistic omission) and missing descriptions (an optional field). The "FAIL" band includes one genuine deviation (Cursor's bare URL format) and two fundamentally different document types (Type 2 Full files that don't attempt to conform to the index grammar). Without a validation schema, there is no way for authors to discover that their file deviates, and no way for consumers to distinguish between "intentional Type 2 document" and "broken Type 1 document."

### Priority: **P0** (Critical — this IS DocStratum's core value proposition)

---

## 5. Gap Analysis: Caching Recommendations

### The Gap

No guidance on how consumers should cache llms.txt files. No TTL, no `Cache-Control` recommendations, no change-detection mechanism.

### Real-World Consequences

| Consequence | Severity | Evidence |
|---|---|---|
| Over-fetching — agents re-download on every request | Medium | Without caching hints, well-behaved agents must fetch fresh copies each time |
| Stale data — agents use outdated cached copies | Medium | Without invalidation signals, cached files may persist indefinitely |
| Server load — popular llms.txt files could see high request volume | Low | Not yet a problem at current adoption levels, but will be as AI agents proliferate |

### Community Workarounds

1. **Standard HTTP caching:** Sites rely on `Cache-Control` headers — but these are server-side, not in the file.
2. **No workaround:** Most implementers don't think about caching at all.

### Proposed Schema Extension

```yaml
meta:
  cache:
    ttl_hours: 24               # Suggested re-fetch interval
    etag: "abc123"              # Content hash for change detection
    last_modified: "2026-02-05T14:30:00Z"  # ISO 8601 timestamp
```

Additionally, DocStratum should recommend HTTP headers:

```
Cache-Control: public, max-age=86400
ETag: "abc123"
Last-Modified: Wed, 05 Feb 2026 14:30:00 GMT
```

### Priority: **P2** (Nice to have — HTTP headers handle most cases)

---

## 6. Gap Analysis: Multi-Language Support

### The Gap

No mechanism to indicate the language of the llms.txt file or the documentation it points to. No support for multilingual documentation sites.

### Real-World Consequences

| Consequence | Severity | Evidence |
|---|---|---|
| Language mismatch — LLM receives content in unexpected language | Medium | International sites may serve llms.txt in their primary language; visiting agents may expect English |
| No language negotiation | Medium | Unlike HTTP `Accept-Language`, no way for consumers to request a specific language |
| Multilingual sites need multiple files | Low | Sites with docs in 5 languages need 5 separate llms.txt files with no linking mechanism |

### Community Workarounds

1. **English by default:** Nearly all observed llms.txt files are in English.
2. **URL-based separation:** Some sites use `/en/llms.txt`, `/ja/llms.txt` — no standard.
3. **No workaround:** Most sites simply don't address this.

### Proposed Schema Extension

```yaml
meta:
  language: "en"                        # ISO 639-1 language code
  available_languages:
    - code: "en"
      url: "/llms.txt"
    - code: "ja"
      url: "/ja/llms.txt"
    - code: "de"
      url: "/de/llms.txt"
```

### Priority: **P2** (Nice to have — English dominance makes this low urgency)

---

## 7. Gap Analysis: Concept / Terminology Definitions

### The Gap

No mechanism for defining key terms, concepts, or domain-specific vocabulary. The spec treats documentation as a flat list of links, with no semantic layer.

### Real-World Consequences

| Consequence | Severity | Evidence |
|---|---|---|
| Terminology confusion — LLMs conflate similar terms | Critical | Stripe must explicitly state "PaymentIntent is NOT the same as Charge" because the spec has no concept definition mechanism |
| No disambiguation — same term, different meanings | High | "Function" means different things in Vercel (serverless function) vs JavaScript (language feature) |
| Missing dependency relationships | High | Without "A depends on B" declarations, LLMs may recommend advanced concepts before prerequisites |
| No anti-concept support | Medium | Common misconceptions go unaddressed because there's no place to put "X is NOT Y" statements |

### Community Workarounds

1. **LLM Instructions section:** Stripe embeds concept guidance in prose instructions — works but not machine-parseable.
2. **Prose descriptions:** Some files include explanatory paragraphs — but LLMs may miss nuance in free text.
3. **No workaround:** Most implementations have no concept layer.

### Proposed Schema Extension

This is a **core DocStratum innovation** — Layer 2 of the architecture:

```yaml
concepts:
  - id: "auth-oauth2"
    name: "OAuth2 Authentication"
    definition: >
      OAuth2 is the primary authentication method for user-facing
      applications that require access to user data.
    related_pages:
      - "https://docs.example.com/api/auth"
      - "https://docs.example.com/getting-started"
    depends_on:
      - "concept-api-keys"
    anti_patterns:
      - "OAuth2 is NOT required for server-to-server integrations."
      - "OAuth2 tokens expire after 1 hour. Do NOT hardcode tokens."
    aliases:
      - "OAuth"
      - "OAuth 2.0"
    see_also:
      - "auth-api-keys"
      - "auth-jwt"
```

### Concept Relationship Types

| Relationship | Meaning | Example |
|---|---|---|
| `depends_on` | Must understand A before B | "OAuth2 depends on API Keys" |
| `see_also` | Related but independent concepts | "OAuth2 see also JWT" |
| `replaces` | A is the modern version of B | "PaymentIntent replaces Charge" |
| `conflicts_with` | A and B are mutually exclusive | "Direct charges conflicts with destination charges" |

### Empirical Validation (11 Specimens)

Zero of 11 specimens include any form of structured concept definitions. Even the two largest specimens (AI SDK at 1.3 MB, Claude full at 25 MB), which contain extensive inline documentation, present concepts within the flow of their documentation text rather than in a structured, cross-referenced format. The closest observed pattern is Cloudflare's 34-section product taxonomy — each H2 section name (`## Workers`, `## Pages`, `## R2`, `## D1`, etc.) implicitly defines a product concept, but without relationships, anti-patterns, or aliases. LangChain similarly uses its H2 section (`## Docs by LangChain`) as an implicit namespace but provides no concept structure beneath it.

**Key observation:** The complete absence of concept definitions across all 11 specimens — including files from sophisticated platforms like OpenAI, Cloudflare, and Anthropic — confirms that this gap is not merely theoretical. No existing implementation has attempted to fill it. This validates concepts as DocStratum's strongest differentiation opportunity.

### Priority: **P0** (Critical — this IS DocStratum's differentiator)

---

## 8. Gap Analysis: Example Q&A Pairs (Few-Shot)

### The Gap

No mechanism for including sample questions and ideal answers. LLMs have no pre-written examples to learn the correct answer format and content for a specific domain.

### Real-World Consequences

| Consequence | Severity | Evidence |
|---|---|---|
| Inconsistent answer format — LLMs make up their own format | High | Without few-shot examples, each LLM formats answers differently; some cite sources, some don't |
| Hallucinated details — LLMs fill gaps with plausible but wrong content | Critical | Without ideal answers, LLMs may confidently state incorrect API parameters or deprecated patterns |
| No quality benchmark — impossible to measure "correct" answer | Medium | A/B testing (v0.3.5) needs a ground truth; few-shot examples provide it |
| Missed best practices — LLMs don't know the "preferred" approach | High | Stripe addresses this with prose ("prefer PaymentIntent over Charges") but a structured Q&A would be more reliable |

### Community Workarounds

1. **Stripe's LLM Instructions:** Inline prose guidance — effective but not structured.
2. **README examples:** Some projects include example usage in body text — not tagged as few-shot training data.
3. **No workaround:** No observed implementation includes structured Q&A pairs.

### Proposed Schema Extension

This is Layer 3 of the DocStratum architecture:

```yaml
few_shot_examples:
  - id: "fse-001"
    intent: "User wants to authenticate a web application"
    question: "How do I add login to my React app?"
    ideal_answer: |
      To add OAuth2 login to a React app:
      1. Register your app at /api/auth#register
      2. Install the SDK: `npm install @example/auth-sdk`
      3. Initialize with your client ID
      4. Call `auth.login()` to trigger the OAuth2 flow
      See: https://docs.example.com/getting-started
    source_pages:
      - "https://docs.example.com/getting-started"
      - "https://docs.example.com/api/auth"
    tags:
      - "authentication"
      - "react"
      - "getting-started"
    difficulty: "beginner"

  - id: "fse-002"
    intent: "User is using deprecated API"
    question: "How do I create a charge with the Charges API?"
    ideal_answer: |
      The Charges API is deprecated. Please use the PaymentIntent API instead.
      Migration guide: https://docs.example.com/migrate-charges
    source_pages:
      - "https://docs.example.com/migrate-charges"
    tags:
      - "migration"
      - "deprecated"
    difficulty: "intermediate"
```

### Few-Shot Design Principles

1. **Cover common intents:** Start with the top 10 questions your docs receive
2. **Include negative examples:** Show how to handle deprecated/wrong approaches
3. **Cite sources:** Every ideal answer should reference specific pages
4. **Tag by difficulty:** Helps agents adjust explanation depth
5. **Keep answers concise:** 3-7 steps or 2-4 paragraphs maximum

### Empirical Validation (11 Specimens)

Zero of 11 specimens include any form of Q&A pairs, example interactions, or structured few-shot content. This is unsurprising given that the base spec does not define a mechanism for it, but it is notable that even Type 2 Full documents — which embed entire documentation sets inline — do not curate example interactions. AI SDK's 38,717 lines include extensive code examples within documentation pages, but these are not structured as Q&A pairs that an LLM could use as few-shot templates. Claude full's 956,573 lines similarly contain rich examples but without the intent/question/ideal_answer structure needed for reliable few-shot prompting.

**Key observation:** The absence of few-shot examples is the most universally missing capability across all specimens. While some gaps (like missing blockquotes) have community workarounds, few-shot examples have no workaround at all — the concept simply does not exist in the current ecosystem. This makes it the highest-opportunity Layer 3 innovation for DocStratum.

### Priority: **P0** (Critical — this IS DocStratum's differentiator)

---

## Summary: Priority Matrix

| Gap | Priority | Rationale | Implementation Target |
|---|---|---|---|
| Concept/Terminology Definitions | **P0** | Core DocStratum differentiator; Layer 2 | v0.1.2 |
| Example Q&A Pairs | **P0** | Core DocStratum differentiator; Layer 3 | v0.1.2 |
| Validation Schema | **P0** | Core value proposition of the project | v0.1.2, v0.2.4 |
| Maximum File Size | **P0** | Usability blocker for LLM consumers | v0.1.2 |
| Required Metadata | **P0** | Trust and freshness signals | v0.1.2 |
| Versioning Scheme | **P1** | Important for accuracy, not MVP-blocking | v0.1.2 |
| Caching Recommendations | **P2** | HTTP headers cover most cases | v0.1.2 (optional) |
| Multi-Language Support | **P2** | English dominance reduces urgency | v0.1.2 (optional) |

---

## Evidence Base Cross-References

| Gap | Evidence from Wild Examples | Evidence from Stripe Pattern |
|---|---|---|
| File Size | HMSAAB: 107M tokens; Rangita: 1.5M tokens — both unusable | Stripe: ~44K tokens — large but manageable |
| Metadata | No observed files include structured metadata | Stripe implicitly identifies itself but no machine-readable meta |
| Versioning | Nuxt separates v3/v4 by URL path | Stripe: "default to latest version" (manual instruction) |
| Validation | Wide variation in structure across 1000+ directory entries | Stripe is well-structured but no formal schema validates it |
| Caching | No observed files address caching | Not addressed |
| Multi-Language | All observed files are English-only | Not addressed |
| Concepts | No observed files define concepts structurally | Stripe defines concepts in prose ("PaymentIntent is the primary...") |
| Few-Shot | No observed files include Q&A pairs | Stripe's instructions function as implicit few-shot guidance |

### Evidence from 11 Specimen Files (Enrichment Pass)

| Gap | Empirical Finding | Specimen Evidence |
|---|---|---|
| File Size | Bimodal distribution: Type 1 (1.1 KB–225 KB) vs Type 2 (1.3 MB–25 MB). No specimens in the 225 KB–1.3 MB range. | Resend: 1.1 KB (minimal); Claude full: 25 MB (~481K tokens, unusable) |
| Metadata | 0/11 include any structured metadata. H1 titles provide only human-readable identification. | All 11 specimens lack frontmatter, dates, version, or maintainer info |
| Versioning | No specimens include version information of any kind. | No file_version, product_version, or api_version observed |
| Validation | 3/11 PASS (100%), 5/11 MOSTLY PASS (80–95%), 3/11 FAIL (≤20%). Without a schema, authors cannot discover deviations. | Astro, Deno, OpenAI: gold standard; Cursor: bare URLs; AI SDK, Claude: Type 2 |
| Caching | No specimens address caching. Consistent with community workaround "none." | No in-file cache hints, ETags, or TTL suggestions in any specimen |
| Multi-Language | All 11 specimens are English-only. | Consistent with existing evidence; English dominance continues |
| Concepts | 0/11 include structured concept definitions. Cloudflare's 34 H2 sections implicitly define product concepts but without relationships or anti-patterns. | Closest pattern: Cloudflare product taxonomy via H2 sections |
| Few-Shot | 0/11 include Q&A pairs or structured examples. Type 2 files contain code examples within docs but not as structured few-shot templates. | Universally absent — no workaround exists in the ecosystem |

---

## Deliverables

- [x] Deep analysis of all 8 spec gaps
- [x] Real-world consequences documented with evidence
- [x] Community workarounds cataloged for each gap
- [x] Schema extension proposals with field-level detail
- [x] Priority matrix (P0/P1/P2) for all extensions
- [x] Cross-reference to evidence base
- [x] Empirical validation sections added to 5 gaps (Size, Metadata, Validation, Concepts, Few-Shot) with data from 11 specimens
- [x] Extended evidence base cross-reference table with specimen-specific findings

---

## Acceptance Criteria

- [x] All 8 gaps from v0.0.1 analyzed with real-world consequences
- [x] Each gap has at least one documented community workaround (or "none observed")
- [x] Each gap has a concrete schema extension proposal with example YAML/code
- [x] Priority assigned to every extension
- [x] Proposals are backward-compatible with the base llms.txt spec
- [x] Clear traceability from each gap to the implementation module that addresses it
- [x] Gap severity validated against 11 real-world specimens with empirical data
- [x] All P0 gaps confirmed as genuinely unaddressed by any observed implementation

---

## Next Step

Once this sub-part is complete, proceed to:

**v0.0.1c — Processing & Expansion Methods**

Understanding what the spec lacks (this document) sets the stage for understanding how existing tools work around those limitations through various processing strategies.
