---
name: patrol
description: "Run a SecureNow security patrol: review recent detections, the alert review queue, pending approvals and enforcement already in place, separate real attacks from noise, and propose responses for the user to approve. Use for a periodic production security check or a what-happened-since-last-time summary."
---

# SecureNow security patrol

A read-first pass over the account's production security. Nothing changes unless the user approves it.

## 1. Snapshot (read-only)

- Detections: `securenow_notifications_unread`, `securenow_notifications_list`
- Alert review queue, noisy or paused rules grouped by root cause: `securenow_alert_review_runs_count`, `securenow_alert_review_runs_list`
- Requires Human approval queue: `securenow_human_actions_grouped_list`, or `securenow_human_actions_list`
- Enforcement in place: `securenow_blocklist_stats`, `securenow_blocklist_pending_list`, `securenow_rate_limit_list`, `securenow_firewall_apps`

CLI fallback: `npx securenow status`, `npx securenow notifications list`, `npx securenow human list`, `npx securenow blocklist list`.

## 2. Triage

For each significant item, check the evidence before judging: `securenow_notifications_get`, `securenow_ip_lookup`, `securenow_traces_list`, `securenow_traces_show`, `securenow_logs_for_trace`. Sort items into real attack activity, noise, and already handled.

## 3. Report

- What happened since the last patrol, or in the window the user named
- What existing blocks, rate limits and rules already handled
- What needs a decision, most urgent first, each with its evidence and a recommendation
- Rules that keep producing noise and are worth tuning

## 4. Propose, then wait

Propose concrete actions: block, rate limit, tune or pause a noisy rule, add an exclusion, or create a missing detection. Execute nothing until the user approves each action in this conversation. Then use the matching write tool with `confirm: true` and a `reason` that cites the evidence, and verify the result.

Scheduled or unattended patrols only report. The SecureNow MCP server also offers a `soc_patrol` prompt; never pass `confirmWrites: true` to it unless the user explicitly set up autonomous decisions. To decide approval-queue items one by one, use the SecureNow `queue` skill.
