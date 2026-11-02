---
name: agent-tooling-safety
description: Review MCP servers, editor extensions, agent skills and agent permissions before trusting them with your machine, tokens and code. Use before installing an MCP server, extension or skill pack, when configuring agent permissions, or after a supply-chain incident in the AI tooling ecosystem.
---

# Agent tooling safety

AI coding agents run with your permissions: your files, your shell, your SSH keys, your
tokens. Every MCP server, editor extension and skill you add extends that trust to
someone else's code or instructions. In 2025 alone, more than 24,000 unique secrets were
found exposed in MCP configuration files on public GitHub (see `data/security-stats.md`),
and editor extension marketplaces have served tampered builds of popular extensions.

## Before installing anything

For each MCP server, extension or skill:

- **Publisher**: is it the vendor itself, or a well-known maintainer? Check the publisher ID in the marketplace, not just the display name; look-alike publishers are common.
- **Source**: is there a public repository, and does the published package match it?
- **Activity**: a long history, real issues and real maintainers beat a fresh upload with a big README.
- **What it can do**: list the tools an MCP server exposes; read what an extension activates on; read every `SKILL.md` and any scripts a skill ships. A skill is instructions your agent will follow — read it like code.
- **Network and shell**: does it make network calls, spawn processes, or ask for broad filesystem access? Is that necessary for what it claims to do?

## Secrets in agent config

- Never commit MCP config with literal tokens. Reference environment variables instead:
  ```json
  { "mcpServers": { "github": { "command": "...", "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" } } } }
  ```
  (Exact interpolation syntax depends on the client; where unsupported, keep the config file out of git.)
- Give MCP servers the narrowest token that works: fine-grained, single-repo, read-only where possible, with an expiry.
- Add agent config folders to `.gitignore` when they hold machine-specific or secret values (`.cursor/mcp.json`, local `.claude/settings.local.json`, `.vscode/settings.json` if it has tokens).

## Agent permissions

- Prefer "ask before running" for shell commands; allowlist specific safe commands rather than allowing all.
- Deny network access by default where the client supports it; allow specific domains.
- Keep agents out of directories they don't need (home folder, `~/.ssh`, `~/.aws`, password manager exports).
- Run untrusted or experimental agents in a container, VM or cloud sandbox, not on the machine with your keys.

## After a tooling incident

When a marketplace, extension or MCP package is reported compromised:

1. Check whether you had the affected publisher, package and versions installed during the window (extension list, `npm ls -g`, MCP config).
2. Look for persistence the incident describes. A common pattern is an added SSH key: inspect `~/.ssh/authorized_keys` (Windows: `%USERPROFILE%\.ssh\authorized_keys`) for keys you did not add. Updating or uninstalling the extension does not remove what it already added.
3. Rotate any token that was present in that environment: env vars, agent config, `~/.npmrc`, git credentials, cloud CLIs.
4. Follow `leak-response` for each rotated credential.

## Procedure

1. Inventory: MCP servers (all client configs), editor extensions (`code --list-extensions --show-versions`, `cursor --list-extensions --show-versions`), installed skills and their sources.
2. For each, record publisher, source repo, last update, capabilities; flag anything that fails the checks above.
3. Scan agent config files for literal secrets (`secrets-scan`).
4. Review agent permission settings; propose a tighter configuration.
5. If the user mentions a specific incident, run the incident checklist.

## Output

An inventory table (name, type, publisher, source, capabilities, verdict), secrets found in config, and a proposed permissions config.
