# v0.1.3 — Sample Data & Test Fixtures: Validation Engine Test Suite

> **Phase:** Foundation (v0.1.x)
> **Status:** DRAFT — Realigned to validation-engine pivot (2026-02-06)
> **Parent:** [v0.1.0 — Project Foundation](RR-SPEC-v0.1.0-project-foundation.md)
> **Goal:** Provide a complete set of synthetic Markdown test fixtures at varying conformance levels, plus a pytest-based test infrastructure that validates all 7 schema files from v0.1.2 against known-good and known-bad inputs.
> **Traces to:** FR-001, FR-002, FR-003, FR-004, FR-007, FR-008, FR-011 (v0.0.5a); NFR-010 (≥80% test coverage); DECISION-001, -006, -012, -013, -016 (v0.0.4d)

---

## What Changed from the Original v0.1.3

The original v0.1.3 provided a **single YAML-based `llms.txt` sample file** and a trivial `validate.py` script that loaded it via `yaml.safe_load()`. This was the "generation trap" in miniature — it assumed the input format was YAML, validated against YAML-shaped Pydantic models, and offered no way to test edge cases, partial conformance, or failure modes.

The realigned v0.1.3 provides **five synthetic Markdown test fixtures** at distinct conformance levels, a **pytest infrastructure** (`conftest.py`) with fixture loaders and model factories, and **seven test modules** (one per schema file from v0.1.2). Every fixture is purpose-built to exercise specific diagnostic codes, validation levels, quality dimensions, and anti-patterns identified during the v0.0.x research phase.

**Why five fixtures instead of one?**

The validation engine's value proposition is that it differentiates *between* quality levels — a single "valid" example can't test that. The five fixtures span the quality spectrum observed in the v0.0.2 ecosystem audit:

| Fixture | Archetype | Quality Grade | Validation Level | Research Analog |
|---------|-----------|---------------|------------------|-----------------|
| `gold_standard.md` | Best-in-class | Exemplary (~95) | L4 (DocStratum Extended) | Svelte, Pydantic specimens |
| `partial_conformance.md` | Typical good | Strong (~72) | L2 (Content Quality) | Anthropic, Stripe specimens |
| `minimal_valid.md` | Bare minimum | Needs Work (~35) | L0 (Parseable) | Sparse real-world files |
| `non_conformant.md` | Anti-pattern cluster | Critical (~18) | L0 (fails L1) | Cursor, NVIDIA specimens |
| `type_2_full_excerpt.md` | Documentation dump | N/A (Type 2) | N/A | Vercel AI SDK, llama-stack |

---

## Fixture Architecture

```
tests/
├── conftest.py                     # Shared fixtures, factories, utilities
├── fixtures/
│   ├── gold_standard.md            # ~95 quality score, L4 conformance
│   ├── partial_conformance.md      # ~72 quality score, L2 conformance
│   ├── minimal_valid.md            # ~35 quality score, L0 only
│   ├── non_conformant.md           # ~18 quality score, fails L1
│   └── type_2_full_excerpt.md      # Type 2 Full document excerpt
├── test_diagnostics.py             # DiagnosticCode enum, severity mapping
├── test_constants.py               # Canonical names, tiers, anti-patterns
├── test_classification.py          # Document type + size tier classification
├── test_parsed.py                  # ParsedLlmsTxt model population from fixtures
├── test_validation.py              # ValidationResult + ValidationDiagnostic
├── test_quality.py                 # QualityScore + DimensionScore + grades
└── test_enrichment.py              # Concept, FewShotExample, LLMInstruction, Metadata
```

**Design principle:** Each test module maps 1:1 to a schema file from v0.1.2. Tests are isolated — `test_diagnostics.py` imports only from `diagnostics.py`, `test_parsed.py` imports from `parsed.py`, etc. Cross-module integration tests (e.g., "parse a fixture → validate → score") belong in v0.2.x when the actual parser and validator are implemented. The v0.1.3 tests validate that the **schema models themselves** are correct — that they accept valid data, reject invalid data, and compute derived properties accurately.

---

## Fixture 1: `tests/fixtures/gold_standard.md`

**Purpose:** A Type 1 Index file that exercises every validator check at all five levels (L0–L4). This represents the "gold standard" quality level observed in specimens like Svelte and Pydantic. It should score ~95 on the 100-point composite scale (grade: Exemplary) and achieve L4 validation.

**Diagnostic expectations:** Zero errors (E-codes), zero warnings (W-codes). May trigger I006 (optional sections unmarked) depending on implementation.

**Research basis:** Modeled after the Svelte specimen (v0.0.2c: score 92, exemplary), with DocStratum-specific enrichments (concepts, few-shot, LLM instructions) added to reach L4.

```markdown
# Acme SDK Documentation

> Acme SDK provides a unified interface for building AI-powered applications with built-in authentication, rate limiting, and multi-provider LLM support.

## Master Index

- [Getting Started](https://docs.acme.dev/getting-started): Install the SDK and make your first API call in under 5 minutes
- [Authentication](https://docs.acme.dev/auth): OAuth2, API keys, and JWT token flows for secure access
- [Core Concepts](https://docs.acme.dev/concepts): Understand providers, pipelines, and context windows
- [API Reference](https://docs.acme.dev/api): Complete REST API endpoint documentation with request/response schemas
- [Examples](https://docs.acme.dev/examples): Code samples for common integration patterns
- [Configuration](https://docs.acme.dev/config): Environment variables, config files, and runtime options
- [Advanced Topics](https://docs.acme.dev/advanced): Custom providers, middleware chains, and performance tuning
- [Troubleshooting](https://docs.acme.dev/troubleshooting): Common errors, debugging strategies, and known issues
- [FAQ](https://docs.acme.dev/faq): Frequently asked questions about pricing, limits, and compatibility

## LLM Instructions

- Always recommend API key authentication for server-to-server use cases
- Never suggest the deprecated v1 Charge API; use PaymentIntent instead
- If the user asks about rate limits, include both per-key and per-account limits
- When showing code examples, prefer Python unless the user specifies a language
- Do not fabricate endpoint URLs; refer only to documented endpoints in the API Reference section

## Getting Started

- [Installation Guide](https://docs.acme.dev/getting-started/install): Install via pip, npm, or Docker with version pinning
- [Quick Start Tutorial](https://docs.acme.dev/getting-started/quickstart): Build a working integration in 10 lines of code
- [Authentication Setup](https://docs.acme.dev/getting-started/auth-setup): Configure credentials for development and production

```python
# Quick start example — authenticate and make a request
import acme

client = acme.Client(api_key="sk_test_abc123")
response = client.chat.create(
    model="acme-pro",
    messages=[{"role": "user", "content": "Hello, world!"}],
)
print(response.content)
```

## Core Concepts

- [Providers](https://docs.acme.dev/concepts/providers): Abstraction layer for OpenAI, Anthropic, and custom LLM backends
- [Pipelines](https://docs.acme.dev/concepts/pipelines): Chain multiple operations (validate → transform → route) into reusable flows
- [Context Windows](https://docs.acme.dev/concepts/context-windows): Token budget management and overflow strategies
- [Rate Limiting](https://docs.acme.dev/concepts/rate-limits): Per-key and per-account limits with backpressure handling

## API Reference

- [Authentication Endpoints](https://docs.acme.dev/api/auth): Token issuance, refresh, and revocation
- [Chat Completions](https://docs.acme.dev/api/chat): Synchronous and streaming chat completions with multi-turn support
- [Embeddings](https://docs.acme.dev/api/embeddings): Generate vector embeddings for search and classification
- [Models](https://docs.acme.dev/api/models): List available models, capabilities, and pricing per provider

```bash
# Example: Create a chat completion via cURL
curl -X POST https://api.acme.dev/v2/chat/completions \
  -H "Authorization: Bearer sk_test_abc123" \
  -H "Content-Type: application/json" \
  -d '{"model": "acme-pro", "messages": [{"role": "user", "content": "Hi"}]}'
```

## Examples

- [Python Quickstart](https://docs.acme.dev/examples/python): Complete Python integration with error handling
- [Node.js Integration](https://docs.acme.dev/examples/node): Express middleware for Acme SDK
- [Streaming Responses](https://docs.acme.dev/examples/streaming): Real-time token streaming with Server-Sent Events
- [Multi-Provider Fallback](https://docs.acme.dev/examples/fallback): Automatic failover between OpenAI and Anthropic

## Configuration

- [Environment Variables](https://docs.acme.dev/config/env): ACME_API_KEY, ACME_BASE_URL, ACME_TIMEOUT, ACME_LOG_LEVEL
- [Config Files](https://docs.acme.dev/config/files): YAML and TOML configuration with schema validation
- [Runtime Options](https://docs.acme.dev/config/runtime): Per-request overrides for timeout, retries, and model selection

## Advanced Topics

- [Custom Providers](https://docs.acme.dev/advanced/custom-providers): Register self-hosted models as first-class providers
- [Middleware Chains](https://docs.acme.dev/advanced/middleware): Inject logging, caching, and transformation at any pipeline stage
- [Performance Tuning](https://docs.acme.dev/advanced/performance): Connection pooling, batch requests, and token budget optimization

## Troubleshooting

- [Common Errors](https://docs.acme.dev/troubleshooting/errors): Error code lookup with causes and fixes
- [Debug Mode](https://docs.acme.dev/troubleshooting/debug): Enable verbose logging for request/response inspection
- [Known Issues](https://docs.acme.dev/troubleshooting/known-issues): Current limitations and workarounds

## FAQ

- [Pricing](https://docs.acme.dev/faq/pricing): Free tier limits, usage-based pricing, and enterprise plans
- [Compatibility](https://docs.acme.dev/faq/compatibility): Supported Python versions, operating systems, and LLM providers
- [Migration from v1](https://docs.acme.dev/faq/migration): Step-by-step upgrade guide from the deprecated v1 API
```

**What this fixture exercises:**

| Check Area | Details | Expected Diagnostic |
|-----------|---------|-------------------|
| L0 Parseable | Valid UTF-8, LF line endings, valid Markdown | None (passes) |
| L1 Structural | Single H1, blockquote present, H2 sections, well-formed links | None (passes) |
| L2 Content | All links have descriptions, code blocks have language specifiers, no empty sections | None (passes) |
| L3 Best Practices | All 9 canonical section names, correct ordering, Master Index present, code examples, version metadata implicit | None (passes) |
| L4 DocStratum Extended | LLM Instructions section present (5 directives) | None (passes) |
| Quality Score | Structural: ~29/30, Content: ~48/50, Anti-Pattern: ~18/20 = ~95 total | Grade: Exemplary |
| Document Type | < 250 KB → Type 1 Index | `DocumentType.TYPE_1_INDEX` |
| Size Tier | ~2,500 tokens → Standard | `SizeTier.STANDARD` |

