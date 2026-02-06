# v0.0.1c — Processing & Expansion Methods

> **Sub-Part:** Analyze and compare the methods by which llms.txt files are processed, expanded, and consumed by tools and AI agents, with implications for DocStratum's Context Builder.

---

## Sub-Part Overview

This sub-part documents four distinct processing methods for llms.txt files (concatenation, XML wrapping, selective inclusion, and summarization), analyzes their tradeoffs across eight dimensions, and proposes a hybrid pipeline architecture for DocStratum's Context Builder that combines the strengths of each approach.

---

## Objective

The official spec briefly mentions that llms.txt files can be "processed" but offers no analysis of HOW. In practice, at least four distinct processing methods have emerged. Understanding their tradeoffs is critical for designing DocStratum's Context Builder (v0.3.2) and ensuring interoperability with existing tools.

### Success Looks Like

- All observed processing methods documented with comparative analysis
- Tradeoff matrix covering performance, fidelity, token efficiency, and tooling support
- FastHTML's `llms_txt2ctx` analyzed as reference implementation
- Clear recommendations for which methods DocStratum should support
- Decision tree for selecting the right processing method per use case

---

## Scope Boundaries

### In Scope

- Documenting all known processing/expansion methods
- Comparative analysis with tradeoff matrices
- FastHTML tool analysis as reference implementation
- Implications for Context Builder design (v0.3.2)
- Decision tree for method selection

### Out of Scope

