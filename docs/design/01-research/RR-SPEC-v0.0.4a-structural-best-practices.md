# v0.0.4a: Structural Best Practices

**Sub-Part Objective:** Establish definitive rules and guidelines for the structural organization of llms.txt files to ensure consistency, consumability, and optimal LLM parsing across all implementations.

**Version:** v0.0.4a
**Status:** COMPLETE
**Last Updated:** 2026-02-06
**Dependencies:** v0.0.1a (Grammar), v0.0.2c (Pattern Analysis), v0.0.3 (Audit Findings)

---

## 1. Scope & Boundaries

### In Scope
- File location strategies and primary/fallback patterns
- Markdown format compliance and technical requirements
- File size budgets and token allocation per section
- Canonical section ordering and hierarchy rules
- Section naming conventions derived from frequency analysis
- "Optional" section usage guidelines
- Tiered file strategy (single vs. multi-file approaches)
- Structural compliance validation framework

### Out of Scope
- Content quality (covered in v0.0.4b)
- Anti-pattern catalog (covered in v0.0.4c)
- Strategic decision rationale (covered in v0.0.4d)
- LLM consumption behavior analysis
- Specific tool implementations

---

## 2. Dependencies Diagram

```
┌─────────────────────────────────────────────────────────┐
│ v0.0.4a: Structural Best Practices                     │
└─────────────────────────────────────────────────────────┘
         ↑                    ↑                    ↑
         │                    │                    │
    ┌────┴─────┐      ┌──────┴────────┐    ┌─────┴──────┐
    │v0.0.1a   │      │v0.0.2c        │    │v0.0.3      │
    │Grammar & │      │Pattern        │    │Audit       │
    │Syntax    │      │Analysis       │    │Findings    │
    └──────────┘      └───────────────┘    └────────────┘
         │                    │                    │
         └────────────────────┴────────────────────┘
                 (Informs Section Naming
                  & Structural Rules)
```

---

## 3. File Location Rules

### 3.1 Primary Location

```
project-root/
└── llms.txt
```

**Requirements:**
- Located at repository root (web root for websites)
- Filename: `llms.txt` (lowercase, no variations)
- Single location per project/domain
- Accessible via `https://domain.com/llms.txt` (web context)
- No authentication required for access

### 3.2 Fallback Locations

When primary location is unavailable or inappropriate:

| Scenario | Fallback Location | Use Case |
|----------|-------------------|----------|
| Web framework routing | `/public/llms.txt` | Rails, Django, Laravel apps |
| Static site generators | `/.well-known/llms.txt` | Hugo, Jekyll, Next.js |
| API-only projects | `/docs/llms.txt` | API documentation context |
| Monorepos | `/{service}/llms.txt` | Multi-service projects |
| Private codebase | `docs/internal/llms.txt` | Internal documentation |

### 3.3 Location Discovery Protocol

LLMs should follow this priority order:
1. `/llms.txt` (root)
2. `/.well-known/llms.txt` (standards directory)
3. `/docs/llms.txt` (documentation)
4. `/public/llms.txt` (web framework public dir)
5. Search repository for any `llms.txt`

---

## 4. File Format Requirements

### 4.1 Markdown Compliance

**Standard:** CommonMark 0.30 with GitHub Flavored Markdown extensions

| Requirement | Specification | Validation |
|-------------|---------------|-----------|
| **Encoding** | UTF-8 only (no BOM) | File header inspection |
| **Line Endings** | LF (`\n`) only | No CRLF (`\r\n`) |
| **Headings** | ATX style (`#`) only | No setext style underlines |
| **Code Blocks** | Fenced (triple backtick) | No indented blocks |
| **Lists** | Consistent markers (- or *) | No mixed markers in sequence |
| **Links** | Reference or inline | No auto-linked bare URLs |

### 4.2 Technical Encoding

```markdown
# UTF-8 Encoding Requirements

File must start with:
- No Byte Order Mark (BOM)
- First bytes: `23 20 ...` (# character)

Line endings:
- LF: `0x0A` (Unix/Linux/Mac)
- NOT CRLF: `0x0D 0x0A` (Windows)

Character restrictions:
- No null bytes (0x00)
- No control characters (0x01-0x08, 0x0B-0x0C, 0x0E-0x1F)
- Unicode 5.0+ permitted
```

