# Commenting Standards — DocStratum

<aside>

**Scope:** All phases (v0.1.x through v0.6.x)

**Status:** Active

**Applies To:** All Python source files in `src/`, `schemas/`, `tests/`, and any scripts

**Deliverable:** Enforceable docstring templates, type hint requirements, inline comment rules, and TODO conventions for all project code

</aside>

---

## Purpose

This document defines commenting and documentation-in-code standards for the DocStratum project. It covers docstrings, type hints, inline comments, and TODO conventions.

NFR-007 requires every public API to have docstrings with examples. NFR-013 requires substantial comments in complex modules. This document operationalizes both with specific templates and enforcement rules.

---

## Commenting Philosophy

### Core Principles

1. **Docstrings describe the contract.** What a function does, what it takes, what it returns, and when it fails.
2. **Comments explain why, not what.** The code shows what happens; comments explain non-obvious reasoning.
3. **Type hints are mandatory.** They serve as machine-checkable documentation and enable IDE support.
4. **Less is more.** A clear function name eliminates the need for a comment. Over-commenting obscures the code.
5. **Stale comments are bugs.** Update or remove comments when the code changes.

---

## Docstring Style: Google Convention

All docstrings follow **Google style** as documented in the [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings).

---

## Module Docstrings

Every `.py` file starts with a module-level docstring:

```python
"""Loader module for parsing and loading llms.txt files.

Provides the Loader class which handles file reading, line normalization,
and initial parsing of llms.txt documents into structured data.

Classes:
    Loader: Main entry point for loading llms.txt files.
    LineNormalizer: Handles whitespace and encoding normalization.

Implementation Status:
    - [x] File loading (v0.3.1a)
    - [x] Line normalization (v0.3.1b)
    - [ ] Streaming support (v0.3.1c)
    - [ ] Caching (v0.3.1d)

Related:
    - schemas/llms_schema.py: Pydantic models this module produces
    - src/context/builder.py: Consumes Loader output
"""
```

### Module Docstring Rules

1. **First line:** One-sentence summary of the module's purpose
2. **Body:** Expanded description if needed
3. **Classes/Functions section:** List public names
4. **Implementation Status:** Checklist with version references (optional during early phases)
5. **Related:** Links to closely coupled modules

---

## Class Docstrings

```python
class ContextBuilder:
    """Assembles context from parsed llms.txt data within a token budget.

    Takes a parsed Document and a query, selects relevant sections,
    and assembles a Context object that fits within the specified
    token budget.

    Attributes:
        token_budget: Maximum tokens allowed in the assembled context.
        strategy: Selection strategy ('relevance', 'recency', 'priority').

    Example:
        >>> builder = ContextBuilder(token_budget=4000)
        >>> context = builder.build(document, query="What APIs exist?")
        >>> assert context.token_count <= 4000
    """

    def __init__(self, token_budget: int, strategy: str = "relevance") -> None:
        """Initialize the ContextBuilder.

        Args:
            token_budget: Maximum token count for assembled context.
                Must be positive.
            strategy: Section selection strategy. One of 'relevance',
                'recency', or 'priority'. Defaults to 'relevance'.

        Raises:
            ValueError: If token_budget is not positive.
            ValueError: If strategy is not a recognized value.
        """
```

### Class Docstring Rules

1. **Class docstring:** Describes what the class represents and how to use it
2. **Attributes section:** Documents public instance attributes
3. **Example section:** Shows typical usage (required for public classes)
4. **`__init__` docstring:** Documents constructor parameters in `Args:` section
5. **`__init__` does NOT duplicate** the class-level description

---

## Function and Method Docstrings

### Public Functions (Required)

```python
def load(self, path: str | Path) -> Document:
    """Load and parse an llms.txt file into a Document model.

    Reads the file at the given path, normalizes line endings,
    and parses the content into a structured Document object.

    Args:
        path: File system path to the llms.txt file. Accepts
            string or Path objects. Must point to an existing file.

    Returns:
        A Document object containing parsed sections and entries.

    Raises:
        FileNotFoundError: If the path does not exist.
        ValidationError: If the file content fails schema validation.

    Example:
        >>> loader = Loader()
        >>> doc = loader.load("examples/llms.txt")
        >>> print(f"Loaded {len(doc.sections)} sections")
        Loaded 5 sections
    """
```

### Docstring Sections (in order)

| Section | When Required | Content |
|---------|--------------|---------|
| **Summary** | Always | One-line description |
| **Extended description** | When summary is insufficient | Additional context |
| **Args:** | When function takes parameters | Parameter name, type, description |
| **Returns:** | When function returns a value | Description of return value |
| **Raises:** | When function raises exceptions | Exception type and condition |
| **Example:** | Required for all public functions | Working code example |
| **Note:** | When there are important caveats | Non-obvious behavior |

