# Checklists & Rituals

Pre-flight checks and post-action rituals.

---

## Before Pasting Terminal Output

**STOP. Check for secrets.**

- [ ] No API keys (sk-..., ghp_..., sk-ant-...)
- [ ] No tokens or passwords
- [ ] No .env file contents
- [ ] No OAuth credentials
- [ ] No personal identifiable information

If secrets are exposed:
1. Immediately revoke the token
2. Generate new credentials
3. Update .env files
4. Continue work

---

## Before Starting AI Session

- [ ] State time constraint: "I have X hours"
- [ ] Check relevant topic files in this repo
- [ ] Review recent decisions in [decisions/](decisions/)
- [ ] Clear previous context if starting fresh

---

## Before Committing AI-Generated Code

- [ ] Read the code (don't just skim)
- [ ] Run it locally
- [ ] Verify it does what was intended
- [ ] Check for obvious issues (hardcoded values, missing error handling)
- [ ] Run linter: `ruff check . --fix && ruff format .`

---

## After Completing AI Session

- [ ] Log any high-leverage decisions to [decisions/](decisions/)
- [ ] Update relevant topic files with new learnings
- [ ] Add wins to README.md wins board
- [ ] Commit and push notes changes

---

## Before Pushing Code

- [ ] Tests pass: `pytest` / `npm test` / `cargo test`
- [ ] Linter passes: `ruff check .` / `npm run lint`
- [ ] No secrets in diff: `git diff --staged`
- [ ] Commit message is conventional: `feat:`, `fix:`, `docs:`, etc.

---

## Before Interview

### Day Before
- [ ] Review study materials (not cramming)
- [ ] Prepare 3-5 questions to ask
- [ ] Test tech setup (camera, mic, screen share)
- [ ] Get good sleep (no studying after midnight)

### Day Of
- [ ] Review company/role notes
- [ ] Have water nearby
- [ ] Close unnecessary apps
- [ ] Take 5 deep breaths before joining

---

## After Interview

Do this within 1 hour:

1. [ ] Write 3 things you did well
2. [ ] Write 1 thing to improve next time
3. [ ] Note objective positive signals (callbacks, "next steps" mentioned)
4. [ ] Walk away for 24 hours before analyzing

**Do NOT:**
- Spiral into self-criticism
- Reanalyze every answer
- Assume the worst from ambiguous signals

---

## Monthly GitHub Profile Audit

- [ ] Bio still accurate (≤160 chars)
- [ ] Profile README renders correctly
- [ ] 4 pinned repos max, all flagship projects
- [ ] All pinned repos have:
  - [ ] Description
  - [ ] Topics (5-10)
  - [ ] README with quick start
- [ ] No broken badges or dead links
- [ ] Recent activity visible

---

## New Machine Setup

```bash
# Shell
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Git
git config --global user.name "AreteDriver"
git config --global user.email "your-email"
gh auth login

# Python
sudo apt install python3-venv python3-pip
pip install --user ruff pytest

# Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env

# Node (if needed)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Claude Code
npm install -g @anthropic-ai/claude-code
```

---

## Project Health Check

Run quarterly on each flagship project:

- [ ] CI passing on main
- [ ] Dependencies up to date (no critical vulns)
- [ ] README accurate and helpful
- [ ] No dead code or commented-out blocks
- [ ] Tests exist and pass
- [ ] Linter configured and passing

---

## Decision Logging Checklist

Before logging a decision, verify:

- [ ] Decision is irreversible or high-leverage
- [ ] Will matter in 30+ days
- [ ] Removes options or locks direction
- [ ] Not just an implementation detail

If yes to all, log to [decisions/](decisions/) using the template.

---

*Last updated: 2026-01-18*
