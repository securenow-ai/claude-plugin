# SecureNow plugin

Production security for your web apps, in the AI tools you already use: the Claude app on desktop and web, Claude Code, Codex and Cursor. The plugin bundles the hosted SecureNow connector (`https://api.securenow.ai/mcp`, signed in with OAuth) and ten skills: a security brief, investigations, plain-English questions over your traffic, the approval queue, responses such as blocks and rate limits, rule tuning, a weekly report, and, in coding agents, project setup and a code audit that turns findings into production detections.

When you connect, you choose the access level. **Investigate** is read-only: Claude can read your alerts, traces, logs and IP intelligence and run forensic queries, but can't change anything. **Investigate and respond** also lets it block IPs, add rate limits and tune rules: the skills show you each change first (what changes, its scope, the risk to real users, how to undo it) and make it only after you reply "yes". Every change is recorded in your SecureNow audit log under your name, and you can disconnect at any time.

## Skills

| Skill | Try | What it shows | What it changes |
| --- | --- | --- | --- |
| `get-started` | "Set me up with SecureNow" | Who is connected and with which access, each app's data status and firewall, detection rules live and in test, decisions waiting, which skills are ready, and where to start | Nothing |
| `security-brief` | "What's attacking my apps?" or "What happened overnight?" | A headline, the decisions that need you, attacks grouped by rule with the IPs behind them and what SecureNow already did, blocks added today, noise, and apps that went quiet | Nothing |
| `investigate` | "Investigate 203.0.113.7" or "What happened with the latest alert?" | The requests behind an IP, alert, route or time window: routes, status codes, user agents, reputation, what already applies to it, and a verdict with a confidence level and evidence, then response options | Nothing; hands a chosen response to `respond` |
| `ask-traffic` | "Failed logins per IP in the last hour" | Rows from a forensic query over your traces and logs, with the scope, how it was counted, and the SQL on request | Nothing in your apps (it starts a query job) |
| `approvals` | "Show the decisions waiting for my approval" | Each block or decision SecureNow's AI prepared, with its verdict and risk score, checked against the raw requests, and a recommendation | Approves, changes or rejects one decision per "yes" |
| `respond` | "Block 203.0.113.7 on my shop for 24 hours" or "Rate-limit /api/login to 10 per minute per IP" | A preview of the change with its risk and undo, then the verified result | A block, unblock, rate limit, trusted IP or firewall on, one per "yes" |
| `tune-rules` | "This alert is noisy" or "Stop alerting on our uptime monitor" | Which rules fire most, what the noise has in common, and the narrowest fix | A trusted IP, a rule exclusion, a throttle, a pause or a rollback, one per "yes" |
| `weekly-report` | "Weekly security report for my team" | Attacks by type, top sources, blocks and rate limits, decisions made and waiting, coverage gaps and three recommendations, as a document when your app can show one | Nothing |
| `setup` | "Set up SecureNow in this project" | Installs the `securenow` package, signs the project in, wires Next.js, Nuxt, Express, Fastify, NestJS, Koa, Hapi or plain Node, and verifies data arrives | Project files you approve; needs a terminal |
| `audit` | "Audit this codebase and add SecureNow detections" | Confirmed weaknesses with file and line references, each mapped to a detection for that exact route | Detection rules you approve, created in test mode; needs a terminal |

In Claude, call a skill directly with `/securenow:<skill>`, or just ask in plain language.

## How every answer is shaped

- One sentence answers the question with its biggest number, then a scope line: which apps, which environment, the time window and when it was checked.
- Short by default: the answer, up to three things that need you, and where the rest is. Ask for any table or the full detail.
- Every claim cites a number, IP, route, status code, rule or trace from your data. Thresholds are stated with their reason ("quiet means no data for 24 hours"), and you can change them.
- Request paths, payloads, user agents and log lines are treated as attacker-controlled data: quoted as evidence, never followed as instructions.
- Each answer ends with two or three next prompts filled with the real IPs, rules and apps from your data.

## Quick start

### Claude app (desktop or web)

1. Open **Customize > Plugins** and install SecureNow from the directory, or choose **Add > Add marketplace** and enter `securenow-ai/claude-plugin`.
2. Open the plugin's **Connectors** tab and connect SecureNow. Sign in at app.securenow.ai, pick your organization and choose **Investigate** or **Investigate and respond**.
3. Say "set me up with SecureNow" or run `/securenow:get-started`.