### 4.3 Validation Script (Pseudocode)

```python
def validate_format(file_path):
    """Validate llms.txt format compliance"""

    # Check encoding
    with open(file_path, 'rb') as f:
        content = f.read()

    # No BOM
    assert not content.startswith(b'\xef\xbb\xbf'), "UTF-8 BOM present"

    # LF only
    assert b'\r\n' not in content, "CRLF line endings found"
    assert b'\r' not in content, "CR line endings found"

    # Decode and parse
    text = content.decode('utf-8')

    # Check for control characters
    for char in text:
        assert ord(char) >= 0x20 or char in '\n\t', \
            f"Control character found: {repr(char)}"

    # Parse Markdown
    ast = parse_markdown(text)

    # Validate structure (see Section 5)
    validate_structure(ast)

    return True
```

---

## 5. File Size Guidelines & Token Budgets

### 5.1 Overall Size Targets

| File Tier | Lines | Characters | Estimated Tokens | Use Case |
|-----------|-------|-----------|-------------------|----------|
| **Standard** | 200-500 | 5K-15K | 1.5K-4.5K | Most projects |
| **Comprehensive** | 500-1500 | 15K-40K | 4.5K-12K | Large projects |
| **Minimal** | 50-200 | 1.5K-5K | 500-1.5K | Very focused docs |
| **Full** | 1500+ | 40K+ | 12K+ | Master index (llms-full.txt) |

### 5.2 Section Token Budgets (Standard 3K-Token File)

```
ALLOCATION: 3000 tokens total

┌─────────────────────────────────────┐
│ H1 Title & Blockquote       200 tok  │ 7%
├─────────────────────────────────────┤
│ Master Index & Links        600 tok  │ 20%
├─────────────────────────────────────┤
│ Getting Started             400 tok  │ 13%
├─────────────────────────────────────┤
│ Core Concepts              800 tok  │ 27%
├─────────────────────────────────────┤
│ Advanced Features           600 tok  │ 20%
├─────────────────────────────────────┤
│ FAQ/Troubleshooting         300 tok  │ 10%
├─────────────────────────────────────┤
│ Optional Sections           100 tok  │ 3%
└─────────────────────────────────────┘
```

### 5.2b Section Token Budgets (Comprehensive 12K-Token File)

```
ALLOCATION: 12000 tokens total

┌─────────────────────────────────────┐
│ H1 Title & Blockquote       300 tok  │  2.5%
├─────────────────────────────────────┤
│ Master Index & Links        800 tok  │  6.7%
├─────────────────────────────────────┤
│ LLM Instructions            600 tok  │  5.0%
├─────────────────────────────────────┤
│ Getting Started            1200 tok  │ 10.0%
├─────────────────────────────────────┤
│ Core Concepts              2500 tok  │ 20.8%
├─────────────────────────────────────┤
│ API Reference              2000 tok  │ 16.7%
├─────────────────────────────────────┤
│ Examples / Use Cases       1500 tok  │ 12.5%
├─────────────────────────────────────┤
│ Configuration               800 tok  │  6.7%
├─────────────────────────────────────┤
│ Advanced Topics            1000 tok  │  8.3%
├─────────────────────────────────────┤
│ Troubleshooting / FAQ       800 tok  │  6.7%
├─────────────────────────────────────┤
│ Optional Sections           500 tok  │  4.2%
└─────────────────────────────────────┘
```

**Rationale:** At 12K tokens, there is room to include an LLM Instructions section (the strongest quality differentiator per v0.0.2d) and expand Core Concepts with code examples (the strongest quality predictor per v0.0.2c, r ≈ 0.65). The API Reference and Examples sections receive significant allocations because concrete code samples are the single strongest driver of LLM-friendliness.

### 5.2c Section Token Budgets (Full 40K-Token File)

