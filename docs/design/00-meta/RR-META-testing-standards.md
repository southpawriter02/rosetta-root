# Testing Standards — DocStratum

<aside>

**Scope:** All phases (v0.1.x through v0.6.x)

**Status:** Active

**Applies To:** All test code, test configuration, CI pipelines, and coverage reporting

**Deliverable:** Enforceable testing patterns, naming conventions, fixture strategies, and coverage targets for every module in the project

</aside>

---

## Purpose

This document defines testing standards for the DocStratum project. It establishes the test framework, file structure, naming conventions, fixture patterns, mocking strategies, and coverage targets that apply across all implementation phases.

NFR-010 requires >= 80% line coverage for core modules. This document operationalizes that requirement with specific per-module targets, test categories, and enforcement mechanisms.

---

## Testing Philosophy

### Core Principles

1. **Tests are a deliverable, not a chore.** Every version includes test tasks alongside implementation tasks.
2. **Test the contract, not the implementation.** Assert on public behavior and outputs, not internal state.
3. **One assertion per behavior.** Each test validates a single expectation. Multiple assertions are acceptable only when testing a single logical outcome.
4. **Tests are documentation.** A well-named test tells the next developer exactly what the code does.
5. **Fast by default.** Unit tests run in milliseconds. Slow tests (API calls, file I/O) are explicitly marked.
6. **Deterministic always.** No test should pass or fail based on timing, ordering, or external state.

---

## Framework and Tools

### Required Stack

| Tool | Version | Purpose |
|------|---------|---------|
| **pytest** | >= 8.0.0 | Test runner, fixtures, parametrize, markers |
| **pytest-cov** | >= 4.0.0 | Coverage measurement and reporting |
| **pytest-mock** | >= 3.12.0 | `mocker` fixture for clean mocking |
| **monkeypatch** | (built-in) | Environment variable and attribute patching |

### Prohibited

- `unittest.TestCase` classes — use pytest functions instead
- `nose` or `nose2` — not supported
- `print()` for debugging tests — use `caplog`, `capsys`, or pytest `--tb=short`

---

## Test File Structure

### Directory Layout

```
tests/
├── __init__.py
├── conftest.py                    # Shared fixtures (project-wide)
├── test_schema.py                 # Unit tests for schemas/llms_schema.py
├── test_loader.py                 # Unit tests for loader module
├── test_context_builder.py        # Unit tests for context builder
├── test_validator.py              # Unit tests for validator
├── test_agent.py                  # Unit tests for agent integration
├── integration/
│   ├── __init__.py
│   ├── conftest.py                # Integration-specific fixtures
│   ├── test_pipeline.py           # End-to-end pipeline tests
│   ├── test_agent_integration.py  # Agent with real LLM calls
│   └── test_a_b_harness.py        # A/B testing harness
└── fixtures/
    ├── sample_minimal.txt         # Minimal valid llms.txt
    ├── sample_full.txt            # Full-featured llms.txt
    ├── sample_invalid.txt         # Known-bad input for error tests
    └── sample_large.txt           # Performance test input
```

### File Naming Rules

- Test files: `test_{module_name}.py` — mirrors the source file being tested
- Fixture files: `conftest.py` — at directory level where fixtures are shared
- Test data: `tests/fixtures/` — no test data in `src/` or project root

---

## Test Naming Convention

### Pattern

```
test_{method_or_function}_{scenario}_{expected_result}
```

### Examples

```python
# Good — clear what's being tested, the scenario, and what should happen
def test_parse_llms_txt_valid_input_returns_model():
    ...

def test_parse_llms_txt_empty_file_raises_validation_error():
    ...

def test_context_builder_token_limit_exceeded_truncates_output():
    ...

def test_loader_missing_file_raises_file_not_found():
    ...

# Bad — vague, no scenario, no expected result
def test_parser():
    ...

def test_it_works():
    ...

def test_edge_case():
    ...
```

### Naming Rules

1. Always start with `test_`
2. Include the function/method being tested
3. Describe the scenario or input condition
4. State the expected outcome
5. Use snake_case throughout
6. Be specific enough that a failure message identifies the problem

---

## Test Structure: AAA Pattern

Every test follows **Arrange-Act-Assert**:

```python
def test_canonical_page_valid_url_creates_model():
    # Arrange — set up inputs and expected state
    url = "https://example.com/docs/getting-started"
    title = "Getting Started"

    # Act — perform the action being tested
    page = CanonicalPage(url=url, title=title, content_type=ContentType.GUIDE)

    # Assert — verify the outcome
    assert page.url == url
    assert page.title == title
    assert page.content_type == ContentType.GUIDE
```

