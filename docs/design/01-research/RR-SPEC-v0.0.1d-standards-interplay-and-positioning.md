# v0.0.1d — Standards Interplay & Positioning

> **Sub-Part:** Analyze how llms.txt relates to, interacts with, and is positioned alongside existing web standards (robots.txt, sitemap.xml, humans.txt, schema.org), and define DocStratum's strategic positioning.

---

## Sub-Part Overview

This sub-part analyzes how llms.txt relates to and interacts with six major web standards (robots.txt, sitemap.xml, humans.txt, schema.org, .well-known, and ai.txt), establishing both compliance requirements and strategic opportunities. It models the emerging "AI-Readability Stack" as a layered framework spanning from access control through agent instructions, and positions DocStratum as a semantic translation layer that extends llms.txt from a simple page index into a comprehensive content guidance system for AI agents.

---

## Objective

The official spec briefly compares llms.txt to robots.txt but stops short of analyzing how these standards interact in practice. Understanding this interplay is critical for positioning DocStratum in the broader ecosystem, ensuring compliance with existing standards, and identifying strategic opportunities.

### Success Looks Like

- Comprehensive comparison of llms.txt with 5+ related standards
- Practical interaction analysis (compliance, conflicts, synergies)
- Strategic positioning framework for DocStratum
- Recommendations for standard-aware tooling
- Clear narrative: where llms.txt fits in the "stack" of web standards

---

## Scope Boundaries

### In Scope

- Comparing llms.txt with robots.txt, sitemap.xml, humans.txt, schema.org, and .well-known conventions
- Documenting practical interactions between standards
- Defining DocStratum's strategic position
- Identifying compliance requirements
- Mapping the "AI-readability stack"

### Out of Scope

