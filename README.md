# AreteDriver Knowledge Base

Living documentation for AI-assisted development. Shared reference between Claude and AreteDriver.

**Purpose:** Avoid repeating mistakes, build on wins, refine approaches, compound judgment over time.

---

## Quick Start

### Before Any Session
1. State time constraint: "I have X hours"
2. Check relevant topic files below
3. Reference decision log for prior choices

### After Any Session
1. Log decisions to [decisions/](decisions/)
2. Update relevant topic files
3. Add wins/errors to this README
4. Commit changes

---

## Index

### Core References

| File | Purpose |
|------|---------|
| [PROFILE.md](PROFILE.md) | Career positioning, thesis, flagship projects |
| [PHILOSOPHIES.md](PHILOSOPHIES.md) | Working principles and AI usage patterns |
| [PROMPTS.md](PROMPTS.md) | Battle-tested prompt templates |
| [CHECKLISTS.md](CHECKLISTS.md) | Pre-flight checks and rituals |

### Topics

| Topic | File |
|-------|------|
| CI/CD & GitHub Actions | [topics/ci-cd.md](topics/ci-cd.md) |
| Flutter & Firebase | [topics/flutter.md](topics/flutter.md) |
| Frontend Patterns | [topics/frontend.md](topics/frontend.md) |
| Git Workflow | [topics/git.md](topics/git.md) |
| Python & pytest | [topics/python.md](topics/python.md) |
| Security | [topics/security.md](topics/security.md) |

### Decision Log

High-leverage decisions that lock architecture, scope, or philosophy.

→ [decisions/](decisions/)

### Session Logs

Chronological record of work sessions.

→ [sessions/](sessions/)

---

## Decision Quality Scorecard

*From analysis of 40+ sessions (Oct-Dec 2025)*

| Category | Grade | Notes |
|----------|-------|-------|
| Career Strategy | A- | Clear thesis, needs consistent execution |
| Learning Speed | A | HID Origo in 48 hours |
| AI Usage Patterns | A- | Deliverable-focused |
| Security Hygiene | C+ | Token exposures need addressing |
| Focus/Prioritization | B- | Too many projects |
| Technical Decisions | A- | Good tool selection |

---

## Wins Board

