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

### Adaptive Rate Limit Recovery

**Problem:** Multiplicative recovery can fail to increase at low values.

```python
# WRONG - int(3 * 1.2) = 3, no increase
new_limit = int(current_limit * recovery_factor)

# CORRECT - guarantee at least +1 increase
new_limit = min(
    base_limit,
    max(current_limit + 1, int(current_limit * recovery_factor)),
)
```

### Cross-Process Rate Limiting with SQLite

**Pattern:** Use SQLite with exclusive locking for multi-process coordination.

```python
class SQLiteRateLimiter:
    def _acquire_sync(self, key: str, limit: int, window_seconds: int):
        window_start = int(time.time() // window_seconds) * window_seconds

        conn = sqlite3.connect(self._db_path, timeout=10.0, isolation_level="EXCLUSIVE")
        try:
            # Atomic upsert
            cursor = conn.execute("""
                UPDATE rate_limits SET count = count + 1
                WHERE key = ? AND window_start = ?
                RETURNING count
            """, (key, window_start))
            row = cursor.fetchone()
            if not row:
                conn.execute("INSERT INTO rate_limits VALUES (?, ?, 1)", (key, window_start))
                current = 1
            else:
                current = row[0]
            conn.commit()
        finally:
            conn.close()

        return current <= limit
```

**Key points:**
- `isolation_level="EXCLUSIVE"` ensures cross-process safety
- `timeout=10.0` handles lock contention
- Sliding window: `int(time.time() // window_seconds) * window_seconds`

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

### Mocking Decorated Methods

**Problem:** Methods wrapped with decorators (retry, resilience, rate limiting) call real APIs even when mocked.

**Why:** `importlib.reload()` with patched decorators is fragile — decorator binding happens at class definition time.

**Solution:** Mock the decorated method directly with `patch.object`:

```python
# WRONG - reload doesn't reliably rebind decorators
@patch("module.resilient_call", noop_decorator)
def test_api(self):
    importlib.reload(mod)  # Fragile, may still call real API
    client.generate("prompt")

# CORRECT - bypass decorators by mocking the method
with patch.object(client, "_call_api", return_value="mocked"):
    result = client.generate("prompt")
    assert result == "mocked"

# For async
with patch.object(client, "_call_api_async", new_callable=AsyncMock, return_value="ok"):
    result = await client.generate_async("prompt")
```

---

## Security Scanning with Bandit

### Configuration for Hardware/Systems Projects

Hardware drivers and systems code often trigger false positives. Configure appropriate skips:

```toml
[tool.bandit]
exclude_dirs = ["tests", ".venv", "build", "dist"]
skips = [
    "B101",  # assert used (tests)
    "B108",  # hardcoded /tmp paths (lock files, IPC)
    "B110",  # try/except/pass (hardware cleanup)
    "B112",  # try/except/continue (file loading loops)
    "B311",  # random module (visual effects, not crypto)
    "B404",  # subprocess import (system tools)
    "B603",  # subprocess without shell=True (safe usage)
    "B607",  # partial executable path (system binaries)
]
```

**When each skip is appropriate:**
| Skip | Use Case |
|------|----------|
| B108 | Lock files in /tmp, IPC sockets |
| B110 | Device cleanup that must not fail |
| B112 | Loading multiple config files, skip corrupt ones |
| B311 | Visual effects, shuffling, non-security randomness |
| B607 | System tools like `xdotool`, `wmctrl`, `xrandr` |

### Running Bandit

```bash
# With pyproject.toml config
bandit -r src/ -c pyproject.toml

# Quiet mode (errors only)
bandit -r src/ -c pyproject.toml -q

# Show specific severity
bandit -r src/ -c pyproject.toml -ll  # Medium+ only
```

---

## Testing Graphics/Drawing Code

### Pure Logic Testing Without Hardware

Graphics primitives (pixels, lines, shapes) can be tested without display hardware:

