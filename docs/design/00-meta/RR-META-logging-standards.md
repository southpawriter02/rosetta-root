# Logging Standards — DocStratum

<aside>

**Scope:** All phases (v0.1.x through v0.6.x)

**Status:** Active

**Applies To:** All Python modules in `src/`, `schemas/`, and any scripts that produce operational output

**Deliverable:** Enforceable logging patterns, level contracts, format specifications, and security rules for all runtime log output

</aside>

---

## Purpose

This document defines logging standards for the DocStratum project. It establishes the logging framework, format, level contracts, and security rules that apply to all modules.

FR-067 requires all modules to log key decisions at INFO level. NFR-006 requires clear, actionable error messages. This document operationalizes both requirements with specific patterns and enforcement rules.

---

## Logging Philosophy

### Core Principles

1. **Logs are operational documentation.** They tell the story of what happened at runtime.
2. **Every key decision is logged.** File loaded, entries parsed, context selected, query answered — all logged at INFO.
3. **Errors are actionable.** Every ERROR log tells the operator what happened and what to do about it.
4. **No secrets in logs.** API keys, tokens, and credentials are never logged, even at DEBUG.
5. **Structure over prose.** Use consistent format strings, not conversational sentences.
6. **Performance-safe.** Use lazy `%s` formatting. Never evaluate expensive expressions just for a log message.

---

## Framework: Python Standard Library Only

### Required

```python
import logging

logger = logging.getLogger(__name__)
```

### Prohibited

| Library | Why Prohibited |
|---------|---------------|
| `loguru` | Non-standard; breaks centralized configuration |
| `structlog` | Adds unnecessary dependency for this project's scope |
| `print()` | No level, no format, no control. Prohibited in `src/` modules. |
| `sys.stdout.write()` | Same problems as `print()` |

### Exception

`print()` is allowed in:
- `verify_setup.py` (user-facing setup script)
- CLI entry points (before logging is configured)
- Test files (use `caplog` instead when testing log output)

---

## Logger Configuration

### Centralized Setup

All logging configuration lives in a single module:

```python
# src/logging_config.py

import logging
import os

LOG_FORMAT = "%(asctime)s | %(levelname)-8s | %(name)s:%(funcName)s:%(lineno)d | %(message)s"
LOG_DATE_FORMAT = "%Y-%m-%d %H:%M:%S"


def setup_logging(level: str | None = None) -> None:
    """Configure logging for the DocStratum application.

    Args:
        level: Override log level. If None, reads from DOCSTRATUM_LOG_LEVEL
               environment variable, defaulting to INFO.
    """
    log_level = level or os.getenv("DOCSTRATUM_LOG_LEVEL", "INFO")

    logging.basicConfig(
        level=getattr(logging, log_level.upper(), logging.INFO),
        format=LOG_FORMAT,
        datefmt=LOG_DATE_FORMAT,
    )

    # Suppress noisy third-party loggers
    logging.getLogger("httpx").setLevel(logging.WARNING)
    logging.getLogger("openai").setLevel(logging.WARNING)
    logging.getLogger("langchain").setLevel(logging.WARNING)
```

### Per-Module Logger Pattern

Every module creates its own logger using `__name__`:

```python
# src/loader.py

import logging

logger = logging.getLogger(__name__)


class Loader:
    def load(self, path: str) -> Document:
        logger.info("Loading llms.txt from %s", path)
        ...
```

### Environment Configuration

