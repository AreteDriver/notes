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

*Last updated: 2026-01-19*