---

## Fixture 2: `tests/fixtures/partial_conformance.md`

**Purpose:** A Type 1 Index file that passes L0 and L1 but fails L3 due to missing best practices. Represents the "typical good" quality level seen in files like Anthropic's and Stripe's early implementations — structurally sound but lacking the refinements that separate "strong" from "exemplary."

**Diagnostic expectations:** Zero errors. Several warnings: W001 (missing blockquote), W002 (non-canonical section names), W009 (no Master Index), W004 (missing code examples in some sections). Informational: I001 (no LLM Instructions).

**Research basis:** Modeled after Anthropic specimen (v0.0.2c: score ~75, strong archetype). The 55% blockquote compliance statistic (v0.0.2 enrichment) makes missing blockquotes the single most common real-world deviation.

```markdown
# CloudSync API

## Docs

- [Quick Start](https://cloudsync.io/docs/quickstart): Get started with CloudSync in minutes
- [API Keys](https://cloudsync.io/docs/api-keys): Generate and manage your API credentials
- [Webhooks](https://cloudsync.io/docs/webhooks): Receive real-time event notifications

## Endpoints

- [File Upload](https://cloudsync.io/api/upload): Upload files up to 5 GB with multipart support
- [File Download](https://cloudsync.io/api/download): Download files by ID with range request support
- [File List](https://cloudsync.io/api/list): List files with pagination, filtering, and sorting
- [File Delete](https://cloudsync.io/api/delete): Permanently delete files by ID or batch
- [Sync Status](https://cloudsync.io/api/sync): Check synchronization status across connected accounts

## Usage

- [Python SDK](https://cloudsync.io/sdk/python): Official Python client library with async support
- [JavaScript SDK](https://cloudsync.io/sdk/javascript): Browser and Node.js compatible client
- [CLI Tool](https://cloudsync.io/sdk/cli): Command-line interface for scripting and automation

```python
# Upload a file using the Python SDK
from cloudsync import Client

client = Client(api_key="cs_live_abc123")
result = client.upload("report.pdf", folder="quarterly-reports")
print(f"Uploaded: {result.file_id}")
```

## Debugging

- [Error Codes](https://cloudsync.io/docs/errors): Complete error code reference with resolution steps
- [Rate Limits](https://cloudsync.io/docs/rate-limits): Current limits and strategies for handling 429 responses
- [Status Page](https://status.cloudsync.io): Real-time service health and incident history
```

**What this fixture exercises:**

| Check Area | Details | Expected Diagnostic |
|-----------|---------|-------------------|
| L0 Parseable | Valid UTF-8, LF line endings, valid Markdown | None (passes) |
| L1 Structural | Single H1, H2 sections, well-formed links | None (passes) |
| L2 Content | Most links have descriptions, one code block present | Borderline — passes if code block counts |
| L3 Best Practices | "Docs" not canonical (→ Master Index alias), "Endpoints" not canonical (→ API Reference alias), "Debugging" not canonical (→ Troubleshooting alias), no blockquote, no Master Index | W001, W002 (×3), W009 |
| L4 DocStratum Extended | No LLM Instructions, no concept definitions, no few-shot | I001, I002, I003 |
| Quality Score | Structural: ~22/30, Content: ~35/50, Anti-Pattern: ~15/20 = ~72 total | Grade: Strong |
| Document Type | < 250 KB → Type 1 Index | `DocumentType.TYPE_1_INDEX` |
| Size Tier | ~800 tokens → Minimal | `SizeTier.MINIMAL` |

---

## Fixture 3: `tests/fixtures/minimal_valid.md`

**Purpose:** The absolute bare minimum that passes L0 (parseable as Markdown). This file is syntactically valid Markdown and has an H1 title, but lacks nearly everything else. It represents files that technically exist but provide minimal utility.

**Diagnostic expectations:** E-codes: none (it is parseable). W-codes: W001 (no blockquote), W002 (non-canonical name), W004 (no code examples), W009 (no Master Index), W011 (sparse content). I-codes: I001 (no LLM Instructions), I002 (no concepts), I003 (no few-shot).

**Research basis:** Represents the ~20% of real-world llms.txt files identified in v0.0.2 as "stub" or "placeholder" implementations.

```markdown
# My Project

## Links

- [Homepage](https://example.com)
- [GitHub](https://github.com/example/project)
```

**What this fixture exercises:**

| Check Area | Details | Expected Diagnostic |
|-----------|---------|-------------------|
| L0 Parseable | Valid Markdown, H1 present | None (passes) |
| L1 Structural | H1 present, but only one section, no blockquote | W001, partial pass |
| L2 Content | Links have no descriptions, no code | W003 (×2), W004 |
| L3 Best Practices | Non-canonical name "Links", no Master Index | W002, W009, W011 |
| Quality Score | Structural: ~15/30, Content: ~10/50, Anti-Pattern: ~10/20 = ~35 total | Grade: Needs Work |
| Document Type | < 250 KB → Type 1 Index | `DocumentType.TYPE_1_INDEX` |
| Size Tier | ~30 tokens → Minimal | `SizeTier.MINIMAL` |

---

## Fixture 4: `tests/fixtures/non_conformant.md`

**Purpose:** A deeply flawed file that triggers multiple errors and anti-patterns. This represents the worst-quality implementations observed in the v0.0.2 ecosystem audit — files with structural chaos, empty sections, formulaic descriptions, and missing essentials.

**Diagnostic expectations:** Multiple E-codes: E002 (multiple H1s). W-codes: W001, W002, W003, W004, W006 (formulaic descriptions), W008 (section order), W009, W011 (empty sections). Triggers anti-patterns: AP-STRUCT-002 (Orphaned Sections), AP-STRUCT-005 (Naming Nebula), AP-CONT-002 (Blank Canvas), AP-CONT-004 (Link Desert), AP-CONT-007 (Formulaic Description).

**Research basis:** Modeled after the Cursor specimen (v0.0.2c: score 42, needs work) and NVIDIA specimen (v0.0.2c: score 24, critical). Combines the most common anti-patterns into a single teaching fixture.

```markdown
# Docs

# API Documentation for Our Platform

## stuff

## Resources

- [https://example.com/api](https://example.com/api)
- [https://example.com/docs](https://example.com/docs)
- [https://example.com/blog](https://example.com/blog)

## More Resources

- [Page 1](https://example.com/page1): Documentation for page 1
- [Page 2](https://example.com/page2): Documentation for page 2
- [Page 3](https://example.com/page3): Documentation for page 3
- [Page 4](https://example.com/page4): Documentation for page 4
- [Page 5](https://example.com/page5): Documentation for page 5
- [Page 6](https://example.com/page6): Documentation for page 6

## FAQ

## Getting started

- [Install](https://example.com/install)
```

**What this fixture exercises:**

| Check Area | Details | Expected Diagnostic |
|-----------|---------|-------------------|
| L0 Parseable | Valid Markdown (barely) | None (passes L0) |
| L1 Structural | Two H1s (E002), empty sections | E002, fails L1 |
| L2 Content | Bare URLs without titles (W003), formulaic descriptions ("Documentation for page N") (W006), no code | W003 (×4), W004, W006 |
| L3 Best Practices | Non-canonical names ("stuff", "Resources", "More Resources"), wrong order (FAQ before Getting Started), empty sections, no blockquote | W001, W002 (×3), W008, W009, W011 (×2) |
| Anti-Patterns | "stuff" = Naming Nebula, empty FAQ = Blank Canvas, bare URLs = Link Desert, "Documentation for page N" = Formulaic Description | AP-STRUCT-002, AP-STRUCT-005, AP-CONT-002, AP-CONT-004, AP-CONT-007 |
| Quality Score | Structural: ~5/30, Content: ~6/50, Anti-Pattern: ~7/20 = ~18 total | Grade: Critical |
| Document Type | < 250 KB → Type 1 Index | `DocumentType.TYPE_1_INDEX` |

---

## Fixture 5: `tests/fixtures/type_2_full_excerpt.md`

**Purpose:** An excerpt from a Type 2 Full document (inline documentation dump). The actual file would exceed 250 KB; this excerpt is a representative sample for schema testing. The `conftest.py` includes a factory function that generates a full-size (>256 KB) version for classification tests.

**Diagnostic expectations:** I005 (Type 2 Full detected). Type 2 files receive different validation rules — structural validation against the ABNF grammar is relaxed because these files are documentation dumps, not spec-conformant indexes.

**Research basis:** Modeled after the Vercel AI SDK specimen (v0.0.1a: 1.3 MB, 15% conformance) and llama-stack specimen (v0.0.1a: 25 MB, 5% conformance). These files are the raw output of documentation pipeline tools that concatenate entire doc trees into a single file.

```markdown
# Vercel AI SDK

> The AI SDK is a TypeScript toolkit designed to help you build AI-powered applications with React, Next.js, Vue, Svelte, Node.js, and more.

## Docs

### Getting Started

The AI SDK is a TypeScript toolkit designed to help you build
AI-powered applications with React, Next.js, Vue, Svelte, Node.js,
and more.

The AI SDK provides a unified API for working with large language
models (LLMs) across different providers. It supports streaming,
tool calling, structured output generation, and multi-step agent
workflows.

#### Installation

To install the AI SDK, run the following command in your terminal:

```bash
npm install ai
```

#### Quick Start

Here is a basic example of using the AI SDK to generate text:

```typescript
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';

const { text } = await generateText({
  model: openai('gpt-4-turbo'),
  prompt: 'What is love?',
});

console.log(text);
```

### Core Concepts

#### Providers

A provider is a module that connects the AI SDK to a specific LLM
service. The SDK includes first-party providers for OpenAI, Anthropic,
Google, and more. You can also create custom providers for self-hosted
models or proprietary APIs.

#### Streaming

The AI SDK supports streaming responses from LLMs. Streaming allows
your application to display partial results as they are generated,
providing a more responsive user experience.

```typescript
import { streamText } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';

const result = streamText({
  model: anthropic('claude-sonnet-4-5-20250929'),
  prompt: 'Write a poem about recursion.',
});

