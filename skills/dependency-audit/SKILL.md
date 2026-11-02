---
name: dependency-audit
description: Check dependencies for known vulnerabilities, risky install scripts, typosquats and abandoned packages, and propose the smallest safe upgrade. Use before release, after adding a package, or when the user asks "are my dependencies safe".
---

# Dependency audit

Most of the code in a typical project is code you did not write. Two different risks live
there: **known vulnerabilities** (a CVE in a real package) and **malicious packages**
(a typosquat, a hijacked maintainer account, a postinstall script that steals tokens).
Auditors catch the first; only reading catches the second.

## 1. Known vulnerabilities

| Ecosystem | Command |
|---|---|
| npm | `npm audit --omit=dev` (production only), `npm audit` (all) |
| pnpm | `pnpm audit --prod` |
| yarn | `yarn npm audit --environment production` (berry) / `yarn audit --groups dependencies` (v1) |
| Python | `pip install pip-audit && pip-audit` |
| Any (incl. Go, Rust, Java) | `osv-scanner scan source -r .` (osv-scanner v2) |

Prioritise:
1. Critical/high in **production** dependencies that are reachable from your code.
2. Critical/high in dev/build tooling (matters for supply chain, less for runtime).
3. Everything else, in batches.

Fix with the smallest change: patch version first, then minor, then major. `npm audit fix` without `--force` only does semver-compatible updates; read the diff of `--force` before accepting it.

## 2. Malicious or risky packages

For every **direct** dependency added in the last few months, check:

- **Name**: is it one letter off a popular package (`reqeusts`, `lodahs`, `cross-env.js`)? Does it impersonate an org scope?
- **Publisher and age**: a package published last week by a new account doing something a well-known package already does is a red flag.
- **Install scripts**: `npm view <pkg> scripts` — `preinstall`/`postinstall` that download or run anything deserve a read.
- **Downloads vs. stars vs. repo link**: no repository link, or a repo that does not match the package contents.
- **Recent maintainer change** on a long-stable package.

Useful defaults:

```bash
npm ci --ignore-scripts          # in CI, then run needed scripts explicitly
npm config set ignore-scripts true   # optional, per machine; some packages need scripts to build
```

## 3. Hygiene

- Commit the lockfile. Install with `npm ci` / `pip install -r requirements.txt --require-hashes` / `uv sync --frozen` in CI so builds use exactly what was reviewed.
- Remove unused dependencies (`npx depcheck`, `pip-autoremove`, or read the imports).
- Turn on Dependabot alerts and security updates (`github-hardening`).
- Prefer fewer, well-maintained packages over many tiny ones.

## Procedure

1. Detect ecosystems from manifest/lock files.
2. Run the auditor(s). Group findings by severity and prod/dev.
3. For each critical/high: the fixed version, whether it is a breaking upgrade, and the exact command.
4. Review direct dependencies for the malicious-package signals above; flag anything suspicious with the reason.
5. Check lockfile presence and CI install command.

## Output

A table of vulnerabilities (package, current → fixed, severity, prod/dev, command), a short list of suspicious packages with reasons, and hygiene items missing.