| Variable | Default | Options |
|----------|---------|---------|
| `DOCSTRATUM_LOG_LEVEL` | `INFO` | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` |

---

## Log Format

### Standard Format String

```
%(asctime)s | %(levelname)-8s | %(name)s:%(funcName)s:%(lineno)d | %(message)s
```

### Example Output

```
2026-02-05 14:32:01 | INFO     | src.loader:load:42 | Loading llms.txt from /data/example.txt
2026-02-05 14:32:01 | INFO     | src.loader:load:58 | Parsed 47 entries in 0.12s
2026-02-05 14:32:02 | DEBUG    | src.context:build:31 | Token budget: 4000, estimated usage: 3200
2026-02-05 14:32:02 | WARNING  | src.validator:check:89 | Section "APIs" has no entries — skipping
2026-02-05 14:32:03 | ERROR    | src.agent:answer:112 | LLM call failed: rate_limit_exceeded — retrying in 5s
```

### Format Rules

- **Pipe delimiters** (`|`) separate timestamp, level, location, and message
- **Left-aligned level** (`%-8s`) for visual alignment
- **Module:function:line** for precise log location
- **ISO-8601 timestamps** (`%Y-%m-%d %H:%M:%S`)

---

## Level Contract

### Level Definitions

| Level | When to Use | Example | Action Required |
|-------|-------------|---------|-----------------|
| **CRITICAL** | System cannot continue; data loss imminent | `"Configuration file missing — cannot start"` | Fix immediately; system is down |
| **ERROR** | Operation failed; module degraded but system continues | `"LLM call failed: %s — falling back to baseline"` | Investigate; may need intervention |
| **WARNING** | Unexpected condition; operation succeeded with caveats | `"Section '%s' has no entries — skipping"` | Review; may indicate upstream issue |
| **INFO** | Key operational events; the "audit trail" | `"Loaded %d entries from %s in %.2fs"` | Normal; used for monitoring and debugging |
| **DEBUG** | Detailed internal state; developer troubleshooting | `"Token estimate for section '%s': %d tokens"` | Off by default; enable when investigating |

### INFO Level Contract (FR-067)

Every module must log these events at INFO level:

| Module | Required INFO Messages |
|--------|----------------------|
| **Loader** | File path loaded, entry count parsed, parse duration |
| **Validator** | Validation level applied, issues found count, pass/fail |
| **Context Builder** | Token budget, sections selected, final token count |
| **Agent (Baseline)** | Query received, LLM provider used, response time |
| **Agent (DocStratum)** | Query received, context injected (token count), response time |
| **A/B Harness** | Test started, query count, results summary |
| **Demo App** | App started, page loaded, user action |

### Level Selection Decision Tree

```
Is the system unable to continue?
  └── Yes → CRITICAL

Did an operation fail?
  └── Yes → ERROR

Did something unexpected happen but the operation succeeded?
  └── Yes → WARNING

Is this a key operational event (load, parse, build, query, respond)?
  └── Yes → INFO

Is this internal state useful only for debugging?
  └── Yes → DEBUG
```

---

## Message Formatting

### Use `%s`-Style (Lazy Evaluation)

```python
# CORRECT — lazy evaluation; string formatting only happens if level is enabled
logger.info("Loaded %d entries from %s in %.2fs", count, path, elapsed)
logger.debug("Token estimate for section '%s': %d tokens", section.title, tokens)
logger.error("LLM call failed: %s — retrying in %ds", error, delay)

# WRONG — f-string always evaluates, even if level is disabled
logger.info(f"Loaded {count} entries from {path} in {elapsed:.2f}s")

# WRONG — .format() always evaluates
logger.info("Loaded {} entries from {}".format(count, path))

# WRONG — string concatenation
logger.info("Loaded " + str(count) + " entries")
```

### Message Style Rules

1. **Start with a verb or noun:** `"Loading..."`, `"Parsed 47 entries"`, `"Token budget: 4000"`
2. **Include measurable values:** counts, durations, sizes — not just "done"
3. **Include identifiers:** file paths, section names, query text (truncated if long)
4. **Use consistent vocabulary:** "Loading" / "Loaded" (not "Reading" / "Fetching" / "Grabbing")

### Vocabulary Table

| Action | Present Tense (start) | Past Tense (complete) |
|--------|----------------------|----------------------|
| File I/O | Loading | Loaded |
| Parsing | Parsing | Parsed |
| Validation | Validating | Validated |
| Context building | Building context | Built context |
| LLM call | Calling LLM | LLM responded |
| Query processing | Processing query | Processed query |

---

## Security: No Secrets in Logs

### Masking Pattern

```python
def mask_secret(value: str, visible_chars: int = 4) -> str:
    """Mask a secret value, showing only the last N characters.

    Args:
        value: The secret string to mask.
        visible_chars: Number of trailing characters to show.

    Returns:
        Masked string like '****abcd'.
    """
    if len(value) <= visible_chars:
        return "****"
    return "****" + value[-visible_chars:]
```

### Usage

```python
logger.info("Using API key %s for provider %s", mask_secret(api_key), provider)
# Output: Using API key ****a1b2 for provider openai
```

### Rules

1. **Never log raw API keys**, tokens, passwords, or credentials
2. **Never log full URLs with query parameters** that may contain tokens
3. **Use `mask_secret()`** when logging any reference to a credential
4. **Audit log output** before release — search for `sk-`, `key=`, `token=`, `password=`

---

## Error Logging

### Error Log Pattern

```python
try:
    result = call_llm(prompt, provider=provider)