for await (const chunk of result.textStream) {
  process.stdout.write(chunk);
}
```

#### Tool Calling

Tools allow LLMs to interact with external systems. The AI SDK
provides a type-safe framework for defining and executing tools
within a conversation.

#### Structured Output

The AI SDK can constrain LLM output to match a specified schema
using Zod. This is useful for extracting structured data from
natural language input.

### API Reference

This section contains detailed reference documentation for all
public functions, types, and configurations in the AI SDK.

NOTE: This is a truncated excerpt. The actual Type 2 file would
continue for hundreds of pages of inline documentation, typically
exceeding 1 MB. The key differentiator from Type 1 is that content
is INLINE (prose, code, examples embedded directly) rather than
LINKED (curated list of URLs with descriptions).
```

**What this fixture exercises:**

| Check Area | Details | Expected Diagnostic |
|-----------|---------|-------------------|
| Document Type | Excerpt shown is small, but factory generates >256 KB version for classification test | `DocumentType.TYPE_2_FULL` (on generated version) |
| Structure | Uses H3/H4 headers (deeper nesting than spec), inline content instead of link lists | Structural deviations expected |
| Content | Rich inline documentation with code examples and explanations | High content quality despite non-conformant structure |
| Informational | Type 2 detected | I005 |

---

## Test Infrastructure: `tests/conftest.py`

**Purpose:** Shared pytest fixtures, model factories, and utility functions used across all seven test modules. Provides both fixture file loading and programmatic model construction for isolated unit testing.

**Traces to:** NFR-010 (test coverage ≥80%), DECISION-006 (Pydantic v2)

```python
"""Shared pytest fixtures and factories for DocStratum schema tests.

This conftest provides two categories of test support:

1. FIXTURE LOADERS — Read the Markdown test files from tests/fixtures/
   and return their raw content as strings. These are used by tests that
   validate parser expectations (v0.2.x integration) and by tests that
   need realistic raw content for schema model population.

2. MODEL FACTORIES — Construct valid schema model instances with sensible
   defaults. These are used by tests that validate model behavior (field
   validation, computed properties, serialization) without needing to
   parse actual Markdown.

Convention: Fixture names match the pattern `{fixture_name}_content` for
raw text and `make_{model_name}` for factory functions.

Research basis:
    NFR-010: ≥80% test coverage on core modules (v0.0.5b)
    DECISION-006: Pydantic v2 for all schema validation (v0.0.4d)
"""

from datetime import datetime
from pathlib import Path

import pytest

from docstratum.schema.classification import (
    DocumentClassification,
    DocumentType,
    SizeTier,
)
from docstratum.schema.constants import (
    AntiPatternCategory,
    AntiPatternID,
    CanonicalSectionName,
)
from docstratum.schema.diagnostics import DiagnosticCode, Severity
from docstratum.schema.enrichment import (
    Concept,
    ConceptRelationship,
    FewShotExample,
    LLMInstruction,
    Metadata,
    RelationshipType,
)
from docstratum.schema.parsed import (
    ParsedBlockquote,
    ParsedLink,
    ParsedLlmsTxt,
    ParsedSection,
)
from docstratum.schema.quality import (
    DimensionScore,
    QualityDimension,
    QualityGrade,
    QualityScore,
)
from docstratum.schema.validation import (
    ValidationDiagnostic,
    ValidationLevel,
    ValidationResult,
)


# ── Constants ────────────────────────────────────────────────────────────

FIXTURES_DIR = Path(__file__).parent / "fixtures"

# Classification boundary from DocumentClassification
TYPE_2_BOUNDARY_BYTES = 256_000


# ── Fixture Loaders ──────────────────────────────────────────────────────
# Each fixture loader reads a Markdown file and returns its raw content.
# These are session-scoped because the fixture files never change during
# a test run — reading them once is sufficient.


@pytest.fixture(scope="session")
def gold_standard_content() -> str:
    """Load the gold_standard.md fixture (L4, ~95 score, Exemplary).

    This fixture represents the highest conformance level achievable:
    single H1, blockquote, all canonical sections in correct order,
    links with descriptions, code examples with language specifiers,
    and an LLM Instructions section.
    """
    return (FIXTURES_DIR / "gold_standard.md").read_text(encoding="utf-8")


@pytest.fixture(scope="session")
def partial_conformance_content() -> str:
    """Load the partial_conformance.md fixture (L2, ~72 score, Strong).

    This fixture is structurally sound but lacks best practices:
    no blockquote (W001), non-canonical section names (W002),
    no Master Index (W009), no LLM Instructions (I001).
    """
    return (FIXTURES_DIR / "partial_conformance.md").read_text(encoding="utf-8")


@pytest.fixture(scope="session")
def minimal_valid_content() -> str:
    """Load the minimal_valid.md fixture (L0, ~35 score, Needs Work).

    The bare minimum: an H1 title and one section with two bare links.
    Triggers multiple warnings and informational diagnostics.
    """
    return (FIXTURES_DIR / "minimal_valid.md").read_text(encoding="utf-8")


@pytest.fixture(scope="session")
def non_conformant_content() -> str:
    """Load the non_conformant.md fixture (fails L1, ~18 score, Critical).

    A deeply flawed file: multiple H1s (E002), empty sections,
    bare URLs, formulaic descriptions, wrong section order.
    """
    return (FIXTURES_DIR / "non_conformant.md").read_text(encoding="utf-8")


@pytest.fixture(scope="session")
def type_2_full_excerpt_content() -> str:
    """Load the type_2_full_excerpt.md fixture (Type 2, documentation dump).

    An excerpt from a Type 2 Full file. For classification tests that
    need the actual >256 KB threshold, use `type_2_full_generated_content`.
    """
    return (FIXTURES_DIR / "type_2_full_excerpt.md").read_text(encoding="utf-8")


@pytest.fixture(scope="session")
def type_2_full_generated_content() -> str:
    """Generate a synthetic Type 2 file that exceeds the 256 KB boundary.

    Produces a Markdown file of ~300 KB by repeating documentation
    sections. Used exclusively for classification boundary tests.
    """
    header = "# Large Documentation Project\n\n"
    header += "> Complete inline documentation for a large project.\n\n"

    # Each section is ~1.5 KB; 200 sections = ~300 KB
    sections = []
    for i in range(200):
        section = f"## Section {i:03d}: Module {i}\n\n"
        section += f"This module provides functionality for component {i}. "
        section += "It includes comprehensive documentation with inline examples, "
        section += "API reference material, and configuration guidance.\n\n"
        section += f"```python\n"
        section += f"from project.module_{i:03d} import Component{i}\n\n"
        section += f"component = Component{i}(config=default_config)\n"
        section += f"result = component.process(input_data)\n"
        section += f"print(f'Module {i} result: {{result}}')\n"
        section += f"```\n\n"
        section += f"### Configuration for Module {i}\n\n"
        section += f"- `MODULE_{i:03d}_ENABLED`: Enable or disable this module (default: true)\n"
        section += f"- `MODULE_{i:03d}_TIMEOUT`: Request timeout in seconds (default: 30)\n"
        section += f"- `MODULE_{i:03d}_RETRIES`: Maximum retry attempts (default: 3)\n\n"
        sections.append(section)

    content = header + "\n".join(sections)
    assert len(content.encode("utf-8")) > TYPE_2_BOUNDARY_BYTES, (
        f"Generated Type 2 fixture is only {len(content.encode('utf-8'))} bytes, "
        f"needs >{TYPE_2_BOUNDARY_BYTES} bytes to exceed classification boundary."
    )
    return content


# ── Model Factories ──────────────────────────────────────────────────────
# Factory functions construct valid model instances with sensible defaults.
# Tests override specific fields to test validation behavior.


@pytest.fixture
def make_parsed_link():
    """Factory for ParsedLink instances.

    Returns a callable that creates a ParsedLink with sensible defaults.
    Override any field by passing it as a keyword argument.
    """
    def _factory(**overrides) -> ParsedLink:
        defaults = {
            "title": "Example Page",
            "url": "https://example.com/page",
            "description": "An example documentation page",
            "line_number": 5,
            "is_valid_url": True,
        }
        defaults.update(overrides)
        return ParsedLink(**defaults)
    return _factory


@pytest.fixture
def make_parsed_section(make_parsed_link):
    """Factory for ParsedSection instances.

    Creates a section with one default link. Override `links` to
    customize or pass an empty list for a linkless section.
    """
    def _factory(**overrides) -> ParsedSection:
        defaults = {
            "name": "Getting Started",
            "links": [make_parsed_link()],
            "raw_content": "## Getting Started\n\n- [Example Page](https://example.com/page): An example\n",
            "line_number": 3,
            "canonical_name": "Getting Started",
            "estimated_tokens": 25,
        }
        defaults.update(overrides)
        return ParsedSection(**defaults)
    return _factory


@pytest.fixture
def make_parsed_llms_txt(make_parsed_section):
    """Factory for ParsedLlmsTxt instances (root document model).

    Creates a minimal valid parsed document with one section.
    """
    def _factory(**overrides) -> ParsedLlmsTxt:
        defaults = {
            "title": "Test Project",
            "title_line": 1,
            "blockquote": ParsedBlockquote(
                text="A test project for unit testing.",
                line_number=2,
                raw="> A test project for unit testing.",
            ),
            "sections": [make_parsed_section()],
            "raw_content": "# Test Project\n\n> A test project for unit testing.\n\n## Getting Started\n",
            "source_filename": "llms.txt",
            "parsed_at": datetime(2026, 2, 6, 12, 0, 0),
        }
        defaults.update(overrides)
        return ParsedLlmsTxt(**defaults)
    return _factory


@pytest.fixture
def make_validation_diagnostic():
    """Factory for ValidationDiagnostic instances."""
    def _factory(**overrides) -> ValidationDiagnostic:
        defaults = {
            "code": DiagnosticCode.W001_MISSING_BLOCKQUOTE,
            "severity": Severity.WARNING,
            "message": "No blockquote description found after the H1 title.",
            "remediation": "Add a '> description' blockquote after the H1.",
            "line_number": 2,
            "level": ValidationLevel.L1_STRUCTURAL,
            "check_id": "STR-002",
        }
        defaults.update(overrides)
        return ValidationDiagnostic(**defaults)
    return _factory