```python
def test_set_pixel():
    """Test pixel bit packing without hardware."""
    canvas = Canvas()
    canvas.set_pixel(0, 0, True)
    assert canvas.get_pixel(0, 0) is True
    # Verify bit-level storage
    assert canvas._buffer[0] == 0x01

def test_draw_line():
    """Test Bresenham line algorithm."""
    canvas = Canvas()
    canvas.draw_line(0, 0, 10, 0)
    for x in range(11):
        assert canvas.get_pixel(x, 0) is True

def test_out_of_bounds_ignored():
    """Boundary conditions don't crash."""
    canvas = Canvas()
    canvas.set_pixel(-1, 0, True)  # Should not raise
    canvas.set_pixel(9999, 0, True)
    assert canvas.get_pixel(-1, 0) is False
```

**Testable without hardware:**
- Pixel set/get with bit packing
- Line drawing (Bresenham)
- Rectangle fill/outline
- Progress bars
- Region inversion
- Canvas blitting
- Serialization (to_bytes/from_bytes)

**Requires mocking or hardware:**
- Actual display output
- Hardware refresh timing
- Input device events

---

## Portable Packaging for Games/Apps

### Launcher Script Pattern

For Python apps that need to run without pip install:

```bash
#!/bin/bash
# run_game.sh - Portable launcher

set -e
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
export MY_APP_ROOT="$SCRIPT_DIR"  # For resource path resolution

# Parse args
for arg in "$@"; do
    case $arg in
        --wayland) export SDL_VIDEODRIVER=wayland ;;
        --x11) export SDL_VIDEODRIVER=x11 ;;
        --help) echo "Usage: $0 [--wayland|--x11]"; exit 0 ;;
    esac
done

# Find Python 3.9+
find_python() {
    for cmd in python3 python; do
        if command -v "$cmd" &> /dev/null; then
            version=$("$cmd" -c 'import sys; print(f"{sys.version_info.major}.{sys.version_info.minor}")')
            major=$(echo "$version" | cut -d. -f1)
            minor=$(echo "$version" | cut -d. -f2)
            if [ "$major" -ge 3 ] && [ "$minor" -ge 9 ]; then
                echo "$cmd"; return 0
            fi
        fi
    done
    return 1
}

PYTHON=$(find_python) || { echo "Python 3.9+ required"; exit 1; }

# Auto-install deps if missing
if ! "$PYTHON" -c "import pygame" 2>/dev/null; then
    "$PYTHON" -m pip install --user -r "$SCRIPT_DIR/requirements.txt"
fi

cd "$SCRIPT_DIR"
exec "$PYTHON" main.py "$@"
```

### Platform Detection (Wayland/X11)

```python
# platform_init.py - Must run BEFORE pygame.init()
import os
import sys

def init_platform() -> str:
    """Configure SDL video driver before pygame."""
    if sys.platform != 'linux':
        return 'default'

    # Respect user override
    if os.environ.get('SDL_VIDEODRIVER'):
        return os.environ['SDL_VIDEODRIVER']

    # Auto-detect session type
    if os.environ.get('WAYLAND_DISPLAY'):
        driver = 'wayland'
        os.environ['SDL_VIDEODRIVER'] = driver
        os.environ.setdefault('SDL_VIDEO_WAYLAND_ALLOW_LIBDECOR', '1')
    elif os.environ.get('DISPLAY'):
        driver = 'x11'
        os.environ['SDL_VIDEODRIVER'] = driver
    else:
        driver = 'default'

    return driver
```

**Usage in main.py:**
```python
from platform_init import init_platform
init_platform()  # MUST be before pygame import

import pygame
pygame.init()
```

### Desktop Entry Installation

