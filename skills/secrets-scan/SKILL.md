---
name: secrets-scan
description: Find API keys, tokens, passwords and private keys in the working tree and the full git history, and tell real leaks from noise. Use before any push to a public repo, before open-sourcing, or when the user asks "did I leak anything".
---

# Secrets scan

In 2025, 28.65 million new hardcoded secrets were pushed to public GitHub, and commits
made with Claude Code leaked at roughly twice the baseline rate (3.2% vs 1.5%; sources in
`data/security-stats.md`). Most of those were not in the latest version of the file. They
were in history, in a commit that "removed" them. Scan both.

## Tools

Use whichever is installed; install one if neither is.

| Tool | Install | Working tree | Full history |
|---|---|---|---|
| gitleaks | `brew install gitleaks` / release binary | `gitleaks dir . -v` | `gitleaks git . -v` |
| trufflehog | `brew install trufflehog` / release binary | `trufflehog filesystem .` | `trufflehog git file://. --only-verified` |

Older gitleaks (< 8.19) uses `gitleaks detect --no-git` and `gitleaks detect` instead of `dir` / `git`.

`--only-verified` in trufflehog checks the credential against the provider. A verified hit is a live key; treat it as an incident.

## Also check by hand

Scanners miss things that are not shaped like a known key. Grep for these and read the hits:

```bash
git grep -n -I -E "(password|passwd|secret|token|api[_-]?key|private[_-]?key)\s*[:=]" -- . ':!*.lock' ':!*lock.json'
git log -p -S "BEGIN RSA PRIVATE KEY" --all
ls -a | grep -E "^\.env"          # .env, .env.local, .env.production
```

Places that leak most often and that people forget:

- `.env*` files committed before `.gitignore` existed.
- MCP and agent config: `mcp.json`, `.cursor/mcp.json`, `.claude/settings*.json`, `.vscode/settings.json`. Tokens are often pasted straight into `env:` blocks.
- Jupyter notebooks (outputs keep printed values).
- Test fixtures and seed data with "temporary" real keys.
- Frontend bundles: anything in `NEXT_PUBLIC_*`, `VITE_*`, `REACT_APP_*` ships to every visitor.
- Docker images and `docker-compose.yml`.
- CI logs that echo variables.

## Triage

For each hit, decide one of:

- **Live secret** (verified, or a format that matches a real provider and you cannot rule it out) → stop, run `leak-response`.
- **Revoked / test / example** → confirm it really is dead (try it, or check the provider dashboard), then allowlist it with a comment explaining why.
- **False positive** (a hash, a UUID, a placeholder like `sk-xxxx`) → allowlist.

Allowlists: `.gitleaksignore` (fingerprint per line) or a `[allowlist]` in `.gitleaks.toml`. Never allowlist a whole file or directory to make a scan pass.

## Prevent the next one

- Add `.env`, `.env.*` (but not `.env.example`) to `.gitignore`.
- Commit a `.env.example` with every variable and a dummy value.
- Add a pre-commit hook: `gitleaks git --pre-commit --staged` (or the `pre-commit` framework hook from the gitleaks repo).
- Turn on GitHub secret scanning and push protection (`github-hardening`).
- Load MCP tokens from environment variables, not literal values in the config file.

## Procedure

1. Run the scanner on the working tree, then on full history.
2. Run the manual greps. List agent/editor config files and read them.
3. Triage every hit into live / dead / false positive. Never print full secret values; show first 4 + last 4 characters.
4. If anything is live → `leak-response` before anything else.
5. Add the prevention items that are missing.

## Output

A table: file, line or commit, type, verdict (live / dead / false positive), action. Then the prevention items added.