```
ALLOCATION: 40000 tokens total

┌─────────────────────────────────────┐
│ H1 Title & Blockquote       400 tok  │  1.0%
├─────────────────────────────────────┤
│ Master Index & Links       1200 tok  │  3.0%
├─────────────────────────────────────┤
│ LLM Instructions           1500 tok  │  3.8%
├─────────────────────────────────────┤
│ Getting Started            3000 tok  │  7.5%
├─────────────────────────────────────┤
│ Core Concepts              6000 tok  │ 15.0%
├─────────────────────────────────────┤
│ Architecture               4000 tok  │ 10.0%
├─────────────────────────────────────┤
│ API Reference              6000 tok  │ 15.0%
├─────────────────────────────────────┤
│ Examples / Use Cases       4000 tok  │ 10.0%
├─────────────────────────────────────┤
│ Configuration              2000 tok  │  5.0%
├─────────────────────────────────────┤
│ Integrations / SDKs        2500 tok  │  6.3%
├─────────────────────────────────────┤
│ Advanced Topics            3000 tok  │  7.5%
├─────────────────────────────────────┤
│ Troubleshooting / FAQ      2400 tok  │  6.0%
├─────────────────────────────────────┤
│ Cross-Cutting Patterns     2000 tok  │  5.0%
├─────────────────────────────────────┤
│ Optional Sections          2000 tok  │  5.0%
└─────────────────────────────────────┘
```

**Rationale:** At 40K tokens, every canonical section type from v0.0.2c receives dedicated allocation. Architecture and Cross-Cutting Patterns — first-class sections in gold standard implementations (Pydantic, Svelte) — receive their own budgets. The anti-pattern threshold of 50K tokens (v0.0.2d) is respected with a 20% buffer. The Full tier targets the "deep dives, research, offline agents" use case.

**Anti-pattern warning:** Files exceeding 50K tokens enter the "degradation zone" where RAG quality declines (Svelte guidance). Files exceeding 100K tokens require decomposition into per-product or per-domain variants (Cloudflare pattern from v0.0.2c).

### 5.3 Token Budget Enforcement

**For Comprehensive Files (12K tokens):**
- Primary sections: 2.5K-3.5K each
- Secondary sections: 1.5K-2.5K each
- Metadata/index: 500-1K
- Optional: 0-2K

**Token calculation method:**
```python
def estimate_tokens(text: str) -> int:
    """Rough token estimate: ~0.3 tokens per character"""
    return len(text) / 3.3

def validate_token_budget(section: str, limit: int) -> bool:
    tokens = estimate_tokens(section)
    return tokens <= limit * 1.1  # 10% buffer
```

---

## 6. Section Ordering Rules

### 6.1 Canonical H1 → Blockquote → Body Pattern

```markdown
# [TITLE]

> [DESCRIPTION BLOCKQUOTE]

[INTRODUCTORY PARAGRAPH - optional]

## Section 1: [First Section]
...

## Section 2: [Second Section]
...

## Optional: [Optional Content]
...
```

**Rationale from v0.0.2c:** Analysis of 18 audited llms.txt implementations shows:
- 100% (18/18) of audited files use H1 title + blockquote as their opening pattern
- 5-star implementations (Svelte, Shadcn UI, Pydantic, Vercel AI SDK) all follow strict H1 → blockquote → body ordering with 98% structural element compliance
- Sites scoring 2–3/5 (Cursor, NVIDIA) had significant structural gaps — missing versioning, maintainer info, and organizational depth
- The ABNF grammar (v0.0.1a) enforces this ordering: `h1-title → [blockquote-desc] → [body-content] → 1*file-list-section`
- LLM preference: explicit structure signals section hierarchy, enabling predictable parsing

### 6.2 Mandatory First Section: Master Index

```markdown
# Project Documentation

> Description of the project in 1-2 sentences

## 1. Master Index

**Quick Navigation:**

| Section | Purpose | Link |
|---------|---------|------|
| Getting Started | Quick onboarding | [→](#getting-started) |
| Architecture | System design | [→](#architecture) |
| API Reference | Endpoints & schemas | [→](#api-reference) |

**Key Concepts:** [concept1], [concept2], [concept3]

**Resources:** [external link 1], [external link 2]
```

