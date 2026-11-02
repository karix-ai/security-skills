# Security statistics and sources

Numbers used in the README and skills, with where they come from. Corrections with a source are welcome.

| Claim | Value | Source |
|---|---|---|
| New hardcoded secrets in public GitHub commits, 2025 | 28.65 million (+34% year over year) | GitGuardian, State of Secrets Sprawl 2026 |
| Public GitHub commits, 2025 | ~1.94 billion (+43%) | same |
| AI-service secrets leaked, 2025 | 1,275,105 (+81%) | same |
| Secret-leak rate in Claude Code-assisted commits vs. all public commits | 3.2% vs. 1.5% | same |
| Unique secrets exposed in MCP configuration files | 24,008, of which 2,117 valid | same |
| Internal vs. public repos | internal ~6× more likely to contain hardcoded secrets | same |
| Incidents originating outside code (Slack, Jira, Confluence) | ~28% | same |
| Credentials leaked in 2022 still valid | ~70% in Jan 2025, >64% in Jan 2026 | same |
| Compromised developer machines in the Shai-Hulud 2 dataset that were CI/CD runners | 59% | same |

Sources:
- GitGuardian, *The State of Secrets Sprawl 2026*: https://gitguardian.com/state-of-secrets-sprawl-report-2026
- Summary of the report's figures: https://dev.to/gitguardian/the-state-of-secrets-sprawl-2026-ai-service-leaks-surge-81-and-29m-secrets-hit-public-github-2bgj
- The Hacker News, *The State of Secrets Sprawl 2026: 9 takeaways*: https://thehackernews.com/2026/03/the-state-of-secrets-sprawl-2026-9.html

Figures as of November 2026.