### Private Methods

Private methods (prefixed with `_`) require docstrings only when:
- The method is longer than 10 lines
- The method contains non-obvious logic
- The method is called from multiple places within the class

```python
def _estimate_tokens(self, text: str) -> int:
    """Estimate token count using the 4-chars-per-token heuristic.

    This is an approximation. For precise counts, use tiktoken.
    The 4:1 ratio is conservative and works well for English text.

    Args:
        text: Input text to estimate tokens for.

    Returns:
        Estimated token count.
    """
    return len(text) // 4
```

---

## Type Hints

### Required On

- All public function signatures (parameters and return type)
- All public method signatures
- All class attributes in `__init__`
- Module-level constants

### Examples

```python
from pathlib import Path
from typing import Optional


def load(self, path: str | Path) -> Document:
    ...

def build(
    self,
    document: Document,
    query: str,
    max_sections: int = 10,
) -> Context:
    ...

def validate(
    self,
    content: str,
    level: int = 1,
    strict: bool = False,
) -> list[ValidationIssue]:
    ...

# Module-level constant
DEFAULT_TOKEN_BUDGET: int = 4000
```

### Type Hint Rules

1. Use `str | Path` (union syntax) over `Union[str, Path]` (Python 3.10+)
2. Use `list[str]` over `List[str]` (Python 3.9+)
3. Use `dict[str, int]` over `Dict[str, int]`
4. Use `X | None` over `Optional[X]`
5. Return type `-> None` is required for functions that return nothing
6. Dataclass fields must have type annotations (enforced by dataclass itself)

---

## Dataclass and Pydantic Model Comments

### Inline Field Comments

```python
from pydantic import BaseModel, Field


class CanonicalPage(BaseModel):
    """A single page entry in the Master Index layer."""

    url: str = Field(description="Canonical HTTPS URL for the page")
    title: str = Field(description="Human-readable page title")
    content_type: ContentType = Field(description="Classification of page content")
    last_updated: str | None = Field(
        default=None,
        description="ISO-8601 date of last content update",
    )
```

### Rules

- Pydantic `Field(description=...)` serves as the docstring for each field
- For plain dataclasses, use inline comments on complex fields:

```python
@dataclass
class Context:
    sections: list[Section]  # Selected sections within token budget
    token_count: int         # Actual token count of assembled context
    query: str               # Original query that drove section selection
    strategy: str            # Selection strategy used ('relevance', etc.)
```

---

## Inline Comments

### When to Comment

- **Non-obvious business logic:** Why a specific threshold was chosen
- **Workarounds:** Temporary fixes with context for why they exist
- **Algorithm explanations:** Complex logic that benefits from annotation
- **Performance decisions:** Why a less readable approach was chosen for speed

### When NOT to Comment

- **Obvious code:** `i += 1  # Increment i` — never
- **What the code does:** The code already says that
- **Entire function narration:** Use better function/variable names instead

### Good vs. Bad Examples

```python
# GOOD — explains WHY
# 4:1 ratio is conservative but avoids exceeding token limits
# when the actual tokenizer isn't available
estimated_tokens = len(text) // 4

# GOOD — documents a non-obvious constraint
# OpenAI rate limits reset every 60s; back off to avoid cascading failures
await asyncio.sleep(60)

# GOOD — explains a magic number
# 0.85 threshold from v0.0.4b best practices analysis (Table 3)
SIMILARITY_THRESHOLD = 0.85

# BAD — restates the code
# Set the token budget to 4000
token_budget = 4000

# BAD — obvious from the function name
# Load the file
document = loader.load(path)
```

---

## Section Comments

For functions longer than 30 lines, use section comments to break up logical blocks:

```python
def build(self, document: Document, query: str) -> Context:
    # -- Select candidate sections --
    candidates = self._rank_sections(document.sections, query)

    # -- Apply token budget --
    selected = []
    remaining_tokens = self.token_budget
    for section in candidates:
        tokens = self._estimate_tokens(section.content)
        if tokens <= remaining_tokens:
            selected.append(section)
            remaining_tokens -= tokens

    # -- Assemble context --
    return Context(
        sections=selected,
        token_count=self.token_budget - remaining_tokens,
        query=query,
        strategy=self.strategy,
    )
```

### Section Comment Format

```python
# -- Section Description --
```

Use `# --` prefix with dashes for visual separation. Keep descriptions short (2-5 words).

---

## TODO Comments

### Format

```python
# TODO (vX.Y.Z): Brief description of what needs to be done
```

### Examples

```python
# TODO (v0.3.1c): Add streaming support for large files
# TODO (v0.3.4): Replace heuristic token count with tiktoken
# TODO (v0.5.0): Add performance regression test for this function
```

