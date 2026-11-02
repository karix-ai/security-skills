# Contributing

## Add or improve a skill

1. Each skill lives in `skills/<name>/SKILL.md` with YAML frontmatter:

   ```
   ---
   name: <name>            # lower-case, hyphenated, matches the folder
   description: <one sentence: what it does + when an agent should use it>
   ---
   ```

2. Write for an agent: short imperative rules, a numbered procedure, an explicit output format.
3. Every command must be tested. Say which tool version it was tested with if syntax changed between versions.
4. Any statistic must have a source; add it to `data/security-stats.md` in the same PR.
5. Defensive only: detection, prevention, response. No exploitation techniques.

## Validate locally

```bash
npx skills add . --list
```

## What we want

- More providers in the `leak-response` rotation table (with the exact UI path)
- Header config snippets for more hosts and frameworks in `website-security`
- Incident write-ups for AI tooling supply-chain events, with sources, for `agent-tooling-safety`
- Translations of the checklists

## What we don't merge

- Offensive tooling, exploit code, or instructions for attacking systems you don't own
- Skills that are a pitch for a paid product
