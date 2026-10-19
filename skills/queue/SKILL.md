---
name: queue
description: "Work the SecureNow approval queue (Requires Human): check each pending decision, such as a proposed IP block, a suspected false positive, rule tuning or a case-level action, against trace evidence, recommend approve, modify or reject, and record only the decisions the user approves. Use when the user asks to review pending approvals or the human action queue."
---

# Work the SecureNow approval queue

The queue holds decisions SecureNow prepared but did not execute: IP blocks below the auto-block threshold, suspected false positives, rule tuning, and case-level actions. You check the evidence; the user decides.

## 1. Load the queue

Most urgent first: `securenow_human_actions_grouped_list` (root-cause groups with a batch-safety preview) or `securenow_human_actions_list`.

CLI: `npx securenow human list`, then `npx securenow human show <row>`.

## 2. Check each row or group

- Read the full report: `securenow_human_action_report` (investigation steps, proofs, SecureNow's proposed decision) and `securenow_notifications_get` for the case.
- Check it against raw evidence: `securenow_ip_traces`, `securenow_traces_show`, `securenow_logs_for_trace`, `securenow_ip_lookup`.
- Reject vague proposals that lack a concrete rule id, route, method, status code, or matching samples.

## 3. Recommend

For each row: approve, modify or reject, with the evidence, the exact effect (what gets blocked, excluded or closed, and its scope), and how to undo it.

## 4. Record only what the user approves

Execute one decision at a time, or a whole group only when the user approved that group. Each write needs `confirm: true` and a `reason` that cites the evidence:

- Approve a block: `securenow_human_action_block`
- Scoped false positive: `securenow_human_action_false_positive`
- Close with no enforcement: `securenow_human_action_close_no_write`
- Add an audit note without changing status: `securenow_human_action_decision_report_add`
- Case-level proposed actions such as `tune_rule` or `create_exclusion`: `securenow_human_case_action_update`

CLI equivalents: `npx securenow human block <row>`, `human fp <row>`, `human report <row>` and `human action <row>`, each with `--reason "..."`. Pass `--yes` only after the user approved that exact decision.

## 5. Alert review queue

Noisy or paused alert rules wait in a separate queue: `securenow_alert_review_runs_list`, `securenow_alert_review_run_get`, `securenow_alert_review_group_similar`. Decisions there (`securenow_alert_review_run_resolve`, `securenow_alert_review_run_dismiss`, `securenow_alert_review_run_keep_alive`, `securenow_alert_review_run_apply_group_decision`) follow the same rule: recommend, then act only on the user's yes.

## 6. Summarize

What was decided, what was left open and why, and any rule that keeps generating noise.

The SecureNow MCP server also offers `work_human_actions` and `work_alert_review` prompts. Never pass `confirmWrites: true` to them unless the user explicitly asks for autonomous execution.