### Rules

1. **Always include the version reference** in parentheses — ties the TODO to a specific spec
2. **Be specific:** "Add streaming support" not "Fix this later"
3. **Don't use TODO for bugs:** Bugs get issues or immediate fixes, not TODO comments
4. **Review TODOs at phase transitions:** Remove completed TODOs, update version references for deferred ones

### Prohibited TODO Styles

```python
# TODO: fix this                 # No version, too vague
# FIXME                          # No description, no version
# HACK: this is terrible         # Unprofessional; explain the constraint instead
# XXX: needs work                # Use TODO with version and description
```

---

## File Header Convention

### No Boilerplate Headers

Do NOT add file headers with author, date, copyright, or license information. Git tracks authorship and history. The LICENSE file covers copyright.

```python
# WRONG — unnecessary boilerplate
# Author: Deftness
# Date: 2026-02-05
# Copyright (c) 2026 DocStratum
# License: MIT

# CORRECT — just start with the module docstring
"""Loader module for parsing and loading llms.txt files.
...
"""
```

---

## Test File Comments

Test files follow relaxed rules:

```python
"""Tests for the Loader module.

Tests cover file loading, line normalization, error handling,
and performance benchmarks. See testing_standards.md for
naming conventions and fixture patterns.
"""
```

### Test Docstring Rules

1. **Module docstring:** Brief description of what's tested
2. **Test function docstrings:** Optional; the test name should be descriptive enough
3. **Complex test docstrings:** Required when the test setup is non-obvious

```python
def test_loader_handles_bom_marker():
    """Verify that files with UTF-8 BOM markers are parsed correctly.

    Some Windows editors prepend BOM (\\xef\\xbb\\xbf) to UTF-8 files.
    The loader must strip this before parsing.
    """
```

---

## Enforcement

### Tools

| Tool | Purpose | Configuration |
|------|---------|---------------|
| **pydocstyle** | Enforces Google-style docstrings | `--convention=google` |
| **Ruff** | Lints for missing docstrings, type hints | `select = ["D", "ANN"]` |
| **mypy** | Validates type hint correctness | `--strict` (optional; `--check-untyped-defs` minimum) |

### Ruff Configuration (pyproject.toml)

```toml
[tool.ruff]
select = [
    "D",    # pydocstyle
    "ANN",  # flake8-annotations
]

[tool.ruff.pydocstyle]
convention = "google"
```

### Pre-Commit Check

```bash
# Check docstring coverage
ruff check src/ schemas/ --select D

# Check type annotations
ruff check src/ schemas/ --select ANN

# Type check
mypy src/ schemas/ --check-untyped-defs
```

---

## Dos and Don'ts

### Do

- Write docstrings on all public modules, classes, and functions
- Include working `Example:` sections in public docstrings
- Use Google-style docstring format consistently
- Add type hints to all public signatures
- Use `# TODO (vX.Y.Z):` format for deferred work
- Use section comments (`# -- Section --`) in long functions
- Comment the *why*, not the *what*

### Don't

- Add file header boilerplate (author, date, copyright)
- Comment obvious code (`x += 1  # Increment x`)
- Write empty docstrings (`"""."""` or `"""TODO"""`)
- Use `FIXME`, `HACK`, `XXX` without version and description
- Duplicate information between class docstring and `__init__` docstring
- Use `Optional[X]` when `X | None` is available
- Skip docstrings on public functions because "the name is clear enough"

---

## Acceptance Criteria (for this document)

- [ ] Google-style docstring convention mandated
- [ ] Module docstring template with required sections
- [ ] Class docstring template with Attributes and Example sections
- [ ] Function docstring template with Args, Returns, Raises, Example
- [ ] Private method docstring rules (>10 lines or complex)
- [ ] Type hint requirements for public signatures
- [ ] Dataclass/Pydantic inline comment patterns
- [ ] Inline comment rules (why not what) with good/bad examples
- [ ] Section comment format for long functions
- [ ] TODO format with version reference requirement
- [ ] File header convention (no boilerplate)
- [ ] Enforcement tools configured (pydocstyle, Ruff, mypy)
- [ ] All five standards documents cross-reference each other

---

## Related Documents

- [Testing Standards](RR-META-testing-standards.md) — Test docstrings and test naming conventions
- [Logging Standards](RR-META-logging-standards.md) — Docstrings for logging functions
- [Documentation Requirements](RR-META-documentation-requirements.md) — External documentation standards
- [Development Workflow](RR-META-development-workflow.md) — Commenting as part of the development lifecycle (step 7)
- [NFR Specification](../01-research/RR-SPEC-v0.0.5b-non-functional-requirements-and-constraints.md) — NFR-007 (documentation coverage), NFR-013 (documentation-to-code ratio)
