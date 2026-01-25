# Lessons Learned - GitHub Maintenance Session
**Date:** 2026-01-25

## Session Summary

Comprehensive GitHub maintenance across 10+ repos: PR merges, CI fixes, profile polish, artifact cleanup.

---

## CI/CD Fixes

### 1. Branch Protection Required Checks Mismatch (EVE_Rebellion)
**Problem:** Auto-merge blocked because required check `test` didn't match actual job names `test (3.9)`, `test (3.10)`, etc.

**Solution:** Update branch protection via API to match actual job names:
```bash
gh api repos/{owner}/{repo}/branches/main/protection/required_status_checks --method PATCH \
  --input - << 'EOF'
{
  "strict": true,
  "checks": [
    {"context": "lint"},
    {"context": "test (3.9)"},
    {"context": "test (3.10)"},
    {"context": "test (3.11)"},
    {"context": "test (3.12)"}
  ]
}
EOF
```

---

### 2. GitHub Actions Artifact Storage Quota (EVE_Quartermaster)
**Problem:** Gitleaks secret scan failed with `Artifact storage quota has been hit`.

**Solution (immediate):** Disable artifact upload in workflow:
```yaml
- name: Run Gitleaks
  uses: gitleaks/gitleaks-action@v2
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    GITLEAKS_ENABLE_UPLOAD_ARTIFACT: false
```

**Solution (long-term):** Clean old artifacts:
```bash
# List artifacts
gh api repos/OWNER/REPO/actions/artifacts --jq '.artifacts[] | "\(.id) \(.size_in_bytes)"'

# Delete old artifacts (older than date)
gh api repos/OWNER/REPO/actions/artifacts --paginate \
  --jq '.artifacts[] | select(.created_at < "2026-01-18") | .id' | \
  while read id; do
    gh api repos/OWNER/REPO/actions/artifacts/$id -X DELETE
  done
```

**Note:** GitHub recalculates storage quota every 6-12 hours after deletions.

---

### 3. Draft PRs Need Manual Ready Status
**Problem:** CI doesn't run on draft PRs from Copilot/external contributors.

**Solution:** Mark ready and retrigger:
```bash
gh pr ready NUM -R OWNER/REPO
# If CI still doesn't run, close and reopen:
gh pr close NUM -R OWNER/REPO && gh pr reopen NUM -R OWNER/REPO
```

---

## Repository Management

### Enable Auto-Merge for Repository
```bash
gh api repos/{owner}/{repo} --method PATCH -f allow_auto_merge=true
```

### Batch Merge Safe PRs
**Safe to merge without testing:**
- CI action version bumps (actions/checkout, actions/setup-python, etc.)
- Security patches (lodash, tar, etc.)
- Type definition updates (@types/node, @types/react)
- Minor version bumps in grouped PRs

**Require local testing:**
- Major framework bumps (next.js 14→16, electron 39→40)
- Major SDK bumps (stripe 14→20, openai 4→6)
- React Navigation major versions (6→7)

---

## GitHub Profile Polish

### Fix Broken Repo Links
```bash
# Check actual repo name
gh repo list OWNER --json name --jq '.[] | .name' | grep -i PATTERN

# Update profile README
gh api repos/OWNER/OWNER/contents/README.md --jq '.content' | base64 -d > /tmp/readme.md
# Edit file, then push
```

### Add GitHub Stats Card
```markdown
<p align="center">
  <img src="https://github-readme-stats.vercel.app/api?username=USERNAME&show_icons=true&theme=dark&hide_border=true" alt="GitHub Stats" />
</p>
```

---

## Issue Organization

### Create Milestones for Development Gates
```bash
gh api repos/OWNER/REPO/milestones -f title="Gate 1: Boot Scene" -f description="Project builds, enters Boot scene"

# Assign issues to milestones
gh issue edit NUM -R OWNER/REPO --milestone "Gate 1: Boot Scene"
```

---

## GitHub CLI Commands Reference

```bash
# Check all repos for open PRs
gh search prs --owner OWNER --state open --json repository,title,number

# Check artifact storage across repos
for repo in REPO1 REPO2; do
  count=$(gh api repos/OWNER/$repo/actions/artifacts --jq '.total_count')
  echo "$repo: $count artifacts"
done

# Merge with auto-merge enabled
gh pr merge NUM -R OWNER/REPO --merge --auto

# Check workflow runs
gh run list -R OWNER/REPO --limit 5

# View failed run logs
gh run view RUN_ID -R OWNER/REPO --log-failed

# Create project milestones
gh api repos/OWNER/REPO/milestones -f title="Milestone" -f description="Description"
```

---

## Effective AI Usage Patterns

### What Works Well
1. **Concise commands** - "merge it", "check ci", "1" for options
2. **Batch decisions** - "1 and 2" to select multiple options
3. **Trust + verify** - Let AI work, check results after
4. **Know when to skip** - "skip for now" preserves focus
5. **Natural scope expansion** - Start specific, broaden as needed

### Session Flow
```
Specific task → Related maintenance → Cross-repo sweep → Polish → Done
```

---

## Key Takeaways

1. **Branch protection checks must match exact job names** - Including matrix suffixes like `(3.9)`
2. **Artifact storage accumulates silently** - Set retention policies or clean periodically
3. **Draft PRs from bots need manual promotion** - Mark ready to trigger CI
4. **Profile README is code** - Treat broken links like bugs
5. **Batch similar operations** - Merge all CI bumps together, all security patches together
6. **Major version bumps need testing** - Don't auto-merge next.js, electron, stripe major versions
