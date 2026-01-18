# Battle-Tested Prompts

Curated prompts that consistently produce good results.

---

## Universal Prompt Template

Use this structure for any new prompt:

```markdown
# [TASK NAME]

## Context
**Who I am:** [Role, experience level]
**Current state:** [What exists now]
**Target state:** [What should exist after]
**Time constraint:** [X hours/minutes]

## Objective
[One clear sentence]

## Deliverables
1. [Artifact 1] — [Format, validation criteria]
2. [Artifact 2] — [Format, validation criteria]

## Anti-Patterns
- ❌ [What to avoid]

## Exit Criteria
Done when:
1. [Criterion 1]
2. [Criterion 2]
```

---

## CI/CD Diagnostic Fix

Use when CI fails. Diagnose before fixing.

```markdown
# CI/CD Diagnostic Protocol

## Step 1: Identify the Actual Error

Check the failing job logs. Classify:
- A) Lock file issues: "Missing: <package> from lock file"
- B) Test failures: "FAIL src/tests/..."
- C) Build failures: "error[E0433]: failed to resolve..."
- D) Environment issues: "Node.js version X not supported"

## Step 2: Reproduce Locally

# Node.js
rm -rf node_modules package-lock.json && npm install && npm test

# Python
rm -rf venv && python -m venv venv && source venv/bin/activate
pip install -e ".[dev]" && pytest -v

# Rust
cargo clean && cargo build && cargo test

## Step 3: Fix Based on Diagnosis

Don't regenerate files blindly. Fix the specific issue.

## Step 4: Verify

git push && gh run watch
```

---

## GitHub Profile Optimization

Use for portfolio work and job search prep.

```markdown
# GitHub Profile Optimization

## Objective
Transform GitHub into professional portfolio for [TARGET ROLE].

## Deliverables

### 1. Profile README
- Role clear in first line
- Tech stack visible without scrolling
- Featured projects table
- No buzzwords without backing projects

### 2. Repository Metadata
For each pinned repo:
- Description: ≤350 chars, keyword-rich
- Topics: 5-10 relevant tags
- Screenshot: 1200x630px for social preview

### 3. Validation
5-second test: Role clear, tech visible, best work pinned
30-second test: Can understand each pinned repo
2-minute test: Evidence of code quality

## Anti-Patterns
- ❌ "10x engineer", "ninja", "rockstar"
- ❌ Listing every technology ever used
- ❌ Pinning incomplete projects
- ❌ Empty READMEs
```

---

## Project Kickoff

Use when starting a new project.

```markdown
# Project Kickoff: [NAME]

## Context
I have [X hours] to make progress on [PROJECT].

## Current State
[What exists now]

## Target State
[What should exist after this session]

## Constraints
- Time: [X hours]
- Tech: [Stack constraints]
- Scope: [What's explicitly out of scope]

## Deliverables
1. [First deliverable]
2. [Second deliverable]

## Checkpoints
After each checkpoint, commit.

### Checkpoint 1: [Name]
- [ ] Task 1
- [ ] Task 2
`git commit -m "feat: checkpoint 1"`

### Checkpoint 2: [Name]
- [ ] Task 3
- [ ] Task 4
`git commit -m "feat: checkpoint 2"`
```

---

## Code Review Request

Use when asking for code review.

```markdown
# Code Review: [FILE/FEATURE]

## Context
[What this code does, why it exists]

## Specific Concerns
1. [Area of uncertainty]
2. [Area of uncertainty]

## Review Depth
[Quick review | Standard | Deep dive]

## Code
[Paste code or file path]
```

---

## Debugging Session

Use when stuck on a bug.

```markdown
# Debug: [SYMPTOM]

## What I Expected
[Expected behavior]

## What Actually Happened
[Actual behavior]

## Error Message
```
[Exact error]
```

## What I've Tried
1. [Attempt 1] — [Result]
2. [Attempt 2] — [Result]

## Relevant Code
[File paths or code snippets]

## Environment
- OS: [Ubuntu X.X]
- Language: [Python 3.X / Node X.X / Rust X.X]
- Relevant deps: [versions]
```

---

## Learning a New Platform

Use when ramping up on unfamiliar technology.

```markdown
# Rapid Platform Learning: [PLATFORM]

## Context
I need to understand [PLATFORM] for [PURPOSE].
Time available: [X hours]

## Learning Goals
1. [Architectural understanding]
2. [Key concepts]
3. [Integration points]

## Deliverables
1. Condensed notes (max 5 pages)
2. Key questions to ask in interview/discussion
3. Hands-on exercise or prototype

## Focus Areas
- How does [X] work?
- What are the failure modes?
- How does it integrate with [Y]?

## Explicitly Skip
- [Low-priority details]
- [Implementation minutiae]
```

---

## Post-Session Reflection

Use at end of session to capture learnings.

```markdown
# Session Reflection: [DATE]

## What We Built
- [Artifact 1]
- [Artifact 2]

## Decisions Made
- [Decision 1] — [Reasoning]
- [Decision 2] — [Reasoning]

## What Worked
- [Pattern/approach that worked]

## What to Improve
- [Thing that didn't work well]

## Next Session
- [ ] [Task 1]
- [ ] [Task 2]
```

---

## Quick Triggers

| You want... | Start with... |
|-------------|---------------|
| Code review | "Review this code, [quick/standard/deep]" |
| New feature | "I have X hours to add [feature]" |
| Fix bug | "Debug: [symptom]. Expected [X], got [Y]" |
| Learn platform | "Rapid learning: [platform] for [purpose]" |
| Portfolio work | "GitHub optimization for [target role]" |
| CI fix | "CI is failing with [error]. Diagnose first." |

---

*Last updated: 2026-01-18*
