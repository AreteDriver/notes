# Git Workflow

Git commands, patterns, and conventions.

---

## Conventional Commits

Format: `type: description`

| Type | Use For |
|------|---------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `refactor` | Code change that neither fixes nor adds |
| `test` | Adding tests |
| `chore` | Maintenance, deps, config |
| `style` | Formatting (no code change) |

### Examples
```
feat: add dark mode with theme toggle
fix: correct API endpoint path
docs: update README with setup instructions
refactor: extract validation logic to util
test: add unit tests for auth module
chore: update dependencies
```

### With Scope
```
feat(auth): add OAuth2 login
fix(api): handle timeout errors
```

---

## Commit Message Format

```
type: short description (imperative mood)

Longer explanation if needed. Explain the "why" not the "what".
The code shows what changed; the message explains why.

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

### Using HEREDOC for Multi-line
```bash
git commit -m "$(cat <<'EOF'
feat: add user authentication

Implement JWT-based auth with refresh tokens.
Includes login, logout, and token refresh endpoints.

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

---

## Common Operations

### Rebase with Unstaged Changes
```bash
git stash
git pull --rebase
git stash pop
```

### Interactive Rebase (Squash Commits)
```bash
git rebase -i HEAD~3  # Last 3 commits
# Mark commits as 'squash' or 's'
```

### Undo Last Commit (Keep Changes)
```bash
git reset --soft HEAD~1
```

### Amend Last Commit
```bash
git commit --amend -m "new message"
# Or just add more changes:
git add .
git commit --amend --no-edit
```

### Cherry Pick
```bash
git cherry-pick <commit-hash>
```

---

## Branch Strategy

### Feature Branches
```bash
git checkout -b feat/add-dark-mode
# ... work ...
git push -u origin feat/add-dark-mode
gh pr create
```

### Naming Convention
- `feat/` - New features
- `fix/` - Bug fixes
- `refactor/` - Code improvements
- `docs/` - Documentation
- `test/` - Test additions

---

## GitHub CLI (gh)

### PRs
```bash
# Create PR
gh pr create --title "Add feature" --body "Description"

# List PRs
gh pr list

# Merge PR
gh pr merge 123 --squash --delete-branch

# View PR
gh pr view 123
```

### Issues
```bash
gh issue create --title "Bug" --body "Description"
gh issue list
gh issue close 123
```

### Repos
```bash
gh repo clone owner/repo
gh repo view owner/repo
gh repo create my-new-repo --public
```

### Runs (CI)
```bash
gh run list
gh run view 123456 --log-failed
gh run rerun 123456
```

---

## Safety Rules

1. **Never force push to main/master** without team agreement
2. **Never commit secrets** - use `.env` files, add to `.gitignore`
3. **Run tests before pushing** - `pytest` / `npm test`
4. **Run formatters before committing** - `ruff format .` / `prettier --write`

---

## .gitignore Essentials

```gitignore
# Dependencies
node_modules/
.venv/
__pycache__/

# Build artifacts
dist/
build/
*.tsbuildinfo

# Environment
.env
.env.local

# IDE
.idea/
.vscode/
*.swp

# OS
.DS_Store
Thumbs.db
```

---

## Troubleshooting

### "Updates were rejected" on Push
```bash
git pull --rebase
git push
```

### Accidentally Committed to Wrong Branch
```bash
# Undo commit but keep changes
git reset --soft HEAD~1
git stash
git checkout correct-branch
git stash pop
git commit
```

### Remove File from Git but Keep Locally
```bash
git rm --cached filename
echo "filename" >> .gitignore
git commit -m "chore: stop tracking filename"
```

---

*Last updated: 2026-01-18*
