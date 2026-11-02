---
name: github-hardening
description: Harden a GitHub account and repo — 2FA, secret scanning with push protection, Dependabot, branch protection, least-privilege Actions and app installs. Use when setting up a new repo, before going public, or after any security scare.
---

# GitHub hardening

The repo is only as safe as the account that owns it and the workflows that can write to it.
Most of this is settings toggles; the part that needs reading is GitHub Actions.

## Account

- [ ] **2FA** with an authenticator app or passkey (Settings → Password and authentication). Save recovery codes offline.
- [ ] **Passkey** added as a second method, so losing the phone is not losing the account.
- [ ] **Email privacy**: Settings → Emails → keep private + block pushes that expose it.
- [ ] **Sessions**: Settings → Sessions; revoke anything unfamiliar.
- [ ] **Tokens**: Settings → Developer settings → Personal access tokens. Delete unused; prefer fine-grained tokens scoped to one repo with an expiry.
- [ ] **Apps**: Settings → Applications (Authorized OAuth Apps, Authorized GitHub Apps, Installed GitHub Apps). Remove what you don't use; for what stays, limit to selected repositories.

## Repository (Settings → Code security / Advanced Security)

- [ ] **Secret scanning** on.
- [ ] **Push protection** on — blocks pushes that contain recognised secrets before they land.
- [ ] **Dependabot alerts** and **Dependabot security updates** on.
- [ ] **Private vulnerability reporting** on, plus a short `SECURITY.md` saying how to report.

Check from the terminal:

```bash
gh api repos/OWNER/REPO --jq '.security_and_analysis'
```

## Branch protection (Settings → Rules / Branches)

For `main`:
- Require a pull request before merging (even solo, it gives CI a chance to run).
- Require status checks to pass.
- Block force pushes and deletions.

## GitHub Actions — read every workflow

1. **Default token permissions to read-only.** At the top of every workflow:
   ```yaml
   permissions:
     contents: read
   ```
   Grant more per job only where needed (`contents: write` for releases, `pull-requests: write` for comments). Also set Settings → Actions → General → Workflow permissions → **Read repository contents**.
2. **Pin third-party actions to a full commit SHA**, not a tag: `uses: some-org/some-action@<40-char sha> # v1.2.3`. Tags can be moved; SHAs cannot. First-party `actions/*` can stay on major tags if you accept that trade-off.
3. **Never check out PR code in `pull_request_target`.** That trigger runs with secrets and write access in the context of the base repo; checking out and running the PR's code hands both to whoever opened the PR. Use `pull_request` for anything that builds or tests contributor code.
4. **Don't interpolate untrusted input into `run:`.** `${{ github.event.issue.title }}` inside a shell line is command injection. Pass it through `env:` and quote it.
5. **Secrets only where needed**: environment-scoped secrets with required reviewers for deploys.
6. **Settings → Actions → General**: allow only actions from GitHub and verified creators, or a specific allowlist; require approval for workflows from first-time contributors.

## Procedure

1. Walk the account checklist with the user (UI paths above); tick each item.
2. Read repo security settings via `gh api`; list what is off.
3. Read every file in `.github/workflows/`. For each: permissions, triggers, third-party actions and how they are pinned, any `${{ github.event.* }}` inside `run:`.
4. Propose a patch for each workflow (permissions block, SHA pins, env-quoted inputs).
5. Recommend branch protection rules matching how the user actually works.

## Output

Account checklist (done / to do), repo settings (on / off), a per-workflow findings table with the patch, and the UI steps the user must do by hand.
