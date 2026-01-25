# CI/CD & GitHub Actions

Patterns and fixes for continuous integration.

---

## Python Dependency Conflicts

**Problem:** Explicit sub-dependency versions conflict with framework requirements.

**Example:** `starlette>=0.52.0` conflicted with `fastapi>=0.128.0` (requires starlette<0.51.0)

**Solution:** Let frameworks manage their own dependencies. Remove explicit versions of sub-dependencies from requirements.txt.

---

## Firebase Initialization in CI

**Problem:** `Firebase: Error (auth/invalid-api-key)` during build when env vars aren't set.

**Solution:** Guard initialization:
```javascript
const canInitialize = !!process.env.NEXT_PUBLIC_FIREBASE_API_KEY;
export const app = canInitialize ? initializeApp(firebaseConfig) : null;
export const auth = app ? getAuth(app) : null;
```

---

## PyGObject Installation

**Problem:** System packages (`python3-gi`) not available to `actions/setup-python` Python.

**Solution:**
```yaml
- name: Install system dependencies
  run: |
    sudo apt-get install -y \
      libgirepository1.0-dev \
      libcairo2-dev \
      pkg-config \
      python3-dev \
      gir1.2-gtk-3.0

- name: Install test dependencies
  run: pip install "PyGObject<3.50" pycairo
```

**Note:** Pin `PyGObject<3.50` - version 3.50+ requires girepository-2.0 not available on Ubuntu 24.04.

---

## Headless GUI Testing

**Problem:** GTK/Qt tests fail without display in CI.

**Solution:**
```yaml
- run: sudo apt-get install -y xvfb
- run: xvfb-run -a pytest tests/
```

For Qt apps:
```yaml
env:
  QT_QPA_PLATFORM: offscreen
```

---

## pynput X11 Errors

**Problem:** `Xlib.error.ConnectionClosedError` - pynput keyboard.Listener needs real X11.

**Solution:** Mock the listener in tests:
```python
with patch("module.keyboard.Listener"):
    widget._start_recording()
```

---

## Temp Directory Fallback

**Problem:** `PermissionError` when creating paths like `/shared_data/` in CI.

**Solution:**
```python
import tempfile

def _get_default_path() -> Path:
    env_path = os.environ.get("MY_PATH")
    if env_path:
        return Path(env_path)

    shared_path = Path("/shared_data/file.db")
    if shared_path.parent.exists():
        return shared_path

    # Fallback for CI/dev
    return Path(tempfile.gettempdir()) / "file.db"
```

---

## Locale Differences

**Problem:** Tests expected 2-char language codes but CI returns `'C'` locale.

**Solution:**
```python
def test_returns_language_code(self):
    result = get_system_language()
    assert result.islower() or result == "C"  # Accept 'C' in minimal CI
```

---

## Ruff Formatting

**Problem:** CI fails on `ruff format --check`.

**Solution:** Always run before committing:
```bash
ruff check . --fix
ruff format .
```

---

## Dependabot Management

### Rebasing Stale PRs
When merging multiple Dependabot PRs that touch the same files:
```
@dependabot rebase
```

### Safe vs Risky Updates
- **Safe to auto-merge:** Minor/patch versions, CI action updates
- **Review first:** Major version bumps

---

## GitHub CLI Commands

```bash
# List failing CI
gh run list --repo OWNER/REPO --branch main --json conclusion

# View failure logs
gh run view RUN_ID --repo OWNER/REPO --log-failed

# Merge PR
gh pr merge NUM --repo OWNER/REPO --squash --delete-branch

# Request Dependabot rebase
gh pr comment NUM --repo OWNER/REPO --body "@dependabot rebase"

# Delete remote branch
gh api -X DELETE repos/OWNER/REPO/git/refs/heads/BRANCH_NAME
```

---

## GitHub Actions Multiline Outputs

**Problem:** `EOF` delimiter in multiline outputs can conflict with content.

**Error:** `Matching delimiter not found 'EOF'`

**Solution:** Use unique delimiters:
```yaml
- name: Generate changelog
  id: changelog
  run: |
    {
      echo 'CHANGELOG<<CHANGELOG_EOF'
      git log --pretty=format:"- %s (%h)" | head -50
      echo ''
      echo 'CHANGELOG_EOF'
    } >> $GITHUB_OUTPUT
```

---

## PyPI Trusted Publishing

**Setup:** Configure at https://pypi.org/manage/project/PROJECTNAME/settings/publishing/

**Required workflow settings:**
```yaml
permissions:
  id-token: write  # Required for OIDC

jobs:
  publish:
    environment:
      name: pypi
      url: https://pypi.org/project/PROJECTNAME/
    steps:
      - uses: pypa/gh-action-pypi-publish@release/v1
        # No API token needed with OIDC
```

**Note:** Must create `pypi` environment in GitHub repo settings and configure trusted publisher on PyPI first.

---

## Key Principles

1. **CI environments are minimal** - Don't assume system packages, displays, or writable paths
2. **Mock external dependencies** - pynput, Firebase, etc. should be mocked in CI
3. **Let frameworks manage sub-dependencies** - Don't pin starlette if using FastAPI
4. **Run formatters locally** - Catch ruff/prettier issues before pushing
5. **Batch Dependabot merges carefully** - Merge one at a time or request rebases

---

*Last updated: 2026-01-24*
