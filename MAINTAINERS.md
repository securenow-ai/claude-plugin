# Maintaining the SecureNow plugin

This repository is published from the `integrations/claude-code-plugin/` folder of SecureNow's main repository. Make changes there; a direct edit here is overwritten by the next sync. The plugin lives at the repository root.

## Which tool reads which file

| Path | Read by |
| --- | --- |
| `.claude-plugin/plugin.json` | Claude Code manifest. Codex and Cursor fall back to it when their own manifest is missing. Also carries the Anthropic directory listing fields. |
| `.claude-plugin/marketplace.json` | Claude Code marketplace (`/plugin marketplace add securenow-ai/claude-plugin`). Codex and Cursor read it as a fallback marketplace. |
| `.codex-plugin/plugin.json` | Codex manifest. Codex prefers it over the Claude manifest; it holds the `interface` metadata shown in the Codex plugin browser. |
| `.agents/plugins/marketplace.json` | Codex marketplace (`codex plugin marketplace add securenow-ai/claude-plugin`). |
| `.cursor-plugin/plugin.json` | Cursor manifest. Points `mcpServers` at `.mcp.json`. |
| `.cursor-plugin/marketplace.json` | Cursor marketplace, used when a team imports this repository as a team marketplace. |
| `.mcp.json` | The MCP server for all three tools. The only place the server version is pinned. |
| `skills/<name>/SKILL.md` | The five skills, shared by all three tools. |
| assets folder | The listing logo, referenced by the three plugin manifests. |
| `README.md`, `LICENSE` | Listing description and the ISC license that the directories require. |

Discovery order, checked against the Codex source and Cursor 3.11 on 2026-10-19:

- **Codex** manifests: a root `plugin.json` that declares the agent-plugins.org `$schema`, then `.codex-plugin/`, then `.claude-plugin/`, then `.cursor-plugin/`. Marketplaces: `.agents/plugins/marketplace.json`, then `.claude-plugin/marketplace.json`, then `.cursor-plugin/marketplace.json`. MCP config for these manifests: `.mcp.json`.
- **Cursor** manifests: `.cursor-plugin/plugin.json`, then `.claude-plugin/plugin.json`. MCP config: `.mcp.json`, then `mcp.json`.

Don't add a root `plugin.json` with the agent-plugins.org `$schema` unless you migrate to that format completely: Codex and Cursor would prefer it and would then expect a root `mcp.json` with a transport `type` on each server.

## Rules for skills

- Frontmatter holds only `name` and `description`, the subset every tool and the Agent Skills spec accept. Write the description as one double-quoted string.
- Keep the text provider-neutral: no Claude-only syntax such as `$ARGUMENTS` (Claude Code appends arguments on its own; Codex and Cursor don't expand it).
- Every `securenow_*` tool name must exist in the MCP catalog of the pinned `securenow` version.
- Every write (block, unblock, rate limit, rule, exclusion, trusted or allowlist entry, firewall setting, queue decision, app creation) must tell the agent to ask the user first. App creation costs $99 per app per month after the trial.

## Release checklist

1. Set the same new `version` in `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json` and `.cursor-plugin/plugin.json`. Claude Code keeps users on their installed copy until this value changes.
2. When moving to a new `securenow` npm release, update the pin in `.mcp.json` and the two examples in `README.md` (`grep -rn "securenow@8" .` finds them all), and the tool counts in `README.md` if the catalog changed. Never use a range or `@latest`: the Anthropic directory blocks unpinned launchers.
3. From this folder in the main repository, check that the skills only name real MCP tools:

   ```bash
   node -e "const fs=require('fs');const c=require('../../securenow-npm/mcp/catalog.js');const k=new Set([...c.TOOLS,...c.PROMPTS].map(t=>t.name));let bad=0;for(const d of fs.readdirSync('skills')){const s=fs.readFileSync('skills/'+d+'/SKILL.md','utf8');for(const m of new Set(s.match(/securenow_[a-z_]+/g)||[]))if(!k.has(m)){bad++;console.log('unknown tool',d,m)}}console.log(bad?'FAIL':'ok')"
   ```

4. Validate:
   - Claude Code: `claude plugin validate . --strict` must print `Validation passed`.
   - Cursor: run `scripts/validate-template.mjs` from the `cursor/plugin-template` repository with this folder as the working directory.
   - Codex: with `CODEX_HOME` pointing at an empty temporary folder, run `codex plugin marketplace add <this folder>`, then `codex plugin add securenow@securenow`, then `codex mcp list`.
   - Load the plugin in each tool and run every skill once against a real project.
5. Sync the folder to the root of `securenow-ai/claude-plugin` and push.

## Listings

- **Anthropic directory**: submit from https://claude.ai/directory/manage (paid claude.ai plan), choose Plugin bundle, enter `securenow-ai/claude-plugin`, run Validate and fix anything marked Blocking. The repository must be public before the listing goes live. A launcher that runs a pinned npm package is always held for a reviewer, so expect a review step. The claude.ai web app ignores local MCP servers; there users get the skills only.
- **Cursor Marketplace**: submit the repository link at https://cursor.com/marketplace/publish. Plugins must be open source with a permissive license, and every update is reviewed.
- **OpenAI plugin directory** (ChatGPT and Codex): the public directory takes a remote HTTPS MCP endpoint with OAuth, a skills-only upload, or the desktop-only route for local MCP servers. Until SecureNow's hosted MCP endpoint is ready, users install through the repository marketplace (`codex plugin marketplace add securenow-ai/claude-plugin`), which needs no listing.

## Known caveats

- The MCP server reads `.securenow/admin.json` from its working directory and falls back to `~/.securenow/`. Claude Code starts plugin servers in the project folder. If a tool starts the server somewhere else, users need `npx securenow login --global`.
- Plugin skills are namespaced in Claude Code (`/securenow:setup`). Other tools may show the bare skill name.
