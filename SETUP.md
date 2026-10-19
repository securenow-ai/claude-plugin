# Setting up SecureNow for your team

This page is for whoever administers SecureNow and the AI tools your team uses. Individual users only need the quick start in the [README](README.md).

## How the connection works

The plugin connects to the hosted SecureNow MCP server at `https://api.securenow.ai/mcp` with OAuth 2.1. When someone connects:

1. Their AI app sends them to app.securenow.ai to sign in (email code, Google, or their existing session).
2. On the consent screen they pick one SecureNow organization and an access level:
   - **Investigate**: read-only. Alerts, traces, logs, IP intelligence, rules, queues and forensic queries.
   - **Investigate and respond**: also blocks, unblocks, rate limits, trusted IPs, rule and automation changes, and approval-queue decisions.
3. The app receives a short-lived access token (1 hour) and a refresh token (30 days, rotated on every use). Tokens are stored hashed on SecureNow's side.

A connection reaches only the organization picked on the consent screen. To use another organization, connect again and pick it.

## Roles cap what a connection can do

On every request, SecureNow intersects the connection's access level with the person's current role in that organization. A role change or a removal takes effect on their next request.

| Role | Read data | Forensic queries | Block, rate-limit, tune rules, decide approvals |
| --- | --- | --- | --- |
| Owner, Admin | Yes | Yes | Yes, with Investigate and respond |
| Security Analyst | Yes | Yes | Yes, with Investigate and respond |
| Developer | Yes | No | No |
| Viewer | Yes | No | No |
| Billing | No | No | No |

Manage roles in app.securenow.ai under the organization's member settings.

## Disconnecting and auditing

- Each person sees their own connected apps at app.securenow.ai > **Settings > Connected apps**, with the organization, the access level and when it was last used. **Disconnect** revokes the tokens at once.
- Every change made through the connector is written to the organization's audit log under the person who approved the connection, marked as coming from an AI app, with the tool's reason text.
- Removing someone from the organization cuts off their connections to it on their next request.

## Before your team starts

- **Instrument at least one app.** Briefs and investigations read telemetry that the `securenow` package sends from your app. Run the `setup` skill in a coding agent, or follow https://securenow.ai/docs.
- **Add detection rules.** New accounts start with none, so nothing alerts until you add some. The `audit` skill turns a code review into rules for your actual routes; rules start in test mode, where they alert but can't trigger blocks or other automations until promoted.
- **Check each app's firewall.** Blocks are enforced inside your app by the SecureNow package; an app whose firewall is off records blocks but doesn't enforce them. The `get-started` skill shows the state per app.
- **Decide who responds.** Give the Security Analyst role to people who should approve blocks and tune rules; everyone else can connect with Investigate.

## Claude Team and Enterprise

Workspace owners may need to add the SecureNow connector (`https://api.securenow.ai/mcp`) before members can connect it from the plugin. The connector supports Client ID Metadata Documents and Dynamic Client Registration, so no client ID needs to be configured.

## Billing

$99 per app per month, after a 14-day trial for the whole account. The skills never create an app unless someone explicitly asks for one and accepts the cost.
