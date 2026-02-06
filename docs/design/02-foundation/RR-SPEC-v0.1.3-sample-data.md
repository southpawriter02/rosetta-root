# v0.1.3 — Sample Data

> **Task:** Create a working example `llms.txt` file that validates against the schema.
> 

---

## Task Overview

---

## Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ 1. Create   │───▶│ 2. Write    │───▶│ 3. Validate │
│   file      │    │   content   │    │   schema    │
└─────────────┘    └─────────────┘    └─────────────┘
```

---

## Sample File

### File: `examples/llms.txt`

```yaml
# llms.txt — The DocStratum Example
# This file demonstrates the Platinum Standard for AI-ready documentation.

schema_version: "1.0"
site_name: "DocStratum Demo API"
site_url: "https://docs.docstratumroot.dev"
last_updated: "2026-02-05"

# ═══════════════════════════════════════════════════════════════════
# SECTION 1: CANONICAL PAGES
# These are the authoritative documentation pages for this site.
# ═══════════════════════════════════════════════════════════════════

pages:
  - url: "https://docs.docstratumroot.dev/getting-started"
    title: "Getting Started Guide"
    content_type: "tutorial"
    last_verified: "2026-02-01"
    summary: "Install the SDK, configure credentials, and make your first API call in under 5 minutes."

  - url: "https://docs.docstratumroot.dev/authentication"
    title: "Authentication Reference"
    content_type: "reference"
    last_verified: "2026-02-01"
    summary: "Complete reference for OAuth2, API keys, and JWT token authentication methods."

  - url: "https://docs.docstratumroot.dev/concepts/context-window"
    title: "Understanding Context Windows"
    content_type: "concept"
    last_verified: "2026-01-15"
    summary: "Explains how LLM context windows work and strategies for efficient token usage."

  - url: "https://docs.docstratumroot.dev/api/endpoints"
    title: "API Endpoints Reference"
    content_type: "reference"
    last_verified: "2026-02-03"
    summary: "Complete list of REST API endpoints with request/response schemas."

  - url: "https://docs.docstratumroot.dev/changelog"
    title: "Changelog"
    content_type: "changelog"
    last_verified: "2026-02-05"
    summary: "Version history and release notes for all API versions."

# ═══════════════════════════════════════════════════════════════════
# SECTION 2: CONCEPTS
# Semantic concepts with explicit relationships and anti-patterns.
# ═══════════════════════════════════════════════════════════════════

concepts:
  - id: "auth-api-key"
    name: "API Key Authentication"
    definition: "API Key authentication uses a secret string passed in the X-API-Key header for server-to-server communication."
    related_pages:
      - "https://docs.docstratumroot.dev/authentication"
    depends_on: []
    anti_patterns:
      - "API keys should NEVER be exposed in client-side code or version control."
      - "API keys do NOT provide user-level permissions; use OAuth2 for user contexts."

  - id: "auth-oauth2"
    name: "OAuth2 Authentication"
    definition: "OAuth2 authentication enables third-party applications to access user data through token-based authorization flows."
    related_pages:
      - "https://docs.docstratumroot.dev/authentication"
      - "https://docs.docstratumroot.dev/getting-started"
    depends_on:
      - "auth-api-key"
    anti_patterns:
      - "OAuth2 is NOT required for server-to-server integrations; use API keys instead."
      - "Access tokens expire after 1 hour; refresh tokens must be stored securely."

  - id: "context-window"
    name: "Context Window"
    definition: "A context window is the maximum number of tokens an LLM can process in a single request, including both input and output."
    related_pages:
      - "https://docs.docstratumroot.dev/concepts/context-window"
    depends_on: []
    anti_patterns:
      - "Context window size is NOT the same as maximum output length."
      - "Exceeding the context window causes truncation, not errors."

# ═══════════════════════════════════════════════════════════════════
# SECTION 3: FEW-SHOT EXAMPLES
# Golden Q&A pairs that teach the agent how to respond.
# ═══════════════════════════════════════════════════════════════════

