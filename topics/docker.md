# Docker Notes

Patterns, gotchas, and solutions for Docker builds and deployment.

---

## pyproject.toml readme Field in Docker Builds

**Problem:** Switching from requirements.txt to pyproject.toml for Docker builds causes pip install to fail if the readme field references a file excluded by .dockerignore.

**Example:** pyproject.toml had readme = "CLAUDE.md" but *.md was in .dockerignore.

**Solution:**
1. Change readme field to reference README.md
2. Add exception to .dockerignore: !README.md

**Lesson:** The readme field matters for pip install metadata generation even in Docker where you never read the README. Always check .dockerignore against pyproject.toml metadata fields.

---

## Key Principles

1. Check .dockerignore against pyproject.toml metadata fields
2. Use README.md as the readme field (most standard, least likely to be ignored)

---

*Last updated: 2026-01-28*