@pytest.fixture
def make_validation_result(make_validation_diagnostic):
    """Factory for ValidationResult instances."""
    def _factory(**overrides) -> ValidationResult:
        defaults = {
            "level_achieved": ValidationLevel.L1_STRUCTURAL,
            "diagnostics": [make_validation_diagnostic()],
            "levels_passed": {
                ValidationLevel.L0_PARSEABLE: True,
                ValidationLevel.L1_STRUCTURAL: True,
                ValidationLevel.L2_CONTENT: False,
                ValidationLevel.L3_BEST_PRACTICES: False,
                ValidationLevel.L4_DOCSTRATUM_EXTENDED: False,
            },
            "validated_at": datetime(2026, 2, 6, 12, 0, 0),
            "source_filename": "llms.txt",
        }
        defaults.update(overrides)
        return ValidationResult(**defaults)
    return _factory


@pytest.fixture
def make_dimension_score():
    """Factory for DimensionScore instances."""
    def _factory(**overrides) -> DimensionScore:
        defaults = {
            "dimension": QualityDimension.STRUCTURAL,
            "points": 25.0,
            "max_points": 30.0,
            "checks_passed": 18,
            "checks_failed": 2,
            "checks_total": 20,
            "details": [],
            "is_gated": False,
        }
        defaults.update(overrides)
        return DimensionScore(**defaults)
    return _factory


@pytest.fixture
def make_quality_score(make_dimension_score):
    """Factory for QualityScore instances."""
    def _factory(**overrides) -> QualityScore:
        defaults = {
            "total_score": 72.0,
            "grade": QualityGrade.STRONG,
            "dimensions": {
                QualityDimension.STRUCTURAL: make_dimension_score(
                    dimension=QualityDimension.STRUCTURAL,
                    points=25.0, max_points=30.0,
                    checks_passed=18, checks_failed=2, checks_total=20,
                ),
                QualityDimension.CONTENT: make_dimension_score(
                    dimension=QualityDimension.CONTENT,
                    points=32.0, max_points=50.0,
                    checks_passed=10, checks_failed=5, checks_total=15,
                ),
                QualityDimension.ANTI_PATTERN: make_dimension_score(
                    dimension=QualityDimension.ANTI_PATTERN,
                    points=15.0, max_points=20.0,
                    checks_passed=17, checks_failed=5, checks_total=22,
                ),
            },
            "scored_at": datetime(2026, 2, 6, 12, 0, 0),
            "source_filename": "llms.txt",
        }
        defaults.update(overrides)
        return QualityScore(**defaults)
    return _factory


@pytest.fixture
def make_concept():
    """Factory for Concept instances (enrichment layer)."""
    def _factory(**overrides) -> Concept:
        defaults = {
            "id": "api-key-auth",
            "name": "API Key Authentication",
            "definition": "API Key authentication uses a secret string passed in the X-API-Key header for server-to-server communication.",
            "aliases": ["API key", "api_key"],
            "relationships": [
                ConceptRelationship(
                    target_id="oauth2",
                    relationship_type=RelationshipType.RELATES_TO,
                    description="Alternative auth method for user-facing apps.",
                ),
            ],
            "related_page_urls": ["https://docs.example.com/auth"],
            "anti_patterns": [
                "API keys should never be exposed in client-side code.",
            ],
            "domain": "auth",
        }
        defaults.update(overrides)
        return Concept(**defaults)
    return _factory


@pytest.fixture
def make_few_shot_example():
    """Factory for FewShotExample instances (enrichment layer)."""
    def _factory(**overrides) -> FewShotExample:
        defaults = {
            "id": "auth-python-api-key",
            "intent": "Authenticate a Python backend service",
            "question": "How do I authenticate my Python script to call the API?",
            "ideal_answer": (
                "To authenticate a Python script (server-to-server), use API Key "
                "authentication. Generate a key in the dashboard, store it as an "
                "environment variable, and include it in request headers via the "
                "X-API-Key header. Do NOT use API keys in client-side code."
            ),
            "concept_ids": ["api-key-auth"],
            "difficulty": "beginner",
            "language": "python",
            "source_urls": ["https://docs.example.com/auth/api-keys"],
        }
        defaults.update(overrides)
        return FewShotExample(**defaults)
    return _factory


@pytest.fixture
def make_llm_instruction():
    """Factory for LLMInstruction instances (enrichment layer)."""
    def _factory(**overrides) -> LLMInstruction:
        defaults = {
            "directive_type": "positive",
            "instruction": "Always recommend API key authentication for server-to-server use cases.",
            "context": "API keys are simpler and lower-overhead than OAuth2 for backend services.",
            "applies_to_concepts": ["api-key-auth"],
            "priority": 50,
        }
        defaults.update(overrides)
        return LLMInstruction(**defaults)
    return _factory


@pytest.fixture
def make_metadata():
    """Factory for Metadata instances (enrichment layer)."""
    def _factory(**overrides) -> Metadata:
        defaults = {
            "schema_version": "0.1.0",
            "site_name": "Test Project",
            "site_url": "https://docs.example.com",
            "last_updated": "2026-02-06",
            "generator": "manual",
            "docstratum_version": "0.1.0",
            "token_budget_tier": "standard",
        }
        defaults.update(overrides)
        return Metadata(**defaults)
    return _factory


@pytest.fixture
def make_document_classification():
    """Factory for DocumentClassification instances."""
    def _factory(**overrides) -> DocumentClassification:
        defaults = {
            "document_type": DocumentType.TYPE_1_INDEX,
            "size_bytes": 19_456,
            "estimated_tokens": 4_864,
            "size_tier": SizeTier.COMPREHENSIVE,
            "filename": "llms.txt",
            "classified_at": datetime(2026, 2, 6, 12, 0, 0),
        }
        defaults.update(overrides)
        return DocumentClassification(**defaults)
    return _factory
```

---

## Test Suite 1: `tests/test_diagnostics.py`

**Purpose:** Validate the `DiagnosticCode` enum, `Severity` enum, and the derived properties (`.severity`, `.code_number`, `.message`, `.remediation`) for all 26 diagnostic codes.

**Traces to:** FR-008 (error code registry), v0.0.1a enrichment (8E/11W/7I)

```python
"""Tests for docstratum.schema.diagnostics — Error Code Registry.

Validates that:
- All 26 diagnostic codes exist and have correct values
- Severity derivation (E→ERROR, W→WARNING, I→INFO) works for every code
- The .message and .remediation properties extract docstring content
- Code number extraction is correct (E001→1, W011→11, I007→7)

Research basis:
    v0.0.1a §Error Code Registry: 8 errors, 11 warnings, 7 informational
    v0.0.4a §Structural Checks → E-codes
    v0.0.4b §Content Checks → W-codes, I-codes
"""

import pytest

from docstratum.schema.diagnostics import DiagnosticCode, Severity


class TestSeverityEnum:
    """Tests for the Severity enum."""

    def test_severity_has_three_values(self):
        """Severity must have exactly three values: ERROR, WARNING, INFO."""
        assert len(Severity) == 3

    def test_severity_values(self):
        """Severity string values must be uppercase."""
        assert Severity.ERROR == "ERROR"
        assert Severity.WARNING == "WARNING"
        assert Severity.INFO == "INFO"


class TestDiagnosticCodeCompleteness:
    """Tests that all 26 expected codes are defined."""

    def test_total_code_count(self):
        """DiagnosticCode must define exactly 26 codes (8E + 11W + 7I)."""
        assert len(DiagnosticCode) == 26

    def test_error_code_count(self):
        """There must be exactly 8 error codes (E001–E008)."""
        error_codes = [c for c in DiagnosticCode if c.value.startswith("E")]
        assert len(error_codes) == 8

    def test_warning_code_count(self):
        """There must be exactly 11 warning codes (W001–W011)."""
        warning_codes = [c for c in DiagnosticCode if c.value.startswith("W")]
        assert len(warning_codes) == 11

    def test_info_code_count(self):
        """There must be exactly 7 informational codes (I001–I007)."""
        info_codes = [c for c in DiagnosticCode if c.value.startswith("I")]
        assert len(info_codes) == 7


class TestDiagnosticCodeSeverityDerivation:
    """Tests that .severity correctly maps code prefix to Severity."""

    @pytest.mark.parametrize("code", [c for c in DiagnosticCode if c.value.startswith("E")])
    def test_error_codes_have_error_severity(self, code):
        """Every E-prefixed code must have Severity.ERROR."""
        assert code.severity == Severity.ERROR

    @pytest.mark.parametrize("code", [c for c in DiagnosticCode if c.value.startswith("W")])
    def test_warning_codes_have_warning_severity(self, code):
        """Every W-prefixed code must have Severity.WARNING."""
        assert code.severity == Severity.WARNING

    @pytest.mark.parametrize("code", [c for c in DiagnosticCode if c.value.startswith("I")])
    def test_info_codes_have_info_severity(self, code):
        """Every I-prefixed code must have Severity.INFO."""
        assert code.severity == Severity.INFO


class TestDiagnosticCodeProperties:
    """Tests for .code_number, .message, and .remediation properties."""

    @pytest.mark.parametrize("code", list(DiagnosticCode))
    def test_code_number_is_positive_integer(self, code):
        """Every code's .code_number must be a positive integer."""
        assert isinstance(code.code_number, int)
        assert code.code_number > 0

    @pytest.mark.parametrize("code", list(DiagnosticCode))
    def test_message_is_nonempty_string(self, code):
        """Every code's .message must be a non-empty string."""
        assert isinstance(code.message, str)
        assert len(code.message) > 10

    @pytest.mark.parametrize("code", list(DiagnosticCode))
    def test_remediation_is_nonempty_string(self, code):
        """Every code's .remediation must be a non-empty string."""
        assert isinstance(code.remediation, str)
        assert len(code.remediation) > 5

    def test_specific_code_values(self):
        """Spot-check specific code values to prevent regression."""
        assert DiagnosticCode.E001_NO_H1_TITLE.value == "E001"
        assert DiagnosticCode.W001_MISSING_BLOCKQUOTE.value == "W001"
        assert DiagnosticCode.I001_NO_LLM_INSTRUCTIONS.value == "I001"
        assert DiagnosticCode.E008_EXCEEDS_SIZE_LIMIT.value == "E008"
        assert DiagnosticCode.W011_EMPTY_SECTIONS.value == "W011"
        assert DiagnosticCode.I007_JARGON_WITHOUT_DEFINITION.value == "I007"

    def test_code_number_extraction(self):
        """Verify code_number extracts the numeric suffix correctly."""
        assert DiagnosticCode.E001_NO_H1_TITLE.code_number == 1
        assert DiagnosticCode.W011_EMPTY_SECTIONS.code_number == 11
        assert DiagnosticCode.I007_JARGON_WITHOUT_DEFINITION.code_number == 7