few_shot_examples:
  - intent: "User wants to authenticate a backend service"
    question: "How do I authenticate my Python script to call the API?"
    ideal_answer: |
      To authenticate a Python script (server-to-server), use API Key authentication:
      
      1. Generate an API key in the dashboard: https://docs.docstratumroot.dev/authentication#api-keys
      2. Store the key securely (environment variable recommended)
      3. Include it in your requests:
      
```

import requests

headers = {"X-API-Key": os.environ["DOCSTRATUM_API_KEY"]}

response = requests.get("[https://api.docstratumroot.dev/v1/data](https://api.docstratumroot.dev/v1/data)", headers=headers)

```
    
    ⚠️ Note: API keys are for server-side use only. For user-facing apps, use OAuth2.
    
    Source: https://docs.docstratumroot.dev/authentication
  source_pages:
    - "https://docs.docstratumroot.dev/authentication"
    - "https://docs.docstratumroot.dev/getting-started"

- intent: "User wants to understand token limits"
  question: "What happens if my prompt is too long?"
  ideal_answer: |
    If your prompt exceeds the context window limit:
    
    1. **Input truncation**: The oldest tokens are silently dropped
    2. **No error returned**: The API processes what fits
    3. **Output affected**: Less space remains for the response
    
    To avoid this:
    - Check token count before sending (use `/v1/tokenize` endpoint)
    - Use summarization for long documents
    - Implement chunking strategies
    
    Current limits:
    - Standard tier: 8,192 tokens
    - Premium tier: 32,768 tokens
    
    Source: https://docs.docstratumroot.dev/concepts/context-window
  source_pages:
    - "https://docs.docstratumroot.dev/concepts/context-window"
```

---

## Validation Script

### File: [`validate.py`](http://validate.py)

```python
#!/usr/bin/env python
"""Validate an llms.txt file against the schema."""

import sys
import yaml
from pathlib import Path
from schemas import LlmsTxt
from pydantic import ValidationError

def validate_file(filepath: str) -> bool:
    """Validate an llms.txt file.
    
    Args:
        filepath: Path to the llms.txt file.
        
    Returns:
        True if valid, False otherwise.
    """
    path = Path(filepath)
    
    if not path.exists():
        print(f"❌ File not found: {filepath}")
        return False
    
    try:
        with open(path) as f:
            data = yaml.safe_load(f)
        
        llms = LlmsTxt(**data)
        
        print(f"✅ Valid llms.txt file!")
        print(f"   Site: {llms.site_name}")
        print(f"   Pages: {len(llms.pages)}")
        print(f"   Concepts: {len(llms.concepts)}")
        print(f"   Examples: {len(llms.few_shot_examples)}")
        return True
        
    except yaml.YAMLError as e:
        print(f"❌ YAML parsing error: {e}")
        return False
        
    except ValidationError as e:
        print(f"❌ Schema validation error:")
        for error in e.errors():
            loc = " → ".join(str(x) for x in error['loc'])
            print(f"   {loc}: {error['msg']}")
        return False

if __name__ == '__main__':
    if len(sys.argv) != 2:
        print("Usage: python validate.py <path-to-llms.txt>")
        sys.exit(1)
    
    success = validate_file(sys.argv[1])
    sys.exit(0 if success else 1)
```

---

## Run Validation

```bash
python validate.py examples/llms.txt
```

**Expected Output:**

```
✅ Valid llms.txt file!
   Site: DocStratum Demo API
   Pages: 5
   Concepts: 3
   Examples: 2
```

---

## Acceptance Criteria

- [ ]  `examples/llms.txt` exists with 5+ pages
- [ ]  File includes 3+ concepts with relationships
- [ ]  File includes 2+ few-shot examples
- [ ]  `python [validate.py](http://validate.py) examples/llms.txt` returns exit code 0
- [ ]  All `depends_on` references are valid

---

## Phase v0.1.0 Complete Checklist

With this task done, verify the entire phase:

- [ ]  v0.1.1: Environment setup verified
- [ ]  v0.1.2: Schema models implemented
- [ ]  v0.1.3: Sample data validates
- [ ]  All `pytest` tests pass

**→ Ready to proceed to v0.2.0: Data Preparation**