### 6.3 Section Ordering Hierarchy

**Strict Order (MUST follow):**

1. **H1 Title** - Project/product name
2. **Blockquote** - One-sentence description
3. **Master Index** - Navigation and key links
4. **Getting Started** - First steps
5. **Core Concepts** - Central ideas
6. **[Domain Sections]** - Architecture, API, etc.
7. **Advanced Topics** - Deeper dives
8. **FAQ/Troubleshooting** - Common issues
9. **Optional** - Non-essential content
10. **Resources** - External links (optional)

---

## 7. Section Naming Conventions

### 7.1 Frequency Analysis Results (from v0.0.2c)

Analysis of 18 audited llms.txt implementations across 6 categories identified the following section frequency patterns:

| Rank | Section Name | Count | Frequency | Canonical? |
|------|--------------|-------|-----------|-----------|
| 1 | Getting Started / Overview | 14/18 | 78% | ✓ YES |
| 2 | API Reference | 12/18 | 67% | ✓ YES |
| 3 | Core Concepts / Architecture | 10/18 | 56% | ✓ YES |
| 4 | Configuration / Settings | 8/18 | 44% | ✓ YES |
| 5 | Integrations / SDKs / Ecosystem | 8/18 | 44% | ✓ YES |
| 6 | Examples / Use Cases / Tutorials | 6/18 | 33% | ✓ YES |
| 7 | Deployment / Hosting / Infrastructure | 6/18 | 33% | ✓ YES |
| 8 | Troubleshooting / Error Handling / FAQ | 5/18 | 28% | ✓ YES |
| 9 | Changelog / Release Notes | 4/18 | 22% | ✓ YES |
| 10 | Best Practices / Performance Tips | 3/18 | 17% | ✓ YES |
| 11 | Community / Support / Contributing | 3/18 | 17% | ✗ NO* |

*Contributing and community content should reside in `CONTRIBUTING.md` or external community pages, not llms.txt

**Sample context:** This data is drawn from 18 diverse implementations across AI/ML (6), Framework (4), Platform (3), Tool (3), Enterprise (2), and Database (1) categories. While the sample is small for formal statistical inference, it provides strong directional indicators validated against v0.0.1's independent spec analysis. The correlation between having 5–7 thoughtfully organized sections and quality scores is strong (r ≈ 0.60).

### 7.2 Canonical Section Names

When choosing section names, prefer these canonical forms:

```
CANONICAL NAMES (Use These)
├── Getting Started
├── Architecture
├── Core Concepts
├── Configuration
├── API Reference
├── Examples / Usage Examples
├── Advanced Topics
├── FAQ / Frequently Asked Questions
├── Troubleshooting
├── Best Practices
├── Integration Guides
├── Performance Tuning
└── Resources

DEPRECATED NAMES (Avoid These)
├── Introduction (use "Getting Started")
├── System Design (use "Architecture")
├── Guide (be specific)
├── Details (too vague)
├── Misc / Miscellaneous (organize properly)
└── Other (find real category)
```

### 7.3 Naming Rules

**Rule 1: Be Specific**
- ✗ Bad: "Details"
- ✓ Good: "Database Schema"

**Rule 2: Use Gerunds for Actions**
- ✗ Bad: "Deploy"
- ✓ Good: "Deploying Your Application"

**Rule 3: Avoid Generic Abbreviations**
- ✗ Bad: "API Ref"
- ✓ Good: "API Reference"

**Rule 4: Order by Logical Flow, Not Alphabetical**
- ✗ Bad: Architecture, API Reference, Getting Started
- ✓ Good: Getting Started, Architecture, API Reference

---

## 8. "Optional" Section Usage Guide

### 8.1 Purpose and Scope

The "Optional" section contains content that is:
- Non-essential for core understanding
- Supplementary or advanced
- Lower token priority
- Can be safely skipped without impact

### 8.2 What Goes in Optional

