---
name: codex-adapter
description: How to run this plugin's workflow skills (start-init, dev, tester, audit, identity) inside OpenAI Codex — tool translation from the Claude Code wording, question behaviour, skill discovery, Notati MCP discovery and the private HTTP fallback, and per-project MCP registration without putting the secret URL in the repository. Load it at the start of any workflow skill when running in Codex.
user-invocable: false
---

# codex-adapter — running the workflows in Codex

The workflow skills are written once, in Claude Code wording. **Do not create a separate Codex version** of any workflow: read the selected skill on every invocation and execute it with the translations below. A later edit to a workflow must apply to both agents.

## 1. Shared files and authority

- Follow the project's `AGENTS.md` and the priority of `docs/PROJECT_MEMORY.md`.
- Read and edit the **same** project files as Claude (`docs/PROJECT_CONFIG.md`, phase files, `notati-map.md`, memory, lessons, audit log, ADRs, source, tests). Never create Codex-specific task states, memories, lessons or logs. Never depend on any agent's automatic or private memory.
- Preserve the exact status literals from the config and the evidence prefixes (`🧪 إثبات التنفيذ`, `✔️ اختبار`, `🛡️ تدقيق`, `🎨 هوية`). You may add `(Codex)` after the date inside the parentheses.
- The `tester` closing delegation applies identically in Codex. `dev` still ends at review; `audit` keeps completion unless a finding blocks the task.

## 2. Tool translation

| Claude wording | In Codex |
|---|---|
| `$ARGUMENTS` | The text following the skill mention in the user's message (e.g. `$<plugin>:dev P3` → `P3`). Not a shell variable. |
| `Read`, `Glob`, `Grep` | Read UTF-8 files; `rg` / `rg --files`. Resolve project paths from the repository root even when started in a subdirectory. |
| `Edit`, `Write` | `apply_patch` (or the available edit tool). |
| `AskUserQuestion` | Codex's question tool if available. **Same behaviour as Claude:** ask once at the allowed point, max questions as the skill says, recommendation first. If no question tool exists, ask the same questions as plain text and **wait for the answer** before continuing. Never infer approval or a business decision from silence. |
| `Skill` / "load the X skill" | Find and read that skill's `SKILL.md` (this plugin's skills are siblings: `../<name>/SKILL.md`). For stack skills from `stack_skills`, use them only if they are actually available. Claude-only plugins are not Codex tools — never claim an unavailable tool ran. |
| `${CLAUDE_PLUGIN_ROOT}` / `${CLAUDE_SKILL_DIR}` | The plugin root / the current skill's folder, as installed in Codex. |
| MCP tool names (`mcp__<server>__get_context`...) | Discover tools by capability (the Notati server exposes `get_context`, `get_task`, `update_task`, ...). Do not assume Claude's literal names. |
| Stack MCP tools (analyzers, test runners) | Prefer them when available; otherwise run the configured `verify` commands in the shell and keep their output. |
| Artifact publishing (identity preview) | Not available: save the HTML preview to `identity.preview_file` and give the path. |

Follow the current host's permissions. The workflows' "do not stop" rules never authorize inventing missing decisions or marking blocked requirements complete.

## 3. Notati in Codex

1. Discover the Notati tools of the server named `notati.server_codex` (or `notati.server` when that is empty).
2. Run the project guard from the `notati-sync` skill (`get_context`, compare `key` + `project_id`). Mismatch → stop before any write.
3. Keep every `notati-sync` rule: `append_description` only, checklist ids, exact status ids from the config, `ignored_tasks`, phase file wins.

### 3.1 HTTP fallback (tools not loaded in this session)

Use only when the native tools are missing and the owner has registered the server:

1. Read the URL **privately** from the Codex MCP configuration (project `.codex/config.toml`, else the user's `~/.codex/config.toml`) for the configured server name. Never print it or copy it outside that MCP configuration into another project file, script, or log; never pass it to a command whose output you show.
2. POST JSON-RPC `tools/call` with the tool name and arguments. Start with `get_context` and run the guard.
3. Treat a call as successful only if there is no JSON-RPC `error` **and** `result.isError` is not true.
4. If tool arguments are unclear, read the endpoint's own GET guide (without echoing the URL).
5. If neither native tools nor the fallback work: say so at the start, continue on files only.

### 3.2 Registering the server per project (manual, URL never shown)

Codex has no project-scoped `mcp add` and does not expand environment variables inside a server `url`. The supported per-project option is a **git-ignored** project config in a **trusted** project:

1. Make sure `.gitignore` contains `.codex/config.toml` (the `start-init` skill adds it).
2. The user opens `<project>/.codex/config.toml` and adds:
   ```toml
   [mcp_servers.notati]
   url = "<paste the MCP URL here>"
   ```
3. The project must be trusted in Codex (Codex asks on first run, or `trust_level = "trusted"` for this path in the user's `~/.codex/config.toml`).
4. Start a new Codex session so the server loads.

Alternative (global): `codex mcp add notati-<key> --url "<URL>"` writes to the user's `~/.codex/config.toml` for all projects. Then set `notati.server_codex: "notati-<key>"` in the project config so each project talks only to its own server.

Never write these steps with a real URL; show only the placeholder.