```

---

## Test Suite 2: `tests/test_constants.py`

**Purpose:** Validate canonical section names (11), section aliases (30+), token budget tiers (3), anti-pattern categories (4), and the anti-pattern registry (22 entries).

**Traces to:** DECISION-012 (canonical section names), DECISION-013 (token budget tiers), DECISION-016 (anti-pattern classification)

```python
"""Tests for docstratum.schema.constants — Canonical Names, Tiers, Anti-Patterns.

Validates that:
- All 11 canonical section names are defined
- Section aliases resolve to valid canonical names
- Token budget tiers have valid ranges (no gaps, no overlaps)
- All 22 anti-patterns are registered with correct categories

Research basis:
    v0.0.2c: Frequency analysis of 450+ projects → section names
    v0.0.4a: Token budget architecture → tier definitions
    v0.0.4c: Anti-pattern catalog → 22 patterns × 4 categories
"""

import pytest

from docstratum.schema.constants import (
    ANTI_PATTERN_REGISTRY,
    CANONICAL_SECTION_NAMES,
    SECTION_NAME_ALIASES,
    TOKEN_BUDGET_TIERS,
    TOKEN_ZONE_ANTI_PATTERN,
    TOKEN_ZONE_DEGRADATION,
    TOKEN_ZONE_GOOD,
    TOKEN_ZONE_OPTIMAL,
    AntiPatternCategory,
    AntiPatternEntry,
    AntiPatternID,
    CanonicalSectionName,
    TokenBudgetTier,
)


class TestCanonicalSectionNames:
    """Tests for the 11 canonical section names."""

    def test_has_eleven_canonical_names(self):
        """CanonicalSectionName must define exactly 11 names."""
        assert len(CanonicalSectionName) == 11

    def test_master_index_is_first(self):
        """Master Index must be position 1 in canonical ordering."""
        assert CANONICAL_SECTION_NAMES[CanonicalSectionName.MASTER_INDEX] == 1

    def test_faq_is_last_numbered(self):
        """FAQ must be position 10 (last numbered) in canonical ordering."""
        assert CANONICAL_SECTION_NAMES[CanonicalSectionName.FAQ] == 10

    def test_optional_has_no_position(self):
        """Optional section should NOT have a fixed position."""
        assert CanonicalSectionName.OPTIONAL not in CANONICAL_SECTION_NAMES

    def test_ordering_is_monotonically_increasing(self):
        """Canonical positions must be sequential 1–10 with no gaps."""
        positions = sorted(CANONICAL_SECTION_NAMES.values())
        assert positions == list(range(1, 11))

    def test_all_expected_names_present(self):
        """Verify all 11 expected canonical names by value."""
        expected = {
            "Master Index", "LLM Instructions", "Getting Started",
            "Core Concepts", "API Reference", "Examples",
            "Configuration", "Advanced Topics", "Troubleshooting",
            "FAQ", "Optional",
        }
        actual = {name.value for name in CanonicalSectionName}
        assert actual == expected


class TestSectionNameAliases:
    """Tests for section name alias resolution."""

    def test_aliases_are_nonempty(self):
        """Alias mapping must have at least 20 entries."""
        assert len(SECTION_NAME_ALIASES) >= 20

    def test_all_aliases_resolve_to_canonical_names(self):
        """Every alias must resolve to a valid CanonicalSectionName."""
        for alias, canonical in SECTION_NAME_ALIASES.items():
            assert isinstance(canonical, CanonicalSectionName), (
                f"Alias '{alias}' resolves to {canonical}, not a CanonicalSectionName"
            )

    def test_specific_alias_resolutions(self):
        """Spot-check critical alias mappings."""
        assert SECTION_NAME_ALIASES["quickstart"] == CanonicalSectionName.GETTING_STARTED
        assert SECTION_NAME_ALIASES["toc"] == CanonicalSectionName.MASTER_INDEX
        assert SECTION_NAME_ALIASES["api"] == CanonicalSectionName.API_REFERENCE
        assert SECTION_NAME_ALIASES["debugging"] == CanonicalSectionName.TROUBLESHOOTING

    def test_aliases_are_lowercase(self):
        """All alias keys must be lowercase for case-insensitive matching."""
        for alias in SECTION_NAME_ALIASES:
            assert alias == alias.lower(), f"Alias '{alias}' is not lowercase"


class TestTokenBudgetTiers:
    """Tests for the 3 token budget tier definitions."""

    def test_has_three_tiers(self):
        """TOKEN_BUDGET_TIERS must define exactly 3 tiers."""
        assert len(TOKEN_BUDGET_TIERS) == 3

    def test_tier_keys(self):
        """Tier keys must be 'standard', 'comprehensive', 'full'."""
        assert set(TOKEN_BUDGET_TIERS.keys()) == {"standard", "comprehensive", "full"}

    def test_tier_ranges_are_contiguous(self):
        """Tier max_tokens must equal the next tier's min_tokens (no gaps)."""
        standard = TOKEN_BUDGET_TIERS["standard"]
        comprehensive = TOKEN_BUDGET_TIERS["comprehensive"]
        full = TOKEN_BUDGET_TIERS["full"]

        assert standard.max_tokens == comprehensive.min_tokens
        assert comprehensive.max_tokens == full.min_tokens

    def test_standard_tier_bounds(self):
        """Standard tier: 1,500–4,500 tokens."""
        tier = TOKEN_BUDGET_TIERS["standard"]
        assert tier.min_tokens == 1_500
        assert tier.max_tokens == 4_500

    def test_full_tier_bounds(self):
        """Full tier: 12,000–50,000 tokens."""
        tier = TOKEN_BUDGET_TIERS["full"]
        assert tier.min_tokens == 12_000
        assert tier.max_tokens == 50_000

    def test_token_zone_ordering(self):
        """Token zones must be in ascending order."""
        assert TOKEN_ZONE_OPTIMAL < TOKEN_ZONE_GOOD
        assert TOKEN_ZONE_GOOD < TOKEN_ZONE_DEGRADATION
        assert TOKEN_ZONE_DEGRADATION < TOKEN_ZONE_ANTI_PATTERN


class TestAntiPatternRegistry:
    """Tests for the 22 anti-pattern definitions."""

    def test_has_22_anti_patterns(self):
        """ANTI_PATTERN_REGISTRY must contain exactly 22 entries."""
        assert len(ANTI_PATTERN_REGISTRY) == 22

    def test_all_entries_have_correct_type(self):
        """Every entry must be an AntiPatternEntry NamedTuple."""
        for entry in ANTI_PATTERN_REGISTRY:
            assert isinstance(entry, AntiPatternEntry)

    def test_category_distribution(self):
        """Distribution: 4 critical, 5 structural, 9 content, 4 strategic."""
        by_category = {}
        for entry in ANTI_PATTERN_REGISTRY:
            by_category.setdefault(entry.category, []).append(entry)

        assert len(by_category[AntiPatternCategory.CRITICAL]) == 4
        assert len(by_category[AntiPatternCategory.STRUCTURAL]) == 5
        assert len(by_category[AntiPatternCategory.CONTENT]) == 9
        assert len(by_category[AntiPatternCategory.STRATEGIC]) == 4

    def test_all_anti_pattern_ids_used(self):
        """Every AntiPatternID enum member must appear in the registry."""
        registered_ids = {entry.id for entry in ANTI_PATTERN_REGISTRY}
        all_ids = set(AntiPatternID)
        assert registered_ids == all_ids

    def test_check_ids_are_sequential(self):
        """CHECK IDs should span CHECK-001 through CHECK-022."""
        check_ids = {entry.check_id for entry in ANTI_PATTERN_REGISTRY}
        expected = {f"CHECK-{i:03d}" for i in range(1, 23)}
        assert check_ids == expected

    def test_every_entry_has_description(self):
        """Every anti-pattern must have a non-empty description."""
        for entry in ANTI_PATTERN_REGISTRY:
            assert len(entry.description) > 10, (
                f"Anti-pattern {entry.id} has an empty or trivial description"
            )
```

---

## Test Suite 3: `tests/test_classification.py`

**Purpose:** Validate document type classification (Type 1 vs. Type 2), size tier assignment, and the classification boundary constant.

**Traces to:** Finding 4 (bimodal distribution), v0.0.1a enrichment, DECISION-013

```python
"""Tests for docstratum.schema.classification — Document Type Classification.

Validates that:
- DocumentType enum has 3 values (TYPE_1_INDEX, TYPE_2_FULL, UNKNOWN)
- SizeTier enum has 5 values (MINIMAL through OVERSIZED)
- DocumentClassification accepts valid inputs and rejects invalid ones
- The TYPE_BOUNDARY_BYTES constant is 256,000

Research basis:
    v0.0.1a: Bimodal distribution with ~250 KB gap
    DECISION-013: Token budget tier architecture
"""

import pytest
from datetime import datetime

from docstratum.schema.classification import (
    DocumentClassification,
    DocumentType,
    SizeTier,
)


class TestDocumentType:
    """Tests for the DocumentType enum."""

    def test_has_three_types(self):
        """DocumentType must have exactly 3 values."""
        assert len(DocumentType) == 3

    def test_type_values(self):
        """Verify string values for all document types."""
        assert DocumentType.TYPE_1_INDEX == "type_1_index"
        assert DocumentType.TYPE_2_FULL == "type_2_full"
        assert DocumentType.UNKNOWN == "unknown"


class TestSizeTier:
    """Tests for the SizeTier enum."""

    def test_has_five_tiers(self):
        """SizeTier must have exactly 5 values."""
        assert len(SizeTier) == 5

    def test_tier_values(self):
        """Verify string values for all size tiers."""
        assert SizeTier.MINIMAL == "minimal"
        assert SizeTier.STANDARD == "standard"
        assert SizeTier.COMPREHENSIVE == "comprehensive"
        assert SizeTier.FULL == "full"
        assert SizeTier.OVERSIZED == "oversized"