**Include in Optional:**
- Theoretical background/academic references
- Historical evolution of the project
- Benchmarking details
- Edge cases and corner cases
- Deprecated features (with migration paths)
- Alternative approaches (not recommended)
- Links to academic papers
- Performance metrics

**Keep in Main Sections:**
- Security implications
- Breaking changes
- Critical warnings
- Core functionality
- Common use cases

### 8.3 Optional Section Format

```markdown
## Optional: Historical Context & Research

This section contains background information that enriches understanding
but is not required for using [project].

> **Note:** This content has ~500 tokens. Safe to skip if optimizing for
> context window.

### Academic Foundation

[Background information...]

### Evolution Timeline

[Timeline of changes...]

### Theoretical Extensions

[Advanced theory...]
```

### 8.4 Token Accounting

```python
# In token budget calculations, Optional sections get lower priority:

PRIORITY_WEIGHTING = {
    "Getting Started": 1.0,      # Critical
    "Core Concepts": 1.0,        # Critical
    "API Reference": 1.0,        # Critical
    "Examples": 0.8,             # Important
    "Troubleshooting": 0.7,      # Useful
    "Advanced Topics": 0.5,      # Nice to have
    "Optional": 0.2,             # Lowest priority
}
```

---

## 9. Tiered File Strategy

### 9.1 Single File Strategy (llms.txt only)

**Use when:**
- Project is small/focused
- <100 pages of documentation
- <5 major feature areas
- Consistent documentation depth

**Structure:**
```
llms.txt (3K-5K tokens)
├── Getting Started
├── Core Concepts
├── [Feature 1-3 sections]
├── Troubleshooting
└── Optional
```

**Example Projects:**
- CLI tools
- Small libraries
- Feature documentation
- API specs (<50 endpoints)

### 9.2 Dual File Strategy (llms.txt + llms-full.txt)

**Use when:**
- Project has 100-500 pages
- 5-20 major feature areas
- Mixed audience (quick start + reference)
- Need context window optimization

**Distribution:**
```
llms.txt (3K-5K tokens) - "Quick Start"
├── Getting Started
├── Core Concepts
├── Essential Features
└── Link to llms-full.txt

llms-full.txt (12K-40K tokens) - "Master Index"
├── Getting Started (detailed)
├── Architecture (complete)
├── [All features]
├── [All advanced topics]
└── [Optional content]
```

**Consumption Pattern:**
```
LLM Decision Tree:
- Has context space for < 5K tokens?
  → Use llms.txt (fast, focused)
- Has context space for > 5K tokens?
  → Use llms-full.txt (comprehensive)
- Has context for 5K-10K tokens?
  → Use llms.txt + selective sections from llms-full.txt
```

### 9.3 Multi-File Strategy (llms.txt + service files)

**Use when:**
- Monorepo with multiple services
- Each service needs its own docs
- Central coordination needed

**Structure:**
```
llms.txt (Master Index - 1K tokens)
├── Service A: [description] → ./services/llms-a.txt
├── Service B: [description] → ./services/llms-b.txt
├── Service C: [description] → ./services/llms-c.txt
└── Cross-service: [shared concepts]

./services/llms-a.txt (3K tokens per service)
./services/llms-b.txt
./services/llms-c.txt
```

**Coordination Rules:**
- Master llms.txt is always accessible
- Each service file is self-contained
- Avoid circular references
- Document dependencies clearly

### 9.4 Gold Standard Evidence for Tiered Strategy

The tiered approach is validated by the highest-scoring implementations in v0.0.2c:

**Svelte (5/5 — Primary Gold Standard):**
- Implements three tiers: small (~500KB), medium (~1.5MB), full (~3MB+)
- Includes a dedicated `/docs/llms` guidance page explaining which tier to use for different contexts
- Explicitly warns about RAG quality degradation with larger files
- Naming convention: variant-based (small/medium/full)

**Anthropic (4/5):**
- Implements dual-tier: llms.txt (~8K tokens) + llms-full.txt (~481K tokens)
- Co-developed the llms-full.txt convention with Mintlify (November 2024)
- The compact llms.txt serves as a quick-reference index; the full file provides comprehensive documentation