| Date | Win |
|------|-----|
| 2026-01-27 | GameSpace: 5 features shipped — unified roster model (ESPN/Yahoo/Sleeper merge), accuracy tracking (outcome recording + metrics), Basketball/Pro Football Reference scrapers, MLB/NHL scoring (6 new functions), trends+matchup wiring. 161 tests, all CI green |
| 2026-01-27 | Gorgon: Split PR #29 (5474 lines, 28 files) into 3 focused PRs, resolved cherry-pick conflicts, added prompt sanitization + 51 Slack/cost tracker tests, merged all 3 |
| 2026-01-27 | GithubDesktopLinux: Fixed 2 high-severity tar vulnerabilities via npm override, 0 Dependabot alerts remaining, PR #21 merged |
| 2026-01-27 | Chefwise: Migrated `next lint` to ESLint CLI flat config, fixed broken codemod output, 512 tests + lint + build green, PR #64 merged |
| 2026-01-27 | Chefwise: Reverted next/eslint major bumps, fixed 4 test suites (31 failures → 0), locked dependabot to minor/patch for frameworks |
| 2026-01-27 | Batch-merged 14 dependabot PRs across 4 repos (Gorgon, Chefwise, EVE_Quartermaster, GithubDesktopLinux), resolved 3 merge conflicts |
| 2026-01-26 | EVE_Gatekeeper: 1604 tests, MCP server 84% coverage, mobile (46 tests) + desktop (29 tests) suites verified |
| 2026-01-26 | RedOPS: Fixed 7 test failures + 32 deprecation warnings, replaced passlib with bcrypt, 5011 tests passing, 0 warnings |
| 2026-01-26 | EVE_Gatekeeper: Coverage 97%, intel parser enhancements (threat types, ship detection, direction, stats/nearby endpoints), 1399 total tests (+51) |
| 2026-01-26 | EVE_Gatekeeper: ESI structure discovery + route comparison, bridge-routing integration verified, 1348 total tests |
| 2026-01-26 | EVE_Gatekeeper: Jump Bridge validation (highsec/WH rules) + bulk operations, 65 new tests, 1338 total |
| 2026-01-26 | EVE_Gatekeeper: Jump Bridge Manager enhancements - individual bridge CRUD, stats endpoint, 35 new tests, 1308 total |
| 2026-01-26 | EVE_Gatekeeper: Coverage 94%→96%, 1273 total tests (+197), fixed starmap imports, 131 new starmap tests |
| 2026-01-25 | EVE_Gatekeeper: Coverage 69%→94%, 1141 total tests (zkill_listener 58%→99%, auth 69%→99%, webhooks 83%→99%, system_notes 88%→97%, routing 89%→94%) |
| 2026-01-25 | EVE_Gatekeeper: 6 features (ESI location, intel parser, jump fatigue, route sharing, system notes, external links) - 279 new tests, 1076 total |
| 2026-01-25 | GameSpace: 6 features shipped (dark mode, db backups, Yahoo OAuth, Prometheus metrics, stats sync job, WebSocket real-time) |
| 2026-01-25 | Gorgon: Full parallel agent implementation - adaptive rate limiting (429 backoff), distributed cross-process limiting (SQLite/Redis), 1693 tests passing |
| 2026-01-25 | Gorgon: Parallel agent execution complete (async providers, rate-limited executor, adapter migration, 1675 tests passing) |
| 2026-01-25 | EVE_Gatekeeper: Ship-type risk profiles (8 profiles, security/kill multipliers, 15 tests) |
| 2026-01-25 | EVE_Gatekeeper: Discord/Slack webhook alerts (subscriptions, rich formatting, 36 tests) |
| 2026-01-25 | EVE_Gatekeeper: Route history endpoint (deque cache, auto-log on calculation, 9 tests) |
| 2026-01-25 | EVE_Gatekeeper: Public zkill stats API (single/bulk/hot endpoints, 14 tests) |
| 2026-01-25 | EVE_Gatekeeper: Route bookmarks feature (CRUD API, 30 tests, 99% coverage) |
| 2026-01-25 | EVE_Gatekeeper: 122 new tests, coverage 69%→90% (character, cache, token_store, zkill_stats, database, status, logging) |
| 2026-01-25 | EVE_Sentinel: WalletAnalyzer (RMT detection) + ActivityAnalyzer (TZ/engagement), 36 new tests |
| 2026-01-25 | EVE_Gatekeeper: AI route analysis with DangerLevel classification, 43 new tests |
| 2026-01-25 | LikX: 300+ new tests across 8 modules (minimap, onboarding, quick_actions, undo_history, history, queue, recorder, scroll_capture) |
| 2026-01-25 | LikX v3.30.0: Minimap, onboarding, quick actions, undo history + security fixes |
| 2026-01-25 | RedOPS: Plugin system integration, Shodan/Censys intel modules, 49+ new tests |
| 2026-01-24 | RedOPS v1.2.0: Web UI (FastAPI), MCP server, Groq provider, 6 AI providers total |
| 2026-01-24 | RedOPS v1.1.0: Docker, notifications, PDF/STIX reports, shell completions, man page |
| 2026-01-24 | VDC Portfolio: Test coverage 84%→89%, Docker verified, CI green |
| 2026-01-24 | Dotfiles: Migrated to git bare repo, 90 files tracked, nvim/lazygit/kitty configs |
| 2026-01-20 | Gorgon: Workflow versioning feature complete + 86 new tests (28 API, 58 unit) |
| 2026-01-19 | G13_Linux: Fixed 4 TODOs + test bugs, project now feature-complete |
| 2026-01-19 | Chefwise mobile: Full Firebase integration (Auth, Firestore, Cloud Functions) in one session |
| 2026-01-18 | Gorgon Notion integration: full CRUD client + 3 workflows + 26 tests |
| 2026-01-18 | All 7 Gorgon frontend pages in one session |
| 2026-01-18 | Fixed CI across 6 repos in parallel |
| 2025-12 | Advanced to final round at HID (learned platform in 48hrs) |

---

## Error Patterns

