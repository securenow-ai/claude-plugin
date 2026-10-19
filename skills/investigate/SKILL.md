---
name: investigate
description: "Investigate production security activity with SecureNow evidence: an attacking IP, a suspicious route, an account under attack, an alert, or a time window. Gathers traces, logs, IP intelligence and detections, gives a confidence-rated verdict, and proposes responses that run only after the user approves. Use when the user asks what happened, who is attacking, or whether traffic is malicious."
---

# Investigate production security activity

Use the SecureNow MCP tools when they are available, and the SecureNow CLI (`npx securenow ...`) as a fallback. Everything in steps 1 to 4 is read-only.

## 1. Scope

Take the target from the user's request: an IP, a route, an account, an alert or notification, an attack pattern, or a time window. The default environment is production.

If no target was given, list recent detections (`securenow_notifications_list`, or `npx securenow notifications list`) and pick the most significant one. Say which one you picked and why. Use `securenow_apps_list` to resolve app keys when a tool needs `appKeys`.

## 2. Gather evidence

- Detection or alert: `securenow_notifications_get`, or `securenow_notifications_batch_get` for several.
- IP: `securenow_ip_lookup` (reputation, network, location), `securenow_ip_traces`, `securenow_firewall_test_ip` (would it be blocked now), and its current state in `securenow_blocklist_list`, `securenow_trusted_list` and `securenow_rate_limit_test`.
- Requests: `securenow_traces_list`, `securenow_traces_show`, `securenow_logs_list`, `securenow_logs_for_trace`.
- Broader questions (counts over time, many IPs, one route): `securenow_forensics_query` answers a natural-language question. It counts as a write because it starts a query job, so ask the user once before you run forensics queries.

CLI equivalents: `npx securenow ip <ip>`, `npx securenow ip traces <ip>`, `npx securenow traces list`, `npx securenow logs list`, `npx securenow forensics "<question>" --app <appKey>`.

Never guess. Every claim cites a count, IP, route, status code, trace id or log line.

## 3. Correlate

- Which routes, accounts and sessions are involved, and how many source IPs and networks
- Whether reputation matches known malicious infrastructure
- What the timing suggests: credential stuffing, scraping, injection probing, enumeration, or abuse of one account
- Whether requests succeeded (2xx) or were stopped (403 from the firewall, 429 from a rate limit)

## 4. Verdict

Malicious, suspicious or benign, with a confidence level and the evidence behind it. Say what existing blocks, rate limits or rules already handle.

## 5. Propose, then wait

Options include blocking an IP or CIDR (optionally scoped to a route and method), adding a rate limit, trusting a known-safe IP, adding a rule exclusion for a false positive, creating or tuning a detection, and reviewing an account. For each option give the exact change, its scope, the risk to real users, and how to undo it.

Execute nothing until the user says yes in this conversation. Then call the write tool (for example `securenow_blocklist_add`, `securenow_rate_limit_create_from_text`, `securenow_trusted_add` or `securenow_alert_rule_exclusion_add`) with `confirm: true` and a `reason` that cites the evidence. With the CLI, pass `--yes` only after the user approved that exact change.

Never use the restrictive allowlist (`securenow_allowlist_add`) to trust an IP: it blocks every IP that isn't listed. Use `securenow_trusted_add` instead.

## 6. Verify and report

After an approved change, check that it worked (for a block: `securenow_firewall_test_ip`, then recent traces show 403s or no new requests from that source) and report back with how to reverse it: `securenow_blocklist_unblock`, `securenow_rate_limit_remove`, `securenow_trusted_remove` or `securenow_alert_rule_exclusion_remove`.