```bash
#!/bin/bash
# install_desktop.sh - Create menu entry

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
DESKTOP_DIR="${XDG_DATA_HOME:-$HOME/.local/share}/applications"

mkdir -p "$DESKTOP_DIR"
cat > "$DESKTOP_DIR/MyApp.desktop" << EOF
[Desktop Entry]
Version=1.0
Type=Application
Name=My App
Exec=$SCRIPT_DIR/run_game.sh
Icon=$SCRIPT_DIR/assets/icon.png
Terminal=false
Categories=Game;
EOF

chmod +x "$SCRIPT_DIR/run_game.sh"
```

**Note:** Never commit .desktop files with unexpanded `${INSTALL_DIR}` - use a template or generate at install time.

### Icon Generation from SVG

```python
# Generate PNG and ICO from SVG
import cairosvg
from PIL import Image
import io

# PNG for Linux desktop
cairosvg.svg2png(url='icon.svg', write_to='icon.png',
                 output_width=256, output_height=256)

# ICO for Windows (multiple sizes)
sizes = [16, 32, 48, 256]
images = []
for size in sizes:
    png_data = cairosvg.svg2png(url='icon.svg',
                                output_width=size, output_height=size)
    img = Image.open(io.BytesIO(png_data)).convert('RGBA')
    images.append(img)

images[-1].save('icon.ico', format='ICO', append_images=images[:-1])
```

**Dependencies:** `cairosvg`, `Pillow`

### Release Workflow (GitHub Actions)

```yaml
build-linux-portable:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v6
    - name: Package
      run: |
        mkdir -p portable/MyApp
        cp *.py portable/MyApp/
        cp -r core assets data portable/MyApp/
        cp run_game.sh install_desktop.sh requirements.txt portable/MyApp/
        chmod +x portable/MyApp/*.sh
        cd portable && tar -czvf ../MyApp-${{ github.ref_name }}-linux-portable.tar.gz MyApp
    - uses: softprops/action-gh-release@v2
      with:
        files: MyApp-${{ github.ref_name }}-linux-portable.tar.gz
```

---

## Mypy Configuration Patterns

### Strict Mode vs Practical Mode

**Problem:** `strict = true` enables all pedantic checks, causing noise from third-party libs.

**Practical config:**
```toml
[tool.mypy]
python_version = "3.11"
strict = false
warn_return_any = false
warn_unused_configs = true
disallow_untyped_defs = false
check_untyped_defs = true
ignore_missing_imports = true
```

**When to use strict:** Libraries with public APIs where type safety is critical.

**When to relax:** Applications using many third-party libs without stubs.

### Third-Party Library Stubs

**Pattern:** Add `type: ignore[import-untyped]` for libs without stubs:

```python
from weasyprint import CSS, HTML  # type: ignore[import-untyped]
import joblib  # type: ignore[import-untyped]
from sklearn.ensemble import GradientBoostingClassifier  # type: ignore[import-untyped]
from authlib.integrations.starlette_client import OAuth  # type: ignore[import-untyped]
from cachetools import TTLCache  # type: ignore[import-untyped]
```

**Common libs without stubs:** weasyprint, sklearn, joblib, authlib, cachetools

### Pydantic Field Aliases and Mypy

**Problem:** Pydantic aliases confuse mypy - using field name in constructor fails.

```python
class Response(BaseModel):
    json_str: str = Field(..., alias="json")
    model_config = {"populate_by_name": True}

# Mypy error: Unexpected keyword argument "json_str"
Response(json_str="...")  # Works at runtime, mypy complains

# Fix: Use the alias name in constructor
Response(json="...")  # Both mypy and runtime happy
```

**Alternative:** Install `pydantic` mypy plugin, but alias approach is simpler.

### Optional Type for FastAPI Dependencies

**Problem:** FastAPI dependency with default `None` needs explicit Optional type.

```python
# WRONG - mypy error: default has type "None", argument has type "Foo"
async def endpoint(dep: MyDependency = None):
    ...

# CORRECT
async def endpoint(dep: MyDependency | None = None):
    if dep is None:
        raise HTTPException(401, "Auth required")
    # Now dep is narrowed to MyDependency
    await use_dependency(dep)
```