| Error | Solution |
|-------|----------|
| Token/secret exposure | Pre-paste checklist: "Contains secrets?" |
| Port already in use | `lsof -i :PORT`, use different port |
| Git rebase with unstaged | `git stash && git pull --rebase && git stash pop` |
| Accepting untested code | "Touch it locally" rule before commit |
| Pre-interview cramming | No studying after midnight |
| Project sprawl | 4 flagship projects max |
| gitignore `lib/` blocks nested dirs | Use `/lib/` for root-only, not `lib/` |
| pytest --cov uses massive memory | Use `--no-cov` for quick runs, coverage in CI only |
| Tests hang on mocked code | Verify patch target matches actual import path |
| FastAPI route not matching | Static paths (`/versions/compare`) must be defined before dynamic (`/versions/{version}`) |
| Module-level dict not patched | Use `patch.dict("module._DICT", {...})` instead of patching functions |
| Test needs DB table that's mocked | Run actual migrations before mocking `run_migrations` in lifespan |
| nvim treesitter-textobjects loop error | Add `init` function to disable RTP plugin loading at startup (see dotfiles) |
| fpdf2 Unicode bullet characters | Use ASCII `-` instead of `•` - fpdf2 latin-1 encoding can't handle Unicode |
| pytest async tests without plugin | Use `asyncio.new_event_loop()` not `get_event_loop()` to avoid "no current event loop" |
| GitHub Actions multiline output EOF | Use unique delimiter like `CHANGELOG_EOF` - `EOF` can conflict with content |
| Anthropic model 404 errors | Model names changed: use `claude-sonnet-4-20250514` not `claude-3-5-sonnet-*` |
| Pydantic validator blocks valid input | Check validator logic - may need to handle special cases (e.g., `plugin:name` syntax) |
| Test class named `Test*` collected by pytest | Rename sample classes to `Sample*` to avoid pytest collection warnings |
| `tempfile.mktemp()` race condition | Use `tempfile.mkstemp()` - creates file atomically, returns (fd, path) |
| GTK code tests fail without display | Test class structure/methods with `hasattr()`, check source with `inspect.getsource()` |
| Tests pass alone but fail in suite | Cached data from other tests affects results - don't assert specific values that depend on shared cache state |
| Patching httpx in FastAPI endpoint | httpx imported inside function - patch at httpx module level, not endpoint module. Or use respx library for cleaner HTTP mocking |
| FastAPI Query() defaults in tests | Query(24) returns Query object when called directly - pass params explicitly in tests: `func(hours=24)` |
| Function named `test_*` collected by pytest | Rename non-test functions to avoid `test_` prefix (e.g., `send_test_message` not `test_webhook`) |
| `next-lint-to-eslint-cli` codemod broken | Generates 3 errors (missing .js ext, non-iterable spread, nested extends). Write flat config manually using `@next/eslint-plugin-next` `flatConfig` API |
| Multiplicative recovery doesn't increase | `int(3 * 1.2) = 3` - use `max(current + 1, int(current * factor))` to guarantee at least +1 |
| AsyncMock makes all attrs async | SQLAlchemy `db.add()` is sync - set `mock_db.add = MagicMock()` explicitly |
| FastAPI dependency not mocked | Use `app.dependency_overrides[get_dep] = lambda: mock` not `patch("...get_dep")` |
| Patch location for imports | Top-level `from x import y` → patch at usage. Dynamic `import` inside function → patch at source |
| SQLAlchemy event listeners with MagicMock | `event.listens_for()` fails on mocks - also patch `event` module: `mock_event.listens_for.return_value = lambda f: f` |
| passlib 'crypt' deprecation warning | Replace passlib with direct bcrypt: `bcrypt.hashpw(pw.encode(), bcrypt.gensalt()).decode()` |
| Fingerprint includes mutable field | Don't include severity in dedup fingerprints - severity changes should be tracked as modifications, not new/resolved |
| Test writes to real config file | Monkeypatch config path functions to return `tmp_path / "file.json"` for test isolation |
| @testing-library/react-native v14 not found | Use v13 - v14 only has alpha/beta releases. Also use `--legacy-peer-deps` for React 19 compatibility |
| Background agent sandbox denies cd/sed to /tmp | Use `git -C /path` instead of `cd /path && git`. Resolve conflicts in main session, not background agents |
| Cherry-pick conflicts when splitting PR | Commits that build on each other conflict when cherry-picked independently. Resolve by keeping both sides (ours + theirs) for additive changes |
| `str.format()` prompt injection | User input with `{__class__.__init__.__globals__}` can leak Python internals. Escape braces: `text.replace("{", "{{").replace("}", "}}")` before interpolation |
| GitHub "not mergeable" right after push | GitHub needs a few seconds to compute merge status. Wait and retry, or check `gh pr view --json mergeable` |
| Multiple dependabot PRs conflict after first merge | Merge in order of least conflict; later PRs may need rebase after each merge |
| Dependabot auto-merges major version bumps | Add `ignore` rules for `version-update:semver-major` on framework packages in `dependabot.yml` |
| Hatchling "no package found" editable install | Add `[tool.hatch.build.targets.wheel] packages = ["."]` to pyproject.toml when package root is non-standard |
| Missing dep in pyproject.toml but in requirements.txt | Always sync both files. `pip install -e .[dev]` fails silently on missing deps if only in requirements.txt |
| Jest test UIDs fail Firebase validation silently | Use realistic test fixtures (20+ char alphanumeric UIDs) matching production validation regexes |
| Jest module-level `process.env.X` is undefined | Env vars captured at import time, before `beforeEach`. Set env before import or pass values explicitly in test bodies |
| Tests pass but don't match source model | Test drift — source evolved (2-tier → 3-tier) but tests used aliases that resolved to valid-but-wrong values. Review tests when changing data models |

---

## Top 5 Rules

1. **Every session ends with a runnable artifact**
2. **State time constraints upfront** ("I have 2 hours")
3. **Pre-paste checklist** for secrets
4. **Touch it locally** before committing AI output
5. **4 flagship projects**, archive the rest

---

## Commands Reference

```bash
# Projects
cd /home/arete/projects/

# Notes
cd /home/arete/projects/notes
git add -A && git commit -m "docs: update" && git push

# Python setup
python3 -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
pytest && ruff check . --fix && ruff format .

# Rust
cargo build && cargo test && cargo clippy && cargo fmt

# Git
git log --oneline -10
gh pr create --title "..." --body "..."
gh run list --json conclusion
```

---

*Last updated: 2026-01-27 (Session 8)*
