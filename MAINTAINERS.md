# Maintaining the SecureNow plugin

This repository is published from the `integrations/claude-code-plugin/` folder of SecureNow's main repository. Make changes there; a direct edit here is overwritten by the next sync. The plugin lives at the repository root.

## Which tool reads which file

| Path | Read by |
| --- | --- |
| `.claude-plugin/plugin.json` | Claude manifest (Claude app and Claude Code). Codex and Cursor fall back to it when their own manifest is missing. Its `description` is the Overview text in the Claude directory. |
| `.claude-plugin/marketplace.json` | Claude marketplace (`/plugin marketplace add securenow-ai/claude-plugin`). Codex and Cursor read it as a fallback marketplace. |
| `.codex-plugin/plugin.json` | Codex manifest, with the `interface` metadata shown in the Codex plugin browser. |
| `.agents/plugins/marketplace.json` | Codex marketplace (`codex plugin marketplace add securenow-ai/claude-plugin`). |
| `.cursor-plugin/plugin.json`, `.cursor-plugin/marketplace.json` | Cursor manifest and team marketplace. |
| `.mcp.json` | The connector for every tool: the hosted server `https://api.securenow.ai/mcp` over streamable HTTP, signed in with OAuth. Nothing runs locally. |
| `skills/<name>/SKILL.md` | The ten skills, shared by every tool. |
| `README.md`, `SETUP.md`, `DATA_HANDLING.md`, `SECURITY.md`, `LICENSE` | The listing's documentation. The Claude directory shows README.md on the Contents tab. |
| assets folder | Logos referenced by the manifests. |

## Skills

- Every skill carries the same "SecureNow rules" block between `<!-- securenow-rules:start -->` and `<!-- securenow-rules:end -->`. Its source is `integrations/plugin-tools/rules.md` in the main repository; never edit the copies. After changing it, run `node integrations/plugin-tools/skills.mjs sync`.
- Each skill follows the same outline: the rules, then its own section with a call budget, a Needs table (tool, role, what happens without it), numbered Steps, Output, Edge cases, and Next prompts.
- Frontmatter holds only `name` (the folder name) and `description` (one double-quoted string), the subset every tool and the Agent Skills spec accept.
- Keep the text provider-neutral: no Claude-only syntax such as `$ARGUMENTS`.
- Name only tools the hosted server exposes to OAuth connections (it hides `securenow_apps_create` and `securenow_alert_rule_query_update`), and only their real arguments.
- Every write goes through the preview-and-yes gate in the rules block. App creation costs $99 per app per month after the trial; no skill creates apps.

## Release checklist

1. From the main repository root, `node integrations/plugin-tools/skills.mjs check` must print `ok`: rules blocks in sync, frontmatter valid, every tool and named argument present in the hosted catalog.
2. Set the same new `version` in `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json` and `.cursor-plugin/plugin.json`. Clients keep users on their installed copy until it changes.
3. Validate:
   - Claude: `claude plugin validate . --strict` must print `Validation passed`.
   - Codex: with `CODEX_HOME` pointing at an empty temporary folder, `codex plugin marketplace add <this folder>`, `codex plugin add securenow@securenow`, then `codex mcp list` shows `securenow` at `https://api.securenow.ai/mcp`.
   - Cursor: `scripts/validate-template.mjs` from the `cursor/plugin-template` repository, run in this folder.
4. Run each skill once against a SecureNow account with data (the reviewer demo account works: see `integrations/review-demo/` in the main repository).
5. Rebuild the ChatGPT package (`node integrations/openai-plugin/build.mjs`), which reuses these skills.
6. Sync the folder to the root of `securenow-ai/claude-plugin` and push.

## Listings

- **Claude directory:** the plugin bundle (this repository) and the MCP connector (`https://api.securenow.ai/mcp`) are separate submissions at https://claude.ai/directory/manage. The directory re-scans the repository; a new version waits for review before it goes live.
- **Cursor Marketplace:** submit the repository link at https://cursor.com/marketplace/publish. Every update is reviewed.
- **OpenAI plugin directory:** the ChatGPT package from `integrations/openai-plugin/` (hosted connector plus these skills without CLI steps).
