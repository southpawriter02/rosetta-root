# v0.1.2 — Schema Definition: Validation Engine Models

> **Phase:** Foundation (v0.1.x)
> **Status:** DRAFT — Realigned to validation-engine pivot (2026-02-06)
> **Parent:** [v0.1.0 — Project Foundation](RR-SPEC-v0.1.0-project-foundation.md)
> **Goal:** Define the complete Pydantic v2 model hierarchy for the DocStratum validation engine — parsed document models, validation result models, quality scoring models, document type classification, error code registry, enrichment schema, and constants.
> **Traces to:** FR-001, FR-002, FR-003, FR-004, FR-007, FR-008 (v0.0.5a); DECISION-001, -002, -003, -004, -005, -006, -012, -013, -015, -016 (v0.0.4d)

---

## What Changed from the Original v0.1.2

The original v0.1.2 defined four Pydantic models (`LlmsTxt`, `CanonicalPage`, `Concept`, `FewShotExample`) that treated llms.txt as a YAML-based data format for generation output. This was the "generation trap" — modeling what a generated file *should* contain rather than what an existing file *actually* contains.

The realigned v0.1.2 defines **seven schema files** across three model categories:

| Category | Purpose | Files | Models |
|----------|---------|-------|--------|
| **Core (what exists)** | Represent a parsed Markdown llms.txt file | `parsed.py`, `classification.py` | `ParsedLlmsTxt`, `ParsedSection`, `ParsedLink`, `ParsedBlockquote`, `DocumentType`, `DocumentClassification`, `SizeTier` |
| **Validation (what the engine reports)** | Represent validation results, diagnostics, and quality scores | `validation.py`, `quality.py`, `diagnostics.py` | `ValidationLevel`, `ValidationDiagnostic`, `ValidationResult`, `QualityDimension`, `QualityGrade`, `QualityScore`, `DimensionScore`, `DiagnosticCode`, `Severity` |
| **Extended (DocStratum enrichment)** | Represent semantic enrichment that DocStratum adds | `enrichment.py`, `constants.py` | `Metadata`, `Concept`, `ConceptRelationship`, `RelationshipType`, `FewShotExample`, `LLMInstruction`, `CanonicalSectionName`, `AntiPatternID`, `AntiPatternCategory` |

**Why this matters:** The validation engine ingests *Markdown*, not YAML. It produces *diagnostics*, not enriched files. The schema must model the entire validation pipeline from input (parsed Markdown) through processing (validation levels, quality scoring) to output (structured results with error codes).

---

## Model Architecture

```
                        ┌─────────────────────┐
                        │   Raw Markdown File  │
                        │   (llms.txt input)   │
                        └──────────┬──────────┘
                                   │
                                   ▼
                    ┌──────────────────────────┐
                    │   classification.py       │
                    │   DocumentClassification  │
                    │   (Type 1 Index or        │
                    │    Type 2 Full?)           │
                    └──────────┬───────────────┘
                               │
                               ▼
                    ┌──────────────────────────┐
                    │   parsed.py               │
                    │   ParsedLlmsTxt           │
                    │   ├── ParsedSection[]     │
                    │   │   └── ParsedLink[]    │
                    │   └── ParsedBlockquote    │
                    └──────────┬───────────────┘
                               │
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
    ┌───────────────┐ ┌───────────────┐ ┌───────────────┐
    │ validation.py │ │  quality.py   │ │ enrichment.py │
    │ L0→L4 levels  │ │ 0-100 score   │ │ Concepts,     │
    │ Diagnostics   │ │ Grades        │ │ Few-shot,     │
    │ Error codes   │ │ Dimensions    │ │ Instructions  │
    └───────┬───────┘ └───────┬───────┘ └───────────────┘
            │                 │
            ▼                 ▼
    ┌─────────────────────────────────┐
    │   diagnostics.py                │
    │   DiagnosticCode enum           │
    │   (8 errors, 11 warnings,       │
    │    7 informational codes)        │
    └─────────────────────────────────┘
            │
            ▼
    ┌─────────────────────────────────┐
    │   constants.py                  │
    │   Canonical section names (11)  │
    │   Token budget tiers (3)        │
    │   Anti-pattern registry (22)    │
    └─────────────────────────────────┘
```

---

## File 1: `src/docstratum/schema/diagnostics.py` — Error Code Registry

This file is listed first because it has no internal dependencies — all other schema files may reference it.

**Traces to:** FR-008 (error code registry), v0.0.1a enrichment (8E/11W/7I codes)

