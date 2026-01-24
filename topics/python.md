# Python Notes

Patterns, gotchas, and solutions for Python development.

---

## Testing with pytest

### Coverage Memory Issue

**Problem:** `pytest --cov` can use 15-50GB RAM on large codebases.

**Solution:**
- Use `--no-cov` for local development
- Run coverage only in CI or targeted files
- If pyproject.toml has `addopts = "--cov=..."`, override with `--no-cov`

```bash
# Fast local run
pytest tests/test_specific.py --no-cov -v

# CI with coverage
pytest --cov=mypackage --cov-report=xml
```

### Mocking Import Gotchas

**Problem:** Tests hang when mock patches wrong target.

**Root cause:** When code does `from x import y` inside a function, you must patch `x.y` (source module), not the calling module.

**Example:**
```python
# Code in cli.py
def cmd_run(args):
    if args.simple:
        from .device import open_g13  # Imported inside function
        h = open_g13()
        while True:
            data = h.read(timeout_ms=100)  # h.read, not read_event
            ...
```

```python
# WRONG - patches non-existent attribute
patch("g13_linux.cli.open_g13")  # Error: cli has no open_g13

# WRONG - patches wrong function
patch("g13_linux.device.read_event")  # Code calls h.read(), not read_event

# CORRECT - patch at source module
patch("g13_linux.device.open_g13")
mock_handle.read.side_effect = KeyboardInterrupt  # Mock the method on returned object
```

### Testing Infinite Loops

**Pattern:** Mock the iteration source to eventually raise exception.

```python
def test_event_loop():
    mock_handle = MagicMock()
    call_count = [0]

    def fake_read(timeout_ms=100):
        call_count[0] += 1
        if call_count[0] >= 3:
            raise KeyboardInterrupt
        return b"\x00" * 8

    mock_handle.read.side_effect = fake_read
    # Test will exit after 3 iterations
```

---

## Project Structure

### Modern Packaging (pyproject.toml)

```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "mypackage"
version = "1.0.0"
dependencies = ["dep1>=1.0"]

[project.optional-dependencies]
dev = ["pytest", "ruff", "mypy"]

[project.scripts]
mycli = "mypackage.cli:main"
```

### Entry Points for CLI + GUI

```toml
[project.scripts]
myapp = "mypackage.cli:main"
myapp-gui = "mypackage.gui:main"
```

---

## Common Gotchas

| Issue | Fix |
|-------|-----|
| Import inside function not mockable at module level | Patch at source module |
| `MagicMock().method` vs `MagicMock.method` | Former is instance, latter is class |
| pytest fixtures not found | Check conftest.py location |
| Coverage excludes files | Check `[tool.coverage.run]` source setting |
| `datetime.utcnow()` deprecated (Python 3.12+) | Use `datetime.now(timezone.utc)` |

---

## Datetime Deprecation (Python 3.12+)

`datetime.utcnow()` is deprecated and will be removed in Python 3.14.

```python
# OLD (deprecated)
from datetime import datetime
timestamp = datetime.utcnow()

# NEW (timezone-aware)
from datetime import datetime, timezone
timestamp = datetime.now(timezone.utc)
```

For ISO format strings:
```python
# OLD
datetime.utcnow().isoformat() + "Z"

# NEW
datetime.now(timezone.utc).isoformat()  # Includes +00:00
# Or for explicit Z suffix:
datetime.now(timezone.utc).strftime("%Y-%m-%dT%H:%M:%SZ")
```

---

*Last updated: 2026-01-24*