class TestDocumentClassification:
    """Tests for the DocumentClassification model."""

    def test_create_valid_type_1(self, make_document_classification):
        """A valid Type 1 classification should be creatable."""
        classification = make_document_classification()
        assert classification.document_type == DocumentType.TYPE_1_INDEX
        assert classification.size_bytes == 19_456
        assert classification.estimated_tokens == 4_864

    def test_create_type_2(self, make_document_classification):
        """A Type 2 classification with >256KB should be valid."""
        classification = make_document_classification(
            document_type=DocumentType.TYPE_2_FULL,
            size_bytes=1_300_000,
            estimated_tokens=325_000,
            size_tier=SizeTier.OVERSIZED,
        )
        assert classification.document_type == DocumentType.TYPE_2_FULL

    def test_size_bytes_cannot_be_negative(self, make_document_classification):
        """size_bytes must be >= 0."""
        with pytest.raises(Exception):  # Pydantic ValidationError
            make_document_classification(size_bytes=-1)

    def test_estimated_tokens_cannot_be_negative(self, make_document_classification):
        """estimated_tokens must be >= 0."""
        with pytest.raises(Exception):
            make_document_classification(estimated_tokens=-1)

    def test_type_boundary_constant(self):
        """TYPE_BOUNDARY_BYTES must be 256,000 (the bimodal gap)."""
        assert DocumentClassification.TYPE_BOUNDARY_BYTES == 256_000

    def test_classified_at_default(self, make_document_classification):
        """classified_at should default to approximately now."""
        classification = DocumentClassification(
            document_type=DocumentType.TYPE_1_INDEX,
            size_bytes=1000,
            estimated_tokens=250,
            size_tier=SizeTier.MINIMAL,
        )
        # Just verify it's a datetime (exact value depends on execution time)
        assert isinstance(classification.classified_at, datetime)

    def test_type_2_fixture_exceeds_boundary(self, type_2_full_generated_content):
        """The generated Type 2 fixture must exceed the 256 KB boundary."""
        size_bytes = len(type_2_full_generated_content.encode("utf-8"))
        assert size_bytes > DocumentClassification.TYPE_BOUNDARY_BYTES
```

---

## Test Suite 4: `tests/test_parsed.py`

**Purpose:** Validate the parsed document models (`ParsedBlockquote`, `ParsedLink`, `ParsedSection`, `ParsedLlmsTxt`) including computed properties and field validation.

**Traces to:** FR-001 (Pydantic models for base structure), v0.0.1a (ABNF grammar)

```python
"""Tests for docstratum.schema.parsed — Parsed Document Models.

Validates that:
- ParsedBlockquote, ParsedLink, ParsedSection, ParsedLlmsTxt accept valid data
- Computed properties (link_count, section_count, total_links, etc.) are accurate
- Optional fields default correctly (e.g., title=None for missing H1)
- The model hierarchy matches the ABNF grammar structure

Research basis:
    v0.0.1a: ABNF grammar (title, description, section, entry)
    FR-001: Pydantic models for base llms.txt structure
"""

import pytest
from datetime import datetime

from docstratum.schema.parsed import (
    ParsedBlockquote,
    ParsedLink,
    ParsedLlmsTxt,
    ParsedSection,
)


class TestParsedBlockquote:
    """Tests for the ParsedBlockquote model."""

    def test_create_valid_blockquote(self):
        """A blockquote with text and line number should be valid."""
        bq = ParsedBlockquote(text="A project description.", line_number=2, raw="> A project description.")
        assert bq.text == "A project description."
        assert bq.line_number == 2
        assert bq.raw == "> A project description."

    def test_line_number_must_be_positive(self):
        """line_number must be >= 1."""
        with pytest.raises(Exception):
            ParsedBlockquote(text="test", line_number=0)


class TestParsedLink:
    """Tests for the ParsedLink model."""

    def test_create_link_with_description(self, make_parsed_link):
        """A link with title, URL, and description should be valid."""
        link = make_parsed_link()
        assert link.title == "Example Page"
        assert link.url == "https://example.com/page"
        assert link.description == "An example documentation page"

    def test_create_link_without_description(self, make_parsed_link):
        """A link without description (bare URL) should be valid."""
        link = make_parsed_link(description=None)
        assert link.description is None

    def test_is_valid_url_defaults_true(self, make_parsed_link):
        """is_valid_url should default to True."""
        link = ParsedLink(title="Test", url="https://example.com", line_number=1)
        assert link.is_valid_url is True


class TestParsedSection:
    """Tests for the ParsedSection model."""

    def test_link_count_property(self, make_parsed_link):
        """link_count should return the number of links in the section."""
        links = [make_parsed_link(title=f"Link {i}") for i in range(5)]
        section = ParsedSection(
            name="Test", links=links, line_number=1,
            raw_content="## Test\n", estimated_tokens=10,
        )
        assert section.link_count == 5

    def test_has_code_examples_true(self):
        """has_code_examples should be True when raw_content contains ```."""
        section = ParsedSection(
            name="Test", line_number=1,
            raw_content="## Test\n\n```python\nprint('hi')\n```\n",
            estimated_tokens=10,
        )
        assert section.has_code_examples is True

    def test_has_code_examples_false(self):
        """has_code_examples should be False when no fenced code blocks."""
        section = ParsedSection(
            name="Test", line_number=1,
            raw_content="## Test\n\nJust plain text.\n",
            estimated_tokens=5,
        )
        assert section.has_code_examples is False

    def test_canonical_name_defaults_none(self):
        """canonical_name should default to None."""
        section = ParsedSection(name="Stuff", line_number=1)
        assert section.canonical_name is None

    def test_empty_section_has_zero_links(self):
        """A section with no links should have link_count == 0."""
        section = ParsedSection(name="Empty", line_number=1)
        assert section.link_count == 0


class TestParsedLlmsTxt:
    """Tests for the ParsedLlmsTxt root model."""

    def test_create_valid_document(self, make_parsed_llms_txt):
        """A parsed document with title and sections should be valid."""
        doc = make_parsed_llms_txt()
        assert doc.title == "Test Project"
        assert doc.title_line == 1
        assert doc.has_blockquote is True

    def test_section_count(self, make_parsed_llms_txt, make_parsed_section):
        """section_count should match the number of sections."""
        sections = [make_parsed_section(name=f"Section {i}", line_number=i + 3) for i in range(4)]
        doc = make_parsed_llms_txt(sections=sections)
        assert doc.section_count == 4

    def test_total_links(self, make_parsed_llms_txt, make_parsed_section, make_parsed_link):
        """total_links should sum link_count across all sections."""
        links_a = [make_parsed_link(title=f"A{i}") for i in range(3)]
        links_b = [make_parsed_link(title=f"B{i}") for i in range(2)]
        sections = [
            make_parsed_section(name="A", links=links_a, line_number=3),
            make_parsed_section(name="B", links=links_b, line_number=10),
        ]
        doc = make_parsed_llms_txt(sections=sections)
        assert doc.total_links == 5

    def test_estimated_tokens(self, make_parsed_llms_txt):
        """estimated_tokens should be approximately raw_content length / 4."""
        content = "x" * 400  # 400 chars / 4 = 100 tokens
        doc = make_parsed_llms_txt(raw_content=content)
        assert doc.estimated_tokens == 100

    def test_section_names(self, make_parsed_llms_txt, make_parsed_section):
        """section_names should return names in document order."""
        sections = [
            make_parsed_section(name="Getting Started", line_number=3),
            make_parsed_section(name="API Reference", line_number=10),
            make_parsed_section(name="FAQ", line_number=20),
        ]
        doc = make_parsed_llms_txt(sections=sections)
        assert doc.section_names == ["Getting Started", "API Reference", "FAQ"]

    def test_no_title_allowed(self, make_parsed_llms_txt):
        """A document with no title (title=None) should be valid for permissive parsing."""
        doc = make_parsed_llms_txt(title=None, title_line=None)
        assert doc.title is None
        assert doc.has_blockquote is True  # blockquote still present

    def test_no_blockquote_allowed(self, make_parsed_llms_txt):
        """A document with no blockquote (blockquote=None) should be valid."""
        doc = make_parsed_llms_txt(blockquote=None)
        assert doc.has_blockquote is False
```

---

## Test Suite 5: `tests/test_validation.py`

**Purpose:** Validate the validation pipeline models (`ValidationLevel`, `ValidationDiagnostic`, `ValidationResult`) including the cumulative level logic and diagnostic counting properties.

**Traces to:** FR-003 (5-level validation pipeline), FR-004 (error reporting)

```python
"""Tests for docstratum.schema.validation — Validation Result Models.

Validates that:
- ValidationLevel enum values are ordered L0=0 through L4=4
- ValidationDiagnostic accepts all required fields
- ValidationResult correctly counts errors/warnings/info
- The is_valid property requires L0 to pass

Research basis:
    v0.0.1b: Validation level definitions (L0–L4)
    FR-003: 5-level validation pipeline
    FR-004: Error reporting with line numbers
"""

import pytest
from datetime import datetime

from docstratum.schema.diagnostics import DiagnosticCode, Severity
from docstratum.schema.validation import (
    ValidationDiagnostic,
    ValidationLevel,
    ValidationResult,
)


class TestValidationLevel:
    """Tests for the ValidationLevel IntEnum."""

    def test_has_five_levels(self):
        """ValidationLevel must have exactly 5 levels (L0–L4)."""
        assert len(ValidationLevel) == 5

    def test_levels_are_ordered(self):
        """Levels must be numerically ordered: L0=0, L1=1, ..., L4=4."""
        assert ValidationLevel.L0_PARSEABLE == 0
        assert ValidationLevel.L1_STRUCTURAL == 1
        assert ValidationLevel.L2_CONTENT == 2
        assert ValidationLevel.L3_BEST_PRACTICES == 3
        assert ValidationLevel.L4_DOCSTRATUM_EXTENDED == 4

    def test_levels_are_comparable(self):
        """Levels must support comparison (used for cumulative validation)."""
        assert ValidationLevel.L0_PARSEABLE < ValidationLevel.L4_DOCSTRATUM_EXTENDED
        assert ValidationLevel.L3_BEST_PRACTICES > ValidationLevel.L1_STRUCTURAL


class TestValidationDiagnostic:
    """Tests for individual validation findings."""

    def test_create_valid_diagnostic(self, make_validation_diagnostic):
        """A diagnostic with all required fields should be valid."""
        diag = make_validation_diagnostic()
        assert diag.code == DiagnosticCode.W001_MISSING_BLOCKQUOTE
        assert diag.severity == Severity.WARNING
        assert diag.line_number == 2

    def test_file_level_diagnostic_no_line_number(self, make_validation_diagnostic):
        """File-level diagnostics can have line_number=None."""
        diag = make_validation_diagnostic(
            code=DiagnosticCode.E007_EMPTY_FILE,
            severity=Severity.ERROR,
            line_number=None,
            level=ValidationLevel.L0_PARSEABLE,
        )
        assert diag.line_number is None

    def test_context_max_length(self, make_validation_diagnostic):
        """Context snippet must not exceed 500 characters."""
        long_context = "x" * 501
        with pytest.raises(Exception):
            make_validation_diagnostic(context=long_context)