---

## Ruff Linting Patterns

### Exception Chaining (B904)

**Problem:** `raise` inside `except` should chain exceptions.

```python
# WRONG - B904 error
try:
    value = parse(data)
except ValueError:
    raise HTTPException(400, "Invalid data")

# CORRECT - explicit chain (preserves traceback)
try:
    value = parse(data)
except ValueError as e:
    raise HTTPException(400, f"Invalid data: {e}") from e

# CORRECT - suppress chain (intentional, clean error)
try:
    value = parse(data)
except ValueError:
    raise HTTPException(400, "Invalid data") from None
```

**When to use `from e`:** When original error context is useful for debugging.
**When to use `from None`:** When you're intentionally hiding internal details (API responses).

### Import Sorting with Type Ignore

**Problem:** Ruff I001 (isort) reorders imports, breaking multi-line type: ignore.

```python
# Before ruff format (single line)
from sklearn.model_selection import cross_val_score, train_test_split  # type: ignore[import-untyped]

# After ruff format (multi-line, comment moved correctly)
from sklearn.model_selection import (  # type: ignore[import-untyped]
    cross_val_score,
    train_test_split,
)
```

**Note:** Ruff handles this correctly - the comment moves to the `from` line.

### ML Naming Conventions

**Problem:** ML code uses `X` for feature matrices, violating N803/N806 (lowercase).

```toml
[tool.ruff.lint]
ignore = [
    "E501",  # Line too long (handled by formatter)
    "N803",  # Argument name should be lowercase (X is ML convention)
    "N806",  # Variable should be lowercase (X, X_train are ML convention)
]
```

---

### Pillow Animated GIF with Per-Frame Durations

**Pattern:** Save multi-frame GIF with different durations per frame using Pillow.

```python
from PIL import Image

frames = [img1, img2, img3]  # List of PIL Image objects
durations = [4000, 5000, 3000]  # Milliseconds per frame

frames[0].save(
    "output.gif",
    save_all=True,
    append_images=frames[1:],
    duration=durations,  # List = per-frame timing
    loop=0,              # 0 = infinite loop
    optimize=True,
)
```

**Key points:**
- `duration` accepts a list for per-frame timing or a single int for uniform timing
- `loop=0` means infinite loop
- `optimize=True` reduces file size
- For further optimization, use ImageMagick: `convert -layers Optimize in.gif out.gif`

*Last updated: 2026-01-27*

### Ruff Format Before Commit (Critical)

**Problem:** Pushing code with formatting inconsistencies causes CI lint job failure.

**Error:** CI reports `Would reformat: <file.py>` from `ruff format --check` step. PR cannot merge.

**Root cause:** Developer skipped local formatting step. Ruff has specific rules for multi-line constructs (dicts, function calls, set literals, imports) that differ from single-line defaults. Test files are especially susceptible due to multi-line test data.

**Fix pattern:**

```bash
# Before committing ANY code
ruff format --check .

# If "Would reformat" appears, fix it
ruff format .

# Stage formatting changes separately
git add .
git commit -m "chore(lint): reformat code with ruff"

# Then stage actual code changes
git add .
git commit -m "feat: actual feature..."
```

**Key insight:** Formatting should be a separate commit from functional changes. Makes git history cleaner and easier to revert if needed.

**Automation:** Add to pre-commit hook:

```bash
#!/bin/bash
# .git/hooks/pre-commit

ruff format --check . && ruff check . || {
    echo "Run 'ruff format .' and 'ruff check .' before committing"
    exit 1
}
```

**Session pattern discovered:**
- CI failed on first push: "Would reformat: <file>"
- Added follow-up `chore(lint)` commit to fix
- Pattern: Always run both `ruff format .` and `ruff check .` before `git push`
- Test files with multi-line dicts/function calls particularly prone to this

