# GitHub Dependency and Secret Scanning Test

This repository is a deliberately small Laravel application with login and a
protected dashboard. It contains dependency manifests for Composer, npm, and
GitHub Actions so GitHub can exercise its supply-chain security features.

## What Detects What

- **Dependabot alerts** detect known vulnerabilities in dependencies.
- **Dependabot version updates** open pull requests for outdated dependencies.
- **GitHub Secret Scanning** detects hardcoded credentials and tokens.
- **Push protection** may block provider-shaped credentials before they land.

The nonfunctional canaries are in
`config/secret-scanning-canary.php`. They must never be replaced with live
credentials.

The dependency manifests cover common production concerns without implementing
healthcare workflows:

- authorization and audit trails;
- typed data and filtered API queries;
- media uploads, PDF output, and S3 storage;
- Redis, error monitoring, Twilio, Slack, and OpenAI integrations;
- data tables, forms, schema validation, dates, HTTP requests, and charts.

## Repository Setup

1. Push this repository to an isolated GitHub test repository.
2. Open **Settings > Security and analysis**.
3. Enable the dependency graph, Dependabot alerts, Dependabot security updates,
   secret scanning, and push protection as available for the repository plan.
4. Open **Security** to review Dependabot and secret-scanning alerts.
5. Open **Insights > Dependency graph > Dependabot** to inspect update jobs.

Public repositories receive GitHub's provider secret scanning automatically.
Private repositories may require GitHub Secret Protection.

## Deterministic Canary Patterns

GitHub supports MySQL connection URLs as a generic pattern. Enable non-provider
patterns where available. Invalid provider canaries may be filtered or marked
inactive, so repository custom patterns can make this test deterministic.

Create these patterns in **Settings > Security and analysis > Secret scanning >
New pattern**, then test each expression before publishing it:

| Name | Secret format |
| --- | --- |
| Twilio canary SID | `AC[0-9]{32}` |
| Slack canary token | `xoxb-[0-9]{12}-[0-9]{12}-[0-9]{24}` |
| OpenAI canary key | `sk-proj-[0-9]{48}` |
| MySQL canary password | `MYSQL_CANARY_PASSWORD_[0-9]{6}` |

For the MySQL password pattern, use this additional context:

```text
Before secret: (?i)(mysql|db)[^"\n]{0,80}(password|url)[^"\n]{0,20}
```

Test the custom pattern in GitHub's pattern editor before publishing it.

## Expected Results

Provider detection and validity checks evolve over time. The Slack and OpenAI
values are shaped like provider tokens but are intentionally invalid. Twilio's
account SID and auth token are included as a credential pair. GitHub may mark
canaries as inactive, classify them differently, or require a custom pattern.

At scaffold time, `npm audit` reports a critical advisory through the direct
development dependency `concurrently` and its `shell-quote` dependency. That
locked state is intentionally retained to give Dependabot an immediate npm
alert/update scenario. Do not deploy it unchanged.

Do not use this repository as a deployment template while the canary file is
present.
