---
name: setup
description: "Set up SecureNow production security in the current Node.js project: install the securenow package, sign in, connect the project to a SecureNow app, wire the runtime (Next.js, Nuxt, Express, Fastify, NestJS, Koa, Hapi or plain Node), verify telemetry, and summarize the protection baseline. Use when the user asks to add, install, connect or verify SecureNow in a project."
---

# Set up SecureNow in this project

Goal: the project sends traces and logs to SecureNow, the firewall runs in the app, and the SecureNow CLI and MCP tools are signed in. Work from the project root (the folder with `package.json`).

Ask before every step that installs a package, opens a browser sign-in, edits project files, or changes security settings. Start by telling the user what setup does and get a go-ahead.

## Cost to mention up front

Pricing: $99 per app per month, 14-day trial for the whole account, no credit card, no free tier. Each app is billed once the trial ends. During sign-in the user picks an existing app or creates one in the browser; remind them before they create a new one.

Never create apps yourself: do not call `securenow_apps_create`, and do not run `npx securenow apps create` or `npx securenow apps discover`, unless the user explicitly asks for a new app and accepts the cost.

## Steps

1. **Install the SDK and CLI.** `npm install securenow` (or the project's package manager: `pnpm add securenow`, `yarn add securenow`, `bun add securenow`).
2. **Sign in.** Run `npx securenow login` and wait while the user completes the browser flow. It writes two files:
   - `.securenow/admin.json`: sign-in for the CLI and the SecureNow MCP server
   - `.securenow/runtime.json`: SDK runtime config and the app's runtime API key

   If the user wants one sign-in shared by every project and agent tool, use `npx securenow login --global` (saved under `~/.securenow/`). If the browser flow fails, the user can copy a CLI token from the dashboard settings and run `npx securenow login --token <token>` themselves; never ask them to paste the token into the chat.
3. **Wire the runtime.** Show the user what will change, then run `npx securenow init`. It detects the framework, writes secure defaults to `.securenow/runtime.json` (tracing, logs, request body capture and the firewall turned on), and then:
   - Next.js: adds or explains the instrumentation and `next.config` changes
   - Nuxt: asks for `securenow/nuxt` in the `modules` array of `nuxt.config`
   - Express, Fastify, NestJS, Koa, Hapi or plain Node: prints the start script with the preload, `node -r securenow/register <entry>`. Apply it to `package.json` after showing the diff.

   Tell the user that with the firewall on, the app enforces the account's SecureNow blocklist on incoming requests.
4. **Keep credentials private.** Check that `.securenow/admin.json`, `.securenow/runtime.json` and any `.securenow/credentials*.json` are git-ignored (login and init add these entries) without ignoring the whole `.securenow/` folder. Never print, paste or commit their contents. Don't create a `.env` file for SecureNow; the SDK reads `.securenow/runtime.json`.
5. **Verify.** Run `npx securenow whoami`, `npx securenow doctor`, `npx securenow test-span`, then `npx securenow status`. If the SecureNow MCP tools are available, `securenow_auth_status` and `securenow_apps_list` confirm the same from the agent side.
6. **Summarize the baseline.** Which app the project reports to, which protections are on (tracing, logs, body capture, firewall), and what to look at first in the dashboard at https://app.securenow.ai.

## Production deploys

For production, `npx securenow credentials runtime --env production` creates a runtime credentials file to store as a deployment secret. Explain it; run it only if the user asks.

## Guardrails

- Don't block traffic, add rate limits, create rules or change firewall settings during setup without asking first.
- If the MCP server can read resources, `securenow://docs/npm-readme` has the full framework guide.