**Pydantic (5/5):**
- Dual-file strategy with semantic tiering: llms.txt answers "What is Pydantic?" while llms-full.txt answers "How do I use Pydantic?"
- Clearest expression of semantic (not size-based) tier differentiation in the audit

**Cloudflare (4/5 — Cautionary Example):**
- llms-full.txt reaches 3.7M tokens — exceeds all current context windows
- Effective strategy is the per-product decomposition (Workers, R2, D1), not the monolithic file
- Demonstrates why the anti-pattern threshold of 100K tokens per file exists

**Correlation data (v0.0.2c):** Tiered implementations average 4.5/5 quality vs. 3.9/5 for single-file implementations. The correlation is weak-moderate (r ≈ 0.35) because quality depends more on content organization than file count — but tiering enables better token budget management, which is foundational for MCP-served consumption.

### 9.5 Strategy Decision Matrix

| Factor | Single File | Dual File | Multi-File |
|--------|------------|-----------|-----------|
| **Docs Scale** | <100 pages | 100-500 | 500+ |
| **Features** | <5 | 5-20 | 20+ |
| **Token Range** | 3K-5K | 3K-5K (index) + 12K-50K (full) | 1K (master) + 3K-5K per service |
| **Maintenance** | Easy | Medium | Complex |
| **LLM Performance** | Fast | Balanced | Requires aggregation |
| **Discovery** | Excellent | Good | Requires navigation |
| **Gold Standard** | — | Pydantic, Anthropic | Cloudflare (per-product) |
| **Examples** | Small CLI tools, libraries | SaaS APIs, frameworks | Microservices, monorepos |

---

## 10. Structural Compliance Checklist

### 10.1 Automated Validation Checklist

Use this checklist for automated v0.2.4 validation pipeline:

```yaml
STRUCTURAL_COMPLIANCE:
  encoding:
    - id: ENC-001
      check: "File has UTF-8 encoding"
      test: "file_encoding == 'UTF-8' and not has_bom"
      severity: CRITICAL
      pass_fail: true

    - id: ENC-002
      check: "Line endings are LF only"
      test: "'\r\n' not in content and '\r' not in content"
      severity: CRITICAL
      pass_fail: true

  structure:
    - id: STR-001
      check: "H1 title exists"
      test: "exactly_one_h1_at_start"
      severity: CRITICAL
      pass_fail: true

    - id: STR-002
      check: "Blockquote follows H1"
      test: "first_non_whitespace_after_h1 is blockquote"
      severity: CRITICAL
      pass_fail: true

    - id: STR-003
      check: "Master Index in first section"
      test: "first_h2 contains 'Master Index' or 'Quick Navigation'"
      severity: CRITICAL
      pass_fail: true

    - id: STR-004
      check: "Sections in canonical order"
      test: "section_order_follows_canonical"
      severity: HIGH
      pass_fail: true

    - id: STR-005
      check: "No duplicate section headings"
      test: "len(unique_h2_titles) == len(all_h2_titles)"
      severity: HIGH
      pass_fail: true

  markdown:
    - id: MD-001
      check: "Valid Markdown syntax"
      test: "parses_as_valid_gfm"
      severity: CRITICAL
      pass_fail: true

    - id: MD-002
      check: "No setext-style headings"
      test: "not contains_setext_headings"
      severity: MEDIUM
      pass_fail: true

    - id: MD-003
      check: "Code blocks are fenced"
      test: "all_code_blocks_are_fenced"
      severity: MEDIUM
      pass_fail: true

  links:
    - id: LNK-001
      check: "All link entries have valid syntax"
      test: "all_file_entries_match '- [title](url)' pattern"
      severity: HIGH
      pass_fail: true

    - id: LNK-002
      check: "No empty URLs in link entries"
      test: "no_link_entry_has_empty_url"
      severity: HIGH
      pass_fail: true

    - id: LNK-003
      check: "URLs use absolute paths (not relative)"
      test: "all_link_urls_are_absolute or documented_relative_rationale"
      severity: MEDIUM
      pass_fail: false

  naming:
    - id: NAM-001
      check: "Section names match canonical list"
      test: "all_section_names_in_canonical_list or documented_custom_names"
      severity: MEDIUM
      pass_fail: false

    - id: NAM-002
      check: "No deprecated section names used"
      test: "no_section_name_in_deprecated_list"
      severity: LOW
      pass_fail: false

  hierarchy:
    - id: HIR-001
      check: "No H3+ headers used outside H2 sections"
      test: "all_h3_plus_nested_within_h2_sections"
      severity: MEDIUM
      pass_fail: true

    - id: HIR-002
      check: "No empty H2 sections (header with no content)"
      test: "all_h2_sections_have_content"
      severity: MEDIUM
      pass_fail: true

  size:
    - id: SIZ-001
      check: "File within size guidelines"
      test: "file_tokens between 500 and 50000"
      severity: MEDIUM
      pass_fail: false

    - id: SIZ-002
      check: "Sections within token budgets"
      test: "all_sections_within_allocated_budget"
      severity: LOW
      pass_fail: false

    - id: SIZ-003
      check: "File does not exceed anti-pattern threshold"
      test: "file_tokens < 100000"
      severity: HIGH
      pass_fail: true
```

