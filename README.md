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

*Last updated: 2026-01-25 (Session 3)*