### Rules

- **Arrange:** Set up inputs, create objects, configure mocks. No assertions here.
- **Act:** Call the function or method being tested. Ideally one line.
- **Assert:** Verify the result. Each assertion tests one aspect of the outcome.
- **Separate sections** with blank lines and optional `# Arrange / # Act / # Assert` comments for longer tests.

---

## Fixtures

### conftest.py Pattern

Shared fixtures live in `conftest.py` at the appropriate directory level:

```python
# tests/conftest.py — project-wide fixtures

import pytest
from pathlib import Path


@pytest.fixture
def sample_llms_txt():
    """Return the content of the minimal sample llms.txt file."""
    fixture_path = Path(__file__).parent / "fixtures" / "sample_minimal.txt"
    return fixture_path.read_text()


@pytest.fixture
def sample_llms_txt_path():
    """Return the path to the minimal sample llms.txt file."""
    return Path(__file__).parent / "fixtures" / "sample_minimal.txt"


@pytest.fixture
def mock_env(monkeypatch):
    """Set standard environment variables for testing."""
    monkeypatch.setenv("OPENAI_API_KEY", "sk-test-key-not-real")
    monkeypatch.setenv("ANTHROPIC_API_KEY", "sk-ant-test-key-not-real")
    monkeypatch.setenv("DOCSTRATUM_LOG_LEVEL", "DEBUG")
```

### Fixture Rules

1. **Scope appropriately:** Use `scope="session"` for expensive setup (file reads), `scope="function"` (default) for mutable state
2. **Name descriptively:** `sample_llms_txt` not `data` or `fixture1`
3. **No side effects:** Fixtures should not modify global state. Use `monkeypatch` for environment variables.
4. **Yield for cleanup:** Use `yield` fixtures when teardown is needed

```python
@pytest.fixture
def temp_output_dir(tmp_path):
    """Create a temporary output directory for test artifacts."""
    output = tmp_path / "output"
    output.mkdir()
    yield output
    # Cleanup is automatic with tmp_path
```

---

## Mocking Standards

### When to Mock

| Situation | Mock Strategy |
|-----------|---------------|
| LLM API calls (OpenAI, Anthropic) | Always mock in unit tests; use `@pytest.mark.api` for real calls |
| File system reads | Mock only if testing logic, not I/O; use `tmp_path` for write tests |
| Environment variables | Use `monkeypatch.setenv()` |
| External HTTP requests | Use `responses` library or `monkeypatch` |
| Time-dependent code | Mock `datetime.now()` or `time.time()` |
| Configuration | Provide test Config objects; don't read real `.env` |

### LLM Mock Pattern

```python
@pytest.fixture
def mock_llm_response():
    """Return a canned LLM response for testing agent logic."""
    return {
        "content": "The /llms.txt file contains 3 sections...",
        "usage": {"prompt_tokens": 150, "completion_tokens": 50},
        "model": "gpt-4",
    }


def test_docstratum_agent_formats_response(mock_llm_response, mocker):
    # Arrange
    mocker.patch(
        "src.agent.call_llm",
        return_value=mock_llm_response,
    )
    agent = DocStratumAgent(config=test_config)

    # Act
    result = agent.answer("What sections exist?")

    # Assert
    assert "3 sections" in result.content
```

### Mocking Rules

1. **Mock at the boundary, not deep inside.** Mock `call_llm()`, not the HTTP library it uses.
2. **Prefer `monkeypatch` for simple patches**, `mocker` (pytest-mock) for complex mocking.
3. **Never mock the thing you're testing.** If you're testing `Loader.load()`, don't mock `Loader.load()`.
4. **Keep mock data realistic.** Use actual response shapes from the APIs.

---

## Markers

### Required Markers

Define these markers in `pyproject.toml` or `pytest.ini`:

```ini
[tool:pytest]
markers =
    unit: Unit tests (no I/O, no network, no database)
    integration: Integration tests (may use files, database, or multiple modules)
    api: Tests requiring real API keys (OpenAI, Anthropic) — skipped in CI by default
    slow: Tests taking > 5 seconds (benchmarks, large file processing)
    smoke: Quick sanity checks for CI pipeline health
```

### Usage

```python
@pytest.mark.unit
def test_section_model_validates_title():
    ...

@pytest.mark.integration
def test_loader_parses_real_file():
    ...

@pytest.mark.api
def test_openai_agent_returns_response():
    ...

@pytest.mark.slow
def test_large_file_parses_under_500ms(benchmark_llms_txt):
    ...
```

### Running by Marker

