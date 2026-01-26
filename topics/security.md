# Security Practices

Patterns for secure development.

---

## Secrets Management

### Never Commit
- API keys (`sk-...`, `ghp_...`, `sk-ant-...`)
- Passwords
- OAuth tokens
- .env files
- Private keys

### Before Pasting Terminal Output
**Always check for secrets first.**

```bash
# Bad - exposes token
$ echo $GITHUB_TOKEN
ghp_xxxxxxxxxxxxxxxxxxxx

# Good - reference without revealing
$ echo "GITHUB_TOKEN is set: ${GITHUB_TOKEN:+yes}"
GITHUB_TOKEN is set: yes
```

### If Secrets Are Exposed
1. **Immediately** revoke the token
2. Generate new credentials
3. Update all .env files
4. Check if token was used maliciously
5. Continue work

### Storage Patterns

```bash
# Local development
.env          # Never committed
.env.example  # Template, no real values

# CI/CD
GitHub Secrets > Settings > Secrets and variables

# Production
Environment variables or secrets manager
```

---

## Input Validation

### Always Validate
- User input
- External API responses
- File paths (prevent directory traversal)
- URLs (prevent SSRF)

### Python Example

```python
from pathlib import Path

def safe_read_file(user_path: str, base_dir: Path) -> str:
    """Read file only if within allowed directory."""
    requested = (base_dir / user_path).resolve()

    # Prevent directory traversal
    if not requested.is_relative_to(base_dir):
        raise ValueError("Path traversal detected")

    return requested.read_text()
```

### SQL Injection Prevention

```python
# Bad - SQL injection vulnerable
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

# Good - parameterized query
cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
```

---

## Authentication

### Token Best Practices
- Short expiration times
- Refresh tokens for long-lived sessions
- Scope tokens to minimum needed permissions

### GitHub Token Scopes
Only request what you need:
- `repo` - Full repository access (often overkill)
- `public_repo` - Public repos only
- `read:user` - Read user profile
- `workflow` - GitHub Actions

---

## Dependency Security

### Regular Audits

```bash
# Python
pip-audit

# Node.js
npm audit
npm audit fix

# Rust
cargo audit
```

### Dependabot
Enable in GitHub: Settings > Security > Dependabot alerts

### npm Overrides for Transitive Dependencies

When a vulnerability exists in a transitive dependency that the direct dependency hasn't updated yet:

```json
// package.json
{
  "overrides": {
    "tar": "^7.5.4"  // Force version across all dependencies
  }
}
```

Then run `npm install` to apply. Verify with `npm ls <package>`.

**Use case:** `@capacitor/cli` depends on `tar@^6.1.11` (vulnerable). Override forces `tar@7.5.4` (patched) without waiting for Capacitor to update.

---

## CI/CD Security

### Don't Log Secrets

```yaml
# Bad
- run: echo "Token is ${{ secrets.API_KEY }}"

# Good - secrets are masked, but still avoid logging
- run: ./deploy.sh
  env:
    API_KEY: ${{ secrets.API_KEY }}
```

### Limit Token Permissions

```yaml
permissions:
  contents: read  # Only what's needed
  # Not: write-all
```

---

## OWASP Top 10 Quick Reference

| Vulnerability | Prevention |
|---------------|------------|
| Injection | Parameterized queries, input validation |
| Broken Auth | Strong passwords, MFA, secure sessions |
| Sensitive Data | Encryption, don't log secrets |
| XXE | Disable external entities in XML parsers |
| Broken Access | Check permissions on every request |
| Misconfig | Secure defaults, remove debug mode |
| XSS | Escape output, CSP headers |
| Insecure Deserialization | Don't deserialize untrusted data |
| Vulnerable Components | Keep dependencies updated |
| Logging Failures | Log security events, don't log secrets |

---

## Security Checklist

Before shipping:

- [ ] No hardcoded credentials
- [ ] Input validation on all user data
- [ ] Parameterized database queries
- [ ] Dependencies audited
- [ ] Secrets in environment variables
- [ ] HTTPS only
- [ ] Error messages don't leak internals

---

## Shell History Protection

Secrets can leak into shell history. Prevent with HISTIGNORE.

### Bash (~/.bashrc)
```bash
export HISTIGNORE="*API_KEY*:*TOKEN*:*SECRET*:*PASSWORD*:*CREDENTIAL*:*sk-ant-*:*sk-proj-*:*ghp_*:*pypi-*:*AKIA*"
```

### Zsh (~/.zshrc)
```bash
HISTORY_IGNORE="(*API_KEY*|*TOKEN*|*SECRET*|*PASSWORD*|*CREDENTIAL*|*sk-ant-*|*sk-proj-*|*ghp_*|*pypi-*|*AKIA*)"
```

### Clean Existing History
```bash
# Remove lines containing secrets
sed -i '/sk-ant-\|sk-proj-\|ghp_\|AKIA\|pypi-/d' ~/.zsh_history ~/.bash_history
```

