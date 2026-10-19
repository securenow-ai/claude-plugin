# Data handling

## What the plugin reads

Through the SecureNow connector, and only within the organization picked when connecting, at the person's current role:

- Applications and their status (names, keys, last data times, firewall state).
- Security alerts and their investigations: rule, severity, IPs, sample request rows, AI verdicts and proposed actions.
- Request telemetry: traces (method, route, status code, timing, client IP, user agent, and request bodies when body capture is on) and logs, kept by SecureNow for 7 days.
- IP intelligence: reputation, network and country for an IP.
- Enforcement state: blocklist, pending blocks, rate limits, trusted IPs, automations and detection rules.

The skills read what a question needs, not everything: each skill states its call budget and the tools it uses.

## What the plugin writes

Only with Investigate and respond access, only after a preview and the person's "yes" in the next message, one change per "yes":

- Blocks and unblocks, rate limits, trusted IPs, and turning an app's firewall on.
- Approval-queue decisions (approve, change, reject, close) with a decision report.
- Rule tuning: exclusions, throttle, pause, rollback, and closing alert-review rows.

Every change is recorded in the organization's audit log under the person who approved the connection. Forensic queries create a query job in SecureNow but change nothing in the person's apps.

## What leaves SecureNow

Tool results go to the AI app the person connected (Claude, Codex, Cursor), under that app's own data terms. The skills keep request bodies, headers, user emails and account ids out of the shareable weekly report. The plugin keeps no memories, schedules or caches of its own.

## What SecureNow receives from the AI app

Tool calls and their arguments (for example an IP to look up or a plain-English forensic question), logged with redacted arguments for security and auditing. SecureNow doesn't receive the conversation. The `audit` skill reads source code inside the person's session; only the detection queries they choose to validate or create are sent.

## Retention and deletion

- Telemetry: 7 days.
- Alerts, blocks, decisions and audit logs: for the life of the account.
- Connector tokens: access tokens expire after 1 hour, refresh tokens after 30 days without use; disconnecting revokes both at once.
- Account deletion removes the account's data; see the privacy policy.

Full policy: https://securenow.ai/privacy-policy. Questions: contact@securenow.ai.