```python
"""Error code registry for the DocStratum validation engine.

Defines the complete diagnostic code catalog derived from the v0.0.1a
enrichment pass. Every validation finding references a DiagnosticCode
that includes the severity level, a human-readable message template,
and a remediation hint.

The code format follows the pattern: {SEVERITY_PREFIX}{NUMBER}
    E001–E008:  Errors (8 codes) — Structural failures that prevent valid parsing
    W001–W011:  Warnings (11 codes) — Deviations from best practices
    I001–I007:  Informational (7 codes) — Observations and suggestions

Research basis:
    v0.0.1a §Error Code Registry (enrichment pass)
    v0.0.4a §Structural Checks (ENC-001/002, STR-001–005, MD-001–003, etc.)
    v0.0.4b §Content Checks (CNT-001–015)
    v0.0.4c §Anti-Pattern Checks (CHECK-001–022)
"""

from enum import StrEnum


class Severity(StrEnum):
    """Diagnostic severity levels.

    Aligned with the three-tier output format mandated by NFR-006
    (clear CLI errors with severity + code + message + remediation).

    Attributes:
        ERROR: Structural failure that prevents valid parsing or breaks spec conformance.
               Maps to validation levels L0–L1 (parseable, structural).
        WARNING: Deviation from best practices that degrades quality but doesn't break parsing.
                 Maps to validation levels L2–L3 (content, best practices).
        INFO: Observation or suggestion for improvement. Non-blocking.
              Maps to validation level L4 (DocStratum extended).
    """

    ERROR = "ERROR"
    WARNING = "WARNING"
    INFO = "INFO"


class DiagnosticCode(StrEnum):
    """Complete diagnostic code catalog for the validation engine.

    Every code has a unique identifier, severity, message template,
    and remediation hint. The code prefix indicates severity:
        E = Error, W = Warning, I = Info

    Codes are organized by validation dimension:
        E001–E008: Structural errors (v0.0.4a checks ENC-001/002, STR-001–005, MD-001, LNK-002)
        W001–W011: Quality warnings (v0.0.4a/b checks NAM-001, CNT-004–009, CNT-015, SIZ-001)
        I001–I007: Informational (v0.0.4b checks CNT-010–014, classification notes)

    Usage:
        from docstratum.schema.diagnostics import DiagnosticCode, Severity

        code = DiagnosticCode.E001_NO_H1_TITLE
        print(code.severity)      # Severity.ERROR
        print(code.message)       # "No H1 title found..."
        print(code.remediation)   # "Add a single H1 title..."
    """

    # ── ERRORS (E001–E008): Structural failures ──────────────────────────
    # These prevent the file from passing L1 (Structural) validation.

    E001_NO_H1_TITLE = "E001"
    """No H1 title found. Every llms.txt file MUST begin with exactly one H1 title.
    Maps to: STR-001 (v0.0.4a). Severity: ERROR.
    Remediation: Add a single '# Title' as the first line of the file."""

    E002_MULTIPLE_H1 = "E002"
    """Multiple H1 titles found. The spec requires exactly one H1.
    Maps to: STR-001 (v0.0.4a). Severity: ERROR.
    Remediation: Remove all but the first H1 title. Use H2 for section headers."""

    E003_INVALID_ENCODING = "E003"
    """File is not valid UTF-8 encoding.
    Maps to: ENC-001 (v0.0.4a). Severity: ERROR.
    Remediation: Convert the file to UTF-8 encoding. Remove any BOM markers."""

    E004_INVALID_LINE_ENDINGS = "E004"
    """File uses non-LF line endings (CR or CRLF detected).
    Maps to: ENC-002 (v0.0.4a). Severity: ERROR.
    Remediation: Convert line endings to LF (Unix-style). Most editors have this option."""

    E005_INVALID_MARKDOWN = "E005"
    """File contains invalid Markdown syntax that prevents parsing.
    Maps to: MD-001 (v0.0.4a). Severity: ERROR.
    Remediation: Fix Markdown syntax errors. Use a Markdown linter to identify issues."""

    E006_BROKEN_LINKS = "E006"
    """Section contains links with empty or malformed URLs.
    Maps to: LNK-002 (v0.0.4a), CHECK-004 (v0.0.4c Ghost File anti-pattern). Severity: ERROR.
    Remediation: Fix or remove links with empty href values. Ensure all URLs are well-formed."""

    E007_EMPTY_FILE = "E007"
    """File is empty or contains only whitespace.
    Maps to: CHECK-001 (v0.0.4c Ghost File anti-pattern). Severity: ERROR.
    Remediation: Add content to the file. At minimum: H1 title, blockquote, one H2 section."""

    E008_EXCEEDS_SIZE_LIMIT = "E008"
    """File exceeds the maximum recommended size (>100K tokens).
    Maps to: SIZ-003 (v0.0.4a), CHECK-003 (v0.0.4c Monolith Monster). Severity: ERROR.
    Remediation: Decompose into a tiered file strategy (index + full + per-section files)."""

    # ── WARNINGS (W001–W011): Quality deviations ────────────────────────
    # These prevent the file from achieving L3 (Best Practices) validation.

    W001_MISSING_BLOCKQUOTE = "W001"
    """No blockquote description found after the H1 title.
    Maps to: STR-002 (v0.0.4a). Severity: WARNING.
    Note: 55% real-world compliance (v0.0.2 enrichment), so this is a warning, not an error.
    Remediation: Add a '> description' blockquote immediately after the H1 title."""

    W002_NON_CANONICAL_SECTION_NAME = "W002"
    """Section name does not match any of the 11 canonical names.
    Maps to: NAM-001 (v0.0.4a). Severity: WARNING.
    Remediation: Use canonical names where possible (see CanonicalSectionName enum)."""

    W003_LINK_MISSING_DESCRIPTION = "W003"
    """Link entry has no description text (bare URL only).
    Maps to: CNT-004 (v0.0.4b), CHECK-010 (v0.0.4c Link Desert). Severity: WARNING.
    Remediation: Add a description after the link: '- [Title](url): Description of the page'."""

    W004_NO_CODE_EXAMPLES = "W004"
    """File contains no code examples (no fenced code blocks found).
    Maps to: CNT-007 (v0.0.4b). Severity: WARNING.
    Note: Code examples are the strongest quality predictor (r ~ 0.65, v0.0.2c).
    Remediation: Add code examples with language specifiers (```python, ```bash, etc.)."""

    W005_CODE_NO_LANGUAGE = "W005"
    """Code block found without a language specifier.
    Maps to: CNT-008 (v0.0.4b). Severity: WARNING.
    Remediation: Add a language identifier after the opening triple backticks."""

    W006_FORMULAIC_DESCRIPTIONS = "W006"
    """Multiple sections use identical or near-identical description patterns.
    Maps to: CNT-005 (v0.0.4b), CHECK-015 (v0.0.4c Formulaic Description). Severity: WARNING.
    Remediation: Write unique, specific descriptions for each section."""

    W007_MISSING_VERSION_METADATA = "W007"
    """No version or last-updated metadata found in the file.
    Maps to: CNT-015 (v0.0.4b). Severity: WARNING.
    Remediation: Add version metadata (e.g., 'Last updated: 2026-02-06')."""

    W008_SECTION_ORDER_NON_CANONICAL = "W008"
    """Sections do not follow the canonical 10-step ordering.
    Maps to: STR-004 (v0.0.4a). Severity: WARNING.
    Remediation: Reorder sections to match canonical sequence (see v0.0.4a §6)."""

    W009_NO_MASTER_INDEX = "W009"
    """No Master Index found as the first H2 section.
    Maps to: STR-003 (v0.0.4a), DECISION-010. Severity: WARNING.
    Note: Files with Master Index achieve 87% vs. 31% LLM success rate.
    Remediation: Add a Master Index as the first H2 section with navigation links."""

    W010_TOKEN_BUDGET_EXCEEDED = "W010"
    """File exceeds the recommended token budget for its tier.
    Maps to: SIZ-001 (v0.0.4a), DECISION-013. Severity: WARNING.
    Remediation: Trim content to stay within the tier's token budget."""

    W011_EMPTY_SECTIONS = "W011"
    """One or more sections contain no meaningful content (placeholder text only).
    Maps to: CHECK-011 (v0.0.4c Blank Canvas anti-pattern). Severity: WARNING.
    Remediation: Add content or remove empty sections. Placeholder sections waste tokens."""

    # ── INFORMATIONAL (I001–I007): Suggestions ──────────────────────────
    # These are non-blocking observations for L4 (DocStratum Extended).

    I001_NO_LLM_INSTRUCTIONS = "I001"
    """No LLM Instructions section found.
    Maps to: CNT-010 (v0.0.4b). Severity: INFO.
    Note: 0% current adoption (v0.0.2), but strongest quality differentiator.
    Remediation: Add an LLM Instructions section with positive/negative directives."""

    I002_NO_CONCEPT_DEFINITIONS = "I002"
    """No structured concept definitions found.
    Maps to: CNT-013 (v0.0.4b). Severity: INFO.
    Remediation: Add concept definitions with IDs, relationships, and aliases."""

    I003_NO_FEW_SHOT_EXAMPLES = "I003"
    """No few-shot Q&A examples found.
    Maps to: v0.0.1b Gap #2 (P0). Severity: INFO.
    Remediation: Add intent-tagged Q&A pairs linked to concepts."""

    I004_RELATIVE_URLS_DETECTED = "I004"
    """Relative URLs found in link entries (may need resolution).
    Maps to: LNK-003 (v0.0.4a). Severity: INFO.
    Remediation: Convert relative URLs to absolute or document the base URL."""

    I005_TYPE_2_FULL_DETECTED = "I005"
    """File classified as Type 2 Full (inline documentation dump, >250 KB).
    Maps to: Document Type Classification (v0.0.1a enrichment). Severity: INFO.
    Note: Type 2 files are not spec-conformant but are valid in MCP contexts.
    Remediation: Consider creating a Type 1 Index companion file."""

    I006_OPTIONAL_SECTIONS_UNMARKED = "I006"
    """Optional sections not explicitly marked with token estimates.
    Maps to: DECISION-011 (v0.0.4d). Severity: INFO.
    Remediation: Mark optional sections so consumers can skip them to save context."""

    I007_JARGON_WITHOUT_DEFINITION = "I007"
    """Domain-specific jargon used without inline definition.
    Maps to: CNT-014 (v0.0.4b). Severity: INFO.
    Remediation: Define jargon inline or link to a concept definition."""

    @property
    def severity(self) -> Severity:
        """Derive severity from the code prefix (E=Error, W=Warning, I=Info)."""
        prefix = self.value[0]
        return {
            "E": Severity.ERROR,
            "W": Severity.WARNING,
            "I": Severity.INFO,
        }[prefix]

    @property
    def code_number(self) -> int:
        """Extract the numeric portion of the code (e.g., E001 -> 1)."""
        return int(self.value[1:])

    @property
    def message(self) -> str:
        """Return the first line of the docstring as the message template."""
        doc = self.__class__.__dict__[self.name].__doc__
        if doc:
            return doc.strip().split("\n")[0]
        return f"Diagnostic {self.value}"

    @property
    def remediation(self) -> str:
        """Extract the remediation hint from the docstring."""
        doc = self.__class__.__dict__[self.name].__doc__
        if doc:
            for line in doc.strip().split("\n"):
                stripped = line.strip()
                if stripped.startswith("Remediation:"):
                    return stripped[len("Remediation:"):].strip()
        return "No remediation available."
```

---

## File 2: `src/docstratum/schema/constants.py` — Canonical Names, Tiers, Anti-Patterns

**Traces to:** DECISION-012 (canonical section names), DECISION-013 (token budget tiers), DECISION-016 (anti-pattern classification), v0.0.2c (450+ project frequency analysis), v0.0.4a (token budget architecture), v0.0.4c (22 anti-patterns)

