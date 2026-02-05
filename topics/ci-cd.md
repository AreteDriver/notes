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

### Ruff Version Mismatch

**Problem:** Local ruff version differs from CI, causing "Would reformat" failures even after running `ruff format .` locally.

**Symptom:** CI fails with formatting errors on files you just formatted. Different ruff versions have slightly different formatting rules.

**Solution:** Keep local ruff version in sync with CI:
```bash
# Check versions
ruff --version           # Local
# CI typically uses latest via pip install ruff

# Upgrade local (if using pipx)
pipx upgrade ruff

# Or reinstall
pip install --upgrade ruff
```

**Pattern:** When CI uses `pip install ruff` without pinning, it gets the latest. If your local is behind, formatting will differ. Either:
1. Keep local ruff updated (`pipx upgrade ruff` regularly)
2. Pin ruff version in CI to match local
3. Add ruff version to pyproject.toml dev deps and install from there

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

### Blocking Major Version Bumps
After Next.js 14→16 and ESLint 8→9 broke CI via auto-merge:

```yaml
# dependabot.yml
ignore:
  - dependency-name: "next"
    update-types: ["version-update:semver-major"]
  - dependency-name: "eslint"
    update-types: ["version-update:semver-major"]
  - dependency-name: "eslint-config-next"
    update-types: ["version-update:semver-major"]
```

**Rule:** Any package with breaking migration steps (Next.js, ESLint, React, TypeScript) should have major bumps ignored in dependabot. Upgrade these manually on a branch with a migration plan.

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

## Concurrency Groups

**Problem:** Rapid pushes waste Actions minutes running duplicate workflows.

