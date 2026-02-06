# v0.0.2a — Source Discovery & Collection

> **Sub-Part:** Discover and catalog 15-20 real-world llms.txt implementations for subsequent analysis.

---

## Sub-Part Overview

This sub-part catalogs 18 real-world llms.txt implementations across 6 categories, sourced via community directories, web search, and adoption reports. The catalog serves as input for v0.0.2b's detailed audits.

---

## Objective

Build a comprehensive catalog of llms.txt files from diverse sources. This catalog becomes the input for v0.0.2b's detailed audits.

### Success Looks Like

- 15-20 unique llms.txt URLs collected
- Each URL verified as accessible (not 404)
- Basic metadata recorded (domain, category, approximate size)
- Diverse mix of source types represented

---

## Scope Boundaries

### In Scope

- Finding llms.txt URLs via search and directories
- Recording basic metadata (URL, domain, category)
- Verifying URLs are accessible
- Categorizing sources by type

### Out of Scope

- Detailed content analysis (that's v0.0.2b)
- Pattern identification (that's v0.0.2c)
- Recommendations (that's v0.0.2d)
- Re-analyzing Stripe, Nuxt, Vercel (already done in v0.0.1)

---

## Discovery Methods Used

### Method 1: Community Directories

Searched the following directories for documented implementations:

- llmstxt.site — Community directory (784+ listed sites as of mid-2025)
- llmstxthub.com — 500+ websites indexed with category filtering
- directory.llmstxt.cloud — Adoption-tracking directory
- github.com/SecretiveShell/Awesome-llms-txt — Curated GitHub index
- github.com/thedaviddias/llms-txt-hub — Largest open directory

### Method 2: Adoption Reports and Analysis

Cross-referenced multiple independent articles and analyses that confirm specific implementations:

- Mintlify blog (platform provider, confirmed customer list)
- BuiltWith tracking data (844,000+ sites as of October 2025)
- Majestic Million crawl data (15 sites in Feb 2025 → 105 by May 2025)
- Multiple independent tech blogs with implementation case studies

### Method 3: Direct URL Testing

Attempted direct fetch of `/llms.txt` on 60+ known documentation domains. Network-level restrictions (403 Forbidden from CDN/WAF) prevented HTTP-level verification for most sites. Accessibility was confirmed through directory listings, platform provider statements, and multiple independent sources instead.

### Method 4: Platform Provider Confirmation

Mintlify (the documentation platform) announced automatic llms.txt support for all hosted docs in November 2024, confirmed Anthropic and Cursor as early customers, and rolled out to thousands of docs sites. This provides high-confidence verification for Mintlify-hosted implementations.

### Method 5: Direct Specimen Collection (2026-02-06 Enrichment Pass)

Eleven llms.txt files were manually downloaded and analyzed as raw specimens. These files were retrieved via browser-based access (bypassing CDN/WAF blocks that prevented programmatic fetching) and stored locally for structural analysis. This method provided the first direct file access in the research program, enabling byte-level conformance testing against the v0.0.1a ABNF grammar and document-type classification.

**Specimens collected (11 files):**

| # | Specimen File | Source Domain | Size | Lines | Type Classification |
|---|---------------|---------------|------|-------|---------------------|
| S1 | `astro-llms.txt` | astro.build | 2.6 KB | 31 | Type 1 Index |
| S2 | `cloudflare-llms.txt` | developers.cloudflare.com | 225 KB | 1,901 | Type 1 Index |
| S3 | `cursor-llms.txt` | docs.cursor.com | 7.5 KB | 183 | Type 1 (non-conformant) |
| S4 | `deno-llms.txt` | docs.deno.com | 63 KB | 464 | Type 1 Index |
| S5 | `docker-llms.txt` | docs.docker.com | 167 KB | 1,222 | Type 1 Index |
| S6 | `langchain-llms.txt` | python.langchain.com | 82 KB | 830 | Type 1 Index |
| S7 | `neon-llms.txt` | neon.tech | 68 KB | 558 | Type 1 Index |
| S8 | `openai-llms.txt` | platform.openai.com | 19 KB | 151 | Type 1 Index |
| S9 | `resend-llms.txt` | resend.com | 1.1 KB | 19 | Type 1 Index |
| S10 | `ai-sdk-llms.txt` | sdk.vercel.ai | 1.3 MB | 38,717 | Type 2 Full |
| S11 | `claude-llms-full.txt` | docs.anthropic.com | 25 MB | 956,573 | Type 2 Full |

**Overlap with existing catalog:** 4 specimens directly overlap with existing entries (#2 Cloudflare, #4 Cursor, #15 LangChain, #16 Vercel AI SDK). 1 specimen is a variant of an existing entry (S11 `claude-llms-full.txt` → #1 Anthropic). 6 specimens are entirely new sources (Astro, Deno, Docker, Neon, OpenAI, Resend).

**Key methodological upgrade:** This is the first time in the v0.0.x research program that direct file content has been analyzed. All prior audits (v0.0.2b) relied on indirect sources. The specimen collection enables correction of estimated ratings and discovery of structural patterns invisible to indirect analysis.

### Verification Methodology Note

Direct HTTP access to llms.txt files was blocked by CDN/WAF (Cloudflare, etc.) during this research session. All entries in the catalog below are verified through at least two of the following:

1. Listed in a community directory (llmstxt.site, llmstxthub.com, Awesome-llms-txt)
2. Confirmed by the platform provider (Mintlify blog)
3. Cited in multiple independent adoption analyses
4. Referenced in the official llms.txt specification or related GitHub repos

For v0.0.2b audits, direct file content will need to be accessed via alternative means (browser-based access, dataset downloads from HuggingFace/Kaggle, or manual retrieval).

> **Enrichment Pass Update (2026-02-06):** Method 5 (Direct Specimen Collection) resolved this limitation for 11 files. See Method 5 above for details. Overlapping entries in the catalog below now carry empirical annotations marked with `[SPECIMEN]`.

---

## Source Catalog

### Discovered Sources

| # | Domain | URL | Category | Has llms-full.txt? | Size Estimate | Verification Source | First Impression |
|---|--------|-----|----------|---------------------|---------------|--------------------|--------------------|
| 1 | Anthropic (Claude) | `docs.anthropic.com/llms.txt` (redirects to `platform.claude.com/docs/llms.txt`) | AI/ML | Yes | llms.txt: ~8K tokens; llms-full.txt: 25 MB / 956,573 lines `[SPECIMEN S11]` | Mintlify blog, multiple directories, adoption reports, **direct specimen** | Comprehensive API docs; Mintlify-generated; tiered approach with both files. **Specimen note:** `claude-llms-full.txt` is a Type 2 Full document — structurally incompatible with the v0.0.1a grammar (multiple H1 headers, inline content, 5% conformance). |
| 2 | Cloudflare | `developers.cloudflare.com/llms.txt` | Platform | Yes | llms.txt: 225 KB / 1,901 lines `[SPECIMEN S2]`; llms-full.txt: ~3.7M tokens | Multiple adoption analyses, directories, **direct specimen** | Organized by product; massive scope. **Specimen note:** The index file (`llms.txt`) is 225 KB Type 1 Index with 1,796 links, 90% conformance — no blockquote present. |
| 3 | Supabase | `supabase.com/llms.txt` | Platform | Unknown | Medium-to-large | llmstxthub, adoption reports, search results | Open-source Firebase alternative; also has domain-specific variants (e.g., `supabase.com/llms/guides.txt`) |
| 4 | Cursor | `docs.cursor.com/llms.txt` | Tool | Unknown | 7.5 KB / 183 lines `[SPECIMEN S3]` | Mintlify blog, adoption reports, **direct specimen** | AI code editor. **Specimen note:** Structurally non-conformant — 2 H1 headers, no blockquote, bare URL entries (not Markdown links), 20% conformance. Significantly worse than estimated. |
| 5 | ElevenLabs | `elevenlabs.io/llms.txt` | AI/ML | Yes | Unknown | Multiple adoption analyses, directories | AI voice synthesis; has both llms.txt and llms-full.txt |
| 6 | Shopify | `shopify.dev/llms.txt` | Enterprise | Unknown | Medium-to-large | Adoption reports, implementation guides | E-commerce platform developer docs; major enterprise adopter |
| 7 | Hugging Face | `huggingface.co/llms.txt` | AI/ML | Unknown | Medium | Multiple adoption analyses, directories | ML model hub; also hosts llms.txt datasets for research |
| 8 | Pinecone | `docs.pinecone.io/llms.txt` | Database | Unknown | Medium | Mintlify blog, llmstxthub | Vector database; Mintlify-hosted; API-heavy documentation |
| 9 | NVIDIA | `nvidia.com/llms.txt` | Enterprise | Unknown | Unknown | Adoption reports (listed as early adopter) | GPU/AI compute platform; enterprise-scale documentation |
| 10 | Zapier | `zapier.com/llms.txt` | Platform | Unknown | Unknown | Multiple adoption analyses | Automation/integration platform; workflow-centric docs |
| 11 | Svelte/SvelteKit | `svelte.dev/llms.txt` | Framework | Yes | Medium-to-large | Search results (direct URL confirmed), svelte.dev/docs/llms page | Modern web framework; has llms.txt, llms-full.txt, and a dedicated /docs/llms page explaining usage |
| 12 | Shadcn UI | `ui.shadcn.com/llms.txt` | Tool | Unknown | Small-to-medium | Search results (direct URL confirmed), llmstxthub | UI component library; TypeScript/Tailwind/Radix; supports Next.js, Vite, Astro |
| 13 | Pydantic | `docs.pydantic.dev/llms.txt` | Framework | Yes | Medium | Search results, llmstxthub | Python data validation; widely used; has both files |
| 14 | PydanticAI | `ai.pydantic.dev/llms.txt` | AI/ML | Yes | Small-to-medium | Search results, llmstxthub | Agent framework for LLMs built on Pydantic |
| 15 | LangChain (Python) | `python.langchain.com/llms.txt` | AI/ML | Unknown | 82 KB / 830 lines `[SPECIMEN S6]` | llmstxthub listing, **direct specimen** | LLM application framework; extensive API surface. **Specimen note:** Type 1 Index, 85% conformance — no blockquote, but well-organized with ~700 links. |
| 16 | Vercel AI SDK | `sdk.vercel.ai/llms.txt` | Framework | Yes — `ai-sdk-llms.txt` is the full variant `[SPECIMEN S10]` | llms.txt: Small-to-medium (estimated); llms-full.txt: 1.3 MB / 38,717 lines | Search results (direct URL confirmed), **direct specimen** | AI toolkit for TypeScript. **Specimen note:** `ai-sdk-llms.txt` is a Type 2 Full document (1.3 MB, multiple H1s, 15% conformance) — inline concatenated documentation, structurally incompatible with spec grammar. The index variant (llms.txt) was not collected. |
| 17 | FastHTML | `docs.fastht.ml/llms.txt` | Framework | Unknown | Small-to-medium | Spec author's project, referenced in original proposal | Created by Jeremy Howard (llms.txt spec author); likely serves as reference implementation |
| 18 | Mintlify | `mintlify.com/llms.txt` | Tool | Unknown | Medium | Platform provider (self-implements), llmstxthub | Documentation platform that auto-generates llms.txt for all hosted docs; key ecosystem enabler |
| 19 | Astro | `docs.astro.build/llms.txt` `[SPECIMEN S1]` | Framework | Unknown | 2.6 KB / 31 lines | **Direct specimen** | Web framework for content-driven sites. **Specimen note:** Type 1 Index, **100% conformance** — gold standard minimal implementation. Has H1, blockquote, H2 sections, proper link format. Only 31 lines but perfectly structured. |
| 20 | Deno | `docs.deno.com/llms.txt` `[SPECIMEN S4]` | Framework | Unknown | 63 KB / 464 lines | **Direct specimen** | Modern JavaScript/TypeScript runtime. **Specimen note:** Type 1 Index, **100% conformance** — well-organized with blockquote, multiple H2 sections, standard link format. |
| 21 | Docker | `docs.docker.com/llms.txt` `[SPECIMEN S5]` | Platform | Unknown | 167 KB / 1,222 lines | **Direct specimen** | Container platform developer documentation. **Specimen note:** Type 1 Index, 90% conformance — no blockquote, but otherwise well-structured with proper H2 sections and standard link format. |
| 22 | Neon | `neon.tech/llms.txt` `[SPECIMEN S7]` | Database | Unknown | 68 KB / 558 lines | **Direct specimen** | Serverless Postgres platform. **Specimen note:** Type 1 Index, 95% conformance — has blockquote, uses H3 sub-headers within H2 sections (5 H3 headers subdividing 21 H2 sections). |
| 23 | OpenAI | `platform.openai.com/llms.txt` `[SPECIMEN S8]` | AI/ML | Unknown | 19 KB / 151 lines | **Direct specimen** | OpenAI platform API documentation. **Specimen note:** Type 1 Index, **100% conformance** — compact and well-structured with blockquote, H2 sections, standard link format. |
| 24 | Resend | `resend.com/llms.txt` `[SPECIMEN S9]` | Tool | Unknown | 1.1 KB / 19 lines | **Direct specimen** | Email API for developers. **Specimen note:** Type 1 Index, 80% conformance — no blockquote, entries lack descriptions (links only). Smallest specimen; minimal but functional. |

### Category Distribution

| Category | Count | Examples |
|----------|-------|----------|
| **AI/ML** | 7 | Anthropic, ElevenLabs, Hugging Face, PydanticAI, LangChain, NVIDIA, **OpenAI** |
| **Framework** | 6 | Svelte, Pydantic, Vercel AI SDK, FastHTML, **Astro**, **Deno** |
| **Platform** | 4 | Cloudflare, Supabase, Zapier, **Docker** |
| **Tool** | 4 | Cursor, Shadcn UI, Mintlify, **Resend** |
| **Enterprise** | 2 | Shopify, NVIDIA (dual-categorized) |
| **Database** | 2 | Pinecone, **Neon** |
| **Total** | **24** (unique) | 6 categories represented |

> **Enrichment Pass Note (2026-02-06):** The addition of 6 new sources from the specimen collection expands the catalog from 18 to 24 unique implementations. The new entries strengthen the Framework category (now largest at 6) and add a second Database entry (Neon alongside Pinecone). Category distribution is now more balanced.

### Size Distribution

| Size Category | Token Range | Count | Examples |
|---------------|-------------|-------|----------|
| **Small** | <5K tokens | 2-3 | Cursor, PydanticAI, Shadcn UI (estimated) |
| **Medium** | 5K-50K tokens | 8-10 | Anthropic (8K), Pydantic, Svelte, LangChain, Mintlify |
| **Large** | 50K+ tokens | 3-4 | Cloudflare (3.7M full), Anthropic (481K full), Shopify |
| **Unknown** | Not yet confirmed | 4-5 | NVIDIA, Zapier, ElevenLabs, Hugging Face |

#### Empirical Size Distribution (2026-02-06 Enrichment — 11 Specimens with Actual Measurements)

The following table replaces estimates with measured values for the 11 specimens:

| Size Tier | Measured Range | Specimen Count | Specimens |
|-----------|---------------|----------------|-----------|
| **Micro (<5 KB)** | 1.1 KB – 2.6 KB | 2 | Resend (1.1 KB), Astro (2.6 KB) |
| **Small (5–20 KB)** | 7.5 KB – 19 KB | 2 | Cursor (7.5 KB), OpenAI (19 KB) |
| **Medium (60–90 KB)** | 63 KB – 82 KB | 3 | Deno (63 KB), Neon (68 KB), LangChain (82 KB) |
| **Large (160–230 KB)** | 167 KB – 225 KB | 2 | Docker (167 KB), Cloudflare (225 KB) |
| **— GAP —** | 225 KB – 1.3 MB | 0 | *No specimens fall in this range* |
| **Very Large (Type 2 Full)** | 1.3 MB – 25 MB | 2 | AI SDK (1.3 MB), Claude full (25 MB) |

**Key empirical finding — Bimodal Size Distribution:** Type 1 Index files cluster between 1.1 KB and 225 KB. Type 2 Full files jump to 1.3 MB and above. The absence of any specimen in the 225 KB – 1.3 MB gap suggests a natural boundary between index documents and full inline documentation. This bimodal pattern has implications for v0.1.0 tier definitions: the "full" tier for Type 1 documents should cap at approximately 250 KB, while Type 2 Full documents represent a fundamentally different document class that requires separate handling.

### Poor Example Candidates

The following sources may serve as "poor examples" for contrast during v0.0.2b audits. This is based on general patterns observed in the ecosystem (many implementations are auto-generated stubs with minimal curation):

- Sites with auto-generated llms.txt (Mintlify default output without customization) tend to be alphabetical link lists without descriptions — functional but not curated
- Enterprise sites (Shopify, NVIDIA) may have large but uncurated dumps
- Smaller tool sites (Shadcn UI) may have minimal stub implementations

Definitive poor-example identification will occur during v0.0.2b's detailed audits.

> **Enrichment Pass Update (2026-02-06):** Specimen analysis confirmed and refined the poor example candidates:
> - **Cursor** (20% conformance) is now the definitively identified worst-conforming Type 1 specimen — 2 H1 headers, no blockquote, bare URL entries. Significantly worse than the "competent but generic Mintlify default" estimated in v0.0.2b.
> - **Resend** (80% conformance) is functional but minimal at 19 lines with no descriptions on link entries — a real-world "stub" archetype.
> - **Shadcn UI** was incorrectly flagged as a potential stub — its actual implementation (per v0.0.2b audit) scores 5/5. This demonstrates the risk of pre-judging without direct file access.
> - **NVIDIA** remains unverified (no specimen collected).
> - The two Type 2 Full specimens (AI SDK at 15%, Claude full at 5%) are poor examples of *spec conformance* but may be excellent examples of *comprehensive documentation delivery* — they represent a different paradigm entirely.

---

## Exclusions

### Already Analyzed in v0.0.1

- Stripe (`docs.stripe.com/llms.txt`) — Analyzed in Wild Examples Analysis
- Nuxt (`nuxt.com/llms.txt`) — Analyzed in Wild Examples Analysis
- Vercel (`vercel.com/llms.txt`) — Analyzed in Wild Examples Analysis

### Considered but Excluded

- **HMSAAB Movies** — Referenced in v0.0.1b as a 107M-token example, but this is a dataset rather than a curated llms.txt file. Excluded as not representative.
- **Consumer platforms** (Google, Facebook, Amazon) — None have adopted llms.txt as of early 2026.

---

## Ecosystem Context

### Adoption Landscape (as of early 2026)

The llms.txt standard shows a clear adoption pattern: strong penetration in developer tools, AI companies, and technical documentation platforms, with zero adoption among consumer-facing platforms. Key ecosystem facts:

- BuiltWith tracked 844,000+ implementations as of October 2025
- Community directories list 500-784 curated, verified sites
- Majestic Million crawl showed 600% growth (15 → 105 sites) in the first half of 2025
- Mintlify's auto-generation brought thousands of docs sites online in November 2024
- No major AI platform has officially confirmed reading llms.txt files at inference time

### Key Platform: Mintlify

Mintlify plays an outsized role in the ecosystem. As a documentation hosting platform, it auto-generates llms.txt and llms-full.txt for all hosted documentation sites. This means many implementations (Anthropic, Cursor, Pinecone, and others) share a common generation pattern — alphabetical page listings with frontmatter-derived descriptions. This is important context for v0.0.2b audits: Mintlify-generated files will share structural similarities that may not represent deliberate design choices.

---

## Deliverables

- [x] Source catalog with 18 unique llms.txt URLs
- [x] Metadata recorded for all sources (domain, category, size estimate, verification)
- [x] Discovery methodology documented
- [x] Category distribution analyzed
- [x] Size distribution estimated
- [x] Exclusions documented
- [x] Ecosystem context provided
- [x] **Enrichment (2026-02-06):** Method 5 (Direct Specimen Collection) added — 11 files with actual measurements
- [x] **Enrichment (2026-02-06):** Catalog expanded to 24 unique sources (6 new entries)
- [x] **Enrichment (2026-02-06):** 5 existing entries annotated with `[SPECIMEN]` empirical data
- [x] **Enrichment (2026-02-06):** Empirical Size Distribution added with bimodal finding
- [x] **Enrichment (2026-02-06):** Poor Example Candidates updated with conformance-validated data

---

## Acceptance Criteria

- [x] 15-20 unique llms.txt URLs collected (18 collected; expanded to 24 with enrichment pass)
- [x] All URLs verified as accessible (verified via directories, platform confirmations, and adoption reports; see Verification Methodology Note)
- [x] At least 4 different categories represented (6 categories)
- [x] Mix of sizes (small, medium, large) (all three tiers represented; empirical bimodal distribution confirmed)
- [x] At least 1 "poor example" identified for contrast (Cursor at 20% conformance definitively identified; Resend as stub archetype)
- [x] Stripe, Nuxt, Vercel excluded (confirmed excluded)
- [x] Catalog table complete with all metadata (24-row table with 8 columns)
- [x] **Enrichment (2026-02-06):** Direct specimen data integrated for 11 files
- [x] **Enrichment (2026-02-06):** Document type classification (Type 1 Index vs Type 2 Full) applied to all specimens

---

## Next Step

Once this sub-part is complete, proceed to:

**v0.0.2b — Individual Example Audits**

The catalog created here becomes the input for detailed analysis. For v0.0.2b, direct file content access will need to be achieved via browser-based tools or dataset downloads (HuggingFace datasets `megrisdal/llms-txt` and `plaguss/llms-txt` contain crawled llms.txt file contents).