```python
"""Constants for the DocStratum validation engine.

Canonical section names (11), token budget tier definitions (3),
and the anti-pattern registry (22 patterns across 4 categories).

All values are derived from empirical research:
    - Section names: frequency analysis of 450+ projects (v0.0.2c, DECISION-012)
    - Token budgets: specimen analysis + gold standard calibration (v0.0.4a, DECISION-013)
    - Anti-patterns: 18 audited implementations + ecosystem survey (v0.0.4c, DECISION-016)
"""

from enum import StrEnum
from typing import NamedTuple


# ── Canonical Section Names ─────────────────────────────────────────────
# Derived from frequency analysis of 450+ llms.txt projects (v0.0.2c).
# The 10-step mandatory ordering sequence (v0.0.4a §6) uses these names.
# Non-canonical names trigger W002_NON_CANONICAL_SECTION_NAME.


class CanonicalSectionName(StrEnum):
    """The 11 standard section names validated across 450+ projects.

    These are ordered by the canonical 10-step sequence (v0.0.4a §6).
    Some names are aliases or variants of the same logical section;
    the validator normalizes to the primary name.

    Usage:
        from docstratum.schema.constants import CanonicalSectionName

        if section_name.lower() in CanonicalSectionName.values():
            # Known canonical name
    """

    MASTER_INDEX = "Master Index"
    LLM_INSTRUCTIONS = "LLM Instructions"
    GETTING_STARTED = "Getting Started"
    CORE_CONCEPTS = "Core Concepts"
    API_REFERENCE = "API Reference"
    EXAMPLES = "Examples"
    CONFIGURATION = "Configuration"
    ADVANCED_TOPICS = "Advanced Topics"
    TROUBLESHOOTING = "Troubleshooting"
    FAQ = "FAQ"
    OPTIONAL = "Optional"


# Mapping of common aliases to canonical names for normalization.
# The validator uses this to recognize non-standard but equivalent names.
SECTION_NAME_ALIASES: dict[str, CanonicalSectionName] = {
    "table of contents": CanonicalSectionName.MASTER_INDEX,
    "toc": CanonicalSectionName.MASTER_INDEX,
    "index": CanonicalSectionName.MASTER_INDEX,
    "docs": CanonicalSectionName.MASTER_INDEX,
    "documentation": CanonicalSectionName.MASTER_INDEX,
    "instructions": CanonicalSectionName.LLM_INSTRUCTIONS,
    "agent instructions": CanonicalSectionName.LLM_INSTRUCTIONS,
    "quickstart": CanonicalSectionName.GETTING_STARTED,
    "quick start": CanonicalSectionName.GETTING_STARTED,
    "installation": CanonicalSectionName.GETTING_STARTED,
    "setup": CanonicalSectionName.GETTING_STARTED,
    "concepts": CanonicalSectionName.CORE_CONCEPTS,
    "key concepts": CanonicalSectionName.CORE_CONCEPTS,
    "fundamentals": CanonicalSectionName.CORE_CONCEPTS,
    "api": CanonicalSectionName.API_REFERENCE,
    "reference": CanonicalSectionName.API_REFERENCE,
    "endpoints": CanonicalSectionName.API_REFERENCE,
    "usage": CanonicalSectionName.EXAMPLES,
    "use cases": CanonicalSectionName.EXAMPLES,
    "tutorials": CanonicalSectionName.EXAMPLES,
    "recipes": CanonicalSectionName.EXAMPLES,
    "config": CanonicalSectionName.CONFIGURATION,
    "settings": CanonicalSectionName.CONFIGURATION,
    "options": CanonicalSectionName.CONFIGURATION,
    "advanced": CanonicalSectionName.ADVANCED_TOPICS,
    "internals": CanonicalSectionName.ADVANCED_TOPICS,
    "debugging": CanonicalSectionName.TROUBLESHOOTING,
    "common issues": CanonicalSectionName.TROUBLESHOOTING,
    "known issues": CanonicalSectionName.TROUBLESHOOTING,
    "frequently asked questions": CanonicalSectionName.FAQ,
    "supplementary": CanonicalSectionName.OPTIONAL,
    "appendix": CanonicalSectionName.OPTIONAL,
    "extras": CanonicalSectionName.OPTIONAL,
}

# Canonical section ordering (position in the 10-step sequence)
CANONICAL_SECTION_NAMES: dict[CanonicalSectionName, int] = {
    CanonicalSectionName.MASTER_INDEX: 1,
    CanonicalSectionName.LLM_INSTRUCTIONS: 2,
    CanonicalSectionName.GETTING_STARTED: 3,
    CanonicalSectionName.CORE_CONCEPTS: 4,
    CanonicalSectionName.API_REFERENCE: 5,
    CanonicalSectionName.EXAMPLES: 6,
    CanonicalSectionName.CONFIGURATION: 7,
    CanonicalSectionName.ADVANCED_TOPICS: 8,
    CanonicalSectionName.TROUBLESHOOTING: 9,
    CanonicalSectionName.FAQ: 10,
    # OPTIONAL has no fixed position — always last
}


# ── Token Budget Tiers ──────────────────────────────────────────────────
# Three enforced tiers with per-section allocations (DECISION-013).
# Files exceeding their tier budget trigger W010_TOKEN_BUDGET_EXCEEDED.


class TokenBudgetTier(NamedTuple):
    """Token budget tier definition.

    Attributes:
        name: Tier name for display.
        min_tokens: Lower bound of the token range (inclusive).
        max_tokens: Upper bound of the token range (inclusive).
        use_case: Description of when this tier applies.
        file_strategy: Recommended file organization (single, dual, multi).
    """

    name: str
    min_tokens: int
    max_tokens: int
    use_case: str
    file_strategy: str


TOKEN_BUDGET_TIERS: dict[str, TokenBudgetTier] = {
    "standard": TokenBudgetTier(
        name="Standard",
        min_tokens=1_500,
        max_tokens=4_500,
        use_case="Small projects, <100 pages, <5 features",
        file_strategy="single",
    ),
    "comprehensive": TokenBudgetTier(
        name="Comprehensive",
        min_tokens=4_500,
        max_tokens=12_000,
        use_case="Medium projects, 100–500 pages, 5–20 features",
        file_strategy="dual (index + full)",
    ),
    "full": TokenBudgetTier(
        name="Full",
        min_tokens=12_000,
        max_tokens=50_000,
        use_case="Large projects, 500+ pages, 20+ features",
        file_strategy="multi (master + per-service)",
    ),
}

# Anti-pattern thresholds (v0.0.4a §Token Budget Architecture)
TOKEN_ZONE_OPTIMAL = 20_000  # No decomposition needed
TOKEN_ZONE_GOOD = 50_000  # Consider dual-file strategy
TOKEN_ZONE_DEGRADATION = 100_000  # Tiering strongly recommended
TOKEN_ZONE_ANTI_PATTERN = 500_000  # Exceeds all current context windows


# ── Anti-Pattern Registry ───────────────────────────────────────────────
# 22 named patterns across 4 severity categories (DECISION-016, v0.0.4c).


class AntiPatternCategory(StrEnum):
    """Anti-pattern severity categories (DECISION-016).

    The four categories map to the composite scoring pipeline:
        CRITICAL: Gate the structural score (cap total at 29)
        STRUCTURAL: Reduce the structural dimension
        CONTENT: Reduce the content dimension
        STRATEGIC: Deduction-based penalties
    """

    CRITICAL = "critical"
    STRUCTURAL = "structural"
    CONTENT = "content"
    STRATEGIC = "strategic"


class AntiPatternID(StrEnum):
    """All 22 anti-patterns cataloged in v0.0.4c.

    Format: AP-{CATEGORY}-{NUMBER}
    Each maps to a CHECK-{NNN} automated detection rule.
    """

    # Critical (4) — prevent LLM consumption entirely
    AP_CRIT_001 = "AP-CRIT-001"  # Ghost File
    AP_CRIT_002 = "AP-CRIT-002"  # Structure Chaos
    AP_CRIT_003 = "AP-CRIT-003"  # Encoding Disaster
    AP_CRIT_004 = "AP-CRIT-004"  # Link Void

    # Structural (5) — break navigation
    AP_STRUCT_001 = "AP-STRUCT-001"  # Sitemap Dump
    AP_STRUCT_002 = "AP-STRUCT-002"  # Orphaned Sections
    AP_STRUCT_003 = "AP-STRUCT-003"  # Duplicate Identity
    AP_STRUCT_004 = "AP-STRUCT-004"  # Section Shuffle
    AP_STRUCT_005 = "AP-STRUCT-005"  # Naming Nebula

    # Content (9) — degrade quality
    AP_CONT_001 = "AP-CONT-001"  # Copy-Paste Plague
    AP_CONT_002 = "AP-CONT-002"  # Blank Canvas
    AP_CONT_003 = "AP-CONT-003"  # Jargon Jungle
    AP_CONT_004 = "AP-CONT-004"  # Link Desert
    AP_CONT_005 = "AP-CONT-005"  # Outdated Oracle
    AP_CONT_006 = "AP-CONT-006"  # Example Void
    AP_CONT_007 = "AP-CONT-007"  # Formulaic Description
    AP_CONT_008 = "AP-CONT-008"  # Silent Agent
    AP_CONT_009 = "AP-CONT-009"  # Versionless Drift

    # Strategic (4) — undermine long-term value
    AP_STRAT_001 = "AP-STRAT-001"  # Automation Obsession
    AP_STRAT_002 = "AP-STRAT-002"  # Monolith Monster
    AP_STRAT_003 = "AP-STRAT-003"  # Meta-Documentation Spiral
    AP_STRAT_004 = "AP-STRAT-004"  # Preference Trap


class AntiPatternEntry(NamedTuple):
    """Registry entry for an anti-pattern.

    Attributes:
        id: Unique anti-pattern identifier.
        name: Human-readable name.
        category: Severity category (critical/structural/content/strategic).
        check_id: Corresponding CHECK-NNN from v0.0.4c.
        description: One-line description of the pattern.
    """

    id: AntiPatternID
    name: str
    category: AntiPatternCategory
    check_id: str
    description: str


ANTI_PATTERN_REGISTRY: list[AntiPatternEntry] = [
    # Critical
    AntiPatternEntry(AntiPatternID.AP_CRIT_001, "Ghost File", AntiPatternCategory.CRITICAL, "CHECK-001", "Empty or near-empty file that exists but provides no value"),
    AntiPatternEntry(AntiPatternID.AP_CRIT_002, "Structure Chaos", AntiPatternCategory.CRITICAL, "CHECK-002", "File lacks recognizable Markdown structure (no headers, no sections)"),
    AntiPatternEntry(AntiPatternID.AP_CRIT_003, "Encoding Disaster", AntiPatternCategory.CRITICAL, "CHECK-003", "Non-UTF-8 encoding or mixed line endings that break parsers"),
    AntiPatternEntry(AntiPatternID.AP_CRIT_004, "Link Void", AntiPatternCategory.CRITICAL, "CHECK-004", "All or most links are broken, empty, or malformed"),
    # Structural
    AntiPatternEntry(AntiPatternID.AP_STRUCT_001, "Sitemap Dump", AntiPatternCategory.STRUCTURAL, "CHECK-005", "Entire sitemap dumped as flat link list with no organization"),
    AntiPatternEntry(AntiPatternID.AP_STRUCT_002, "Orphaned Sections", AntiPatternCategory.STRUCTURAL, "CHECK-006", "Sections with headers but no links or content"),
    AntiPatternEntry(AntiPatternID.AP_STRUCT_003, "Duplicate Identity", AntiPatternCategory.STRUCTURAL, "CHECK-007", "Multiple sections with identical or near-identical names"),
    AntiPatternEntry(AntiPatternID.AP_STRUCT_004, "Section Shuffle", AntiPatternCategory.STRUCTURAL, "CHECK-008", "Sections in illogical order (e.g., Advanced before Getting Started)"),
    AntiPatternEntry(AntiPatternID.AP_STRUCT_005, "Naming Nebula", AntiPatternCategory.STRUCTURAL, "CHECK-009", "Section names that are vague, inconsistent, or non-standard"),
    # Content
    AntiPatternEntry(AntiPatternID.AP_CONT_001, "Copy-Paste Plague", AntiPatternCategory.CONTENT, "CHECK-010", "Large blocks of content duplicated from other sources without curation"),
    AntiPatternEntry(AntiPatternID.AP_CONT_002, "Blank Canvas", AntiPatternCategory.CONTENT, "CHECK-011", "Sections with placeholder text or no meaningful content"),
    AntiPatternEntry(AntiPatternID.AP_CONT_003, "Jargon Jungle", AntiPatternCategory.CONTENT, "CHECK-012", "Heavy use of domain jargon without definitions"),
    AntiPatternEntry(AntiPatternID.AP_CONT_004, "Link Desert", AntiPatternCategory.CONTENT, "CHECK-013", "Links without descriptions (bare URL lists)"),
    AntiPatternEntry(AntiPatternID.AP_CONT_005, "Outdated Oracle", AntiPatternCategory.CONTENT, "CHECK-014", "Content references deprecated or outdated information"),
    AntiPatternEntry(AntiPatternID.AP_CONT_006, "Example Void", AntiPatternCategory.CONTENT, "CHECK-015", "No code examples despite being a technical project"),
    AntiPatternEntry(AntiPatternID.AP_CONT_007, "Formulaic Description", AntiPatternCategory.CONTENT, "CHECK-019", "Auto-generated descriptions with identical patterns (Mintlify risk)"),
    AntiPatternEntry(AntiPatternID.AP_CONT_008, "Silent Agent", AntiPatternCategory.CONTENT, "CHECK-020", "No LLM-facing guidance despite being an AI documentation file"),
    AntiPatternEntry(AntiPatternID.AP_CONT_009, "Versionless Drift", AntiPatternCategory.CONTENT, "CHECK-021", "No version or date metadata, impossible to assess freshness"),
    # Strategic
    AntiPatternEntry(AntiPatternID.AP_STRAT_001, "Automation Obsession", AntiPatternCategory.STRATEGIC, "CHECK-016", "Fully auto-generated with no human curation or review"),
    AntiPatternEntry(AntiPatternID.AP_STRAT_002, "Monolith Monster", AntiPatternCategory.STRATEGIC, "CHECK-017", "Single file exceeding 100K tokens with no decomposition"),
    AntiPatternEntry(AntiPatternID.AP_STRAT_003, "Meta-Documentation Spiral", AntiPatternCategory.STRATEGIC, "CHECK-018", "File documents itself or the llms.txt standard rather than the project"),
    AntiPatternEntry(AntiPatternID.AP_STRAT_004, "Preference Trap", AntiPatternCategory.STRATEGIC, "CHECK-022", "Content crafted to manipulate LLM behavior (trust laundering)"),
]
```