```bash
# Run only unit tests (fast, no external deps)
pytest -m unit

# Run unit + integration (no API calls)
pytest -m "not api and not slow"

# Run everything including API tests
pytest -m "" --run-api

# Run smoke tests in CI
pytest -m smoke
```

---

## Parametrized Testing

Use `@pytest.mark.parametrize` for testing multiple inputs against the same logic:

```python
@pytest.mark.parametrize("content_type,expected", [
    ("guide", ContentType.GUIDE),
    ("reference", ContentType.REFERENCE),
    ("tutorial", ContentType.TUTORIAL),
    ("api", ContentType.API_REFERENCE),
])
def test_content_type_from_string(content_type, expected):
    assert ContentType(content_type) == expected


@pytest.mark.parametrize("url", [
    "",
    "not-a-url",
    "ftp://invalid.protocol",
    "http://no-tls.example.com",  # Reject HTTP per NFR-017
])
def test_canonical_page_rejects_invalid_url(url):
    with pytest.raises(ValidationError):
        CanonicalPage(url=url, title="Test", content_type=ContentType.GUIDE)
```

### When to Parametrize

- Testing the same function with multiple valid inputs
- Validating error handling for various invalid inputs
- Boundary value testing (empty, minimal, maximal, just-over-limit)

---

## Coverage Targets

### Per-Module Targets (from NFR-010 and per-module quality standards)

| Module | Source Directory | Coverage Target | Rationale |
|--------|-----------------|-----------------|-----------|
| Schema & Validation | `schemas/` | >= 85% | Core data models; high confidence needed |
| Parser & Loader | `src/loader.py`, `src/parser.py` | >= 85% | Grammar parsing; edge cases critical |
| Content Structure | `src/content/` | >= 80% | Layer building; moderate complexity |
| Context Builder | `src/context/` | >= 75% | Token budgeting; some LLM-dependent paths |
| Agent Integration | `src/agent/` | >= 70% | LLM calls hard to test; mock-heavy |
| A/B Testing Harness | `src/harness/` | >= 70% | Statistical methods; known test inputs |
| Demo Layer | `src/app.py` | >= 60% | Streamlit UI; visual testing limited |

### Phase Coverage Progression

| Phase | Minimum Coverage | Focus |
|-------|-----------------|-------|
| Phase 2 (Foundation) | 80% for `schemas/` | Schema validation, model creation |
| Phase 3 (Data Prep) | 80% for `schemas/` + data modules | Concept extraction, YAML authoring |
| Phase 4 (Logic Core) | 75% overall | Loader, context builder, agents |
| Phase 5 (Demo Layer) | 70% overall | Pipeline integration, UI smoke tests |
| Phase 6 (Testing) | 80% overall | Gap filling, regression tests |
| Phase 7 (Release) | 80% overall (final) | No new code; coverage must not drop |

### Running Coverage

```bash
# Basic coverage report
pytest --cov=src --cov=schemas --cov-report=term-missing

# HTML report for detailed analysis
pytest --cov=src --cov=schemas --cov-report=html

# Fail if coverage drops below threshold
pytest --cov=src --cov=schemas --cov-fail-under=80
```

---

## Edge Cases and Error Testing

### Required Edge Cases

Every module must test these categories:

| Category | Examples |
|----------|----------|
| **Empty input** | Empty string, empty file, empty list |
| **Minimal valid input** | Single-entry llms.txt, one-field model |
| **Boundary values** | Token limit exactly at budget, file size at 50MB |
| **Invalid types** | String where int expected, None where required |
| **Missing required fields** | Pydantic model missing mandatory fields |
| **Malformed input** | Truncated file, invalid YAML, broken URLs |
| **Unicode and encoding** | Non-ASCII titles, emoji in content, BOM markers |

### Error Testing Pattern

```python
def test_loader_empty_file_raises_validation_error():
    with pytest.raises(ValidationError, match="empty"):
        Loader().load("")


def test_loader_binary_file_raises_type_error():
    with pytest.raises(TypeError, match="text"):
        Loader().load(b"\x00\x01\x02")
```

---

## Integration Tests

### Separation from Unit Tests

- Integration tests live in `tests/integration/`
- They may read real files, connect multiple modules, or (with `@pytest.mark.api`) call real APIs
- They are slower and run separately from the default test suite

### Integration Test Pattern

```python
# tests/integration/test_pipeline.py

@pytest.mark.integration
def test_full_pipeline_produces_valid_context(sample_llms_txt_path):
    """End-to-end: load file -> parse -> build context -> validate output."""
    # Arrange
    loader = Loader()
    builder = ContextBuilder(token_budget=4000)

    # Act
    document = loader.load(sample_llms_txt_path)
    context = builder.build(document, query="What APIs are available?")

    # Assert
    assert context.token_count <= 4000
    assert len(context.sections) > 0
    assert context.query == "What APIs are available?"
```

