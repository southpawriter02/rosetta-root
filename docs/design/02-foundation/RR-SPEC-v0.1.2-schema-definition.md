# v0.1.2 — Schema Definition

> **Task:** Define the Pydantic models that validate the `llms.txt` file structure.
> 

---

## Task Overview

---

## Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ 1. Create   │───▶│ 2. Define   │───▶│ 3. Add      │───▶│ 4. Test     │
│   folders   │    │   models    │    │   validators│    │   schema    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

---

## Directory Structure

```
docstratum/
├── schemas/
│   ├── __init__.py
│   └── llms_schema.py    ← Main schema file
├── tests/
│   ├── __init__.py
│   └── test_schema.py
├── examples/
│   └── llms.txt          ← Sample file (v0.1.3)
├── requirements.txt
└── verify_setup.py
```

---

## Schema Implementation

### File: `schemas/__init__.py`

```python
from .llms_schema import (
    LlmsTxt,
    CanonicalPage,
    Concept,
    FewShotExample,
    ContentType
)

__all__ = [
    'LlmsTxt',
    'CanonicalPage',
    'Concept',
    'FewShotExample',
    'ContentType'
]
```

### File: `schemas/llms_[schema.py](http://schema.py)`

```python
"""Pydantic schema for the llms.txt file format.

This module defines the data models used to validate and parse
llms.txt files, ensuring they conform to the DocStratum specification.
"""

from pydantic import BaseModel, HttpUrl, Field, field_validator
from typing import Literal
from datetime import date

# Type alias for content types
ContentType = Literal["tutorial", "reference", "changelog", "concept", "faq", "guide"]

class CanonicalPage(BaseModel):
    """A single page in the documentation index.
    
    Attributes:
        url: The canonical URL of the page.
        title: Human-readable page title.
        content_type: Classification of the page content.
        last_verified: Date when the page was last verified as accurate.
        summary: Brief description (max 280 characters).
    """
    url: HttpUrl
    title: str = Field(..., min_length=1, max_length=200)
    content_type: ContentType
    last_verified: date
    summary: str = Field(..., max_length=280)
    
    @field_validator('summary')
    @classmethod
    def summary_not_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError('Summary cannot be empty or whitespace only')
        return v.strip()

class Concept(BaseModel):
    """A semantic concept extracted from the documentation.
    
    Attributes:
        id: Unique identifier (e.g., 'auth-oauth2').
        name: Human-readable concept name.
        definition: One-sentence definition. No pronouns.
        related_pages: URLs of pages that discuss this concept.
        depends_on: IDs of concepts this one requires understanding of.
        anti_patterns: Common misconceptions to avoid.
    """
    id: str = Field(..., pattern=r'^[a-z0-9-]+$')
    name: str = Field(..., min_length=1, max_length=100)
    definition: str = Field(..., min_length=10, max_length=500)
    related_pages: list[str] = Field(default_factory=list)
    depends_on: list[str] = Field(default_factory=list)
    anti_patterns: list[str] = Field(default_factory=list)
    
    @field_validator('definition')
    @classmethod
    def definition_no_pronouns(cls, v: str) -> str:
        """Warn if definition contains ambiguous pronouns."""
        pronouns = [' it ', ' they ', ' this ', ' that ']
        for pronoun in pronouns:
            if pronoun.lower() in v.lower():
                # Warning only, don't raise
                import warnings
                warnings.warn(f"Definition contains pronoun '{pronoun.strip()}' which may be ambiguous")
        return v

class FewShotExample(BaseModel):
    """A question-answer pair for in-context learning.
    
    Attributes:
        intent: The user's underlying goal (e.g., 'authenticate a web app').
        question: A realistic user question.
        ideal_answer: The expected response format and content.
        source_pages: URLs used to construct the answer.
    """
    intent: str = Field(..., min_length=5, max_length=200)
    question: str = Field(..., min_length=10, max_length=500)
    ideal_answer: str = Field(..., min_length=50)
    source_pages: list[str] = Field(default_factory=list, min_length=1)

class LlmsTxt(BaseModel):
    """Root model for the llms.txt file.
    
    Attributes:
        schema_version: Version of the llms.txt schema (e.g., '1.0').
        site_name: Human-readable name of the documentation site.
        site_url: Base URL of the documentation site.
        last_updated: Date when this file was last updated.
        pages: List of indexed documentation pages.
        concepts: List of semantic concepts.
        few_shot_examples: List of Q&A examples for the agent.
    """
    schema_version: str = Field(default="1.0", pattern=r'^\d+\.\d+$')
    site_name: str = Field(..., min_length=1, max_length=200)
    site_url: HttpUrl
    last_updated: date
    pages: list[CanonicalPage] = Field(default_factory=list, min_length=1)
    concepts: list[Concept] = Field(default_factory=list)
    few_shot_examples: list[FewShotExample] = Field(default_factory=list)
    
    @field_validator('concepts')
    @classmethod
    def validate_concept_dependencies(cls, v: list[Concept]) -> list[Concept]:
        """Ensure all depends_on references point to valid concept IDs."""
        concept_ids = {c.id for c in v}
        for concept in v:
            for dep_id in concept.depends_on:
                if dep_id not in concept_ids:
                    raise ValueError(
                        f"Concept '{concept.id}' depends on unknown concept '{dep_id}'"
                    )
        return v
```