### Cross-Reference: Validation Check IDs → Diagnostic Codes → Anti-Pattern IDs

The validation engine uses three overlapping ID systems inherited from different research phases. This mapping table makes the relationships explicit so that any finding can be traced from its v0.0.4a/b/c check ID through the diagnostic code it triggers to the anti-pattern it may indicate.

**How to read this table:** A validation check (left column) produces a diagnostic code (middle column) when it fires. Some diagnostics also indicate an anti-pattern (right column) — anti-patterns are quality scoring deductions, while diagnostic codes are validation pipeline outputs. Not every diagnostic code has a corresponding anti-pattern, and not every anti-pattern has a single diagnostic code trigger.

| v0.0.4a/b/c Check ID | DiagnosticCode | Anti-Pattern ID | Relationship |
|-----------------------|----------------|-----------------|-------------|
| **Structural checks (v0.0.4a)** | | | |
| ENC-001 (UTF-8 encoding) | E003_INVALID_ENCODING | AP-CRIT-003 (Encoding Disaster) | Encoding failure → error + critical anti-pattern |
| ENC-002 (LF line endings) | E004_INVALID_LINE_ENDINGS | AP-CRIT-003 (Encoding Disaster) | Line ending failure → error + critical anti-pattern |
| STR-001 (single H1 title) | E001_NO_H1_TITLE, E002_MULTIPLE_H1 | AP-CRIT-002 (Structure Chaos) | H1 violations → error + critical anti-pattern |
| STR-002 (blockquote present) | W001_MISSING_BLOCKQUOTE | — | Warning only; 55% compliance makes it non-gating |
| STR-003 (Master Index first) | W009_NO_MASTER_INDEX | — | Warning; DECISION-010 justifies priority |
| STR-004 (section ordering) | W008_SECTION_ORDER_NON_CANONICAL | AP-STRUCT-004 (Section Shuffle) | Non-canonical order → warning + structural anti-pattern |
| MD-001 (valid Markdown syntax) | E005_INVALID_MARKDOWN | AP-CRIT-002 (Structure Chaos) | Parse failure → error + critical anti-pattern |
| LNK-002 (well-formed URLs) | E006_BROKEN_LINKS | AP-CRIT-004 (Link Void) | Broken links → error + critical anti-pattern |
| LNK-003 (absolute vs. relative) | I004_RELATIVE_URLS_DETECTED | — | Informational only |
| NAM-001 (canonical names) | W002_NON_CANONICAL_SECTION_NAME | AP-STRUCT-005 (Naming Nebula) | Non-canonical → warning + structural anti-pattern |
| SIZ-001 (token budget tier) | W010_TOKEN_BUDGET_EXCEEDED | AP-STRAT-002 (Monolith Monster) | Budget exceeded → warning; extreme cases → strategic anti-pattern |
| SIZ-003 (max size 100K tokens) | E008_EXCEEDS_SIZE_LIMIT | AP-STRAT-002 (Monolith Monster) | Hard limit → error + strategic anti-pattern |
| **Content checks (v0.0.4b)** | | | |
| CNT-004 (link descriptions) | W003_LINK_MISSING_DESCRIPTION | AP-CONT-004 (Link Desert) | Missing descriptions → warning + content anti-pattern |
| CNT-005 (unique descriptions) | W006_FORMULAIC_DESCRIPTIONS | AP-CONT-007 (Formulaic Description) | Repetitive patterns → warning + content anti-pattern |
| CNT-007 (code examples present) | W004_NO_CODE_EXAMPLES | AP-CONT-006 (Example Void) | No code → warning + content anti-pattern |
| CNT-008 (language specifiers) | W005_CODE_NO_LANGUAGE | — | Warning only; no dedicated anti-pattern |
| CNT-010 (LLM Instructions) | I001_NO_LLM_INSTRUCTIONS | AP-CONT-008 (Silent Agent) | Missing instructions → info + content anti-pattern |
| CNT-013 (concept definitions) | I002_NO_CONCEPT_DEFINITIONS | — | Informational only; L4 extended feature |
| CNT-014 (jargon definitions) | I007_JARGON_WITHOUT_DEFINITION | AP-CONT-003 (Jargon Jungle) | Undefined jargon → info + content anti-pattern |
| CNT-015 (version metadata) | W007_MISSING_VERSION_METADATA | AP-CONT-009 (Versionless Drift) | No version → warning + content anti-pattern |
| **Anti-pattern checks (v0.0.4c)** | | | |
| CHECK-001 (Ghost File) | E007_EMPTY_FILE | AP-CRIT-001 (Ghost File) | Empty file → error + critical anti-pattern |
| CHECK-002 (Structure Chaos) | E001/E002/E005 (structural errors) | AP-CRIT-002 (Structure Chaos) | Aggregate of multiple structural errors |
| CHECK-003 (Encoding Disaster) | E003/E004 (encoding errors) | AP-CRIT-003 (Encoding Disaster) | Aggregate of encoding errors |
| CHECK-004 (Link Void) | E006_BROKEN_LINKS | AP-CRIT-004 (Link Void) | All/most links broken |
| CHECK-005 (Sitemap Dump) | — (detected by link-to-section ratio heuristic) | AP-STRUCT-001 (Sitemap Dump) | No dedicated diagnostic; heuristic-based |
| CHECK-006 (Orphaned Sections) | W011_EMPTY_SECTIONS | AP-STRUCT-002 (Orphaned Sections) | Empty sections → warning + structural anti-pattern |
| CHECK-007 (Duplicate Identity) | — (detected by name similarity analysis) | AP-STRUCT-003 (Duplicate Identity) | No dedicated diagnostic; heuristic-based |
| CHECK-008 (Section Shuffle) | W008_SECTION_ORDER_NON_CANONICAL | AP-STRUCT-004 (Section Shuffle) | Same as STR-004 |
| CHECK-009 (Naming Nebula) | W002_NON_CANONICAL_SECTION_NAME | AP-STRUCT-005 (Naming Nebula) | Same as NAM-001 |
| CHECK-010 (Copy-Paste Plague) | — (detected by content similarity analysis) | AP-CONT-001 (Copy-Paste Plague) | No dedicated diagnostic; heuristic-based |
| CHECK-011 (Blank Canvas) | W011_EMPTY_SECTIONS | AP-CONT-002 (Blank Canvas) | Same diagnostic as CHECK-006, different anti-pattern |
| CHECK-012 (Jargon Jungle) | I007_JARGON_WITHOUT_DEFINITION | AP-CONT-003 (Jargon Jungle) | Same as CNT-014 |
| CHECK-013 (Link Desert) | W003_LINK_MISSING_DESCRIPTION | AP-CONT-004 (Link Desert) | Same as CNT-004 |
| CHECK-014 (Outdated Oracle) | — (detected by date heuristic) | AP-CONT-005 (Outdated Oracle) | No dedicated diagnostic; requires date parsing |
| CHECK-015 (Example Void) | W004_NO_CODE_EXAMPLES | AP-CONT-006 (Example Void) | Same as CNT-007 |
| CHECK-016 (Automation Obsession) | — (detected by signature patterns) | AP-STRAT-001 (Automation Obsession) | No dedicated diagnostic; Mintlify/Yoast signatures |
| CHECK-017 (Monolith Monster) | E008_EXCEEDS_SIZE_LIMIT | AP-STRAT-002 (Monolith Monster) | Same as SIZ-003 |
| CHECK-018 (Meta-Documentation Spiral) | — (detected by self-referential content) | AP-STRAT-003 (Meta-Documentation Spiral) | No dedicated diagnostic; content analysis |
| CHECK-019 (Formulaic Description) | W006_FORMULAIC_DESCRIPTIONS | AP-CONT-007 (Formulaic Description) | Same as CNT-005 |
| CHECK-020 (Silent Agent) | I001_NO_LLM_INSTRUCTIONS | AP-CONT-008 (Silent Agent) | Same as CNT-010 |
| CHECK-021 (Versionless Drift) | W007_MISSING_VERSION_METADATA | AP-CONT-009 (Versionless Drift) | Same as CNT-015 |
| CHECK-022 (Preference Trap) | — (detected by manipulation patterns) | AP-STRAT-004 (Preference Trap) | No dedicated diagnostic; behavioral analysis |