---

## Test Pyramid

Target distribution of test types:

```
        ┌─────────┐
        │  E2E /  │  ~10% — Full pipeline with real files
        │   API   │  Marked: @pytest.mark.integration or @pytest.mark.api
        ├─────────┤
        │         │
        │ Integr- │  ~20% — Multi-module, real fixtures
        │  ation  │  Marked: @pytest.mark.integration
        ├─────────┤
        │         │
        │         │
        │  Unit   │  ~70% — Single function, mocked dependencies
        │         │  Marked: @pytest.mark.unit
        │         │
        └─────────┘
```

---

## Performance Testing

### Benchmark Tests

For modules with NFR performance targets:

```python
@pytest.mark.slow
def test_parse_performance_under_500ms(sample_llms_txt_path):
    """NFR-001: Parse time < 500ms for typical files."""
    import time

    loader = Loader()

    start = time.perf_counter()
    loader.load(sample_llms_txt_path)
    elapsed = time.perf_counter() - start

    assert elapsed < 0.5, f"Parse took {elapsed:.3f}s, exceeds 500ms target"


@pytest.mark.slow
def test_context_build_under_2s(sample_document):
    """NFR-002: Context build time < 2s for typical llms.txt."""
    import time

    builder = ContextBuilder(token_budget=4000)

    start = time.perf_counter()
    builder.build(sample_document, query="test query")
    elapsed = time.perf_counter() - start

    assert elapsed < 2.0, f"Context build took {elapsed:.3f}s, exceeds 2s target"
```

---

## Log Testing

Test that modules emit expected log messages using `caplog`:

```python
def test_loader_logs_file_path_on_load(caplog, sample_llms_txt_path):
    """FR-067: Loader logs loaded file at INFO level."""
    import logging

    with caplog.at_level(logging.INFO):
        Loader().load(sample_llms_txt_path)

    assert "Loading" in caplog.text
    assert str(sample_llms_txt_path) in caplog.text
```

See [Logging Standards](RR-META-logging-standards.md) for the full logging contract and test patterns.

---

## Dos and Don'ts

### Do

- Write tests alongside implementation, not after
- Run the full test suite before committing: `pytest`
- Use descriptive test names that explain the scenario
- Test error paths, not just happy paths
- Use fixtures for shared setup; avoid copy-paste
- Mark slow/integration/API tests with appropriate markers
- Keep unit tests fast (< 100ms per test)

### Don't

- Test private methods directly (test through the public API)
- Use `sleep()` in tests (mock time instead)
- Write tests that depend on execution order
- Leave `print()` statements in test code
- Catch exceptions to suppress them (let pytest report failures)
- Mock everything (some real objects are fine in unit tests)
- Write a single "mega test" that validates 10 things

---

## Acceptance Criteria (for this document)

- [ ] pytest as sole test framework with prohibited alternatives listed
- [ ] Test directory structure defined with file naming conventions
- [ ] Test naming convention with pattern and examples
- [ ] AAA (Arrange-Act-Assert) pattern documented with code examples
- [ ] Fixture patterns for conftest.py, scope, and cleanup
- [ ] Mocking standards for LLM calls, file I/O, env vars
- [ ] Custom markers defined (unit, integration, api, slow, smoke)
- [ ] Parametrized testing patterns with examples
- [ ] Per-module coverage targets aligned with NFR-010
- [ ] Phase coverage progression defined
- [ ] Edge case categories listed
- [ ] Integration test separation and patterns documented
- [ ] Test pyramid targets defined (70/20/10)
- [ ] Performance testing patterns for NFR targets
- [ ] Log testing with caplog documented
- [ ] All five standards documents cross-reference each other

---

## Related Documents

- [Logging Standards](RR-META-logging-standards.md) — Log testing with `caplog`, log format verification
- [Commenting Standards](RR-META-commenting-standards.md) — Test docstrings and test documentation
- [Documentation Requirements](RR-META-documentation-requirements.md) — Test documentation in README and CHANGELOG
- [Development Workflow](RR-META-development-workflow.md) — Tests as part of the development lifecycle (step 5)
- [NFR Specification](../01-research/RR-SPEC-v0.0.5b-non-functional-requirements-and-constraints.md) — NFR-010 (coverage), NFR-001–005 (performance targets)
- [FR Specification](../01-research/RR-SPEC-v0.0.5a-functional-requirements-specification.md) — Functional requirements that tests verify