### 10.2 Manual Compliance Review

**Checklist for human review:**

- [ ] File location is `/llms.txt` (or documented fallback with rationale)
- [ ] File opens with H1 title (STR-001)
- [ ] Blockquote immediately follows H1 (STR-002)
- [ ] Master Index is first H2 section (STR-003)
- [ ] Sections follow canonical ordering: Getting Started → Core Concepts → Domain Sections → Advanced → FAQ → Optional (STR-004)
- [ ] Section names match the v0.0.2c canonical list or have documented rationale for custom names (NAM-001)
- [ ] No deprecated section names ("Introduction," "Details," "Misc") are used (NAM-002)
- [ ] Optional sections are prefixed with "Optional:" and include token count guidance
- [ ] File size is within the appropriate tier: Standard (1.5K–4.5K), Comprehensive (4.5K–12K), or Full (12K–50K tokens)
- [ ] No file exceeds 100K tokens (anti-pattern threshold from v0.0.2d)
- [ ] Token budget is allocated sensibly per the section templates in §5.2–5.2c
- [ ] No orphaned sections or navigation dead ends in the Master Index
- [ ] All link entries follow `- [title](url): description` syntax (LNK-001)
- [ ] All URLs are absolute, not relative (LNK-003)
- [ ] No empty H2 sections — every section has content (HIR-002)
- [ ] Code examples use fenced blocks with language specifiers (MD-003)
- [ ] Lists use consistent markers (- or *) without mixing within a sequence
- [ ] No control characters or invalid UTF-8 (ENC-001)
- [ ] Line endings are Unix (LF) only (ENC-002)
- [ ] Cross-cutting concerns are elevated as first-class sections (per Pydantic gold standard)

---

## 11. Code Examples: Correct vs Incorrect Structure

### 11.1 Incorrect Structure Example

```markdown
Introduction

Here's my documentation about the project.

# My Project Documentation

This document describes [project].

## Getting Started
[content]

## API Reference
[content]

Getting Started (Advanced)

[more content - WRONG: duplicate section]

## Architecture
[content]
```

**Issues Found:**
- Missing H1 at start (CRITICAL)
- No blockquote (CRITICAL)
- No Master Index (CRITICAL)
- Wrong section order: Getting Started before title
- Duplicate section name

### 11.2 Correct Structure Example

```markdown
# My Project Documentation

> A comprehensive guide to building applications with [Project].

## 1. Master Index

**Quick Navigation:**
- [Getting Started](#getting-started)
- [Architecture](#architecture)
- [API Reference](#api-reference)
- [FAQ](#faq)

## Getting Started

### Installation

[installation steps]

## Architecture

### System Design

[architecture details]

## API Reference

### Endpoints

[endpoint documentation]

## FAQ

[frequently asked questions]

## Optional: Historical Background

> This section provides context but is not required for understanding [Project].

[historical information]
```