---

## Test File

### File: `tests/test_[schema.py](http://schema.py)`

```python
"""Tests for the llms.txt schema validation."""

import pytest
from datetime import date
from pydantic import ValidationError
from schemas import LlmsTxt, CanonicalPage, Concept, FewShotExample

class TestCanonicalPage:
    """Tests for CanonicalPage model."""
    
    def test_valid_page(self):
        page = CanonicalPage(
            url="https://example.com/docs",
            title="Getting Started",
            content_type="tutorial",
            last_verified=date(2026, 1, 1),
            summary="A quick start guide for new users."
        )
        assert page.title == "Getting Started"
    
    def test_invalid_content_type_raises(self):
        with pytest.raises(ValidationError):
            CanonicalPage(
                url="https://example.com",
                title="Test",
                content_type="INVALID",
                last_verified=date(2026, 1, 1),
                summary="Test summary"
            )
    
    def test_summary_too_long_raises(self):
        with pytest.raises(ValidationError):
            CanonicalPage(
                url="https://example.com",
                title="Test",
                content_type="reference",
                last_verified=date(2026, 1, 1),
                summary="x" * 300  # Over 280 chars
            )

class TestConcept:
    """Tests for Concept model."""
    
    def test_valid_concept(self):
        concept = Concept(
            id="auth-oauth2",
            name="OAuth2 Authentication",
            definition="OAuth2 is an authorization framework for secure API access."
        )
        assert concept.id == "auth-oauth2"
    
    def test_invalid_id_format_raises(self):
        with pytest.raises(ValidationError):
            Concept(
                id="Invalid ID!",  # Spaces and special chars not allowed
                name="Test",
                definition="A valid definition that is long enough."
            )

class TestLlmsTxt:
    """Tests for root LlmsTxt model."""
    
    def test_valid_llms_txt(self):
        llms = LlmsTxt(
            site_name="Example Docs",
            site_url="https://docs.example.com",
            last_updated=date(2026, 2, 1),
            pages=[
                CanonicalPage(
                    url="https://docs.example.com/start",
                    title="Start",
                    content_type="tutorial",
                    last_verified=date(2026, 1, 1),
                    summary="Getting started guide."
                )
            ]
        )
        assert llms.schema_version == "1.0"
    
    def test_invalid_dependency_raises(self):
        with pytest.raises(ValidationError):
            LlmsTxt(
                site_name="Test",
                site_url="https://example.com",
                last_updated=date(2026, 1, 1),
                pages=[...],  # Add valid page
                concepts=[
                    Concept(
                        id="concept-a",
                        name="Concept A",
                        definition="A valid definition for concept A.",
                        depends_on=["nonexistent-concept"]
                    )
                ]
            )
```

---

## Acceptance Criteria

- [ ]  `schemas/llms_[schema.py](http://schema.py)` contains all 4 models
- [ ]  All models have docstrings
- [ ]  `from schemas import LlmsTxt` works
- [ ]  `pytest tests/test_[schema.py](http://schema.py)` passes
- [ ]  Invalid data raises `ValidationError`

---

## Design Decisions Log