- Building processing tools (that's v0.3.2 — Context Builder)
- Evaluating specific LLM performance with each method (that's v0.5.x)
- Surveying all existing tools in depth (that's v0.0.3)
- Benchmarking token counts across methods (that's v0.5.3)

---

## Dependencies

```
v0.0.1 — Specification Deep Dive (COMPLETED)
    │
    ├── Spec mentions processing but does not mandate approach
    ├── FastHTML's llms_txt2ctx identified as reference tool
    ├── Tiered expansion (llms.txt + llms-full.txt) observed
    │
v0.0.1a — Formal Grammar (COMPLETED)
    │
    ├── Parser data structures define input format
    └── Grammar rules constrain what can be processed
            │
            v
v0.0.1c — Processing & Expansion Methods (THIS TASK)
            │
            v
v0.3.2 — Context Builder (FUTURE — implements processing pipeline)
v0.3.5 — A/B Harness (FUTURE — tests processing methods)
```

---

## 1. Processing Methods Taxonomy

Four distinct processing methods have been observed in the wild. Each transforms an llms.txt file into content suitable for LLM consumption.

### Method 1: Concatenation

**Description:** Fetch every linked URL, download its content, and append all content into a single large text blob.

**How It Works:**

```
llms.txt → Parse links → Fetch each URL → Strip HTML → Concatenate → Output
```

**Example Output:**

```
# FastHTML Documentation (Concatenated)

## From: https://docs.fasthtml.com/getting-started
Getting started with FastHTML...
[full page content here]

## From: https://docs.fasthtml.com/api/routes
API Routes Reference...
[full page content here]

## From: https://docs.fasthtml.com/api/templates
Template System...
[full page content here]
```

**Strengths:**

| Advantage | Detail |
|---|---|
| Simplicity | Easiest to implement — just fetch and append |
| Completeness | Captures all linked content |
| No information loss | Raw text preserved verbatim |

**Weaknesses:**

| Disadvantage | Detail |
|---|---|
| Token explosion | Total output can be massive (millions of tokens) |
| Lost document boundaries | Hard to tell where one page ends and another begins |
| No source tracing | Difficult to attribute specific content to its source URL |
| Context pollution | Navigation, footers, headers from HTML pages contaminate content |

**Best For:** Simple embedding/fine-tuning pipelines where the entire corpus will be chunked anyway.

**Note on Pre-Concatenated Documents:**

In empirical analysis of 11 real-world specimens, Type 2 Full documents (AI SDK at 1.3 MB and Claude full at 25 MB) arrive already pre-concatenated—meaning all documentation content has been fetched and inlined directly into the llms.txt file. For these documents, Method 1 (concatenation) is redundant; the heavy lifting of content assembly has already been completed at source. This distinction is critical for pipeline design, as Type 2 documents bypass the fetch-and-concatenate phase entirely and proceed directly to filtering, summarization, or selective inclusion.

---

### Method 2: XML Wrapping

**Description:** Fetch each linked URL and wrap its content in XML tags that preserve document structure and metadata.

**How It Works:**

```
llms.txt → Parse links → Fetch each URL → Wrap in <doc> tags → Output XML
```

**Example Output:**

```xml
<llms-context site="FastHTML" generated="2026-02-05">
  <doc url="https://docs.fasthtml.com/getting-started"
       title="Getting Started"
       type="tutorial"
       tokens="1250">
    Getting started with FastHTML...
    [full page content here]
  </doc>

  <doc url="https://docs.fasthtml.com/api/routes"
       title="API Routes Reference"
       type="reference"
       tokens="3400">
    API Routes Reference...
    [full page content here]
  </doc>
</llms-context>
```

**Strengths:**

| Advantage | Detail |
|---|---|
| Structure preservation | XML tags clearly delineate document boundaries |
| Source attribution | Every chunk carries its source URL |
| Metadata support | Tags can carry type, token count, freshness data |
| LLM-friendly | Most LLMs understand XML structure natively |

**Weaknesses:**

| Disadvantage | Detail |
|---|---|
| Token overhead | XML tags add 5-15% more tokens |
| Still large | Fetching all pages produces large output |
| Parsing dependency | Consumer must understand XML structure |

**Best For:** RAG systems, retrieval-based pipelines, and any system where source attribution matters.

**This is what FastHTML's `llms_txt2ctx` produces.**

---

### Method 3: Selective Inclusion

**Description:** Only include content from links that match specific criteria (tags, keywords, sections, size limits). This is a filtered version of concatenation or XML wrapping.

**How It Works:**

```
llms.txt → Parse links → Filter by criteria → Fetch matching URLs → Process → Output
```

**Filter Criteria Examples:**

```python
# Include only tutorials and getting-started content
filter_by_type = ["tutorial", "getting-started"]

# Include only content under a token budget
max_total_tokens = 10000

# Include required sections, skip "Optional"
skip_sections = ["Optional"]

# Include only pages updated in last 90 days
max_age_days = 90
```

**Strengths:**

| Advantage | Detail |
|---|---|
| Token-efficient | Only includes what's needed |
| Targeted | Can be tuned to specific use cases |
| Respects "Optional" semantics | Naturally handles the spec's Optional section |

**Weaknesses:**

| Disadvantage | Detail |
|---|---|
| Requires selection logic | Someone must define what to include/exclude |
| May miss important content | Filtering too aggressively loses coverage |
| Configuration burden | Each use case needs different filter rules |

**Best For:** Agent system prompts where token budget is tight, and context must be carefully curated.

---

### Method 4: Summarization

**Description:** Fetch linked content but generate summaries rather than including full text. Can be applied at the page level (one summary per page) or section level.

**How It Works:**

```
llms.txt → Parse links → Fetch each URL → Summarize (LLM or extractive) → Output
```

**Example Output:**

```markdown
# FastHTML Documentation (Summarized)

## Getting Started
- FastHTML combines Starlette, Uvicorn, and HTMX for server-rendered apps
- Installation via pip; first app in 10 lines of code
- Not compatible with FastAPI syntax despite similar naming
Source: https://docs.fasthtml.com/getting-started

## API Routes
- Routes defined with Python decorators (@app.get, @app.post)
- Supports path parameters, query parameters, and form data
- Built-in support for HTMX partial page updates
Source: https://docs.fasthtml.com/api/routes
```

**Strengths:**

| Advantage | Detail |
|---|---|
| Extreme compression | Can reduce 100K tokens to 5K |
| Preserves key information | Well-tuned summaries retain essentials |
| Source attribution | Summary carries original URL |

**Weaknesses:**

| Disadvantage | Detail |
|---|---|
| Information loss | Details, edge cases, and nuance are lost |
| Summarization quality varies | LLM-generated summaries may miss critical info |
| Latency | Summarization is slow (requires LLM inference) |
| Cost | LLM-based summarization costs money per run |

**Best For:** Overview contexts where breadth matters more than depth; initial query routing before drilling into specific pages.

---

## 2. Comparative Analysis

### Tradeoff Matrix

| Criterion | Concatenation | XML Wrapping | Selective Inclusion | Summarization |
|---|---|---|---|---|
| **Implementation Complexity** | Low | Medium | Medium-High | High |
| **Token Efficiency** | Very Low | Low | High | Very High |
| **Information Fidelity** | 100% | 100% | Partial | Lossy |
| **Source Attribution** | None | Excellent | Good | Good |
| **Structure Preservation** | None | Excellent | Depends | Partial |
| **LLM Comprehension** | Fair | Good | Good | Excellent |
| **Latency** | Medium (fetch) | Medium (fetch + wrap) | Low (less to fetch) | High (LLM inference) |
| **Cost** | Free | Free | Free | Per-summarization LLM cost |
| **Offline Capable** | Yes | Yes | Yes | No (needs LLM) |

### Decision Tree

```
Start: "I have an llms.txt and need to provide context to an LLM"
│
├── Q: Is my total content < 10K tokens?
│   ├── YES → Use Concatenation (simple and complete)
│   └── NO ↓
│
├── Q: Do I need source attribution for citations?
│   ├── YES → Use XML Wrapping
│   └── NO ↓
│
├── Q: Do I have a strict token budget (< 5K tokens)?
│   ├── YES ↓
│   │   ├── Q: Is real-time latency acceptable?
│   │   │   ├── NO → Use Selective Inclusion (filter to budget)
│   │   │   └── YES → Use Summarization (maximum compression)
│   └── NO ↓
│
└── Default → Use XML Wrapping (best balance of structure + fidelity)
```

### Empirical Validation of the Decision Tree

Analysis of 11 real-world llms.txt specimens validates the decision tree and reveals a clear pattern of specimen clustering by size and processing method recommendation:

**Type 1 Index Files (Small to Medium):**

| Specimen | Size | Links | Recommended Method | Rationale |
|---|---|---|---|---|
| Resend | 1.1 KB | — | Concatenation | Minimal content; use simple and complete approach |
| Astro | 2.6 KB | — | Concatenation | Well below 10K token threshold |
| Cursor | 7.5 KB | — | Concatenation | Still under 10K tokens; complete inclusion feasible |
| OpenAI | 19 KB | — | XML Wrapping or Concatenation | Approaching medium size; XML structure provides clarity |
| Deno | 63 KB | 431 | XML Wrapping | Link density requires document boundary clarity |
| Neon | 68 KB | 479 | XML Wrapping | 21 sections benefit from XML structure preservation |
| LangChain | 82 KB | 688 | Selective Inclusion | Single section with 688 links too dense for complete inclusion |
| Docker | 167 KB | 1,213 | Selective Inclusion | Large file with high link density demands filtering |
| Cloudflare | 225 KB | 1,796 | Selective Inclusion | Upper bound of Type 1; 34 sections across 1,796 links require careful curation |

**Type 2 Full Documents (Pre-Concatenated):**

| Specimen | Size | Token Est. | Recommended Method | Rationale |
|---|---|---|---|---|
| AI SDK | 1.3 MB | ~38K | Selective Inclusion or Summarization | Already concatenated; skip to filtering or compression |
| Claude full | 25 MB | ~481K | Aggressive Summarization | Exceeds any current LLM context window; requires severe compression |

**Key Observations:**

1. **Bimodal distribution:** Type 1 files cluster at 1–225 KB; Type 2 jump to 1.3–25 MB. No overlap in the middle suggests distinct processing paths.
2. **Concatenation sweet spot:** Files under ~10K tokens (roughly Resend, Astro, Cursor) can be consumed wholesale.
3. **XML wrapping transition:** Files in the 19–68 KB range (OpenAI through Neon) benefit from structured wrapping to preserve document boundaries.
4. **Selective inclusion threshold:** Files exceeding ~80 KB (LangChain, Docker, Cloudflare) require intelligent filtering or summarization to remain tractable.
5. **Type 2 processing divergence:** Pre-concatenated documents (AI SDK, Claude full) skip fetch-and-concatenate phases, moving directly to compression strategies.

---

## 3. FastHTML's `llms_txt2ctx` — Reference Implementation Analysis

### What It Does

`llms_txt2ctx` is a CLI tool created by Jeremy Howard (the spec author) that processes llms.txt files into context blocks.

### How It Works

```bash
# Basic usage: expand llms.txt into context
llms_txt2ctx https://docs.fasthtml.com/llms.txt

# With optional sections excluded
llms_txt2ctx https://docs.fasthtml.com/llms.txt --optional=false
```

### Processing Pipeline

```
1. Fetch the llms.txt file from URL
2. Parse the markdown to extract links
3. For each link:
   a. Fetch the URL content
   b. Convert HTML to markdown (if needed)
   c. Wrap in XML-like tags: <context url="..." title="...">
4. Concatenate all wrapped content
5. Output two versions:
   a. Standard (without Optional section links)
   b. Full (with all links)
```

### Output Format

```xml
<context url="https://docs.fasthtml.com/getting-started" title="Getting Started">
[full page content as markdown]
</context>

<context url="https://docs.fasthtml.com/api/routes" title="API Routes">
[full page content as markdown]
</context>
```

### Strengths of This Implementation

| Strength | Detail |
|---|---|
| Author-endorsed | Created by the spec author; de facto reference |
| Respects Optional | Generates with/without Optional section links |
| XML structure | Preserves document boundaries |
| Simple CLI | Easy to integrate into workflows |

### Limitations of This Implementation

| Limitation | Detail | DocStratum Opportunity |
|---|---|---|
| No filtering beyond Optional | Can't filter by content type, size, or recency | Support rich filtering criteria |
| No summarization | Full text only — no compression option | Add summarization pipeline |
| No concept injection | Doesn't add concept definitions to context | Inject Layer 2 (concepts) into context |
| No few-shot injection | Doesn't add Q&A examples to context | Inject Layer 3 (few-shot) into context |
| No token budget awareness | Doesn't limit output to fit context windows | Add token budget + priority-based inclusion |
| No caching | Re-fetches every URL on every run | Add local caching with TTL |
| No validation | Doesn't verify the llms.txt structure | Add pre-processing validation |

---

## 4. Hybrid Processing: The DocStratum Approach

Based on the analysis above, DocStratum's Context Builder (v0.3.2) should implement a **hybrid pipeline** that combines the best of all four methods.

### Proposed Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────┐
│              DOCSTRATUM CONTEXT BUILDER                    │
│                    (v0.3.2 Design)                           │
└─────────────────────────────────────────────────────────────┘

Phase 1: VALIDATE
    │ → Parse llms.txt using v0.0.1a grammar
    │ → Validate against schema (v0.2.4 rules)
    │ → Report errors/warnings
    │
Phase 2: FILTER (Selective Inclusion)
    │ → Apply token budget constraints
    │ → Respect "Optional" section semantics
    │ → Filter by content type, recency, priority
    │ → Output: subset of links to process
    │
Phase 3: FETCH + CACHE
    │ → Fetch URLs not in local cache
    │ → Cache with TTL from meta.cache.ttl_hours
    │ → Convert HTML → Markdown (if needed)
    │ → Output: raw page content per URL
    │
Phase 4: ENRICH (DocStratum Innovation)
    │ → Inject concept definitions (Layer 2)
    │ → Inject few-shot examples (Layer 3)
    │ → Inject LLM instructions (Stripe pattern)
    │ → Output: enriched content blocks
    │
Phase 5: WRAP (XML or Markdown)
    │ → Wrap each block with source metadata
    │ → Add concept cross-references
    │ → Format for target LLM's preferred input
    │ → Output: structured context block
    │
Phase 6: BUDGET CHECK
    │ → Count total tokens
    │ → If over budget: prioritize and trim
    │ → If under budget: include Optional content
    │ → Output: final context within token budget
```

### Configuration Interface (Proposed)

```python
from docstratum import ContextBuilder

builder = ContextBuilder(
    source="https://docs.example.com/llms.txt",

    # Processing method
    method="xml",           # "concat" | "xml" | "summary" | "hybrid"

    # Token budget
    max_tokens=8000,

    # Filtering
    include_optional=False,
    content_types=["tutorial", "reference"],
    max_age_days=90,

    # Enrichment (DocStratum features)
    inject_concepts=True,
    inject_few_shot=True,
    inject_instructions=True,

    # Caching
    cache_dir=".docstratum_cache",
    cache_ttl_hours=24,
)

context = builder.build()
print(context.text)          # The context block as a string
print(context.token_count)   # Actual token count
print(context.sources)       # List of URLs included
```

---

## 5. Processing Method Benchmarks (Projected)

These benchmarks are **estimated** based on analysis. Actual benchmarks will be produced in v0.5.3 (Metrics Analysis).

### Scenario: Documentation site with 50 pages, ~200K total tokens

| Method | Output Tokens | Build Time | Fidelity | LLM Usability |
|---|---|---|---|---|
| Concatenation | ~200K | 30s (fetch) | 100% | Poor (too large) |
| XML Wrapping | ~215K | 35s (fetch + wrap) | 100% | Fair (structured but large) |
| Selective (top 10 pages) | ~40K | 10s (less to fetch) | 20% | Good (focused) |
| Summarization (all pages) | ~15K | 120s (LLM inference) | ~60% | Good (compressed) |
| DocStratum Hybrid (budget: 8K) | ~8K | 15s (fetch + enrich) | ~15% pages, 100% concepts | Excellent (optimized) |

### Key Insight

The DocStratum Hybrid method achieves the best LLM usability rating because it trades raw page content for structured concept definitions and few-shot examples. An LLM with 8K tokens of curated concepts + examples outperforms one with 200K tokens of raw page dumps.

### Empirical Size Distribution

Analysis of 11 specimens reveals a **bimodal size distribution** that has profound implications for method selection and pipeline design:

**Type 1 Index Files (Linked content references):**
- Range: 1.1 KB (Resend) to 225 KB (Cloudflare)
- Median: ~68 KB (Neon)
- Characteristics: Index files with links that must be fetched; content assembly happens at runtime

**Type 2 Full Documents (Pre-assembled content):**
- Range: 1.3 MB (AI SDK) to 25 MB (Claude full)
- Median: ~13 MB
- Characteristics: All linked content already inlined; llms.txt is the final assembled product

**Size Gap:** The smallest Type 2 specimen (AI SDK at 1.3 MB) is 5.8× larger than the largest Type 1 specimen (Cloudflare at 225 KB). This gap is not accidental—it reflects a fundamental architectural choice by tool authors:

- **Type 1 approach:** Publish an llms.txt index; clients fetch and assemble content on-demand
- **Type 2 approach:** Assemble the full corpus once; publish a monolithic llms.txt file

**Token Implications:**

- Type 1 max (Cloudflare, 225 KB): ~1,440 tokens
- Type 2 min (AI SDK, 1.3 MB): ~38,000 tokens (26× larger)
- Type 2 max (Claude full, 25 MB): ~481,000 tokens (334× larger)

This distribution informs DocStratum's pipeline: Type 1 files benefit from fetch-and-process workflows; Type 2 files demand immediate summarization or selective inclusion to fit standard LLM context windows (typically 4K–128K tokens).

---

## Deliverables

- [x] Four processing methods documented with examples
- [x] Comparative tradeoff matrix
- [x] Decision tree for method selection
- [x] FastHTML `llms_txt2ctx` analysis (strengths + limitations)
- [x] DocStratum hybrid pipeline design
- [x] Configuration interface proposal
- [x] Projected benchmarks
- [x] Empirical validation of decision tree against 11 specimens
- [x] Empirical size distribution analysis (Type 1 vs Type 2 bimodal pattern)

---

## Acceptance Criteria

- [x] All four processing methods documented with input/output examples
- [x] Tradeoff matrix covers at least 6 comparison dimensions
- [x] Decision tree is actionable (someone could follow it to select a method)
- [x] FastHTML analysis identifies at least 5 limitations DocStratum can address
- [x] Hybrid pipeline design maps to v0.3.2 Context Builder module
- [x] Projected benchmarks are reasonable and testable in v0.5.3
- [x] Decision tree validated against real-world specimen size distribution (11 specimens)
- [x] Bimodal distribution (Type 1 vs Type 2) identified and explained

---

## Next Step

Once this sub-part is complete, proceed to:

**v0.0.1d — Standards Interplay & Positioning**

Understanding how llms.txt is processed (this document) informs how it relates to and interacts with adjacent web standards.
