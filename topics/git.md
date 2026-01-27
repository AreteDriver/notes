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

## Dotfiles with Git Bare Repo

Manage dotfiles without symlinks. Files stay in `$HOME`, tracked by bare repo.

### Setup
```bash
# Initialize bare repo
git init --bare ~/.dotfiles

# Add alias (add to .bashrc/.zshrc)
alias dotfiles='git --git-dir=$HOME/.dotfiles --work-tree=$HOME'

# Hide untracked files (essential)
dotfiles config --local status.showUntrackedFiles no

# Add files
dotfiles add ~/.bashrc ~/.gitconfig
dotfiles commit -m "feat: initial dotfiles"
dotfiles remote add origin git@github.com:user/dotfiles.git
dotfiles push -u origin main
```

### Clone to New Machine
```bash
git clone --bare https://github.com/user/dotfiles.git $HOME/.dotfiles
alias dotfiles='git --git-dir=$HOME/.dotfiles --work-tree=$HOME'

# Backup existing files, then checkout
mkdir -p ~/.dotfiles-backup
dotfiles checkout 2>&1 | grep -E "\s+\." | awk '{print $1}' | \
  xargs -I{} mv {} ~/.dotfiles-backup/{}
dotfiles checkout

dotfiles config --local status.showUntrackedFiles no
```

### Daily Usage
```bash
dotfiles status
dotfiles add ~/.new-config
dotfiles commit -m "feat: add new config"
dotfiles push
```

---

## Batch Dependabot PR Merges

### List All Open PRs Across Repos
```bash
# Check each repo for open PRs
for repo in AreteDriver/Chefwise AreteDriver/Gorgon; do
  echo "=== $repo ==="
  gh pr list --repo "$repo"
done
```

### Batch Merge Clean PRs
```bash
gh pr merge 60 --repo AreteDriver/Chefwise --merge
gh pr merge 59 --repo AreteDriver/Chefwise --merge
# Run all in parallel if independent
```

### Resolve Dependabot Conflicts
```bash
# Clone, checkout branch, rebase onto main
gh repo clone owner/repo /tmp/repo-fix -- --depth=50
git -C /tmp/repo-fix fetch origin branch-name
git -C /tmp/repo-fix checkout -b branch-name FETCH_HEAD
git -C /tmp/repo-fix rebase origin/main

# Resolve conflicts (accept dependabot + keep main's other changes)
git -C /tmp/repo-fix checkout --theirs package.json package-lock.json
# Manually adjust any versions that should come from main
git -C /tmp/repo-fix add .
git -C /tmp/repo-fix rebase --continue

# Force push and merge
git -C /tmp/repo-fix push --force origin branch-name
gh pr merge N --repo owner/repo --merge
```

### Gotchas
- Merging one dependabot PR can make others conflict (shared package.json)
- Use `git -C /path` not `cd /path && git` (works better in sandboxed environments)
- Check CI status before merging — pre-existing failures may block

---

## Splitting a Large PR with Cherry-Pick

When a PR has grown too large, split it by cherry-picking commits into new branches.

### Process
```bash
# 1. Close the original PR with explanation
gh pr close 29 --comment "Splitting into 3 focused PRs..."

# 2. For each group of commits, create a branch from main
git checkout main && git pull
git checkout -b feat/part-a
git cherry-pick <commit1> <commit2> <commit3>
git push -u origin feat/part-a
gh pr create --title "Part A" --body "..."

# 3. Repeat for other groups
```

### Conflict Resolution
When later commits depend on earlier ones (e.g., commit 4 adds enums after commit 1 added enums):
- Cherry-picking commit 4 alone onto main will conflict
- Resolve by keeping both sides: main's content + the new additions
- After merging earlier PRs, later PRs may need a merge from main before they're mergeable

### Gotchas
- GitHub takes a few seconds to recompute merge status after pushing conflict resolution
- If `gh pr merge` fails with "not mergeable", check `gh pr view --json mergeable` and retry
- Merge PRs in dependency order (independent first, then dependent ones)

---

*Last updated: 2026-01-27*
