---
name: pre-publish-check
description: The last check before flipping a repo from private to public or open-sourcing a project — secrets, history, personal data, licenses, files that should not ship. Use right before publishing, or when the user says "I'm about to make this public".
---

# Pre-publish check

Making a repo public publishes **all of its history**, every branch and every tag, not just
what is on `main` today. This is the one moment to catch everything; after the switch,
assume anything in the repo has been copied.

## Checklist

### Secrets and credentials (blocking)
- [ ] `secrets-scan` on the working tree **and full history** across all branches: zero live findings.
- [ ] No `.env`, `*.pem`, `*.key`, `id_rsa*`, `*.p12`, `credentials.json`, `service-account*.json` anywhere in history:
  ```bash
  git log --all --name-only --format='' | sort -u | grep -E "(\.env|\.pem|\.key|id_rsa|\.p12|credentials\.json|service-account)"
  ```
- [ ] Agent/MCP config files contain no literal tokens.

### Personal data (blocking)
- [ ] Commit authors use noreply or deliberately public emails (`git-identity-hygiene`).
- [ ] Images stripped of GPS/EXIF; screenshots don't show private tabs, names or files.
- [ ] No customer data, real names or internal URLs in fixtures, seeds, logs or docs.

### What ships
- [ ] Branches and tags: delete the ones you don't want public (`git branch -a`, `git tag`). Old experimental branches are the usual leak.
- [ ] No large binaries, database dumps, `node_modules`, build output or `.DS_Store`.
- [ ] `.gitignore` covers env files, build output, editor and OS files.
- [ ] Issues, PRs, wiki and Actions logs (if the repo was private with history there): review or disable before switching.

### Legal
- [ ] A `LICENSE` file (MIT / Apache-2.0 for maximum adoption) — without one, nobody may legally use the code.
- [ ] Third-party code you copied is compatible with that license and attributed.
- [ ] No code or assets you don't have rights to (company code, paid fonts, stock images, other people's artwork).

### Security posture
- [ ] `github-hardening` items on: 2FA, secret scanning, push protection, Dependabot, read-only workflow token.
- [ ] `dependency-audit`: no critical/high in production dependencies.
- [ ] A `SECURITY.md` explaining how to report a vulnerability privately.

## If history is dirty

Prefer publishing a **fresh repository** over rewriting history when the project is small and history has little value:

```bash
git checkout --orphan public-main
git add -A && git commit -m "Initial public release"
# push this single commit to a new empty public repo
```

Rewriting with `git filter-repo` is the alternative when history matters (`leak-response`, `git-identity-hygiene`). Either way, rotate any secret that was ever committed — removing it from history does not make it safe.

## Procedure

1. Run every blocking check; stop at the first failure and fix it.
2. Run the non-blocking checks; list what is missing.
3. Recommend fresh-repo vs. rewrite if history has problems.
4. Give a go / no-go with the reasons.

## Output

```
Publish: GO / NO-GO
Blocking: <list, or "none">
Fix soon: <list>
Commands: <exact commands for this repo>
```