**Key observations:**

- **26 diagnostic codes** map to **22 anti-patterns** through **~35 validation check IDs** — a many-to-many relationship.
- **7 anti-patterns have no dedicated diagnostic code** (CHECK-005, -007, -010, -014, -016, -018, -022). These require heuristic detection logic that will be implemented in v0.2.4. Their anti-pattern registry entries exist in v0.1.2, but their detection is deferred.
- **Some diagnostics map to multiple anti-patterns** (e.g., W011 maps to both Orphaned Sections and Blank Canvas — same symptom, different severity interpretation depending on whether the section has a header vs. placeholder text).
- The v0.0.4a/b check IDs (structural prefix format like `STR-001`, `CNT-004`) represent the original research taxonomy. The CHECK-NNN IDs (v0.0.4c) represent the anti-pattern catalog. Both systems are preserved in the diagnostic code docstrings for full traceability.

---

## File 3: `src/docstratum/schema/classification.py` — Document Type Classification

**Traces to:** Finding 4 (bimodal distribution), v0.0.1a enrichment (Type 1 vs. Type 2), DECISION-013 (token budget tiers)

```python
"""Document type classification models for the DocStratum validation engine.

The research (v0.0.1a enrichment) established that llms.txt files fall into
two distinct types with a bimodal distribution and no overlap zone:

    Type 1 Index: Curated link catalogs (1.1 KB – 225 KB, 80–100% conformance)
    Type 2 Full: Inline documentation dumps (1.3 MB – 25 MB, 5–15% conformance)

The ~250 KB boundary serves as the classification heuristic threshold.
Classification happens BEFORE validation because different document types
receive different validation rule sets.
"""

from datetime import datetime
from enum import StrEnum

from pydantic import BaseModel, Field


class DocumentType(StrEnum):
    """Document type classification.

    Based on the bimodal distribution observed across 11 specimens (v0.0.1a).
    No specimens exist in the 225 KB – 1.3 MB range, confirming a natural boundary.

    Attributes:
        TYPE_1_INDEX: Curated link catalog following the spec's intended format.
                      Receives full ABNF-based structural validation.
        TYPE_2_FULL: Inline documentation dump (llms-full.txt convention).
                     Receives size-appropriate diagnostics only.
        UNKNOWN: Classification could not be determined (e.g., empty file).
    """

    TYPE_1_INDEX = "type_1_index"
    TYPE_2_FULL = "type_2_full"
    UNKNOWN = "unknown"


class SizeTier(StrEnum):
    """Token budget size tier (DECISION-013).

    Files are assigned to tiers based on estimated token count.
    Each tier has recommended token budgets and file strategies.

    Attributes:
        MINIMAL: Under 1,500 tokens. Very small files (stubs, placeholders).
        STANDARD: 1,500–4,500 tokens. Small projects, <100 pages.
        COMPREHENSIVE: 4,500–12,000 tokens. Medium projects, 100–500 pages.
        FULL: 12,000–50,000 tokens. Large projects, 500+ pages.
        OVERSIZED: Over 50,000 tokens. Exceeds recommended limits.
    """

    MINIMAL = "minimal"
    STANDARD = "standard"
    COMPREHENSIVE = "comprehensive"
    FULL = "full"
    OVERSIZED = "oversized"


class DocumentClassification(BaseModel):
    """Result of classifying an llms.txt file before validation.

    The classifier runs first in the pipeline, determining which
    validation rules to apply and which token budget tier to enforce.

    Attributes:
        document_type: Type 1 Index or Type 2 Full (or Unknown).
        size_bytes: Raw file size in bytes.
        estimated_tokens: Approximate token count (bytes / 4 heuristic).
        size_tier: Assigned token budget tier.
        filename: Original filename (for display/logging).
        classified_at: Timestamp of classification.

    Example:
        classification = DocumentClassification(
            document_type=DocumentType.TYPE_1_INDEX,
            size_bytes=19_456,
            estimated_tokens=4_864,
            size_tier=SizeTier.COMPREHENSIVE,
            filename="llms.txt",
        )
    """

    document_type: DocumentType = Field(
        description="Whether this file is a Type 1 Index or Type 2 Full document."
    )
    size_bytes: int = Field(ge=0, description="Raw file size in bytes.")
    estimated_tokens: int = Field(
        ge=0,
        description="Approximate token count. Heuristic: bytes / 4.",
    )
    size_tier: SizeTier = Field(
        description="Token budget tier based on estimated token count.",
    )
    filename: str = Field(
        default="llms.txt",
        description="Original filename for logging and display.",
    )
    classified_at: datetime = Field(
        default_factory=datetime.now,
        description="When classification was performed.",
    )

    # ── Classification boundaries ────────────────────────────────────
    # These are class-level constants used by the classifier logic
    # (implemented in v0.3.1, but defined here for schema reference).

    TYPE_BOUNDARY_BYTES: int = 256_000  # ~250 KB — the bimodal gap
    """Byte threshold separating Type 1 from Type 2.
    Files above this are classified as Type 2 Full.
    Derived from: largest Type 1 specimen = Cloudflare at 225 KB;
    smallest Type 2 specimen = Vercel AI SDK at 1.3 MB.
    """
```

---

## File 4: `src/docstratum/schema/parsed.py` — Parsed Document Models

**Traces to:** FR-001 (Pydantic models for base llms.txt structure), v0.0.1a (ABNF grammar), DECISION-001 (Markdown format)

```python
"""Parsed document models for the DocStratum validation engine.

These models represent the PARSED structure of an llms.txt Markdown file —
what the parser (v0.3.1) produces after reading raw Markdown input.
They are the canonical in-memory representation of an llms.txt file.

The parser follows the "permissive input, strict output" principle (v0.0.1a):
    - Missing blockquotes → warning, not error (55% real-world compliance)
    - Relative URLs → info-level note with resolution hint
    - Partial results returned with diagnostic annotations

ABNF grammar reference (v0.0.1a):
    llms-txt   = title CRLF description CRLF *section
    title      = "#" SP title-text CRLF
    description = ">" SP desc-text CRLF
    section    = "##" SP section-name CRLF *entry
    entry      = "-" SP "[" link-title "](" url ")" [": " desc] CRLF
"""

from datetime import datetime
from typing import Optional

from pydantic import BaseModel, Field


class ParsedBlockquote(BaseModel):
    """The blockquote description immediately following the H1 title.

    In the ABNF grammar, this maps to the `description` rule.
    Empirical data: only 55% of real-world files include this (v0.0.2 enrichment).
    Missing blockquotes generate W001, not an error.

    Attributes:
        text: The blockquote text content (without the '>' prefix).
        line_number: Line number where the blockquote starts (1-indexed).
        raw: The original raw text including the '>' prefix.
    """

    text: str = Field(
        description="Cleaned blockquote text (without '>' prefix, stripped).",
    )
    line_number: int = Field(
        ge=1,
        description="Line number in the source file (1-indexed).",
    )
    raw: str = Field(
        default="",
        description="Original raw text including '>' prefix.",
    )


class ParsedLink(BaseModel):
    """A single link entry within an H2 section.

    In the ABNF grammar, this maps to the `entry` rule:
        entry = "-" SP "[" link-title "](" url ")" [": " desc] CRLF

    Attributes:
        title: The link text (content within square brackets).
        url: The URL (content within parentheses).
        description: Optional description after the link (content after ': ').
        line_number: Line number in the source file.
        is_valid_url: Whether the URL appears well-formed (syntactic check only).
    """

    title: str = Field(
        description="Link text from [title](url) format.",
    )
    url: str = Field(
        description="URL from [title](url) format. May be relative or absolute.",
    )
    description: Optional[str] = Field(
        default=None,
        description="Optional description text after ': ' delimiter.",
    )
    line_number: int = Field(
        ge=1,
        description="Line number in the source file (1-indexed).",
    )
    is_valid_url: bool = Field(
        default=True,
        description="Whether the URL passes basic syntactic validation.",
    )


class ParsedSection(BaseModel):
    """An H2 section within the llms.txt file.

    In the ABNF grammar, this maps to the `section` rule:
        section = "##" SP section-name CRLF *entry

    Each section may contain links (the standard format), freeform content
    (common in real-world files), or both.

    Attributes:
        name: The section header text (content after '## ').
        links: List of parsed link entries within this section.
        raw_content: The full raw text content of the section (between this H2 and the next).
        line_number: Line number of the H2 header.
        canonical_name: The matched canonical section name, if any.
        link_count: Number of links in this section (computed).
        estimated_tokens: Approximate token count for this section's content.
    """

    name: str = Field(
        description="Section header text (without '## ' prefix).",
    )
    links: list[ParsedLink] = Field(
        default_factory=list,
        description="Link entries found within this section.",
    )
    raw_content: str = Field(
        default="",
        description="Full raw text of the section (headers, links, prose, code blocks).",
    )
    line_number: int = Field(
        ge=1,
        description="Line number of the H2 header (1-indexed).",
    )
    canonical_name: Optional[str] = Field(
        default=None,
        description="Matched canonical section name, or None if non-canonical.",
    )
    estimated_tokens: int = Field(
        default=0,
        ge=0,
        description="Approximate token count (len(raw_content) / 4 heuristic).",
    )

    @property
    def link_count(self) -> int:
        """Number of links in this section."""
        return len(self.links)

    @property
    def has_code_examples(self) -> bool:
        """Whether this section contains fenced code blocks."""
        return "```" in self.raw_content


