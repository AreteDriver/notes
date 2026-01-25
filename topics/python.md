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

### Async Generator Exception Testing

**Problem:** Testing exception handling in async generators that use `async for`.

**Wrong approach:** `async for` doesn't properly test exception paths.

**Correct approach:** Use generator's `athrow` method:

```python
async def test_generator_rollback_on_exception():
    gen = get_db()  # Returns async generator
    session = await gen.__anext__()  # Get yielded session

    # Inject exception into generator
    with pytest.raises(ValueError):
        await gen.athrow(ValueError("test error"))

    # Now verify rollback was called
    session.rollback.assert_called_once()
```

### AsyncMock vs MagicMock for Context Managers

**Problem:** `AsyncMock` coroutine doesn't support async context manager protocol.

**Error:** `TypeError: 'coroutine' object does not support the async context manager protocol`

**Solution:** Use `MagicMock` for the context manager, `AsyncMock` for the async methods:

```python
# WRONG
mock_engine = AsyncMock()
mock_engine.begin.return_value = mock_context  # Coroutine, not context manager

# CORRECT
mock_context = MagicMock()
mock_context.__aenter__ = AsyncMock(return_value=mock_conn)
mock_context.__aexit__ = AsyncMock(return_value=None)

mock_engine = MagicMock()
mock_engine.begin.return_value = mock_context  # Proper async context manager
```

### Mocking httpx.AsyncClient

**Pattern for testing code that uses httpx:**

```python
async def test_httpx_call():
    mock_response = MagicMock()
    mock_response.status_code = 200
    mock_response.json.return_value = {"data": "test"}
    mock_response.raise_for_status = MagicMock()

    mock_client = AsyncMock()
    mock_client.get.return_value = mock_response
    mock_client.__aenter__.return_value = mock_client
    mock_client.__aexit__.return_value = None

    with patch("mymodule.httpx.AsyncClient", return_value=mock_client):
        result = await fetch_data()
        assert result == {"data": "test"}
```

### Mocking Redis with Byte Strings

**Problem:** Redis returns byte strings, not regular strings.

```python
@pytest.fixture
def mock_redis():
    mock = AsyncMock()
    # Redis returns bytes
    mock.hgetall.return_value = {
        b"character_id": b"12345",
        b"access_token": b"token",
        b"refresh_token": b"refresh",
    }
    mock.get.return_value = b'{"cached": "data"}'
    return mock
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
| fpdf2 can't encode `•` bullet | Use ASCII `-` - fpdf2 defaults to latin-1 encoding |

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

---

## PDF Generation with fpdf2

**Problem:** Unicode characters like `•` cause encoding errors.

**Error:** `UnicodeEncodeError: 'latin-1' codec can't encode character '\u2022'`

**Solution:** Use ASCII alternatives:
```python
# BAD
pdf.cell(txt="• Item one")

# GOOD
pdf.cell(txt="- Item one")
```

Or add a Unicode font:
```python
pdf.add_font("DejaVu", fname="/path/to/DejaVuSans.ttf")
pdf.set_font("DejaVu", size=12)
```

---

## Async Patterns

### Native Async with SDK Clients

**Pattern:** Use native async clients instead of wrapping sync in executor.

```python
# WRONG - wastes thread pool
async def complete_async(self, request):
    loop = asyncio.get_event_loop()
    return await loop.run_in_executor(None, self.complete, request)

# CORRECT - native async
from anthropic import AsyncAnthropic
self._async_client = AsyncAnthropic(api_key=api_key)

async def complete_async(self, request):
    response = await self._async_client.messages.create(...)
    return response
```

### Per-Provider Rate Limiting with Semaphores

**Pattern:** Use asyncio semaphores to limit concurrent API calls per provider.

```python
class RateLimitedExecutor:
    def __init__(self):
        self._semaphores = {
            "anthropic": asyncio.Semaphore(5),   # ~60 RPM safe
            "openai": asyncio.Semaphore(8),      # ~90 RPM safe
        }

    async def call_with_limit(self, provider: str, coro):
        semaphore = self._semaphores.get(provider, asyncio.Semaphore(10))
        async with semaphore:
            return await coro
```

### Async Subprocess Execution

**Pattern:** Use `asyncio.create_subprocess_exec` for async CLI calls.

```python
async def run_cli_async(cmd: list[str], timeout: int = 300) -> str:
    process = await asyncio.create_subprocess_exec(
        *cmd,
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE,
    )
    try:
        stdout, stderr = await asyncio.wait_for(
            process.communicate(), timeout=timeout
        )
    except asyncio.TimeoutError:
        process.kill()
        await process.wait()
        raise RuntimeError(f"Command timed out after {timeout}s")

    if process.returncode != 0:
        raise RuntimeError(f"Command failed: {stderr.decode()}")
    return stdout.decode()
```

### Lazy Semaphore Creation

**Problem:** Semaphores must be created in async context (same event loop).

```python
# WRONG - created at __init__ before event loop exists
def __init__(self):
    self._semaphore = asyncio.Semaphore(5)  # May fail or wrong loop

# CORRECT - lazy creation
def __init__(self):
    self._semaphores = None

def _get_semaphores(self):
    if self._semaphores is None:
        self._semaphores = {
            "provider": asyncio.Semaphore(5)
        }
    return self._semaphores
```

---

## Deprecation Patterns

### Deprecation Warning with Guidance

**Pattern:** Emit warnings that guide users to the replacement.

```python
import warnings

class OldClass:
    def __init__(self):
        warnings.warn(
            "OldClass is deprecated. Use NewClass from module.new instead. "
            "NewClass provides X, Y, Z improvements.",
            DeprecationWarning,
            stacklevel=2,  # Point to caller, not this line
        )
```

### Adapter for Incremental Migration

**Pattern:** Wrap new implementation with old interface for gradual migration.

```python
class OldEngineAdapter:
    """Provides OldEngine interface using NewExecutor internally."""

    def __init__(self):
        self._executor = NewExecutor()

    def old_method(self, old_obj: OldFormat) -> OldResult:
        # Convert old format to new
        new_config = convert_old_to_new(old_obj)

        # Execute with new implementation
        new_result = self._executor.execute(new_config)

        # Convert result back to old format
        return convert_result_to_old(new_result)
```

**Benefits:**
- Same interface = minimal changes at call sites
- Migrate one file at a time
- Deprecation warnings guide without breaking

---

## Testing Patterns

### Kwargs Passthrough in Handlers

**Problem:** Dynamic executors may pass extra kwargs to handlers.

```python
# WRONG - fails if executor adds kwargs
def handler():
    return "done"

# CORRECT - accept and ignore extra kwargs
def handler(**kwargs):
    return "done"
```

### Test File Side Effects

**Problem:** Adding files to directories scanned at startup can break tests.

**Example:** Adding `test-workflow.yaml` to a workflows directory that's auto-imported on app startup causes tests expecting empty state to fail.

**Solution:** Use temporary directories or cleanup in test fixtures.

### Mock Paths During Migration

**Problem:** After migrating from `OldClass` to `NewAdapter`, tests fail because mocks still patch `OldClass`.

**Error:** `AttributeError: <module 'mymodule'> does not have the attribute 'OldClass'`

**Solution:** Update all test mock paths to match new imports:

```python
# Before migration
with patch("mymodule.OldClass") as mock:
    ...

# After migration (module imports NewAdapter now)
with patch("mymodule.NewAdapter") as mock:
    ...
```

**Tip:** Use `grep -r "OldClass" tests/` to find all mocks that need updating.

---

*Last updated: 2026-01-25*