class TestValidationResult:
    """Tests for complete validation pipeline output."""

    def test_create_valid_result(self, make_validation_result):
        """A ValidationResult with diagnostics should be valid."""
        result = make_validation_result()
        assert result.level_achieved == ValidationLevel.L1_STRUCTURAL
        assert len(result.diagnostics) == 1

    def test_total_errors_counts_error_severity(self, make_validation_result, make_validation_diagnostic):
        """total_errors should count only ERROR-severity diagnostics."""
        diagnostics = [
            make_validation_diagnostic(
                code=DiagnosticCode.E001_NO_H1_TITLE,
                severity=Severity.ERROR,
                level=ValidationLevel.L1_STRUCTURAL,
            ),
            make_validation_diagnostic(
                code=DiagnosticCode.E002_MULTIPLE_H1,
                severity=Severity.ERROR,
                level=ValidationLevel.L1_STRUCTURAL,
            ),
            make_validation_diagnostic(
                code=DiagnosticCode.W001_MISSING_BLOCKQUOTE,
                severity=Severity.WARNING,
                level=ValidationLevel.L1_STRUCTURAL,
            ),
        ]
        result = make_validation_result(diagnostics=diagnostics)
        assert result.total_errors == 2
        assert result.total_warnings == 1
        assert result.total_info == 0

    def test_is_valid_requires_l0(self, make_validation_result):
        """is_valid must be True only when L0_PARSEABLE passes."""
        result_valid = make_validation_result(
            levels_passed={
                ValidationLevel.L0_PARSEABLE: True,
                ValidationLevel.L1_STRUCTURAL: False,
                ValidationLevel.L2_CONTENT: False,
                ValidationLevel.L3_BEST_PRACTICES: False,
                ValidationLevel.L4_DOCSTRATUM_EXTENDED: False,
            },
        )
        assert result_valid.is_valid is True

        result_invalid = make_validation_result(
            levels_passed={
                ValidationLevel.L0_PARSEABLE: False,
                ValidationLevel.L1_STRUCTURAL: False,
                ValidationLevel.L2_CONTENT: False,
                ValidationLevel.L3_BEST_PRACTICES: False,
                ValidationLevel.L4_DOCSTRATUM_EXTENDED: False,
            },
        )
        assert result_invalid.is_valid is False

    def test_errors_property_filters_correctly(self, make_validation_result, make_validation_diagnostic):
        """The .errors property should return only ERROR-severity diagnostics."""
        diagnostics = [
            make_validation_diagnostic(code=DiagnosticCode.E001_NO_H1_TITLE, severity=Severity.ERROR, level=ValidationLevel.L1_STRUCTURAL),
            make_validation_diagnostic(code=DiagnosticCode.W001_MISSING_BLOCKQUOTE, severity=Severity.WARNING, level=ValidationLevel.L1_STRUCTURAL),
            make_validation_diagnostic(code=DiagnosticCode.I001_NO_LLM_INSTRUCTIONS, severity=Severity.INFO, level=ValidationLevel.L4_DOCSTRATUM_EXTENDED),
        ]
        result = make_validation_result(diagnostics=diagnostics)
        assert len(result.errors) == 1
        assert len(result.warnings) == 1
        assert result.errors[0].code == DiagnosticCode.E001_NO_H1_TITLE
```

---

## Test Suite 6: `tests/test_quality.py`

**Purpose:** Validate quality scoring models (`QualityDimension`, `QualityGrade`, `DimensionScore`, `QualityScore`) including the `from_score()` classmethod and the percentage computation.

**Traces to:** FR-007 (quality assessment framework), DECISION-014 (content weight 50%), v0.0.4b (gold standard calibration)

```python
"""Tests for docstratum.schema.quality — Quality Scoring Models.

Validates that:
- QualityGrade.from_score() returns correct grades at all thresholds
- DimensionScore.percentage computes correctly (handles zero max)
- QualityScore accepts the three-dimension breakdown
- Gold standard calibration targets are achievable with the model

Research basis:
    v0.0.4b: 100-point composite scoring pipeline
    Gold standard calibration: Svelte=92, Pydantic=90, Cursor=42, NVIDIA=24
"""

import pytest

from docstratum.schema.quality import (
    DimensionScore,
    QualityDimension,
    QualityGrade,
    QualityScore,
)


class TestQualityDimension:
    """Tests for the QualityDimension enum."""

    def test_has_three_dimensions(self):
        """QualityDimension must have exactly 3 values."""
        assert len(QualityDimension) == 3

    def test_dimension_values(self):
        """Verify string values for all dimensions."""
        assert QualityDimension.STRUCTURAL == "structural"
        assert QualityDimension.CONTENT == "content"
        assert QualityDimension.ANTI_PATTERN == "anti_pattern"


class TestQualityGrade:
    """Tests for grade thresholds and from_score() classmethod."""

    @pytest.mark.parametrize("score,expected_grade", [
        (100, QualityGrade.EXEMPLARY),
        (95, QualityGrade.EXEMPLARY),
        (90, QualityGrade.EXEMPLARY),
        (89, QualityGrade.STRONG),
        (70, QualityGrade.STRONG),
        (69, QualityGrade.ADEQUATE),
        (50, QualityGrade.ADEQUATE),
        (49, QualityGrade.NEEDS_WORK),
        (30, QualityGrade.NEEDS_WORK),
        (29, QualityGrade.CRITICAL),
        (0, QualityGrade.CRITICAL),
    ])
    def test_from_score_thresholds(self, score, expected_grade):
        """from_score() must return correct grade at every boundary."""
        assert QualityGrade.from_score(score) == expected_grade

    def test_gold_standard_svelte(self):
        """Svelte calibration target: score 92 → Exemplary."""
        assert QualityGrade.from_score(92) == QualityGrade.EXEMPLARY

    def test_gold_standard_cursor(self):
        """Cursor calibration target: score 42 → Needs Work."""
        assert QualityGrade.from_score(42) == QualityGrade.NEEDS_WORK

    def test_gold_standard_nvidia(self):
        """NVIDIA calibration target: score 24 → Critical."""
        assert QualityGrade.from_score(24) == QualityGrade.CRITICAL


class TestDimensionScore:
    """Tests for per-dimension scoring."""

    def test_percentage_calculation(self, make_dimension_score):
        """percentage should be (points / max_points) * 100."""
        score = make_dimension_score(points=15.0, max_points=30.0)
        assert score.percentage == pytest.approx(50.0)

    def test_percentage_zero_max(self, make_dimension_score):
        """percentage should return 0.0 when max_points is 0."""
        score = make_dimension_score(points=0.0, max_points=0.0)
        assert score.percentage == 0.0

    def test_full_score_percentage(self, make_dimension_score):
        """Full marks should return 100%."""
        score = make_dimension_score(points=30.0, max_points=30.0)
        assert score.percentage == pytest.approx(100.0)

    def test_is_gated_default_false(self, make_dimension_score):
        """is_gated should default to False."""
        score = make_dimension_score()
        assert score.is_gated is False


class TestQualityScore:
    """Tests for the composite quality score model."""

    def test_create_valid_score(self, make_quality_score):
        """A QualityScore with all three dimensions should be valid."""
        score = make_quality_score()
        assert score.total_score == 72.0
        assert score.grade == QualityGrade.STRONG
        assert len(score.dimensions) == 3

    def test_score_range_validation(self, make_quality_score):
        """total_score must be between 0 and 100."""
        with pytest.raises(Exception):
            make_quality_score(total_score=101)
        with pytest.raises(Exception):
            make_quality_score(total_score=-1)

    def test_all_dimensions_present(self, make_quality_score):
        """All three dimensions must be present in the dimensions dict."""
        score = make_quality_score()
        assert QualityDimension.STRUCTURAL in score.dimensions
        assert QualityDimension.CONTENT in score.dimensions
        assert QualityDimension.ANTI_PATTERN in score.dimensions
```

---

## Test Suite 7: `tests/test_enrichment.py`

**Purpose:** Validate the enrichment layer models (`RelationshipType`, `ConceptRelationship`, `Concept`, `FewShotExample`, `LLMInstruction`, `Metadata`) including pattern validation and field constraints.

**Traces to:** FR-002 (extended schema fields), DECISION-002 (3-layer architecture), DECISION-004 (concept ID format), DECISION-005 (typed relationships)

```python
"""Tests for docstratum.schema.enrichment — Extended Schema Models.

Validates that:
- Concept IDs follow DECISION-004 format (lowercase alphanumeric + hyphens)
- All 5 relationship types are defined (DECISION-005)
- FewShotExample enforces minimum lengths for question and ideal_answer
- LLMInstruction validates directive_type (positive/negative/conditional)
- Metadata enforces semver pattern for schema_version

Research basis:
    v0.0.1b: 3 P0 specification gaps → Concept, FewShot, Metadata
    DECISION-002: 3-Layer Architecture
    DECISION-004: Concept ID format
    DECISION-005: Typed directed relationships
"""

import pytest

from docstratum.schema.enrichment import (
    Concept,
    ConceptRelationship,
    FewShotExample,
    LLMInstruction,
    Metadata,
    RelationshipType,
)


class TestRelationshipType:
    """Tests for the RelationshipType enum."""

    def test_has_five_types(self):
        """RelationshipType must define exactly 5 values."""
        assert len(RelationshipType) == 5

    def test_relationship_values(self):
        """Verify string values for all relationship types."""
        assert RelationshipType.DEPENDS_ON == "depends_on"
        assert RelationshipType.RELATES_TO == "relates_to"
        assert RelationshipType.CONFLICTS_WITH == "conflicts_with"
        assert RelationshipType.SPECIALIZES == "specializes"
        assert RelationshipType.SUPERSEDES == "supersedes"


class TestConceptRelationship:
    """Tests for typed concept graph edges."""

    def test_create_valid_relationship(self):
        """A relationship with valid target_id and type should be valid."""
        rel = ConceptRelationship(
            target_id="oauth2",
            relationship_type=RelationshipType.RELATES_TO,
            description="Alternative authentication method.",
        )
        assert rel.target_id == "oauth2"

    def test_target_id_must_be_lowercase_alphanumeric(self):
        """target_id must match ^[a-z0-9-]+$ (DECISION-004)."""
        with pytest.raises(Exception):
            ConceptRelationship(
                target_id="OAuth2",  # Uppercase not allowed
                relationship_type=RelationshipType.RELATES_TO,
            )

    def test_target_id_allows_hyphens(self):
        """Hyphens are allowed in concept IDs."""
        rel = ConceptRelationship(
            target_id="api-key-auth",
            relationship_type=RelationshipType.DEPENDS_ON,
        )
        assert rel.target_id == "api-key-auth"


