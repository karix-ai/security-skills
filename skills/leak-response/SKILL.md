---
name: leak-response
description: Step-by-step response when a secret, token or private key was committed or published — rotate first, then check for abuse, then clean history. Use the moment a live secret is found, or when the user says "I pushed my API key".
---

# Leak response

The order matters more than speed in any single step. Deleting the file, force-pushing,
or making the repo private does **not** un-leak a secret: bots scrape public GitHub
commits within minutes, and forks, caches and clones keep the old history. Credentials
also stay valid for years if nobody rotates them — most secrets leaked in 2022 were still
valid in 2026 (see `data/security-stats.md`). Rotation is the fix. History cleanup is hygiene.

## 1. Rotate (first, always)

- Revoke the leaked credential at the provider and issue a new one. Do this before touching git.
- Update every place that uses it: local `.env`, CI secrets, hosting provider env vars, teammates.
- If it was a private SSH or GPG key: remove the public half from every server's `authorized_keys`, from GitHub (Settings → SSH and GPG keys), and generate a new pair.
- If it was a GitHub token: revoke it at Settings → Developer settings → Personal access tokens, and review Settings → Applications.

Where to revoke, for the most common providers:

| Provider | Where |
|---|---|
| Anthropic | console.anthropic.com → API keys |
| OpenAI | platform.openai.com → API keys |
| AWS | IAM → Users → Security credentials (deactivate, then delete) |
| Google Cloud | IAM & Admin → Service accounts → Keys |
| Stripe | Dashboard → Developers → API keys → Roll key |
| GitHub | Settings → Developer settings → Tokens |
| Telegram bot | @BotFather → /revoke |

## 2. Check for abuse

- Provider usage and billing dashboards for the exposure window: unusual spend, regions, models, or request volume.
- Provider audit logs (AWS CloudTrail, GCP audit logs, GitHub security log).
- For SSH: `~/.ssh/authorized_keys` on every machine the key could reach; look for keys you did not add.
- Set a spending limit or budget alert if the provider supports one.

## 3. Clean history (after rotation)

Only once the credential is dead:

```bash
pip install git-filter-repo

# Remove a whole file from all history
git filter-repo --invert-paths --path .env

# Or replace the value everywhere (one rule per line in replacements.txt):
#   sk-live-abc123==>REDACTED
git filter-repo --replace-text replacements.txt

git push --force --all && git push --force --tags
```

Then:
- Ask collaborators to re-clone (their clones still have the secret).
- For public repos, cached views on GitHub can persist; GitHub Support can purge cached views and PR refs if needed.
- Forks are not rewritten. If someone forked during the exposure, assume the secret is public forever — which is why step 1 comes first.

## 4. Prevent

Run `secrets-scan` prevention steps: `.gitignore`, `.env.example`, pre-commit hook, push protection.

## Procedure

1. Identify exactly what leaked, where, and since when (first commit containing it).
2. Walk the user through rotation for that provider. Confirm the old credential is revoked.
3. List the dashboards/logs to check for the exposure window.
4. Only then, produce the `git filter-repo` commands for this repo.
5. Add prevention.

## Output

A checklist with the exposure window, the rotation steps (ticked as the user confirms), the abuse checks, and the history-cleanup commands.