**Solution:** Add concurrency groups to cancel in-progress runs:
```yaml
on:
  push:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

**Applied to:** All repos (2026-01-27). RedOPS Security workflow also got this.

---

## Actions Budget Exhaustion

**Problem:** "The job was not started because an Actions budget is preventing further use." All jobs fail with this message when monthly free tier minutes are consumed.

**Symptoms:** Every workflow across all repos fails simultaneously. Not a code issue.

**Mitigations:**
- Add concurrency groups (prevents duplicate runs)
- Avoid matrix builds unless needed (e.g., 3 Python versions = 3x minutes)
- Use `paths` filters to skip CI on docs-only changes
- Cache pip/cargo/node_modules aggressively
- Budget resets on billing cycle (check Settings > Billing)

---

## CodeQL Action Deprecation

**Problem:** CodeQL Action v3 deprecated December 2026.

**Solution:** Upgrade all `github/codeql-action/*@v3` to `@v4`:
```yaml
- uses: github/codeql-action/init@v4
- uses: github/codeql-action/autobuild@v4
- uses: github/codeql-action/analyze@v4
- uses: github/codeql-action/upload-sarif@v4
```

---

## TruffleHog Action Version

**Problem:** `trufflesecurity/trufflehog@main` is unstable — can break without warning.

**Solution:** Pin to a release tag:
```yaml
- uses: trufflesecurity/trufflehog@v3.88.0
```

### TruffleHog Base/Head for Push Events

**Problem:** `base: ${{ github.ref }}` and `head: ${{ github.ref }}` causes "BASE and HEAD are the same" error on push events.

**Solution:** Use event-specific SHAs:
```yaml
- uses: trufflesecurity/trufflehog@v3.88.0
  with:
    base: ${{ github.event.before || github.event.pull_request.base.sha }}
    head: ${{ github.event.after || github.event.pull_request.head.sha }}
```

---

## Bandit SARIF Upload Failure

**Problem:** `Path does not exist: bandit-results.sarif` — upload step fails when Bandit finds no issues or errors out.

**Solution:**
```yaml
- name: Run Bandit
  run: bandit -r src/package_name/ -f sarif -o bandit-results.sarif || true

- name: Upload Bandit results
  uses: github/codeql-action/upload-sarif@v4
  if: always() && hashFiles('bandit-results.sarif') != ''
  with:
    sarif_file: 'bandit-results.sarif'
```

**Key:** Use `hashFiles()` conditional to skip upload when file doesn't exist.

---

## Missing pyproject.toml Extras in CI

**Problem:** `pip install -e ".[dev,test]"` silently ignores non-existent extras (e.g., `test` doesn't exist).

**Solution:** Check actual extras with:
```python
import tomllib
with open('pyproject.toml', 'rb') as f:
    d = tomllib.load(f)
print(d['project']['optional-dependencies'].keys())
```

Match CI install command to actual extras. RedOPS fix: `[dev,test]` → `[dev,web,full]`.

---

## Paths-Ignore Filters

**Problem:** CI runs on every push, even docs-only changes (README edits, LICENSE updates), wasting Actions minutes.

**Solution:** Add `paths-ignore` to skip CI on non-code changes:
```yaml
on:
  push:
    branches: [main]
    paths-ignore:
      - '*.md'
      - 'docs/**'
      - 'LICENSE'
      - '.gitignore'
      - '.github/dependabot.yml'
  pull_request:
    branches: [main]
    paths-ignore:
      - '*.md'
      - 'docs/**'
      - 'LICENSE'
      - '.gitignore'
      - '.github/dependabot.yml'
```

**For Rust repos**, also add `- 'assets/**'` (non-code static files).

**Applied to:** 11 repos (2026-01-27). Combined with concurrency groups, this significantly reduces wasted Actions minutes.

---

## wasm-bindgen CLI Version Mismatch

**Problem:** `cargo install wasm-bindgen-cli` installs latest version, but `Cargo.lock` pins an older `wasm-bindgen` crate. Schema versions must match exactly.

**Error:** `rust Wasm file schema version: 0.2.106 / this binary schema version: 0.2.108`

**Solution:** Pin the CLI version to match `Cargo.lock`:
```yaml
- name: Install wasm-bindgen-cli
  run: cargo install wasm-bindgen-cli --version 0.2.106
```

Check locked version with: `grep -A1 'name = "wasm-bindgen"' Cargo.lock`

**Applied to:** eve_rebellion_rust (2026-01-27). Pin in ci.yml, pages.yml, and release.yml.

---

## Key Principles

1. **CI environments are minimal** - Don't assume system packages, displays, or writable paths
2. **Mock external dependencies** - pynput, Firebase, etc. should be mocked in CI
3. **Let frameworks manage sub-dependencies** - Don't pin starlette if using FastAPI
4. **Run formatters locally** - Catch ruff/prettier issues before pushing
5. **Batch Dependabot merges carefully** - Merge one at a time or request rebases
6. **Pin security action versions** - Never use `@main` for TruffleHog/Trivy/etc.
7. **Add concurrency groups** - Every CI workflow should cancel duplicate runs
8. **Verify extras exist** - pip silently ignores non-existent optional-dependencies
9. **Use paths-ignore** - Skip CI on docs-only changes to conserve Actions minutes
10. **Pin wasm-bindgen-cli** - Must match Cargo.lock version exactly

---

*Last updated: 2026-01-27*

## GitHub Branch Protection Configuration

**Setup:** After upgrading to GitHub Pro, configure branch protection rules on all private repos via GitHub API.

**Pattern:** Use `gh api` to put branch protection:
```bash
gh api repos/{OWNER}/{REPO}/branches/main/protection \
  --method PUT \
  -f required_status_checks:strict=true \
  -f required_status_checks:contexts='["CI","Security","Tests (3.10)"]' \
  -f enforce_admins=false \
  -f allow_force_pushes=false \
  -f allow_deletions=false \
  -f require_code_reviews=false
```

### Key Parameters
- `enforce_admins=false` — Critical for solo devs. Allows you to bypass protection in emergencies without removing protection
- `required_status_checks:strict=true` — Branch must be up-to-date before merge (prevents stale CI)
- `required_status_checks:contexts` — List of required check names, must match exact job names from CI workflow

### Discovering CI Job Names
```bash
# Get last run ID
gh run list -R owner/repo --workflow=.github/workflows/ci.yml --limit 1 --json databaseId --jq '.[0].databaseId'

# View exact job names (including matrix suffixes)
gh run view {RUN_ID} -R owner/repo --json jobs --jq '.jobs[].name'
```

**Example output:**
```
Tests (3.10)
Tests (3.11)
Tests (3.12)
Lint
Type Check
Security Scan
```

All matrix jobs must be listed individually if you want all variants required. Alternatively, use a single summary job if your workflow supports it.

### Gotchas
- Job names are case-sensitive and must include matrix suffixes like `(3.10)` or `(PostgreSQL)`
- If a job name doesn't match exactly, the check will never complete (stays pending forever)
- `enforce_admins=false` is intentional — admins can still merge without checks if truly necessary (e.g., emergency hotfix)
- To check current protection settings: `gh api repos/{owner}/{repo}/branches/main/protection`

### Batch Application
Protect all repos in a list:
```bash
REPOS=(
  "Gorgon"
  "GameSpace"
  "Chefwise"
  "RedOPS"
  "EVE_Rebellion"
  "EVE_Sentinel"
  "EVE_Quartermaster"
  "EVE_Gatekeeper"
  "G13_Linux"
  "RazerControls"
  "Argus_Overview"
  "vdc-portfolio"
)

for repo in "${REPOS[@]}"; do
  echo "Protecting $repo/main..."
  gh api repos/AreteDriver/$repo/branches/main/protection \
    --method PUT \
    -f required_status_checks:strict=true \
    -f required_status_checks:contexts='["CI","Lint","Security"]' \
    -f enforce_admins=false \
    -f allow_force_pushes=false \
    -f allow_deletions=false
done
```

### Workflow Impact
- Direct pushes to main are now blocked
- All changes must go through PRs
- PRs require all status checks to pass
- PRs require branch to be up-to-date (no stale PR merges)
- Admins retain emergency bypass if needed

---

*Last updated: 2026-01-27*


---

## Pip Cache in GitHub Actions

Use setup-python built-in cache: cache: pip. Caches ~/.cache/pip between runs.

---

## Consolidating requirements.txt into pyproject.toml

Use pyproject.toml as single source of truth. CI: pip install -e .[dev]
Gotcha: readme field in pyproject.toml must reference file present in Docker context.

---

## pytest --cov Scope Gotcha

Multiple --cov= flags with --cov-fail-under measures ALL paths combined.
Including untestable Streamlit app.py files drops coverage from 91% to 60%.
Solution: scope to testable modules only (shared, modules dirs).
Rule: Never include framework entry points in coverage paths.

---

*Last updated: 2026-02-04*
