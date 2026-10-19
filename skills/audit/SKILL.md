---
name: audit
description: "Security-audit this codebase, then map each confirmed finding to a SecureNow production detection that watches for exploitation of that exact route or pattern. Covers authentication, authorization (IDOR/BOLA), injection, input validation, uploads, secrets and rate-limit gaps. Use when the user asks for a security review, threat model or audit of their code with production coverage for what is found."
---

# Code security audit, mapped to production detections

Find it in code. Watch it in production.

The audit runs in this coding agent session. SecureNow's servers don't receive the source code; the only audit output sent to SecureNow is the detection SQL that the user validates or creates in step 3.

## 1. Scope

Authentication and sessions, authorization (IDOR/BOLA), injection (SQL, NoSQL, command, template), input validation, file upload and download, secrets handling, rate-limit and abuse gaps, and framework-specific risks. If the user named an area, focus there.

For a deep pass on one category, use SecureNow's threat-modeling prompt for it. `npx securenow threat-model --list` shows the 25 categories; `npx securenow threat-model <category> --out threat-prompt.md` saves one prompt (fetched from securenow.ai) to a file to follow. The prompts are long, so save them to a file instead of printing them, and don't commit the file. If the installed CLI doesn't have the command yet, the same prompts are at https://securenow.ai/threat-modeling.

## 2. Confirm findings in the code

Read the code before claiming a finding. Each finding needs a file and line reference, the affected route or function, and a concrete exploitation scenario. Drop anything you can't support.

## 3. For each confirmed finding

- **Code fix:** propose it; apply it only if the user asks.
- **Production watch:** a SecureNow detection for exploitation attempts against that exact route or pattern.
  1. Check existing coverage first: `securenow_alert_rules_list`, `securenow_alert_rule_get`.
  2. Draft the detection SQL. `npx securenow telemetry schema` lists the trace and log columns.
  3. Validate it read-only: `npx securenow alerts rules validate --sql @rule.sql --app <appKey>`.
  4. Create it only after the user approves: `npx securenow alerts rules create --name "..." --sql @rule.sql --apps <appKey>`.

  New rules start in test mode: they detect and notify but don't block. Promoting a rule so it can act (`npx securenow alerts rules promote <id>`) is a separate decision for the user.

## 4. Summary

Finish with a prioritized list: fix now, watch in production, and fine as is.
