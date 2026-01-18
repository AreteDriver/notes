# Arete Decision Log (ADL)

Records **irreversible or high-leverage decisions** to preserve reasoning, prevent re-litigation, and compound judgment over time.

This is not a journal. This is **operational memory**.

---

## Rules

Log only decisions that:
- Remove options
- Lock architecture, UX, scope, or philosophy
- Will matter in 30+ days

Each entry takes **≤60 seconds** to write.
No edits after logging—only addendums.

---

## Decision Classes

| Class | Use For |
|-------|---------|
| `ARCH` | Architecture, system design, data models, APIs |
| `UX` | Interactions, controls, user experience |
| `SCOPE` | Cuts, exclusions, prioritization, MVP boundaries |
| `TOOLING` | Languages, frameworks, platforms, build systems |
| `PHIL` | Guiding principles, strategy, posture |

---

## Template

Copy this for new entries:

```markdown
## ADL-YYYYMMDD-###

**Project:**
**Decision Class:** [ARCH | UX | SCOPE | TOOLING | PHIL]

**Decision:**
[One sentence]

**Chosen Option:**
[The selected approach]

**Rejected Options:**
- [Option A]
- [Option B]

**Why:**
- [Reason 1]
- [Reason 2]

**Tradeoffs Accepted:**
- [What you gave up]

**Revisit If:**
- [Condition that invalidates this]
```

---

## Naming Convention

`ADL-YYYYMMDD-###`

- `ADL` = Arete Decision Log
- `YYYYMMDD` = Date (e.g., 20260118)
- `###` = Sequential number for that day (001, 002)

---

## Quick Examples

### ARCH
```
## ADL-20260118-001
Project: Gorgon
Decision Class: ARCH
Decision: Use Zustand for state management instead of Redux
Chosen Option: Zustand with persist middleware
Rejected Options: Redux Toolkit, React Context, Jotai
Why:
- Simpler API, less boilerplate
- Built-in persistence
- Good enough for this scale
Tradeoffs Accepted:
- Less ecosystem tooling than Redux
Revisit If:
- State complexity exceeds 10 stores
```

### SCOPE
```
## ADL-20260118-002
Project: EVE_Rebellion
Decision Class: SCOPE
Decision: MVP is single campaign (Minmatar Rebellion) only
Chosen Option: One complete campaign before adding others
Rejected Options: All 4 campaigns in parallel, stub campaigns
Why:
- Ship something demonstrable
- Vertical slice proves architecture
- Reduces scope creep
Tradeoffs Accepted:
- Delayed multi-campaign features
Revisit If:
- First campaign takes > 6 weeks
```

---

## Index

Decisions are stored in dated files: `YYYY-MM.md`

| Month | File |
|-------|------|
| 2026-01 | [2026-01.md](2026-01.md) |

---

*This system adapted from the Arete Decision Log methodology.*