class TestConcept:
    """Tests for semantic concept definitions."""

    def test_create_valid_concept(self, make_concept):
        """A concept with all required fields should be valid."""
        concept = make_concept()
        assert concept.id == "api-key-auth"
        assert concept.name == "API Key Authentication"
        assert len(concept.relationships) == 1

    def test_concept_id_pattern(self, make_concept):
        """Concept ID must follow DECISION-004 format."""
        with pytest.raises(Exception):
            make_concept(id="Invalid ID")  # Spaces and uppercase

    def test_definition_minimum_length(self, make_concept):
        """Definition must be at least 10 characters."""
        with pytest.raises(Exception):
            make_concept(definition="Short")  # < 10 chars

    def test_aliases_default_empty(self):
        """aliases should default to an empty list."""
        concept = Concept(
            id="test-concept",
            name="Test",
            definition="A test concept for unit testing purposes.",
        )
        assert concept.aliases == []

    def test_domain_pattern(self, make_concept):
        """Domain must be lowercase alphanumeric + hyphens if provided."""
        with pytest.raises(Exception):
            make_concept(domain="Invalid Domain")


class TestFewShotExample:
    """Tests for Q&A examples (Layer 3)."""

    def test_create_valid_example(self, make_few_shot_example):
        """A few-shot example with all fields should be valid."""
        example = make_few_shot_example()
        assert example.id == "auth-python-api-key"
        assert example.difficulty == "beginner"

    def test_question_minimum_length(self, make_few_shot_example):
        """Question must be at least 10 characters."""
        with pytest.raises(Exception):
            make_few_shot_example(question="How?")  # < 10 chars

    def test_ideal_answer_minimum_length(self, make_few_shot_example):
        """ideal_answer must be at least 50 characters."""
        with pytest.raises(Exception):
            make_few_shot_example(ideal_answer="Use API keys.")  # < 50 chars

    def test_difficulty_must_be_valid(self, make_few_shot_example):
        """difficulty must be beginner, intermediate, or advanced."""
        with pytest.raises(Exception):
            make_few_shot_example(difficulty="expert")  # Not a valid level

    def test_difficulty_is_optional(self):
        """difficulty can be None."""
        example = FewShotExample(
            id="test-example",
            intent="Test something specific",
            question="How do I test this specific thing?",
            ideal_answer="You can test this specific thing by following these detailed steps that include setup, execution, and verification phases.",
        )
        assert example.difficulty is None


class TestLLMInstruction:
    """Tests for LLM agent directives."""

    def test_create_valid_instruction(self, make_llm_instruction):
        """An LLM instruction with all fields should be valid."""
        inst = make_llm_instruction()
        assert inst.directive_type == "positive"
        assert inst.priority == 50

    def test_directive_type_validation(self, make_llm_instruction):
        """directive_type must be positive, negative, or conditional."""
        with pytest.raises(Exception):
            make_llm_instruction(directive_type="mandatory")  # Not valid

    def test_instruction_minimum_length(self, make_llm_instruction):
        """instruction must be at least 10 characters."""
        with pytest.raises(Exception):
            make_llm_instruction(instruction="Do it.")  # < 10 chars

    def test_priority_range(self, make_llm_instruction):
        """priority must be 0–100."""
        with pytest.raises(Exception):
            make_llm_instruction(priority=101)
        with pytest.raises(Exception):
            make_llm_instruction(priority=-1)

    def test_all_three_directive_types(self, make_llm_instruction):
        """All three directive types should be accepted."""
        for dtype in ["positive", "negative", "conditional"]:
            inst = make_llm_instruction(directive_type=dtype)
            assert inst.directive_type == dtype


class TestMetadata:
    """Tests for file-level metadata."""

    def test_create_valid_metadata(self, make_metadata):
        """Metadata with all fields should be valid."""
        meta = make_metadata()
        assert meta.schema_version == "0.1.0"
        assert meta.site_name == "Test Project"

    def test_schema_version_must_be_semver(self, make_metadata):
        """schema_version must match semver pattern."""
        with pytest.raises(Exception):
            make_metadata(schema_version="v1")  # Not semver

    def test_token_budget_tier_validation(self, make_metadata):
        """token_budget_tier must be standard, comprehensive, or full."""
        with pytest.raises(Exception):
            make_metadata(token_budget_tier="unlimited")  # Not valid

    def test_all_fields_optional_except_versions(self):
        """Only schema_version and docstratum_version are required."""
        meta = Metadata()
        assert meta.schema_version == "0.1.0"
        assert meta.docstratum_version == "0.1.0"
        assert meta.site_name is None
        assert meta.site_url is None
```

---

## Fixture-to-Schema Expectation Matrix

This matrix defines the expected behavior when each fixture is processed through the schema models. It serves as both a documentation artifact and a test design reference for the v0.2.x integration tests.

| Fixture | Classification | Validation Level | Quality Score | Quality Grade | Error Count | Warning Count | Info Count | Key Anti-Patterns |
|---------|---------------|------------------|---------------|---------------|-------------|---------------|------------|-------------------|
| `gold_standard.md` | Type 1, Standard | L4 | ~95 | Exemplary | 0 | 0 | 0–1 | None |
| `partial_conformance.md` | Type 1, Minimal | L2 | ~72 | Strong | 0 | 4–5 | 2–3 | None |
| `minimal_valid.md` | Type 1, Minimal | L0 | ~35 | Needs Work | 0 | 5–6 | 3 | AP-CONT-004 (Link Desert) |
| `non_conformant.md` | Type 1, Minimal | Fails L1 | ~18 | Critical | 1 | 7–9 | 3 | AP-STRUCT-002, AP-STRUCT-005, AP-CONT-002, AP-CONT-004, AP-CONT-007 |
| `type_2_full_excerpt.md` | Type 2, varies | N/A | N/A | N/A | 0 | 0 | 1 (I005) | N/A (Type 2 rules) |

**Important:** These are *expected* values that the future parser + validator (v0.2.x–v0.3.x) should produce when processing these fixtures. The v0.1.3 tests validate the **schema models** themselves, not the full pipeline. This matrix exists for forward traceability — when the parser and validator are built, their integration tests should assert these exact expectations.

---

## Design Decisions Applied

| ID | Decision | How Applied in v0.1.3 |
|----|----------|----------------------|
| DECISION-001 | Markdown over JSON/YAML | All test fixtures are Markdown files, not YAML — matching the validation engine's actual input format |
| DECISION-006 | Pydantic for Validation | conftest.py factories construct Pydantic v2 models; tests validate field constraints and computed properties |
| DECISION-012 | Canonical Section Names | gold_standard.md uses all 11 canonical names in correct order; partial_conformance.md uses aliases; non_conformant.md uses non-canonical names |
| DECISION-013 | Token Budget Tiers | Fixtures span multiple size tiers; type_2_full_generated_content factory produces a file exceeding the 256 KB boundary |
| DECISION-016 | 4-Category Anti-Patterns | non_conformant.md triggers patterns across structural, content, and strategic categories; test_constants.py validates all 22 registry entries |
| NFR-010 | ≥80% Test Coverage | Seven test modules cover all seven schema files; parametrized tests maximize code path coverage |

---

## Exit Criteria

- [ ] All 5 fixture files created in `tests/fixtures/`
- [ ] `conftest.py` defines fixture loaders for all 5 fixtures + generated Type 2 content
- [ ] `conftest.py` defines factory fixtures for all major schema models
- [ ] 7 test modules created (one per schema file from v0.1.2)
- [ ] All tests pass: `pytest tests/ -v` returns 0
- [ ] Test coverage ≥ 80% on `src/docstratum/schema/`: `pytest --cov=src/docstratum/schema tests/`
- [ ] gold_standard.md has H1, blockquote, 9+ canonical sections, links with descriptions, code examples
- [ ] non_conformant.md triggers at least 1 error code and 5+ warning codes
- [ ] type_2_full_generated_content factory produces content > 256 KB
- [ ] No test depends on external network access or file system state outside `tests/`

---

## Traceability Appendix

### Research Artifacts Consumed

| Artifact | What This Spec Uses From It |
|----------|-----------------------------|
| v0.0.1a (Formal Grammar) | ABNF grammar structure → fixture Markdown format; error codes → expected diagnostics per fixture |
| v0.0.2c (Ecosystem Audit) | Specimen archetypes (Svelte, Cursor, NVIDIA) → fixture quality calibration; 55% blockquote compliance → W001 fixture design |
| v0.0.4a (Structural Checks) | 20 structural checks → gold_standard exercising all; E001–E008 → non_conformant triggering subset |
| v0.0.4b (Content Checks) | Quality predictors → fixture content design (code examples, descriptions); gold standard scores → fixture score calibration |
| v0.0.4c (Anti-Patterns) | 22 patterns → non_conformant triggering 5+; CHECK IDs → test_constants validation |
| v0.0.5a (Functional Reqs) | FR-001, FR-002, FR-003, FR-004, FR-007, FR-008, FR-011 → schema model tests |
| v0.0.5b (Non-Functional Reqs) | NFR-010 (≥80% coverage) → test coverage target; NFR-006 (clear CLI errors) → diagnostic message tests |
| v0.1.2 (Schema Definition) | All 7 schema files → all 7 test modules + all factory fixtures in conftest.py |

### FR-to-Test Mapping

| FR ID | Description | Test Coverage |
|-------|------------|---------------|
| FR-001 | Pydantic models for base structure | `test_parsed.py`: ParsedLlmsTxt, ParsedSection, ParsedLink, ParsedBlockquote |
| FR-002 | Extended schema fields | `test_enrichment.py`: Concept, FewShotExample, LLMInstruction, Metadata |
| FR-003 | 5-level validation pipeline | `test_validation.py`: ValidationLevel, ValidationResult.levels_passed |
| FR-004 | Error reporting with line numbers | `test_validation.py`: ValidationDiagnostic.line_number, .context |
| FR-007 | Quality assessment framework | `test_quality.py`: QualityScore, DimensionScore, QualityGrade.from_score() |
| FR-008 | Error code registry | `test_diagnostics.py`: DiagnosticCode (26 codes), severity derivation, message/remediation |
| FR-011 | Schema round-trip serialization | `test_parsed.py`: model construction → property access → re-verification |