- Implementing cross-standard tooling (that's v0.3.x)
- Building auto-generators from sitemaps (that's v0.2.x)
- Deep technical analysis of each standard's spec (focus is on interaction with llms.txt)
- Advocacy or marketing strategy

---

## Dependencies

```
v0.0.1 — Specification Deep Dive (COMPLETED)
    │
    ├── Identified relationship to robots.txt, sitemap.xml, humans.txt
    ├── Key distinction: robots.txt = access control; llms.txt = content curation
    │
v0.0.1b — Spec Gap Analysis (COMPLETED)
    │
    ├── Metadata gaps overlap with schema.org
    ├── Caching gaps overlap with HTTP standards
    │
v0.0.1c — Processing Methods (COMPLETED)
    │
    └── Processing tools need to respect robots.txt
            │
            v
v0.0.1d — Standards Interplay & Positioning (THIS TASK)
            │
            v
v0.0.3 — Ecosystem & Tooling Survey (NEXT — broader landscape)
v0.1.2 — Schema Definition (FUTURE — informed by standards compatibility)
```

---

## 1. Standards Comparison Matrix

### Overview Table

| Standard | Purpose | Format | Location | Audience | Governance |
|---|---|---|---|---|---|
| **robots.txt** | Control crawler access | Plain text (custom syntax) | `/robots.txt` | Web crawlers | De facto standard (RFC 9309) |
| **sitemap.xml** | List crawlable URLs | XML | `/sitemap.xml` or referenced in robots.txt | Search engines | sitemaps.org protocol |
| **humans.txt** | Credit contributors | Plain text (free-form) | `/humans.txt` | Humans (curious visitors) | humanstxt.org |
| **schema.org** | Structured data markup | JSON-LD / Microdata / RDFa | Embedded in HTML pages | Search engines, knowledge graphs | W3C Community Group |
| **llms.txt** | LLM-friendly content index | Markdown | `/llms.txt` | AI agents / LLMs | Informal proposal (Jeremy Howard) |
| **.well-known** | Service discovery | Various | `/.well-known/` directory | Automated clients | IANA registry (RFC 8615) |
| **ai.txt** | AI training permissions | Plain text | `/ai.txt` | AI training crawlers | Spawning.ai proposal |

### Functional Comparison

| Capability | robots.txt | sitemap.xml | humans.txt | schema.org | llms.txt |
|---|---|---|---|---|---|
| Lists URLs | Implied (via Disallow) | Yes (comprehensive) | No | No (embedded) | Yes (curated) |
| Content descriptions | No | Optional `<lastmod>` | Free text | Rich structured data | Yes (link notes) |
| Access control | Yes (primary purpose) | No | No | No | No |
| Semantic structure | No | Minimal (priority, frequency) | No | Yes (full ontology) | Minimal (H2 sections) |
| Machine-parseable | Yes (formal syntax) | Yes (XML schema) | No | Yes (JSON-LD) | Partially (Markdown) |
| Human-readable | Somewhat | No (XML) | Yes (primary purpose) | No (JSON-LD) | Yes (Markdown) |
| Freshness signals | No | `<lastmod>` | No | `dateModified` | Not in spec (DocStratum adds) |
| LLM-optimized | No | No | No | No | Yes (primary purpose) |

---

## 2. Practical Interactions

### 2.1 robots.txt and llms.txt

**Interaction Type:** Compliance requirement

**The Rule:** If a URL is disallowed in robots.txt, it MUST NOT appear in llms.txt.

**Why This Matters:**

- robots.txt is a legally recognized mechanism for expressing access preferences
- Including a disallowed URL in llms.txt creates a contradiction: "crawlers can't access this, but LLMs should read it"
- Some AI companies respect robots.txt for training data; violating it erodes trust

**Practical Scenarios:**

| Scenario | Correct Behavior |
|---|---|
| `/admin/` is Disallow'd in robots.txt | Do NOT include any `/admin/*` URLs in llms.txt |
| robots.txt blocks all bots (`User-agent: *; Disallow: /`) | llms.txt is still valid — it's for "on demand" inference, not crawling. But linked pages may be inaccessible |
| robots.txt has specific `User-agent: GPTBot; Disallow: /` | llms.txt may still include those URLs (different access paradigm), but consumers should respect robots.txt when fetching |

**Recommendation for DocStratum:**

```yaml
# Validation rule: Cross-reference robots.txt
validation:
  check_robots_compliance: true
  robots_url: "/robots.txt"   # Default, can be overridden
  on_violation: "warning"     # "error" | "warning" | "ignore"
```

---

### 2.2 sitemap.xml and llms.txt

**Interaction Type:** Complementary data sources

**Key Difference:** sitemap.xml lists ALL crawlable pages. llms.txt lists CURATED pages for LLM consumption. Sitemaps are comprehensive; llms.txt is opinionated.

**Practical Interactions:**

| Interaction | Detail |
|---|---|
| Sitemap as llms.txt seed | Auto-generators can use sitemap.xml as a starting point, then filter/curate |
| llms.txt as sitemap subset | Every URL in llms.txt SHOULD appear in sitemap.xml (but not vice versa) |
| Priority overlap | sitemap.xml has `<priority>` (0.0-1.0); llms.txt has no equivalent (DocStratum could add) |
| Freshness overlap | sitemap.xml has `<lastmod>`; llms.txt has no equivalent (DocStratum adds `last_updated`) |

**Anti-Pattern: Sitemap Dump**

One of the most common anti-patterns observed in v0.0.2 analysis: sites that auto-generate llms.txt by dumping their entire sitemap into link-list format. This defeats the purpose — llms.txt should be curated, not comprehensive.

**Recommendation for DocStratum:**

```python
# Tool: sitemap_to_llms_seed.py
# Takes a sitemap.xml and generates a DRAFT llms.txt
# that a human then curates. NOT for direct publishing.

def generate_seed(sitemap_url: str) -> str:
    """
    Generate a draft llms.txt from a sitemap.
    Applies heuristics to suggest which pages to include.
    MUST be human-reviewed before publishing.
    """
    # Heuristics:
    # - Include pages with high priority (>0.7)
    # - Include recently modified pages
    # - Exclude /admin/, /login/, /search/ paths
    # - Group by URL path patterns
    pass
```

---

### 2.3 humans.txt and llms.txt

**Interaction Type:** Philosophical parallels

**Similarity:** Both are "about this site" files placed at the root. humans.txt says "humans made this"; llms.txt says "LLMs, read this."

**Practical Interactions:**

| Interaction | Detail |
|---|---|
| Attribution | llms.txt could reference humans.txt for contributor credits |
| Complementary signals | humans.txt proves human authorship; llms.txt enables AI consumption |
| Style inspiration | humans.txt is free-form and human-friendly; llms.txt follows suit with Markdown |

**Recommendation:** DocStratum could optionally include a `humans_url` field in metadata to cross-reference.

---

### 2.4 schema.org and llms.txt

**Interaction Type:** Overlapping concerns, different mechanisms

**Key Difference:** schema.org embeds structured data WITHIN HTML pages. llms.txt is a SEPARATE file that indexes content ACROSS pages.

**Where They Overlap:**

| Concern | schema.org Approach | llms.txt Approach |
|---|---|---|
| Content type labeling | `@type: "HowTo"`, `"APIReference"` | H2 section names ("Docs", "Examples") |
| Freshness | `dateModified` property | Not in spec (DocStratum adds `last_updated`) |
| Descriptions | `description` property | Link notes (`[Title](URL): description`) |
| Relationships | `isPartOf`, `relatedLink` | Not in spec (DocStratum adds `depends_on`, `see_also`) |

**Strategic Insight:**

schema.org is powerful but requires embedding markup in every HTML page. llms.txt provides a centralized, lightweight alternative. The two are complementary: schema.org for search engines, llms.txt for AI agents.

**Recommendation for DocStratum:**

```yaml
# Optional: Map DocStratum content types to schema.org types
content_type_mappings:
  tutorial: "HowTo"
  reference: "APIReference"
  changelog: "ChangeLog"
  concept: "DefinedTerm"
  faq: "FAQPage"
```

---

### 2.5 .well-known and llms.txt

**Interaction Type:** Alternative location convention

**Background:** RFC 8615 defines `/.well-known/` as the standard directory for site-wide metadata files. Many services use it: `/.well-known/security.txt`, `/.well-known/openid-configuration`, etc.

**The Question:** Should llms.txt be at `/.well-known/llms.txt` instead of `/llms.txt`?

**Arguments For `/llms.txt` (current spec):**

- Simpler (no nested directory)
- Follows robots.txt and humans.txt precedent
- Easier to discover (shorter URL)
- Already established by the community

**Arguments For `/.well-known/llms.txt`:**

- IANA-registered convention for metadata files
- Avoids polluting the root directory
- Better organizational hygiene for sites with many metadata files

**Recommendation:** DocStratum should support BOTH locations, with `/llms.txt` as the primary and `/.well-known/llms.txt` as a fallback.

```python
LLMS_TXT_LOCATIONS = [
    "/llms.txt",                    # Primary (spec standard)
    "/.well-known/llms.txt",        # Fallback (well-known convention)
    "/docs/llms.txt",               # Alternative (documentation subpath)
]
```

---

### 2.6 ai.txt and llms.txt

**Interaction Type:** Adjacent but distinct purpose

**Background:** `ai.txt` (proposed by Spawning.ai) is focused on AI TRAINING permissions — who can use your content for model training. llms.txt is focused on AI INFERENCE — helping models use your content at query time.

**Critical Distinction:**

| Aspect | ai.txt | llms.txt |
|---|---|---|
| Purpose | Permission for training data | Guidance for inference usage |
| Tone | Restrictive ("don't use this for training") | Invitational ("use this for context") |
| Audience | AI training pipelines | AI agents at query time |
| Content | Permission declarations | Content index + guidance |

**Recommendation:** DocStratum should acknowledge ai.txt's existence and recommend including training permissions in llms.txt metadata:

```yaml
meta:
  ai_training_policy: "no"          # "yes" | "no" | "conditional"
  ai_training_details: "Content may be used for inference but not for model training."
  ai_txt_url: "/ai.txt"            # Cross-reference
```

---

### 2.7 MCP (Model Context Protocol) and llms.txt

**Interaction Type:** Transport and delivery mechanism

**Background:** The Model Context Protocol (MCP) is a JSON-RPC-based protocol originally developed by Anthropic and now stewarded by the Linux Foundation. MCP provides a standardized way for LLMs and AI agents to connect to external data sources and tools.

**Critical Finding: MCP is the Validated Consumption Pathway**

Empirical analysis of 11 real-world llms.txt specimens reveals that llms.txt is NOT primarily consumed by search engines or general chat LLMs. Instead, the validated consumption pathway is:

```
AI coding assistants (Cursor, Claude Desktop, Windsurf)
    ↓
MCP server (fetch & inject mechanism)
    ↓
llms.txt file (fetched from /llms.txt or /.well-known/llms.txt)
    ↓
Context window (injected by MCP into LLM's system context)
```

**Practical Evidence:**

- The Cursor specimen explicitly references MCP-based documentation tools as its consumption pattern
- Cursor, Claude Desktop, and Windsurf all expose native MCP server interfaces for fetching external documents
- No specimens in the audit showed evidence of llms.txt being consumed by general-purpose chat interfaces (ChatGPT, Gemini, Claude.ai)
- This suggests llms.txt is optimized for TOOL-USING AGENTS rather than conversational LLMs

**Distinction from Other Standards:**

- robots.txt and sitemap.xml are consumed by HTTP crawlers
- schema.org is parsed from HTML during indexing
- ai.txt is consumed by batch training pipelines
- **llms.txt via MCP is consumed by interactive agents at query time**

**Interaction Details:**

| Aspect | Detail |
|---|---|
| Protocol | JSON-RPC 2.0 (request-response over stdio or HTTP) |
| Discovery | MCP servers are configured in AI tool settings, not auto-discovered |
| Freshness | MCP fetches happen at each session start (not cached long-term) |
| Transport | HTTP GET with optional authentication |
| Format expectations | Raw Markdown; MCP handles injection and formatting |

**Recommendation for DocStratum:**

1. **Native MCP Server Interface:** The Context Builder (v0.3.2) should expose an MCP server endpoint that:
   - Serves llms.txt with proper MIME type (`text/markdown`)
   - Implements the MCP resource protocol for fetching documents
   - Supports optional authentication (API key, Bearer token)
   - Caches results with appropriate TTL (default: 1 hour)

2. **Configuration Pattern:** Recommend users add DocStratum as an MCP resource in their AI tool:
   ```json
   {
     "mcpServers": {
       "docstratum": {
         "command": "python",
         "args": ["-m", "docstratum.mcp_server"],
         "env": {
           "DOCSTRATUM_ROOT_URL": "https://docs.example.com",
           "DOCSTRATUM_ROOT_API_KEY": "${env:DOCSTRATUM_ROOT_API_KEY}"
         }
       }
     }
   }
   ```

3. **Documentation Pattern:** DocStratum should document the MCP consumption model as the primary use case, with traditional HTTP fetch as a fallback.

---

## 3. The AI-Readability Stack

### Conceptual Model

Just as the web has a "stack" of standards for human readability (HTML + CSS + JS), AI-readability is developing its own stack:

```
┌─────────────────────────────────────────────────────────┐
│  LAYER 5: AGENT INSTRUCTIONS                            │
│  (How to behave with this content)                      │
│  → LLM Instructions section (Stripe pattern)            │
│  → Few-shot examples (DocStratum Layer 3)             │
├─────────────────────────────────────────────────────────┤
│  LAYER 4: SEMANTIC UNDERSTANDING                        │
│  (What the content means)                               │
│  → Concept definitions (DocStratum Layer 2)           │
│  → schema.org structured data                           │
├─────────────────────────────────────────────────────────┤
│  LAYER 3: CONTENT CURATION                              │
│  (What content matters)                                 │
│  → llms.txt (curated index)                             │
│  → llms-full.txt (complete index)                       │
├─────────────────────────────────────────────────────────┤
│  LAYER 2: CONTENT DISCOVERY                             │
│  (What content exists)                                  │
│  → sitemap.xml (all pages)                              │
│  → .md URLs (markdown versions)                         │
├─────────────────────────────────────────────────────────┤
│  LAYER 1: ACCESS CONTROL                                │
│  (What content is accessible)                           │
│  → robots.txt (crawler permissions)                     │
│  → ai.txt (training permissions)                        │
│  → .well-known/security.txt                             │
├─────────────────────────────────────────────────────────┤
│  LAYER 0: TRANSPORT & DELIVERY                          │
│  (How content reaches the LLM)                          │
│  → MCP (Model Context Protocol) — primary pathway       │
│  → Direct HTTP fetch — fallback pathway                 │
│  → Caching and TTL management                           │
└─────────────────────────────────────────────────────────┘
```

**Amendment Note on Layer 0:** The empirical findings reveal that Layer 0 (transport) is critical and often overlooked. MCP via AI coding assistants is the validated primary consumption pathway for llms.txt, NOT search engines or batch LLM indexing. This implies that optimization for MCP injection should be a primary design goal, and fallback HTTP fetch mechanisms should be secondary.

### DocStratum's Position

DocStratum spans Layers 3-5 of this stack. The official llms.txt spec covers Layer 3 only. DocStratum's innovation is extending into Layers 4 and 5 with concepts, few-shot examples, and LLM instructions.

---

## 4. Compliance Requirements for DocStratum

Based on the standards analysis, DocStratum tooling must:

### Must (P0)

1. **Respect robots.txt:** The validator should warn when llms.txt includes URLs disallowed by robots.txt
2. **Support standard location:** `/llms.txt` as primary location
3. **Output valid Markdown:** llms.txt files must be valid Markdown that renders correctly
4. **Preserve backward compatibility:** Extended fields must not break basic llms.txt parsers

### Should (P1)

1. **Cross-reference sitemap.xml:** Warn about URLs in llms.txt not found in sitemap
2. **Support .well-known fallback:** Check both `/llms.txt` and `/.well-known/llms.txt`
3. **Map content types to schema.org:** Enable interoperability with search engines
4. **Include freshness metadata:** Align with sitemap.xml's `<lastmod>` convention

### Could (P2)

1. **Generate from sitemap.xml:** Seed tool that creates draft llms.txt from sitemap
2. **Include ai.txt cross-reference:** Document training permissions alongside inference guidance
3. **Support humans.txt attribution:** Optional link to contributor credits

---

### 4.1 Empirical Compliance Observations

Analysis of 11 real-world specimen llms.txt files reveals significant gaps between recommended practices (§2) and actual adoption:

| Recommended Practice | Implementation Rate | Significance |
|---|---|---|
| robots.txt cross-reference (§2.1) | 0/11 (0%) | No specimens include validation against robots.txt or reference to it |
| ai.txt references (§2.6) | 0/11 (0%) | No specimens mention ai.txt or training permissions |
| Structured metadata (YAML frontmatter) | 0/11 (0%) | All specimens use plain Markdown with no metadata header |
| Grammar conformance (clean H2/H3 structure) | 3/11 (27%) | Only Astro, Deno, and OpenAI achieve perfect grammar and structure |
| Content curation (selective vs. sitemap dump) | 7/11 (64%) | Cloudflare and others demonstrate intentional curation |
| Link descriptions (with notes) | 9/11 (82%) | Most specimens include brief descriptions for linked resources |
| H2 section headers (organized grouping) | 10/11 (91%) | Near-universal adoption of sectional organization |

**Key Finding: The Compliance Gap**

The gap between "should do" (recommended in §2-4) and "actually do" (observed in real world) is wide. This creates both a challenge and an opportunity:

- **Challenge:** Sites are not adopting cross-standard validations, creating potential for subtle conflicts (e.g., llms.txt linking to robots.txt-disallowed URLs)
- **Opportunity:** DocStratum can differentiate by providing these compliance guarantees out-of-the-box

**"Gold Standard" Exemplars:**

Three specimens achieve 100% grammar conformance and demonstrate proper content curation (Layer 3 of the AI-Readability Stack):

1. **Astro** — Clean section structure, concise link descriptions, no syntax errors
2. **Deno** — Well-organized by product area, proper Markdown formatting, strategic URL selection
3. **OpenAI** — Comprehensive but selective index, clear information hierarchy

None of these three extend into Layer 4 (concepts) or Layer 5 (agent instructions), suggesting these layers are still emerging.

**Cloudflare's Product-Nested Hierarchy Pattern:**

Cloudflare demonstrates an emerging de facto pattern: 34 H2 sections map directly to their product portfolio (Workers, Pages, R2, D1, etc.). This creates a product taxonomy within the flat llms.txt structure — an implicit interaction with schema.org's content type system. This pattern could be formalized in DocStratum as a content organization convention.

---

## 5. Strategic Positioning Summary

### DocStratum's Value Proposition (Relative to Standards)

| Standard | What it does | What DocStratum adds |
|---|---|---|
| robots.txt | "Can you access this?" | "Here's HOW to use it" |
| sitemap.xml | "Here's everything that exists" | "Here's what actually matters" |
| humans.txt | "Humans made this" | "And here's how AI should understand it" |
| schema.org | "This page is a HowTo" | "And here's how this HowTo relates to 12 other concepts" |
| llms.txt (base) | "Read these pages" | "Understand these concepts, follow these patterns, avoid these mistakes" |

### One-Line Positioning

> **DocStratum transforms llms.txt from a page index into a semantic translation layer — bridging the gap between documentation designed for humans and context optimized for AI agents.**

### 5.1 The llms-full.txt Convention

**Emerging Standard:** Analysis reveals that `llms-full.txt` is emerging as a tiered convention alongside the original `llms.txt`, co-developed by Mintlify and Anthropic.

**Parallel Structure:**

The relationship between `llms.txt` and `llms-full.txt` mirrors the sitemap indexing pattern:

| File | Purpose | Audience | Pattern Parallel |
|---|---|---|---|
| `llms.txt` | Curated index with link descriptions | AI agents with constrained context | `sitemap.xml` (primary index) |
| `llms-full.txt` | Complete page index with full content | AI agents with unlimited context | `sitemap_index.xml` (aggregation) |

**Practical Distinction:**

- **llms.txt:** ~2-5 KB, 20-50 carefully selected pages, used in MCP context injection where context window is precious
- **llms-full.txt:** ~50-200+ KB, comprehensive page listing, used in scenarios where the AI agent has full context budget

**Recommendation for DocStratum:**

1. **Dual-File Support:** DocStratum should enable generation of both files from a single source configuration
2. **Validation:** Ensure all URLs in llms.txt also appear in llms-full.txt (subset relationship)
3. **Documentation:** Clarify the use cases: "Use llms.txt for limited context; use llms-full.txt for comprehensive indexing"
4. **Tool Configuration:**
   ```yaml
   outputs:
     curated:
       filename: "llms.txt"
       max_size: "5KB"
       description: "For MCP injection with limited context"
     comprehensive:
       filename: "llms-full.txt"
       max_size: "unlimited"
       description: "For full knowledge base indexing"
   ```

---

## Deliverables

- [x] Comparison matrix of 7 related standards
- [x] Practical interaction analysis for each standard pair
- [x] AI-Readability Stack conceptual model (Layers 0-5)
- [x] MCP (Model Context Protocol) interaction analysis
- [x] Compliance requirements (P0/P1/P2)
- [x] Strategic positioning summary
- [x] Empirical compliance observations from 11 specimen llms.txt files
- [x] Recommendations for cross-standard tooling
- [x] llms.txt + llms-full.txt tiered convention analysis

---

## Acceptance Criteria

- [x] At least 5 related standards compared with llms.txt
- [x] Each comparison includes practical interaction scenarios
- [x] MCP identified as primary transport mechanism for llms.txt consumption
- [x] Compliance requirements are prioritized and actionable
- [x] Strategic positioning is clear and differentiated
- [x] AI-Readability Stack model is coherent and illustrative (includes Layer 0)
- [x] Empirical compliance gap documented (recommended practices vs. real-world adoption)
- [x] llms-full.txt emerging convention recognized and positioned
- [x] Cross-references to relevant DocStratum modules (v0.1.2, v0.2.4, v0.3.x)

---

## Next Step

With all four v0.0.1 sub-parts complete:

- **v0.0.1a:** Formal grammar and parsing rules defined
- **v0.0.1b:** Spec gaps analyzed with extension proposals
- **v0.0.1c:** Processing methods compared with hybrid design
- **v0.0.1d:** Standards interplay mapped with positioning (this document)

**Mark v0.0.1 as COMPLETE** and proceed to **v0.0.2 — Wild Examples Audit**.