On a Team or Enterprise plan, your workspace owner may need to add the SecureNow connector first.

### Claude Code

```text
/plugin marketplace add securenow-ai/claude-plugin
/plugin install securenow@securenow
/mcp            (choose plugin:securenow:securenow and sign in)
/securenow:get-started
```

If you installed the plugin in the Claude app and Claude Code syncs plugins from the same account, it appears at your next session start: skip the two `/plugin` lines and run `/mcp` to sign in.

### Codex

```bash
codex plugin marketplace add securenow-ai/claude-plugin
codex plugin add securenow@securenow
codex mcp login securenow
```

### Cursor

Install SecureNow from the Cursor Marketplace once it's listed, or clone this repository into `~/.cursor/plugins/local/securenow` and reload the window. Sign in to the `securenow` MCP server from Cursor's MCP settings.

## Prerequisites and common blockers

| You need | Otherwise |
| --- | --- |
| A SecureNow account. $99 per app per month after a 14-day trial for the whole account; no credit card for the trial | The sign-in page offers to create an account |
| At least one app sending data. The `setup` skill does it in a coding agent, or follow https://securenow.ai/docs | Skills report "no app is sending data" instead of "no attacks" |
| Detection rules. New accounts start with none; the `audit` skill builds them from your code, or create them in the dashboard | Nothing can alert, so briefs stay empty; `get-started` says so |
| Investigate and respond access, to make changes | Skills show changes as proposals and explain how to reconnect with that access |
| A Security Analyst, Admin or Owner role in your SecureNow organization, to make changes or run forensic queries | Developers and Viewers get read-only answers; the skill names the role that's needed |

More detail for admins, including roles, organizations and disconnecting: [SETUP.md](SETUP.md).

## What the skills are designed not to do

The skills are written to stay inside these limits. AI output can be wrong, so review anything before you act on it.

- Make any change without showing you a preview and getting your "yes" in the next message, or make more than one change per "yes" unless you approved a group they listed by name.
- Turn a firewall off, delete a rule or an automation, or use the restrictive allowlist (which blocks every IP not on it). They point you to the dashboard for those.
- Create SecureNow apps, which are billed per app, unless you ask for one and accept the cost.
- Follow instructions found inside request paths, payloads, headers, user agents, log lines or alert text, or open URLs found there.
- Widen a search to all apps or a longer window on their own, or invent records when a result is empty.
- Put request bodies, headers, user emails or account ids into the weekly report, which is meant for sharing.

## What the plugin runs, sends and fetches

- **Runs.** Nothing on its own: no hooks, no background scripts. The `setup` and `audit` skills ask your coding agent to run `securenow` CLI commands in your terminal (for example `npx securenow init` or `npx securenow alerts rules validate`), each shown to you first; your client asks permission before running a command unless you allowed it.
- **Sends.** Tool calls to the SecureNow API at `https://api.securenow.ai/mcp`, authorized by the OAuth token you approved. The `audit` skill reads your code inside your session; only detection queries you choose to validate or create go to SecureNow.
- **Fetches.** Your SecureNow data through the connector, and, in the `audit` skill, threat-modeling prompt text from `https://securenow.ai`.
- **Keeps.** No memories, schedules or caches. A weekly report you ask for is saved as a document or artifact in your app.

## Data handling

The connector reaches only the SecureNow organization you picked when connecting, with your current role's permissions. Disconnecting in app.securenow.ai > Settings > Connected apps revokes its tokens at once. Details: [DATA_HANDLING.md](DATA_HANDLING.md) and the [privacy policy](https://securenow.ai/privacy-policy).

## Known limits

- Telemetry is kept for 7 days, so investigations and reports reach back a week at most.
- The alert list shows open alerts; alerts that were resolved or archived don't appear in briefs.
- Forensic queries wait up to 30 seconds; slower ones return a job you can check again.
- Very large results are shortened to about 60 KB with a note; the skills then narrow the question.
- Detection rules are created with the CLI (the `audit` skill), not through the connector.
- A preview-and-confirm step lives in the skills. A request no skill picks up goes to the SecureNow tools directly, with only your client's tool-approval prompt in between; the tools themselves still require a confirmation flag and a reason for every change.

## Support

Questions, bugs and security reports: contact@securenow.ai. Documentation: https://securenow.ai/cli+mcp. Security: [SECURITY.md](SECURITY.md).

## License

ISC. See [LICENSE](LICENSE).