class ParsedLlmsTxt(BaseModel):
    """Root model representing a fully parsed llms.txt Markdown file.

    This is the primary output of the parser (v0.3.1) and the primary
    input to the validator (v0.2.4) and quality scorer.

    The model preserves the original file structure while providing
    typed access to all components. It does NOT enforce validity —
    that is the validator's job. A ParsedLlmsTxt can represent
    a partially conformant or even broken file.

    Attributes:
        title: The H1 title text (without '# ' prefix).
        title_line: Line number of the H1 title.
        blockquote: The parsed blockquote description (may be None if missing).
        sections: List of H2 sections in document order.
        raw_content: The complete raw file content (for re-serialization).
        source_filename: Original filename for provenance.
        parsed_at: Timestamp of parsing.

    Example:
        doc = ParsedLlmsTxt(
            title="Stripe Documentation",
            title_line=1,
            blockquote=ParsedBlockquote(text="Stripe API docs", line_number=2),
            sections=[
                ParsedSection(name="Docs", links=[...], line_number=4),
            ],
            raw_content="# Stripe Documentation\\n> Stripe API docs\\n...",
        )

    Traces to: FR-001 (base structure), FR-011 (round-trip serialization)
    """

    title: Optional[str] = Field(
        default=None,
        description="H1 title text. None if no H1 found (triggers E001).",
    )
    title_line: Optional[int] = Field(
        default=None,
        ge=1,
        description="Line number of the H1 title.",
    )
    blockquote: Optional[ParsedBlockquote] = Field(
        default=None,
        description="Blockquote description. None if missing (triggers W001).",
    )
    sections: list[ParsedSection] = Field(
        default_factory=list,
        description="H2 sections in document order.",
    )
    raw_content: str = Field(
        default="",
        description="Complete raw file content for round-trip serialization.",
    )
    source_filename: str = Field(
        default="llms.txt",
        description="Original filename for provenance tracking.",
    )
    parsed_at: datetime = Field(
        default_factory=datetime.now,
        description="Timestamp when parsing completed.",
    )

    @property
    def section_count(self) -> int:
        """Total number of H2 sections."""
        return len(self.sections)

    @property
    def total_links(self) -> int:
        """Total number of links across all sections."""
        return sum(s.link_count for s in self.sections)

    @property
    def estimated_tokens(self) -> int:
        """Approximate total token count (heuristic: chars / 4)."""
        return len(self.raw_content) // 4

    @property
    def has_blockquote(self) -> bool:
        """Whether a blockquote description is present."""
        return self.blockquote is not None

    @property
    def section_names(self) -> list[str]:
        """List of section names in document order."""
        return [s.name for s in self.sections]
```

---

## File 5: `src/docstratum/schema/validation.py` — Validation Result Models

**Traces to:** FR-003 (5-level validation pipeline), FR-004 (error reporting with line numbers), v0.0.1b (validation level definitions)

```python
"""Validation result models for the DocStratum validation engine.

Represents the output of the 5-level validation pipeline (L0–L4).
Each level builds on the previous:

    L0 — Parseable:           File can be read and parsed as Markdown.
    L1 — Structurally Valid:  H1 title exists, sections use H2, links are well-formed.
    L2 — Content Quality:     Descriptions are non-empty, URLs resolve, no placeholders.
    L3 — Best Practices:      Canonical names, Master Index, code examples, token budgets.
    L4 — DocStratum Extended: Concept definitions, few-shot examples, LLM instructions.

Research basis:
    v0.0.1b §Validation Level Definitions
    v0.0.4a §Structural Checks (20 checks → L0, L1)
    v0.0.4b §Content Checks (15 checks → L2, L3)
    v0.0.4c §Anti-Pattern Checks (22 checks → cross-level deductions)
"""

from datetime import datetime
from enum import IntEnum
from typing import Optional

from pydantic import BaseModel, Field

from docstratum.schema.diagnostics import DiagnosticCode, Severity


class ValidationLevel(IntEnum):
    """The 5-level validation pipeline.

    Levels are cumulative — achieving L3 means L0, L1, and L2 also pass.
    The highest level where ALL checks pass is the file's validation level.

    Attributes:
        L0_PARSEABLE: File can be read and parsed as Markdown.
        L1_STRUCTURAL: Basic structural elements present (H1, H2s, links).
        L2_CONTENT: Content quality checks pass (non-empty, resolving).
        L3_BEST_PRACTICES: Best practices followed (canonical names, examples).
        L4_DOCSTRATUM_EXTENDED: DocStratum enrichment present (concepts, few-shot).
    """

    L0_PARSEABLE = 0
    L1_STRUCTURAL = 1
    L2_CONTENT = 2
    L3_BEST_PRACTICES = 3
    L4_DOCSTRATUM_EXTENDED = 4


class ValidationDiagnostic(BaseModel):
    """A single validation finding (error, warning, or info).

    Produced by the validation pipeline for each check that fails or
    triggers a note. Includes the diagnostic code, source location,
    context snippet, and remediation hint.

    Attributes:
        code: The DiagnosticCode enum value (e.g., E001_NO_H1_TITLE).
        severity: Derived from the code prefix (ERROR, WARNING, INFO).
        message: Human-readable description of the finding.
        remediation: Suggested fix.
        line_number: Line in the source file where the issue was found (1-indexed).
        column: Column number if applicable (1-indexed), or None.
        context: Snippet of the surrounding source text for display.
        level: Which validation level this diagnostic belongs to.
        check_id: The v0.0.4 check ID (e.g., "STR-001", "CNT-007").

    Example:
        diagnostic = ValidationDiagnostic(
            code=DiagnosticCode.W001_MISSING_BLOCKQUOTE,
            severity=Severity.WARNING,
            message="No blockquote description found after the H1 title.",
            remediation="Add a '> description' blockquote after the H1.",
            line_number=2,
            level=ValidationLevel.L1_STRUCTURAL,
            check_id="STR-002",
        )
    """

    code: DiagnosticCode = Field(
        description="Diagnostic code from the error code registry.",
    )
    severity: Severity = Field(
        description="ERROR, WARNING, or INFO.",
    )
    message: str = Field(
        description="Human-readable finding description.",
    )
    remediation: str = Field(
        default="",
        description="Suggested fix for this issue.",
    )
    line_number: Optional[int] = Field(
        default=None,
        ge=1,
        description="Source line number (1-indexed). None for file-level issues.",
    )
    column: Optional[int] = Field(
        default=None,
        ge=1,
        description="Source column number (1-indexed). None if not applicable.",
    )
    context: Optional[str] = Field(
        default=None,
        max_length=500,
        description="Source text snippet surrounding the issue.",
    )
    level: ValidationLevel = Field(
        description="Which validation level this diagnostic belongs to.",
    )
    check_id: Optional[str] = Field(
        default=None,
        description="v0.0.4 check ID (e.g., 'STR-001', 'CNT-007', 'CHECK-011').",
    )


class ValidationResult(BaseModel):
    """Complete output of the validation pipeline for a single file.

    Contains all diagnostics, the highest validation level achieved,
    and per-level pass/fail status. This is the primary output model
    of the `docstratum-validate` command.

    Attributes:
        level_achieved: Highest validation level where ALL checks pass.
        diagnostics: All findings (errors, warnings, info) from the pipeline.
        levels_passed: Dict mapping each level to pass/fail status.
        total_errors: Count of ERROR-severity diagnostics.
        total_warnings: Count of WARNING-severity diagnostics.
        total_info: Count of INFO-severity diagnostics.
        validated_at: Timestamp of validation.
        source_filename: File that was validated.

    Example:
        result = ValidationResult(
            level_achieved=ValidationLevel.L1_STRUCTURAL,
            diagnostics=[...],
            levels_passed={
                ValidationLevel.L0_PARSEABLE: True,
                ValidationLevel.L1_STRUCTURAL: True,
                ValidationLevel.L2_CONTENT: False,
                ValidationLevel.L3_BEST_PRACTICES: False,
                ValidationLevel.L4_DOCSTRATUM_EXTENDED: False,
            },
        )

    Traces to: FR-003 (5-level pipeline), FR-004 (error reporting)
    """

    level_achieved: ValidationLevel = Field(
        description="Highest level where all checks pass.",
    )
    diagnostics: list[ValidationDiagnostic] = Field(
        default_factory=list,
        description="All validation findings.",
    )
    levels_passed: dict[ValidationLevel, bool] = Field(
        default_factory=lambda: {level: False for level in ValidationLevel},
        description="Per-level pass/fail status.",
    )
    validated_at: datetime = Field(
        default_factory=datetime.now,
        description="When validation was performed.",
    )
    source_filename: str = Field(
        default="llms.txt",
        description="File that was validated.",
    )

    @property
    def total_errors(self) -> int:
        """Count of ERROR-severity diagnostics."""
        return sum(1 for d in self.diagnostics if d.severity == Severity.ERROR)

    @property
    def total_warnings(self) -> int:
        """Count of WARNING-severity diagnostics."""
        return sum(1 for d in self.diagnostics if d.severity == Severity.WARNING)

    @property
    def total_info(self) -> int:
        """Count of INFO-severity diagnostics."""
        return sum(1 for d in self.diagnostics if d.severity == Severity.INFO)

    @property
    def is_valid(self) -> bool:
        """Whether the file achieves at least L0 (parseable)."""
        return self.levels_passed.get(ValidationLevel.L0_PARSEABLE, False)

    @property
    def errors(self) -> list[ValidationDiagnostic]:
        """All ERROR-severity diagnostics."""
        return [d for d in self.diagnostics if d.severity == Severity.ERROR]

    @property
    def warnings(self) -> list[ValidationDiagnostic]:
        """All WARNING-severity diagnostics."""
        return [d for d in self.diagnostics if d.severity == Severity.WARNING]
