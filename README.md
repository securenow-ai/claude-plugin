# SecureNow plugin for Claude Code, Codex and Cursor

[SecureNow](https://securenow.ai) is production security for Node.js apps, built for coding agents. This plugin gives your coding agent your app's live security data (attacks, traces, logs, IP intelligence and detections) and SecureNow's response controls through the SecureNow MCP server, plus five skills for the most common jobs. Every block, rate limit, rule change and queue decision waits for your approval.

## What's included

| Skill | What it does |
| --- | --- |
| `setup` | Installs the `securenow` package, signs you in, connects the project to a SecureNow app, wires the runtime (Next.js, Nuxt, Express, Fastify, NestJS, Koa, Hapi or plain Node) and verifies telemetry. |
| `investigate` | Investigates an IP, route, account, alert or time window with traces, logs and IP intelligence, gives a confidence-rated verdict, and proposes responses. |
| `patrol` | Reviews recent detections, the alert review queue, pending approvals and the enforcement already in place, and reports what needs a decision. |
| `queue` | Works the Requires Human approval queue: checks each pending decision against trace evidence and records only the decisions you approve. |
| `audit` | Security-audits your code and maps each confirmed finding to a SecureNow detection for that exact route. Find it in code. Watch it in production. |

Ask in plain language ("who is attacking /login?") or call a skill directly: `/securenow:investigate` in Claude Code, `/skills` or `$` in Codex, `/` in Cursor chat.

The plugin also starts the **SecureNow MCP server** on your machine (`npx -y securenow@8.12.4 mcp`, over stdio). It provides 90 tools: 44 read-only tools for traces, logs, IP intelligence, detections, queues, rules and blocklist state, and 46 write tools for blocks, rate limits, rules, exclusions, trusted IPs and queue decisions. Write tools refuse to run without an explicit `confirm: true` and a reason.

## Before you start

- **A SecureNow account.** Pricing: $99 per app per month, 14-day trial for the whole account, no credit card, no free tier.
- **Node.js with `npx`**, which runs the MCP server and the `securenow` CLI.
- **Sign in once.** Run `npx securenow login` in the project you open in your agent; it saves credentials in that project's `.securenow/` folder. To share one sign-in across projects and agent tools, run `npx securenow login --global` instead (saved in `~/.securenow/`). The `setup` skill can walk you through it.

## Install

### Claude Code

Inside a Claude Code session:

```text
/plugin marketplace add securenow-ai/claude-plugin
/plugin install securenow@securenow
```

On Claude Code 2.1.275 or later you can do both in one step: `/plugin install securenow --marketplace securenow-ai/claude-plugin`.

From your shell:

```bash
claude plugin marketplace add securenow-ai/claude-plugin
claude plugin install securenow@securenow
```

On Claude Code 2.1.292 or later, `claude plugin install securenow --marketplace securenow-ai/claude-plugin` does both. Get new versions with `claude plugin update securenow@securenow`.

### Codex

```bash
codex plugin marketplace add securenow-ai/claude-plugin
```

Then start `codex`, open `/plugins`, and install SecureNow from the SecureNow marketplace, or run `codex plugin add securenow@securenow`. Start a new session to use the skills and MCP tools. Codex in the ChatGPT desktop app lists configured marketplaces as sources in its Plugins Directory.

### Cursor

- From the Cursor Marketplace, once SecureNow is listed there: open **Customize**, search for SecureNow, and select **Install**.
- From this repository: clone it into your local plugins folder, then run **Developer: Reload Window** and check **Customize** for the SecureNow skills and MCP server.

  ```bash
  git clone https://github.com/securenow-ai/claude-plugin ~/.cursor/plugins/local/securenow
  ```

- On Teams and Enterprise plans, an admin can import this repository as a team marketplace under **Dashboard > Plugins & MCPs > Import from Repo**.

### MCP server only

Any MCP client can run the server without the plugin's skills. For example, in `.cursor/mcp.json`:

```json
{ "mcpServers": { "securenow": { "command": "npx", "args": ["-y", "securenow@8.12.4", "mcp"] } } }
```

## Security and privacy

- **Writes need your approval.** The skills tell the agent to show you the exact change, its scope and how to undo it, then wait for your yes before any block, unblock, rate limit, rule, exclusion, trusted IP, firewall setting or queue decision. The MCP write tools additionally require `confirm: true` and a reason. Every app is billed per month after the trial, so the skills never create an app without your explicit confirmation.
- **What runs.** The plugin's only component that runs on its own is the MCP server: `npx -y securenow@8.12.4 mcp` downloads that exact version of the `securenow` package from the npm registry and runs it locally. The skills may ask the agent to run `securenow` CLI commands in your terminal, such as `npx securenow login` or `npx securenow doctor`, plus any change you approve. The plugin has no hooks and no scripts of its own.
- **What is sent.** The MCP server and the CLI call the SecureNow API at `https://api.securenow.ai` with the credentials that `npx securenow login` saved in `.securenow/admin.json` (or `~/.securenow/admin.json`). They don't read credentials from environment variables. `npx securenow threat-model` fetches prompt text from `https://securenow.ai`. The `audit` skill reads your code inside your agent session and doesn't send it to SecureNow; only the detection queries you choose to validate or create go to the SecureNow API.
- **Credentials stay local.** Keep `.securenow/admin.json` and `.securenow/runtime.json` out of version control. The skills check your `.gitignore` and never print their contents.

More detail: [Security](https://securenow.ai/security), [Privacy policy](https://securenow.ai/privacy-policy), [Terms of service](https://securenow.ai/tos).

## Support

Questions, bugs and security reports: contact@securenow.ai. Documentation: https://securenow.ai/cli+mcp

## License

ISC. See [LICENSE](LICENSE).