**Passes All Checks:**
- ✓ H1 at start
- ✓ Blockquote follows H1
- ✓ Master Index is first section
- ✓ Sections in canonical order
- ✓ Clear hierarchy

### 11.3 Line Ending Example

**Incorrect (CRLF - Windows style):**
```
# Title\r\n
\r\n
> Description\r\n
```

**Correct (LF - Unix style):**
```
# Title\n
\n
> Description\n
```

**Validation:**
```bash
# Check line endings in file
file llms.txt
# Output should say "LF line terminators" not "CRLF"

# Convert CRLF to LF if needed
dos2unix llms.txt
# or
sed -i 's/\r$//' llms.txt
```

---

## 12. Deliverables Checklist

- [x] Comprehensive section ordering rules documented with rationale (§6 — strict 10-step ordering with ABNF grammar and v0.0.2c evidence)
- [x] Canonical section naming list derived from v0.0.2c frequency analysis (§7 — 11 canonical sections from 18-site audit with frequency data)
- [x] File location discovery protocol defined (§3 — 5-step priority discovery with fallback table)
- [x] Token budget allocation model created (§5.2/5.2b/5.2c — templates for 3K, 12K, and 40K files with per-section allocations)
- [x] Format validation script (pseudocode) provided (§4.3 — encoding, line ending, and control character validation)
- [x] Structural compliance checklist with 15+ automated checks (§10.1 — 20 automated checks across 7 categories: encoding, structure, markdown, links, naming, hierarchy, size)
- [x] "Optional" section usage guide with examples (§8 — purpose, content guidance, format template, and token priority weighting)
- [x] Tiered file strategy matrix (§9.1–9.5 — single, dual, multi-file strategies with gold standard evidence and decision matrix)
- [x] Before/after code examples showing correct vs incorrect structure (§11 — incorrect/correct structure, line ending examples, and validation commands)
- [x] Automated test suite ready for v0.2.4 implementation (§10.1 — 20 YAML-formatted checks with IDs, severity levels, and test expressions)

---

## 13. Acceptance Criteria

| Criteria | Measurement | Pass/Fail | Evidence |
|----------|------------|-----------|----------|
| **Completeness** | All 14 main sections covered with examples | PASS | §3–§11 fully populated |
| **Specificity** | Rules are actionable (not just recommendations) | PASS | YAML test expressions in §10, pseudocode in §4.3 |
| **Testability** | Validation checklist has 15+ automated checks | PASS | 20 checks across 7 categories (§10.1) |
| **Clarity** | All rules explained with before/after examples | PASS | §11.1 (incorrect) vs §11.2 (correct), §11.3 (line endings) |
| **Consistency** | Terminology matches v0.0.2c and v0.0.1a | PASS | All frequency data verified against 18-site v0.0.2c audit; ABNF references verified against v0.0.1a |
| **Practicality** | Rules apply to 90%+ of llms.txt use cases | PASS | Three tiers (§5.2–5.2c) and three strategies (§9.1–9.3) cover all project scales |
| **Reference-ability** | Each rule has unique ID (ENC-001, STR-001, etc.) | PASS | 20 IDs: ENC (2), STR (5), MD (3), LNK (3), NAM (2), HIR (2), SIZ (3) |
| **Scope Integrity** | No content duplication with v0.0.4b/4c/4d | PASS | Content quality → 4b, anti-patterns → 4c, decisions → 4d |

---

## 14. Next Steps

This document feeds directly into:

1. **v0.1.0: Implementation** - Use these rules to build llms.txt generator
2. **v0.2.4: Validation Pipeline** - Implement automated checks from Section 10
3. **v0.2.5: Linter Tool** - Create standalone linter enforcing Section 4-7 rules
4. **v0.3.0: LLM Consumption Testing** - Verify token budgets from Section 5 work in practice

**Immediate Next Action:** Create companion checklist template (Markdown) for project teams to self-validate their llms.txt files.

---

**Document End**
Reference: v0.0.4a | Status: COMPLETE | Phase: Best Practices Synthesis
Verified: 2026-02-06
