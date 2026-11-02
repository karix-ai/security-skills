---
name: security-audit
description: Score a repo's security posture 0–100 and list what to fix first, in one pass. Use first — when the user asks "is this safe to publish", "check my repo for security issues", or before making anything public.
---

# Security audit

One pass, one score, one ordered list. The goal is not a pentest; it is to catch the
mistakes that actually get small projects burned: a key in a commit, a personal email in
history, a dependency with a known hole, a CI workflow that hands out write access.

## Scoring (100 points)

### Secrets — 35 points (see `secrets-scan`, `leak-response`)

| Check | Points |
|---|---|
| No secrets in the working tree (scanner run, zero verified findings) | 15 |
| No secrets anywhere in git history | 10 |
| `.env` and similar files are in `.gitignore`; a `.env.example` exists with dummy values | 5 |
| No secrets in MCP / agent / editor config files committed to the repo | 5 |

### Personal data — 15 points (see `git-identity-hygiene`)

| Check | Points |
|---|---|
| Every commit author email is a noreply or a deliberately public address | 6 |
| No personal data in images (EXIF GPS), PDFs or docs | 4 |
| No real names, addresses, phone numbers or internal URLs in code, comments or tests | 5 |

### Dependencies — 20 points (see `dependency-audit`)

| Check | Points |
|---|---|
| Lockfile committed | 5 |
| No known critical/high vulnerabilities in production dependencies | 10 |
| No dependency that was added without a reason (unknown publisher, typo-like name, < 1 month old) | 5 |

### GitHub & CI — 20 points (see `github-hardening`)

| Check | Points |
|---|---|
| Owner account has 2FA | 5 |
| Secret scanning + push protection on | 5 |
| Workflows set `permissions:` explicitly (default read-only) | 5 |
| No `pull_request_target` workflow that checks out PR code | 5 |

### Agent tooling — 10 points (see `agent-tooling-safety`)

| Check | Points |
|---|---|
| Every MCP server / extension / skill in use has a known publisher and was reviewed | 5 |
| Agent permissions are scoped (no blanket "allow all" for shell or network) | 5 |

A single verified live secret anywhere is an automatic **stop**, whatever the score. Rotate it before doing anything else.

## Procedure

1. Detect the stack (package files, lockfiles, `.github/workflows`, agent config folders like `.claude/`, `.cursor/`, `.vscode/`, `mcp.json`).
2. Run the secret scanner on the tree and history (`secrets-scan`). If anything verified is found, stop and run `leak-response`.
3. List commit author emails (`git log --format='%ae' | sort -u`) and check images for EXIF (`git-identity-hygiene`).
4. Run the dependency auditor for each ecosystem present (`dependency-audit`).
5. Read every workflow file; check permissions and triggers (`github-hardening`).
6. List MCP servers, editor extensions and skills referenced in the repo (`agent-tooling-safety`).
7. Score strictly. Partial credit only where the table allows.

## Output

```
Security score: NN/100

Fix first:
- [STOP] Verified AWS key in config/prod.js (commit a1b2c3) → leak-response, now
- [10] lodash 4.17.15: prototype pollution (high) → dependency-audit (5 min)
- [6]  3 commits authored with personal@gmail.com → git-identity-hygiene (10 min)
- ...

Already good: <one line>
Next: <the single next action>
```

Never print a full secret value in the output. Show the first 4 and last 4 characters and the file/commit.