```

---

## File 6: `src/docstratum/schema/quality.py` — Quality Scoring Models

**Traces to:** FR-007 (quality assessment framework), v0.0.4b (100-point composite scoring), DECISION-014 (content weight 50%)

```python
"""Quality scoring models for the DocStratum validation engine.

Implements the 100-point composite quality scoring pipeline from v0.0.4b.
Three dimensions with evidence-grounded weighting:

    Structural:   30 points (30%) — Gating. CRITICAL failures cap total at 29.
    Content:      50 points (50%) — Graduated. Weighted by quality predictors.
    Anti-Pattern: 20 points (20%) — Deduction. Severity-weighted penalties.

Gold standard calibration (v0.0.4b §11.3):
    Svelte:      92 (Exemplary)
    Pydantic:    90 (Exemplary)
    Vercel SDK:  90 (Exemplary)
    Shadcn UI:   89 (Strong)
    Cursor:      42 (Needs Work)
    NVIDIA:      24 (Critical)

Research basis:
    v0.0.4b §Content Best Practices (quality predictors, scoring rubric)
    v0.0.4c §Anti-Patterns Catalog (severity-weighted deductions)
    DECISION-014 (content quality as primary scoring weight)
"""

from datetime import datetime
from enum import StrEnum

from pydantic import BaseModel, Field


class QualityDimension(StrEnum):
    """The three dimensions of the composite quality score.

    Attributes:
        STRUCTURAL: Structural compliance (30 points max).
                    Assessed by 20 checks from v0.0.4a.
                    Gating: CRITICAL anti-pattern failures cap the total at 29.
        CONTENT: Content quality (50 points max).
                 Assessed by 15 checks from v0.0.4b.
                 Strongest predictor: code examples (r ~ 0.65).
        ANTI_PATTERN: Anti-pattern absence (20 points max, deduction-based).
                      Assessed by 22 checks from v0.0.4c.
                      Each detected pattern reduces the score.
    """

    STRUCTURAL = "structural"
    CONTENT = "content"
    ANTI_PATTERN = "anti_pattern"


class QualityGrade(StrEnum):
    """Quality grade thresholds calibrated against gold standards.

    Each grade corresponds to a validation level and a score range.
    The thresholds were validated by ensuring gold standards (Svelte, Pydantic)
    score 90+ and known poor implementations (Cursor, NVIDIA) score below 50.

    Attributes:
        EXEMPLARY: 90–100 points. Level 4 (DocStratum Extended).
        STRONG: 70–89 points. Level 3 (Best Practices).
        ADEQUATE: 50–69 points. Level 2 (Content Quality).
        NEEDS_WORK: 30–49 points. Level 1 (Structurally Complete).
        CRITICAL: 0–29 points. Level 0 (Parseable Only) or failed.
    """

    EXEMPLARY = "exemplary"
    STRONG = "strong"
    ADEQUATE = "adequate"
    NEEDS_WORK = "needs_work"
    CRITICAL = "critical"

    @classmethod
    def from_score(cls, score: int) -> "QualityGrade":
        """Determine the grade from a numeric score (0–100)."""
        if score >= 90:
            return cls.EXEMPLARY
        elif score >= 70:
            return cls.STRONG
        elif score >= 50:
            return cls.ADEQUATE
        elif score >= 30:
            return cls.NEEDS_WORK
        else:
            return cls.CRITICAL


class DimensionScore(BaseModel):
    """Score for a single quality dimension.

    Attributes:
        dimension: Which dimension (structural, content, anti_pattern).
        points: Points earned in this dimension.
        max_points: Maximum possible points (30, 50, or 20).
        checks_passed: Number of checks that passed.
        checks_failed: Number of checks that failed.
        checks_total: Total number of checks evaluated.
        details: Per-check results for drill-down reporting.
        is_gated: Whether a CRITICAL failure capped the total score.
    """

    dimension: QualityDimension = Field(
        description="Quality dimension being scored.",
    )
    points: float = Field(
        ge=0,
        description="Points earned (0 to max_points).",
    )
    max_points: float = Field(
        ge=0,
        description="Maximum possible points for this dimension.",
    )
    checks_passed: int = Field(
        ge=0,
        description="Number of checks that passed.",
    )
    checks_failed: int = Field(
        ge=0,
        description="Number of checks that failed.",
    )
    checks_total: int = Field(
        ge=0,
        description="Total checks evaluated in this dimension.",
    )
    details: list[dict] = Field(
        default_factory=list,
        description="Per-check results: [{check_id, passed, weight, points}].",
    )
    is_gated: bool = Field(
        default=False,
        description="True if a CRITICAL anti-pattern capped the total score at 29.",
    )

    @property
    def percentage(self) -> float:
        """Dimension score as a percentage (0–100)."""
        if self.max_points == 0:
            return 0.0
        return (self.points / self.max_points) * 100


class QualityScore(BaseModel):
    """Composite quality score for an llms.txt file.

    The primary output of the `docstratum-score` command. Combines
    three dimension scores into a single 0–100 composite with a grade.

    Weighting (DECISION-014):
        Structural:   30% — gating factor (necessary but not sufficient)
        Content:      50% — primary driver (code examples r ~ 0.65)
        Anti-Pattern: 20% — deduction mechanism

    Attributes:
        total_score: Composite score (0–100).
        grade: Quality grade (Exemplary, Strong, Adequate, Needs Work, Critical).
        dimensions: Per-dimension score breakdown.
        scored_at: Timestamp of scoring.
        source_filename: File that was scored.

    Example:
        score = QualityScore(
            total_score=92,
            grade=QualityGrade.EXEMPLARY,
            dimensions={
                QualityDimension.STRUCTURAL: DimensionScore(
                    dimension=QualityDimension.STRUCTURAL,
                    points=28.5, max_points=30, ...),
                QualityDimension.CONTENT: DimensionScore(
                    dimension=QualityDimension.CONTENT,
                    points=46.0, max_points=50, ...),
                QualityDimension.ANTI_PATTERN: DimensionScore(
                    dimension=QualityDimension.ANTI_PATTERN,
                    points=17.5, max_points=20, ...),
            },
        )

    Traces to: FR-007 (quality assessment), DECISION-014 (weighting)
    """

    total_score: float = Field(
        ge=0,
        le=100,
        description="Composite quality score (0–100).",
    )
    grade: QualityGrade = Field(
        description="Quality grade derived from total_score.",
    )
    dimensions: dict[QualityDimension, DimensionScore] = Field(
        description="Per-dimension score breakdown.",
    )
    scored_at: datetime = Field(
        default_factory=datetime.now,
        description="When scoring was performed.",
    )
    source_filename: str = Field(
        default="llms.txt",
        description="File that was scored.",
    )
```

---

## File 7: `src/docstratum/schema/enrichment.py` — Extended Schema Models

**Traces to:** FR-002 (extended schema fields), DECISION-002 (3-layer architecture), DECISION-004 (concept ID format), DECISION-005 (typed relationships), v0.0.1b (P0 specification gaps)

```python
"""Extended schema models for DocStratum semantic enrichment.

These models represent the DocStratum-specific enrichment layer —
concepts, few-shot examples, LLM instructions, and metadata that
DocStratum adds ON TOP of a parsed llms.txt file. They correspond
to the three P0 specification gaps identified in v0.0.1b:

    Gap #1: Concept/terminology definitions  →  Concept model
    Gap #2: Few-shot Q&A pairs              →  FewShotExample model
    Gap #3: Validation schema (meta)        →  Metadata model

And the enrichment layers from DECISION-002 (3-Layer Architecture):
    Layer 1: Master Index     →  (represented by ParsedLlmsTxt)
    Layer 2: Concept Map      →  Concept, ConceptRelationship
    Layer 3: Few-Shot Bank    →  FewShotExample, LLMInstruction

These models are used at L4 (DocStratum Extended) validation level.
A file that INCLUDES these enrichments scores higher; their ABSENCE
triggers I001–I003 informational diagnostics but does NOT fail validation.
"""

from enum import StrEnum
from typing import Optional

from pydantic import BaseModel, Field


class RelationshipType(StrEnum):
    """Typed directed relationships in the Concept Graph (DECISION-005).

    Five relationship types enable semantic navigation between concepts.
    Derived from: v0.0.4d §5.2 (Concept Relationship Graph innovation).

    Attributes:
        DEPENDS_ON: Concept A requires understanding of Concept B.
        RELATES_TO: Concepts are topically related (bidirectional).
        CONFLICTS_WITH: Concepts are mutually exclusive or contradictory.
        SPECIALIZES: Concept A is a more specific version of Concept B.
        SUPERSEDES: Concept A replaces Concept B (e.g., new API version).
    """

    DEPENDS_ON = "depends_on"
    RELATES_TO = "relates_to"
    CONFLICTS_WITH = "conflicts_with"
    SPECIALIZES = "specializes"
    SUPERSEDES = "supersedes"