except RateLimitError as e:
    logger.error("LLM rate limit exceeded for %s: %s — retrying in %ds", provider, e, delay)
    raise
except ConnectionError as e:
    logger.error("Failed to connect to %s: %s — check network and API status", provider, e)
    raise
```

### Rules

1. **Log before re-raising:** The log captures context that the exception may not carry.
2. **Include the exception message:** Use `%s` with the exception object.
3. **Include remediation:** "retrying in Ns", "check network", "see Troubleshooting section".
4. **Don't log and swallow:** If you catch an exception, either re-raise it or handle it — don't just log and continue silently.

---

## Testing Log Output

### Using `caplog`

```python
def test_loader_logs_entry_count(caplog, sample_llms_txt_path):
    """Verify the loader logs the number of parsed entries at INFO level."""
    import logging

    with caplog.at_level(logging.INFO):
        Loader().load(sample_llms_txt_path)

    # Assert specific content in log messages
    assert any("Parsed" in record.message and "entries" in record.message
               for record in caplog.records)


def test_loader_logs_at_correct_level(caplog, sample_llms_txt_path):
    """Verify log messages use appropriate levels."""
    import logging

    with caplog.at_level(logging.DEBUG):
        Loader().load(sample_llms_txt_path)

    info_messages = [r for r in caplog.records if r.levelno == logging.INFO]
    debug_messages = [r for r in caplog.records if r.levelno == logging.DEBUG]

    assert len(info_messages) >= 1, "Expected at least one INFO message from Loader"
```

### Log Testing Rules

1. Use `caplog` (pytest built-in), not custom log capture
2. Test both message content and log level
3. Don't assert on exact message strings (fragile) — assert on key substrings
4. Test that sensitive data does NOT appear in logs

See [Testing Standards](RR-META-testing-standards.md) for full test patterns and fixtures.

---

## Module Logging Checklist

Before considering a module complete, verify:

- [ ] Module creates logger with `logging.getLogger(__name__)`
- [ ] No `print()` statements in module code
- [ ] All key operational events logged at INFO (per FR-067 table above)
- [ ] All error paths log at ERROR with context and remediation
- [ ] No secrets appear in any log message (audit for `sk-`, `key=`, `token=`)
- [ ] `%s`-style formatting used (no f-strings in log calls)
- [ ] At least one `caplog` test verifies expected log output

---

## Dos and Don'ts

### Do

- Use `logging.getLogger(__name__)` in every module
- Log the start and completion of key operations (with timing)
- Include counts, identifiers, and durations in log messages
- Use `%s`-style lazy formatting in all log calls
- Mask credentials with `mask_secret()` before logging
- Test log output with `caplog`
- Centralize configuration in `logging_config.py`

### Don't

- Use `print()` in `src/` modules
- Use f-strings or `.format()` in log calls
- Log raw API keys, tokens, or passwords
- Log at WARNING or ERROR for normal operations
- Create logger objects with hardcoded names (`getLogger("my_module")`)
- Log stack traces at INFO level (use ERROR or DEBUG)
- Add third-party logging libraries

---

## Acceptance Criteria (for this document)

- [ ] Python `logging` stdlib mandated; alternatives explicitly prohibited
- [ ] Centralized `setup_logging()` pattern defined
- [ ] Per-module `getLogger(__name__)` pattern documented
- [ ] Log format string specified with example output
- [ ] Five-level contract defined with examples and action guidance
- [ ] INFO-level contract per module aligned with FR-067
- [ ] `%s`-style formatting mandated; f-string/format() prohibited in log calls
- [ ] Secret masking pattern with `mask_secret()` provided
- [ ] Error logging pattern with remediation guidance
- [ ] `caplog` testing pattern documented
- [ ] Module logging checklist provided
- [ ] All five standards documents cross-reference each other

---

## Related Documents

- [Testing Standards](RR-META-testing-standards.md) — Log testing with `caplog`, test fixtures
- [Commenting Standards](RR-META-commenting-standards.md) — Docstrings for logging functions
- [Documentation Requirements](RR-META-documentation-requirements.md) — Operational documentation via logs
- [Development Workflow](RR-META-development-workflow.md) — Logging as part of the development lifecycle (step 6)
- [NFR Specification](../01-research/RR-SPEC-v0.0.5b-non-functional-requirements-and-constraints.md) — NFR-006 (error messages), NFR-013 (documentation-to-code ratio)
- [FR Specification](../01-research/RR-SPEC-v0.0.5a-functional-requirements-specification.md) — FR-067 (logging requirement)
