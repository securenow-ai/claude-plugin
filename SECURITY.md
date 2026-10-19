# Security

## Reporting a vulnerability

Email contact@securenow.ai with "Security" in the subject, a description of the issue, steps to reproduce, and its impact. Please don't open a public GitHub issue for a vulnerability. We'll reply and keep you updated until it's fixed.

In scope: this plugin's skills and manifests, the hosted MCP server at `https://api.securenow.ai/mcp`, its OAuth endpoints, and the consent and Connected apps pages at app.securenow.ai.

## How the plugin is built to be safe

- **No local code.** The plugin has no hooks or scripts; its connector is a remote HTTPS server signed in with OAuth 2.1 (PKCE S256, rotating refresh tokens with reuse detection, tokens stored hashed).
- **Least privilege.** Each connection reaches one organization, at the access level the person chose, capped by their current role on every request.
- **Changes need two confirmations.** The skills preview each change and wait for the person's "yes"; the server additionally rejects any write that lacks an explicit confirmation flag and a reason.
- **Untrusted input stays data.** Skills treat request paths, payloads, headers, user agents, log lines and alert text as attacker-controlled: quoted as evidence, never followed as instructions.

More: https://securenow.ai/security