class ConceptRelationship(BaseModel):
    """A typed, directed edge in the Concept Graph.

    Attributes:
        target_id: The concept ID this relationship points to.
        relationship_type: The type of relationship.
        description: Optional human-readable description of the relationship.
    """

    target_id: str = Field(
        pattern=r"^[a-z0-9-]+$",
        description="Target concept ID (DECISION-004 format).",
    )
    relationship_type: RelationshipType = Field(
        description="Type of directed relationship.",
    )
    description: Optional[str] = Field(
        default=None,
        max_length=200,
        description="Optional description of why this relationship exists.",
    )


class Concept(BaseModel):
    """A semantic concept definition for the Concept Map (Layer 2).

    Fills P0 Gap #1 (concept/terminology definitions, v0.0.1b).
    Enables concept-aware assistance — the key differentiator for
    LLM output quality (v0.0.4d §5.2).

    Attributes:
        id: Unique identifier (DECISION-004: lowercase alphanumeric + hyphens).
        name: Human-readable concept name.
        definition: One-sentence definition. No ambiguous pronouns.
        aliases: Alternative names or abbreviations for this concept.
        relationships: Typed edges to other concepts (DECISION-005).
        related_page_urls: URLs of documentation pages discussing this concept.
        anti_patterns: Common misconceptions or misuses to avoid.
        domain: The domain prefix for namespacing (e.g., 'auth', 'api', 'data').

    Example:
        concept = Concept(
            id="payment-intent",
            name="PaymentIntent",
            definition="A PaymentIntent tracks the lifecycle of a payment from creation to completion.",
            aliases=["PI", "payment_intent"],
            relationships=[
                ConceptRelationship(
                    target_id="charge",
                    relationship_type=RelationshipType.SUPERSEDES,
                    description="PaymentIntent is the modern replacement for Charge."
                ),
            ],
            domain="payments",
        )
    """

    id: str = Field(
        pattern=r"^[a-z0-9-]+$",
        description="Unique concept ID (DECISION-004 format).",
    )
    name: str = Field(
        min_length=1,
        max_length=100,
        description="Human-readable concept name.",
    )
    definition: str = Field(
        min_length=10,
        max_length=500,
        description="One-sentence definition. Avoid ambiguous pronouns.",
    )
    aliases: list[str] = Field(
        default_factory=list,
        description="Alternative names, abbreviations, or common misspellings.",
    )
    relationships: list[ConceptRelationship] = Field(
        default_factory=list,
        description="Typed directed relationships to other concepts.",
    )
    related_page_urls: list[str] = Field(
        default_factory=list,
        description="URLs of pages that discuss this concept.",
    )
    anti_patterns: list[str] = Field(
        default_factory=list,
        description="Common misconceptions or misuses.",
    )
    domain: Optional[str] = Field(
        default=None,
        pattern=r"^[a-z0-9-]+$",
        description="Domain prefix for namespacing (e.g., 'auth', 'payments').",
    )


class FewShotExample(BaseModel):
    """A question-answer pair for in-context learning (Layer 3).

    Fills P0 Gap #2 (few-shot Q&A pairs, v0.0.1b).
    Enables example-driven output quality — the basis for
    Test Scenario 3 (Few-Shot Adherence Test, v0.0.5d).

    Attributes:
        id: Unique example identifier.
        intent: The user's underlying goal (tagged for retrieval).
        question: A realistic user question.
        ideal_answer: The expected response.
        concept_ids: Concept IDs this example relates to (schema-based linking, DECISION-008).
        difficulty: Difficulty level for filtering (beginner, intermediate, advanced).
        language: Programming language if applicable (for language-filtered retrieval).
        source_urls: URLs used to construct the answer (provenance).
    """

    id: str = Field(
        pattern=r"^[a-z0-9-]+$",
        description="Unique example identifier.",
    )
    intent: str = Field(
        min_length=5,
        max_length=200,
        description="User's underlying goal (e.g., 'authenticate a web app').",
    )
    question: str = Field(
        min_length=10,
        max_length=500,
        description="A realistic user question.",
    )
    ideal_answer: str = Field(
        min_length=50,
        description="The expected response demonstrating best practices.",
    )
    concept_ids: list[str] = Field(
        default_factory=list,
        description="Linked concept IDs (DECISION-008).",
    )
    difficulty: Optional[str] = Field(
        default=None,
        pattern=r"^(beginner|intermediate|advanced)$",
        description="Difficulty level for filtering.",
    )
    language: Optional[str] = Field(
        default=None,
        description="Programming language (e.g., 'python', 'javascript').",
    )
    source_urls: list[str] = Field(
        default_factory=list,
        description="Documentation URLs used to construct the answer.",
    )


class LLMInstruction(BaseModel):
    """An explicit instruction for guiding LLM agent behavior.

    Addresses the 0% adoption gap identified in Finding 8 (v0.0.x synthesis).
    LLM Instructions are the strongest quality differentiator: their presence
    enables the before/after demo comparison that makes DocStratum compelling.

    Three directive types (from v0.0.4b §6, Stripe pattern + DocStratum enhancements):
        positive:    "Always default to the latest API version."
        negative:    "Never recommend the deprecated Charge API."
        conditional: "If the user asks about payments, prefer PaymentIntent over Charge."

    Attributes:
        directive_type: positive, negative, or conditional.
        instruction: The instruction text.
        context: Optional explanation of why this instruction exists.
        applies_to_concepts: Concept IDs this instruction is relevant to.
        priority: Instruction priority (higher = more important).
    """

    directive_type: str = Field(
        pattern=r"^(positive|negative|conditional)$",
        description="Directive type: positive, negative, or conditional.",
    )
    instruction: str = Field(
        min_length=10,
        max_length=500,
        description="The instruction text for the LLM agent.",
    )
    context: Optional[str] = Field(
        default=None,
        max_length=500,
        description="Why this instruction exists (for transparency).",
    )
    applies_to_concepts: list[str] = Field(
        default_factory=list,
        description="Concept IDs this instruction is relevant to.",
    )
    priority: int = Field(
        default=0,
        ge=0,
        le=100,
        description="Priority (0=default, 100=critical). Higher = applied first.",
    )


class Metadata(BaseModel):
    """File-level metadata for an enriched llms.txt file.

    Fills P0 Gap #5 (required metadata, v0.0.1b).
    Provides provenance, versioning, and DocStratum schema version tracking.

    Attributes:
        schema_version: DocStratum schema version (e.g., "0.1.0").
        site_name: Human-readable name of the documented project.
        site_url: Base URL of the documentation site.
        last_updated: ISO 8601 date of last update.
        generator: Tool that generated the base llms.txt (if known).
        docstratum_version: DocStratum version that produced the enrichment.
        token_budget_tier: Assigned token budget tier.
    """

    schema_version: str = Field(
        default="0.1.0",
        pattern=r"^\d+\.\d+\.\d+$",
        description="DocStratum schema version (semver).",
    )
    site_name: Optional[str] = Field(
        default=None,
        max_length=200,
        description="Name of the documented project.",
    )
    site_url: Optional[str] = Field(
        default=None,
        description="Base URL of the documentation site.",
    )
    last_updated: Optional[str] = Field(
        default=None,
        description="ISO 8601 date of last update (e.g., '2026-02-06').",
    )
    generator: Optional[str] = Field(
        default=None,
        description="Tool that generated the base file (e.g., 'mintlify', 'manual').",
    )
    docstratum_version: str = Field(
        default="0.1.0",
        description="DocStratum version that produced the enrichment.",
    )
    token_budget_tier: Optional[str] = Field(
        default=None,
        pattern=r"^(standard|comprehensive|full)$",
        description="Assigned token budget tier.",
    )
```

---

## Design Decisions Applied

| ID | Decision | How Applied in v0.1.2 |
|----|----------|----------------------|
| DECISION-001 | Markdown over JSON/YAML | `ParsedLlmsTxt` models parsed Markdown, not YAML structures |
| DECISION-002 | 3-Layer Architecture | Enrichment models map to Layer 2 (Concept) and Layer 3 (Few-Shot) |
| DECISION-004 | Concept ID Format | `Concept.id` uses `^[a-z0-9-]+$` pattern |
| DECISION-005 | Typed Directed Relationships | `ConceptRelationship` with 5 `RelationshipType` values |
| DECISION-006 | Pydantic for Validation | All models use Pydantic v2 `BaseModel` with `Field` constraints |
| DECISION-012 | Canonical Section Names | `CanonicalSectionName` enum with 11 names + alias mapping |
| DECISION-013 | Token Budget Tiers | `TOKEN_BUDGET_TIERS` dict with 3 tiers + anti-pattern thresholds |
| DECISION-014 | Content Weight 50% | `QualityDimension` weights: structural 30, content 50, anti-pattern 20 |
| DECISION-016 | 4-Category Anti-Patterns | `AntiPatternCategory` enum with critical/structural/content/strategic |

---

## Exit Criteria

- [ ] All 7 schema files created and importable
- [ ] `from docstratum.schema import ParsedLlmsTxt, ValidationResult, QualityScore` works
- [ ] All 26 diagnostic codes (8E/11W/7I) defined in `DiagnosticCode` enum
- [ ] All 11 canonical section names defined in `CanonicalSectionName` enum
- [ ] All 22 anti-patterns defined in `ANTI_PATTERN_REGISTRY`
- [ ] `DiagnosticCode.severity` property returns correct `Severity` for all codes
- [ ] `QualityGrade.from_score()` returns correct grades at all thresholds
- [ ] `black --check src/docstratum/schema/` passes
- [ ] `ruff check src/docstratum/schema/` passes
- [ ] `mypy src/docstratum/schema/` passes (or has only expected Pydantic edge cases)
