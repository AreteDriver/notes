# Working Philosophies

Principles that guide how we work together.

---

## Token Economy Mode

> Maximize progress per token. Build, don't explain.

### Principles
- **Assume and continue** — If not truly blocked, make smallest reasonable assumption
- **Edit over create** — Prefer editing existing files
- **Deterministic over clever** — Testable beats elegant
- **Build, don't explain** — No unnecessary prose unless asked

### In Practice
- State time constraints upfront: "I have 2 hours"
- Request artifacts, not explanations: "Give me a file I can run"
- Keep scope locked to current task
- Detect scope creep early—call it out

---

## Senior Software Engineer Mode

> Direct feedback, production thinking, teach the why.

### Core Behaviors
- **Mentoring mindset** — Help develop skills, not just deliver solutions
- **Unfiltered feedback** — Be direct about issues, then explain fixes
- **Production thinking** — Consider scale, security, maintainability, failure modes

### Code Review Dimensions
1. **Correctness** — Does it work? Edge cases? Race conditions?
2. **Security** — Input validation, injection risks, OWASP top 10
3. **Performance** — Complexity, unnecessary operations, leaks
4. **Maintainability** — Readability, naming, DRY, single responsibility
5. **Testing** — Coverage, edge cases, failure scenarios
6. **Production-readiness** — Error handling, logging, graceful degradation

### Trigger Contexts
| Phrase | Response Depth |
|--------|----------------|
| "Quick review" | Surface scan |
| "Review this code" | All 6 dimensions |
| "Deep dive" | Comprehensive architecture |
| "Help me debug" | Root cause investigation |
| "Teach me" | Educational with alternatives |

---

## Execution Assets Over Information

> Every AI session should end with at least one runnable artifact.

### Good Requests
- "Give me a toolkit file I can run tonight"
- "Create a Claude Code prompt for this"
- "Build me a 2-hour sprint plan"

### Avoid
- Open-ended "explain X" without deliverable
- Information gathering without action items

---

## Correction Loops

> Immediately correct errors. Don't let bad assumptions compound.

### Mental Checkpoint
Before accepting long outputs: "Did Claude get the core assumption right?"

### Examples
- "It's EVE_Gatekeeper, not EVE_Starmap"
- "Target role is AI Enablement, not IAM Architect"

---

## Time-Bounded Work

> Explicit time limits force prioritization and prevent scope creep.

### Pattern
Start every session with: "I have X hours/minutes"

This triggers:
- Focused responses
- Prioritization decisions
- Realistic scope

---

## Touch It Locally

> Every generated artifact gets local execution before commit.

### Why
- Catches bugs before they propagate
- Forces understanding of what was generated
- Builds confidence in the code

### Rule
No commit without:
1. Running the code OR
2. Reading and understanding the changes

---

## Decision Logging

> Log only decisions that remove options or lock direction.

### What to Log
- Architecture choices
- Technology selections
- Scope cuts
- Strategic pivots

### What NOT to Log
- Reversible choices
- Implementation details
- Daily work items

See [decisions/](decisions/) for format.

---

## Anti-Patterns

### Avoid These
- God objects/functions (> 200 lines)
- Magic numbers without constants
- Catch-all exception handlers
- Commented-out code in commits
- TODO without ticket reference
- Print debugging in production
- Synchronous calls where async is available
- Marathon sessions without breaks

### Session Anti-Patterns
- Studying after midnight before interviews
- Accepting generated content without testing
- Pasting terminal output without checking for secrets
- Starting implementation before reading documentation

---

## Coding Standards Quick Reference

### Python
- Type hints on all functions
- Docstrings for public functions (Google style)
- `snake_case` for functions/variables
- Virtual env: `python3 -m venv .venv`
- Testing: `pytest`
- Linting: `ruff check . && ruff format .`

### Rust
- `cargo fmt` before commits
- `cargo clippy` for lints
- No `unwrap()` in production code
- Proper error types with `thiserror`

### TypeScript
- Strict mode enabled
- `camelCase` for functions/variables
- Type definitions for all exports

### Git
- Conventional commits: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`
- Commit messages explain "why", not just "what"
- Run tests before pushing
- Never commit secrets

---

*Last updated: 2026-01-18*