---

## Security Audit Tools

### gitleaks - Secret Scanner
```bash
# Install
curl -sL -o /tmp/gitleaks.tar.gz "https://github.com/gitleaks/gitleaks/releases/download/v8.18.4/gitleaks_8.18.4_linux_x64.tar.gz"
cd /tmp && tar xzf gitleaks.tar.gz && mv gitleaks ~/.local/bin/

# Scan git repo (respects .gitignore)
gitleaks detect

# Scan directory (all files)
gitleaks detect --no-git

# Verbose output
gitleaks detect -v
```

### pip-audit - Python Vulnerability Scanner
```bash
# Install
pipx install pip-audit

# Scan current environment
pip-audit

# Scan requirements file
pip-audit -r requirements.txt
```

---

## File Permissions Checklist

| Path | Required | Check |
|------|----------|-------|
| `~/.ssh/` | 700 | `chmod 700 ~/.ssh` |
| `~/.ssh/id_*` (private) | 600 | `chmod 600 ~/.ssh/id_ed25519` |
| `~/.ssh/config` | 600 | `chmod 600 ~/.ssh/config` |
| `~/.gnupg/` | 700 | `chmod 700 ~/.gnupg` |
| `~/.aws/credentials` | 600 | `chmod 600 ~/.aws/credentials` |
| `.env` files | 600 | `chmod 600 .env` |

### Find World-Writable Files
```bash
find ~ -type f -perm -002 2>/dev/null
```

---

## Secret Patterns Reference

| Pattern | Service |
|---------|---------|
| `sk-ant-api03-*` | Anthropic API |
| `sk-proj-*`, `sk-*` | OpenAI API |
| `ghp_*` | GitHub PAT |
| `gho_*` | GitHub OAuth |
| `AKIA*` | AWS Access Key |
| `pypi-*` | PyPI Token |
| `npm_*` | npm Token |
| `-----BEGIN * PRIVATE KEY-----` | Private keys |

---

## System Audit Quick Commands

```bash
# Check SSH permissions
ls -la ~/.ssh/

# Check for exposed secrets in history
grep -E 'sk-ant-|sk-proj-|ghp_|AKIA' ~/.zsh_history ~/.bash_history

# Scan git repos for secrets
gitleaks detect

# Check Python dependencies
pip-audit

# Find world-writable files
find ~ -type f -perm -002 2>/dev/null

# Check sensitive file permissions
stat -c '%a %n' ~/.ssh/id_* ~/.gnupg/private-keys-v1.d/* 2>/dev/null
```

---

## Temp File Security

### Race Condition Risk (TOCTOU)

`tempfile.mktemp()` is **deprecated** - it only generates a name, doesn't create the file, leaving a window for race conditions.

```python
# BAD - vulnerable to race condition
import tempfile
temp_file = tempfile.mktemp(suffix=".png")  # File doesn't exist yet!
# Attacker could create file here
open(temp_file, "w").write(data)  # Writing to attacker's file

# GOOD - atomically creates file
import os
import tempfile
fd, temp_path = tempfile.mkstemp(suffix=".png")  # File created atomically
os.close(fd)  # Close fd, use path
Path(temp_path).write_bytes(data)
```

### Best Practices

| Method | Use Case |
|--------|----------|
| `mkstemp()` | Need path, will manage cleanup |
| `NamedTemporaryFile(delete=False)` | Need file object + path |
| `TemporaryDirectory()` | Need temp directory |
| `SpooledTemporaryFile()` | Memory-first, disk fallback |

---

---

## Game EULA/TOS Compliance

When building tools for games, EULA/TOS compliance is a security concern - violating it can result in account bans and legal issues.

### EVE Online Specific Rules

| Feature | Status | Rule |
|---------|--------|------|
| Input broadcasting | **BANNED** | One keypress to multiple clients = permaban |
| Input multiplexing | **BANNED** | Same as broadcasting |
| Cache scraping | **BANNED** | Reading game memory/cache |
| Automation/macros | **BANNED** | Anything beyond 1:1 input |
| Window management | Allowed | Arranging windows, previews |
| Hotkey cycling | Allowed | One key = focus one window |
| Overlay tools | Allowed | If no game data reading |

### The "One Key = One Action" Rule

```
ALLOWED:     Press F1 → Focus Window A
BANNED:      Press F1 → Send F1 to Windows A, B, C, D
```

### Before Building Game Tools

1. **Read the EULA/TOS** - Specifically sections on third-party software
2. **Check for policy updates** - Rules change (EVE banned broadcasting in 2015)
3. **When in doubt, don't** - Account bans are permanent
4. **Document compliance** - Explain why features are EULA-safe

### Reference Links
- EVE Online: https://support.eveonline.com/hc/en-us/articles/204873262
- General: Check each game's official policy page

---

*Last updated: 2026-01-26*
